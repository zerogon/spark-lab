## 2-1 환경사전 준비

### 2-1-1 필요한서버 & 패키지

| **OS**            | Ubuntu 20.04 / CentOS 7         |
| ----------------- | ------------------------------- |
| **Java**          | OpenJDK 8 이상 (JDK 11 권장)    |
| **Python**        | Python 3.8 이상                 |
| **Hadoop (선택)** | HDFS 저장을 원할 경우 설치 필요 |
| **Spark**         | Apache Spark                    |
| **SSH & SCP**     | 클러스터 구성 시 필요           |

### 2-1-2 서버구성

| **Master**  | `spark-master`   | Spark 클러스터의 중앙 제어 노드 |
| ----------- | ---------------- | ------------------------------- |
| **Worker1** | `spark-worker-1` | Spark 작업을 실행하는 노드      |
| **Worker2** | `spark-worker-2` | Spark 작업을 실행하는 노드      |

## 2-2 자바&파이썬 설치

```bash
# Java 설치 (Ubuntu)
sudo apt update && sudo apt install openjdk-11-jdk -y

# Java 설치 (CentOS)
sudo yum install java-11-openjdk -y

# 환경 변수 설정
echo "export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which javac))))" >> ~/.bashrc
source ~/.bashrc

# Python 설치 (Ubuntu)
sudo apt install python3 python3-pip -y

# Python 설치 (CentOS)
sudo yum install python3 python3-pip -y

# Java & Python 버전 확인
java -version
python3 --version

```

## 2-3 스파크 설치

```bash
# 설치
wget https://downloads.apache.org/spark/spark-3.4.0/spark-3.4.0-bin-hadoop3.tgz
tar -xvzf spark-3.4.0-bin-hadoop3.tgz
mv spark-3.4.0-bin-hadoop3 /opt/spark

# 환경변수
export SPARK_HOME=/opt/spark
export PATH=$SPARK_HOME/bin:$SPARK_HOME/sbin:$PATH
export PYSPARK_PYTHON=/usr/bin/python3
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

source ~/.bashrc

# 실행
# Master 실행
$SPARK_HOME/sbin/start-master.sh

# Worker 실행 (코어 4개, 메모리 4GB 할당)
$SPARK_HOME/sbin/start-worker.sh spark://localhost:7077 --cores 4 --memory 4G

```

Master Web UI 확인: http://localhost:8080

## 2-4 스파크 실행

```bash
from pyspark.sql import SparkSession
from pyspark.sql.functions import count

# Spark 세션 생성 (Standalone Mode)
spark = SparkSession.builder \
    .appName("WebLogAnalysis") \
    .master("spark://localhost:7077") \
    .getOrCreate()

# 로그 데이터 읽기
log_df = spark.read.csv("file:///home/user/weblogs.csv", header=True)

# 이벤트별 카운트 계산
event_count_df = log_df.groupBy("event_type").agg(count("*").alias("count"))

# 결과 저장
event_count_df.write.mode("overwrite").csv("file:///home/user/output/event_counts")

# 종료
spark.stop()

```

```bash
#spark Submit 실행
spark-submit --master spark://localhost:7077 \
    --deploy-mode client \
    /home/user/log_analysis.py

## --master spark://localhost:7077 → Standalone Mode에서 실행
## --deploy-mode client → 실행 결과를 즉시 확인 가능

```
