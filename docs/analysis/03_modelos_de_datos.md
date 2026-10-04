# Análisis de Modelos de Datos del Proyecto Mizu Ecosystem

## Introducción
Este documento presenta el modelo relacional único (PostgreSQL) para los cinco módulos del ecosistema Mizu: Mizu Experience, Mizu Go, Mizu Order Hub, Mizu Stock y Mizu Admin. En esta línea base **no hay almacenamiento no relacional**: las colecciones MongoDB de la versión anterior (logs, métricas, análisis de uso, notificaciones push) se eliminaron; todo el modelo es relacional.

Cada entidad está trazada a al menos un caso de uso (CU) o requisito no funcional (NFR) de `Especificacion_IEEE830.md` (v1.1). La convención de tipos y de claves primarias se define en la sección [6. Optimización de Base de Datos](#6-optimización-de-base-de-datos).

Convención de trazabilidad: el marcador **◆** indica la trazabilidad fijada explícitamente por el usuario en el checkpoint C1 (04/10/2026); el resto se infiere del catálogo de CUs/NFRs de IEEE830 y es revisable en la verificación (T5).[^checkpoint]

## Alcance
El modelo cubre exactamente **26 entidades**, organizadas en cuatro áreas de negocio:

1. **Acceso y Personas** (sección 1): Usuario, Credencial, Rol, Usuario_Rol.
2. **Sucursal y RRHH** (sección 2): Sucursal, Mesa, Empleado, Asistencia, Nomina.
3. **Menú y Ventas** (sección 3): CategoriaProducto, Producto, Oferta, Pedido, DetallePedido, Pago, Reserva, Puntos, TransaccionPuntos.
4. **Inventario y Finanzas** (sección 4): Ingrediente, MovimientoInventario, ProductoIngrediente, Proveedor, ProveedorIngrediente, OrdenCompra, OrdenCompraDetalle, Gasto.

Para cada entidad se documentan: atributos (línea base aprobada), relaciones, reglas de integridad mínimas (unidades, FKs, rangos, checks), índices mínimos (FKs + paths calientes) y trazabilidad a requisitos. Cerrando el documento: integridad referencial global (sección 5) y optimización de base de datos (sección 6).

---

## 1. Acceso y Personas

### 1.1 `Usuario`
**Descripción**: Cuenta de cualquier persona que interactúa con el ecosistema (cliente, empleado, administrador); el acceso se resuelve por roles (`Rol`/`Usuario_Rol`), no por tipo en la cuenta.

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| nombre | TEXT | NOT NULL |
| email | TEXT | NOT NULL, UNIQUE, formato de correo validado en app |
| telefono | TEXT | Nullable |
| fecha_registro | TIMESTAMPTZ | NOT NULL; no futura |
| activo | BOOLEAN | NOT NULL, DEFAULT TRUE (desactivación, nunca borrado) |
| preferencias | JSONB | Idioma, notificaciones, etc.; índice GIN |

**Relaciones:** 1—N `Credencial`; 1—N `Usuario_Rol`; 1—N `Empleado`; 1—N `Pedido` (cliente_id); 1—N `Reserva` (cliente_id); 1—N `Puntos`; 1—N `TransaccionPuntos`; 1—N `Gasto` (registrado_por).

**Reglas de integridad:**
- `email` es único en todo el sistema.
- `fecha_registro` ≤ ahora.
- Una cuenta con historial (puntos, gastos registrados) no se elimina; se desactiva (`activo = FALSE`).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`email`) | Autenticación por correo |
| GIN (`preferencias`) | Búsqueda dentro del JSONB |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-GO-01` | Registro y autenticación |
| `CU-AD-03` | Gestión de usuarios administradores |
| `NFR-ST-02` | Base del control de acceso por rol |

### 1.2 `Credencial`
**Descripción**: Credencial de autenticación de un usuario; nunca se almacena el valor en texto plano.

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| usuario_id | UUID | NOT NULL, FK → `Usuario` |
| tipo | TEXT | NOT NULL; CHECK en ('password','oauth_google','oauth_facebook') |
| valor_hash | TEXT | NOT NULL; hash seguro, nunca texto plano |
| activo | BOOLEAN | NOT NULL, DEFAULT TRUE |

**Relaciones:** N—1 `Usuario` (varias credenciales por usuario: password + OAuth).

**Reglas de integridad:**
- `valor_hash` no puede ser nulo o vacío.
- UNIQUE (`usuario_id`, `tipo`): una credencial por método de autenticación.

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`usuario_id`, `tipo`) | Integridad y lookup de credencial al iniciar sesión |
| FK (`usuario_id`) | Índice explícito de la FK (convención) |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-GO-01` | Registro y autenticación |
| `NFR-EX-02` | Seguridad en transacciones |

### 1.3 `Rol`
**Descripción**: Rol de acceso del sistema; sus permisos incluyen flags de alto riesgo que obligan a MFA en la app (◆).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| nombre | TEXT | NOT NULL, UNIQUE |
| descripcion | TEXT | Nullable |
| permisos | JSONB | NOT NULL; incluye flag de alto riesgo → exige MFA (◆) |
| activo | BOOLEAN | NOT NULL, DEFAULT TRUE |

**Relaciones:** 1—N `Usuario_Rol` (asignación a usuarios).

**Reglas de integridad:**
- `nombre` es único.
- Un permiso de alto riesgo en `permisos` dispara MFA; validado en la capa de autorización (app).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`nombre`) | Integridad de catálogo |
| GIN (`permisos`) | Búsqueda de roles por permiso |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-AD-03` ◆ | Gestión de usuarios administradores y permisos |
| `NFR-ST-02` ◆ | Roles de acceso diferenciados |

### 1.4 `Usuario_Rol`
**Descripción**: Tabla junction de la relación muchos-a-muchos `Usuario` ↔ `Rol`.

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| usuario_id | UUID | NOT NULL, FK → `Usuario` |
| rol_id | UUID | NOT NULL, FK → `Rol` |

**Relaciones:** N—1 `Usuario`, N—1 `Rol`.

**Reglas de integridad:**
- UNIQUE (`usuario_id`, `rol_id`): un solo registro por pareja.

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`usuario_id`, `rol_id`) | Integridad y consulta de roles por usuario |
| FK (`rol_id`) | Índice explícito de la FK (convención) |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-AD-03` | Asignación de roles a usuarios |
| `NFR-ST-02` | Autorización por rol |

## 2. Sucursal y RRHH

### 2.1 `Sucursal`
**Descripción**: Ubicación física de la cadena; sus coordenadas soportan el criterio GPS de `CU-GO-02` (◆).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| nombre | TEXT | NOT NULL |
| direccion | TEXT | |
| ciudad | TEXT | |
| telefono | TEXT | |
| email | TEXT | |
| horario_apertura | TIME | NOT NULL |
| horario_cierre | TIME | NOT NULL |
| activo | BOOLEAN | NOT NULL, DEFAULT TRUE (cierre de tienda = desactivación, no borrado) |
| coordenadas_lat | NUMERIC(10,8) | CHECK entre -90 y 90 (◆ trazado a `CU-GO-02`) |
| coordenadas_lng | NUMERIC(11,8) | CHECK entre -180 y 180 (◆ trazado a `CU-GO-02`) |

**Relaciones:** 1—N `Mesa`, 1—N `Empleado`, 1—N `Asistencia`, 1—N `Pedido`, 1—N `Reserva`, 1—N `Ingrediente`, 1—N `OrdenCompra`, 1—N `Gasto`.

**Reglas de integridad:**
- `horario_cierre` > `horario_apertura`.
- Rango válido de coordenadas (`CHECK`).
- Una sucursal con historial contable u operativo no se elimina; se desactiva.

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| Parcial: `WHERE activo = TRUE` | Path caliente de listas de sucursales operativas |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `NFR-ST-01` | Escalabilidad multi-sucursal |
| `CU-GO-02` ◆ | Coordenadas para el criterio GPS de pedidos por app |

### 2.2 `Mesa`
**Descripción**: Mesa física dentro de una sucursal y su estado operativo en tiempo real.

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| sucursal_id | UUID | NOT NULL, FK → `Sucursal` |
| numero | TEXT | NOT NULL; identificador de la mesa en la sucursal |
| capacidad | INTEGER | NOT NULL; CHECK > 0 |
| estado | TEXT | NOT NULL; CHECK en ('libre','ocupada','reservada','mantenimiento') |

**Relaciones:** N—1 `Sucursal`; 1—N `Pedido` (mesa_id, cuando el pedido es en sala); 1—N `Reserva` (mesa_id).

**Reglas de integridad:**
- UNIQUE (`sucursal_id`, `numero`): una mesa por número y sucursal.
- `capacidad` > 0.

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`sucursal_id`, `numero`) | Integridad e índice de la FK |
| Parcial: `(sucursal_id, numero) WHERE estado IN ('ocupada','reservada')` | Tablero del POS (path caliente de `CU-OH-02`) |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-EX-02` | Reserva por mesa |
| `CU-OH-02` | Registro de pedidos por mesa |

### 2.3 `Empleado`
**Descripción**: Expediente laboral de un empleado de la cadena; "empleado" es además un rol de acceso (`NFR-ST-02`), pero la entidad solo soporta expedientes para nómina (◆).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| usuario_id | UUID | NOT NULL, UNIQUE, FK → `Usuario` |
| sucursal_id | UUID | FK → `Sucursal` (sucursal principal; nullable) |
| numero_empleado | TEXT | NOT NULL, UNIQUE |
| fecha_ingreso | DATE | NOT NULL |
| fecha_egreso | DATE | Nullable; null = activo (sustituye a `activo`) |
| salario_base | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 |

**Relaciones:** N—1 `Usuario`, N—1 `Sucursal`; 1—N `Nomina`, 1—N `Asistencia`; 1—N `Pedido` (empleado_id, quien registró el pedido).

**Reglas de integridad:**
- UNIQUE (`usuario_id`) y UNIQUE (`numero_empleado`).
- `fecha_egreso` ≥ `fecha_ingreso` cuando no es null.
- Un empleado con historial (nómina, asistencia) no se elimina; se cierra con `fecha_egreso`.

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`usuario_id`) | Integridad e índice de la FK |
| UNIQUE (`numero_empleado`) | Integridad |
| FK (`sucursal_id`) | Índice explícito de la FK |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-AD-05` | Expediente laboral que alimenta la nómina (◆) |
| `NFR-ST-02` | "Empleado" como rol de acceso en el establecimiento |

### 2.4 `Asistencia`
**Descripción**: Registro de entrada y salida de un empleado por día (una fila por empleado/día); lo captura Order Hub (`CU-OH-05`) y lo consultan `CU-AD-06` y `CU-AD-05` (◆).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| empleado_id | UUID | NOT NULL, FK → `Empleado` |
| sucursal_id | UUID | NOT NULL, FK → `Sucursal` |
| fecha | DATE | NOT NULL; una fila por empleado/día |
| hora_entrada | TIME | NOT NULL |
| hora_salida | TIME | Nullable; CHECK > `hora_entrada` |
| registrado_por | UUID | FK → `Usuario` (operador que capturó el registro; nullable) |

**Relaciones:** N—1 `Empleado`, N—1 `Sucursal`, N—1 `Usuario` (registrado_por).

**Reglas de integridad:**
- UNIQUE (`empleado_id`, `fecha`): máximo un registro por empleado y día.
- `hora_salida` > `hora_entrada` cuando ambas existen.
- `registrado_por` debe ser un usuario con acceso de establecimiento (validado en app, `NFR-ST-02`).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`empleado_id`, `fecha`) | Integridad e índice de la FK de `empleado` |
| `(sucursal_id, fecha)` | Filtros de `CU-AD-06` por sucursal y período |
| FK (`registrado_por`) | Índice explícito de la FK |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-OH-05` ◆ | Captura individual de entrada/salida (fuente del registro) |
| `CU-AD-06` ◆ | Consulta en solo lectura en Admin |
| `CU-AD-05` ◆ | Horas trabajadas que alimentan la nómina |

### 2.5 `Nomina`
**Descripción**: Registro de nómina por empleado y período; lo automatizable (horas, devengados, deducciones, neto) se calcula en la app antes de guardar (nota a del checkpoint).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| empleado_id | UUID | NOT NULL, FK → `Empleado` |
| periodo_inicio | DATE | NOT NULL |
| periodo_fin | DATE | NOT NULL; CHECK ≥ `periodo_inicio` |
| horas_normales | NUMERIC(5,2) | NOT NULL; CHECK ≥ 0 |
| horas_extra | NUMERIC(5,2) | NOT NULL; CHECK ≥ 0 |
| bonificaciones | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 |
| descuentos | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 |
| total_devengado | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 |
| total_deducciones | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 |
| neto_a_pagar | NUMERIC(12,2) | NOT NULL; CHECK = `total_devengado − total_deducciones` |
| estado | TEXT | NOT NULL; CHECK en ('pendiente','aprobada','pagada') |
| fecha_pago | DATE | Nullable; NOT NULL cuando `estado = 'pagada'` |
| metodo_pago | TEXT | Nullable; CHECK en ('transferencia','cheque','efectivo'); NOT NULL cuando `estado = 'pagada'` |
| observaciones | TEXT | Nullable |

**Relaciones:** N—1 `Empleado` (un período por empleado).

**Reglas de integridad:**
- UNIQUE (`empleado_id`, `periodo_inicio`, `periodo_fin`): una nómina por empleado y período.
- `neto_a_pagar = total_devengado − total_deducciones` (`CHECK`).
- `estado = 'pagada'` exige `fecha_pago` y `metodo_pago` (CHECK de estado).
- No se elimina una nómina pagada; el ciclo lo cubre `estado`.

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| FK (`empleado_id`) | Índice explícito de la FK |
| Parcial: `(empleado_id, periodo_inicio) WHERE estado = 'pendiente'` | Path caliente de generación de nómina (`CU-AD-05`) |

**Trazabilidad (dual):**

| Requisito | Aporte |
|-----------|--------|
| `CU-AD-01` + `CU-AD-05` ◆ | Registro de egresos de nómina y cálculo por período (trazabilidad dual) |

## 3. Menú y Ventas

### 3.1 `CategoriaProducto`
**Descripción**: Agrupa productos del menú para su organización y orden de presentación.

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| nombre | TEXT | NOT NULL, UNIQUE |
| orden_display | INTEGER | NOT NULL; CHECK ≥ 0 |
| activo | BOOLEAN | NOT NULL, DEFAULT TRUE (catálogo) |

**Relaciones:** 1—N `Producto`.

**Reglas de integridad:**
- `nombre` es único; una categoría con productos no se elimina, se desactiva.

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`nombre`) | Integridad |
| Parcial: `WHERE activo = TRUE` | Listado de menú |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-EX-01` | Menú organizado por categorías |
| `CU-OH-02` | Registro de pedidos por categoría en POS |

### 3.2 `Producto`
**Descripción**: Item vendible del menú, con precio, costo y receta declarada.

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| categoria_id | UUID | NOT NULL, FK → `CategoriaProducto` |
| nombre | TEXT | NOT NULL |
| precio_base | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 |
| costo_preparacion | NUMERIC(12,2) | CHECK ≥ 0 |
| tiempo_preparacion_estandar | INTEGER | CHECK ≥ 0 (minutos) |
| ingredientes | JSONB | Lista de ingredientes del producto (consulta del menú) |
| imagen_url | TEXT | Nullable |
| disponible | BOOLEAN | NOT NULL, DEFAULT TRUE |
| activo | BOOLEAN | NOT NULL, DEFAULT TRUE (catálogo) |

**Relaciones:** N—1 `CategoriaProducto`; 1—N `DetallePedido`, 1—N `Oferta`, 1—N `ProductoIngrediente` (receta formal para inventario).

**Reglas de integridad:**
- `precio_base`, `costo_preparacion` y `tiempo_preparacion_estandar` ≥ 0.
- Un producto con ventas históricas no se elimina; se desactiva (`activo`/`disponible`).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| FK (`categoria_id`) | Índice explícito de la FK |
| `(categoria_id, disponible)` | Browsing de menú por categoría (`CU-EX-01`/`CU-OH-02`) |
| GIN (`ingredientes`) | Búsqueda dentro del JSONB |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-EX-01` | Catálogo consultable |
| `CU-OH-02` | Registro de pedidos en POS |

### 3.3 `Oferta`
**Descripción**: Promoción ligada a un producto con vigencia y valor (soporta `CU-GO-03` ◆).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| producto_id | UUID | NOT NULL, FK → `Producto` |
| tipo_descuento | TEXT | NOT NULL; CHECK en ('porcentaje','fijo') |
| valor | NUMERIC(12,2) | NOT NULL; CHECK: 'porcentaje' → 0..100; 'fijo' → > 0 |
| fecha_inicio | DATE | NOT NULL |
| fecha_fin | DATE | NOT NULL; CHECK ≥ `fecha_inicio` |
| activa | BOOLEAN | NOT NULL, DEFAULT TRUE (catálogo) |

**Relaciones:** N—1 `Producto`.

**Reglas de integridad:**
- Rangos de `valor` según `tipo_descuento` (`CHECK`).
- No deben coexistir dos ofertas activas solapadas del mismo producto (regla de negocio en app).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| Parcial: FK (`producto_id`) WHERE `activa = TRUE` | Ofertas vigentes por producto (path caliente de `CU-GO-03`) |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-GO-03` ◆ | Promociones y descuentos por producto |

### 3.4 `Pedido`
**Descripción**: Pedido completo registrado en web, app o establecimiento; los 5 timestamps de estado sustituyen a un historial de estados, y el concepto "Envio" quedó cubierto por `estado`/`origen`/`direccion_entrega`.

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| numero | TEXT | NOT NULL, UNIQUE; número legible del pedido |
| sucursal_id | UUID | NOT NULL, FK → `Sucursal` |
| mesa_id | UUID | FK → `Mesa` (nullable; vacío en domicilio/recogida) |
| cliente_id | UUID | FK → `Usuario` (nullable; solo clientes registrados) |
| empleado_id | UUID | FK → `Empleado` (nullable; quien registró el pedido) |
| tipo | TEXT | NOT NULL; CHECK en ('web','app','presencial','telefonico') |
| estado | TEXT | NOT NULL; CHECK en ('pendiente','confirmado','preparacion','listo','entregado','cancelado') |
| fecha_hora | TIMESTAMPTZ | NOT NULL; creación del pedido |
| fecha_hora_confirmado | TIMESTAMPTZ | Nullable; ≥ `fecha_hora` |
| fecha_hora_preparacion_inicio | TIMESTAMPTZ | Nullable; ≥ `fecha_hora_confirmado` |
| fecha_hora_listo | TIMESTAMPTZ | Nullable; ≥ `fecha_hora_preparacion_inicio` |
| fecha_hora_entregado | TIMESTAMPTZ | Nullable; ≥ `fecha_hora_listo` |
| subtotal | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 |
| descuento | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 y ≤ `subtotal` |
| impuestos | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 |
| total | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0; invariant `total = subtotal − descuento + impuestos` (calculado en app) |
| metodo_pago | TEXT | CHECK en ('efectivo','tarjeta','transferencia','vale','puntos') |
| referencia_pago | TEXT | Identificador del gateway (conciliación `CU-AD-01`) |
| notas_cliente | TEXT | Ej. "sin cebolla" |
| origen | TEXT | CHECK en ('domicilio','para_recoger','en_local') |
| direccion_entrega | TEXT | NOT NULL cuando `origen = 'domicilio'` (CHECK) |

**Relaciones:** N—1 `Sucursal`; N—1 `Mesa`, `Usuario` (cliente), `Empleado` (todas nullable); 1—N `DetallePedido`; 1—N `Pago`; 1—N `TransaccionPuntos`.

**Reglas de integridad:**
- Rangos de moneda no negativos; `descuento ≤ subtotal`.
- Monotonía de los 5 timestamps de estado (cada uno ≥ el anterior; garantizada en app).
- `direccion_entrega` obligatoria en domicilio.
- Un pedido con pagos asociados no se elimina (ver integridad referencial global); su ciclo lo cubre `estado`.

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`numero`) | Número legible del pedido |
| FKs: `sucursal_id`, `mesa_id`, `cliente_id`, `empleado_id` | Índices explícitos de las FK |
| `(sucursal_id, estado)` | Tablero POS y reportes por sucursal (`CU-OH-02`, `CU-AD-07`) |
| Parcial: `(sucursal_id, fecha_hora) WHERE estado IN ('pendiente','confirmado','preparacion','listo')` | Paths calientes operativos por rango de fechas |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-OH-01`, `CU-OH-02`, `CU-OH-03` | Recepción, registro y gestión de estados en establecimiento |
| `CU-EX-03` | Pedido en línea con confirmación automática |

### 3.5 `DetallePedido`
**Descripción**: Línea individual de un pedido (producto, cantidad y precio aplicado).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| pedido_id | UUID | NOT NULL, FK → `Pedido` |
| producto_id | UUID | NOT NULL, FK → `Producto` |
| cantidad | INTEGER | NOT NULL; CHECK ≥ 1 |
| precio_unitario | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 (puede diferir de `precio_base` por oferta) |
| subtotal | NUMERIC(12,2) | NOT NULL; CHECK = `cantidad × precio_unitario` |
| notas_especiales | TEXT | Personalizaciones (sin cebolla, bien cocido) |

**Relaciones:** N—1 `Pedido`, N—1 `Producto`.

**Reglas de integridad:**
- `cantidad` ≥ 1; `subtotal = cantidad × precio_unitario` (`CHECK`).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| FK (`pedido_id`) | Detalle por pedido |
| FK (`producto_id`) | Ventas por producto (reportes de `CU-AD-07`) |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-OH-02` | Registro de líneas del pedido en POS |

### 3.6 `Pago`
**Descripción**: Pago de un pedido (permite pagos divididos); su comprobante lo exige la conciliación contable.

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| pedido_id | UUID | NOT NULL, FK → `Pedido` |
| monto | NUMERIC(12,2) | NOT NULL; CHECK > 0 |
| metodo | TEXT | NOT NULL; CHECK en ('efectivo','tarjeta','transferencia','vale','puntos','otros') |
| referencia_externa | TEXT | Comprobante del gateway/banco; lo exige la conciliación de `CU-AD-01` y la confirmación de `CU-EX-03` (◆) |
| estado | TEXT | NOT NULL; CHECK en ('pendiente','procesando','exitoso','fallido','revertido') |
| fecha_hora | TIMESTAMPTZ | NOT NULL; inicio del pago |

**Relaciones:** N—1 `Pedido`.

**Reglas de integridad:**
- `monto` > 0; rangos permitidos de `metodo`/`estado`.
- La suma de pagos exitosos de un pedido cubre su `total` (conciliación en app, `CU-AD-01`).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| FK (`pedido_id`) | Pagos por pedido |
| Parcial: `WHERE estado = 'exitoso'` | Path caliente de conciliación (`CU-AD-01`) |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-EX-03` | Confirmación de pago del pedido en línea |
| `CU-AD-01` ◆ | Ingreso conciliable en contabilidad |

### 3.7 `Reserva`
**Descripción**: Reserva de mesa por un cliente, con rango horario y contacto (núcleo trazado a `CU-EX-02` ◆).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| sucursal_id | UUID | NOT NULL, FK → `Sucursal` |
| mesa_id | UUID | NOT NULL, FK → `Mesa` |
| cliente_id | UUID | FK → `Usuario` (nullable; reserva anónima con contacto) |
| fecha_hora_inicio | TIMESTAMPTZ | NOT NULL |
| fecha_hora_fin | TIMESTAMPTZ | NOT NULL; CHECK > `fecha_hora_inicio` |
| numero_personas | INTEGER | NOT NULL; CHECK ≥ 1 y ≤ `Mesa.capacidad` (app) |
| estado | TEXT | NOT NULL; CHECK en ('pendiente','confirmada','en_curso','finalizada','cancelada','no_show') |
| contacto_nombre | TEXT | NOT NULL |
| contacto_telefono | TEXT | NOT NULL |
| contacto_email | TEXT | Nullable |

**Relaciones:** N—1 `Sucursal`, N—1 `Mesa`, N—1 `Usuario` (cliente, nullable).

**Reglas de integridad:**
- `fecha_hora_fin > fecha_hora_inicio`.
- `numero_personas` entre 1 y la capacidad de la mesa (regla de negocio en app).
- La mesa pertenece a la sucursal de la reserva (regla de negocio en app).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| FKs: `sucursal_id`, `mesa_id`, `cliente_id` | Índices explícitos de las FK |
| `(sucursal_id, fecha_hora_inicio)` | Disponibilidad de mesas por sucursal y rango (`CU-EX-02`) |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-EX-02` ◆ | Reserva desde web (y app) con confirmación de contacto |

### 3.8 `Puntos`
**Descripción**: Saldo del programa de fidelización de un usuario (uno por usuario).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| usuario_id | UUID | NOT NULL, UNIQUE, FK → `Usuario` |
| saldo | INTEGER | NOT NULL; CHECK ≥ 0 |
| total_acumulado | INTEGER | NOT NULL; CHECK ≥ 0 (ganancia histórica) |
| nivel | TEXT | Nivel del programa (valor definido en app) |

**Relaciones:** N—1 `Usuario` (1—1); 1—N `TransaccionPuntos` (mismo `usuario_id`).

**Reglas de integridad:**
- UNIQUE (`usuario_id`): un registro de puntos por usuario.
- Invariant en app: `saldo = total_acumulado − canjeos` (sobre `TransaccionPuntos`).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`usuario_id`) | Integridad 1—1 e índice de la FK |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-GO-04` | Sistema de puntos y recompensas |

### 3.9 `TransaccionPuntos`
**Descripción**: Movimiento individual de puntos del usuario (ganancia por pedido o canjeo).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| usuario_id | UUID | NOT NULL, FK → `Usuario` |
| pedido_id | UUID | FK → `Pedido` (nullable; canjeo sin pedido asociado) |
| puntos | INTEGER | NOT NULL; CHECK > 0 |
| tipo | TEXT | NOT NULL; CHECK en ('ganancia','canjeo') |
| fecha | TIMESTAMPTZ | NOT NULL |

**Relaciones:** N—1 `Usuario`, N—1 `Pedido` (nullable).

**Reglas de integridad:**
- `puntos` > 0.
- Un canjeo no puede superar el saldo del usuario (regla de negocio transaccional en app, sobre `Puntos.saldo`).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| FK (`usuario_id`) | Historial de puntos por usuario (`CU-GO-04`) |
| FK (`pedido_id`) | Índice explícito de la FK |
| `(usuario_id, fecha)` | Path caliente de historial por período |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-GO-04` | Ganancia y canjeo de puntos |

## 4. Inventario y Finanzas

### 4.1 `Ingrediente`
**Descripción**: Existencia de un ingrediente por sucursal y lote; sustituye al concepto vago "Inventario" del documento anterior.

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| sucursal_id | UUID | NOT NULL, FK → `Sucursal` |
| nombre | TEXT | NOT NULL |
| unidad | TEXT | NOT NULL; unidad de medida ('kg', 'unidad', 'ml', etc.) |
| stock_actual | NUMERIC(14,3) | NOT NULL; CHECK ≥ 0 |
| stock_minimo | NUMERIC(14,3) | NOT NULL; CHECK ≥ 0 |
| lote | TEXT | Lote del proveedor |
| numero_serie | TEXT | Serie (trazabilidad del lote) |
| fecha_vencimiento | DATE | Alerta de vencimiento en app |

**Relaciones:** N—1 `Sucursal`; 1—N `MovimientoInventario`, 1—N `ProductoIngrediente` (receta), 1—N `ProveedorIngrediente` (catálogo de suministro).

**Reglas de integridad:**
- `stock_actual` y `stock_minimo` ≥ 0, en la unidad declarada.
- Bajo stock: `stock_actual ≤ stock_minimo` dispara la alerta de `CU-ST-04` (query, no restricción).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| FK (`sucursal_id`) | Inventario por sucursal (`CU-ST-01`) |
| Parcial: `(sucursal_id, stock_actual) WHERE stock_actual <= stock_minimo` | Path caliente de alerta de bajo stock (`CU-ST-04`) |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-ST-01` ◆ | Control de inventario por sucursal |
| `CU-ST-03` ◆ | Existencias y costeo |
| `CU-ST-04` ◆ | Alertas de bajo stock y vencimiento |

### 4.2 `MovimientoInventario`
**Descripción**: Entrada o salida de inventario (auditoría de existencias por `Ingrediente`).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| ingrediente_id | UUID | NOT NULL, FK → `Ingrediente` |
| tipo | TEXT | NOT NULL; CHECK en ('entrada','salida') |
| cantidad | NUMERIC(14,3) | NOT NULL; CHECK > 0, en la unidad del ingrediente |
| fecha | TIMESTAMPTZ | NOT NULL |
| referencia | TEXT | Orden de compra, mermas, ajuste (contexto del movimiento) |
| registrado_por | UUID | FK → `Usuario` (nullable) |

**Relaciones:** N—1 `Ingrediente`, N—1 `Usuario` (registrado_por, nullable).

**Reglas de integridad:**
- `cantidad` > 0.
- Una "salida" no puede dejar `stock_actual` negativo (validación transaccional en app al conciliar contra `Ingrediente`).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| FK (`ingrediente_id`) | Historial por ingrediente (`CU-ST-01`) |
| FK (`registrado_por`) | Índice explícito de la FK |
| `(ingrediente_id, fecha)` | Path caliente de reportes de consumo por período (`CU-ST-03`) |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-ST-01` ◆ | Registro de entradas y salidas de inventario |

### 4.3 `ProductoIngrediente`
**Descripción**: Receta formal: ingrediente y cantidad que compone un producto (costeo de recetas).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| producto_id | UUID | NOT NULL, FK → `Producto` |
| ingrediente_id | UUID | NOT NULL, FK → `Ingrediente` |
| cantidad | NUMERIC(14,3) | NOT NULL; CHECK > 0 |
| unidad | TEXT | NOT NULL; unidad de la receta |

**Relaciones:** N—1 `Producto`, N—1 `Ingrediente`.

**Reglas de integridad:**
- UNIQUE (`producto_id`, `ingrediente_id`): una fila por pareja receta.
- `cantidad` > 0, en `unidad` coherente con `Ingrediente.unidad` (regla de negocio en app).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`producto_id`, `ingrediente_id`) | Integridad e índices de ambas FK |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-ST-04` ◆ | Recetas para costeo y consumo proyectado |

### 4.4 `Proveedor`
**Descripción**: Proveedor de insumos de la cadena (catálogo).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| nombre | TEXT | NOT NULL |
| contacto_nombre | TEXT | |
| contacto_telefono | TEXT | |
| contacto_email | TEXT | |
| rubro | TEXT | Rubro del proveedor |
| activo | BOOLEAN | NOT NULL, DEFAULT TRUE (catálogo) |

**Relaciones:** 1—N `ProveedorIngrediente` (catálogo de suministro), 1—N `OrdenCompra`.

**Reglas de integridad:**
- Un proveedor con órdenes asociadas no se elimina; se desactiva (`activo = FALSE`).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| Parcial: `WHERE activo = TRUE` | Listado de proveedores activos |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-ST-02` ◆ | Gestión de proveedores |

### 4.5 `ProveedorIngrediente`
**Descripción**: Catálogo de qué proveedor suministra qué ingrediente y a qué precio (tabla junction de suministro).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| proveedor_id | UUID | NOT NULL, FK → `Proveedor` |
| ingrediente_id | UUID | NOT NULL, FK → `Ingrediente` |
| precio | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 |
| condiciones | TEXT | Condiciones de venta |

**Relaciones:** N—1 `Proveedor`, N—1 `Ingrediente`.

**Reglas de integridad:**
- UNIQUE (`proveedor_id`, `ingrediente_id`).
- `precio` ≥ 0.

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| UNIQUE (`proveedor_id`, `ingrediente_id`) | Integridad e índices de ambas FK |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-ST-02` ◆ | Catálogo de suministro por proveedor |

### 4.6 `OrdenCompra`
**Descripción**: Orden de compra a un proveedor hacia una sucursal (origen de las entradas de inventario).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| proveedor_id | UUID | NOT NULL, FK → `Proveedor` |
| sucursal_id | UUID | NOT NULL, FK → `Sucursal` |
| fecha | DATE | NOT NULL; fecha de la orden |
| estado | TEXT | NOT NULL; CHECK en ('pendiente','confirmada','recibida','cancelada') |
| total | NUMERIC(12,2) | CHECK ≥ 0; = Σ líneas de `OrdenCompraDetalle` (app) |
| referencia | TEXT | Referencia del proveedor |

**Relaciones:** N—1 `Proveedor`, N—1 `Sucursal`; 1—N `OrdenCompraDetalle`.

**Reglas de integridad:**
- `estado` en valores permitidos; el ciclo de vida lo cubre `estado` (no se eliminan órdenes recibidas).
- `total` consistente con el detalle (validado en app).

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| FKs: `proveedor_id`, `sucursal_id` | Índices explícitos de las FK |
| `(fecha)` | Path caliente de costeo por período (`CU-ST-03`) |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-ST-02` ◆ | Gestión de proveedores y pedidos de insumos |
| `CU-ST-03` ◆ | Costeo de compras por período y sucursal |

### 4.7 `OrdenCompraDetalle`
**Descripción**: Línea de una orden de compra (ingrediente, cantidad y precio unitario).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| orden_compra_id | UUID | NOT NULL, FK → `OrdenCompra` |
| ingrediente_id | UUID | NOT NULL, FK → `Ingrediente` |
| cantidad | NUMERIC(14,3) | NOT NULL; CHECK > 0 |
| precio_unitario | NUMERIC(12,2) | NOT NULL; CHECK ≥ 0 |

**Relaciones:** N—1 `OrdenCompra`, N—1 `Ingrediente`.

**Reglas de integridad:**
- `cantidad` > 0 y `precio_unitario` ≥ 0.

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| FK (`orden_compra_id`) | Líneas por orden |
| FK (`ingrediente_id`) | Índice explícito de la FK |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-ST-02` | Detalle de órdenes de compra |

### 4.8 `Gasto`
**Descripción**: Egreso de una sucursal (fuente de los reportes económicos y de la utilidad).

**Atributos:**

| Atributo | Tipo | Restricciones y notas |
|----------|------|------------------------|
| id | UUID (v7) | PK, NOT NULL |
| sucursal_id | UUID | NOT NULL, FK → `Sucursal` |
| fecha | DATE | NOT NULL |
| categoria | TEXT | NOT NULL; categoría de gasto |
| concepto | TEXT | Detalle del gasto |
| monto | NUMERIC(12,2) | NOT NULL; CHECK > 0 |
| metodo_pago | TEXT | Método del pago |
| referencia | TEXT | Comprobante |
| registrado_por | UUID | FK → `Usuario` (nullable) |

**Relaciones:** N—1 `Sucursal`, N—1 `Usuario` (registrado_por, nullable).

**Reglas de integridad:**
- `monto` > 0; `categoria` no vacía.
- No se eliminan gastos registrados; el egreso es parte del historial contable.

**Índices mínimos:**

| Índice | Justificación |
|--------|---------------|
| PK (`id`) | Búsqueda por ID |
| FKs: `sucursal_id`, `registrado_por` | Índices explícitos de las FK |
| `(fecha)` | Path caliente de ingresos vs. egresos por período (`CU-AD-07`) |

**Trazabilidad:**

| Requisito | Aporte |
|-----------|--------|
| `CU-AD-01` ◆ | Registro de egresos para reportes contables |

## 5. Integridad Referencial Global

Las 38 FKs del modelo definen `ON DELETE` en tres grupos, con justificación breve de cada regla de negocio. `ON UPDATE` es **CASCADE** para todas las FKs (convención: las PK UUID v7 son inmutables, por lo que la acción es de cumplimiento preventivo).

**Grupo 1 — `ON DELETE CASCADE`** (el hijo pierde sentido sin el padre; ninguna es registro de auditoría):

| FK (tabla.columna) | Justificación |
|--------------------|-----------------------------|
| `Credencial.usuario_id` | Las credenciales solo existen con su cuenta |
| `Usuario_Rol.usuario_id`, `Usuario_Rol.rol_id` | La asignación desaparece si falta cualquiera de los extremos |
| `Mesa.sucursal_id` | Una mesa es activo de su sucursal |
| `Empleado.usuario_id` | El expediente laboral solo existe con la cuenta |
| `DetallePedido.pedido_id` | La línea vive y muere con el pedido |
| `ProductoIngrediente.producto_id` | La receta deja de existir si el producto se retira |
| `ProveedorIngrediente.proveedor_id`, `ProveedorIngrediente.ingrediente_id` | El par de suministro deja de existir si falta un extremo |
| `OrdenCompraDetalle.orden_compra_id` | La línea vive y muere con la orden |

**Grupo 2 — `ON DELETE SET NULL`** (la FK es nullable y el registro conserva valor sin la referencia):

| FK (tabla.columna) | Justificación |
|--------------------|-----------------------------|
| `Empleado.sucursal_id` | El expediente laboral sobrevive a la sucursal (alta en RRHH) |
| `Pedido.mesa_id` | El pedido sigue válido en domicilio/recogida |
| `Pedido.cliente_id` | El pedido de un cliente anónimo no depende de la cuenta |
| `Pedido.empleado_id` | El pedido conserva su registro aunque el expediente del empleado se retire |
| `Reserva.cliente_id` | La reserva anónima sobrevive sin cuenta |
| `TransaccionPuntos.pedido_id` | El historial de puntos es del usuario, no del pedido |
| `Asistencia.registrado_por`, `MovimientoInventario.registrado_por`, `Gasto.registrado_por` | El registro conserva su valor aunque el usuario que lo capturó se retire |

**Grupo 3 — `ON DELETE RESTRICT`** (el padre tiene libro contable, historial o es requisito del hijo; se desactiva en su lugar — `activo`, `estado`, `fecha_egreso` — y nunca se elimina):

| FK (tabla.columna) | Justificación |
|--------------------|-----------------------------|
| `Asistencia.empleado_id`, `Asistencia.sucursal_id` | La asistencia es libro laboral de RRHH |
| `Nomina.empleado_id` | La nómina pagada es libro laboral |
| `Producto.categoria_id` | La categoría con productos se desactiva, no se elimina |
| `Oferta.producto_id` | La oferta depende del producto del catálogo |
| `Pedido.sucursal_id` | El pedido es libro de ventas de la sucursal |
| `DetallePedido.producto_id` | El precio histórico depende del producto (se desactiva con `activo`/`disponible`) |
| `Pago.pedido_id` | El pedido con pagos no se elimina (libro contable) |
| `Reserva.sucursal_id`, `Reserva.mesa_id` | La reserva pertenece a una mesa de una sucursal |
| `Puntos.usuario_id`, `TransaccionPuntos.usuario_id` | El saldo y su historial no se eliminan; la cuenta se desactiva |
| `Ingrediente.sucursal_id` | El inventario es libro de la sucursal |
| `MovimientoInventario.ingrediente_id` | El movimiento es libro de existencias |
| `ProductoIngrediente.ingrediente_id` | El ingrediente con consumo histórico no se elimina |
| `OrdenCompra.proveedor_id`, `OrdenCompra.sucursal_id` | La orden es libro de compras |
| `OrdenCompraDetalle.ingrediente_id` | El libro de órdenes depende del ingrediente |
| `Gasto.sucursal_id` | El gasto es libro contable de la sucursal |

## 6. Optimización de Base de Datos

Convenciones aprobadas en el checkpoint C1 (Apéndice A.2 de la spec), derivadas de las skills `postgresql` y `postgres-best-practices`:

1. **PK UUID v7 en las 26 tablas.** Opaco e in-advinable y ordenado por tiempo: rendimiento de inserción y de índices comparable a un BIGINT secuencial, sin la fragmentación de B-Tree que produce el UUID v4. Generado en la capa de aplicación o con la extensión `pg_uuidv7` (compatible con PostgreSQL 15+).
2. **Tipos canónicos.** `TIMESTAMPTZ` (nunca `TIMESTAMP` sin zona horaria); `TEXT` (nunca `VARCHAR(n)`); `NUMERIC(p,s)` para todo monto monetario (nunca `FLOAT`/`REAL`). `TIME`/`DATE` solo para horas del día y fechas de negocio acopladas (horarios de sucursal, turnos de asistencia, períodos de nómina).
3. **Índice explícito en todas las FKs.** PostgreSQL no crea automáticamente índice sobre las columnas de clave foránea; el modelo los declara de forma explícita en cada entidad.
4. **Índices GIN en los tres campos JSONB.** `Usuario.preferencias`, `Producto.ingredientes` y `Rol.permisos` llevan índice GIN para consultas dentro del JSON.
5. **Índices parciales/cobertura en paths calientes.** Se declaran sobre estados activos y rangos de fechas (tableros operativos, alertas de bajo stock, reportes de período); no se indexa de forma especulativa.
6. **Sin soft-delete en entidades de vida.** El campo `estado` cubre el ciclo (pedido, nómina, orden de compra, pago, reserva); `activo` (o `activa`) existe únicamente en entidades de catálogo (Usuario, Sucursal, CategoriaProducto, Producto, Rol, Proveedor, Oferta).
7. **Sin particionamiento ni sharding en V1.0.** Se revisará únicamente si alguna tabla supera ~100M de filas.
8. **Seguridad por capas.** La opacidad del UUID v7 es una defensa en profundidad; el control de acceso real vive en la API (autenticación + autorización por rol, `NFR-ST-02`, `CU-AD-03`), con **RLS (Row-Level Security)** en la base de datos como segunda capa.

[^checkpoint]: **Nota al pie (checkpoint C1).** La línea base de entidades y atributos fue aprobada por el usuario en el checkpoint del **04/10/2026** (spec 002, criterio C1), ejecutado **en una sola sesión con el usuario** (no por lotes). Las 26 entidades y sus atributos documentados en este documento son la línea base acordada en ese checkpoint, codificada en el **Apéndice A** de `specs/002-requisitos_admin_y_analysis/spec.md` y registrada en el checklist de la tarea T3 de `specs/002-requisitos_admin_y_analysis/tasks.md`.
