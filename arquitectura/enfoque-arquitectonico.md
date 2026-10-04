# Enfoque arquitectónico de Closer

**Enfoque seleccionado: Clean Architecture (Arquitectura Limpia).** Se aplica al backend modular para separar reglas y casos de uso de la presentación, la persistencia y los proveedores de IA.

El estilo describe módulos y despliegue; el enfoque define la organización del código y sus dependencias dentro de esos módulos.

## Diagrama de dependencias

```mermaid
flowchart TB
    subgraph B["BACKEND CLOSER · ORGANIZACIÓN INTERNA"]
        subgraph EXT["CAPAS EXTERNAS · DETALLES TECNOLÓGICOS"]
            P["PRESENTACIÓN<br/>Controladores REST · DTOs<br/>Validación y contexto del usuario"]
            I["INFRAESTRUCTURA<br/>Repositorios Supabase · Adaptadores Gemini<br/>Identidad · Pagos · Notificaciones"]
        end
        subgraph APP["APLICACIÓN · ORQUESTACIÓN"]
            C["CASOS DE USO<br/>IniciarSimulacion · FinalizarSesion<br/>EvaluarSesion · ConsultarProgreso"]
            PU["PUERTOS / CONTRATOS<br/>RepositorioSesiones · RepositorioEvaluaciones<br/>EvaluadorConversacion · ProveedorVoz"]
        end
        subgraph DOM["DOMINIO · NÚCLEO DEL NEGOCIO"]
            E["ENTIDADES Y VALORES<br/>SesionPractica · Escenario · Prospecto<br/>Evaluacion · CuotaDeUso"]
            R["REGLAS<br/>Transiciones válidas · Cuota disponible<br/>Evaluar solo sesiones finalizadas"]
        end
        P -->|Invoca| C
        C -->|Utiliza| PU
        C -->|Opera entidades| E
        E --> R
        I -.->|Implementa contratos| PU
        I -->|Mapea datos| E
    end
    T["TECNOLOGÍAS EXTERNAS<br/>Supabase · Google GenAI SDK<br/>APIs de pagos y notificaciones"]
    I -->|Utiliza| T

    classDef presentacion fill:#DBEAFE,stroke:#2563EB,color:#1E3A8A,stroke-width:2px
    classDef aplicacion fill:#CCFBF1,stroke:#0D9488,color:#134E4A,stroke-width:2px
    classDef dominio fill:#FEF3C7,stroke:#D97706,color:#78350F,stroke-width:2px
    classDef infra fill:#EDE9FE,stroke:#7C3AED,color:#4C1D95,stroke-width:2px
    classDef externo fill:#F1F5F9,stroke:#64748B,color:#0F172A
    class P presentacion
    class C,PU aplicacion
    class E,R dominio
    class I infra
    class T externo
    style B fill:#F8FAFC,stroke:#334155,stroke-width:2px
    style EXT fill:#F5F3FF,stroke:#C4B5FD
    style APP fill:#F0FDFA,stroke:#5EEAD4
    style DOM fill:#FFFBEB,stroke:#FBBF24
```

**Lectura:** las flechas continuas representan dependencias de código y la discontinua, implementación de contratos. El dominio no depende de las capas externas. Los puertos están en aplicación porque expresan las necesidades de los casos de uso; infraestructura los implementa.

La composición del backend conecta contratos y adaptadores al iniciar la aplicación. Durante la ejecución, el caso de uso llama al adaptador mediante su contrato; su código no depende del cliente de Supabase ni del SDK de Google.

## Responsabilidades

| Capa | Aplicación en Closer | Límite |
|---|---|---|
| Presentación | Recibir solicitudes REST, validar formato y trasladar el contexto autenticado. | No calcular evaluaciones ni acceder directamente a tablas. |
| Aplicación | Coordinar prácticas, comprobar propiedad de recursos y solicitar evaluación y persistencia mediante puertos. | No importar SDKs de proveedores. |
| Dominio | Representar sesiones, escenarios y evaluaciones; controlar estados y reglas de cuotas. | No conocer HTTP, React ni almacenamiento. |
| Infraestructura | Implementar repositorios e integraciones de IA, identidad, pagos y notificaciones. | No decidir reglas comerciales. |

La aplicación web Next.js/React consume la API y gestiona la interacción y el audio. Este diagrama desarrolla la organización interna del backend. Las capas son separaciones lógicas, no despliegues independientes, y se aplican dentro de cada módulo.

## Ejemplo: finalizar y evaluar una sesión

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuario
    participant W as Aplicación web
    participant P as API REST
    participant A as Casos de uso
    participant D as Dominio
    participant R as Repositorios / Supabase
    participant G as Evaluador / Gemini

    U->>W: Finalizar práctica
    W->>W: Detener micrófono y cerrar canal de voz
    W->>P: Solicitar finalización
    P->>A: FinalizarSesion(usuario, sesión)
    A->>R: Recuperar sesión y conversación registrada
    R-->>A: Sesión y transcripción
    A->>A: Verificar propiedad de la sesión
    A->>D: Validar transición de estado
    D-->>A: Sesión finalizada
    A->>R: Guardar cierre y evaluación pendiente
    A-->>P: Finalización registrada
    P-->>W: Sesión finalizada
    W->>P: Solicitar evaluación
    P->>A: EvaluarSesion(usuario, sesión)
    A->>R: Recuperar sesión y evaluación existente
    A->>A: Verificar propiedad y evitar duplicados
    A->>D: Validar sesión finalizada
    A->>G: Evaluar conversación mediante contrato
    alt Evaluación disponible
        G-->>A: Resultado normalizado
        A->>D: Validar estructura y valores
        A->>R: Guardar métricas y recomendaciones
        A-->>P: Evaluación disponible
        P-->>W: Resultados
        W-->>U: Radar, métricas y feedback
    else Proveedor no disponible
        G-->>A: Error de integración
        A->>R: Registrar fallo recuperable
        A-->>P: Sesión conservada, evaluación no disponible
        P-->>W: Estado y opción de reintento
        W-->>U: Mostrar estado de evaluación
    end
```

Este diagrama representa ejecución, no dependencias de código. Repositorios y evaluador son adaptadores invocados a través de puertos. La transcripción se registra durante la práctica o al cerrarla. La evaluación ocurre después del cierre, separada del audio.

Ante un reintento, se reutiliza una evaluación completada. La coordinación de solicitudes simultáneas debe protegerse mediante persistencia transaccional para evitar duplicados. El ejemplo desarrolla el flujo autorizado; las solicitudes sin identidad válida, sin propiedad o con estados incompatibles se rechazan antes de modificar datos o invocar IA.

## Justificación

| Elemento | Aplicación a Closer |
|---|---|
| Problema resuelto | Evitar que cambios en Gemini o Supabase se propaguen a las reglas de sesiones, cuotas y evaluación. |
| DA06 · Mantenibilidad | Cambiar adaptadores conservando contratos y casos de uso cuando su comportamiento siga siendo compatible. |
| DA04 · Integración con IA | Encapsular solicitudes, respuestas y errores del proveedor. |
| DA03 · Seguridad | Comprobar propiedad y permisos en los casos de uso, complementando autenticación y RLS. |
| DA07 · Persistencia | Consultar y guardar mediante repositorios, sin clientes de base de datos en el dominio. |
| Beneficios | Probar reglas con implementaciones simuladas, localizar cambios técnicos y delimitar responsabilidades. |
| Compromisos | Introduce contratos y conversiones de datos; deben responder a necesidades reales. |

## Documentos relacionados

- [Estilo arquitectónico](estilo-arquitectonico.md)
- [Arquitectura inicial](arquitectura-inicial.md)
- [Drivers arquitectónicos](../analisis-de-sistema/06-driver-arquitectonicos.md)
