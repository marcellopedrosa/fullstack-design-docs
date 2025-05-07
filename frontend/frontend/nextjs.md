# Melhores Práticas para Next.js

## Gerenciamento de Dependências

1. **Nunca instale componentes via npm com versão @latest**
   - Sempre especifique a versão exata dos pacotes
   - Isso garante consistência entre ambientes e evita quebras inesperadas
   - Exemplo: `npm install react-day-picker@8.9.0` (em vez de `npm install react-day-picker@latest`)

2. **Nunca utilize a flag --legacy-peer-deps**
   - Sempre compatibilize as dependências através do downgrade ou upgrade das versões dos componentes
   - Usar --legacy-peer-deps mascara problemas de compatibilidade que podem causar bugs difíceis de rastrear
   - Resolver conflitos de dependências adequadamente é essencial para a estabilidade do projeto

3. **Mantenha um registro de dependências**
   - Documente por que cada dependência importante foi escolhida
   - Registre quaisquer peculiaridades ou limitações conhecidas

4. **Avalie novas dependências cuidadosamente**
   - Verifique tamanho, manutenção ativa, compatibilidade e segurança
   - Prefira bibliotecas com menos dependências aninhadas
   - Verifique compatibilidade com a versão do React e do Next.js utilizada

## Estrutura do Projeto

1. **Organização de diretórios**
   - `/app`: Para rotas e componentes específicos de página (App Router)
   - `/components`: Componentes reutilizáveis
   - `/lib`: Utilitários e funções auxiliares
   - `/public`: Arquivos estáticos
   - `/styles`: Estilos globais e temas

2. **Padrão de Componentes**
   - Adote uma abordagem consistente (Atomic Design, por exemplo)
   - Mantenha componentes pequenos e com responsabilidade única
   - Implemente componentes puros quando possível

## Performance

1. **Otimização de imagens**
   - Use o componente Next/Image para otimização automática
   - Especifique dimensões para evitar layout shifts

2. **Code Splitting**
   - Use dynamic imports para carregamento sob demanda
   - Implemente lazy loading para componentes pesados

3. **Estratégias de Rendering**
   - Use Server Components para conteúdo estático ou que não precisa de interatividade
   - Use Client Components apenas quando necessário interatividade no lado do cliente
   - Implemente streaming quando adequado para melhorar TTFB

## SEO e Acessibilidade

1. **Metadados**
   - Implemente metadados adequados para cada página
   - Use os recursos de metadados do Next.js para otimizar SEO

2. **Acessibilidade**
   - Siga as diretrizes WCAG
   - Teste com leitores de tela e ferramentas de acessibilidade

## Estado da Aplicação

1. **Gerenciamento de Estado**
   - Use React Context, Zustand ou outras soluções leves quando possível
   - Mantenha o estado o mais próximo possível de onde é usado

2. **Fetch de Dados**
   - Utilize React Server Components para busca de dados no servidor
   - Implemente caching estratégico

## Internacionalização

1. **Suporte a múltiplos idiomas**
   - Utilize o sistema de i18n do Next.js ou bibliotecas como next-intl
   - Evite strings hard-coded no código

## Testes

1. **Implementação de testes**
   - Escreva testes unitários para componentes e funções
   - Implemente testes de integração para fluxos críticos
   - Use ferramentas como Jest, React Testing Library e Cypress

## Deployment e CI/CD

1. **Automação**
   - Configure pipelines de CI/CD para testes e deployment automáticos
   - Implemente verificações de qualidade de código

2. **Monitoramento**
   - Configure ferramentas para monitorar performance e erros em produção

## Segurança

1. **Validação de Entrada**
   - Sempre valide e sanitize dados de entrada do usuário
   - Implemente tratamento adequado de erros

2. **Prevenção contra ataques comuns**
   - Proteja contra XSS, CSRF e outras vulnerabilidades comuns
   - Mantenha dependências atualizadas para evitar vulnerabilidades conhecidas

## Versionamento

1. **Estratégia de branches**
   - Adote Git Flow ou outro modelo de branching adequado
   - Mantenha histórico de commits limpo e descritivo

2. **Documentação de alterações**
   - Mantenha um arquivo de CHANGELOG
   - Use mensagens de commit semânticas 