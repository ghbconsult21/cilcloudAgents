---
name: orquestar-feature
description: >-
  Orquesta funcionalidades o correcciones que cruzan múltiples repositorios de CICLOUD
  (CIcloudMapearDatabase, CicloudBE, CIcloudFEAngular y/o cicloud_movil). Activar cuando
  un requerimiento involucra 2 o más repositorios o requiere coordinar BD, API, clientes y seguridad.
---

# Orquestación Multi-Repo CICLOUD

Procedimiento estándar para coordinar subagentes minimizando consumo de tokens y evitando retrabajo por desajustes de contrato.

## 1. Análisis Inicial (Orquestador)
1. Identificar qué repositorios se ven afectados (no involucrar los que no cambian).
2. Definir en 5–10 líneas el contrato objetivo: entidad/tabla, endpoint HTTP (`método + ruta`), DTO Request/Response y validaciones clave.
3. Si el cambio es trivial (1–3 archivos en un solo repo), ejecutarlo directamente sin lanzar subagentes.

## 2. Orden de Ejecución por Dependencias
1. **Fase BD (opcional)** → Invocar `cicloud-db` solo si hay tablas, vistas o columnas nuevas.
   - Esperar su resumen con entidades/propiedades actualizadas en `CicloudBE/DbModel/`.
2. **Fase Backend** → Invocar `cicloud-backend` pasando el resumen de BD (si hubo) y el contrato objetivo.
   - Esperar su retorno con la firma exacta del endpoint y los DTOs (`Model/RequestResponse/`).
3. **Fase Clientes (en paralelo)** → Con el contrato exacto del backend, invocar en paralelo solo los clientes afectados:
   - `cicloud-frontend` (Angular 20 / PrimeNG / Sakai)
   - `cicloud-movil` (KMP / Compose / Ktor)
   - Pasar en cada prompt la ruta exacta del endpoint y los campos del DTO para evitar que lean código de `CicloudBE/`.
4. **Fase Validación (en paralelo)** → Cuando los clientes terminen:
   - Invocar `cicloud-seguridad` sobre el diff o archivos modificados si hubo cambios de auth, multitenencia, permisos, datos sensibles o nuevos endpoints.
   - Invocar `cicloud-qa` si hubo cambios de contrato entre BE/FE/Móvil o se requieren pruebas/verificación integral.

## 3. Reglas de Cierre
- Si `cicloud-seguridad` reporta hallazgos `CRÍTICO` o `ALTO`, reutilizar el subagente del repo responsable para corregirlos antes de cerrar.
- Entregar al usuario un resumen consolidado: cambios por repo, estado de compilaciones/pruebas y hallazgos de seguridad/QA resueltos o pendientes.
