# Subdocumento 4.1 — Arquitectura lógica de la solución

| Antecedente | Detalle |
|---|---|
| Proponente | OVERFORE S.A. |
| Licitación | TFEP-01/2026 — Caso 01 · Minería |
| Mandante | Compañía Minera Altos de Aranda S.A. — Rajo Aranda, Sierra Gorda, Región de Antofagasta |
| Contenido | a) Esquema de solución. b) Arquitectura lógica: capas, módulos, límites de contexto, responsabilidades e interfaces |
| Ponderación | 16 % de la evaluación técnica (numeral 4.1 de la pauta) |
| Documentos que rigen | Bases Administrativas FEP01.26 · Bases Técnicas Transversales FEP02.26 · Bases Técnicas del Caso 01, v1.0 del 18-08-2026 |
| Insumos internos | Subdocumento 3 v1.4 · Catálogo de requerimientos v2.4 · Matriz de trazabilidad v1.1 · Registro de supuestos v1.6 · Registro de reglas de negocio v1.2 · **Formulario T-12 v1.2**, que es la versión contra la que se verifica el reparto del apartado 19. El **T-12 v1.3** es consecuencia de este entregable, no insumo suyo, y es donde cada código apunta a su apartado de esta versión 4.1 |
| Diagrama que acompaña | `Diagrama_ArquitecturaLogica_Overfore_v3.5` |
| Versión | **4.2** — 2 de septiembre de 2026. Las versiones 1.0 a 3.1 describían **cómo dibujar** el diagrama; desde la 4.0 el documento describe **qué es** la arquitectura, para que el CLIENTE pueda verificarla. La 4.1 incorporó la revisión de verificación de códigos. La **4.2** cierra los requisitos que se declaraban sin desarrollarse y repara la trazabilidad: canales de notificación y búsqueda global (apartados 16.6 y 16.9), puntos de entrada de báscula, balanza y portería (10.1), derivación del volumen de la cola de reconciliación (12.6, SUP-40), C-22 en sus cinco capas (8 y 8.1), aritmética del reparto de códigos (19) y la lectura de los criterios n.º 5 y n.º 9 frente a la Etapa 2 (18). En el Formulario T-12 v1.3, los ochenta y cinco códigos que este subdocumento sostiene apuntan ahora a sus apartados de esta versión. |

---

## 1. Cómo leer este documento

### 1.1 Qué responde y qué no

Este subdocumento responde una sola pregunta: **de qué está hecha la solución y por qué está hecha así**. Describe las capas, los módulos, sus responsabilidades, sus fronteras y las interfaces por las que se hablan. Toda afirmación es verificable contra un requerimiento del caso, una regla de negocio o un supuesto declarado.

No responde, y lo declara para que nadie lo busque aquí:

| Pregunta | Dónde se responde |
|---|---|
| Con qué producto y versión se construye cada componente | Subdocumento 4.2 |
| Dónde se ejecuta cada componente: nube, faena o borde | Subdocumento 4.2, tabla de emplazamiento del Artículo 16.º 2 |
| Cómo es el modelo de datos físico, el linaje y el catálogo | Subdocumento 5 |
| Cómo se prueba, se despliega y se opera | Subdocumentos 6, 7, 9, 10 y 11 |
| Cuánto cuesta | Oferta Económica |

La arquitectura se describe conforme a **ISO/IEC/IEEE 42010** con las cinco vistas que exige el RT-02.03. Tres de ellas viven en este documento: la **lógica** (apartados 5 a 10), la **de procesos** (11, 12, 14 y 15) y la **de seguridad** (13). La de despliegue está en el Subdocumento 4.2 y la de datos en el Subdocumento 5, salvo el modelo de dominio del negocio, que el RT-02.13 exige aquí y está en el apartado 9.

### 1.2 Cómo verificar lo que este documento afirma

Cada afirmación de diseño lleva su origen entre paréntesis. Las convenciones son las del catálogo v2.4 y de la matriz v1.1:

- **RT-xx.xx** — requisito de las Bases Técnicas Transversales.
- **RF-xxx-000** y **RNF-xxx-000** — requerimiento funcional o no funcional del catálogo v2.4.
- **RN-00** — regla de negocio del registro v1.2.
- **SUP-00** — supuesto del registro v1.5. Un supuesto es una afirmación que el PROPONENTE hace bajo su responsabilidad y somete a validación.
- **C-00** — componente lógico. Los veintidós son los mismos del catálogo y de la matriz.
- **ADR-00** — decisión de arquitectura registrada, con su alternativa descartada. El índice está en el apartado 21 y el registro completo es entregable contractual (RT-02.04).
- **Criterio n.º 0** — criterio de aceptación del Capítulo 18 del caso.

Si el CLIENTE quiere comprobar un compromiso concreto, el apartado 18 lo lleva desde cada uno de los catorce criterios de aceptación hasta los componentes que lo sostienen.

### 1.3 Cómo leer el diagrama

El diagrama que acompaña este texto se lee **de izquierda a derecha, en el orden en que viaja una transacción**. Ocho columnas:

`Actores` · `Borde y exposición` · `Presentación` · `Puerta operacional` · `Servicios de negocio` · `Integración y eventos` · `Sistemas del CLIENTE` · `Datos`

Sobre ellas cruzan dos bandas horizontales, **Seguridad transversal** y **Observabilidad transversal**, rotuladas ambas con **C-22** porque son la forma en que la plataforma base se dibuja: no viven en una columna, aplican a todas.

**Los seis colores de conector**, con su leyenda al pie:

| Color | Qué recorre |
|---|---|
| Ámbar | Actores de terreno hacia C-03 |
| Azul | Operación, planificación y oficina hacia las superficies internas |
| Morado | Laboratorio, que entrega por su propio sistema y no por interfaz |
| Verde | Actores externos: borde, puerta de exposición y portales |
| Verde azulado | Bus de eventos, capa 5 |
| Gris pizarra | Adaptador hacia sistema del CLIENTE |
| Gris claro discontinuo | Frontera «Opera sin enlace · 48 h» |

Que el verde de los actores externos y el verde azulado del bus sean colores distintos no es estética: es lo que permite comprobar de un vistazo la afirmación del apartado 6, según la cual **ninguna ruta externa toca el registro de terreno**.

Tres detalles del dibujo que conviene tener presentes antes de leer el resto:

1. La columna `Borde y exposición` —que contiene la capa de borde y, bajo ella, la `Puerta de exposición`— sólo está en la ruta de los **actores externos** —empresa contratista, comprador y autoridad—. Un viaje de camión no las atraviesa en ningún punto de su recorrido. Es una decisión, no una omisión, y el apartado 6 la explica.
2. Los marcos de trazo discontinuo rotulados **«Opera sin enlace · 48 h»** encierran C-03 en Presentación y C-04 más la caché de credenciales en Integración. Son la misma frontera dibujada en dos columnas.
3. Tres componentes aparecen dos veces, rotulados `interfaz` y `lógica`: **C-16, C-19 y C-20**. No es un error de dibujo. El numeral 2.1 transversal prohíbe que la capa de presentación contenga lógica de negocio, de modo que la pantalla y el motor que la alimenta son piezas distintas del mismo componente y se despliegan por separado.

---

## 2. Esquema de solución (numeral 4.1 letra a de la Pauta): la solución y el problema que ataca

### 2.1 Qué es

Un **sistema de registro y trazabilidad del mineral y de las personas** que convive con el panorama de sistemas que el Capítulo 5 del caso declara que se mantienen —el ERP corporativo, el despacho de flota con contrato vigente hasta 2031, el sistema de gestión de laboratorio, el software de planificación minera, el historiador de proceso de planta en sólo lectura, los tres portales de telemetría de fabricante y la plataforma de control de fatiga— y les agrega la capa de trazabilidad que hoy no existe entre ellos.

**No reemplaza ninguno.** Esa frase es una decisión de arquitectura y no una cortesía comercial: determina que los siete puntos de contacto con sistemas del CLIENTE sean adaptadores reemplazables y no integraciones profundas, y es la razón por la que el diagrama tiene una columna entera dedicada a ello.

### 2.2 La tesis que ordena todo lo demás

La solución no es un catálogo de módulos. Es un **cambio del punto de captura del dato**.

Los tres problemas que el Subdocumento 2 identifica —trazabilidad comercial, productividad y evidencia ante terceros— comparten una sola causa: **el dato no nace donde ocurre el hecho**. Nace en una planilla que alguien digita horas o días después, y para entonces ya perdió el polígono de origen, la hora exacta, la persona que operaba y la razón del cambio. Todo lo que sigue en este documento se explica por esa tesis. Si el CLIENTE quiere verificar una sola cosa de la arquitectura, que verifique ésta: **cada hecho se registra en el lugar y el momento en que ocurre, por quien lo ejecuta, en un dispositivo inventariado, con identificador propio y firma**.

### 2.3 Las cuatro decisiones que estructuran la arquitectura

| N.º | Decisión | Qué obliga en el diseño | Regla | Supuesto |
|---|---|---|---|---|
| 1 | El **viaje de camión** es la unidad atómica de trazabilidad. El polígono es un atributo del viaje; el turno, el lote de chancado y el lote comercial son agregaciones derivadas | El agregado raíz del contexto Mineral es el viaje. Toda corrección es un evento nuevo, nunca una modificación del registro original | RN-01 | SUP-01 |
| 2 | La **balanza de alimentación de planta** es la fuente de verdad del tonelaje. El pesaje de a bordo se usa para atribución espacial prorrateada; la báscula de camiones es punto de calibración | Las tres mediciones se persisten tal como se generan y la diferencia se publica por punto de medición. Ninguna se sobrescribe con otra | RN-04 | SUP-02 |
| 3 | La **persona operadora se identifica por proximidad** de credencial en el relevo, sin login ni contraseña en terreno. Si nadie se identifica, el registro se marca «operador no identificado» y el equipo **no se detiene** | La identidad de terreno es un componente propio (C-05) y no un módulo del portal. La autenticación de terreno y la de oficina son mecanismos distintos sobre el mismo maestro de personas | RN-22 | SUP-04 |
| 4 | El diseño es **desconectado desde el origen**, no por excepción | La operación desconectada no es un componente: es una propiedad de la Etapa 1 completa, materializada por el nodo de borde (C-04) y por el registro local firmado de la aplicación de terreno (C-03) | RN-30 | SUP-09 |

### 2.4 Qué se construye, qué se integra y qué compra el CLIENTE

**Se construye:** el modelo de trazabilidad del mineral, el registro en terreno con operación desconectada, la identidad y acreditación de personas, el portal de empresas contratistas, la conciliación metalúrgica y los tableros de indicadores por turno.

**Se integra, sin sustituir:** el ERP corporativo, el sistema de despacho de flota, el sistema de gestión de laboratorio en ambos sentidos, el software de planificación minera, el historiador de proceso en sólo lectura, los tres portales de telemetría de fabricante y el sistema de control de fatiga. Cada uno queda detrás de un adaptador reemplazable con capa anticorrupción.

**Se especifica pero lo adquiere el CLIENTE:** el hardware de terreno —dispositivos, lectores, etiquetas e instrumentación adicional— con referencia, cantidad, características, accesorios y costo unitario estimado, conforme al RT-08.10 y al Formulario T-11.

---

## 3. Recorrido de extremo a extremo: qué le pasa a un viaje de camión

El apartado que sigue describe las capas una por una. Antes conviene recorrerlas en el orden real, siguiendo el hecho más frecuente del sistema: uno de los **342.000 viajes de camión al año** que el numeral 14.1 declara.

1. **Cabina de la pala.** La persona operadora acerca su credencial al lector en el relevo. La sesión se abre en **un segundo, sin retirar el guante** (SUP-29). Si el material cargado no corresponde al planificado, presiona **un botón único**: una acción, sin seleccionar causa ni escribir texto (RF-CAR-003). Ésa es toda la interacción que se le pide en un turno de doce horas.
2. **Aplicación de terreno, C-03.** El hecho se convierte en un evento con **identificador generado en el propio dispositivo**, sello de tiempo de origen y firma con clave del almacén respaldado por hardware. Se guarda en el registro local, cifrado a nivel de campo. Hasta aquí no ha hecho falta ninguna red.
3. **Nodo de borde, C-04.** El evento entra en la cola de sincronización del gabinete de borde del sitio. Si hay enlace, sale en segundos. Si no lo hay, espera: el compromiso es **48 horas continuas** de operación y registro sin enlace.
4. **Puerta operacional.** Cuando el enlace está disponible, el evento cruza la puerta de enlace de servicios que atiende a terreno y a nodos de borde. Prioriza latencia, admite lotes grandes de reconciliación y aplica calidad de servicio (RT-03.24). No es alcanzable desde Internet.
5. **Bus de eventos.** El evento se publica. El bus es la **columna vertebral de la trazabilidad**, no un accesorio de integración: entrega al menos una vez, deduplica en el consumidor y garantiza orden dentro del agregado —el lote, la sesión de operador, el equipo— cuando el proceso lo exige.
6. **Servicios de negocio.** El núcleo de trazabilidad **C-01** incorpora el viaje a la cadena del mineral. El plan versionado **C-02** aporta la versión del plan vigente en ese instante. La identidad **C-05** ya había validado al operador. Las detenciones **C-15** leen del mismo flujo. La analítica **C-19** sólo consume: no origina dato.
7. **Adaptadores.** En paralelo, **C-10** venía entregando el viaje desde el despacho de flota, **C-11** la versión del plan y **C-12** el maestro de personal. Cada uno detrás de su capa anticorrupción.
8. **Datos.** El viaje se persiste en el almacén transaccional. El indicador de turno que lo incluye se calcula en el almacén analítico, separado por diseño para que ninguna consulta degrade la operación (RT-05.05).
9. **Meses después.** Cuando el área comercial reconstruye el origen de un lote embarcado, recorre el árbol: lote comercial, lotes de chancado, retomas, viajes, polígonos, palas, fechas y operadores. **No recalcula una estimación: recorre hechos registrados.** De ahí sale el compromiso de responder en **menos de dos horas al mes 15**, contra las cuatro a seis semanas actuales.

Ese recorrido es el que el diagrama dibuja. Los apartados siguientes lo desarman.

---

## 4. Estilo arquitectónico escogido, y el descartado

El CLIENTE no impone estilo. Exige que el escogido sea explícito, coherente con el volumen del caso y justificado, y que se comparen al menos dos alternativas considerando **complejidad operacional, tamaño y competencias del equipo, costo de infraestructura y capacidad del CLIENTE de operarla al término del Contrato**. El propio documento transversal advierte que adoptar microservicios para un caso cuyo volumen no lo justifica es un error de ingeniería y se evalúa como tal.

### 4.1 Lo escogido: servicios de negocio modulares con despliegue independiente

La capa de servicios de negocio se organiza en **diez servicios de grano medio**, desplegables de forma independiente, sin estado y con contrato versionado. No se subdivide en microservicios de grano fino.

Las cifras que sostienen la decisión:

| Dimensión | Valor declarado | Fuente |
|---|---|---|
| Transacciones por segundo en régimen | 40 | SUP-31 |
| Transacciones por segundo en peak, en el cambio de turno | 200 | SUP-31 |
| Equipo de TI del CLIENTE que recibe la operación | 11 personas | Restricción n.º 10 |
| Sitios operacionales a cubrir | 7, incluido un acopio en recinto portuario de terceros, con capacidad de incorporar un octavo sin rediseño | Numeral 14.1, RNF-CAP-011 |
| Duración de la Operación tras la implementación | 36 meses, del mes 21 al 56 | Artículo 17.º |

Doscientas transacciones por segundo en peak no justifican descomposición fina. Once personas no pueden operar una malla de servicios al término del Contrato. **La decisión se toma por la restricción n.º 10 antes que por preferencia técnica** (ADR-01).

### 4.2 Lo descartado, y por qué

| Criterio del numeral 2.3 | Microservicios de grano fino | Servicios modulares (escogido) |
|---|---|---|
| Complejidad operacional | Malla de servicios, descubrimiento, trazas distribuidas sobre decenas de procesos, gestión de versiones cruzadas | Diez unidades desplegables, trazas correlacionadas sobre un número acotado de saltos |
| Tamaño y competencias del equipo | Exige especialidad en plataforma que el CLIENTE declara no tener | Operable con perfiles de administración de aplicaciones y de base de datos |
| Costo de infraestructura | Sobredimensionado para 40 transacciones por segundo en régimen | Escala horizontal por servicio sólo donde el peak lo exige |
| Capacidad del CLIENTE al término del Contrato | Riesgo alto de dependencia permanente del ADJUDICATARIO | Transferencia viable, conforme al numeral 17.6 punto 7 |

**El monolito único también se descarta**, y no por preferencia: lo prohíbe el RT-02.02, que exige poder desplegar de forma independiente los componentes críticos. El registro de terreno y la conciliación tienen ciclos de cambio y ventanas de intervención incompatibles —la conciliación no se toca durante el cierre contable, el registro de terreno no se toca durante el cambio de turno—, y un monolito obliga a desplegarlos juntos.

### 4.3 Qué significa «desplegable de forma independiente» en concreto

Cuatro piezas son críticas y se despliegan por separado, cada una con su contrato versionado: el **núcleo de trazabilidad (C-01)**, la **identidad y control de acceso de personas (C-05)**, el **nodo de borde (C-04)** y la **puerta de enlace de servicios**.

La consecuencia verificable: un despliegue de la analítica (C-19) o del módulo ambiental (C-17) **no puede detener el registro de un viaje**.

---

## 5. Las ocho capas

El numeral 2.1 transversal fija ocho capas de existencia obligatoria (RT-02.01). A continuación, qué contiene cada una en esta solución, qué hace y —tan importante como lo anterior— **qué no hace**.

### Capa 1 · Presentación

**Responsabilidad.** Interfaces de las personas usuarias: portal web, aplicación móvil, terminales operacionales y pantallas de terreno.

**Exigencia mínima.** Diseño adaptativo, accesible y **sin lógica de negocio**. Ninguna interfaz accede directamente a la base de datos.

**Qué contiene.** Seis frentes de presentación: `C-03` captura en terreno, `C-06` portal de empresas contratistas, `C-07` portal de trazabilidad de lote, y las interfaces de `C-16`, `C-19` y `C-20`.

**El caso singular es C-03.** Es la única pieza de presentación que conserva **estado local persistente**, porque debe operar 48 horas sin enlace. Ese estado es un registro de eventos firmado, no una copia de la base de datos central, y no contiene lógica de negocio: contiene **hechos observados pendientes de reconciliación**. La distinción no es semántica. Un dispositivo que guardara lógica de negocio quedaría desactualizado respecto del servidor durante el corte y produciría dos verdades; uno que guarda hechos, no.

**Lo que condiciona el diseño de terreno.** Guantes gruesos y lentes de seguridad; luminosidad de sol directo a oscuridad total; temperaturas de −6 °C a 34 °C; polvo en suspensión permanente. **Toda interfaz que exija retirar el guante es inoperable.** En cabina el único elemento nuevo es un botón único.

### Capa 2 · Borde y exposición

**Responsabilidad.** Único punto de entrada público: distribución de contenidos, balanceo, protección perimetral y terminación de cifrado.

**Exigencia mínima.** Distribución de contenidos, cortafuegos de aplicación gestionado, protección contra denegación de servicio en capas 3, 4 y 7, y terminación TLS 1.3.

**Qué contiene.** Un único frente público para los dos portales autenticados: empresas contratistas (C-06) y trazabilidad de lote para compradores (C-07). **No existe portal abierto a la ciudadanía**: el caso lo excluye.

**La advertencia que importa.** Esta capa **no está en la ruta del registro de terreno**. Un viaje de camión no atraviesa el borde público. Confundirlo haría que un ataque de denegación de servicio al portal del comprador detuviera la mina. El aislamiento de recursos del portal respecto del núcleo transaccional es requisito propio (RNF-DIS-010), con umbral de **cero degradación de los umbrales de terreno ante diez veces el tráfico medio del portal**.

### Capa 3 · Puerta de enlace de servicios

**Responsabilidad.** Publicación, autenticación, autorización, cuotas, límites de tasa, versionado y observabilidad de las interfaces de programación.

**Exigencia mínima.** Validación de esquema, inspección de carga útil, trazabilidad por transacción y catálogo de servicios.

**Qué contiene: dos puertas lógicas con política distinta**, porque los perfiles de tráfico son incompatibles (ADR-07).

| | Puerta operacional | Puerta de exposición |
|---|---|---|
| A quién sirve | Terreno y nodos de borde | Portales externos e integraciones de sistemas |
| Qué prioriza | Latencia; admite lotes grandes de reconciliación al restablecerse el enlace; calidad de servicio conforme al RT-03.24 | Cuota y límite de tasa **por consumidor** |
| Autenticación | Autenticación mutua TLS del nodo de borde y del dispositivo inventariado | OAuth 2.1 con credenciales de cliente; sesión de persona usuaria en los portales |
| Qué hace con un mensaje que no valida | **Nunca lo pierde.** Va a la cola de mensajes fallidos con su carga íntegra y se resuelve | Rechaza con error explícito |
| Alcanzable desde Internet | **No** | Sí, tras la capa de borde |

La diferencia de la última fila es la que sostiene el criterio n.º 13: el hecho de terreno ocurrió, y los eventos reconciliados deben coincidir exactamente con los generados.

**Contratos.** Servicios síncronos documentados en **OpenAPI 3.1**; flujos dirigidos por eventos en **AsyncAPI 3.0**; ambos generados desde el código (RT-05.16). Versionado semántico con compatibilidad hacia atrás y obsolescencia con **preaviso mínimo de seis meses** (RT-05.17). Queda prohibida la clave estática en la ruta de la dirección web, sin excepción (RT-05.18).

### Capa 4 · Servicios de negocio

**Responsabilidad.** Lógica del proceso de negocio del caso, organizada en módulos con límites de contexto explícitos.

**Exigencia mínima.** Sin estado, desplegables de forma independiente, con contratos versionados y compatibilidad hacia atrás.

**Qué contiene.** Los **diez servicios** de grano medio: `C-01`, `C-02`, `C-05`, `C-08`, `C-15`, `C-16`, `C-17`, `C-18`, `C-19` y `C-20`, repartidos en **seis de los nueve límites de contexto** del apartado 7.

**Sin estado (RT-02.05).** El estado de sesión y el estado de proceso residen en almacenes externos de alta disponibilidad, nunca en memoria del servicio. La única excepción declarada es el estado local del contexto Terreno, que es un registro de eventos pendientes de reconciliación y no estado de sesión.

**La regla que gobierna esta capa.** Los umbrales, plazos, tolerancias y catálogos son **parámetros de administración versionados, no constantes de código** (RN-27, RT-16.02). Un período anterior se recalcula con la versión de regla que regía entonces. Sin esa propiedad, los criterios de aceptación n.º 3 y n.º 4 no se pueden verificar sobre historia ya cerrada.

### Capa 5 · Integración y eventos

**Responsabilidad.** Comunicación asíncrona, desacoplamiento, orquestación y coreografía de procesos entre módulos y con sistemas externos.

**Exigencia mínima.** Bus o intermediario de mensajería con persistencia, cola de mensajes fallidos, reintento y deduplicación.

**Qué contiene.** El **bus de eventos**, el **nodo de borde C-04** con su caché de credenciales, y los **siete adaptadores externos**.

**El bus es la columna vertebral de la trazabilidad, no un accesorio de integración** (ADR-04). Todo hecho de terreno entra al sistema como evento con identificador generado en el dispositivo de origen, lo que hace la reconciliación idempotente y reintentable (RN-30, RT-02.06). Entrega al menos una vez con deduplicación en el consumidor, y orden garantizado dentro del agregado cuando el proceso lo exige (RT-02.07): el orden importa dentro de un lote y dentro de una sesión de operador, **no globalmente**.

La alternativa descartada fue la integración punto a punto por servicio síncrono. Se descarta porque la trazabilidad exige conservar el linaje del hecho, el punto a punto no lo conserva, y sobre todo **no sobrevive al corte de enlace**.

**Los siete adaptadores**, cada uno con capa anticorrupción y reemplazable por separado:

`C-09` laboratorio · `C-10` despacho de flota · `C-11` planificación minera · `C-12` ERP · `C-13` telemetría, con **un adaptador por cada uno de los tres fabricantes** · `C-14` historiador de proceso con segregación TI/TO · y el **adaptador del sistema de control de fatiga**, única pieza de integración sin componente propio en el catálogo: pertenece a `C-18` pero se ejecuta en esta capa, porque transporta dato personal sensible bajo la Ley N.º 21.719 y queda aislado tras la misma capa anticorrupción que los demás.

### Capa 6 · Datos

**Responsabilidad.** Persistencia transaccional, analítica, documental, de series de tiempo y de archivos.

**Exigencia mínima.** Separación entre lo transaccional y lo analítico, cifrado en reposo, respaldo y retención declarados.

**Qué contiene: cinco naturalezas de persistencia operacional/analítica, más el repositorio de consulta histórica (seis almacenes lógicos en total).** El detalle físico es materia del Subdocumento 5; aquí se declara la separación lógica y su razón.

| Naturaleza | Qué guarda | Por qué está separada |
|---|---|---|
| Transaccional | Viajes, lotes, personas, habilitaciones, detenciones, plan versionado | Ninguna consulta analítica puede degradar el desempeño de la operación (RT-05.05) |
| Analítica | Indicadores por turno, conciliación, series de cumplimiento | El caso fija tres latencias distintas (RT-05.29): indicadores de turno ≤ 15 minutos, indicadores de gestión y cumplimiento ≤ 4 horas, conciliación cerrada en ≤ 3 días hábiles. La primera es incompatible con los umbrales de terreno, y es la que obliga a separar el almacén |
| Series de tiempo | Telemetría de equipos, lecturas del historiador, mediciones ambientales | Volumen y patrón de escritura distintos del transaccional |
| Documental | Certificados de análisis, documentación de habilitación, reportes firmados | Versionado, metadato y búsqueda por contenido (RT-16.15) |
| Archivos | Evidencia fotográfica de inspecciones e incidentes, respaldos de grabación | Tamaño y ciclo de vida propios |
| Repositorio de consulta histórica | Lo que queda fuera de la ventana de migración: trazabilidad y embarques del sexto al décimo año, mantenimiento anterior a tres años, laboratorio anterior a cinco | Sólo lectura. Responde consultas con la marca «origen anterior al sistema». **No alimenta** conciliación, indicadores ni recálculo retroactivo (SUP-37, RN-28) |

**Retención por tipo de registro, sin borrado anticipado (RN-28):** seguridad y salud ocupacional 10 años; ambientales y de monitoreo 10; trazabilidad de mineral y embarques 10; operacionales 5; tributarios 7, consultados en el ERP que no se reemplaza.

### Capa 7 · Seguridad transversal

**Responsabilidad.** Identidad, autorización, gestión de secretos, cifrado, registro de auditoría y detección.

**Exigencia mínima.** Aplicada a **todas las capas, no como capa perimetral única**. Por eso se dibuja como banda.

**Qué contiene: tres mecanismos de identidad sobre un solo maestro de personas**, porque los perfiles operacionales son irreductibles entre sí (ADR-06).

| Población | Mecanismo | Por qué |
|---|---|---|
| Personal propio, 1.780 personas | Federación con el directorio corporativo del CLIENTE, con **doble factor**, sin credenciales compartidas y con registro completo de accesos | Restricción n.º 9 en su enunciado completo, interpretada como aplicable al personal propio (SUP-08, RF-IDE-001) |
| Personas contratistas: 2.400 en régimen, de unas 90 empresas; hasta 6.700 en detención mayor de planta (8.500 personas totales con acceso al sitio minero, sumando propios) | Credencial física emitida en la acreditación. En terreno basta el toque sobre dispositivo inventariado; fuera del ciclo productivo se exige además PIN numérico. **Administración delegada a cada empresa** desde el portal | RT-12.12 exige mecanismo autoservido. Provisionar hasta 8.500 identidades en el directorio corporativo con 11 personas de TI es inviable operacionalmente (SUP-08, RF-IDE-003). Esta delimitación de la Restricción n.º 9 se someterá formalmente a ratificación en el período de consultas del Artículo 43.º de las Bases Administrativas |
| Persona operadora en equipo compartido | Credencial de proximidad leída en cabina en el relevo, **sin login ni contraseña escrita en terreno** | RT-12.11 fija el perfil operacional y prohíbe la contraseña alfanumérica en terreno (SUP-04) |

El apartado 13 desarrolla el resto de la vista de seguridad.

### Capa 8 · Observabilidad transversal

**Responsabilidad.** Métricas, registros y trazas distribuidas correlacionadas.

**Exigencia mínima.** Instrumentación conforme a **OpenTelemetry**, cobertura de nube y faena **sin puntos ciegos**.

**Qué contiene.** Una sola plataforma con alertamiento unificado para nube y faena (RT-03.16, RT-14.01). El nodo de borde emite su propia telemetría de salud y, mientras está sin enlace, la acumula localmente junto con los eventos de negocio: **la observabilidad se degrada igual que el resto y se reconcilia igual que el resto**.

**Indicador propio de este caso.** La observabilidad debe exhibir, por sitio, las **horas acumuladas sin enlace** y el **retraso de reconciliación pendiente**. Es la evidencia con que se verifica el criterio de aceptación n.º 13. El apartado 14 lo desarrolla.

---

## 6. Cómo entra cada actor al sistema

El diagrama agrupa los diecinueve actores en cuatro familias, cada una con su propio color de conector. La agrupación no es cosmética: **cada familia entra por una ruta distinta, y esa ruta es una decisión de arquitectura**.

| Familia | Quiénes | Por dónde entran | Por qué |
|---|---|---|---|
| **Terreno** | Operador de pala, de camión y de perforadora; supervisor de turno; portería; persona que toma la muestra | Directo a `C-03`, y de ahí al nodo de borde `C-04` | Deben poder registrar sin red. Si pasaran por el borde público, un corte de enlace o un ataque al portal los detendría |
| **Operación, planificación y oficina** | Despacho, metalurgia, planificación minera, mantenimiento, seguridad y salud, sustentabilidad, comercial, Centro Integrado de Operaciones, administrador | A las superficies internas `C-16`, `C-19` y `C-20`, y de ahí a la **puerta operacional** | Son personal propio, dentro de la red corporativa, con federación y doble factor |
| **Tercero que entrega dato por sistema** | Laboratorio | **No usa la interfaz.** Entrega el resultado validado por su propio sistema de gestión, que el adaptador `C-09` lee | El laboratorio no cambia de herramienta. La solución se adapta a él, no al revés |
| **Externos** | Empresa contratista, comprador, autoridad ambiental, y —conforme al RT-12.12— el operador del terminal portuario y los laboratorios externos | Capa de **borde y exposición**, luego **puerta de exposición**, y sólo entonces `C-06`, `C-07` o el acceso acotado que sirve `C-09` | Es la única superficie alcanzable desde Internet. Todo lo demás queda detrás |

La lectura que el CLIENTE debe poder hacer del diagrama es ésta: **las dos bandas de exposición sólo tocan la ruta verde**. Ningún conector naranja ni azul las atraviesa. Ése es el aislamiento del que depende que el portal del comprador no sea una vía hacia la mina.

---

## 7. Los nueve límites de contexto

El RT-02.02 exige límites de contexto explícitos. Un contexto tiene lenguaje propio, un agregado raíz y una **frontera declarada de lo que no le pertenece**. La frontera es la parte que se evalúa: un contexto sin exclusiones declaradas no es un límite, es una etiqueta.

| Contexto | Componentes | Agregado raíz | Lo que **no** le pertenece |
|---|---|---|---|
| **Mineral** | C-01, C-02, C-08, más los adaptadores C-10, C-11 y C-12 que lo alimentan (C-12 también provee el maestro de personal propio hacia Personas) | El **viaje** | La identidad de quien operó el equipo, que es de Personas; el resultado de laboratorio, que es de Calidad; la cifra contable oficial, que la emite el ERP (restricción n.º 4) |
| **Terreno** | C-03, C-04 | El **evento de terreno**, con identificador generado en el dispositivo | La interpretación de negocio del evento. El terreno registra que un camión descargó en un punto a una hora; **no calcula la ley ni el cumplimiento del plan** |
| **Personas** | C-05, C-06 | La **persona habilitada** con sus habilitaciones vigentes | El dato laboral y de remuneraciones, que vive en el maestro del ERP y no se duplica |
| **Calidad** | C-09 | La **muestra** y su análisis | La decisión comercial de liberar un lote. Calidad publica que un análisis es válido y vigente; la máquina de estados del lote vive en Mineral |
| **Activos** | C-13, C-14, C-15, C-16 | El **equipo** y su línea de tiempo de estados | La orden de trabajo de mantenimiento, que vive en el ERP; la operación autónoma de equipos, excluida del alcance |
| **Cumplimiento** | C-17, C-18 | El **compromiso** —ambiental, de seguridad, de comunidad o de cierre de faena— y su evidencia | El dato operacional de origen. **El cumplimiento no mide, referencia**: cada cifra de un reporte se navega hasta su medición de origen |
| **Comercial** | C-07, más la vista de exposición del contexto Mineral | El **certificado derivado** de un lote embarcado | **La ley por polígono, que no se expone en ningún caso.** Es la frontera más importante del sistema |
| **Analítico** | C-19 | El **indicador** con su definición versionada | Originar dato. **Sólo consume eventos publicados.** Si un indicador necesita un dato que nadie publica, la corrección es publicar el evento, no leer la base transaccional del vecino |
| **Plataforma** | C-20, C-21, C-22 | El **parámetro versionado** y el **asiento de auditoría** | Ninguna regla de negocio del caso. La plataforma **custodia** las reglas; no las decide |

### 7.1 Los tres contextos que no tienen servicio propio en la capa 4

Seis contextos tienen servicio en la capa de negocio. Tres no, y es deliberado:

- **Terreno.** Su lógica vive en el borde (C-04) y en el dispositivo (C-03), porque debe operar 48 horas sin enlace.
- **Calidad.** Su lógica vive en el sistema de gestión de laboratorio del CLIENTE, que no se reemplaza y al que sólo se accede por adaptador (C-09).
- **Comercial.** Es una vista de exposición filtrada sobre el núcleo de trazabilidad, no un servicio con estado propio.

**Un contexto sin servicio en la capa 4 no es un contexto vacío: es un contexto cuya lógica se ejecuta en otra capa.** En el diagrama, esos tres se rotulan sobre el grupo al que su lógica pertenece.

### 7.2 Cómo se hablan los contextos entre sí

| Aguas arriba | Aguas abajo | Patrón |
|---|---|---|
| Terreno | Mineral, Personas, Activos, Cumplimiento | Publicación de eventos. El productor no conoce a sus consumidores |
| Personas | Mineral, Terreno (caché de credenciales), Cumplimiento | Conformista: el consumidor acepta el modelo de identidad sin traducirlo |
| Calidad | Mineral, Comercial | Conformista sobre el resultado del laboratorio, que es fuente de verdad de la ley (RN-10) |
| Mineral | Comercial, Analítico, Cumplimiento | Publicación de eventos, más vista de exposición filtrada hacia Comercial |
| Sistemas del CLIENTE | Todos | **Capa anticorrupción por sistema. El modelo externo nunca entra al núcleo** |
| Plataforma | Todos | Núcleo compartido acotado: sólo parámetros versionados, perfiles y auditoría |

### 7.3 Las reglas duras que cada contexto impone

Estas son las que el CLIENTE puede verificar en operación:

- **Mineral.** Ninguna diferencia de conciliación se cierra como «no explicada». La diferencia residual queda atribuida a un punto de medición de la cadena. Es la excepción declarada de RN-06 y sostiene el criterio n.º 3.
- **Terreno.** Para hechos observados en terreno **prevalece el evento de terreno**; para datos maestros prevalece el registro central (RN-30). En el relevo sin enlace, la sesión saliente se cierra y la entrante se abre localmente, **sin fusión de eventos**.
- **Personas.** Una persona ingresa sólo con habilitación vigente, validada en portería en **no más de 1,5 segundos** (RN-20). El acceso se revoca en no más de quince minutos desde el término de la relación con la faena (SUP-28). **El equipo nunca se detiene por falta de identificación**: el hueco es indicador gestionable, no bloqueo (RN-22).
- **Calidad.** Ningún lote se libera a embarque sin análisis válido y vigente, y la liberación **no admite excepción manual** (RN-11). El muestreo obligatorio se ubica donde el material ya se detiene o donde lo opera un rol distinto del ciclo de mina, **nunca en el ciclo de pala o de camión** (RN-12).
- **Activos.** La detención se clasifica con taxonomía **cerrada y jerárquica de tres niveles, sin texto libre**, de forma automática desde señales que el despacho, el plan y la telemetría ya generan. **El operador de equipo no clasifica nunca**; el supervisor resuelve sólo el residuo desde lista cerrada (RN-17). El turno no cierra con detenciones sin clasificar (RN-18).
- **Cumplimiento.** Todo reporte a la autoridad se genera desde el sistema y **ninguno se construye en planilla al margen** (RN-25). Los factores de emisión se conservan versionados y cada dato de actividad conserva su evidencia, de modo que un cálculo pasado se reproduce exactamente (RN-26).
- **Comercial.** La liquidación se calcula sobre un **vector de elementos** —cobre, oro, plata y arsénico como elemento deletéreo— y no sobre la ley de cobre sola (RN-14). El arsénico proyectado se alerta **antes** de conformar el lote definitivo: es el control que ataca las cuatro penalizaciones de USD 1,9 millones.
- **Analítico.** El turno operacional es de doce horas, día de 08:00 a 20:00 y noche de 20:00 a 08:00, y todo indicador por turno se calcula sobre esa definición (RN-23).
- **Plataforma.** Toda regla y todo parámetro tienen vigencia temporal versionada, y un período anterior se recalcula con la versión que regía entonces (RN-27).

---

## 8. Los veintidós componentes

Los componentes lógicos son los mismos `C-01` a `C-22` del catálogo v2.4 y de la matriz v1.1. La columna de etapa proviene de la matriz.

| Componente | Qué hace | Capa | Contexto | Etapa |
|---|---|---|---|---|
| **C-01** Núcleo de trazabilidad | Mantiene la cadena del mineral: viaje, stock, lote de chancado, lote comercial, con relación padre-hijo y proporción | Servicios | Mineral | 1 |
| **C-02** Plan minero versionado | Conserva cada versión del plan como entidad inmutable, contra la que se mide todo movimiento | Servicios | Mineral | 1 |
| **C-03** Captura en terreno | Registra el hecho donde ocurre, con firma, identificador de origen y operación desconectada | Presentación | Terreno | 1, salvo índices de dureza de RF-PET-006 |
| **C-04** Nodo de borde | Registro local, cola de eventos, caché de credenciales y sincronización por sitio | Integración | Terreno | 1 |
| **C-05** Identidad y acreditación | Maestro único de personas, tres mecanismos de identidad, habilitaciones y control de acceso | Servicios y Seguridad | Personas | 1 |
| **C-06** Portal de empresas contratistas | Administración delegada de identidades y documentación por cada empresa | Presentación | Personas | 1 |
| **C-07** Portal de trazabilidad de lote | Certificado derivado para el comprador, filtrado según RN-16 | Presentación | Comercial | **2** (7 de 8 requerimientos) |
| **C-08** Motor de conciliación | Balance metalúrgico con factores versionados; atribuye la diferencia a un punto de medición | Servicios | Mineral | 1 |
| **C-09** Integración con laboratorio | Solicita análisis y recibe resultado validado. Bidireccional | Integración | Calidad | 1 |
| **C-10** Integración con despacho de flota | Lee el ciclo mina: pala de origen, destino, tonelaje, posición y hora | Integración | Mineral | 1 |
| **C-11** Integración con planificación minera | Lee la versión vigente del plan. **No** interviene el modelo de bloques | Integración | Mineral | 1 |
| **C-12** Integración con ERP | Entrega tonelaje, ley y metal contenido en cada cierre; recibe el maestro de personal | Integración | Mineral (con provisión a Personas) | 1 |
| **C-13** Adaptadores de telemetría | Tres adaptadores, uno por fabricante, cada uno con capa anticorrupción | Integración | Activos | **2** (8 de 11) |
| **C-14** Lectura del historiador | OPC UA con segregación TI/TO. **Sólo lectura, sin excepción** | Integración | Activos | 1 |
| **C-15** Detenciones y productividad | Clasificación automática con taxonomía cerrada; escalamiento del residuo | Servicios | Activos | 1 |
| **C-16** Vista única de equipo | Compone estado, historial, alarmas y pautas desde C-10, C-12 y C-13 | Presentación y Servicios | Activos | Versión completa en 2 |
| **C-17** Ambiental, huella de carbono y comunidad | Compromisos de la RCA, huella con alcances 1, 2 y 3, cierre de faenas | Servicios | Cumplimiento | **2** (11 de 11) |
| **C-18** Seguridad y salud ocupacional | Incidentes, inspecciones, gestión de alertas de fatiga | Servicios | Cumplimiento | **2** (5 de 6) |
| **C-19** Analítica, indicadores y reportería | Calcula el indicador contra su definición versionada, sobre el almacén analítico separado | Presentación y Servicios | Analítico | 1 |
| **C-20** Administración, parametrización y auditoría | Motor de vigencia temporal de parámetros, perfiles, catálogo de dispositivos, asiento de auditoría | Presentación y Servicios | Plataforma | 1 |
| **C-21** Migración de datos históricos | Carga la ventana migrada y alimenta el repositorio de consulta histórica | Datos | Plataforma | 1, salvo mantenimiento y monitoreo ambiental |
| **C-22** Plataforma base | Infraestructura, seguridad técnica, identidad técnica y observabilidad, y el sustrato del bus de eventos | Borde, Puerta, Integración, Seguridad y Observabilidad | Plataforma | 1 |

### 8.1 Dos notas que evitan malas lecturas del diagrama

**C-22 concentra 35 requerimientos** y es el componente más cargado del catálogo. No es un componente de negocio: es la **plataforma base**, y por eso aparece en **cinco capas**: borde, puerta de enlace, integración —donde sostiene el bus de eventos, que no es un componente de negocio y por eso no lleva código propio—, seguridad y observabilidad. En el diagrama se representa por las dos bandas transversales, rotuladas con su código, y por las columnas de borde y de puerta, no como una caja de negocio.

**C-16, C-19 y C-20 figuran a la vez en Presentación y en Servicios.** Es deliberado y responde a la exigencia del numeral 2.1 de que la capa de presentación no contenga lógica de negocio:

- **C-20.** El RT-16.02 exige que las reglas sean configurables desde la interfaz de administración y el RT-16.03 exige aprobación de un segundo perfil para todo cambio con impacto operacional. La interfaz está en Presentación; el **motor de vigencia temporal de parámetros**, en Servicios.
- **C-16.** La pantalla está en Presentación; la **composición del cuadro** a partir de C-10, C-12 y C-13 es lógica de negocio. Es lo que permite medir el criterio n.º 6 con una prueba cronometrada contra los 15 a 20 minutos y cuatro sistemas actuales.
- **C-19.** Los tableros están en Presentación; el **cálculo del indicador contra su definición versionada** está en Servicios, sobre el almacén analítico separado que exige el RT-05.05.

---

## 9. El modelo de dominio: qué existe y qué lo mueve

El RT-02.13 exige el modelo de dominio del negocio en este subdocumento, y exige declarar **los eventos, no sólo las entidades**. Los eventos son el contrato real entre contextos.

### 9.1 Entidades principales

| Entidad | Contexto | Relaciones esenciales |
|---|---|---|
| Polígono | Mineral | Atributo de origen del Viaje. No es agregado: hereda su propio error |
| **Viaje** | Mineral | Uno a uno con Pala de origen y Polígono; uno a muchos con Pesaje; destino a Stock o a Chancado |
| Pesaje | Mineral | Tres orígenes por viaje o por lote: pesaje de a bordo, báscula de camiones, balanza de alimentación. **Los tres se conservan** |
| Stock | Mineral | Uno a muchos con Aporte y con Retoma; mantiene ley ponderada vigente |
| Aporte a stock | Mineral | Conserva polígono de origen, tonelaje, ley y fecha |
| Retoma de stock | Mineral | Conserva la proporción con que cada polígono contribuyó |
| Lote de chancado | Mineral | Agregación de Viajes y Retomas |
| Lote comercial | Mineral / Comercial | Registro **inmutable** de eventos, con relación padre-hijo hacia los lotes que lo componen y su proporción |
| Embarque | Comercial | Uno a uno con Lote comercial; uno a muchos con Liquidación |
| Muestra | Calidad | Asociada a Punto de muestreo y a Lote o Viaje |
| Análisis | Calidad | Vector de elementos: Cu, Au, Ag, As. Estado de validez y vigencia |
| Plan minero (versión) | Mineral | **Versionada e inmutable**; cada movimiento referencia la versión vigente en su instante |
| Persona | Personas | Uno a muchos con Habilitación y con Credencial |
| Habilitación | Personas | Tipo, vigencia, estado. **Dato sensible bajo Ley N.º 21.719** |
| Empresa contratista | Personas | Uno a muchos con Persona; administra su propia carga documental |
| Sesión de operador | Terreno / Personas | Persona, Equipo, apertura y cierre; abierta por proximidad en el relevo |
| Equipo | Activos | Uno a muchos con Detención, Pauta y Alarma |
| Detención | Activos | Causa de taxonomía cerrada de tres niveles; estado de clasificación |
| Compromiso | Cumplimiento | Frecuencia, punto de medición, destinatario, vencimiento |
| Medición ambiental | Cumplimiento | Origen navegable hasta el dato de actividad |
| Parámetro / Regla | Plataforma | Vigencia temporal versionada; autor, fecha y justificación |
| Evento de terreno | Terreno | Identificador generado en dispositivo; sello de tiempo y posición de origen |

### 9.2 Eventos de negocio

**Cadena del mineral:** `ViajeRegistrado` · `PesajeCapturado` · `AporteAStockRegistrado` · `RetomaDeStockRegistrada` · `LoteDeChancadoConformado` · `LoteComercialConformado` · `LoteLiberado` · `LoteBloqueado` · `EmbarqueDespachado` · `DevoluciónDeConcentradoRegistrada` · `LiquidaciónRecibida`

**Calidad:** `MuestraTomada` · `AnálisisPublicado` · `AnálisisRechazado` · `RemuestreoSolicitado` · `ArbitrajeResuelto` · `ArsénicoProyectadoSobreLímite`

**Personas:** `PersonaAcreditada` · `HabilitaciónPorVencer` · `HabilitaciónVencida` · `AccesoConcedido` · `AccesoDenegado` · `OperadorIdentificado` · `OperadorNoIdentificado` · `AccesoRevocado`

**Activos y productividad:** `DetenciónDetectada` · `DetenciónClasificadaAutomáticamente` · `DetenciónPendienteEscalada` · `AlarmaDeFabricanteRecibida`

**Plan y conciliación:** `PlanPublicado` · `CumplimientoDeTurnoCalculado` · `ConciliaciónCerrada` · `DiferenciaAtribuidaAPuntoDeMedición`

**Cumplimiento:** `InspecciónRegistrada` · `IncidenteRegistrado` · `AlertaDeFatigaRecibida` · `AcciónDeFatigaDocumentada` · `ReporteGenerado` · `HuellaDeCarbonoCalculada`

**Terreno y plataforma:** `EnlacePerdido` · `EnlaceRestablecido` · `ReconciliaciónCompletada` · `ConflictoResuelto` · `ParámetroModificado` · `ReglaVersionada`

### 9.3 La propiedad que sostiene el compromiso más visible

`LoteComercialConformado` conserva la relación padre-hijo hacia los lotes que lo componen y su proporción; cada uno hacia sus retomas y viajes; cada viaje hacia su polígono, su pala, su fecha y su operador identificado.

**Reconstruir el origen de un lote embarcado es recorrer ese árbol, no recalcular una estimación.** De ahí sale el compromiso del criterio n.º 1: menos de dos horas al mes 15, contra las cuatro a seis semanas actuales.

---

## 10. Interfaces

### 10.1 Con los siete sistemas del CLIENTE

| Sistema | Sentido | Protocolo y patrón | Frecuencia | Componente |
|---|---|---|---|---|
| ERP corporativo | **Bidireccional.** Recibe tonelaje, ley, metal contenido, consumos y movimientos en cada cierre; entrega el maestro de personal propio | Servicio síncrono en OpenAPI 3.1, más archivo por lote en el cierre | En cada cierre contable; maestro diario | C-12 |
| Despacho de flota, contrato hasta 2031 | Lectura principal del ciclo mina. Escritura sólo de la marca de estado que el propio despacho admita | Adaptador con capa anticorrupción; ingesta continua de eventos | Continua | C-10 |
| Sistema de gestión de laboratorio | **Bidireccional.** Envía solicitud de análisis y recibe resultado validado | Adaptador con capa anticorrupción; evento en AsyncAPI 3.0 | Continua; ~71.000 muestras al año | C-09 |
| Software de planificación minera | Lectura de la versión vigente del plan. **No** interviene el modelo de bloques | Adaptador; ingesta por publicación de versión | 3 a 4 veces por semana | C-11 |
| Historiador de proceso de planta | **Sólo lectura. Nunca se escribe en la red de control** | OPC UA con segregación conforme a IEC 62443 | Continua | C-14 |
| Tres portales de telemetría de fabricante | Lectura de estado, posición, horómetro y alarmas | **Un adaptador por fabricante**, cada uno reemplazable | Latencia de ingesta declarada de 5 minutos (SUP-30) | C-13 |
| Sistema de control de fatiga | Lectura de alertas emitidas | Adaptador; dato personal sensible, cifrado a nivel de campo | Continua | C-18 |

**Los tres puntos de entrada de instrumentación, que no son sistemas de gestión.** El Capítulo 5 del caso lista, además de los siete anteriores, el sistema de pesaje de báscula de camiones y el control de acceso de portería, y el RT-17.06 los enumera entre los periféricos a integrar. No son sistemas de gestión con modelo propio, sino instrumentos que entregan una lectura, y por eso no llevan adaptador con capa anticorrupción sino un punto de captura:

| Instrumento | Cómo entra la lectura | Por qué así | Componente |
|---|---|---|---|
| **Báscula de camiones**, en la salida del área de descarga y en el despacho a puerto | Punto de captura del nodo de borde del sitio, que lee el indicador de la báscula y publica el evento `PesajeCapturado` con identificador de origen | El caso admite reemplazar su software local; la lectura es un hecho de terreno y debe sobrevivir al corte de enlace igual que los demás. El umbral de 3 segundos del apartado 14.2 se mide aquí | C-04, publica a C-01 |
| **Balanza de alimentación de planta**, fuente de verdad del tonelaje por la decisión n.º 2 | Lectura del historiador de proceso por OPC UA, sin escritura, como cualquier otra señal de planta | Es instrumentación de la planta y su señal ya vive en el historiador. Traerla por otra vía crearía una segunda lectura del mismo instrumento | C-14 |
| **Torniquetes de portería** | Validación contra la caché local de credenciales del nodo de borde, con respuesta en 1,5 segundos sostenida durante un corte (RN-20) | El equipamiento se mantiene o se amplía; lo que se incorpora es la gestión de la acreditación, que es de C-05. La validación no puede depender del enlace | C-04 y C-05 |

Las tres mediciones de tonelaje —pesaje de a bordo por C-10, báscula de camiones por C-04 e historiador por C-14— **se persisten tal como se generan y ninguna sobrescribe a otra** (RN-04). Es la condición del criterio n.º 3: sin las tres, la diferencia no se puede atribuir a un punto de medición.

**Por qué los siete usan capa anticorrupción (RT-02.14).** No es sofisticación: el Capítulo 5 del caso declara que el CLIENTE **no tiene documentadas** las interfaces, las versiones ni las condiciones contractuales de acceso de sus propios sistemas. Un adaptador aislado permite descubrir la interfaz real en ejecución sin propagar el hallazgo al núcleo. Es también la mitigación del riesgo mayor de la propuesta: si un fabricante de telemetría no cede acceso programático, **cambia la fuente del dato y no el compromiso** (SUP-10, ADR-10).

**Marco de referencia.** El Capítulo 12 del caso y el RT-05.23 fijan la interoperabilidad industrial sobre **OPC UA**, **ISA-95** y los **estándares abiertos de intercambio de datos de equipos mineros**. ISA-95 ordena la relación entre el nivel de operación y el de gestión, y es el marco que separa lo que se lee del historiador —nivel de control, sólo lectura, C-14— de lo que se intercambia con el ERP —nivel de gestión, C-12— y de lo que ocurre en el nivel de ejecución, donde vive C-01. La huella de carbono se calcula conforme al **protocolo de gases de efecto invernadero**, alcances 1, 2 y 3 (RN-26), con verificación por tercero, que el caso exige al comprador a partir de 2029.

### 10.2 Entre componentes internos

**Síncronas, sólo donde la latencia lo exige** y el consumidor no puede continuar sin respuesta. Son tres, y ninguna más:

| Consumidor | Proveedor | Para qué | Umbral |
|---|---|---|---|
| Portería, superficie de C-05 | Caché local de credenciales del nodo de borde (C-04) | Validar habilitación vigente sin depender del enlace | ≤ 1,5 segundos (RN-20), sostenido también durante un corte |
| Cabina (C-03) | Caché local de credenciales (C-04) | Abrir sesión de operador por proximidad | 1 segundo desde la lectura, sin retirar el guante (SUP-29) |
| Vista única (C-16) | C-10 despacho, C-12 ERP, C-13 telemetría | Reunir estado, historial, pautas y posición | Cuadro completo en menos de los 15 a 20 minutos actuales |

**Asíncronas por evento**, que es el caso por omisión. Todo hecho de terreno, toda conformación de lote, toda clasificación de detención y todo cálculo de indicador viajan por el bus.

**La consecuencia de diseño que el CLIENTE puede verificar: ningún componente de negocio consulta la base de datos de otro.** Si necesita un dato, se suscribe a su evento. Ésa es la propiedad que permite desplegar los diez servicios por separado sin coordinar esquemas.

### 10.3 Contratos y autenticación entre sistemas

Servicios síncronos en **OpenAPI 3.1** y flujos por evento en **AsyncAPI 3.0**, generados desde el código y mantenidos actualizados automáticamente (RT-05.16). El requisito fija AsyncAPI 2.6 como piso; se adopta la 3.0 porque la serie 2.x está superada desde diciembre de 2023 y el cambio no tiene costo en esta etapa.

Los adaptadores que consumen servicios expuestos emplean **OAuth 2.1 con credenciales de cliente**; los enlaces máquina a máquina de terreno y la lectura del historiador emplean **autenticación mutua TLS**. Toda integración registra la transacción de entrada y de salida con un **identificador de correlación común**, que es el mismo que la capa 8 usa para la traza distribuida (RT-05.19). Toda operación de escritura expuesta a reintentos es idempotente, con clave declarada por el cliente y ventana de deduplicación documentada (RT-02.06).

---

## 11. Qué pasa cuando se corta el enlace

Esta es la parte de la arquitectura que el caso condiciona más y la que el CLIENTE debería verificar primero.

**El punto de partida.** El enlace de fibra tiene un único proveedor, con tres a seis cortes al año de cuatro a veinte horas. El respaldo satelital sostiene telefonía y correo, **no** la operación transaccional. La red inalámbrica del rajo tiene zonas de sombra permanentes que se degradan con el avance de la explotación. La restricción n.º 2 no admite excepciones.

### 11.1 Qué se ejecuta en el borde

**Cuatro gabinetes de borde**, más la sala técnica de faena: **portería, planta, depósito de relaves y acopio de puerto**. El acopio de puerto es el más expuesto: está en recinto de terceros y la conectividad la provee un operador **sin acuerdo de nivel de servicio** con la compañía (SUP-19), de modo que se diseña para no depender de él.

Cada dispositivo de terreno y cada nodo de borde mantiene: registro local firmado y cifrado a nivel de campo; reloj monótono con sello de tiempo propio y registro de deriva; identificador de evento generado localmente; cola de sincronización idempotente y reintentable; caché de credenciales y habilitaciones vigentes.

### 11.2 El umbral comprometido

**48 horas continuas** de operación y registro sin enlace, y sincronización posterior en **no más de 4 horas sin intervención manual**. El umbral excede el mínimo transversal del RT-03.10, que fija 24 horas o el mayor que fije el caso.

**Cómo se verifica:** corte controlado de 48 horas **con un cambio de turno dentro de la ventana**. Los eventos reconciliados deben coincidir exactamente con los generados. Es el criterio n.º 13 y se verifica desde la marcha blanca.

### 11.3 Qué opera íntegro sin enlace

Registro del ciclo mina —carguío, transporte, descarga y pesaje—; identificación del operador por proximidad y apertura y cierre de sesión en el relevo; validación de acceso en portería contra caché; registro de inspecciones, incidentes y detenciones; toma de muestra y su registro; movimiento de stock.

### 11.4 Qué **no** opera sin enlace, y qué lo suple

El RT-03.13 exige declarar qué funciones no operan durante un corte y qué procedimiento manual las suple, y **califica la ausencia de esta declaración como observación grave**.

| Función no disponible | Por qué | Procedimiento que la suple |
|---|---|---|
| Acreditación de una persona contratista **nueva** | Requiere validación documental contra el maestro central y contra el ERP | Se suspende el alta. **La persona ya acreditada sí ingresa**: la portería valida contra la caché local |
| Alta o modificación de datos maestros: personas, polígonos, puntos de muestreo, taxonomía, parámetros | El registro central prevalece para datos maestros (RN-30). Permitir alta local crearía dos maestros | Se posterga. Los eventos que referencian un maestro inexistente quedan en cuarentena y se resuelven al reconciliar |
| Conciliación metalúrgica y cierre del mes | Requiere las tres mediciones y el resultado de laboratorio consolidados | Se posterga. La ventana de cierre ya está protegida por RN-24 |
| **Liberación de un lote a embarque** | Exige análisis válido y vigente publicado por el laboratorio, que es sistema externo | **No se libera ningún lote.** Regla dura de RN-11, sin excepción manual |
| Publicación al portal del comprador y a la autoridad | Son destinos externos | Se posterga. Ningún reporte se construye en planilla al margen (RN-25) |
| Tableros analíticos y recálculo de indicadores | La capa analítica está separada de la transaccional por diseño | El supervisor opera con el estado local del turno. Se recalculan al reconciliar |
| Alertas anticipadas de vencimiento de habilitación | Requiere canal de notificación externo | La portería sigue advirtiendo en el ingreso contra la caché local; la alerta anticipada se emite al restablecerse |
| Ingesta de telemetría de fabricante | Depende de tres portales web externos | La detención se sigue detectando desde las señales del despacho que el nodo de borde recibe localmente |

### 11.5 Degradación elegante

Ante la indisponibilidad de un componente no crítico el sistema continúa en modo reducido e **informa la degradación a la persona usuaria**; nunca falla de forma total (RT-02.09). La aplicación de terreno muestra de manera permanente su estado de enlace, las horas acumuladas sin sincronizar y la cantidad de eventos pendientes.

Es requisito de usabilidad y también de confianza: **el operador que no sabe si su registro se guardó deja de registrar**, y ahí se pierde el criterio n.º 14.

---

## 12. Carga, escalado y degradación

### 12.1 La volumetría que dimensiona la arquitectura

| Dimensión | Valor | Fuente |
|---|---|---|
| **Viajes de camión de extracción** | **~342.000 al año**, proyección de 367.000 a tres años | Numeral 14.1 |
| **Ciclos de carguío de pala** | **~1,37 millones al año**, proyección de 1,47 millones | Numeral 14.1 |
| Movimiento total mina | 82 Mt al año, proyección de 88 Mt | Numeral 14.1 |
| Mineral enviado a planta | 21,5 Mt al año | Numerales 2.2 y 14.1 |
| Concentradora | 59.000 t al día, capacidad nominal | Numeral 2.2 |
| Flota | 38 camiones de 240 t de tres generaciones y dos fabricantes; 5 palas eléctricas; 3 cargadores frontales; 9 perforadoras; 62 equipos de apoyo; 310 vehículos livianos | Numeral 2.3 |
| Equipos con telemetría de fabricante | 52 hoy, 70 proyectados, en tres portales distintos | Numeral 14.1 |
| Personas | 1.780 propias y ~2.400 contratistas de ~90 empresas; 4.180 con acceso en régimen; hasta 8.500 en detención mayor de planta | Numerales 2.4 y 14.1 |
| Ingresos y salidas de portería | ~2.900 diarios | Numeral 14.1 |
| Muestras analizadas | ~71.000 al año | Numeral 14.1 |
| Pozos de perforación | ~46.000 al año; ~340 tronaduras | Numeral 14.1 |
| Viajes de concentrado a puerto | ~12.400 al año | Numeral 14.1 |
| Embarques | 31 al año, de ~12.000 t cada uno | Numeral 2.2 |
| Sitios operacionales | 7, incluido acopio de 25.000 t en recinto portuario de terceros | Numeral 14.1 |
| Operación | 24 horas, 365 días, dos turnos de 12 horas, régimen 7x7 | Numeral 2.4, restricción n.º 1 |

Esta volumetría es la que el RT-09.02 manda sostener manteniendo los umbrales del numeral 9.1, y la que alimenta el cálculo de capacidad del RT-09.01.

**Las dos primeras filas son las que dimensionan el agregado raíz.** Trescientos cuarenta y dos mil viajes al año son unos 940 diarios, y son el hecho que la Etapa 1 debe capturar íntegro. Los 12.400 viajes de concentrado a puerto son un flujo distinto y mucho menor: confundirlos subdimensiona el sistema en un factor de veintisiete.

**Crecimiento proyectado y compromiso de capacidad 3x sin rediseño (RT-09.03).** El requisito transversal RT-09.03 exige comprometer que la arquitectura admita un crecimiento de al menos tres veces la volumetría inicial, en un horizonte de tres años, sin requerir rediseño arquitectónico. Aunque la proyección de extracción propia del rajo indica un crecimiento orgánico moderado de +7 % (342.000 a 367.000 viajes anuales), la arquitectura está dimensionada, parametrizada y evaluada para sostener **3x la volumetría inicial**:
- **Régimen:** escalamiento elástico de 40 a **120 transacciones por segundo**.
- **Peak:** absorción de 200 a **600 transacciones por segundo** durante cambios de turno masivos.
- **Carga anual:** procesamiento garantizado de hasta **1,02 millones de viajes** y **4,1 millones de ciclos de carguío** anuales.
Este factor de escala se absorbe de forma puramente elástica mediante el escalado horizontal de contenedores en Capa 4 y Capa 5 (RT-02.10, RT-09.04) y el particionamiento de bases de datos en Capa 6, sin alterar contratos de interfaz, esquemas de eventos ni fronteras de contexto.

### 12.2 Cómo escala

Las capas de aplicación e integración escalan de forma **horizontal y automática** (RT-02.10, RT-09.04). La decisión propia de este caso es *cómo se dispara* ese escalado.

**El peak dimensionante no es aleatorio: es el cambio de turno.** Ocurre dos veces al día, a las 08:00 y a las 20:00 conforme a RN-23, dura de 45 a 60 minutos y alcanza **200 transacciones por segundo contra 40 en régimen** (SUP-31). Un escalado puramente reactivo llega tarde por construcción: reacciona cuando la métrica ya subió, y para cuando la capacidad está disponible el peak lleva un tercio transcurrido, justo en la franja en que se cierra y se abre una sesión de operador por cada equipo de la flota.

**Decisión (ADR-14): escalado anticipado por calendario como mecanismo primario, y escalado reactivo como red de seguridad.** La capacidad se provisiona antes de cada cambio de turno, anclada a la definición de turno de RN-23 —que es un parámetro versionado de C-20, no una constante— y el escalado reactivo cubre lo que el calendario no prevé: una detención programada de planta, una reconciliación masiva tras un corte, un embarque.

**Sin pérdida de transacciones en curso.** Los servicios no guardan estado en memoria (RT-02.05), de modo que retirar una instancia no pierde una sesión; y toda escritura reintentable es idempotente (RT-02.06), de modo que un reintento tras el retiro no duplica el hecho.

### 12.3 Cómo degrada cuando se supera la capacidad

Al superarse la capacidad la solución **encola, limita la tasa y lo informa explícitamente; nunca devuelve un error genérico ni pierde una transacción en silencio** (RT-09.08). El comportamiento se diferencia por origen:

| Origen | Comportamiento bajo saturación |
|---|---|
| **Terreno (C-03, C-04)** | **Nunca recibe rechazo.** El evento queda en la cola local firmada, la misma que sostiene las 48 horas sin enlace. Para la persona operadora, saturación y corte de enlace son el mismo comportamiento, y no tiene que distinguirlos |
| Portales externos (C-06, C-07) | Límite de tasa por consumidor con respuesta explícita y tiempo de reintento sugerido. Es la primera carga que se sacrifica |
| Analítica y reportería (C-19) | Encolamiento de la consulta con aviso de espera; las exportaciones grandes ya son asíncronas (RT-16.29) |
| Integraciones (C-09 a C-14) | Cola persistente por adaptador con reintento y retroceso exponencial; el cortacircuito abre antes de propagar la saturación al núcleo |

**El orden de sacrificio es una decisión, no una consecuencia:** primero el portal del comprador, después la analítica, después las integraciones, y **el registro de terreno nunca**.

### 12.4 Resiliencia

Reintento con retroceso exponencial y variación aleatoria; cortacircuitos por adaptador externo; mamparos de aislamiento entre el tráfico de portales y el operacional; límite de tasa; y **tiempo de espera explícito en toda llamada remota, sin excepción** (RT-02.08).

### 12.5 Los cuatro puntos únicos de falla que subsisten

El RT-02.11 exige declararlos y justificar por qué son aceptables, y advierte que **omitir la declaración cuando existen se evalúa como observación grave**. Subsisten cuatro. **Ninguno está en la ruta del registro de terreno.**

| Punto único de falla | Por qué subsiste | Por qué es aceptable | Mitigación |
|---|---|---|---|
| Enlace de fibra de un único proveedor entre faena y Antofagasta | La construcción de la red no está en el alcance; el respaldo satelital no sostiene tráfico transaccional | **La arquitectura no depende de él.** Es la razón de ser del diseño desconectado: 48 horas de autonomía cubren el peor corte histórico registrado, de 20 horas | Nodos de borde en cuatro sitios; sincronización idempotente |
| Sistema de gestión de laboratorio del CLIENTE | Es sistema del CLIENTE, no se reemplaza | Su caída no detiene la operación: detiene la **liberación** de lotes, que es el comportamiento correcto y exigido por RN-11 | Cortacircuito en C-09; cola de solicitudes con reintento |
| ERP corporativo como emisor contable único | Restricción n.º 4: no se crea una segunda verdad contable | Su indisponibilidad posterga la emisión contable, no el registro operacional | Cola persistente en C-12; ajuste de período anterior previsto por RN-05 |
| Balanza de alimentación de planta como fuente de verdad del tonelaje | Es una decisión de negocio deliberada (SUP-02), no una limitación técnica | Las otras dos mediciones se siguen registrando durante su indisponibilidad; la conciliación se recalcula al recuperarla | Báscula de camiones como punto de calibración; las tres mediciones se conservan siempre |

**Objetivos de recuperación (SUP-32):** para los componentes de terreno, RTO ≤ 1 hora y **RPO de cero eventos perdidos** para el registro operacional.

---

### 12.6 Qué crecimiento admite el diseño y qué componente se satura primero (RT-09.03, RT-09.05, numeral 17.4 punto 10)

El numeral 17.4 punto 10 del caso plantea dos preguntas: **qué crecimiento admite el diseño** y **qué componente se satura primero**. El RT-09.03 fija la primera y el RT-09.05 la segunda.

**Qué crecimiento admite el diseño (RT-09.03).** El requisito exige sostener, sin rediseño, **al menos tres veces la volumetría inicial** en un horizonte de tres años. La proyección que el propio caso entrega en el numeral 14.1 es mucho menor —de 342.000 a 367.000 viajes al año, un 7 %—, de modo que el margen exigido no lo consume el crecimiento vegetativo de la faena sino tres escenarios que sí lo consumirían: la incorporación del octavo sitio operacional, la extensión a una segunda operación conforme al apartado 15.7, y el aumento de equipos con telemetría de 52 a 70. Las tres capas que absorben ese crecimiento —presentación, servicios de negocio e integración— escalan horizontalmente y sin estado, de modo que el factor de tres se resuelve con capacidad y no con rediseño. Lo que **no** escala por adición de capacidad es la capa 6, y por eso la separación transaccional / analítica de ADR-08 y el repositorio de consulta histórica están decididos desde el diseño y no dejados para después.

**Qué componente se satura primero (RT-09.05).** No es el registro de terreno, que por construcción absorbe la contrapresión en la cola local del apartado 12.3. Son dos, cada uno en su capa:

| Capa | Componente | Momento en que se satura |
|---|---|---|
| 5 · Integración y eventos | El **bus de eventos** y los consumidores de ingesta, sostenidos por la plataforma base **C-22** | La **ventana de reconciliación posterior a un corte**, si el enlace se restablece dentro del cambio de turno: coinciden las 200 transacciones por segundo del peak en vivo con la descarga en paralelo de la cola acumulada de hasta cuatro sitios |
| 4 · Servicios de negocio | El **motor de conciliación C-08** | La corrida del balance del mes, por el cálculo iterativo sobre el vector de elementos de RN-14 —cobre, oro, plata y arsénico— a lo largo de toda la cadena de lotes |

**Volumen de la cola de reconciliación (SUP-40).** El numeral 14.2 del caso pide esta cifra con su método, y el método es prorratear la volumetría del numeral 14.1 a la ventana de 48 horas:

| Hecho | Base anual del numeral 14.1 | En 48 horas | Eventos que genera |
|---|---|---|---|
| Viajes de camión de extracción | 342.000 | 1.874 | 3.748, con su pesaje asociado |
| Ciclos de carguío de pala | 1,37 millones | 7.507 | 7.507 |
| Ingresos y salidas de portería | ~2.900 diarios | 5.800 | 5.800 |
| Sesiones de operador | ~120 equipos, 2 relevos al día | 480 | 960, apertura y cierre |
| Muestras | ~71.000 | 389 | 389 |
| Pozos de perforación | ~46.000 | 252 | 252 |
| Inspecciones e incidentes | ~19.300 | 106 | 106 |
| | | **Subtotal** | **~18.800** |

Sumados los agregados de telemetría y de detención que el borde compacta antes de sincronizar, y un margen de diseño de 1,8 veces sobre el estimado, **el techo de diseño de la cola se declara en 35.000 eventos por sitio**. Si el supuesto resulta bajo, el impacto es tiempo de drenaje y no pérdida: la cola local es persistente y firmada, y el plazo de 4 horas del criterio n.º 13 tiene el margen del apartado 11.2. Se valida con el corte controlado de la marcha blanca.

**Cómo se detecta.** Por los mismos mecanismos del apartado 14, no por umbrales de infraestructura aislados:

- **Retraso de reconciliación pendiente por sitio**, que ya es el indicador comprometido del criterio n.º 13: alarma cuando el tiempo de drenaje proyectado pone en riesgo el plazo de 4 horas.
- **Tiempo de ingesta extremo a extremo** contra los umbrales del numeral 9.1 transversal —500 ms para una consulta simple, 800 ms para una escritura transaccional— y contra los umbrales de terreno que el Capítulo 15 fija en RT-09.01, siempre medidos en el percentil 95 y sobre la experiencia real.
- **Tiempo de ejecución de la conciliación** contra su ventana comprometida de 3 días hábiles, con alerta cuando la proyección la excede.
- Saturación sostenida de recursos en las unidades de proceso del bus y del nodo de borde, como señal de apoyo y no como disparador principal.

**Cómo se resuelve, sin rediseño.**

- **Escalado horizontal automático** de los consumidores del bus, con tiempo de reacción declarado y verificado en las pruebas de carga del Subdocumento 9 (RT-09.04).
- **Partición del flujo por sitio y por equipo**, de modo que la descarga masiva de un sitio no bloquee la de los demás.
- **Control de flujo en el nodo de borde C-04**, que regula la tasa de subida de la cola acumulada y **da prioridad al evento en vivo sobre el evento histórico reconciliado**. Es la aplicación concreta del orden de sacrificio del apartado 12.3.
- **Aislamiento de la conciliación** sobre el almacén analítico separado, sin competir por recursos con la operación (RT-05.05).

---

## 13. Seguridad

Este apartado decide el **modelo, las fronteras y los controles lógicos**. El producto, la versión y el emplazamiento de cada control son materia del Subdocumento 4.2. El programa de gestión de vulnerabilidades, el centro de operaciones de seguridad y la respuesta a incidentes son actividades de la Operación, en los Subdocumentos 10 y 11.

### 13.1 El modelo, y su única excepción

La arquitectura adopta **Zero Trust conforme a NIST SP 800-207** (RT-11.01): verificación explícita de cada solicitud, privilegio mínimo y presunción de compromiso. **Ninguna posición de red concede confianza.** Que una petición provenga de la red de faena no la autoriza, y por eso los siete adaptadores externos autentican como cualquier otro consumidor y ningún componente de datos es alcanzable desde Internet (RT-03.04).

El modelo tiene **una excepción, y la impone el caso**. Durante las 48 horas sin enlace el punto de decisión de política central es inalcanzable, y verificar cada solicitud contra él detendría la mina —exactamente lo que prohíben las restricciones n.º 1 y n.º 2. La excepción se resuelve **delegando el punto de decisión, no suprimiéndolo** (ADR-12):

| Elemento del modelo | Con enlace | En el enclave de terreno sin enlace |
|---|---|---|
| Punto de decisión de política | Servicio central de autorización sobre C-05 | Delegado al nodo de borde C-04, con política **firmada por C-05** y vigencia declarada |
| Verificación de identidad | Contra el maestro de personas de C-05 | Contra la caché local de credenciales y habilitaciones, firmada y fechada en origen |
| Privilegio mínimo | Rol, área y sitio (RF-IDE-018) | El mismo conjunto de roles, congelado en la caché: **en terreno no se conceden privilegios nuevos** |
| Presunción de compromiso | Sesión corta y reautenticación | El evento viaja firmado con el identificador del dispositivo inventariado, y **la aserción de identidad con que se admitió viaja dentro del evento** |
| Momento de la verificación | En la solicitud | En la solicitud contra la caché, **y otra vez en la reconciliación** contra el maestro central |

**La propiedad que hace aceptable la excepción es que la verificación no se omite: se difiere y se repite.** Un evento admitido en terreno contra una credencial que el maestro central había revocado se reconcilia igualmente —el hecho ocurrió y RN-30 manda conservarlo— pero queda marcado, alertado y trazable hasta la aserción con que se admitió.

**Limitación residual, declarada.** La caché de credenciales tiene vigencia máxima de **72 horas**, con margen sobre las 48 comprometidas, y se refresca en cada sincronización. Durante un corte prolongado una revocación no se propaga, de modo que el umbral de quince minutos de SUP-28 se mide sobre operación con enlace. Los controles compensatorios son tres: la persona cuya habilitación se revoca ya está físicamente dentro del recinto y el control de portería lo sabe; el nodo de borde deja de admitir la credencial al vencer la caché aunque no haya recuperado el enlace; y todo evento admitido en la ventana queda marcado en la reconciliación. Queda incorporado al Registro de supuestos v1.5 como **SUP-38**.

### 13.2 Clasificación de la información

Cuatro niveles (RT-11.03). **La clasificación es atributo del dato y viaja con él**: la hereda el evento, el índice de búsqueda, la exportación y el respaldo.

| Nivel | Qué comprende en este caso | Controles diferenciados |
|---|---|---|
| **Público** | Ninguna información de la solución. No existe portal abierto a la ciudadanía | No aplica. **Que este nivel esté vacío es una decisión de diseño, no un olvido** |
| **Interno** | Registro operacional: viajes, pesajes, detenciones, inspecciones, estados de equipo, indicadores | Cifrado en tránsito y en reposo; control de acceso por rol, área y sitio; auditoría de modificación |
| **Confidencial · competitivamente sensible** | **Ley por polígono, modelo de bloques, plan minero versionado, liquidaciones, penalizaciones y márgenes** | Lo anterior, más: prohibición de exposición al exterior en cualquier forma derivada (RN-16); aprobación del área comercial para toda publicación; registro de **toda consulta**; exportación restringida por rol y trazada |
| **Personal sensible · Ley N.º 21.719** | Habilitación, exámenes preocupacionales, salud ocupacional, control de fatiga, control de alcohol y drogas | Lo anterior, más: **cifrado a nivel de campo** (RT-11.10); auditoría de consulta (RN-29); prohibición de aparecer en registros; prohibición de uso en ambientes no productivos; retención de 10 años sin borrado anticipado |

**La consecuencia arquitectónica** es que el nivel confidencial y el personal sensible **no comparten el mismo control, y por eso no comparten componente**: la ley por polígono vive en el contexto Mineral y se filtra en la vista de exposición hacia el Comercial; el dato personal sensible vive en Personas y en Cumplimiento, cifrado a nivel de campo. Un diseño que los tratara con un solo control sobreprotegería el primero y subprotegería el segundo (ADR-13).

### 13.3 Superficie de exposición: qué es alcanzable desde fuera

**Toda publicación de servicios se realiza exclusivamente por la capa de borde** (RT-11.07). La afirmación es verificable, porque la superficie alcanzable desde fuera de la red del CLIENTE es ésta y ninguna otra. Las dos puertas de enlace aplican sobre ella autenticación, autorización, cuotas, límite de tasa, validación de esquema e inspección de carga útil (RT-11.11), con los parámetros que distingue el apartado 5, capa 3. La enumeración concreta de nombres de dominio, puertos y servicios que el RT-11.13 exige se deriva de esta tabla en el Subdocumento 4.2:

| Qué se expone | A quién | Autenticación | Control de borde |
|---|---|---|---|
| Portal de empresas contratistas (C-06) | ~90 empresas contratistas | Credencial de empresa con doble factor, administración delegada | CDN, cortafuegos de aplicaciones web con reglas gestionadas **y personalizadas**, protección DDoS en capas 3, 4 y 7, TLS 1.3 |
| Portal de trazabilidad de lote (C-07) | Compradores, en Etapa 2 | Credencial nominada por comprador, con registro de toda consulta | Lo mismo, más límite de tasa y cuota por consumidor |
| Puerta de enlace de exposición | Integraciones autorizadas | OAuth 2.1 con credenciales de cliente o autenticación mutua TLS | Validación de esquema e inspección de carga útil antes de alcanzar cualquier servicio |
| Acceso acotado de laboratorios externos y del operador del terminal portuario, servido por `C-09` | Las dos poblaciones que el RT-12.12 declara además de contratistas y compradores | Credencial nominada, con registro de toda consulta | Lo mismo que los portales, con visibilidad limitada a la información que su función requiere |

**Nada más.** En particular: la puerta operacional **no** es alcanzable desde Internet; ningún componente de datos lo es; y el registro de un viaje **no atraviesa el borde público en ningún punto de su recorrido**.

**Protección contra bots (RT-11.12).** Reto progresivo escalonado por señal de riesgo y **nunca por defecto**: primero límite de tasa por credencial, luego reto interactivo, y sólo después bloqueo. La razón del escalonamiento es del caso: quien usa el portal de contratistas es personal administrativo de empresas pequeñas, con conectividad y dispositivos desiguales, y un reto agresivo se traduce en acreditaciones que no se completan y en el criterio n.º 8 incumplido.

### 13.4 Cifrado y custodia de claves

**En tránsito (RT-11.08):** TLS 1.3, con prohibición expresa de TLS 1.0 y 1.1 y de conjuntos de cifrado obsoletos; HSTS con precarga en los dos portales; gestión automatizada de certificados con rotación y alerta anticipada de vencimiento.

**En reposo (RT-11.09):** la totalidad de los datos en reposo está cifrada, en las seis naturalezas de persistencia **y también en el registro local del dispositivo de terreno y del nodo de borde**, que es donde el dato pasa hasta 48 horas fuera del centro de datos y es el punto más expuesto a la pérdida física del soporte.

**Custodia de claves:** servicio de gestión de claves o módulo de seguridad de hardware, con política de rotación declarada y **separación de funciones**: quien administra la plataforma no custodia las claves del nivel personal sensible, y ninguna persona reúne por sí sola la capacidad de descifrar un campo de salud ocupacional. Las claves de firma del registro local residen en el almacén respaldado por hardware del propio dispositivo y **no salen de él**: el nodo de borde verifica la firma, no la produce.

### 13.5 Amenazas principales y su control

Metodología declarada: **STRIDE** (RT-11.02), aplicado a los veintidós componentes y a las siete integraciones externas, con revisión obligatoria ante todo cambio que altere un límite de contexto, una interfaz o una clasificación. El modelo es entregable contractual y se versiona junto con los ADR del apartado 21. Las de mayor consecuencia en este caso:

| Amenaza | Componentes | Control lógico |
|---|---|---|
| Préstamo o uso indebido de credencial de proximidad entre personas del mismo turno | C-05, C-03 | Detección por patrones sobre eventos de acceso ya registrados (RF-IDE-012); credencial ligada a dispositivo inventariado |
| Alteración del registro local en un dispositivo extraviado o intervenido | C-03, C-04 | Registro firmado con clave del almacén de hardware; identificador de evento generado en origen; **corrección como evento nuevo, nunca modificación** |
| Registro sin persona identificada, o corrección posterior sin autor | C-01, C-20 | Marca «operador no identificado» como indicador gestionable (RN-22); asiento de auditoría inalterable con autor, fecha y justificación |
| **Fuga de la ley por polígono** al comprador o a un tercero, por portal, exportación o informe | **C-07, C-19, C-01** | **La vista de exposición filtra en el servicio y no en la pantalla; toda exportación y búsqueda respetan el control de acceso; la publicación requiere aprobación comercial y toda consulta queda registrada** |
| Acceso indebido a habilitación, salud ocupacional o fatiga | C-05, C-18 | Cifrado a nivel de campo y auditoría **de la consulta** además de la modificación |
| Ataque de denegación de servicio al portal que arrastre al núcleo | C-07, C-22 | Mamparo de recursos con umbral de cero degradación de los umbrales de terreno ante diez veces el tráfico medio del portal |
| Saturación del enlace al restablecerse, por la cola de cuatro sitios | C-04, C-22 | Calidad de servicio en la puerta operacional; reconciliación por lotes con límite de tasa |
| Uso de cuenta de administración para alterar un parámetro con impacto operacional | C-20 | Aprobación de un segundo perfil (RT-16.03); elevación temporal con sesión grabada; doble factor resistente a suplantación |
| **Escritura en la red de control de proceso** | C-14 | La restricción n.º 3 es absoluta: el adaptador **no implementa ninguna operación de escritura**. No es una regla de configuración: es una **capacidad ausente del componente** |

### 13.6 Casos de detección propios de este caso

Los eventos de seguridad se registran de forma **centralizada e inalterable**, con retención de **doce meses en línea y veinticuatro meses adicionales en archivo recuperable** (RT-11.14). El nodo de borde acumula localmente sus eventos de seguridad mientras está sin enlace y los reconcilia con el mismo mecanismo que los de negocio. Sobre ese registro, el RT-11.15 exige casos de uso de detección definidos para el proceso de negocio y no sólo genéricos de infraestructura:

- Credencial usada en dos sitios en un intervalo físicamente imposible, cruzando eventos de portería y de apertura de sesión en cabina con la distancia entre sitios. Es la forma concreta que toma el préstamo de credencial en una faena de siete sitios.
- Exportación o consulta masiva de trazabilidad por un perfil comercial. **Es la vía por la que la ley por polígono saldría sin romper ningún control de acceso.**
- Consulta de dato personal sensible fuera del ámbito del rol o fuera de horario.
- Cambio de parámetro sin la aprobación del segundo perfil, o dentro de ventana prohibida. Un cambio de umbral durante el cierre contable altera cifras ya declaradas al ERP.
- Evento de terreno cuyo sello de tiempo es inconsistente con el registro de deriva de reloj de su dispositivo. Es la señal de manipulación del registro local, y la única disponible mientras el dispositivo estuvo aislado.
- Cualquier intento de escritura hacia la red de control de proceso. **La detección existe aunque la capacidad de escritura no esté implementada, precisamente para evidenciar que no se usa.**
- Acceso al recinto técnico fuera de ventana de intervención declarada.

### 13.7 Acceso a producción y ambientes no productivos

**Las personas desarrolladoras no tienen acceso interactivo directo a producción** (RT-11.27). Todo acceso excepcional es temporal, aprobado previamente, registrado y con sesión grabada. La consecuencia de diseño es que **el diagnóstico de un incidente debe poder hacerse desde la observabilidad y no desde la base de datos**: si para entender una falla hay que entrar a producción, el diseño de observabilidad falló.

Queda prohibido el uso de datos productivos reales en ambientes no productivos sin anonimización verificable (RT-11.25). En este caso la prohibición muerde en dos lugares:

- **El dato personal sensible no se copia nunca**, ni siquiera seudonimizado. Las pruebas usan datos sintéticos, porque la seudonimización de una población de 4.180 personas con habilitaciones, exámenes y resultados de fatiga es reversible por cruce.
- **La ley por polígono se ofusca** en todo ambiente no productivo, conservando la distribución estadística pero no los valores reales.

### 13.8 Resumen verificable de la vista de seguridad

| Frente | Decisión |
|---|---|
| Modelo | Zero Trust NIST SP 800-207, con la excepción acotada del enclave de terreno |
| Identidad | Tres mecanismos sobre un maestro único |
| Segundo factor | Obligatorio para administradoras, todo acceso privilegiado y todo acceso desde fuera de la red corporativa (RT-12.03), con factor resistente a suplantación tipo FIDO2 o clave de acceso en los perfiles administradores (RT-12.04). El inicio de sesión único con cierre propagado, la política de sesión y el procedimiento de acceso de emergencia rigen para los componentes de oficina (RT-12.02, RT-12.07). En terreno el segundo factor es la posesión física de la credencial sobre dispositivo inventariado |
| Autorización | Mínimo privilegio por rol, área y sitio. La búsqueda global y toda exportación respetan el control de acceso |
| Segregación TI/TO | Lectura del historiador por OPC UA conforme a IEC 62443. **No se escribe en la red de control** |
| Dato personal sensible | Cifrado a nivel de campo; se audita la consulta además de la modificación |
| Auditoría | Asiento inalterable de toda modificación y de toda consulta sensible. Retención por tipo de registro sin borrado anticipado (RN-28) |
| Aislamiento del portal externo | Mamparo de recursos y caché; cero degradación de los umbrales de terreno ante diez veces el tráfico medio del portal |
| Recinto técnico | Control de acceso biométrico facial con AFIS de respaldo, obligatorio por RT-06.20. **No confundir** con la biometría de portería, que está excluida del Contrato |

---

## 14. Observabilidad y niveles de servicio

### 14.1 Un solo identificador para todo

Una sola plataforma para nube y faena, con alertamiento unificado y **sin puntos ciegos**, instrumentada conforme a **OpenTelemetry** (RT-14.01, RT-03.16). Métricas, registros y trazas se correlacionan por un **identificador único de transacción**, que es el mismo identificador de correlación que el RT-05.19 exige para las integraciones. **No son dos identificadores distintos**, y ésa es la decisión que permite seguir una operación de negocio a través de la solución y de los sistemas del CLIENTE con un solo hilo (ADR-17).

El identificador **nace en el dispositivo de terreno**, no en el borde de entrada. La razón es la misma que sostiene ADR-05: un identificador asignado al recibir no puede correlacionar lo que ocurrió durante las 48 horas en que no hubo recepción.

### 14.2 Qué se mide, y dónde

Los indicadores de nivel de servicio se miden **sobre la experiencia real de la persona usuaria y no sobre pruebas sintéticas** (RT-14.03). En este caso la distinción no es doctrinaria: **una prueba sintética lanzada desde el centro de datos no atraviesa la red inalámbrica del rajo**, que es donde están las zonas de sombra, y por lo tanto mediría un sistema que nadie usa.

| Indicador | Umbral | Dónde se mide |
|---|---|---|
| Registro de un movimiento de material desde el equipo en terreno | **2 segundos en el percentil 95** | Desde la acción de la persona hasta la confirmación en pantalla, instrumentado en el dispositivo |
| Registro en báscula | 3 segundos | En el punto de pesaje |
| Consulta de habilitación en portería | 1,5 segundos | En el torniquete, contra la caché local del nodo de borde |
| Apertura de sesión de operador por proximidad | 1 segundo desde la lectura, sin retirar el guante | En cabina |
| Disponibilidad de un indicador de turno en la capa analítica | 15 minutos | Desde la ocurrencia de la transacción |

Las pruebas sintéticas se emplean **como complemento**, con un propósito acotado: verificar desde cada uno de los siete sitios que la ruta está viva cuando no hay actividad de personas, de modo que un corte nocturno no se descubra al inicio del turno de día.

### 14.3 Sobre qué se alerta

El alertamiento se basa en **síntomas de negocio y no sólo en umbrales de infraestructura** (RT-14.04). La regla que ordena la lista: **se alerta sobre lo que compromete un criterio de aceptación**, no sobre lo que un tablero de infraestructura consideraría anómalo.

| Síntoma de negocio | Por qué es la señal correcta | Criterio |
|---|---|---|
| Viajes registrados por hora bajo la línea base del turno, por sitio | Un nodo de borde puede estar sano y aun así no recibir registros, porque la aplicación de terreno dejó de usarse. **Es la única alerta que detecta el modo de falla que hundió las dos iniciativas anteriores** | n.º 14 |
| Horas acumuladas sin enlace y retraso de reconciliación, por sitio | Es la evidencia directa del compromiso de 48 horas y de la sincronización en 4 horas | n.º 13 |
| Detenciones sin clasificar acercándose al cierre del turno | Anticipa el escalamiento de RN-18 en vez de constatarlo | n.º 5 |
| Lotes retenidos por análisis vencido o no publicado | Si el laboratorio no responde, el lote no sale, y eso debe verse antes de la ventana de embarque | n.º 1, n.º 2 |
| **Arsénico proyectado sobre el límite antes de conformar el lote definitivo** | Es el control que ataca las cuatro penalizaciones de USD 1,9 millones. La alerta debe llegar **antes** de la conformación | n.º 1 |
| Habilitaciones por vencer sin acción registrada, por empresa contratista | Evita el rechazo en portería y se dirige a quien puede resolverlo: la empresa empleadora | n.º 7, n.º 8 |
| Eventos en cuarentena por referenciar un dato maestro inexistente | Es el residuo previsto del apartado 11.4 y crece silenciosamente si nadie lo mira | n.º 13 |
| Diferencia de conciliación proyectada sobre el umbral | Anticipa el cierre en vez de descubrirlo el día 3 | n.º 2, n.º 3 |
| Certificado próximo a vencer en cualquier ruta de integración o de terreno | Su vencimiento detiene una reconciliación completa de sitio | n.º 13 |

La cobertura de guardia se declara contra el régimen real de la faena: la operación es continua 24x7x365 y el cambio de turno de las 08:00 y las 20:00 es el momento de mayor consecuencia, de modo que **la guardia no puede tener su propio relevo en esa franja**.

### 14.4 Qué no puede aparecer en un registro

Los registros de la solución **no contienen datos personales sensibles ni credenciales** (RT-14.07). Aplicado a este caso: un registro puede consignar que se consultó la habilitación de una persona, con su identificador y el resultado de la validación, y **no puede consignar el contenido de esa habilitación** —el examen, su resultado, la condición de salud, el resultado del control de fatiga o de alcohol y drogas.

La consecuencia de diseño es que **el filtrado ocurre en la instrumentación y no en la ingesta**: el dato sensible no sale del componente hacia la plataforma de observabilidad. Un filtro en la ingesta deja el dato en tránsito y en la memoria del recolector, y eso ya es un tratamiento bajo la Ley N.º 21.719.

### 14.5 Acceso del CLIENTE y retención

El CLIENTE dispone de **acceso propio y permanente** a los tableros operacionales y de negocio, con datos en tiempo real y capacidad de exportación, **sin intervención del ADJUDICATARIO** (RT-14.02). No es una concesión: es condición de la transferencia del numeral 17.6 y del hecho de que el equipo de once personas recibe la operación al término del Contrato. El acceso respeta el control por rol, área y sitio, y la exportación respeta la clasificación del apartado 13.2 (RT-16.27): **un tablero no es una vía para sacar la ley por polígono.** La retención del asiento de auditoría la fija el caso por tipo de registro y no el mínimo supletorio de cinco años del RT-16.10.

La retención de cada señal se declara a continuación, y su costo asociado se distingue entre almacenamiento en línea y archivado en la Oferta Económica (RT-14.08).

| Señal | En línea | Archivo recuperable |
|---|---|---|
| Métricas | 13 meses | — |
| Registros de aplicación | 3 meses | 12 meses |
| Registros de eventos de seguridad | 12 meses | 24 meses adicionales |
| Trazas distribuidas | 30 días, con muestreo | 90 días para transacciones que fallaron |
| Asiento de auditoría | Según RN-28 y el valor que el Capítulo 15 fija en RT-05.10, por tipo de registro | Sin borrado anticipado |

Trece meses de métricas permiten comparar un mes contra el mismo mes del año anterior, que es como se lee una operación con estacionalidad de detenciones programadas en marzo y septiembre. El muestreo de trazas **no se aplica a las trazas con error ni a las de terreno**: se conservan íntegras.

### 14.6 Detección proactiva de anomalías (RT-14.09)

Se compromete, y se acota a donde el comportamiento histórico existe y es estable: **tasa de registro por sitio y por turno, tiempo de ciclo del viaje, deriva de reloj por dispositivo y latencia de ingesta por fabricante de telemetría**. La alerta se emite antes de que el incidente afecte la operación.

Se declara también lo que **no** hace: no es analítica predictiva de negocio —eso es el RT-05.30— y no se compromete en este apartado.

---

## 15. Cómo cambia el sistema sin detener la mina

Tres cosas cambian durante la vida del sistema: los parámetros de negocio, la configuración técnica y la versión del software. Las tres están gobernadas por la misma restricción, la n.º 1: **la operación no se detiene**.

### 15.1 Configuración técnica y parámetro de negocio no son lo mismo

Toda configuración está **externalizada del artefacto y gestionada por ambiente** (RT-04.08): el mismo artefacto se promueve de QA a Preproducción y a Producción sin recompilación. Sobre esa base, la decisión propia de este caso es distinguir dos cosas que suelen confundirse en un solo mecanismo.

| | Configuración técnica | Parámetro de negocio |
|---|---|---|
| Qué es | Puntos de conexión, tamaños de lote, tiempos de espera, credenciales de integración | Umbrales, plazos, tolerancias, catálogos, taxonomía de detenciones, factores de emisión, definición de turno |
| Dónde vive | Almacén de configuración por ambiente, fuera del artefacto | **C-20**, con vigencia temporal versionada |
| Quién la cambia | El equipo de plataforma, por el flujo de despliegue | **El CLIENTE**, desde la interfaz de administración (RT-16.02), con aprobación de un segundo perfil si tiene impacto operacional (RT-16.03) |
| Efecto de un cambio | Rige desde el despliegue; no tiene historia | **Rige desde su fecha de vigencia, y un período anterior se recalcula con la versión que regía entonces** (RN-27) |
| Auditoría | Registro de cambio de configuración | Asiento con autor, fecha y justificación, conservado según RN-28 |

**Por qué importa la distinción (ADR-16).** Si un umbral de conciliación fuera configuración técnica, cambiarlo **reescribiría el pasado** y los criterios de aceptación n.º 3 y n.º 4 dejarían de ser verificables sobre historia ya cerrada.

Los secretos no son ninguna de las dos: residen en un gestor de secretos con rotación automática y auditoría de acceso, y **ninguna credencial se embebe en código, imagen ni archivo de configuración**.

### 15.2 Simular un cambio de parámetro antes de aplicarlo

Existe un ambiente de simulación que permite probar el efecto de un cambio de parámetro antes de llevarlo a producción (RT-16.05). En este caso tiene una forma concreta que el requisito genérico no anticipa: **la simulación se ejecuta sobre una copia de historia ya cerrada**, no sobre datos sintéticos, porque lo que se necesita saber antes de cambiar un factor de conciliación o una tolerancia es *qué habría pasado con los meses que ya se cerraron*.

Es la misma capacidad que RN-27 exige para el recálculo retroactivo, usada en sentido prospectivo. La copia respeta la ofuscación de la ley por polígono y excluye el dato personal sensible.

### 15.3 Despliegue sin interrupción

**Estrategia: despliegue canario, con progresión por sitio** y verificación en Preproducción antes de cada paso a producción (RT-04.07, ADR-15).

Se escoge sobre azul-verde por una razón del caso: **los siete sitios no son equivalentes ni fallan igual** —el acopio de puerto está en recinto de terceros con conectividad de un operador sin acuerdo de nivel de servicio— y el canario permite avanzar sitio por sitio y detenerse en el primero que se degrade, mientras que azul-verde conmuta todo a la vez y obliga a revertir todo a la vez.

**La progresión nunca cruza un cambio de turno.** Las ventanas son las del RT-10.05: intervenciones mayores sólo en las detenciones programadas de planta de marzo y septiembre; menores, previa aprobación, entre las 03:00 y las 05:00; **prohibidas durante el cierre contable del mes y durante las 48 horas previas a un embarque** (RN-24).

**Migraciones de esquema (RT-04.10):** versionadas, reversibles y automatizadas, con estrategia de **expansión y contracción**. Primero se agrega la estructura nueva sin retirar la anterior; se despliega la versión que escribe en ambas; se migra el dato; se despliega la versión que lee sólo la nueva; y sólo entonces se retira la anterior, en un despliegue posterior.

En este sistema la propiedad no es opcional por dos razones: la operación no se detiene, y **un nodo de borde que estuvo 48 horas sin enlace puede reconciliar eventos generados por la versión anterior de la aplicación de terreno**, de modo que el esquema debe aceptar durante toda la ventana de convivencia lo que produjo la versión que quedó aislada.

### 15.4 Administración de los dispositivos de borde y de terreno

Inventario, configuración, actualización de firmware y de aplicación, bloqueo y borrado remoto (RT-03.18). **No es accesorio: el inventario es condición de la propia autenticación**, porque RF-IDE-003 admite el toque de la credencial sólo sobre un dispositivo previamente inventariado. Un dispositivo que no está en el inventario no abre sesión.

El bloqueo y el borrado remoto son el control compensatorio de la pérdida física de un dispositivo que contiene hasta 48 horas de registro local cifrado. **La actualización de aplicación se aplaza mientras el dispositivo tenga eventos pendientes de reconciliación: primero se sincroniza, después se actualiza, nunca al revés.**

### 15.5 Qué se procesa en el borde, y qué no

Se compromete procesamiento en el borde (RT-03.19), acotado a lo que reduce volumen **sin trasladar lógica de negocio**, que es la frontera que el apartado 7 declara para el contexto Terreno:

- **Filtrado y agregación previa** de telemetría y de señales del historiador: se transmite el cambio de estado y la agregación por intervalo, no la muestra cruda.
- **Detección local de la detención de un equipo** a partir de las señales que el nodo de borde ya recibe del despacho, para que la detección siga operando sin enlace.
- **Deduplicación y compactación de la cola** antes de sincronizar, de modo que el lote de reconciliación tras 48 horas transporte hechos y no reintentos.

**No se ejecuta en el borde** la clasificación de la detención con taxonomía cerrada, ni el cálculo de la ley, ni el cumplimiento del plan: son lógica de negocio de Activos y de Mineral.

### 15.6 Reversibilidad y bloqueo por proveedor (RT-03.07)

| Grado | Qué comprende | Fundamento |
|---|---|---|
| **Portable sin rediseño** | Los diez servicios de negocio, el nodo de borde C-04 y la aplicación de terreno C-03 | Sin estado, contrato versionado, ejecución en contenedor. **La decisión de grano medio de ADR-01 es lo que los mantiene portables**: una malla de grano fino habría atado la solución a la plataforma que la orquesta |
| **Portable con reescritura acotada** | Bus de eventos, puerta de enlace, plataforma de observabilidad | El contrato es estándar —AsyncAPI 3.0, OpenAPI 3.1, OpenTelemetry— y por eso lo que se reescribe es la integración, no la lógica |
| **Atado al proveedor** | Servicios administrados de persistencia, gestión de claves y análisis | **Atadura deliberada**: el RT-03.05 manda privilegiar servicios administrados cuando reduzcan el riesgo operacional, y once personas de TI no pueden operar bases de datos autoadministradas al término del Contrato. **Se acepta el bloqueo y se declara, en vez de fingir portabilidad** |
| **Independiente por diseño** | Los siete sistemas del CLIENTE | Capa anticorrupción y adaptador reemplazable: sustituir un sistema externo cambia un adaptador y no toca el núcleo |

El dato es exportable en formato abierto en todo momento, que es la condición práctica de la reversibilidad.

---

### 15.7 Replicación a una segunda operación y expansión internacional hacia 2030 (RT-02.12, Capítulo 15)

El Capítulo 15 del caso y el requisito **RT-02.12** exigen que la solución admita replicarse por parametrización, sin rediseño arquitectónico, con **catálogos, unidades organizacionales y reglas de negocio independientes**. El Capítulo 15 fija el destino concreto: la compañía evalúa incorporar una operación en el extranjero hacia 2030, y el numeral 13.2 la registra como hito externo no confirmado. La malla de turnos se agrega a esa lista por decisión del PROPONENTE, porque un régimen distinto del 7x7 rompería la definición de turno de RN-23 sobre la que se calcula todo indicador.

La arquitectura lógica resuelve esta exigencia mediante **particionamiento multi-inquilino (*multi-tenancy*) lógico**:

1. **Aislamiento de datos y eventos por unidad organizacional:** cada evento, transacción, lote y registro histórico lleva el identificador de la operación como atributo obligatorio de primer nivel. Las políticas de acceso de la capa 7 segregan las consultas por operación, de modo que la información competitivamente sensible de una faena no es alcanzable desde la otra. Es la misma frontera del apartado 13.2 aplicada a un eje adicional.
2. **Catálogos y taxonomías independientes en C-20:** las flotas de equipos, los polígonos geolocalizados, las mallas de turno (que pueden diferir del régimen 7x7 chileno), los husos horarios de registro y las monedas de liquidación comercial se configuran como datos maestros parametrizados en C-20, sin tocar una sola línea de código.
3. **Reglas y umbrales específicos por país:** el motor de vigencia temporal de parámetros (RN-27) permite aplicar normas laborales locales, factores de emisión de huella de carbono específicos de la matriz energética de destino y tolerancias de pesaje diferenciadas por país.
4. **Adaptadores desacoplados para integraciones locales:** si la segunda operación usa un ERP o un despacho de flota distintos, se despliega una instancia nueva del adaptador correspondiente en la capa 5, con su propia capa anticorrupción, sin tocar el núcleo de trazabilidad C-01 ni el motor de conciliación C-08. Es la propiedad que el apartado 10.1 declara para los siete adaptadores, usada en otro eje.

### 15.8 Ambientes no productivos y certificaciones sectoriales (RT-15.02)

El código **RT-15.02** es uno de los que el Subdocumento 3 declaró **desalineados** entre el documento transversal y el Capítulo 15 del caso. Se acreditan aquí las dos lecturas por separado, para que ninguna quede sin declarar cualquiera sea la que la Comisión Evaluadora aplique.

**Lectura del documento transversal — los ambientes no productivos se apagan o reducen fuera del horario de uso.** Se compromete. La consecuencia que sí es de arquitectura lógica, y por eso se declara aquí, es la **excepción**: la reducción no se aplica durante las ventanas de prueba de carga sobre Preproducción, ni durante las detenciones programadas de planta de marzo y septiembre, cuando el volumen de acreditación multiplica la carga de validación. El calendario de reducción por ambiente y su ahorro asociado son materia del Subdocumento 4.2.

**Lectura del Capítulo 15 del caso — certificaciones sectoriales del adjudicatario: ISO 45001 e ISO 14001 vigentes, conocimiento acreditado del Reglamento de Seguridad Minera y experiencia comprobable en faena minera en operación.** Es una exigencia sobre el PROPONENTE y no sobre la arquitectura; se acredita en los antecedentes de la propuesta. Se consigna aquí sólo para que el código no quede huérfano entre entregables.

---

## 16. El canal de terreno

El RT-17.02 no admite describir la aplicación de terreno sin declarar su naturaleza, y exige justificarla frente a tres dimensiones: operación desconectada, acceso a periféricos y costo de mantención.

### 16.1 Las cuatro exigencias que acotan la decisión

1. Operar **48 horas continuas sin enlace**, con registro local firmado y cifrado a nivel de campo, reloj monótono, sello de tiempo propio e identificador de evento generado en el dispositivo. No es una caché de conveniencia: es un registro con **integridad criptográfica** que debe sobrevivir al cierre de la aplicación, al reinicio y al agotamiento de batería.
2. Leer la **credencial de proximidad** en cabina, sin retirar el guante, en un segundo.
3. Operar con guantes gruesos y lentes de seguridad, a la intemperie, bajo sol directo y hasta −6 °C, en **dispositivos compartidos por turno**, con personas sin correo corporativo y **sin contraseña alfanumérica en terreno**.
4. Sostener el sistema operativo móvil vigente y **las dos versiones anteriores** (RT-17.03).

A ello se suma la restricción n.º 10: el costo de mantención que el requisito manda ponderar **no es el del ADJUDICATARIO durante la implementación, sino el del CLIENTE durante los 36 meses de Operación y después de ellos**, con once personas.

### 16.2 La comparación

| Dimensión | Web progresiva | Nativa por plataforma | **Híbrida (escogida)** |
|---|---|---|---|
| Operación desconectada 48 h | **Insuficiente.** El almacenamiento del navegador es desalojable por el sistema operativo bajo presión de memoria o de disco, y no ofrece garantía de persistencia para un registro con valor probatorio | Suficiente | **Suficiente.** El contenedor nativo aporta almacenamiento persistente no desalojable y acceso al almacén de claves |
| Acceso a periféricos | **Insuficiente.** La lectura de proximidad no está disponible de forma uniforme y estable en el rango de versiones que el RT-17.03 obliga a soportar | Suficiente | **Suficiente.** Se implementa en el contenedor y se expone al núcleo web por una interfaz acotada y versionada |
| Costo de mantención | Bajo: una base de código | **Alto:** dos bases de código nativas, dos ciclos de publicación y dos matrices de prueba, sostenidos por once personas después del Contrato | **Bajo en la mayor parte:** una sola base de código para captura, interfaz y registro local; superficie nativa mínima |

### 16.3 La decisión

**C-03 es híbrida** (ADR-11): un núcleo web único que contiene la interfaz, la lógica de captura y la cola de sincronización, embebido en un contenedor nativo por plataforma cuya superficie se limita a cuatro capacidades:

1. **Almacenamiento persistente no desalojable** para el registro local, que sostiene las 48 horas y el RPO de cero eventos perdidos.
2. **Almacén de claves respaldado por hardware**, que sostiene la firma del evento y el cifrado a nivel de campo.
3. **Lectura de credencial de proximidad**, que sostiene la identificación del operador en el relevo dentro del segundo comprometido.
4. **Ciclo de vida en segundo plano**, para que la reconciliación se reanude al recuperarse el enlace **sin que nadie abra la aplicación**, que es lo que hace que la sincronización ocurra «sin intervención manual» como exige el criterio n.º 13.

La decisión es reversible en la dirección que importa: la superficie nativa está aislada tras una interfaz propia y acotada, de modo que **sustituir el contenedor no obliga a reescribir la lógica de captura**.

**Alcance.** Rige para C-03. Los componentes que no operan en terreno —C-06, C-07, C-16, C-19 y la consola de C-20— son aplicaciones web servidas por la capa de borde, sin contenedor nativo, porque ninguno de los cuatro fundamentos les aplica.

### 16.4 Dispositivos de bajo costo y de generaciones anteriores (RT-17.08)

Se compromete una **interfaz reducida** de C-03, servida por el mismo núcleo y sin base de código separada, que se activa automáticamente cuando el dispositivo declara memoria o resolución bajo el umbral. **Conserva íntegras las funciones de registro** —identificación por proximidad, botón único, detención, inspección e incidente— y suprime las de consulta y visualización, que se resuelven en el terminal del supervisor.

La razón es de alcance del criterio n.º 14: el compromiso de adopción se mide sobre eventos registrados, y **ninguna función de registro puede quedar fuera del alcance de un dispositivo por su generación**. Importa por la composición del parque: hay **310 vehículos de flota liviana, mayoritariamente de contratistas**, y las personas contratistas no usan dispositivos provistos por el CLIENTE.

**Umbral definido por el PROPONENTE (SUP-39):** la interfaz reducida sostiene los umbrales de terreno en dispositivos de **dos generaciones anteriores** a la vigente, con la misma tasa de éxito de lectura de proximidad. Se valida en la marcha blanca, **sobre el parque real y no sobre el especificado**.

### 16.5 Modo oscuro, personalización e idiomas (RT-13.12)

| Parte | Compromiso | Fundamento |
|---|---|---|
| **Modo oscuro** | Se compromete, **como requisito operacional** del turno de noche. Conmuta de forma automática por hora de turno y por luz ambiente, con conmutación manual disponible | La cabina de noche es un entorno de visión adaptada a la oscuridad: una pantalla clara la destruye durante varios minutos. **Es seguridad operacional** |
| **Personalización por persona** | Sólo en los componentes de oficina —C-16, C-19 y la consola de C-20—. **En terreno la interfaz es fija** | Los dispositivos de terreno son compartidos por turno: una preferencia personal produciría una interfaz distinta en cada relevo, contra el compromiso de capacitación en ≤ 30 minutos y contra el criterio n.º 14 |
| **Múltiples idiomas** | Español de Chile, con la terminología del glosario del Capítulo C. La arquitectura queda preparada para un idioma adicional, **pero no se compromete un segundo idioma en el alcance del Contrato** | El requisito lo condiciona a que el caso lo justifique, y el caso no declara personal que no opere en español |

Accesibilidad conforme a **WCAG 2.2 nivel AA**, verificada con herramientas automatizadas y con pruebas manuales (RT-13.01), y navegación íntegra por teclado con orden de foco lógico en los componentes de oficina (RT-13.11).

### 16.6 Canales de notificación y acción desde el canal (RT-16.26)

**Los canales.** El RT-16.26 exige al menos tres canales y el Capítulo 15 del caso fija cuáles, con un quinto que no es un canal de aplicación sino de faena:

| Canal | A quién sirve | Qué lleva |
|---|---|---|
| Correo electrónico | Personal propio y empresas contratistas | Vencimientos, reportes y resúmenes de turno |
| Notificación en la aplicación | Toda persona usuaria con sesión | Detención pendiente, lote retenido, evento en cuarentena |
| Mensajería instantánea | Empresas contratistas | Acreditación y vencimiento de habilitación, porque el personal administrativo de empresas pequeñas no tiene correo corporativo |
| Mensaje de texto | Destinatarios de alertas críticas y de seguridad | Alerta de fatiga, incidente y arsénico proyectado sobre el límite |
| **Radio y megafonía de faena** | Toda la faena | **Alertas de evacuación.** No es un canal de la solución: la solución entrega el mensaje al sistema existente de radio y megafonía. La arquitectura no interpone ningún componente propio en la ruta de una evacuación, porque un canal de emergencia no puede depender de la disponibilidad de esta solución |

Las plantillas son administrables y versionadas por el CLIENTE (RT-16.21), las preferencias de canal son configurables por persona salvo las que el CLIENTE declare obligatorias, y el envío es asíncrono, con reintento, control de duplicados y registro de entrega.

**La acción desde el canal** se compromete **acotada**, porque la frontera entre notificar y capturar es la misma que separa un registro con valor probatorio de uno sin él.

Se compromete en dos flujos, y en ambos la acción es una elección desde lista cerrada, no captura de texto libre:

- **Vencimiento de habilitación**, dirigido a la empresa contratista y a la persona: permite adjuntar el documento de renovación y disparar el flujo de acreditación **sin entrar al portal**. Es la vía más corta hacia el criterio n.º 7.
- **Detención pendiente de clasificar**, dirigido al supervisor: permite escoger la causa desde la **misma taxonomía cerrada de tres niveles**. No es una excepción a la regla: es la regla ejecutada por otro canal.

**No se compromete, y se declara por qué:** ninguna acción que escriba en la cadena de trazabilidad del mineral —registrar un viaje, un pesaje, un movimiento de stock, conformar o liberar un lote— es ejecutable desde un canal conversacional. El hecho de terreno nace en el dispositivo inventariado, firmado y con identificador de origen; admitirlo por un canal que no ofrece esas tres propiedades **abriría una segunda vía de captura sin valor probatorio y rompería el criterio n.º 1**.

---

### 16.7 Consumo de datos móviles y autonomía de batería (RT-17.07)

El RT-17.07 exige optimizar el consumo de datos móviles y de batería, y **declarar el consumo estimado por turno de trabajo**. Ambas cifras son estimaciones del PROPONENTE y se incorporan al Registro de supuestos para validarse en la marcha blanca sobre el parque real:

| Consumo | Valor declarado | Cómo se consigue |
|---|---|---|
| **Datos móviles** | No superior a **2,5 MB por turno de 12 horas** por dispositivo | Los eventos viajan en formato binario compacto y comprimido, por lotes, y sólo con la diferencia respecto del estado ya sincronizado. La evidencia fotográfica viaja por canal separado y sólo con enlace disponible, conforme al apartado 13.3 |
| **Autonomía de batería** | Cobertura del turno completo de 12 horas sin recarga en cabina, con margen declarado sobre esa duración | Modo oscuro automático en el turno de noche; los periféricos de proximidad y de posición se activan por evento y no de forma continua; **la aplicación no realiza consultas periódicas al servidor**: durante un corte espera pasivamente la señal del sistema operativo, que es la misma propiedad de ciclo de vida en segundo plano del apartado 16.3 |

La autonomía es además requisito de usabilidad por RNF-DES-014, que exige batería suficiente para un turno de doce horas. El consumo de datos importa porque la red inalámbrica del rajo tiene zonas de sombra permanentes y porque el acopio de puerto depende de la conectividad de un operador sin acuerdo de nivel de servicio: **cuanto menos dato viaja, menos veces se está esperando red**.

### 16.8 Ergonomía ambiental en el rajo (RT-13.08)

El RT-13.08, con el valor que le fija el Capítulo 15 del caso, exige operación con guantes, legibilidad bajo radiación solar directa, funcionamiento entre **−6 °C y 34 °C**, resistencia a polvo y a vibración, y operación sin conexión. Cada condición se traduce en una decisión de interfaz verificable:

| Condición del caso | Decisión de interfaz | Umbral y origen |
|---|---|---|
| Operación con guantes de invierno | Objetivos táctiles amplios y separados, acción primaria de un solo toque, confirmación por proximidad o por toque sostenido. **Sin gestos compuestos ni teclado virtual en cabina** | Objetivo táctil no inferior a **44 × 44 píxeles** (RNF-USA-014); tasa de éxito al primer intento superior al **95 %** con guante grueso (RNF-USA-001); apertura de sesión en 1 segundo sin retirar el guante (SUP-29) |
| Legibilidad bajo radiación solar directa | Alto contraste y tipografía de trazo pesado, con la misma jerarquía en modo claro y en modo oscuro | Luminancia igual o superior a **1.000 cd/m²** (RNF-USA-002); contraste conforme a **WCAG 2.2 nivel AA**, que es el nivel que compromete el RT-13.01 |
| Temperatura de −6 °C a 34 °C | Ninguna interacción depende de la sensibilidad fina del dedo desnudo ni de la respuesta de una pantalla fría | Funcionamiento validado en todo el rango (RNF-USA-003) |
| Polvo y vibración | La interfaz tolera el toque accidental inducido por la vibración: la acción primaria exige intención sostenida y toda acción con efecto sobre el registro es confirmable y reversible como evento nuevo | RNF-USA-003; decisión n.º 1 del apartado 2.3, que convierte toda corrección en evento nuevo |
| Operación sin conexión | Resuelta en el apartado 11: registro local firmado, con estado de sincronización visible de forma permanente | Criterio n.º 13 |

La ventana de tolerancia concreta frente al rebote táctil, como cualquier otro parámetro de interacción, es **parámetro de administración versionado de C-20** y no una constante de código, conforme a la regla que gobierna la capa 4.

### 16.9 Búsqueda global (RT-16.27)

El RT-16.27 exige búsqueda global con indexación de texto completo, tolerancia a errores de escritura, filtros facetados y respeto del control de acceso de la persona que busca. Las tres primeras propiedades son de producto y se cierran en el Subdocumento 4.2. Las dos decisiones que sí son de arquitectura lógica, y por eso se toman aquí:

**El índice hereda la clasificación del dato, no la relaja.** La búsqueda se resuelve sobre un índice invertido separado de los almacenes operacionales, y cada documento indexado conserva su nivel del apartado 13.2 y su ámbito de rol, área y sitio. El filtrado por permiso ocurre **en la consulta al índice y no sobre el resultado**: un resultado que la persona no puede ver no llega a existir, de modo que ni el recuento de coincidencias ni las facetas filtran información por su sola forma.

**Qué no se indexa, por decisión.** El atributo clasificado como personal sensible no entra al índice de texto completo en ningún caso —habilitación y su resultado, salud ocupacional, fatiga, alcohol y drogas, y el relato de incidente—, porque un índice invertido es, por construcción, una copia consultable del contenido y el cifrado a nivel de campo del RT-11.10 quedaría anulado por él. Esos registros se localizan por sus atributos estructurados y por su identificador, nunca por su contenido. La **ley por polígono** tampoco se indexa, por la misma razón aplicada al nivel confidencial: es la vía por la que saldría sin romper ningún control de acceso, que es justo el caso de detección declarado en el apartado 13.6.

---

## 17. Qué queda fuera del Contrato

Tres capacidades quedan **excluidas del Contrato en ambas etapas**, con mecanismo de cambio del Artículo 72.º de las Bases Administrativas. **La arquitectura las contempla como puntos de extensión declarados, no como funciones latentes**: es decir, incorporarlas después no exige rediseñar nada.

| Excluido | Punto de extensión previsto | Requerimiento |
|---|---|---|
| Prorrateo automático del viaje que carga de dos polígonos | El viaje lleva marca de proximidad a borde. El prorrateo sería una regla nueva sobre el mismo agregado, **sin cambio de modelo** | RF-CAR-008, C-01 |
| Biometría en portería | El control de acceso admite un proveedor de verificación adicional detrás de la misma interfaz. **La biometría del recinto técnico de servidores sí está en alcance**: la exige el RT-06.20 | RF-IDE-013, C-05 |
| Factor de corrección de pesaje por equipo y período | El motor de conciliación ya parametriza factores versionados por RN-27; el factor por equipo sería **un parámetro más, no un componente** | RF-PES-006, C-08 |

---

## 18. Cómo verificar cada compromiso

Esta tabla es la que el CLIENTE puede usar para auditar la arquitectura contra el Capítulo 18: lleva de cada criterio de aceptación a los componentes que lo sostienen.

| Criterio | Qué se compromete | Componentes |
|---|---|---|
| n.º 1 | Origen de cualquier lote embarcado con evidencia en **menos de 2 horas**, mes 15 | C-01, C-09, C-03, C-04, C-10, C-22 |
| n.º 2 | Conciliación cerrada en **no más de 3 días hábiles**, mes 19 | C-08, C-09, C-01, C-12 |
| n.º 3 | Diferencia bajo **2 %** al mes 21, con **100 % atribuido a un punto de medición** | C-08, C-09, C-01 |
| n.º 4 | Cumplimiento del plan al inicio del turno siguiente, contra la versión vigente | C-02, C-19, C-01, C-11 |
| n.º 5 | **100 % de detenciones clasificadas** antes del cierre del turno siguiente, mes 15 | C-15, C-19, C-10, C-13 |
| n.º 6 | Estado, historial, alarmas y pautas **en una sola vista**. Base mes 15, completa mes 21 | C-13, C-16, C-10, C-12 |
| n.º 7 | **Cero rechazos en portería** por documentación vencida advertible, mes 15 | C-05, C-06, C-04, C-22 |
| n.º 8 | Acreditación de contratista en **menos de 48 horas**, mes 15 | C-06, C-05, C-20 |
| n.º 9 | **100 % de registros de seguridad nacidos digitales**, mes 15 | C-20, C-05, C-03, C-22, C-18 |
| n.º 10 | **100 % de alertas de fatiga con acción documentada**, mes 21 | C-18, C-20, C-16 |
| n.º 11 | **100 % de reportes ambientales generados desde el sistema**, mes 21 | C-17, C-19, C-20, C-09, C-18 |
| n.º 12 | Huella de carbono mensual auditable, mes 18 | C-17, C-20, C-19, C-01, C-14 |
| n.º 13 | **48 horas sin enlace y sincronización en 4 horas**, desde marcha blanca | C-04, C-22, C-03 |
| n.º 14 | **90 % de eventos registrados sin intervención del supervisor**, mes 15 | C-22, C-03, C-05, C-21 |

**Sobre los criterios que se comprometen al mes 15 apoyados en componentes de Etapa 2.** Los criterios n.º 5 y n.º 9 se alcanzan al mes 15, dentro de la Etapa 1, y entre sus componentes figuran C-13 y C-18, que el apartado 8 sitúa mayoritariamente en Etapa 2. No es una contradicción, y se declara para que no se lea como tal:

- **Criterio n.º 5**, clasificación de detenciones. Se sostiene al mes 15 con C-15 y C-10: la detección y la clasificación automática operan desde las señales que el despacho ya genera, y el apartado 11.4 declara que esa detección sigue funcionando incluso sin telemetría. C-13 enriquece la causa con la alarma del fabricante y llega en Etapa 2; su ausencia no impide clasificar el 100 %.
- **Criterio n.º 9**, registros de seguridad nacidos digitales. Se sostiene al mes 15 con C-03, C-05 y C-20: el registro nace en terreno, firmado y con identificador de origen. De C-18, la inspección y el incidente son el requerimiento de Etapa 1 —el uno de seis que el apartado 8 deja fuera del «5 de 6»—; lo que llega en Etapa 2 es la gestión de alertas de fatiga, que es el criterio n.º 10 y se compromete al mes 21.

**Cómo leer esta tabla frente a la matriz de trazabilidad.** El orden de cada fila es el del peso con que cada componente sostiene el criterio en la matriz v1.1, medido por número de requerimientos asociados. La tabla incluye además algún componente que la matriz no asocia formalmente pero sin el cual el compromiso no se sostiene —C-03 y C-04 en el criterio n.º 1, porque el dato que se reconstruye nace ahí—. **Donde la matriz y esta tabla difieran, la matriz manda para efectos de trazabilidad de requerimientos, y esta tabla explica el diseño.**

---

## 19. Requisitos transversales que este subdocumento acredita

La hoja «Pendientes por grupo» del **Formulario T-12 v1.2** asignó a «Subdoc. 4.1» **noventa y siete códigos**. Se reparten así, y la suma cierra:

- **Cincuenta** se acreditan íntegramente en este subdocumento.
- **Uno**, **RT-03.24**, se parte entre este entregable y el 4.2: la calidad de servicio y la priorización de tráfico se declaran aquí; el estudio de cobertura del rajo, allá.
- **Cuarenta y seis** se reasignan con fundamento, para que ninguno quede huérfano entre entregables.

La tabla siguiente enumera **ochenta y tres** códigos: los cincuenta y uno anteriores más **treinta y dos** que el T-12 v1.2 no había asignado a este subdocumento y que aquí quedan acreditados de todos modos, porque su decisión es de arquitectura lógica. En el **Formulario T-12 v1.3**, los ochenta y cinco códigos que este subdocumento sostiene —los ochenta y tres de la tabla, más RT-03.10 y RT-03.24 en su parte de aquí— apuntan a su apartado de esta versión 4.2, cumpliendo el punto 3 del numeral 1.5 transversal: no basta declarar «cumple», hay que señalar dónde se desarrolla.

**Sobre RT-09.01 y RT-09.02.** El cálculo de capacidad del RT-09.01 y la concurrencia del RT-09.02 se derivan aquí, en los apartados 12.1 y 12.6, a partir de la volumetría del numeral 14.1 y de SUP-31 y SUP-40. Lo que **no** se cierra aquí es el dimensionamiento de la infraestructura que sostiene esa carga —cantidad de instancias, tamaños y costo—, que es del Subdocumento 4.2 por el Artículo 16.º 2, y la ejecución de las pruebas de carga del RT-09.06, que es del Subdocumento 9.

| Familia | Códigos acreditados aquí | Dónde |
|---|---|---|
| Arquitectura (RT-02) | RT-02.01 a RT-02.14. RT-02.12 —multi-tenencia hacia 2030— es uno de los treinta y dos extras: el T-12 v1.2 no lo había asignado aquí | Apartados 1, 4, 5, 7, 9, 10, 11, 12, 15 y 21 |
| Continuidad y borde (RT-03) | RT-03.04, RT-03.07, RT-03.10, RT-03.13, RT-03.16, RT-03.18, RT-03.19, RT-03.24 (parcial) | Apartados 3, 5, 11, 13, 14 y 15 |
| Construcción con impacto arquitectónico (RT-04) | RT-04.07, RT-04.08, RT-04.10 | Apartado 15 |
| Integración (RT-05) | RT-05.05, RT-05.10, RT-05.16 a RT-05.19, RT-05.23, RT-05.29 | Apartados 5, 10, 12 y 14 |
| Desempeño (RT-09) | RT-09.01, RT-09.02, RT-09.03, RT-09.04, RT-09.05, RT-09.08 | Apartados 12 y 14 |
| Seguridad (RT-11) | RT-11.01 a RT-11.03, RT-11.07 a RT-11.15, RT-11.25, RT-11.27 | Apartado 13 |
| Identidad (RT-12) | RT-12.02, RT-12.03, RT-12.04, RT-12.07, RT-12.11, RT-12.12 | Apartados 5 y 13 |
| Experiencia (RT-13) | RT-13.01, RT-13.08 (ergonomía en rajo), RT-13.11, RT-13.12 | Apartado 16 |
| Observabilidad (RT-14) | RT-14.01 a RT-14.04, RT-14.07 a RT-14.09 | Apartado 14 |
| Sostenibilidad (RT-15) | RT-15.02, en sus dos lecturas | Apartado 15.8 |
| Administración (RT-16) | RT-16.02, RT-16.03, RT-16.05, RT-16.10, RT-16.15, RT-16.21, RT-16.26 (canales), RT-16.27 (búsqueda global), RT-16.29 | Apartados 5, 12, 14, 15 y 16 |
| Canales (RT-17) | RT-17.02, RT-17.03, RT-17.07 (consumos de datos y batería), RT-17.08 | Apartado 16 |

**Los cuarenta y seis reasignados** se reparten así: proveedor de nube, regiones, infraestructura como código, servicios administrados, endurecimiento, redundancia del enlace y ancho de banda por sitio van al **Subdocumento 4.2**, porque son decisiones de emplazamiento y dimensionamiento; ambientes, ramas, integración continua, cobertura de pruebas y métricas de entrega van a los **Subdocumentos 6 y 7**, porque son proceso de construcción y no estructura de la solución; ejecutar e informar las pruebas de carga va al **Subdocumento 9**.

---

## 20. Qué no resuelve este documento, y quién lo resuelve

| Qué falta | De quién | Por qué importa |
|---|---|---|
| Incorporación de **SUP-40** al Registro de supuestos | Registro de supuestos, v1.6 | El apartado 12.6 declara el techo de 35.000 eventos por sitio con su derivación. Mientras no esté en el registro, es una cifra sin instancia de validación declarada |
| **Producto y versión concretos de cada componente** | Subdoc. 4.2 | El numeral 1.5 transversal advierte que declarar «Cumple» sin individualizar producto y versión equivale a no declarar, y la Comisión califica el requisito como **no acreditado**. Es el pendiente de mayor riesgo de puntaje que este subdocumento no puede cerrar solo |
| Tabla de emplazamiento componente por componente, nube y faena | Subdoc. 4.2 | Exigida por el Artículo 16.º 2. Determina qué se sostiene en faena durante el corte de fibra |
| Especificación de la sala técnica y de los cuatro gabinetes de borde | Subdoc. 4.2 | La sala actual de 40 m² no cumple el estándar del Capítulo 6 |
| Especificación del hardware de terreno | Subdoc. 4.2, Formulario T-11 | Referencia, cantidad y características por sitio (RT-08.10) |
| Modelo de datos físico, linaje y catálogo | Subdoc. 5 | Debe soportar las entidades y eventos del apartado 9 |
| Paquetes de la EDT | Informe 2 | Las 228 filas de la matriz de trazabilidad dicen «Se define en el Informe 2» |
| Emplazamiento, producto y versión de los controles de seguridad y observabilidad | Subdoc. 4.2 | Este subdocumento decide el **modelo y los controles lógicos**; el producto y el emplazamiento son del 4.2 |

**Consolidación antes de la entrega.** El equipo de este subdocumento verifica que los nombres de componente, los nombres de sitio y la lista de funciones degradadas del apartado 11.4 coincidan **literalmente** entre los Subdocumentos 3, 4.1, 4.2 y 5.

---

## 21. Índice de decisiones de arquitectura

Cada decisión está respaldada por un ADR fechado con la alternativa escogida, las descartadas y el criterio de decisión. El registro es entregable contractual y se actualiza durante toda la ejecución (RT-02.04).

| ADR | Decisión | Alternativa descartada | Criterio |
|---|---|---|---|
| ADR-01 | Servicios de negocio modulares con despliegue independiente | Microservicios de grano fino; monolito único | 200 transacciones por segundo en peak y 11 personas de TI del CLIENTE |
| ADR-02 | El viaje de camión como agregado raíz | El polígono; el turno | El viaje ya se registra hoy en el despacho: no agrega captura nueva y es la granularidad más fina que responde con evidencia |
| ADR-03 | Nueve límites de contexto con capa anticorrupción hacia lo externo | Modelo canónico único compartido con los sistemas del CLIENTE | El CLIENTE no tiene documentadas las interfaces ni las versiones de sus sistemas |
| ADR-04 | Bus de eventos como columna vertebral | Integración punto a punto por servicio síncrono | La trazabilidad exige linaje del hecho; el punto a punto no lo conserva y no sobrevive al corte |
| ADR-05 | Identificador de evento generado en el dispositivo | Identificador asignado por el servidor al recibir | Sin identificador local, una reconexión parcial obliga a resolver duplicados a mano |
| ADR-06 | Tres mecanismos de identidad sobre un maestro único | Federación corporativa estricta para toda la población | Provisionar hasta 8.500 identidades con 11 personas de TI es inviable |
| ADR-07 | Dos puertas de enlace lógicas | Puerta única | Perfiles de tráfico incompatibles: un ataque al portal del comprador no puede afectar el registro de terreno |
| ADR-08 | Separación transaccional / analítica desde el diseño | Base única con réplicas de lectura | La capa analítica tolera 15 minutos de latencia y el terreno no |
| ADR-09 | Reglas y umbrales como parámetros versionados con vigencia temporal | Constantes de código | Sin recálculo con la versión vigente en el período, los criterios n.º 3 y n.º 4 no se verifican sobre historia cerrada |
| ADR-10 | Un adaptador por fabricante de telemetría | Adaptador único con modelo unificado | La propiedad contractual del dato no está resuelta para los tres; aislar permite que el compromiso no dependa del resultado de la negociación |
| ADR-11 | Aplicación de terreno híbrida | Web progresiva; nativa por plataforma | La web progresiva no garantiza persistencia ni lectura de proximidad; la nativa duplica la base de código sobre un equipo de once personas |
| ADR-12 | Zero Trust con punto de decisión delegado y firmado en el borde | Zero Trust estricto sin excepción; confianza por posición de red | Verificar contra el servicio central detendría la mina durante un corte. Delegar y volver a verificar conserva las tres propiedades de NIST SP 800-207 |
| ADR-13 | Clasificación en cuatro niveles con control diferenciado | Dos niveles con control común | Un control común sobreprotege la ley por polígono y subprotege el dato personal sensible |
| ADR-14 | Escalado anticipado por calendario, con reactivo como red de seguridad | Escalado reactivo puro | El peak es predecible: 08:00 y 20:00, de 45 a 60 minutos, 200 transacciones por segundo |
| ADR-15 | Despliegue canario con progresión por sitio y migraciones por expansión y contracción | Azul-verde con conmutación completa | Los siete sitios no fallan igual, y un nodo de borde reconcilia eventos de la versión anterior tras 48 horas aislado |
| ADR-16 | Configuración técnica y parámetro de negocio en mecanismos separados | Un único mecanismo por ambiente | Si un umbral de conciliación fuera configuración técnica, cambiarlo reescribiría el pasado |
| ADR-17 | Un solo identificador de correlación, nacido en el dispositivo | Identificador de traza distinto del de integración | Dos identificadores obligan a unirlos para seguir una operación extremo a extremo |

---

## 22. Referencias

International Organization for Standardization. (2022). *ISO/IEC/IEEE 42010:2022 — Software, systems and enterprise: Architecture description*.

International Electrotechnical Commission. (2013-2024). *IEC 62443 — Security for industrial automation and control systems* [serie]. Se aplican en particular la 62443-3-3 sobre requisitos de sistema y niveles de seguridad, la 62443-4-1 sobre el ciclo de vida de desarrollo seguro, la 62443-4-2 sobre requisitos técnicos de componente y la 62443-2-1:2024 sobre el programa de seguridad del operador.

International Society of Automation. (2013). *ANSI/ISA-95 — Enterprise-control system integration*, adoptada como IEC 62264.

National Institute of Standards and Technology. (2020). *SP 800-207 — Zero Trust Architecture*.

International Organization for Standardization. (2018). *ISO 45001 — Occupational health and safety management systems*; *ISO 14001 — Environmental management systems*.

Ley N.º 21.719 de 2024. Regula la protección y el tratamiento de los datos personales y crea la Agencia de Protección de Datos Personales. 13 de diciembre de 2024. Diario Oficial de la República de Chile. **Entra en vigencia el 1 de diciembre de 2026**, esto es, antes del mes 1 del cronograma contractual, que SUP-21 sitúa no después de abril de 2027: la ley rige desde el primer día del Contrato y ninguna de sus obligaciones se difiere.

Ley N.º 21.663 de 2024. Ley Marco de Ciberseguridad e Infraestructura Crítica de la Información. 8 de abril de 2024. Diario Oficial de la República de Chile.

Ley N.º 21.459 de 2022. Establece normas sobre delitos informáticos. 20 de junio de 2022. Diario Oficial de la República de Chile.

Ley N.º 19.799 de 2002. Sobre documentos electrónicos, firma electrónica y servicios de certificación de dicha firma. 12 de abril de 2002. Diario Oficial de la República de Chile. Sostiene el valor probatorio del registro local firmado del apartado 11 y de los reportes firmados de la capa documental.

Decreto Supremo N.º 132 de 2002 [Ministerio de Minería]. Aprueba Reglamento de Seguridad Minera. 7 de febrero de 2004. Diario Oficial de la República de Chile.

Parker, H. M. (2012). Reconciliation principles for the mining industry. *Mining Technology, 121*(3), 160-176. https://doi.org/10.1179/1743286312Y.0000000007

World Business Council for Sustainable Development y World Resources Institute. (2004). *The Greenhouse Gas Protocol: A corporate accounting and reporting standard* (ed. rev.). Base del cálculo de alcances 1, 2 y 3 que exige RN-26.
