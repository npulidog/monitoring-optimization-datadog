Se llevó a cabo una revisión detallada del estado actual de los monitores configurados en Datadog, como parte del objetivo general del proyecto de mejorar su utilidad operativa, reducir alertas innecesarias y establecer una estructura más clara y sostenible.

El enfoque se centró en tres criterios principales:

1. Agrupar los monitores según el tipo de evento que monitorean (fallas en transacciones, uso de recursos, errores de red, eventos relacionados con seguridad).  
2. Verificar su estado operativo (activos, en alerta, sin datos).  
3. Evaluar su relevancia actual, identificar duplicados o configuraciones obsoletas.

Adicionalmente, se buscó alinear este trabajo con los objetivos estratégicos de seguridad y ciberseguridad de la organización. Aunque Datadog no es una herramienta de protección activa como un firewall o un SIEM especializado, sí representa una pieza fundamental en la **detección temprana de comportamientos anómalos**, errores recurrentes que podrían ser **indicios de ataques o abuso del sistema**, y permite generar alertas automatizadas ante condiciones que vulneren la estabilidad o integridad de las transacciones.

Al incorporar reglas específicas para errores como `Bad Request`, `Invalid`, `unauthorized` o encabezados HTTP mal formados, se está dando visibilidad a eventos que pueden representar tanto errores operativos como posibles vectores de ataque (inyección, uso indebido de credenciales, acceso externo desde orígenes no permitidos, etc.). Esto contribuye directamente a los objetivos de ciberseguridad como:

- Reducción de superficie de ataque mediante detección de tráfico malicioso.
- Identificación de posibles fallas de configuración o uso indebido.
- Detección de comportamientos anómalos antes de que escalen a incidentes mayores.
- Visibilidad en tiempo real para actuar sobre transacciones sospechosas.

---

## Punto 1: Limpieza, Consolidación y Creación de Monitoreos

Este paso es clave porque elimina ruido innecesario y fortalece la visibilidad sobre indicadores de riesgo. Al consolidar y depurar los monitores, se reduce la posibilidad de que un incidente real pase desapercibido entre falsos positivos. La creación de nuevos monitores basados en errores reales como `Bad Request`, `Invalid`, `unauthorized` y trazas de `exception` permite actuar de forma proactiva frente a comportamientos anómalos que, si no se controlan, podrían ser indicativos de fallos de configuración o intentos de explotación de vulnerabilidades. Así, se mejora la capacidad de detección temprana, un pilar esencial en cualquier estrategia de ciberseguridad.

### Monitores a Eliminar

- [VTEX] Low Éxito PSE approved percentage  
- CCAPI no responde  

### Monitores a Consolidar

- `Worldpay communication problem!`, `Error de conexión Amex`, `Alert Redeban without transactions` → Consolidar en un único monitor de errores de comunicación por gateway.  
- `NGINX CONNECTIONS LIMIT NOCAPI` y similares → Unificar bajo un solo monitor con tags o agrupación por servicio.  

### Nuevos Monitores a Crear (adaptados a logs y solicitudes específicas)

**Relación con objetivos de ciberseguridad:**  
Cada uno de los siguientes monitores está diseñado para cubrir eventos que podrían representar una amenaza de seguridad si no se detectan a tiempo. Desde intentos de acceso no autorizado hasta solicitudes maliciosas o malformadas, estos patrones son comunes tanto en errores operativos como en ataques dirigidos. Al implementarlos, se establece una primera línea de defensa en la detección de incidentes mediante el análisis de comportamiento y errores que, si son ignorados, podrían escalar a vulneraciones más serias.

### Monitor 1: Aumento de errores HTTP 400 - `Bad Request`

**Descripción:** Detecta solicitudes mal formadas (inputs inválidos, JSON incompleto, etc.)  

**Consulta:**  
```text
status:error AND message:"Bad Request"
```

**Condición:** Más de 20 eventos en 5 minutos por servicio.

**Objetivo:** Detección de errores que pueden ser usados como vectores de inyección u omisiones en validación de entrada.

### Monitor 2: Patrones `Invalid` (campos o cabeceras)

**Descripción:** Identifica errores como `Invalid HTTP_HOST`, headers no válidos, etc.

**Consulta:**  
```text
status:error AND message:"Invalid"
```

**Condición:** Más de 15 eventos por servicio en 10 minutos.

**Objetivo:** Detectar configuraciones mal aplicadas o ataques por manipulación de cabeceras.

### Monitor 3: Traceback en módulo `request_dec`

**Descripción:** Errores en lógica de procesamiento de solicitudes.

**Consulta:**  
```text
@module:request_dec AND status:error
```

**Condición:** Más de 10 eventos en 5 minutos.

**Objetivo:** Visibilidad sobre fallos silenciosos que comprometen el flujo transaccional.

### Monitor 4: Spike in `exception` module

**Descripción:** Detecta aumentos inusuales de errores provenientes del módulo `exception`, comúnmente asociados a errores internos severos.

**Consulta:**  
```text
@module:exception AND status:error
```

**Condición:** Más de 50 errores en la última hora.

**Objetivo:** Priorizar módulos vulnerables con alta frecuencia de fallos para detección proactiva de puntos de falla crítica o explotación.

### Monitor 5: `Not Found` con `report not found`

**Descripción:** Endpoint obsoleto o cliente mal configurado.

**Consulta:**  
```text
status:error AND message:"not found"
```

**Condición:** 5 eventos en 10 minutos por host o servicio.

**Objetivo:** Evitar consumo de recursos por tráfico no válido o prueba de rutas no autorizadas.

---

## Punto 2: Dashboards Propuestos

### Dashboard 1: Seguridad Transaccional

Este dashboard actúa como un centro de vigilancia que permite monitorear errores frecuentes asociados a tráfico anómalo, integraciones vulnerables y patrones de posibles ataques. La visibilidad en tiempo real sobre errores como `403 Forbidden`, `Bad Request` o `Invalid Header` facilita la detección de intentos de acceso indebido, ataques de fuerza bruta, inyecciones maliciosas y fallos en el control de acceso. Además, la agrupación de eventos por módulo (`request_dec`, `exception`) permite identificar los puntos más débiles dentro del flujo de procesamiento.

**Elementos:**

- Gráfico de errores HTTP 403 por servicio
- Gráfico de errores 4xx/5xx por integración
- Lista de rechazos por `unauthorized` o `invalid key`
- Trazabilidad de errores por módulo (`request_dec`, `exception`)
- Tabla con top mensajes por error (`Bad Request`, `Invalid`, etc.)
- Filtros por servicio, región, método de pago

### Dashboard 2: Rendimiento de Transacciones

Este dashboard, aunque centrado en rendimiento, permite detectar comportamientos anómalos como latencia elevada o caídas en tasa de aprobación, que podrían derivar de integraciones maliciosas, sobrecargas o ataques. Aporta datos históricos y en tiempo real para una toma de decisiones rápida ante posibles eventos de seguridad operativa.

**Elementos:**

- Tasa de aprobación por servicio
- Latencia promedio
- Volumen de transacciones (OK vs FALLIDAS)
- Heatmap por horario
- Panel comparativo semanal

---

## Próximos Pasos

- Eliminar monitores sin datos para reducir ruido y mejorar eficiencia.
- Consolidar monitores redundantes para mejorar trazabilidad.
- Crear los nuevos monitores orientados a errores críticos y seguridad.
- Implementar dashboards para visibilidad técnica y de seguridad.
- Validar resultados con el equipo de soporte
