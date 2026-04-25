# Sistema de Processamento de Transações Financeiras

![.NET](https://img.shields.io/badge/.NET-9.0-purple) ![SQL Server](https://img.shields.io/badge/SQL_Server-2022-blue) ![Docker](https://img.shields.io/badge/Docker-Ready-lightblue) ![Tests](https://img.shields.io/badge/Tests-62%20Passing-green) ![Clean Architecture](https://img.shields.io/badge/Architecture-Clean-success)

## 📋 Sobre o Projeto

Sistema de processamento de transações financeiras desenvolvido em **C# .NET 9** utilizando **Clean Architecture**. O sistema foi projetado para lidar com alto volume de transações concorrentes, garantindo **idempotência**, **atomicidade** e **integridade de dados**.

### 🎯 Características Principais

- ✅ **Clean Architecture** - Separação clara de responsabilidades
- ✅ **Controle de Concorrência** - Locks pessimistas para operações simultâneas
- ✅ **Idempotência** - Garantida via `reference_id`
- ✅ **Transações Atômicas** - Operações All-or-Nothing
- ✅ **Observabilidade** - Logs estruturados, métricas e health checks
- ✅ **Validação Robusta** - FluentValidation em todas as entradas
- ✅ **62 Testes** - Cobertura completa de casos de sucesso e falha
- ✅ **Docker** - Deploy simplificado com docker-compose

---

## 🚀 Como Executar

### Pré-requisitos

- **.NET 9 SDK** - [Download](https://dotnet.microsoft.com/download/dotnet/9.0)
- **Docker & Docker Compose** - [Download](https://www.docker.com/products/docker-desktop)
- **SQL Server** (opcional) - Se não usar Docker

### Opção 1: Executar com Docker (Recomendado)

```bash
# 1. Clonar o repositório
git clone <repository-url>
cd desafio-meta

# 2. Subir toda a infraestrutura (SQL Server + API)
docker-compose up -d

# 3. Executar as migrações do banco de dados
docker-compose exec api dotnet ef database update --project src/PagueVeloz.TransactionProcessor.Infrastructure

# A API estará disponível em:
# - HTTP: http://localhost:5000
# - HTTPS: https://localhost:5001
# - Swagger: https://localhost:5001/swagger
```

### Opção 2: Executar Localmente

```bash
# 1. Restaurar dependências
dotnet restore

# 2. Compilar o projeto
dotnet build

# 3. Executar testes
dotnet test

# 4. Executar migrações
dotnet ef database update --project src/PagueVeloz.TransactionProcessor.Infrastructure

# 5. Executar a API
dotnet run --project src/PagueVeloz.TransactionProcessor.API
```

---

## 📖 Como Usar a API

### Documentação Interativa

Acesse o Swagger UI após iniciar a API:
- **Swagger**: `http://localhost:5000/swagger`

### 1. Criar uma Conta

```bash
curl -X POST http://localhost:5000/api/accounts \
  -H "Content-Type: application/json" \
  -d '{
    "clientId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "initialBalance": 1000.00,
    "creditLimit": 500.00
  }'
```

**Resposta:**
```json
{
  "accountId": "550e8400-e29b-41d4-a716-446655440000",
  "clientId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "balance": 1000.00,
  "creditLimit": 500.00,
  "status": "active"
}
```

### 2. Consultar uma Conta

```bash
curl http://localhost:5000/api/accounts/550e8400-e29b-41d4-a716-446655440000
```

### 3. Processar uma Transação de Crédito

```bash
curl -X POST http://localhost:5000/api/transactions \
  -H "Content-Type: application/json" \
  -d '{
    "accountId": "550e8400-e29b-41d4-a716-446655440000",
    "referenceId": "TXN-CREDIT-001",
    "operation": "credit",
    "amount": 500.00,
    "currency": "BRL"
  }'
```

**Resposta:**
```json
{
  "transactionId": "660e8400-e29b-41d4-a716-446655440001",
  "status": "success",
  "balance": 1500.00,
  "reservedBalance": 0.00,
  "availableBalance": 1500.00,
  "timestamp": "2025-10-27T10:30:00Z",
  "errorMessage": null
}
```

### 4. Processar uma Transação de Débito

```bash
curl -X POST http://localhost:5000/api/transactions \
  -H "Content-Type: application/json" \
  -d '{
    "accountId": "550e8400-e29b-41d4-a716-446655440000",
    "referenceId": "TXN-DEBIT-001",
    "operation": "debit",
    "amount": 200.00,
    "currency": "BRL"
  }'
```

### 5. Operação de Reserva (Reserve)

```bash
curl -X POST http://localhost:5000/api/transactions \
  -H "Content-Type: application/json" \
  -d '{
    "accountId": "550e8400-e29b-41d4-a716-446655440000",
    "referenceId": "TXN-RESERVE-001",
    "operation": "reserve",
    "amount": 300.00,
    "currency": "BRL"
  }'
```

### 6. Capturar Reserva (Capture)

```bash
curl -X POST http://localhost:5000/api/transactions \
  -H "Content-Type: application/json" \
  -d '{
    "accountId": "550e8400-e29b-41d4-a716-446655440000",
    "referenceId": "TXN-CAPTURE-001",
    "operation": "capture",
    "amount": 300.00,
    "currency": "BRL"
  }'
```

### 7. Consultar Métricas do Sistema

```bash
curl http://localhost:5000/api/metrics
```

### 8. Consultar Health Check

```bash
curl http://localhost:5000/health
```

---

## 🎓 Tipos de Operações Financeiras

### 1. **Credit** (Crédito)
Adiciona valor ao saldo da conta.

**Exemplo:**
```json
{
  "operation": "credit",
  "accountId": "ACC-001",
  "amount": 1000.00,
  "currency": "BRL",
  "referenceId": "TXN-CREDIT-001"
}
```

### 2. **Debit** (Débito)
Remove valor do saldo da conta, considerando saldo disponível + limite de crédito.

**Exemplo:**
```json
{
  "operation": "debit",
  "accountId": "ACC-001",
  "amount": 200.00,
  "currency": "BRL",
  "referenceId": "TXN-DEBIT-001"
}
```

### 3. **Reserve** (Reservar)
Move valor do saldo disponível para o saldo reservado (usado em pagamentos pendentes).

**Exemplo:**
```json
{
  "operation": "reserve",
  "accountId": "ACC-001",
  "amount": 300.00,
  "currency": "BRL",
  "referenceId": "TXN-RESERVE-001"
}
```

### 4. **Capture** (Capturar)
Confirma uma reserva, debitando o valor da conta.

**Exemplo:**
```json
{
  "operation": "capture",
  "accountId": "ACC-001",
  "amount": 300.00,
  "currency": "BRL",
  "referenceId": "TXN-CAPTURE-001"
}
```

### 5. **Reversal** (Reversão)
Reverte uma operação anterior.

**Exemplo:**
```json
{
  "operation": "reversal",
  "accountId": "ACC-001",
  "amount": 100.00,
  "currency": "BRL",
  "referenceId": "TXN-REVERSAL-001",
  "metadata": {
    "originalTransactionId": "TXN-DEBIT-001"
  }
}
```

### 6. **Transfer** (Transferência)
Transfere valor entre duas contas.

**Exemplo:**
```json
{
  "operation": "transfer",
  "accountId": "ACC-001",
  "amount": 500.00,
  "currency": "BRL",
  "referenceId": "TXN-TRANSFER-001",
  "metadata": {
    "destinationAccountId": "ACC-002"
  }
}
```

---

## 🏗️ Arquitetura

### Clean Architecture

```
src/
├── PagueVeloz.TransactionProcessor.Domain/     # Camada de Domínio
│   ├── Entities/                                # Entidades de negócio
│   │   ├── Account.cs                           # Conta com lógica de negócio
│   │   ├── Client.cs                            # Cliente
│   │   └── Transaction.cs                       # Transação
│   └── Interfaces/                              # Contratos do domínio
│       ├── IRepository.cs
│       ├── IAccountRepository.cs
│       ├── ITransactionRepository.cs
│       └── IUnitOfWork.cs
│
├── PagueVeloz.TransactionProcessor.Application/ # Camada de Aplicação
│   ├── DTOs/                                    # Objetos de transferência
│   ├── Services/                                # Serviços de aplicação
│   │   ├── TransactionService.cs               # Orquestração de transações
│   │   ├── AccountService.cs
│   │   └── MetricsService.cs                   # Métricas do sistema
│   └── Validators/                             # FluentValidation
│       ├── CreateAccountRequestValidator.cs
│       └── ProcessTransactionRequestValidator.cs
│
├── PagueVeloz.TransactionProcessor.Infrastructure/ # Camada de Infraestrutura
│   ├── Data/                                   
│   │   └── TransactionProcessorDbContext.cs    # DbContext do EF Core
│   └── Repositories/                           
│       ├── AccountRepository.cs               # Implementação concreta
│       ├── TransactionRepository.cs
│       └── UnitOfWork.cs                       # Gerenciamento de transações
│
├── PagueVeloz.TransactionProcessor.API/       # Camada de Apresentação
│   ├── Controllers/                           
│   │   ├── AccountsController.cs              # Endpoints de contas
│   │   ├── TransactionsController.cs          # Endpoints de transações
│   │   └── MetricsController.cs               # Endpoints de métricas
│   ├── Middleware/                            
│   │   └── TransactionLoggingMiddleware.cs    # Logging com CorrelationId
│   └── Program.cs                              # Configuração da aplicação
│
└── PagueVeloz.TransactionProcessor.Tests/     # Camada de Testes
    ├── ConcurrencyTests.cs                    # Testes de concorrência
    └── UnitTests/                             # Testes unitários
        ├── AccountTests.cs
        ├── TransactionTests.cs
        └── ValidatorTests.cs
```

---

## 💡 Escolhas Técnicas

### 1. **Clean Architecture**

**Por quê?**
- Separação clara de responsabilidades facilita manutenção e testes
- Domínio independente de frameworks (EF Core, API, etc.)
- Facilita evolução para microsserviços futuros
- Torna o código mais testável e reutilizável

**Benefícios:**
- Cada camada tem sua responsabilidade bem definida
- Business logic isolada no Domain
- Testes unitários mais simples
- Facilita substituição de infraestrutura

### 2. **Locks Pessimistas (FOR UPDATE)**

**Por quê?**
- Garante integridade de dados sob concorrência
- Evita race conditions em operações financeiras
- Uma transação espera pela outra terminar
- Adequado para operações críticas (dinheiro)

**Implementação:**
```sql
SELECT * FROM Accounts WHERE Id = @AccountId FOR UPDATE
```

**Benefícios:**
- ✅ Garante consistência absoluta
- ✅ Evita deadlocks comitidos intencionalmente
- ✅ Performance adequada para o volume esperado
- ✅ 5 testes de concorrência validam o comportamento

### 3. **Idempotência via `reference_id`**

**Por quê?**
- Evita duplicação de transações em retries
- Cliente pode refazer requisição sem efeitos colaterais
- Essencial para sistemas financeiros
- Facilita debugging e auditoria

**Implementação:**
- Verifica se `reference_id` já existe no banco
- Se existe, retorna o resultado anterior
- Sempre falha ou sempre sucede de forma idêntica

**Benefícios:**
- ✅ Segurança contra duplicação
- ✅ Idempotência testada com 100+ requisições simultâneas
- ✅ Facilita troubleshooting

### 4. **Entity Framework Core com Migrations**

**Por quê?**
- Migrations facilitam versionamento de schema
- Code First mantém consistência entre código e banco
- Performance adequada para o volume esperado
- Suporta transações atômicas nativamente

**Benefícios:**
- ✅ Versionamento de schema simplificado
- ✅ Consistência garantida
- ✅ Migrations revertíveis
- ✅ Suporte a múltiplos bancos

### 5. **Repository Pattern + Unit of Work**

**Por quê?**
- Abstração de acesso a dados facilita testes
- Encapsula lógica de persistência
- Facilita mock em testes unitários
- Unit of Work garante consistência transacional

**Benefícios:**
- ✅ Separação clara de responsabilidades
- ✅ Testabilidade melhorada
- ✅ Facilita mudanças de ORM
- ✅ Transações mais explícitas

### 6. **FluentValidation**

**Por quê?**
- Validações declarativas e legíveis
- Separa validações de lógica de negócio
- Mensagens de erro customizáveis
- Testável independentemente

**Benefícios:**
- ✅ Código mais limpo e expressivo
- ✅ 31 testes de validação garantem cobertura
- ✅ Mensagens de erro claras para o usuário
- ✅ Validações reutilizáveis

### 7. **Serilog para Logging Estruturado**

**Por quê?**
- Logs estruturados facilitam análise e debugging
- CorrelationId vincula logs de uma transação
- Logs em arquivo e console
- Integração com ferramentas de observabilidade

**Benefícios:**
- ✅ Logs estruturados em JSON
- ✅ CorrelationId facilita rastreamento
- ✅ Auditoria completa de transações
- ✅ Integração com ELK/Grafana

### 8. **Docker & Docker Compose**

**Por quê?**
- Ambiente de desenvolvimento consistente
- Deploy simplificado
- Isolamento de dependências
- Reproduzibilidade garantida

**Benefícios:**
- ✅ Setup instantâneo: `docker-compose up`
- ✅ Sem conflitos com ambiente local
- ✅ Facilita CI/CD
- ✅ Simula ambiente de produção

### 9. **Health Checks**

**Por quê?**
- Monitoramento de saúde da aplicação
- Alertas proativos para problemas
- Integração com orquestradores (Kubernetes)
- Verifica conectividade com SQL Server

**Benefícios:**
- ✅ Endpoint `/health` para monitoramento
- ✅ Detecta problemas antes que afetem usuários
- ✅ Integração com ferramentas de monitoramento
- ✅ Status de dependências críticas

### 10. **Métricas Customizadas**

**Por quê?**
- Visibilidade em tempo real da performance
- Contadores de transações por operação
- Duração de operações (min, max, avg)
- Saldos de contas para auditoria

**Benefícios:**
- ✅ Endpoint `/api/metrics` para consulta
- ✅ Identificação de gargalos
- ✅ Monitoramento de padrões de uso
- ✅ Dados para otimização futura

---

## 🧪 Testes

### Executar Todos os Testes

```bash
dotnet test
```

### Executar Testes Específicos

```bash
# Apenas testes unitários
dotnet test --filter "FullyQualifiedName~UnitTests"

# Apenas testes de concorrência
dotnet test --filter "FullyQualifiedName~Concurrency"
```

### Cobertura de Testes

- ✅ **62 testes passando (100%)**
- ✅ **57 testes unitários** (Account, Transaction, Validators)
- ✅ **5 testes de concorrência** (100+ transações simultâneas)

#### Tipos de Testes

1. **Testes Unitários** - Lógica de negócio isolada
   - Validações de saldo
   - Cálculos de disponibilidade
   - Estados de transações

2. **Testes de Concorrência** - Operações simultâneas
   - Múltiplas transações na mesma conta
   - Validação de locks pessimistas
   - Idempotência concorrente
   - Integridade de dados

---

## 📊 Exemplos de Fluxos

### Fluxo 1: Crédito Simples

```bash
# 1. Criar conta
POST /api/accounts
{
  "clientId": "CLI-001",
  "initialBalance": 0,
  "creditLimit": 1000
}

# 2. Depositar crédito
POST /api/transactions
{
  "accountId": "ACC-001",
  "referenceId": "TXN-CREDIT-001",
  "operation": "credit",
  "amount": 500,
  "currency": "BRL"
}

# Resultado: Balance = 500, AvailableBalance = 500
```

### Fluxo 2: Débito com Limite de Crédito

```bash
# 1. Conta com saldo = 300, limite = 500
# 2. Débito de 600 (usa saldo + limite)

POST /api/transactions
{
  "accountId": "ACC-001",
  "referenceId": "TXN-DEBIT-001",
  "operation": "debit",
  "amount": 600,
  "currency": "BRL"
}

# Resultado: Balance = -300, AvailableBalance = 200 (saldo + limite)
```

### Fluxo 3: Reserva e Captura

```bash
# 1. Conta com 1000 disponível
# 2. Reservar 300 para pagamento pendente

POST /api/transactions
{
  "accountId": "ACC-001",
  "referenceId": "TXN-RESERVE-001",
  "operation": "reserve",
  "amount": 300,
  "currency": "BRL"
}

# Resultado: Balance = 1000, ReservedBalance = 300, AvailableBalance = 700

# 3. Capturar a reserva (confirma o pagamento)

POST /api/transactions
{
  "accountId": "ACC-001",
  "referenceId": "TXN-CAPTURE-001",
  "operation": "capture",
  "amount": 300,
  "currency": "BRL"
}

# Resultado: Balance = 700, ReservedBalance = 0, AvailableBalance = 700
```

---

## 🔒 Segurança e Validações

### Validações Implementadas

1. **Validação de Entrada**
   - `accountId`: Deve ser GUID válido
   - `amount`: Deve ser positivo e maior que zero
   - `currency`: Deve ter 3 caracteres (BRL, USD, EUR)
   - `referenceId`: Obrigatório, não vazio

2. **Validação de Conta**
   - Conta deve existir
   - Conta deve estar ativa
   - Cliente da conta deve estar ativo

3. **Validação de Saldo**
   - Débito não pode exceder saldo + limite
   - Reserva não pode exceder saldo disponível
   - Captura não pode exceder saldo reservado

4. **Prevenção de SQL Injection**
   - Uso de parâmetros parametrizados
   - EF Core cuida automaticamente

5. **Auditoria**
   - Logs de todas as operações
   - IP e User-Agent capturados
   - Timestamp de todas as transações

---

## 📈 Monitoramento

### Endpoints de Monitoramento

- **Health Check**: `GET /health`
- **Métricas**: `GET /api/metrics`
- **Logs**: Arquivo em `logs/pagueveloz-YYYY-MM-DD.txt`

### Logs Estruturados

Os logs incluem:
- CorrelationId (para rastreamento)
- IP do cliente
- User-Agent
- Timestamp
- Nível de log (Info, Warning, Error)

---

## 🐛 Troubleshooting

### Problema: Banco de dados não conecta

**Solução:**
```bash
# Verificar se SQL Server está rodando
docker-compose ps

# Ver logs do SQL Server
docker-compose logs sqlserver

# Recriar banco de dados
docker-compose down
docker-compose up -d
```

### Problema: Migrations não aplicadas

**Solução:**
```bash
# Aplicar migrations manualmente
dotnet ef database update --project src/PagueVeloz.TransactionProcessor.Infrastructure
```

### Problema: Porta já em uso

**Solução:**
Edite `docker-compose.yml` e altere as portas:
```yaml
ports:
  - "5002:8080"  # Altere 5000 para 5002
```

---

## 📚 Documentação Adicional

- **Especificação**: `docs/specification.md`
- **Tarefas**: `docs/tasks.md`
- **Prompt**: `docs/prompt.md`
