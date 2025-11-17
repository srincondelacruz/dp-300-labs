# Cuestionario: Monitorización y Rendimiento en Azure SQL

## Pregunta 1: Optimización de Almacenamiento para TempDB en Azure VMs

La estrategia recomendada para la ubicación de los archivos de TempDB en SQL Server ejecutándose en Azure Virtual Machines (VMs) es utilizar la **unidad D:**, que es el SSD conectado localmente. Es crucial colocar TempDB en el almacenamiento de menor latencia disponible debido a su utilización intensiva para diversas tareas (tablas temporales, *sorting*, almacén de versiones). Se utiliza la unidad D: porque los archivos de TempDB se recrean al reiniciar el servidor, por lo que no existe riesgo de pérdida de datos.

### Diferencia en la Configuración

La diferencia en la configuración entre las VMs desplegadas desde Azure Marketplace y las instalaciones manuales radica en la automatización del proceso.

* **VMs desplegadas desde Azure Marketplace:** Al usar las plantillas de SQL Server en Azure Marketplace, se reduce la complejidad. El proveedor de recursos de SQL Server soporta la adición de TempDB a la unidad SSD local y crea una tarea programada para crear la carpeta de TempDB en el momento del arranque del sistema.
* **Instalaciones Manuales de SQL Server:** El administrador debe gestionar manualmente la configuración para utilizar el SSD local.

### Consideraciones para Failover Cluster Instances (FCI)

SQL Server puede utilizar File Storage (almacenamiento de archivos) como destino de almacenamiento para una Failover Cluster Instance.

---

## Pregunta 2: Configuración de Número de Archivos TempDB

### Algoritmo Round-Robin para Asignación de Páginas

El algoritmo que rige la asignación de páginas cuando se tienen múltiples archivos de datos se conoce como **relleno proporcional (proportional fill)**. SQL Server distribuye los datos basándose en el tamaño de cada archivo. Si los archivos son de tamaño uniforme (ej. 2 GB cada uno), los datos se distribuyen uniformemente. Sin embargo, si un archivo es de 10 GB y el otro de 1 GB, alrededor de 900 MB irían al archivo más grande y 100 MB al más pequeño.

### Mejora de la Concurrencia

Aumentar el número de archivos de datos mejora la concurrencia porque se utiliza más de un archivo para particionar las asignaciones para las tablas de TempDB. En TempDB (*write-intensive*), un patrón de escritura desigual puede crear un cuello de botella en el archivo más grande. Al aumentar el número de archivos y mantener un tamaño uniforme, se distribuye la actividad de E/S, evitando la contención. El instalador de SQL Server detecta la cantidad de CPUs disponibles y configura un número adecuado de archivos (hasta ocho), asegurando que tengan un tamaño uniforme.

### Desventajas o Limitaciones de Archivos

Las fuentes establecen límites fijos o escalados para TempDB en Azure SQL:
* En **Azure SQL Managed Instance**, se obtienen **12 archivos**, independientemente del número de vCores.
* En **Azure SQL Database**, el número de archivos se escala con el número de vCores, hasta un máximo de **16**.

---

## Pregunta 3: Selección de Tipos de Disco para Cargas de Trabajo SQL Server

Las cargas de trabajo de producción de SQL Server utilizan típicamente **Ultra Disk** o **Premium SSD**. Los discos administrados de Azure son el tipo de almacenamiento más utilizado.

### Comparación y Casos de Uso

1.  **Ultra Disk:**
    * **Características:** Soportan cargas de trabajo de E/S elevadas para bases de datos de misión crítica. Ofrecen flexibilidad para escalar IOPS, rendimiento (throughput) y tamaño de forma independiente.
    * **Latencia y Uso:** Se utilizan cuando se busca una latencia **submiliseundo** (< 1 ms). Recomendados para cargas de trabajo que requieren la latencia más baja posible.

2.  **Premium SSD:**
    * **Características:** Discos de alto rendimiento (*high-throughput*) y baja latencia para la mayoría de las cargas de trabajo. Costos más bajos y mayor flexibilidad que Ultra Disk.
    * **Latencia y Uso:** Típicamente tienen una latencia de **un solo dígito en milisegundos**. Soportan el almacenamiento en caché de lectura (*read-caching*).

3.  **Premium SSD v2:**
    * *(Las fuentes proporcionadas no contienen información específica sobre Premium SSD v2).*

### Escenarios Específicos

* **Archivos de Datos (Data Files):** Se recomienda habilitar el **almacenamiento en caché de lectura (read caching)**. Esta caché se almacena en el SSD local (unidad D: en Windows), beneficiando las cargas de trabajo intensivas en lectura.
* **Archivos de Log de Transacciones (Log Files):** Se debe crear un volumen separado. Es crucial **no habilitar ningún tipo de almacenamiento en caché (caching)** en el volumen de archivos de log. Ultra Disk es ideal para logs muy exigentes.

### Consideraciones sobre Latencia, IOPS y Limitaciones

Es vital diseñar el almacenamiento y la VM para satisfacer los picos de carga sin latencia significativa (planificar un 20% extra de IOPS). Cada tipo de VM tiene un **límite intrínseco de IOPS**. Para aumentar las IOPS en Premium SSD, se requiere la técnica de ***striping*** de datos a través de múltiples discos (usando Storage Spaces en Windows, sin redundancia).

---

## Pregunta 4: Implementación de Resource Governor - Arquitectura

Resource Governor (en SQL Server y Azure SQL MI) permite un control granular sobre CPU, E/S física y memoria.

### Conceptos Fundamentales

1.  **Resource Pool (Pool de Recursos):**
    * Representa los recursos físicos disponibles.
    * SQL Server tiene dos pools incorporados: `default` e `internal` (reservado para funciones críticas).
    * Los pools de usuario pueden configurarse con límites (Min/Max CPU, Min/Max Memoria, Min/Max IOPS por volumen).

2.  **Workload Group (Grupo de Carga de Trabajo):**
    * Sirve como un contenedor para las solicitudes de sesión.
    * Tiene dos grupos incorporados: `default` e `internal`.
    * Cada grupo está asociado a un único Resource Pool.

3.  **Classification (Clasificación):**
    * Es el proceso que define cómo se tratan las conexiones. Se utiliza una **función clasificadora** para subdividir las sesiones en grupos de carga de trabajo.

### Funcionamiento Conjunto para Gestionar Recursos

Cuando Resource Governor está habilitado, utiliza una **función clasificadora** para asignar conexiones entrantes a **grupos de carga de trabajo**. Cada grupo de carga de trabajo está asociado a un **pool de recursos**, que aplica los límites configurados (CPU, E/S, memoria) a todas las sesiones de ese grupo.

### Propósito y Ejecución de la Función Clasificadora

* **Propósito:** Clasifica cada conexión entrante en un grupo de carga de trabajo (útil para *multitenancy* o limitar operaciones de mantenimiento). Si la función devuelve NULL, `default` o un grupo inexistente, la sesión va al grupo `default`.
* **Ejecución:** La función se ejecuta en el momento en que se establece una conexión. Debe ser probada para verificar su eficiencia y evitar impactos en el rendimiento.

---

## Pregunta 5: Configuración Práctica de Resource Governor

Para implementar una restricción (ej. 40% CPU, 50% Memoria) para la aplicación "ReportApp", debe seguir estos pasos en orden (usando T-SQL):

1.  **Crear el Resource Pool:** Crear un nuevo pool que contenga los límites de recursos (Max CPU 40%, Max Memoria 50%). Los límites de CPU son flexibles (solo se aplican durante contención), pero los de memoria (Min/Max) son límites fijos (*hard limits*).
2.  **Crear el Workload Group:** Crear un nuevo grupo que sirva como contenedor para las sesiones de "ReportApp" y asociarlo al Resource Pool recién creado.
3.  **Crear/Modificar la Función Clasificadora (Classifier Function):** Definir una función T-SQL que identifique las conexiones de "ReportApp" (ej. por nombre de usuario o aplicación) y devuelva el nombre del grupo de carga de trabajo.
4.  **Asignar la Función Clasificadora:** Asignar la función recién creada al Resource Governor.

### Comando T-SQL Final para Aplicar Cambios

Una vez configurado todo, debe ejecutar el siguiente comando para que los cambios surtan efecto:

```sql
ALTER RESOURCE GOVERNOR RECONFIGURE;
```
## Pregunta 6: Incrementos de Crecimiento de TempDB

Los archivos de TempDB en Azure SQL Managed Instance tienen valores predeterminados de crecimiento de 254 MB (datos) y 64 MB (log).

### Importancia de la Configuración de Incrementos de Crecimiento

Es crucial configurar adecuadamente los archivos de TempDB para un rendimiento óptimo. En Azure SQL, la opción `AUTOGROW_ALL_FILES` está en ON, lo cual es recomendado.

### Afectación al Algoritmo Round-Robin (Proportional Fill)

El algoritmo de **relleno proporcional (proportional fill)** distribuye los datos basándose en el tamaño de cada archivo.

Si los incrementos de crecimiento no son uniformes o si los archivos de datos crecen de manera desigual, tendrán tamaños desiguales. Esto afecta al algoritmo, creando un **patrón de escritura desigual** y generando un cuello de botella en el archivo más grande, que manejará la mayor parte de las escrituras.

---

## Pregunta 7: Storage Pools y Capping en Azure VMs

### Concepto de "Capping" o Throttling

El *capping* o *throttling* (limitación) se refiere a la restricción de rendimiento impuesta por el tamaño de la máquina virtual (VM) en Azure. Cada tipo de Azure Virtual Machine tiene un **límite intrínseco de IOPS**.

### Cómo una VM Limita el Rendimiento

Una VM puede limitar el rendimiento (capping) incluso si los discos adjuntos tienen una mayor capacidad de IOPS, porque el límite de la VM es la restricción final para el rendimiento de E/S.

### Estrategias para Abordar el Capping

1.  **Configuración de Storage Pools y Striping:**
    * Para Premium SSDs, se realiza el ***striping*** de datos a través de múltiples discos usando **Storage Spaces** en Windows (sin configurar redundancia).
    * El pool resultante tendrá la suma de las IOPS y el volumen de todos los discos que lo componen, ayudando a alcanzar el máximo rendimiento de E/S permitido por la VM.
    * *(Esta técnica no se aplica a Ultra Disk, que escala IOPS de forma independiente)*.
2.  **Uso de Caché (Read Caching):**
    * Se debe habilitar el **almacenamiento en caché de lectura (read caching)** en el volumen de archivos de datos (NUNCA en los archivos de log).
    * El caché se almacena en el SSD local (unidad D:) y beneficia las cargas de trabajo intensivas en lectura.
3.  **Planificación:**
    * Se recomienda planificar un 20% extra de IOPS y rendimiento para manejar picos de carga.

---

## Pregunta 8: Selección de Tamaño de VM para SQL Server

Las características principales a considerar son la **cantidad de memoria** disponible y el número de **IOPS** que la VM puede realizar.

### Comparación de Series de VMs

Las VMs se clasifican en dos familias principales para SQL Server:

1.  **Familias Optimizadas para Memoria (ej. Edsv5/Ebdsv5):** Para cargas de trabajo más grandes que requieren más memoria y/o CPU.
2.  **Familias de Propósito General (ej. Ddsv5):** Muchas aplicaciones de producción pueden ejecutarse cómodamente en estas.

### Escenarios para Núcleos Restringidos (Constrained Cores)

La función de **núcleos restringidos (constrained cores)** permite reducir el costo de la licencia de software (basada en el número de núcleos) mientras se mantiene la cantidad total de memoria, almacenamiento y ancho de banda de E/S de una VM más grande. Es beneficioso para cargas de trabajo de bases de datos que no son intensivas en CPU pero sí en memoria o E/S.

---

## Pregunta 9: Caché de Almacenamiento para Archivos SQL Server

### Diferencias entre las Políticas de Caché

1.  **ReadOnly (Caché de Lectura):** Compatible con Premium SSD. Las lecturas pueden servirse desde una caché.
2.  **None (Sin Caché):** Política recomendada para el volumen de archivos de log de transacciones.
3.  **ReadWrite:** *(Las fuentes proporcionadas no la describen)*.

### Recomendación de Caché para Discos de Datos y Log

* **Archivos de Datos:** Se recomienda habilitar el **caché de lectura (read caching)**. Beneficia a las cargas de trabajo intensivas en lectura (*read-heavy*).
* **Archivos de Log:** Se recomienda **no habilitar ningún tipo de caché**.

### Mejora del Rendimiento y Límites de IOPS

El caché de lectura (almacenado en el SSD local, unidad D:) mejora el rendimiento al reducir el número de viajes al disco real. Las operaciones de lectura servidas desde allí no necesitan llegar al disco administrado adjunto, el cual sí está sujeto a límites de IOPS.

---

## Pregunta 10: Resource Governor en Diferentes Plataformas Azure

Resource Governor (RG) permite un control granular sobre CPU, E/S física y memoria.

### Disponibilidad y Configurabilidad

Resource Governor está disponible y es **totalmente configurable** en las siguientes plataformas:
1.  **SQL Server on-premises**
2.  **Azure SQL Managed Instance** (También soporta la configuración de MAXDOP mediante RG).

### Gestión en Azure SQL Database e Implicaciones

Los administradores **no pueden** configurar Resource Governor directamente en **Azure SQL Database**.

En Azure SQL Database, la gestión del rendimiento de E/S se realiza principalmente mediante la **elección del nivel de servicio y los vCores**. Si una aplicación experimenta restricciones de IOPS, el administrador debe escalar los vCores o moverse a los niveles Business Critical o Hyperscale. (Aunque sí se puede configurar `MAXDOP` usando `ALTER DATABASE SCOPED CONFIGURATION`).

Los administradores de Azure SQL Database carecen de la capacidad de RG para aplicar un control granular y limitar recursos para cargas de trabajo específicas (ej. *multitenancy*).