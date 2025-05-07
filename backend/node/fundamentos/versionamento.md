# Padrões de Versionamento do CONFIG-SERVICE

## Introdução

Este documento define os padrões de versionamento adotados no projeto CONFIG-SERVICE, baseados no [SemVer (Semantic Versioning)](https://semver.org/) e adaptados às necessidades específicas do nosso serviço backend.

O versionamento semântico nos permite comunicar claramente as mudanças introduzidas em cada versão, facilitando o gerenciamento de dependências e a manutenção do software.

## Formato da Versão

O versionamento do CONFIG-SERVICE segue o padrão **N1.N2.N3** (Major.Minor.Patch), onde:

- **N1 (Major)**: Incrementado quando há mudanças incompatíveis na API ou alterações estruturais significativas
- **N2 (Minor)**: Incrementado quando há novas funcionalidades adicionadas de maneira compatível
- **N3 (Patch)**: Incrementado quando há correções de bugs sem alteração de funcionalidades

Esta adaptação do SemVer está alinhada com a natureza do nosso serviço backend, onde a estabilidade da API e o gerenciamento de dados são elementos centrais.

### Alinhamento com o SemVer

Nosso padrão se alinha com o SemVer da seguinte forma:

1. **Versão Major (N1)** - Incrementada quando:
   - Breaking changes na API (alterações em contratos existentes)
   - Mudanças significativas na arquitetura do sistema
   - Alterações incompatíveis no modelo de dados
   - Mudanças que requerem migração de dados
   - Atualizações major do framework NestJS

2. **Versão Minor (N2)** - Incrementada quando:
   - Novos endpoints são adicionados
   - Novas entidades ou domínios são criados
   - Novos recursos são adicionados de forma compatível
   - Melhorias significativas em módulos existentes
   - Depreciação de endpoints (que serão removidos em versões Major futuras)

3. **Versão Patch (N3)** - Incrementada quando:
   - Correções de bugs
   - Ajustes de performance
   - Pequenas melhorias que não alteram a API
   - Atualizações de dependências sem breaking changes

## Exemplos Práticos

- **1.0.0**: Versão inicial estável do CONFIG-SERVICE
- **1.1.0**: Adição de novos endpoints de atributos
- **1.1.1**: Correção de bug na validação de dados
- **2.0.0**: Refatoração completa da camada de autenticação

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

#### [Módulo/Funcionalidade]
- **[TIPO]** Descrição da alteração
```

### Tipos de Alteração

- **[ADIÇÃO]**: Novo endpoint, módulo ou funcionalidade
- **[ALTERAÇÃO]**: Mudança em endpoint ou funcionalidade existente
- **[CORREÇÃO]**: Correção de bug
- **[REMOÇÃO]**: Remoção de endpoint ou funcionalidade
- **[SEGURANÇA]**: Correção de vulnerabilidade
- **[PERFORMANCE]**: Melhoria de performance
- **[DOCUMENTAÇÃO]**: Alterações na documentação
- **[ARQUITETURA]**: Alteração estrutural do sistema
- **[REFATORAÇÃO]**: Refatoração de código sem alteração de comportamento
- **[TESTE]**: Adição ou correção de testes
- **[DEPENDÊNCIA]**: Atualização ou adição de dependências
- **[MELHORIA]**: Aprimoramento geral que não se encaixa nas categorias acima

## Quando Incrementar Cada Versão

### Incremento de Versão Major (N1)

- Breaking changes em endpoints existentes
- Alterações significativas no modelo de dados
- Mudanças na arquitetura do sistema
- Alterações que quebram compatibilidade com versões anteriores
- Remoção de endpoints previamente depreciados
- Atualizações major do NestJS ou dependências críticas

### Incremento de Versão Minor (N2)

- Novos endpoints
- Novos módulos ou domínios
- Novas funcionalidades compatíveis com versões anteriores
- Melhorias significativas em módulos existentes
- Adição de novas entidades
- Depreciação de endpoints (que serão removidos em versão major futura)
- Novas integrações com serviços externos

### Incremento de Versão Patch (N3)

- Correções de bugs em endpoints
- Ajustes de validação de dados
- Melhorias de performance sem alteração na API
- Correções em regras de negócio
- Atualizações de dependências sem impacto funcional
- Refatorações internas que não afetam a API
- Melhorias na documentação

## Regras Adicionais

1. Quando uma versão Minor é incrementada, a versão Patch deve ser resetada para 0 (exemplo: 1.1.3 → 1.2.0)
2. Quando uma versão Major é incrementada, as versões Minor e Patch devem ser resetadas para 0 (exemplo: 1.2.3 → 2.0.0)
3. Versões alfa, beta ou release candidates devem ser claramente marcadas como tal
4. Não devem ser feitas alterações retroativas em versões já lançadas
5. A mesma versão do arquivo .md criado no changelog atualize o valor do campo version no package.json
6. A próxima versão do próximo arquivo de changelog deve seguir uma sequência, sempre observando a última numeração do arquivo .md no diretorio changelog

## Propósito

Este sistema de versionamento tem como objetivos:

1. Garantir estabilidade e previsibilidade nas APIs
2. Facilitar o gerenciamento de dependências
3. Documentar claramente mudanças na API e correções
4. Auxiliar na comunicação com consumidores da API
5. Servir como referência histórica para a equipe de desenvolvimento
6. Facilitar a manutenção e evolução do serviço

## Referências

- [SemVer - Semantic Versioning 2.0.0](https://semver.org/)
- [Documentação de Changelog do CONFIG-SERVICE](/docs/changelog/README.md)
- [NestJS Release Conventions](https://docs.nestjs.com/)