# Instruções do projeto para agentes de IA

## Propósito

Este repositório contém o e-commerce didático do Módulo 6. O scaffolding inicial entrega listagem de produtos do navegador ao PostgreSQL, incluindo API, containers e testes E2E. Leia docs/scaffolding-specification.md antes de implementar ou alterar a estrutura.

## Stack aprovada

- Backend: JavaScript ES Modules, Node.js 24 LTS e Express.
- Frontend: HTML, CSS e JavaScript vanilla, sem framework de UI.
- Persistência: PostgreSQL 18 com Prisma.
- Pacotes independentes em backend/, frontend/ e e2e/, cada um com package.json e package-lock.json; a raiz mantém scripts npm, sem workspaces.
- Nginx serve arquivos do frontend e encaminha /api ao backend.
- Testes: Vitest, Supertest e Playwright em containers.
- Qualidade: ESLint e Prettier.
- Execução: Docker Compose e Makefile; alternativas npm para Windows sem Make.

## Arquitetura

- Backend: Route → Controller → Service → Repository → Prisma/PostgreSQL.
- Frontend: Presentation → Domain → API.
- Fluxo: navegador → Nginx → proxy /api → Express → camadas backend → Prisma → PostgreSQL.
- Mantenha as fronteiras: Controllers não consultam Prisma; views não fazem fetch.
- A API inicial de produtos é somente leitura. Não implemente escrita de produto, carrinho, cupons, pedidos, pagamento, autenticação ou regras de estoque sem requisito aprovado.

## Produto e API

- Produto: id inteiro sequencial, name, description, price e stock; sem timestamps.
- Nome e descrição obrigatórios; remova espaços externos e rejeite vazio; preserve espaços internos.
- Preço decimal não negativo, até duas casas, texto decimal na API e BRL na interface.
- Estoque inteiro não negativo; zero é válido.
- GET /api/products retorna { "products": [...] } por id crescente.
- GET /health informa estado do PostgreSQL: 200 quando pronto, 503 se o banco estiver indisponível.
- Erros usam { "error": { "code": "...", "message": "..." } }; não exponha stack traces, credenciais ou URLs sensíveis.
- Seed acrescenta exemplos apenas quando a tabela está vazia, sem sobrescrever dados locais.
- A interface mostra carregando, lista, lista vazia e erro da API.

## Containers e ambiente

- Compose inclui PostgreSQL, backend, frontend Nginx e runners sob demanda.
- Backend aguarda healthcheck do banco, aplica migrações e seed antes de iniciar.
- Use volume nomeado e não publique PostgreSQL 5432 no host.
- make down e make test-end param containers e preservam o volume; make clean-data apaga dados e é destrutivo.
- make test roda suíte completa e deixa os serviços principais ativos; runners são temporários.
- Valores padrão Compose são locais. .env é ignorado; .env.example não contém segredos reais.
- Fixe versões das imagens; nunca use latest. A versão da imagem Playwright corresponde ao pacote npm.
- Backend como não-root; .dockerignore para cada contexto.
- Compose é didático/local, não produção; revisar segredos, TLS, limites, reinícios e operação para deploy.

## Testes e qualidade

- Vitest para unidade/integração; Supertest para endpoints contra backend ativo; Playwright para E2E via Nginx/API.
- Testes usam banco de desenvolvimento e fixtures próprias identificáveis, removidas em teardown; não dependem da contagem total.
- Testes de interface podem simular carregando, vazio e erro sem alterar o banco.
- Não alegue validação sem executar os comandos e observar os resultados.
- Use ESLint e Prettier conforme scripts dos pacotes.
- Não execute testes nesta etapa sem solicitação do usuário; ao executar, relate os resultados observados.

## Convenções

- Documentação em português; código e identificadores em inglês.
- Não invente regras de negócio ausentes; registre lacunas para decisão humana.
- Preserve o escopo e leia os documentos da tarefa antes de alterar código.
- Explique decisões em mudanças maiores e mantenha o usuário informado.
- Não faça commit, push, merge ou publicação sem solicitação explícita.
