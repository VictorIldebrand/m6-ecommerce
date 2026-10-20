# Instruções do projeto para agentes de IA

## Propósito

Este repositório implementa o projeto prático do Módulo 6: um e-commerce didático usado para aprender desenvolvimento assistido por LLM com revisão e decisão humanas. A primeira entrega é o scaffolding e a listagem de produtos. Carrinho, cupons e pedidos pertencem a etapas posteriores e devem ser implementados apenas quando solicitados, a partir de suas especificações.

## Stack aprovada

- Backend: JavaScript, Node.js 24 LTS e Express.
- Frontend: HTML, CSS e JavaScript vanilla.
- Persistência: PostgreSQL 18 com Prisma ORM.
- Pacotes: npm e lockfiles versionados.
- Testes: Vitest, Supertest e Playwright.
- Qualidade: ESLint e Prettier.
- Execução: Docker Compose, backend em Node e frontend servido por Nginx.
- A branch estável planejada é `clone-base`; a feature do carrinho deve usar `feat/carrinho` em worktree, seguindo o roteiro.

## Arquitetura

- Backend: Controller → Service → Repository. Controllers cuidam da borda HTTP; Services coordenam regras de aplicação; Repositories encapsulam persistência via Prisma.
- Frontend: Presentation (`views/`) → Domain (`domain/`) → API (`api/`). Views manipulam DOM; Domain guarda regras locais de exibição/UX; API encapsula `fetch()`.
- Mantenha as camadas separadas. Não coloque consultas Prisma em Controllers nem lógica de negócio junto à manipulação de DOM.
- PostgreSQL é um serviço próprio no Compose, conectado ao backend pela rede interna.

## Regras de trabalho

- Leia `docs/scaffolding-specification.md` e os documentos da tarefa antes de alterar código.
- O usuário está aprendendo: explique decisões, mostre planos antes de mudanças maiores e peça revisão dos planos. Não trate sugestões da IA como decisões já aprovadas.
- Preserve o escopo documentado. Não adicione frameworks, serviços externos, autenticação, pagamentos ou funcionalidades fora de escopo sem discutir primeiro.
- Prefira tarefas pequenas e commits descritivos. Não faça commit, push ou publicação sem instrução explícita do usuário.
- Mantenha documentação do projeto em português. Use nomes de variáveis, funções, classes e módulos em inglês.
- Não invente requisitos ausentes. Registre lacunas e pergunte ao usuário antes de escolher regras de negócio.

## Docker, ambiente e segurança

- O fluxo normal de avaliação deve funcionar com `docker compose up` na raiz.
- Use tags de imagem específicas, nunca `latest`.
- Inclua healthchecks; o backend só deve iniciar após o PostgreSQL ficar saudável.
- Use volume nomeado para os dados do banco. Explique que `docker compose down -v` apaga os dados.
- Inclua `.dockerignore` adequado para cada contexto; exclua `.git`, `.env`, `node_modules`, builds, cobertura e logs.
- Backend roda como usuário não-root no container.
- Nunca grave segredos reais em arquivos versionados, imagem, log ou teste. `.env` é local e ignorado; `.env.example` contém apenas nomes e exemplos não sensíveis.
- Valores padrão do Compose são apenas para desenvolvimento local e não devem ser apresentados como configuração de produção.
- Não exponha a porta do PostgreSQL no host por padrão.

## Banco e dados

- Alterações de schema devem vir com migrações Prisma versionadas.
- Seeds precisam ser idempotentes; reiniciar o Compose não deve duplicar produtos.
- Não dependa de dados inseridos manualmente. Testes devem preparar seus próprios dados ou usar seed isolado e previsível.
- Para a listagem, produto é somente leitura pela API inicial; não crie endpoints de escrita sem requisito aprovado.

## Testes e qualidade

- Use Vitest para unidade e integração, Supertest para HTTP/API e Playwright para fluxos no navegador.
- `make test` (ou alternativa npm documentada) é a interface principal para toda a suíte.
- Adicione ou altere testes junto das mudanças correspondentes, mas só execute comandos de teste após informar o usuário e obter sua autorização quando a execução pedir aprovação do sistema.
- Não alegue que algo foi validado sem executar e observar o resultado.
- Formate com Prettier e verifique lint com ESLint conforme os comandos documentados.

## Comandos

Os comandos definitivos serão documentados no README e no Makefile após a implementação do scaffolding. A interface planejada inclui `make up`, `make down`, `make logs`, `make test`, `make migrate`, `make seed` e `make clean-data`.
