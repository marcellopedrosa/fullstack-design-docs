# OAuth2

Este módulo implementa o protocolo OAuth 2.0 para autenticação e autorização no CSV Service, fornecendo uma camada de abstração sobre a implementação específica do Keycloak.

## Estrutura

```
oauth2/
├── guards/            # Guards OAuth2
├── decorators/        # Decorators de autorização
├── types/             # Types e interfaces
└── utils/             # Utilitários OAuth2
```

## Tipos e Interfaces

### Token Content

```typescript
export interface TokenContentType {
  preferred_username: string;
  email?: string;
  sub: string;
  client_id?: string;
  realm_access?: {
    roles: string[];
  };
  resource_access?: Record<string, { roles: string[] }>;
}
```

### OAuth2 Config

```typescript
export interface OAuth2Config {
  clientId: string;
  clientSecret: string;
  tokenUrl: string;
  authorizationUrl: string;
  userInfoUrl: string;
  scope: string[];
  grantTypes: string[];
}
```

## Guards

### OAuth2 Guard

```typescript
@Injectable()
export class OAuth2Guard implements CanActivate {
  constructor(
    private readonly reflector: Reflector,
    private readonly tokenService: TokenService
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const isPublic = this.reflector.getAllAndOverride<boolean>('isPublic', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) {
      return true;
    }

    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);
    
    if (!token) {
      throw new UnauthorizedException();
    }

    try {
      const payload = await this.tokenService.validateToken(token);
      request['user'] = payload;
      return true;
    } catch {
      throw new UnauthorizedException();
    }
  }

  private extractTokenFromHeader(request: Request): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}
```

### Scope Guard

```typescript
@Injectable()
export class ScopeGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredScopes = this.reflector.getAllAndOverride<string[]>('scopes', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredScopes) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();
    return this.matchScopes(requiredScopes, user.scope);
  }

  private matchScopes(required: string[], userScopes: string): boolean {
    const scopeArray = userScopes.split(' ');
    return required.every(scope => scopeArray.includes(scope));
  }
}
```

## Decorators

### Scope Decorator

```typescript
export const Scopes = (...scopes: string[]) => SetMetadata('scopes', scopes);

// Uso
@Scopes('read:attributes', 'write:attributes')
@Get('/protected')
async protectedRoute() {
  // ...
}
```

### Public Endpoint

```typescript
export const Public = () => SetMetadata('isPublic', true);

// Uso
@Public()
@Get('/public')
async publicRoute() {
  // ...
}
```

## Token Handling

### Token Validation

```typescript
@Injectable()
export class TokenValidationService {
  constructor(private readonly configService: ConfigService) {}

  async validateToken(token: string): Promise<TokenContentType> {
    try {
      const decoded = jwt.verify(
        token,
        this.configService.get('JWT_PUBLIC_KEY'),
        {
          algorithms: ['RS256']
        }
      );

      return decoded as TokenContentType;
    } catch (error) {
      if (error instanceof jwt.TokenExpiredError) {
        throw new TokenExpiredException();
      }
      throw new TokenValidationException();
    }
  }

  extractScopes(token: TokenContentType): string[] {
    return token.scope?.split(' ') || [];
  }
}
```

### Token Renewal

```typescript
@Injectable()
export class TokenRenewalService {
  constructor(
    private readonly httpService: HttpService,
    private readonly configService: ConfigService
  ) {}

  async renewToken(refreshToken: string): Promise<TokenResponse> {
    const tokenUrl = this.configService.get('OAUTH_TOKEN_URL');
    const clientId = this.configService.get('OAUTH_CLIENT_ID');
    const clientSecret = this.configService.get('OAUTH_CLIENT_SECRET');

    const params = new URLSearchParams({
      grant_type: 'refresh_token',
      refresh_token: refreshToken,
      client_id: clientId,
      client_secret: clientSecret
    });

    try {
      const response = await this.httpService
        .post(tokenUrl, params.toString(), {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        })
        .toPromise();

      return response.data;
    } catch (error) {
      throw new TokenRenewalException();
    }
  }
}
```

## Integração com NestJS

### Module Configuration

```typescript
@Module({
  imports: [
    HttpModule,
    ConfigModule
  ],
  providers: [
    TokenValidationService,
    TokenRenewalService,
    {
      provide: APP_GUARD,
      useClass: OAuth2Guard
    }
  ],
  exports: [
    TokenValidationService,
    TokenRenewalService
  ]
})
export class OAuth2Module {}
```

## Error Handling

### Custom Exceptions

```typescript
export class OAuth2Exception extends UnauthorizedException {
  constructor(message: string = 'OAuth2 authentication failed') {
    super(message);
  }
}

export class ScopeException extends ForbiddenException {
  constructor(message: string = 'Insufficient scope') {
    super(message);
  }
}
```

## Boas Práticas

1. **Segurança**
   - Valide todos os tokens JWT
   - Use HTTPS para todas as comunicações
   - Implemente rate limiting
   - Gerencie refresh tokens com segurança

2. **Performance**
   - Cache de chaves públicas
   - Validação eficiente de tokens
   - Otimize renovações de token
   - Monitore uso de recursos

3. **Manutenção**
   - Centralize configuração OAuth2
   - Documente scopes e roles
   - Mantenha logs de autenticação
   - Atualize dependências regularmente

4. **Integração**
   - Padronize formato de tokens
   - Use tipos fortemente tipados
   - Implemente retry com backoff
   - Mantenha compatibilidade

## Monitoramento

### Métricas
- Taxa de sucesso de autenticação
- Tempo de validação de tokens
- Uso de refresh tokens
- Erros de autenticação

### Logs
```typescript
{
  timestamp: string;
  level: 'info' | 'warn' | 'error';
  event: 'token_validation' | 'token_renewal' | 'authentication';
  success: boolean;
  error?: string;
  user?: string;
  scopes?: string[];
}
```

## Troubleshooting

### Problemas Comuns

1. **Token Inválido**
- Verifique formato JWT
- Valide assinatura
- Confira expiração
- Verifique emissor

2. **Scope Insuficiente**
- Compare scopes requeridos
- Verifique permissões
- Valide roles
- Confira configuração

## Referências

- [OAuth 2.0 RFC](https://tools.ietf.org/html/rfc6749)
- [JWT RFC](https://tools.ietf.org/html/rfc7519)
- [NestJS Authentication](https://docs.nestjs.com/security/authentication)
- [OAuth 2.0 Security](https://oauth.net/articles/authentication/)