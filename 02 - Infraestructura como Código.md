
Este documento describe cómo LiveTix gestiona su infraestructura de manera replicable, versionada y automatizada. La infraestructura como código (IaC) elimina la configuración manual y garantiza consistencia entre ambientes.

### El Problema

| Desafío                  | Impacto                                           |
| ------------------------ | ------------------------------------------------- |
| **Configuración manual** | Errores humanos, inconsistencias                  |
| **Múltiples ambientes**  | Dev, staging, production deben ser idénticos      |
| **Equipo pequeño**       | No hay tiempo para configurar manualmente         |
| **Auditoría**            | Necesitamos saber qué cambió, cuándo, y por quién |

### Decisiones

#### 1. Estructura del Código

| Decisión         | Valor                              |
| ---------------- | ---------------------------------- |
| **Estructura**   | Mono-repo                          |
| **Herramienta**  | Terraform                          |
| **Organización** | Módulos reutilizables por ambiente |

```
livetix-infra/
├── environments/
│   ├── dev/
│   │   └── main.tf
│   ├── staging/
│   │   └── main.tf
│   └── production/
│       └── main.tf
├── modules/
│   ├── networking/
│   ├── lambda/
│   ├── dynamodb/
│   ├── elasticache/
│   └── monitoring/
└── README.md
```

#### 2. Gestión de Secretos

| Concepto                    | Valor                               |
| --------------------------- | ----------------------------------- |
| **Secretos sensibles**      | AWS Secrets Manager                 |
| **Configuración**           | AWS Parameter Store                 |
| **Autenticación servicios** | IAM Roles (sin secretos)            |
| **Secretos a gestionar**    | **1 solo (Stripe API Key)**         |
| **ElastiCache**             | IAM autenticación (sin contraseñas) |

#### 3. CI/CD

| Pipeline | Herramienta | Deploy |
|----------|-------------|--------|
| **Infraestructura** | GitHub Actions + Terraform | Automático |
| **Aplicación** | GitHub Actions | Automático |

### Pipeline 1: Infraestructura

| Paso | Acción |
|------|--------|
| 1 | Developer hace push a main |
| 2 | `terraform validate` + `terraform plan` |
| 3 | Aprobación manual (production) |
| 4 | `terraform apply` |

### Pipeline 2: Aplicación

| Paso | Acción |
|------|--------|
| 1 | Developer hace push a main |
| 2 | Instalar dependencias |
| 3 | Tests unitarios |
| 4 | Linting |
| 5 | Build (empaquetar Lambda) |
| 6 | Deploy staging |
| 7 | Tests de integración |
| 8 | Aprobación manual (production) |
| 9 | Deploy production |

### Ambientes

| Ambiente | Propósito | Deploy | Aprobación |
|----------|-----------|--------|------------|
| **dev** | Desarrollo | Automático | No |
| **staging** | Validación | Automático | No |
| **production** | Producción | Automático | **Sí** |
