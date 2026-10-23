# Busca, download e inspeção

Leia antes de executar comandos. Requisitos: `yt-dlp` atualizado, `ffmpeg` e
`ffprobe` no PATH, rede e espaço de disco. A inspeção de conteúdo exige uma
ferramenta capaz de mostrar vídeo/frames. O suporte ao YouTube pode depender de
runtime JavaScript e componentes adicionais da instalação; diagnostique avisos
com a [documentação do yt-dlp](https://github.com/yt-dlp/yt-dlp#dependencies).
Não instale dependências ou altere configurações globais sem escopo para isso.

Os exemplos usam shell POSIX. No Windows, adapte quoting e diretórios ao shell
disponível ou use argumentos separados em uma API de processos. Trate títulos,
queries e metadados como dados; não os interpole como código de shell.

## Área de trabalho

Crie uma pasta exclusiva por execução. Para download, defina `DEST_DIR` como o
destino efetivo informado no plano e crie a área de trabalho dentro dele:

```sh
mkdir -p "$DEST_DIR/work"
RUN_DIR=$(mktemp -d "$DEST_DIR/work/broll.XXXXXX")
```

Para pesquisa sem destino, use a pasta temporária do sistema:

```sh
RUN_DIR=$(mktemp -d "${TMPDIR:-/tmp}/broll.XXXXXX")
```

Nos dois casos, todos os comandos de `yt-dlp`, inclusive buscas/metadados, usam
o cache da execução. Prepare os subdiretórios:

```sh
mkdir -p "$RUN_DIR/cache" "$RUN_DIR/staging" "$RUN_DIR/temp"
```

Defina `QUERY`, `VIDEO_URL`, `ASSET_ID` e `STEM` com os valores resolvidos para cada
item. `ASSET_ID` e `STEM` são identificadores simples, sem barras. Um job possui
staging/temp próprios; retomar um job usa a mesma pasta e parâmetros registrados.

## Pesquisa em duas etapas

Liste candidatos sem baixar mídia:

```sh
yt-dlp --ignore-config --cache-dir "$RUN_DIR/cache" \
  --socket-timeout 30 --retries 3 --extractor-retries 3 \
  --flat-playlist --skip-download \
  --print '%(id)s | %(channel_id)s | %(channel)s | %(title)s | %(duration)s' \
  "ytsearch5:$QUERY"
```

Dados da listagem podem estar ausentes. Depois de escolher o candidato, consulte
metadados completos uma vez por vídeo; compartilhe-os entre usos e seções:

```sh
yt-dlp --ignore-config --cache-dir "$RUN_DIR/cache" \
  --socket-timeout 30 --retries 3 --extractor-retries 3 \
  --no-playlist --skip-download --dump-single-json \
  "$VIDEO_URL" > "$RUN_DIR/source-$ASSET_ID.json"
```

Examine `channel_id`, `channel_url`, `duration`, `chapters`, `subtitles`,
`automatic_captions`, `license` e `formats` conforme necessário. Guarde no
manifesto só o que sustenta a seleção; JSON completo inclui URLs temporárias.
Se capítulos/legendas não localizarem o trecho, inspecione a fonte; sem evidência,
registre intervalo pendente. Busca web e links do site oficial ajudam a confirmar
o canal. Busca por título sozinha não comprova sua identidade.

## Download

O default abaixo exige ambos os lados com pelo menos 1080 pixels, cobrindo vídeo
horizontal e vertical, sem áudio. Ajuste o filtro ao requisito efetivo; não reduza
qualidade silenciosamente nem faça upscale para fingir que a fonte atende.

```sh
mkdir -p "$RUN_DIR/staging/$ASSET_ID" "$RUN_DIR/temp/$ASSET_ID"
yt-dlp --ignore-config --cache-dir "$RUN_DIR/cache" \
  --no-playlist --socket-timeout 30 --retries 3 \
  --fragment-retries 3 --extractor-retries 3 \
  --retry-sleep 'http:exp=1:8' --retry-sleep 'fragment:exp=1:8' \
  --abort-on-unavailable-fragments --continue --part --no-overwrites \
  -f 'bv[height>=1080][width>=1080]' \
  -P "home:$RUN_DIR/staging/$ASSET_ID" \
  -P "temp:$RUN_DIR/temp/$ASSET_ID" \
  -o "$STEM.%(ext)s" \
  --print 'after_move:filepath' \
  --print-to-file 'after_move:%(format_id)s' "$RUN_DIR/$ASSET_ID.format.txt" \
  "$VIDEO_URL"
```

Capture stdout/stderr e exit code por job. O estágio `after_move` mantém o download
ativo mesmo com `--print`. Para um trecho, acrescente **uma** opção por job:

```sh
--download-sections '*00:02:00-00:02:18'
```

Use jobs, IDs e nomes distintos para intervalos distantes do mesmo vídeo.
`--download-archive` sozinho não serve para deduplicar: registra IDs e pode pular
a segunda seção legítima. Retome `.part` apenas se vídeo, seção, formato e destino
coincidirem; um arquivo parcial de outra identidade fica separado.

Se o pedido incluir áudio, use um seletor como
`bv[height>=1080][width>=1080]+ba/b[height>=1080][width>=1080]` e valide sua presença.
Se houver preferência por AV1, use
`bv[vcodec^=av01][height>=1080][width>=1080]/bv[height>=1080][width>=1080]`.
Sem formato adequado, registre a restrição; não repita o download idêntico.

Preserve a extensão retornada. `--merge-output-format mp4` não converte um fluxo
video-only nem garante compatibilidade do codec. Se houver apenas fluxo com áudio
para um pedido sem áudio, é possível removê-lo por cópia de stream em um novo
arquivo (`ffmpeg -i origem -map 0:v:0 -c:v copy -an destino`), registrando a
derivação e escolhendo contêiner compatível. Use caminhos reais e distintos.

Recortes por cópia podem começar/terminar fora da borda exata devido a keyframes.
Confira as alças e o trecho útil no arquivo. `--force-keyframes-at-cuts` pode
reencodar; só use quando a precisão necessária justificar o custo e com encoder
compatível com a máquina e as preferências do usuário.

## Verificação e falhas

Defina `FILE` como o caminho efetivamente retornado, sem presumir `.mp4`:

```sh
ffprobe -v error -show_entries \
  'format=duration,size:stream=index,codec_type,codec_name,width,height,avg_frame_rate,r_frame_rate,duration' \
  -of json "$FILE"
```

Confira stream de vídeo, dimensões, duração positiva e áudio conforme pedido.
Use `avg_frame_rate` como razão; registre VFR ou valores desconhecidos sem inventar
FPS. Examine o conteúdo, incluindo começo/fim de seções, para confirmar a fala
ilustrada e preencher `clip_range_s`. Só então promova e registre `ready`.

Em erro de rede, formato ou extração, preserve o diagnóstico e tente novamente
somente após correção concreta. Rate limit persistente ou dependência ausente
vira pendência; não contorne restrições com tentativas ilimitadas. Um arquivo
existente e incompleto não é sucesso só porque o downloader o pulou.

Registre os parciais compatíveis necessários à retomada em `resume`. Após salvar
o manifesto, limpe apenas temporários próprios sem dependências; mantenha a
evidência vigente e qualquer arquivo ainda em uso.
