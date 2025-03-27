# Optimización de Monitoreo en Datadog

Se llevó a cabo una revisión detallada del estado actual de los monitores configurados en Datadog, como parte del objetivo general del proyecto de mejorar su utilidad operativa, reducir alertas innecesarias y establecer una estructura más clara y sostenible.

El análisis abarcó una gran cantidad de monitores asociados a servicios como VTEX, Worldpay, Redeban, CCAPI, entre otros. El enfoque se centró en tres criterios principales:

1. Agrupar los monitores según el tipo de evento que monitorean (fallas en transacciones, uso de recursos, errores de red, eventos relacionados con seguridad).  
2. Verificar su estado operativo (activos, en alerta, sin datos).  
3. Evaluar su relevancia actual, identificar duplicados o configuraciones obsoletas.

A partir de esta revisión, se construyó una tabla donde se clasificaron los monitores según su categoría funcional y se incluyeron observaciones específicas para cada uno.

---

## Limpieza, Consolidación y Creación de Monitoreos

### Monitores a Eliminar (por obsolescencia o "NO DATA")

- [VTEX] Low Éxito PSE approved percentage  
- CCAPI no responde  

### 🔄 Monitores a Consolidar (por redundancia o duplicidad)

- `Worldpay communication problem!`, `Error de conexión Amex`, `Alert Redeban without transactions`  
  → Consolidar en un único monitor de errores de comunicación por gateway.  
- `NGINX CONNECTIONS LIMIT NOCAPI` y otros similares  
  → Unificar bajo un solo monitor con tags o agrupación por servicio.  

### Nuevos Monitores a Crear

| Nombre del Monitor                                 | Descripción                                                                 | Condición                                                             | Acción                                                        |
|----------------------------------------------------|-----------------------------------------------------------------------------|----------------------------------------------------------------------|---------------------------------------------------------------|
| Errores HTTP 403 recurrentes por servicio          | Detecta posibles accesos o integraciones bloqueadas.                       | Más de 10 errores HTTP 403 por servicio en 10 minutos.              | Notificación a soporte técnico y seguridad.                  |
| Caída repentina en porcentaje de aprobación        | Puede indicar un fallo en procesos críticos o un intento de fraude.       | Tasa de aprobación < 85% sostenida durante 10 minutos.              | Escalamiento a equipos de operaciones y monitoreo.           |
| Aumento súbito de transacciones fallidas           | Puede sugerir ataques automatizados o integraciones mal implementadas.    | Duplicación del promedio histórico de fallos por 15 minutos.        | Investigación de logs, posible bloqueo de IP o cliente.      |
| Rechazos por credenciales inválidas/no autorizadas | Indica posible abuso de APIs o pruebas maliciosas.                         | Más de 5 rechazos por "unauthorized" o "invalid API key" en 5 min. | Alerta y escalamiento a revisión antifraude.                 |

---

## Dashboards Propuestos

### Seguridad Transaccional

**Objetivo:** Visualizar indicadores clave que pueden reflejar eventos de seguridad o integraciones comprometidas.

**Elementos:**

- Gráfico de errores HTTP 403 por servicio.  
- Gráfico de errores 4xx/5xx agrupados por integración.  
- Lista de transacciones rechazadas con motivo `unauthorized` o `invalid key`.  
- Alertas activas por caídas de tasa de aprobación.  
- Tendencia de transacciones fallidas por hora/día.  


---

### Rendimiento de Transacciones

**Objetivo:** Monitorear la salud y comportamiento de las transacciones en producción.

**Elementos:**

- Porcentaje de aprobación por pasarela.  
- Latencia promedio por servicio.  
- Errores por servicio (tasa de error).  
- Gráfico de volumen de transacciones (aprobadas vs rechazadas).  
- Uso de recursos (si está disponible).  

---

## Próximos Pasos

- Implementar la limpieza inmediata de los monitores sin datos.  
- Consolidar monitores redundantes y estandarizar su nomenclatura.  
- Configurar al menos 3 de los nuevos monitores propuestos.  
- Construir los dos dashboards sugeridos con filtros funcionales.  
- Validar resultados con el equipo de soporte.  
