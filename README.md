# BIG_DATA


Tarea 3

Primero, actualizamos el sistema y verificamos dependencias.


  sudo apt update && sudo apt upgrade -y.
  
  sudo apt install python3 python3-pip openjdk-11-jdk wget nano -y


Instalación de PySpark y Kafka-Python.


  pip install --upgrade pip.
  pip install pyspark kafka-python pandas matplotlib
  pip install pyspark kafka-python pandas matplotlib seaborn

  
Descarga e instalación de Apache Kafka.

wget https://downloads.apache.org/kafka/3.6.2/kafka_2.13-3.6.2.tgz
tar -xzf kafka_2.13-3.6.2.tgz
sudo mv kafka_2.13-3.6.2 /opt/Kafka


# Iniciar Zookeeper
sudo /opt/Kafka/bin/zookeeper-server-start.sh /opt/Kafka/config/zookeeper.properties &

# Iniciar Kafka
sudo /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties &


Creamos el canal de comunicación llamado sensor_data
/opt/kafka/bin/kafka-topics.sh --create \
--bootstrap-server localhost:9092 \
--replication-factor 1 \
--partitions 1 \
--topic sensor_data



Verifica que se haya creado correctamente
/opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
sensor_data

Script del Productor — kafka_producer.py

mkdir ~/spark_kafka_project
cd ~/spark_kafka_project

mport time, json, random
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

  
  
