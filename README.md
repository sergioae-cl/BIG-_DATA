# BIG_DATA


Tarea 3

Primero, actualizamos el sistema y verificamos dependencias.


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


