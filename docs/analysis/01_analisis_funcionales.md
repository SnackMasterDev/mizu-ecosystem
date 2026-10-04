# Análisis Funcional del Proyecto Mizu Ecosystem

## Introducción
Este documento presenta el análisis detallado de los requisitos funcionales del ecosistema Mizu, organizado por subproyecto. Incluye casos de uso, historias de usuario, criterios de aceptación, flujos de proceso y priorización, estableciendo un vínculo verificable con los requisitos originales especificados en `Especificacion_IEEE830.md`.

## Alcance
El análisis funcional cubre los cinco subproyectos del ecosistema Mizu:
1. Mizu Experience (Web)
2. Mizu Go (App móvil)
3. Mizu Order Hub (Escritorio - TPV/POS)
4. Mizu Stock (Escritorio - Inventario)
5. Mizu Admin (Escritorio - Gerencia)

## Metodología
Para cada subproyecto, se han identificado:
- Casos de uso funcionales (CU) según `Especificacion_IEEE830.md`
- Historias de usuario sugeridas con estimación de esfuerzo en puntos
- Criterios de aceptación derivados de los casos de uso
- Flujos de proceso principales
- Priorización basada en valor de negocio y dependencias

## 1. Mizu Experience (Web)

### Casos de Uso Funcionales
- **CU-EX-01**: Consultar menú
- **CU-EX-02**: Realizar reserva desde la web
- **CU-EX-03**: Hacer pedido en línea con confirmación automática
- **CU-EX-04**: Generar historial de pedidos del cliente

### Historias de Usuario Sugeridas
| ID | Descripción | Puntos |
|----|-------------|--------|
| HU-EX-01 | Consultar menú | 3 pts |
| HU-EX-02 | Realizar reserva | 4 pts |
| HU-EX-03 | Hacer pedido en línea | 5 pts |
| HU-EX-04 | Generar historial de pedidos | 3 pts |

### Criterios de Aceptación
**CU-EX-01 (Consultar menú):**
- El sistema debe mostrar el menú completo organizado por categorías
- Debe permitir filtrado por tipo de comida, precio y disponibilidad
- Debe indicar claramente los platos disponibles y los que están fuera de stock
- Debe cargar el menú en menos de 2 segundos (según NFR-EX-03)

**CU-EX-02 (Realizar reserva desde la web):**
- El sistema debe permitir seleccionar fecha, hora y número de comensales
- Debe validar la disponibilidad de mesas para el horario solicitado
- Debe solicitar información de contacto mínima (nombre, teléfono)
- Debe enviar confirmación inmediata por email o SMS

**CU-EX-03 (Hacer pedido en línea con confirmación automática):**
- El sistema debe permitir agregar productos al carrito desde el menú
- Debe calcular el total incluyendo impuestos y propinas sugeridas
- Debe ofrecer múltiples métodos de pago (tarjeta, efectivo al entregar, etc.)
- Debe generar un número de pedido y estimar el tiempo de preparación
- Debe enviar una confirmación automática por email/SMS al completar el pago

**CU-EX-04 (Generar historial de pedidos del cliente):**
- El sistema debe almacenar el historial de pedidos por cliente (registrado o no)
- Debe permitir filtrado por fecha, estado y total del pedido
- Debe mostrar el detalle de cada pedido con productos, cantidades y precios
- Debe permitir repetir pedidos anteriores con un solo clic

### Flujos de Proceso
**Flujo de Consulta de Menú:**
1. Cliente accede a la web Mizu Experience
2. Sistema carga y muestra menú organizado por categorías
3. Cliente filtra o busca platos específicos
4. Cliente visualiza detalles de platos seleccionados (ingredientes, alérgenos, precio)

**Flujo de Pedido en Línea:**
1. Cliente consulta menú y selecciona productos
2. Cliente revisa carrito y procede al pago
3. Sistema solicita información de entrega y pago
4. Sistema procesa pago y genera confirmación
5. Sistema notifica a cocina y asigna tiempo estimado
6. Cliente recibe notificación de confirmación y seguimiento

### Priorización
1. **CU-EX-01** (Consultar menú) - Alta prioridad (punto de entrada principal)
2. **CU-EX-03** (Hacer pedido en línea) - Alta prioridad (genera ingresos directos)
3. **CU-EX-04** (Historial de pedidos) - Media prioridad (mejora experiencia cliente)
4. **CU-EX-02** (Reservas desde web) - Media prioridad (dependiente de política local)

### Matriz de Trazabilidad
| CU ID | Descripción | HU ID | Puntos | NFR Relacionados |
|-------|-------------|-------|--------|------------------|
| CU-EX-01 | Consultar menú | HU-EX-01 | 3 pts | NFR-EX-01, NFR-EX-03 |
| CU-EX-02 | Realizar reserva desde la web | HU-EX-02 | 4 pts | NFR-EX-01, NFR-EX-03 |
| CU-EX-03 | Hacer pedido en línea con confirmación automática | HU-EX-03 | 5 pts | NFR-EX-02, NFR-EX-03 |
| CU-EX-04 | Generar historial de pedidos del cliente | HU-EX-04 | 3 pts | NFR-EX-03 |

## 2. Mizu Go (App móvil)

### Casos de Uso Funcionales
- **CU-GO-01**: Registro y autenticación de clientes
- **CU-GO-02**: Realizar pedidos y reservas desde la app
- **CU-GO-03**: Mostrar promociones personalizadas
- **CU-GO-04**: Sistema de puntos y recompensas

### Historias de Usuario Sugeridas
| ID | Descripción | Puntos |
|----|-------------|--------|
| HU-GO-01 | Registro de usuario | 3 pts |
| HU-GO-02 | Pedido desde la app | 5 pts |
| HU-GO-03 | Ver promociones personalizadas | 3 pts |
| HU-GO-04 | Consultar y canjear puntos | 3 pts |

### Criterios de Aceptación
**CU-GO-01 (Registro y autenticación de clientes):**
- El sistema debe permitir registro con email, teléfono o redes sociales
- Debe validar la fuerza de la contraseña y confirmar la cuenta vía email/SMS
- Debe ofrecer autenticación biométrica (huella, Face ID) en dispositivos compatibles
- Debe mantener la sesión activa con renovación segura de tokens

**CU-GO-02 (Realizar pedidos y reservas desde la app):**
- El sistema debe ofrecer funcionalidad equivalente a la web pero optimizada para móvil
- Debe usar GPS para sugerir sucursales cercanas y calcular tiempos de llegada
- Debe permitir guardar direcciones favoritas y métodos de pago
- Debe enviar notificaciones push sobre el estado del pedido y promociones

**CU-GO-03 (Mostrar promociones personalizadas):**
- El sistema debe analizar el historial de compras para ofrecer promociones relevantes
- Debe permitir la configuración de preferencias de comunicación (push, email, SMS)
- Debe mostrar promociones en tiempo real basado en ubicación y hora del día
- Debe rastrear el uso de promociones para evitar fraudes y abusos

**CU-GO-04 (Sistema de puntos y recompensas):**
- El sistema debe acumular puntos por cada compra (por ejemplo: 1 punto por $1 gastado)
- Debe permitir el canjeo de puntos por descuentos, productos gratis o beneficios especiales
- Debe mostrar el balance de puntos y el historial de transacciones en tiempo real
- Debe ofrecer niveles de lealtad con beneficios crecientes (bronce, plata, oro)

### Flujos de Proceso
**Flujo de Registro y Autenticación:**
1. Usuario descarga e instala Mizu Go desde app store
2. Usuario elige método de registro (email/tel/redes sociales)
3. Sistema envía código de verificación y valida identidad
4. Usuario crea perfil con preferencias y datos de pago opcionales
5. Sistema otorga acceso y opcionalmente activa autenticación biométrica

**Flujo de Pedido desde la App:**
1. Usuario abre app y permite acceso a ubicación (opcional)
2. Sistema muestra menú y sucursales cercanas ordenadas por distancia
3. Usuario selecciona productos, personaliza orden y agrega al carrito
4. Sistema calcula total, aplica promociones y sugiere método de pago
5. Usuario confirma pago y recibe estimado de preparación/llegada
6. Sistema envía notificaciones en tiempo real del estado del pedido

### Priorización
1. **CU-GO-01** (Registro y autenticación) - Alta prioridad (requisito fundamental)
2. **CU-GO-02** (Pedidos y reservas desde app) - Alta prioridad (funcionalidad principal)
3. **CU-GO-04** (Sistema de puntos y recompensos) - Alta prioridad (fidelización clave)
4. **CU-GO-03** (Promociones personalizadas) - Media prioridad (mejora experiencia)

### Matriz de Trazabilidad
| CU ID | Descripción | HU ID | Puntos | NFR Relacionados |
|-------|-------------|-------|--------|------------------|
| CU-GO-01 | Registro y autenticación de clientes | HU-GO-01 | 3 pts | NFR-GO-01, NFR-GO-02 |
| CU-GO-02 | Realizar pedidos y reservas desde la app | HU-GO-02 | 5 pts | NFR-GO-01, NFR-GO-02, NFR-GO-03 |
| CU-GO-03 | Mostrar promociones personalizadas | HU-GO-03 | 3 pts | NFR-GO-02 |
| CU-GO-04 | Sistema de puntos y recompensas | HU-GO-04 | 3 pts | NFR-GO-02 |

## 3. Mizu Order Hub (Escritorio - TPV/POS)

### Casos de Uso Funcionales
- **CU-OH-01**: Recepción de pedidos en tiempo real
- **CU-OH-02**: Registro manual de pedidos en el establecimiento
- **CU-OH-03**: Gestión de estado de pedidos (pendiente, preparación, entregado)
- **CU-OH-04**: Generación de reportes diarios de pedidos
- **CU-OH-05**: Registrar entrada y salida individual de empleados

### Historias de Usuario Sugeridas
| ID | Descripción | Puntos |
|----|-------------|--------|
| HU-OH-01 | Ver pedidos en tiempo real | 4 pts |
| HU-OH-02 | Registro manual de pedidos | 3 pts |
| HU-OH-03 | Cambiar estado de pedidos | 3 pts |
| HU-OH-04 | Generar reportes diarios | 5 pts |
| HU-OH-05 | Registrar entrada/salida de empleados | 3 pts |

### Criterios de Aceptación
**CU-OH-01 (Recepción de pedidos en tiempo real):**
- El sistema debe mostrar los pedidos entrantes de los canales web y app en tiempo real
- Debe permitir filtrado por hora, tipo de canal y estado de preparación
- Debe generar notificaciones visuales y sonoras para nuevos pedidos
- Debe integrarse con impresoras de cocina para generar tickets automáticamente

**CU-OH-02 (Registro manual de pedidos en el establecimiento):**
- El sistema debe permitir la creación rápida de pedidos para clientes presenciales
- Debe ofrecer el menú completo con modificadores y opciones de personalización
- Debe calcular el total incluyendo impuestos y aplicar descuentos autorizados
- Debe generar el número de pedido y enviarlo a la cocina inmediatamente

**CU-OH-03 (Gestión de estado de pedidos):**
- El sistema debe permitir actualización manual de estados: pendiente → preparación → listo para entrega → entregado
- Debe registrar timestamps automáticos para cada cambio de estado
- Debe alertar cuando un pedido excede tiempos estándar de preparación
- Debe permitir agregar notas especiales (alergias, urgencias, etc.)

**CU-OH-04 (Generación de reportes diarios de pedidos):**
- El sistema debe generar reportes automáticos cada 24 horas (según NFR-OH-03)
- Debe incluir métricas: volumen de pedidos, ticket promedio, tiempos de preparación
- Debe permitir exportación en múltiples formatos (PDF, Excel, CSV)
- Debe incluir desglose por canal de venta (web, app, presencial)

**CU-OH-05 (Registrar entrada y salida individual de empleados):**
- El sistema debe permitir al personal del establecimiento capturar la entrada y salida de cada empleado individualmente (no por equipo ni por turno)
- Debe registrar el evento con empleado, sucursal, fecha y hora (timestamp automático)
- Los registros generados deben quedar disponibles para consulta en solo lectura en Mizu Admin (CU-AD-06)
- Los registros de asistencia deben alimentar el cálculo de horas trabajadas de la nómina en Mizu Admin (CU-AD-05)

### Flujos de Proceso
**Flujo de Pedido Web/App:**
1. Cliente realiza pedido mediante Mizu Experience o Mizu Go
2. Sistema envía pedido a cola de Order Hub en tiempo real
3. Order Hub muestra notificación y coloca pedido en lista pendiente
4. Operador acepta pedido y cambia estado a "preparación"
5. Sistema envía ticket a cocina y notifica al cliente cuando está listo
6. Operador marca pedido como "entregado" al completar la entrega

**Flujo de Pedido Presencial:**
1. Cliente realiza pedido en mostrador o mesa
2. Operador ingresa pedido manualmente en Order Hub
3. Sistema valida items aplicando reglas de negocio (mínimos, etc.)
4. Sistema envía ticket a cocina y muestra número de pedido al cliente
5. Operador actualiza estado conforme avanza la preparación
6. Sistema notifica al cliente cuando el pedido está listo para recogida

**Flujo de Registro de Asistencia:**
1. Operador del establecimiento selecciona a un empleado individual
2. Operador marca entrada o salida del empleado al ingreso o fin de su turno
3. Sistema registra el evento de asistencia con empleado, sucursal, fecha y hora (timestamp automático)
4. Sistema valida que el evento no se duplique y lo almacena como registro de asistencia
5. Los registros quedan disponibles para consulta en solo lectura en Mizu Admin (CU-AD-06) y alimentan el cálculo de horas de la nómina (CU-AD-05)

**Dependencia entre módulos (Order Hub → Admin):** la captura de asistencia por empleado individual en Order Hub (CU-OH-05) es la fuente de datos que alimenta la consulta en solo lectura en Admin (CU-AD-06) y el cálculo de la nómina (CU-AD-05). Mizu Admin no captura asistencias; únicamente consulta los registros generados aquí.

### Priorización
1. **CU-OH-01** (Recepción de pedidos en tiempo real) - Alta prioridad (core del TPV)
2. **CU-OH-02** (Registro manual de pedidos) - Alta prioridad (funcionalidad esencial)
3. **CU-OH-04** (Generación de reportes diarios) - Alta prioridad (requisito NFR-OH-03)
4. **CU-OH-03** (Gestión de estado de pedidos) - Media prioridad (mejora operativa)
5. **CU-OH-05** (Registrar entrada y salida individual de empleados) - Alta prioridad (fuente de datos para CU-AD-06 y CU-AD-05)

### Matriz de Trazabilidad
| CU ID | Descripción | HU ID | Puntos | NFR Relacionados |
|-------|-------------|-------|--------|------------------|
| CU-OH-01 | Recepción de pedidos en tiempo real | HU-OH-01 | 4 pts | NFR-OH-01, NFR-OH-02, NFR-OH-03 |
| CU-OH-02 | Registro manual de pedidos en el establecimiento | HU-OH-02 | 3 pts | NFR-OH-01 |
| CU-OH-03 | Gestión de estado de pedidos (pendiente, preparación, entregado) | HU-OH-03 | 3 pts | NFR-OH-01 |
| CU-OH-04 | Generación de reportes diarios de pedidos | HU-OH-04 | 5 pts | NFR-OH-03 |
| CU-OH-05 | Registrar entrada y salida individual de empleados | HU-OH-05 | 3 pts | NFR-OH-01, NFR-OH-02 |

## 4. Mizu Stock (Escritorio - Inventario)

### Casos de Uso Funcionales
- **CU-ST-01**: Registrar entradas y salidas de inventario
- **CU-ST-02**: Gestión de proveedores
- **CU-ST-03**: Generar alertas de bajo stock
- **CU-ST-04**: Reportes de consumo de ingredientes

### Historias de Usuario Sugeridas
| ID | Descripción | Puntos |
|----|-------------|--------|
| HU-ST-01 | Registrar entrada de inventario | 4 pts |
| HU-ST-02 | Registrar salida de inventario | 4 pts |
| HU-ST-03 | Gestión de proveedores | 5 pts |
| HU-ST-04 | Ver alertas de bajo stock | 3 pts |
| HU-ST-05 | Generar reportes de consumo | 5 pts |

### Criterios de Aceptación
**CU-ST-01 (Registrar entradas y salidas de inventario):**
- El sistema debe permitir registro de recepción de mercancía de proveedores
- Debe validar cantidades recibidas contra órdenes de compra
- Debe registrar lotes, fechas de vencimiento y números de serie cuando aplique
- Debe actualizar automáticamente niveles de stock y generar asientos contables

**CU-ST-02 (Gestión de proveedores):**
- El sistema debe mantener maestro de proveedores con información de contacto
- Debe permitir evaluación de desempeño (calidad, puntualidad, precios)
- Debe gestionar catálogo de productos por proveedor con precios y condiciones
- Debe generar automáticamente órdenes de compra basado en niveles mínimos

**CU-ST-03 (Generar alertas de bajo stock):**
- El sistema debe monitorear continuamente los niveles de inventario
- Debe comparar niveles actuales contra puntos de reposición configurables
- Debe generar notificaciones automáticas cuando el stock llega al mínimo
- Debe sugerir cantidades de reposición basado en el consumo histórico y el lead time

**CU-ST-04 (Reportes de consumo de ingredientes):**
- El sistema debe calcular consumo real basado en recetas y producción
- Debe permitir análisis por período (diario, semanal, mensual) y por centro de costo
- Debe identificar variaciones entre consumo teórico y real (mermas, desperdicios)
- Debe generar reportes de tendencia y pronósticos de necesidades futuras

### Flujos de Proceso
**Flujo de Recepción de Inventario:**
1. Proveedor entrega mercancía con documentación (guía, factura)
2. Operador verifica cantidad recibida contra orden de compra
3. Operador registra entrada en Mizu Stock especificando proveedor y documentos
4. Sistema valida calidad y registra lotes/vencimientos cuando aplicable
5. Sistema actualiza inventario y genera asiento de ingreso
6. Operador archiva documentación y notifica cuentas por pagar

**Flujo de Consumo en Producción:**
1. Cocina solicita ingredientes para preparación de platos
2. Operador autoriza salida de inventario especificando receta y cantidad
3. Sistema descarta cantidades de inventario basado en distinta base
4. Sistema registra consumo y actualiza niveles en tiempo real
5. Sistema alerta si consumo supera estándares o provoca stock bajo

**Flujo de Generación de Alertas de Bajo Stock:**
1. Sistema monitorea niveles de inventario en tiempo real
2. Cuando stock llega al punto de reposición, sistema evalúa necesidad
3. Sistema considera lead time del proveedor y consumo proyectado
4. Sistema genera alerta con sugerencia de orden de compra
5. Operador revisa y aprueba orden o ajusta parámetros según sea necesario

### Priorización
1. **CU-ST-01** (Registrar entradas y salidas de inventario) - Alta prioridad (funcionalidad básica)
2. **CU-ST-02** (Gestión de proveedores) - Alta prioridad (cadena de suministro crítica)
3. **CU-ST-04** (Reportes de consumo de ingredientes) - Alta prioridad (control de costos esencial)
4. **CU-ST-03** (Generar alertas de bajo stock) - Media prioridad (mejora preventiva)

### Matriz de Trazabilidad
| CU ID | Descripción | HU ID | Puntos | NFR Relacionados |
|-------|-------------|-------|--------|------------------|
| CU-ST-01 | Registrar entradas y salidas de inventario | HU-ST-01 | 4 pts | NFR-ST-01, NFR-ST-02, NFR-ST-03 |
| CU-ST-02 | Gestión de proveedores | HU-ST-02 | 5 pts | NFR-ST-01, NFR-ST-02 |
| CU-ST-03 | Generar alertas de bajo stock | HU-ST-03 | 3 pts | NFR-ST-03 |
| CU-ST-04 | Reportes de consumo de ingredientes | HU-ST-04 | 5 pts | NFR-ST-01 |

## 5. Mizu Admin (Escritorio - Gerencia)

### Casos de Uso Funcionales
- **CU-AD-01**: Registrar ingresos y egresos
- **CU-AD-02**: Generar reportes contables (estados financieros y KPIs contables)
- **CU-AD-03**: Control y gestión de usuarios administradores
- **CU-AD-04**: Exportación de datos a Excel/PDF
- **CU-AD-05**: Generar nómina de empleados (cálculo por período de horas, devengados y deducciones; neto a pagar; registro del pago)
- **CU-AD-06**: Ver registros de entrada y salida de empleados (consultas y filtros por empleado, sucursal y período; solo lectura; los registros los captura Order Hub con CU-OH-05)
- **CU-AD-07**: Ver gráficas de reportes de ventas y económicos (ventas por período y sucursal, ingresos vs egresos y utilidad)

### Historias de Usuario Sugeridas
| ID | Descripción | Puntos |
|----|-------------|--------|
| HU-AD-01 | Registrar ingreso financiero | 4 pts |
| HU-AD-02 | Generar reportes contables | 5 pts |
| HU-AD-03 | Gestionar usuarios y permisos | 4 pts |
| HU-AD-04 | Exportar datos a formatos estándar | 3 pts |
| HU-AD-05 | Generar nómina de empleados | 5 pts |
| HU-AD-06 | Ver registros de entrada/salida de empleados | 3 pts |
| HU-AD-07 | Ver gráficas de reportes de ventas y económicos | 5 pts |

### Criterios de Aceptación
**CU-AD-01 (Registrar ingresos y egresos):**
- El sistema debe permitir registro manual de transacciones financieras
- Debe validar contraasientos y mantener equilibrio contable
- Debe clasificar ingresos y egresos según plan de cuentas establecido
- Debe generar automáticamente asientos desde otros módulos (ventas, inventario, etc.)

**CU-AD-02 (Generar reportes contables):**
- El sistema debe generar estados financieros básicos (balance, resultados, flujo de caja)
- Debe permitir análisis comparativo por período y por centro de responsabilidad
- Debe incluir KPIs contables (liquidez, endeudamiento y rentabilidad)
- Debe permitir drill-down desde resúmenes consolidadas hasta transacciones detalle
- Nota de reencuadre: las gráficas de reportes de ventas y económicos están cubiertas por CU-AD-07; CU-AD-02 se limita a reportes contables (estados financieros y KPIs contables)

**CU-AD-03 (Control y gestión de usuarios administradores):**
- El sistema debe permitir creación, modificación y desactivación de usuarios
- Debe gestionar roles y permisos basado en principio de menor privilegio
- Debe registrar auditoría de acciones críticas (cambios de configuración, accesos sensibles)
- Debe requerir autenticación multifactor para operaciones de alto riesgo

**CU-AD-04 (Exportación de datos a Excel/PDF):**
- El sistema debe permitir exportación de cualquier reporte o lista a formatos estándar
- Debe mantener formato y estructura de datos para facilitar análisis externo
- Debe permitir programación de exportaciones automáticas vía email o carpeta compartida
- Debe incluir metadatos de generación (fecha, hora, usuario, filtros aplicados)

**CU-AD-05 (Generar nómina de empleados):**
- El sistema debe calcular la nómina por período y por empleado a partir de las horas trabajadas (registros de asistencia de Order Hub, CU-OH-05), los devengados y las deducciones
- Debe presentar el neto a pagar por empleado para el período
- Debe registrar el pago realizado (fecha, medio y operador)
- Debe hacer consultable el resultado de la nómina por empleado, sucursal y período

**CU-AD-06 (Ver registros de entrada y salida de empleados):**
- El sistema debe mostrar los registros de entrada/salida generados en Order Hub (CU-OH-05) en modo solo lectura
- Debe permitir filtros por empleado, sucursal y período
- Debe mostrar fecha y hora de cada registro
- No debe permitir modificar ni eliminar registros de asistencia desde Admin (la captura corresponde a Order Hub)

**CU-AD-07 (Ver gráficas de reportes de ventas y económicos):**
- El sistema debe presentar en gráficas las ventas por período y por sucursal
- Debe comparar ingresos y egresos y mostrar la utilidad en gráficas
- Debe permitir selección de período y sucursal para el análisis comparativo
- Debe permitir exportación de las vistas de gráficas mediante la exportación de datos (CU-AD-04)

### Flujos de Proceso
**Flujo de Registro Financiero:**
1. Transacción financiera ocurre (venta, gasto, ajuste, etc.)
2. Sistema captura automáticamente desde módulo origen o requiere ingreso manual
3. Sistema valida contrapartida y clasifica según plan de cuentas
4. Sistema genera asiento contactual y actualiza libros
5. Sistema hace disponible transacción para reportes y conciliación

**Flujo de Generación de Reportes Contables:**
1. Usuario selecciona tipo de reporte y período de análisis
2. Sistema aplica filtros según dimensiones solicitadas (sucursal, producto, etc.)
3. Sistema calcula métricas y genera estados financieros requeridos
4. Sistema aplica formato profesional y incluye gráficos de apoyo
5. Sistema permite vista previa, impresión y exportación en múltiples formatos

**Flujo de Gestión de Usuarios y Permisos:**
1. Administrador solicita creación de nuevo usuario o modificación de existente
2. Sistema valida identidad y verifica necesidad business
3. Sistema asigna rol predefinido o permisos personalizados según responsabilidad
4. Sistema notifica al usuario y configura métodos de autenticación
5. Sistema registra acción en log de auditoría para seguimiento

**Flujo de Generación de Nómina (CU-AD-05):**
1. Usuario selecciona período y sucursal para generar la nómina
2. Sistema recoge las horas trabajadas de cada empleado desde los registros de asistencia de Order Hub (CU-OH-05)
3. Sistema calcula devengados y deducciones por empleado
4. Sistema presenta el neto a pagar por empleado del período
5. Usuario valida el cálculo y registra el pago realizado (fecha, medio y operador)
6. Sistema deja consultable el resultado de la nómina y permite su exportación (CU-AD-04)

**Flujo de Consulta de Asistencia y Gráficas Económicas (CU-AD-06 / CU-AD-07):**
1. Usuario elige el tipo de consulta: registros de asistencia (CU-AD-06) o gráficas de ventas/económicos (CU-AD-07)
2. Usuario aplica filtros por empleado (CU-AD-06), sucursal y período
3. Para CU-AD-06: el sistema muestra los registros de entrada/salida generados en Order Hub (CU-OH-05) en solo lectura
4. Para CU-AD-07: el sistema muestra gráficas de ventas por período y sucursal, ingresos vs egresos y utilidad
5. El sistema permite exportar las vistas generadas (CU-AD-04)

**Dependencia entre módulos (Order Hub → Admin):** la captura de asistencia por empleado individual en Order Hub (CU-OH-05) es la fuente de datos que alimenta la consulta en solo lectura en Admin (CU-AD-06) y el cálculo de horas de la nómina (CU-AD-05). Mizu Admin no captura asistencias; únicamente consulta los registros generados en Order Hub.

### Priorización
1. **CU-AD-02** (Generar reportes contables) - Alta prioridad (toma de decisiones)
2. **CU-AD-01** (Registrar ingresos y egresos) - Alta prioridad (integridad contable)
3. **CU-AD-03** (Control y gestión de usuarios administradores) - Alta prioridad (seguridad)
4. **CU-AD-04** (Exportación de datos a Excel/PDF) - Media prioridad (interoperabilidad)
5. **CU-AD-05** (Generar nómina de empleados) - Alta prioridad (gestión financiera y de personal)
6. **CU-AD-06** (Ver registros de entrada y salida de empleados) - Alta prioridad (dato de solo lectura que alimenta CU-AD-05)
7. **CU-AD-07** (Ver gráficas de reportes de ventas y económicos) - Alta prioridad (toma de decisiones económicas)

### Matriz de Trazabilidad
| CU ID | Descripción | HU ID | Puntos | NFR Relacionados |
|-------|-------------|-------|--------|------------------|
| CU-AD-01 | Registrar ingresos y egresos | HU-AD-01 | 4 pts | NFR-AD-01, NFR-AD-02 |
| CU-AD-02 | Generar reportes contables (estados financieros y KPIs contables) | HU-AD-02 | 5 pts | NFR-AD-01, NFR-AD-03 |
| CU-AD-03 | Control y gestión de usuarios administradores | HU-AD-03 | 4 pts | NFR-AD-01 |
| CU-AD-04 | Exportación de datos a Excel/PDF | HU-AD-04 | 3 pts | NFR-AD-01 |
| CU-AD-05 | Generar nómina de empleados | HU-AD-05 | 5 pts | NFR-AD-01, NFR-AD-02 |
| CU-AD-06 | Ver registros de entrada y salida de empleados | HU-AD-06 | 3 pts | NFR-AD-03 |
| CU-AD-07 | Ver gráficas de reportes de ventas y económicos | HU-AD-07 | 5 pts | NFR-AD-01, NFR-AD-03 |

## Preguntas Abiertas y Decisiones Pendientes
- ¿Cómo manejaremos los casos de uso que requieren funcionalidad offline temporal en caso de pérdida de conectividad?
- ¿Qué nivel de detalle deberíamos incluir en los flujos de proceso para balancear claridad con mantenibilidad del documento?
- ¿Deberíamos incluir métricas de éxito específicas para cada historia de usuario más allá de los puntos de esfuerzo estimados?
- ¿Cómo abordaremos los requisitos de accesibilidad (WCAG 2.1 AA) en el desarrollo de las interfaces web y móviles?

## Conclusión
Este análisis funcional proporciona una base sólida para las fases subsiguientes de diseño e implementación. Cada caso de uso ha sido trazado explícitamente a su ID original en `Especificacion_IEEE830.md`, cumpliendo con el primer criterio EARS especificado en la spec de organización del análisis.

Los análisis por subproyecto incluyen:
- Descripción detallada de casos de uso funcionales
- Historias de usuario sugeridas con estimación de esfuerzo
- Criterios de aceptación claros y verificables
- Flujos de proceso principales
- Priorización basada en valor de negocio y dependencias
- Matrices de trazabilidad que vinculan el análisis con los requisitos fuente

Este documento está listo para revisión y sirve como entrada para el análisis no funcional, modelos de datos, contratos de API, arquitectura técnica y diseño UI/UX que seguirán en los subsiguientes documentos de analysis.