---
name: broll
description: Busca vídeos de apoio no YouTube a partir de um roteiro, relaciona trechos às falas e, quando solicitado, baixa e valida a mídia. Use também para completar uma seleção existente ou retomar downloads pendentes sem duplicar arquivos.
---

# B-roll a partir de um roteiro

Transforme um roteiro em uma seleção de imagens de apoio com origem verificável
e uma relação clara entre cada trecho e a fala que ele ilustra. Atenda qualquer
tema, idioma ou organização de roteiro; não exija nomes de arquivo ou rubricas
específicos. Edição e inserção na timeline ficam fora deste fluxo.

## 1. Entender o pedido e reaproveitar o que existe

Leia o roteiro completo, fornecido como arquivo ou texto na conversa, e as
instruções do projeto. Se não conseguir ler o formato, peça uma versão acessível;
não complete partes ausentes por suposição. Aproveite contexto e preferências já
informados. Pergunte apenas pelo que impedir a próxima etapa.

Distinga o escopo pelo pedido:

- **Pesquisa:** encontre candidatos, fontes e intervalos. Pedir sugestões ou
  buscar vídeos não autoriza baixar mídia. Entregue uma tabela; salve o plano
  quando solicitado ou quando o projeto já mantiver um manifesto.
- **Pesquisa e download:** pedidos para baixar, entregar arquivos ou retomar
  downloads autorizam executar esse trabalho. Apresente o plano compacto e siga
  sem repetir aprovação já dada. Resolva escolhas materiais ainda abertas em uma
  única pergunta, continuando os itens independentes.

Para downloads, use o projeto/destino indicado. Sem destino explícito, use uma
pasta `broll/` ao lado do arquivo de roteiro, informando essa escolha; se houver
apenas texto na conversa e nenhum projeto identificado, peça o destino. Resolva
symlinks antes de inventariar, evitando criar uma segunda entrega para a mesma raiz.

Defaults ajustáveis: fonte oficial, mínimo 1080p, vídeo sem áudio e codec original.
O usuário pode definir canais, material licenciado de terceiros, resolução, áudio,
formato, duração ou limites de download. Confira disponibilidade das ferramentas
em [references/yt-dlp.md](references/yt-dlp.md) antes de usar o terminal. A pesquisa
pode usar busca web/browser quando `yt-dlp` não estiver disponível.

Inventarie arquivos e seleção existentes antes de pesquisar. Ao criar ou retomar
um manifesto, leia [references/manifest.md](references/manifest.md). Preserve IDs,
nomes e registros fora do escopo. Reuse mídia íntegra e sua verificação; mudanças
no roteiro exigem revisar os usos, não baixar novamente arquivos intactos.

Identifique as indicações visuais explícitas e proponha imagens para as demais
passagens que se beneficiem delas. Cada uso registra segmento, ordem e frase
literal com contexto para desambiguar repetições. Um mesmo asset pode ilustrar
várias falas. Deduplique por vídeo + intervalo da fonte + formato/áudio, nunca só
pelo ID do YouTube. Preserve o nome de material fornecido pelo usuário.

**Saída desta etapa:** tabela `fala/segmento · imagem desejada · fonte esperada ·
reutilizar/buscar`, destino quando aplicável e lacunas do roteiro identificadas.

## 2. Encontrar a fonte e o trecho certos

Pesquise apenas os itens faltantes ou invalidados, agrupando consultas iguais.
Liste alguns candidatos antes de selecionar; os comandos estão em
[references/yt-dlp.md](references/yt-dlp.md). Use termos de ação visual, assunto,
organização e data/versão quando relevantes, não apenas o título do roteiro.

Confirme a identidade do canal da organização, criador ou detentor da fonte por
evidência verificável, como um link do site oficial para o canal. Registre URL do
vídeo, ID do canal e evidência da confirmação. Nome, título e primeiro resultado
não bastam. Se não houver fonte oficial adequada, refine com a informação ausente;
se uma nova busca não trouxer candidato útil, registre a lacuna. Use terceiros
quando o pedido permitir, identificando claramente a procedência.

Verifique se as imagens mostram a ação/produto/versão da fala. Metadados e
transcrições ajudam a localizar candidatos; só a inspeção do conteúdo confirma a
adequação visual. Se não puder inspecionar, declare essa limitação e mantenha o
trecho como candidato. Registre licença/permissão conhecida separadamente: fonte
oficial não significa autorização de reutilização.

Para fontes longas (aproximadamente 15 minutos ou mais), encontre o intervalo por
capítulos, legendas/transcrição ou inspeção dirigida. Acrescente pequenas alças
para corte e baixe só o trecho necessário. Não invente timestamps nem baixe uma
keynote inteira para contornar uma localização incerta. Intervalos distantes da
mesma fonte são assets separados; iguais podem compartilhar arquivo. Só una
intervalos sobrepostos antes do download, preservando os usos individuais.

**Saída desta etapa:** cada uso tem candidato/fonte e intervalo quando necessário,
ou uma pendência com motivo. Em pesquisa apenas, entregue a tabela com links,
timestamps confirmados ou incertezas e encerre sem baixar os vídeos.

## 3. Baixar e verificar

Siga [references/yt-dlp.md](references/yt-dlp.md) para staging, seleção de formato,
retomada e comandos. Defina o nome antes de baixar: `01-assunto.ext`, `02-assunto.ext`
ou a convenção do projeto. Use slug simples e a extensão real do contêiner. Nomes
permanecem estáveis mesmo se as falas mudarem de ordem.

Limite a concorrência conforme rede e máquina; dois jobs é um começo razoável.
Registre saída e exit code. Depois dos retries limitados, uma nova tentativa exige
diagnóstico e correção concreta. Falha persistente fica registrada por item, sem
bloquear os demais. Não rebaixe a mesma fonte esperando uma resolução inexistente.

Faça um `ffprobe` por arquivo novo/alterado: dimensões, duração, FPS racional,
codec e presença de áudio. Confira visualmente o trecho útil e as bordas dos
recortes. Se faltar ferramenta de inspeção, mantenha o arquivo em staging e a
verificação pendente; integridade técnica sozinha não torna o asset pronto.

Preserve codec e resolução. Transcodifique apenas para atender requisito de
compatibilidade; escolha encoder disponível na máquina e respeite preferências
do usuário, sem exigir hardware específico. Registre a derivação e preserve o
original necessário. Cortes sem reencode podem variar nas bordas: o intervalo
solicitado na fonte e o trecho útil observado no arquivo são campos distintos.

Promova somente arquivos aprovados. Confira colisões antes de mover: reutilize
conteúdo idêntico, dê outro nome a conteúdo diferente e preserve mídia existente.
Atualize o manifesto atomicamente, conforme a referência.

**Saída desta etapa:** cada item está pronto, pendente ou falhou com motivo; todo
arquivo pronto tem origem, propriedades, verificação e ao menos um uso completo.

## 4. Fechar e entregar

Após salvar o manifesto e conferir dependências, remova apenas temporários sem
referências criados nesta execução. Preserve originais, entregas, evidências que
sustentam a seleção e parciais retomáveis com motivo/próxima ação. Aguarde jobs
ativos antes de limpar suas pastas. Caches globais e acervos alheios ficam fora
desta limpeza.

Entregue uma tabela compacta `arquivo ou candidato · fonte · trecho ·
resolução/tamanho quando medidos · fala/segmento`, o manifesto se houver e as
pendências nominais. Diferencie candidato localizado, arquivo baixado e arquivo
validado. Uma reexecução sem mudanças deve reconhecer a entrega existente.
