# Documentação de Mappers

## Visão Geral

Os **mappers** são classes estáticas responsáveis por converter objetos entre diferentes camadas da aplicação, como:
- **Models** (entidades de domínio) ↔ **DTOs** (Data Transfer Objects)
- **DTOs** de criação ↔ **Models** de entidade
- **DTOs** de atualização ↔ **Models** existentes

## Por que usar Mappers?

1. **Separação de Responsabilidades**: Isola a lógica de conversão dos serviços
2. **Reutilização**: Evita duplicação de código de mapeamento
3. **Manutenibilidade**: Centraliza as regras de conversão em um único lugar
4. **Testabilidade**: Facilita a criação de testes unitários
5. **Performance**: Evita reflexão em tempo de execução

## Estrutura dos Mappers

### Padrão de Nomenclatura
- `{Entidade}Mapper.cs` (ex: `CustomerMapper.cs`)
- Métodos de extensão para facilitar o uso
- Nomes descritivos: `ToDto()`, `ToEntity()`, `UpdateEntity()`

### Métodos Comuns

```csharp
// Conversão de Entidade → DTO
public static CustomerDto ToDto(this Customer customer)

// Conversão de DTO de Criação → Entidade
public static Customer ToEntity(this CreateCustomerDto dto)

// Atualização de Entidade existente
public static void UpdateEntity(this Customer customer, CreateCustomerDto dto)

// Conversão de Lista → Lista de DTOs
public static IEnumerable<CustomerDto> ToDtos(this IEnumerable<Customer> customers)
```

## Exemplos de Uso

### 1. **CustomerMapper**

```csharp
// Em um Service
var customer = dto.ToEntity();           // CreateCustomerDto → Customer
customer.UserId = GenerateId();

var customerDto = customer.ToDto();     // Customer → CustomerDto

// Atualização
existingCustomer.UpdateEntity(dto);     // Atualiza propriedades
var updatedDto = existingCustomer.ToDto();

// Lista
var customerDtos = customers.ToDtos();  // List<Customer> → List<CustomerDto>
```

### 2. **AdminMapper**

```csharp
// Criação
var admin = createAdminDto.ToEntity();
admin.UserId = _admins.Count + 1;

// Conversão para DTO
var adminDto = admin.ToDto();

// Atualização
admin.UpdateEntity(updateDto);
```

### 3. **CarMapper** (com lógica adicional)

```csharp
// Conversão com cálculo de duração
var carDto = car.ToDto(); // Calcula automaticamente DurationInParkingLot

// Operações de estacionamento
car.RecordEntry();  // Registra entrada
car.RecordExit();   // Registra saída e calcula duração
```

## Implementação nos Serviços

### Antes (com código repetido):
```csharp
public async Task<CustomerDto> CreateCustomerAsync(CreateCustomerDto dto)
{
    var customer = new Customer
    {
        Email = dto.Email,
        Name = dto.Name,
        Cpf = dto.Cpf,
        // ... mais propriedades
        UserId = _customers.Count + 1
    };

    _customers.Add(customer);

    return new CustomerDto
    {
        Id = customer.UserId,
        Email = customer.Email,
        Name = customer.Name,
        // ... mais propriedades
        CreatedAt = customer.CreatedAt,
        UpdatedAt = customer.UpdatedAt
    };
}
```

### Depois (com mapper):
```csharp
public async Task<CustomerDto> CreateCustomerAsync(CreateCustomerDto dto)
{
    var customer = dto.ToEntity();
    customer.UserId = _customers.Count + 1;

    _customers.Add(customer);

    return customer.ToDto();
}
```

## Benefícios Alcançados

### 1. **Código Limpo**
- ✅ Redução de ~50% linhas de código nos serviços
- ✅ Eliminação de duplicação
- ✅ Métodos mais legíveis e concisos

### 2. **Manutenção Centralizada**
- ✅ Mudanças na estrutura de DTOs afetam apenas o mapper
- ✅ Lógica de conversão em um único lugar
- ✅ Fácil adicionar novas regras de conversão

### 3. **Performance**
- ✅ Sem reflexão em tempo de execução
- ✅ Compilação estática
- ✅ Métodos inline quando possível

## Boas Práticas

### 1. **Nomenclatura Consistente**
```csharp
// ✅ Bom
customer.ToDto()
dto.ToEntity()

// ❌ Ruim
customer.MapToDto()
dto.CreateFrom()
```

### 2. **Tratamento de Valores Nulos**
```csharp
public static CarDto ToDto(this Car car)
{
    if (car == null) return null;
    
    // Lógica de mapeamento
}
```

### 3. **Lógica de Negócio nos Mappers**
```csharp
// ✅ Aceitável: Cálculos simples
TimeSpan? duration = car.LastExit.HasValue && car.LastEntry.HasValue 
    ? car.LastExit.Value - car.LastEntry.Value 
    : null;

// ❌ Evitar: Lógica complexa de negócio
if (car.CustomerId == currentUser.Id && HasPermission())
{
    // Lógica complexa deve ficar no service
}
```

### 4. **Validações**
```csharp
public static Customer ToEntity(this CreateCustomerDto dto)
{
    // Validações básicas se necessário
    if (string.IsNullOrWhiteSpace(dto.Email))
        throw new ArgumentException("Email é obrigatório");
    
    return new Customer
    {
        Email = dto.Email,
        // ...
    };
}
```

## Extensibilidade

### Adicionando Novos Mappers

1. **Criar arquivo**: `{Entidade}Mapper.cs`
2. **Implementar métodos padrão**
3. **Adicionar lógica específica se necessário**
4. **Atualizar serviços para usar o mapper**

### Exemplo: ProductMapper
```csharp
public static class ProductMapper
{
    public static ProductDto ToDto(this Product product)
    {
        return new ProductDto
        {
            Id = product.Id,
            Name = product.Name,
            Price = product.Price,
            FormattedPrice = $"R$ {product.Price:F2}" // Formatação específica
        };
    }
    
    public static Product ToEntity(this CreateProductDto dto)
    {
        return new Product
        {
            Name = dto.Name,
            Price = dto.Price,
            CreatedAt = DateTime.UtcNow
        };
    }
}
```

## Conclusão

Os mappers são uma ferramenta poderosa para manter o código limpo, organizado e manutenível. Eles seguem os princípios SOLID e facilitam a evolução da aplicação, permitindo que as camadas de negócio permaneçam focadas em suas responsabilidades principais.
