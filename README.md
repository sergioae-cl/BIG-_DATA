# BIG_DATA


Tarea 4

El presente es una guía interactiva de autoría propia con el fin de ilustrar el proceso de creacion de una Base de datos en Mongo para Iustrar un problema practico y mediante cosultas resolver el problema, este guia es de autoria propia autoría.


Caso de Uso Seleccionado: Catálogo de Productos para E-commerce
¿Por qué MongoDB? Un catálogo de productos es el caso de uso para bases de datos orientadas a documentos. Ademas de guardar toda la información del producto (incluyendo comentarios y especificaciones técnicas) en un solo documento, evitando JOINs costosos.

Diseño del Esquema:

Base de Datos: TechStoreDB
Colección Principal: productos
Jerarquia de la coleccion:
```
{
  "_id": ObjectId("..."),
  "nombre": "Laptop Gaming X1",
  "categoria": "Laptops",
  "precio": 1200.50,
  "stock": 15,
  "activo": true,
  "fecha_lanzamiento": ISODate("2023-01-15T00:00:00Z"),
  "caracteristicas": {  // Objeto anidado
    "marca": "TechBrand",
    "color": "Negro",
    "peso": "2.5kg"
  },
  "etiquetas": ["gaming", "oferta", "alto rendimiento"], // Array
  "valoraciones": [ // Array de objetos (One-to-Few relationship)
    {
      "usuario": "user123",
      "puntaje": 5,
      "comentario": "Excelente máquina"
    }
  ]
}
```

Creacion de la BD

Mediante la conexion ala localhost se crea la base de datos con lo siguientes parametros

<img width="632" height="495" alt="image" src="https://github.com/user-attachments/assets/00a3890e-73dc-4e64-b3f8-172e4e5ab831" />


Añadir la informacion mediante el sigioente escript que generarar masivamente los registros
```
// 1. Usar la base de datos
use TechStoreDB;

// 3. Script para generar 100 productos
var categorias = ["Laptops", "Smartphones", "Accesorios", "Monitores", "Audio"];
var marcas = ["Sony", "Samsung", "Apple", "Dell", "Logitech"];
var etiquetas_lista = ["oferta", "nuevo", "envio_gratis", "gaming", "oficina", "pro"];

var productos_batch = [];

for (var i = 1; i <= 100; i++) {
    // Selección aleatoria de datos
    var cat_random = categorias[Math.floor(Math.random() * categorias.length)];
    var marca_random = marcas[Math.floor(Math.random() * marcas.length)];
    var precio_random = parseFloat((Math.random() * 2000 + 10).toFixed(2));
    var stock_random = Math.floor(Math.random() * 100);
    
    // Generar etiquetas aleatorias (1 o 2 etiquetas)
    var tags = [etiquetas_lista[Math.floor(Math.random() * etiquetas_lista.length)]];
    if(Math.random() > 0.5) tags.push(etiquetas_lista[Math.floor(Math.random() * etiquetas_lista.length)]);

    var doc = {
        nombre: "Producto " + cat_random + " " + i,
        categoria: cat_random,
        precio: precio_random,
        stock: stock_random,
        activo: true,
        fecha_creacion: new Date(),
        caracteristicas: {
            marca: marca_random,
            garantia: "1 año",
            origen: "Importado"
        },
        etiquetas: tags,
        rating_promedio: Math.floor(Math.random() * 5) + 1 // 1 a 5 estrellas
    };
    productos_batch.push(doc);
}

// 4. Insertar masivamente
db.productos.insertMany(productos_batch);

print("¡Éxito! Se han insertado " + db.productos.countDocuments() + " documentos.");

```









  sudo apt update && sudo apt upgrade -y.
  
  sudo apt install python3 python3-pip openjdk-11-jdk wget nano -y


<img width="933" height="629" alt="image" src="https://github.com/user-attachments/assets/6437673a-93a2-4585-b54e-bbb7a3d9ec9b" />

Instalación de PySpark y Kafka-Python.

 
  pip install pyspark kafka-python pandas matplotlib
  pip install pyspark kafka-python pandas matplotlib seaborn


  
Descarga e instalación de Apache Kafka.

wget https://downloads.apache.org/kafka/3.6.2/kafka_2.13-3.6.2.tgz
tar -xzf kafka_2.13-3.6.2.tgz
sudo mv kafka_2.13-3.6.2 /opt/Kafka


<img width="1132" height="259" alt="image" src="https://github.com/user-attachments/assets/621093da-87f1-418f-89e0-4b343472182b" />



# Iniciar Zookeeper
sudo /opt/Kafka/bin/zookeeper-server-start.sh /opt/Kafka/config/zookeeper.properties &

# Iniciar Kafka
sudo /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties &

<img width="1078" height="595" alt="image" src="https://github.com/user-attachments/assets/8cf82995-ef6b-486e-aa94-fd76cbc18cec" />

Creamos el canal de comunicación llamado sensor_data
/opt/Kafka/bin/kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --replication-factor 1 \
  --partitions 1 \
  --topic sensor_data

<img width="1006" height="187" alt="image" src="https://github.com/user-attachments/assets/928f929e-c0a5-461f-8d23-9480e37a06a5" />


Verifica que se haya creado correctamente
/opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092

<img width="1084" height="93" alt="image" src="https://github.com/user-attachments/assets/50e63e82-5931-4935-af72-a3c6441b0bef" />


Script del Productor — kafka_producer.py

mkdir ~/spark_kafka_project
cd ~/spark_kafka_project

nano kafka_producer.py


```
import time, json, random
from kafka import KafkaProducer

def generate_sensor_data():
    return {
        "sensor_id": random.randint(1, 10),
        "temperature": round(random.uniform(20, 30), 2),
        "humidity": round(random.uniform(30, 70), 2),
        "timestamp": int(time.time())
    }

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

try:
    while True:
        data = generate_sensor_data()
        producer.send('sensor_data', value=data)
        print("Sent:", data)
        time.sleep(1)
except KeyboardInterrupt:
    print("Producer detenido")
finally:
    producer.close()
```

Script del Consumidor — spark_streaming_consumer.py

nano spark_streaming_consumer.py

```
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col, window, expr, to_timestamp
from pyspark.sql.types import StructType, StructField, IntegerType, DoubleType, LongType

spark = SparkSession.builder \
    .appName("KafkaSparkStreaming") \
    .getOrCreate()

spark.sparkContext.setLogLevel("WARN")

schema = StructType([
    StructField("sensor_id", IntegerType()),
    StructField("temperature", DoubleType()),
    StructField("humidity", DoubleType()),
    StructField("timestamp", LongType())
])

df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "sensor_data") \
    .load()

json_df = df.selectExpr("CAST(value AS STRING) as json_value")

parsed = json_df.select(from_json(col("json_value"), schema).alias("data")).select("data.*")

parsed = parsed.withColumn("ts", to_timestamp(col("timestamp")))

windowed = parsed.groupBy(window(col("ts"), "1 minute"), col("sensor_id")) \
    .agg(
        expr("avg(temperature) as avg_temp"),
        expr("avg(humidity) as avg_hum"),
        expr("count(*) as events_count")
    ).orderBy("window.start")

query = windowed.writeStream \
    .outputMode("complete") \
    .format("console") \
    .start()

query.awaitTermination()
```

Script de procesamiento por lotes — spark_batch_processing.py

nano spark_batch_processing.py
```
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, to_timestamp, when, isnan, count
from pyspark.sql.types import StructType, StructField, IntegerType, DoubleType, LongType

spark = SparkSession.builder.appName("BatchProcessing").getOrCreate()
spark.sparkContext.setLogLevel("WARN")

schema = StructType([
    StructField("sensor_id", IntegerType(), True),
    StructField("temperature", DoubleType(), True),
    StructField("humidity", DoubleType(), True),
    StructField("timestamp", LongType(), True)
])

# Reemplaza con tu ruta al archivo CSV
input_path = "sensor_history.csv"

df = spark.read.csv(input_path, header=True, schema=schema)
df = df.withColumn("ts", to_timestamp(col("timestamp")))

df_clean = df.filter(col("sensor_id").isNotNull()) \
             .withColumn("temperature", when(col("temperature").isNull(), -999.0).otherwise(col("temperature")))

null_counts = df_clean.select([count(when(col(c).isNull() | isnan(col(c)), c)).alias(c) for c in df_clean.columns])
null_counts.show(truncate=False)

df_clean.describe("temperature", "humidity").show()

df_clean.write.mode("overwrite").parquet("output/processed_sensors.parquet")

print("Procesamiento batch completado.")
spark.stop()
```
Iniciar Zookeeper y Kafka
sudo /opt/kafka/bin/zookeeper-server-start.sh /opt/kafka/config/zookeeper.properties &
sudo /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties &

/opt/kafka/bin/kafka-topics.sh --create --bootstrap-server localhost:9092 \
--replication-factor 1 --partitions 1 --topic sensor_data

d ~/spark_kafka_project
python3 kafka_producer.py
  

Sent: {'sensor_id': 2, 'temperature': 24.55, 'humidity': 63.2, 'timestamp': 1697650000}

<img width="797" height="616" alt="image" src="https://github.com/user-attachments/assets/9bda8570-0a14-49ed-9738-9c7713f50f44" />


Ejecutar el consumidor en la otra terminal

cd ~/spark_kafka_project
spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.3 spark_streaming_consumer.py

spark-submit spark_batch_processing.py


