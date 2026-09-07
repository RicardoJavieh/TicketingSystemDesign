La observabilidad permite detectar problemas **antes de que los usuarios se den cuenta**. Este documento describe cómo LiveTix monitorea la plataforma para identificar y resolver incidentes proactivamente.

### El Problema

| Desafío | Impacto |
|---------|---------|
| **Sistema distribuido** | Múltiples servicios, múltiples regiones |
| **Picos de tráfico** | Problemas pueden aparecer y desaparecer rápidamente |
| **Equipo pequeño** | No hay personas dedicadas a monitoreo 24/7 |
| **Usuarios globales** | Problemas pueden afectar solo una región |

### Los 3 Pilares de Observabilidad

#### 1. Métricas (Metrics)

**¿Qué medimos?**

| Categoría | Métricas Clave | Alerta |
|-----------|----------------|--------|
| **Salud** | Errores 5xx, Errores 4xx | > 1% errores |
| **Salud** | Latencia p95/p99 | > 2 segundos |
| **Salud** | Lambda errors | > 0 |
| **Salud** | DynamoDB throttling | > 0 |
| **Salud** | Stripe errores | > 5% de intentos |
| **Negocio** | Ventas por minuto | Si cae > 50% |
| **Negocio** | Boletos vendidos | Si es 0 por 5 min |
| **Negocio** | Carritos abandonados | Si sube > 30% |

#### 2. Logs

**¿Qué registramos?**

| Tipo de Log | Propósito | Retención |
|-------------|-----------|-----------|
| **Application logs** | Debugging reciente | 30 días |
| **Error logs** | Investigar problemas | 90 días |
| **Audit logs** (pagos, cambios) | Cumplimiento legal | 1 año |
| **Metrics** | Tendencias históricas | 1 año |

#### 3. Tracing (Distributed Tracing)

**¿Qué rastreamos?**

| Trace | Propósito |
|-------|-----------|
| **Compra completa** | Desde selección hasta confirmación |
| **Pago** | Desde iniciación hasta respuesta de Stripe |
| **Reserva de asiento** | Desde click hasta confirmación |

### Canales de Notificación

| Tipo de alerta | Canal | Ejemplo |
|----------------|-------|---------|
| **Crítica** (sistema caído) | **Llamada + SMS + Slack + Email** | Lambda errors > 1%, DynamoDB caído |
| **Importante** (degradado) | **SMS + Slack + Email** | Latencia > 2s, pagos fallidos > 5% |
| **Informativa** | **Email** | Resumen diario de ventas |

### Stack de Observabilidad

| Capa | Herramienta | Propósito |
|------|-------------|-----------|
| **Métricas** | CloudWatch Metrics | Métricas nativas de AWS |
| **Logs** | CloudWatch Logs | Centralización de logs |
| **Tracing** | AWS X-Ray | Rastreo distribuido |
| **Alertas** | CloudWatch Alarms → PagerDuty | Orquestación central de alertas |
| **Notificaciones** | PagerDuty | Llamada, SMS, Slack, Email |
| **On-call** | PagerDuty | Manejo de guardias |
| **Incidentes** | PagerDuty | Tracking de incidentes |

### Tradeoffs

| Decisión | Ventaja | Sacrificio |
|----------|---------|------------|
| CloudWatch sobre Datadog | Integrado con AWS, sin costo extra | Menos features que Datadog |
| X-Ray sobre Jaeger | Managed, sin infra adicional | Vendor lock-in |
| PagerDuty sobre SNS | Orquestación central, on-call, escalation | Costo adicional |
| Retención variable | Balance entre costo y cumplimiento | Complejidad de configuración |
