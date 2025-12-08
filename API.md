# Documentação da API - Marketplace Místico

## Índice
1. [Visão Geral](#visão-geral)
2. [Autenticação](#autenticação)
3. [Endpoints](#endpoints)
4. [Exemplos de Uso](#exemplos-de-uso)

---

## Visão Geral

O sistema Marketplace Místico utiliza uma API RESTful com autenticação via JWT (JSON Web Tokens). Todos os endpoints estão disponíveis em `/api`.

### Base URL
```
http://localhost:3000/api (desenvolvimento)
https://seu-dominio.vercel.app/api (produção)
```

---

## Autenticação

### Como funciona a autenticação

O sistema utiliza **JWT (JSON Web Tokens)** para autenticação. Após o login bem-sucedido:
1. O servidor gera um token JWT válido por 7 dias
2. O cliente armazena o token no `localStorage`
3. O token deve ser enviado no header `Authorization` em todas as requisições autenticadas

### Header de Autenticação
```
Authorization: Bearer <seu-token-jwt>
```

---

## Endpoints

### 1. Registro de Usuário

**POST** `/api/auth/register`

Cria uma nova conta de usuário (cliente ou vendedor).

#### Request Body

**Para Cliente:**
```json
{
  "tipo": "cliente",
  "nome": "João Silva",
  "email": "joao@exemplo.com",
  "senha": "senha123456",
  "telefone": "(11) 98765-4321",
  "cpf_cnpj": "123.456.789-00"
}
```

**Para Vendedor:**
```json
{
  "tipo": "vendedor",
  "nome": "Maria Santos",
  "email": "maria@exemplo.com",
  "senha": "senha123456",
  "telefone": "(11) 98765-4321",
  "cpf_cnpj": "12.345.678/0001-90",
  "nomeLoja": "Loja da Maria",
  "categoria": "Cristais e Pedras",
  "descricaoLoja": "Loja especializada em cristais energéticos"
}
```

#### Campos Obrigatórios
- `tipo`: "cliente" ou "vendedor"
- `nome`: Nome completo do usuário
- `email`: Email único no sistema
- `senha`: Mínimo 8 caracteres

#### Campos Opcionais
- `telefone`: Telefone de contato
- `cpf_cnpj`: CPF (11 dígitos) ou CNPJ (14 dígitos)

#### Campos para Vendedor
- `nomeLoja`: Nome da loja (obrigatório para vendedores)
- `categoria`: Categoria da loja (obrigatório para vendedores)
- `descricaoLoja`: Descrição da loja (opcional)

#### Response - Sucesso (201)
```json
{
  "success": true,
  "message": "Usuário criado com sucesso",
  "user": {
    "id": 1,
    "tipo": "vendedor",
    "nome": "Maria Santos",
    "email": "maria@exemplo.com",
    "telefone": "(11) 98765-4321",
    "nomeLoja": "Loja da Maria"
  }
}
```

#### Response - Erro (400)
```json
{
  "error": "Email já cadastrado"
}
```

#### Como a senha é armazenada?
A senha é criptografada usando **bcrypt** com 10 rounds antes de ser armazenada no banco de dados. Nunca armazenamos senhas em texto plano.

---

### 2. Login

**POST** `/api/auth/login`

Autentica um usuário e retorna um token JWT.

#### Request Body
```json
{
  "email": "maria@exemplo.com",
  "senha": "senha123456"
}
```

#### Campos Obrigatórios
- `email`: Email cadastrado
- `senha`: Senha do usuário

#### Response - Sucesso (200)
```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "tipo": "vendedor",
    "nome": "Maria Santos",
    "email": "maria@exemplo.com",
    "telefone": "(11) 98765-4321",
    "cpf_cnpj": "12.345.678/0001-90",
    "seller_id": 1,
    "nomeLoja": "Loja da Maria",
    "categoria": "Cristais e Pedras",
    "descricaoLoja": "Loja especializada em cristais energéticos",
    "endereco": null
  }
}
```

#### Response - Erro (401)
```json
{
  "error": "Email ou senha incorretos"
}
```

#### Processo de Login
1. Frontend envia email e senha via POST
2. Backend busca usuário no banco de dados
3. Backend valida senha usando bcrypt.compare()
4. Backend gera token JWT com dados do usuário
5. Backend busca informações adicionais (vendedor/endereço)
6. Frontend armazena token e dados do usuário no localStorage
7. Frontend atualiza interface e redireciona para dashboard

---

### 3. Listar Produtos

**GET** `/api/products`

Lista todos os produtos publicados no marketplace.

#### Query Parameters (Opcionais)
- `categoria`: Filtra por categoria (ex: "Cristais e Pedras")
- `seller_id`: Filtra por vendedor específico

#### Request
```http
GET /api/products
GET /api/products?categoria=Cristais%20e%20Pedras
GET /api/products?seller_id=1
```

#### Headers
Não requer autenticação para listar produtos públicos.

#### Response - Sucesso (200)
```json
{
  "success": true,
  "products": [
    {
      "id": 1,
      "seller_id": 1,
      "nome": "Quartzo Rosa",
      "categoria": "Cristais e Pedras",
      "descricao": "Quartzo rosa natural, pedra do amor",
      "preco": 49.90,
      "estoque": 10,
      "imagem_url": "https://exemplo.com/quartzo-rosa.jpg",
      "publicado": true,
      "created_at": "2024-01-15T10:30:00.000Z",
      "nome_loja": "Loja da Maria",
      "vendedor_id": 1
    }
  ]
}
```

---

### 4. Obter Produto por ID

**GET** `/api/products/[id]`

Obtém detalhes de um produto específico.

#### Request
```http
GET /api/products/1
```

#### Headers
Não requer autenticação.

#### Response - Sucesso (200)
```json
{
  "success": true,
  "product": {
    "id": 1,
    "seller_id": 1,
    "nome": "Quartzo Rosa",
    "categoria": "Cristais e Pedras",
    "descricao": "Quartzo rosa natural, pedra do amor",
    "preco": 49.90,
    "estoque": 10,
    "imagem_url": "https://exemplo.com/quartzo-rosa.jpg",
    "publicado": true,
    "created_at": "2024-01-15T10:30:00.000Z",
    "nome_loja": "Loja da Maria",
    "vendedor_id": 1
  }
}
```

#### Response - Erro (404)
```json
{
  "error": "Produto não encontrado"
}
```

---

### 5. Criar Produto

**POST** `/api/products`

Cria um novo produto. **Requer autenticação de vendedor.**

#### Headers
```
Authorization: Bearer <token-jwt>
Content-Type: application/json
```

#### Request Body
```json
{
  "nome": "Ametista",
  "categoria": "Cristais e Pedras",
  "descricao": "Ametista natural para proteção espiritual",
  "preco": 89.90,
  "estoque": 5,
  "imagemUrl": "https://exemplo.com/ametista.jpg",
  "publicado": true
}
```

#### Campos Obrigatórios
- `nome`: Nome do produto
- `categoria`: Categoria do produto
- `preco`: Preço do produto (número)

#### Campos Opcionais
- `descricao`: Descrição detalhada
- `estoque`: Quantidade em estoque (padrão: 0)
- `imagemUrl`: URL da imagem do produto
- `publicado`: Se o produto está publicado (padrão: false)

#### Response - Sucesso (201)
```json
{
  "success": true,
  "product": {
    "id": 2,
    "seller_id": 1,
    "nome": "Ametista",
    "categoria": "Cristais e Pedras",
    "descricao": "Ametista natural para proteção espiritual",
    "preco": 89.90,
    "estoque": 5,
    "imagem_url": "https://exemplo.com/ametista.jpg",
    "publicado": true,
    "created_at": "2024-01-15T11:00:00.000Z"
  }
}
```

#### Response - Erro (403)
```json
{
  "error": "Acesso negado"
}
```

---

### 6. Excluir Produto

**DELETE** `/api/products/[id]`

Exclui um produto. **Requer autenticação de vendedor e propriedade do produto.**

#### Headers
```
Authorization: Bearer <token-jwt>
```

#### Request
```http
DELETE /api/products/2
```

#### Response - Sucesso (200)
```json
{
  "success": true,
  "message": "Produto deletado"
}
```

#### Response - Erro (404)
```json
{
  "error": "Produto não encontrado ou sem permissão"
}
```

---

### 7. Obter Perfil do Usuário

**GET** `/api/users/profile`

Obtém informações do perfil do usuário autenticado.

#### Headers
```
Authorization: Bearer <token-jwt>
```

#### Request
```http
GET /api/users/profile
```

#### Response - Sucesso (200)
```json
{
  "success": true,
  "user": {
    "id": 1,
    "tipo": "vendedor",
    "nome": "Maria Santos",
    "email": "maria@exemplo.com",
    "telefone": "(11) 98765-4321",
    "cpf_cnpj": "12.345.678/0001-90",
    "tipo_documento": "CNPJ",
    "created_at": "2024-01-10T08:00:00.000Z",
    "seller_id": 1,
    "nome_loja": "Loja da Maria",
    "categoria": "Cristais e Pedras",
    "descricao_loja": "Loja especializada em cristais energéticos"
  }
}
```

#### Response - Erro (401)
```json
{
  "error": "Não autenticado"
}
```

---

## Exemplos de Uso

### Exemplo 1: Fluxo Completo de Registro e Login

#### Passo 1: Registrar um novo vendedor
```javascript
const registroResponse = await fetch('/api/auth/register', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    tipo: 'vendedor',
    nome: 'Maria Santos',
    email: 'maria@exemplo.com',
    senha: 'senha123456',
    telefone: '(11) 98765-4321',
    cpf_cnpj: '12.345.678/0001-90',
    nomeLoja: 'Loja da Maria',
    categoria: 'Cristais e Pedras',
    descricaoLoja: 'Loja especializada em cristais energéticos'
  })
});

const registroData = await registroResponse.json();
console.log('Registro:', registroData);
```

#### Passo 2: Fazer login
```javascript
const loginResponse = await fetch('/api/auth/login', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    email: 'maria@exemplo.com',
    senha: 'senha123456'
  })
});

const loginData = await loginResponse.json();
const token = loginData.token;

// Armazenar token e usuário no localStorage
localStorage.setItem('authToken', token);
localStorage.setItem('currentUser', JSON.stringify(loginData.user));
```

---

### Exemplo 2: Criar um Produto (Vendedor Autenticado)

```javascript
const token = localStorage.getItem('authToken');

const response = await fetch('/api/products', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    nome: 'Ametista',
    categoria: 'Cristais e Pedras',
    descricao: 'Ametista natural para proteção espiritual',
    preco: 89.90,
    estoque: 5,
    imagemUrl: 'https://exemplo.com/ametista.jpg',
    publicado: true
  })
});

const data = await response.json();
console.log('Produto criado:', data);
```

---

### Exemplo 3: Listar Produtos com Filtro

```javascript
// Listar todos os produtos
const todosResponse = await fetch('/api/products');
const todosData = await todosResponse.json();
console.log('Todos os produtos:', todosData.products);

// Listar produtos de uma categoria específica
const cristaisResponse = await fetch('/api/products?categoria=Cristais%20e%20Pedras');
const cristaisData = await cristaisResponse.json();
console.log('Cristais:', cristaisData.products);

// Listar produtos de um vendedor específico
const vendedorResponse = await fetch('/api/products?seller_id=1');
const vendedorData = await vendedorResponse.json();
console.log('Produtos do vendedor:', vendedorData.products);
```

---

### Exemplo 4: Obter Perfil do Usuário

```javascript
const token = localStorage.getItem('authToken');

const response = await fetch('/api/users/profile', {
  method: 'GET',
  headers: {
    'Authorization': `Bearer ${token}`
  }
});

const data = await response.json();
console.log('Perfil do usuário:', data.user);
```

---

### Exemplo 5: Excluir um Produto

```javascript
const token = localStorage.getItem('authToken');
const productId = 2;

const response = await fetch(`/api/products/${productId}`, {
  method: 'DELETE',
  headers: {
    'Authorization': `Bearer ${token}`
  }
});

const data = await response.json();
console.log('Resultado:', data);
```

---

## Resumo dos Métodos HTTP Utilizados

| Endpoint | Método | Autenticação | Descrição |
|----------|--------|--------------|-----------|
| `/api/auth/register` | POST | Não | Registrar novo usuário |
| `/api/auth/login` | POST | Não | Fazer login |
| `/api/products` | GET | Não | Listar produtos |
| `/api/products` | POST | Sim (Vendedor) | Criar produto |
| `/api/products/[id]` | GET | Não | Obter produto específico |
| `/api/products/[id]` | DELETE | Sim (Vendedor) | Excluir produto |
| `/api/users/profile` | GET | Sim | Obter perfil do usuário |

---

## Códigos de Status HTTP

| Código | Significado |
|--------|-------------|
| 200 | Sucesso - Requisição processada com sucesso |
| 201 | Criado - Recurso criado com sucesso |
| 400 | Bad Request - Dados inválidos ou faltando |
| 401 | Não Autorizado - Token inválido ou ausente |
| 403 | Proibido - Usuário não tem permissão |
| 404 | Não Encontrado - Recurso não existe |
| 405 | Método Não Permitido - HTTP method incorreto |
| 500 | Erro Interno - Erro no servidor |

---

## Segurança

### Criptografia de Senhas
- Todas as senhas são criptografadas com **bcrypt** (10 rounds)
- Senhas nunca são armazenadas em texto plano
- Senhas nunca são retornadas nas respostas da API

### JWT (JSON Web Token)
- Token expira em 7 dias
- Token contém: `id`, `email`, `tipo` do usuário
- Secret JWT: configurado em `process.env.JWT_SECRET`
- Token deve ser enviado no header `Authorization: Bearer <token>`

### CORS
Todas as APIs têm CORS configurado para aceitar requisições de qualquer origem (`Access-Control-Allow-Origin: *`). Em produção, recomenda-se configurar origens específicas.

---

## Categorias Disponíveis

- Cristais e Pedras
- Tarot e Oráculos
- Incensos e Ervas
- Velas e Rituais
- Jóias Místicas
- Livros Esotéricos
- Decoração Mística
- Óleos Essenciais

---

## Estados Brasileiros Suportados

O sistema suporta todos os 27 estados brasileiros:
AC, AL, AP, AM, BA, CE, DF, ES, GO, MA, MT, MS, MG, PA, PB, PR, PE, PI, RJ, RN, RS, RO, RR, SC, SP, SE, TO

---

## Notas Técnicas

### Banco de Dados
- Usa **Neon Database** (@neondatabase/serverless)
- Queries retornam arrays de resultados
- Conexão configurada via `process.env.DATABASE_URL`

### Autenticação no Frontend
O frontend (`public/app.js`) implementa:
- `apiRequest()`: Helper que automaticamente adiciona token JWT aos headers
- `localStorage`: Persiste token e dados do usuário entre sessões
- Verificação automática de sessão no carregamento da página

### Validações no Frontend
- Email: Validação de formato via regex
- Senha: Mínimo 8 caracteres
- CPF/CNPJ: Validação de formato (11 ou 14 dígitos)
- Campos obrigatórios: Verificação antes do envio

---

## Troubleshooting

### Erro: "Token não recebido do servidor"
- Verifique se JWT_SECRET está configurado
- Verifique se o banco de dados está acessível
- Verifique logs do servidor para erros de autenticação

### Erro: "Email já cadastrado"
- O email deve ser único no sistema
- Use um email diferente ou faça login

### Erro: "Acesso negado"
- Verifique se o token JWT está válido
- Verifique se o usuário tem permissão (vendedor para criar/deletar produtos)
- Verifique se o token está sendo enviado no header Authorization

### Erro: "Não autenticado"
- Faça login novamente para obter um novo token
- Verifique se o token não expirou (validade de 7 dias)
- Verifique se o token está sendo enviado corretamente
