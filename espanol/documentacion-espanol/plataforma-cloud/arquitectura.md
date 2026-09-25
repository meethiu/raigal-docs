---
description: "Arquitectura, tratamiento de datos, integración con GitHub App y controles de seguridad en Raigal Cloud."
icon: network-wired
---

# Arquitectura y seguridad de la plataforma

Raigal Cloud (`https://app.raigal.dev`) es la aplicación web y servicio de coordinación de Raigal. Gestiona comprobaciones de pull requests, organiza incidencias en tableros de remediación, autentica a los usuarios y firma licencias fuera de línea.

## Topología del sistema

```mermaid
graph TD
    subgraph Entornos de Cómputo del Cliente
        Dev["Puesto Local: raigal scan / fix"]
        Agent["Agente Autónomo: raigal agent --pr"]
        CI["GitHub Actions / Runners de CI"]
    end

    subgraph Nube de GitHub
        GHApp["GitHub App Oficial (raigal-cloud)"]
        GHChecks["API de Comprobaciones y Estados de GitHub"]
    end

    subgraph Plano de Control Raigal Cloud (app.raigal.dev)
        Gateway["Pasarela de API REST (HTTPS / TLS 1.3)"]
        Auth["Identidad Clerk y Sincronización de Organizaciones"]
        OIDC["Verificador OIDC de GitHub Actions"]
        CheckManager["Orquestador de Check Runs y Estados"]
        LeaseSigner["Firmador Criptográfico Ed25519"]
        Store[("Almacén de Metadatos Multi-Inquilino Cifrado")]
    end

    subgraph Interfaz de Gestión del Cliente
        Board["Tablero Kanban de Remediación en 4 Etapas"]
        Drawers["Paneles Laterales de Inspección Contextual"]
        Telemetry["Historial de Ejecuciones y Auditoría de Diffs"]
        Settings["Gobernanza de Tokens y Repositorios"]
    end

    Dev -- "raigal login (Flujo OAuth Device)" --> Auth
    Dev -- "POST /v1/runs (Metadatos y Puntuación)" --> Gateway
    Agent -- "raigal agent --pr (PR y Estadísticas de Diff)" --> Gateway
    CI -- "X-GitHub-OIDC o RAIGAL_TOKEN" --> Gateway
    GHApp -- "Webhook (Eventos de PR e Instalación)" --> Gateway

    Gateway --> Store
    Gateway --> LeaseSigner
    Gateway --> CheckManager
    CheckManager -- "Crear / Resolver Checks" --> GHChecks

    Store --> Board
    Store --> Drawers
    Store --> Telemetry
    Auth --> Settings
```

## Tratamiento del código fuente

El código fuente del cliente nunca se envía, procesa ni almacena en los servidores de Raigal Cloud.

Separación entre cómputo local y servicios en la nube:
1. Ejecución local y en runners: el análisis sintáctico (AST), linters (oxlint, ruff, biome), análisis de complejidad y detección de patrones se ejecutan íntegramente en su máquina o runner de CI.
2. Envío exclusivo de metadatos: al terminar el escaneo, solo se transmiten resúmenes diagnósticos:
   - Puntuación final (0-100) y desglose por categorías
   - Identificadores de reglas (como `ai-slop/trivial-comment` o `security/no-eval`)
   - Rutas relativas de archivos y números de línea
   - Métricas de diff (archivos modificados, líneas añadidas y eliminadas)
3. Depuración previa de secretos: el CLI elimina cabeceras de autorización, tokens, claves privadas y patrones de credenciales antes de enviar los datos.

## Integración con la GitHub App

Raigal se conecta con GitHub mediante la app oficial `raigal-cloud`:

- Comprobaciones basadas en eventos: al abrir o actualizar un pull request, GitHub notifica a Raigal Cloud, que crea un check pendiente `"Raigal Quality Gate"` en el commit head.
- Resolución de comprobaciones: tras la evaluación en CI, el runner envía la puntuación a Raigal Cloud. Raigal marca el check como aprobado o rechazado, resolviendo las reglas de protección de rama.
- Eventos de ciclo de vida: las fusiones y cierres se reciben mediante webhooks de GitHub, manteniendo el tablero actualizado sin scripts adicionales.
- Limpieza de comprobaciones pendientes: si un trabajo de CI se cancela o interrumpe, Raigal Cloud elimina comprobaciones huérfanas tras 20 minutos para no bloquear pull requests.

## Autenticación y control de accesos

### Aislamiento multi-inquilino
Raigal Cloud aísla los datos de cada organización mediante Clerk. Repositorios, ejecuciones, hallazgos y tokens de API se particionan por identificador de organización.

### OIDC en GitHub Actions
Para flujos de CI, los runners se autentican con tokens efímeros:
- El runner solicita un token OIDC a `token.actions.githubusercontent.com`.
- El runner envía el token en la cabecera `X-GitHub-OIDC`.
- Raigal verifica la firma y valida que el repositorio pertenece a su organización.
- Tras la validación, se concede permiso temporal para registrar esa ejecución concreta.

### Autenticación del CLI (RFC 8628)
Los desarrolladores inician sesión con `raigal login`. El comando utiliza el flujo OAuth 2.0 Device Authorization (RFC 8628), abriendo una ventana del navegador para autorizar el equipo.

### Licencias fuera de línea (Ed25519)
Para escanear sin conexión de red continua:
- Al autenticarse, Raigal Cloud emite una licencia firmada con Ed25519.
- El CLI verifica la firma en local con una clave pública incluida en el ejecutable.
- La licencia es válida durante 72 horas, lo que permite realizar escaneos sin conexión a internet.

## Gestión de repositorios

Los administradores gestionan los repositorios desde **Configuración** > **Repositorios**:
- Las claves de API se almacenan con hash SHA-256; los valores en texto plano nunca se guardan en la base de datos.
- Al eliminar un repositorio, se borran en cascada todos sus registros históricos de ejecuciones, tarjetas de pull request y hallazgos.
