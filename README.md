# Skills do ViktorKav

Coleção de skills reutilizáveis para agentes de IA, em português brasileiro. Cada pasta em `skills/` reúne as instruções e referências de uma skill e pode ser instalada separadamente.

## Disponíveis

| Skill | O que faz |
| --- | --- |
| [broll](skills/broll/SKILL.md) | Pesquisa vídeos no YouTube para ilustrar um roteiro, relaciona cada apoio à fala e, quando solicitado, baixa e confere os arquivos. |

A coleção começa com a evolução da [broll-skill](https://github.com/viktorkav/broll-skill), agora sem depender de uma estrutura pessoal de pastas, canal ou formato de roteiro. Novas skills entram em `skills/<nome>/` com seu próprio `SKILL.md` e arquivos de apoio.

## Instalação

Clone o repositório em uma pasta de sua escolha:

```sh
git clone https://github.com/viktorkav/skills.git viktorkav-skills && cd viktorkav-skills
```

Copie a pasta **completa** da skill para o diretório pessoal do seu agente:

| Agente | Destino da pasta `broll` |
| --- | --- |
| Claude Code | `~/.claude/skills/broll` |
| Codex | `~/.agents/skills/broll` |

No macOS ou Linux, execute o bloco abaixo a partir da raiz do clone. Para Codex, troque o destino da primeira linha pela alternativa indicada:

```sh
(
  skill_destination="$HOME/.claude/skills/broll"
  # Para Codex: skill_destination="$HOME/.agents/skills/broll"
  if [ ! -f skills/broll/SKILL.md ]; then
    printf '%s\n' 'Execute este bloco na raiz do repositório clonado.' >&2
    exit 1
  fi
  mkdir -p "$(dirname "$skill_destination")" &&
    mkdir "$skill_destination" &&
    cp -R skills/broll/. "$skill_destination/"
)
```

Se o destino já existir, a cópia para sem sobrescrevê-lo ou criar uma pasta `broll/broll`. Para atualizar, compare as versões e preserve suas alterações antes de substituir a instalação. Atualizar o clone com `git pull` não atualiza a cópia instalada.

No Windows, copie manualmente `skills/broll` para `.claude/skills/broll` ou `.agents/skills/broll` dentro da sua pasta de usuário. Mantenha `SKILL.md` e os arquivos de apoio juntos; se o destino existir, compare antes de substituir.

Os caminhos seguem a documentação de [skills do Claude Code](https://code.claude.com/docs/en/skills) e de [skills do Codex](https://developers.openai.com/codex/skills). Se a skill não aparecer no agente, reinicie a sessão.

## Usar a broll

Forneça um roteiro em texto legível pelo agente, colado na conversa ou em um arquivo que ele consiga ler. Informe se quer somente pesquisa ou também downloads e indique a pasta de saída quando houver arquivos.

Exemplos de pedidos:

```text
Use a skill broll para pesquisar apoios para este roteiro. Quero apenas
links de fontes oficiais e sugestões de trechos; não baixe os vídeos.
```

```text
Use a skill broll para buscar e baixar apoios para roteiro.md na pasta
./apoios. Quero pelo menos 1080p, sem áudio, com um manifesto fala → asset.
```

O fluxo usa fontes oficiais e só inclui fontes alternativas se você pedir. A entrega registra a relação entre fala, vídeo, trecho sugerido, arquivo e estado de verificação. Retomadas conferem o manifesto e os arquivos para evitar duplicações por vídeo, intervalo e formato.

Nos downloads, o padrão é resolução mínima de 1080p, sem áudio e preservação do codec original. Você pode ajustar esses requisitos no pedido. A skill registra lacunas quando não encontra material adequado e só declara um arquivo pronto após verificação técnica e inspeção visual.

## Ferramentas necessárias

- **Pesquisa:** acesso à web ou navegador; também pode usar `yt-dlp` quando disponível. A ausência do CLI não impede uma pesquisa de links.
- **Download:** `yt-dlp` e os executáveis `ffmpeg` e `ffprobe` acessíveis ao agente.
- **Conferência:** capacidade de abrir imagens ou vídeos para inspecionar os arquivos baixados.

O suporte completo do `yt-dlp` ao YouTube usa `yt-dlp-ejs` e um runtime JavaScript compatível. A configuração depende da forma de instalação; consulte as [dependências oficiais do yt-dlp](https://github.com/yt-dlp/yt-dlp#dependencies). Instalar a pasta da skill não instala essas ferramentas.

## Licença

As instruções deste repositório usam a [licença MIT](LICENSE); os vídeos encontrados continuam sujeitos aos direitos e às condições de uso de seus respectivos titulares.
