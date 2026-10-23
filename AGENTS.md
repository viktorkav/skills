# Skills

## Contexto

Coleção pública de skills reutilizáveis. Cada pasta em `skills/` deve funcionar
quando instalada sozinha, sem acesso ao ambiente do autor.

## Stack

Markdown com frontmatter YAML; metadados opcionais em `agents/openai.yaml`.
Dependências de execução são declaradas por skill. O repositório não precisa
de credenciais, serviço ou ambiente de desenvolvimento próprio.

## Dev

Ao alterar uma skill, confira o frontmatter, os links relativos e os exemplos.
Use o validador de skills do ambiente, se disponível. Valide comandos alterados
com a ferramenta correspondente; não baixe mídia para testar apenas documentação.
Para mudanças de comportamento, exercite um pedido realista em pasta temporária.

## Arquitetura

`README.md` é o catálogo e a instalação. `skills/<nome>/SKILL.md` é a entrada
de cada skill; referências e recursos necessários ficam dentro da mesma pasta.

## Gotchas

- Generalizar regras pessoais antes de publicar: caminhos, canais, editores e
  hardware do autor não são requisitos dos usuários.
- Manter o nome do diretório igual ao `name` do frontmatter.
- Preservar compatibilidade entre agentes; instruções não dependem de nomes de
  ferramentas exclusivos de um produto.
- Fixtures usam dados fictícios. Não incluir roteiros privados, mídia, cookies,
  manifests de produção ou arquivos de configuração pessoais.
- A licença do repositório cobre a skill, não os vídeos encontrados por ela.
