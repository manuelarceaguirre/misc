# Notas sobre Apache Spark y Apache Airflow

Estas notas resumen los conceptos básicos y avanzados de dos herramientas muy utilizadas en el procesamiento de datos y la orquestación de workflows: Apache Spark y Apache Airflow.

------------------------------------------------------------------

## Apache Spark

### Introducción
Apache Spark es un motor de procesamiento distribuido diseñado para el análisis rápido de grandes volúmenes de datos. Su arquitectura permite ejecutar tareas en paralelo y optimizar recursos en clústeres.

### Componentes de Spark

- Core:  
  Procesamiento distribuido básico, responsable de la gestión de memoria y la distribución de tareas.

- SQL:  
  Permite ejecutar consultas estructuradas sobre datos utilizando DataFrames y el lenguaje SQL.

- Streaming:  
  Procesa datos en tiempo real, facilitando la ingesta y procesamiento de flujos continuos.

- MLib:  
  Biblioteca para Machine Learning, con algoritmos para clasificación, regresión, clustering, entre otros.

- GraphX:  
  Framework para el procesamiento de grafos y redes, útil en análisis de relaciones y redes sociales.

### Arquitectura del Clúster

- Driver:  
  Coordina la ejecución de tareas en el clúster. Es el punto central de control.

- Worker:  
  Nodo que ejecuta las tareas asignadas por el driver.

- Executor:  
  Proceso que se ejecuta en cada worker y realiza el procesamiento de datos.

- Task:  
  Unidad básica de trabajo que se ejecuta en los executors.

- Core:  
  Núcleos del CPU disponibles que determinan el nivel de paralelismo.

### Características Principales

- Lazy Evaluation:  
  La ejecución de transformaciones es diferida; el procesamiento se inicia solo cuando se invoca una acción.

- Transformations vs Actions:  
  - Transformations: Operaciones que crean un nuevo RDD o DataFrame (ej. map, filter, join).  
  - Actions: Operaciones que ejecutan el plan de procesamiento y devuelven un resultado (ej. count, collect).

- Narrow vs Wide Transformations:  
  - Narrow: Operaciones que no requieren intercambio de datos entre nodos (ej. map, filter).  
  - Wide: Operaciones que implican un "shuffle" o intercambio de datos entre particiones (ej. reduceByKey, groupBy).

- Shuffling:  
  Proceso de redistribución de datos entre nodos, que puede resultar costoso en términos de rendimiento.

### Ejemplo de Código en Spark (Scala)

-------------------------------------------------
import org.apache.spark.sql.SparkSession

object SparkExample {
  def main(args: Array[String]): Unit = {
    // Crear la sesión de Spark
    val spark = SparkSession.builder
      .appName("Ejemplo de Spark")
      .master("local[*]")
      .getOrCreate()

    import spark.implicits._

    // Crear un DataFrame de ejemplo
    val df = Seq(
      ("Ana", 34),
      ("Luis", 29),
      ("María", 41)
    ).toDF("Nombre", "Edad")

    // Mostrar los datos
    df.show()

    // Filtrar personas mayores de 30 años y contar el resultado
    val mayoresDe30 = df.filter($"Edad" > 30)
    println("Cantidad de personas mayores de 30 años: " + mayoresDe30.count())

    spark.stop()
  }
}
-------------------------------------------------

### Fórmulas y Cálculos en Spark

Para optimizar la distribución de datos, se puede calcular el número óptimo de particiones. Por ejemplo, si se conoce el tamaño del dataset y se tiene un tamaño de partición deseado:

   Número de Particiones = (Tamaño del Dataset (GB)) / (Tamaño de Partición Deseado (GB))

Esta fórmula ayuda a distribuir equitativamente los datos entre los nodos del clúster.

### Optimización y Monitoreo

- Reading Spark UI:  
  Permite visualizar el rendimiento a nivel de Jobs, Stages y Tasks para identificar cuellos de botella y errores.

- Estrategias para controlar el shuffling:  
  - Reducir la cantidad de datos antes de realizar un shuffle.  
  - Usar un número adecuado de particiones.  
  - Cachear o persistir RDDs y DataFrames que se reutilizan.

- Optimización de recursos:  
  - Configuración dinámica de recursos.  
  - Ajuste de memoria y núcleos a través de parámetros como:  
      - spark.executor.memory  
      - spark.executor.cores

------------------------------------------------------------------

## Apache Airflow

### Tabla de Contenido

- **Components**  
  - Metadata Database  
  - Web Server Daemon  
  - Scheduler Daemon  
  - Worker Daemon  
  - Executor & Queues (Local, Celery, Kubernetes, Mesos)  

- **Development Concepts**  
  - XCOM  
  - Macros  
  - Jinja Templates  
  - Operators & Sensors  
  - Hooks  
  - Trigger Rules  
  - Variables & Connections  

- **Advanced Patterns**  
  - DAG Factories  
  - Custom Operators  
  - RBAC  
  - DagBag  
  - TaskFlows  

### Introducción
Apache Airflow es una plataforma de código abierto para crear, programar y monitorear workflows mediante DAGs (Directed Acyclic Graphs). Se utiliza para orquestar tareas en entornos de Big Data y ETL.

### Componentes de Airflow

- Metadata Database:  
  Base de datos que almacena estados y metadatos de los workflows. Se suele utilizar PostgreSQL o MySQL.

- Web Server Daemon:  
  Proporciona una interfaz web para gestionar y visualizar los DAGs y el estado de las tareas.

- Scheduler Daemon:  
  Programa y distribuye las tareas según la definición de los DAGs.

- Worker Daemon:  
  Ejecuta las tareas definidas en los DAGs.

- Executor & Queues:  
  Determinan cómo y dónde se ejecutan las tareas:  
    - Local: Tareas secuenciales en un solo nodo.  
    - Celery: Ejecución distribuida mediante colas de tareas.  
    - Kubernetes: Orquestación en contenedores.  
    - Mesos: Framework distribuido para la ejecución de tareas.

### Ejemplo de Código en Airflow (Python)

-------------------------------------------------
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

# Definición de argumentos por defecto para el DAG
default_args = {
    'owner': 'usuario',
    'depends_on_past': False,
    'start_date': datetime(2023, 1, 1),
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Creación del DAG
dag = DAG(
    'ejemplo_dag',
    default_args=default_args,
    description='Un ejemplo simple de DAG en Airflow',
    schedule_interval=timedelta(days=1),
)

# Definición de tareas
tarea_1 = BashOperator(
    task_id='tarea_1',
    bash_command='echo "Hola desde Airflow!"',
    dag=dag,
)

tarea_2 = BashOperator(
    task_id='tarea_2',
    bash_command='echo "Ejecutando segunda tarea"',
    dag=dag,
)

# Definir la secuencia de ejecución (tarea_1 -> tarea_2)
tarea_1 >> tarea_2
-------------------------------------------------

### Conceptos Clave y Fórmulas

En Airflow, las dependencias entre tareas se representan en un DAG. Por ejemplo, la relación:

   tarea_1 -> tarea_2

indica que la tarea_2 se ejecutará después de que la tarea_1 haya finalizado exitosamente.

### Buenas Prácticas

- Organización:  
  Mantener los DAGs en archivos separados y estructurados facilita el mantenimiento y la escalabilidad.

- Gestión de Variables y Conexiones:  
  Utilizar variables de entorno y gestionar las conexiones de forma centralizada para proteger información sensible.

- Monitoreo:  
  Aprovechar la interfaz web de Airflow para monitorear la ejecución de los DAGs, identificar errores y optimizar tiempos de ejecución.

------------------------------------------------------------------

## Integración de Apache Spark y Apache Airflow

### Uso Conjunto de Spark y Airflow

Apache Spark y Apache Airflow pueden integrarse para crear pipelines de datos robustos y automatizados. Airflow se encarga de orquestar los trabajos de Spark, programando y monitoreando su ejecución, mientras que Spark realiza el procesamiento distribuido de los datos. Esta combinación es ideal para flujos de trabajo de ETL (Extract, Transform, Load) a gran escala.

### Ejemplo de Integración (Python)

A continuación, se presenta un ejemplo de cómo usar Airflow para ejecutar un trabajo de Spark mediante el operador SparkSubmitOperator.

-------------------------------------------------
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'usuario',
    'depends_on_past': False,
    'start_date': datetime(2023, 1, 1),
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'spark_airflow_integration',
    default_args=default_args,
    description='Ejemplo de integración entre Spark y Airflow',
    schedule_interval=timedelta(days=1),
)

spark_job = SparkSubmitOperator(
    task_id='spark_job',
    application='/path/to/spark_script.py',
    conn_id='spark_default',
    dag=dag,
)

spark_job
-------------------------------------------------

En este ejemplo, el SparkSubmitOperator ejecuta un script de Spark (spark_script.py) que puede contener lógica de procesamiento como la mostrada en la sección de Spark. El parámetro conn_id hace referencia a una conexión preconfigurada en Airflow que define los detalles del clúster de Spark, como la URL del master.

### Mejores Prácticas para la Integración

- Configuración de Conexiones:  
  Asegúrate de configurar correctamente la conexión de Spark en Airflow (por ejemplo, spark_default) a través de la interfaz de administración de Airflow, especificando el master de Spark y otros parámetros necesarios.

- Manejo de Dependencias:  
  Si el script de Spark depende de librerías externas, estas deben estar disponibles en el clúster de Spark o incluirse mediante el parámetro application_args del SparkSubmitOperator.

- Monitoreo y Depuración:  
  Usa la interfaz de Airflow para monitorear el estado del trabajo de Spark y revisa los logs en caso de errores. Además, la Spark UI puede ser útil para depurar problemas relacionados con el procesamiento de datos.

### Beneficios de la Integración

La integración de Spark y Airflow permite aprovechar las fortalezas de ambas herramientas: la capacidad de procesamiento distribuido de Spark y la orquestación avanzada de Airflow. Esto resulta en pipelines de datos más eficientes, escalables y fáciles de mantener, especialmente en entornos donde se manejan grandes volúmenes de datos y se requiere automatización.
