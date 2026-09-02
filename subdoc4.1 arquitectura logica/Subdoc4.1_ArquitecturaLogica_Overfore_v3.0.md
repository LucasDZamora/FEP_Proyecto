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
| Versión | 3.0 — 1 de septiembre de 2026. Tercera emisión: incorpora la vista de seguridad completa (apartado 11), la observabilidad y los niveles de servicio (apartado 12), la capacidad, configuración y despliegue sin interrupción (apartado 13) y los canales y la movilidad (apartado 16); corrige las inconsistencias internas detectadas en la revisión cruzada y alinea las citas de origen con la convención del catálogo y de la matriz. |

Este subdocumento acredita el Capítulo 2 del documento transversal —RT-02.01 a RT-02.14—, que hasta la v1.2 del Formulario T-12 figuraba en blanco por no existir todavía este entregable, y los requisitos de otros capítulos que enumera el apartado 12.1. No repite el problema, que se desarrolla en el Subdocumento 2, ni el alcance, que se desarrolla en el Subdocumento 3. El emplazamiento físico de cada componente se desarrolla en el Subdocumento 4.2 y el modelo de datos en el Subdocumento 5.

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

Cada decisión de este subdocumento está respaldada por un ADR fechado con la alternativa escogida, las descartadas y el criterio de decisión. El índice de ADR está en el apartado 18. El registro es entregable contractual y se actualiza durante toda la ejecución.

### 1.3 Cómo debe leerse el diagrama que acompaña a este texto

Este subdocumento es la especificación textual completa. El diagrama de la arquitectura lógica que exige el RT-02.01 se dibuja a partir del apartado 17, que entrega la lista literal de columnas, cajas y conectores. Se adopta la representación por **columnas verticales por capa**, de izquierda a derecha en el orden en que viaja una transacción, con flechas de color por actor y su leyenda.

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

La capa de servicios de negocio se organiza en **diez servicios de grano medio**, desplegables de forma independiente, sin estado y con contrato versionado. No se subdivide en microservicios de grano fino.

Los diez son C-01, C-02, C-05, C-08, C-15, C-16, C-17, C-18, C-19 y C-20, y se reparten en **seis** de los nueve límites de contexto del apartado 6. Los otros tres contextos no tienen servicio propio en esta capa, y es deliberado: la lógica del **Contexto Terreno** vive en el borde (C-04) y en el dispositivo (C-03) porque debe operar 48 horas sin enlace; la del **Contexto Calidad** vive en el sistema de gestión de laboratorio del CLIENTE, que no se reemplaza y al que sólo se accede por adaptador (C-09); y la del **Contexto Comercial** es una vista de exposición filtrada sobre el núcleo de trazabilidad, no un servicio con estado propio. Un contexto sin servicio en la capa 4 no es un contexto vacío: es un contexto cuya lógica se ejecuta en otra capa, y el apartado 6 declara cuál.

Cifras que sostienen la decisión:

| Dimensión | Valor declarado | Fuente |
|---|---|---|
| Transacciones por segundo en régimen | 40 | SUP-31 |
| Transacciones por segundo en peak (cambio de turno) | 200 | SUP-31 |
| Equipo de TI del CLIENTE que recibe la operación | 11 personas | Restricción n.º 10 |
| Sitios operacionales a cubrir | 7, incluido un acopio en recinto portuario de terceros, con capacidad de incorporar un octavo sin rediseño | Numeral 14.1. El Capítulo 3 se contradice consigo mismo —el texto dice seis emplazamientos y la tabla enumera siete— y por eso RNF-CAP-011 fija el dato duro en el numeral 14.1 y la Consulta N.º 3 lo somete al CLIENTE |
| Duración de la Operación tras la implementación | 36 meses, del mes 21 al 56 | Artículo 17.º |

Doscientas transacciones por segundo en peak no justifican descomposición fina. Once personas no pueden operar una malla de servicios al término del Contrato. La decisión se toma por la restricción n.º 10 antes que por preferencia técnica.

### 3.2 Alternativa descartada: microservicios de grano fino por capacidad de negocio

| Criterio del numeral 2.3 | Microservicios de grano fino | Servicios modulares (escogido) |
|---|---|---|
| Complejidad operacional | Malla de servicios, descubrimiento, trazas distribuidas sobre decenas de procesos, gestión de versiones cruzadas | Diez unidades desplegables, trazas correlacionadas sobre un número acotado de saltos |
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
- **Contrato de interfaz:** servicios síncronos documentados en OpenAPI 3.1 y flujos dirigidos por eventos en AsyncAPI 3.0, generados desde el código (RT-05.16). Versionado semántico con compatibilidad hacia atrás y obsolescencia con preaviso mínimo de seis meses (RT-05.17). La autenticación entre sistemas es OAuth 2.1 con credenciales de cliente o autenticación mutua TLS, y queda prohibida la clave estática en la ruta de la dirección web (RT-05.18).

### Capa 4 · Servicios de negocio

- **Responsabilidad:** lógica del proceso de negocio del caso, organizada en módulos con límites de contexto explícitos.
- **Exigencia mínima:** sin estado, desplegables de forma independiente, con contratos versionados y compatibilidad hacia atrás.
- **Cómo se materializa:** diez servicios de grano medio —C-01, C-02, C-05, C-08, C-15, C-16, C-17, C-18, C-19 y C-20— repartidos en seis de los nueve límites de contexto del apartado 6, según el reparto que justifica el apartado 3.1. Sin estado conforme al RT-02.05: el estado de sesión y el estado de proceso residen en almacenes externos de alta disponibilidad, nunca en memoria del servicio.
- **Regla que gobierna esta capa:** los umbrales, plazos, tolerancias y catálogos son **parámetros de administración versionados, no constantes de código** (RN-27, RT-16.02). Un período anterior se recalcula con la versión de regla que regía entonces. Sin esa propiedad, los criterios de aceptación n.º 3 y n.º 4 no se pueden verificar sobre historia ya cerrada.

### Capa 5 · Integración y eventos

- **Responsabilidad:** comunicación asíncrona, desacoplamiento, orquestación y coreografía de procesos entre módulos y con sistemas externos.
- **Exigencia mínima:** bus o intermediario de mensajería con persistencia, cola de mensajes fallidos, reintento y deduplicación.
- **Cómo se materializa:** el bus de eventos es la **columna vertebral de la trazabilidad**, no un accesorio de integración. Todo hecho de terreno entra al sistema como evento con identificador generado en el dispositivo de origen, lo que hace la reconciliación idempotente y reintentable (RN-30, RT-02.06). Entrega al menos una vez con deduplicación en el consumidor, y orden garantizado dentro del agregado cuando el proceso lo exige (RT-02.07): el orden importa dentro de un lote y dentro de una sesión de operador, no globalmente.
- **Los siete adaptadores externos viven en esta capa,** cada uno con capa anticorrupción y reemplazable por separado: **C-09** — Integración con el sistema de gestión de laboratorio · **C-10** — Integración con el sistema de despacho de flota · **C-11** — Integración con el software de planificación minera · **C-12** — Integración con el ERP corporativo · **C-13** — Adaptadores de telemetría de fabricante con capa anticorrupción, uno por cada uno de los tres fabricantes · **C-14** — Lectura del historiador de proceso con segregación TI/TO · y el **adaptador del sistema de control de fatiga**, que es la única pieza de integración sin componente propio en el catálogo: pertenece a **C-18** — Seguridad y salud ocupacional, pero **se ejecuta en esta capa** y no en la de servicios de negocio, porque transporta dato personal sensible bajo la Ley N.º 21.719 y queda aislado tras la misma capa anticorrupción que los demás.

### Capa 6 · Datos

- **Responsabilidad:** persistencia transaccional, analítica, documental, de series de tiempo y de archivos, según lo requiera el caso.
- **Exigencia mínima:** separación entre lo transaccional y lo analítico, cifrado en reposo, respaldo y retención declarados.
- **Cómo se materializa:** cinco naturalezas de persistencia, más el repositorio de consulta histórica. El detalle es materia del Subdocumento 5; aquí se declara la separación lógica y la razón de cada una.

| Naturaleza | Qué guarda | Por qué separada |
|---|---|---|
| Transaccional | Viajes, lotes, personas, habilitaciones, detenciones, plan versionado | Ninguna consulta analítica puede degradar el desempeño de la operación (RT-05.05) |
| Analítica | Indicadores por turno, conciliación, series de cumplimiento | El caso fija tres latencias distintas para la capa analítica (RT-05.29): indicadores operacionales de turno no superior a 15 minutos, indicadores de gestión y de cumplimiento no superior a 4 horas, y conciliación metalúrgica cerrada en no más de 3 días hábiles. La más estricta de las tres es incompatible con los umbrales de terreno del numeral 9.1, y es la que obliga a separar el almacén |
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
| Personal propio (1.780 personas) | Federación con el directorio corporativo del CLIENTE, con **doble factor de autenticación**, sin credenciales compartidas y con registro completo de accesos | Restricción n.º 9 en su enunciado completo, interpretada como aplicable al personal propio (SUP-08, RF-IDE-001) |
| Personas contratistas (2.400 en régimen, de unas 90 empresas; hasta 8.500 personas con acceso a faena en detención mayor de planta, numeral 14.1) | Credencial física emitida en la acreditación; en terreno basta el toque sobre dispositivo inventariado; fuera del ciclo productivo se exige además PIN numérico. Administración delegada a cada empresa desde el portal | RT-12.12 exige mecanismo autoservido. Provisionar hasta 8.500 identidades corporativas con 11 personas de TI es inviable (SUP-08, RF-IDE-003) |
| Persona operadora en equipo compartido | Credencial de proximidad leída en cabina en el relevo, sin login ni contraseña escrita en terreno | RT-12.11 fija el perfil operacional y prohíbe la contraseña alfanumérica en terreno (SUP-04) |

- **Segundo factor y sesión (RT-12.03, RT-12.02, RT-12.07):** el doble factor es obligatorio para las personas usuarias administradoras, para todo acceso privilegiado y para todo acceso originado fuera de la red corporativa. No aplica al terreno, donde el segundo factor es la posesión física de la credencial de proximidad sobre un dispositivo inventariado, y donde RT-12.11 prohíbe la contraseña alfanumérica. El inicio de sesión único con cierre propagado (RF-IDE-021), la política de sesión (RF-IDE-023) y el procedimiento de acceso de emergencia (RF-IDE-024) rigen para los componentes de oficina.
- **Dato personal sensible, Ley N.º 21.719:** habilitación, salud ocupacional, control de fatiga y control de alcohol y drogas se cifran **a nivel de campo**, y se audita **la consulta además de la modificación** (RN-29, RF-IDE-020). El acceso de emergencia queda registrado con persona, motivo y período de uso.
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
| Presentación | C-03 Captura en terreno · C-06 Portal de empresas contratistas · C-07 Portal de trazabilidad de lote · C-16 Vista única de equipo (interfaz) · C-19 Analítica, indicadores y reportería (interfaz) · C-20 Administración, parametrización y auditoría (interfaz) | C-07 mayoritariamente en Etapa 2, con una fila en Etapa 1; C-03 en Etapa 1, salvo los índices de dureza de RF-PET-006, que viajan en Etapa 2; C-16 con versión completa en Etapa 2; el resto en Etapa 1 |
| Borde y exposición | Parte de C-22 Plataforma base | Etapa 1 |
| Puerta de enlace de servicios | Parte de C-22 Plataforma base | Etapa 1 |
| Servicios de negocio | C-01 Núcleo de trazabilidad · C-02 Plan minero versionado · C-05 Identidad y acreditación · C-08 Motor de conciliación · C-15 Detenciones y productividad · C-16 Vista única de equipo (lógica) · C-17 Ambiental, huella de carbono y comunidad · C-18 Seguridad y salud ocupacional · C-19 Analítica, indicadores y reportería (lógica) · C-20 Administración y auditoría (lógica) | C-17 y la mayor parte de C-18 en Etapa 2; el resto en Etapa 1 |
| Integración y eventos | C-04 Nodo de borde: registro local, cola de eventos y sincronización · C-09 Integración con el sistema de gestión de laboratorio · C-10 Integración con el sistema de despacho de flota · C-11 Integración con el software de planificación minera · C-12 Integración con el ERP corporativo · C-13 Adaptadores de telemetría de fabricante con capa anticorrupción · C-14 Lectura del historiador de proceso con segregación TI/TO · adaptador del sistema de control de fatiga, perteneciente a C-18 | C-13 mayoritariamente en Etapa 2; el resto en Etapa 1 |
| Datos | C-21 Migración de datos históricos · almacenes declarados en el apartado 4, capa 6 | C-21 en Etapa 1, salvo mantenimiento y monitoreo ambiental que viajan en Etapa 2 con RF-MIG-008 |
| Seguridad transversal | C-05 (aplicada a todas las capas) · parte de C-22 | Etapa 1 |
| Observabilidad transversal | Parte de C-22 | Etapa 1 |

**Nota sobre C-22.** Concentra 35 requerimientos, el componente más cargado del catálogo. No es un componente de negocio: es la plataforma base —infraestructura, seguridad, identidad técnica y observabilidad— y por eso aparece en cuatro capas. En el diagrama se representa como banda transversal, no como caja dentro de una columna.

**Nota sobre los tres componentes que aparecen en dos capas.** C-16, C-19 y C-20 figuran a la vez en Presentación y en Servicios de negocio. Es deliberado y responde a la exigencia del numeral 2.1 de que la capa de presentación **no contenga lógica de negocio**: la superficie de pantalla y la lógica que la alimenta son piezas distintas del mismo componente, y se despliegan por separado.

- **C-20 — Administración, parametrización y auditoría:** el RT-16.02 exige que las reglas sean configurables desde la interfaz de administración y el RT-16.03 exige aprobación de un segundo perfil para todo cambio con impacto operacional. La interfaz está en Presentación; el motor de vigencia temporal de parámetros, en Servicios de negocio.
- **C-16 — Vista única de equipo y apoyo al mantenimiento:** la pantalla está en Presentación; la composición del cuadro a partir de C-10, C-12 y C-13 es lógica de negocio y está en la capa 4. Es lo que permite medir el criterio n.º 6 con una prueba cronometrada contra los 15 a 20 minutos actuales.
- **C-19 — Analítica, indicadores y reportería:** los tableros están en Presentación; el cálculo del indicador contra su definición versionada está en Servicios de negocio, sobre el almacén analítico separado que exige el RT-05.05.

---

## 6. Límites de contexto (RT-02.02)

Nueve contextos. Cada uno tiene lenguaje propio, un agregado raíz y una frontera explícita de lo que **no** le pertenece. La frontera es la parte que se evalúa: un contexto sin exclusiones declaradas no es un límite, es una etiqueta.

### 6.1 Contexto Mineral

- **Componentes:** C-01, C-02, C-08, más los tres adaptadores que lo alimentan: **C-10** — Integración con el sistema de despacho de flota, **C-11** — Integración con el software de planificación minera y **C-12** — Integración con el ERP corporativo.
- **Agregado raíz:** el **viaje**. Los demás agregados —aporte a stock, retoma, lote de chancado, lote comercial— son agregaciones derivadas con relación padre-hijo hacia los viajes que los componen y su proporción.
- **Lenguaje propio:** polígono, viaje, pala de origen, stock, aporte, retoma, ley ponderada, lote de chancado, lote comercial, metal contenido, punto de medición, diferencia de conciliación.
- **No le pertenece:** la identidad de la persona que operó el equipo (es del contexto Personas, que le entrega un identificador ya validado); el resultado de laboratorio (es del contexto Calidad); la cifra contable oficial (la emite el ERP, restricción n.º 4).
- **Relación con vecinos:** consume identidad del contexto Personas y ley medida del contexto Calidad, ambos como **conformista** —acepta el modelo del otro sin traducirlo—. Publica eventos de trazabilidad al contexto Analítico y al Comercial.
- **Regla dura:** ninguna diferencia de conciliación se cierra como «no explicada». La diferencia residual queda atribuida a un punto de medición de la cadena. Es la excepción declarada de **RN-06**, y es la propiedad que sostiene el criterio de aceptación n.º 3.
- **Por qué los tres adaptadores pertenecen a este contexto:** la regla de asignación es que un adaptador pertenece al contexto cuyo agregado raíz alimenta o consume. C-10 entrega el viaje mismo —pala de origen, destino, tonelaje, posición y hora—, que es el agregado raíz; C-11 entrega la versión vigente del plan, contra la que se mide todo movimiento; y C-12 recibe el tonelaje, la ley y el metal contenido en cada cierre, y es el emisor contable único (RN-05, restricción n.º 4). El maestro de personal propio viaja por ese mismo adaptador C-12, y el contexto Personas lo consume como **conformista**, sin traducirlo. Con esta regla los veintidós componentes quedan asignados a un límite de contexto: ninguno queda fuera.

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
- **Agregado raíz:** el **compromiso** —ambiental, de seguridad, de comunidad o de cierre de faena— y su evidencia.
- **Lenguaje propio:** compromiso de la Resolución de Calificación Ambiental, compromiso del plan de cierre de faena, punto de medición, frecuencia, destinatario, inspección, incidente, alerta de fatiga, factor de emisión, alcance 1, 2 y 3, cadena de custodia de producción responsable.
- **Alcance del agregado «compromiso»:** cubre los cuatro frentes que declara el Capítulo 12 del caso, no tres. Además del ambiental, el de seguridad y salud ocupacional y el de comunidad, incluye el **cierre de faenas**, que obliga a información trazable sobre el avance de la operación y sobre las obras comprometidas, con obligación de información periódica. Los **esquemas de aseguramiento de cobre producido responsablemente y de cadena de custodia** —que el caso señala como el origen de esta licitación— se registran también como compromisos, y su evidencia se expone por el contexto Comercial dentro de los límites de RN-16.
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
| Sistema de gestión de laboratorio | **Bidireccional.** Envía solicitud de análisis y recibe resultado validado | Adaptador con capa anticorrupción; evento en AsyncAPI 3.0 | Continua; del orden de 71.000 muestras al año | C-09 |
| Software de planificación minera | Lectura de la versión vigente del plan. **No** interviene el modelo de bloques | Adaptador; ingesta por publicación de versión | 3 a 4 veces por semana, según la frecuencia real de cambio del plan | C-11 |
| Historiador de proceso de planta | **Sólo lectura.** Nunca se escribe en la red de control | OPC UA con segregación conforme a IEC 62443 | Continua | C-14 |
| Tres portales de telemetría de fabricante | Lectura de estado, posición, horómetro y alarmas | **Un adaptador por fabricante**, cada uno con capa anticorrupción y reemplazable | Latencia de ingesta declarada de 5 minutos (SUP-30) | C-13 |
| Sistema de control de fatiga | Lectura de alertas emitidas | Adaptador; dato personal sensible, cifrado a nivel de campo | Continua | C-18 |

**Marco de referencia de la integración.** El Capítulo 12 del caso fija la interoperabilidad industrial sobre tres piezas, y el RT-05.23 remite a los estándares sectoriales que el caso identifique: **OPC UA**, **ISA-95** y los **estándares abiertos de intercambio de datos de equipos mineros**. ISA-95 ordena la relación entre el nivel de operación y el de gestión, y es el marco que separa lo que la solución lee del historiador de proceso —nivel de control, sólo lectura, C-14— de lo que intercambia con el ERP —nivel de gestión, C-12— y de lo que ocurre en el nivel de ejecución de la operación, donde vive el núcleo de trazabilidad C-01. Los adaptadores de telemetría C-13 emplean el estándar abierto de intercambio de datos de equipos mineros **cuando el fabricante lo soporte**, y el protocolo propietario del portal sólo cuando no lo soporte: ésa es además la vía alternativa que declara SUP-10 si algún fabricante no cede el acceso programático. La huella de carbono se calcula conforme al **protocolo de gases de efecto invernadero**, alcances 1, 2 y 3 (RN-26), con verificación por tercero, que el caso exige al comprador a partir de 2029.

**Fundamento del patrón.** Los siete usan capa anticorrupción y adaptador reemplazable (RT-02.14). No es sofisticación: el Capítulo 5 del caso declara que el CLIENTE **no tiene documentadas** las interfaces, las versiones ni las condiciones contractuales de acceso de sus propios sistemas. Un adaptador aislado permite descubrir la interfaz real en ejecución sin propagar el hallazgo al núcleo.

### 8.2 Interfaces internas

**Síncronas**, sólo donde la latencia lo exige y el consumidor no puede continuar sin respuesta:

| Consumidor | Proveedor | Para qué | Umbral |
|---|---|---|---|
| Portería, superficie de C-05 | Caché local de credenciales y habilitaciones vigentes del nodo de borde (C-04) | Validar habilitación vigente sin depender del enlace | Consulta no superior a 1,5 segundos (RN-20), sostenida también durante un corte |
| Cabina (C-03) | Caché local de credenciales (C-04) | Abrir sesión de operador por proximidad | 1 segundo desde la lectura, sin retirar el guante (SUP-29) |
| Vista única (C-16) | C-10 despacho, C-12 ERP, C-13 telemetría | Reunir estado, historial, pautas y posición | Cuadro completo en menos de los 15 a 20 minutos actuales |

**Asíncronas por evento**, que es el caso por omisión. Todo hecho de terreno, toda conformación de lote, toda clasificación de detención y todo cálculo de indicador viajan por el bus. Consecuencia de diseño: **ningún componente de negocio consulta la base de datos de otro**. Si necesita un dato, se suscribe a su evento.

### 8.3 Contrato de interfaz

Servicios síncronos en **OpenAPI 3.1** y flujos por evento en **AsyncAPI 3.0**, generados desde el código y mantenidos actualizados automáticamente (RT-05.16). El requisito fija AsyncAPI 2.6 como piso —«2.6 o superior»—; se adopta la 3.0 porque la serie 2.x está superada desde diciembre de 2023 y el cambio no tiene costo en esta etapa del proyecto. Versionado semántico con compatibilidad hacia atrás y política de obsolescencia con preaviso mínimo de seis meses (RT-05.17).

**Autenticación entre sistemas (RT-05.18).** Los adaptadores que consumen servicios expuestos emplean **OAuth 2.1 con credenciales de cliente**; los enlaces máquina a máquina de terreno y la lectura del historiador emplean **autenticación mutua TLS**. Queda prohibida la autenticación por clave estática en la ruta de la dirección web, sin excepción. Toda integración registra la transacción de entrada y de salida con un **identificador de correlación común**, que es el mismo que la capa 8 usa para la traza distribuida, de modo que una operación de negocio se sigue a través de todos los sistemas involucrados (RT-05.19).

Toda operación de escritura expuesta a reintentos es idempotente, con clave de idempotencia declarada por el cliente y ventana de deduplicación documentada (RT-02.06).

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
| **Viajes de camión de extracción** | **~342.000 al año**, con proyección de 367.000 a tres años | Numeral 14.1 |
| **Ciclos de carguío de pala** | **~1,37 millones al año**, con proyección de 1,47 millones | Numeral 14.1 |
| Movimiento total mina | 82 Mt al año, con proyección de 88 Mt | Numeral 14.1 |
| Mineral enviado a planta | 21,5 Mt al año | Numeral 2.2 y numeral 14.1 |
| Concentradora | 59.000 t al día, capacidad nominal | Numeral 2.2 |
| Flota | 38 camiones de extracción de 240 t, de tres generaciones y dos fabricantes; 5 palas eléctricas de cable; 3 cargadores frontales; 9 perforadoras; 62 equipos de apoyo; 310 vehículos de flota liviana | Numeral 2.3 |
| Equipos con telemetría de fabricante | 52 hoy, 70 proyectados, en tres portales distintos | Numeral 14.1 |
| Personas | 1.780 propias y ~2.400 contratistas permanentes de unas 90 empresas; 4.180 con acceso a faena en régimen; hasta 8.500 con acceso en detención mayor de planta | Numerales 2.4 y 14.1 |
| Ingresos y salidas de portería | ~2.900 diarios | Numeral 14.1 |
| Muestras analizadas en laboratorio | ~71.000 al año | Numeral 14.1 |
| Pozos de perforación | ~46.000 al año; ~340 tronaduras al año | Numeral 14.1 |
| Viajes de concentrado a puerto | ~12.400 al año | Numeral 14.1 |
| Embarques | 31 al año, de ~12.000 t cada uno | Numeral 2.2 |
| Sitios operacionales | 7, incluido acopio de 25.000 t en recinto portuario de terceros, con capacidad de incorporar un octavo | Numeral 14.1 (RNF-CAP-011) |
| Operación | 24 horas, 365 días, dos turnos de 12 horas, régimen 7x7 | Numeral 2.4 y restricción n.º 1 |

Las dos primeras filas son las que dimensionan el agregado raíz. Trescientos cuarenta y dos mil viajes al año son unos 940 diarios, y son el hecho que la Etapa 1 debe capturar íntegro; los 12.400 viajes de concentrado a puerto son un flujo distinto y mucho menor, y confundirlos subdimensiona el sistema en un factor de veintisiete.

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

El apartado 1.1 declara esta vista como propia de este subdocumento. Aquí se deciden el **modelo, las fronteras y los controles lógicos**. El producto, la versión y el emplazamiento de cada control son materia del Subdocumento 4.2. El programa de gestión de vulnerabilidades, el centro de operaciones de seguridad, el plan de respuesta a incidentes y los ejercicios de simulación son actividades de la Operación y se desarrollan en los Subdocumentos 10 y 11; el apartado 15 declara esa frontera código por código.

### 11.1 Modelo Zero Trust y la excepción declarada del enclave de terreno (RT-11.01)

La arquitectura adopta **Zero Trust conforme a NIST SP 800-207**: verificación explícita de cada solicitud, privilegio mínimo y presunción de compromiso. Ninguna posición de red concede confianza. Que una petición provenga de la red de faena no la autoriza, y por eso los siete adaptadores externos autentican como cualquier otro consumidor (RT-05.18) y la red se segmenta sin que ningún componente de datos sea alcanzable desde Internet (RT-03.04).

El modelo tiene **una excepción, y la impone el caso**. Durante las 48 horas de operación sin enlace el punto de decisión de política central es inalcanzable, y verificar cada solicitud contra él detendría la mina, que es exactamente lo que prohíben la restricción n.º 1 y la restricción n.º 2. La excepción se resuelve delegando el punto de decisión, no suprimiéndolo:

| Elemento del modelo | Con enlace | En el enclave de terreno sin enlace |
|---|---|---|
| Punto de decisión de política | Servicio central de autorización sobre C-05 | Delegado al nodo de borde C-04, con política **firmada por C-05** y vigencia declarada |
| Verificación de identidad | Contra el maestro de personas de C-05 | Contra la caché local de credenciales y habilitaciones vigentes, firmada y fechada en origen |
| Privilegio mínimo | Rol, área y sitio (RF-IDE-018) | El mismo conjunto de roles, congelado en la caché: en terreno no se conceden privilegios nuevos |
| Presunción de compromiso | Sesión corta y reautenticación | El evento viaja firmado con el identificador del dispositivo inventariado, y **la aserción de identidad con que se admitió viaja dentro del evento** |
| Momento de la verificación | En la solicitud | En la solicitud contra la caché, **y otra vez en la reconciliación** contra el maestro central |

La propiedad que hace aceptable la excepción es que **la verificación no se omite: se difiere y se repite**. Un evento admitido en terreno contra una credencial que el maestro central había revocado se reconcilia igualmente —el hecho ocurrió y RN-30 manda conservarlo—, pero queda marcado, alertado y trazable hasta la aserción con que se admitió. Es la regla de RN-30 aplicada a la identidad.

**Limitación residual que se declara.** La caché de credenciales tiene vigencia máxima de **72 horas**, con margen sobre las 48 comprometidas, y se refresca en cada sincronización. Durante un corte prolongado una revocación no se propaga, de modo que el umbral de quince minutos de SUP-28 se mide sobre operación con enlace y no durante el corte. Los controles compensatorios son tres: la persona cuya habilitación se revoca ya está físicamente dentro del recinto y el control de portería lo sabe; el nodo de borde deja de admitir la credencial al vencer la vigencia de la caché aunque no haya recuperado el enlace; y todo evento admitido en la ventana queda marcado en la reconciliación. Se incorpora como umbral del PROPONENTE al Registro de supuestos, con validación en el corte controlado de 48 horas del apartado 9.2.

### 11.2 Modelado de amenazas por componente y por integración (RT-11.02)

Metodología declarada: **STRIDE**, aplicado a cada uno de los veintidós componentes y a cada una de las siete integraciones externas, con revisión obligatoria ante todo cambio que altere un límite de contexto, una interfaz o una clasificación de información. El modelo es entregable contractual y se versiona junto con los ADR del apartado 18.

La tabla no reemplaza al modelo completo: fija las amenazas de mayor consecuencia de este caso, que son las que ordenan los controles del resto del apartado.

| Categoría STRIDE | Amenaza principal del caso | Componente o integración | Control lógico que la trata |
|---|---|---|---|
| Suplantación | Préstamo o uso indebido de credencial de proximidad entre personas del mismo turno | C-05, C-03 | Detección por patrones sobre los eventos de acceso ya registrados (RF-IDE-012); credencial ligada a dispositivo inventariado; caso de detección propio del apartado 11.7 |
| Manipulación | Alteración del registro local de eventos en un dispositivo extraviado o intervenido | C-03, C-04 | Registro firmado con clave del almacén respaldado por hardware (apartado 16.3); identificador de evento generado en origen; corrección como evento nuevo, nunca modificación (decisión n.º 1 del apartado 2.2) |
| Repudio | Registro sin persona identificada, o corrección posterior sin autor | C-01, C-20 | Marca «operador no identificado» como indicador gestionable (RN-22); asiento de auditoría inalterable con autor, fecha y justificación (RN-27) |
| **Divulgación de información** | **Fuga de la ley por polígono al comprador o a un tercero, por el portal, por una exportación o por un informe** | **C-07, C-19, C-01** | **Es la frontera más importante del sistema (apartado 6.7). La vista de exposición filtra en el servicio y no en la pantalla; toda exportación y toda búsqueda respetan el control de acceso (RT-16.27, RF-IDE-018); la publicación requiere aprobación del área comercial y toda consulta queda registrada (RN-16, SUP-17)** |
| Divulgación de información | Acceso indebido a habilitación, salud ocupacional, fatiga o control de alcohol y drogas | C-05, C-18 | Cifrado a nivel de campo y auditoría **de la consulta** además de la modificación (RN-29, RF-IDE-020, RT-11.10) |
| Denegación de servicio | Ataque al portal del comprador que arrastre al núcleo transaccional | C-07, C-22 | Mamparo de recursos y caché con umbral de cero degradación de los umbrales de terreno ante diez veces el tráfico medio del portal (RNF-DIS-010); el registro de terreno no atraviesa el borde público (apartado 4, capa 2) |
| Denegación de servicio | Saturación del enlace al restablecerse, por la cola acumulada de cuatro sitios | C-04, C-22 | Calidad de servicio en la puerta operacional (RT-03.24); reconciliación por lotes con límite de tasa; el terreno sigue registrando localmente mientras tanto |
| Elevación de privilegio | Uso de una cuenta de administración para alterar un parámetro con impacto operacional | C-20 | Aprobación de un segundo perfil para todo cambio con impacto operacional (RT-16.03); elevación temporal a demanda con sesión grabada (RF-IDE-022); doble factor resistente a suplantación (apartado 4, capa 7) |
| Elevación de privilegio | Escritura en la red de control de proceso desde la red de gestión | C-14 | La restricción n.º 3 es absoluta: la lectura es por OPC UA con segregación conforme a IEC 62443 y el adaptador **no implementa ninguna operación de escritura**. No es una regla de configuración: es una capacidad ausente del componente |

### 11.3 Clasificación de la información y controles por nivel (RT-11.03)

Cuatro niveles. La clasificación es atributo del dato y viaja con él: la hereda el evento, el índice de búsqueda, la exportación y el respaldo.

| Nivel | Qué comprende en este caso | Controles diferenciados |
|---|---|---|
| **Público** | Ninguna información de la solución. No existe portal abierto a la ciudadanía: el caso lo excluye | No aplica. Que este nivel esté vacío es una decisión de diseño, no un olvido |
| **Interno** | Registro operacional: viajes, pesajes, detenciones, inspecciones, estados de equipo, indicadores por turno | Cifrado en tránsito y en reposo; control de acceso por rol, área y sitio; auditoría de modificación |
| **Confidencial · competitivamente sensible** | **Ley por polígono, modelo de bloques, plan minero versionado, liquidaciones, penalizaciones y márgenes** | Todo lo anterior, más: prohibición de exposición al exterior en cualquier forma derivada (RN-16); aprobación del área comercial para toda publicación; registro de **toda consulta**, no sólo de la modificación; exportación restringida por rol y trazada |
| **Personal sensible · Ley N.º 21.719** | Habilitación, exámenes preocupacionales, salud ocupacional, control de fatiga, control de alcohol y drogas | Todo lo anterior, más: **cifrado a nivel de campo** (RT-11.10); auditoría de consulta (RN-29, RF-IDE-020); prohibición de aparecer en registros de la solución (apartado 12.4); prohibición de uso en ambientes no productivos sin seudonimización (apartado 11.8); retención de diez años sin borrado anticipado (RN-28) |

La consecuencia arquitectónica de la tabla es que **el nivel confidencial y el nivel personal sensible no comparten el mismo control**, y por eso no comparten componente: la ley por polígono vive en el contexto Mineral y se filtra en la vista de exposición hacia el Comercial; el dato personal sensible vive en el contexto Personas y en el de Cumplimiento, cifrado a nivel de campo. Un diseño que los tratara con un solo control sobreprotegería el primero y subprotegería el segundo.

### 11.4 Superficie de exposición y controles de borde (RT-11.07, RT-11.12, RT-11.13)

**Toda publicación de servicios se realiza exclusivamente por la capa de borde** (RT-11.07). No hay excepción, y la afirmación es verificable: la superficie alcanzable desde fuera de la red del CLIENTE es la siguiente y ninguna otra.

| Qué se expone | A quién | Autenticación | Control de borde |
|---|---|---|---|
| Portal de empresas contratistas (C-06) | Unas 90 empresas contratistas | Credencial de empresa con doble factor, administración delegada | Red de distribución de contenidos, cortafuegos de aplicaciones web con reglas gestionadas **y personalizadas**, protección contra denegación de servicio en capas 3, 4 y 7, TLS 1.3 |
| Portal de trazabilidad de lote (C-07) | Compradores, en Etapa 2 | Credencial nominada por comprador, con registro de toda consulta | Lo mismo, más límite de tasa y cuota por consumidor en la puerta de exposición |
| Puerta de enlace de exposición | Integraciones de sistemas autorizadas | OAuth 2.1 con credenciales de cliente o autenticación mutua TLS (RT-05.18) | Validación de esquema e inspección de carga útil antes de alcanzar cualquier servicio |

**Nada más.** En particular: la puerta operacional **no** es alcanzable desde Internet —los nodos de borde llegan a ella por el enlace privado del Subdocumento 4.2—; ningún componente de datos es alcanzable desde fuera (RT-03.04); y el registro de un viaje no atraviesa el borde público en ningún punto de su recorrido. La enumeración concreta de nombres de dominio, puertos y direcciones es materia del Subdocumento 4.2, que la deriva de esta tabla.

**Protección contra bots y abuso automatizado (RT-11.12).** Reto progresivo en los dos portales, escalonado por señal de riesgo y **nunca por defecto**: primero límite de tasa por credencial, luego reto interactivo, y sólo después bloqueo. El requisito exige que no degrade la accesibilidad, y en este caso hay una razón adicional para el escalonamiento: quien usa el portal de empresas contratistas es personal administrativo de empresas pequeñas, con conectividad y dispositivos desiguales, y un reto agresivo se traduce en acreditaciones que no se completan y en el criterio de aceptación n.º 8 incumplido. El reto no se aplica a personas usuarias autenticadas con sesión vigente.

### 11.5 Cifrado en tránsito, cifrado en reposo y custodia de claves (RT-11.08, RT-11.09)

**En tránsito (RT-11.08):** TLS 1.3, con prohibición expresa de TLS 1.0 y 1.1 y de conjuntos de cifrado obsoletos, HSTS con precarga en los dos portales, y gestión automatizada de certificados con rotación y alerta anticipada de vencimiento. La autenticación mutua TLS rige los enlaces máquina a máquina de terreno y la lectura del historiador. El vencimiento de un certificado es un modo de falla que detiene la reconciliación de un sitio, y por eso su alerta es de las que el apartado 12.3 clasifica como síntoma de negocio y no como umbral de infraestructura.

**En reposo (RT-11.09):** la totalidad de los datos en reposo está cifrada, en las seis naturalezas de persistencia del apartado 4 y también en **el registro local del dispositivo de terreno y del nodo de borde**, que es donde el dato pasa hasta 48 horas fuera del centro de datos y es el punto más expuesto a la pérdida física del soporte.

**Custodia de claves:** claves gestionadas en un servicio de gestión de claves o módulo de seguridad de hardware, con política de rotación declarada y **separación de funciones**: quien administra la plataforma no custodia las claves del nivel personal sensible, y ninguna persona reúne por sí sola la capacidad de descifrar un campo de salud ocupacional. Las claves de firma del registro local de terreno residen en el almacén respaldado por hardware del propio dispositivo (apartado 16.3) y no salen de él: el nodo de borde verifica la firma, no la produce.

### 11.6 Controles de la puerta de enlace de servicios (RT-11.11)

Las dos puertas lógicas del apartado 4, capa 3, aplican el mismo conjunto de controles con parámetros distintos, porque los perfiles de tráfico son incompatibles y ya se separaron por ADR-07.

| Control | Puerta operacional | Puerta de exposición |
|---|---|---|
| Autenticación | Autenticación mutua TLS del nodo de borde y del dispositivo inventariado | OAuth 2.1 con credenciales de cliente; sesión de persona usuaria en los portales |
| Autorización | Rol congelado en la caché, verificado de nuevo en la reconciliación (apartado 11.1) | Rol, área y sitio evaluados en cada solicitud (RF-IDE-018) |
| Cuotas y límite de tasa | Dimensionado para absorber la reconciliación de cuatro sitios tras 48 horas, no para restringirla | Cuota y límite de tasa **por consumidor**, que es el mecanismo que protege al núcleo del tráfico de portal |
| Validación de esquema | Estricta contra el contrato AsyncAPI 3.0 y OpenAPI 3.1; el evento que no valida va a la cola de mensajes fallidos, **nunca se descarta** | Estricta contra OpenAPI 3.1; la solicitud que no valida se rechaza con error explícito |
| Inspección de carga útil | Tamaño, profundidad y tipo; la evidencia fotográfica viaja por canal separado con límite propio | Tamaño, profundidad y tipo, más reglas del cortafuegos de aplicaciones web |

La diferencia que importa: en la puerta operacional **un evento nunca se pierde por fallar una validación**. Va a la cola de mensajes fallidos con su carga íntegra y se resuelve, porque el hecho de terreno ocurrió y el criterio de aceptación n.º 13 exige que los eventos reconciliados coincidan exactamente con los generados.

### 11.7 Registro de seguridad, correlación y casos de detección propios del caso (RT-11.14, RT-11.15)

Los eventos de seguridad se registran de forma **centralizada e inalterable**, con retención de **doce meses en línea y veinticuatro meses adicionales en archivo recuperable**. El nodo de borde acumula localmente sus eventos de seguridad mientras está sin enlace y los reconcilia con el mismo mecanismo que los de negocio: la seguridad se degrada igual que el resto y se recupera igual que el resto.

La correlación se realiza en una plataforma SIEM. El requisito exige casos de uso de detección **definidos para el proceso de negocio del caso y no sólo genéricos de infraestructura**, y ésa es la parte que corresponde decidir aquí, porque depende del dominio y no del producto:

| Caso de detección | Qué señal cruza | Por qué es propio de este caso |
|---|---|---|
| Credencial usada en dos sitios en un intervalo físicamente imposible | Eventos de portería (C-05) y de apertura de sesión en cabina (C-03), con la distancia entre sitios | Es la forma concreta que toma el préstamo de credencial en una faena de siete sitios. Alimenta a RF-IDE-012 |
| Exportación o consulta masiva de trazabilidad por un perfil comercial | Auditoría de consulta de C-01 y de C-19, contra la línea base del rol | Es la vía por la que la ley por polígono saldría sin romper ningún control de acceso. Ataca la frontera de RN-16 |
| Consulta de dato personal sensible fuera del ámbito del rol o fuera de horario | Auditoría de consulta de C-05 y C-18 (RF-IDE-020) | La Ley N.º 21.719 obliga a auditar la consulta; el valor está en detectarla, no sólo en registrarla |
| Cambio de parámetro sin la aprobación del segundo perfil, o dentro de ventana prohibida | Asiento de auditoría de C-20 contra RT-16.03 y contra las ventanas de RN-24 | Un cambio de umbral durante el cierre contable altera cifras ya declaradas al ERP |
| Evento de terreno cuyo sello de tiempo es inconsistente con el registro de deriva de reloj de su dispositivo | Reconciliación en C-04 | Es la señal de manipulación del registro local, y la única disponible mientras el dispositivo estuvo aislado |
| Cualquier intento de escritura hacia la red de control de proceso | Segregación TI/TO de C-14 | La restricción n.º 3 es absoluta. La detección existe aunque la capacidad de escritura no esté implementada, precisamente para evidenciar que no se usa |
| Acceso al recinto técnico fuera de ventana de intervención declarada | Bitácora del control biométrico del recinto (RF-IDE-014, RT-06.20) | Cruza el control físico con la ventana operacional protegida de RT-10.05 |

### 11.8 Acceso a producción y datos en ambientes no productivos (RT-11.25, RT-11.27)

**Las personas desarrolladoras no tienen acceso interactivo directo a producción** (RT-11.27). Todo acceso excepcional es temporal, aprobado previamente, registrado y con sesión grabada, por el mismo mecanismo de elevación de RF-IDE-022. La consecuencia de diseño es que el diagnóstico de un incidente debe poder hacerse **desde la observabilidad y no desde la base de datos**, y por eso el apartado 12 exige traza correlacionada extremo a extremo: si para entender una falla hay que entrar a producción, el diseño de observabilidad falló.

**Queda prohibido el uso de datos productivos reales en ambientes no productivos sin anonimización o seudonimización verificable** (RT-11.25). En este caso la prohibición muerde en dos lugares concretos y por eso se declara aquí y no sólo en la metodología:

- **El dato personal sensible no se copia nunca**, ni siquiera seudonimizado, a un ambiente no productivo. Las pruebas que lo requieren usan datos sintéticos, porque la seudonimización de una población de 4.180 personas con habilitaciones, exámenes y resultados de fatiga es reversible por cruce y no resiste el estándar de «verificable».
- **La ley por polígono se ofusca** en todo ambiente no productivo, conservando la distribución estadística pero no los valores reales, porque un ambiente de pruebas con menor control expone la misma información competitivamente sensible que el nivel confidencial protege.

El volumen representativo que exige Preproducción se consigue con generación sintética a partir de la volumetría del apartado 10.1, no con copia de producción.

### 11.9 Resumen de la vista de seguridad

| Frente | Decisión lógica |
|---|---|
| Modelo | Zero Trust conforme a NIST SP 800-207, con la excepción declarada y acotada del enclave de terreno del apartado 11.1 |
| Identidad | Tres mecanismos sobre un maestro único, según apartado 4 capa 7. La federación corporativa aplica al personal propio, con doble factor, sin credenciales compartidas y con registro completo de accesos (restricción n.º 9, RF-IDE-001); las personas contratistas se rigen por esquema autoservido |
| Segundo factor | Obligatorio para personas usuarias administradoras, para todo acceso privilegiado y para todo acceso originado fuera de la red corporativa (RT-12.03), con factor resistente a la suplantación tipo FIDO2 o clave de acceso en los perfiles administradores (RT-12.04). En terreno el segundo factor es la posesión física de la credencial de proximidad sobre un dispositivo inventariado: RT-12.11 prohíbe la contraseña alfanumérica escrita |
| Autorización | Mínimo privilegio por rol, área y sitio (RF-IDE-018). La búsqueda global y toda exportación respetan el control de acceso (RT-16.27) |
| Clasificación | Cuatro niveles con controles diferenciados, apartado 11.3 |
| Segregación TI/TO | Lectura del historiador por OPC UA con segregación conforme a IEC 62443. No se escribe en la red de control (restricción n.º 3), y la capacidad de escritura no está implementada en C-14 |
| Dato personal sensible | Cifrado a nivel de campo para habilitación, salud ocupacional, control de fatiga y control de alcohol y drogas. Se audita la consulta además de la modificación (RN-29, Ley N.º 21.719) |
| Auditoría | Asiento inalterable de toda modificación y de toda consulta sensible. La retención la fija el caso por tipo de registro y no admite borrado anticipado (RN-28): diez años para seguridad y salud ocupacional, para ambientales y de monitoreo y para trazabilidad de mineral y embarques; siete para tributarios; cinco para operacionales. El RT-16.10 fija cinco años sólo en defecto de que el caso lo fije, y el caso lo fija |
| Aislamiento del portal externo | Mamparo de recursos y caché entre los portales expuestos a Internet y el núcleo transaccional. Umbral: cero degradación de los umbrales de terreno ante diez veces el tráfico medio del portal (RNF-DIS-010) |
| Recinto técnico | Control de acceso biométrico facial con AFIS de respaldo, exigido con carácter obligatorio por el RT-06.20 (RF-IDE-014). **No confundir** con la biometría de portería, que está excluida del Contrato mientras subsista la objeción sindical |

### 11.10 Trazabilidad de este apartado

| Requisito | Dónde se acredita | Componente que lo satisface |
|---|---|---|
| RT-11.01 | Apartado 11.1 | C-22 — Plataforma base; C-05 — Identidad, acreditación y control de acceso de personas |
| RT-11.02 | Apartado 11.2 | Todos los componentes y las siete integraciones |
| RT-11.03 | Apartado 11.3 | C-22; C-01 — Núcleo de trazabilidad; C-05 |
| RT-11.07 | Apartado 11.4 | C-22 (capa de borde y exposición) |
| RT-11.08 | Apartado 11.5 | C-22 |
| RT-11.09 | Apartado 11.5 | C-22; C-04 — Nodo de borde; C-03 — Captura en terreno |
| RT-11.11 | Apartado 11.6 | C-22 (puerta de enlace de servicios) |
| RT-11.12 | Apartado 11.4 | C-22; C-06 — Portal de empresas contratistas; C-07 — Portal de trazabilidad de lote |
| RT-11.13 | Apartado 11.4, con la enumeración concreta en el Subdoc. 4.2 | C-22 |
| RT-11.14 | Apartado 11.7 | C-22; C-20 — Administración, parametrización y auditoría |
| RT-11.15 | Apartado 11.7 | C-22; C-05; C-20 |
| RT-11.25 | Apartado 11.8 | C-22; C-21 — Migración de datos históricos |
| RT-11.27 | Apartado 11.8 | C-22 |
| RT-12.03, RT-12.04 | Apartados 4 capa 7 y 11.9 | C-05 |
| RT-06.20 | Apartado 11.9 | C-22, con RF-IDE-014 |

---

## 12. Observabilidad, niveles de servicio y alertamiento

El apartado 4, capa 8, declara la observabilidad como capa transversal obligatoria. Aquí se decide qué se instrumenta, sobre qué se mide el nivel de servicio y qué dispara una alerta. El libro de operación, la guía de resolución y el análisis de causa raíz son actividades de la Operación y se desarrollan en los Subdocumentos 10 y 11.

### 12.1 Instrumentación e identificador de correlación (RT-14.01, RT-05.19)

Una sola plataforma de observabilidad para nube y faena, con alertamiento unificado y **sin puntos ciegos**, instrumentada conforme a **OpenTelemetry**. Métricas, registros y trazas se correlacionan por un **identificador único de transacción** que es el mismo identificador de correlación que exige el RT-05.19 para las integraciones: no son dos identificadores distintos, y ésa es la decisión que permite seguir una operación de negocio a través de la solución y de los sistemas del CLIENTE con un solo hilo.

El identificador **nace en el dispositivo de terreno**, junto con el identificador de evento de RN-30, y no en el borde de entrada. La razón es la misma que sostiene ADR-05: un identificador asignado al recibir no puede correlacionar lo que ocurrió durante las 48 horas en que no hubo recepción. Cuando el nodo de borde reconcilia, la traza del evento ya tiene su historia local adjunta, incluida la espera.

El nodo de borde emite su propia telemetría de salud y, mientras está sin enlace, la acumula localmente junto con los eventos de negocio. La observabilidad se degrada igual que el resto y se reconcilia igual que el resto.

### 12.2 Indicadores de nivel de servicio medidos sobre la experiencia real (RT-14.03)

Los indicadores de nivel de servicio se miden **sobre la experiencia real de la persona usuaria y no sobre pruebas sintéticas**. En este caso la distinción no es doctrinaria: una prueba sintética lanzada desde el centro de datos no atraviesa la red inalámbrica del rajo, que es donde están las zonas de sombra, y por lo tanto mediría un sistema que nadie usa.

| Indicador | Umbral | Dónde se mide | Origen |
|---|---|---|---|
| Registro de un movimiento de material desde el equipo en terreno | 2 segundos en el percentil 95 | Desde la acción de la persona operadora hasta la confirmación en pantalla, instrumentado en el dispositivo | RT-09.01, RNF-DES-001 |
| Registro en báscula | 3 segundos | En el punto de pesaje | RT-09.01 |
| Consulta de habilitación en portería | 1,5 segundos | En el torniquete, contra la caché local del nodo de borde | RT-09.01, RN-20 |
| Apertura de sesión de operador por proximidad | 1 segundo desde la lectura, sin retirar el guante | En cabina | SUP-29 |
| Disponibilidad de un indicador de turno en la capa analítica | 15 minutos | Desde la ocurrencia de la transacción | RT-05.29 |

Las pruebas sintéticas se emplean **como complemento** y con un propósito acotado que este caso hace necesario: verificar desde cada uno de los siete sitios que la ruta está viva cuando no hay actividad de personas, de modo que un corte nocturno no se descubra al inicio del turno de día.

### 12.3 Alertamiento por síntoma de negocio (RT-14.04)

El alertamiento se basa en **síntomas de negocio y no sólo en umbrales de infraestructura**, con supresión de ruido, agrupación y escalamiento automático. La regla que ordena la lista: se alerta sobre lo que compromete un criterio de aceptación, no sobre lo que un tablero de infraestructura consideraría anómalo.

| Síntoma de negocio | Por qué es la señal correcta | Criterio que protege |
|---|---|---|
| Viajes registrados por hora bajo la línea base del turno, por sitio | Un nodo de borde puede estar sano y aun así no estar recibiendo registros, porque la aplicación de terreno dejó de usarse. Es la única alerta que detecta el modo de falla que hundió las dos iniciativas anteriores | n.º 14 |
| Horas acumuladas sin enlace y retraso de reconciliación pendiente, por sitio | Es la evidencia directa del compromiso de 48 horas y de la sincronización en 4 horas | n.º 13 |
| Detenciones sin clasificar acercándose al cierre del turno | El turno no cierra con detenciones sin clasificar y las pendientes escalan a la guardia siguiente (RN-18). La alerta anticipa el escalamiento en vez de constatarlo | n.º 5 |
| Lotes retenidos por análisis vencido o no publicado | La liberación no admite excepción manual (RN-11): si el laboratorio no responde, el lote no sale, y eso debe verse antes de la ventana de embarque | n.º 1, n.º 2 |
| Arsénico proyectado sobre el límite antes de conformar el lote definitivo | Es el control que ataca las cuatro penalizaciones de USD 1,9 millones. La alerta debe llegar **antes** de la conformación, no después | n.º 1 |
| Habilitaciones por vencer sin acción registrada, por empresa contratista | Es la señal que evita el rechazo en portería, y se dirige a quien puede resolverlo: la empresa empleadora | n.º 7, n.º 8 |
| Eventos en cuarentena por referenciar un dato maestro inexistente | Es el residuo previsto del apartado 9.3 y crece silenciosamente si nadie lo mira | n.º 13 |
| Diferencia de conciliación proyectada sobre el umbral comprometido | Anticipa el cierre en vez de descubrirlo el día 3 | n.º 2, n.º 3 |
| Certificado próximo a vencer en cualquier ruta de integración o de terreno | Su vencimiento detiene una reconciliación completa de sitio, no un servicio aislado | n.º 13 |

Los turnos de disponibilidad se declaran contra el régimen real de la faena: la operación es continua 24x7x365 y el cambio de turno de las 08:00 y las 20:00 es el momento de mayor consecuencia, de modo que la cobertura de guardia no puede tener su propio relevo en esa franja.

### 12.4 Qué no puede aparecer en un registro (RT-14.07)

Los registros de la solución **no contienen datos personales sensibles ni credenciales**, y su acceso está controlado y auditado. En este caso la regla se aplica a un contenido concreto: un registro puede consignar que se consultó la habilitación de una persona, con su identificador y el resultado de la validación, y **no puede consignar el contenido de esa habilitación** —el examen, su resultado, la condición de salud, el resultado del control de fatiga o de alcohol y drogas—, que es dato del nivel personal sensible del apartado 11.3 y vive cifrado a nivel de campo.

La consecuencia de diseño es que el filtrado ocurre **en la instrumentación y no en la ingesta**: el dato sensible no sale del componente hacia la plataforma de observabilidad, en vez de salir y ser depurado después. Un filtro en la ingesta deja el dato en tránsito y en la memoria del recolector, y eso ya es un tratamiento bajo la Ley N.º 21.719.

### 12.5 Acceso del CLIENTE a los tableros (RT-14.02)

El CLIENTE dispone de **acceso propio y permanente** a los tableros operacionales y de negocio, con datos en tiempo real y capacidad de exportación, sin intervención del ADJUDICATARIO. No es una concesión: es condición de la transferencia del numeral 17.6 y del hecho de que el equipo de once personas del CLIENTE recibe la operación al término del Contrato. El acceso respeta el control de acceso por rol, área y sitio, y la exportación respeta la clasificación del apartado 11.3: un tablero no es una vía para sacar la ley por polígono.

### 12.6 Retención de métricas, registros y trazas (RT-14.08)

| Señal | En línea | Archivo recuperable | Por qué |
|---|---|---|---|
| Métricas | 13 meses | — | Trece meses permiten comparar un mes contra el mismo mes del año anterior, que es como se lee una operación con estacionalidad de detenciones programadas en marzo y septiembre |
| Registros de aplicación | 3 meses | 12 meses | Cubre el ciclo de diagnóstico sin retener contenido operacional más allá de lo necesario |
| Registros de eventos de seguridad | 12 meses | 24 meses adicionales | Lo fija el RT-11.14 y se cumple en el apartado 11.7 |
| Trazas distribuidas | 30 días, con muestreo | 90 días para las trazas de transacciones que fallaron | El muestreo no se aplica a las trazas con error ni a las de terreno: se conservan íntegras |
| Asiento de auditoría | Según RN-28, por tipo de registro | Sin borrado anticipado | No es observabilidad: es registro de negocio, y lo gobierna RN-28 |

El costo asociado se declara en la Oferta Económica, distinguiendo el almacenamiento en línea del archivado, conforme al requisito.

### 12.7 Detección proactiva de anomalías (RT-14.09)

Se compromete, y se acota a donde el comportamiento histórico existe y es estable: tasa de registro por sitio y por turno, tiempo de ciclo del viaje, deriva de reloj por dispositivo y latencia de ingesta por fabricante de telemetría. La alerta se emite **antes de que el incidente afecte la operación**, y se declara explícitamente lo que no hace: no es analítica predictiva de negocio, que es el RT-05.30 y no se compromete en este apartado.

### 12.8 Trazabilidad de este apartado

| Requisito | Dónde se acredita | Componente que lo satisface |
|---|---|---|
| RT-14.01, RT-03.16 | Apartados 4 capa 8 y 12.1 | C-22 — Plataforma base; C-04 — Nodo de borde |
| RT-05.19 | Apartados 8.3 y 12.1 | C-22 y los siete adaptadores |
| RT-14.02 | Apartado 12.5 | C-19 — Analítica, indicadores y reportería; C-22 |
| RT-14.03 | Apartado 12.2 | C-22; C-03 — Captura en terreno |
| RT-14.04 | Apartado 12.3 | C-22, con las señales de C-01, C-04, C-05, C-08, C-15 y C-17 |
| RT-14.07 | Apartado 12.4 | C-22; C-05 |
| RT-14.08 | Apartado 12.6 | C-22 |
| RT-14.09 | Apartado 12.7 | C-22 |

---

## 13. Capacidad, configuración y despliegue sin interrupción

Este apartado reúne las decisiones de arquitectura que hacen que el sistema soporte su carga, cambie de parámetros y cambie de versión **sin detener una operación que no se detiene**. Las tres están gobernadas por la misma restricción: la n.º 1.

### 13.1 Escalado ante un peak predecible (RT-09.04, RT-02.10)

Las capas de aplicación e integración escalan de forma **horizontal y automática**. La decisión propia de este caso es *cómo* se dispara ese escalado.

El peak dimensionante no es aleatorio: es el **cambio de turno**, ocurre dos veces al día a las 08:00 y a las 20:00 conforme a RN-23, dura de 45 a 60 minutos y alcanza 200 transacciones por segundo contra 40 en régimen (SUP-31). Un escalado puramente reactivo llega tarde por construcción: reacciona cuando la métrica ya subió, y para cuando la capacidad está disponible el peak lleva un tercio de su duración transcurrido, justo en la franja en que se abren y cierran mil ochocientas sesiones de operador.

**Decisión: escalado anticipado por calendario como mecanismo primario, y escalado reactivo como red de seguridad.** La capacidad se provisiona antes de cada cambio de turno, anclada a la definición de turno de RN-23 —que es un parámetro versionado de C-20, no una constante—, y el escalado reactivo cubre lo que el calendario no prevé: una detención programada de planta, una reconciliación masiva tras un corte, un embarque. El tiempo de reacción del mecanismo reactivo se declara y se verifica en las pruebas de carga del Subdocumento 9.

**Sin pérdida de transacciones en curso.** El requisito lo exige y aquí se materializa en dos propiedades ya declaradas: los servicios de negocio no guardan estado en memoria (RT-02.05), de modo que retirar una instancia no pierde una sesión; y toda escritura reintentable es idempotente con clave declarada (RT-02.06), de modo que un reintento tras el retiro de una instancia no duplica el hecho. El drenaje de conexiones antes del retiro es condición de despliegue del apartado 13.5.

### 13.2 Degradación controlada al superarse la capacidad (RT-09.08)

Al superarse la capacidad la solución **encola, limita la tasa y lo informa explícitamente; nunca devuelve un error genérico ni pierde una transacción en silencio**. El comportamiento se diferencia por origen, y la diferencia es la que protege el criterio n.º 14:

| Origen | Comportamiento bajo saturación |
|---|---|
| Terreno (C-03, C-04) | **Nunca recibe rechazo.** La contrapresión se absorbe donde el diseño ya la absorbe: el evento queda en la cola local firmada, que es la misma que sostiene las 48 horas sin enlace, y la aplicación informa el estado de sincronización que muestra de manera permanente (apartado 9.4). Para la persona operadora, saturación y corte de enlace son el mismo comportamiento, y no tiene que distinguirlos |
| Portales externos (C-06, C-07) | Límite de tasa por consumidor con respuesta explícita y tiempo de reintento sugerido. Es la primera carga que se sacrifica, por ADR-07 |
| Analítica y reportería (C-19) | Encolamiento de la consulta con aviso de espera; las exportaciones de gran volumen ya son asíncronas por RT-16.29 |
| Integraciones (C-09 a C-14) | Cola persistente por adaptador con reintento y retroceso exponencial; el cortacircuito abre antes de propagar la saturación al núcleo (RT-02.08) |

El orden de sacrificio es una decisión, no una consecuencia: primero el portal del comprador, después la analítica, después las integraciones, y **el registro de terreno nunca**.

### 13.3 Configuración técnica y parámetros de negocio son cosas distintas (RT-04.08)

Toda configuración está **externalizada del artefacto y gestionada por ambiente**, de modo que un mismo artefacto se promueve de QA a Preproducción y a Producción sin recompilación. Sobre esa base, la decisión propia de este caso es distinguir dos cosas que suelen confundirse en un solo mecanismo:

| | Configuración técnica | Parámetro de negocio |
|---|---|---|
| Qué es | Puntos de conexión, tamaños de lote, tiempos de espera, credenciales de integración | Umbrales, plazos, tolerancias, catálogos, taxonomía de detenciones, factores de emisión, definición de turno |
| Dónde vive | Almacén de configuración por ambiente, fuera del artefacto | **C-20**, con vigencia temporal versionada |
| Quién la cambia | El equipo de plataforma, por el flujo de despliegue | El CLIENTE, desde la interfaz de administración (RT-16.02), con aprobación de un segundo perfil si tiene impacto operacional (RT-16.03) |
| Efecto de un cambio | Rige desde el despliegue; no tiene historia | **Rige desde su fecha de vigencia, y un período anterior se recalcula con la versión que regía entonces** (RN-27) |
| Auditoría | Registro de cambio de configuración | Asiento de auditoría con autor, fecha y justificación, conservado según RN-28 |

Confundirlas rompería el requisito más consecuente de la capa 4: si un umbral de conciliación fuera configuración técnica, cambiarlo reescribiría el pasado y los criterios de aceptación n.º 3 y n.º 4 dejarían de ser verificables sobre historia ya cerrada. Los secretos no son ninguna de las dos: residen en un gestor de secretos con rotación automática y auditoría de acceso, y ninguna credencial se embebe en código, imagen ni archivo de configuración.

### 13.4 Ambiente de simulación de cambios de parámetro (RT-16.05)

Existe un **ambiente de simulación que permite probar el efecto de un cambio de parámetro antes de aplicarlo a producción**. Se compromete, y en este caso tiene una forma concreta que el requisito genérico no anticipa: la simulación se ejecuta **sobre una copia de historia ya cerrada**, no sobre datos sintéticos, porque lo que se necesita saber antes de cambiar un factor de conciliación o una tolerancia es *qué habría pasado con los meses que ya se cerraron*.

Es la misma capacidad que RN-27 exige para el recálculo retroactivo, usada en sentido prospectivo, y por eso no es un componente nuevo: es una función de C-20 sobre el repositorio analítico. La copia de historia respeta la ofuscación del apartado 11.8 para la ley por polígono y excluye el dato personal sensible.

### 13.5 Despliegue sin interrupción y migraciones de esquema (RT-04.07, RT-04.10)

**Estrategia declarada: despliegue canario**, con progresión por sitio y verificación en Preproducción antes de cada paso a producción. Se escoge sobre azul-verde por una razón del caso: los siete sitios no son equivalentes ni fallan igual —el acopio de puerto está en recinto de terceros con conectividad de un operador sin acuerdo de nivel de servicio (SUP-19)— y el canario permite avanzar sitio por sitio y detenerse en el primero que se degrade, mientras que azul-verde conmuta todo a la vez y obliga a revertir todo a la vez.

La progresión **nunca cruza un cambio de turno**: el registro de terreno no se toca durante el relevo, por la misma razón que el apartado 3.2 usa para justificar el despliegue independiente. Las ventanas son las de RT-10.05: intervenciones mayores sólo en las detenciones programadas de planta de marzo y septiembre; menores, previa aprobación, entre las 03:00 y las 05:00; **prohibidas durante el cierre contable del mes y durante las 48 horas previas a un embarque** (RN-24).

**Migraciones de esquema (RT-04.10):** versionadas, reversibles y automatizadas, con estrategia de **expansión y contracción** que permite convivir dos versiones de la aplicación durante el despliegue. Primero se agrega la estructura nueva sin retirar la anterior; se despliega la versión que escribe en ambas; se migra el dato; se despliega la versión que lee sólo la nueva; y sólo entonces se retira la anterior, en un despliegue posterior. En este sistema la propiedad no es opcional por dos razones: la operación no se detiene (restricción n.º 1), y un nodo de borde que estuvo 48 horas sin enlace **puede reconciliar eventos generados por la versión anterior de la aplicación de terreno**, de modo que el esquema debe aceptar durante toda la ventana de convivencia lo que produjo la versión que quedó aislada.

### 13.6 Administración remota de dispositivos de borde y de terreno (RT-03.18)

Los dispositivos de borde y de terreno se administran de forma remota y centralizada: **inventario, configuración, actualización de firmware y de aplicación, bloqueo y borrado remoto**. Es capacidad de C-22 sobre el catálogo de dispositivos de C-20, y no es accesoria: el inventario es condición de la propia autenticación, porque RF-IDE-003 admite el toque de la credencial sólo **sobre un dispositivo previamente inventariado**. Un dispositivo que no está en el inventario no abre sesión.

El bloqueo y el borrado remoto son el control compensatorio de la pérdida física de un dispositivo que contiene hasta 48 horas de registro local cifrado, y se ejecutan en cuanto el dispositivo recupera enlace. Mientras no lo recupera, la protección es el cifrado en reposo y la firma del apartado 11.5. La actualización de aplicación **se aplaza mientras el dispositivo tenga eventos pendientes de reconciliación**: primero se sincroniza, después se actualiza, nunca al revés.

### 13.7 Procesamiento en el borde (RT-03.19)

Se compromete, acotado a lo que reduce volumen sin trasladar lógica de negocio al borde, que es la frontera que el apartado 6.2 declara para el contexto Terreno:

- **Filtrado y agregación previa de telemetría y de señales del historiador**: se transmite el cambio de estado y la agregación por intervalo, no la muestra cruda. Es lo que hace sostenible el ancho de banda de siete sitios y lo que permite que la ingesta de 52 equipos con telemetría, proyectados a 70, no compita con el registro operacional.
- **Detección local de la detención de un equipo** a partir de las señales que el nodo de borde ya recibe del despacho, para que la detección siga operando sin enlace conforme al apartado 9.3.
- **Deduplicación y compactación de la cola** antes de sincronizar, de modo que el lote de reconciliación tras 48 horas transporte hechos y no reintentos.

No se ejecuta en el borde la clasificación de la detención con taxonomía cerrada, ni el cálculo de la ley, ni el cumplimiento del plan: son lógica de negocio de los contextos Activos y Mineral, y el apartado 6.2 declara expresamente que no pertenecen al Terreno.

### 13.8 Reversibilidad y mitigación del bloqueo por proveedor (RT-03.07)

El requisito exige identificar qué componentes son portables, cuáles no lo son y cuál sería el esfuerzo de una migración. La declaración lógica es la siguiente; la cuantificación del esfuerzo y la identificación del proveedor concreto son materia del Subdocumento 4.2.

| Grado | Qué comprende | Fundamento |
|---|---|---|
| **Portable sin rediseño** | Los diez servicios de negocio de la capa 4, el nodo de borde C-04 y la aplicación de terreno C-03 | Sin estado (RT-02.05), contrato versionado y ejecución en contenedor. La decisión de grano medio de ADR-01 es lo que los mantiene portables: una malla de servicios de grano fino habría atado la solución a la plataforma que la orquesta |
| **Portable con reescritura acotada** | Bus de eventos, puerta de enlace, plataforma de observabilidad | El contrato es estándar —AsyncAPI 3.0, OpenAPI 3.1, OpenTelemetry— y por eso lo que se reescribe es la integración, no la lógica. Es el propósito de exigir esos contratos en el RT-05.16 |
| **Atado al proveedor** | Servicios administrados de persistencia, gestión de claves y análisis de la capa de datos | Es una atadura **deliberada**: RT-03.05 manda privilegiar servicios administrados cuando reduzcan el riesgo operacional, y once personas de TI no pueden operar bases de datos autoadministradas al término del Contrato. Se acepta el bloqueo y se declara, en vez de fingir portabilidad |
| **Independiente por diseño** | Los siete sistemas del CLIENTE | Capa anticorrupción y adaptador reemplazable (RT-02.14): sustituir un sistema externo cambia un adaptador y no toca el núcleo |

La abstracción de proveedor se aplica en la capa de datos y en la de identidad, conforme al apartado 10, y el dato es exportable en formato abierto en todo momento, que es la condición práctica de la reversibilidad.

### 13.9 Trazabilidad de este apartado

| Requisito | Dónde se acredita | Componente que lo satisface |
|---|---|---|
| RT-02.10, RT-09.04 | Apartados 10 y 13.1 | C-22 — Plataforma base |
| RT-09.08 | Apartado 13.2 | C-22; C-03 — Captura en terreno; C-04 — Nodo de borde |
| RT-04.08 | Apartado 13.3 | C-22; C-20 — Administración, parametrización y auditoría |
| RT-16.05 | Apartado 13.4 | C-20; C-19 — Analítica, indicadores y reportería |
| RT-04.07 | Apartado 13.5 | C-22 |
| RT-04.10 | Apartado 13.5 | C-22; C-21 — Migración de datos históricos |
| RT-03.18 | Apartado 13.6 | C-22; C-20; C-03; C-04 |
| RT-03.19 | Apartado 13.7 | C-04; C-13 — Adaptadores de telemetría; C-14 — Lectura del historiador |
| RT-03.07 | Apartado 13.8 | Todos los componentes, con el detalle de esfuerzo en el Subdoc. 4.2 |
| RT-03.04 | Apartados 11.4 y 13.8 | C-22, con el diseño de subredes en el Subdoc. 4.2 |

---

## 14. Trazabilidad de este subdocumento

### 14.1 Requisitos transversales que acredita

Este subdocumento es la sección de la propuesta que acredita, en el Formulario T-12, los requisitos que hasta la v1.2 figuraban en blanco por inexistencia del entregable:

Los requisitos que este subdocumento acredita, con el apartado en que cada uno queda declarado y verificable:

| Capítulo transversal | Requisitos que acredita este subdocumento | Apartados |
|---|---|---|
| **02 · Arquitectura** | RT-02.01 · RT-02.02 · RT-02.03 · RT-02.04 · RT-02.05 · RT-02.06 · RT-02.07 · RT-02.08 · RT-02.09 · RT-02.10 · RT-02.11 · RT-02.13 · RT-02.14 · numeral 2.3 | 1.1 · 1.2 · 3 · 3.3 · 4 · 6 · 7 · 8.1 · 8.3 · 9.4 · 10 · 10.2 · 17 · 18 |
| **03 · Modelo híbrido** | RT-03.04 · RT-03.07 · RT-03.11 · RT-03.13 · RT-03.16 · RT-03.18 · RT-03.19 | 9 · 9.3 · 4 capa 8 · 11.4 · 12.1 · 13.6 · 13.7 · 13.8 |
| **04 · Entrega continua** | RT-04.07 · RT-04.08 · RT-04.10 | 13.3 · 13.5 |
| **05 · Datos e integración** | RT-05.05 · RT-05.16 · RT-05.17 · RT-05.18 · RT-05.19 · RT-05.23 · RT-05.29 | 4 capa 6 · 8.1 · 8.3 · 12.1 · ADR-08 |
| **06 · Site on-premise** | RT-06.20 | 11.9 |
| **09 · Desempeño y capacidad** | RT-09.04 · RT-09.08 | 13.1 · 13.2 |
| **11 · Seguridad de la información** | RT-11.01 · RT-11.02 · RT-11.03 · RT-11.07 · RT-11.08 · RT-11.09 · RT-11.11 · RT-11.12 · RT-11.13 · RT-11.14 · RT-11.15 · RT-11.25 · RT-11.27 | 11.1 a 11.10 |
| **12 · Identidad y acceso** | RT-12.03 · RT-12.04 | 4 capa 7 · 11.9 |
| **13 · Usabilidad** | RT-13.12 | 16.5 |
| **14 · Observabilidad** | RT-14.01 · RT-14.02 · RT-14.03 · RT-14.04 · RT-14.07 · RT-14.08 · RT-14.09 | 12.1 a 12.8 |
| **15 · Sostenibilidad** | RT-15.02, en su lectura transversal | 16.6 |
| **16 · Módulos transversales** | RT-16.05 · RT-16.26 | 13.4 · 16.7 |
| **17 · Canales y movilidad** | RT-17.02 · RT-17.08 | 16.2 · 16.3 · 16.4 |

Cada apartado cierra con su propia tabla de trazabilidad requisito por requisito y componente por componente: 11.10 para la seguridad, 12.8 para la observabilidad, 13.9 para la capacidad y el despliegue, y 16.7 para los canales.

**RT-02.12** no figura en la tabla porque ya estaba declarado «Cumple» en la v1.2 del Formulario T-12, con C-20 como componente. Este subdocumento lo desarrolla igualmente en el apartado 10 y no altera esa declaración.

**RT-03.24 se acredita a medias y así se declara:** la calidad de servicio y la priorización del tráfico operacional frente al administrativo están en el apartado 4, capa 3; el estudio de cobertura del rajo con las zonas de sombra y su proyección a tres años, que es lo que el caso asocia a ese código, es materia del Subdocumento 4.2 y del RF-DES-013.

**Actualización que este entregable obliga a hacer en el Formulario T-12.** Las filas del capítulo 2 y las de los demás códigos de la tabla anterior estaban en blanco y quedan declaradas con los apartados que aquí se indican. La fila de **RT-03.13** figuraba como «Cumple parcialmente» con la nota de que la declaración se remitía a este subdocumento; el apartado 9.3 la entrega y pasa a «Cumple». La celda de RT-03.13 que remite a este subdocumento el producto y la versión debe corregirse: la individualización de producto y versión es materia del Subdocumento 4.2, conforme al apartado 1.3 y al apartado 15.

### 14.2 Componente por criterio de aceptación del Capítulo 18

| Criterio | Qué se compromete | Componentes que lo sostienen |
|---|---|---|
| n.º 1 | Origen de cualquier lote embarcado con evidencia en menos de 2 horas, mes 15 | C-01, C-09, C-03, C-04, C-10, C-07, C-22 |
| n.º 2 | Conciliación cerrada en no más de 3 días hábiles, mes 19 | C-08, C-09, C-01, C-12 |
| n.º 3 | Diferencia bajo 2 % al mes 21, con 100 % atribuido a un punto | C-08, C-09, C-01 |
| n.º 4 | Cumplimiento del plan al inicio del turno siguiente, contra la versión vigente | C-02, C-19, C-01, C-11 |
| n.º 5 | 100 % de detenciones clasificadas antes del cierre del turno siguiente, mes 15 | C-15, C-19, C-10, C-13 |
| n.º 6 | Estado, historial, alarmas y pautas en una sola vista. Base mes 15, completa mes 21 | C-13, C-16, C-10, C-12 |
| n.º 7 | Cero rechazos en portería por documentación vencida advertible, mes 15 | C-05, C-06, C-04, C-22 |
| n.º 8 | Acreditación de contratista en menos de 48 horas, mes 15 | C-06, C-05, C-20 |
| n.º 9 | 100 % de registros de seguridad nacidos digitales, mes 15 | C-20, C-05, C-03, C-22, C-18 |
| n.º 10 | 100 % de alertas de fatiga con acción documentada, mes 21 | C-18, C-20, C-16 |
| n.º 11 | 100 % de reportes ambientales generados desde el sistema, mes 21 | C-17, C-19, C-20, C-09, C-18 |
| n.º 12 | Huella de carbono mensual auditable, mes 18 | C-17, C-20, C-19, C-01, C-14 |
| n.º 13 | 48 horas sin enlace y sincronización en 4 horas, desde marcha blanca | C-04, C-22, C-03 |
| n.º 14 | 90 % de eventos registrados sin intervención del supervisor, mes 15 | C-22, C-03, C-05, C-21 |

**Cómo leer esta tabla frente a la matriz de trazabilidad.** El orden de cada fila es el del peso con que cada componente sostiene el criterio en la matriz v1.1, medido por número de requerimientos asociados. La tabla incluye además algún componente que la matriz no asocia formalmente al criterio pero sin el cual el compromiso no se sostiene —C-03 y C-04 en el criterio n.º 1, porque el dato que se reconstruye nace ahí—. Donde la matriz y esta tabla difieran en la lista, la matriz manda para efectos de trazabilidad de requerimientos, y esta tabla explica el diseño.

---

## 15. Qué falta y de qué subdocumento proviene

| Qué falta | De quién | Insumo exacto |
|---|---|---|
| Producto y versión concretos de cada componente C-xx | Subdoc. 4.2 | El numeral 1.5 transversal advierte que declarar «Cumple» sin individualizar producto y versión equivale a no declarar, y la Comisión califica el requisito como **no acreditado**. Es el pendiente de mayor riesgo de puntaje que este subdocumento no puede cerrar solo |
| Tabla de emplazamiento componente por componente, nube y faena, con justificación | Subdoc. 4.2 | Exigida por el Artículo 16.2. Determina qué se sostiene en faena durante el corte de fibra |
| Especificación de la sala técnica y de los cuatro gabinetes de borde | Subdoc. 4.2 | Sitios, dimensionamiento y condiciones. La sala actual de 40 m² no cumple el estándar del Capítulo 6 |
| Especificación del hardware de terreno | Subdoc. 4.2, Formulario T-11 | Referencia, cantidad y características por sitio (RT-08.10) |
| Modelo de datos físico, linaje y catálogo | Subdoc. 5 | Debe soportar las entidades y eventos del apartado 7 de este subdocumento |
| Paquetes de la EDT | Informe 2 | Las 228 filas de la matriz de trazabilidad dicen «Se define en el Informe 2». La columna existe vacía |
| Detalle de emplazamiento, producto y versión de los controles de seguridad, observabilidad e infraestructura | Subdoc. 4.2 | Este subdocumento decide el **modelo y los controles lógicos** de seguridad (apartado 11), observabilidad (12) y capacidad, configuración y despliegue (13). El producto, la versión, el emplazamiento y el dimensionamiento son del 4.2, y sin ellos el numeral 1.5 transversal considera el requisito no acreditado |

**Frontera de alcance frente al Formulario T-12.** La hoja «Pendientes por grupo» del T-12 v1.2 asignó a «Subdoc. 4.1» noventa y siete códigos, incluidos capítulos completos que no son materia de arquitectura lógica. Este subdocumento acredita los cincuenta y dos que sí lo son, según la tabla del apartado 14.1, y **reasigna los cuarenta y cinco restantes con fundamento**, para que ninguno quede huérfano entre entregables. La reasignación se refleja en la versión siguiente del Formulario T-12.

| Códigos que se reasignan | A qué entregable | Por qué no son arquitectura lógica |
|---|---|---|
| RT-03.01 · RT-03.02 · RT-03.03 · RT-03.05 · RT-03.06 · RT-03.08 · RT-03.09 · RT-03.14 · RT-03.15 · RT-03.17 · RT-03.20 · RT-03.21 · RT-03.22 | **Subdoc. 4.2 — Arquitectura física** | Proveedor de nube, regiones, zonas de disponibilidad, infraestructura como código, servicios administrados, FinOps, nivel RAID, endurecimiento CIS, redundancia del enlace, ancho de banda por sitio, enlace privado y acceso remoto son decisiones de emplazamiento y dimensionamiento. El apartado 1.1 declara la vista de despliegue como propia del 4.2, y el Artículo 16.2 exige allí la tabla de emplazamiento |
| RT-04.01 · RT-04.02 · RT-04.03 · RT-04.04 · RT-04.05 · RT-04.06 · RT-04.09 · RT-04.11 · RT-04.12 · RT-04.13 · RT-04.14 | **Subdoc. 6 y 7 — Metodologías y plan de trabajo**, con el dimensionamiento de ambientes en el Subdoc. 4.2 | Ambientes, ramas protegidas, flujo de integración continua, cobertura de pruebas, métricas de entrega y ambientes efímeros son proceso de construcción, no estructura de la solución. Las tres decisiones de este capítulo que **sí** alteran la arquitectura —configuración externalizada, estrategia de despliegue y migraciones de esquema compatibles— se deciden aquí, en los apartados 13.3 y 13.5 |
| RT-09.06 · RT-09.07 · RT-09.09 · RT-09.10 | **Subdoc. 9 — Calidad y pruebas**, con la capacidad en Operación en el Subdoc. 10 | Ejecutar las pruebas de carga, informarlas y gestionar la capacidad durante la Operación son actividades. Lo que la arquitectura debe decidir —cómo escala y cómo degrada— está en los apartados 13.1 y 13.2 |
| RT-11.04 · RT-11.05 · RT-11.06 · RT-11.16 · RT-11.17 · RT-11.18 · RT-11.19 · RT-11.21 · RT-11.22 · RT-11.23 · RT-11.24 · RT-11.26 · RT-11.28 | **Subdoc. 10 y 11 — Operación** (gestión de vulnerabilidades, detección y respuesta, centro de operaciones de seguridad, respuesta a incidentes, notificación de brechas, ejercicios), **Subdoc. 6 y 7** (cadena de suministro de software, inventario de componentes, firma de artefactos, aprobación de dependencias, madurez del desarrollo seguro) y **Subdoc. 4.2** (matriz de controles ISO/IEC 27001 y 27002, controles 27017 y 27018) | Son programas, procedimientos y matrices de control con evidencia documental. La arquitectura decide el modelo de seguridad, las fronteras, la clasificación y los controles lógicos, y eso está en el apartado 11 |
| RT-14.05 · RT-14.06 | **Subdoc. 10 y 11 — Operación** | El libro de operación, la guía de resolución y el análisis de causa raíz son entregables de la Operación. Lo que la arquitectura decide —qué se instrumenta, sobre qué se mide el nivel de servicio y qué dispara una alerta— está en el apartado 12 |
| RT-16.24 · RT-16.33 | **Oferta Económica** y **Subdoc. 10** | El proveedor y el costo unitario de cada canal de notificación y la estimación de reducción de la atención asistida son compromisos económicos y de servicio, no estructura |



**Responsable de la consolidación.** Antes de la entrega, el equipo de este subdocumento verifica que los nombres de componente, los nombres de sitio y la lista de funciones degradadas del apartado 9.3 coincidan **literalmente** entre los Subdocumentos 3, 4.1, 4.2 y 5.

---

## 16. Canales, movilidad y decisión de la aplicación de terreno

Este apartado cierra la decisión que el apartado 4 describe funcionalmente sin tomar: qué clase de aplicación es el componente **C-03 — Captura en terreno, aplicación móvil con operación desconectada**. El RT-17.02 no admite describir la aplicación sin declarar su naturaleza, y exige justificarla frente a tres dimensiones concretas: la operación desconectada, el acceso a periféricos y el costo de mantención. Es una decisión de arquitectura de canal y no una elección de producto, y por eso se resuelve aquí y no en el Subdocumento 4.2.

### 16.1 Las cuatro exigencias que acotan la decisión

| Exigencia | Origen | Por qué acota la decisión |
|---|---|---|
| Operar 48 horas continuas sin enlace, con registro local firmado y cifrado a nivel de campo, reloj monótono, sello de tiempo propio e identificador de evento generado en el dispositivo | SUP-09, RN-30, criterio de aceptación n.º 13 | No es una caché de conveniencia: es un registro de eventos con integridad criptográfica que debe sobrevivir al cierre de la aplicación, al reinicio del dispositivo y al agotamiento de batería |
| Leer la credencial personal de proximidad en cabina, sin retirar el guante, en un segundo desde la lectura hasta abrir la sesión | SUP-04, SUP-29, RN-22, RF-IDE-003 | Es acceso a periférico de comunicación de campo cercano. Determina qué capacidades del dispositivo debe alcanzar la aplicación |
| Operar con guantes gruesos y lentes de seguridad, a la intemperie, bajo sol directo y hasta −6 °C, en dispositivos compartidos por turno, con personas sin correo corporativo, y sin contraseña alfanumérica en terreno | RT-12.11, RT-13.08, RNF-USA-002 | Fija el tamaño del objetivo táctil, el contraste y el modelo de sesión, no la tecnología |
| Sostener el sistema operativo móvil vigente y las dos versiones anteriores | RT-17.03, RNF-POR-004 | Con dispositivos compartidos y renovación desigual del parque, el rango de versiones soportadas es amplio y estable en el tiempo |

A ello se suma una restricción que no viene del terreno sino del contrato: el equipo de TI del CLIENTE es de **once personas** (restricción n.º 10) y recibe la operación al término del Contrato, conforme al numeral 17.6. El costo de mantención que el RT-17.02 manda ponderar no es el del ADJUDICATARIO durante la implementación, sino el del CLIENTE durante los treinta y seis meses de Operación y después de ellos.

### 16.2 Comparación de las tres alternativas que el RT-17.02 enumera

| Dimensión que exige el RT-17.02 | Web progresiva | Nativa por plataforma | **Híbrida con núcleo web y contenedor nativo** |
|---|---|---|---|
| Operación desconectada 48 h | Insuficiente. El almacenamiento del navegador es desalojable por el sistema operativo bajo presión de memoria o de disco, y no ofrece garantía de persistencia para un registro con valor probatorio. El acceso a un almacén de claves respaldado por hardware no está garantizado, y sin él la firma del registro local queda expuesta | Suficiente y directa | **Suficiente.** El contenedor nativo aporta almacenamiento persistente no desalojable y acceso al almacén de claves del dispositivo; el núcleo web no participa de la persistencia |
| Acceso a periféricos | Insuficiente. La lectura de credencial de proximidad no está disponible de forma uniforme y estable en el rango de versiones que el RT-17.03 obliga a soportar | Suficiente | **Suficiente.** La lectura de proximidad se implementa en el contenedor nativo y se expone al núcleo web por una interfaz propia, acotada y versionada |
| Costo de mantención | Bajo: una base de código | Alto: dos bases de código nativas, dos ciclos de publicación y dos matrices de prueba, sostenidos por once personas después del Contrato | **Bajo en la mayor parte:** una sola base de código para la lógica de captura, la interfaz y el registro local; superficie nativa mínima, acotada a proximidad, persistencia, almacén de claves y ciclo de vida en segundo plano |

### 16.3 Decisión declarada (RT-17.02)

La aplicación de terreno **C-03 — Captura en terreno, aplicación móvil con operación desconectada** es **híbrida**: un núcleo web único que contiene la interfaz, la lógica de captura y la cola de sincronización, embebido en un contenedor nativo por plataforma cuya superficie se limita a cuatro capacidades que el núcleo web no puede garantizar por sí mismo:

1. **Almacenamiento persistente no desalojable** para el registro local de eventos, que es la pieza sobre la que descansan las 48 horas sin enlace y el RPO de cero eventos perdidos de SUP-32.
2. **Almacén de claves respaldado por hardware**, que sostiene la firma del evento y el cifrado a nivel de campo que exigen SUP-09 y RN-29.
3. **Lectura de credencial de proximidad**, que sostiene la identificación del operador en el relevo (RN-22) dentro del segundo que fija SUP-29.
4. **Ciclo de vida en segundo plano**, para que la reconciliación se reanude al recuperarse el enlace sin que nadie abra la aplicación, que es lo que hace que la sincronización ocurra «sin intervención manual» como exige el criterio de aceptación n.º 13.

Se descarta la **web progresiva** porque falla en las dos dimensiones que el propio requisito nombra primero, y hacerlo sobre el registro que da valor probatorio a los datos de seguridad —origen de seis de las once observaciones de la última auditoría— sería un riesgo desproporcionado. Se descarta la **nativa por plataforma** porque su ventaja marginal sobre la híbrida se concentra en capacidades que este caso no usa, mientras su costo de mantención recae sobre un equipo de once personas al término del Contrato, que es exactamente el criterio que el numeral 2.3 manda ponderar y el mismo que sostiene ADR-01.

La decisión es reversible en la dirección que importa: la superficie nativa está aislada tras una interfaz propia y acotada, de modo que sustituir el contenedor no obliga a reescribir la lógica de captura. Es el mismo patrón de abstracción de proveedor que el RT-02.14 valora y que el apartado 10 declara para las demás capas.

**Alcance de la decisión.** Rige para C-03. Los componentes de presentación que no operan en terreno —**C-06** Portal de empresas contratistas, **C-07** Portal de trazabilidad de lote para compradores, **C-16** Vista única de equipo y apoyo al mantenimiento, **C-19** Analítica, indicadores y reportería y la consola de **C-20** Administración, parametrización y auditoría— son aplicaciones web servidas por la capa de borde y exposición, sin contenedor nativo, porque ninguno de los cuatro fundamentos anteriores les aplica.

### 16.4 Cobertura en dispositivos de bajo costo y de generaciones anteriores (RT-17.08)

El RT-17.08 es deseable y se compromete. La decisión híbrida lo favorece: el núcleo web se ejecuta sobre el motor de presentación del propio sistema operativo, de modo que la línea base de hardware la fija el dispositivo y no un tiempo de ejecución adicional.

Se compromete una **interfaz reducida** de C-03, servida por el mismo núcleo y sin base de código separada, que se activa automáticamente cuando el dispositivo declara memoria o resolución bajo el umbral. La interfaz reducida conserva íntegras las funciones de registro —identificación por proximidad, botón único de aviso de cambio de material, registro de detención, inspección e incidente— y suprime las de consulta y visualización, que se resuelven en el terminal del supervisor. La razón es de alcance del criterio de aceptación n.º 14: el compromiso de adopción se mide sobre eventos registrados, y ninguna función de registro puede quedar fuera del alcance de un dispositivo por su generación.

Esto importa por la composición del parque: hay **310 vehículos de flota liviana, mayoritariamente de contratistas**, y las personas contratistas no usan dispositivos provistos por el CLIENTE. El hardware de terreno que el CLIENTE adquiere se especifica en el Formulario T-11 conforme al RT-08.10, pero la aplicación no puede suponer que todo dispositivo que la ejecuta salió de esa especificación.

**Umbral definido por el PROPONENTE**, a incorporar al Registro de supuestos: la interfaz reducida sostiene los umbrales de terreno del numeral 9.1 en dispositivos de dos generaciones anteriores a la vigente, con la misma tasa de éxito de lectura de proximidad. Se valida en la marcha blanca de la Etapa 1, sobre el parque real y no sobre el especificado.

### 16.5 Modo oscuro, personalización e idiomas (RT-13.12)

El RT-13.12 es deseable y se compromete de forma desigual entre sus tres partes, porque en esta faena sólo una de las tres resuelve un problema real.

| Parte del requisito | Compromiso | Fundamento |
|---|---|---|
| **Modo oscuro** | Se compromete, y no como preferencia estética sino como requisito operacional del turno de noche, de 20:00 a 08:00 (RN-23). La aplicación conmuta de forma automática por hora de turno y por luz ambiente, con conmutación manual disponible. El modo claro conserva el alto contraste que exige RNF-USA-002 para el sol directo | La cabina de noche es un entorno de visión adaptada a la oscuridad: una pantalla clara la destruye durante varios minutos. Es seguridad operacional |
| **Personalización por persona usuaria** | Se compromete **sólo en los componentes de oficina** —C-16, C-19 y la consola de C-20— y **no en C-03**. En terreno la interfaz es fija | Los dispositivos de terreno son **compartidos por turno** (RT-12.11): una preferencia personal en una pantalla compartida produce una interfaz distinta en cada relevo, lo que contradice SUP-35, que compromete capacitación en no más de treinta minutos, y erosiona el criterio de aceptación n.º 14 |
| **Múltiples idiomas** | La interfaz se entrega en **español de Chile**, con la terminología del glosario del Capítulo C del caso. La arquitectura queda preparada para un idioma adicional —textos externalizados, sin cadenas embebidas en el código— pero **no se compromete un segundo idioma en el alcance del Contrato** | El requisito lo condiciona a que el caso lo justifique, y el caso no lo justifica: no declara personal que no opere en español. Preparar la arquitectura no cuesta; traducir y mantener dos juegos de terminología minera, sí |

La accesibilidad conforme a **WCAG 2.2 nivel AA** (RNF-USA-007, RT-13.01) y la navegación íntegra por teclado con orden de foco lógico (RT-13.11) rigen para los componentes de oficina, y el modo oscuro se verifica contra los mismos umbrales de contraste.

### 16.6 Apagado de ambientes no productivos y certificaciones sectoriales (RT-15.02)

El código RT-15.02 es uno de los que el Subdocumento 3 declaró **desalineados** entre el documento transversal y el Capítulo 15 del caso. Se acreditan aquí **ambas lecturas por separado**, para que ninguna quede sin declarar cualquiera sea la que la Comisión Evaluadora aplique.

**Lectura del documento transversal — los ambientes no productivos se apagarán o reducirán fuera del horario de uso.** Se compromete. La consecuencia que sí es de arquitectura lógica y se declara aquí es la excepción: la reducción **no se aplica** durante las ventanas de prueba de carga sobre Preproducción, ni durante las detenciones programadas de planta de marzo y septiembre, cuando el volumen de acreditación multiplica la carga de validación. El calendario de reducción por ambiente, con su ahorro asociado, es materia del Subdocumento 4.2, que desarrolla los cinco ambientes del numeral 4.1 transversal.

**Lectura del Capítulo 15 del caso — certificaciones sectoriales del ADJUDICATARIO.** El caso asocia a ese código la exigencia de **ISO 45001 e ISO 14001 vigentes**, **conocimiento acreditado del Reglamento de Seguridad Minera** y **experiencia comprobable en faena minera en operación**. No es materia de arquitectura lógica: son atributos de la empresa proponente y su evidencia documental corresponde al Subdocumento 1. Se consigna aquí la remisión para que el requisito no quede huérfano entre ambos subdocumentos, que es precisamente el riesgo que la desalineación de códigos produce.

La consecuencia arquitectónica del Reglamento de Seguridad Minera sí está en este documento y no se repite: el valor probatorio del registro nacido digital en el lugar del hecho, con persona identificada y sello de tiempo y posición de origen, que sostienen el apartado 6.3 y el criterio de aceptación n.º 9.

### 16.7 Canales conversacionales y acción desde la notificación (RT-16.26)

El RT-16.26 es deseable y se compromete **acotado**, porque en este caso la frontera entre notificar y capturar es la misma que separa un registro con valor probatorio de uno sin él.

**Se compromete la acción desde el canal** en dos flujos, y en ambos la acción es una elección desde una lista cerrada, no una captura de texto libre:

- **Vencimiento de habilitación**, dirigido a la empresa contratista y a la persona: la notificación permite adjuntar el documento de renovación y disparar el flujo de acreditación de RF-IDE-009 sin entrar al portal. Es la vía más corta hacia el criterio n.º 7, porque quien debe actuar es personal administrativo de una empresa pequeña que no vive en el portal.
- **Detención pendiente de clasificar**, dirigido al supervisor de turno: la notificación permite escoger la causa desde la **misma taxonomía cerrada y jerárquica de tres niveles** de RN-17. No es una excepción a la regla: es la regla ejecutada por otro canal, y por eso el supervisor sigue resolviendo sólo el residuo y desde lista cerrada.

**No se compromete, y se declara por qué:** ninguna acción que escriba en la cadena de trazabilidad del mineral —registrar un viaje, un pesaje, un movimiento de stock, conformar o liberar un lote— es ejecutable desde un canal conversacional. El hecho de terreno nace en el dispositivo inventariado, firmado y con identificador de origen (decisión n.º 1 del apartado 2.2 y RN-30); admitirlo por un canal que no ofrece esas tres propiedades abriría una segunda vía de captura sin valor probatorio y rompería el criterio n.º 1. Tampoco se admite la liberación de un lote, que RN-11 declara sin excepción manual.

Toda acción ejecutada desde el canal se registra con el mismo asiento de auditoría que la ejecutada desde la interfaz, identificando el canal de origen.

### 16.8 Trazabilidad de este apartado

| Requisito | Dónde se acredita | Componente que lo satisface |
|---|---|---|
| RT-17.02 | Apartados 16.2 y 16.3 | C-03 — Captura en terreno, aplicación móvil con operación desconectada |
| RT-17.08 | Apartado 16.4 | C-03 — Captura en terreno, aplicación móvil con operación desconectada |
| RT-13.12 | Apartado 16.5 | C-03; C-16 — Vista única de equipo y apoyo al mantenimiento; C-19 — Analítica, indicadores y reportería; C-20 — Administración, parametrización y auditoría |
| RT-15.02, lectura transversal | Apartado 16.6, con el calendario en el Subdoc. 4.2 | C-22 — Plataforma base: infraestructura, seguridad, identidad técnica y observabilidad |
| RT-15.02, lectura del Capítulo 15 del caso | Apartado 16.6, con remisión al Subdocumento 1 | No aplica: atributo de la empresa proponente |
| RT-16.26 | Apartado 16.7 | C-20 — Administración, parametrización y auditoría; C-05 — Identidad y acreditación; C-15 — Gestión de detenciones operacionales y productividad |

**Qué queda derivado al Subdocumento 4.2:** el producto y la versión del contenedor nativo y del motor de presentación; la matriz de dispositivos, sistemas operativos y versiones efectivamente soportados; y la especificación del hardware de terreno del Formulario T-11, incluida la línea base de memoria y resolución que activa la interfaz reducida del apartado 16.4; y el proveedor y el costo unitario de cada canal de notificación del apartado 16.7, que el RT-16.24 lleva a la Oferta Económica.

---

## 17. Especificación del diagrama de arquitectura lógica

Este apartado no contiene el diagrama: contiene su especificación literal, para que quien lo dibuje no invente nombres. Representación por **columnas verticales por capa**, de izquierda a derecha en el orden en que viaja una transacción.

**Cinco columnas principales, en este orden:**

1. **Actores** — Persona operadora de pala · Persona operadora de camión · Persona operadora de perforadora · Supervisor de turno · Despachador · Portería · Persona que toma la muestra · Laboratorio · Metalurgia · Superintendencia de Planificación Minera · Jefatura de Mantenimiento · Área de Seguridad y Salud Ocupacional · Área de Sustentabilidad · Área comercial · Empresa contratista · Comprador · Autoridad ambiental · Centro Integrado de Operaciones · Administrador del CLIENTE
2. **Presentación** — C-03 Captura en terreno · C-06 Portal de empresas contratistas · C-07 Portal de trazabilidad de lote · C-16 Vista única de equipo · C-19 Tableros e indicadores · C-20 Consola de administración
3. **Servicios de negocio**, agrupados por límite de contexto, con el nombre del contexto rotulado sobre el grupo — Mineral: C-01, C-02, C-08 · Personas: C-05 · Activos: C-15, C-16 · Cumplimiento: C-17, C-18 · Analítico: C-19 · Plataforma: C-20. Son los **diez** servicios de la capa 4, en **seis** de los nueve contextos. C-16, C-19 y C-20 aparecen también en la columna 2: se dibujan dos veces, rotulando `interfaz` en la columna 2 y `lógica` en la columna 3, conforme a la nota del apartado 5. Los tres contextos sin caja propia en esta columna —Terreno, Calidad y Comercial— se rotulan sobre el grupo al que su lógica pertenece: Terreno sobre C-03 y C-04, Calidad sobre C-09 y Comercial sobre C-07
4. **Integración y eventos** — C-04 Nodo de borde · C-09 Laboratorio · C-10 Despacho de flota · C-11 Planificación minera · C-12 ERP · C-13 Telemetría, tres adaptadores · C-14 Historiador OPC UA · adaptador del sistema de control de fatiga, dibujado en esta columna y rotulado `pertenece a C-18`
5. **Datos** — Transaccional · Analítica · Series de tiempo · Documental · Archivos · Repositorio de consulta histórica · C-21 Migración

**Dos bandas transversales** que cruzan las cinco columnas, no cajas dentro de una: **Seguridad transversal** (C-05 aplicada a todas las capas, más la parte de C-22) y **Observabilidad transversal** (parte de C-22).

**Dos bandas de exposición** entre Actores y Presentación, que aplican **sólo** a los actores externos —Empresa contratista, Comprador, Autoridad—: **Borde y exposición** y **Puerta de enlace de exposición**. Los actores internos de terreno entran por la **Puerta operacional**, que se dibuja entre Presentación y Servicios de negocio.

**Marco punteado rotulado «Opera sin enlace, 48 h»** que encierra C-03, C-04 y la caché de credenciales.

**Bloque externo a la derecha, rotulado «Sistemas del CLIENTE que no se reemplazan»** — ERP corporativo · Despacho de flota · Sistema de gestión de laboratorio · Software de planificación minera · Historiador de proceso, sólo lectura · Tres portales de telemetría · Sistema de control de fatiga.

**Marcas obligatorias en el dibujo:** etiqueta de sentido en cada conector externo, con `sólo lectura` explícito en el historiador; rótulo `ACL` en los siete adaptadores, incluido el de control de fatiga; rótulo `Etapa 2` en C-07, C-13 y C-17.

**Las tres capacidades excluidas del Contrato se dibujan como cajas, no como leyenda:** tres rectángulos de trazo discontinuo y relleno neutro, **fuera** del marco del sistema, rotulados `Prorrateo automático del viaje de borde` junto a C-01, `Biometría en portería` junto a C-05 y `Factor de corrección de pesaje por equipo y período` junto a C-08, cada uno unido a su componente por una línea discontinua y con la nota común `Excluido del Contrato · Art. 72.º BB.AA.`. Dibujarlas como cajas y no como texto al pie es deliberado: muestra que la arquitectura tiene el punto de extensión previsto, que es lo que declara el apartado 2.4.

**No incluir** en este diagrama columnas laterales de hardware, software ni servicios externos contratados: son decisiones tecnológicas del Subdocumento 4.2 y mezclarlas aquí es el riesgo que la propia pauta advierte.

---

## 18. Índice de decisiones de arquitectura (RT-02.04)

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
| ADR-11 | La aplicación de terreno C-03 es híbrida: núcleo web único en contenedor nativo, con superficie nativa acotada a persistencia no desalojable, almacén de claves por hardware, lectura de proximidad y ciclo de vida en segundo plano | Web progresiva; nativa por plataforma | La web progresiva no garantiza persistencia del registro local ni lectura de proximidad en el rango de versiones del RT-17.03, y ambas son condición del criterio n.º 13 y de RN-22. La nativa duplica la base de código y su mantención recae sobre el equipo de once personas del CLIENTE al término del Contrato (restricción n.º 10) |
| ADR-12 | Zero Trust con el punto de decisión de política **delegado y firmado** en el nodo de borde durante la operación sin enlace, y verificación repetida en la reconciliación | Zero Trust estricto sin excepción; confianza implícita por posición de red en la faena | Verificar cada solicitud contra el servicio central detendría la mina durante un corte, que es lo que prohíben las restricciones n.º 1 y n.º 2. Conceder confianza por red anularía el modelo. Delegar y volver a verificar conserva las tres propiedades de NIST SP 800-207 y respeta RN-30 |
| ADR-13 | Clasificación de la información en cuatro niveles, con control diferenciado entre lo competitivamente sensible y lo personal sensible | Dos niveles, interno y confidencial, con un control común | La ley por polígono y un examen preocupacional exigen controles distintos: el primero se protege por filtrado en la vista de exposición y registro de consulta (RN-16); el segundo, por cifrado a nivel de campo y auditoría de consulta (RN-29, RT-11.10). Un control común sobreprotege el primero y subprotege el segundo |
| ADR-14 | Escalado anticipado por calendario del cambio de turno, con escalado reactivo como red de seguridad | Escalado reactivo puro por métrica de carga | El peak es predecible: 08:00 y 20:00, de 45 a 60 minutos, 200 transacciones por segundo (RN-23, SUP-31). El escalado reactivo llega cuando el peak lleva un tercio transcurrido, justo en la franja de apertura y cierre de sesiones de operador |
| ADR-15 | Despliegue canario con progresión por sitio y migraciones de esquema por expansión y contracción | Azul-verde con conmutación completa | Los siete sitios no fallan igual y el acopio de puerto depende de un operador sin acuerdo de nivel de servicio (SUP-19). El canario permite detenerse en el primer sitio que se degrade. La convivencia de dos versiones es además obligatoria porque un nodo de borde reconcilia eventos generados por la versión anterior tras 48 horas aislado |
| ADR-16 | Configuración técnica y parámetro de negocio en mecanismos separados | Un único mecanismo de configuración por ambiente | Si un umbral de conciliación fuera configuración técnica, cambiarlo reescribiría el pasado y los criterios n.º 3 y n.º 4 dejarían de ser verificables sobre historia cerrada (RN-27, RT-16.02) |
| ADR-17 | Un solo identificador de correlación, nacido en el dispositivo, compartido por la observabilidad y por las integraciones | Identificador de traza propio de la observabilidad, distinto del de correlación de integración | Dos identificadores obligan a unirlos para seguir una operación de negocio extremo a extremo. Uno asignado al recibir no correlaciona lo ocurrido durante las 48 horas sin recepción, por la misma razón de ADR-05 (RT-14.01, RT-05.19) |

---

## 19. Referencias

International Organization for Standardization. (2022). *ISO/IEC/IEEE 42010:2022 — Software, systems and enterprise: Architecture description*.

International Electrotechnical Commission. (2013-2024). *IEC 62443 — Security for industrial automation and control systems* [serie]. Se aplican en particular la 62443-3-3 sobre requisitos de sistema y niveles de seguridad, la 62443-4-1 sobre el ciclo de vida de desarrollo seguro, la 62443-4-2 sobre requisitos técnicos de componente y la 62443-2-1:2024 sobre el programa de seguridad del operador. El nivel de seguridad objetivo de la zona de lectura del historiador se declara en el Subdocumento 4.2.

International Society of Automation. (2013). *ANSI/ISA-95 — Enterprise-control system integration*, adoptada como IEC 62264. Modelo de referencia de la integración entre el nivel de operación y el nivel de gestión.

International Organization for Standardization. (2018). *ISO 45001 — Occupational health and safety management systems*; *ISO 14001 — Environmental management systems*.

Ley N.º 21.719 de 2024. Regula la protección y el tratamiento de los datos personales y crea la Agencia de Protección de Datos Personales. 13 de diciembre de 2024. Diario Oficial de la República de Chile. **Entra en vigencia el 1 de diciembre de 2026**, dentro del plazo de ejecución de este Contrato: las obligaciones que impone rigen desde la Etapa 1 y no se difieren.

Ley N.º 21.663 de 2024. Ley Marco de Ciberseguridad e Infraestructura Crítica de la Información. 8 de abril de 2024. Diario Oficial de la República de Chile.

Ley N.º 21.459 de 2022. Establece normas sobre delitos informáticos. 20 de junio de 2022. Diario Oficial de la República de Chile.

Ley N.º 19.799 de 2002. Sobre documentos electrónicos, firma electrónica y servicios de certificación de dicha firma. 12 de abril de 2002. Diario Oficial de la República de Chile. Sostiene el valor probatorio del registro local firmado del apartado 9 y de los reportes firmados de la capa documental.

Decreto Supremo N.º 132 de 2002 [Ministerio de Minería]. Aprueba Reglamento de Seguridad Minera. 7 de febrero de 2004. Diario Oficial de la República de Chile.

Parker, H. M. (2012). Reconciliation principles for the mining industry. *Mining Technology, 121*(3), 160-176. https://doi.org/10.1179/1743286312Y.0000000010

World Business Council for Sustainable Development y World Resources Institute. (2004). *The Greenhouse Gas Protocol: A corporate accounting and reporting standard* (ed. rev.). Base del cálculo de alcances 1, 2 y 3 que exige RN-26.
