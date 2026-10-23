# Especificação do scaffolding — E-commerce do Módulo 6

## Objetivo e escopo

Preparar a base de um e-commerce didático com listagem funcional de produtos do navegador ao PostgreSQL. Inclui API, frontend, containers locais e testes de saúde do fluxo. A entrega inicial é somente leitura de produtos. Carrinho, cupons, pedidos, pagamentos, autenticação, autorização e escrita de produtos ficam para etapas futuras, com especificações próprias.

## Decisões aprovadas

| Tema | Decisão |
|---|---|
| Repositório | Um Git com pacotes independentes em backend/, frontend/ e e2e/, cada um com package.json e package-lock.json próprios. |
| Backend | JavaScript ES Modules, Node.js 24 LTS e Express. |
| Frontend | HTML, CSS e JavaScript vanilla, em Presentation, Domain e API. |
| Banco | PostgreSQL 18 em container próprio; Prisma para schema, cliente e migrações. |
| Servidor frontend | Nginx em container próprio serve arquivos estáticos e encaminha /api ao backend. |
| Portas | Frontend host 8080, API 3000; PostgreSQL somente na rede Compose. |
| Testes | Vitest para unidade/integração, Supertest para endpoints HTTP e Playwright para E2E de navegador; runners em containers. |
| Dados de teste | Usam o banco de desenvolvimento. Fixtures identificáveis são removidas ao final; testes não dependem da contagem total. |
| Qualidade | ESLint e Prettier. |
| Ambiente | Compose sobe sem .env, com valores padrão apenas locais; .env.example documenta as variáveis. |
| Comandos | Makefile: up, down, logs, test, test-end, migrate, seed, clean-data. Alternativas npm para Windows sem Make. |
| Produto | id inteiro sequencial; name, description, price e stock; sem timestamps. |
| Validação | Nome/descrição obrigatórios, espaços externos removidos e internos preservados; preço decimal >= 0, máximo de 2 casas; estoque inteiro >= 0. |
| Preço | Decimal exato no banco, texto decimal na API (ex.: "19.90") e BRL na interface (ex.: R$ 19,90). |
| API | GET /api/products retorna { "products": [...] } em id crescente. Erros usam { "error": { "code": "...", "message": "..." } }. |
| Saúde | GET /health informa o estado do banco; 200 quando pronto, 503 quando PostgreSQL indisponível. |
| Seed | Idempotente; insere exemplos somente se a tabela estiver vazia. |
| Idioma | Documentação em português; nomes de código em inglês. |

Usar tags específicas nas imagens Docker, nunca latest. A imagem do runner Playwright deve corresponder exatamente à versão npm do Playwright; fixar também a versão da imagem Nginx. Lockfiles são versionados.

## Fluxo e arquitetura

Navegador → Nginx (arquivos e proxy /api) → Express Route → Controller → Service → Repository → Prisma → PostgreSQL.

Backend: Routes registram endpoints; Controllers cuidam da borda HTTP; Services coordenam; Repositories encapsulam consultas Prisma. app.js monta Express sem listener para permitir testes; server.js conecta dependências e inicia HTTP. Não colocar consultas Prisma em Controllers.

Frontend:
- api/: fetch(), URL relativa /api, leitura de JSON e erros HTTP.
- domain/: regras locais sem DOM, incluindo formatação de preço.
- views/: HTML, renderização e interação com DOM.
- Estados visíveis: carregando, lista com produtos, lista vazia e falha da API. São situações de apresentação, não camadas ou rotas.

O Nginx serve http://localhost:8080 e encaminha /api ao backend:3000. A página e chamadas API compartilham a mesma origem. PostgreSQL não é exposto ao navegador nem ao host.

### Modelo Product

- id: inteiro sequencial gerado pelo banco.
- name e description: obrigatórios e não vazios após trim das extremidades.
- price: decimal não negativo com até duas casas.
- stock: inteiro não negativo; zero é válido.
- Remover espaços externos de name/description, preservando espaços internos. Ex.: “  Smartphone  XPTO  ” vira “Smartphone  XPTO”.
- Sem timestamps. Logs poderão ser discutidos quando houver requisito futuro.

## Endpoints e contratos

GET /health retorna 200 com estado da API e do banco quando disponíveis; 503 se não conseguir verificar o PostgreSQL. Não incluir credenciais, URLs ou detalhes internos.

GET /api/products retorna 200 e { "products": [...] }. Cada item tem id, name, description, price como texto decimal e stock; ordem id crescente. Sem paginação, busca ou filtros neste scaffolding.

Erros usam { "error": { "code": "identificador", "message": "mensagem segura" } } e status apropriado, sem stack traces ou credenciais.

## Estrutura proposta

    m6-ecommerce/
    ├── AGENTS.md, .dockerignore, .env.example, .gitignore, Makefile
    ├── README.md, docker-compose.yml, package.json, package-lock.json
    ├── scripts/npm-task.js
    ├── docs/scaffolding-specification.md
    ├── backend/
    │   ├── Dockerfile, .dockerignore, package.json, package-lock.json
    │   ├── prisma/schema.prisma, prisma/migrations/, prisma/seed.js
    │   ├── src/app.js, src/server.js
    │   ├── src/config/, src/routes/, src/controllers/
    │   ├── src/services/, src/repositories/, src/middlewares/, src/domain/
    │   └── tests/unit/, tests/integration/, tests/api/
    ├── frontend/
    │   ├── Dockerfile, .dockerignore, nginx.conf
    │   ├── package.json, package-lock.json, index.html, css/
    │   └── src/api/, src/domain/, src/views/, tests/unit/
    └── e2e/
        ├── Dockerfile, .dockerignore
        ├── package.json, package-lock.json, playwright.config.js
        └── specs/health.spec.js, products-api.spec.js, product-list.spec.js

O package.json raiz oferece scripts de orquestração, sem npm workspaces e sem dependências de runtime. A divisão interna pode ser ajustada sem quebrar fronteiras de camada.

## Dependências e ferramentas

- Backend runtime: express e @prisma/client. Desenvolvimento: prisma CLI, vitest, supertest, eslint e prettier.
- Frontend: sem framework nem build obrigatório; vitest, eslint e prettier para desenvolvimento/teste.
- E2E: Playwright e browsers na imagem oficial alinhada à versão do pacote npm.
- Infraestrutura: Docker Compose, PostgreSQL 18 e Nginx com tags fixadas.
- Usar npm ci; coordenar versões de prisma CLI e @prisma/client; não usar latest.

## Docker e ambiente

Compose:
- db: PostgreSQL 18, healthcheck pg_isready, volume nomeado, sem porta publicada no host.
- backend: Node 24 de backend/Dockerfile, usuário não-root, publica host 3000 para a porta interna 3000.
- frontend: Nginx, arquivos estáticos na porta 80, host 8080 e proxy /api para backend:3000.
- backend-tests: runner sob demanda para Vitest, integração e Supertest na rede Compose e no banco de desenvolvimento.
- e2e: runner Playwright sob demanda na mesma rede.

Backend aguarda db saudável, aplica prisma migrate deploy, executa seed idempotente e inicia API. make up deixa a listagem pronta sem dependências no host nem migração manual.

Usar volume nomeado. make down e make test-end param serviços e preservam dados; make clean-data remove o volume e apaga dados locais, com aviso claro. Cada contexto tem .dockerignore que exclui .git, node_modules, .env, builds, coverage e logs. Não copiar segredos para imagens nem usar ARG para segredos. Backend como não-root. Organizar Dockerfiles para aproveitar cache de dependências e estágio de desenvolvimento/teste.

Compose é local/didático, não produção. Um deploy exige revisão de segredos, TLS, limites de recursos, reinícios e operação.

Variáveis: POSTGRES_DB, POSTGRES_USER e POSTGRES_PASSWORD configuram banco local; DATABASE_URL conecta Prisma ao host interno db:5432; API_PORT padrão 3000; NODE_ENV define ambiente; FRONTEND_PORT padrão 8080. .env é opcional, local e ignorado. .env.example contém exemplos não sensíveis. Defaults Compose são somente desenvolvimento.

## Testes

- Unitários: Vitest testa regras isoladas backend/frontend; sem banco quando possível.
- Integração: Vitest cobre services/repositories com Prisma e PostgreSQL Compose.
- E2E de endpoints: Supertest chama backend ativo pela rede Compose e verifica /health, /api/products, status, JSON, estado do banco e fixtures.
- E2E frontend: Playwright abre página no Nginx, percorre proxy /api e confirma produto seed. Estados carregando/vazio/erro podem usar respostas controladas sem alterar banco.
- Fixtures são criadas com identificadores exclusivos e removidas em teardown mesmo se asserções falharem. Não depender da contagem total ou serviços externos.
- make test espera healthchecks, executa toda a suíte e falha se qualquer fase falhar. Runners temporários são removidos; os serviços principais permanecem ativos até make test-end ou make down.

## Comandos

| Comando | Ação |
|---|---|
| make up | Construir e iniciar db, backend e frontend; migração e seed automáticos. |
| make down | Parar serviços preservando volume. |
| make logs | Seguir logs. |
| make test | Suíte unitária, integração e E2E completa em containers. |
| make test-end | Parar a stack como make down; preserva dados. |
| make migrate | Aplicar migrações Prisma pendentes. |
| make seed | Reexecutar seed idempotente. |
| make clean-data | Remover volume PostgreSQL local, apagando dados. |

O package.json raiz fornece npm run up/down/logs/test/test-end/migrate/seed/clean-data via scripts multiplataforma. Pacotes continuam independentes, com seus próprios manifests e lockfiles. README documentará Make e npm.

## Convenções

- Documentação em português; código em inglês; JavaScript com ES Modules.
- camelCase para variáveis/funções; PascalCase para classes; nomes de módulos consistentes.
- Backend Route → Controller → Service → Repository; Prisma só em Repository, seed e migrações.
- Frontend Presentation → Domain → API; views não fazem fetch diretamente.
- Testes unitários/integração terminam em .test.js; Playwright usa .spec.js.
- ESLint e Prettier conforme scripts dos pacotes.
- Branch estável do scaffolding: clone-base; feature futura do carrinho: feat/carrinho em worktree, após blueprint próprio.
- Não fazer commit, push ou publicar sem solicitação explícita.

## Critérios de aceite

1. make up (ou npm run up) inicia PostgreSQL, backend e Nginx sem dependências no host ou etapa manual de migração.
2. /health confirma API e banco com 200 e informa indisponibilidade do banco com 503.
3. http://localhost:8080 lista produtos carregados via Nginx, API, Prisma e PostgreSQL.
4. Contrato, campos, validações, preço BRL e ordem por ID são respeitados.
5. Migrações e seed são repetíveis; seed não duplica nem sobrescreve produtos locais.
6. make test cobre unidade, integração, API E2E e frontend E2E em containers.
7. Fixtures são removidas; down/test-end preserva volume; clean-data é o comando para apagar dados.
8. .env.example, README e AGENTS documentam configuração e comandos.
9. Imagens estão fixadas, sem latest; sem credenciais reais ou porta PostgreSQL publicada.
