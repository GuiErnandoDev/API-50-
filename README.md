# API PI-Web — Etapa 1 (cerca de 50%) · Express + MySQL + Insomnia

API REST em **Node.js + Express** com banco **MySQL** (criado e consultado no **MySQL Workbench**),
testada no **Insomnia** com corpo em **JSON**.

## O que está pronto e o que falta

| Bloco do diagrama | Onde está | Situação |
|---|---|---|
| ServidorExpress (CORS, JSON 100 kb, log, iniciar) | `server.js` | Pronto |
| HealthRouter — `verificar()` | `server.js` | Pronto (200 ou 503) |
| ErroAPI (400, 404, 413, 500) | `server.js` → `tratarErros()` | Pronto |
| ProdutoRouter — `listar()` GET | `routes/produtos.js` | Pronto |
| ProdutoRouter — `criar()` POST (recebe JSON) | `routes/produtos.js` | Pronto (201 ou 400) |
| ValidadorProduto — `validarCorpo`, `normalizarTexto`, `validarSegmento`, `normalizarPreco` | `validacao.js` | Pronto |
| BancoMySQL — `conectar`, `executar` | `database.js` | Pronto |
| Tabelas Produto e Segmento | `database/schema.sql` | Pronto |
| ProdutoRouter — `consultar()`, `atualizar()`, `excluir()` | `routes/produtos.js` | **Etapa 2** (hoje 501) |
| ValidadorProduto — `validarId` | `validacao.js` | **Etapa 2** |

## Passo a passo

### 1. Criar o banco no MySQL Workbench
1. Abra o Workbench e conecte-se ao seu servidor MySQL (conexão "Local instance").
2. **File > Open SQL Script...** e escolha `database/schema.sql`.
3. Clique no raio (**Execute**). No final aparecem os 5 produtos.
4. (Opcional) Para ver o desenho das tabelas: **Database > Reverse Engineer...** e siga até o fim.
   O Workbench gera o diagrama EER com a ligação `produtos.segment_id → segmentos.id`.

### 2. Configurar e rodar a API
```bash
npm install
copy .env.example .env        # no Mac/Linux: cp .env.example .env
```
Abra o `.env` e coloque a senha do seu MySQL em `DB_PASSWORD`. Depois:
```bash
npm start       # http://localhost:3000
npm test        # testes do validador (não precisam do MySQL)
```
No terminal deve aparecer `[db] MySQL conectado` e `[api] Express executando na porta 3000`.

### 3. Testar no Insomnia
1. **Application > Import > From File** e escolha `insomnia/PI-Web-API-mysql.json`.
2. Envie as requisições (a URL base está na variável `base_url`).
3. Cada requisição traz na descrição o status esperado. Nas da pasta **3**, o JSON enviado está em **Body**.
4. Depois do POST válido, volte ao Workbench e rode `SELECT * FROM produtos;` para ver a nova linha.

## Rotas

| Verbo | Rota | Resposta |
|---|---|---|
| GET | `/api/health` | 200 `{ "status": "ok", "database": "connected" }` ou 503 se o MySQL estiver fora |
| GET | `/api/produtos` | 200 com a lista |
| POST | `/api/produtos` | 201 com o produto (e cabeçalho `Location`) ou 400 com `campos` |
| GET, PUT, DELETE | `/api/produtos/:id` | 501 (etapa 2) |
| qualquer outra | — | 404 `{ "erro": "Rota não encontrada." }` |

Exemplo de corpo do POST:
```json
{
  "name": "Cadinho de teste",
  "segmentId": "laboratorio",
  "material": "Porcelana de alta alumina",
  "description": "Peça criada pelo Insomnia.",
  "price": 12.5
}
```
`segmentId` aceita: `fundicao`, `laboratorio`, `analise`, `aquario`. `price` é opcional.

## Problemas comuns
- `Sem conexão com o MySQL: ECONNREFUSED` — o servidor MySQL não está ligado (confira no Workbench).
- `Access denied for user` — senha errada no `.env`.
- `Unknown database 'pi_web'` — falta executar o `schema.sql` (passo 1).
- Health devolvendo 503 — a API está no ar, mas sem conexão com o banco.
