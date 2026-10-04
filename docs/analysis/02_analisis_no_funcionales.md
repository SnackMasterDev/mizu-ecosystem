# Análisis No Funcional del Proyecto Mizu Ecosystem

## Introducción
Este documento presenta el análisis detallado de los requisitos no funcionales del ecosistema Mizu, organizado por categoría y subproyecto. Incluye métricas, umbrales aceptables y aplicación concreta a cada subproyecto, estableciendo un vínculo verificable con los requisitos originales especificados en `Especificacion_IEEE830.md` y alineado con los principios de la `CONSTITUTION.md`.

## Alcance
El análisis no funcional cubre las siguientes categorías de requisitos:
1. Rendimiento y tiempos de respuesta
2. Seguridad y protección de datos
3. Disponibilidad y confiabilidad
4. Escalabilidad
5. Compatibilidad y portabilidad
6. Usabilidad y experiencia de usuario
7. Operatividad y mantenibilidad
8. Cumplimiento normativo

## Metodología
Para cada requisito no funcional (NFR) identificado en `Especificacion_IEEE830.md`, se han definido:
- Descripción detallada del requisito
- Métricas específicas para medir cumplimiento
- Umbrales aceptables basados en estándares de industria y requisitos del negocio
- Aplicación concreta a cada subproyecto del ecosistema Mizu
- Relación con principios de la Constitución donde aplique
- Estrategias de verificación y validación

## 1. Rendimiento y Tiempos de Respuesta

### NFR-EX-03: Disponibilidad 24/7 (Experience)
- **Descripción**: El sistema web debe estar disponible las 24 horas del día, los 7 días de la semana
- **Métrica**: Porcentaje de uptime mensual
- **Umbral aceptable**: ≥ 99.5% de uptime mensual (menos de 3.65 horas de downtime mensual)
- **Aplicación a subproyectos**:
  - Mizu Experience: Requisito directo
  - Mizu Go: Requisito directo para backend y APIs
  - Mizu Order Hub/Admin/Stock: Requisito directo para acceso remoto y sincronización
- **Principios relacionados**: 
  - [Principio 3] Disponibilidad y Confiabilidad
  - [Principio 9] Escalabilidad
- **Estrategias de verificación**:
  - Monitoreo continuo con health checks
  - Reportes de disponibilidad mensuales
  - Pruebas de failover y recuperación

### NFR-GO-03: Tiempo de respuesta < 2 segundos en endpoints críticos (Go)
- **Descripción**: Las APIs críticas de la aplicación móvil deben responder en menos de 2 segundos
- **Métrica**: Tiempo de respuesta promedio (p95) de endpoints críticos
- **Umbral aceptable**: ≤ 2 segundos para p95 de tiempo de respuesta
- **Endpoints críticos identificados**:
  - Autenticación y refresco de tokens
  - Consulta de menú y productos
  - Envío de pedidos
  - Consulta de historial de pedidos
  - Consultas de puntos y recompensas
- **Aplicación a subproyectos**:
  - Mizu Go: Requisito directo para experiencia de usuario
  - Mizu Experience: Compartir mismos endpoints backend
  - Mizu Order Hub/Admin/Stock: Aplicar estándares similares para consistencia
- **Principios relacionados**:
  - [Principio 5] Rendimiento Óptimo
  - [Principio 16] Interfaz amigable para análisis rápido (Admin)
- **Estrategias de verificación**:
  - Monitoreo de performance con APM (Application Performance Monitoring)
  - Pruebas de carga periódicas
  - Optimización de consultas de base de datos y caché

## 2. Seguridad y Protección de Datos

### NFR-EX-02: Seguridad en transacciones (HTTPS y cifrado de datos sensibles) (Experience)
- **Descripción**: Todas las transacciones y comunicaciones deben utilizar HTTPS y cifrado de datos sensibles
- **Métrica**: Porcentaje de tráfico cifrado y cumplimiento de estándares de cifrado
- **Umbral aceptable**: 100% de tráfico externo usando TLS 1.2 o superior
- **Datos sensibles a proteger**:
  - Credenciales de autenticación (passwords, tokens)
  - Información de pago (números de tarjeta, CVV - aunque no se almacenen completos)
  - Información personal de clientes (nombre, dirección, teléfono, email)
  - Historial de pedidos y preferencias
- **Aplicación a subproyectos**:
  - Todos los subproyectos: Requisito transversal para todas las comunicaciones
- **Principios relacionados**:
  - [Principio 2] Seguridad en Transacciones
  - [Principio 18] Privacidad de Datos Sensibles (frontend)
- **Estrategias de verificación**:
  - Escaneos de vulnerabilidades periódicas (OWASP ZAP)
  - Pruebas de penetración
  - Auditoría de configuración de TLS y certificados
  - Verificación de uso de algoritmos de cifrado aprobados (AES-256, etc.)

### NFR-AD-02: Alta confiabilidad en almacenamiento (Admin)
- **Descripción**: El sistema de almacenamiento debe garantizar alta confiabilidad para datos críticos de negocio
- **Métrica**: Durabilidad de datos y RPO/RTO en caso de fallos
- **Umbral aceptable**: 
  - Durabilidad: 99.999999999% (11 nueves) anual
  - RPO global: ≤ 15 minutos para todo el sistema (Recovery Point Objective)
    - Excepción financiera (pagos y nómina): RPO = 0, garantizado por las transacciones ACID de PostgreSQL (no por el mecanismo de respaldo)
    - Capas más estrictas pueden conservar techos menores (p. ej. datos de usuario: RPO = 5 minutos)
  - RTO global: ≤ 4 horas para todo el sistema (Recovery Time Objective)
    - Servicios críticos (pedidos, pagos): RTO = 15 minutos
    - Servicios importantes (usuarios, inventario): RTO = 30 minutos
    - Servicios auxiliares (reportes, analítica): RTO = 4 horas
- **Tipos de datos críticos**:
  - Transacciones financieras
  - Datos de usuarios y credenciales
  - Configuración del sistema y parametrización
  - Logs de auditoría y seguridad
- **Aplicación a subproyectos**:
  - Mizu Admin: Requisito directo para reportes y datos contables
  - Todos los subproyectos: Aplicar según criticidad de los datos manejados
- **Principios relacionados**:
  - [Principio 8] Respaldo y Recuperación de Datos
  - [Principio 9] Escalabilidad
- **Estrategias de verificación**:
  - Pruebas de backup y restauración periódicas
  - Arquitectura con replicación geográfica
  - Monitoreo de integridad de almacenamiento
  - Simulaciones de desastres

## 3. Disponibilidad y Confiabilidad

### NFR-EX-03: Disponibilidad 24/7 (Experience) - Detallado
- Ya descrito en sección de Rendimiento

### NFR-OH-03: Respaldo automático cada 24 horas (Order Hub)
- **Descripción**: El sistema debe realizar respaldos automáticos de datos críticos cada 24 horas
- **Métrica**: Frecuencia y éxito de respaldos programmados
- **Umbral aceptable**: 100% de respaldos exitosos cada 24 horas
- **Datos a respaldar**:
  - Configuración de pedidos y estados
  - Historial de transacciones del día
  - Mapeo de menu y precios
  - Reportes generados
- **Aplicación a subproyectos**:
  - Mizu Order Hub: Requisito directo
  - Mizu Stock: Extender a datos de inventario críticos
  - Mizu Admin: Ya cumple con estándares más altos (NFR-AD-02)
- **Principios relacionados**:
  - [Principio 8] Respaldo y Recuperación de Datos
- **Estrategias de verificación**:
  - Monitoreo de jobs de backup
  - Pruebas de restauración desde backups
  - Verificación de integridad de datos respaldados
  - Almacenamiento en ubicación geográficamente diversa

## 4. Escalabilidad

### NFR-ST-01: Escalabilidad para múltiples sucursales (Stock)
- **Descripción**: El sistema debe escalar horizontalmente para soportar múltiples sucursales sin degradación significativa del rendimiento
- **Métrica**: Rendimiento por sucursal adicional y latencia bajo carga distribuida
- **Umbral aceptable**: 
  - Degradación de rendimiento ≤ 10% al duplicar número de sucursales
  - Soporte mínimo para 50 sucursales concurrentes
- **Aspectos de escalabilidad**:
  - Escalabilidad de base de datos (réplicas; **sharding post-V1.0** — regla del 03: se revisa solo si alguna tabla supera ~100M filas)
  - Escalabilidad de servicios de aplicación (contenedores, load balancing)
  - Escalabilidad de recursos de cómputo (auto-scaling)
  - Escalabilidad de almacenamiento (distribución de datos)
- **Aplicación a subproyectos**:
  - Mizu Stock: Requisito directo para gestión multi-sucursal
  - Mizu Order Hub: Escalabilidad para manejar pedidos de múltiples locaciones
  - Mizu Admin: Agregación de datos de múltiples sucursales
  - Mizu Experience/Go: Filtrado y presentación por sucursal
- **Principios relacionados**:
  - [Principio 9] Escalabilidad
  - [Principio 1] Adaptabilidad y Responsive Design
- **Estrategias de verificación**:
  - Pruebas de carga con simulación de múltiples sucursales
  - Arquitectura basada en microservicios y contenedores
  - Monitoreo de métricas por instancia y sucursal
  - Pruebas de failover entre zonas de disponibilidad

## 5. Compatibilidad y Portabilidad

### NFR-GO-01: Compatibilidad con Android e iOS (Go)
- **Descripción**: La aplicación móvil debe ser compatible con las versiones actuales y dos versiones anteriores de Android e iOS
- **Métrica**: Porcentaje de dispositivos compatibles en el mercado objetivo
- **Umbral aceptable**: ≥ 95% de cobertura de dispositivos activos en mercado latinoamericano
- **Versiones objetivo**:
  - Android: 8.0 (Oreo) y superior
  - iOS: 12.0 y superior
- **Consideraciones de compatibilidad**:
  - Adaptación a diferentes tamaños y densidades de pantalla
  - Compatibilidad con versiones de SDK y frameworks
  - Pruebas en dispositivos reales y emuladores
- **Aplicación a subproyectos**:
  - Mizu Go: Requisito directo
  - Mizu Experience: Compatibilidad web con navegadores modernos (Chrome, Firefox, Safari, Edge)
  - Mizu Order Hub/Admin/Stock: Compatibilidad con Windows 10 y 11
- **Principios relacionados**:
  - [Principio 4] Compatibilidad Multiplataforma
- **Estrategias de verificación**:
  - Matriz de compatibilidad de dispositivos
  - Pruebas automatizadas en servicios de testing de dispositivos
  - Pruebas manuales en dispositivos representativos
  - Monitoreo de crash reports y errores por versión de OS

## 6. Usabilidad y Experiencia de Usuario

### NFR-OH-01: Interfaz sencilla e intuitiva para empleados (Order Hub)
- **Descripción**: La interfaz debe ser fácil de aprender y usar para empleados con mínima capacitación
- **Métrica**: Tiempo para completar tareas principales y puntuación SUS (System Usability Scale)
- **Umbral aceptable**:
  - Tiempo promedio para completar tarea de pedido < 2 minutos
  - Puntuación SUS ≥ 75 (aceptable)
- **Características de usabilidad**:
  - Navegación clara y jerarquía visual
  - Terminología consistente con dominio del restaurante
  - Accesos comunes con un mínimo de clics o taps
  - Feedback visual e inmediato para acciones del usuario
- **Aplicación a subproyectos**:
  - Mizu Order Hub: Requisito directo
  - Mizu Stock: Aplicar principios similares para operadores de inventario
  - Mizu Admin: Adaptar según rol de usuario (analista vs administrador)
  - Mizu Experience/Go: Enfoque en clientes finales (estándares diferentes)
- **Principios relacionados**:
  - [Principio 6] Sencillez e Intuitividad
  - [Principio 16] Interfaz amigable para análisis rápido (Admin)
  - [Principio 13] Alcance y Simplicidad (frontend)
- **Estrategias de verificación**:
  - Pruebas de usabilidad con usuarios reales
  - Análisis de flujos de tarea (task analysis)
  - Métricas de uso y adopción en producción
  - Encuestas de satisfacción periódicas

### NFR-AD-03: Interfaz amigable para análisis rápido (Admin)
- **Descripción**: La interfaz debe permitir acceso rápido a información clave para toma de decisiones
- **Métrica**: Tiempo para acceder a reportes clave y cantidad de clics necesarios
- **Umbral aceptable**:
  - Acceso a reporte principal < 3 clics desde login
  - Tiempo para generar reporte complejo < 5 segundos
- **Características de interfaz**:
  - Dashboards personalizables con KPIs en tiempo real
  - Navegación jerárquica (general → detalle)
  - Exportación rápida a formatos comunes
  - Visualización de datos con gráficos interactivos
- **Aplicación a subproyectos**:
  - Mizu Admin: Requisito directo
  - Mizu Order Hub: Adaptar para reportes operativos
  - Mizu Stock: Adaptar para reportes de inventario
  - Mizu Experience/Go: Métricas de negocio y conversión
- **Principios relacionados**:
  - [Principio 16] Interfaz amigable para análisis rápido
- **Estrategias de verificación**:
  - Pruebas de eficiencia con usuarios analistas
  - Análisis de rutas de acceso críticos
  - Monitoreo de uso de funcionalidades de reporteo
  - Feedback continuo de usuarios de negocio

## 7. Operatividad y Mantenibilidad

### Monitoreo de Salud
- **Descripción**: El sistema debe incluir health checks para detectar y aislar fallos rápidamente
- **Métrica**: Tiempo medio para detectar fallos (MTTD) y tiempo medio para recuperarse (MTTR)
- **Umbral aceptable**:
  - MTTD ≤ 5 minutos para fallos críticos
  - MTTR ≤ 30 minutos para incidentes de menor impacto
- **Componentes a monitorear**:
  - Disponibilidad de APIs y servicios
  - Latencia y tasas de error
  - Uso de recursos (CPU, memoria, disco, red)
  - Colas de mensajes y backlogs
  - Estado de bases de datos y cachés
- **Aplicación a subproyectos**: Transversal a todos los subproyectos
- **Principios relacionados**:
  - [Principio 3] Disponibilidad y Confiabilidad
  - [Principio 15] Monitoreo de Salud (implícito en límites y verificación)
- **Estrategias de verificación**:
  - Implementación de health checks en todos los servicios
  - Integración con sistemas de monitoreo (Prometheus, Grafana)
  - Definición y prueba de alertas
  - Revisiones periódicas de umbrales y notificaciones

## 8. Cumplimiento Normativo

### NFR-AD-01: Cumplimiento normativo contable (Admin)
- **Descripción**: El sistema debe cumplir con todas las regulaciones contables aplicables
- **Métrica**: Número de hallazgos en auditorías contables y cumplimiento de estándares locales
- **Umbral aceptable**: Cero hallazgos mayores en auditorías externas anuales
- **Regulaciones aplicables**:
  - Normas Internacionales de Información Financiera (NIIF) para PYMES
  - Legislación tributaria local y nacional
  - Requisitos de facturación electrónica (según jurisdicción)
  - Normas de retención de documentos contables
- **Aspectos de cumplimiento**:
  - Plan de cuentas adecuado y aplicado consistentemente
  - Asientos contables balanceados y con documentación de soporte
  - Retención adecuada de documentos electrónicos
  - Reportes regulatorios generados automáticamente
- **Aplicación a subproyectos**:
  - Mizu Admin: Requisito directo
  - Mizu Stock: Generar datos correctos para valor de inventario y COGS
  - Mizu Order Hub: Capturar datos de ventas completos y precisos
  - Mizu Experience/Go: Asegurar captura completa de transacciones
- **Principios relacionados**:
  - [Principio 12] Cumplimiento Normativo
- **Estrategias de verificación**:
  - Auditorías contables internas periódicas
  - Revisiones de cumplimiento con asesores externos
  - Pruebas de generación de reportes regulatorios
  - Monitoreo de cambios legislativos y actualizaciones oportunas

## Matriz de Trazabilidad de Requisitos No Funcionales

| NFR ID | Descripción | Métrica | Umbral Aceptable | Subproyectos Afectados | Principios Relacionados |
|--------|-------------|---------|------------------|------------------------|-------------------------|
| NFR-EX-01 | Interfaz adaptable (responsive) | % de dispositivos con experiencia adecuada | ≥ 95% | Experience, Go | [Principio 1] |
| NFR-EX-02 | Seguridad en transacciones | % tráfico TLS 1.2+ | 100% | Todos | [Principio 2] |
| NFR-EX-03 | Disponibilidad 24/7 | Uptime mensual | ≥ 99.5% | Todos | [Principio 3] |
| NFR-GO-01 | Compatibilidad Android/iOS | % dispositivos compatibles | ≥ 95% | Go, Experience (web) | [Principio 4] |
| NFR-GO-03 | Tiempo respuesta < 2s | p95 tiempo respuesta | ≤ 2s | Go, Experience, Order Hub, Admin, Stock | [Principio 5] |
| NFR-OH-01 | Interfaz sencilla e intuitiva | Tiempo tarea pedido / SUS | < 2 min / ≥ 75 | Order Hub, Stock, Admin | [Principio 6] |
| NFR-OH-02 | Integración con Mizu Stock | % éxito sincronización | ≥ 99.9% | Order Hub, Stock | [Principio 7] |
| NFR-OH-03 | Respaldo automático cada 24h | % respaldos exitosos | 100% | Order Hub, Stock, Admin | [Principio 8] |
| NFR-ST-01 | Escalabilidad múltiple sucursal | Degradación rendimiento | ≤ 10% al 2x sucursales | Stock, Order Hub, Admin, Experience, Go | [Principio 9] |
| NFR-ST-02 | Roles de acceso diferenciados | Cumplimiento matriz roles | 100% | Stock, Admin | [Principio 10] |
| NFR-ST-03 | Disponibilidad offline | Tiempo sync tras reconexión | ≤ 2 min | Stock, Order Hub | [Principio 11] |
| NFR-AD-01 | Cumplimiento normativo contable | Hallazgos auditoría | 0 mayores | Admin, Stock, Order Hub, Experience, Go | [Principio 12] |
| NFR-AD-02 | Alta confiabilidad almacenamiento | Durabilidad/RPO/RTO | 11 nueves; RPO global ≤15min (excepción financiera RPO=0 por ACID de PostgreSQL); RTO global ≤4h | Admin, Stock, Order Hub | [Principio 8] |
| NFR-AD-03 | Interfaz amigable análisis rápido | Tiempo acceso reporte | < 3 clics / < 5s | Admin, Order Hub, Stock | [Principio 16] |
| NFR-EX-01 (FE) | Alcance y Simplicidad | Responsabilidad única por componente | Verificación código | Experience, Go, Order Hub, Admin, Stock (frontend) | [Principio 13] |
| NFR-EX-02 (FE) | Tipos Estrictos | Uso de any y strictNullChecks | Verificación código | Experience, Go, Order Hub, Admin, Stock | [Principio 14] |
| NFR-EX-03 (FE) | Fuente de Datos Controlada | Acceso directo a APIs desde componentes | Verificación código | Experience, Go, Order Hub, Admin, Stock | [Principio 15] |
| NFR-EX-04 (FE) | Accesibilidad y Verificación | Cumplimiento WCAG 2.1 AA | Verificación pruebas | Experience, Go, Order Hub, Admin, Stock (frontend) | [Principio 16] |
| NFR-EX-05 (FE) | Dependencias Minimizadas | Valor claro y alternativa ligera | Revisión dependencias | Experience, Go, Order Hub, Admin, Stock | [Principio 17] |
| NFR-EX-06 (FE) | Privacidad de Datos Sensibles | Almacenamiento tokens/PII | Verificación almacenamiento | Experience, Go, Order Hub, Admin, Stock | [Principio 18] |

## Preguntas Abiertas y Decisiones Pendientes
- ¿Cómo equilibraremos los requisitos de seguridad estrictos con la usabilidad para empleados que deben realizar transacciones rápidas durante horas pico?
- ¿Qué nivel de detalle deberíamos incluir en los planes de recuperación ante desastres para diferentes tipos de fallos (parciales vs completos)?
- ¿Deberíamos establecer umbrales diferentes para métricas de rendimiento según el tipo de subproyecto (ej. más estrictos para interfaces de cliente vs internos)?
- ¿Cómo abordaremos la evolución de requisitos regulatorios con el tiempo sin requerir reingeniería significativa del sistema?

## Conclusión
Este análisis no funcional proporciona una base sólida para las fases subsiguientes de diseño e implementación. Cada requisito no funcional ha sido analizado en términos de métricas específicas, umbrales aceptables y aplicación concreta a cada subproyecto del ecosistema Mizu, cumpliendo con el segundo criterio EARS especificado en la spec de organización del análisis.

Los análisis por categoría de NFR incluyen:
- Descripción detallada de cada requisito no funcional
- Métricas específicas para medir cumplimiento
- Umbrales aceptables basados en estándares de industria y requisitos del negocio
- Aplicación concreta a cada subproyecto
- Relación con principios de la Constitución donde aplique
- Estrategias de verificación y validación
- Matriz de trazabilidad que vincula el análisis con los requisitos fuente

Este documento está listo para revisión y sirve como entrada para los modelos de datos, contratos de API, arquitectura técnica y diseño UI/UX que seguirán en los subsiguientes documentos de analysis.