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
  • Transformations: Operaciones que crean un nuevo RDD o DataFrame (ej. map, filter, join).
  • Actions: Operaciones que ejecutan el plan de procesamiento y devuelven un resultado (ej. count, collect).

- Narrow vs Wide Transformations:
  • Narrow: Operaciones que no requieren intercambio de datos entre nodos (ej. map, filter).
  • Wide: Operaciones que implican un "shuffle" o intercambio de datos entre particiones (ej. reduceByKey, groupBy).

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
  • Reducir la cantidad de datos antes de realizar un shuffle.
  • Usar un número adecuado de particiones.
  • Cachear o persistir RDDs y DataFrames que se reutilizan.

- Optimización de recursos:
  • Configuración dinámica de recursos.
  • Ajuste de memoria y núcleos a través de parámetros como:
      - spark.executor.memory
      - spark.executor.cores

------------------------------------------------------------------

## Apache Airflow

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
    • Local: Tareas secuenciales en un solo nodo.
    • Celery: Ejecución distribuida mediante colas de tareas.
    • Kubernetes: Orquestación en contenedores.
    • Mesos: Framework distribuido para la ejecución de tareas.

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

## Conclusión

Apache Spark y Apache Airflow son herramientas fundamentales en el ecosistema de Big Data. Mientras Spark se centra en el procesamiento distribuido y el análisis en tiempo real, Airflow permite orquestar y automatizar complejas cadenas de tareas. Juntas, estas tecnologías permiten desarrollar soluciones escalables, eficientes y fáciles de mantener en entornos de datos masivos.
