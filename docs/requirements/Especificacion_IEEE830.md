# Especificación de Requerimientos del Software Mizu
**Versión:** 1.0  
**Autor:** Juan Camilo Garcia Gomez  
**Fecha:** Marzo 2026

---

## Introducción
El proyecto **Mizu** es un ecosistema digital para el sector gastronómico compuesto por cinco subproyectos: **Mizu Experience (web)**, **Mizu Go (app móvil)**, **Mizu Order Hub (escritorio)**, **Mizu Stock (escritorio)** y **Mizu Admin (escritorio)**.  
Este documento contiene la especificación de **requerimientos funcionales** (Casos de Uso CU) y **requerimientos no funcionales** (NFR) para la Versión 1.0. Se mantiene como documento vivo y deberá actualizarse conforme avance el desarrollo.

---

## Control de versiones
| **Versión** | **Fecha** | **Autor** | **Descripción** |
|---|---:|---|---|
| 1.0 | 29/03/2026 | Juan Camilo Garcia Gomez | Documento inicial con requisitos funcionales y no funcionales por subproyecto. |

---

## Requerimientos por subproyecto

### Mizu Experience Web

**Casos de uso funcionales CU**
- **CU-EX-01** Consultar menú  
- **CU-EX-02** Realizar reserva desde la web  
- **CU-EX-03** Hacer pedido en línea con confirmación automática  
- **CU-EX-04** Generar historial de pedidos del cliente  

**Requisitos no funcionales NFR**
- **NFR-EX-01** Interfaz adaptable (responsive) para móviles y escritorio  
- **NFR-EX-02** Seguridad en transacciones (HTTPS y cifrado de datos sensibles)  
- **NFR-EX-03** Disponibilidad 24/7  

---

### Mizu Go App móvil

**Casos de uso funcionales CU**
- **CU-GO-01** Registro y autenticación de clientes  
- **CU-GO-02** Realizar pedidos y reservas desde la app  
- **CU-GO-03** Mostrar promociones personalizadas  
- **CU-GO-04** Sistema de puntos y recompensas  

**Requisitos no funcionales NFR**
- **NFR-GO-01** Compatibilidad con Android e iOS  
- **NFR-GO-02** Notificaciones push  
- **NFR-GO-03** Tiempo de respuesta < 2 segundos en endpoints críticos  

---

### Mizu Order Hub Escritorio

**Casos de uso funcionales CU**
- **CU-OH-01** Recepción de pedidos en tiempo real  
- **CU-OH-02** Registro manual de pedidos en el establecimiento  
- **CU-OH-03** Gestión de estado de pedidos (pendiente, preparación, entregado)  
- **CU-OH-04** Generación de reportes diarios de pedidos  

**Requisitos no funcionales NFR**
- **NFR-OH-01** Interfaz sencilla e intuitiva para empleados  
- **NFR-OH-02** Integración con Mizu Stock  
- **NFR-OH-03** Respaldo automático cada 24 horas  

---

### Mizu Stock Escritorio

**Casos de uso funcionales CU**
- **CU-ST-01** Registrar entradas y salidas de inventario  
- **CU-ST-02** Gestión de proveedores  
- **CU-ST-03** Generar alertas de bajo stock  
- **CU-ST-04** Reportes de consumo de ingredientes  

**Requisitos no funcionales NFR**
- **NFR-ST-01** Escalabilidad para múltiples sucursales  
- **NFR-ST-02** Roles de acceso diferenciados (administrador, empleado)  
- **NFR-ST-03** Disponibilidad offline con sincronización  

---

### Mizu Admin Escritorio

**Casos de uso funcionales CU**
- **CU-AD-01** Registrar ingresos y egresos  
- **CU-AD-02** Generar reportes contables y gráficos  
- **CU-AD-03** Control y gestión de usuarios administradores  
- **CU-AD-04** Exportación de datos a Excel/PDF  

**Requisitos no funcionales NFR**
- **NFR-AD-01** Cumplimiento normativo contable  
- **NFR-AD-02** Alta confiabilidad en almacenamiento  
- **NFR-AD-03** Interfaz amigable para análisis rápido  

---

## Mapeo CU a Historias de Usuario HU sugeridas
| **CU** | **HU Issue ID sugerido** | **Descripción breve** |
|---|---|---|
| CU-EX-01 | HU-EX-01 | Consultar menú (3 pts) |
| CU-EX-02 | HU-EX-02 | Realizar reserva (4 pts) |
| CU-EX-03 | HU-EX-03 | Hacer pedido en línea (5 pts) |
| CU-GO-01 | HU-GO-01 | Registro de usuario (3 pts) |
| CU-GO-02 | HU-GO-02 | Pedido desde la app (5 pts) |
| CU-OH-01 | HU-OH-01 | Ver pedidos en tiempo real (4 pts) |
| CU-ST-01 | HU-ST-01 | Registrar entrada de inventario (4 pts) |
| CU-AD-01 | HU-AD-01 | Registrar ingreso financiero (4 pts) |

## Conclusión
Este documento consolida los requerimientos funcionales y no funcionales del ecosistema Mizu y proporciona el mapeo necesario para convertir la especificación en trabajo rastreable en GitHub. Mantén este archivo actualizado y registra cada cambio en la sección de Control de versiones.
