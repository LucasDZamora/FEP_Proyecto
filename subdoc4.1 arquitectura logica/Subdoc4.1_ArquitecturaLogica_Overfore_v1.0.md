# Subdocumento 4.1 — Arquitectura lógica de la solución

| Antecedente | Detalle |
|---|---|
| Proponente | OVERFORE S.A. |
| Licitación | TFEP-01/2026 — Caso 01 · Minería |
| Mandante | Compañía Minera Altos de Aranda S.A. (empresa ficticia) — Rajo Aranda, Sierra Gorda, Región de Antofagasta |
| Contenido | a) Esquema de solución. b) Arquitectura lógica: capas, módulos, límites de contexto, responsabilidades e interfaces |
| Ponderación | 16 % de la evaluación técnica (numeral 4.1 de la pauta) |
| Documentos que rigen | Bases Administrativas FEP01.26 · Bases Técnicas Transversales FEP02.26 · Bases Técnicas del Caso 01, v1.0 del 18-08-2026 |
| Insumos internos | Subdocumento 3 v1.4 · Catálogo de requerimientos v2.4 · Matriz de trazabilidad v1.1 · Registro de supuestos v1.4 · Registro de reglas de negocio v1.2 · Formulario T-12 v1.2 |
| Versión | 1.0 — 1 de septiembre de 2026. Primera emisión. |

Este subdocumento acredita los requisitos RT-02.01 a RT-02.14 del documento transversal, que hasta la v1.2 del Formulario T-12 figuraban en blanco por no existir todavía este entregable. No repite el problema, que se desarrolla en el Subdocumento 2, ni el alcance, que se desarrolla en el Subdocumento 3. El emplazamiento físico de cada componente se desarrolla en el Subdocumento 4.2 y el modelo de datos en el Subdocumento 5.

**Advertencia de coherencia.** El Artículo 57.2 de las Bases Administrativas habilita descuento de puntaje por incoherencia entre secciones de la propuesta. Los nombres de componente (C-01 a C-22), los nombres de sitio y la lista de funciones degradadas usados aquí son los mismos del Subdocumento 3, del catálogo de requerimientos y de la matriz de trazabilidad, y deben permanecer literalmente iguales en los Subdocumentos 4.2 y 5.

---

## 1. Marco de descripción y método

### 1.1 Norma de descripción arquitectónica (RT-02.03)

La arquitectura se describe conforme a **ISO/IEC/IEEE 42010**, con las cinco vistas que exige el requisito. La tabla declara dónde vive cada vista, para que la Comisión Evaluadora no busque en el documento equivocado.

| Vista ISO/IEC/IEEE 42010 | Qué responde | Dónde se desarrolla |
|---|---|---|
| Lógica | Qué capas y módulos existen, qué hace cada uno y qué interfaces exponen | **Este subdocumento**, apartados 4 a 8 |
| De procesos | Cómo se comporta el sistema en ejecución: concurrencia, eventos, sincronización, degradación | **Este subdocumento**, apartados 9 y 10 |
| De despliegue | Dónde se ejecuta cada componente: nube, faena, borde | Subdocumento 4.2 (tabla de emplazamiento exigida por el Artículo 16.2) |
| De datos | Entidades, linaje, retención, volumetría | Subdocumento 5. El modelo de dominio de negocio va en el apartado 7 de este subdocumento, por exigencia expresa del RT-02.13 |
| De seguridad | Identidad, autorización, cifrado, segregación TI/TO, auditoría | **Este subdocumento**, apartado 11; el detalle de controles va en el Subdocumento 4.2 |

### 1.2 Registro de decisiones de arquitectura (RT-02.04)

Cada decisión de este subdocumento está respaldada por un ADR fechado con la alternativa escogida, las descartadas y el criterio de decisión. El índice de ADR está en el apartado 15. El registro es entregable contractual y se actualiza durante toda la ejecución.

### 1.3 Cómo debe leerse el diagrama que acompaña a este texto

Este subdocumento es la especificación textual completa. El diagrama de la arquitectura lógica que exige el RT-02.01 se dibuja a partir del apartado 14, que entrega la lista literal de columnas, cajas y conectores. Se adopta la representación por **columnas verticales por capa**, de izquierda a derecha en el orden en que viaja una transacción, con flechas de color por actor y su leyenda.

Se descarta expresamente la representación de capas apiladas con columnas laterales de hardware, software y servicios externos: mezcla decisiones lógicas con decisiones tecnológicas, y la tecnología es materia del Subdocumento 4.2. Si el equipo de diseño gráfico necesita mostrar el stack, va como anexo rotulado, fuera del marco de la arquitectura lógica.

---

## 2. Esquema de solución (numeral 4.1 letra a)

### 2.1 Qué es la solución

Un **sistema de registro y trazabilidad del mineral y de las personas** que convive con los cinco sistemas que el CLIENTE mantiene —el ERP corporativo, el despacho de flota con contrato vigente hasta 2031, el sistema de gestión de laboratorio, el software de planificación minera y el historiador de proceso de planta, este último de sólo lectura— y les agrega la capa de trazabilidad que hoy no existe entre ellos. No reemplaza ninguno.

La solución no es un catálogo de módulos. Es un **cambio del punto de captura del dato**. Los tres problemas del Subdocumento 2 —trazabilidad comercial, productividad y evidencia ante terceros— comparten una sola causa: el dato no nace donde ocurre el hecho. Toda la arquitectura está ordenada por esa tesis, y de ahí se siguen las cuatro decisiones que la estructuran.

### 2.2 Las cuatro decisiones que estructuran la arquitectura

| N.º | Decisión | Consecuencia arquitectónica | Regla | Supuesto |
|---|---|---|---|---|
| 1 | El **viaje de camión** es la unidad atómica de trazabilidad. El polígono es un atributo del viaje; turno, lote de chancado y lote comercial son agregaciones derivadas | El agregado raíz del contexto Mineral es el viaje, no el polígono ni el turno. Toda corrección es evento nuevo, nunca modificación del registro original | RN-01 | SUP-01 |
| 2 | La **balanza de alimentación de planta** es la fuente de verdad del tonelaje. El pesaje de a bordo se usa para atribución espacial prorrateada; la báscula de camiones es punto de calibración | Las tres mediciones se persisten tal como se generan; la diferencia se publica por punto de medición. Ninguna se sobrescribe con la otra | RN-04 | SUP-02 |
| 3 | La **persona operadora se identifica por proximidad** de credencial en el relevo, sin login ni contraseña en terreno. Si nadie se identifica, el registro se marca «operador no identificado» y el equipo no se detiene | La identidad de terreno es un componente propio (C-05) y no un módulo del portal. La autenticación de terreno y la de oficina son mecanismos distintos sobre el mismo maestro de personas | RN-22 | SUP-04 |
| 4 | El diseño es **desconectado desde el origen**, no por excepción | La operación desconectada no es un componente: es una propiedad de la Etapa 1 completa, materializada por el nodo de borde (C-04) y por el registro local firmado de la aplicación de terreno (C-03) | RN-30 | SUP-09 |

### 2.3 Qué se construye, qué se integra y qué se compra

**Se construye:** el modelo de trazabilidad del mineral, el registro en terreno con operación desconectada, la identidad y acreditación de personas, el portal de empresas contratistas, la conciliación metalúrgica y los tableros de indicadores por turno.

**Se integra, sin sustituir:** el ERP corporativo, el sistema de despacho de flota, el sistema de gestión de laboratorio en ambos sentidos, el software de planificación minera, el historiador de proceso en sólo lectura y los tres portales de telemetría de fabricante. Cada uno queda detrás de un adaptador reemplazable con capa anticorrupción.

**Se especifica pero lo adquiere el CLIENTE:** el hardware de terreno —dispositivos, lectores, etiquetas e instrumentación adicional— con referencia, cantidad, características, accesorios y costo unitario estimado, conforme al RT-08.10.

### 2.4 Qué queda fuera del Contrato

Tres capacidades quedan **excluidas del Contrato en ambas etapas**, con mecanismo de cambio del Artículo 72.º de las Bases Administrativas. La arquitectura las contempla como puntos de extensión declarados, no como funciones latentes:

| Excluido | Punto de extensión previsto | Requerimiento |
|---|---|---|
| Prorrateo automático del viaje que carga de dos polígonos | El viaje lleva marca de proximidad a borde. El prorrateo sería una regla nueva sobre el mismo agregado, sin cambio de modelo | RF-CAR-008, componente C-01 |
| Biometría en portería | El control de acceso de personas admite un proveedor de verificación adicional detrás de la misma interfaz. La biometría del recinto técnico de servidores **sí** está en alcance: la exige el RT-06.20 | RF-IDE-013, componente C-05 |
| Factor de corrección de pesaje por equipo y período | El motor de conciliación ya parametriza factores versionados por RN-27; el factor por equipo sería un parámetro más, no un componente | RF-PES-006, componente C-08 |

---

## 3. Estilo arquitectónico y alternativa descartada (numeral 2.3 transversal)

El CLIENTE no impone estilo. Exige que el escogido sea explícito, coherente con el volumen del caso y justificado, y que se comparen al menos dos alternativas considerando complejidad operacional, tamaño y competencias del equipo, costo de infraestructura y **capacidad del CLIENTE de operarla al término del Contrato**. El propio documento advierte que adoptar microservicios para un caso cuyo volumen no lo justifica es un error de ingeniería y se evalúa como tal.

### 3.1 Estilo escogido: servicios de negocio modulares con despliegue independiente

La capa de servicios de negocio se organiza en **nueve servicios de grano medio**, cada uno alineado a un límite de contexto del apartado 6, desplegable de forma independiente, sin estado y con contrato versionado. No se subdivide en microservicios de grano fino.

Cifras que sostienen la decisión:

| Dimensión | Valor declarado | Fuente |
|---|---|---|
| Transacciones por segundo en régimen | 40 | SUP-31 |
| Transacciones por segundo en peak (cambio de turno) | 200 | SUP-31 |
| Equipo de TI del CLIENTE que recibe la operación | 11 personas | Restricción n.º 10 |
| Sitios operacionales a cubrir | 7, incluido un acopio en recinto portuario de terceros | Capítulo 3 del caso |
| Duración de la Operación tras la implementación | 36 meses, del mes 21 al 56 | Artículo 17.º |

Doscientas transacciones por segundo en peak no justifican descomposición fina. Once personas no pueden operar una malla de servicios al término del Contrato. La decisión se toma por la restricción n.º 10 antes que por preferencia técnica.

### 3.2 Alternativa descartada: microservicios de grano fino por capacidad de negocio

| Criterio del numeral 2.3 | Microservicios de grano fino | Servicios modulares (escogido) |
|---|---|---|
| Complejidad operacional | Malla de servicios, descubrimiento, trazas distribuidas sobre decenas de procesos, gestión de versiones cruzadas | Nueve unidades desplegables, trazas correlacionadas sobre un número acotado de saltos |
| Tamaño y competencias del equipo | Exige especialidad en plataforma que el CLIENTE declara no tener | Operable con perfiles de administración de aplicaciones y base de datos |
| Costo de infraestructura | Sobredimensionado para 40 transacciones por segundo en régimen | Escala horizontal por servicio sólo donde el peak lo exige |
| Capacidad del CLIENTE al término del Contrato | Riesgo alto de dependencia permanente del ADJUDICATARIO | Transferencia viable, conforme al numeral 17.6 punto 7 |

**Alternativa también descartada: monolito único.** Lo rechaza el RT-02.02, que exige poder desplegar de forma independiente los componentes críticos. El registro de terreno y la conciliación tienen ciclos de cambio y ventanas de intervención distintos: la conciliación no se toca durante el cierre contable, el registro de terreno no se toca durante el cambio de turno.

### 3.3 Cómo se materializa el requisito de despliegue independiente (RT-02.02)

Son críticos y desplegables por separado, cada uno con su propio contrato versionado: el núcleo de trazabilidad (C-01), la identidad y control de acceso de personas (C-05), el nodo de borde (C-04) y la puerta de enlace de servicios. Un despliegue de la analítica (C-19) o del módulo ambiental (C-17) no puede detener el registro de un viaje.

---

## 4. Las ocho capas obligatorias (RT-02.01)

La solución se organiza en las ocho capas del numeral 2.1 del documento transversal. Las capas son de existencia obligatoria. La tabla reproduce la responsabilidad y la exigencia mínima que fija el documento, y agrega cómo las materializa esta solución.

### Capa 1 · Presentación

- **Responsabilidad (numeral 2.1):** interfaces de las personas usuarias: portal web, aplicación móvil, terminales operacionales y pantallas de terreno.
- **Exigencia mínima:** diseño adaptativo, accesible y sin lógica de negocio. Ninguna interfaz puede acceder directamente a la base de datos.
- **Cómo se materializa:** seis frentes de presentación. La aplicación de terreno (C-03) es el caso singular: es la única pieza de presentación que conserva **estado local persistente**, porque debe operar 48 horas sin enlace. Ese estado es un registro de eventos firmado, no una copia de la base de datos central, y no contiene lógica de negocio: contiene hechos observados pendientes de reconciliación.
- **Restricción de terreno que condiciona el diseño:** operación con guantes gruesos y lentes de seguridad, luminosidad de sol directo a oscuridad total, temperaturas de −6 °C a 34 °C, polvo en suspensión permanente. Toda interfaz que exija retirar el guante es inoperable. En cabina el único elemento nuevo es un **botón único**.

### Capa 2 · Borde y exposición

- **Responsabilidad:** único punto de entrada público: distribución de contenidos, balanceo, protección perimetral y terminación de cifrado.
- **Exigencia mínima:** distribución de contenidos, cortafuegos de aplicación gestionado, protección contra denegación de servicio en capas 3, 4 y 7, y terminación TLS 1.3.
- **Cómo se materializa:** un único frente público para los dos portales autenticados —empresas contratistas (C-06) y trazabilidad de lote para compradores (C-07)—. No existe portal abierto a la ciudadanía: el caso lo excluye. El aislamiento de recursos del portal respecto del núcleo transaccional es requisito propio (RNF-DIS-010), con umbral de cero degradación de los umbrales de terreno ante diez veces el tráfico medio del portal.
- **Advertencia:** esta capa no está en la ruta del registro de terreno. Un viaje de camión no atraviesa el borde público. Confundirlo haría que un ataque de denegación de servicio al portal del comprador detuviera la mina.

### Capa 3 · Puerta de enlace de servicios

- **Responsabilidad:** publicación, autenticación, autorización, cuotas, límites de tasa, versionado y observabilidad de las interfaces de programación.
- **Exigencia mínima:** validación de esquema, inspección de carga útil, trazabilidad por transacción y catálogo de servicios.
- **Cómo se materializa:** dos puertas de enlace lógicas con política distinta, porque los perfiles de tráfico son incompatibles:
  - **Puerta operacional**, para terreno y nodos de borde: prioriza latencia, admite lotes de reconciliación grandes al restablecerse el enlace y aplica calidad de servicio conforme al RT-03.24.
  - **Puerta de exposición**, para portales externos e integraciones de sistemas: prioriza cuota y límite de tasa por consumidor.
- **Contrato de interfaz:** servicios síncronos documentados en OpenAPI 3.1 y flujos dirigidos por eventos en AsyncAPI 2.6 o superior, generados desde el código (RT-05.16). Versionado semántico con compatibilidad hacia atrás y obsolescencia con preaviso mínimo de seis meses (RT-05.17).

### Capa 4 · Servicios de negocio

- **Responsabilidad:** lógica del proceso de negocio del caso, organizada en módulos con límites de contexto explícitos.
- **Exigencia mínima:** sin estado, desplegables de forma independiente, con contratos versionados y compatibilidad hacia atrás.
- **Cómo se materializa:** nueve servicios, uno por límite de contexto del apartado 6. Sin estado conforme al RT-02.05: el estado de sesión y el estado de proceso residen en almacenes externos de alta disponibilidad, nunca en memoria del servicio.
- **Regla que gobierna esta capa:** los umbrales, plazos, tolerancias y catálogos son **parámetros de administración versionados, no constantes de código** (RN-27, RT-16.02). Un período anterior se recalcula con la versión de regla que regía entonces. Sin esa propiedad, los criterios de aceptación n.º 3 y n.º 4 no se pueden verificar sobre historia ya cerrada.

### Capa 5 · Integración y eventos

- **Responsabilidad:** comunicación asíncrona, desacoplamiento, orquestación y coreografía de procesos entre módulos y con sistemas externos.
- **Exigencia mínima:** bus o intermediario de mensajería con persistencia, cola de mensajes fallidos, reintento y deduplicación.
- **Cómo se materializa:** el bus de eventos es la **columna vertebral de la trazabilidad**, no un accesorio de integración. Todo hecho de terreno entra al sistema como evento con identificador generado en el dispositivo de origen, lo que hace la reconciliación idempotente y reintentable (RN-30, RT-02.06). Entrega al menos una vez con deduplicación en el consumidor, y orden garantizado dentro del agregado cuando el proceso lo exige (RT-02.07): el orden importa dentro de un lote y dentro de una sesión de operador, no globalmente.
- **Los seis adaptadores externos viven en esta capa,** cada uno con capa anticorrupción: C-09 laboratorio, C-10 despacho de flota, C-11 planificación minera, C-12 ERP, C-13 telemetría de tres fabricantes, C-14 historiador de proceso.

### Capa 6 · Datos

- **Responsabilidad:** persistencia transaccional, analítica, documental, de series de tiempo y de archivos, según lo requiera el caso.
- **Exigencia mínima:** separación entre lo transaccional y lo analítico, cifrado en reposo, respaldo y retención declarados.
- **Cómo se materializa:** cinco naturalezas de persistencia, más el repositorio de consulta histórica. El detalle es materia del Subdocumento 5; aquí se declara la separación lógica y la razón de cada una.

| Naturaleza | Qué guarda | Por qué separada |
|---|---|---|
| Transaccional | Viajes, lotes, personas, habilitaciones, detenciones, plan versionado | Ninguna consulta analítica puede degradar el desempeño de la operación (RT-05.05) |
| Analítica | Indicadores por turno, conciliación, series de cumplimiento | Latencia de la capa analítica de 15 minutos (RT-05.29), incompatible con los umbrales de terreno |
| Series de tiempo | Telemetría de equipos, lecturas del historiador, mediciones ambientales | Volumen y patrón de escritura distintos del transaccional |
| Documental | Certificados de análisis, documentación de habilitación, reportes firmados | Versionado, metadato y búsqueda por contenido (RT-16.15) |
| Archivos | Evidencia fotográfica de inspecciones e incidentes, respaldos de grabación | Tamaño y ciclo de vida propios |
| Repositorio de consulta histórica | Lo que queda fuera de la ventana de migración: trazabilidad y embarques del sexto al décimo año, mantenimiento anterior a tres años, laboratorio anterior a cinco | Sólo lectura. Responde consultas con la marca «origen anterior al sistema». **No alimenta** conciliación, indicadores ni recálculo retroactivo (SUP-37, RN-28) |

- **Retención por tipo de registro, sin borrado anticipado (RN-28):** seguridad y salud ocupacional 10 años; ambientales y de monitoreo 10; trazabilidad de mineral y embarques 10; operacionales 5; tributarios 7, consultados en el ERP que no se reemplaza.

### Capa 7 · Seguridad transversal

- **Responsabilidad:** identidad, autorización, gestión de secretos, cifrado, registro de auditoría y detección.
- **Exigencia mínima:** aplicada a **todas las capas, no como capa perimetral única**.
- **Cómo se materializa:** tres mecanismos de identidad sobre un solo maestro de personas, porque los perfiles operacionales son irreductibles entre sí:

| Población | Mecanismo | Fundamento |
|---|---|---|
| Personal propio (1.780 personas) | Federación con el directorio corporativo del CLIENTE | Restricción n.º 9, interpretada como aplicable al personal propio (SUP-08) |
| Personas contratistas (2.400 en régimen, hasta 8.500 en detención mayor, de unas 90 empresas) | Credencial física emitida en la acreditación; en terreno basta el toque sobre dispositivo inventariado; fuera del ciclo productivo se exige además PIN numérico. Administración delegada a cada empresa desde el portal | RT-12.12 exige mecanismo autoservido. Provisionar hasta 8.500 identidades corporativas con 11 personas de TI es inviable (SUP-08) |
| Persona operadora en equipo compartido | Credencial de proximidad leída en cabina en el relevo, sin login ni contraseña escrita en terreno | RT-12.11 fija el perfil operacional y prohíbe la contraseña alfanumérica en terreno (SUP-04) |

- **Dato personal sensible, Ley N.º 21.719:** habilitación, salud ocupacional, control de fatiga y control de alcohol y drogas se cifran **a nivel de campo**, y se audita **la consulta además de la modificación** (RN-29). El acceso de emergencia queda registrado con persona, motivo y período de uso.
- **Segregación TI/TO:** la lectura del historiador de proceso es por OPC UA con segregación conforme a IEC 62443. **No se escribe en la red de control** en ninguna circunstancia (restricción n.º 3, componente C-14).

### Capa 8 · Observabilidad transversal

- **Responsabilidad:** métricas, registros y trazas distribuidas correlacionadas.
- **Exigencia mínima:** instrumentación conforme a OpenTelemetry, cobertura de nube y on-premise **sin puntos ciegos**.
- **Cómo se materializa:** una sola plataforma de observabilidad con alertamiento unificado para nube y faena (RT-03.16 y RT-14.01). El nodo de borde emite su propia telemetría de salud y, mientras está sin enlace, la acumula localmente junto con los eventos de negocio: la observabilidad se degrada igual que el resto y se reconcilia igual que el resto.
- **Indicador propio del caso:** la observabilidad debe exhibir, por sitio, las horas acumuladas sin enlace y el retraso de reconciliación pendiente. Es la evidencia con que se verifica el criterio de aceptación n.º 13.

---

## 5. Los veintidós componentes ubicados en las capas

Los componentes lógicos son los mismos C-01 a C-22 del catálogo y de la matriz de trazabilidad. La columna de etapa proviene de la matriz v1.1.

| Capa | Componentes | Etapa |
|---|---|---|
| Presentación | C-03 Captura en terreno · C-06 Portal de empresas contratistas · C-07 Portal de trazabilidad de lote · C-16 Vista única de equipo · C-19 Analítica, indicadores y reportería · C-20 Administración, parametrización y auditoría (interfaz) | C-07 en Etapa 2; el resto en Etapa 1, C-16 con versión completa en Etapa 2 |
| Borde y exposición | Parte de C-22 Plataforma base | Etapa 1 |
| Puerta de enlace de servicios | Parte de C-22 Plataforma base | Etapa 1 |
| Servicios de negocio | C-01 Núcleo de trazabilidad · C-02 Plan minero versionado · C-05 Identidad y acreditación · C-08 Motor de conciliación · C-15 Detenciones y productividad · C-17 Ambiental, huella de carbono y comunidad · C-18 Seguridad y salud ocupacional · C-20 Administración y auditoría (lógica) | C-17 y la mayor parte de C-18 en Etapa 2; el resto en Etapa 1 |
| Integración y eventos | C-04 Nodo de borde · C-09 Laboratorio · C-10 Despacho de flota · C-11 Planificación minera · C-12 ERP · C-13 Telemetría de fabricante · C-14 Historiador de proceso | C-13 mayoritariamente en Etapa 2; el resto en Etapa 1 |
| Datos | C-21 Migración de datos históricos · almacenes declarados en el apartado 4, capa 6 | C-21 en Etapa 1, salvo mantenimiento y monitoreo ambiental que viajan en Etapa 2 con RF-MIG-008 |
| Seguridad transversal | C-05 (aplicada a todas las capas) · parte de C-22 | Etapa 1 |
| Observabilidad transversal | Parte de C-22 | Etapa 1 |

**Nota sobre C-22.** Concentra 35 requerimientos, el componente más cargado del catálogo. No es un componente de negocio: es la plataforma base —infraestructura, seguridad, identidad técnica y observabilidad— y por eso aparece en cuatro capas. En el diagrama se representa como banda transversal, no como caja dentro de una columna.

**Nota sobre C-20.** Aparece en dos capas porque tiene interfaz de administración y lógica de parametrización. Es deliberado: el RT-16.02 exige que las reglas sean configurables desde la interfaz, y el RT-16.03 exige aprobación de un segundo perfil para todo cambio con impacto operacional.

---

## 6. Límites de contexto (RT-02.02)

Nueve contextos. Cada uno tiene lenguaje propio, un agregado raíz y una frontera explícita de lo que **no** le pertenece. La frontera es la parte que se evalúa: un contexto sin exclusiones declaradas no es un límite, es una etiqueta.

### 6.1 Contexto Mineral

- **Componentes:** C-01, C-02, C-08
- **Agregado raíz:** el **viaje**. Los demás agregados —aporte a stock, retoma, lote de chancado, lote comercial— son agregaciones derivadas con relación padre-hijo hacia los viajes que los componen y su proporción.
- **Lenguaje propio:** polígono, viaje, pala de origen, stock, aporte, retoma, ley ponderada, lote de chancado, lote comercial, metal contenido, punto de medición, diferencia de conciliación.
- **No le pertenece:** la identidad de la persona que operó el equipo (es del contexto Personas, que le entrega un identificador ya validado); el resultado de laboratorio (es del contexto Calidad); la cifra contable oficial (la emite el ERP, restricción n.º 4).
- **Relación con vecinos:** consume identidad del contexto Personas y ley medida del contexto Calidad, ambos como **conformista** —acepta el modelo del otro sin traducirlo—. Publica eventos de trazabilidad al contexto Analítico y al Comercial.
- **Regla dura:** ninguna diferencia de conciliación se cierra como «no explicada». La diferencia residual queda atribuida a un punto de medición de la cadena (RN-06).

### 6.2 Contexto Terreno

- **Componentes:** C-03, C-04
- **Agregado raíz:** el **evento de terreno**, con identificador generado en el dispositivo.
- **Lenguaje propio:** dispositivo, sesión de operador, relevo, evento local, reloj monótono, sello de tiempo de origen, deriva de reloj, cola de sincronización, reconciliación.
- **No le pertenece:** la interpretación de negocio del evento. El terreno registra que un camión descargó en un punto a una hora; no calcula la ley ni el cumplimiento del plan.
- **Relación con vecinos:** es **aguas arriba** de todo el sistema. Publica eventos; no consulta lógica de negocio para poder operar sin enlace. La única dependencia de lectura es la caché local de credenciales y habilitaciones vigentes que le entrega el contexto Personas.
- **Regla dura de resolución de conflictos (RN-30):** para hechos observados en terreno **prevalece el evento de terreno**; para datos maestros prevalece el registro central. En el relevo sin enlace, la sesión saliente se cierra y la entrante se abre localmente, **sin fusión de eventos**.

### 6.3 Contexto Personas

- **Componentes:** C-05, C-06
- **Agregado raíz:** la **persona habilitada**, con su conjunto de habilitaciones vigentes.
- **Lenguaje propio:** persona, empresa contratista, credencial, habilitación, examen preocupacional, curso, inducción, acreditación, vencimiento, revocación, enrolamiento.
- **No le pertenece:** el dato laboral y de remuneraciones, que vive en el maestro del ERP y no se duplica (exclusión del Capítulo 11). La solución consume el maestro; no lo sustituye.
- **Relación con vecinos:** es **aguas arriba** del contexto Mineral y del contexto Cumplimiento, a los que entrega identidad ya validada. Recibe del ERP el maestro de personal propio mediante adaptador.
- **Reglas duras:** una persona ingresa sólo con habilitación vigente, validada en línea en portería con consulta no superior a 1,5 segundos y alerta anticipada del vencimiento a la persona, a su empleador y al área de seguridad (RN-20). El acceso se revoca en no más de quince minutos desde el término de la relación con la faena (SUP-28). El equipo **nunca se detiene** por falta de identificación: el hueco es indicador gestionable, no bloqueo (restricción n.º 1, RN-22).

### 6.4 Contexto Calidad

- **Componentes:** C-09
- **Agregado raíz:** la **muestra** y su análisis.
- **Lenguaje propio:** punto de muestreo, muestra, análisis, ley medida, elemento deletéreo, remuestreo, laboratorio árbitro, certificado de análisis.
- **No le pertenece:** la decisión comercial de liberar un lote. El contexto Calidad publica que un análisis es válido y vigente; la máquina de estados del lote vive en el contexto Mineral.
- **Relación con vecinos:** el resultado del sistema de gestión de laboratorio es la **fuente de verdad de la ley medida** (RN-10). El modelo de bloques y el plan son estimaciones que se reconcilian contra ella, y la diferencia se publica por polígono como indicador de calidad del modelo.
- **Reglas duras:** ningún lote se libera a embarque sin análisis válido y vigente, y la liberación **no admite excepción manual** (RN-11). El muestreo obligatorio se ubica donde el material ya se detiene o donde lo opera un rol distinto del ciclo de mina, **nunca en el ciclo de pala o de camión** (RN-12). Volumetría: del orden de 71.000 muestras analizadas al año, con espera de resultado de 4 a 36 horas.

### 6.5 Contexto Activos

- **Componentes:** C-13, C-14, C-15, C-16
- **Agregado raíz:** el **equipo** y su línea de tiempo de estados.
- **Lenguaje propio:** equipo, estado, detención, causa, taxonomía, disponibilidad física, utilización efectiva, pauta de mantenimiento, alarma de fabricante.
- **No le pertenece:** la orden de trabajo de mantenimiento, que vive en el ERP; la operación autónoma de equipos, excluida del alcance.
- **Relación con vecinos:** consume telemetría de tres fabricantes a través de **adaptadores con capa anticorrupción, uno por fabricante**. Ese aislamiento es la mitigación del riesgo mayor de la propuesta: si un fabricante no cede acceso programático, cambia la fuente del dato y no el compromiso (SUP-10).
- **Reglas duras:** la detención se clasifica con taxonomía **cerrada y jerárquica de tres niveles, sin texto libre**. La clasificación es automática desde señales que el despacho, el plan y la telemetría ya generan. **El operador de equipo no clasifica nunca**; el supervisor resuelve sólo el residuo desde lista cerrada (RN-17). El turno no cierra con detenciones sin clasificar: las pendientes escalan al supervisor de la guardia siguiente (RN-18).

### 6.6 Contexto Cumplimiento

- **Componentes:** C-17, C-18
- **Agregado raíz:** el **compromiso** —ambiental, de seguridad o de comunidad— y su evidencia.
- **Lenguaje propio:** compromiso de la Resolución de Calificación Ambiental, punto de medición, frecuencia, destinatario, inspección, incidente, alerta de fatiga, factor de emisión, alcance 1, 2 y 3.
- **No le pertenece:** el dato operacional de origen, que produce el contexto Mineral o el de Activos. El cumplimiento **no mide, referencia**: cada cifra de un reporte se navega hasta su medición de origen.
- **Reglas duras:** todo reporte a la autoridad se genera desde el sistema y ningún reporte se construye en planilla al margen (RN-25). Los factores de emisión se conservan versionados y cada dato de actividad conserva su evidencia, de modo que un cálculo pasado se reproduce exactamente (RN-26).

### 6.7 Contexto Comercial

- **Componentes:** C-07, más la vista de exposición del contexto Mineral
- **Agregado raíz:** el **certificado derivado** de un lote embarcado.
- **Lenguaje propio:** lote embarcado, origen por sector, cadena de custodia, ley pagable, deducción por humedad, cargo de tratamiento y refinación, penalización por elemento deletéreo, liquidación.
- **No le pertenece, y es la frontera más importante del sistema:** la **ley por polígono no se expone en ningún caso**. Es información competitivamente sensible del yacimiento. Al comprador se le entrega origen por sector, fechas, stocks recorridos, análisis y cadena de custodia; nunca la cadena completa (RN-16, SUP-17). La publicación requiere aprobación del área comercial y toda consulta queda registrada.
- **Regla dura:** la liquidación se calcula sobre un **vector de elementos** —cobre, oro, plata y arsénico como elemento deletéreo— y no sobre la ley de cobre sola (RN-14). El contenido de arsénico proyectado se alerta **antes** de conformar el lote definitivo: es el control que ataca las cuatro penalizaciones de USD 1,9 millones.

### 6.8 Contexto Analítico

- **Componentes:** C-19
- **Agregado raíz:** el **indicador** con su definición versionada.
- **Lenguaje propio:** indicador, turno, línea base, meta, desfase, tablero.
- **No le pertenece:** originar dato. El contexto Analítico **sólo** consume eventos publicados por los demás. Si un indicador necesita un dato que nadie publica, la corrección es publicar el evento, no leer la base transaccional del vecino.
- **Regla dura:** el turno operacional es de doce horas, día de 08:00 a 20:00 y noche de 20:00 a 08:00, y todo indicador por turno se calcula sobre esa definición (RN-23).

### 6.9 Contexto Plataforma

- **Componentes:** C-20, C-21, C-22
- **Agregado raíz:** el **parámetro versionado** y el **asiento de auditoría**.
- **Lenguaje propio:** parámetro, regla versionada, vigencia temporal, perfil, rol, asiento de auditoría, dispositivo inventariado, ventana de intervención.
- **No le pertenece:** ninguna regla de negocio del caso. La plataforma **custodia** las reglas; no las decide.
- **Regla dura y transversal (RN-27):** toda regla de negocio y todo parámetro tienen vigencia temporal versionada, y un período anterior se recalcula con la versión que regía en ese momento. Es la propiedad que hace verificables sobre historia ya cerrada los criterios de aceptación n.º 3 y n.º 4.

### 6.10 Mapa de relaciones entre contextos

| Aguas arriba | Aguas abajo | Patrón de integración |
|---|---|---|
| Terreno | Mineral, Personas, Activos, Cumplimiento | Publicación de eventos. El productor no conoce a sus consumidores |
| Personas | Mineral, Terreno (caché de credenciales), Cumplimiento | Conformista. El consumidor acepta el modelo de identidad sin traducirlo |
| Calidad | Mineral, Comercial | Conformista sobre el resultado del laboratorio, que es fuente de verdad de la ley |
| Mineral | Comercial, Analítico, Cumplimiento | Publicación de eventos, más vista de exposición filtrada hacia Comercial |
| Sistemas del CLIENTE (ERP, despacho, planificación, historiador, telemetría) | Todos | **Capa anticorrupción por sistema.** El modelo externo nunca entra al núcleo |
| Plataforma | Todos | Núcleo compartido acotado: sólo parámetros versionados, perfiles y auditoría |

---

## 7. Modelo de dominio del negocio (RT-02.13)

### 7.1 Entidades principales y sus relaciones

| Entidad | Pertenece a | Relaciones esenciales |
|---|---|---|
| Polígono | Mineral | Atributo de origen del Viaje. No es agregado: hereda su propio error |
| Viaje | Mineral | Uno a uno con Pala de origen y Polígono; uno a muchos con Pesaje; destino a Stock o a Chancado |
| Pesaje | Mineral | Tres orígenes por viaje o por lote: pesaje de a bordo, báscula de camiones, balanza de alimentación. **Los tres se conservan** |
| Stock | Mineral | Uno a muchos con Aporte y con Retoma; mantiene ley ponderada vigente |
| Aporte a stock | Mineral | Conserva polígono de origen, tonelaje, ley y fecha |
| Retoma de stock | Mineral | Conserva la proporción con que cada polígono contribuyó |
| Lote de chancado | Mineral | Agregación de Viajes y Retomas |
| Lote comercial | Mineral / Comercial | Registro inmutable de eventos, con relación padre-hijo hacia los lotes que lo componen y su proporción |
| Embarque | Comercial | Uno a uno con Lote comercial; uno a muchos con Liquidación |
| Muestra | Calidad | Asociada a Punto de muestreo y a Lote o Viaje |
| Análisis | Calidad | Vector de elementos: Cu, Au, Ag, As. Estado de validez y vigencia |
| Plan minero (versión) | Mineral | Entidad **versionada e inmutable**; cada movimiento referencia la versión vigente en su instante |
| Persona | Personas | Uno a muchos con Habilitación y con Credencial |
| Habilitación | Personas | Tipo, fecha de vigencia, estado; dato sensible bajo Ley N.º 21.719 |
| Empresa contratista | Personas | Uno a muchos con Persona; administra su propia carga documental |
| Sesión de operador | Terreno / Personas | Persona, Equipo, apertura y cierre; abierta por proximidad en el relevo |
| Equipo | Activos | Uno a muchos con Detención, Pauta y Alarma |
| Detención | Activos | Causa de taxonomía cerrada de tres niveles; estado de clasificación |
| Compromiso | Cumplimiento | Frecuencia, punto de medición, destinatario, vencimiento |
| Medición ambiental | Cumplimiento | Origen navegable hasta el dato de actividad |
| Parámetro / Regla | Plataforma | Vigencia temporal versionada; autor, fecha y justificación |
| Evento de terreno | Terreno | Identificador generado en dispositivo; sello de tiempo y posición de origen |

### 7.2 Eventos de negocio que modifican el estado

El RT-02.13 exige declarar los eventos, no sólo las entidades. Son el contrato real entre contextos.

**Cadena del mineral:** `ViajeRegistrado` · `PesajeCapturado` · `AporteAStockRegistrado` · `RetomaDeStockRegistrada` · `LoteDeChancadoConformado` · `LoteComercialConformado` · `LoteLiberado` · `LoteBloqueado` · `EmbarqueDespachado` · `DevoluciónDeConcentradoRegistrada` · `LiquidaciónRecibida`

**Calidad:** `MuestraTomada` · `AnálisisPublicado` · `AnálisisRechazado` · `RemuestreoSolicitado` · `ArbitrajeResuelto` · `ArsénicoProyectadoSobreLímite`

**Personas:** `PersonaAcreditada` · `HabilitaciónPorVencer` · `HabilitaciónVencida` · `AccesoConcedido` · `AccesoDenegado` · `OperadorIdentificado` · `OperadorNoIdentificado` · `AccesoRevocado`

**Activos y productividad:** `DetenciónDetectada` · `DetenciónClasificadaAutomáticamente` · `DetenciónPendienteEscalada` · `AlarmaDeFabricanteRecibida`

**Plan y conciliación:** `PlanPublicado` (nueva versión inmutable) · `CumplimientoDeTurnoCalculado` · `ConciliaciónCerrada` · `DiferenciaAtribuidaAPuntoDeMedición`

**Cumplimiento:** `InspecciónRegistrada` · `IncidenteRegistrado` · `AlertaDeFatigaRecibida` · `AcciónDeFatigaDocumentada` · `ReporteGenerado` · `HuellaDeCarbonoCalculada`

**Terreno y plataforma:** `EnlacePerdido` · `EnlaceRestablecido` · `ReconciliaciónCompletada` · `ConflictoResuelto` · `ParámetroModificado` · `ReglaVersionada`

### 7.3 La propiedad que sostiene el criterio de aceptación n.º 1

`LoteComercialConformado` conserva la relación padre-hijo hacia los lotes que lo componen y su proporción; cada uno hacia sus retomas y viajes; cada viaje hacia su polígono, pala, fecha y operador identificado. Reconstruir el origen de un lote embarcado es recorrer ese árbol, no recalcular una estimación. De ahí sale el compromiso de **menos de dos horas al mes 15**, contra las cuatro a seis semanas actuales.

---

## 8. Interfaces

### 8.1 Interfaces externas

| Sistema del CLIENTE | Sentido | Protocolo y patrón | Frecuencia | Componente |
|---|---|---|---|---|
| ERP corporativo | Bidireccional. Recibe tonelaje, ley, metal contenido, consumos y movimientos en cada cierre; entrega el maestro de personal propio | Servicio síncrono documentado en OpenAPI 3.1, más archivo por lote en el cierre | En cada cierre contable; maestro de personas diario | C-12 |
| Despacho de flota (contrato hasta 2031) | Lectura principal del ciclo mina: pala de origen, destino, tonelaje, posición y hora. Escritura sólo de la marca de estado que el propio despacho admita | Adaptador con capa anticorrupción; ingesta continua de eventos | Continua | C-10 |
| Sistema de gestión de laboratorio | **Bidireccional.** Envía solicitud de análisis y recibe resultado validado | Adaptador con capa anticorrupción; evento en AsyncAPI 2.6 | Continua; del orden de 71.000 muestras al año | C-09 |
| Software de planificación minera | Lectura de la versión vigente del plan. **No** interviene el modelo de bloques | Adaptador; ingesta por publicación de versión | 3 a 4 veces por semana, según la frecuencia real de cambio del plan | C-11 |
| Historiador de proceso de planta | **Sólo lectura.** Nunca se escribe en la red de control | OPC UA con segregación conforme a IEC 62443 | Continua | C-14 |
| Tres portales de telemetría de fabricante | Lectura de estado, posición, horómetro y alarmas | **Un adaptador por fabricante**, cada uno con capa anticorrupción y reemplazable | Latencia de ingesta declarada de 5 minutos (SUP-30) | C-13 |
| Sistema de control de fatiga | Lectura de alertas emitidas | Adaptador; dato personal sensible, cifrado a nivel de campo | Continua | C-18 |

**Fundamento del patrón.** Los siete usan capa anticorrupción y adaptador reemplazable (RT-02.14). No es sofisticación: el Capítulo 5 del caso declara que el CLIENTE **no tiene documentadas** las interfaces, las versiones ni las condiciones contractuales de acceso de sus propios sistemas. Un adaptador aislado permite descubrir la interfaz real en ejecución sin propagar el hallazgo al núcleo.

### 8.2 Interfaces internas

**Síncronas**, sólo donde la latencia lo exige y el consumidor no puede continuar sin respuesta:

| Consumidor | Proveedor | Para qué | Umbral |
|---|---|---|---|
| Portería (C-05) | Maestro de habilitaciones (C-05) | Validar habilitación vigente | Consulta no superior a 1,5 segundos (RN-20) |
| Cabina (C-03) | Caché local de credenciales (C-04) | Abrir sesión de operador por proximidad | 1 segundo desde la lectura, sin retirar el guante (SUP-29) |
| Vista única (C-16) | C-10 despacho, C-12 ERP, C-13 telemetría | Reunir estado, historial, pautas y posición | Cuadro completo en menos de los 15 a 20 minutos actuales |

**Asíncronas por evento**, que es el caso por omisión. Todo hecho de terreno, toda conformación de lote, toda clasificación de detención y todo cálculo de indicador viajan por el bus. Consecuencia de diseño: **ningún componente de negocio consulta la base de datos de otro**. Si necesita un dato, se suscribe a su evento.

### 8.3 Contrato de interfaz

Servicios síncronos en OpenAPI 3.1 y flujos por evento en AsyncAPI 2.6 o superior, generados desde el código y mantenidos actualizados automáticamente (RT-05.16). Versionado semántico con compatibilidad hacia atrás y política de obsolescencia con preaviso mínimo de seis meses (RT-05.17). Toda operación de escritura expuesta a reintentos es idempotente, con clave de idempotencia declarada por el cliente y ventana de deduplicación documentada (RT-02.06).

---

## 9. Comportamiento sin enlace

El enlace de fibra tiene un único proveedor, con tres a seis cortes al año de cuatro a veinte horas. El respaldo satelital sostiene telefonía y correo, **no** la operación transaccional. La red inalámbrica del rajo tiene zonas de sombra permanentes que se degradan con el avance de la explotación. La restricción n.º 2 no admite excepciones.

### 9.1 Qué se ejecuta en el borde

Cuatro gabinetes de borde, más la sala técnica de faena: **portería, planta, depósito de relaves y acopio de puerto**. El acopio de puerto es el caso más expuesto: está en recinto de terceros y la conectividad la provee un operador sin acuerdo de nivel de servicio con la compañía (SUP-19), de modo que se diseña para no depender de él.

Cada dispositivo de terreno y cada nodo de borde mantiene: registro local firmado y cifrado a nivel de campo; reloj monótono con sello de tiempo propio y registro de deriva; identificador de evento generado localmente; cola de sincronización idempotente y reintentable; caché de credenciales y habilitaciones vigentes.

### 9.2 Umbral comprometido

**48 horas continuas** de operación y registro sin enlace, y sincronización posterior en **no más de 4 horas sin intervención manual**. El umbral del caso excede el mínimo transversal del RT-03.10, que fija 24 horas o el mayor que fije el caso. Verificación: corte controlado de 48 horas **con un cambio de turno dentro de la ventana**; los eventos reconciliados deben coincidir exactamente con los generados.

### 9.3 Funciones no disponibles en modo desconectado (RT-03.13)

El RT-03.13 exige declarar qué funciones **no** operan durante un corte y qué procedimiento manual las suple, y califica la ausencia de esta declaración como **observación grave**. En el Formulario T-12 v1.2 este requisito figura como «Cumple parcialmente» precisamente porque la lista faltaba. Se declara aquí y con esto queda cerrado.

| Función no disponible sin enlace | Por qué | Procedimiento manual que la suple |
|---|---|---|
| Acreditación de una persona contratista nueva | Requiere validación documental contra el maestro central y contra el ERP | Se suspende el alta. La persona ya acreditada **sí** ingresa: la portería valida contra la caché local |
| Alta o modificación de datos maestros: personas, polígonos, puntos de muestreo, taxonomía de detenciones, parámetros | El registro central prevalece para datos maestros (RN-30). Permitir alta local crearía dos maestros | Se posterga al restablecimiento. Los eventos que referencian un maestro inexistente quedan en cuarentena y se resuelven al reconciliar |
| Conciliación metalúrgica y cierre del mes | Requiere las tres mediciones y el resultado de laboratorio consolidados | Se posterga. La ventana de cierre contable ya está protegida por RN-24, que prohíbe intervenciones |
| Liberación de un lote a embarque | Exige análisis válido y vigente publicado por el laboratorio, que es sistema externo | **No se libera ningún lote.** Es regla dura de RN-11 y no admite excepción manual |
| Publicación al portal del comprador y a la autoridad | Son destinos externos | Se posterga. Ningún reporte se construye en planilla al margen del sistema (RN-25) |
| Tableros analíticos y recálculo de indicadores | La capa analítica está separada de la transaccional por diseño | El supervisor opera con el estado local del turno. Los indicadores se recalculan al reconciliar |
| Alertas de vencimiento de habilitación a la persona, al empleador y al área de seguridad | Requiere canal de notificación externo | La portería sigue advirtiendo en el momento del ingreso contra la caché local; la alerta anticipada se emite al restablecerse |
| Ingesta de telemetría de fabricante | Depende de tres portales web externos | La detención se sigue detectando desde las señales del despacho que el nodo de borde recibe localmente |

**Lo que sí opera sin enlace, íntegro:** registro del ciclo mina —carguío, transporte, descarga y pesaje—; identificación del operador por proximidad y apertura y cierre de sesión en el relevo; validación de acceso en portería contra caché; registro de inspecciones, incidentes y detenciones; toma de muestra y su registro; movimiento de stock.

### 9.4 Degradación elegante (RT-02.09)

Ante la indisponibilidad de un componente no crítico el sistema continúa en modo reducido e **informa la degradación a la persona usuaria**; nunca falla de forma total. La aplicación de terreno muestra de manera permanente su estado de enlace, las horas acumuladas sin sincronizar y la cantidad de eventos pendientes. Es requisito de usabilidad y también de confianza: el operador que no sabe si su registro se guardó deja de registrar, y ahí se pierde el criterio de aceptación n.º 14.

---

## 10. Propiedades de ejecución exigidas

| Requisito | Cómo se satisface |
|---|---|
| **RT-02.05** Servicios sin estado | La capa de servicios de negocio no guarda estado en memoria. Sesión y estado de proceso residen en almacenes externos de alta disponibilidad. Única excepción declarada y fundada: el estado local del contexto Terreno, que es un registro de eventos pendientes de reconciliación, no estado de sesión de servicio |
| **RT-02.06** Idempotencia | Toda escritura reintentable lleva clave de idempotencia. En terreno la clave es el identificador de evento generado en el dispositivo. Ventana de deduplicación documentada por tipo de evento |
| **RT-02.07** Entrega al menos una vez | Bus con persistencia, deduplicación en el consumidor y orden garantizado dentro del agregado —lote, sesión de operador, equipo— cuando el proceso lo exige. No se pretende orden global |
| **RT-02.08** Resiliencia | Reintento con retroceso exponencial y variación aleatoria; cortacircuitos por adaptador externo; mamparos de aislamiento entre el tráfico de portales y el operacional; límite de tasa; **tiempo de espera explícito en toda llamada remota**, sin excepción |
| **RT-02.09** Degradación elegante | Apartado 9.4 |
| **RT-02.10** Escalado horizontal | Las capas de aplicación e integración escalan automáticamente. Umbrales, límites superiores y costo asociado se declaran en la oferta económica. El peak dimensionante es el cambio de turno: 45 a 60 minutos, 200 transacciones por segundo, con degradación no superior al 20 % del tiempo de respuesta (SUP-31) |
| **RT-02.12** Replicación | Parametrización y multi-tenencia en C-20, sin rediseño arquitectónico. El caso lo pide con destino conocido: el Capítulo 13.2 menciona la evaluación de una operación en el extranjero hacia 2030 |
| **RT-02.14** Arquitectura evolutiva | Capa anticorrupción frente a los siete sistemas externos; abstracción de proveedor en la capa de datos y en la de identidad; estrangulamiento progresivo de las planillas de turno, que se retiran a medida que cada punto de captura entra en producción, no de una vez |

### 10.1 Volumetría que dimensiona la arquitectura

| Dimensión | Valor | Fuente |
|---|---|---|
| Movimiento total | 82 Mt al año; 21,5 Mt al año a planta | Capítulo 2.1 |
| Concentradora | 59.000 t al día | Capítulo 2.1 |
| Flota | 38 camiones de tres generaciones y dos fabricantes, 5 palas eléctricas, 9 perforadoras, 310 vehículos livianos | Capítulo 2.1 |
| Equipos con telemetría | 52 hoy, 70 proyectados, en tres portales distintos | Capítulos 4.7 y 14.1 |
| Personas | 1.780 propias, ~2.400 contratistas en régimen, hasta 8.500 en detención mayor | Capítulo 14.1 |
| Ingresos y salidas | ~2.900 diarios | Capítulo 14.1 |
| Muestras analizadas | ~71.000 al año | Capítulo 14.1 |
| Pozos de tronadura | ~46.000 al año | Capítulo 14.1 |
| Viajes de concentrado | ~12.400 al año | Capítulo 14.1 |
| Embarques | 31 al año, de ~12.000 t cada uno | Capítulo 2.1 |
| Sitios operacionales | 7, incluido acopio de 25.000 t en recinto portuario de terceros | Capítulo 3 |
| Operación | 24 horas, 365 días, dos turnos de 12 horas | Capítulo 2.4 |

### 10.2 Puntos únicos de falla que subsisten (RT-02.11)

El requisito exige declararlos y justificar por qué son aceptables, y advierte que **omitir la declaración cuando existen se evalúa como observación grave**. Subsisten cuatro, ninguno de ellos en la ruta del registro de terreno.

| Punto único de falla | Por qué subsiste | Por qué es aceptable | Mitigación |
|---|---|---|---|
| Enlace de fibra de un único proveedor entre faena y Antofagasta | La construcción de la red no está en el alcance; el respaldo satelital no sostiene tráfico transaccional | **La arquitectura no depende de él.** Es la razón de ser del diseño desconectado: 48 horas de autonomía cubren el peor corte histórico registrado, de 20 horas | Nodos de borde en cuatro sitios; sincronización idempotente |
| Sistema de gestión de laboratorio del CLIENTE | Es sistema del CLIENTE, no se reemplaza | Su caída no detiene la operación: detiene la **liberación** de lotes, que es el comportamiento correcto y exigido por RN-11 | Cortacircuito en el adaptador C-09; cola de solicitudes de análisis con reintento |
| ERP corporativo como emisor contable único | Restricción n.º 4: el ERP es el emisor único y no se crea una segunda verdad contable | Su indisponibilidad posterga la emisión contable, no el registro operacional | Cola persistente en C-12; ajuste de período anterior previsto por RN-05 |
| Balanza de alimentación de planta como fuente de verdad del tonelaje | Es una decisión de negocio deliberada (SUP-02), no una limitación técnica | Las otras dos mediciones se siguen registrando durante su indisponibilidad; la conciliación se recalcula al recuperarla | Báscula de camiones como punto de calibración; las tres mediciones se conservan siempre |

**Objetivos de recuperación** (SUP-32): para los componentes de terreno, RTO no superior a una hora y **RPO de cero eventos perdidos** para el registro operacional.

---

## 11. Vista de seguridad

| Frente | Decisión lógica |
|---|---|
| Identidad | Tres mecanismos sobre un maestro único, según apartado 4 capa 7. La federación corporativa aplica al personal propio; las personas contratistas se rigen por esquema autoservido |
| Autorización | Mínimo privilegio por rol (RF-IDE-018). La búsqueda global y toda exportación respetan el control de acceso (RT-16.27) |
| Segregación TI/TO | Lectura del historiador por OPC UA con segregación conforme a IEC 62443. No se escribe en la red de control (restricción n.º 3) |
| Dato personal sensible | Cifrado a nivel de campo para habilitación, salud ocupacional, control de fatiga y control de alcohol y drogas. Se audita la consulta además de la modificación (RN-29, Ley N.º 21.719) |
| Auditoría | Asiento inalterable de toda modificación y de toda consulta sensible, con retención no inferior a cinco años (RT-16.10) y hasta diez para los tipos que RN-28 declara |
| Aislamiento del portal externo | Mamparo de recursos y caché entre los portales expuestos a Internet y el núcleo transaccional. Umbral: cero degradación de los umbrales de terreno ante diez veces el tráfico medio del portal (RNF-DIS-010) |
| Recinto técnico | Control de acceso biométrico facial con AFIS de respaldo, exigido con carácter obligatorio por el RT-06.20. **No confundir** con la biometría de portería, que está excluida del Contrato mientras subsista la objeción sindical |

---

## 12. Trazabilidad de este subdocumento

### 12.1 Requisitos transversales que acredita

Este subdocumento es la sección de la propuesta que acredita, en el Formulario T-12, los requisitos que hasta la v1.2 figuraban en blanco por inexistencia del entregable:

**RT-02.01** apartados 4 y 14 · **RT-02.02** apartados 3.3 y 6 · **RT-02.03** apartado 1.1 · **RT-02.04** apartados 1.2 y 15 · **RT-02.05** apartado 10 · **RT-02.06** apartados 8.3 y 10 · **RT-02.07** apartado 10 · **RT-02.08** apartado 10 · **RT-02.09** apartados 9.4 y 10 · **RT-02.10** apartado 10 · **RT-02.11** apartado 10.2 · **RT-02.13** apartado 7 · **RT-02.14** apartados 8.1 y 10 · **RT-03.13** apartado 9.3, que pasa de «Cumple parcialmente» a «Cumple» · **RT-05.16 y RT-05.17** apartado 8.3 · numeral 2.3 apartado 3.

### 12.2 Componente por criterio de aceptación del Capítulo 18

| Criterio | Qué se compromete | Componentes que lo sostienen |
|---|---|---|
| n.º 1 | Origen de cualquier lote embarcado con evidencia en menos de 2 horas, mes 15 | C-01, C-03, C-04, C-09, C-07 |
| n.º 2 | Conciliación cerrada en no más de 3 días hábiles, mes 19 | C-08, C-01, C-09, C-12 |
| n.º 3 | Diferencia bajo 2 % al mes 21, con 100 % atribuido a un punto | C-08, C-01, C-09 |
| n.º 4 | Cumplimiento del plan al inicio del turno siguiente, contra la versión vigente | C-02, C-01, C-19 |
| n.º 5 | 100 % de detenciones clasificadas antes del cierre del turno siguiente, mes 15 | C-15, C-10, C-13 |
| n.º 6 | Estado, historial, alarmas y pautas en una sola vista. Base mes 15, completa mes 21 | C-16, C-10, C-12, C-13 |
| n.º 7 | Cero rechazos en portería por documentación vencida advertible, mes 15 | C-05, C-06, C-04 |
| n.º 8 | Acreditación de contratista en menos de 48 horas, mes 15 | C-06, C-05 |
| n.º 9 | 100 % de registros de seguridad nacidos digitales, mes 15 | C-03, C-05, C-18 |
| n.º 10 | 100 % de alertas de fatiga con acción documentada, mes 21 | C-18 |
| n.º 11 | 100 % de reportes ambientales generados desde el sistema, mes 21 | C-17, C-19 |
| n.º 12 | Huella de carbono mensual auditable, mes 18 | C-17, C-01, C-14 |
| n.º 13 | 48 horas sin enlace y sincronización en 4 horas, desde marcha blanca | C-04, C-03, C-22 |
| n.º 14 | 90 % de eventos registrados sin intervención del supervisor, mes 15 | C-03, C-05 |

---

## 13. Qué falta y de qué subdocumento proviene

| Qué falta | De quién | Insumo exacto |
|---|---|---|
| Producto y versión concretos de cada componente C-xx | Subdoc. 4.2 | El numeral 1.5 transversal advierte que declarar «Cumple» sin individualizar producto y versión equivale a no declarar, y la Comisión califica el requisito como **no acreditado**. Es el pendiente de mayor riesgo de puntaje que este subdocumento no puede cerrar solo |
| Tabla de emplazamiento componente por componente, nube y faena, con justificación | Subdoc. 4.2 | Exigida por el Artículo 16.2. Determina qué se sostiene en faena durante el corte de fibra |
| Especificación de la sala técnica y de los cuatro gabinetes de borde | Subdoc. 4.2 | Sitios, dimensionamiento y condiciones. La sala actual de 40 m² no cumple el estándar del Capítulo 6 |
| Especificación del hardware de terreno | Subdoc. 4.2, Formulario T-11 | Referencia, cantidad y características por sitio (RT-08.10) |
| Modelo de datos físico, linaje y catálogo | Subdoc. 5 | Debe soportar las entidades y eventos del apartado 7 de este subdocumento |
| Paquetes de la EDT | Informe 2 | Las 228 filas de la matriz de trazabilidad dicen «Se define en el Informe 2». La columna existe vacía |

**Responsable de la consolidación.** Antes de la entrega, el equipo de este subdocumento verifica que los nombres de componente, los nombres de sitio y la lista de funciones degradadas del apartado 9.3 coincidan **literalmente** entre los Subdocumentos 3, 4.1, 4.2 y 5.

---

## 14. Especificación del diagrama de arquitectura lógica

Este apartado no contiene el diagrama: contiene su especificación literal, para que quien lo dibuje no invente nombres. Representación por **columnas verticales por capa**, de izquierda a derecha en el orden en que viaja una transacción.

**Cinco columnas principales, en este orden:**

1. **Actores** — Persona operadora de pala · Persona operadora de camión · Persona operadora de perforadora · Supervisor de turno · Despachador · Portería · Persona que toma la muestra · Laboratorio · Metalurgia · Superintendencia de Planificación Minera · Jefatura de Mantenimiento · Área de Seguridad y Salud Ocupacional · Área de Sustentabilidad · Área comercial · Empresa contratista · Comprador · Autoridad ambiental · Centro Integrado de Operaciones · Administrador del CLIENTE
2. **Presentación** — C-03 Captura en terreno · C-06 Portal de empresas contratistas · C-07 Portal de trazabilidad de lote · C-16 Vista única de equipo · C-19 Tableros e indicadores · C-20 Consola de administración
3. **Servicios de negocio**, agrupados por límite de contexto, con el nombre del contexto rotulado sobre el grupo — Mineral: C-01, C-02, C-08 · Personas: C-05 · Activos: C-15, C-16 · Cumplimiento: C-17, C-18 · Analítico: C-19 · Plataforma: C-20
4. **Integración y eventos** — C-04 Nodo de borde · C-09 Laboratorio · C-10 Despacho de flota · C-11 Planificación minera · C-12 ERP · C-13 Telemetría, tres adaptadores · C-14 Historiador OPC UA
5. **Datos** — Transaccional · Analítica · Series de tiempo · Documental · Archivos · Repositorio de consulta histórica · C-21 Migración

**Dos bandas transversales** que cruzan las cinco columnas, no cajas dentro de una: **Seguridad transversal** (C-05 aplicada a todas las capas, más la parte de C-22) y **Observabilidad transversal** (parte de C-22).

**Dos bandas de exposición** entre Actores y Presentación, que aplican **sólo** a los actores externos —Empresa contratista, Comprador, Autoridad—: **Borde y exposición** y **Puerta de enlace de exposición**. Los actores internos de terreno entran por la **Puerta operacional**, que se dibuja entre Presentación y Servicios de negocio.

**Marco punteado rotulado «Opera sin enlace, 48 h»** que encierra C-03, C-04 y la caché de credenciales.

**Bloque externo a la derecha, rotulado «Sistemas del CLIENTE que no se reemplazan»** — ERP corporativo · Despacho de flota · Sistema de gestión de laboratorio · Software de planificación minera · Historiador de proceso, sólo lectura · Tres portales de telemetría · Sistema de control de fatiga.

**Marcas obligatorias en el dibujo:** etiqueta de sentido en cada conector externo, con `sólo lectura` explícito en el historiador; rótulo `ACL` en los siete adaptadores; rótulo `Etapa 2` en C-07, C-13 y C-17; y las tres capacidades excluidas del Contrato **fuera** del marco del sistema, con la nota `Art. 72.º BB.AA.`.

**No incluir** en este diagrama columnas laterales de hardware, software ni servicios externos contratados: son decisiones tecnológicas del Subdocumento 4.2 y mezclarlas aquí es el riesgo que la propia pauta advierte.

---

## 15. Índice de decisiones de arquitectura (RT-02.04)

| ADR | Decisión | Alternativa descartada | Criterio |
|---|---|---|---|
| ADR-01 | Servicios de negocio modulares con despliegue independiente | Microservicios de grano fino; monolito único | 200 transacciones por segundo en peak y 11 personas de TI del CLIENTE (restricción n.º 10) |
| ADR-02 | El viaje de camión como agregado raíz de la trazabilidad | El polígono; el turno | El viaje ya se registra hoy en el despacho: no agrega captura nueva (restricción n.º 8) y es la granularidad más fina que responde con evidencia |
| ADR-03 | Nueve límites de contexto con capa anticorrupción hacia lo externo | Modelo canónico único compartido con los sistemas del CLIENTE | El CLIENTE no tiene documentadas las interfaces ni las versiones de sus sistemas (Capítulo 5) |
| ADR-04 | Bus de eventos como columna vertebral, no como accesorio | Integración punto a punto por servicio síncrono | La trazabilidad exige linaje del hecho; el punto a punto no lo conserva y no sobrevive al corte de enlace |
| ADR-05 | Identificador de evento generado en el dispositivo | Identificador asignado por el servidor al recibir | Sin identificador local, una reconexión parcial obliga a resolver duplicados a mano y se pierde el criterio n.º 13 |
| ADR-06 | Tres mecanismos de identidad sobre un maestro único | Federación corporativa estricta para toda la población | Provisionar hasta 8.500 identidades con 11 personas de TI es inviable; RT-12.12 exige mecanismo autoservido |
| ADR-07 | Dos puertas de enlace lógicas, operacional y de exposición | Puerta única | Perfiles de tráfico incompatibles: un ataque al portal del comprador no puede afectar el registro de terreno |
| ADR-08 | Separación transaccional / analítica desde el diseño | Base única con réplicas de lectura | RT-05.05 prohíbe que una consulta analítica degrade la operación; la capa analítica tolera 15 minutos de latencia y el terreno no |
| ADR-09 | Reglas y umbrales como parámetros versionados con vigencia temporal | Constantes de código con despliegue por cambio | Sin recálculo con la versión vigente en el período, los criterios n.º 3 y n.º 4 no se pueden verificar sobre historia cerrada |
| ADR-10 | Un adaptador por fabricante de telemetría | Adaptador único con modelo unificado | La propiedad contractual del dato no está resuelta para los tres; aislar permite que el compromiso de vista única no dependa del resultado de la negociación (SUP-10) |

---

## 16. Referencias

International Organization for Standardization. (2022). *ISO/IEC/IEEE 42010:2022 — Software, systems and enterprise: Architecture description*.

International Electrotechnical Commission. (2018). *IEC 62443 — Security for industrial automation and control systems*.

Ley N.º 21.719 de 2024. Regula la protección y el tratamiento de los datos personales y crea la Agencia de Protección de Datos Personales. 13 de diciembre de 2024. Diario Oficial de la República de Chile.

Decreto Supremo N.º 132 de 2002 [Ministerio de Minería]. Aprueba Reglamento de Seguridad Minera. 7 de febrero de 2004. Diario Oficial de la República de Chile.

Parker, H. M. (2012). Reconciliation principles for the mining industry. *Mining Technology, 121*(3), 160-176.
