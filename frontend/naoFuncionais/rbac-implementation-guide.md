# Guia de Implementação de RBAC para Novas Telas

## Introdução

Este documento fornece diretrizes para a implementação do controle de acesso baseado em roles (RBAC) ao criar novas telas no CSC. O sistema utiliza uma abordagem em camadas para garantir a segurança, com verificações em três níveis diferentes.

## Estrutura do RBAC

### 1. Nível de Middleware (Global)

O primeiro nível de proteção é implementado no arquivo `middleware.ts`. Toda nova tela protegida deve ser registrada aqui.

```typescript
// Em middleware.ts
const protectedRoutes = {
  '/dashboard': ['web_admin', 'web_viewer'],
  '/sua-nova-tela': ['roles_necessarias'],  // Adicione sua rota aqui
}
```

#### Roles Disponíveis
- `web_admin`: Acesso administrativo completo
- `web_viewer`: Acesso somente leitura

#### Como Implementar

1. Identifique as roles necessárias para sua tela
2. Adicione a rota no objeto `protectedRoutes`
3. Use o formato: `'/caminho-da-rota': ['role1', 'role2']`

### 2. Nível de Layout (Protected Routes)

Telas dentro do diretório `app/[locale]/(protected)/` herdam automaticamente a proteção básica do layout protegido.

#### Como Implementar

1. Crie sua tela dentro do diretório protected:
```
app/[locale]/(protected)/sua-nova-tela/page.tsx
```

2. O componente será automaticamente envolvido com o HOC withRbac:
```typescript
export default withRbac(SuaNovaTelaPage, {
  requiredRoles: ["web_admin", "web_viewer"],
  redirectTo: "/login"
})
```

### 3. Nível de Componente (Granular)

Para controle mais fino dentro da tela, use o componente RoleGuard.

#### Como Implementar

```typescript
import { RoleGuard } from "@/components/ui/role-guard"

// Em sua tela ou componente:
<RoleGuard 
  requiredRoles="web_admin"
  fallback={<AcessoNegadoPersonalizado />}  // Opcional
>
  <ComponenteProtegido />
</RoleGuard>
```

## Fluxo de Implementação

1. **Definir Permissões**
   - Identifique quais roles podem acessar a nova tela
   - Defina permissões granulares para funcionalidades específicas

2. **Registrar no Middleware**
   - referenciar types/paths.ts para as rotas
   - referenciar types/roles.ts para as roles
   - as roles e os paths são centralizados para poderem ser a fonte centralizada
   ```typescript
   // Em middleware.ts
   const protectedRoutes = {
     // ...rotas existentes...
     '/sua-nova-tela': ['web_admin'], // Se apenas admins podem ver
     // ou
     '/sua-nova-tela': ['web_admin', 'web_viewer'], // Se viewers também podem ver
   }
   ```

3. **Criar a Tela**
   - Coloque no diretório protected
   - Implemente o controle de acesso granular onde necessário

4. **Testar Acessos**
   - Teste com usuário sem autenticação
   - Teste com usuário autenticado sem as roles necessárias
   - Teste com usuário com as roles corretas

## Boas Práticas

1. **Princípio do Privilégio Mínimo**
   - Conceda apenas as permissões estritamente necessárias
   - Use `web_viewer` como padrão para acesso somente leitura
   - Reserve `web_admin` para operações sensíveis

2. **Granularidade**
   - Use RoleGuard para controles específicos dentro da tela
   - Não confie apenas na proteção do middleware
   - Proteja também os endpoints da API relacionados

3. **Feedback ao Usuário**
   - Elementos inacessíveis devem ser ocultados ou desabilitados
   - Use o componente de acesso negado quando apropriado
   - Forneça mensagens claras sobre falta de permissão

4. **Segurança em Camadas**
   - Implemente proteção no middleware (rota)
   - Implemente proteção nos componentes (UI)
   - Implemente proteção nos endpoints da API

## Exemplo Completo

```typescript
// 1. Registro no middleware
// Em middleware.ts
const protectedRoutes = {
  '/relatorios': ['web_admin', 'web_viewer'],
}

// 2. Criação da página
// Roles.WEB_ADMIN em types/roles.ts
// Em app/[locale]/(protected)/relatorios/page.tsx
export default function RelatoriosPage() {
  return (
    <div>
      <h1>Relatórios</h1>
      
      {/* Visualização disponível para todos com acesso */}
      <TabelaRelatorios />
      
      {/* Ações administrativas apenas para admin */}
      <RoleGuard requiredRoles=[Roles.WEB_ADMIN]>
        <div>
          <BotaoGerarNovoRelatorio />
          <BotaoExcluirRelatorio />
        </div>
      </RoleGuard>
    </div>
  )
}
```

## Troubleshooting

### Problemas Comuns

1. **Usuário Redirecionado Inesperadamente**
   - Verifique se a rota está corretamente registrada no middleware
   - Confirme que o usuário possui as roles necessárias
   - Verifique se o token está válido

2. **Componente Visível Para Usuário Sem Permissão**
   - Verifique se está usando RoleGuard corretamente
   - Confirme que as roles estão corretamente especificadas
   - Verifique se o contexto RBAC está disponível

3. **Erro de Contexto RBAC**
   - Certifique-se que a página está dentro do diretório protected
   - Verifique se RbacProvider está presente na árvore de componentes

## Considerações de Segurança

1. **Nunca confie apenas no frontend**
   - Implemente verificações de permissão também no backend
   - Valide tokens e permissões em todas as chamadas de API
   - Mantenha logs de tentativas de acesso não autorizado

2. **Mantenha as roles atualizadas**
   - Revise periodicamente as permissões concedidas
   - Remova acessos desnecessários
   - Documente alterações nas permissões

3. **Teste de segurança**
   - Realize testes de penetração regularmente
   - Verifique tentativas de bypass de autenticação
   - Monitore padrões de acesso suspeitos