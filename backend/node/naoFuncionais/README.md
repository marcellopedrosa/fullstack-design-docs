# Requisitos Não Funcionais

Este documento descreve os requisitos não funcionais do CSV Service, abordando aspectos técnicos, operacionais e de qualidade do microserviço.

## Performance e Escalabilidade

### Métricas de Performance
- **Tempo de Resposta**: < 200ms para 95% das requisições
- **Throughput**: 1000 requisições/segundo por instância
- **Latência**: < 100ms para operações de leitura
- **Concorrência**: Suporte a 100 usuários simultâneos por instância

### Escalabilidade
- Escalabilidade horizontal via containers
- Auto-scaling baseado em métricas de uso
- Balanceamento de carga entre instâncias
- Cache distribuído para dados frequentes

## Disponibilidade e Resiliência

### SLA (Service Level Agreement)
- Disponibilidade: 99.95% em horário comercial
- MTTR (Mean Time To Recovery): < 30 minutos
- MTBF (Mean Time Between Failures): > 720 horas
- RPO (Recovery Point Objective): < 5 minutos
- RTO (Recovery Time Objective): < 15 minutos

### Estratégias de Resiliência
1. **Circuit Breaker**
   - Timeout: 5 segundos
   - Failure Threshold: 50%
   - Reset Timeout: 30 segundos

2. **Retry Policy**
   - Max Retries: 3
   - Backoff: Exponencial
   - Jitter: 100-300ms

3. **Fallback**
   - Cache local
   - Modo degradado
   - Respostas default

## Segurança

### Autenticação e Autorização
1. **Keycloak Integration**
   - JWT validation
   - Role-based access control
   - Token refresh
   - SSO support

2. **API Security**
   - Rate limiting: 1000 req/min por IP
   - CORS configuration
   - Input validation
   - Output sanitization

### Proteção de Dados
- Criptografia em trânsito (TLS 1.3)
- Criptografia em repouso (AES-256)
- Mascaramento de dados sensíveis
- Auditoria de acessos

## Observabilidade

### Logging
1. **Padrões**
   - Formato JSON estruturado
   - Correlação de requisições (trace ID)
   - Níveis de log padronizados
   - Rotação de logs

2. **Informações Registradas**
   ```json
   {
     "timestamp": "2025-05-07T10:00:00Z",
     "level": "INFO",
     "service": "csv-service",
     "traceId": "abc-123",
     "method": "GET",
     "path": "/api/v1/attributes",
     "statusCode": 200,
     "duration": 45,
     "userId": "user-123"
   }
   ```

### Métricas
1. **Métricas de Sistema**
   - CPU Usage
   - Memory Usage
   - Disk I/O
   - Network Traffic

2. **Métricas de Aplicação**
   - Request Count
   - Response Time
   - Error Rate
   - Active Users

3. **Métricas de Negócio**
   - Atributos criados/hora
   - Consultas/minuto
   - Taxa de cache hit
   - Uso por domínio

### Tracing
- Distributed tracing (OpenTelemetry)
- Trace sampling (5% das requisições)
- Span collection
- Performance profiling

## Infraestrutura

### Requisitos de Hardware
- **CPU**: 2 vCPUs por instância
- **Memória**: 4GB RAM por instância
- **Disco**: 20GB SSD
- **Rede**: 1Gbps

### Container
```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    cpu: "2000m"
    memory: "4Gi"
```

### Configuração
1. **Environment Variables**
   ```env
   NODE_ENV=production
   PORT=3000
   DB_CONNECTION=postgresql
   CACHE_PROVIDER=redis
   LOG_LEVEL=info
   TRACING_ENABLED=true
   ```

2. **Feature Flags**
   ```json
   {
     "enableCache": true,
     "enableTracing": true,
     "enableMetrics": true,
     "enableRateLimit": true
   }
   ```

## Cache

### Estratégia
1. **Níveis de Cache**
   - Cache L1: Memory (Node.js)
   - Cache L2: Redis
   - Cache L3: CDN (assets)

2. **Políticas**
   - TTL: 5 minutos (default)
   - Cache-Control: public, max-age=300
   - Invalidação: por tag
   - Warm-up: endpoints críticos

## Monitoramento

### Alertas
1. **Thresholds**
   - CPU > 80% por 5 minutos
   - Memória > 85% por 5 minutos
   - Error Rate > 1% por 5 minutos
   - Response Time > 500ms por 5 minutos

2. **Canais**
   - Email para equipe de suporte
   - Webhook para Slack
   - PagerDuty para on-call
   - Dashboard em tempo real

### Health Check
```typescript
{
  "/health": {
    "interval": "30s",
    "timeout": "5s",
    "httpGet": {
      "path": "/health",
      "port": 3000
    }
  }
}
```

## Backup e Recuperação

### Estratégia de Backup
- Backup completo: Diário às 00:00
- Backup incremental: A cada 4 horas
- Retenção: 30 dias
- Localização: Multi-região

### Disaster Recovery
1. **Plano de Recuperação**
   - Failover automático
   - Recuperação de dados
   - Validação de integridade
   - Rollback procedure

## Referências

- [NestJS Performance](https://docs.nestjs.com/techniques/performance)
- [Kubernetes Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [Redis Cache Patterns](https://redis.io/topics/patterns)
- [Microservices Patterns](https://microservices.io/patterns/index.html)
