# Drivers arquitectónicos

Los drivers arquitectónicos representan los requisitos, atributos de calidad y restricciones que tienen mayor influencia sobre las decisiones de arquitectura del sistema Closer.

## Drivers arquitectónicos identificados

| ID | Driver arquitectónico | Origen | ¿Por qué influye en la arquitectura? |
|---|---|---|---|
| **DA01** | La simulación debe permitir conversación bidireccional por voz con baja latencia. | AC01 – Rendimiento / RF07 | Condiciona el uso de streaming de audio, conexiones persistentes y procesamiento en tiempo real. |
| **DA02** | El sistema debe soportar múltiples sesiones de práctica simultáneas. | AC03 – Escalabilidad | Puede influir en la gestión de conexiones, recursos de servidor y estrategia de escalamiento. |
| **DA03** | El sistema debe proteger datos personales, transcripciones, métricas y sesiones. | AC04 – Seguridad | Influye en autenticación, autorización, políticas RLS y manejo seguro de credenciales y sesiones. |
| **DA04** | El sistema debe integrarse con modelos de IA de Google para voz y análisis. | RC06 – Motor de IA | Condiciona interfaces de integración, formato del audio, manejo de contexto y tratamiento de respuestas de IA. |
| **DA05** | El frontend debe comunicarse con el backend mediante HTTPS/REST y con el servicio de voz en tiempo real mediante WSS. | RC04, RC05 | Define los principales mecanismos de comunicación entre componentes. |
| **DA06** | La solución debe permitir sustituir o modificar integraciones externas sin afectar el núcleo del sistema. | AC05 – Mantenibilidad / AC07 – Interoperabilidad | Favorece una separación clara entre lógica de negocio e infraestructura externa. |
| **DA07** | El sistema debe almacenar sesiones, métricas, transcripciones y prospectos de forma estructurada. | RC07 – Persistencia | Influye en el modelo de datos y en el uso combinado de PostgreSQL, JSONB y almacenamiento de archivos. |
