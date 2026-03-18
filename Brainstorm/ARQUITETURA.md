Arquitetura do Projeto Life-OS (Monorepo)
Padrão: Clean Architecture + Microsserviços + Kafka
Stack: Java 21, Quarkus, Maven




![[Pasted image 20260316220313.png]]


```
LifeOS-Monorepo/
├── pom.xml                        # Parent POM: Gerencia as versões das dependências para todos os serviços
├── docker-compose.yml             
│
├── api-gateway/                   # Microsserviço de Entrada
│   ├── pom.xml
│   └── src/main/java/br/com/lifeos/gateway/
│       └── GatewayRoutes.java     # Roteamento central das requisições para os microsserviços
│
├── shared-contracts/              # Biblioteca compartilhada (Apenas Contratos, SEM regra de negócio)
│   ├── pom.xml
│   └── src/main/java/br/com/lifeos/shared/
│       └── events/
│           ├── UserDeletedEvent.java      # Modelos de mensagens que viajam pelo Kafka
│           └── TransactionCreatedEvent.java
│
├── finance-service/               # Exemplo de Microsserviço (Os outros seguem a mesma estrutura)
│   ├── pom.xml
│   └── src/main/
│       ├── resources/
│       │   └── application.properties     # Credenciais do DB, Configurações do Kafka e JWT
│       │
│       └── java/br/com/lifeos/finance/
│
│           ├── api/                       # CAMADA 1: Apresentação (Porta de Entrada REST)
│           │   ├── controllers/
│           │   │   └── TransactionController.java  # Recebe HTTP (GET, POST), valida JWT e chama o Application
│           │   └── dtos/
│           │       ├── CreateTransactionDTO.java   # "Escudo": Filtra o JSON recebido do React
│           │       └── TransactionResponseDTO.java # Formata a saída de dados para o React
│           │
│           ├── application/               # CAMADA 2: Aplicação (Casos de Uso / Orquestração)
│           │   ├── services/
│           │   │   └── TransactionService.java     # Recebe o DTO, aplica regras, aciona o Domain e o Repository
│           │   └── mappers/
│           │       └── TransactionMapper.java      # Converte DTO <-> Entity
│           │
│           ├── domain/                    # CAMADA 3: Domínio (Coração do Negócio - Puro Java)
│           │   ├── entities/
│           │   │   └── Transaction.java            # Entidade principal com regras financeiras
│           │   ├── exceptions/
│           │   │   └── InvalidAmountException.java # Erros específicos do negócio
│           │   └── repositories/
│           │       └── TransactionRepository.java  # INTERFACE: Define que a transação deve ser salva (Contrato)
│           │
│           └── infrastructure/            # CAMADA 4: Infraestrutura (O Mundo Externo)
│               ├── database/
│               │   ├── entities/
│               │   │   └── TransactionPanacheEntity.java # Mapeamento da tabela do Postgres (Hibernate)
│               │   └── repositories/
│               │       └── TransactionRepositoryImpl.java # Implementa a Interface do Domain salvando no BD
│               │
│               └── messaging/
│                   ├── KafkaProducer.java          # Publica eventos no Broker
│                   └── KafkaConsumer.java          # Ouve eventos de outros serviços (ex: UserDeleted)
|
├── user-service/                  # Microsserviço de Usuários (Postgres)
│   └── ... (mesma estrutura interna)
├── routine-service/               # Microsserviço de Rotina e Estudos (Postgres + Redis)
│   └── ... (mesma estrutura interna)
├── economic-service/              # Microsserviço Macroeconômico (MongoDB + Redis)
│   └── ... (mesma estrutura interna)
└── health-service/                # Microsserviço de Treinos (MongoDB)
    └── ... (mesma estrutura interna)
```


