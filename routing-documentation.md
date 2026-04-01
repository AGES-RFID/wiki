# Documentação de Rotas Frontend-Backend

## Visão Geral

Este documento explica como criar e gerenciar rotas entre o frontend e o backend no sistema de gerenciamento de estacionamento IMPINJ. O sistema utiliza ASP.NET Core Web API para o backend e segue princípios RESTful para o design da API.

## Estrutura das Rotas

### URL Base
- **Desenvolvimento**: `https://localhost:7000/api`
- **Produção**: `https://seu-dominio.com/api`

### Convenções de Nomenclatura de Rotas
- Use substantivos no plural para nomes de recursos (ex: `/customers`, `/cars`)
- Use verbos HTTP apropriadamente (GET, POST, PUT, DELETE)
- Use kebab-case para nomes de recursos com múltiplas palavras
- Inclua versionamento se necessário: `/api/v1/customers`

## Métodos HTTP e Seus Usos

| Método | Uso | Exemplo |
|--------|-----|---------|
| GET | Recuperar recursos | `GET /api/customers` |
| POST | Criar novo recurso | `POST /api/customers` |
| PUT | Atualizar recurso inteiro | `PUT /api/customers/{id}` |
| PATCH | Atualização parcial | `PATCH /api/customers/{id}` |
| DELETE | Remover recurso | `DELETE /api/customers/{id}` |

## Comunicação Frontend-Backend

### Estrutura da Requisição
```javascript
// Exemplo de requisição frontend
const response = await fetch('/api/customers', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify(customerData)
});

const data = await response.json();
```

### Estrutura da Resposta
```json
{
  "success": true,
  "data": { /* dados do recurso */ },
  "message": "Cliente criado com sucesso",
  "errors": [] // erros de validação se houver
}
```

## Autenticação & Autorização

### Estrutura do Token JWT
```json
{
  "sub": "user_id",
  "email": "user@example.com",
  "role": "Customer|Admin",
  "exp": 1234567890
}
```

### Cabeçalhos de Autorização
```javascript
headers: {
  'Authorization': `Bearer ${jwtToken}`
}
```

## Tratamento de Erros

### Resposta de Erro Padrão
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dados de entrada inválidos",
    "details": [
      {
        "field": "email",
        "message": "Email é obrigatório"
      }
    ]
  }
}
```

### Códigos de Status HTTP
- `200 OK` - GET/PUT/DELETE bem-sucedidos
- `201 Created` - POST bem-sucedido
- `400 Bad Request` - Erros de validação
- `401 Unauthorized` - Não autenticado
- `403 Forbidden` - Não autorizado
- `404 Not Found` - Recurso não encontrado
- `500 Internal Server Error` - Erro do servidor

## Visão Geral dos Endpoints da API

### Gerenciamento de Clientes
- `POST /api/customers/register` - Registrar novo cliente
- `GET /api/customers/{id}` - Obter detalhes do cliente
- `PUT /api/customers/{id}` - Atualizar cliente

### Gerenciamento de Administradores
- `POST /api/admins/register` - Registrar novo administrador
- `GET /api/admins/{id}` - Obter detalhes do administrador
- `PUT /api/admins/{id}` - Atualizar administrador

### Saldo da Conta
- `GET /api/customers/{id}/balance` - Obter total de fundos
- `GET /api/customers/{id}/transactions` - Obter histórico de transações
- `POST /api/customers/{id}/deposit` - Adicionar fundos
- `POST /api/customers/{id}/withdraw` - Retirar fundos

### Tags RFID
- `POST /api/rfid-tags` - Criar nova tag
- `POST /api/rfid-tags/{id}/deactivate` - Desativar tag
- `POST /api/rfid-tags/{id}/reactivate` - Reativar tag

### Carros
- `GET /api/cars` - Obter todos os carros
- `POST /api/cars` - Registrar novo carro
- `DELETE /api/cars/{id}` - Excluir carro (e tag associada)

## Exemplos de Integração com Frontend

### Exemplo de Serviço React
```javascript
// services/customerService.js
class CustomerService {
  async register(customerData) {
    const response = await fetch('/api/customers/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(customerData)
    });
    return response.json();
  }

  async getBalance(customerId) {
    const response = await fetch(`/api/customers/${customerId}/balance`);
    return response.json();
  }

  async getTransactions(customerId) {
    const response = await fetch(`/api/customers/${customerId}/transactions`);
    return response.json();
  }
}
```

## Objetos de Transferência de Dados (DTOs)

### O que é um DTO?

Um **DTO (Data Transfer Object)** é um padrão de design usado para transferir dados entre subsistemas de uma aplicação. No contexto de APIs web, DTOs são objetos que:

- **Transportam dados** entre o cliente (frontend) e o servidor (backend)
- **Isolam a camada de dados** da camada de apresentação
- **Validam e formatam dados** antes de serem processados
- **Evitam expor informações sensíveis** do modelo de domínio

### Por que usar DTOs?

1. **Segurança**: Não expõe campos internos do banco de dados
2. **Validação**: Permite validação específica para cada operação
3. **Flexibilidade**: Diferentes formatos para diferentes contextos
4. **Manutenção**: Mudanças no modelo não afetam a API

### Exemplo Prático

```csharp
// Modelo de Domínio (interno)
public class Customer
{
    public int CustomerId { get; set; }
    public required string Email { get; set; }
    public required string Name { get; set; }
    public required string Cpf { get; set; }
    public string? InternalNotes { get; set; } // Campo interno
    public DateTime CreatedAt { get; set; }
}

// DTO para criação (recebe dados do frontend)
public class CreateCustomerDto
{
    [Required(ErrorMessage = "Email é obrigatório")]
    [EmailAddress(ErrorMessage = "Email inválido")]
    public string Email { get; set; } = string.Empty;

    [Required(ErrorMessage = "Nome é obrigatório")]
    public string Name { get; set; } = string.Empty;

    [Required(ErrorMessage = "CPF é obrigatório")]
    [StringLength(11, MinimumLength = 11)]
    public string Cpf { get; set; } = string.Empty;
}

// DTO para resposta (envia dados ao frontend)
public class CustomerDto
{
    public int Id { get; set; }
    public string Email { get; set; } = string.Empty;
    public string Name { get; set; } = string.Empty;
    public string Cpf { get; set; } = string.Empty;
    // InternalNotes não é incluído na resposta
    public DateTime CreatedAt { get; set; }
}
```

### O que é uma Interface?

Uma **interface** é um contrato que define um conjunto de métodos e propriedades que uma classe deve implementar. Em C#, interfaces são usadas para:

- **Definir contratos**: Garantem que classes sigam uma estrutura específica
- **Injeção de Dependência**: Permitem desacoplamento entre componentes
- **Testabilidade**: Facilitam a criação de mocks para testes

### Exemplo de Interface

```csharp
// Definição da interface
public interface ICustomerService
{
    Task<CustomerDto> CreateCustomerAsync(CreateCustomerDto dto);
    Task<CustomerDto> GetCustomerAsync(int id);
    Task<IEnumerable<CustomerDto>> GetAllCustomersAsync();
}

// Implementação concreta
public class CustomerService : ICustomerService
{
    public async Task<CustomerDto> CreateCustomerAsync(CreateCustomerDto dto)
    {
        // Lógica de criação
        return new CustomerDto();
    }

    public async Task<CustomerDto> GetCustomerAsync(int id)
    {
        // Lógica de busca
        return new CustomerDto();
    }

    public async Task<IEnumerable<CustomerDto>> GetAllCustomersAsync()
    {
        // Lógica de listagem
        return new List<CustomerDto>();
    }
}

// Uso no Controller (injeção de dependência)
[ApiController]
public class CustomersController(ICustomerService customerService) : ControllerBase
{
    private readonly ICustomerService _customerService = customerService;

    [HttpPost]
    public async Task<ActionResult<CustomerDto>> CreateCustomer(CreateCustomerDto dto)
    {
        var customer = await _customerService.CreateCustomerAsync(dto);
        return CreatedAtAction(nameof(GetCustomer), new { id = customer.Id }, customer);
    }
}
```

### Como Usar DTOs e Interfaces

#### 1. **Fluxo de Criação**
```
Frontend → CreateCustomerDto → Controller → Service → Repository → Database
```

#### 2. **Fluxo de Leitura**
```
Database → Repository → Service → CustomerDto → Controller → Frontend
```

#### 3. **Boas Práticas**

**Para DTOs:**
- Use nomes descritivos (CreateCustomerDto, UpdateCustomerDto)
- Inclua validações com Data Annotations
- Mantenha DTOs simples e focados
- Não inclua lógica de negócio nos DTOs

**Para Interfaces:**
- Use prefixo "I" (ICustomerService)
- Mantenha interfaces coesas e focadas
- Prefira interfaces pequenas e específicas
- Use interfaces para injeção de dependência

#### 4. **Exemplo Completo**

```csharp
// 1. Interface do serviço
public interface ICustomerService
{
    Task<CustomerDto> CreateCustomerAsync(CreateCustomerDto dto);
}

// 2. DTOs
public class CreateCustomerDto
{
    [Required]
    public string Name { get; set; } = string.Empty;
}

public class CustomerDto
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
}

// 3. Implementação do serviço
public class CustomerService : ICustomerService
{
    public async Task<CustomerDto> CreateCustomerAsync(CreateCustomerDto dto)
    {
        // Validação e lógica de negócio
        var customer = new Customer { Name = dto.Name };
        
        // Salvar no banco
        
        // Retornar DTO
        return new CustomerDto { Id = customer.Id, Name = customer.Name };
    }
}

// 4. Controller usando a interface
[ApiController]
[Route("api/customers")]
public class CustomersController(ICustomerService customerService) : ControllerBase
{
    [HttpPost]
    public async Task<ActionResult<CustomerDto>> CreateCustomer(CreateCustomerDto dto)
    {
        var customer = await customerService.CreateCustomerAsync(dto);
        return CreatedAtAction(nameof(GetCustomer), new { id = customer.Id }, customer);
    }
}
```

Este padrão garante **separação de responsabilidades**, **testabilidade** e **manutenibilidade** do código.


## Melhores Práticas

### Backend
1. **Validação**: Sempre valide os dados de entrada
2. **Tratamento de Erros**: Retorne respostas de erro consistentes
3. **Documentação**: Use Swagger/OpenAPI para documentação da API
4. **Segurança**: Implemente autenticação e autorização adequadas
5. **Logging**: Registre requisições e erros para depuração

### Frontend
1. **Tratamento de Erros**: Lide com erros de API de forma elegante
2. **Estados de Carregamento**: Mostrue indicadores de carregamento durante requisições
3. **Cache**: Cacheie dados acessados frequentemente
4. **Lógica de Retentativa**: Implemente retentativas para requisições falhas
5. **Segurança de Tipos**: Use TypeScript para segurança de tipos

## Configuração CORS

### Configuração CORS no Backend
```csharp
// Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:3000")
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials();
    });
});

app.UseCors("AllowFrontend");
```

## Testando as Rotas

### Usando curl
```bash
# Registrar cliente
curl -X POST https://localhost:7000/api/customers/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","name":"John Doe","cpf":"12345678901","cellphone":"11999999999","cep":"01310100","address":"Rua Augusta, 1000"}'

# Obter saldo do cliente
curl -X GET https://localhost:7000/api/customers/1/balance \
  -H "Authorization: Bearer SEU_JWT_TOKEN"
```

### Usando Postman
1. Crie uma coleção para a API IMPINJ
2. Defina a URL base: `{{baseUrl}}/api`
3. Configure variáveis de ambiente para diferentes ambientes
4. Adicione cabeçalhos de autenticação conforme necessário
