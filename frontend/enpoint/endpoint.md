# Diretrizes para Implementação de Endpoints

Este documento fornece diretrizes para a implementação de endpoints na API do CSC, garantindo consistência, segurança e manutenibilidade.

## Estrutura de URL

### Padrões de Nomenclatura
- Use substantivos no plural para recursos (ex: `/plants`, `/companies`)
- Evite verbos nas URLs, use métodos HTTP para indicar ações
- Use kebab-case para múltiplas palavras (ex: `/alerts-status`)
- Mantenha URLs em minúsculas

### Versionamento
- Inclua a versão na URL quando necessário (ex: `/api/v1/plants`)
- Mantenha compatibilidade com versões anteriores
- Documente alterações que quebram compatibilidade

### Hierarquia de Recursos
- Use URLs aninhadas para representar relacionamentos
- Limite o aninhamento a 2-3 níveis para manter clareza
- Exemplo: `/plants/{id}/alerts`

## Métodos HTTP

### GET
- Recuperação de recursos
- Nunca modifica dados
- Suporta cache
- Exemplo: `GET /plants/{id}`

### POST
- Criação de novos recursos
- Pode ter efeitos colaterais
- Não é idempotente
- Exemplo: `POST /plants`

### PUT
- Atualização completa de recursos
- Idempotente
- Requer todos os campos obrigatórios
- Exemplo: `PUT /plants/{id}`

### PATCH
- Atualização parcial de recursos
- Permite modificar campos específicos
- Exemplo: `PATCH /plants/{id}`

### DELETE
- Remoção de recursos
- Idempotente
- Exemplo: `DELETE /plants/{id}`

## Parâmetros e Consultas

### Parâmetros de URL
- Use para identificadores de recursos
- Siga o padrão {recurso}/{id}
- Exemplo: `/plants/123`

### Parâmetros de Consulta
- Use para filtragem, ordenação e paginação
- Mantenha consistência nos nomes dos parâmetros
- Exemplos:
  - Filtragem: `?status=ativo`
  - Ordenação: `?sort=nome&order=asc`
  - Paginação: `?page=1&limit=20`

## Respostas

### Códigos de Status HTTP
- 200: OK (GET, PUT, PATCH bem-sucedidos)
- 201: Created (POST bem-sucedido)
- 204: No Content (DELETE bem-sucedido)
- 400: Bad Request (erro de validação)
- 401: Unauthorized (autenticação necessária)
- 403: Forbidden (sem permissão)
- 404: Not Found (recurso não encontrado)
- 500: Internal Server Error (erro do servidor)

### Formato de Resposta
- Use JSON consistentemente
- Inclua mensagens de erro descritivas
- Mantenha estrutura uniforme de resposta

Exemplo de sucesso:
```json
{
  "data": {
    "id": "123",
    "nome": "Usina Solar SP",
    "status": "ativo"
  },
  "meta": {
    "timestamp": "2025-05-07T10:00:00Z"
  }
}
```

Exemplo de erro:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Campo nome é obrigatório",
    "details": [
      {
        "field": "nome",
        "message": "Este campo não pode estar vazio"
      }
    ]
  }
}
```

## Segurança

### Autenticação e Autorização
- Use tokens JWT para autenticação
- Verifique permissões em todas as rotas protegidas
- Implemente rate limiting
- Use HTTPS em todos os ambientes

### Validação
- Valide todas as entradas do usuário
- Use esquemas de validação (Zod/Joi)
- Sanitize dados antes de processá-los
- Implemente limites para tamanho de payload

### Proteção contra Ataques
- Implemente proteção CSRF
- Configure cabeçalhos de segurança
- Previna injeção de SQL
- Limite tamanho de requisições

## Cache

### Estratégias
- Use cache de resposta quando apropriado
- Configure cabeçalhos Cache-Control
- Implemente ETags para recursos estáticos
- Use cache distribuído para dados frequentemente acessados

### Invalidação
- Defina política clara de invalidação
- Atualize cache quando recursos são modificados
- Implemente purga seletiva de cache

## Documentação

### API Reference
- Documente todos os endpoints
- Inclua exemplos de requisição/resposta
- Descreva parâmetros e tipos de dados
- Mantenha documentação atualizada

### Changelog
- Registre todas as alterações
- Documente breaking changes
- Mantenha histórico de versões
- Inclua instruções de migração

## Performance

### Otimizações
- Implemente paginação para listas grandes
- Use compressão gzip/brotli
- Minimize payload de resposta
- Otimize consultas ao banco de dados

### Monitoramento
- Monitore tempo de resposta
- Registre erros e exceções
- Configure alertas para problemas
- Mantenha métricas de uso

## Testes

### Tipos de Teste
- Testes unitários para lógica de negócio
- Testes de integração para endpoints
- Testes de carga para performance
- Testes de segurança

### Cobertura
- Mantenha alta cobertura de testes
- Teste casos de erro
- Valide respostas de API
- Automatize testes na CI/CD

## Versionamento

### Estratégia
- Use versionamento semântico
- Mantenha compatibilidade retroativa
- Documente breaking changes
- Planeje descontinuação de APIs

### Evolução
- Prefira adições não destrutivas
- Deprecie recursos gradualmente
- Comunique mudanças com antecedência
- Forneça período de transição