# CICLOUD Agents (`cilcloudAgents`)

Plugin centralizado de Antigravity para coordinar el ecosistema multi-repositorio de **CICLOUD** (`CicloudBE`, `CIcloudFEAngular`, `cicloud_movil`, `CIcloudMapearDatabase`).

## Estructura
```text
cilcloudAgents/
├── plugin.json                     # Manifiesto del plugin
├── rules/
│   └── AGENTS.md                   # Reglas siempre activas (mapa, eficiencia de tokens y orquestación)
├── agents/                         # Subagentes especializados con ventana de contexto aislada
│   ├── cicloud-backend.md          # CicloudBE (.NET 8 / EF Core / multitenant)
│   ├── cicloud-frontend.md         # CIcloudFEAngular (Angular 20 / PrimeNG / Tailwind)
│   ├── cicloud-movil.md            # cicloud_movil (KMP / Compose / Ktor / Koin)
│   ├── cicloud-db.md               # CIcloudMapearDatabase y CicloudBE/DbModel
│   ├── cicloud-seguridad.md        # Auditoría de seguridad (solo lectura)
│   └── cicloud-qa.md               # Pruebas y paridad de contratos BE/FE/Móvil
└── skills/                         # Flujos detallados cargados solo bajo demanda
    ├── orquestar-feature/SKILL.md
    ├── auditoria-seguridad/SKILL.md
    └── sincronizar-contratos/SKILL.md
```

## Activación en el Workspace
Este plugin queda vinculado automáticamente al abrir `/home/flehuar/Proyectos/Cicloud` mediante `.agents/plugins.json`:

```json
{
  "entries": [
    {
      "path": ".",
      "include_only": ["cilcloudAgents"]
    }
  ]
}
```

## Principio de Eficiencia de Tokens
- **`rules/AGENTS.md`** se mantiene compacto (< 120 líneas) porque se inyecta en cada turno.
- **Procedimientos y checklists largos** viven en `skills/<nombre>/SKILL.md` (progressive disclosure).
- **Subagentes en `agents/`** aíslan la exploración de cada repo y devuelven únicamente resúmenes estructurados de máx. 20 líneas al Orquestador.
