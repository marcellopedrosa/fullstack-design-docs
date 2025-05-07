# Database

Este módulo fornece a configuração e implementação da camada de persistência do CSV Service, utilizando TypeORM para interação com o banco de dados PostgreSQL.

## Estrutura

```
database/
├── config/              # Configurações do TypeORM
├── migrations/          # Migrações do banco de dados
├── repositories/        # Repositórios base
└── utils/               # Utilitários de banco de dados
```

## Configuração TypeORM

### Conexão
```typescript
{
  type: 'postgres',
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT, 10),
  username: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  autoLoadEntities: true,
  synchronize: process.env.NODE_ENV !== 'production'
}
```

### Pool de Conexões
```typescript
{
  // Pool Configuration
  extra: {
    max: 20,                 // Máximo de conexões no pool
    min: 2,                  // Mínimo de conexões mantidas
    idleTimeoutMillis: 30000 // Timeout para conexões ociosas
  }
}
```

## Migrations

### Criação
```bash
npm run migration:create -- -n NomeDaMigracao
```

### Estrutura
```typescript
export class NomeDaMigracao implements MigrationInterface {
    public async up(queryRunner: QueryRunner): Promise<void> {
        // Alterações a serem aplicadas
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
        // Rollback das alterações
    }
}
```

### Execução
- Development: `npm run migration:run`
- Production: Via pipeline CI/CD

## Repositories

### Base Repository
```typescript
export abstract class BaseRepository<T> {
    constructor(
        @InjectRepository(EntityClass)
        protected readonly repository: Repository<T>
    ) {}

    async findById(id: string): Promise<T> {
        const entity = await this.repository.findOne({ where: { id } });
        if (!entity) {
            throw new NotFoundException();
        }
        return entity;
    }

    // ... outros métodos comuns
}
```

### Extensão
```typescript
@Injectable()
export class CustomRepository extends BaseRepository<CustomEntity> {
    constructor(
        @InjectRepository(CustomEntity)
        repository: Repository<CustomEntity>
    ) {
        super(repository);
    }

    // Métodos específicos
}
```

## Transactions

### Gerenciamento de Transações

```typescript
@Injectable()
export class TransactionService {
    constructor(
        @InjectConnection()
        private connection: Connection
    ) {}

    async executeInTransaction<T>(
        operation: (entityManager: EntityManager) => Promise<T>
    ): Promise<T> {
        const queryRunner = this.connection.createQueryRunner();
        await queryRunner.connect();
        await queryRunner.startTransaction();

        try {
            const result = await operation(queryRunner.manager);
            await queryRunner.commitTransaction();
            return result;
        } catch (err) {
            await queryRunner.rollbackTransaction();
            throw err;
        } finally {
            await queryRunner.release();
        }
    }
}
```

### Uso em Serviços
```typescript
@Injectable()
export class CustomService {
    constructor(private transactionService: TransactionService) {}

    async operacaoTransacional() {
        return this.transactionService.executeInTransaction(async (manager) => {
            // Operações dentro da transação
        });
    }
}
```

## Boas Práticas

### 1. Segurança
- Sempre use parâmetros preparados (prepared statements)
- Implemente controle de acesso por linha quando necessário
- Encrypt dados sensíveis antes de persistir
- Use conexões SSL em produção

### 2. Performance
- Indexe colunas frequentemente consultadas
- Otimize consultas complexas
- Monitore o uso do pool de conexões
- Implemente caching quando apropriado

### 3. Manutenção
- Mantenha migrações idempotentes
- Documente alterações no schema
- Faça backup regular dos dados
- Monitore o crescimento do banco

### 4. Convenções
- Use snake_case para nomes de tabelas e colunas
- Prefixe tabelas por domínio (ex: auth_users)
- Mantenha consistência nos tipos de dados
- Use UUIDs para chaves primárias

## Monitoramento

### Métricas Importantes
- Tempo de resposta de queries
- Uso de conexões do pool
- Taxa de cache hits/misses
- Crescimento do banco de dados

### Logs
- Queries lentas (> 1 segundo)
- Erros de conexão
- Falhas em transações
- Alterações no schema

## Troubleshooting

### Problemas Comuns

1. **Conexões Pendentes**
```sql
SELECT * FROM pg_stat_activity 
WHERE state = 'idle' 
AND state_change < NOW() - INTERVAL '30 minutes';
```

2. **Queries Lentas**
```sql
SELECT pid, query, query_start 
FROM pg_stat_activity 
WHERE state = 'active' 
AND query_start < NOW() - INTERVAL '5 minutes';
```

### Soluções
- Ajuste do pool de conexões
- Otimização de índices
- Vacuum e analyze regulares
- Monitoramento proativo

## Referências

- [TypeORM Documentation](https://typeorm.io/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [NestJS Database](https://docs.nestjs.com/techniques/database)
- [Node-Postgres Best Practices](https://node-postgres.com/guides/project-structure)