# arquitectura-gcp-banco--Ricardo_Toro-
arquitectura-gcp-banco-[Ricardo_Toro]

Detección de Fraude y BI de Clientes — Caso Bancario (GCP)

Descripción del Caso

El banco busca detectar transacciones fraudulentas en tiempo real sin afectar la experiencia de los clientes legítimos, y a la vez generar inteligencia de negocio (BI) sobre el comportamiento de los usuarios, productos y comercios.

Se dispone de:

Transacciones de tarjetas (streaming) en tiempo real.

Maestros de clientes y productos (batch) cargados diariamente.

Catálogo de comercios (batch) actualizado de forma programada.

El objetivo es integrar y procesar ambos tipos de datos para obtener alertas de fraude instantáneas y tableros ejecutivos (KPIs) en Looker Studio o Looker.

-Diagrama de Arquitectura
Archivo: arquitectura_fraude_gcp_v2.drawio

Flujo general:
Ingesta
Streaming: Pub/Sub recibe eventos de transacciones.
Batch: Cloud Storage almacena archivos diarios (clientes, comercios).

-Procesamiento
Stream: Dataflow (Beam) limpia, enriquece y envía datos a Vertex AI o BigQuery.
Batch: Cloud Composer orquesta cargas hacia BigQuery.
Almacenamiento / Analytics
BigQuery maneja zonas RAW, STAGING y PROD.
BI Engine optimiza dashboards en Looker Studio.
Machine Learning (Vertex AI)
Entrenamiento y despliegue de modelos de fraude.
Endpoints online para scoring en milisegundos.
Serving / Integración
Cloud Run ejecuta el microservicio de scoring y publica alertas vía Pub/Sub.
Gobierno, Seguridad y Operación
IAM, KMS, Audit Logs, Data Catalog, Monitoring, Budgets & Alerts.

-KPIs (Indicadores Clave)
Categoría	Indicador	Meta
Eficacia del modelo	Recall de fraude	≥ 90%
	Falsos positivos	≤ 5%
Tiempo de respuesta	Latencia promedio de scoring	< 2 segundos
Valor financiero	Monto total de fraude evitado	↑ constante
Operacional	Alertas procesadas por hora	≥ 5.000
Calidad de datos	Registros válidos / totales	≥ 98%
BI / negocio	Crecimiento clientes activos	+5% mensual
	Monto promedio transacción	Monitorizado por segmento
 
-Decisiones Técnicas
Componente	Decisión	Justificación
Pub/Sub	Streaming de eventos	Baja latencia y alta elasticidad
Cloud Storage	Batch files	Almacenamiento económico y durable
Dataflow (Beam)	ETL unificado stream/batch	Serverless y escalable
Cloud Composer	Orquestación batch	DAGs gestionados con Airflow
BigQuery	Data Warehouse	Serverless, SQL estándar y costo por consulta
BI Engine + Looker	Dashboards	Visualización rápida y gobernada
Vertex AI	ML pipeline completo	Entrenamiento, registro y endpoints integrados
Cloud Run	API scoring	Microservicio sin servidores (auto-escalable)
IAM + KMS + VPC-SC	Seguridad	Control de acceso, cifrado y perímetro
Monitoring + Logging	Operabilidad	Métricas, alertas y auditoría en tiempo real


-Guía para Leer el Diagrama

Lectura horizontal izquierda → derecha:
Representa el flujo de datos: ingesta → procesamiento → almacenamiento → ML → integración → gobierno.

Flechas etiquetadas:
Indican tipo de flujo (JSON streaming, archivos batch, scoring response, etc.).

Color gris claro (bloques grandes):
Secciones lógicas o dominios del sistema.

Bloques blancos:
Servicios de GCP y sus funciones específicas.

Flechas bidireccionales:
Representan consultas o interacciones (por ejemplo, entre Cloud Run ↔ Vertex AI Endpoint).
Gobierno y operación (columna derecha):
Abarca de forma transversal toda la arquitectura.

-Supuestos del Diseño

El banco ya cuenta con políticas de seguridad y cumplimiento PCI-DSS.
Los identificadores sensibles (PAN, RUT) están tokenizados antes de entrar al flujo.
Todos los servicios están desplegados en la misma región GCP (e.g., us-central1).
Los datasets de BigQuery están particionados por fecha de transacción.
Los modelos de fraude se actualizan mensualmente con datasets históricos balanceados.
Cloud Run se comunica internamente con Vertex AI y BigQuery usando Service Accounts dedicadas.
Se monitorea continuamente drift de datos y eficacia del modelo con Vertex AI Model Monitoring.
