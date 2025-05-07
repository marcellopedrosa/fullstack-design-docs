# Arquitetura do Sistema CSC

## Visão Geral

O CSC é uma aplicação Next.js moderna que utiliza a arquitetura App Router para gerenciamento de rotas. A estrutura segue os princípios de modularidade, separação de responsabilidades e facilidade de manutenção.

## Princípios Arquiteturais

- **Componentização**: Toda a interface é construída a partir de componentes reutilizáveis
- **Roteamento Dinâmico**: Utilização do sistema de roteamento baseado em arquivos do Next.js 15+
- **Internacionalização**: Suporte nativo a múltiplos idiomas via middleware
- **Autenticação Centralizada**: Sistema de autenticação unificado para controle de acesso
- **Responsividade**: Design adaptativo para diferentes dispositivos

## Estrutura de Diretórios

```
/app                 # Contém páginas e layouts (App Router)
  /(protected)       # Rotas que requerem autenticação
  /[locale]          # Rotas específicas por idioma
  /api               # Endpoints de API
  
/components          # Componentes reutilizáveis
  /ui                # Componentes de UI básicos
  /layout            # Componentes de layout
  
/hooks               # Hooks personalizados React
/i18n                # Configuração e arquivos de internacionalização
/lib                 # Utilitários e funções auxiliares
/services            # Serviços para comunicação com APIs
/public              # Arquivos estáticos
/styles              # Estilos globais e utilitários
/types               # Definições de tipos TypeScript
```

## Padrões Arquiteturais

1. **Container/Presentation Pattern**: Separação entre componentes com lógica e componentes puramente de apresentação
2. **Composition Pattern**: Componentes pequenos e focados que são combinados para criar interfaces complexas
3. **Server Components / Client Components**: Utilização apropriada de componentes de servidor e cliente
4. **Route Handlers**: API endpoints implementados como Route Handlers no App Router

## Diretrizes de Implementação

- Componentes Server-side por padrão, Client-side apenas quando necessário
- Carregamento de dados preferencialmente no servidor
- Implementar suspense e carregamento progressivo
- Manter componentes pequenos e com responsabilidade única
- Reutilizar componentes existentes sempre que possível

## Tecnologias Principais

- **Next.js 15+**: Framework React com renderização híbrida
- **React 19**: Biblioteca para construção de interfaces
- **TypeScript**: Tipagem estática para melhor manutenção
- **Tailwind CSS**: Framework CSS utilitário
- **Radix UI**: Biblioteca de componentes acessíveis e sem estilo
- **Zod**: Validação de esquemas e tipos em runtime 