# Estilo arquitectónico de Closer

**Estilo seleccionado: monolito modular.** El backend se divide por responsabilidades de negocio y se despliega como una sola aplicación. La interfaz web y los proveedores administrados quedan fuera de ese límite.

## Diagrama del estilo

```mermaid
flowchart TB
    U["USUARIOS<br/>Vendedor · AE · SDR · Emprendedor"]
    A["ADMINISTRADOR<br/>Usuarios y planes"]
    W["APLICACIÓN WEB<br/>Next.js · React · TypeScript"]
    subgraph B["BACKEND CLOSER · MONOLITO MODULAR · UN DESPLIEGUE"]
        direction TB
        API["API REST<br/>Autenticación · Autorización · Validación"]
        subgraph M["MÓDULOS DE NEGOCIO"]
            UP["Usuarios y perfiles"]
            EP["Escenarios y prospectos"]
            SP["Sesiones de práctica"]
            EC["Evaluación y coaching"]
            HP["Historial y progreso"]
            PL["Planes y límites"]
        end
        I["ADAPTADORES DE INFRAESTRUCTURA<br/>Persistencia · IA · Identidad · Pagos · Notificaciones"]
        API --> UP & EP & SP & EC & HP & PL
        UP & EP & SP & EC & HP & PL --> I
    end
    subgraph S["SUPABASE · SERVICIOS ADMINISTRADOS"]
        AUTH["Auth<br/>Identidad · Google OAuth"]
        DB[("PostgreSQL<br/>Perfiles · Sesiones · Transcripciones<br/>Prospectos · Evaluaciones")]
        ST["Storage<br/>Archivos opcionales"]
    end
    subgraph X["INTEGRACIONES EXTERNAS"]
        VOZ["GEMINI · VOZ<br/>Prospecto virtual en tiempo real"]
        IA["GEMINI · ANÁLISIS<br/>Generación de escenarios y coaching"]
        OT["Pagos y notificaciones"]
    end
    U & A --> W
    W -->|HTTPS / REST| API
    W <-->|WSS · audio y eventos| VOZ
    W <-->|HTTPS · autenticación| AUTH
    I -->|Persistencia| DB
    I -->|Identidad| AUTH
    I -.->|Si se requieren archivos| ST
    I -->|Preparar acceso autorizado| VOZ
    I -->|API · fuera del flujo de audio| IA
    I --> OT

    classDef actor fill:#F1F5F9,stroke:#64748B,color:#0F172A
    classDef interfaz fill:#DBEAFE,stroke:#2563EB,color:#1E3A8A,stroke-width:2px
    classDef modulo fill:#CCFBF1,stroke:#0D9488,color:#134E4A,stroke-width:2px
    classDef infra fill:#EDE9FE,stroke:#7C3AED,color:#4C1D95,stroke-width:2px
    classDef datos fill:#FEF3C7,stroke:#D97706,color:#78350F
    classDef externo fill:#FFE4E6,stroke:#E11D48,color:#881337
    class U,A actor
    class W,API interfaz
    class UP,EP,SP,EC,HP,PL modulo
    class I infra
    class AUTH,DB,ST datos
    class VOZ,IA,OT externo
    style B fill:#F8FAFC,stroke:#334155,stroke-width:2px
    style M fill:#F0FDFA,stroke:#5EEAD4
    style S fill:#FFFBEB,stroke:#F59E0B
    style X fill:#FFF1F2,stroke:#FDA4AF
```

**Lectura:** azul = acceso; verde = negocio; violeta = adaptadores; ámbar = datos e identidad; rosa = proveedores. Las flechas indican comunicación, no dependencias de código. La línea discontinua representa almacenamiento opcional. Los bloques Gemini son capacidades externas, no microservicios propios.

## Módulos y responsabilidades

| Módulo | Responsabilidad | Requisitos |
|---|---|---|
| Usuarios y perfiles | Perfil comercial e identidad vinculada con Supabase Auth. | RF01–RF02 |
| Escenarios y prospectos | Producto, buyer persona, etapa y voz; generación y reutilización de perfiles. | RF03–RF06, RF15, RF17 |
| Sesiones de práctica | Inicio, estado, finalización y registro de la conversación. | RF07–RF09, RF14 |
| Evaluación y coaching | Métricas, competencias y recomendaciones posteriores. | RF10–RF13 |
| Historial y progreso | Consulta de resultados, evolución y eliminación de sesiones. | RF16–RF17 |
| Planes y límites | Cuotas y opciones de suscripción. | RF18 |

El soporte de RF19 se atiende mediante un caso de uso y el adaptador de notificaciones, sin exigir un módulo independiente.

## Reglas de organización

1. Los módulos comparten un despliegue y se comunican mediante contratos internos. Cada módulo controla sus datos; los demás consumen sus operaciones públicas.
2. El frontend gestiona micrófono, reproducción e indicadores. El backend verifica identidad, propiedad de recursos y cuota disponible.
3. La voz utiliza WSS con el proveedor y la gestión utiliza HTTPS/REST. El backend prepara el acceso autorizado; las credenciales permanentes permanecen en el servidor. El mecanismo concreto de acceso del navegador se verificará al implementar la integración.
4. La evaluación se ejecuta después del cierre de la conversación, fuera del flujo interactivo de audio. No se presupone un sistema de colas.
5. PostgreSQL está administrado por Supabase. Transcripciones y métricas son datos almacenados. La autorización se complementa con RLS cuando corresponda.

## Justificación y drivers

| Driver | Respuesta del diseño |
|---|---|
| DA01 · Baja latencia | Separar el streaming de voz de las operaciones de gestión y del análisis posterior. |
| DA02 · Concurrencia | Evitar retransmitir todo el audio por el backend; medir sesiones simultáneas, recursos y cuotas del proveedor. |
| DA03 · Seguridad | Verificar identidad, propiedad y cuotas; aplicar autorización y RLS. |
| DA04–DA05 · IA y comunicación | Adaptadores Google GenAI y separación entre WSS y HTTPS/REST. |
| DA06 · Mantenibilidad | Módulos con responsabilidades delimitadas e integraciones encapsuladas. |
| DA07 · Persistencia | PostgreSQL para información estructurada y Storage solo si se necesitan archivos. |

El monolito modular permite comenzar con una unidad de despliegue y una organización clara. Frente a microservicios, evita introducir inicialmente coordinación distribuida y despliegues independientes para cada módulo.

**Compromisos:** los módulos comparten recursos y despliegue; un fallo del backend puede afectar varias funciones. La disponibilidad también depende de los proveedores. La modularidad no garantiza rendimiento ni escalabilidad: se requieren mediciones.

## Documentos relacionados

- [Enfoque arquitectónico](enfoque-arquitectonico.md)
- [Arquitectura inicial](arquitectura-inicial.md)
- [Drivers arquitectónicos](../analisis-de-sistema/06-driver-arquitectonicos.md)
