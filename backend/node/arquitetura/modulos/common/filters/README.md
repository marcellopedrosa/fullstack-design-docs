# Filters

O módulo de filtros fornece tratamento centralizado de exceções e transformação de respostas para o CSV Service, garantindo consistência no formato das respostas e logging adequado de erros.

## Estrutura

```
filters/
├── exception/          # Filtros de exceção
├── http/               # Filtros HTTP
└── validation/         # Filtros de validação
```

## Exception Filters

### Global Exception Filter

```typescript
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(private readonly logger: Logger) {}

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const errorResponse = this.createErrorResponse(exception, request);
    
    this.logger.error({
      path: request.url,
      method: request.method,
      error: errorResponse
    });

    response
      .status(errorResponse.statusCode)
      .json(errorResponse);
  }
}
```

### Tipos de Exceções Tratadas

1. **HTTP Exceptions**
   - BadRequestException
   - UnauthorizedException
   - ForbiddenException
   - NotFoundException
   - ConflictException

2. **Validação**
   - ValidationException
   - InvalidInputException
   - ConstraintViolationException

3. **Negócio**
   - BusinessException
   - DomainException
   - AttributeException

4. **Sistema**
   - DatabaseException
   - IntegrationException
   - ConfigurationException

## HTTP Filters

### Transformação de Resposta

```typescript
@Injectable()
export class ResponseTransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        data,
        meta: {
          timestamp: new Date().toISOString(),
          path: context.switchToHttp().getRequest().url
        }
      }))
    );
  }
}
```

### Formato de Resposta

1. **Sucesso**
```json
{
  "data": {
    // payload principal
  },
  "meta": {
    "timestamp": "2025-05-07T10:00:00Z",
    "path": "/api/v1/attributes"
  }
}
```

2. **Erro**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Erro de validação nos dados de entrada",
    "details": [
      {
        "field": "nome",
        "message": "O nome é obrigatório"
      }
    ]
  },
  "meta": {
    "timestamp": "2025-05-07T10:00:00Z",
    "path": "/api/v1/attributes",
    "requestId": "abc-123"
  }
}
```

## Validation Filters

### Class Validator Integration

```typescript
@Injectable()
export class ValidationPipe implements PipeTransform<any> {
  async transform(value: any, metadata: ArgumentMetadata) {
    const object = plainToClass(metadata.metatype, value);
    const errors = await validate(object);
    
    if (errors.length > 0) {
      throw new ValidationException({
        message: 'Validation failed',
        details: this.formatErrors(errors)
      });
    }
    
    return value;
  }
}
```

### Validações Customizadas

```typescript
@Injectable()
export class CustomValidationPipe extends ValidationPipe {
  constructor(
    private readonly customValidator: CustomValidatorService
  ) {
    super();
  }

  async transform(value: any, metadata: ArgumentMetadata) {
    // Validação padrão
    await super.transform(value, metadata);
    
    // Validações customizadas
    await this.customValidator.validate(value, metadata);
    
    return value;
  }
}
```

## Logging Middleware

### Request Logging

```typescript
@Injectable()
export class LoggingMiddleware implements NestMiddleware {
  constructor(private readonly logger: Logger) {}

  use(req: Request, res: Response, next: Function) {
    const startTime = Date.now();
    const requestId = uuid();

    // Log request
    this.logger.debug({
      type: 'request',
      method: req.method,
      path: req.path,
      requestId
    });

    // Response logging
    res.on('finish', () => {
      const duration = Date.now() - startTime;
      this.logger.debug({
        type: 'response',
        method: req.method,
        path: req.path,
        statusCode: res.statusCode,
        duration,
        requestId
      });
    });

    next();
  }
}
```

## Configuração

### Registro Global

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Exception Filter
  app.useGlobalFilters(new GlobalExceptionFilter());

  // Response Transform
  app.useGlobalInterceptors(new ResponseTransformInterceptor());

  // Validation
  app.useGlobalPipes(new ValidationPipe());

  // Logging
  app.use(new LoggingMiddleware());
}
```

## Integração com Observabilidade

### Correlação de Logs

```typescript
@Injectable()
export class CorrelationMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: Function) {
    const correlationId = req.headers['x-correlation-id'] || uuid();
    req['correlationId'] = correlationId;
    
    // Adiciona correlationId ao contexto de logging
    ClsService.set('correlationId', correlationId);
    
    res.setHeader('x-correlation-id', correlationId);
    next();
  }
}
```

## Boas Práticas

1. **Tratamento de Erros**
   - Sempre forneça mensagens de erro claras
   - Inclua detalhes úteis para debugging
   - Mantenha consistência no formato
   - Evite expor detalhes sensíveis

2. **Logging**
   - Log todas as exceções não tratadas
   - Inclua contexto suficiente
   - Use níveis de log apropriados
   - Mantenha informações de rastreamento

3. **Validação**
   - Valide entrada em todas as rotas
   - Use decorators para validações comuns
   - Implemente validações customizadas
   - Mantenha mensagens de erro claras

4. **Performance**
   - Cache resultados de validações
   - Otimize transformações de resposta
   - Monitore overhead de logging
   - Use sampling quando apropriado

## Referências

- [NestJS Exception Filters](https://docs.nestjs.com/exception-filters)
- [Class Validator](https://github.com/typestack/class-validator)
- [NestJS Pipes](https://docs.nestjs.com/pipes)
- [NestJS Middleware](https://docs.nestjs.com/middleware)