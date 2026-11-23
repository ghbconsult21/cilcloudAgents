# CICLOUD — Reglas del Ecosistema y Eficiencia de Tokens

Reglas activas para todos los agentes que operan sobre el ecosistema CICLOUD. Mantener este archivo breve (< 120 líneas); cualquier procedimiento extenso o checklist debe vivir en `skills/<nombre>/SKILL.md` para cargarse solo bajo demanda (progressive disclosure).

## 1. Mapa del Ecosistema (evitar exploración innecesaria)
| Carpeta | Stack | Guía local |
| :--- | :--- | :--- |
| `CicloudBE/` | .NET 8, C# 12, EF Core 8, SQL Server multitenant | `CicloudBE/AGENTS.md` |
| `CIcloudFEAngular/` | Angular 20 (Standalone), PrimeNG 20, TailwindCSS 4 | `CIcloudFEAngular/AGENTS.md` |
| `cicloud_movil/` | Kotlin Multiplatform, Compose Multiplatform, Ktor | `cicloud_movil/AGENTS.md` |
| `CIcloudMapearDatabase/` | .NET Scaffold de entidades (`CICLOUD`, `GhbClientes`) | `CIcloudMapearDatabase/AGENTS.md` |

## 2. Reglas Obligatorias para Reducir el Consumo de Tokens

### A. Lectura quirúrgica de archivos
- **Buscar antes de abrir**: localizar símbolos, rutas o referencias con `rg`/`grep` o `find` acotado antes de leer archivos.
- **Leer por rangos**: en archivos medianos o grandes (`_CicloudContext.cs`, `Program.cs`, perfiles AutoMapper, páginas Angular o ViewModels largos), leer solo las líneas relevantes (`StartLine`/`EndLine`), nunca el archivo completo.
- **Zonas prohibidas para lectura o escaneo masivo**:
  - Dependencias y compilados: `node_modules/`, `bin/`, `obj/`, `dist/`, `.angular/`, `build/`, `.gradle/`, `.git/`, `package-lock.json`, binarios y archivos minificados.
  - Scaffolds masivos: **nunca** listar ni leer en bloque `CicloudBE/DbModel/CICLOUD/` ni `CIcloudMapearDatabase/MapearBaseDatos/CICLOUD/`; abrir únicamente la entidad exacta involucrada.
- **No releer lo que ya está en contexto** salvo que el archivo haya cambiado.

### B. Salida de terminal acotada
- Usar modos compactos al compilar o probar:
  - .NET: `dotnet build --verbosity minimal` / `dotnet test --verbosity minimal`
  - Gradle: `./gradlew :composeApp:assembleDebug -q`
  - Git: ejecutar `git status -s` y `git diff --stat` antes de pedir un `git diff` detallado (y acotar el diff a archivos concretos).
- Truncar salidas largas de comandos de exploración (`| head -n 40` o `| tail -n 50`) para no inundar la ventana de contexto.

### C. Delegación y subagentes sin desperdicio
- **No sobre-orquestar**: cambios simples (1–3 archivos en un solo repositorio) se resuelven directamente sin lanzar subagentes.
- **Delegación dirigida**: en tareas medianas/grandes o multi-repo, invocar únicamente a los subagentes de los repos afectados, pasando en el prompt las rutas exactas, endpoints y DTOs ya identificados para evitar que re-exploren el código desde cero.
- **Reutilizar subagentes**: para correcciones o seguimientos del mismo dominio, continuar con el subagente existente en lugar de crear uno nuevo.
- **Retorno compacto obligatorio**: todo subagente debe devolver un resumen estructurado breve (máx. 15–20 líneas) con: archivos modificados, cambios de contrato (API/DTO/BD), resultado de build/validación y riesgos pendientes. Nunca pegar archivos completos en su respuesta.

### D. Edición y respuestas sin relleno
- Preferir ediciones puntuales (`edit_file`) sobre reescrituras completas (`write_file`).
- No imprimir archivos completos ni bloques sin cambios en las respuestas al usuario.
- No repetir reglas ya cargadas ni explicaciones redundantes; priorizar listas concisas y enlaces a archivos.
- En sesiones largas al cambiar de tarea o fase, sugerir `/compact` para liberar ventana de contexto.

## 3. Orquestación y Subagentes
El agente principal del workspace actúa como **Orquestador** del ecosistema CICLOUD: planifica, define contratos y delega según dominio.

| Subagente | Dominio | Cuándo invocarlo |
| :--- | :--- | :--- |
| `cicloud-db` | `CIcloudMapearDatabase/`, `CicloudBE/DbModel/` | Nuevas tablas, vistas, columnas o sincronización de scaffold |
| `cicloud-backend` | `CicloudBE/` | Endpoints, lógica `Bussnies`, repositorios, DTOs, reportes |
| `cicloud-frontend` | `CIcloudFEAngular/` | Páginas Angular 20, PrimeNG, servicios HTTP, modelos TS, claro/oscuro |
| `cicloud-movil` | `cicloud_movil/` | Pantallas Compose, ViewModels, Ktor, Koin, modelos `@Serializable` |
| `cicloud-seguridad` | Ecosistema (solo lectura) | Auditoría de multitenencia (`Institucion`), JWT, OWASP, `DbModel`, secretos |
| `cicloud-qa` | Ecosistema (pruebas/verificación) | Consistencia de contratos BE/FE/Móvil, pruebas unitarias y validaciones |

- **Orden multi-repo**: `cicloud-db` → `cicloud-backend` → `cicloud-frontend` + `cicloud-movil` (en paralelo) → `cicloud-seguridad` / `cicloud-qa`.
- Para flujos detallados multi-repo, auditorías o alineación de DTOs, activar la skill correspondiente (`orquestar-feature`, `auditoria-seguridad`, `sincronizar-contratos`).
