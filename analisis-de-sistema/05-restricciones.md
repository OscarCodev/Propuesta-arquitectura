# Restricciones

Las restricciones definen las condiciones y decisiones técnicas que deben respetarse durante el diseño y desarrollo del sistema Closer.

## Restricciones del sistema

| ID | Restricción | Descripción |
|---|---|---|
| **RC01** | **Aplicación web** | El sistema debe desarrollarse como una aplicación accesible mediante un navegador web moderno. |
| **RC02** | **Control de versiones** | El código fuente debe gestionarse utilizando Git y mantenerse en un repositorio compartido. |
| **RC03** | **Frontend web** | La interfaz principal debe implementarse con Next.js, React y TypeScript. |
| **RC04** | **Comunicación HTTP** | Las operaciones convencionales entre frontend y backend deben realizarse mediante HTTPS y endpoints tipo REST. |
| **RC05** | **Streaming de voz** | La comunicación de audio en tiempo real debe utilizar un canal bidireccional compatible con WebSockets seguros (WSS) o el mecanismo requerido por el proveedor de IA. |
| **RC06** | **Motor de IA** | La simulación de voz debe integrarse con Google GenAI y un modelo Gemini con capacidad de audio nativo en tiempo real. |
| **RC07** | **Persistencia** | La información principal de la aplicación debe almacenarse en PostgreSQL mediante Supabase. |
| **RC08** | **Autenticación** | La autenticación debe gestionarse con Supabase Auth y permitir integración OAuth con Google. |
| **RC09** | **Seguridad de datos** | El acceso a los datos debe controlarse mediante políticas de autorización y Row Level Security cuando corresponda. |
| **RC10** | **Integraciones externas** | Los servicios de pagos y notificaciones deben integrarse mediante APIs externas sin acoplar directamente la lógica principal a un proveedor específico. |
