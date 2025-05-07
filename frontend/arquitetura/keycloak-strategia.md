# Estratégia 2: Utilização da Interface de Login do Keycloak

## Visão Geral

Esta estratégia envolve a utilização completa da interface de login nativa do Keycloak, delegando todo o processo de autenticação para o servidor Keycloak. O CSC redirecionará o usuário para a página de login do Keycloak e receberá um token após autenticação bem-sucedida, implementando o fluxo padrão de Authorization Code do OAuth 2.0/OpenID Connect.

## Arquitetura

```
┌────────────────┐        ┌─────────────────────┐        ┌─────────────┐
│   CSC      │───1───►│  KEYCLOAK UI        │        │  KEYCLOAK   │
│   Frontend     │◄──4────│  (Login/Register)   │◄──────►│   SERVER    │
└────────────────┘        └─────────────────────┘        └─────────────┘
        │                                                      ▲
        │                                                      │
        └──────────────────────5─────────────────────────────►│
                        (Validação/uso de token)               │
```

## Componentes

1. **CSC Frontend**: Aplicação principal que consumirá os serviços de autenticação
2. **Keycloak UI**: Interface de login/registro fornecida pelo Keycloak
3. **Keycloak Server**: Servidor de identidade que gerencia autenticação e autorização

## Fluxo de Autenticação (Authorization Code Flow)

1. Usuário não autenticado acessa o CSC
2. CSC redireciona para o endpoint de autorização do Keycloak
3. Keycloak apresenta sua própria página de login/registro
4. Usuário se autentica diretamente no Keycloak
5. Keycloak redireciona de volta para o CSC com um código de autorização
6. CSC troca o código por tokens de acesso/refresh
7. CSC armazena os tokens e autentica o usuário
8. Usuário é redirecionado para a página inicial com acesso autenticado

## Implementação Técnica

### 1. Configuração do Keycloak

```javascript
// Configuração do cliente Keycloak no CSC (services/auth/keycloak-config.ts)
export const keycloakConfig = {
  realm: 'CSC',
  clientId: 'CSC-frontend',
  url: 'https://[keycloak-server-url]/auth',
  redirectUri: 'https://CSC.example.com/api/auth/callback',
}
```

### 2. Inicialização do Redirecionamento para Login

```typescript
// Função para redirecionar ao login do Keycloak (lib/auth.ts)
export function redirectToKeycloakLogin() {
  const keycloakAuthUrl = `${keycloakConfig.url}/realms/${keycloakConfig.realm}/protocol/openid-connect/auth`
  
  // Gerar estado para prevenir CSRF
  const state = generateRandomString(32)
  const nonce = generateRandomString(32)
  
  // Armazenar estado temporariamente para validação no retorno
  sessionStorage.setItem('keycloak_state', state)
  sessionStorage.setItem('keycloak_nonce', nonce)
  
  // Construir URL de redirecionamento
  const params = new URLSearchParams({
    client_id: keycloakConfig.clientId,
    redirect_uri: keycloakConfig.redirectUri,
    state: state,
    response_type: 'code',
    scope: 'openid profile email',
    nonce: nonce,
  })
  
  // Redirecionar para Keycloak
  window.location.href = `${keycloakAuthUrl}?${params.toString()}`
}
```

### 3. Página de Login Adaptada (redirecionamento para Keycloak)

```typescript
// Página de login adaptada (app/[locale]/login/page.tsx)
"use client"

import { useEffect } from 'react'
import { useRouter, useSearchParams } from 'next/navigation'
import { redirectToKeycloakLogin } from '@/lib/auth'
import { Loader2 } from 'lucide-react'
import { useTranslations } from '@/hooks/use-translations'

export default function LoginPage() {
  const router = useRouter()
  const searchParams = useSearchParams()
  const { t } = useTranslations()
  
  // Verificar se usuário já está autenticado
  useEffect(() => {
    const checkAuth = async () => {
      try {
        const response = await fetch('/api/auth/me')
        
        if (response.ok) {
          // Usuário já autenticado, redirecionar para dashboard
          const redirectTo = searchParams.get('redirect') || '/dashboard'
          router.push(redirectTo)
        } else {
          // Usuário não autenticado, redirecionar para Keycloak
          redirectToKeycloakLogin()
        }
      } catch (error) {
        console.error('Erro ao verificar autenticação:', error)
        // Em caso de erro, tentar redirecionar para Keycloak
        redirectToKeycloakLogin()
      }
    }
    
    checkAuth()
  }, [router, searchParams])
  
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center">
        <Loader2 className="h-8 w-8 animate-spin mx-auto text-blue-600" />
        <p className="mt-4 text-lg text-slate-600">{t.common.redirectingToLogin}</p>
      </div>
    </div>
  )
}
```

### 4. Implementação do Callback para Processar a Autenticação

```typescript
// API Route para callback do Keycloak (app/api/auth/callback/route.ts)
import { NextRequest, NextResponse } from 'next/server'
import { cookies } from 'next/headers'
import { keycloakConfig } from '@/services/auth/keycloak-config'

export async function GET(request: NextRequest) {
  // Obter o código e estado da query string
  const searchParams = request.nextUrl.searchParams
  const code = searchParams.get('code')
  const state = searchParams.get('state')
  
  // Obter o estado armazenado na sessão (cookie)
  const storedState = request.cookies.get('keycloak_state')?.value
  
  // Verificar se o estado corresponde (prevenção CSRF)
  if (!code || !state || state !== storedState) {
    return NextResponse.redirect(new URL('/login?error=invalid_request', request.url))
  }
  
  try {
    // Trocar o código por tokens
    const tokenEndpoint = `${keycloakConfig.url}/realms/${keycloakConfig.realm}/protocol/openid-connect/token`
    
    const formData = new URLSearchParams()
    formData.append('client_id', keycloakConfig.clientId)
    formData.append('grant_type', 'authorization_code')
    formData.append('code', code)
    formData.append('redirect_uri', keycloakConfig.redirectUri)
    
    // Se o cliente tiver um client_secret
    if (process.env.KEYCLOAK_CLIENT_SECRET) {
      formData.append('client_secret', process.env.KEYCLOAK_CLIENT_SECRET)
    }
    
    const response = await fetch(tokenEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: formData,
    })
    
    if (!response.ok) {
      console.error('Erro ao obter tokens:', await response.text())
      return NextResponse.redirect(new URL('/login?error=token_exchange_error', request.url))
    }
    
    const tokens = await response.json()
    
    // Armazenar tokens em cookies seguros
    const cookieOptions = {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax' as const, // Necessário para o fluxo de redirecionamento
      maxAge: 60 * 60 * 24, // 1 dia para access token
      path: '/',
    }
    
    const refreshCookieOptions = {
      ...cookieOptions,
      maxAge: 60 * 60 * 24 * 30, // 30 dias para refresh token
    }
    
    cookies().set('access_token', tokens.access_token, cookieOptions)
    cookies().set('refresh_token', tokens.refresh_token, refreshCookieOptions)
    cookies().set('id_token', tokens.id_token, cookieOptions)
    
    // Limpar o cookie de estado
    cookies().delete('keycloak_state')
    
    // Redirecionar para a página de destino ou dashboard
    const redirectTo = request.cookies.get('auth_redirect')?.value || '/dashboard'
    cookies().delete('auth_redirect')
    
    return NextResponse.redirect(new URL(redirectTo, request.url))
  } catch (error) {
    console.error('Erro no processo de callback:', error)
    return NextResponse.redirect(new URL('/login?error=server_error', request.url))
  }
}
```

### 5. Middleware para Proteção de Rotas

```typescript
// Middleware para proteção de rotas (middleware.ts)
import { NextRequest, NextResponse } from 'next/server'
import { locales, defaultLocale } from '@/i18n/config'
import { verifyJWT } from '@/lib/jwt'

export function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname
  
  // Verificar se a requisição é para recursos estáticos ou APIs públicas
  if (pathname.includes('/_next') || 
      pathname.includes('/api/auth/callback') || 
      pathname.includes('/api/public')) {
    return NextResponse.next()
  }
  
  // Verificar se o caminho já tem um locale
  const pathnameHasLocale = locales.some(
    (locale) => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  )
  
  // Tratar localização
  const locale = pathnameHasLocale ? pathname.split('/')[1] : defaultLocale
  const pathWithoutLocale = pathnameHasLocale ? pathname.slice(locale.length + 1) : pathname
  
  // Verificar se é uma rota protegida
  const isProtectedRoute = pathWithoutLocale.startsWith('/dashboard') ||
                           pathWithoutLocale.startsWith('/configuracoes') ||
                           pathWithoutLocale.startsWith('/usinas') ||
                           // adicionar outras rotas protegidas
                           
  if (isProtectedRoute) {
    // Verificar token de autenticação
    const token = request.cookies.get('access_token')?.value
    
    if (!token) {
      // Armazenar URL de destino para redirecionamento após login
      const redirectUrl = new URL(`/${locale}/login`, request.url)
      const response = NextResponse.redirect(redirectUrl)
      
      // Armazenar caminho original para redirecionamento após login bem-sucedido
      response.cookies.set('auth_redirect', pathname, {
        path: '/',
        httpOnly: true,
        maxAge: 60 * 10, // 10 minutos
      })
      
      return response
    }
    
    try {
      // Verificar validade do token
      const payload = verifyJWT(token)
      
      // Verificar se o token expirou
      if (payload.exp && Date.now() >= payload.exp * 1000) {
        // Token expirado - implementar refresh token
        // Neste caso, redirecionamos para uma rota que tentará atualizar o token
        const refreshUrl = new URL('/api/auth/refresh', request.url)
        refreshUrl.searchParams.set('redirect', pathname)
        return NextResponse.redirect(refreshUrl)
      }
      
      // Token válido, prosseguir
      return NextResponse.next()
    } catch (error) {
      // Token inválido, redirecionar para login
      const redirectUrl = new URL(`/${locale}/login`, request.url)
      const response = NextResponse.redirect(redirectUrl)
      
      // Armazenar caminho original para redirecionamento após login
      response.cookies.set('auth_redirect', pathname, {
        path: '/',
        httpOnly: true,
        maxAge: 60 * 10, // 10 minutos
      })
      
      return response
    }
  }
  
  // Rotas públicas ou outros casos
  return NextResponse.next()
}
```

### 6. Implementação do Logout

```typescript
// API Route para logout (app/api/auth/logout/route.ts)
import { NextRequest, NextResponse } from 'next/server'
import { cookies } from 'next/headers'
import { keycloakConfig } from '@/services/auth/keycloak-config'

export async function GET(request: NextRequest) {
  // Obter o ID token (necessário para o endpoint de logout)
  const idToken = cookies().get('id_token')?.value
  
  // Limpar cookies
  cookies().delete('access_token')
  cookies().delete('refresh_token')
  cookies().delete('id_token')
  
  // Construir URL de logout do Keycloak
  const logoutUrl = new URL(
    `${keycloakConfig.url}/realms/${keycloakConfig.realm}/protocol/openid-connect/logout`
  )
  
  // Adicionar parâmetros
  const redirectUri = new URL('/', request.url).toString()
  logoutUrl.searchParams.set('post_logout_redirect_uri', redirectUri)
  
  // Adicionar ID token se disponível
  if (idToken) {
    logoutUrl.searchParams.set('id_token_hint', idToken)
  }
  
  // Redirecionar para o endpoint de logout do Keycloak
  return NextResponse.redirect(logoutUrl)
}
```

### 7. Implementação do Refresh Token

```typescript
// API Route para refresh token (app/api/auth/refresh/route.ts)
import { NextRequest, NextResponse } from 'next/server'
import { cookies } from 'next/headers'
import { keycloakConfig } from '@/services/auth/keycloak-config'

export async function GET(request: NextRequest) {
  // Obter o redirect path e o refresh token
  const redirectPath = request.nextUrl.searchParams.get('redirect') || '/dashboard'
  const refreshToken = cookies().get('refresh_token')?.value
  
  if (!refreshToken) {
    // Sem refresh token, redirecionar para login
    return NextResponse.redirect(new URL('/login', request.url))
  }
  
  try {
    // Chamar endpoint de token do Keycloak para refresh
    const tokenEndpoint = `${keycloakConfig.url}/realms/${keycloakConfig.realm}/protocol/openid-connect/token`
    
    const formData = new URLSearchParams()
    formData.append('client_id', keycloakConfig.clientId)
    formData.append('grant_type', 'refresh_token')
    formData.append('refresh_token', refreshToken)
    
    // Se o cliente tiver um client_secret
    if (process.env.KEYCLOAK_CLIENT_SECRET) {
      formData.append('client_secret', process.env.KEYCLOAK_CLIENT_SECRET)
    }
    
    const response = await fetch(tokenEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: formData,
    })
    
    if (!response.ok) {
      // Falha no refresh, limpar cookies e redirecionar para login
      cookies().delete('access_token')
      cookies().delete('refresh_token')
      cookies().delete('id_token')
      
      return NextResponse.redirect(new URL('/login', request.url))
    }
    
    // Obter novos tokens
    const tokens = await response.json()
    
    // Armazenar novos tokens
    const cookieOptions = {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'lax' as const,
      maxAge: 60 * 60 * 24, // 1 dia para access token
      path: '/',
    }
    
    const refreshCookieOptions = {
      ...cookieOptions,
      maxAge: 60 * 60 * 24 * 30, // 30 dias para refresh token
    }
    
    cookies().set('access_token', tokens.access_token, cookieOptions)
    cookies().set('refresh_token', tokens.refresh_token, refreshCookieOptions)
    
    if (tokens.id_token) {
      cookies().set('id_token', tokens.id_token, cookieOptions)
    }
    
    // Redirecionar para a página original
    return NextResponse.redirect(new URL(redirectPath, request.url))
  } catch (error) {
    console.error('Erro ao atualizar token:', error)
    
    // Em caso de erro, limpar cookies e redirecionar para login
    cookies().delete('access_token')
    cookies().delete('refresh_token')
    cookies().delete('id_token')
    
    return NextResponse.redirect(new URL('/login', request.url))
  }
}
```

### 8. Personalização da Tela de Login do Keycloak (Opcional)

Para manter a identidade visual do CSC, é possível personalizar a tela de login do Keycloak:

1. **Criar um tema personalizado no Keycloak**:
   - Crie um diretório `CSC` dentro de `themes` no servidor Keycloak
   - Personalize os templates e CSS com a identidade visual do CSC
   - Configure o realm para usar o tema personalizado

2. **Exemplo de estrutura para um tema**:
   ```
   themes/
   └── CSC/
       ├── login/
       │   ├── resources/
       │   │   ├── css/
       │   │   │   └── styles.css
       │   │   └── img/
       │   │       └── logo.png
       │   ├── template.ftl
       │   └── login.ftl
       └── account/
           └── ...
   ```

## Vantagens

1. **Segurança Robusta**: Implementação completa do fluxo OAuth 2.0/OpenID Connect
2. **Facilidade de Implementação**: Menos código para implementar e manter
3. **Funcionalidades Avançadas Prontas**: Suporte imediato a recursos como MFA, recuperação de senha, autenticação social, etc.
4. **Manutenção Reduzida**: Atualizações e melhorias de segurança são gerenciadas pelo Keycloak
5. **Conformidade com Padrões**: Implementação que segue protocolos padrão da indústria

## Desvantagens

1. **Experiência de Usuário Fragmentada**: Usuário é redirecionado para outra interface durante o login
2. **Menor Controle sobre UI**: Personalização limitada da experiência de login
3. **Complexidade de Personalização**: Customizar a interface do Keycloak requer criação de temas
4. **Dependência Externa**: Necessita que o servidor Keycloak esteja sempre disponível

## Considerações de Segurança

1. **Validação de Estado**: Implementar e verificar parâmetros state e nonce
2. **Manipulação Segura de Tokens**: Armazenar tokens em cookies HttpOnly e com flags de segurança
3. **PKCE (Proof Key for Code Exchange)**: Implementar PKCE para aplicações públicas
4. **Verificação de Assinatura**: Validar tokens JWT recebidos
5. **TLS/SSL**: Garantir que todas as comunicações sejam criptografadas

## Próximos Passos para Implementação

1. Configurar um realm no Keycloak para o CSC
2. Criar um client no Keycloak com configuração para Authorization Code Flow
3. Configurar as URLs de redirecionamento permitidas
4. Desenvolver a integração com o fluxo de autenticação do Keycloak
5. Implementar a lógica de proteção de rotas no middleware
6. Desenvolver mecanismos de refresh de token
7. Configurar logout
8. Opcionalmente, criar um tema personalizado no Keycloak
9. Testar todos os fluxos de autenticação
10. Implementar migração de usuários para o Keycloak 