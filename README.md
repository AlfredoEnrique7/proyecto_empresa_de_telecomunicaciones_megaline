# Proyecto empresa de telecomunicaciones Megaline

## Descripción
El objetivo central de este proyecto es realizar un análisis comparativo preliminar entre las tarifas de prepago **Surf** y **Ultimate** de la empresa de telecomunicaciones **Megaline**. Mediante la evaluación del comportamiento de consumo de una muestra representativa de 500 clientes durante el año 2018, se busca identificar **cuál de los dos planes genera un mayor volumen de ingresos comerciales**. 

Los resultados y conclusiones obtenidos permitirán al departamento de marketing y estrategia comercial optimizar de manera eficiente la asignación del presupuesto de publicidad para el próximo ciclo fiscal.


## Conclusiones

### I. Decisiones críticas en el procesamiento y análisis de datos
La forma final del análisis estuvo determinada por una serie de suposiciones y reglas técnicas fundamentales adoptadas en la fase de preparación, garantizando la trazabilidad y precisión del pipeline de datos:

1. **Robustez temporal multi-año:** Ante el riesgo de errores lógicos silenciosos provocados por el sesgo del año, se asumió la necesidad de implementar una **clave compuesta basada en `['user_id', 'year', 'month']`** de forma transversal. Aunque el dataset actual abarcaba exclusivamente el año 2018, esta decisión metodológica blinda la escalabilidad del código y asegura un correcto alineamiento de consumos en un entorno de datos real y multi-año.
2. **Aplicación estricta de la regla de redondeo de Megaline:** 
   * **Llamadas:** Se asumió que cada llamada individual debía ser redondeada al entero superior inmediatamente en la fase inicial (`np.ceil`), acumulando después las duraciones.
   * **Internet:** Se aplicó la política comercial de sumar todos los Megabytes consumidos a lo largo de un mes completo por usuario, para luego transformar dicho total a Gigabytes y redondearlo hacia arriba.
3. **Integridad en el cálculo de ingresos (Casos límite):** Las franquicias base de navegación (15 Gb para Surf y 30 Gb para Ultimate) se gestionaron formalmente como **unidades flotantes exactas (`15.0` y `30.0`)**. Esto evitó truncamientos de tipo de datos que habrían alterado la lógica del algoritmo de facturación en los umbrales de consumo (*overage*), garantizando una de las fases de auditoría financiera de alta confianza.
4. **Validación de supuestos para pruebas de hipótesis:** Antes de ejecutar las pruebas t de Student de muestras independientes, se implementó de forma obligatoria la **Prueba de Levene** para evaluar la igualdad de varianzas (homocedasticidad). Esto permitió configurar el parámetro `equal_var` de manera científicamente correcta, eliminando sesgos en el cálculo de los valores *p*.

---

### II. Conclusiones y hallazgos comerciales

#### 1. Perfil, comportamiento y distribución de los usuarios
* **Hábitos de consumo emparejados:** Sorprendentemente, disponer de una franquicia masiva en el plan **Ultimate** no modifica los hábitos humanos básicos de comunicación. El usuario típico de ambos planes consume volúmenes de minutos de llamadas y mensajes de texto sumamente similares.
* **Sesgo hacia el cero y asimetría en SMS:** Al analizar las distribuciones completas mediante histogramas, se constató de forma contundente que el servicio de mensajería (SMS) exhibe una marcada **asimetría positiva (sesgo a la derecha)** con un pico pronunciado cerca de cero en ambas tarifas. Esto demuestra que la gran mayoría de la población de Megaline prefiere prescindir del servicio de SMS tradicional o tiene necesidades mínimas de mensajería, independientemente de los límites incluidos en su paquete.
* **Crecimiento estacional lineal:** Ambos planes experimentan una tendencia de adopción progresiva a lo largo de los meses, iniciando con mínimos en enero y duplicando o triplicando su interacción en diciembre debido a las festividades de fin de año y al incremento neto de clientes en la red.

#### 2. Dinámica financiera y margen operativo por tarifa
* **Plan Surf (\$20.00 base) - El motor variable impulsado por *outliers*:** Aunque se mercadea como una alternativa económica, el límite de 15 Gb de internet resulta severamente ajustado para el consumidor moderno hacia finales de año (la media de diciembre alcanza los **18.29 Gb**). En el caso de los mensajes, la media de **38.60 SMS** se ubica cerca del umbral de cobro. El análisis de distribución mediante diagramas de caja (*boxplots*) reveló la presencia de una cantidad masiva de **valores atípicos (*outliers*)** de usuarios que superan con creces los 100 mensajes y los límites de datos. Son precisamente estos usuarios extremos los que empujan la media de ingresos hacia arriba, convirtiendo a Surf en un generador constante de ingresos por excedentes y haciendo que en diciembre un usuario promedio de Surf termine facturando casi lo mismo que un cliente premium (medias de **~\$60.00**).
* **Plan Ultimate (\$70.00 base) - El piso fijo de la subutilización absoluta:** Este producto representa un modelo de suscripción limpia de alta confianza. La subutilización es la norma del negocio: en ningún mes del año el promedio de llamadas superó los 500 minutos (de 3,000 incluidos). En el rubro de mensajes, la distribución demuestra que el consumo máximo absoluto de los usuarios más activos se estabiliza alrededor de los 150-180 mensajes mensuales, lo que significa que el cliente más intensivo consume menos del 18% del beneficio y el promedio deja sin utilizar más del 95% del paquete de SMS. En internet, la distribución se resguarda cómodamente por debajo de los 30 GB (pico medio de **18.40 Gb** en diciembre). Comercializada como una opción premium, genera flujos de caja predecibles y de altísima rentabilidad neta al cobrar una tarifa base elevada por recursos que se quedan sin usar casi en su totalidad.

#### 3. Resultados de los contrastes de hipótesis estadísticas
* **Diferencia de ingresos entre planes (Hipótesis 1):** Se **rechazó la hipótesis nula ($H_0$)** con una certeza matemática absoluta (p-value cercano a 0). Esto valida que las estructuras de cobro y los excedentes de datos de Surf provocan que el rendimiento financiero promedio anual por usuario difiera significativamente entre ambos planes.
* **Impacto de la ubicación geográfica (Hipótesis 2):** **Se rechazó la hipótesis nula ($H_0$)** debido a que el análisis arrojó un valor *p* de **0.0335**, el cual es inferior a nuestro nivel de significancia ($\alpha = 0.05$). Esto demuestra con validez estadística que el ingreso promedio mensual de los usuarios del área metropolitana de Nueva York-Nueva Jersey **SÍ difiere de forma significativa** del de los usuarios del resto de las regiones del país. Comercial y operativamente, esto sugiere que los clientes de la zona NY-NJ presentan dinámicas de consumo locales (potencialmente un mayor uso de datos móviles en exceso o una distribución distinta de planes) que impactan la facturación promedio de Megaline en comparación con el mercado nacional.

---

### Recomendación estratégica

El plan **Surf** es el más lucrativo a nivel variable debido a la alta sensibilidad de los usuarios hacia el exceso de datos móviles y la presencia de usuarios atípicos de alto consumo que pagan penalizaciones recurrentes. Por otro lado, el plan **Ultimate** provee el flujo de caja fijo más seguro y estable gracias a la subutilización sistemática de sus beneficios, operando sin riesgo de saturación de infraestructura para la compañía. 

Se recomienda orientar las campañas de marketing hacia la retención y adquisición de usuarios en el plan **Ultimate** debido a su bajo costo marginal de red y alta predictibilidad. Asimismo, se aconseja implementar alertas tempranas de consumo para el plan **Surf** a fin de mitigar posibles descontentos por cargos sorpresa derivados del comportamiento de excedencias detectado al cierre de año.

## Tecnologías Utilizadas
* Python (Pandas, Matplotlib.pyplot, NumPy, Spipy.stats)
* Jupyter Notebook

## Ver el Análisis Completo
👉 [Haz clic aquí para ver el código y los gráficos interactivos](proyecto_empresa_de_telecomunicaciones_megaline.ipynb)
