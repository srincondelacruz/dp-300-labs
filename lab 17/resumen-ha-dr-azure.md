# Estrategias de Alta Disponibilidad y Recuperación ante Desastres en Azure

> **Módulo:** Describe high availability and disaster recovery strategies  
> **Fuente:** [Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/describe-high-availability-disaster-recovery-strategies/)

---

## 1. RTO y RPO: Conceptos Clave

| Métrica | Definición | Ejemplo |
|---------|------------|---------|
| **RTO (Recovery Time Objective)** | Tiempo máximo aceptable de inactividad tras una interrupción | Si el RTO es 1 hora, debes restaurar el servicio en menos de 60 minutos |
| **RPO (Recovery Point Objective)** | Cantidad máxima aceptable de pérdida de datos, medida en tiempo | Si el RPO es 15 minutos, debes garantizar backups o replicación cada 15 min |

**Implicaciones:**
- RTO y RPO más bajos requieren soluciones más avanzadas (y costosas): replicación continua, failover automático, etc.
- Deben alinearse con los requisitos de negocio, regulaciones y SLA. 

---

## 2. Opciones de Alta Disponibilidad y Recuperación ante Desastres

### 2.1 Para Azure SQL Database (PaaS)

| Característica | Descripción | Uso principal |
|----------------|-------------|---------------|
| **Redundancia de zona** | Distribuye réplicas en zonas de disponibilidad dentro de una región | Protección ante fallos de zona |
| **Geo-replicación activa** | Réplica legible en otra región Azure | DR multi-región, lectura distribuida |
| **Grupos de conmutación por error (Failover Groups)** | Agrupación de bases de datos para failover coordinado | DR regional, endpoint de listener |
| **Backups automatizados** | Copias automáticas con retención configurable | Recuperación ante errores de usuario, corrupción |

### 2. 2 Para SQL Server en Azure VMs (IaaS)

| Característica | Descripción | Uso principal |
|----------------|-------------|---------------|
| **Always On Availability Groups** | Réplicas a nivel de base de datos, failover automático o manual | HA + DR, offload de backups y lecturas |
| **Failover Cluster Instances (FCI)** | Clustering a nivel de instancia SQL Server | HA de instancia completa |
| **Log Shipping / Mirroring** | Envío periódico de logs o espejo de bases de datos | DR básico, escenarios legacy |
| **Azure Site Recovery** | Replicación de VMs a otra región y orquestación de failover | DR a nivel de VM |
| **Azure Backup** | Copias de seguridad nativas en la nube | Protección contra pérdida/corrupción de datos |

---

## 3. Características de Azure para HA/DR

| Característica | Tipo | Soporte en VM | Soporte en PaaS | Escenario |
|----------------|------|---------------|-----------------|-----------|
| Always On Availability Group | HA/DR | Sí | No | HA + DR, reporting, backups |
| Failover Cluster Instance | HA | Sí | No | HA a nivel de instancia |
| Geo-replicación | DR | No | Sí | DR multi-región, escalado de lectura |
| Log Shipping | DR | Sí | Limitado | DR programado, HA básica |
| Site Recovery | DR | Sí | No | Failover de VMs |

### Availability Sets y Availability Zones

- **Availability Sets:** Protegen contra fallos de hardware distribuyendo VMs en dominios de error/actualización (SLA 99.95%). 
- **Availability Zones:** Ubicaciones físicas separadas dentro de una región Azure; protegen ante fallos de datacenter (SLA 99. 99%).

---

## 4.  Opciones HA/DR en IaaS vs.  PaaS

| Aspecto | IaaS | PaaS |
|---------|------|------|
| **Control** | Total (SO, red, clustering, DB) | Limitado (capas de infra/red) |
| **Configuración** | Compleja, manual | Simple, gestionada por el proveedor |
| **HA/DR integrado** | Mínimo, debes implementar tú mismo | Extensivo, integrado en la plataforma |
| **Personalización** | Alta | Limitada, pero estandarizada |
| **Ejemplos** | Failover clusters, Always On, Site Recovery | Geo-replicación, Failover Groups |

---

## 5. Soluciones IaaS para HA/DR en Azure

### Componentes principales

1. **Availability Sets:** SLA 99.95% para VMs agrupadas. 
2. **Availability Zones:** SLA 99.99% distribuyendo VMs en zonas. 
3. **Azure Site Recovery (ASR):** Replica VMs a otra región, orquesta failover y failback.
4.  **Azure Backup:** Backups nativos en la nube para VMs y aplicaciones. 
5. **Premium Storage:** Mejora durabilidad y rendimiento (SLA 99. 9% para VM de instancia única).

### Pasos para implementar

- Definir RTO y RPO según requisitos de negocio.
- Desplegar Windows Server Failover Cluster o SQL Server Availability Groups.
- Configurar replicación y failover con Azure Site Recovery. 
- Monitorizar y probar regularmente los planes de recuperación. 

---

## 6. Soluciones Híbridas

Las soluciones híbridas combinan recursos on-premises con Azure para lograr HA y DR:

| Solución | Alta Disponibilidad | Recuperación ante Desastres | Soporte Híbrido |
|----------|--------------------|-----------------------------|-----------------|
| Always On AG | Sí | Sí | Sí |
| Log Shipping | No | Sí | Sí |
| Azure Site Recovery | Sí (nivel VM) | Sí | Sí |
| Azure SQL Geo-Rep | Sí | Sí | Sí |
| Azure Backup | No | Sí | Sí |

### Escenarios típicos

- **SQL Server HA/DR híbrido:** Combina clústeres on-premises con réplicas en Azure VMs usando Availability Groups.
- **Resiliencia de archivos/almacenamiento:** Backup de datos on-premises a Azure o replicación para recuperación rápida.
- **Despliegues multi-región:** Failover a diferentes regiones Azure para VMs y servicios gestionados.

### Herramientas nativas de Azure

- **Azure Site Recovery:** Replica workloads (on-premises o cloud) a Azure y permite failover o pruebas de DR.
- **Azure Backup:** Protege datos en la nube y on-premises para retención a largo plazo y restauración point-in-time. 

---

## 7. Buenas Prácticas

- **Evalúa RTO y RPO** para cada solución y alinéalos con los requisitos de negocio.
- **Usa características de infraestructura Azure** (Availability Zones, Site Recovery) junto con redundancia a nivel de aplicación.
- **Para PaaS**, aprovecha geo-replicación y failover automático; considera despliegues multi-región para servicios críticos. 
- **Prueba regularmente** los procedimientos de failover y recuperación para validar que la solución cumple los objetivos de continuidad.

---

## 8. Referencias y Recursos

- [Describe RTO y RPO](https://learn.microsoft.com/en-us/training/modules/describe-high-availability-disaster-recovery-strategies/2-describe-recovery-time-objective-recovery-point-objective)
- [Explorar opciones de HA/DR](https://learn.microsoft.com/en-us/training/modules/describe-high-availability-disaster-recovery-strategies/3-explore-high-availability-disaster-recovery-options)
- [Características de Azure para HA/DR](https://learn.microsoft.com/en-us/training/modules/describe-high-availability-disaster-recovery-strategies/4-describe-azure-high-availability-disaster-recovery-features)
- [Opciones HA/DR](https://learn.microsoft.com/en-us/training/modules/describe-high-availability-disaster-recovery-strategies/5-describe-high-availability-disaster-recovery-options)
- [Soluciones IaaS para HA/DR](https://learn.microsoft.com/en-us/training/modules/describe-high-availability-disaster-recovery-strategies/6-explore-iaas-high-availability-disaster-recovery-solution)
- [Soluciones híbridas](https://learn.microsoft.com/en-us/training/modules/describe-high-availability-disaster-recovery-strategies/7-describe-hybrid-solutions)
- [Azure SQL Database Business Continuity](https://learn.microsoft.com/en-us/azure/azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview?view=azuresql)
- [SQL Server en Azure VMs: HA/DR](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/business-continuity-high-availability-disaster-recovery-hadr-overview?view=azuresql)
- [Checklist de HA/DR para Azure SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/high-availability-disaster-recovery-checklist?view=azuresql)
- [Explorar soluciones IaaS y PaaS para HA/DR](https://learn.microsoft.com/en-us/training/modules/explore-iaas-paas-platform-tools-for-high-availability-disaster-recovery/)