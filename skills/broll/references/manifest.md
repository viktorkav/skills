# Manifesto de b-roll

Leia ao salvar uma pesquisa, baixar arquivos ou retomar uma seleção existente.
Em um projeto novo, use `manifest.json` dentro do destino efetivo dos arquivos
(`broll/manifest.json` quando adotado o destino padrão). Caminhos de mídia são
relativos à pasta do manifesto. Adapte-se ao manifesto já existente sem migrar seu esquema
silenciosamente. O contrato abaixo é o default desta skill, não uma exigência de
integração com um editor específico.

## Campos

| Campo | Conteúdo |
|---|---|
| `schema_version` | Inteiro `1`. Versões desconhecidas exigem inspeção antes de escrever. |
| `script` | `path` relativo ou `null` para texto na conversa; `sha256` da versão integral do roteiro; `text` quando não há arquivo durável. |
| `settings` | Preferências efetivas: fonte, qualidade mínima, áudio e formato/codec. |
| `assets` | Lista de registros abaixo, inclusive candidatos e lacunas. |

Cada asset:

| Campo | Conteúdo |
|---|---|
| `id` | Identificador estável; independe de nome de arquivo e posição no roteiro. |
| `status` | `candidate`, `pending`, `ready`, `failed` ou `superseded`. |
| `file` | Caminho do arquivo aprovado; `null` enquanto não houver entrega validada. |
| `source` | `video_id`, `url`, `channel_id`, `channel_url`, `official_evidence`, `range_s` solicitado na fonte (`null` para vídeo inteiro), `license` conhecida ou `null`. |
| `download` | `format_selector`, `format_id` efetivamente baixado, `audio` booleano e `derivation` para eventual conversão. |
| `signature` | `bytes`, `mtime_ns` e `sha256` quando necessário para desambiguar; `null` sem arquivo aprovado. |
| `media` | `width`, `height`, `duration_s`, `fps_num`, `fps_den`, `video_codec`, `has_audio`; `null` antes da medição. |
| `verification` | `signature` validada, `technical` e `visual` (`passed`, `pending`, `failed`), `notes` ou ponteiros para evidências. |
| `uses` | Uma ou mais intenções de uso, descritas abaixo. |
| `resume` | `null`, ou caminhos de staging/parciais, identidade do job, motivo e próxima ação. |
| `notes` | Motivo de pendência/falha ou contexto relevante. |

Um uso contém `id`, `segment`, `order`, `script_sha256` da versão que ancora e
`anchor` com `text` literal,
`context_before`, `context_after` e `occurrence` (base 1) quando houver repetição.
Use os títulos de segmento do roteiro ou rótulos simples. `clip_range_s` é o
intervalo útil semiaberto `[in, out)` no arquivo entregue; fica `null` em candidatos.
Esses segundos não são a posição na timeline nem os timestamps do vídeo-fonte.

`ready` exige arquivo existente, assinatura atual, mídia medida, verificações
técnica e visual aprovadas e ao menos um uso com intervalo útil dentro da duração.
Para FPS desconhecido, use `null` e explique; para VFR, registre a razão média e
anote a variabilidade. Não transforme taxas racionais em inteiros aproximados.
Campos ainda desconhecidos ficam `null`; não preencha medições por estimativa.
`candidate` identifica uma opção localizada, `pending` uma escolha/verificação
em aberto e `failed` uma tentativa que não se completou, sempre com motivo.

## Reuso e escrita

- A identidade de download inclui `video_id`, `range_s`, formato selecionado e
  política de áudio; derivações também incluem seus parâmetros. Vídeos com o
  mesmo ID e intervalos diferentes continuam sendo assets distintos.
- Acrescente usos a um asset existente, mantendo ID e nome. Mudanças de roteiro
  atualizam âncoras e versão do roteiro; preserve a associação de usos ainda não
  revisados com o hash anterior, sem reinterpretá-los como pertencentes ao novo.
- Reuse QA se assinatura e requisitos continuarem compatíveis. Tamanho/mtime
  alterados, origem incerta ou colisão exigem inspeção/hash do item afetado.
- Em substituições, mantenha versões anteriores e suas dependências enquanto
  houver uso. `superseded` não autoriza excluir a mídia.
- Arquivos fornecidos pelo usuário mantêm nomes e conteúdo. Ao adotar material
  fora da pasta de entrega, registre o caminho relativo e a origem; não mova o
  arquivo. Resolva caminhos reais antes de qualquer escrita ou limpeza.
- Um coordenador consolida o manifesto. Preserve campos desconhecidos e assets
  não alterados; workers não escrevem o mesmo JSON simultaneamente.
- Escreva um temporário no mesmo diretório, valide JSON, IDs únicos, caminhos e
  requisitos de `ready`, então substitua atomicamente. Conserve o manifesto
  anterior se a validação falhar. Salve falhas por item junto com os sucessos.

## Exemplo de pesquisa pendente

Dados fictícios, sem alegar que um vídeo ou download foi verificado:

```json
{
  "schema_version": 1,
  "script": {
    "path": null,
    "sha256": "05c9b687baacf14d9cddc9ca7da4276ebc7060cf99f863fb13cab3197a4882fa",
    "text": "Agora vamos ao teste em movimento. A câmera mantém o horizonte estável durante a caminhada. Observe as bordas do quadro."
  },
  "settings": {"source_policy": "official", "min_short_edge": 1080, "audio": false, "codec": "source"},
  "assets": [
    {
      "id": "asset-camera-demo",
      "status": "pending",
      "file": null,
      "source": {
        "video_id": null,
        "url": null,
        "channel_id": null,
        "channel_url": null,
        "official_evidence": null,
        "range_s": null,
        "license": null
      },
      "download": {"format_selector": null, "format_id": null, "audio": false, "derivation": null},
      "signature": null,
      "media": null,
      "verification": {"signature": null, "technical": "pending", "visual": "pending", "notes": null},
      "uses": [
        {
          "id": "use-camera-stabilization",
          "segment": "Estabilização",
          "order": 1,
          "script_sha256": "05c9b687baacf14d9cddc9ca7da4276ebc7060cf99f863fb13cab3197a4882fa",
          "anchor": {
            "text": "A câmera mantém o horizonte estável durante a caminhada.",
            "context_before": "Agora vamos ao teste em movimento.",
            "context_after": "Observe as bordas do quadro.",
            "occurrence": 1
          },
          "clip_range_s": null
        }
      ],
      "resume": null,
      "notes": "Localizar uma demonstração oficial da estabilização."
    }
  ]
}
```

O hash acima corresponde ao texto UTF-8 completo de `script.text`, sem quebra de
linha final. Para arquivos, calcule sobre os bytes lidos do roteiro.
