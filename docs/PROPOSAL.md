# Propuesta del Proyecto

## Resumen

El objetivo principal es resolver el problema de mantener la información clínica accesible y consistente tanto con conexión a internet como sin ella. 

Para lograrlo, la solución propuesta se basa en una arquitectura de sincronización bidireccional:

1.Base de Datos Local (Room): La aplicación guarda todos los datos en el dispositivo, permitiendo su uso completo sin conexión.

2.Backend en la Nube (Supabase): Los datos locales se sincronizan con un servidor central, asegurando que la información esté respaldada y sea accesible desde diferentes dispositivos.

Además, el proyecto incluye funcionalidades avanzadas como:

\-Sincronización automática en segundo plano (WorkManager).

\-Gestión clínica completa (recetas, notas, múltiples doctores).

\-Reportes inteligentes generados por Inteligencia Artificial (Google Gemini) a partir de los datos de los pacientes.

## Alcance

* Módulos/funcionalidades incluidos:

-Gestión de Pacientes y Citas: Formularios para crear, ver y editar la información de pacientes y sus citas programadas.

-Listado de Pacientes: Visualización de pacientes de forma paginada y ordenada.

-Sincronización Bidireccional: Sincronización automática y manual entre la base de datos local (Room) y el backend en la nube (Supabase).

-Sincronización en Segundo Plano: Uso de WorkManager para ejecutar la sincronización periódicamente (cada hora) sin intervención del usuario.

-Funcionalidades Clínicas Adicionales: Registro de recetas médicas y notas clínicas asociadas a los pacientes.

-Soporte Multi-doctor: Capacidad para asignar pacientes a diferentes doctores/usuarios.

-Reportes con IA: Generación de resúmenes y análisis en lenguaje natural a partir de los datos de los pacientes utilizando la API de Google Gemini.



* Fuera de alcance en esta fase:

-Funcionalidades más complejas como la facturación, un portal web para pacientes, o la integración directa con sistemas de laboratorio (LIS) no forman parte de la fase actual del proyecto.



## Arquitectura

* Diagrama y descripción (Room + Supabase + WorkManager + Hilt): 

-La arquitectura se centra en un modelo de datos dual con sincronización:

\-Room (Base de Datos Local): Almacena todos los datos de la aplicación en el dispositivo, permitiendo un funcionamiento completo sin conexión a internet.

\-Supabase (Backend en la Nube): Actúa como la fuente central de verdad, utilizando una base de datos Postgres para almacenar la información de forma segura y accesible desde múltiples dispositivos.

\-WorkManager (Tareas en Segundo Plano): Orquesta un SyncWorker que se ejecuta periódicamente para realizar la sincronización de datos sin afectar el rendimiento de la interfaz de usuario.

\-Hilt (Inyección de Dependencias): Gestiona las dependencias de todo el proyecto, facilitando la creación y provisión de componentes como los repositorios, ViewModels, y los clientes de Supabase.



* Sincronización y manejo de conflictos:

\-Se utiliza una tabla dedicada llamada SyncMetadata para rastrear el estado de sincronización de cada registro (creado, modificado, sincronizado).

\-Un SyncRepository centraliza la lógica de leer los metadatos, enviar los cambios locales a Supabase, y traer las actualizaciones del servidor.

\-Los Mappers y DTOs (Data Transfer Objects) se encargan de convertir los datos entre el formato de las entidades de Room y el formato esperado por la API de Supabase.



## Requerimientos

* Funcionales:

◦El sistema debe permitir el registro, edición y borrado de pacientes.

◦Debe ser posible crear y gestionar citas para cada paciente.

◦La aplicación debe funcionar offline, guardando los cambios localmente.

◦Los datos deben sincronizarse automáticamente con el servidor.

◦El usuario debe poder forzar una sincronización manualmente.

◦La aplicación debe poder generar reportes descriptivos usando IA.

* No Funcionales:

◦Persistencia: Los datos no deben perderse si la aplicación se cierra o no hay conexión.

◦Rendimiento: La interfaz de usuario debe ser fluida, y la sincronización en segundo plano no debe consumir recursos excesivos.

◦Seguridad: Las claves de API y credenciales de la base de datos (GEMINI\_API\_KEY, SUPABASE\_URL, SUPABASE\_ANON\_KEY) deben manejarse de forma segura y no ser expuestas en el código fuente.

◦Compatibilidad: La aplicación debe ser compatible con las versiones de Android especificadas en la configuración de build.gradle.kts.



## Cambios vs propuesta original

Formalización del Esquema de Sincronización con Metadatos: La idea original podría haber sido una sincronización más simple (por ejemplo, borrar y reemplazar todos los datos). La creación de una tabla específica sync\_metadata y un SyncRepository para gestionar el estado de cada registro individualmente (creado, modificado, sincronizado) es un salto cualitativo enorme. Esto demuestra una arquitectura mucho más robusta y escalable, capaz de manejar sincronizaciones parciales y eficientes.

## Riesgos y mitigaciones

* Riesgo 1: Conflictos de Sincronización. Si un mismo dato es modificado en el dispositivo y en el servidor antes de sincronizarse, podría haber una pérdida de datos.

\-Mitigación: Aunque no se detalla una estrategia de resolución de conflictos (ej. "el más reciente gana"), la existencia de la tabla SyncMetadata es el primer paso para implementar dicha lógica. El SyncRepository sería el lugar para añadir reglas que decidan qué versión del dato prevalece.



* Riesgo 2: Gestión de Claves de API. Exponer las claves de Gemini o Supabase en el repositorio comprometería la seguridad del backend y podría generar costos inesperados.

\-Mitigación: El proyecto implementa la mitigación estándar y recomendada: utiliza el archivo local.properties (ignorado por Git) para almacenar las claves. Además, se proporciona un archivo local.properties.example para guiar al desarrollador, asegurando que las claves nunca se suban al control de versiones.



* Riesgo de Migraciones de BD Fallidas: Un error al actualizar la base de datos local (Room) podría corromper datos o hacer que la app falle.

\-Mitigación: Realizar pruebas de migración automatizadas y, como último recurso, usar fallbackToDestructiveMigration() para recrear la base de datos y forzar una sincronización completa desde el servidor.

## 

