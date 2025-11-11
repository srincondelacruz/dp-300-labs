Cuestionario: Monitorización y Rendimiento en Azure SQL
Este documento contiene preguntas clave sobre la monitorización, diagnóstico y optimización del rendimiento en Azure SQL.

1. ¿Cuál es la diferencia entre métricas reactivas y proactivas, y por qué es importante una línea base (baseline)?
Las métricas reactivas (como una alerta estática de CPU > 80%) se disparan cuando se supera un umbral fijo. Las métricas proactivas (como los umbrales dinámicos de Azure Monitor) aprenden el comportamiento histórico y la estacionalidad, alertando cuando el sistema opera de forma "anormal", aunque no haya superado un umbral fijo.

Establecer una línea base (baseline) (el estado normal de rendimiento) es imperativo. Su importancia radica en que permite identificar si un problema actual es una anomalía o si está dentro de los parámetros esperados de la carga de trabajo.

2. Nombra cuatro herramientas de monitoreo de rendimiento y su propósito principal.
Azure Monitor Metrics: Rastrea la salud general de un recurso Azure. Es ideal para visualizar datos de rendimiento y configurar alertas.

Query Performance Insight (QPI): Su propósito es identificar rápidamente las consultas más costosas (por CPU, Data IO, Log IO). Es el primer paso en el ajuste de rendimiento de consultas.

Performance Monitor (Perfmon): Herramienta nativa de Windows Server para monitorear métricas del SO y contadores específicos de SQL Server. Esencial para capturar una buena línea base en VMs o servidores locales.

Database Watcher (Preview): Una solución robusta para Azure SQL (Database y Managed Instance). Proporciona una vista de panel único (single-pane-of-glass) de todo el entorno SQL y análisis de baja latencia.

3. ¿Qué son los Extended Events (EE) y en qué se diferencian de SQL Profiler? ¿Cuáles son sus ventajas en producción?
Los Extended Events (EE) son un sistema de monitoreo ligero y potente en Azure SQL que permite capturar información granular sobre la actividad.

A diferencia de SQL Profiler, EE tiene un impacto de rendimiento mucho menor. Esto se debe a dos ventajas clave:

Filtros Avanzados: EE utiliza filtros para limitar la cantidad de datos recopilados, reduciendo la sobrecarga.

Procesamiento Asíncrono: La mayoría de los destinos de EE procesan datos de forma asíncrona (escribiendo primero en memoria), minimizando el impacto en el rendimiento.

En producción, los EE son ideales para solucionar problemas específicos como bloqueos y deadlocking, identificar consultas de larga duración o monitorear operaciones DDL.

4. ¿Qué son las métricas críticas y cuáles son cinco ejemplos clave?
Las métricas críticas son mediciones de rendimiento que el motor de la base de datos recopila para ayudar a identificar problemas de rendimiento y rastrear la salud general del recurso.

Cinco ejemplos clave son:

Processor(_Total)% Processor Time: Mide la utilización total del CPU e indica la carga de trabajo general.

PhysicalDisk(_Total)\Avg. Disk sec/Read y Avg. Disk sec/Write: Mide la latencia del subsistema de almacenamiento (no debería superar los 20 ms).

System\Processor Queue Length: El número de hilos esperando tiempo de procesador. Si es mayor que cero, indica presión de CPU.

SQLServer:Buffer Manager\Page life expectancy (PLE): Indica cuánto tiempo reside una página en memoria. Caídas repentinas pueden indicar patrones de consulta deficientes o presión de memoria.

SQLServer:SQL Statistics\Batch Requests/sec: Mide cuán ocupado está SQL Server. Se usa junto al % Processor Time para entender la carga de trabajo.

5. ¿Cómo funciona Database Watcher y qué ventajas ofrece sobre las herramientas tradicionales?
Database Watcher funciona recopilando datos de monitoreo de bases de datos, elastic pools e managed instances en un almacén de datos central (Azure Data Explorer o Microsoft Fabric). La recolección de datos ocurre con baja latencia (en segundos).

Ventajas:

Proporciona información completa sobre rendimiento, configuración y salud.

Ofrece dashboards con una vista de panel único (single-pane-of-glass) de todo el entorno SQL.

Soporta alertas personalizadas y plantillas parametrizadas para reglas comunes.

6. ¿Qué es Query Performance Insight (QPI) y cómo ayuda a identificar consultas problemáticas?
Query Performance Insight (QPI) es una herramienta que permite a los administradores identificar rápidamente las consultas más costosas (expensive queries), lo cual es el primer paso en el ajuste de rendimiento.

Proporciona información sobre las cinco consultas principales por CPU, Data IO o Log IO. Al seleccionar una, muestra el texto de la consulta y su ID.

QPI no muestra el plan de ejecución, pero el ID de la consulta que proporciona se correlaciona directamente con el Query Store, permitiendo al administrador extraer el plan de ejecución desde allí para un análisis más profundo.

7. ¿Qué metodología recomiendas para establecer una línea base (baseline) en una base de datos nueva?
Establecer una línea base (baseline) es imperativo para entender el estado normal de rendimiento.

La metodología debe incluir:

Recopilación Histórica: Recopilar datos a lo largo del tiempo para identificar cambios respecto a la normalidad.

Correlación de Métricas: Es crítico correlacionar el rendimiento de SQL Server con el del SO subyacente (ej. % Processor Time y Processor Queue Length) y las estadísticas de espera (Wait Stats).

Manejo de Variabilidad: Para cargas estacionales, se deben usar Umbrales Dinámicos (Dynamic Thresholds) de Azure Monitor. Estos umbrales aprenden el comportamiento histórico de la métrica y detectan la estacionalidad, alertando solo sobre operaciones anormales.

8. ¿Qué son las "wait statistics" (estadísticas de espera) y cómo ayudan a diagnosticar cuellos de botella?
Las Estadísticas de Espera son métricas que SQL Server registra cuando un hilo (thread) es forzado a esperar por un recurso que no está disponible (ej. I/O, memoria, bloqueos).

Son cruciales para determinar la causa raíz de la latencia. Se utilizan para identificar cuellos de botella al indicar si los problemas de rendimiento están relacionados con la ejecución de consultas (código) o con limitaciones de hardware (recursos). Identificar el tipo de espera (disponible en la DMV sys.dm_os_wait_stats) es fundamental para aplicar la solución correcta.

9. Compara el monitoreo a nivel de consulta vs. a nivel de sistema. ¿Cuándo usar cada uno?
Monitoreo de Sistema (ej. Azure Monitor, Perfmon): Rastrea la salud general del recurso o del SO (CPU total, latencia de disco).

Monitoreo de Consulta (ej. QPI, Extended Events, Wait Stats): Se centra en métricas del motor de la base de datos (consultas costosas, bloqueos, deadlocks).

Cuándo usarlos:

Usa el nivel de sistema para diagnosticar cuellos de botella de hardware (almacenamiento físico, presión de CPU a nivel de servidor).

Usa el nivel de consulta como primer paso en el ajuste de rendimiento (identificar consultas costosas) o para solucionar problemas internos (bloqueos).

Ambos se complementan, ya que es fundamental correlacionar el rendimiento de SQL Server con el del SO (ej. latencia de disco del SO vs. latencia de archivos de las DMVs).

10. Describe una estrategia de monitoreo completa para una base de datos crítica en Azure SQL.
Una estrategia completa debe incluir:

Herramientas:

Azure Monitor Metrics: Para la salud general y alertas.

Database Watcher: Para una vista unificada (single-pane-of-glass) y análisis de baja latencia.

Query Performance Insight (QPI): Para identificar las consultas más costosas.

Extended Events (EE): Para solución de problemas granulares (ej. deadlocks).

Métricas Clave:

% Processor Time, Processor Queue Length.

Latencia de disco (Avg. Disk sec/Read and Write).

Page life expectancy (PLE).

Estadísticas de Espera (Wait Stats).

Frecuencia y Retención:

Revisión en tiempo real (ventanas de 30 min) para diagnóstico rápido.

Retención activa de 93 días en Azure Monitor, con archivado a largo plazo en Azure Storage.

Alertas Proactivas:

Usar Umbrales Dinámicos (Dynamic Thresholds) en Azure Monitor. Estos umbrales aprenden el comportamiento histórico y la estacionalidad, alertando solo sobre operaciones "anormales".

Configurar un Grupo de Acción para enviar notificaciones (Email/SMS) o disparar automatización (Azure Function) cuando se active una alerta.