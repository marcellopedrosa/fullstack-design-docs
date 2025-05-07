# Diretrizes de Desenvolvimento

## Padrões de Código

### Nomenclatura
- Componentes: PascalCase (ex: `UserProfile.tsx`)
- Hooks: camelCase com prefixo "use" (ex: `useLocalStorage.ts`)
- Utilitários/funções: camelCase (ex: `formatCurrency.ts`)
- Tipos/interfaces: PascalCase com prefixo descritivo (ex: `UserType.ts`)
- Variáveis de ambiente: UPPER_SNAKE_CASE (ex: `API_BASE_URL`)

### Estrutura de Componentes
- Um componente por arquivo
- Exportar componentes como default quando apropriado
- Separar componentes maiores em sub-componentes quando ultrapassarem 200 linhas
- Manter arquivos relacionados próximos (componente, test, styles)

### TypeScript
- Tipar todos os parâmetros e retornos de funções
- Evitar `any` - usar `unknown` quando não souber o tipo
- Preferir interfaces para objetos públicos e types para tipos internos
- Usar enums para conjuntos de valores finitos

## Práticas Recomendadas

### Performance
- Minimizar a renderização desnecessária com memoização quando apropriado
- Otimizar o carregamento de imagens usando o componente Image do Next.js
- Implementar code-splitting para reduzir o bundle inicial
- Utilizar Server Components quando não for necessária interatividade

### Acessibilidade
- Seguir a especificação WCAG 2.1 nível AA
- Implementar navegação por teclado para todas as interações
- Garantir contraste adequado para texto
- Fornecer textos alternativos para imagens

### Responsividade
- Design mobile-first usando os breakpoints do Tailwind
- Testar em múltiplos dispositivos
- Usar unidades relativas (rem, %, vh/vw) em vez de valores absolutos

### Gerenciamento de Estado
- Preferir estado local com useState para estado de componente
- Considerar useReducer para lógica de estado complexa
- Utilizar Context API para estado global compartilhado entre componentes

## Fluxo de Trabalho

### Commits
- Mensagens claras e descritivas no formato imperativo
- Prefixos para categorizar: feat, fix, docs, style, refactor, test, chore
- Referenciar número de issue quando aplicável

### Pull Requests
- Descrição detalhada das mudanças
- Lista de verificação de requisitos implementados
- Solicitar revisão de pelo menos um membro da equipe

### Testes
- Escrever testes unitários para hooks e funções utilitárias
- Implementar testes de integração para fluxos críticos
- Manter cobertura de testes acima de 70%

## Segurança

- Validar todas as entradas do usuário, tanto no cliente quanto no servidor
- Implementar sanitização de dados para prevenir XSS
- Nunca expor informações sensíveis no código do cliente
- Utilizar HTTPS para todas as comunicações 