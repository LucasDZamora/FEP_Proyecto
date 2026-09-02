# Especificación funcional, interacción por actor y flujo de trabajo de la solución

**OVERFORE S.A. · Licitación TFEP-01/2026 — Caso 01 · Minería**

| | |
|---|---|
| Documento | Especificación funcional, interacción por actor y flujo de trabajo |
| Proponente | OVERFORE S.A. |
| Licitación | TFEP-01/2026 — Caso 01 · Minería. Compañía Minera Aranda |
| Naturaleza | Documento de apoyo interno de la Oferta Técnica. No constituye por sí mismo ninguno de los catorce subdocumentos del Formulario T-7: alimenta al Subdocumento 3 —descripción de la solución y catálogo de requerimientos—, al Subdocumento 4 —arquitectura lógica— y al Subdocumento 7 —implantación— |
| Insumos internos | Catálogo de requerimientos v2.4 · Matriz de trazabilidad v1.1 · Registro de reglas de negocio v1.2 · Registro de supuestos v1.5 · Subdocumento 4.1 v3.1 · Formulario T-12 v1.3 |
| Insumos del CLIENTE | Bases Administrativas TFEP-01/2026 · Bases Técnicas Transversales TFEP-01/2026 · Bases Técnicas del Caso 01 · Minería |
| Versión | 1.0 — 1 de septiembre de 2026 |

**Advertencia de coherencia.** El Artículo 57.2 de las Bases Administrativas habilita descuento de puntaje por incoherencia entre secciones de la propuesta. Los códigos de requerimiento, los nombres de componente C-01 a C-22, los nombres de sitio, las etapas y los umbrales usados aquí son los mismos del catálogo v2.4, de la matriz v1.1 y del Subdocumento 4.1 v3.1, y deben permanecer literalmente iguales en el resto de la propuesta.

---

## 1. Marco del documento

### 1.1 Qué responde este documento y qué no

Responde cuatro preguntas, en este orden:

1. **Qué hace el software**, función por función, con el código de requerimiento que la origina, el componente que la ejecuta y la etapa en que se entrega.
2. **Cómo interactúa cada actor con el software**, con el canal, el dispositivo, la cantidad de acciones que se le piden, el umbral de tiempo que se le compromete y qué ocurre con esa interacción durante un corte de enlace.
3. **Cuál es el flujo de trabajo del nuevo software**, extremo a extremo, siguiendo el ciclo del mineral del banco al embarque y los flujos transversales que lo cruzan.
4. **Por qué la solución cumple**, contrastada contra las Bases Administrativas, las Bases Técnicas Transversales y las Bases Técnicas del Caso.

No responde cómo se construye. La tecnología, el emplazamiento de cada componente, el motor de persistencia, el dimensionamiento de infraestructura y el plan de pruebas son materia de los Subdocumentos 4.2, 5 y 9. Aquí no se nombra ningún producto ni ningún proveedor.

### 1.2 Cómo verificar lo que este documento afirma

Cada función lleva su código del catálogo v2.4. Cada código es rastreable en la matriz de trazabilidad v1.1 hasta su origen en el documento del CLIENTE —capítulo, entrevista, indicador o restricción—, hasta su componente de la arquitectura, hasta su prueba de verificación y hasta el criterio de aceptación del Capítulo 18 que sostiene. Ninguna afirmación de este documento depende de una fuente que no esté declarada.

Convenciones de código:

| Prefijo | Qué designa | Registro que lo define |
|---|---|---|
| RF-XXX-000 | Requerimiento funcional | Catálogo de requerimientos v2.4, hoja RF |
| RNF-XXX-000 | Requerimiento no funcional, con umbral numérico | Catálogo de requerimientos v2.4, hoja RNF |
| RT-00.00 | Requisito transversal obligatorio | Bases Técnicas Transversales, Capítulo 4 |
| RN-00 | Regla de negocio de la industria | Registro de reglas de negocio v1.2 |
| SUP-00 | Supuesto declarado por el PROPONENTE | Registro de supuestos v1.5 |
| C-00 | Componente lógico de la arquitectura | Subdocumento 4.1 v3.1, apartado 5 |

### 1.3 Magnitud de lo especificado

| Dimensión | Cantidad |
|---|---|
| Requerimientos funcionales | 167, en 19 familias |
| Requerimientos no funcionales con umbral | 61, en 11 categorías |
| Componentes lógicos | 22 |
| Reglas de negocio declaradas | 30 |
| Supuestos declarados | 39 |
| Requerimientos de Etapa 1 | 134 |
| Requerimientos de Etapa 2 | 30 |
| Requerimientos fuera del Contrato, con punto de extensión previsto | 3, conforme al Artículo 72.º de las Bases Administrativas |

De los 167 requerimientos funcionales, **105 tienen al «Sistema» como actor**: se ejecutan sin que ninguna persona los dispare. Es la cifra más relevante de este documento y la que gobierna todo el apartado 3. La solución no se sostiene pidiéndole trabajo a las personas de terreno, sino cruzando automáticamente información que los sistemas del CLIENTE ya generan y que hoy no se cruza. De los 62 restantes, 8 no son funciones de pantalla sino obligaciones del ADJUDICATARIO, del PROPONENTE, de la Jefatura de Proyecto o del Comité de Operación: estudio de cobertura, catálogo de integraciones, ventana operacional protegida, migración, marcha blanca, despliegue por área y sitio, declaración de lo parametrizable y registro del riesgo de propiedad de los datos de telemetría. Quedan **54 requerimientos con interacción de una persona**, repartidos entre los actores del apartado 3.

---

## 2. Qué hace el software, función por función

Las diecinueve familias siguen la numeración del catálogo v2.4. Cada bloque abre con la magnitud de la familia, los componentes que la ejecutan y los criterios de aceptación del Capítulo 18 que sostiene; a continuación se enumeran sus funciones, cada una con su actor, su componente y su etapa.

### 2.2 Familia 1 · Planificación minera y cumplimiento del plan

**8 requerimientos** · Etapa 1: 8 · Componentes: C-02, C-08, C-09, C-10, C-11 · Criterios de aceptación: n.º 2 · n.º 3 · n.º 4

- **RF-PLA-001** *(Sistema · C-11 · Etapa 1)* — Consumir automáticamente las salidas del software de planificación minera -polígonos con identificador, banco, geometría, ley estimada por elemento, tonelaje planificado y destino asignado- sin exportación a carpeta compartida ni carga manual, y sin intervenir en el diseño del plan ni en la estimación de recursos y reservas.
- **RF-PLA-002** *(Sistema · C-02 · Etapa 1)* — Almacenar el plan minero como entidad versionada e inmutable, creando una versión con marca de tiempo, autor y motivo en cada publicación o modificación, impidiendo toda alteración retroactiva de una versión publicada, permitiendo consultar la secuencia de versiones de un rango de fechas con el detalle de qué cambió entre versiones consecutivas, y permitiendo recalcular periodos históricos contra las versiones vigentes en cada momento.
- **RF-PLA-003** *(Superintendencia de Planificación Minera · C-02 · Etapa 1)* — Permitir a la Superintendencia de Planificación Minera registrar los cambios de plan que ocurren durante la semana -caída de rampa, desplazamiento del contacto declarado tras una tronadura, cambio de requerimiento de mezcla de planta- generando la versión correspondiente en el momento del cambio, sin que la radio o la mensajería del turno sean la única constancia.
- **RF-PLA-004** *(Sistema · C-10 · Etapa 1)* — Asociar automáticamente a cada movimiento de material la versión del plan vigente en el instante en que ocurrió, sin intervención de la persona operadora ni del despachador.
- **RF-PLA-005** *(Sistema · C-08 · Etapa 1)* — Calcular el cumplimiento del plan -tonelaje y ley planificados contra movidos, por polígono y por destino- contra la versión vigente en el momento de cada movimiento, y entregar el resultado real del turno anterior al inicio de cada turno, de modo que la replanificación diaria se haga con información del turno y no de dos semanas.
- **RF-PLA-006** *(Sistema · C-02 · Etapa 1)* — Ofrecer, como consulta disponible sobre el mismo dato versionado y sin desarrollo adicional comprometido, el cumplimiento calculado contra la versión del plan vigente al inicio del periodo, distinguiéndolo del cumplimiento de ejecución.
- **RF-PLA-009** *(Responsable de tronadura · C-02 · Etapa 1)* — Registrar el evento de liberación de un sector para carguío tras la tronadura, con marca de tiempo y responsable, y notificarlo automáticamente al despacho y a la operación mina.
- **RF-PLA-011** *(Sistema · C-09 · Etapa 1)* — Registrar la ley estimada del modelo de bloques asociada a cada polígono y conservarla como referencia para la reconciliación posterior contra la ley medida.

### 2.3 Familia 2 · Perforación y tronadura

**5 requerimientos** · Etapa 1: 4 · Etapa 2: 1 · Componentes: C-03, C-04, C-15 · Criterios de aceptación: n.º 5 · n.º 13

- **RF-PET-001** *(Sistema · C-03 · Etapa 1)* — Capturar automáticamente desde las perforadoras instrumentadas la posición, la profundidad y los parámetros de perforación de cada pozo, asociándolos a la malla de perforación y al polígono correspondiente.
- **RF-PET-002** *(Persona operadora de perforadora · C-04 · Etapa 1)* — Permitir a la persona operadora de las perforadoras no instrumentadas registrar el avance del turno desde la aplicación móvil, con operación desconectada y con un número acotado de acciones, eliminando la planilla que hoy se entrega al final del turno.
- **RF-PET-003** *(Sistema · C-03 · Etapa 1)* — Poner los datos de perforación a disposición del área de geología en un formato estructurado, procesable y exportable, sin intervención del área de perforación y tronadura.
- **RF-PET-004** *(Sistema · C-15 · Etapa 1)* — Registrar cada tronadura con su malla, su ventana programada, su ejecución efectiva y el sector afectado, asociarla a las detenciones de equipo que provoque y emitir el aviso de evacuación por los canales definidos, incluida la integración con el sistema de radio y megafonía de faena.
- **RF-PET-006** *(Sistema · C-03 · Etapa 2)* — Poner a disposición de geología y planificación los índices de dureza del macizo rocoso derivados de los parámetros de perforación.

### 2.4 Familia 3 · Carguío, transporte y despacho

**8 requerimientos** · Etapa 1: 7 · fuera del Contrato: 1 · Componentes: C-01, C-10 · Criterios de aceptación: n.º 1

- **RF-CAR-001** *(Sistema · C-10 · Etapa 1)* — Integrarse con el sistema de despacho de flota vigente para obtener las asignaciones de camión a pala y a destino, los tiempos de ciclo, el tonelaje del pesaje de a bordo y la posición de los equipos, conservándolo como fuente, sin sustituirlo ni modificarlo, con la frecuencia y el sentido que la propuesta declare.
- **RF-CAR-002** *(Despachador · C-10 · Etapa 1)* — Registrar el cambio de destino de un camión ordenado por el despachador conservando la razón del cambio seleccionada desde una lista cerrada, de modo que la razón deje de perderse en la radio.
- **RF-CAR-003** *(Persona operadora de pala · C-10 · Etapa 1)* — Poner a disposición de la persona operadora de pala un botón único en cabina para avisar que el material cargado cambio respecto de lo planificado -sin selección de causas, sin texto libre y sin esperar confirmación- y asociar cada accionamiento al viaje y al ciclo de carguío en curso, con marca de tiempo, equipo, persona operadora y posición de la pala, notificándolo al despachador.
- **RF-CAR-005** *(Sistema · C-01 · Etapa 1)* — Asignar integramente cada viaje al polígono en que se encuentra posicionada la pala al momento del carguío, sin prorrateo automático en la Etapa 1, y marcar el viaje con un indicador de proximidad a borde de polígono cuando la pala opere a menos de una distancia umbral parametrizable del límite, calculada desde la posición que la telemetría ya entrega y sin pedir acción alguna a la persona operadora.
- **RF-CAR-008** *(Sistema · C-01 · Fuera de alcance (Art. 72.º))* — Permitir el prorrateo automático del tonelaje de un viaje entre dos polígonos cuando la medición en marcha blanca demuestre que la proporción de viajes marcados como borde es materialmente significativa, previa caracterización de umbrales, geometría de borde y precisión de posicionamiento.
- **RF-CAR-009** *(Persona operadora de equipo · C-01 · Etapa 1)* — Identificar a la persona operadora de un equipo compartido mediante la lectura de su credencial de proximidad en cabina, en el punto de relevo, capturada automáticamente por el sistema de despacho y sin login manual, sin contraseña y sin pantallas adicionales durante la operación, abriendo y cerrando una sesión por equipo y por turno a la que quedaran asociados todos los eventos generados por ese equipo.
- **RF-CAR-011** *(Sistema · C-01 · Etapa 1)* — Marcar el registro como operador no identificado y alertar al supervisor de turno cuando nadie se identifique en el relevo, sin detener el equipo ni bloquear la operación, y publicar la proporción de tiempo en esa condición como indicador de gestion con responsable asignado, por equipo, turno y área.
- **RF-CAR-015** *(Persona operadora de equipo · C-01 · Etapa 1)* — Permitir a la persona operadora de equipo levantar un aviso de falla desde la cabina con una sola acción, generando la orden de trabajo de mantenimiento sin que un planificador deba transcribirlo desde la radio.

### 2.5 Familia 4 · Stocks intermedios

**6 requerimientos** · Etapa 1: 5 · Etapa 2: 1 · Componentes: C-01, C-08, C-09, C-20 · Criterios de aceptación: n.º 1 · n.º 2 · n.º 3 · n.º 9

- **RF-STK-001** *(Sistema · C-08 · Etapa 1)* — Registrar cada aporte de material a un stock con polígono de origen, tonelaje, ley por elemento, fecha y hora, equipo y viaje asociado, y mantener en línea el saldo, la composición por polígono de origen y la ley ponderada de cada stock intermedio y del stock de baja ley, reemplazando la planilla del supervisor de turno.
- **RF-STK-002** *(Sistema · C-08 · Etapa 1)* — Recalcular automáticamente la ley ponderada vigente del stock ante cada aporte y cada retoma, y registrar cada retoma con el cargador frontal que la ejecutó, el tonelaje, el destino y la marca de tiempo, conservando la proporción con que cada polígono contribuyó al material retomado.
- **RF-STK-006** *(Cargador frontal / Laboratorio · C-09 · Etapa 1)* — Soportar la campaña puntual de muestreo de verificación de la ley en retoma durante la marcha blanca, registrando la muestra, la ley medida y la ley que predecía el promedio ponderado, y la desviación resultante.
- **RF-STK-007** *(Sistema · C-01 · Etapa 2)* — Permitir modelar el stock por sublotes con una regla de consumo declarada cuando la campaña de verificación demuestre que la desviación es materialmente alta.
- **RF-STK-008** *(Sistema · C-01 · Etapa 1)* — Registrar el traspaso de material entre stocks y el reproceso del stock de baja ley conservando la cadena de origen por polígono.
- **RF-STK-009** *(Sistema · C-20 · Etapa 1)* — Alertar cuando el saldo calculado de un stock diverja del saldo declarado por el levantamiento topográfico o por el control de inventario en más de un umbral parametrizable.

### 2.6 Familia 5 · Pesaje, tonelaje y conciliación metalúrgica

**10 requerimientos** · Etapa 1: 9 · fuera del Contrato: 1 · Componentes: C-08, C-09, C-11 · Criterios de aceptación: n.º 2 · n.º 3

- **RF-PES-001** *(Sistema · C-08 · Etapa 1)* — Registrar y conservar las tres mediciones de tonelaje tal como se generan -pesaje de a bordo, báscula de camiones y balanza de alimentación de planta- sin transformarlas, y publicar la diferencia entre ellas por periodo, por equipo y por punto de la cadena, sin deducir causas ni corregir automáticamente las cifras de origen.
- **RF-PES-002** *(Sistema · C-08 · Etapa 1)* — Utilizar la balanza de alimentación de planta como fuente de verdad oficial del tonelaje total procesado, y calcular la atribución espacial por polígono a partir del pesaje de a bordo prorrateado contra ese total, de modo que exista una sola cifra oficial de tonelaje y el resto sean distribuciones proporcionales sobre ella.
- **RF-PES-004** *(Sistema · C-09 · Etapa 1)* — Registrar la báscula de camiones como punto de calibración periódica del pesaje de a bordo, conservando la muestra de viajes pesados, la desviación medida por equipo y la fecha de cada campaña.
- **RF-PES-006** *(Sistema · C-08 · Fuera de alcance (Art. 72.º))* — Aplicar un factor de corrección del pesaje de a bordo por equipo y por periodo, versionado y con vigencia declarada, cuando el recálculo retroactivo demuestre que la diferencia no queda explicada por punto de medición.
- **RF-PES-007** *(Sistema · C-11 · Etapa 1)* — Calcular la conciliación metalúrgica del periodo cuadrando el cobre estimado en el modelo de bloques, el extraído, el alimentado a planta, el recuperado y el embarcado, expresados en metal contenido, y atribuir la diferencia a cada punto de la cadena en que se produce, de modo que toda diferencia que persista quede explicada por punto.
- **RF-PES-009** *(Sistema · C-08 · Etapa 1)* — Calcular el balance metalúrgico del turno desde los datos registrados, reemplazando la planilla del metalurgista de turno.
- **RF-PES-010** *(Metalurgia · C-08 · Etapa 1)* — Registrar los factores de ajuste del balance metalúrgico del mes como parámetros versionados, con vigencia, responsable y fundamento documentado, de modo que su explicación no dependa de las dos personas que hoy los conocen.
- **RF-PES-012** *(Sistema · C-08 · Etapa 1)* — Entregar a SAP los datos de tonelaje, ley, metal contenido, consumos y movimientos necesarios para el registro contable, y consumir de el los datos maestros, de costo y de mantenimiento que requiera, sin reemplazarlo, sin modificarlo, sin emitir cifra contable propia y sin constituir una segunda verdad contable.
- **RF-PES-014** *(Sistema · C-08 · Etapa 1)* — Registrar la humedad del concentrado y expresar el tonelaje en base seca y en base humeda donde corresponda, conforme a la práctica de comercialización de concentrados.
- **RF-PES-016** *(Sistema · C-08 · Etapa 1)* — Registrar el tonelaje de estéril movido y la razón estéril/mineral efectiva del periodo, diferenciándola de la planificada.

### 2.7 Familia 6 · Laboratorio, muestreo y ley

**12 requerimientos** · Etapa 1: 11 · Etapa 2: 1 · Componentes: C-01, C-07, C-09, C-20 · Criterios de aceptación: n.º 1 · n.º 9

- **RF-LAB-001** *(Sistema · C-09 · Etapa 1)* — Integrarse en ambos sentidos con el sistema de gestion de laboratorio, enviando la solicitud de análisis con la identificación de la muestra y recibiendo el resultado sin transcripción manual a planillas.
- **RF-LAB-002** *(Sistema · C-09 · Etapa 1)* — Tomar el resultado del sistema de gestion de laboratorio como fuente de verdad de la ley medida, tratar la ley del modelo de bloques y la del plan como estimaciones que se reconcilian contra ella, y publicar por polígono la diferencia entre ley estimada y ley medida como indicador de calidad del modelo.
- **RF-LAB-004** *(Persona que toma la muestra / Laboratorio · C-09 · Etapa 1)* — Registrar la cadena de custodia de cada muestra indicando quién la tomó, cuándo, en qué punto, bajo qué protocolo, con qué equipo y quién la recibió en laboratorio, conservando cada traspaso con marca de tiempo.
- **RF-LAB-005** *(Sistema · C-09 · Etapa 1)* — Gestionar el ciclo de vida del lote mediante una máquina de estados con transiciones auditadas -pendiente, analizado, validado, liberado, con las ramas rechazado, remuestreo, revalidación y no liberable- e impedir la liberación a embarque de todo lote sin análisis válido y vigente, rechazando la operación aunque el usuario tenga perfil comercial y aunque exista presión de programación de nave.
- **RF-LAB-007** *(Sistema · C-20 · Etapa 1)* — Disparar el remuestreo con plazo definido ante una muestra rechazada o invalidada, bloqueando comercialmente el lote con motivo registrado desde lista cerrada, y escalar el caso a metalurgia cuando el rechazo en un mismo lote se repita más de un número parametrizable de veces.
- **RF-LAB-009** *(Sistema · C-01 · Etapa 1)* — Registrar los resultados de análisis y mantener por cada lote un vector de elementos -cobre, oro, plata y arsénico como elemento deletéreo- más la humedad, y no un único valor de ley de cobre, calculando la conciliación y la liquidación sobre ese vector.
- **RF-LAB-010** *(Sistema · C-09 · Etapa 1)* — Mantener el catálogo de puntos de muestreo obligatorios ubicados donde el material ya se detiene o donde lo opera un rol distinto del ciclo de mina -chancado, alimentación de planta, relave, báscula de control, retoma de stock y conformación de lote de embarque- y no exigir muestreo dentro del ciclo de la pala ni del camión.
- **RF-LAB-011** *(Sistema · C-09 · Etapa 1)* — Registrar para cada punto de muestreo su costo declarado en segundos de ciclo y su efecto estimado en toneladas por año, de modo que el CLIENTE decida sobre la base de una cifra y no de un arbitraje entre gerencias.
- **RF-LAB-012** *(Sistema · C-09 · Etapa 1)* — Capturar el dato de los muestreadores de proceso automáticos por vía de instrumentación, separando la acción física de muestreo del registro del dato, que no deberá nacer de un formulario.
- **RF-LAB-013** *(Sistema · C-09 · Etapa 1)* — Alertar cuando un punto de transferencia definido como obligatorio no registre muestra dentro de la frecuencia establecida por procedimiento.
- **RF-LAB-014** *(Sistema · C-07 · Etapa 2)* — Registrar el resultado del laboratorio del comprador y, ante discrepancia, el del tercer laboratorio árbitro, asociándolos al lote embarcado y a su liquidación.
- **RF-LAB-015** *(Sistema · C-09 · Etapa 1)* — Registrar el tiempo entre la toma de la muestra y la disponibilidad del resultado, y alertar cuando exceda el plazo definido por tipo de análisis.

### 2.8 Familia 7 · Trazabilidad del mineral

**7 requerimientos** · Etapa 1: 6 · Etapa 2: 1 · Componentes: C-01, C-07, C-08 · Criterios de aceptación: n.º 1 · n.º 2 · n.º 3

- **RF-TRZ-001** *(Sistema · C-08 · Etapa 1)* — Registrar cada viaje de camión de extracción como unidad atómica de trazabilidad, con identificador único, camión, pala de origen, polígono, destino efectivo de descarga, tonelaje de a bordo, posiciones y marcas de tiempo de carguío y descarga, y registrar cada ciclo de carguío de pala con sus pases, tomando todos esos datos del sistema de despacho sin agregar ninguna captura nueva.
- **RF-TRZ-002** *(Sistema · C-01 · Etapa 1)* — Modelar el polígono como atributo del viaje y el turno, el lote de chancado y el lote comercial como agregaciones derivadas, conservando la relación entre viaje, lote de chancado, stock, lote de concentrado y lote comercial, y permitiendo navegar entre niveles en ambos sentidos sin reconstrucción manual.
- **RF-TRZ-003** *(Área comercial / Operación / Autoridad · C-01 · Etapa 1)* — Permitir reconstruir el origen de cualquier lote embarcado -polígonos y su proporción, fechas de extracción, stocks recorridos, fecha de procesamiento y análisis que lo respaldan- con evidencia y no con estimación, y recorrer la cadena en sentido inverso indicando en qué lotes, stocks y embarques terminó el material de un polígono extraído en una fecha determinada.
- **RF-TRZ-005** *(Sistema · C-01 · Etapa 1)* — Almacenar la trazabilidad como registro inmutable de eventos, de modo que toda corrección se exprese como un evento posterior y ninguna altere el evento original.
- **RF-TRZ-007** *(Sistema · C-01 · Etapa 1)* — Declarar una línea base de inventario al arranque, marcando el material acopiado con anterioridad como origen anterior al sistema con la mejor estimación disponible y esa condición visible en toda consulta, informar el plazo durante el cual la trazabilidad será parcial, y permitir verificar que ese material se ha extinguido de los lotes embarcados y que existe evidencia de cadena de custodia suficiente antes de enero de 2029.
- **RF-TRZ-010** *(Área comercial · C-07 · Etapa 2)* — Generar un expediente de trazabilidad de lote exportable, con identificación de la versión de las reglas y de los supuestos aplicados y con firma electrónica, apto para ser entregado al comprador o a la autoridad.
- **RF-TRZ-012** *(Sistema · C-08 · Etapa 1)* — Emitir y registrar digitalmente la guía de despacho y la planilla de pesaje de cada camión de concentrado que sale a puerto, reemplazando el documento en papel, y conservar ese transporte como eslabon trazable asociado al lote de concentrado que transporta.

### 2.9 Familia 8 · Lote comercial, embarque, devolución y liquidación

**9 requerimientos** · Etapa 1: 5 · Etapa 2: 4 · Componentes: C-01, C-07, C-08 · Criterios de aceptación: n.º 1 · n.º 2 · n.º 3

- **RF-LOT-002** *(Terminal portuario / Área comercial · C-08 · Etapa 1)* — Registrar la recepción y el acopio del concentrado en el recinto portuario por lotes, con tonelaje, humedad y fecha, aun cuando el recinto sea administrado por un tercero.
- **RF-LOT-003** *(Área comercial · C-01 · Etapa 1)* — Registrar la conformación del lote comercial de embarque indicando qué lotes del acopio lo componen y en qué proporción, mediante una relación padre-hijo explícita.
- **RF-LOT-004** *(Terminal portuario / Laboratorio · C-07 · Etapa 2)* — Registrar el muestreo del lote comercial conforme al protocolo acordado con el comprador, con su cadena de custodia, la humedad determinada y la ley por elemento.
- **RF-LOT-005** *(Sistema · C-07 · Etapa 1)* — Calcular y alertar el contenido de arsénico proyectado del lote comercial antes de su conformación definitiva, comparandolo contra el límite del contrato del comprador.
- **RF-LOT-006** *(Sistema · C-01 · Etapa 2)* — Proponer combinaciones de material del acopio que mantengan el contenido de elementos deletéreos bajo el límite contractual, como apoyo a la decisión del área comercial.
- **RF-LOT-007** *(Sistema · C-01 · Etapa 1)* — Aplicar firma electrónica avanzada a los registros de seguridad exigidos por la autoridad, a las actas de aceptación de embarque y a los informes de monitoreo ambiental remitidos a la autoridad, conservando cada documento asociado a su lote, registro o compromiso de origen.
- **RF-LOT-008** *(Área comercial · C-07 · Etapa 2)* — Registrar la liquidación del embarque con el comprador incorporando ley pagable, deducciones por humedad, cargos de tratamiento y refinación y penalidades por elementos deletéreos, asociarla al lote comercial aunque ocurra meses después, y contrastar el ingreso estimado por embarque contra la liquidación efectiva publicando la diferencia.
- **RF-LOT-009** *(Área comercial · C-07 · Etapa 2)* — Modelar el concentrado devuelto o rechazado por el comprador como un evento de reversa sobre el lote original, que reabre su trazabilidad y genera un lote de reingreso con relación padre-hijo hacia los lotes que lo componian, sin crear un módulo de logística inversa; exigir remuestreo y reanálisis del reingreso antes de permitir su remezcla, y entregar a SAP el efecto contable como ajuste de periodo anterior.
- **RF-LOT-013** *(Área comercial · C-01 · Etapa 1)* — Registrar la interacción con la nave, el agente de aduana y el organismo fiscalizador asociada a cada embarque, conservando los documentos intercambiados.

### 2.10 Familia 9 · Detenciones operacionales y productividad

**8 requerimientos** · Etapa 1: 8 · Componentes: C-15 · Criterios de aceptación: n.º 5

- **RF-DET-001** *(Sistema · C-15 · Etapa 1)* — Mantener una taxonomía cerrada y jerárquica de tres niveles para las detenciones operacionales, construida sobre el vocabulario estándar de la industria y parametrizable sin desarrollo.
- **RF-DET-002** *(Sistema · C-15 · Etapa 1)* — Clasificar automáticamente cada detención a partir de las señales existentes -posición y estado del despacho, ventana de tronadura del plan, calendario de turno y colación, telemetría y pauta de SAP- sin intervención humana.
- **RF-DET-003** *(Supervisor de turno · C-15 · Etapa 1)* — Presentar al supervisor de turno únicamente las detenciones que la clasificación automática no resolvió, para su clasificación desde lista cerrada y sin texto libre, antes del cierre del turno, sin requerir más de cinco minutos por turno.
- **RF-DET-004** *(Persona operadora de equipo · C-15 · Etapa 1)* — No requerir de la persona operadora de equipo ninguna acción de clasificación de detenciones, ni en cabina ni en ningún otro dispositivo.
- **RF-DET-005** *(Supervisor de turno · C-15 · Etapa 1)* — Impedir el cierre del turno con detenciones sin clasificar, o escalarlas al supervisor de la guardia siguiente con constancia del escalamiento.
- **RF-DET-006** *(Mantenimiento / Superintendencia de Planta · C-15 · Etapa 1)* — Registrar la causa raíz de las detenciones no programadas de planta y de equipo mina, con responsable, acción tomada y estado de cierre.
- **RF-DET-007** *(Sistema · C-15 · Etapa 1)* — Calcular y publicar por equipo, área y turno la disponibilidad física, la utilización efectiva, las horas de detención no programada, el tiempo medio entre falla y diagnostico, la pérdida efectiva por cambio de turno con sus componentes y el porcentaje de tiempo registrado sin clasificar, este último como indicador de gestion con responsable asignado.
- **RF-DET-011** *(Sistema · C-15 · Etapa 1)* — Conservar y exhibir la línea base 2025 de los indicadores de los numerales 7.1, 7.2 y 7.3 junto al valor vigente y a la meta comprometida, de modo que la mejora sea verificable.

### 2.11 Familia 10 · Mantenimiento y telemetría de equipos

**7 requerimientos** · Etapa 1: 3 · Etapa 2: 4 · Componentes: C-13, C-16 · Criterios de aceptación: n.º 6 · n.º 10

- **RF-MTO-001** *(Sistema · C-13 · Etapa 2)* — Consolidar en un repositorio único la telemetría de los tres portales de fabricante -alarmas, horas de operación, consumos y códigos de falla- declarando adaptadores independientes tras una capa anticorrupción y, para cada uno, el protocolo, el sentido, la frecuencia y la latencia, de modo que la indisponibilidad o la negativa de un fabricante no comprometa el resto de la arquitectura.
- **RF-MTO-002** *(Supervisor / Centro Integrado de Operaciones · C-13 · Etapa 1)* — Presentar al supervisor, en una sola vista y ante un equipo detenido, el estado del equipo, su historial reciente, sus alarmas de telemetría, sus pautas pendientes en SAP y su desempeño comparado, permitiéndole responder que le paso al equipo sin abrir otro sistema, sin usar la radio y en menos de dos minutos.
- **RF-MTO-003** *(Sistema · C-13 · Etapa 1)* — Obtener de SAP las pautas preventivas y las órdenes de trabajo, asociarlas al equipo sin duplicar su registro, y cruzar automáticamente las horas de operación provenientes del despacho y de la telemetría con las pautas basadas en horas, alertando la proximidad del vencimiento de una pauta.
- **RF-MTO-005** *(Jefatura de Mantenimiento · C-16 · Etapa 1)* — Permitir configurar, por tipo de alarma, su umbral, su criticidad, su destinatario por rol y área, su canal de notificación y su supresión temporal, de modo que el volumen entregado sea gestionable y no se abandone como ocurrió con las dos iniciativas anteriores.
- **RF-MTO-008** *(Sistema · C-13 · Etapa 2)* — Admitir, como plan alternativo ante la negativa contractual de un fabricante, la ingesta de telemetría desde un dispositivo de a bordo propio o desde el estándar abierto de intercambio de datos de equipos mineros.
- **RF-MTO-009** *(Sistema · C-16 · Etapa 2)* — Ofrecer analítica predictiva de mantenimiento declarando con qué datos se entrena el modelo, cuántas alertas por semana se espera emitir, cuál es la tasa esperada de falsos positivos y qué mecanismo de mejora se aplica cuando la alerta se equivoca.
- **RF-MTO-012** *(Jefatura de Proyecto · C-13 · Etapa 2)* — Registrar como riesgo abierto, con impacto y responsable, el estado de la negociación sobre la propiedad de los datos de telemetría con cada fabricante, y reflejar en la vista de equipo que fuentes están efectivamente disponibles.

### 2.12 Familia 11 · Identidad, acreditación y control de acceso

**18 requerimientos** · Etapa 1: 17 · fuera del Contrato: 1 · Componentes: C-05, C-06, C-20 · Criterios de aceptación: n.º 7 · n.º 8 · n.º 9 · n.º 10

- **RF-IDE-001** *(Sistema · C-05 · Etapa 1)* — Federar la identidad del personal propio con el directorio corporativo del grupo y exigir doble factor de autenticación a la totalidad de las personas usuarias, sin credenciales compartidas y con registro completo de accesos.
- **RF-IDE-002** *(Sistema · C-06 · Etapa 1)* — Utilizar la credencial física de proximidad emitida en la acreditación de portería como identidad digital de la persona contratista, sin crear un segundo sistema de identidad ni exigir correo corporativo.
- **RF-IDE-003** *(Persona usuaria en terreno · C-05 · Etapa 1)* — Autenticar a la persona en terreno con el solo acercamiento de su credencial a un dispositivo previamente inventariado, exigiendo ademas un PIN numérico corto únicamente fuera del ciclo productivo -portal, portería y dispositivos de gestion- sin exigir en terreno contraseña alfanumérica escrita, correo corporativo ni dispositivo personal.
- **RF-IDE-005** *(Empresa contratista · C-06 · Etapa 1)* — Ofrecer un portal autenticado de empresas contratistas en que cada empresa administre las identidades de sus trabajadores, cargue su documentación de habilitación y siga el estado de acreditación, con registro, verificación de identidad y recuperación de acceso autoservidos, sin que el área de seguridad del CLIENTE ni el equipo de tecnologías de información intervengan en el trámite ordinario.
- **RF-IDE-007** *(Sistema · C-05 · Etapa 1)* — Validar automáticamente el estado de habilitación de cada persona -contrato vigente, exámenes preocupacionales y de altura geográfica al día, vigilancia por exposición a sílice, cursos de inducción y de riesgos aprobados y credencial emitida- calcular en línea si puede ingresar y autorizar o rechazar su ingreso en el torniquete de portería con motivo registrado.
- **RF-IDE-008** *(Sistema · C-05 · Etapa 1)* — Alertar con anticipación configurable el vencimiento de cualquier documento de habilitación a la propia persona trabajadora, a su empleador y al área de seguridad y salud ocupacional.
- **RF-IDE-009** *(Empresa contratista / Área de seguridad · C-06 · Etapa 1)* — Gestionar la acreditación de una persona contratista nueva mediante un flujo con estados, responsables, plazos y escalamiento, de modo que se complete en menos de 48 horas y no dependa de la disponibilidad del área de seguridad del CLIENTE.
- **RF-IDE-011** *(Sistema · C-05 · Etapa 1)* — Registrar todos los ingresos y salidas de portería con persona, empresa, marca de tiempo y resultado, y mantener en línea el inventario de personas presentes en faena.
- **RF-IDE-012** *(Sistema · C-05 · Etapa 1)* — Detectar mediante análisis de patrones sobre los eventos de acceso ya registrados los indicios de uso indebido o préstamo de credenciales, y reportarlos al área de seguridad.
- **RF-IDE-013** *(Sistema · C-05 · Fuera de alcance (Art. 72.º))* — Admitir la incorporación posterior de verificación biométrica en portería sin redisenar la arquitectura de control de acceso, sujeta al resultado de la mesa de diálogo laboral que gestiona el CLIENTE.
- **RF-IDE-014** *(Sistema · C-05 · Etapa 1)* — Controlar el acceso al recinto técnico de servidores de faena conforme al estándar transversal, tratándolo como un control distinto e independiente del de la portería general.
- **RF-IDE-018** *(Administrador del CLIENTE · C-05 · Etapa 1)* — Gestionar perfiles y autorizaciones por rol, área y sitio bajo el principio de mínimo privilegio, permitiendo que una misma persona tenga atribuciones distintas en faena, en el Centro Integrado de Operaciones y en el acopio de puerto, y permitir su administración por el CLIENTE sin intervención del ADJUDICATARIO en las operaciones ordinarias.
- **RF-IDE-019** *(Sistema · C-05 · Etapa 1)* — Revocar el acceso de una persona en un plazo no superior a quince minutos desde el termino de su relación con la faena, e impedir su ingreso desde ese instante.
- **RF-IDE-020** *(Sistema · C-20 · Etapa 1)* — Registrar toda consulta a información sensible -habilitación, salud ocupacional, control de fatiga y control de alcohol y drogas- ademas de toda modificación, conservando quién, cuándo y sobre qué persona o registro.
- **RF-IDE-021** *(Sistema · C-05 · Etapa 1)* — Ofrecer inicio de sesión único para todos los módulos de la solución, con cierre de sesión propagado a todos ellos.
- **RF-IDE-022** *(Sistema · C-05 · Etapa 1)* — Gestionar los accesos privilegiados con elevación temporal a demanda, aprobación previa y grabación de sesión para las operaciones de mayor riesgo.
- **RF-IDE-023** *(Sistema · C-05 · Etapa 1)* — Declarar y hacer cumplir la política de sesión: duración máxima, caducidad por inactividad, renovación de la credencial tras la autenticación, revocación inmediata y control de sesiones concurrentes.
- **RF-IDE-024** *(Sistema · C-05 · Etapa 1)* — Disponer de un procedimiento de acceso de emergencia -cuenta de último recurso- con custodia declarada, control de uso y auditoría de cada activación.

### 2.13 Familia 12 · Seguridad y salud ocupacional

**6 requerimientos** · Etapa 1: 2 · Etapa 2: 4 · Componentes: C-04, C-18, C-20 · Criterios de aceptación: n.º 9 · n.º 10 · n.º 13

- **RF-SSO-001** *(Supervisor de terreno · C-04 · Etapa 1)* — Permitir al supervisor de terreno registrar incidentes, cuasi accidentes, observaciones de conducta e inspecciones planificadas en el lugar del hecho, desde la aplicación móvil, con operación desconectada y firma en terreno, asociando cada registro a la persona, el equipo, el área, el turno y la sesión de operador vigente, y permitir a la Superintendencia de Seguridad consultarlos y exportarlos ante una fiscalización sin buscar carpetas físicas.
- **RF-SSO-004** *(Supervisor / Área de seguridad y salud ocupacional · C-18 · Etapa 2)* — Gestionar mediante un flujo con responsable, plazo, estado y escalamiento automático la acción tomada ante cada alerta de fatiga, cada alarma crítica de telemetría, cada acción correctiva derivada de un incidente o de una observación y cada vencimiento de habilitación, de modo que ninguna quede sin acción documentada y trazable.
- **RF-SSO-007** *(Sistema · C-18 · Etapa 1)* — Cifrar la información en tránsito y en reposo con algoritmos vigentes y aplicar cifrado a nivel de campo a los datos de salud ocupacional, exámenes preocupacionales, control de fatiga y control de alcohol y drogas, incluso en el almacenamiento local de los dispositivos de terreno.
- **RF-SSO-008** *(Sistema · C-18 · Etapa 2)* — Recibir e ingerir como evento propio del sistema cada alerta emitida por la plataforma de control de fatiga, con sello de tiempo, persona, equipo, turno y tipo de alerta, conservando el identificador de origen para su conciliación posterior.
- **RF-SSO-009** *(Área de Seguridad y Salud Ocupacional · C-18 · Etapa 2)* — Generar un reporte de conciliación entre las alertas de fatiga emitidas y las acciones registradas, exhibiendo como indicador el residuo de alertas sin acción documentada por período, área y turno.
- **RF-SSO-010** *(Sistema · C-20 · Etapa 2)* — Tratar el dato de fatiga conforme a la Ley N.21.719 declarando finalidad, base de licitud, consentimiento cuando corresponda, destinatarios autorizados, plazo de conservación y cadena de custodia, y registrar toda cesión o consulta del dato.

### 2.14 Familia 13 · Medio ambiente, huella de carbono y comunidad

**12 requerimientos** · Etapa 1: 3 · Etapa 2: 9 · Componentes: C-04, C-09, C-17, C-20 · Criterios de aceptación: n.º 9 · n.º 11 · n.º 12 · n.º 13

- **RF-AMB-001** *(Sistema · C-17 · Etapa 2)* — Capturar automáticamente los datos de la instrumentación ambiental y geotécnica propia -consumo de agua, calidad del aire, ruido, monitoreo geotécnico y de aguas subterráneas del depósito de relaves- sin consolidación manual en el área de sustentabilidad.
- **RF-AMB-002** *(Área de Sustentabilidad · C-17 · Etapa 2)* — Registrar cada compromiso de la Resolución de Calificación Ambiental con su frecuencia, su punto de medición y su destinatario, y alertar la proximidad del vencimiento de cada reporte.
- **RF-AMB-003** *(Área de Sustentabilidad · C-17 · Etapa 2)* — Generar los reportes a la autoridad ambiental desde el propio sistema, con trazabilidad desde el valor reportado hasta la medición de origen, con la fecha y la fuente de cada dato, y sin inconsistencias entre reportes en un mismo periodo.
- **RF-AMB-004** *(Laboratorio externo / Empresa geotécnica · C-09 · Etapa 1)* — Recibir y asociar al compromiso correspondiente los informes de laboratorios externos y de la empresa geotécnica, conservando el documento original y extrayendo el valor reportado.
- **RF-AMB-005** *(Sistema · C-20 · Etapa 1)* — Calcular el consumo de agua, de energía y de combustible por área -planta, mina, campamento y supresión de polvo en caminos- aplicando una regla de reparto parametrizable y declarada como supuesto mientras no exista instrumentación por área.
- **RF-AMB-006** *(Área de Sustentabilidad · C-17 · Etapa 2)* — Calcular la huella de carbono por tonelada de cobre fino con alcances 1, 2 y 3, con periodicidad mensual, conservando versionados los factores de emisión y la evidencia de cada dato de actividad, de modo que un cálculo pasado pueda reproducirse exactamente y ser auditado por un tercero.
- **RF-AMB-008** *(Sistema · C-17 · Etapa 2)* — Obtener desde SAP y desde recursos humanos los datos de compras locales y de empleo local, y generar los indicadores del convenio con la comunidad sin solicitar planillas a las áreas.
- **RF-AMB-009** *(Sistema · C-04 · Etapa 1)* — Almacenar localmente y sincronizar posteriormente los datos de monitoreo del depósito de relaves cuando el radioenlace se interrumpa, sin pérdida de mediciones ni de reportes obligatorios.
- **RF-AMB-010** *(Sistema · C-17 · Etapa 2)* — Alertar la superación de umbrales de monitoreo geotécnico y ambiental a los destinatarios definidos por los canales de notificación establecidos.
- **RF-AMB-013** *(Área de Sustentabilidad · C-17 · Etapa 2)* — Producir la información trazable sobre el avance de la operación y las obras comprometidas que exige la normativa de cierre de faenas mineras.
- **RF-AMB-014** *(Área de Sustentabilidad · C-17 · Etapa 2)* — Reconstruir hacia atrás la serie mensual de huella de carbono sobre los datos de actividad registrados desde la entrada en producción de la Etapa 1, aplicando a cada período los factores de emisión vigentes en ese período y dejando registro del origen de cada dato utilizado.
- **RF-AMB-015** *(Área de Sustentabilidad · C-17 · Etapa 2)* — Generar el expediente de evidencia auditable de la huella de carbono para el verificador tercero, incluyendo datos de actividad, factores de emisión versionados, memoria de cálculo por alcance y trazabilidad de cada cifra hasta su medición de origen, en formato abierto y documentado.

### 2.15 Familia 14 · Operación desconectada, sincronización y continuidad

**9 requerimientos** · Etapa 1: 9 · Componentes: C-04, C-05 · Criterios de aceptación: n.º 7 · n.º 13

- **RF-DES-001** *(Persona usuaria en terreno · C-04 · Etapa 1)* — Permitir que los dispositivos de terreno registren eventos localmente, firmados y cifrados a nivel de campo, cuando no exista cobertura, con identica funcionalidad de captura que en línea -incluido el registro de seguridad con firma en terreno- e indicando de forma visible el estado de sincronización.
- **RF-DES-002** *(Sistema · C-04 · Etapa 1)* — Generar en el propio dispositivo un identificador único por evento y reconciliarlo al recuperar la conexión de forma idempotente, reintentable y automática, sin duplicados, sin pérdida, sin intervención manual y sin degradar la operación en curso.
- **RF-DES-003** *(Sistema · C-04 · Etapa 1)* — Utilizar una fuente horaria común y trazable para la faena, el Centro Integrado de Operaciones y el acopio de puerto, sincronizar periódicamente los dispositivos de terreno, sellar cada evento generado sin conexión con el reloj monótono local y registrar la deriva del reloj respecto de la fuente común.
- **RF-DES-004** *(Persona operadora de equipo · C-04 · Etapa 1)* — Cerrar localmente la sesión de la persona operadora saliente y abrir la de la entrante cuando el relevo ocurra sin señal, encolando ambas sesiones sin fusionar sus eventos.
- **RF-DES-005** *(Sistema · C-04 · Etapa 1)* — Aplicar una política de resolución de conflictos declarada por tipo de dato, en la que el evento de terreno prevalece para los hechos observados en terreno y el registro central prevalece para los datos maestros.
- **RF-DES-006** *(Sistema · C-04 · Etapa 1)* — Validar la credencial y las autorizaciones de una persona en modo desconectado a partir de una caché local de habilitaciones vigentes, con vigencia máxima declarada.
- **RF-DES-009** *(Sistema · C-05 · Etapa 1)* — Operar con gabinetes de borde en portería, planta, depósito de relaves y acopio de puerto, con energía respaldada y capacidad de registrar y reconciliar de forma autónoma.
- **RF-DES-011** *(Centro Integrado de Operaciones / TI · C-04 · Etapa 1)* — Presentar un panel de estado de sincronización por dispositivo y por sitio, indicando eventos pendientes, antigüedad del más antiguo y último instante de sincronización exitosa.
- **RF-DES-013** *(ADJUDICATARIO · C-04 · Etapa 1)* — Incorporar y mantener actualizado el estudio de cobertura del rajo con proyección a tres años, identificando las zonas de sombra y declarando que cobertura adicional faltaria para su funcionamiento pleno, sin construir la red de comunicaciones y sin suponer una cobertura fija.

### 2.16 Familia 15 · Portales para terceros

**3 requerimientos** · Etapa 1: 2 · Etapa 2: 1 · Componentes: C-06, C-07, C-09 · Criterios de aceptación: n.º 1 · n.º 8

- **RF-POR-001** *(Comprador / Operación / Autoridad · C-07 · Etapa 2)* — Ofrecer a los compradores un portal de trazabilidad de lote autenticado que entregue un certificado derivado -origen por sector, fechas, stocks recorridos, análisis y cadena de custodia- sin exponer la ley por polígono ni la cadena completa, exigiendo aprobación del área comercial antes de publicar, registrando cada consulta, y diferenciando perfiles distintos con alcance declarado para el comprador, la operación y la autoridad.
- **RF-POR-005** *(Operador portuario / Laboratorio externo · C-09 · Etapa 1)* — Dar acceso autenticado y acotado al operador del terminal portuario y a los laboratorios externos, limitado a la información que su función requiere.
- **RF-POR-006** *(Sistema · C-06 · Etapa 1)* — No exponer ningún portal abierto a la ciudadanía.

### 2.17 Familia 16 · Integraciones

**10 requerimientos** · Etapa 1: 8 · Etapa 2: 2 · Componentes: C-12, C-13, C-14, C-17, C-18, C-22 · Criterios de aceptación: n.º 1 · n.º 2 · n.º 3 · n.º 6 · n.º 7 · n.º 10 · n.º 11 · n.º 12 · n.º 13

- **RF-INT-005** *(Sistema · C-14 · Etapa 1)* — Leer las señales del historiador de proceso mediante OPC UA, en modo exclusivamente de lectura, sin escribir en ningún caso en la red de control de proceso y sin instalar componentes en ella sin la segregación que la compañía apruebe, conforme a IEC 62443.
- **RF-INT-009** *(Sistema · C-22 · Etapa 1)* — Integrar los periféricos de identificación y pesaje del ciclo productivo -lectores de código y de radiofrecuencia para identificación de camiones y personas, posicionamiento satelital de precisión, básculas de camiones, pesaje de a bordo y torniquetes de portería-, conservando el equipamiento existente y admitiendo el reemplazo del software local de la báscula solo si el PROPONENTE lo justifica.
- **RF-INT-010** *(Sistema · C-12 · Etapa 1)* — Emitir notificaciones por correo electrónico, notificación en aplicación, mensajería instantánea para contratistas, mensaje de texto para alertas críticas y de seguridad, e integración con el sistema de radio y megafonía de faena para las alertas de evacuación.
- **RF-INT-011** *(ADJUDICATARIO · C-12 · Etapa 1)* — Mantener un catálogo de integraciones documentado y versionado con el sistema contraparte, el sentido, el protocolo, la frecuencia y el responsable, actualizado durante todo el contrato y sin protocolos propietarios no documentados.
- **RF-INT-012** *(Sistema · C-13 · Etapa 1)* — Degradarse de forma controlada y declarada por escrito ante la indisponibilidad del enlace, de un portal de fabricante, del sistema de despacho, del historiador, del sistema de laboratorio o de SAP, indicando qué función se mantiene, cuál se difiere y cuál se suspende, sin detener el registro operacional propio.
- **RF-INT-013** *(Sistema · C-17 · Etapa 2)* — Utilizar ISA-95 como modelo de referencia de la integración entre operación y gestion, el estándar abierto de intercambio de datos de equipos mineros para la telemetría cuando el fabricante lo soporte, y el protocolo de gases de efecto invernadero para la huella de carbono.
- **RF-INT-014** *(Sistema · C-12 · Etapa 1)* — Registrar el resultado de cada intercambio con un sistema externo, con reintento automático y alerta cuando la integración falle de forma persistente.
- **RF-INT-015** *(Sistema · C-18 · Etapa 2)* — Integrar las plataformas y periféricos de monitoreo diferidos a la Etapa 2 -cámaras, plataforma de control de fatiga e instrumentación geotécnica y ambiental-, conservando el equipamiento existente y con adaptador reemplazable por proveedor.
- **RF-INT-016** *(Persona usuaria · C-12 · Etapa 1)* — Permitir a cada persona usuaria configurar sus preferencias de canal y de frecuencia de notificación, respetando las que el CLIENTE defina como obligatorias.
- **RF-INT-017** *(Sistema · C-12 · Etapa 1)* — Cumplir la normativa de comunicaciones comerciales en las notificaciones que la requieran y permitir la baja cuando corresponda, sin afectar las notificaciones operacionales y de seguridad.

### 2.18 Familia 17 · Analítica, indicadores y reporteria

**8 requerimientos** · Etapa 1: 7 · Etapa 2: 1 · Componentes: C-17, C-19 · Criterios de aceptación: n.º 4 · n.º 5 · n.º 11 · n.º 12

- **RF-ANA-001** *(Gerencia / Centro Integrado de Operaciones · C-19 · Etapa 1)* — Publicar los indicadores operacionales de turno -movimiento, tiempos de ciclo, disponibilidad, utilización, detenciones clasificadas y cumplimiento del plan- al cierre de cada turno, y los indicadores de gestion -costo por tonelada movida, costo por libra de cobre fino, conciliación y cumplimiento del plan- con la latencia comprometida y con acceso al dato de origen de cada cifra.
- **RF-ANA-003** *(Usuario de negocio · C-17 · Etapa 2)* — Permitir a un usuario de negocio construir y guardar consultas sobre los datos registrados y exportar su resultado en formatos abiertos y documentados, aptos para auditores externos y para el verificador tercero de la huella de carbono, sin requerir desarrollo del ADJUDICATARIO.
- **RF-ANA-004** *(Sistema · C-19 · Etapa 1)* — Separar el almacenamiento transaccional del analítico y permitir escalar la capacidad analítica sin afectar la latencia comprometida de los indicadores operacionales de turno ni degradar el registro operacional en terreno.
- **RF-ANA-005** *(Centro Integrado de Operaciones · C-19 · Etapa 1)* — Poner a disposición del Centro Integrado de Operaciones las vistas que hoy no pudieron trasladarse a Antofagasta por depender de información que solo existe en faena.
- **RF-ANA-008** *(Gerencia / Usuario de negocio · C-19 · Etapa 1)* — Conservar la trazabilidad del dato hacia su sistema de origen y permitir consultar cualquier indicador con el desglose hasta el registro individual que lo compone, indicando de qué sistema proviene cada valor y con qué marca de tiempo se obtuvo, sin que el usuario deba pedir un informe a otra área.
- **RF-ANA-009** *(Sistema · C-19 · Etapa 1)* — Calcular los indicadores por turno conforme a la definición operacional de turno de doce horas -de 08:00 a 20:00 y de 20:00 a 08:00- y a su comportamiento ante el cambio de horario.
- **RF-ANA-010** *(Persona usuaria · C-19 · Etapa 1)* — Ofrecer búsqueda global con indexación de texto completo, tolerancia a errores de escritura, filtros facetados y respeto del control de acceso de la persona que busca.
- **RF-ANA-011** *(Sistema · C-19 · Etapa 1)* — Procesar de forma asíncrona las exportaciones de gran volumen, notificando al completarse y sin bloquear la sesión de la persona usuaria.

### 2.19 Familia 18 · Administración, parametrización y auditoría

**15 requerimientos** · Etapa 1: 15 · Componentes: C-20 · Criterios de aceptación: n.º 8 · n.º 9 · n.º 10 · n.º 11 · n.º 12

- **RF-ADM-001** *(Sistema · C-20 · Etapa 1)* — Conservar una pista de auditoría inalterable de toda modificación de dato, con valor anterior y posterior, persona autora, instante y origen, verificable mediante mecanismo criptográfico de integridad.
- **RF-ADM-002** *(Administrador del CLIENTE · C-20 · Etapa 1)* — Permitir la parametrización de catálogos -incluido el de destinos de material- umbrales, taxonomías, reglas de negocio, plazos y destinatarios de alerta sin requerir desarrollo de software.
- **RF-ADM-003** *(Sistema · C-20 · Etapa 1)* — Mantener un registro de reglas de negocio propias de la industria -conciliación, liquidación, mezcla, clasificación de detenciones y habilitación de personas- documentadas y consultables desde la propia aplicación.
- **RF-ADM-004** *(Administrador del CLIENTE · C-20 · Etapa 1)* — Versionar toda regla de negocio y todo parámetro con vigencia temporal, y permitir recalcular periodos anteriores -conciliación, clasificación de detenciones y cumplimiento- aplicando las reglas vigentes en el periodo recalculado, conservando la clasificación o el resultado anterior junto al nuevo y sin alterar los cierres ya emitidos.
- **RF-ADM-005** *(Sistema · C-20 · Etapa 1)* — Mantener el registro de supuestos declarados con su fundamento, su impacto si resulta equivocado y su instancia de validación, y exhibirlo junto al resultado en toda consulta que dependa de ellos -promedio ponderado en stocks, asignación integra al polígono de la pala con indicador de borde y prorrateo del tonelaje contra la balanza- de modo que la incertidumbre quede visible y auditable y no oculta detras de un cálculo.
- **RF-ADM-006** *(Administrador del CLIENTE · C-20 · Etapa 1)* — Administrar unidades organizacionales, catálogos y reglas de negocio independientes por unidad, de modo que su replicación a una nueva operación se realice por parametrización, sin modificación de código y sin afectar a las demás unidades.
- **RF-ADM-009** *(Sistema · C-20 · Etapa 1)* — Administrar los plazos de conservación por tipo de registro -seguridad y salud ocupacional 10 años, ambientales y de monitoreo 10 años, trazabilidad de mineral y embarques 10 años, auditoría 10 años, tributarios 7 años y operacionales 5 años- y permitir la eliminación anticipada o anonimización y ejecutar la eliminación o anonimización al vencimiento conforme a la Ley N.21.719.
- **RF-ADM-010** *(Sistema · C-20 · Etapa 1)* — Tratar los datos personales conforme a la Ley N.21.719, identificando al responsable del tratamiento -incluidos los datos de contratistas sin relación vigente con la faena- con base de licitud declarada, minimización, plazos definidos por tipo de dato y procedimiento de ejercicio de derechos.
- **RF-ADM-011** *(Administrador del CLIENTE / TI · C-20 · Etapa 1)* — Mantener el catálogo de dispositivos de terreno con su estado, su asignación a equipo, sitio o persona, su versión de software y su última sincronización, permitir su gestion remota e impedir la autenticación por acercamiento desde un dispositivo no inventariado.
- **RF-ADM-012** *(Comité de Operación / ADJUDICATARIO · C-20 · Etapa 1)* — Configurar y hacer cumplir la ventana operacional protegida, admitiendo intervenciones mayores solo en las detenciones programadas de planta de marzo y septiembre, menores previa aprobación entre las 03:00 y las 05:00, e impidiendo toda intervención y todo paso a producción durante el cierre contable del mes y durante las 48 horas previas a un embarque y el embarque mismo.
- **RF-ADM-013** *(Sistema · C-20 · Etapa 1)* — Exigir la aprobación de un segundo perfil para todo cambio de parámetro con impacto operacional, registrando la justificación del cambio.
- **RF-ADM-014** *(PROPONENTE · C-20 · Etapa 1)* — Declarar expresamente qué elementos de la solución son parametrizables y cuáles requieren desarrollo, manteniendo esa declaración actualizada en cada entrega.
- **RF-ADM-015** *(Persona usuaria responsable · C-20 · Etapa 1)* — Presentar a cada responsable una bandeja de tareas unificada con todas sus solicitudes pendientes, priorizadas y con alerta de vencimiento.
- **RF-ADM-016** *(Sistema · C-20 · Etapa 1)* — Generar documentos a partir de plantillas administrables por el CLIENTE, con datos de la transacción y salida en formato abierto.
- **RF-ADM-017** *(Sistema · C-20 · Etapa 1)* — Gestionar los documentos con versionado, metadatos, control de acceso, búsqueda por contenido y por metadato, y previsualización sin descarga.

### 2.20 Familia 19 · Migración, marcha blanca y puesta en producción

**6 requerimientos** · Etapa 1: 5 · Etapa 2: 1 · Componentes: C-05, C-09, C-21, C-22 · Criterios de aceptación: n.º 1 · n.º 2 · n.º 6 · n.º 11 · n.º 14

- **RF-MIG-001** *(ADJUDICATARIO · C-09 · Etapa 1)* — Recibir la migración de los datos históricos que sostienen la trazabilidad, la conciliación y la exposición ante la autoridad, según los plazos definidos -producción y trazabilidad 5 años, seguridad y salud ocupacional 10 años y laboratorio 5 años-, marcando cada registro migrado con su origen, su fecha de migración y su nivel de completitud, y distinguiéndolo del dato nacido en el sistema. Los históricos de mantenimiento y de monitoreo ambiental se migran en la Etapa 2 conforme a RF-MIG-008.
- **RF-MIG-003** *(ADJUDICATARIO / CLIENTE · C-22 · Etapa 1)* — Convivir con la forma actual de trabajar durante cada marcha blanca, permitiendo el registro paralelo y la conciliación entre ambos con posibilidad de volver atras, y medir diariamente los indicadores definidos para declararla cerrada, exhibiendo el avance contra el umbral comprometido.
- **RF-MIG-005** *(Sistema · C-21 · Etapa 1)* — Medir y publicar por área y por turno la adopción de las personas usuarias -tasa de uso del botón de cambio de material, proporción de registros nacidos digitales y proporción de turnos con operador identificado- contra la meta de adopción declarada en el Registro de supuestos SUP-27, exhibiendo el avance por área y activando el plan de refuerzo cuando el avance quede bajo la meta.
- **RF-MIG-006** *(ADJUDICATARIO · C-05 · Etapa 1)* — Permitir el despliegue y la reversión por área y por sitio, sin detener la operación de mina ni de planta, sin que un paso a producción afecte simultáneamente a la mina, la planta, la portería y el puerto, y con procedimiento de reversión probado.
- **RF-MIG-007** *(Sistema · C-21 · Etapa 1)* — Mantener en un repositorio de consulta de solo lectura los datos históricos que no se migran -trazabilidad y embarques del sexto al décimo año, mantenimiento anterior a tres años y laboratorio anterior a cinco-, con el mismo control de acceso por rol, la misma auditoría de consultas y el mismo tratamiento de dato sensible que el registro vivo, durante todo el plazo de retención del Cap. 15.
- **RF-MIG-008** *(Sistema · C-21 · Etapa 2)* — Recibir la migración de los datos históricos de mantenimiento -3 años- y de monitoreo ambiental -10 años- conforme al Cap. 15 del caso, con el mismo marcado de origen, fecha de migración y nivel de completitud de RF-MIG-001, en la Etapa 2 y junto con los módulos que los consumen.
---

## 3. Cómo interactúa cada actor con el software

### 3.1 Criterio de admisión y advertencia previa

Un actor entra en este apartado cuando el catálogo v2.4 le asigna al menos un requerimiento funcional, es decir, cuando existe una función que **él dispara o consume**. Los 105 requerimientos cuyo actor es «Sistema» no generan interacción con persona alguna: se ejecutan sobre información que los sistemas del CLIENTE ya producen.

Cada ficha declara seis cosas: qué hace la persona, con qué requerimiento, sobre qué componente, por qué canal, con qué umbral comprometido y qué ocurre con esa interacción durante un corte de enlace. Se declara además **qué no hace**, porque en este caso la restricción n.º 8 —no se agregan pasos al ciclo productivo— convierte lo que no se pide en parte de la especificación.

**Observación sobre el catálogo v2.4.** La columna «Actor» usa etiquetas no normalizadas para una misma persona: «Supervisor de turno», «Supervisor» y «Supervisor de terreno» designan al mismo rol; «Área de seguridad y salud ocupacional» aparece con dos capitalizaciones distintas; y «Laboratorio» designa unas veces al sistema de gestión de laboratorio y otras al laboratorio externo, que son cosas distintas y con acceso distinto. Este documento normaliza las etiquetas y la corrección debe llevarse al catálogo antes del Informe 2.

### 3.2 Actores de terreno

#### 3.2.1 Persona operadora de equipo

Es el actor decisivo del proyecto. El criterio de aceptación n.º 14 se enuncia sobre él —«el operador de pala usa la solución en su turno, con guantes, y la usa por decisión propia»— y el CLIENTE declara haber visto fracasar dos proyectos correctamente implantados por falta de adopción. Toda la especificación de terreno se subordina a esa frase.

**Interacción total que se le pide en un turno de doce horas: dos gestos.** Acercar la credencial al relevo y, sólo el operador de pala, presionar un botón cuando el material cargado no corresponde al planificado.

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Abre y cierra su sesión acercando la credencial de proximidad en el punto de relevo | RF-CAR-009 | C-03, C-05 | Lector en cabina | 1 segundo desde la lectura hasta la apertura de sesión, sin retirar el guante (RNF-DES-005, SUP-29) | Opera íntegro contra la caché local del nodo de borde |
| Cuando el relevo ocurre sin señal, se cierra localmente la sesión saliente y se abre la entrante, sin fusionar sus eventos | RF-DES-004 | C-04 | Dispositivo de cabina | Cero eventos atribuidos a la persona equivocada | Es precisamente el caso previsto |
| Se autentica con el solo acercamiento de la credencial a un dispositivo previamente inventariado | RF-IDE-003 | C-05 | Lector de proximidad | Ningún dispositivo no inventariado admite credencial | Válido con caché de habilitaciones vigentes (RF-DES-006) |
| **Sólo pala:** presiona un botón único en cabina para avisar que el material cambió respecto de lo planificado, sin seleccionar causa ni escribir | RF-CAR-003 | C-10, C-01 | Botón único en cabina | Una acción, no superior a 2 segundos, con confirmación visible en 1 segundo (RNF-DES-004) | Se encola firmado y se reconcilia al recuperar el enlace |
| **Sólo perforadoras no instrumentadas:** registra el avance del turno desde la aplicación móvil, con número acotado de acciones | RF-PET-002 | C-04 | Aplicación de terreno | Reemplaza la planilla que hoy se entrega al final del turno | Diseñado para operar desconectado |
| Levanta un aviso de falla desde la cabina con una sola acción, generando la orden de trabajo sin que un planificador la transcriba | RF-CAR-015 | C-01, C-12 | Aplicación de terreno o botón | Una acción | Se encola y se sincroniza |
| Registra cualquier evento de terreno con confirmación visible del guardado | RF-DES-001 | C-04 | Aplicación de terreno | 2 segundos en el percentil 95 (RNF-DES-001) | Registro local firmado y cifrado a nivel de campo |

**Qué no hace, y está exigido que no haga.** No clasifica detenciones, ni en cabina ni en ningún otro dispositivo: es una prohibición expresa del catálogo (RF-DET-004) y de la regla RN-17. No escribe texto libre. No selecciona la causa del cambio de material: eso lo resuelve el despachador. No lleva planilla de stock. No introduce contraseña alfanumérica, que RT-12.11 prohíbe en terreno. Y si nadie se identifica en el relevo, **el equipo no se detiene**: el viaje se marca como operador no identificado y se alerta al supervisor (RF-CAR-011, restricción n.º 1, RN-22).

**Condiciones físicas que la interfaz debe sostener.** Operable con guante de invierno grueso, con tasa de éxito al primer intento superior al 95 % (RNF-USA-001); legible bajo radiación solar directa al mediodía, con luminancia igual o superior a 1.000 cd/m² (RNF-USA-002); funcionamiento entre −6 °C y 34 °C con resistencia a polvo y vibración (RNF-USA-003); objetivos táctiles de al menos 44 × 44 píxeles (RNF-USA-014); autonomía de batería suficiente para un turno de doce horas (RNF-DES-014); capacitación no superior a 30 minutos por persona operadora (RNF-MAN-007).

#### 3.2.2 Supervisor de turno

Es el actor con más carga de interacción de toda la solución y el destinatario del criterio de aceptación n.º 6.

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Recibe **únicamente** las detenciones que la clasificación automática no resolvió, y las clasifica desde lista cerrada, sin texto libre | RF-DET-003 | C-15 | Tablero o aplicación | Residuo disponible antes del cierre del turno en que ocurrió (RNF-DES-011) | La clasificación automática se recalcula al reconciliar |
| No puede cerrar el turno con detenciones sin clasificar; las pendientes escalan a la guardia siguiente con constancia | RF-DET-005 | C-15 | Tablero | Cero detenciones sin clasificar al cierre (RN-18) | Opera con el estado local del turno |
| Ante un equipo detenido, dispone en **una sola vista** del estado, el historial reciente, las alarmas de telemetría y las pautas pendientes | RF-MTO-002 | C-16 | Vista única | Menos que los 15 a 20 minutos y cuatro sistemas actuales | Degradación explícita: la vista se arma con las fuentes disponibles (RNF-INT-004) |
| Registra incidentes, cuasi accidentes, observaciones de conducta e inspecciones **en el lugar del hecho** | RF-SSO-001 | C-04, C-18 | Aplicación de terreno | Nacen digitales; hoy son papel digitado días después | Registro local firmado; es función crítica de terreno |
| Gestiona con responsable, plazo, estado y escalamiento la acción tomada ante cada alerta de fatiga y cada alarma crítica | RF-SSO-004 | C-18 | Bandeja de tareas | Residuo de alertas sin acción como indicador visible | La alerta se encola; la acción se registra al reconciliar |
| Es alertado cuando nadie se identifica en el relevo de un equipo | RF-CAR-011 | C-01 | Notificación | El equipo nunca se detiene | Alerta local |

**Qué no hace.** No reconstruye la planilla de stock: los aportes y retomas se registran como eventos y el saldo, la ley ponderada y la composición por polígono se mantienen en línea (RF-STK-001, RF-STK-002). No clasifica la totalidad de las detenciones, sólo el residuo. No digita formularios de papel.

#### 3.2.3 Persona de control de acceso en portería

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Valida la habilitación vigente de cada persona en el torniquete | RF-IDE-007 | C-05 | Torniquete y lector | 1,5 segundos, incluido el modo degradado con caché local (RNF-DES-003, RN-20) | Opera contra la caché local de habilitaciones vigentes (RF-DES-006) |
| Registra el ingreso y la salida con persona, empresa, marca de tiempo y resultado | RF-IDE-011 | C-05 | Torniquete | Inventario de personas presentes en faena en línea | Registro local en el gabinete de borde de portería |
| Advierte en el momento del ingreso el documento por vencer o vencido | RF-IDE-008 | C-05 | Pantalla de portería | La alerta anticipada es lo que sostiene el criterio n.º 7 | La advertencia en portería sigue; la alerta anticipada se emite al restablecerse |

**Qué no hace.** No acredita a una persona contratista nueva durante un corte de enlace: esa alta requiere validación documental contra el maestro central y contra el ERP, y se declara expresamente como función no disponible sin enlace. La persona ya acreditada **sí** ingresa. No captura biometría: está fuera del Contrato por el Artículo 72.º, con el punto de extensión previsto en RF-IDE-013.

#### 3.2.4 Persona que toma la muestra

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Registra la cadena de custodia de cada muestra: quién la tomó, cuándo, en qué punto, bajo qué protocolo, con qué equipo y quién la recibió | RF-LAB-004 | C-09 | Aplicación de terreno | Del orden de 71.000 muestras al año | Opera desconectado; la toma de muestra es función crítica de terreno |
| Durante la marcha blanca, registra la campaña de verificación de la ley en retoma de stock, con la ley medida y la que predecía el promedio ponderado | RF-STK-006 | C-09 | Aplicación de terreno | Valida o refuta el supuesto del promedio ponderado | Se encola |

**Dónde se le pide y dónde no.** Los puntos de muestreo obligatorios están ubicados donde el material ya se detiene o donde lo opera un rol distinto del ciclo de mina —chancado, alimentación, concentrado, relave—, **nunca en el ciclo de pala o de camión** (RF-LAB-010, RN-12). En los muestreadores de proceso automáticos el dato se captura por instrumentación y la acción física de muestreo se separa del registro del dato, que no nace de una digitación (RF-LAB-012).

#### 3.2.5 Responsable de tronadura

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Registra la liberación de un sector para carguío tras la tronadura, con marca de tiempo y responsable, y el sistema lo notifica automáticamente al despacho y a la operación mina | RF-PLA-009 | C-02 | Aplicación de terreno | Reemplaza la coordinación por radio | Se encola con sello de tiempo local |

### 3.3 Actores de operación, planificación y oficina

#### 3.3.1 Despachador

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Al reasignar el destino de un camión, selecciona la razón del cambio desde una **lista cerrada**, de modo que la razón deje de perderse en la radio | RF-CAR-002 | C-10 | Consola del Centro Integrado de Operaciones | Una selección; sin texto libre | El Centro Integrado de Operaciones depende íntegramente del enlace; durante un corte el nodo de borde sigue registrando el viaje y la razón se completa al restablecerse |

Es la única interacción que la solución le pide, y es deliberadamente mínima: el sistema de despacho de flota **se mantiene** por la restricción n.º 5 y tiene contrato vigente hasta 2031. La solución lo lee —asignaciones, tiempos de ciclo, tonelaje de a bordo y posición— y no lo sustituye ni lo modifica (RF-CAR-001). Lo que se agrega es el dato que hoy se pierde: el porqué.

#### 3.3.2 Superintendencia de Planificación Minera

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Registra los cambios de plan que ocurren durante la semana —caída de rampa, desplazamiento del contacto tras una tronadura, cambio de mezcla de planta— generando la versión en el momento del cambio | RF-PLA-003 | C-02 | Portal interno | La radio y la mensajería del turno dejan de ser la única constancia | Función de oficina; se posterga |
| Consulta el cumplimiento del plan del turno anterior al inicio de cada turno | RF-PLA-005 | C-08, C-19 | Tablero | 30 minutos posteriores al cierre del turno (RNF-DES-010); hoy son 10 a 15 días | Los indicadores se recalculan al reconciliar |

**Cómo entra su trabajo al sistema.** El plan se sigue construyendo en el software de planificación minera, que **se mantiene** por la restricción n.º 7. La solución consume sus salidas automáticamente, sin exportación a carpeta compartida ni carga manual (RF-PLA-001), y no interviene el modelo de bloques ni la estimación de recursos y reservas, excluidos por el Capítulo 11 del caso.

#### 3.3.3 Metalurgia

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Registra los factores de ajuste del balance metalúrgico del mes como parámetros versionados, con vigencia, responsable y fundamento documentado | RF-PES-010 | C-08 | Consola de parametrización | La explicación deja de depender de dos personas | Función de oficina |
| Consume el balance del turno y la conciliación del mes calculados desde los datos registrados | RF-PES-009, RF-PES-007 | C-08 | Tablero | Conciliación disponible en 3 días hábiles desde el cierre (RNF-DES-008); hoy son 18 días hábiles | Se posterga: el cierre requiere las tres mediciones y el resultado de laboratorio |

Reemplaza dos planillas: la del metalurgista de turno y la consolidada del mes. La diferencia residual queda atribuida a un punto de medición de la cadena y ninguna diferencia se cierra como no explicada (RN-06, criterio n.º 3).

#### 3.3.4 Mantenimiento y Superintendencia de Planta

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Configura, por tipo de alarma, el umbral, la criticidad, el destinatario por rol y área, el canal de notificación y la supresión temporal | RF-MTO-005 | C-16 | Consola de administración | Controla el volumen de alertas para que no se vuelvan ruido | Función de oficina |
| Registra la causa raíz de las detenciones no programadas de planta y de equipo mina, con responsable, acción y estado de cierre | RF-DET-006 | C-15 | Tablero | Hoy el 61 % de las detenciones no tiene causa raíz | Se posterga |
| Consulta la vista única de equipo y los indicadores de disponibilidad física, utilización efectiva y tiempo medio entre falla | RF-MTO-002, RF-DET-007 | C-16, C-15 | Vista única y tablero | Reemplaza tres portales de fabricante y una consulta a SAP | Degradación explícita |

**Qué no cambia.** Las pautas preventivas y las órdenes de trabajo siguen viviendo en el ERP: la solución las obtiene y las asocia al equipo **sin duplicar su registro** (RF-MTO-003), porque la restricción n.º 4 prohíbe dos verdades contables.

#### 3.3.5 Área de Seguridad y Salud Ocupacional

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Recibe y explota el reporte de conciliación entre alertas de fatiga emitidas y acciones registradas, con el residuo sin acción documentada como indicador | RF-SSO-009 | C-18 | Tablero | Criterio n.º 10: toda alerta de fatiga con acción documentada y trazable | La alerta se encola |
| Gestiona junto al supervisor el flujo de acción ante cada alerta de fatiga y cada alarma crítica | RF-SSO-004 | C-18 | Bandeja de tareas | Responsable, plazo, estado y escalamiento automático | La acción se registra al reconciliar |
| Recibe la alerta anticipada de vencimiento de habilitación, junto con la persona y su empleador | RF-IDE-008 | C-05 | Notificación | Anticipación configurable; sostiene el criterio n.º 7 | Se emite al restablecerse |
| Interviene en el flujo de acreditación de una persona contratista nueva | RF-IDE-009 | C-06 | Portal | Menos de 48 horas, contra los 9 días actuales | El alta se suspende durante el corte |
| Recibe el reporte de indicios de uso indebido o préstamo de credenciales, detectado sobre los eventos de acceso ya registrados | RF-IDE-012 | C-05 | Tablero | Sin captura adicional: analítica sobre datos existentes | Se calcula al reconciliar |

**Cómo se protege el dato.** Los datos de salud ocupacional, exámenes preocupacionales, control de fatiga y control de alcohol y drogas llevan cifrado a nivel de campo (RF-SSO-007, RT-11.10) y toda consulta y toda modificación quedan registradas (RF-IDE-020). El dato de fatiga se trata conforme a la Ley N.º 21.719 con finalidad, base de licitud, destinatarios autorizados y plazo de conservación declarados (RF-SSO-010).

#### 3.3.6 Área de Sustentabilidad

Es el actor con más requerimientos propios después del área comercial, y el único que concentra los seis en un solo componente.

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Registra cada compromiso de la Resolución de Calificación Ambiental con su frecuencia, su punto de medición y su destinatario, y recibe la alerta de proximidad del vencimiento | RF-AMB-002 | C-17 | Portal interno | Ningún compromiso vence sin aviso | Función de oficina |
| **Genera los reportes a la autoridad ambiental desde el propio sistema**, con trazabilidad desde el valor reportado hasta la medición de origen | RF-AMB-003 | C-17 | Portal interno | Criterio n.º 11; RN-25 prohíbe construir el reporte en planilla al margen | Se posterga: es destino externo |
| Calcula la huella de carbono por tonelada de cobre fino con alcances 1, 2 y 3, con periodicidad mensual | RF-AMB-006 | C-17 | Tablero | Criterio n.º 12: mensual y auditable; hoy es anual con consultora externa | Se posterga |
| Produce la información trazable de avance de la operación y obras comprometidas que exige la normativa de cierre de faenas | RF-AMB-013 | C-17 | Portal interno | Cuarto frente de cumplimiento del Capítulo 12 del caso | Se posterga |
| Reconstruye hacia atrás la serie mensual de huella sobre los datos ya registrados, aplicando a cada período los factores vigentes en ese momento | RF-AMB-014 | C-17 | Tablero | Depende de RN-26 y RN-27: factores versionados con vigencia | Se posterga |
| Genera el expediente de evidencia auditable para el verificador tercero | RF-AMB-015 | C-17 | Exportación | Datos de actividad, factores versionados y memoria de cálculo | Se posterga |

**De dónde salen sus datos sin pedirle planillas a nadie.** La instrumentación ambiental y geotécnica se captura automáticamente (RF-AMB-001); los datos de compras locales y empleo local se obtienen del ERP y de recursos humanos sin solicitar planillas semestrales (RF-AMB-008); los informes de laboratorios externos y de la empresa geotécnica se asocian al compromiso conservando el documento original y extrayendo el valor (RF-AMB-004).

#### 3.3.7 Área comercial

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Reconstruye el origen de cualquier lote embarcado: polígonos y su proporción, fechas de extracción, stocks recorridos, fecha de procesamiento y análisis | RF-TRZ-003 | C-01 | Portal interno | 2 horas para la reconstrucción completa; 30 segundos en el percentil 95 para la consulta en línea (RNF-DES-009). Hoy son 4 a 6 semanas | Se posterga; el registro sigue naciendo en terreno |
| Genera el expediente de trazabilidad de lote exportable, con versión de reglas y supuestos aplicados y **firma electrónica** | RF-TRZ-010 | C-07 | Exportación | Apto para el comprador y para auditoría | Se posterga |
| Registra la conformación del lote comercial indicando qué lotes del acopio lo componen y en qué proporción, con relación padre-hijo explícita | RF-LOT-003 | C-01 | Portal interno | Es lo que hace reconstruible el árbol del lote | Se posterga |
| **Aprueba la publicación** de información al portal del comprador | RN-16, SUP-17 | C-07 | Portal interno | Ninguna publicación ocurre sin esa aprobación, y toda consulta queda registrada | Se posterga |
| Registra la liquidación del embarque: ley pagable, deducciones por humedad, cargos de tratamiento y refinación y penalidades por elementos deletéreos | RF-LOT-008 | C-07 | Portal interno | Sobre un vector de elementos, no sobre la ley de cobre sola (RN-14) | Se posterga |
| Modela el concentrado devuelto o rechazado como evento de reversa que reabre la trazabilidad del lote original | RF-LOT-009 | C-07 | Portal interno | Genera lote de reingreso con su propia trazabilidad | Se posterga |
| Registra la interacción con la nave, el agente de aduana y el organismo fiscalizador de cada embarque | RF-LOT-013 | C-01 | Portal interno | Conserva los documentos intercambiados | Se posterga |
| Recibe la propuesta de combinaciones de material del acopio que mantienen los elementos deletéreos bajo el límite contractual | RF-LOT-006 | C-01 | Apoyo a la decisión | Es apoyo, no decisión automática | Se posterga |

**La frontera más importante del sistema.** La ley por polígono **no se expone en ningún caso** al comprador, bajo ningún perfil (RNF-SEG-012, RN-16). El área comercial es la que sostiene esa frontera y la que autoriza cada publicación.

#### 3.3.8 Administrador del CLIENTE

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Gestiona perfiles y autorizaciones por rol, área y sitio bajo mínimo privilegio, permitiendo que una misma persona tenga atribuciones distintas en faena, en el Centro Integrado de Operaciones y en la oficina corporativa | RF-IDE-018 | C-05 | Consola de administración | Doble factor resistente a suplantación tipo FIDO2 o clave de acceso (RT-12.04) | Función de oficina |
| Parametriza catálogos —incluido el de destinos de material—, umbrales, taxonomías, reglas de negocio, plazos y destinatarios de alerta **sin requerir desarrollo** | RF-ADM-002 | C-20 | Consola de administración | La declaración de qué es parametrizable y qué requiere desarrollo se mantiene actualizada en cada entrega (RF-ADM-014) | Función de oficina |
| Versiona toda regla y todo parámetro con vigencia temporal, y recalcula períodos anteriores con la versión que regía entonces | RF-ADM-004 | C-20 | Consola de administración | RN-27; es lo que hace verificables los criterios n.º 3 y n.º 4 sobre historia ya cerrada | Función de oficina |
| Administra unidades organizacionales y catálogos independientes por unidad, de modo que replicar a otra operación sea parametrización y no desarrollo | RF-ADM-006 | C-20 | Consola de administración | Replicación por parametrización | Función de oficina |
| Mantiene el catálogo de dispositivos de terreno con estado, asignación, versión de software y última sincronización, y los gestiona en forma remota | RF-ADM-011 | C-20 | Consola de administración | 450 dispositivos a tres años (RNF-CAP-007) | El panel muestra la antigüedad de la última sincronización |

**Control que se aplica sobre él mismo.** Todo cambio de parámetro con impacto operacional exige la **aprobación de un segundo perfil**, con justificación registrada (RF-ADM-013, RT-16.03). Los accesos privilegiados se gestionan con elevación temporal a demanda, aprobación previa y grabación de sesión (RF-IDE-022). Y existe un ambiente de simulación para probar el efecto de un cambio de parámetro antes de aplicarlo a producción, que en este caso se ejecuta **sobre una copia de historia ya cerrada**.

#### 3.3.9 Centro Integrado de Operaciones y equipo de tecnologías de información

El Centro Integrado de Operaciones es un **sitio**, no una persona: sala de control integrada de mina y planta, en Antofagasta, a 145 km de la faena, hoy al 40 % de su capacidad instalada. El catálogo lo usa como etiqueta de actor para designar a quienes trabajan allí —despacho, planificación de corto plazo y análisis—, ya cubiertos en las fichas anteriores. Lo propio de esta ficha son dos funciones que no pertenecen a ningún otro rol.

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Consulta el panel de estado de sincronización por dispositivo y por sitio: eventos pendientes, antigüedad del más antiguo e instante de la última sincronización exitosa | RF-DES-011 | C-04 | Panel de operación | Es el instrumento con que se verifica el criterio n.º 13 | El panel es justamente lo que se mira durante el corte |
| Dispone de las vistas que hoy no pudieron trasladarse a Antofagasta por depender de información que sólo existe en faena | RF-ANA-005 | C-19 | Tablero | Resultado que el CLIENTE espera aunque no lo declara como requerimiento | Los indicadores se recalculan al reconciliar |

El equipo de tecnologías de información del CLIENTE **son once personas** (restricción n.º 10). La solución debe ser operable en régimen por ellas y toda función que requiera un especialista dedicado que la compañía no tiene se ofrece como servicio y está costeada (RNF-MAN-001). El diagnóstico de incidentes se realiza sin acceder a la base de datos productiva (RNF-MAN-005).

#### 3.3.10 Gerencia y personas usuarias de negocio

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Recibe los indicadores operacionales de turno al cierre de cada turno | RF-ANA-001 | C-19 | Tablero | 15 minutos de latencia para indicadores operacionales; 4 horas para los de gestión (RNF-DES-006) | Se recalculan al reconciliar |
| Construye y guarda consultas propias sobre los datos registrados y exporta el resultado en formatos abiertos y documentados | RF-ANA-003 | C-19 | Autoservicio | Aptos para auditoría | Se posterga |
| Consulta cualquier indicador con desglose hasta el registro individual que lo compone, indicando el sistema de origen | RF-ANA-008 | C-19 | Tablero | Es la propiedad que hace auditable el indicador | Se posterga |
| Usa búsqueda global con tolerancia a errores de escritura y filtros facetados, respetando su control de acceso | RF-ANA-010 | C-19 | Buscador | Respeta el control de acceso de quien busca | Se posterga |
| Configura sus preferencias de canal y frecuencia de notificación, respetando las que el CLIENTE define como obligatorias | RF-INT-016 | C-12 | Preferencias | Correo, notificación en aplicación, mensajería y mensaje de texto para alertas críticas (RF-INT-010) | Notificación diferida |
| Dispone de una bandeja de tareas unificada con sus solicitudes pendientes, priorizadas y con alerta de vencimiento | RF-ADM-015 | C-20 | Bandeja | Una sola bandeja para todos los flujos | Se posterga |

Las exportaciones de gran volumen se procesan de forma asíncrona, notificando al completarse y sin bloquear la sesión (RF-ANA-011). El CLIENTE dispone además de acceso propio y permanente a los tableros, con datos en tiempo real y capacidad de exportación, sin intervención del ADJUDICATARIO (RT-14.02).

### 3.4 Personas usuarias externas

El RT-12.12 fija exactamente cuáles son en este caso: **las empresas contratistas y sus trabajadores, el operador del terminal portuario, los laboratorios externos y los compradores**. Son cuatro y son las cuatro que siguen. Toda su interacción atraviesa la capa de borde y ninguna otra: no existe portal abierto a la ciudadanía (RF-POR-006).

#### 3.4.1 Empresa contratista

Unas 90 empresas activas, con 2.400 contratistas permanentes y hasta 4.300 adicionales en las detenciones mayores de marzo y septiembre.

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Administra desde un portal autenticado las identidades de sus trabajadores, carga su documentación de habilitación y sigue el estado de cada acreditación | RF-IDE-005 | C-06 | Portal en el borde público | Credencial de empresa con doble factor y administración delegada | El portal es de oficina; no depende del enclave de terreno |
| Completa el flujo de acreditación de una persona nueva, con estados, responsables, plazos y escalamiento | RF-IDE-009 | C-06 | Portal o notificación | **Menos de 48 horas**, contra los 9 días actuales (criterio n.º 8) | El alta se suspende durante el corte |
| Adjunta el documento de renovación **desde la propia notificación**, sin entrar al portal | RF-IDE-008, RT-16.26 | C-06 | Notificación accionable | Es la vía más corta al criterio n.º 7 | Diferido |

**Por qué la protección anti-bot es escalonada y nunca por defecto.** Quien usa este portal es personal administrativo de empresas pequeñas, con conectividad y dispositivos desiguales. Un reto agresivo se traduce en acreditaciones que no se completan y en el criterio n.º 8 incumplido. Primero límite de tasa por credencial, luego reto interactivo, y sólo después bloqueo; nunca a una persona autenticada con sesión vigente.

**Su credencial física es su identidad digital.** La credencial de proximidad emitida en la acreditación de portería se usa como identidad digital de la persona contratista, sin crear un segundo sistema de identidad (RF-IDE-002).

#### 3.4.2 Comprador

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Consulta en un portal autenticado el certificado derivado de su lote: origen por sector, fechas, stocks recorridos, análisis y cadena de custodia | RF-POR-001 | C-07 | Portal en el borde público, Etapa 2 | Credencial nominada por comprador; **toda consulta queda registrada**; límite de tasa y cuota por consumidor | Se posterga |
| Aporta el resultado de su propio laboratorio y, ante discrepancia, el del tercer laboratorio árbitro, que quedan asociados al lote y a su liquidación | RF-LAB-014 | C-09 | Integración o carga | Es la práctica de comercialización de concentrados | Se posterga |

**Lo que nunca ve.** La ley por polígono, bajo ningún perfil (RNF-SEG-012). Se le entrega origen por sector, no la cadena completa (RN-16, SUP-17). Es información competitivamente sensible del yacimiento y su exposición sería un daño irreversible.

#### 3.4.3 Operador del terminal portuario

El recinto es administrado por un tercero y su operación **no se pide** —Capítulo 11 del caso—, «sin perjuicio de la integración necesaria con él».

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Accede en forma autenticada y acotada, limitado a la información que su función requiere | RF-POR-005 | C-09 | Portal en el borde público | Mínimo privilegio: sólo lo que su función requiere | El gabinete de borde del acopio de puerto registra y reconcilia de forma autónoma |
| Registra la recepción y el acopio del concentrado por lotes, con tonelaje, humedad y fecha | RF-LOT-002 | C-08 | Portal o integración | Aun cuando el recinto sea administrado por un tercero | Operación autónoma durante la indisponibilidad, sin pérdida de registros (RNF-DIS-009) |
| Participa del muestreo del lote comercial conforme al protocolo acordado con el comprador, con cadena de custodia, humedad y ley por elemento | RF-LOT-004 | C-07 | Aplicación o portal | La cadena de custodia no se interrumpe al cambiar de recinto | Se encola localmente |

El acopio de puerto es el sitio más expuesto de la solución: la conectividad la provee un operador sin acuerdo de nivel de servicio con la compañía (SUP-19), y por eso se diseña para no depender de él.

#### 3.4.4 Laboratorios externos y empresa geotécnica

| Qué hace | Requerimiento | Componente | Canal | Umbral | Sin enlace |
|---|---|---|---|---|---|
| Accede en forma autenticada y acotada a la información que su función requiere | RF-POR-005 | C-09 | Portal en el borde público | Mínimo privilegio | Se posterga |
| Entrega los informes de monitoreo ambiental y geotécnico, que quedan asociados al compromiso correspondiente conservando el documento original y extrayendo el valor | RF-AMB-004 | C-09 | Carga o integración | Sostiene la trazabilidad hasta la medición de origen del criterio n.º 11 | Se posterga; el monitoreo del depósito de relaves se almacena localmente y se sincroniza (RF-AMB-009) |

**No confundir con el sistema de gestión de laboratorio.** El laboratorio químico de la faena opera sobre el sistema de gestión de laboratorio del CLIENTE, que **se mantiene** por la restricción n.º 6 y se integra en ambos sentidos (RF-LAB-001, componente C-09): eso es una integración entre sistemas, no una persona usuaria externa. Los laboratorios externos de esta ficha son terceros distintos —árbitro comercial, laboratorios ambientales— y son los que el RT-12.12 nombra.

### 3.5 Quiénes aparecen como actor y no son personas usuarias del software

| Etiqueta | Qué es en realidad | Consecuencia |
|---|---|---|
| Sistema | Actor de 105 de los 167 requerimientos funcionales | No interactúa: se ejecuta. Es la evidencia de que la solución no descansa en captura manual |
| Laboratorio | El sistema de gestión de laboratorio del CLIENTE, integración bidireccional C-09 | Es una caja de sistema externo, no una columna de actor |
| Autoridad ambiental | **Destinataria** del reporte que el sistema genera y que el Área de Sustentabilidad presenta (RF-AMB-003) | No tiene sesión ni superficie expuesta. No figura entre las personas usuarias externas del RT-12.12 |
| Centro Integrado de Operaciones | Uno de los seis emplazamientos del Capítulo 3 del caso | Pertenece a la vista de despliegue, no a la de actores |
| ADJUDICATARIO, PROPONENTE, Jefatura de Proyecto, Comité de Operación | Obligaciones contractuales y de la propuesta: estudio de cobertura, catálogo de integraciones, ventana operacional protegida, migración, marcha blanca, registro de riesgos | Se cumplen como entregable o como gobierno del Contrato, no como pantalla |

---

## 4. Flujo de trabajo del nuevo software

### 4.1 Las cuatro propiedades que gobiernan todos los flujos

Antes de los flujos concretos, las cuatro propiedades que se repiten en todos ellos y que explican por qué el flujo es como es:

1. **El hecho nace en el dispositivo.** Todo hecho de terreno se genera en un dispositivo previamente inventariado, firmado, cifrado a nivel de campo y con identificador de evento generado localmente (RF-DES-001, RF-DES-002). No existe una segunda vía de captura. Esto es lo que le da valor probatorio al registro y lo que hace posible el criterio n.º 1.
2. **La corrección es un evento nuevo.** La trazabilidad se almacena como registro inmutable de eventos: toda corrección se expresa como un evento posterior y ninguna altera el evento original (RF-TRZ-005, RN-01).
3. **Ningún componente consulta la base de datos de otro.** Si un componente necesita un dato, se suscribe a su evento. El contexto analítico sólo consume eventos publicados; si un indicador necesita un dato que nadie publica, la corrección es publicar el evento.
4. **Todo cálculo se hace contra la versión de la regla que regía en ese momento.** Toda regla y todo parámetro tienen vigencia temporal versionada, y un período anterior se recalcula con la versión vigente entonces (RF-ADM-004, RN-27).

### 4.2 Flujo 1 — Del plan semanal al sector liberado para carguío

**Hoy:** el planificador exporta un archivo cada domingo a una carpeta compartida, la operación lo carga a mano en el despacho, los cambios de la semana se comunican por radio y por el grupo de mensajería, y nadie lleva registro consolidado de cuántas veces se modificó el plan ni por qué.

1. El software de planificación minera publica o modifica el plan. La solución **consume automáticamente** polígonos con identificador, banco, geometría, ley estimada por elemento, tonelaje planificado y destino asignado, sin exportación a carpeta ni carga manual (RF-PLA-001, C-11).
2. El plan se almacena como entidad **versionada e inmutable**: cada publicación crea una versión con marca de tiempo, autor y motivo, y ninguna versión publicada admite alteración retroactiva (RF-PLA-002, C-02).
3. Se registra la ley estimada del modelo de bloques asociada a cada polígono, que queda como referencia para la reconciliación posterior contra la ley medida (RF-PLA-011).
4. Durante la semana, la Superintendencia de Planificación Minera registra cada cambio —caída de rampa, desplazamiento del contacto, cambio de mezcla— generando la versión en el momento en que ocurre (RF-PLA-003).
5. Tras la tronadura, el responsable registra la liberación del sector con marca de tiempo y responsable; el sistema lo notifica automáticamente al despacho y a la operación mina (RF-PLA-009).
6. **Cada movimiento de material posterior queda asociado automáticamente a la versión del plan vigente en el instante en que ocurrió**, sin intervención de la persona operadora ni del despachador (RF-PLA-004). Éste es el paso que hace verificable el criterio n.º 4.
7. Al inicio de cada turno, el cumplimiento del plan del turno anterior está disponible, calculado contra la versión efectivamente vigente en cada movimiento (RF-PLA-005), dentro de los 30 minutos posteriores al cierre del turno (RNF-DES-010).

### 4.3 Flujo 2 — Perforación y tronadura

1. Las dos perforadoras instrumentadas entregan automáticamente posición, profundidad y parámetros de cada pozo, asociados a la malla y al polígono (RF-PET-001).
2. Las siete no instrumentadas: la persona operadora registra el avance del turno desde la aplicación móvil, con operación desconectada y un número acotado de acciones. Desaparece la planilla de fin de turno (RF-PET-002).
3. Los datos de perforación quedan disponibles para geología en formato estructurado, procesable y exportable, **sin intervención del área de perforación y tronadura** (RF-PET-003). Es la corrección de un problema declarado: geología los pidió tres veces y en dos ocasiones los recibió en un formato que no pudo procesar.
4. En Etapa 2, de esos mismos parámetros se derivan los índices de dureza del macizo rocoso y se ponen a disposición de geología y planificación (RF-PET-006).
5. Cada tronadura se registra con su malla, ventana programada, ejecución efectiva y sector afectado; se **asocia automáticamente a las detenciones de equipo que provoca** y se emite el aviso de evacuación por los canales definidos, incluida la integración con radio y megafonía (RF-PET-004). La asociación automática es lo que evita que esas detenciones lleguen al supervisor como residuo por clasificar.

### 4.4 Flujo 3 — Ciclo de carguío y transporte: el flujo central

Del orden de **342.000 viajes al año**, unos 940 diarios, con proyección de 367.000 a tres años, y 1,37 millones de ciclos de carguío de pala.

1. La persona operadora acerca su credencial de proximidad en el punto de relevo. La sesión se abre **en 1 segundo, sin retirar el guante** (RF-CAR-009, RNF-DES-005). Si nadie se identifica, el viaje se marca como operador no identificado y se alerta al supervisor: **el equipo no se detiene** (RF-CAR-011, restricción n.º 1).
2. El sistema de despacho de flota asigna camión a pala y a destino, y la solución obtiene de él asignaciones, tiempos de ciclo, tonelaje de pesaje de a bordo y posición, sin sustituirlo ni modificarlo (RF-CAR-001, C-10, restricción n.º 5).
3. Cada viaje se registra como **unidad atómica de trazabilidad**, con identificador único, camión, pala de origen, polígono, destino efectivo de descarga y persona operadora identificada (RF-TRZ-001). El polígono es atributo del viaje; turno, lote de chancado y lote comercial son agregaciones derivadas (RF-TRZ-002, RN-01).
4. Si el operador de pala observa que el material no corresponde al planificado, **presiona el botón único**: una acción, no más de 2 segundos, con confirmación visible en 1 segundo (RF-CAR-003, RNF-DES-004).
5. El despachador reasigna el destino y **selecciona la razón desde una lista cerrada** (RF-CAR-002). La razón deja de perderse en la radio, que es exactamente el dato que hoy no existe.
6. El viaje se asigna **íntegramente** al polígono en que estaba posicionada la pala al momento del carguío, sin prorrateo automático en Etapa 1, y se marca como viaje de borde cuando corresponde (RF-CAR-005). El prorrateo automático queda como punto de extensión previsto, a habilitar sólo si la marcha blanca demuestra que la proporción de viajes de borde lo justifica (RF-CAR-008, fuera del Contrato por el Artículo 72.º).
7. El camión descarga en chancado, en stock, en stock de baja ley o en botadero. El destino efectivo, no el planificado, es el que queda registrado.
8. Si la persona operadora detecta una falla, levanta el aviso desde la cabina con **una sola acción**, y la orden de trabajo se genera sin que un planificador la transcriba (RF-CAR-015).

**Lo que este flujo no le pide a nadie:** ninguna clasificación de detención, ningún texto libre, ninguna planilla, ninguna contraseña escrita. Dos gestos por turno.

### 4.5 Flujo 4 — Stock intermedio, donde hoy se rompe la trazabilidad

El caso declara este punto como decisión de diseño del PROPONENTE, no como requisito resuelto.

1. Cada aporte a un stock se registra con polígono de origen, tonelaje, ley por elemento, fecha y hora, equipo y viaje asociado (RF-STK-001).
2. El sistema mantiene **en línea** el saldo, la composición por polígono y la ley ponderada vigente, recalculándola ante cada aporte y cada retoma (RF-STK-002). Desaparece la planilla del supervisor de turno.
3. Cada retoma se registra con el cargador frontal que la ejecutó, el tonelaje y el destino.
4. **El supuesto se declara y se mide, no se asume.** Durante la marcha blanca se ejecuta una campaña puntual de muestreo de verificación de la ley en retoma, comparando la ley medida contra la que predecía el promedio ponderado (RF-STK-006).
5. Si la campaña demuestra que la desviación es materialmente alta, se habilita el modelado por sublotes con regla de consumo declarada (RF-STK-007). La decisión se toma con dato medido, no de antemano.
6. Los traspasos entre stocks y el reproceso del stock de baja ley conservan la cadena de origen por polígono (RF-STK-008).
7. Cuando el saldo calculado diverge del levantamiento topográfico o del control de inventario por encima de un umbral parametrizable, el sistema alerta (RF-STK-009).

### 4.6 Flujo 5 — Pesaje, tonelaje y conciliación metalúrgica

**Hoy:** tres números que no coinciden —pesaje de a bordo, báscula de camiones y balanza de planta— y cada área usa el que le conviene para su informe. La diferencia de conciliación está entre 3,1 % y 6,8 %, sin explicación, y el cierre toma 18 días hábiles.

1. Las tres mediciones se registran y conservan **tal como se generan**, sin transformarlas y sin que ninguna sobrescriba a otra (RF-PES-001, RN-04).
2. La **balanza de alimentación de planta es la fuente de verdad** del tonelaje total procesado; la atribución espacial por polígono se calcula a partir del pesaje de a bordo (RF-PES-002).
3. La báscula de camiones queda como **punto de calibración** periódica del pesaje de a bordo, conservando la muestra de viajes pesados y la desviación medida por equipo (RF-PES-004).
4. El balance metalúrgico del turno se calcula desde los datos registrados, reemplazando la planilla del metalurgista de turno (RF-PES-009).
5. Los factores de ajuste del balance del mes se registran como **parámetros versionados** con vigencia, responsable y fundamento documentado (RF-PES-010). Dejan de ser conocimiento de dos personas.
6. La conciliación del período cuadra el cobre estimado en el modelo, el extraído, el alimentado a planta, el recuperado y el embarcado, y **atribuye la diferencia residual a un punto de medición de la cadena**. Ninguna diferencia se cierra como no explicada (RF-PES-007, RN-06).
7. La conciliación está disponible para su cierre en **3 días hábiles** desde el cierre del período, con acceso al dato de origen (RNF-DES-008, criterio n.º 2).
8. Los datos de tonelaje, ley, metal contenido, consumos y movimientos se entregan al ERP para el registro contable; el ERP emite. **No hay dos verdades contables** (RF-PES-012, restricción n.º 4, RN-05).
9. Si el recálculo retroactivo demuestra que la diferencia por equipo es sistemática, se aplica un factor de corrección del pesaje de a bordo por equipo y período, versionado y con vigencia declarada (RF-PES-006, punto de extensión fuera del Contrato).

### 4.7 Flujo 6 — Muestreo, análisis y ley

Del orden de **71.000 muestras al año**, con espera de resultado de 4 a 36 horas según el tipo de análisis.

1. El catálogo de puntos de muestreo obligatorios se mantiene ubicado donde el material ya se detiene o donde lo opera un rol distinto del ciclo de mina, **nunca en el ciclo de pala o de camión** (RF-LAB-010, RN-12). Cada punto lleva declarado su costo en segundos de ciclo y su efecto estimado en toneladas por año, para que el CLIENTE decida con la cifra a la vista (RF-LAB-011).
2. La persona que toma la muestra registra la cadena de custodia: quién, cuándo, en qué punto, bajo qué protocolo, con qué equipo y quién la recibió en laboratorio (RF-LAB-004).
3. La solución envía la solicitud de análisis al sistema de gestión de laboratorio y recibe el resultado, en integración bidireccional (RF-LAB-001, C-09, restricción n.º 6).
4. El resultado del laboratorio es la **fuente de verdad de la ley medida**; la ley del modelo de bloques y la del plan son estimaciones que se reconcilian contra ella, y la diferencia se publica por polígono como indicador de calidad del modelo (RF-LAB-002, RN-10).
5. Cada lote conserva un **vector de elementos** —cobre, oro, plata y arsénico como elemento deletéreo— más la humedad, no un único valor de ley (RF-LAB-009, RN-14).
6. El ciclo de vida del lote avanza por una máquina de estados con transiciones auditadas: pendiente, analizado, validado, liberado, con las ramas rechazado y remuestreado (RF-LAB-005).
7. Ante muestra rechazada o invalidada se dispara el remuestreo con plazo definido, se bloquea comercialmente el lote con motivo desde lista cerrada y se escala si el plazo vence (RF-LAB-007).
8. **Ningún lote se libera a embarque sin análisis válido y vigente, y la liberación no admite excepción manual** (RN-11).
9. En los muestreadores de proceso automáticos, el dato se captura por instrumentación: la acción física de muestreo se separa del registro del dato, que no nace de una digitación (RF-LAB-012).
10. El sistema alerta cuando un punto de transferencia obligatorio no registra muestra dentro de su frecuencia (RF-LAB-013) y cuando el tiempo entre toma y resultado excede el plazo definido por tipo de análisis (RF-LAB-015).

### 4.8 Flujo 7 — Lote comercial, embarque, liquidación y devolución

1. Cada camión de concentrado que sale a puerto lleva **guía de despacho y planilla de pesaje emitidas y registradas digitalmente**, reemplazando el documento en papel (RF-TRZ-012).
2. En el recinto portuario se registra la recepción y el acopio por lotes, con tonelaje, humedad y fecha, aun siendo el recinto administrado por un tercero (RF-LOT-002).
3. El área comercial conforma el lote comercial indicando qué lotes del acopio lo componen y en qué proporción, con **relación padre-hijo explícita** (RF-LOT-003).
4. **Antes de la conformación definitiva**, el sistema calcula y alerta el contenido de arsénico proyectado contra el límite del contrato del comprador (RF-LOT-005). Es el control que ataca directamente las penalizaciones por elementos deletéreos.
5. Como apoyo a la decisión, el sistema propone combinaciones de material del acopio que mantienen los deletéreos bajo el límite contractual (RF-LOT-006). Propone; no decide.
6. Se registra el muestreo del lote conforme al protocolo acordado con el comprador, con cadena de custodia, humedad determinada y ley por elemento (RF-LOT-004).
7. Se aplica **firma electrónica avanzada** a los registros de seguridad exigidos por la autoridad, a las actas de aceptación de embarque y a los informes de monitoreo ambiental que la requieran (RF-LOT-007).
8. Se registra la interacción con la nave, el agente de aduana y el organismo fiscalizador, conservando los documentos intercambiados (RF-LOT-013).
9. Meses después llega la liquidación: ley pagable, deducciones por humedad, cargos de tratamiento y refinación y penalidades, calculada sobre el vector de elementos (RF-LOT-008). Se registran el resultado del laboratorio del comprador y, ante discrepancia, el del laboratorio árbitro (RF-LAB-014).
10. Si el comprador devuelve o rechaza concentrado, se modela como **evento de reversa** sobre el lote original: reabre su trazabilidad y genera un lote de reingreso con su propia trazabilidad (RF-LOT-009).
11. En cualquier momento, el origen del lote se reconstruye recorriendo el árbol de eventos: **menos de 2 horas para la reconstrucción completa y 30 segundos en el percentil 95 para la consulta en línea** (RF-TRZ-003, RNF-DES-009), contra las 4 a 6 semanas actuales con resultado estimado. Es el criterio de aceptación n.º 1.
12. El expediente de trazabilidad exportable lleva identificada la versión de las reglas y de los supuestos aplicados, y firma electrónica (RF-TRZ-010).

### 4.9 Flujo 8 — Detención operacional

**Hoy:** 23 % del tiempo sin clasificar y 61 % de las detenciones sin causa raíz.

1. La taxonomía es **cerrada y jerárquica de tres niveles**, construida sobre el vocabulario estándar de la industria y parametrizable por el CLIENTE (RF-DET-001).
2. La clasificación es **automática** a partir de señales que ya existen: posición y estado del despacho, ventana de tronadura del plan, calendario de turno y colación, alarmas de telemetría (RF-DET-002).
3. **La persona operadora no clasifica nunca**, ni en cabina ni en ningún otro dispositivo (RF-DET-004, RN-17).
4. Al supervisor se le presenta **únicamente el residuo** que la clasificación automática no resolvió, para clasificarlo desde lista cerrada y sin texto libre. El residuo está disponible antes del cierre del turno en que ocurrió (RF-DET-003, RNF-DES-011).
5. El supervisor puede clasificar **desde la propia notificación**, escogiendo de la misma taxonomía de tres niveles. No es una excepción a la regla: es la regla ejecutada por otro canal (RT-16.26).
6. El turno **no cierra** con detenciones sin clasificar: las pendientes escalan al supervisor de la guardia siguiente con constancia del escalamiento (RF-DET-005, RN-18).
7. Mantenimiento y la Superintendencia de Planta registran la causa raíz de las no programadas, con responsable, acción tomada y estado de cierre (RF-DET-006).
8. El sistema calcula y publica por equipo, área y turno la disponibilidad física, la utilización efectiva, las horas de detención no programada y el tiempo medio entre falla y diagnóstico (RF-DET-007), exhibidos junto a la **línea base 2025** y a la meta comprometida, de modo que la mejora sea verificable y no declarativa (RF-DET-011).

### 4.10 Flujo 9 — Acreditación, habilitación e ingreso a faena

**Hoy:** nueve días para acreditar a un contratista nuevo, y el 14 % de los rechazos en portería corresponde a documentos vencidos que nadie advirtió; el trabajador viaja 600 km para que le nieguen el ingreso.

1. La empresa contratista administra desde su portal las identidades de sus trabajadores y carga la documentación de habilitación (RF-IDE-005).
2. El flujo de acreditación avanza con estados, responsables, plazos y escalamiento, y se completa en **menos de 48 horas** (RF-IDE-009, criterio n.º 8).
3. La credencial física de proximidad emitida en la acreditación **es** la identidad digital de la persona contratista: no se crea un segundo sistema de identidad (RF-IDE-002). El personal propio se federa con el directorio corporativo del grupo, con doble factor y sin credenciales compartidas (RF-IDE-001, restricción n.º 9).
4. El sistema valida automáticamente y en continuo el estado de habilitación: contrato vigente, exámenes preocupacionales y de altura geográfica al día, vigilancia por exposición, cursos e inducciones (RF-IDE-007).
5. **Con anticipación configurable**, alerta el vencimiento de cualquier documento a la propia persona, a su empleador y al área de seguridad (RF-IDE-008). La empresa puede adjuntar la renovación desde la propia notificación (RT-16.26). Éste es el mecanismo completo del criterio n.º 7.
6. En portería, la validación ocurre en el torniquete en **1,5 segundos**, incluido el modo degradado contra caché local (RF-DES-006, RNF-DES-003, RN-20).
7. Todo ingreso y toda salida quedan registrados con persona, empresa, marca de tiempo y resultado, y el inventario de personas presentes en faena se mantiene en línea (RF-IDE-011).
8. El acceso se revoca en **no más de quince minutos** desde el término de la relación con la faena, e impide el ingreso desde ese instante (RF-IDE-019, SUP-28).
9. Sobre los eventos de acceso ya registrados, y sin captura adicional, se detectan indicios de uso indebido o préstamo de credenciales y se reportan al área de seguridad (RF-IDE-012).
10. El acceso al recinto técnico de servidores de faena se controla como un control distinto e independiente del de la portería general (RF-IDE-014).

### 4.11 Flujo 10 — Incidente, inspección y alerta de fatiga

**Hoy:** formularios de papel digitados días después; en la última auditoría de la autoridad, seis de once observaciones tuvieron que ver con registros que no se pudieron exhibir. Las alertas de fatiga llegan por correo y teléfono de turno, y la acción no se registra.

1. El supervisor registra incidentes, cuasi accidentes, observaciones de conducta e inspecciones planificadas **en el lugar del hecho**, desde la aplicación de terreno, con operación desconectada (RF-SSO-001). Nacen digitales: es el criterio n.º 9.
2. Cada alerta de la plataforma de control de fatiga se ingiere como **evento propio del sistema**, con sello de tiempo, persona, equipo, turno y tipo de alerta (RF-SSO-008).
3. Se abre un flujo con responsable, plazo, estado y escalamiento automático para la acción tomada ante cada alerta y cada alarma crítica (RF-SSO-004).
4. El área de seguridad explota el **reporte de conciliación** entre alertas emitidas y acciones registradas, con el residuo sin acción documentada como indicador visible (RF-SSO-009). Es el criterio n.º 10, y está construido de modo que su incumplimiento sea imposible de ocultar.
5. Todo este flujo opera bajo cifrado a nivel de campo y con registro de toda consulta y toda modificación (RF-SSO-007, RF-IDE-020), y con el tratamiento de datos personales que exige la Ley N.º 21.719 (RF-SSO-010).

### 4.12 Flujo 11 — Compromiso ambiental, huella de carbono y comunidad

1. La instrumentación ambiental y geotécnica propia se captura **automáticamente**: consumo de agua, calidad del aire, ruido, monitoreo geotécnico y de aguas subterráneas (RF-AMB-001).
2. Los informes de laboratorios externos y de la empresa geotécnica se asocian al compromiso correspondiente, conservando el documento original y extrayendo el valor (RF-AMB-004).
3. Cada compromiso de la Resolución de Calificación Ambiental queda registrado con frecuencia, punto de medición y destinatario, y alerta la proximidad de su vencimiento (RF-AMB-002).
4. Los consumos por área —planta, mina, campamento y supresión de polvo en caminos— se calculan con regla de reparto parametrizable (RF-AMB-005), y los indicadores del convenio con la comunidad se generan desde el ERP y recursos humanos **sin solicitar planillas semestrales** (RF-AMB-008).
5. La huella de carbono por tonelada de cobre fino se calcula **mensualmente** con alcances 1, 2 y 3, conservando versionados los factores de emisión y la evidencia de cada dato de actividad (RF-AMB-006, RN-26). Criterio n.º 12.
6. La serie mensual se reconstruye hacia atrás sobre los datos ya registrados, aplicando a cada período los factores que regían entonces (RF-AMB-014).
7. **Todo reporte a la autoridad se genera desde el sistema**, con trazabilidad desde el valor reportado hasta la medición de origen, con fecha y fuente. Ningún reporte se construye en planilla al margen (RF-AMB-003, RN-25). Criterio n.º 11.
8. Se produce la información trazable de avance de obras que exige la normativa de cierre de faenas (RF-AMB-013) y el expediente de evidencia auditable para el verificador tercero de la huella (RF-AMB-015).
9. El monitoreo del depósito de relaves se almacena localmente y se sincroniza después cuando el radioenlace se interrumpe, sin pérdida de mediciones ni de reportabilidad (RF-AMB-009), y la superación de umbrales geotécnicos y ambientales se alerta a los destinatarios definidos (RF-AMB-010).

### 4.13 Flujo 12 — Corte de enlace de 48 horas y reconciliación

El enlace de fibra tiene un único proveedor, con tres a seis cortes al año de cuatro a veinte horas. Es el flujo que decide el criterio n.º 13 y, con él, buena parte de la evaluación.

1. **Durante el corte sigue operando íntegro:** registro del ciclo mina —carguío, transporte, descarga y pesaje—; identificación del operador por proximidad y apertura y cierre de sesión en el relevo; validación de acceso en portería contra caché; registro de inspecciones, incidentes y detenciones; toma de muestra y su registro; movimiento de stock (RF-DES-001, RNF-DIS-005).
2. Cada dispositivo y cada nodo de borde mantiene registro local firmado y cifrado, reloj monótono con sello de tiempo propio y registro de deriva, identificador de evento generado localmente, cola de sincronización idempotente y caché de credenciales vigentes (RF-DES-002, RF-DES-003, RF-DES-006).
3. Operan cuatro gabinetes de borde: **portería, planta, depósito de relaves y acopio de puerto**, con energía respaldada y capacidad de registrar y reconciliar de forma autónoma (RF-DES-009).
4. Si el relevo ocurre sin señal, la sesión saliente se cierra localmente y la entrante se abre, **sin fusionar sus eventos** (RF-DES-004).
5. **Lo que no opera se declara y tiene procedimiento manual:** alta de una persona contratista nueva; alta o modificación de datos maestros; conciliación metalúrgica y cierre del mes; **liberación de un lote a embarque, que no se libera en ningún caso**; publicación al portal del comprador y a la autoridad; tableros analíticos y recálculo de indicadores; alertas anticipadas de vencimiento; ingesta de telemetría de fabricante. La ausencia de esta declaración es observación grave según el RT-03.13.
6. La aplicación muestra de manera permanente su estado de enlace, las horas acumuladas sin sincronizar y la cantidad de eventos pendientes (RT-02.09). Un operador que no sabe si su registro se guardó deja de registrar, y con eso se pierde el criterio n.º 14.
7. El Centro Integrado de Operaciones sigue el panel de estado de sincronización por dispositivo y por sitio (RF-DES-011).
8. Al restablecerse el enlace, la reconciliación es **idempotente, reintentable y automática, sin duplicados y sin intervención manual**, y concluye en **no más de 4 horas** (RF-DES-002, RNF-DES-012).
9. Los conflictos se resuelven con política declarada por tipo de dato: **el evento de terreno prevalece para los hechos observados en terreno; el registro central prevalece para los datos maestros** (RF-DES-005, RN-30). Los eventos que referencian un maestro inexistente quedan en cuarentena y se resuelven al reconciliar.
10. La verificación comprometida es un **corte controlado de 48 horas con un cambio de turno dentro de la ventana**, en que los eventos reconciliados deben coincidir exactamente con los generados.

### 4.14 Flujo 13 — Cambio de una regla o de un parámetro

Es el flujo que hace que la solución siga siendo del CLIENTE después del Contrato.

1. El administrador propone el cambio desde la consola: catálogos, umbrales, taxonomías, reglas, plazos o destinatarios de alerta, **sin requerir desarrollo** (RF-ADM-002).
2. El efecto se prueba en el **ambiente de simulación sobre una copia de historia ya cerrada**, no sobre datos sintéticos: lo que se necesita saber antes de cambiar un factor de conciliación o una tolerancia es qué habría pasado con los meses que ya se cerraron (RT-16.05).
3. Todo cambio con impacto operacional exige **aprobación de un segundo perfil**, con justificación registrada (RF-ADM-013, RT-16.03).
4. El cambio se versiona con vigencia temporal. Un período anterior se recalcula con la versión que regía en ese momento (RF-ADM-004, RN-27).
5. Queda asiento en la pista de auditoría inalterable, con valor anterior y posterior, persona autora, instante y origen, verificable mediante mecanismo de integridad (RF-ADM-001).
6. El registro de supuestos declarados se exhibe **junto al resultado** en toda consulta que dependa de ellos (RF-ADM-005), y el registro de reglas de negocio de la industria se mantiene documentado y versionado (RF-ADM-003).

### 4.15 Cómo se comporta el flujo a lo largo del año

| Momento | Qué ocurre en el sistema | Requisito |
|---|---|---|
| Cada relevo, 08:00 y 20:00 | Cierre y apertura de una sesión de operador por cada equipo de la flota, en la misma franja. Es el peak predecible que gobierna el dimensionamiento | RNF-CAP-006: 40 transacciones por segundo en régimen, 200 en peak |
| Cierre de cada turno | Indicadores operacionales publicados; residuo de detenciones cerrado; cumplimiento del plan disponible | RNF-DES-006, RNF-DES-010, RN-23 |
| Cierre de mes | Conciliación metalúrgica en 3 días hábiles; huella de carbono mensual | Criterios n.º 2 y n.º 12 |
| 48 horas previas a un embarque y durante el cierre contable | **Ventana operacional protegida:** intervenciones prohibidas | RF-ADM-012, RN-24 |
| Marzo y septiembre, seis días | Detenciones programadas de planta: únicas ventanas de intervención mayor. Hasta 4.300 contratistas adicionales, que multiplican la carga de acreditación | Restricciones n.º 1 y n.º 11 |
| Tres a seis veces al año | Corte de fibra de 4 a 20 horas: se ejercita el flujo 12 | Criterio n.º 13 |
| Una vez al año | Prueba de intrusión por tercero independiente del ADJUDICATARIO | RNF-SEG-009 |

---

## 5. Por qué esta solución cumple

Este apartado contrasta lo especificado contra los tres documentos que gobiernan la licitación, en el orden en que ellos mismos se imponen: primero las reglas del proceso, después el piso técnico obligatorio, y por último el caso, que es el que decide la adjudicación.

### 5.1 Bases Administrativas TFEP-01/2026

| Disposición | Qué exige | Cómo se cumple |
|---|---|---|
| **Artículo 16.º** — Modelo de despliegue híbrido obligatorio | La solución debe ser obligatoriamente híbrida, y el emplazamiento de **cada componente** debe justificarse por latencia, criticidad operacional, volumen, restricciones regulatorias, conectividad y costo total de propiedad. Una asignación no justificada es **observación grave** | Este documento entrega el insumo que hace posible esa justificación: los flujos 3, 9 y 12 demuestran qué funciones no pueden depender del enlace con Antofagasta —registro del ciclo mina, identificación por proximidad, validación en portería, incidentes, muestreo y stock— y por tanto qué componentes deben residir en faena. La justificación componente por componente se emite en el Subdocumento 4.2 |
| **Artículo 57.2** — Coherencia transversal, con descuento de puntaje en cualquier ítem | Que la arquitectura sostenga el alcance y esté reflejada en los costos; que la EDT contenga la totalidad del alcance; que el cronograma sea consistente con la EDT; que los riesgos correspondan a la solución propuesta y no a un catálogo genérico | Todo lo afirmado aquí usa los códigos del catálogo v2.4, los componentes C-01 a C-22 del Subdocumento 4.1 v3.1 y las etapas de la matriz v1.1, sin excepción. Los 167 requerimientos funcionales de este documento son los mismos 167 que la matriz traza hasta la EDT y hasta la prueba de verificación: la EDT no puede quedar incompleta respecto de un alcance que está enumerado |
| **Artículo 58.º** — Condiciones de exclusión | Se excluye a quien obtenga menos de 30 puntos en cualquier ítem, menos de 60 ponderados, omita un subdocumento obligatorio del Formulario T-7 o **no acredite los requisitos transversales obligatorios del Capítulo 4** | La acreditación de los requisitos transversales se lleva en el Formulario T-12, que hoy contiene **374 filas**: 132 declaradas «Cumple», 10 «Cumple parcialmente», 1 «No aplica» y 231 asignadas a subdocumentos aún no emitidos. Ninguna fila queda sin dueño declarado |
| **Artículo 72.º** — Modificaciones contractuales y control de cambios | Toda modificación requiere acuerdo escrito previa evaluación de impacto, con límite acumulado del 20 % del valor original | Las tres capacidades excluidas del Contrato —prorrateo automático del viaje de borde (RF-CAR-008), biometría en portería (RF-IDE-013) y factor de corrección de pesaje por equipo y período (RF-PES-006)— **no se ofrecen ni se costean**, pero la arquitectura deja el punto de extensión previsto en C-01, C-05 y C-08. Si el CLIENTE decide incorporarlas, entran por solicitud formal de cambio y no por rediseño |
| **Formulario T-7** — Catorce subdocumentos, cada uno un ítem de evaluación | La Propuesta Técnica se estructura en catorce subdocumentos en el orden indicado | Este documento **no es** uno de ellos: es insumo del Subdocumento 3 —descripción de la solución, alcance por etapas y catálogo priorizado y trazable—, del Subdocumento 4 —arquitectura lógica— y del Subdocumento 7 —implantación y EDT—. Se declara así para que su existencia no se confunda con un subdocumento omitido |

**Nota sobre las consultas.** Toda ambigüedad detectada al construir esta especificación que no pueda resolverse con el texto de las Bases se canaliza por el procedimiento de consultas del Artículo 43.º, en el formato y el plazo que ese artículo fija, y no se resuelve unilateralmente como supuesto cuando la respuesta cambia el alcance.

### 5.2 Bases Técnicas Transversales TFEP-01/2026

El Capítulo A de los anexos transversales codifica los requisitos por grupo. La tabla recorre los grupos que esta especificación toca, con el mecanismo concreto que los satisface. La acreditación formal, requisito por requisito, es la del Formulario T-12.

| Grupo | Qué exige | Mecanismo de la solución |
|---|---|---|
| **02** · Modelo de arquitectura de referencia (RT-02.01 – RT-02.14) | Ocho capas obligatorias, límites de contexto explícitos, acoplamiento débil, despliegue independiente, degradación elegante, idempotencia, capa anticorrupción hacia lo externo | Las ocho capas y los nueve límites de contexto están declarados en el Subdocumento 4.1 v3.1. Siete integraciones externas, **una capa anticorrupción por cada una** y adaptador reemplazable (RNF-INT-003). Toda escritura expuesta a reintentos es idempotente. La degradación se declara función por función en el flujo 12 |
| **03** · Modelo híbrido (RT-03.01 – RT-03.24) | Nube y on-premise con justificación por componente; declaración de funciones no disponibles en modo desconectado, cuya ausencia es **observación grave** (RT-03.13); procesamiento en el borde; administración remota de dispositivos | El flujo 12 declara las ocho funciones que no operan sin enlace y el procedimiento manual que suple a cada una. La administración remota del parque de terreno está en RF-ADM-011 |
| **04** · Ambientes, entrega continua y configuración (RT-04.01 – RT-04.14) | Cinco ambientes habilitados; separación entre configuración técnica y parámetros de negocio; despliegue sin interrupción; migraciones de esquema | La separación está en el flujo 13: la configuración técnica es del ADJUDICATARIO, los parámetros de negocio son del CLIENTE y se cambian sin desarrollo (RF-ADM-002). El despliegue admite reversión **por área y por sitio**, sin que un paso a producción afecte simultáneamente a mina, planta y puerto (RF-MIG-006) |
| **05** · Datos, integración e interoperabilidad (RT-05.01 – RT-05.30) | Contratos de interfaz documentados y versionados; autenticación entre sistemas; identificador de correlación; estándares sectoriales del caso | Catálogo de integraciones documentado y versionado, con contraparte, sentido, protocolo, frecuencia y responsable, **mantenido durante todo el Contrato** (RF-INT-011). Cada intercambio registra su resultado, con reintento automático y alerta ante falla persistente (RF-INT-014). Los estándares del caso son OPC UA, ISA-95 y el estándar abierto de intercambio de datos de equipos mineros (RF-INT-013) |
| **08** · Hardware, puestos de trabajo y terreno (RT-08.01 – RT-08.19) | El PROPONENTE especifica el hardware de terreno; el CLIENTE lo adquiere | Especificación completa con referencia, cantidad por sitio y características (RNF-INF-003). Dimensionada sobre 450 dispositivos a tres años (RNF-CAP-007) |
| **09** · Desempeño, capacidad y escalabilidad (RT-09.01 – RT-09.10) | Umbrales declarados y medibles; degradación controlada al superarse la capacidad; declaración del componente que satura primero | Catorce umbrales de desempeño y ocho de capacidad, todos numéricos y con método de verificación (categorías 20 y 21 del catálogo). 40 transacciones por segundo en régimen y 200 en peak, con el peak **anticipado por calendario** en los relevos de 08:00 y 20:00. El componente que satura primero está declarado con su volumen y su señal (RNF-ESC-002) |
| **10** · Disponibilidad, continuidad y resiliencia (RT-10.01 – RT-10.09) | Objetivos de disponibilidad, recuperación y punto de recuperación | 99,9 % mensual para operación en terreno y 99,5 % para gestión; RTO no superior a 1 hora y **RPO de cero eventos perdidos** para el registro de terreno (RNF-DIS-002, RNF-DIS-004) |
| **11** · Seguridad de la información (RT-11.01 – RT-11.28) | Zero Trust conforme a NIST SP 800-207; modelado de amenazas; clasificación de la información; publicación exclusiva por la capa de borde; cifrado; registro y correlación | La superficie expuesta son exactamente dos portales autenticados y una puerta de enlace, y **nada más**. Cifrado a nivel de campo para salud ocupacional, fatiga y control de alcohol y drogas (RF-SSO-007, RT-11.10). Prueba de intrusión anual por tercero independiente sin hallazgos críticos ni altos abiertos (RNF-SEG-009) |
| **12** · Identidad, acceso y sesiones (RT-12.01 – RT-12.13) | Federación; doble factor; factores resistentes a suplantación para administradores; **prohibición de contraseña alfanumérica escrita en terreno**; mecanismo autoservido para personas usuarias externas | Tres mecanismos sobre un maestro único: federación corporativa con doble factor para el personal propio; credencial de proximidad sobre dispositivo inventariado en terreno, sin contraseña escrita; y esquema autoservido con administración delegada para las cuatro clases de personas usuarias externas del RT-12.12, todas cubiertas en el apartado 3.4 |
| **13** · Usabilidad, accesibilidad y experiencia (RT-13.01 – RT-13.12) | Accesibilidad verificada; diseño responsivo; profundidad de navegación acotada; interfaces de terreno operables en condiciones adversas | WCAG 2.2 nivel AA (RNF-USA-007); tres interacciones de profundidad máxima (RNF-USA-009); navegación íntegra por teclado (RNF-USA-013); guante grueso con más del 95 % de éxito al primer intento y 1.000 cd/m² de luminancia (RNF-USA-001, RNF-USA-002); interfaz íntegramente en español con el vocabulario de la industria del cobre a rajo abierto (RNF-USA-006) |
| **14** · Observabilidad y gestión del servicio (RT-14.01 – RT-14.09) | Instrumentación; acceso del CLIENTE a los tableros; retención de métricas, registros y trazas; detección proactiva | Acceso propio y permanente del CLIENTE a los tableros, con datos en tiempo real y exportación, **sin intervención del ADJUDICATARIO** (RT-14.02). Diagnóstico de incidentes sin acceder a la base de datos productiva (RNF-MAN-005) |
| **15** · Sostenibilidad, eficiencia y certificaciones (RT-15.01 – RT-15.09) | Certificaciones sectoriales del adjudicatario | ISO 45001 e ISO 14001 vigentes, conocimiento acreditado del Reglamento de Seguridad Minera y experiencia comprobable en faena minera en operación (RNF-CUM-005) |
| **16** · Módulos transversales obligatorios (RT-16.01 – RT-16.34) | Parametrización sin desarrollo; aprobación por segundo perfil; ambiente de simulación; gestión documental; notificaciones; firma electrónica; búsqueda; bandeja de tareas | Quince requerimientos de la familia 18 del catálogo más la gestión documental con versionado, metadatos, control de acceso, búsqueda por contenido y previsualización sin descarga (RF-ADM-017), la generación documental desde plantillas administrables por el CLIENTE (RF-ADM-016) y la firma electrónica avanzada (RF-LOT-007) |
| **17** · Canales digitales y movilidad (RT-17.01 – RT-17.08) | Decisión declarada de canal; soporte de versiones de sistema operativo móvil; cobertura en dispositivos de bajo costo | Decisión de canal declarada en el Subdocumento 4.1 v3.1, apartado 16. Versión vigente del sistema operativo móvil y las dos anteriores (RNF-POR-004), con el umbral autoimpuesto de SUP-39 sobre el parque real de dispositivos, no sobre el especificado |
| **18** · Inteligencia artificial y automatización (RT-18.01 – RT-18.10) | Declaración de datos de entrenamiento, volumen de alertas, tasa de falsos positivos y comportamiento ante datos insuficientes | La analítica predictiva de mantenimiento se ofrece **declarando** con qué datos se entrena, cuántas alertas por semana se espera emitir, cuál es la tasa esperada de falsos positivos y qué hace el modelo cuando el dato no alcanza (RF-MTO-009). La detección de préstamo de credenciales opera sobre eventos ya registrados, sin captura adicional (RF-IDE-012) |

### 5.3 Bases Técnicas del Caso 01 · Minería

#### 5.3.1 Las doce restricciones no negociables

El Capítulo 10 advierte que una propuesta que no las respete será evaluada como falta de comprensión del caso.

| N.º | Restricción | Cómo la respeta la solución |
|---|---|---|
| 1 | La mina y la planta no se detienen; las únicas ventanas de intervención mayor son las detenciones de marzo y septiembre | Ninguna función bloquea la operación. Si nadie se identifica en el relevo, el viaje se marca y se alerta, pero **el equipo no se detiene** (RF-CAR-011, RN-22). La ventana operacional protegida admite intervenciones mayores sólo en esas dos detenciones (RF-ADM-012) |
| 2 | La operación no puede depender del enlace con Antofagasta | Flujo 12 completo: 48 horas continuas de operación y registro sin enlace, con cuatro gabinetes de borde y sincronización en no más de 4 horas sin intervención manual. **Cero funciones críticas de terreno dependientes de conectividad permanente** (RNF-DIS-005) |
| 3 | La red de control de proceso es de sólo lectura | Lectura del historiador por OPC UA, **en modo exclusivamente de lectura**, sin escribir en ningún caso en la red de control y sin instalar componentes en ella sin la segregación aprobada (RF-INT-005, C-14) |
| 4 | El ERP no se reemplaza ni se modifica | La solución le entrega tonelaje, ley, metal contenido, consumos y movimientos, y consume de él los maestros. **El ERP emite; no habrá dos verdades contables** (RF-PES-012, RN-05) |
| 5 | El sistema de despacho de flota se mantiene | Se integra y no se sustituye: lectura de asignaciones, ciclos, tonelaje de a bordo y posición, y escritura sólo de la marca de estado que el propio despacho admita (RF-CAR-001) |
| 6 | El sistema de gestión de laboratorio se mantiene y se integra en ambos sentidos | Integración bidireccional: envío de la solicitud de análisis y recepción del resultado validado, con el resultado como fuente de verdad de la ley medida (RF-LAB-001, RF-LAB-002) |
| 7 | El software de planificación minera se mantiene | Se consumen sus salidas automáticamente, sin carpeta compartida ni carga manual, y no se interviene el modelo de bloques (RF-PLA-001) |
| 8 | **No se agregan pasos al ciclo productivo**; todo registro adicional debe justificarse en segundos y demostrarse en la marcha blanca | Se pide **un único elemento nuevo en cabina**: el botón de cambio de material, con una acción no superior a 2 segundos y confirmación visible en 1 segundo (RF-CAR-003, RNF-DES-004). El sistema declara expresamente que la persona operadora no clasifica detenciones (RF-DET-004). La adopción se mide por área y turno durante la marcha blanca (RF-MIG-005) |
| 9 | Identidad federada con el directorio corporativo, doble factor, sin credenciales compartidas, con registro completo de accesos | RF-IDE-001 en su enunciado completo. En terreno el segundo factor es la posesión física de la credencial sobre un dispositivo inventariado, porque RT-12.11 prohíbe la contraseña alfanumérica escrita |
| 10 | El equipo de tecnologías de información del CLIENTE son **once personas** | La solución es operable en régimen por ellas; toda función que requiera un especialista dedicado que la compañía no tiene se ofrece como servicio costeado (RNF-MAN-001). Es también el criterio que descartó los microservicios de grano fino en favor de servicios modulares |
| 11 | El personal clave de operaciones tiene disponibilidad limitada en marzo y septiembre | Las validaciones que lo requieran se planifican fuera de esas ventanas, y la capacitación considera el régimen 7x7 con al menos dos ciclos completos (RNF-MAN-007) |
| 12 | Toda intervención en faena está sujeta al Reglamento de Seguridad Minera; el personal del ADJUDICATARIO se acredita como cualquier contratista | La presencia en faena considera el traslado desde Antofagasta, los 145 km y los 3.180 metros de altura (RNF-MAN-003), y el conocimiento acreditado del Reglamento es exigencia de certificación (RNF-CUM-005) |

#### 5.3.2 Los catorce criterios de aceptación del Capítulo 18

| N.º | Resultado esperado | Situación actual | Cómo se alcanza | Requerimientos que lo sostienen |
|---|---|---|---|---|
| 1 | Origen de un lote embarcado reconstruido con evidencia en menos de **2 horas** | 4 a 6 semanas, con resultado estimado | El viaje es la unidad atómica; la trazabilidad es un árbol de eventos inmutables que se recorre, no una estimación que se recalcula (flujos 3 y 7) | 36 |
| 2 | Conciliación metalúrgica del mes cierra en **3 días hábiles** | 18 días hábiles | Las tres mediciones se conservan tal como se generan y el balance del turno se calcula desde el dato registrado, no desde planillas (flujo 5) | 21 |
| 3 | La diferencia de conciliación converge y **toda diferencia residual queda explicada por punto de la cadena** | Entre 3,1 % y 6,8 %, sin explicación | RN-06: ninguna diferencia se cierra como no explicada. Los factores de ajuste dejan de ser conocimiento tácito y pasan a ser parámetros versionados | 19 |
| 4 | Cumplimiento del plan conocido al inicio del turno siguiente, contra la versión **efectivamente vigente** | 10 a 15 días de desfase, contra un plan desactualizado | Cada movimiento queda asociado automáticamente a la versión del plan vigente en ese instante (flujo 1) | 13 |
| 5 | La totalidad de las detenciones clasificada dentro del turno siguiente | 23 % del tiempo sin clasificar; 61 % sin causa raíz | Clasificación automática desde señales existentes; al supervisor sólo el residuo; el turno no cierra con pendientes (flujo 8) | 15 |
| 6 | El supervisor dispone de estado, historial, alarmas y pautas **en una sola vista** | Cuatro sistemas y una radio | C-16 compone la vista desde el despacho, el ERP y los tres adaptadores de telemetría, con degradación explícita si una fuente falta | 12 |
| 7 | Ninguna persona rechazada en portería por documentación vencida que el sistema pudo advertir | 14 % de los rechazos | Validación continua de habilitación más alerta anticipada a la persona, al empleador y al área de seguridad, accionable desde la propia notificación (flujo 9) | 13 |
| 8 | Una persona contratista nueva se acredita en **menos de 48 horas** | 9 días | Flujo con estados, responsables, plazos y escalamiento sobre el portal de empresas contratistas, con protección anti-bot escalonada para no expulsar a quien debe usarlo | 15 |
| 9 | Registros de seguridad, inspecciones e incidentes **nacen digitales** en el lugar del hecho | Papel digitado con posterioridad | Registro en terreno con operación desconectada, firmado y con sello de tiempo propio (flujo 10) | 21 |
| 10 | Toda alerta de fatiga con acción documentada y trazable | Sin registro consolidado | La alerta se ingiere como evento propio y abre un flujo con responsable y plazo; el residuo sin acción es un indicador visible | 10 |
| 11 | Reportes a la autoridad generados desde el sistema, con trazabilidad hasta la medición de origen | Planillas construidas a mano | RN-25 prohíbe el reporte al margen del sistema; cada cifra se navega hasta su medición (flujo 11) | 12 |
| 12 | Huella de carbono por tonelada de cobre fino **mensual** y auditable por tercero | Anual, con consultora externa | Factores versionados y evidencia por dato de actividad, con expediente auditable y reconstrucción hacia atrás de la serie | 9 |
| 13 | La operación continúa y registra durante **48 horas sin enlace** y sincroniza sin intervención manual | El corte detiene el registro sistémico | Flujo 12 completo, verificado con corte controlado de 48 horas **con un cambio de turno dentro de la ventana** | 21 |
| 14 | El operador de pala usa la solución en su turno, con guantes, **por decisión propia** | Dos iniciativas previas abandonadas | Dos gestos por turno, un botón único, cero texto libre, cero clasificación, confirmación visible del guardado, estado de enlace siempre a la vista, y adopción medida por área y turno durante la marcha blanca | 14 |

La columna final indica cuántos de los 167 requerimientos funcionales sostienen cada criterio según la matriz v1.1. Los 43 requerimientos que no se asocian a ningún criterio del Capítulo 18 corresponden a exigencias transversales obligatorias que el caso no enuncia como resultado de negocio pero las Bases Técnicas Transversales sí imponen.

#### 5.3.3 Las exclusiones explícitas del Capítulo 11 se respetan

No se interviene el control de proceso ni se modifica su lógica. No se automatizan equipos: la operación autónoma de camiones y perforadoras no forma parte del proyecto. No se diseña el plan minero ni se estiman recursos y reservas. No se gestionan remuneraciones ni administración de personal, que residen en el ERP. No se opera el terminal portuario, administrado por un tercero, sin perjuicio de la integración necesaria con él. No se construye la red de comunicaciones del rajo, aunque sí se evalúa si la cobertura actual alcanza y se especifica qué faltaría (RF-DES-013). El hardware de terreno se especifica exactamente y lo adquiere el CLIENTE.

#### 5.3.4 La advertencia final del mandante

El acta del directorio del 14 de abril consignó que el proyecto se adjudicará a quien demuestre que **entendió la operación**, no a quien ofrezca la tecnología más avanzada, y advierte que una propuesta sofisticada que no resuelva las 48 horas sin enlace, que no diga qué hace con los stocks intermedios o que le pida cinco campos al operador de pala será superada por una más sobria que sí lo resuelva.

Los tres puntos están resueltos y son verificables en este documento: las 48 horas en el flujo 12, con la declaración de las ocho funciones que no operan y su procedimiento manual; los stocks intermedios en el flujo 4, con el supuesto del promedio ponderado **declarado y sometido a medición** en vez de asumido; y al operador de pala no se le piden cinco campos sino **un botón**.

---

## 6. Trazabilidad y verificación

### 6.1 Carga por componente

| Componente | Requerimientos | Componente | Requerimientos |
|---|---|---|---|
| C-01 Núcleo de trazabilidad | 17 | C-12 Integración con el ERP | 5 |
| C-02 Plan minero versionado | 5 | C-13 Adaptadores de telemetría | 11 |
| C-03 Captura en terreno | 10 | C-14 Historiador con segregación TI/TO | 1 |
| C-04 Nodo de borde | 14 | C-15 Detenciones y productividad | 10 |
| C-05 Identidad y acreditación | 18 | C-16 Vista única de equipo | 2 |
| C-06 Portal de empresas contratistas | 8 | C-17 Ambiental, huella y comunidad | 11 |
| C-07 Portal de trazabilidad de lote | 8 | C-18 Seguridad y salud ocupacional | 6 |
| C-08 Motor de conciliación | 16 | C-19 Analítica e indicadores | 7 |
| C-09 Integración con el laboratorio | 15 | C-20 Administración y auditoría | 20 |
| C-10 Integración con el despacho | 4 | C-21 Migración de datos históricos | 3 |
| C-11 Integración con planificación | 2 | C-22 Plataforma base | 35 |

C-22 concentra 35 requerimientos por ser la plataforma base —infraestructura, seguridad, identidad técnica y observabilidad—, no un componente de negocio. C-05 y C-20 la siguen, y ambas cifras son coherentes con el peso que el caso da a la acreditación de personas y a la parametrización sin desarrollo.

### 6.2 Cómo se verifica cada afirmación

Cada uno de los 167 requerimientos lleva en la matriz v1.1 su **prueba de verificación con criterio de paso explícito**. Las pruebas decisivas de esta especificación son cinco:

| Qué se verifica | Prueba comprometida |
|---|---|
| Las 48 horas sin enlace | Corte controlado de 48 horas con un cambio de turno dentro de la ventana; los eventos reconciliados deben coincidir exactamente con los generados, y la sincronización debe concluir en no más de 4 horas sin intervención manual |
| La reconstrucción del lote | Reconstrucción cronometrada del origen de un lote embarcado real, con evidencia, contra el umbral de 2 horas |
| La usabilidad en cabina | Medición sobre el parque real de dispositivos, con guante de invierno grueso y bajo radiación solar directa: tasa de éxito al primer intento superior al 95 % y apertura de sesión en 1 segundo |
| La portería | Medición de la consulta de habilitación en 1,5 segundos, incluido el modo degradado contra caché local |
| La adopción | Medición por área y por turno de la tasa de uso del botón de cambio de material y de la proporción de registros nacidos digitales, durante la marcha blanca (RF-MIG-005) |

---

## 7. Qué queda fuera de este documento y dónde se resuelve

| Materia | Subdocumento que la resuelve |
|---|---|
| Tecnología, productos, emplazamiento de cada componente en nube y on-premise, redes, alta disponibilidad y respaldos | Subdocumento 4.2, conforme al Artículo 16.º |
| Modelo de datos, motor de persistencia, retención, archivado y eliminación segura | Subdocumento 5 |
| Metodología de gestión y de desarrollo, gestión de interesados y de comunicaciones | Subdocumento 6 |
| Estructura de descomposición del trabajo, cronograma, olas de implantación y transferencia | Subdocumento 7 |
| Riesgos de la solución efectivamente propuesta | Subdocumento 8 |
| Plan de pruebas con niveles, ambientes y criterios de paso | Subdocumento 9 |
| Niveles de servicio, mesa de ayuda y presencia en faena | Subdocumento 10 |
| Costeo de cada función y de cada servicio declarado | Oferta Económica |

Este documento no fija metas que el caso no fije, no adelanta decisiones tecnológicas y no reemplaza la acreditación formal del Formulario T-12. Su alcance es el que anuncia: qué hace el software, quién lo toca y cómo, en qué orden ocurren las cosas, y por qué eso satisface lo que los tres documentos de la licitación exigen.
