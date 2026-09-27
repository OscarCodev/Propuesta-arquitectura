# Arquitectura inicial del sistema
## Diagrama de arquitectura
```mermaid
flowchart LR
%% =========================
%% ACTORES
%% =========================
subgraph ACTORES["ACTORES"]
Vendedor["Vendedor"]
AE["AE"]
SDR["SDR"]
Emprendedor["Emprendedor"]
Admin["Administrador"]
end

%% =========================
%% PRESENTACIÓN
%% =========================
subgraph PRESENTACION["PRESENTACIÓN"]
Web["Aplicación web"]
Dashboard["Dashboard"]
Config["Configuración de escenarios"]
Llamada["Simulación de llamada"]
Resultados["Resultados y feedback"]
end

%% =========================
%% LÓGICA DE NEGOCIO
%% =========================
subgraph NEGOCIO["LÓGICA DE NEGOCIO"]
Usuarios["Usuarios y autenticación"]
Escenarios["Gestión de escenarios"]
Sesiones["Sesiones de práctica"]
Evaluacion["Motor de evaluación"]
Historial["Historial y progreso"]
Planes["Planes / facturación"]
end

%% =========================
%% DATOS
%% =========================
subgraph DATOS["DATOS"]
PostgreSQL["PostgreSQL"]
Supabase["Supabase"]
Storage["Storage"]
Transcripciones["Transcripciones"]
Metricas["Métricas"]
Prospectos["Biblioteca de prospectos"]
end

%% =========================
%% SISTEMAS EXTERNOS
%% =========================
subgraph EXTERNOS["SISTEMAS EXTERNOS"]
GenAI["Google GenAI SDK"]
Gemini["Gemini 2.5 Flash Native Audio"]
OAuth["Google OAuth 2.0"]
Pago["Pasarela de pago"]
Notificaciones["Servicio de correo / notificaciones"]
end

%% =========================
%% FLUJO PRINCIPAL
%% =========================
ACTORES -->|HTTPS| PRESENTACION
PRESENTACION -->|HTTPS / REST| NEGOCIO
PRESENTACION -->|WSS / voz| EXTERNOS
NEGOCIO -->|CRUD| DATOS
NEGOCIO -->|Integración API| EXTERNOS
EXTERNOS -->|Resultados / eventos| NEGOCIO

%% =========================
%% DISTRIBUCIÓN
%% =========================
Vendedor ~~~ AE
AE ~~~ SDR
SDR ~~~ Emprendedor
Emprendedor ~~~ Admin

Web ~~~ Dashboard
Dashboard ~~~ Config
Config ~~~ Llamada
Llamada ~~~ Resultados

Usuarios ~~~ Escenarios
Escenarios ~~~ Sesiones
Sesiones ~~~ Evaluacion
Evaluacion ~~~ Historial
Historial ~~~ Planes

PostgreSQL ~~~ Supabase
Supabase ~~~ Storage
Storage ~~~ Transcripciones
Transcripciones ~~~ Metricas
Metricas ~~~ Prospectos

GenAI ~~~ Gemini
Gemini ~~~ OAuth
OAuth ~~~ Pago
Pago ~~~ Notificaciones

%% =========================
%% ESTILOS
%% =========================
style ACTORES fill:#222,stroke:#fff,stroke-width:2px,color:#fff
style PRESENTACION fill:#222,stroke:#fff,stroke-width:2px,color:#fff
style NEGOCIO fill:#222,stroke:#fff,stroke-width:2px,color:#fff
style DATOS fill:#222,stroke:#fff,stroke-width:2px,color:#fff
style EXTERNOS fill:#222,stroke:#fff,stroke-width:2px,color:#fff
```

## Descripción
La arquitectura inicial de Closer se organiza en cuatro grupos principales:
- **Presentación:** permite que los usuarios configuren escenarios, inicien simulaciones y revisen sus resultados desde la aplicación web.
- **Lógica de negocio:** concentra la gestión de usuarios, escenarios, sesiones, evaluación, historial y planes de uso.
- **Datos:** almacena información de usuarios, prospectos, sesiones, transcripciones, métricas y recursos asociados.
- **Sistemas externos:** integra los servicios de Inteligencia Artificial, autenticación, pagos y notificaciones requeridos por la plataforma.

El flujo principal inicia cuando un usuario configura un escenario de práctica, continúa con una simulación de llamada por voz, procesa la conversación mediante los servicios de IA y finaliza almacenando métricas, transcripciones y recomendaciones para su posterior consulta.
