# Arquitetura do CSV Service

// ...existing code...

## Registros no app.module.ts

Todo novo componente desenvolvido no sistema (Controllers, Use Cases, Repositories, etc.) DEVE ser registrado no `app.module.ts` localizado na raiz do projeto. Este é um requisito obrigatório para que o NestJS possa gerenciar corretamente as dependências.

### Padrão de Registro

Para cada novo módulo ou funcionalidade, você deve:

1. **Importar as Dependências**
```typescript
import NewController from '@src/module/new.controller';
import NewPresenterImpl from '@src/module/presenters/new-presenter-impl';
import NewPresenterPort from '@src/module/presenters/new-presenter-port';
import NewRepositoryImpl from '@src/module/repositories/new-repository-impl';
import NewRepositoryPort from '@src/module/repositories/new-repository-port';
import NewUseCaseImpl from '@src/module/use-cases/new-use-case-impl';
```

2. **Registrar no Decorador @Module**
```typescript
@Module({
  controllers: [
    // ...outros controllers
    NewController,
  ],
  providers: [
    // Presenter
    {
      provide: NewPresenterImpl,
      useClass: NewPresenterImpl,
    },
    
    // Repository
    {
      provide: NewRepositoryImpl,
      useFactory: (httpService: HttpService, providerFactoryPort: ProviderFactoryPort) => {
        return new NewRepositoryImpl(httpService, providerFactoryPort);
      },
      inject: [HttpService, ProviderFactoryImpl],
    },
    
    // Use Case
    {
      provide: NewUseCaseImpl,
      useFactory: (
        repositoryPort: NewRepositoryPort,
        presenterPort: NewPresenterPort,
        // outras dependências...
      ) => {
        return new NewUseCaseImpl(
          repositoryPort,
          presenterPort,
          // outras dependências...
        );
      },
      inject: [
        NewRepositoryImpl,
        NewPresenterImpl,
        // outras injeções...
      ],
    },
  ],
})
```

### Convenções de Registro

1. **Agrupamento**: Mantenha os registros relacionados agrupados com comentários claros
   ```typescript
   ```

2. **Ordem de Registro**: 
   - Primeiro registre os Presenters
   - Em seguida os Repositories
   - Por último os Use Cases

3. **Injeção de Dependências**:
   - Use `useClass` para injeções simples
   - Use `useFactory` quando precisar de configuração ou dependências adicionais
   - Liste todas as injeções necessárias no array `inject`

### Exemplo Real do Projeto

```typescript
{
  provide: DailyEnergyInLastWeekPresenterImpl,
  useClass: DailyEnergyInLastWeekPresenterImpl,
},
{
  provide: DailyEnergyInLastWeekWebClientRepositoryImpl,
  useFactory: (
    httpService: HttpService,
    providerFactoryPort: ProviderFactoryPort,
  ) => {
    return new DailyEnergyInLastWeekWebClientRepositoryImpl(
      httpService,
      providerFactoryPort,
    );
  },
  inject: [HttpService, ProviderFactoryImpl],
},
{
  provide: DetailDailyEnergyInLastWeekUseCaseImpl,
  useFactory: (/*...dependências...*/) => {
    return new DetailDailyEnergyInLastWeekUseCaseImpl(/*...parâmetros...*/);
  },
  inject: [/*...injeções...*/],
}
```

// ...existing code...