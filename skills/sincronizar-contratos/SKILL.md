---
name: sincronizar-contratos
description: >-
  Sincroniza contratos API y DTOs entre CicloudBE (C# RequestResponse), CIcloudFEAngular
  (TypeScript models/services) y cicloud_movil (Kotlin @Serializable/Endpoints). Activar cuando
  cambie un endpoint, request o response y haya que alinear clientes sin romper serialización.
---

# Sincronización de Contratos API CICLOUD

## 1. Fuente de Verdad
- La fuente de verdad del contrato es `CicloudBE`:
  - Rutas y verbos HTTP: `CicloudBE/API/Controllers/`
  - DTOs: `CicloudBE/Model/RequestResponse/`
  - Mapeos: `CicloudBE/UtilGeneral/MappingProfiles/`

## 2. Procedimiento Compacto
1. Leer **únicamente** la acción del Controller y los DTOs implicados en `CicloudBE` (por rangos de líneas).
2. Extraer una tabla corta de contrato:
   - Ruta, verbo HTTP, query/path params, headers (`Authorization`, `Institucion`), JSON Request y JSON Response (respetando camelCase de serialización JSON en ASP.NET Core).
3. Delegar en paralelo (solo para los clientes afectados) entregando esa tabla de contrato:
   - `cicloud-frontend`: actualizar interfaces en `src/app/models/` y métodos en `src/app/service/` (sin `any` y sin headers manuales ya cubiertos por `auth-interceptor.ts`).
   - `cicloud-movil`: actualizar data classes `@Serializable` en `models/` (usando `@SerialName` si el nombre difiere) y rutas en `network/Endpoints.kt` / `repository/`.
4. Invocar `cicloud-qa` si el cambio afecta varios endpoints o tipos complejos (fechas, decimales, enums, nullables) para confirmar paridad y compilación.
