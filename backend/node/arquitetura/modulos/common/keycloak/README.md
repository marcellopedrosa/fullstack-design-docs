# Keycloak Integration

Este módulo fornece a integração com Keycloak para autenticação e autorização no CSV Service, implementando Single Sign-On (SSO) e gerenciamento de identidade.

## Estrutura

```
keycloak/
├── config/            # Configurações do Keycloak
├── guards/            # Guards de autenticação
├── decorators/        # Decorators personalizados
└── services/          # Serviços de integração
```

## Configuração

### Keycloak Provider

```typescript
@Injectable()
export class KeycloakConfigService {
  constructor(private configService: ConfigService) {}

  get config(): KeycloakConnectOptions {
    return {
      authServerUrl: this.configService.get<string>('KEYCLOAK_URL'),
      realm: this.configService.get<string>('KEYCLOAK_REALM'),
      clientId: this.configService.get<string>('KEYCLOAK_CLIENT_ID'),
      secret: this.configService.get<string>('KEYCLOAK_CLIENT_SECRET'),
      bearerOnly: true,
      'ssl-required': 'external',
      'confidential-port': 0
    };
  }
}
```

### Environment Variables
```env
KEYCLOAK_URL=https://auth.example.com
KEYCLOAK_REALM=csv-service
KEYCLOAK_CLIENT_ID=backend-service
KEYCLOAK_CLIENT_SECRET=your-secret
```

## Authentication Guards

### JWT Authentication Guard

```typescript
@Injectable()
export class KeycloakAuthGuard implements CanActivate {
  constructor(
    @Inject('KEYCLOAK_INSTANCE')
    private readonly keycloak: any,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    if (!token) {
      throw new UnauthorizedException('No token provided');
    }

    try {
      const grant = await this.keycloak.grantManager.createGrant({
        access_token: token,
      });
      
      request.user = grant.access_token.content;
      return true;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }

  private extractToken(request: Request): string | null {
    const authHeader = request.headers['authorization'];
    return authHeader?.split(' ')[1] || null;
  }
}
```

### Role-Based Guard

```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    @Inject('KEYCLOAK_INSTANCE')
    private readonly keycloak: any,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const requiredRoles = this.reflector.getAllAndMerge<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles.length) {
      return true;
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user;

    return this.matchRoles(requiredRoles, user.realm_access.roles);
  }

  private matchRoles(required: string[], user: string[]): boolean {
    return required.some(role => user.includes(role));
  }
}
```

## Decorators

### Role Decorator

```typescript
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

// Uso
@Roles('admin', 'manager')
@Get('/protected')
async protectedRoute() {
  // ...
}
```

### Public Route Decorator

```typescript
export const Public = () => SetMetadata('isPublic', true);

// Uso
@Public()
@Get('/public')
async publicRoute() {
  // ...
}
```

## Token Management

### Token Service

```typescript
@Injectable()
export class TokenService {
  constructor(
    @Inject('KEYCLOAK_INSTANCE')
    private readonly keycloak: any,
  ) {}

  async validateToken(token: string): Promise<boolean> {
    try {
      const grant = await this.keycloak.grantManager.createGrant({
        access_token: token,
      });
      return grant.isValid();
    } catch {
      return false;
    }
  }

  async decodeToken(token: string): Promise<any> {
    const grant = await this.keycloak.grantManager.createGrant({
      access_token: token,
    });
    return grant.access_token.content;
  }

  async refreshToken(refreshToken: string): Promise<any> {
    return this.keycloak.grantManager.refreshToken(refreshToken);
  }
}
```

## User Management

### User Service

```typescript
@Injectable()
export class KeycloakUserService {
  constructor(
    private readonly httpService: HttpService,
    private readonly configService: KeycloakConfigService,
  ) {}

  async getUserInfo(token: string): Promise<any> {
    const config = this.configService.config;
    const url = `${config.authServerUrl}/realms/${config.realm}/protocol/openid-connect/userinfo`;
    
    const response = await this.httpService.get(url, {
      headers: { Authorization: `Bearer ${token}` },
    }).toPromise();
    
    return response.data;
  }

  async getRoles(token: string): Promise<string[]> {
    const userInfo = await this.getUserInfo(token);
    return userInfo.realm_access?.roles || [];
  }
}
```

## Error Handling

### Token Exceptions

```typescript
export class TokenValidationException extends UnauthorizedException {
  constructor(message = 'Invalid token') {
    super(message);
  }
}

export class TokenExpiredException extends UnauthorizedException {
  constructor(message = 'Token expired') {
    super(message);
  }
}
```

## Boas Práticas

1. **Segurança**
   - Sempre use HTTPS em produção
   - Valide tokens em cada requisição
   - Implemente renovação automática de tokens
   - Gerencie sessões adequadamente

2. **Performance**
   - Cache de validação de tokens
   - Otimize requisições ao Keycloak
   - Implemente circuit breaker
   - Monitore tempos de resposta

3. **Manutenção**
   - Mantenha configurações externalizadas
   - Documente roles e permissões
   - Monitore uso de recursos
   - Implemente logging adequado

4. **Integração**
   - Use clients dedicados por ambiente
   - Implemente retry com backoff
   - Valide configurações no startup
   - Mantenha fallbacks apropriados

## Monitoramento

### Métricas Importantes
- Tempo de validação de tokens
- Taxa de rejeição de tokens
- Uso de recursos do Keycloak
- Latência de autenticação

### Logs
- Tentativas de acesso negadas
- Erros de validação de token
- Renovações de token
- Mudanças de configuração

## Troubleshooting

### Problemas Comuns

1. **Token Inválido**
```typescript
// Verificar formato do token
const isValidFormat = /^[A-Za-z0-9-_]+\.[A-Za-z0-9-_]+\.[A-Za-z0-9-_]*$/;
if (!isValidFormat.test(token)) {
  throw new TokenValidationException('Invalid token format');
}
```

2. **Roles Incorretas**
```typescript
// Debug de roles
this.logger.debug({
  requiredRoles,
  userRoles: user.realm_access.roles,
  hasAccess: this.matchRoles(requiredRoles, user.realm_access.roles)
});
```

## Referências

- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [NestJS Security](https://docs.nestjs.com/security/authentication)
- [OAuth 2.0](https://oauth.net/2/)
- [JWT Best Practices](https://datatracker.ietf.org/doc/html/rfc8725)