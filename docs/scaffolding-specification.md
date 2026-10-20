# Especificação do scaffolding — E-commerce do Módulo 6

## 1. Objetivo

Preparar a base de um e-commerce didático para praticar desenvolvimento assistido por LLM com participação humana em todas as decisões. O scaffolding deve entregar uma listagem de produtos funcional do navegador ao PostgreSQL, uma API verificável por `/health`, ambiente reprodutível com Docker Compose, dados iniciais e uma suíte de testes executável por um comando.

O scaffolding não implementa regras de negócio do carrinho, cupons ou pedidos. Esses itens serão detalhados e construídos em etapas posteriores, conforme o roteiro do Módulo 6.

## 2. Escopo inicial

### Incluído

- API REST em Node.js e Express.
- Frontend estático com HTML, CSS e JavaScript vanilla.
- Listagem de produtos cadastrados no PostgreSQL.
- Endpoint de saúde `GET /health`.
- Persistência relacional com PostgreSQL e Prisma ORM.
- Banco, backend e frontend disponíveis no Docker Compose.
- Migrações Prisma e seed idempotente de produtos de exemplo.
- Testes unitários, de integração, de endpoints e de interface no navegador.
- Makefile com comandos essenciais, incluindo `make test`.

### Fora do escopo do scaffolding

- Cadastro, edição e exclusão de produtos pela interface.
- Carrinho, cupons, pedidos, pagamento e autenticação.
- Pesquisa avançada, upload de imagens e notificações.
- Pipeline de CI/CD e implantação em produção.

## 3. Decisões técnicas aprovadas

| Tema | Decisão | Motivo |
|---|---|---|
| Repositório | Um repositório Git, com pastas `backend/` e `frontend/` | Mantém as aplicações separadas e atende à entrega com Compose na raiz e um único ZIP com histórico Git. |
| Backend | JavaScript com Node.js 24 LTS e Express | Stack escolhida pelo responsável; JavaScript reduz configuração adicional neste projeto. |
| Frontend | HTML, CSS e JavaScript vanilla | Mantém o frontend simples e exercita separação de responsabilidades sem framework de UI. |
| Banco | PostgreSQL 18, em serviço próprio no Compose | Banco relacional em container separado, alinhado à arquitetura do curso. |
| ORM | Prisma | Abstrai o acesso relacional e fornece schema e migrações versionáveis. |
| Pacotes | npm | Acompanha Node.js e usa lockfile para instalações reproduzíveis. Cada aplicação terá seu próprio `package.json` e `package-lock.json`; a raiz poderá ter scripts de orquestração. |
| Testes | Vitest; Supertest para endpoints HTTP; Playwright para testes no navegador | Cobre unidade, integração, API e fluxo de interface. |
| Frontend no Compose | Nginx | Serve os arquivos estáticos em um container separado. |
| Portas locais | Frontend `8080`, API `3000`; PostgreSQL `5432` apenas na rede Docker | Facilita o acesso pelo navegador e mantém o banco sem porta publicada no host. |
| Dados locais | Volume nomeado do Docker para o PostgreSQL | Os dados persistem ao parar os containers; remover o volume apaga o banco local. |
| Ambiente | `.env` local ignorado pelo Git e `.env.example` versionado | Separa configuração local do código e documenta as variáveis necessárias. |
| Documentação e código | Documentos em português; identificadores de código em inglês | Documentação acessível ao responsável e convenções de código consistentes. |
| Branch estável | `clone-base`; feature posterior em `feat/carrinho` | Alinha o fluxo de branches/worktree às etapas posteriores do roteiro. O repositório atual está em `master`, sem commits; a mudança de branch fica para a preparação do primeiro commit. |

As referências de versão são Node.js 24 LTS e PostgreSQL 18. As versões exatas das dependências npm deverão ser resolvidas e travadas nos lockfiles durante a implementação do scaffolding. Não usar tags flutuantes `latest` nas imagens Docker.

## 4. Estrutura proposta

```text
m6-ecommerce/
├── AGENTS.md                         # Regras persistentes para agentes de IA
├── .dockerignore                     # Itens excluídos dos contextos de build
├── .env.example                      # Modelo de configuração sem segredos reais
├── .gitignore                        # Exclusões Git para Node, Docker, env e builds
├── Makefile                          # Comandos de desenvolvimento, Docker e testes
├── README.md                         # Visão geral, portas, comandos e verificação
├── docker-compose.yml                # frontend, backend, banco e runner de testes
├── package.json                      # Scripts auxiliares de orquestração na raiz
├── package-lock.json                 # Lockfile dos scripts/dependências da raiz
├── docs/
│   └── scaffolding-specification.md  # Este documento
├── backend/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── package.json
│   ├── package-lock.json
│   ├── prisma/
│   │   ├── schema.prisma             # Modelo relacional inicial de produtos
│   │   ├── migrations/               # Migrações versionadas
│   │   └── seed.js                   # Seed de produtos idempotente
│   ├── src/
│   │   ├── app.js                    # Montagem Express sem iniciar o listener
│   │   ├── server.js                 # Inicialização HTTP
│   │   ├── config/                   # Configuração validada do ambiente
│   │   ├── controllers/              # Entrada HTTP e respostas
│   │   ├── routes/                   # Rotas e composição de endpoints
│   │   ├── services/                 # Lógica da aplicação
│   │   ├── repositories/             # Acesso a dados via Prisma
│   │   └── middlewares/              # Erros e comportamentos HTTP compartilhados
│   └── tests/
│       ├── unit/                     # Testes de unidade Vitest
│       ├── integration/              # Integração de componentes do backend
│       └── api/                      # Testes HTTP com Vitest e Supertest
├── frontend/
│   ├── nginx.conf                    # Configuração do servidor estático
│   ├── index.html
│   ├── css/
│   └── src/
│       ├── api/                      # fetch() e conversão de respostas HTTP
│       ├── domain/                   # Regras locais de exibição/validação de UX
│       └── views/                    # Renderização e interação com DOM
└── e2e/
    ├── package.json                 # Dependências e scripts Playwright
    ├── package-lock.json
    ├── playwright.config.js
    └── product-list.spec.js         # Fluxo de listagem no navegador
```

Os nomes e a divisão poderão ser ajustados no blueprint, desde que as fronteiras de camada sejam preservadas: Controller → Service → Repository no backend e Presentation → Domain → API no frontend.

## 5. Dependências e ferramentas

### Runtime e infraestrutura

- Node.js 24 LTS para desenvolvimento e backend.
- Express para endpoints REST JSON.
- PostgreSQL 18 como banco relacional.
- Prisma ORM para schema, cliente e migrações.
- Nginx para servir o frontend estático.
- Docker Compose para executar os serviços em conjunto.

### Desenvolvimento e qualidade

- npm com lockfiles versionados.
- Vitest para testes unitários e integração.
- Supertest para exercitar a API HTTP.
- Playwright para testes E2E de navegador.
- ESLint para análise estática de JavaScript.
- Prettier para formatação consistente.
- Make para comandos comuns via Makefile; quando `make` não estiver disponível no Windows, disponibilizar um comando npm equivalente para os testes.

As dependências devem usar versões estáveis compatíveis com Node.js 24 e Prisma, fixadas pelos lockfiles. A implementação deve verificar a compatibilidade das versões exatas antes de instalá-las.

## 6. Modelo e fluxo da listagem

O modelo inicial `Product` deve conter, no mínimo: identificador, nome, descrição, preço, quantidade em estoque e timestamps. A listagem é somente leitura. A validação deve rejeitar preço negativo, campos obrigatórios ausentes e estoque negativo.

Fluxo esperado:

1. O usuário abre `http://localhost:8080`.
2. A camada `views/` solicita produtos à camada `api/`.
3. O módulo de API chama o endpoint de produtos do backend.
4. Controller valida e traduz o pedido HTTP; Service coordena a operação; Repository consulta o PostgreSQL via Prisma.
5. A resposta JSON é apresentada no frontend.

Endpoints iniciais:

- `GET /health` — responde `200` quando a aplicação está pronta e confirma conectividade com o banco.
- `GET /api/products` — responde `200` com uma lista JSON de produtos.

Erros devem retornar códigos HTTP apropriados e uma resposta JSON estável, sem expor stack traces ou credenciais.

## 7. Banco, migrações e seed

- O serviço do PostgreSQL usa um volume nomeado.
- O Compose define healthcheck com `pg_isready`; o backend aguarda `service_healthy`.
- O backend aplica migrações versionadas com `prisma migrate deploy` antes de iniciar a aplicação.
- O seed cria produtos de exemplo com `upsert` ou mecanismo equivalente para ser idempotente.
- `docker compose up` deve deixar a listagem funcional sem instalação de dependências no host nem etapa manual de migração.
- `docker compose down` preserva o volume; `docker compose down -v` remove os dados locais e deve ser descrito como operação destrutiva.

Para permitir a primeira execução sem criar arquivos manualmente, o Compose poderá fornecer valores-padrão exclusivos de desenvolvimento local. Esses valores não são segredos e não podem ser reutilizados fora do ambiente didático/local. `.env.example` documentará como personalizar a configuração. Nenhuma credencial real deve ser versionada.

## 8. Docker e configuração

Serviços previstos no Compose:

- `db`: PostgreSQL com healthcheck e volume nomeado; sem publicação da porta `5432` no host.
- `backend`: imagem construída de `backend/Dockerfile`, porta `3000`, usuário não-root e dependência do banco saudável.
- `frontend`: Nginx servindo `frontend/` na porta host `8080`.
- `e2e`: runner de Playwright em perfil ou execução sob demanda, na mesma rede Docker para acessar a interface e a API.

Requisitos de imagens e segurança:

- Usar tags de versão específicas e estáveis; não usar `latest`.
- Usar `.dockerignore` para excluir `.git`, `node_modules`, builds, cobertura, `.env` e logs dos contextos.
- O backend roda sem root no container.
- Não copiar arquivos `.env` para imagens.
- Definir healthchecks e dependências saudáveis para os serviços que dependem do banco.
- O Compose destina-se ao desenvolvimento e avaliação didática; não é uma configuração de produção.

Variáveis esperadas (nomes exatos podem ser fechados no blueprint):

| Variável | Uso |
|---|---|
| `POSTGRES_DB` | Nome do banco local |
| `POSTGRES_USER` | Usuário do banco local |
| `POSTGRES_PASSWORD` | Senha de desenvolvimento local, sem valor real |
| `DATABASE_URL` | URL usada pelo Prisma para acessar `db:5432` na rede do Compose |
| `API_PORT` | Porta HTTP interna do backend, padrão `3000` |
| `NODE_ENV` | Ambiente da aplicação |

## 9. Testes e critérios de aceitação

- Unitários: regras isoladas de Service e Domain, com dependências substituídas por doubles quando necessário.
- Integração: interação das camadas do backend e Prisma com PostgreSQL de teste/containerizado.
- API: Supertest verifica `GET /health`, `GET /api/products`, formato das respostas e erros relevantes.
- Navegador: Playwright abre o frontend e confirma que os produtos seed aparecem.
- Uma execução única via `make test` deve iniciar ou reutilizar os serviços necessários, executar todos os níveis de teste e terminar com código de saída diferente de zero quando houver falha.
- Deve existir alternativa npm equivalente para ambientes Windows sem `make`.
- Os testes não devem depender de dados manuais, de serviços externos ou de estado residual do volume.

## 10. Comandos e interface de desenvolvimento

O Makefile deve oferecer, no mínimo:

| Comando | Ação |
|---|---|
| `make up` | Construir e iniciar frontend, backend e PostgreSQL |
| `make down` | Parar os serviços sem apagar o volume |
| `make logs` | Seguir os logs dos serviços |
| `make test` | Executar a suíte unitária, integração, API e navegador |
| `make migrate` | Aplicar migrações Prisma no banco local |
| `make seed` | Reaplicar o seed idempotente |
| `make clean-data` | Remover o volume local; avisar que apaga os dados |

O README documentará portas, requisitos (Docker Desktop/Compose e Make ou alternativa npm), como subir/parar, consultar `/health`, ver a listagem e executar testes. A execução de `docker compose up` na raiz deve ser autossuficiente para o avaliador.

## 11. Git e convenções

- Usar a branch estável `clone-base` para o scaffolding e a branch isolada `feat/carrinho` para a feature posterior, de preferência em worktree.
- Commits pequenos e descritivos, um por decisão ou iteração concluída.
- Não versionar `.env`, dependências instaladas, builds, relatórios de cobertura, volumes ou dados locais.
- Antes do primeiro commit, conferir o nome/e-mail local do Git e revisar o conteúdo do `.gitignore`.
- A ausência de remoto Git não impede o desenvolvimento local; configurar e testar o remoto quando o usuário escolher o serviço e fornecer/criar a URL.

## 12. Critérios para considerar o scaffolding pronto

- `docker compose up` inicia os serviços sem instalação de dependências no host.
- `GET /health` responde `200` com o banco disponível.
- A página do frontend exibe os produtos seed via API.
- Migrações e seed são repetíveis sem duplicações.
- `make test` (ou alternativa npm documentada) executa todos os testes exigidos.
- `README.md`, `AGENTS.md` e este documento descrevem o sistema implementado.
- Um revisor em sessão limpa compara a implementação com esta especificação e salva `docs/scaffolding-review-report.md`.

## Referências de versão

- [Ciclo de releases do Node.js](https://nodejs.org/en/about/previous-releases)
- [Pré-requisitos do Prisma ORM](https://www.prisma.io/docs/getting-started)
- [Política e versões suportadas do PostgreSQL](https://www.postgresql.org/support/versioning/)
- Requisitos funcionais e fluxo de entrega: `Modulo 6.pdf`, seções 6.0 a 6.1.
