# Padrões de Versionamento do CSC

## Introdução

Este documento define os padrões de versionamento adotados no projeto CSC, baseados no [SemVer (Semantic Versioning)](https://semver.org/) e adaptados às necessidades específicas do nosso projeto.

O versionamento semântico nos permite comunicar claramente as mudanças introduzidas em cada versão, facilitando o gerenciamento de dependências e a manutenção do software.

## Formato da Versão

O versionamento do CSC segue o padrão **N1.N2.N3** (Major.Minor.Patch), onde:

- **N1 (Major)**: Incrementado quando há novas telas ou mudanças que alteram significativamente a experiência do usuário
- **N2 (Minor)**: Incrementado quando há novos componentes ou funcionalidades adicionadas de maneira compatível
- **N3 (Patch)**: Incrementado quando há correções de bugs sem alteração de funcionalidades

Esta adaptação do SemVer está alinhada com a natureza da nossa aplicação, onde as interfaces de usuário e componentes de UI são elementos centrais do desenvolvimento.

### Alinhamento com o SemVer

Nosso padrão se alinha com o SemVer da seguinte forma:

1. **Versão Major (N1)** - Incrementada quando:
   - Novas telas são adicionadas ao sistema
   - Redesenho completo de telas existentes
   - Alterações incompatíveis na API ou no modelo de dados
   - Mudanças que requerem migração de dados

2. **Versão Minor (N2)** - Incrementada quando:
   - Novos componentes são adicionados
   - Novas funcionalidades são adicionadas de forma compatível com versões anteriores
   - Melhorias significativas em componentes existentes
   - Depreciação de funcionalidades (que serão removidas em versões Major futuras)

3. **Versão Patch (N3)** - Incrementada quando:
   - Correções de bugs
   - Ajustes de performance
   - Pequenas melhorias que não alteram a API ou funcionalidades
   - Correções de texto ou estilo

## Exemplos Práticos

- **1.0.0**: Versão inicial estável do CSC
- **1.1.0**: Adição de novos componentes de filtragem na dashboard
- **1.1.1**: Correção de um bug no componente de gráfico
- **2.0.0**: Adição de novas telas de analytics e redesign da navegação

## Versionamento de Pré-lançamento

Para versões em desenvolvimento que ainda não estão prontas para produção, utilizamos o formato:

```
N1.N2.N3-[identificador].[número]
```

Exemplos:
- **1.2.0-alpha.1**: Primeira versão alpha da 1.2.0
- **1.2.0-beta.3**: Terceira versão beta da 1.2.0
- **1.2.0-rc.1**: Primeira versão candidata a lançamento da 1.2.0

## Changelog

Todas as mudanças entre versões são documentadas no diretório `/docs/changelog/`, com arquivos nomeados de acordo com a versão (e.g., `2.0.1.md`).

### Formato do Arquivo de Changelog

Cada arquivo de changelog segue a estrutura:

```markdown
# Changelog vX.Y.Z

Data: DD/MM/AAAA  
Tipo: [Tipo de lançamento]

## Resumo das Alterações

Breve descrição das principais alterações desta versão.

## Correções / Adições / Alterações

### [Categoria]

#### [Componente/Funcionalidade]
- **[TIPO]** Descrição da alteração
```

### Tipos de Alteração

Utilizamos os seguintes tipos para classificar as alterações:

- **[ADIÇÃO]**: Nova funcionalidade
- **[ALTERAÇÃO]**: Mudança em funcionalidade existente
- **[CORREÇÃO]**: Correção de bug
- **[REMOÇÃO]**: Remoção de funcionalidade
- **[SEGURANÇA]**: Correção de vulnerabilidade
- **[PERFORMANCE]**: Melhoria de performance
- **[DOCUMENTAÇÃO]**: Alterações na documentação
- **[REDESIGN]**: Alteração visual/UX
- **[REFATORAÇÃO]**: Refatoração de código sem alteração de comportamento
- **[TESTE]**: Adição ou correção de testes
- **[DEPENDÊNCIA]**: Atualização ou adição de dependências
- **[MELHORIA]**: Aprimoramento geral que não se encaixa nas categorias acima

## Quando Incrementar Cada Versão

### Incremento de Versão Major (N1)

- Adição de novas telas principais (Dashboard, Analytics, Configurações, etc.)
- Alterações na estrutura de navegação principal
- Mudanças significativas na experiência do usuário
- Alterações que quebram compatibilidade com versões anteriores
- Remoção de funcionalidades previamente depreciadas

### Incremento de Versão Minor (N2)

- Implementação de novos componentes UI (Filtros, Gráficos, Tabelas, etc.)
- Novas funcionalidades dentro de telas existentes
- Melhorias significativas em componentes existentes
- Adição de novos endpoints na API
- Depreciação de funcionalidades (que serão removidas em versão major futura)

### Incremento de Versão Patch (N3)

- Correções de bugs em componentes
- Ajustes de estilo ou layout
- Melhorias de performance sem alteração na API
- Correções de texto ou tradução
- Atualizações de dependências sem impacto funcional
- Refatorações internas que não afetam a interface do usuário

## Regras Adicionais

1. Quando uma versão Minor é incrementada, a versão Patch deve ser resetada para 0 (exemplo: 1.1.3 → 1.2.0)
2. Quando uma versão Major é incrementada, as versões Minor e Patch devem ser resetadas para 0 (exemplo: 1.2.3 → 2.0.0)
3. Versões alfa, beta ou release candidates devem ser claramente marcadas como tal
4. Não devem ser feitas alterações retroativas em versões já lançadas
5. A mesma versão do arquivo .md criado no changelog atualize o valor do campo version no package.json
6. A próxima versão do próximo arquivo de changelog deve seguir uma sequência, sempre observando a última numeração do arquivo .md no diretorio changelog
7. Toda vez que um arquivo de changelog for criado, insira no campo releaseDate do arquivo package.json uma data no formato yyyy-mm-dd hh:mm:ss e que essa data e hora seja a atual

## Propósito

Este sistema de versionamento tem como objetivos:

1. Facilitar o rastreamento de mudanças entre versões
2. Documentar claramente correções de bugs e novos recursos
3. Auxiliar na comunicação com usuários e stakeholders
4. Servir como referência histórica para a equipe de desenvolvimento
5. Facilitar a manutenção e evolução do software

## Referências

- [SemVer - Semantic Versioning 2.0.0](https://semver.org/)
- [Documentação de Changelog do CSC](/docs/changelog/README.md) 