# Blueprint de desenvolvimento — Scaffolding do e-commerce M6

## 1. Visão geral

Este documento planeja a implementação da base didática que percorre o fluxo **navegador → Nginx → Express → Prisma → PostgreSQL**. A entidade `Product` será usada como exemplo fim a fim, e o seed fornecerá 12 produtos fictícios para demonstrar a listagem e permitir validar variações de preço e estoque.

O repositório contém a especificação, o `README.md`, o `AGENTS.md` e o `.gitignore`; os pacotes de aplicação e a infraestrutura definidos neste plano ainda serão criados. A estrutura proposta será implementada preservando os arquivos existentes e atualizando a documentação necessária.

**Critérios de sucesso:** `make up` — ou os scripts npm equivalentes — inicia a aplicação sem etapa manual de migração; `/health` informa corretamente o estado do banco; a página lista os produtos pela API e pelo proxy Nginx; e os testes em containers verificam contratos e estados de apresentação. O seed é repetível, insere dados apenas quando a tabela está vazia e preserva produtos locais existentes.

**Fora de escopo:** escrita de produtos pela API, carrinho, cupons, pedidos, pagamento, autenticação, autorização, regras de estoque, busca, filtros, paginação e configuração de produção. Deploy em produção exigiria decisões próprias sobre segredos, TLS, limites de recursos e operação.

## 2. Contexto técnico e convenções

- Backend: JavaScript ES Modules, Node.js 24 LTS e Express.
- Frontend: HTML, CSS e JavaScript vanilla, sem framework de UI ou build obrigatório.
- Persistência: PostgreSQL 18 em container e Prisma para schema, cliente e migrações.
- Pacotes independentes em `backend/`, `frontend/` e `e2e/`, cada qual com `package.json` e `package-lock.json`; a raiz terá scripts de orquestração sem npm workspaces nem dependências de runtime.
- Backend: Route → Controller → Service → Repository → Prisma/PostgreSQL. Controllers não consultam Prisma. `app.js` monta Express sem listener; `server.js` conecta dependências e inicia HTTP.
- Frontend: Presentation → Domain → API. Views não fazem `fetch()` diretamente; módulos de domínio não dependem de DOM.
- Nginx serve arquivos estáticos e encaminha `/api` para `backend:3000`. Frontend em `http://localhost:8080`, API publicada na porta 3000 e PostgreSQL somente na rede Compose.
- Testes: Vitest para unidade/integração, Supertest para endpoints HTTP e Playwright para E2E. Linters e formatadores: ESLint e Prettier.
- Documentação em português; código e identificadores em inglês; código JavaScript usando camelCase para variáveis/funções e PascalCase para classes. Testes Vitest terminam em `.test.js`; Playwright usa `.spec.js`.

## 3. Implementação de Product

A sequência começa pela persistência e avança até a interface para validar cada fronteira antes de integrar o fluxo completo.

### 3.1 Pacotes, configurações e documentação

- Criar manifests independentes e lockfiles para `backend/`, `frontend/` e `e2e/`; manter scripts de orquestração na raiz sem npm workspaces ou dependências de runtime.
- Configurar ES Modules, ESLint e Prettier conforme os padrões definidos, além de `.env.example` sem segredos reais.
- Documentar setup, comandos Make e alternativas npm no `README.md`; manter documentação em português e identificadores de código em inglês.
- **Resultado esperado:** estrutura de dependências e comandos definida antes do desenvolvimento das camadas.

### 3.2 Persistência, modelo e migração

- Criar `backend/prisma/schema.prisma` com `Product`: `id` inteiro sequencial, `name`, `description`, `price` decimal exato e `stock` inteiro; sem timestamps.
- Representar no schema as restrições do banco compatíveis com o contrato: preço não negativo, precisão de até duas casas e estoque inteiro não negativo. Nome e descrição serão validados na lógica de domínio, incluindo trim externo e rejeição de vazio.
- Gerar e versionar a migração inicial em `backend/prisma/migrations/`; configurar Prisma CLI e `@prisma/client` em versões coordenadas.
- **Resultado esperado:** banco reproduzível a partir da migração e modelo alinhado à especificação.

### 3.3 Domínio, repositório, serviço e endpoint

- Implementar validações e transformações de Product no domínio, com testes unitários independentes de banco sempre que possível.
- Criar repositório que encapsule Prisma e consulte produtos por `id` crescente; serviço que coordene a operação; controller que traduza resultado e erros para HTTP; e rota `GET /api/products`.
- Criar `app.js` para montar Express sem iniciar listener e `server.js` para conectar dependências e iniciar servidor. Controllers não consultam Prisma.
- Respeitar `{ "products": [...] }`, preço como texto decimal (por exemplo, `"19.90"`) e erros no formato `{ "error": { "code": "...", "message": "..." } }`, com status apropriados. Mensagens devem ser seguras. A listagem não terá paginação, busca ou filtros.
- **Resultado esperado:** endpoint somente leitura com contrato estável, ordenação e separação de camadas testáveis.

### 3.4 Health check

- Implementar `GET /health` usando uma verificação real de conectividade com PostgreSQL. Retornar 200 quando API e banco estiverem disponíveis e 503 quando o banco não puder ser verificado.
- Não incluir credenciais, URLs de conexão, stack traces nem detalhes internos na resposta.
- **Resultado esperado:** healthcheck utilizável por Compose e pelos testes de API.

### 3.5 Frontend

- Criar a página estática em `frontend/index.html`, estilos em `frontend/css/` e módulos separados em `frontend/src/api/`, `frontend/src/domain/` e `frontend/src/views/`.
- A API usa `fetch()` com URL relativa `/api`, interpreta JSON e trata respostas HTTP de erro. O domínio contém regras locais sem DOM, incluindo formatação de preço em BRL. A view renderiza e interage com o DOM sem chamar `fetch()` diretamente.
- Exibir os estados carregando, lista com produtos, lista vazia e falha da API. Apresentar preço no formato BRL, por exemplo `R$ 19,90`.
- **Resultado esperado:** interface acessa a API pela mesma origem e apresenta os estados definidos na especificação.

## 4. Plano de seed

- Criar `backend/prisma/seed.js` e conectá-lo ao mecanismo de seed do Prisma e aos comandos de execução.
- Usar operação idempotente que primeiro verifique se a tabela está vazia. Inserir os **12 produtos fictícios** apenas nessa condição; não atualizar, substituir ou duplicar produtos existentes quando o seed for repetido.
- Cobrir variações úteis à validação: preços baixos e altos com duas casas decimais, nomes e descrições em português, espaços internos preservados e diferentes níveis de estoque, incluindo estoque zero. Não usar dados reais ou sensíveis.
- Executar automaticamente o seed na inicialização do backend após a migração e disponibilizar reexecução via `make seed` e `npm run seed`.
- Para manter os testes independentes do total do catálogo, usar fixtures próprias identificáveis, criadas e removidas pelos testes em teardown.

## 5. Containers, comandos e ambiente

### 5.1 Containers

- Criar `docker-compose.yml` com PostgreSQL 18, backend Node.js 24, frontend Nginx e runners sob demanda para testes. Fixar tags de todas as imagens, inclusive Nginx e Playwright; a imagem do Playwright deve corresponder exatamente à versão do pacote npm.
- Configurar healthcheck `pg_isready` no banco e dependência do backend condicionada ao banco saudável. O backend aplica migrações, executa o seed e então inicia o servidor.
- Usar volume PostgreSQL nomeado sem publicar a porta 5432 no host; publicar frontend em 8080 e API em 3000. O Nginx serve os arquivos estáticos e encaminha `/api` para `backend:3000`.
- Criar Dockerfiles e `.dockerignore` para cada contexto. O backend deve rodar como não-root, sem segredos na imagem, com dependências copiadas antes do código para aproveitar cache e estágio apropriado a desenvolvimento/teste.
- A configuração Docker é local/didática, não um perfil de produção. Um deploy exige revisão de segredos, TLS, limites, reinícios e operação.

### 5.2 Comandos

- Criar Makefile com `up`, `down`, `logs`, `test`, `test-end`, `migrate`, `seed` e `clean-data`, além de `scripts/npm-task.js` e scripts npm equivalentes para Windows sem Make.
- `down` e `test-end` param a stack e preservam o volume. `clean-data` remove o volume e seus dados, com aviso claro do efeito destrutivo.
- `make test` aguarda healthchecks, executa toda a suíte e falha se qualquer fase falhar. Os runners são temporários e os serviços principais permanecem ativos até `make test-end` ou `make down`.
- `.env` é opcional, local e ignorado pelo Git. `.env.example` documenta `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `DATABASE_URL`, `API_PORT`, `NODE_ENV` e `FRONTEND_PORT` com valores de exemplo não sensíveis. Os defaults Compose são somente locais.
- `npm ci` deve ser utilizado para instalação reprodutível e lockfiles devem ser versionados.

## 6. Plano de testes

Os testes abaixo são planejados, não executados nesta etapa de blueprint.

### 6.1 Vitest unitário

- Validar nome e descrição: obrigatoriedade, remoção de espaços externos e preservação dos internos.
- Validar preço decimal não negativo com até duas casas; validar estoque inteiro não negativo, permitindo zero.
- Verificar serialização de preço decimal e formatação de BRL sem usar ponto flutuante para representar dinheiro na API.
- Testar regras isoladas de frontend sem DOM quando possível.

### 6.2 Vitest integração

- Cobrir service e repository com Prisma e PostgreSQL Compose.
- Verificar leitura e ordenação crescente por ID, além da integração service → repository → banco.
- Usar banco de desenvolvimento com fixtures identificáveis, sem depender da quantidade total de produtos.

### 6.3 Supertest para endpoints

- Testar `/health` com banco disponível e indisponível, verificando 200 e 503.
- Testar `GET /api/products` quanto a status, envelope JSON, campos, preço textual, ordem por ID e formato de erro seguro.
- Criar fixtures com identificadores exclusivos e removê-las em teardown mesmo se uma asserção falhar.

### 6.4 Playwright para fluxo de navegador

- Abrir a página pelo Nginx, atravessar o proxy `/api` e confirmar que produtos seed são apresentados.
- Verificar formatação BRL e estados carregando, lista vazia e erro da API usando respostas controladas, sem alterar o banco para simular esses estados.
- Manter o runner em container com versão da imagem alinhada exatamente à versão npm do Playwright.

### 6.5 Execução e qualidade

- `make test` executa unidade, integração, API E2E e frontend E2E em containers, aguardando os healthchecks e propagando falhas.
- Documentar scripts de teste, lint e formatação em cada pacote e no README.
- Na implementação, executar os comandos e reportar resultados observados; não afirmar validação sem evidência dos resultados.

## 7. Sequência de implementação

1. Criar manifests independentes, lockfiles, scripts de qualidade e configurações de ESLint/Prettier.
2. Implementar schema Prisma, migração inicial e configuração de conexão.
3. Implementar domínio de Product e testes unitários de validação.
4. Implementar repositório, serviço, controller, rota, `app.js` e `server.js`.
5. Implementar `/health` e testes de unidade/integração correspondentes.
6. Implementar seed idempotente com 12 produtos fictícios e comando de reexecução.
7. Implementar API de produtos e testes Supertest com fixtures isoladas.
8. Implementar módulos API e domínio frontend, incluindo formatação BRL e testes unitários.
9. Implementar HTML, estilos e views com os quatro estados de apresentação.
10. Implementar Nginx e configuração de arquivos estáticos e proxy `/api`.
11. Implementar Dockerfiles, `.dockerignore`, Compose, healthchecks, volume e execução de migração/seed.
12. Implementar runners Vitest/Supertest/Playwright e a suíte Playwright via Nginx.
13. Implementar Makefile e wrapper npm multiplataforma para todos os comandos especificados.
14. Atualizar README e `.env.example` com configuração, fluxo, seed, testes e operações de dados.
15. Executar validação completa em containers e comandos de lint/formatação; corrigir falhas antes de declarar os critérios de aceite atendidos.

## 8. Pontos de atenção, dependências e premissas

- O esquema exato de `price` (precisão e escala) deve permitir valores decimais não negativos com no máximo duas casas; conferir a representação Prisma/PostgreSQL durante a migração sem converter dinheiro para ponto flutuante na API.
- A validação de nome e descrição precisa remover apenas espaços das extremidades e rejeitar resultado vazio, preservando espaços internos.
- A aplicação deve ter estratégia coerente para o caso de PostgreSQL ficar indisponível depois de iniciar; `/health` deve continuar detectando a indisponibilidade e responder 503.
- A sequência de inicialização precisa impedir que a API aceite tráfego antes de migração e seed concluírem; falha em qualquer etapa deve impedir a inicialização saudável.
- Makefile e wrapper npm devem chamar a mesma orquestração para evitar diferença entre ambientes. `clean-data` é a única operação planejada para apagar o volume de dados.
- Não inventar regras de negócio ausentes. Códigos e mensagens concretos de erros devem permanecer seguros e coerentes com o contrato, sem expor detalhes internos.
- Não fazer commit, push, merge ou publicação sem solicitação explícita.

## 9. Critérios de aceite

1. `make up` (ou `npm run up`) inicia PostgreSQL, backend e Nginx sem dependências no host ou etapa manual de migração.
2. `/health` confirma API e banco com 200 e informa indisponibilidade do banco com 503.
3. `http://localhost:8080` lista produtos carregados via Nginx, API, Prisma e PostgreSQL.
4. Contrato, campos, validações, preço BRL e ordem por ID são respeitados.
5. Migrações e seed são repetíveis; seed não duplica nem sobrescreve produtos locais.
6. `make test` cobre unidade, integração, API E2E e frontend E2E em containers.
7. Fixtures são removidas; `down`/`test-end` preservam volume; `clean-data` é o comando para apagar dados.
8. `.env.example`, README e AGENTS documentam configuração e comandos.
9. Imagens estão fixadas, sem `latest`; não há credenciais reais nem porta PostgreSQL publicada no host.
