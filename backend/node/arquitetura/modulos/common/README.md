# Módulo Common

O módulo `common` é responsável por fornecer funcionalidades compartilhadas e infraestrutura base para todo o projeto CSV Service. Este módulo implementa padrões, utilitários e configurações que são utilizados por outros módulos do sistema.

## Estrutura do Módulo

```
src/common/
├── app.controller.ts
├── common.module.ts
├── database/       # Configuração e utilitários de banco de dados
├── filters/        # Filtros globais de exceção
├── keycloak/       # Integração com Keycloak
├── oauth2/         # Implementação OAuth2
├── observability/  # Telemetria e monitoramento
└── providers/      # Providers comuns
```

## Submodules

### 1. Database
[Detalhes em database/README.md](./database/README.md)
- Configuração do TypeORM
- Migrations
- Repositories base
- Conexões e pools
- Transactions

### 2. Filters
[Detalhes em filters/README.md](./filters/README.md)
- Exception filters
- Error handling
- Response transformers
- Logging middleware

### 3. Keycloak
[Detalhes em keycloak/README.md](./keycloak/README.md)
- Autenticação
- Autorização
- Gerenciamento de tokens
- SSO integration

### 4. OAuth2
[Detalhes em oauth2/README.md](./oauth2/README.md)
- Implementação OAuth2
- Roles e permissões
- Token validation
- Guards

### 5. Observability
[Detalhes em observability/README.md](./observability/README.md)
- Logging
- Métricas
- Tracing
- Health checks

### 6. Providers
[Detalhes em providers/README.md](./providers/README.md)
- Factory providers
- Service providers
- Configuration providers
- Utility providers

## Responsabilidades

O módulo `common` tem as seguintes responsabilidades principais:

1. **Infraestrutura Base**
   - Fornecer configurações básicas do NestJS
   - Gerenciar conexões com banco de dados
   - Configurar autenticação e autorização

2. **Cross-Cutting Concerns**
   - Logging centralizado
   - Tratamento de erros
   - Métricas e monitoramento
   - Validações comuns

3. **Reusabilidade**
   - Providers compartilhados
   - Decorators utilitários
   - Interfaces comuns
   - DTOs base

4. **Segurança**
   - Configuração de autenticação
   - Autorização baseada em roles
   - Proteção contra ataques comuns
   - Validação de tokens

## Configuração e Uso

### Importação do Módulo

```typescript
import { CommonModule } from '@common/common.module';

@Module({
  imports: [CommonModule],
  // ...
})
export class YourModule {}
```

### Injeção de Dependências

```typescript
export class YourService {
  constructor(
    private readonly providerFactory: ProviderFactoryPort,
    // outros providers comuns...
  ) {}
}
```

## Boas Práticas

1. **Modularização**
   - Mantenha funcionalidades relacionadas agrupadas
   - Evite dependências circulares
   - Use interfaces para abstração

2. **Configuração**
   - Use environment variables
   - Centralize configurações
   - Valide valores no startup

3. **Logging e Monitoramento**
   - Log eventos importantes
   - Monitore performance
   - Configure alertas

4. **Segurança**
   - Valide todas as entradas
   - Use HTTPS sempre
   - Implemente rate limiting

## Referências

- [NestJS Documentation](https://docs.nestjs.com/)
- [TypeORM Documentation](https://typeorm.io/)
- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [OAuth 2.0 Specification](https://oauth.net/2/)