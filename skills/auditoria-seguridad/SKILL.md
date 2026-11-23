---
name: auditoria-seguridad
description: >-
  Ejecuta una auditoría de seguridad enfocada en los riesgos reales de CICLOUD
  (multitenencia Institucion, JWT, autorización por roles/establecimiento, exposición de DbModel,
  inyección SQL, XSS y secretos). Activar cuando el usuario pida revisar seguridad o antes de liberar cambios críticos.
---

# Auditoría de Seguridad CICLOUD

## 1. Acotar el Alcance (ahorro de tokens)
- Si se audita un cambio reciente, obtener primero `git status -s` y `git diff --stat` de los repos afectados.
- Si se audita un módulo específico, localizar sus archivos exactos con `rg` antes de delegar.
- Delegar el análisis al subagente de solo lectura `cicloud-seguridad` entregándole la lista exacta de archivos o módulo a revisar.

## 2. Checklist Específico CICLOUD
1. **Multitenencia (`CicloudBE`)**:
   - Resolución de BD vía cabecera `Institucion` y `API/dataBase.json`; cero nombres de BD hardcodeados.
   - Sin estado estático/caché en memoria compartido entre instituciones sin clave de tenant.
2. **AuthN / AuthZ**:
   - Controllers y acciones protegidos con `[Authorize]` salvo login/endpoints públicos justificados.
   - Verificación de permisos (`FormPermissions`, usuario/establecimiento/área) tomada de claims JWT y no solo de parámetros del request (prevenir IDOR/BOLA).
3. **Datos y Consultas**:
   - Ningún Controller devuelve ni recibe entidades directas de `DbModel` (usar siempre DTOs de `Model/RequestResponse/`).
   - Sin SQL concatenado (`FromSqlRaw`, `ExecuteSqlRaw`, `SqlCommand` sin parámetros).
   - Manejo de errores sin filtrar stack traces ni cadenas de conexión al cliente.
4. **Frontend (`CIcloudFEAngular`) y Móvil (`cicloud_movil`)**:
   - Sin `innerHTML` ni `bypassSecurityTrust*` con datos no sanitizados.
   - Sin secretos/tokens hardcodeados en `enviremoment/`, `environment.*.properties` ni logs.
   - En móvil: modelos `@Serializable` consistentes y sin exposición de tokens en logs de depuración.

## 3. Salida Esperada
- Veredicto (`APROBADO`, `APROBADO CON OBSERVACIONES` o `BLOQUEADO`) y tabla breve de hallazgos con severidad, `archivo:línea` y mitigación concreta.
