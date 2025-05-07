# Fundamentos de Desenvolvimento

## Princípios Essenciais

### Simplicidade
- Sempre preferir soluções simples e diretas
- Evitar otimizações prematuras
- Implementar apenas o necessário para resolver o problema atual
- Refatorar gradualmente em vez de reescrever completamente

### Modularidade
- Dividir arquivos longos (>300 linhas) em arquivos menores e coesos
- Extrair funções longas em funções menores com propósito único
- Organizar código relacionado em módulos com responsabilidades claras
- Projetar interfaces que escondam detalhes de implementação

### Reutilização
- Evitar duplicação de código (DRY - Don't Repeat Yourself)
- Verificar se existe código semelhante antes de implementar nova funcionalidade
- Criar abstrações compartilhadas para padrões recorrentes
- Considerar a composição de funções/componentes para maximizar reuso

### Manutenibilidade
- Escrever código autoexplicativo com nomes descritivos
- Comentar apenas o "porquê", não o "como" (o código deve ser claro por si só)
- Manter uma arquitetura consistente em todo o projeto
- Considerar como o código será evoluído e mantido no futuro

## Metodologias de Trabalho

### Modo Planejador
Ao entrar neste modo, seguir esta sequência:
1. Analisar profundamente o código existente relacionado à tarefa
2. Mapear o escopo completo das alterações necessárias
3. Formular 4-6 perguntas esclarecedoras baseadas na análise
4. Após respostas, elaborar um plano de ação abrangente
5. Solicitar aprovação antes de implementar
6. Implementar o plano em fases, atualizando o progresso a cada etapa
7. Ao finalizar cada fase, indicar o que foi concluído e os próximos passos

### Modo Depurador
Ao entrar neste modo, seguir rigorosamente esta sequência:
1. Identificar 5-7 possíveis causas do problema
2. Reduzir para 1-2 causas mais prováveis
3. Adicionar logs estratégicos para validar suposições
4. Utilizar ferramentas para capturar logs (console, rede, servidor)
5. Analisar profundamente os dados coletados
6. Sugerir logs adicionais se necessário
7. Implementar correção após identificação da causa raiz
8. Solicitar aprovação para remover logs temporários

## Boas Práticas de Código

### Estruturação
- Manter arquivos com menos de 300 linhas de código
- Evitar funções com mais de 30-40 linhas
- Organizar código por funcionalidade, não por tipo de arquivo
- Seguir padrões de projeto apropriados para o problema

### Gestão de Dependências
- Manter dependências atualizadas
- Minimizar o número de dependências externas
- Documentar o propósito de cada dependência
- Considerar o custo de manutenção ao adicionar novas dependências

### Ambientes
- Escrever código que funcione consistentemente em todos os ambientes (dev, test, prod)
- Configurar variáveis de ambiente apropriadamente para cada contexto
- Nunca inserir dados simulados em ambientes de produção
- Tratar configurações sensíveis com segurança

### Controle de Mudanças
- Ser conservador ao introduzir novos padrões ou tecnologias
- Remover código antigo ao implementar novas abordagens
- Documentar alterações significativas de arquitetura
- Evitar scripts de uso único em arquivos permanentes

## Reflexão e Melhoria Contínua

### Análise Pós-Implementação
- Após cada mudança significativa, refletir sobre:
  - Impacto na escalabilidade do sistema
  - Manutenibilidade a longo prazo
  - Potenciais pontos de falha introduzidos
  - Oportunidades de otimização futuras

### Documentação
- Manter documentação atualizada com as mudanças de código
- Documentar decisões de arquitetura e suas justificativas
- Criar exemplos de uso para APIs e componentes complexos
- Utilizar diagramas quando apropriado para explicar fluxos complexos

### Revisão de Código
- Buscar feedback sobre implementações alternativas
- Verificar se o código segue os padrões do projeto
- Considerar a legibilidade para outros desenvolvedores
- Avaliar o equilíbrio entre elegância e praticidade

## Regras Gerais

- Sempre responder em português brasileiro
- Verificar outras áreas do código para evitar duplicação
- Ser cauteloso ao fazer apenas as mudanças necessárias
- Esgotar opções existentes antes de introduzir novas tecnologias
- Manter o código bem estruturado e organizado
- Evitar arquivos com mais de 300 linhas de código
- Nunca alterar configurações sensíveis (.env) sem confirmação prévia 