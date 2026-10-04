# Análisis de Contratos de API del Proyecto Mizu Ecosystem

## Introducción
Este documento presenta el análisis detallado de los contratos de API requeridos para el ecosistema Mizu, incluyendo definiciones de endpoints API: métodos HTTP, códigos de estado, esquemas de request/response (JSON), versionamiento, estrategias de autenticación y manejo de errores. Establece un vínculo verificable con los requisitos funcionales y no funcionales especificados en `Especificacion_IEEE830.md`, los modelos de datos definidos en `03_modelos_de_datos.md` y alineado con los principios de la `CONSTITUTION.md`.

## Alcance
El análisis de contratos de API cubre:
1. Principios de diseño de API RESTful
2. Estrategias de versionamiento
3. Autenticación y autorización
4. Definición de endpoints por dominio de negocio
5. Esquemas de request y response (JSON)
6. Códigos de estado HTTP y manejo de errores
7. Consideraciones de rendimiento y seguridad
8. Documentación y prueba de APIs

## Metodología
Para cada entidad de negocio identificada en los modelos de datos, se han definido:
- Operaciones CRUD básicas cuando aplique
- Operaciones específicas de negocio basadas en casos de uso
- Endpoints compuestos y agregaciones cuando necesario
- Esquemas de request y response detallados
- Códigos de estado apropiados para cada escenario
- Requisitos de autenticación y autorización
- Estrategias de manejo de errores consistentes
- Consideraciones de versionamiento y compatibilidad

## 1. Principios de Diseño de API

### 1.1. Enfoque RESTful
- Recursos representados como URLs sustantivas (nombres en plural)
- Operaciones realizadas mediante métodos HTTP estándar (GET, POST, PUT, PATCH, DELETE)
- Estados representados mediante códigos de estado HTTP
- Comunicación utilizando JSON como formato de intercambio primario
- Uso apropiado de encabezados HTTP para metadata y control

### 1.2. Convenciones de Nombrado
- Recursos en plural: `/users`, `/orders`, `/products`
- Recursos específicos: `/users/{userId}`, `/orders/{orderId}`
- Operaciones de colección: GET `/users` (listar), POST `/users` (crear)
- Operaciones de instancia: GET `/users/{userId}` (obtener), PUT/PATCH `/users/{userId}` (actualizar), DELETE `/users/{userId}` (eliminar)
- Sub-recursos para relaciones: `/orders/{orderId}/details`, `/users/{userId}/orders`
- Operaciones de acción: POST `/orders/{orderId}/cancel`, POST `/payments/{paymentId}/retry`

### 1.3. Versionamiento
- Estrategia: Versionamiento por URL (más explícito y caché-amigable)
- Formato: `/api/v1/{recurso}` para versión estable actual
- Política de compatibilidad: Mantener hacia atrás durante al menos 2 versiones
- Deprecación: Anunciar con 6 meses de antelación, proporcionar headers de warning
- Sunset: Eliminar versiones antiguas siguiendo proceso anunciado

### 1.4. Formato de Respuesta
**Respuesta Éxito (2xx):**
```json
{
  "success": true,
  "data": { /* contenido específico */ },
  "metadata": {
    "timestamp": "ISO 8601 timestamp",
    "requestId": "uuid único para trazabilidad",
    "version": "api version"
  }
}
```

**Respuesta de Lista con Paginação:**
```json
{
  "success": true,
  "data": {
    "items": [ /* array de elementos */ ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "totalItems": 150,
      "totalPages": 8,
      "hasNext": true,
      "hasPrev": false
    }
  },
  "metadata": { /* como arriba */ }
}
```

**Respuesta Error (4xx, 5xx):**
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE_STRING",
    "message": "Mensaje legible por humanos",
    "details": { /* información técnica adicional cuando aplica */ }
  },
  "metadata": {
    "timestamp": "ISO 8601 timestamp",
    "requestId": "uuid único para trazabilidad",
    "version": "api version"
  }
}
```

### 1.5. Encabezados HTTP Estándar
- `Content-Type: application/json`
- `Accept: application/json`
- `Authorization: Bearer <jwt-token>` o `Basic <credentials>`
- `X-Request-ID: <uuid>` para trazabilidad
- `X-API-Version: v1`
- `RateLimit-Limit: <número>` (cuando aplica límite de tasa)
- `RateLimit-Remaining: <número>` (cuando aplica límite de tasa)
- `RateLimit-Reset: <timestamp>` (cuando aplica límite de tasa)

## 2. Autenticación y Autorización

### 2.1. Estrategia de Autenticación
- **Primary**: JWT (JSON Web Tokens) con refresh tokens
- **Fallback**: API Keys para integraciones de sistema a sistema
- **Expiración**: Access tokens: 15 minutos, Refresh tokens: 7 días
- **Almacenamiento**: 
  - Frontend: Memoria de acceso (no localStorage/sessionStorage por [Principio 18])
  - Backend: Blacklist de tokens revocados + whitelist de refresh tokens válidos
- **Transmisión**: Siempre sobre HTTPS, nunca en query parameters o headers no seguros

### 2.2. Flujo de Autenticación
1. Usuario envía credenciales (email/password) a `/api/v1/auth/login`
2. Sistema valida credenciales y genera:
   - Access token JWT (válido 15 min)
   - Refresh token JWT (válido 7 días, almacenado seguro)
3. Cliente almacena access token en memoria, refresh token en almacenamiento seguro
4. Para cada request: `Authorization: Bearer <access_token>`
5. Cuando access token expira (401): usar refresh token en `/api/v1/auth/refresh`
6. Sistema valida refresh token y devuelve nuevo access token
7. Logout: eliminar tokens del cliente y agregar access token a blacklist

### 2.3. Estrategia de Autorización (RBAC - Role Based Access Control)
- Roles definidos: `cliente`, `empleado_cajero`, `empleado_mesero`, `empleado_cocina`, `gerente`, `administrador`
- Los 6 roles anteriores constituyen la **línea base V1.0 del RBAC** (decisión D1 de la spec 003): son las **semillas del catálogo `Rol`** de `03_modelos_de_datos.md` (entidad `Rol` §1.3 + junction `Usuario_Rol` §1.4); roles futuros se añaden al catálogo sin alterar los contratos
- Permisos granulares por recurso y acción (leer, crear, actualizar, eliminar)
- Herencia de permisos: roles más específicos heredan de roles generales
- Verificación en middleware: verificar rol y permisos antes de ejecutar endpoint
- Superusuario: rol `administrador` con acceso completo (usar con restricciones)

### 2.4. Endpoints de Autenticación
```
POST /api/v1/auth/login
  - Body: { email: string, password: string }
  - Success (200): { access_token: string, refresh_token: string, user: object }
  - Errors: 400 (bad request), 401 (unauthorized), 429 (rate limit)

POST /api/v1/auth/refresh
  - Body: { refresh_token: string }
  - Success (200): { access_token: string, refresh_token: string }
  - Errors: 400 (bad request), 401 (unauthorized), 429 (rate limit)

POST /api/v1/auth/logout
  - Body: { refresh_token: string } 
  - Success (200): { success: true }
  - Errors: 400 (bad request), 401 (unauthorized), 429 (rate limit)

POST /api/v1/auth/register  // solo para auto-registro de clientes
  - Body: { email: string, password: string, nombre: string, telefono: string }
  - Success (201): { user: object (sin password) }
  - Errors: 400 (bad request), 409 (conflict - email exists), 429 (rate limit)
```

## 3. Definición de Endpoints por Dominio de Negocio

### 3.1. Gestión de Usuarios (Users)
```
GET    /api/v1/users                              // Listar usuarios (paginado, filtros)
POST   /api/v1/users                              // Crear nuevo usuario (admin only)
GET    /api/v1/users/{userId}                     // Obtener usuario específico
PUT    /api/v1/users/{userId}                     // Actualizar usuario completo
PATCH  /api/v1/users/{userId}                     // Actualización parcial de usuario
DELETE /api/v1/users/{userId}                     // Eliminar usuario (soft delete)

GET    /api/v1/users/{userId}/orders              // Historial de pedidos del usuario
GET    /api/v1/users/{userId}/reservations        // Historial de reservas del usuario
```

**Esquemas:**
- **UserCreateRequest**: { nombre, email, password, telefono, tipo? }
- **UserUpdateRequest**: { nombre, telefono, activo?, preferencias? }
- **UserResponse**: { id, nombre, email, telefono, tipo, fecha_registro, activo, ultimo_acceso, preferencias }

### 3.2. Gestión de Credenciales (Auth - ya cubierto arriba)
*Los endpoints de autenticación se definieron en la sección 2.4*

### 3.3. Gestión de Sucursales (Branches)
```
GET    /api/v1/branches                           // Listar sucursales (activas por defecto)
POST   /api/v1/branches                           // Crear nueva sucursal
GET    /api/v1/branches/{branchId}                // Obtener sucursal específica
PUT    /api/v1/branches/{branchId}                // Actualizar sucursal completa
PATCH  /api/v1/branches/{branchId}                // Actualización parcial de sucursal
DELETE /api/v1/branches/{branchId}                // Eliminar sucursal (soft delete)

GET    /api/v1/branches/{branchId}/stats          // Estadísticas de la sucursal
GET    /api/v1/branches/{branchId}/employees      // Empleados asignados a la sucursal
GET    /api/v1/branches/{branchId}/inventory      // Inventario de la sucursal (resumen)
```

**Esquemas:**
- **BranchCreateRequest**: { nombre, direccion, ciudad, estado, codigo_postal, pais, telefono, email, horario_apertura, horario_cierre, zona_horaria, capacidad_maxima, coordenadas }
- **BranchUpdateRequest**: { nombre, direccion, ciudad, estado, codigo_postal, pais, telefono, email, horario_apertura, horario_cierre, zona_horaria, capacidad_maxima, activo, coordenadas }
- **BranchResponse**: { id, nombre, direccion, ciudad, estado, codigo_postal, pais, telefono, email, horario_apertura, horario_cierre, zona_horaria, activo, fecha_alta, fecha_baja, capacidad_maxima, mesas_totales, coordenadas }

### 3.4. Gestión de Mesas (Tables)
```
GET    /api/v1/branches/{branchId}/tables         // Listar mesas de una sucursal
POST   /api/v1/branches/{branchId}/tables         // Crear nueva mesa en sucursal
GET    /api/v1/branches/{branchId}/tables/{tableId} // Obtener mesa específica
PUT    /api/v1/branches/{branchId}/tables/{tableId} // Actualizar mesa completa
PATCH  /api/v1/branches/{branchId}/tables/{tableId} // Actualización parcial de mesa
DELETE /api/v1/branches/{branchId}/tables/{tableId} // Eliminar mesa (soft delete)

GET    /api/v1/branches/{branchId}/tables/{tableId}/reservations // Reservas futuras para esta mesa
GET    /api/v1/branches/{branchId}/tables/{tableId}/ordersHist   // Historial de órdenes en esta mesa
```

**Esquemas:**
- **TableCreateRequest**: { numero, capacidad, ubicacion, caracteristicasEspeciales? }
- **TableUpdateRequest**: { numero, capacidad, ubicacion, estado?, caracteristicasEspeciales? }
- **TableResponse**: { id, branchId, numero, capacidad, ubicacion, estado, fechaUltimoActualizacion, caracteristicasEspeciales }

### 3.5. Gestión de Categorías de Producto (Categories)
```
GET    /api/v1/categories                         // Listar categorías activas
POST   /api/v1/categories                         // Crear nueva categoría
GET    /api/v1/categories/{categoryId}            // Obtener categoría específica
PUT    /api/v1/categories/{categoryId}            // Actualizar categoría completa
PATCH  /api/v1/categories/{categoryId}            // Actualización parcial de categoría
DELETE /api/v1/categories/{categoryId}            // Eliminar categoría (soft delete si no tiene productos)

GET    /api/v1/categories/{categoryId}/products   // Productos de esta categoría
```

**Esquemas:**
- **CategoryCreateRequest**: { nombre, descripcion?, imagenUrl?, ordenDisplay }
- **CategoryUpdateRequest**: { nombre?, descripcion?, imagenUrl?, ordenDisplay?, activo? }
- **CategoryResponse**: { id, nombre, descripcion, imagenUrl, ordenDisplay, activo, fechaCreacion, fechaActualizacion }

### 3.6. Gestión de Productos (Products)
```
GET    /api/v1/products                           // Listar productos (paginado, filtros, búsqueda)
POST   /api/v1/products                           // Crear nuevo producto
GET    /api/v1/products/{productId}               // Obtener producto específico
PUT    /api/v1/products/{productId}               // Actualizar producto completo
PATCH  /api/v1/products/{productId}               // Actualización parcial de producto
DELETE /api/v1/products/{productId}               // Eliminar producto (soft delete)

GET    /api/v1/products/search?q={texto}          // Búsqueda de texto en productos
GET    /api/v1/products/category/{categoryId}     // Productos por categoría
GET    /api/v1/products/available                 // Solo productos actualmente disponibles
GET    /api/v1/products/{productId}/inventory     // Disponibilidad en inventario por sucursal
```

**Esquemas:**
- **ProductCreateRequest**: { categoryId, nombre, descripcion, precioBase, costoPreparacion, tiempoPreparacionEstandar, disponible?, fechaDisponibleDesde?, fechaDisponibleHasta?, calorias?, esVegetariano?, esVegano?, contieneGluten?, contieneLacteos?, contieneNueces?, imagenUrl, imagenesAdicionales?, ingredientes?, preparacionPasos? }
- **ProductUpdateRequest**: { nombre?, descripcion?, precioBase?, costoPreparacion?, tiempoPreparacionEstandar?, disponible?, fechaDisponibleDesde?, fechaDisponibleHasta?, calorias?, esVegetariano?, esVegano?, contieneGluten?, contieneLacteos?, contieneNueces?, imagenUrl?, imagenesAdicionales?, ingredientes?, preparacionPasos?, activo? }
- **ProductResponse**: { id, categoryId, nombre, descripcion, precioBase, costoPreparacion, tiempoPreparacionEstandar, disponible, fechaDisponibleDesde, fechaDisponibleHasta, calorias, esVegetariano, esVegano, contieneGluten, contieneLacteos, contieneNueces, imagenUrl, imagenesAdicionales, ingredientes, preparacionPasos, activo, fechaCreacion, fechaActualizacion }

### 3.7. Gestión de Pedidos (Orders)
```
GET    /api/v1/orders                             // Listar pedidos (paginado, filtros amplio)
POST   /api/v1/orders                             // Crear nuevo pedido
GET    /api/v1/orders/{orderId}                   // Obtener pedido específico con detalles
PUT    /api/v1/orders/{orderId}                   // Actualizar pedido completo (raramente usado)
PATCH  /api/v1/orders/{orderId}                   // Actualización parcial de pedido (estado, notas, etc.)
DELETE /api/v1/orders/{orderId}                   // Eliminar pedido (soft delete, solo si pendiente)

GET    /api/v1/orders/{orderId}/details           // Obtener solo los detalles del pedido
GET    /api/v1/orders/{orderId}/payments          // Obtener pagos asociados al pedido
GET    /api/v1/orders/{orderId}/status            // Obtener estado actual y historial de cambios
POST   /api/v1/orders/{orderId}/cancel            // Cancelar pedido (si aplicable)
POST   /api/v1/orders/{orderId}/retry-payment     // Reintentar pago fallido
GET    /api/v1/orders/{orderId}/timeline          // Línea de tiempo detallada del pedido

GET    /api/v1/users/{userId}/orders              // Pedidos de un usuario específico
GET    /api/v1/branches/{branchId}/orders         // Pedidos de una sucursal específica
GET    /api/v1/branches/{branchId}/orders/stats   // Estadísticas de pedidos por sucursal
```

**Esquemas:**
- **OrderCreateRequest**: { branchId, tableId?, customerId?, empleadoId?, tipo, items: [{ productId: string, cantidad: number, notasEspeciales? }], origen?, direccionEntrega?, instruccionesEntrega?, notasCliente?, propina? }
- **OrderUpdateRequest**: { estado?, notasCliente?, notasInterno?, origen?, direccionEntrega?, instruccionesEntrega?, propina? }
- **OrderResponse**: { id, numero, branchId, tableId, customerId, empleadoId, tipo, estado, fechaHora, fechaHoraConfirmado, fechaHoraPreparacionInicio, fechaHoraListo, fechaHoraEntregado, subtotal, descuento, impuestos, total, metodoPago, referenciaPago, propina, notasCliente, notasInterno, origen, direccionEntrega, instruccionesEntrega, activo, fechaCreacion, fechaActualizacion }
- **OrderDetailResponse**: { id, productId, cantidad, precioUnitario, subtotal, notasEspeciales, estadoPreparacion, tiempoInicioPreparacion, tiempoFinPreparacion }
- **OrderItemRequest**: { productId: string, cantidad: number, notasEspeciales? }

### 3.8. Gestión de Pagos (Payments)
```
GET    /api/v1/payments                           // Listar pagos (paginado, filtros)
POST   /api/v1/payments                           // Procesar nuevo pago
GET    /api/v1/payments/{paymentId}               // Obtener pago específico
PUT    /api/v1/payments/{paymentId}               // Actualizar pago (raramente usado)
PATCH  /api/v1/payments/{paymentId}               // Actualización parcial de pago (estado, etc.)
DELETE /api/v1/payments/{paymentId}               // Eliminar pago (soft delete, solo si pendiente)

GET    /api/v1/orders/{orderId}/payments          // Pagos de un pedido específico
POST   /api/v1/payments/{paymentId}/retry         // Reintentar pago fallido
POST   /api/v1/payments/{paymentId}/refund        // Reembolsar pago (parcial o total)
```

**Esquemas:**
- **PaymentCreateRequest**: { orderId, monto, metodo, referenciaExterna?, datosAdicionales? }
- **PaymentUpdateRequest**: { estado?, fechaHoraProcesado?, fechaHoraRevertido?, datosAdicionales? }
- **PaymentResponse**: { id, orderId, monto, metodo, referenciaExterna, estado, fechaHora, fechaHoraProcesado, fechaHoraRevertido, datosAdicionales, activo, fechaCreacion, fechaActualizacion }

### 3.9. Gestión de Reservas (Reservations)
```
GET    /api/v1/reservations                       // Listar reservas (paginado, filtros)
POST   /api/v1/reservations                       // Crear nueva reserva
GET    /api/v1/reservations/{reservationId}       // Obtener reserva específica
PUT    /api/v1/reservations/{reservationId}      // Actualizar reserva completa
PATCH  /api/v1/reservations/{reservationId}      // Actualización parcial de reserva
DELETE /api/v1/reservations/{reservationId}      // Eliminar reserva (soft delete)

GET    /api/v1/users/{userId}/reservations        // Reservas de un usuario específico
GET    /api/v1/branches/{branchId}/reservations   // Reservas de una sucursal específica
GET    /api/v1/branches/{branchId}/reservations/stats // Estadísticas de reservas por sucursal
POST   /api/v1/reservations/{reservationId}/confirm // Confirmar reserva pendiente
POST   /api/v1/reservations/{reservationId}/cancel // Cancelar reserva
```

**Esquemas:**
- **ReservationCreateRequest**: { branchId, tableId, customerId, fechaHoraInicio, fechaHoraFin, numeroPersonas, contactoNombre, contactoTelefono, contactoEmail, ocasionesEspeciales?, solicitudesEspeciales? }
- **ReservationUpdateRequest**: { fechaHoraInicio?, fechaHoraFin?, numeroPersonas?, contactoNombre?, contactoTelefono?, contactoEmail?, ocasionesEspeciales?, solicitudesEspeciales?, estado? }
- **ReservationResponse**: { id, branchId, tableId, customerId, fechaHoraReserva, fechaHoraInicio, fechaHoraFin, numeroPersonas, contactoNombre, contactoTelefono, contactoEmail, ocasionesEspeciales, solicitudesEspeciales, codigoConfirmacion, estado, activo, fechaCreacion, fechaActualizacion }

### 3.10. Gestión de Empleados (Employees)
```
GET    /api/v1/employees                          // Listar empleados (paginado, filtros)
POST   /api/v1/employees                          // Crear nuevo empleado
GET    /api/v1/employees/{employeeId}             // Obtener empleado específico
PUT    /api/v1/employees/{employeeId}             // Actualizar empleado completo
PATCH  /api/v1/employees/{employeeId}             // Actualización parcial de empleado
DELETE /api/v1/employees/{employeeId}             // Eliminar empleado (soft delete)

GET    /api/v1/employees/{employeeId}/payroll     // Historial de nómina del empleado
GET    /api/v1/branches/{branchId}/employees      // Empleados de una sucursal específica
GET    /api/v1/users/{userId}/employee            // Obtener empleado asociado a un usuario (si aplica)
```

**Esquemas:**
- **EmployeeCreateRequest**: { userId, branchId, numeroEmpleado, tipoEmpleado, fechaIngreso, salarioBase, horarioTrabajo?, habilidades?, contactoEmergenciaNombre?, contactoEmergenciaTelefono? }
- **EmployeeUpdateRequest**: { branchId?, numeroEmpleado?, tipoEmpleado?, fechaIngreso?, fechaEgreso?, salarioBase?, horarioTrabajo?, habilidades?, contactoEmergenciaNombre?, contactoEmergenciaTelefono?, activo? }
- **EmployeeResponse**: { id, userId, branchId, numeroEmpleado, tipoEmpleado, fechaIngreso, fechaEgreso, salarioBase, horarioTrabajo, habilidades, contactoEmergenciaNombre, contactoEmergenciaTelefono, activo, fechaCreacion, fechaActualizacion }

### 3.11. Gestión de Nómina (Payroll)
```
GET    /api/v1/payroll                            // Listar registros de nómina (paginado, filtros)
POST   /api/v1/payroll                            // Procesar nueva nómina
GET    /api/v1/payroll/{payrollId}               // Obtener registro de nómina específico
PUT    /api/v1/payroll/{payrollId}               // Actualizar nómina completa (raramente)
PATCH  /api/v1/payroll/{payrollId}               // Actualización parcial de nómina (correcciones)
DELETE /api/v1/payroll/{payrollId}               // Eliminar nómina (soft delete, solo si no procesada)

GET    /api/v1/employees/{employeeId}/payroll     // Nómina de un empleado específico
GET    /api/v1/payroll/stats                     // Estadísticas generales de nómina
```

**Esquemas** (sobre la entidad `Nomina` de `03_modelos_de_datos.md` §2.5: lo automatizable —horas, devengados, deducciones, neto— se calcula en la app antes de guardar):
- **PayrollCreateRequest**: { employeeId, periodoInicio, periodoFin, horasNormales, horasExtra, bonificaciones?, descuentos? }
- **PayrollUpdateRequest**: { periodoInicio?, periodoFin?, horasNormales?, horasExtra?, bonificaciones?, descuentos?, estado?, fechaPago?, metodoPago?, observaciones? }
- **PayrollResponse**: { id, employeeId, periodoInicio, periodoFin, horasNormales, horasExtra, bonificaciones, descuentos, totalDevengado, totalDeducciones, netoAPagar, estado ('pendiente'|'aprobada'|'pagada'), fechaPago?, metodoPago? ('transferencia'|'cheque'|'efectivo'), observaciones? }

### 3.12. Gestión de Inventario y Stock (por ingrediente)

Inventario modelado **por ingrediente** sobre las entidades de `03_modelos_de_datos.md` §4: `Ingrediente` (§4.1), `MovimientoInventario` (§4.2), `ProveedorIngrediente` (§4.5), `OrdenCompra` (§4.6) y `OrdenCompraDetalle` (§4.7).

```
GET    /api/v1/branches/{branchId}/ingredients          // Inventario por sucursal (listado de ingredientes y stock) (CU-ST-01)
GET    /api/v1/ingredients/{ingredientId}/movements    // Historial de movimientos de inventario (entradas/salidas) (CU-ST-01/03)
POST   /api/v1/ingredients/{ingredientId}/movements    // Registrar entrada o salida de inventario (CU-ST-01)
GET    /api/v1/purchase-orders                          // Listar órdenes de compra (paginado, filtros: proveedorId, branchId, estado, fechas) (CU-ST-02)
POST   /api/v1/purchase-orders                          // Crear orden de compra con detalle (CU-ST-02)
GET    /api/v1/purchase-orders/{orderId}                 // Obtener orden de compra con detalle (CU-ST-02/03)
GET    /api/v1/branches/{branchId}/ingredients/low-stock // Alertas de stock bajo y vencimiento (stockActual <= stockMinimo) (CU-ST-04)
```

**Esquemas:**
- **IngredientResponse** (`Ingrediente`, 03 §4.1): { id, branchId, nombre, unidad, stockActual, stockMinimo, lote?, numeroSerie?, fechaVencimiento? }
- **MovementRequest** (`MovimientoInventario`, 03 §4.2): { ingredienteId, tipo ('entrada'|'salida'), cantidad, fecha?, referencia?, registradoPor? }
- **MovementResponse**: { id, ingredienteId, tipo ('entrada'|'salida'), cantidad, fecha, referencia?, registradoPor }
- **PurchaseOrderResponse** (`OrdenCompra` + `OrdenCompraDetalle`, 03 §4.6/4.7): { id, proveedorId, branchId, fecha, estado ('pendiente'|'confirmada'|'recibida'|'cancelada'), total, referencia?, items: [{ ingredienteId, cantidad, precioUnitario }] }
- **ConsumptionReportResponse**: { periodoInicio, periodoFin, totalConsumidoPorIngrediente, variacionesTeoricoReal?, tendencias?, pronosticos? }

**Trazabilidad CU-ST-01…04:** `Ingrediente`/`MovimientoInventario` → inventario por sucursal y registro de entradas/salidas (CU-ST-01); `OrdenCompra`/`OrdenCompraDetalle` con el catálogo de suministro `ProveedorIngrediente` → proveedores y pedidos de insumos (CU-ST-02); consumo y costeo por período (CU-ST-03); alertas de bajo stock y vencimiento (CU-ST-04).
**Fuera de alcance (fase 2, 04-22):** los endpoints de `ProductoIngrediente` (recetas/costo) y de `Proveedor` quedan pendientes: soporte pendiente — fase 2 (spec 004).

### 3.13. Gestión de Reportes y Analítica
```
GET    /api/v1/reports/sales/daily               // Reporte de ventas diarias
GET    /api/v1/reports/sales/monthly             // Reporte de ventas mensuales
GET    /api/v1/reports/inventory/valuation       // Valoración de inventario actual
GET    /api/v1/reports/labor/efficiency          // Eficiencia laboral
GET    /api/v1/reports/performance/kpis          // Indicadores clave de desempeño
POST   /api/v1/reports/custom                    // Generar reporte personalizado
GET    /api/v1/analytics/user-behavior           // Análisis de comportamiento de usuario
GET    /api/v1/analytics/menu-performance        // Rendimiento de productos del menú
GET    /api/v1/analytics/peak-hours              // Análisis de horas pico
```

**Nota**: Los endpoints de reporte suelen ser más flexibles y pueden aceptar parámetros de filtrado complejo

## 4. Esquemas de Request y Response Detallados

### 4.1. Tipos de Datos Comunes
- **UUID**: Formato estándar UUID v7 (opaco y ordenado temporalmente; generado en la capa de aplicación o con la extensión `pg_uuidv7`, PostgreSQL 15+; no mezclar con BIGINT) — ej. "01994f8a-2e1b-7c45-8d3f-6a2b9c1d4e5f" (convención 03 §6 / A.2)
- **Timestamp**: ISO 8601 (ej. "2026-10-03T14:30:00Z")
- **Date**: Solo fecha ISO 8601 (ej. "2026-10-03")
- **Time**: Solo hora ISO 8601 (ej. "14:30:00")
- **Decimal**: Número con precisión matemática (ej. 15.99)
- **Integer**: Número entero (ej. 42)
- **Boolean**: verdadero/falso
- **String**: Texto UTF-8
- **Array**: Lista ordenada de valores
- **Object**: Pares clave-valor JSON

### 4.2. Manejo de Parámetros de Consulta (Query Parameters)
- **Paginación**: `page=1&limit=20` (predeterminados: page=1, limit=20, máximo limit=100)
- **Filtrado**: `estado=activo&tipo=cliente&fechaDesde=2026-10-01&fechaHasta=2026-10-31`
- **Ordenamiento**: `sortBy=fecha_creacion&sortOrder=desc` (predeterminado sortOrder=asc)
- **Búsqueda**: `q=texto+a+buscar` (búsqueda de texto libre en campos relevantes)
- **Inclusión de relaciones**: `include=detalles,pagos` (para evitar múltiples llamadas - "eager loading" opcional)
- **Campos específicos**: `fields=id,nombre,email` (para reducir payload cuando no se necesita todo)

### 4.3. Códigos de Estado HTTP
**Éxito (2xx):**
- 200 OK: Respuesta estándar exitoso
- 201 Created: Recurso creado exitosamente (incluir Location header)
- 204 No Content: Éxito pero no hay contenido para retornar (DELETE exitoso)

**Errores de Cliente (4xx):**
- 400 Bad Request: Solicitud malformed o datos inválidos
- 401 Unauthorized: Falta de autenticación o token inválido/expirado
- 403 Forbidden: Autenticado pero sin permisos para el recurso
- 404 Not Found: Recurso no encontrado
- 409 Conflicto: Conflicto con estado actual (ej. email duplicado)
- 422 Unprocessable Entity: Datos sintácticamente correctos pero semánticamente inválidos
- 429 Too Many Requests: Límite de tasa excedido
- 406 Not Acceptable: No se puede producir el formato solicitado

**Errores de Servidor (5xx):**
- 500 Internal Server Error: Error inesperado en el servidor
- 502 Bad Gateway: Error acting as gateway or proxy
- 503 Service Unavailable: Servicio temporalmente no disponible (overload, mantenimiento)
- 504 Gateway Timeout: Timeout actuando como gateway

### 4.4. Manejo de Errores Consisten
Todos los endpoints de error deben seguir el formato estándar definido en sección 1.4:

**Ejemplo de Error 400 (Validación Fallida):**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Los datos proporcionados no son válidos",
    "details": {
      "fieldErrors": [
        { "field": "email", "message": "El email es requerido", "rule": "required" },
        { "field": "password", "message": "La contraseña debe tener al menos 8 caracteres", "rule": "minLength:8" }
      ]
    }
  },
  "metadata": {
    "timestamp": "2026-10-03T14:30:00Z",
    "requestId": "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8",
    "version": "v1"
  }
}
```

**Ejemplo de Error 401 (No Autenticado):**
```json
{
  "success": false,
  "error": {
    "code": "AUTHENTICATION_REQUIRED",
    "message": "Se requiere autenticación para acceder a este recurso",
    "details": {}
  },
  "metadata": {
    "timestamp": "2026-10-03T14:30:00Z",
    "requestId": "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8",
    "version": "v1"
  }
}
```

**Ejemplo de Error 403 (No Autorizado):**
```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "No tienes suficientes permisos para realizar esta acción",
    "details": {
      "requiredRole": "administrador",
      "requiredPermission": "users:delete"
    }
  },
  "metadata": {
    "timestamp": "2026-10-03T14:30:00Z",
    "requestId": "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8",
    "version": "v1"
  }
}
```

**Ejemplo de Error 404 (No Encontrado):**
```json
{
  "success": false,
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "El recurso solicitado no existe",
    "details": {
      "resourceType": "user",
      "resourceId": "0198c7b2-4d1a-7e6f-9a3b-5c6d7e8f9a0b"
    }
  },
  "metadata": {
    "timestamp": "2026-10-03T14:30:00Z",
    "requestId": "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8",
    "version": "v1"
  }
}
```

### 4.5. Límite de Tasa (Rate Limiting)
- **Estrategia**: Token bucket o fixed window counter
- **Valores rectores** (rectores de `AGENTS.md`, decisión D2 de la spec 003):
  - Usuario autenticado (general): **1000 peticiones/hora** (≈ 17 peticiones/minuto)
  - Anónimo (endpoints públicos, por IP): **60 peticiones/minuto**
- **Límites de clase más estrictos** (se conservan):
  - Clase autenticación: 10 intentos/minuto por IP
  - Clase reporte: 10 peticiones/minuto
- **Respuesta 429 debe incluir** (ejemplo sobre el límite rector de usuario autenticado, 1000 peticiones/hora):
  ```json
  {
    "success": false,
    "error": {
      "code": "RATE_LIMIT_EXCEEDED",
      "message": "Se ha excedido el límite de tasa permitido",
      "details": {
        "limit": 1000,
        "unit": "hour",
        "remaining": 0,
        "reset": "2026-10-03T15:00:00Z"
      }
    },
    "metadata": { /* como estándar */ }
  }
  ```
  Además de headers HTTP: `RateLimit-Limit: 1000`, `RateLimit-Remaining: 0`, `RateLimit-Reset: <timestamp>`

## 5. Consideraciones de Seguridad en APIs

### 5.1. Transporte Seguro
- **HTTPS Obligatorio**: Todas las comunicaciones deben ser sobre TLS 1.2+
- **HSTS**: HTTP Strict Transport Security habilitado
- **Certificate Pinning**: Opcional para aplicaciones móviles de alta seguridad
- **DNSSEC**: Recomendado para prevención de ataques de tipo man-in-the-middle

### 5.2. Protección Contra Ataques Comunes
- **SQL Injection**: Uso de prepared statements/ORM, nunca concatenación de strings
- **Cross-Site Scripting (XSS)**: Sanitización de salida, Content Security Policy
- **Cross-Site Request Forgery (CSRF)**: Tokens en formas stateless (JWT lo hace innecesario en APIs puras)
- **Brute Force**: Límites de tasa en autenticación, delays exponenciales, captcha después de varios intentos
- **Directory Traversal**: Validación estricta de paths en uploads de archivos
- **XML External Entity (XXE)**: Deshabilitar procesamiento de DTDs en parsers XML
- **Insecure Deserialization**: Validar y limitar tipos de objetos deserializados

### 5.3. Validación y Sanitización de Entrada
- **Validación en el borde**: Todas las entradas validadas antes de procesamiento
- **Lista blanca de caracteres**: Cuando aplique (ej. solo números para campos de cantidad)
- **Longitud máxima**: Límites razonables en todos los campos de texto
- **Tipos de datos estrictos**: Validar que JSON coincida con tipos esperados
- **Whitelist de URLs**: Para redirecciones y callbacks cuando aplique
- **Sanitización de HTML**: Cuando se acepta contenido rico que será mostrado

### 5.4. Protección de Datos Sensibles
- **Never expose**: contraseñas, tokens completos, datos de tarjetas de pago completos
- **Masking**: Mostrar solo últimos 4 dígitos de tarjetas, emails parcialmente ocultos
- **Hashing**: Almacenar hash, nunca plain text, para contraseñas y datos sensibles
- **Tokenization**: Para datos de pago cuando se requiere almacenamiento temporal
- **Encryption**: AES-256-GCM para datos sensibles en reposo cuando aplica
- **Key Management**: Sistema robusto de manejo de claves de cifrado

### 5.5. Headers de Seguridad
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `X-XSS-Protection: 1; mode=block`
- `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- `Content-Security-Policy: default-src 'self'` (ajustado según necesidades)
- `Referrer-Policy: strict-origin-when-cross-origin`

## 6. Consideraciones de Rendimiento y Escalabilidad

### 6.1. Optimización de Respuestas
- **Paginación**: Implementada en todos los endpoints que pueden devolver múltiples resultados
- **Selección de campos**: Opcional `fields` parameter para reducir payload
- **Compression**: Gzip/Brotli habilitado para respuestas grandes
- **Caching**: Headers apropiados (Cache-Control, ETag, Last-Modified) para datos estáticos o poco cambiantes
- **Lazy Loading**: Cargar relaciones pesadas solo cuando se soliciten explícitamente
- **Batch Operations**: Endpoints para crear/actualizar múltiples elementos cuando sea eficiente

### 6.2. Manejo de Cargas Altas
- **Colas de trabajo**: Operaciones no críticas en tiempo real enviados a colas (RabbitMQ)
- **Procesamiento asíncrono**: Notificaciones, reportes, analítica enviados a workers separados
- **Circuit breaker**: Para dependencias externas que puedan fallar
- **Bulkheads**: Aislamiento de recursos para diferentes tipos de carga
- **Load balancing**: Distribución de carga entre múltiples instancias de servicio
- **Auto-scaling**: Escalado horizontal basado en métricas de carga y latencia

### 6.3. Optimización de Base de Datos
- **Índices adecuados**: Como se especifica en el modelo de datos
- **Consultas eficientes**: Evitar SELECT *, usar JOINs apropiados
- **Límites de resultados**: Siempre limitar consultas que pueden devolver muchos resultados
- **Materialized views**: Para agregaciones complejas que cambian raramente
- **Read réplicas**: Distribuir carga de lectura entre múltiples instancias
- **Caching de consultas**: Resultados frecuentemente usados en Redis o similar

### 6.4. Timeouts y Límites de Recursos
- **Request timeout**: 30 segundos para la mayoría de endpoints
- **Long-running operations**: 300 segundos para operaciones que lo justifiquen (reportes complejos)
- **Database query timeout**: 10 segundos para prevenir consultas que consuman demasiados recursos
- **Payload size limit**: 10 MB máximo para request body (ajustable según endpoint)
- **Header size limit**: 8 KB máximo para headers HTTP
- **Concurrent requests limit**: Configurable por instancia según capacidad de hardware

## 7. Documentación y Prueba de APIs

### 7.1. Documentación Automática
- **OpenAPI/Swagger 3.0**: Definición formal de todos los endpoints
- **Redoc o Swagger UI**: Interface interactiva para explorar y probar APIs
- **Ejemplos concretos**: Para cada request y response body
- **Descripciones detalladas**: Para cada parámetro, campo y código de estado
- **Esquemas de seguridad**: Definiendo requisitos de autenticación por endpoint o grupo
- **Servidor y versiones**: Información clara sobre hosting y versionamiento

### 7.2. Estrategia de Prueba
- **Pruebas unitarias**: Validación de lógica de negocio y transformación de datos
- **Pruebas de integración**: Verificación de endpoints completos con base de datos
- **Pruebas de contrato**: Asegurar que la implementación cumple con la especificación OpenAPI
- **Pruebas de carga**: Validar rendimiento bajo carga esperada y pico
- **Pruebas de seguridad**: Escaneos de vulnerabilidades y pruebas de penetración
- **Pruebas de contrato del consumidor**: Validar que las APIs cumplen con las necesidades de los clientes frontend

### 7.3. Monitoreo en Producción
- **Health checks**: Endpoints ligeros para monitoreo de disponibilidad
- **Métricas de uso**: Conteo de requests por endpoint, códigos de estado, latencia
- **Métricas de negocio**: Volumen de transacciones, tasas de conversión, ingresos
- **Logs estructurados**: Para facilitar análisis y alertas
- **Alertas automáticas**: Para métricas fuera de umbrales aceptables
- **Distributed tracing**: Para seguir requests a través de microservicios (Jaeger, Zipkin)

## 8. Mapeo a Requisitos Funcionales y No Funcionales

### 8.1. Trazabilidad Completa
Cada endpoint y operación puede ser trazado a uno o más requisitos funcionales y no funcionales origen.

### 8.2. Cobertura de Casos de Uso
La cobertura cubre los CUs de la especificación IEEE830 v1.1; los 4 CUs nuevos de la v1.1 quedan: soporte pendiente — fase 2 (spec 004):
- Consultar menú: GET `/api/v1/products?disponible=true&categoryId={categoria}`
- Realizar reserva: POST `/api/v1/reservations`
- Hacer pedido en línea: POST `/api/v1/orders` + POST `/api/v1/payments`
- Generar historial: GET `/api/v1/users/{userId}/orders`
- Registro y autenticación: POST `/api/v1/auth/register` + POST `/api/v1/auth/login`
- Pedidos y reservas desde app: Mismo conjunto de endpoints optimizado para consumo móvil
- Mostrar promociones personalizadas: GET `/api/v1/promotions/activas` + parámetros de segmentación
- Sistema de puntos y recompensas: Endpoints de puntos (por definir en expansión futura)
- Recepción de pedidos en tiempo real: WebSocket endpoints complementarios + polling eficiente
- Registro manual de pedidos: POST `/api/v1/orders` (interfaz optimizada para empleados)
- Gestión de estado de pedidos: PATCH `/api/v1/orders/{orderId}/estado` + timeline
- Generación de reportes diarios: GET `/api/v1/reports/sales/daily` + otros endpoints de reporte
- Registrar entradas y salidas de inventario: POST `/api/v1/ingredients/{ingredientId}/movements`
- Gestión de proveedores: CRUD en `/api/v1/suppliers` + `/api/v1/purchase-orders`
- Generar alertas de bajo stock: GET `/api/v1/branches/{branchId}/ingredients/low-stock`
- Reportes de consumo de ingredientes: GET `/api/v1/ingredients/{ingredientId}/movements` (reportes por período en §3.13)
- Registrar ingresos y egresos: POST `/api/v1/payments` + `/api/v1/payroll` + endpoints contables
- Generar reportes contables (CU-AD-02): GET `/api/v1/reports/contables/*`
- Gráficas de ventas/económicas (CU-AD-07): soporte pendiente — fase 2 (spec 004)
- Control y gestión de usuarios administradores: CRUD en `/api/v1/users` + `/api/v1/roles` + permisos
- Exportación de datos a Excel/PDF: POST `/api/v1/reports/export` con formato especificado
- Generar nómina (CU-AD-05): soporte pendiente — fase 2 (spec 004) — el §3.11 ya define el esquema sobre `Nomina` del 03
- Ver entrada/salida de empleados, solo lectura (CU-AD-06): soporte pendiente — fase 2 (spec 004)
- Registro individual de entrada/salida en Order Hub (CU-OH-05): soporte pendiente — fase 2 (spec 004)

### 8.3. Cumplimiento de Requisitos No Funcionales
Los contratos de API soportan todos los requisitos no funcionales:
- Rendimiento: Paginação, límite de fields, caching apropiado, timeouts razonables
- Seguridad: Autenticación, autorización, validación de entrada, protección de datos sensibles
- Disponibilidad: Diseño sin estado único, reintentos idempotentes, manejo elegante de errores
- Escalabilidad: API sin estado, facilita escalado horizontal detrás de load balancer
- Compatibilidad: Versionamiento claro, headers estándar, códigos de estado consistentes
- Usabilidad: Mensajes de error claros, ejemplos concretos, estructura predecible
- Operatividad: Health checks, métricas, logging estructurado, documentación automática
- Cumplimiento: Trazabilidad completa, capacidad de generar reportes requeridos, retención adecuada

## 9. WebSockets y Comunicación en Tiempo Real (Complementario a REST)
*Nota: Aunque el enfoque principal es RESTful, algunas funcionalidades se benefician de comunicación en tiempo real*

### 9.1. Endpoints WebSocket
```
WS    /api/v1/ws/orders/{branchId}          // Actualizaciones en tiempo real de pedidos para sucursal
WS    /api/v1/ws/inventory/{branchId}       // Actualizaciones en tiempo real de inventario
WS    /api/v1/ws/notifications/{userId}     // Notificaciones push personalizadas para usuario
WS    /api/v1/ws/kitchen/{branchId}         // Pantalla de cocina con pedidos en preparación
WS    /api/v1/ws/analytics/{branchId}       // Métricas en tiempo real para gerentes
```

### 9.2. Protocolos de Mensaje
**Mensaje desde servidor a cliente:**
```json
{
  "type": "ORDER_STATUS_UPDATE",
  "data": {
    "orderId": "string",
    "estado": "string",
    "timestamp": "ISO 8601 timestamp",
    "branchId": "string"
  }
}
```

**Mensaje desde cliente a servidor:**
```json
{
  "type": "ORDER_ACTION",
  "data": {
    "orderId": "string",
    "action": "aceptar|rechazar|preparando|listo|entregado",
    "timestamp": "ISO 8601 timestamp",
    "notas": "string optional"
  }
}
```

### 9.3. Casos de Uso Adecuados
- Actualizaciones de estado de pedidos en tiempo real para personal de cocina y meseros
- Notificaciones instantáneas a clientes sobre cambios en sus pedidos
- Monitoreo de inventario en tiempo real para gerentes y personal de almacén
- Pantallas de cocina dinámicas que muestran pedidos preparándose
- Métricas de rendimiento en tiempo real para toma de decisiones operativas

## Preguntas Abiertas y Decisiones Pendientes
- ¿Deberíamos implementar GraphQL como alternativa o complemento a REST para ciertas consultas complejas de frontend?
- ¿Qué estrategia de caché de respuesta (CDN, proxy, en memoria) sería más efectiva para nuestros patrones de uso esperados?
- ¿Cómo manejaremos la evolución de versiones de API cuando tengamos clientes móviles que no se actualizan frecuentemente?
- ¿Qué nivel de detalle deberíamos incluir en los esquemas de request/response para balancear exhaustividad con simplicidad de implementación?

## Conclusión
Este análisis de contratos de API proporciona una base sólida para las fases subsiguientes de diseño e implementación. Cada endpoint, método HTTP, código de estado, esquema de request/response y estrategia de autenticación ha sido diseñada para soportar los requisitos funcionales y no funcionales del ecosistema Mizu, cumpliendo con el cuarto criterio EARS especificado en la spec de organización del análisis.

Los análisis incluyen:
- Principios de diseño de API RESTful y convenciones de nombrado
- Estrategias de versionamiento y compatibilidad hacia atrás
- Autenticación basada en JWT con refresh tokens y autorización RBAC
- Definición exhaustiva de endpoints por dominio de negocio con esquemas detallados
- Códigos de estado HTTP apropiados y manejo de errores consistente
- Consideraciones de seguridad, rendimiento y escalabilidad específicas para APIs
- Estrategias de documentación automática y prueba de contratos
- Comunicación en tiempo real complementaria mediante WebSockets cuando aplique
- Mapeo completo a requisitos funcionales y no funcionales origen

Este documento está listo para revisión y sirve como entrada para la arquitectura técnica y diseño UI/UX que seguirán en los subsiguientes documentos de analysis.