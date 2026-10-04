# Mizu: Ecosistema Integral de Gestión para Cadenas de Restaurantes

## Tabla de Contenidos
- [1. Introducción](#1-introducción)
- [2. Definición del Problema](#2-definición-del-problema)
- [3. Objetivo del Proyecto](#3-objetivo-del-proyecto)
- [4. Alcance del Sistema (Módulos)](#4-alcance-del-sistema-módulos)
- [5. Roles de Usuario](#5-roles-de-usuario)
- [6. Estado del Proyecto](#6-estado-del-proyecto)
- [7. Arquitectura del Sistema](#7-arquitectura-del-sistema)
- [8. Tecnologías planificadas](#8-tecnologías-planificadas)
- [9. Estado del repositorio](#9-estado-del-repositorio)
- [10. API Reference](#10-api-reference)
- [11. Base de Datos](#11-base-de-datos)
- [12. Roadmap / Fases próximas](#12-roadmap--fases-próximas)
- [13. Contribución al Proyecto](#13-contribución-al-proyecto)
- [14. Licencia](#14-licencia)
- [15. Contacto y Soporte](#15-contacto-y-soporte)

## 1. Introducción
Mizu es una solución tecnológica avanzada diseñada para centralizar la administración, operación y fidelización de clientes en cadenas de restaurantes con múltiples sedes. El sistema propone un ecosistema interconectado que garantiza la fluidez de la información desde la interacción del cliente hasta la gestión gerencial.

## 2. Definición del Problema
Las cadenas de restaurantes actuales suelen operar con sistemas fragmentados que dificultan la sincronización de datos entre sedes. La falta de una plataforma unificada genera ineficiencias en el control de inventarios, procesos de venta manuales y una desconexión con el comportamiento del consumidor, limitando la capacidad de crecimiento y fidelización del negocio.

## 3. Objetivo del Proyecto
Desarrollar un ecosistema de software robusto y escalable que permita la gestión eficiente de ventas, el control estricto de suministros y la retención de clientes, optimizando la toma de decisiones basada en datos centralizados para una cadena de restaurantes a nivel nacional.

## 4. Alcance del Sistema (Módulos)
El ecosistema Mizu se estructura en cinco aplicaciones clave:

### 4.1 Mizu Experience (Web)
**Plataforma orientada al cliente final** para la exploración del menú y la realización de pedidos rápidos sin necesidad de registro obligatorio, priorizando la conversión de ventas.
- **URL pública**: `experience.mizu.com`
- **Acceso**: Clientes finales (sin autenticación requerida para menú básico)
- **Objetivo**: Maximizar conversión de visitas a pedidos

### 4.2 Mizu Go (Móvil)
**Aplicación enfocada en la retención de clientes recurrentes** mediante un sistema de fidelización de puntos, historial de pedidos y promociones personalizadas.
- **Plataformas**: Android e iOS
- **Acceso**: Clientes registrados (autenticación requerida)
- **Objetivo**: Incrementar frecuencia de compra y valor de vida del cliente

### 4.3 Mizu Order Hub (Escritorio)
**Terminal central de pedidos (POS)** encargado de la gestión operativa local, facturación en caja y la recepción en tiempo real de las órdenes provenientes de Mizu Experience y Mizu Go.
- **Plataforma**: Windows Desktop
- **Acceso**: Personal operativo (cajeros, meseros)
- **Objetivo**: Centralizar gestión de pedidos en punto de venta

### 4.4 Mizu Stock (Escritorio)
**Herramienta especializada para el control riguroso de inventarios**, gestión de proveedores, costeo de recetas y alertas de suministros por sede.
- **Plataforma**: Windows Desktop
- **Acceso**: Personal de almacén y administradores
- **Objetivo**: Optimizar niveles de inventario y reducir mermas

### 4.5 Mizu Admin (Escritorio)
**Panel de control gerencial** para el análisis de reportes consolidados, gestión de talento humano, configuración del sistema y analítica de ventas a nivel nacional.
- **Plataforma**: Windows Desktop
- **Acceso**: Administradores y directivos
- **Objetivo**: Toma de decisiones estratégicas basada en datos

## 5. Roles de Usuario
### 5.1 Clientes
Usuarios externos que interactúan con las interfaces digitales (Experience y Go) para consumo y fidelización.
- Acceso anónimo al menú en Experience
- Registro requerido para Go y funcionalidades avanzadas de Experience

### 5.2 Personal Operativo
Cajeros, meseros y jefes de almacén responsables de la operación diaria (Order Hub y Stock).
- Acceso basado en roles específicos por módulo
- Interfaces simplificadas para tareas operativas

### 5.3 Administradores
Personal directivo encargado de la supervisión global, configuración y rentabilidad del negocio (Admin).
- Acceso completo a todos los módulos
- Funcionalidades de configuración y análisis avanzado

## 6. Estado del Proyecto
El proyecto se encuentra actualmente en la **Fase de Análisis** bajo el programa de formación ADSO (SENA). Las actividades en curso incluyen el levantamiento de requisitos funcionales, el modelado de procesos de negocio y la definición de la arquitectura de datos inicial.

**Próximos hitos:**
- [ ] Fase de Diseño (Arquitectura detallada y prototipos UI/UX)
- [ ] Fase de Implementación (Desarrollo de módulos core)
- [ ] Fase de Testing (Pruebas unitarias, de integración y de usuario)
- [ ] Fase de Despliegue (Deploy a ambiente de prueba)
- [ ] Fase de Capacitación (Training para usuarios finales)
- [ ] Fase de Producción (Lanzamiento oficial)

## 7. Arquitectura del Sistema
Mizu utiliza una arquitectura de microservicios con comunicación mediante APIs RESTful y mensajería asíncrona para garantizar escalabilidad, mantenibilidad y tolerancia a fallos.

### 7.1 Arquitectura de Alto Nivel
```
┌─────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│  Mizu Experience│    │     Mizu Go      │    │  Otros Canales   │
│    (Web)        │    │    (Móvil)       │    │  (Call Center,   │
└─────────────────┘    └──────────────────┘    │     etc.)        │
          │                   │                 └──────────────────┘
          ▼                   ▼                         │
┌───────────────────────────────────────────────────────────────────┐
│                 API Gateway & Load Balancer                       │
└───────────────────────────────────────────────────────────────────┘
          │
          ▼
┌───────────────────────────────────────────────────────────────────┐
│                    Servicios de Aplicación                        │
├─────────────────────┬─────────────────────┬─────────────────────┤
│  Auth Service       │  Menu Service       │  Order Service      │
│  Payment Service    │  Notification Svc   │  Inventory Svc      │
│  User Service       │  Promotion Svc      │  Report Service     │
└─────────────────────┴─────────────────────┴─────────────────────┘
          │                   │                 │
          ▼                   ▼                 ▼
┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐
│   Base de Datos     │ │   Caché (Redis)   │ │   Cola de Mensajes  │
│   (PostgreSQL)      │ │                   │ │   (RabbitMQ)        │
└─────────────────────┘ └─────────────────────┘ └─────────────────────┘
          │                   │                 │
          ▼                   ▼                 ▼
┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐
│ Mizu Order Hub      │ │ Mizu Stock        │ │ Mizu Admin          │
│    (Desktop)        │ │    (Desktop)      │ │    (Desktop)        │
└─────────────────────┘ └─────────────────────┘ └─────────────────────┘
```

### 7.2 Principios Arquitectónicos
- **Separación de Responsabilidades**: Cada servicio tiene una responsabilidad única y bien definida
- **Escalabilidad Horizontal**: Servicios pueden escalarse independientemente según demanda
- **Tolerancia a Fallos**: Diseño resiliente con circuit breakers y retry mechanisms
- **Seguridad por Capas**: Autenticación, autorización, cifrado y auditoría en múltiples niveles
- **Observabilidad**: Logging distribuido, tracing y métricas para monitoreo
- **Deploy Independiente**: Cada servicio puede desplegarse sin afectar otros

## 8. Tecnologías planificadas
*(Tecnologías seleccionadas en la fase de análisis; el código de implementación aún no existe — ver §9 "Estado del repositorio".)*

### 8.1 Frontend
- **Mizu Experience**: React 18 + TypeScript + Tailwind CSS
- **Mizu Go**: React Native + TypeScript + Expo
- **Mizu Order Hub/Admin/Stock**: Electron + React + TypeScript

### 8.2 Backend
- **Lenguaje**: Node.js 20.x + TypeScript
- **Framework**: Express.js + NestJS (para servicios complejos)
- **API**: RESTful con OpenAPI 3.0 Documentation
- **Mensajería**: RabbitMQ para eventos asíncronos
- **Caché**: Redis para sesiones y datos frecuentes

### 8.3 Base de Datos
- **Primaria**: PostgreSQL 15+ (datos relacionales y transaccionales)
- **Búsqueda**: Elasticsearch 8+ (búsqueda de texto completo en menús y reseñas)

### 8.4 Infraestructura y DevOps
- **Contenedores**: Docker 24.x
- **Orquestación**: Kubernetes 1.28+
- **CI/CD**: GitHub Actions + Helm Charts
- **Monitoreo**: Prometheus + Grafana + ELK Stack
- **Seguridad**: OWASP ZAP + SCA tools + Dependabot
- **Hosting**: AWS (ECS/EKS, RDS, S3, CloudFront) o equivalente

### 8.5 Herramientas de Desarrollo
- **IDE**: VS Code + Extensions recomendadas
- **Version Control**: Git + GitHub Flow
- **Testing**: Jest + Cypress + Playwright + Postman/Newman
- **Calidad de Código**: ESLint + Prettier + SonarQube
- **Documentación**: Swagger/OpenAPI + JSDoc + MkDocs

## 9. Estado del repositorio

Este repositorio se encuentra en la **fase de análisis** del programa ADSO (SENA): **aún no existe código de implementación**. Contiene:

- Documentación de requisitos y análisis: `docs/requirements/` y `docs/analysis/`
- Especificaciones del flujo SDD: `specs/`
- Contexto para agentes: `AGENTS.md` y `MEMORY.md`
- Residuos de sesión de desarrollo: `package.json` y `package-lock.json` (ignorados por `.gitignore`; `package.json` **no define scripts ejecutables**)

Las instrucciones de **instalación, desarrollo y uso** de los 5 módulos se publicarán en la **fase de implementación** (ver "Roadmap / Fases próximas", §12).

## 10. API Reference

En la fase de análisis, los contratos de API están documentados en [`docs/analysis/04_contratos_de_api.md`](docs/analysis/04_contratos_de_api.md) (estilo OpenAPI 3.0). El documento de referencia completo, [`docs/API_REFERENCE.md`](docs/API_REFERENCE.md), está **en preparación** (se rellenará en la fase 2, spec 004).

La API cubrirá: autenticación y usuarios, menú y productos, pedidos y pagos, inventario y proveedores, y reportes y analítica.

## 11. Base de Datos

### 11.1 Esquema General
La base de datos única en V1.0 es **PostgreSQL** (modelo relacional; sin almacenamiento no relacional). El siguiente esquema es solo ilustrativo:

```
┌─────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│   Users         │    │   Restaurants    │    │     Menu Items   │
├─────────────────┤    ├──────────────────┤    ├──────────────────┤
│ id (PK)         │    │ id (PK)          │    │ id (PK)          │
│ email           │    │ name             │    │ restaurant_id (FK)│
│ password_hash   │    │ address          │    │ name             │
│ role            │    │ phone            │    │ description      │
│ created_at      │    │ created_at       │    │ price            │
│ updated_at      │    │ updated_at       │    │ category         │
└─────────────────┘    └──────────────────┘    └──────────────────┘
          │                   │                         │
          ▼                   ▼                         ▼
┌─────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│   Orders        │    │   Order Items    │    │   Inventory      │
├─────────────────┤    ├──────────────────┤    ├──────────────────┤
│ id (PK)         │    │ id (PK)          │    │ id (PK)          │
│ user_id (FK)    │    │ order_id (FK)    │    │ ingredient_id (FK)│
│ restaurant_id   │    │ menu_item_id (FK)│    │ quantity         │
│ status          │    │ quantity         │    │ unit             │
│ total_amount    │    │ price            │    │ min_threshold    │
│ created_at      │    │ created_at       │    │ current_stock    │
│ updated_at      │    │ updated_at       │    │ last_updated     │
└─────────────────┘    └──────────────────┘    └──────────────────┘
```

Modelo completo y fuente de verdad: **26 entidades** en [`docs/analysis/03_modelos_de_datos.md`](docs/analysis/03_modelos_de_datos.md) (incluida la integridad referencial global de 38 FKs y la convención de PK UUID v7).

### 11.2 Entidades Principales
Las entidades principales del modelo (lista completa: 26 en `docs/analysis/03_modelos_de_datos.md`):

- **Acceso y personas**: `Usuario`, `Credencial`, `Rol`, `Usuario_Rol`, `Empleado`
- **Sucursal y ventas**: `Sucursal`, `Mesa`, `Reserva`, `CategoriaProducto`, `Producto`, `DetallePedido`, `Pedido`, `Pago`
- **Inventario y compras**: `Ingrediente`, `ProductoIngrediente`, `MovimientoInventario`, `Proveedor`, `ProveedorIngrediente`, `OrdenCompra`, `OrdenCompraDetalle`
- **RRHH y finanzas**: `Asistencia`, `Nomina`, `Gasto`
- **Fidelización y ofertas**: `Puntos`, `TransaccionPuntos`, `Oferta`

### 11.3 Relaciones y Constraints
- **Integridad Referencial**: Todas las FK mantienen consistencia de datos (política global ON DELETE/ON UPDATE: `03_modelos_de_datos.md` §5)
- **Cascading Updates**: Actualizaciones en padres propagan a hijos cuando aplica
- **Restrictive Deletes**: Eliminación prohibida cuando existen registros dependientes
- **Índices Optimizados**: Índices explícitos en todas las FK y en columnas de paths calientes

## 12. Roadmap / Fases próximas

Las fases siguientes son coherentes con los hitos de "Estado del Proyecto" (§6):

| Fase | Contenido | Estado |
|------|-----------|--------|
| **Diseño** | Arquitectura detallada, prototipos UI/UX, contratos de API y de datos | Pendiente |
| **Implementación** | Desarrollo de los 5 módulos y servicios compartidos; en esta fase se publican las instrucciones de instalación y uso | Pendiente |
| **Testing** | Pruebas unitarias, de integración y de usuario | Pendiente |
| **Despliegue** | Despliegue a ambiente de prueba | Pendiente |

Las fases de **Capacitación** y **Producción** siguen listadas en los hitos de §6. El repositorio actual (fase de análisis) no tiene tooling de calidad ni de despliegue configurado.

## 13. Contribución al Proyecto
¡Gracias por considerar contribuir a Mizu! Sigue estas guías para hacer tus contribuciones efectivas.

### 13.1 Cómo Contribuir
1. **Fork** el repositorio
2. **Crea una rama** para tu feature: `git checkout -b feature/nueva-funcionalidad`
3. **Realiza tus cambios** siguiendo las convenciones de código
4. **Agrega pruebas** para tu funcionalidad
5. **Ejecuta las pruebas** para asegurar que no rompes nada existente
6. **Haz commit** de tus cambios con mensaje descriptivo
7. **Push** a tu fork: `git push origin feature/nueva-funcionalidad`
8. **Abre un Pull Request** describiendo tus cambios

### 13.2 Convenciones de Código
- **JavaScript/TypeScript**: ESLint con configuración de Airbnb + Prettier
- **Commits**: Conventional Commits (feat:, fix:, docs:, style:, refactor:, perf:, test:, chore:)
- **Branches**: feature/*, bugfix/*, release/*, hotfix/*
- **Pull Requests**: Descripción clara, referencia a issue, screenshots si aplica UI
- **Code Review**: Mínimo 2 aprobaciones antes de merge

### 13.3 Reportar Issues
Al reportar un issue, por favor incluye:
- **Título descriptivo** y conciso
- **Entorno** (SO, versión de Node, navegador si aplica web)
- **Pasos para reproducir** el problema
- **Comportamiento esperado** vs **comportamiento real**
- **Screenshots** o **logs** relevantes
- **Possible solution** si tienes una idea

### 13.4 Código de Conducta
Nos comprometemos a proporcionar un ambiente acogedor y seguro para todos. Se espera que todos los colaboradores:
- Sean respetuosos y considerados
- Acepten críticas constructivas
- Enfóquense en lo mejor para el proyecto
- Sean empáticos hacia otros colaboradores

## 14. Licencia
Este proyecto **pretende** publicarse bajo la Licencia MIT; el archivo `LICENSE` **aún no existe** en el repositorio y se creará en la fase de implementación.

## 15. Contacto y Soporte
### 15.1 Equipo de Desarrollo
- **Líder de Proyecto**: Juan Camilo Garcia Gomez

### 15.2 Canales de Soporte
- **Issues**: GitHub Issues para reportes de bugs y feature requests
- **Email**: dev@mizu.com para consultas técnicas
- **Documentación**: Este repositorio y el wiki asociado

---

*Documentación mantenida por el equipo de Mizu. Última actualización: 04/10/2026 (alineación fase 1, spec 003).*

*Para sugerencias de mejora en esta documentación, por favor crea un issue o contribuye directamente mediante pull request.*