---
description: "Arquitectura empresarial, garantía de cero exfiltración de código, modelo híbrido de GitHub App y controles de seguridad en Raigal Cloud."
icon: network-wired
---

# Arquitectura y Seguridad de la Plataforma

Raigal Cloud (`https://app.raigal.dev`) opera como el plano de control centralizado de calidad de código para equipos de ingeniería de alto rendimiento. Coordina puertas de calidad automatizadas en Pull Requests, remedia patrones de AI slop, impulsa tableros de control para desarrolladores y emite licencias criptográficas fuera de línea, adhiriéndose estrictamente al modelo de seguridad de **Cero Exfiltración de Código**.

---

## Topología de Alto Nivel

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

---

## La Garantía de Cero Exfiltración de Código

Los equipos de ingeniería empresariales requieren la certeza absoluta de que la propiedad intelectual, los secretos comerciales y el código fuente propietario se mantienen en estricta confidencialidad.

{% hint style="success" %}
**Cero Exfiltración de Código:** Raigal está diseñado específicamente para que **ningún fragmento de código fuente del cliente sea transmitido, procesado ni almacenado en los servidores de Raigal Cloud.**
{% endhint %}

### Separación entre Entorno de Cómputo y Plano de Control:
1. **Ejecución Local y en el Runner**: Todo el análisis sintáctico de AST, linters (oxlint, ruff, biome, etc.), motores de complejidad y algoritmos de detección de AI slop se ejecutan **exclusivamente dentro de su entorno de cómputo** (su terminal local o su runner de GitHub Actions).
2. **Ingesta Exclusiva de Metadatos**: Al finalizar el escaneo, solo se envían metadatos agregados a Raigal Cloud:
   - Puntuación final de calidad (0–100) y desglose por categorías.
   - Identificadores de reglas violadas (p. ej. `ai-slop/trivial-comment`, `security/no-eval`).
   - Rutas relativas de archivos y números de línea afectados.
   - Estadísticas de diferencias (número de archivos modificados, líneas añadidas y eliminadas).
3. **Depuración de Secretos en Origen**: Antes de transmitir la telemetría, el escáner del CLI de Raigal detecta y ofusca automáticamente encabezados de autorización, tokens, claves privadas y patrones de credenciales.

---

## Integración Híbrida con la GitHub App

Raigal se conecta nativamente con GitHub a través de la GitHub App oficial **`raigal-cloud`** (`https://github.com/apps/raigal-cloud`):

- **Comprobaciones Guiadas por Eventos**: Cuando un desarrollador abre o actualiza un Pull Request, GitHub notifica a Raigal Cloud mediante un webhook seguro. Raigal registra de inmediato una comprobación pendiente `"Raigal Quality Gate"` en el SHA del commit.
- **Resolución con Tolerancia a Fallos (Fail-Open)**: Al finalizar la evaluación en CI, el runner envía la puntuación y las métricas a Raigal Cloud. Raigal resuelve inmediatamente el estado de la comprobación en GitHub (aprobado o rechazado), desbloqueando las reglas de protección de rama.
- **Sincronización Automática del Ciclo de Vida**: Las fusiones y cierres se reportan directamente a través de los eventos de GitHub, manteniendo el tablero Kanban siempre actualizado en tiempo real sin necesidad de scripts manuales de limpieza.
- **Limpieza de Comprobaciones Huérfanas**: Si un trabajo de CI se cancela o falla abruptamente, Raigal Cloud limpia de forma automática comprobaciones pendientes tras 20 minutos para evitar bloqueos permanentes.

---

## Autenticación y Control de Accesos

### 1. Aislamiento Multi-Inquilino de Organizaciones
Raigal Cloud aplica límites estrictos de aislamiento entre organizaciones mediante la infraestructura de identidad empresarial de Clerk. Cada repositorio, ejecución, hallazgo y token de API está estrictamente aislado para su organización.

### 2. OIDC sin Secretos para GitHub Actions
Para flujos de trabajo empresariales, Raigal elimina los riesgos de los tokens estáticos:
- El runner de GitHub Actions solicita un token OIDC efímero firmado por GitHub (`token.actions.githubusercontent.com`).
- El runner envía el token a Raigal Cloud mediante el encabezado `X-GitHub-OIDC`.
- Raigal verifica la firma criptográfica y valida que el propietario del repositorio figure en la lista autorizada de su organización.
- Si la validación es satisfactoria, se otorgan derechos temporales de ingesta para esa ejecución específica.

### 3. Autenticación del CLI para Desarrolladores (RFC 8628)
Los desarrolladores autentican sus máquinas locales de manera intuitiva ejecutando `raigal login`. Esto inicia un flujo de autorización de dispositivos OAuth 2.0 (RFC 8628) en el navegador sin necesidad de copiar tokens a mano.

### 4. Concesiones Criptográficas Fuera de Línea (Ed25519)
Para evitar que la latencia de red afecte el trabajo diario y permitir escaneos sin conexión:
- Al autenticarse, Raigal Cloud emite un token de licencia firmado criptográficamente con una clave privada Ed25519.
- El CLI local valida la firma sin conexión utilizando una clave pública integrada en el ejecutable.
- Las concesiones tienen una validez de 72 horas, permitiendo escanear y corregir código durante viajes o en redes aisladas.

---

## Gobernanza de Datos y Ciclo de Vida de Repositorios

Raigal Cloud proporciona a los administradores control total sobre sus datos:

- **Tokens con Hash Seguro**: Las claves de API se almacenan con hash criptográfico SHA-256; los valores en texto claro nunca se guardan en bases de datos.
- **Eliminación de Repositorios Bajo Demanda**: Los administradores pueden eliminar repositorios en cualquier momento desde **Configuración** > **Repositorios**. La eliminación se propaga en cascada en la base de datos, purgando permanentemente ejecuciones históricas, tarjetas de PR y hallazgos asociados.
