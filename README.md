# spark
Setting up spark in Ubuntu OS
# Apache Spark 4.2.0 — Local Setup on Ubuntu

This guide documents how to install and configure **Java 17, Apache Spark 4.2.0, Python, PySpark, and a Python virtual environment** on an Ubuntu Linux machine.

The goal is to run Apache Spark locally, launch PySpark, and work with files stored on the local computer.

---

## 1. Prerequisites

The setup uses:

* Ubuntu Linux
* Java 17
* Python 3.12
* Apache Spark 4.2.0
* PySpark
* Python virtual environment (`venv`)
* pandas
* PyArrow
* grpcio
* grpcio-status
* zstandard

The Spark distribution used in this setup was:

```text
spark-4.2.0-bin-hadoop3-connect
```

---

# 2. Install Java 17

Apache Spark runs on the Java Virtual Machine (JVM), so Java needs to be installed before Spark.

First update the Ubuntu package lists:

```bash
sudo apt update
```

Then install OpenJDK 17:

```bash
sudo apt install openjdk-17-jdk -y
```

### Why `-y`?

The `-y` option automatically answers "yes" to installation prompts.

The correct syntax is:

```bash
sudo apt install openjdk-17-jdk -y
```

and **not**:

```bash
sudo apt install openjdk-17-jdk-y
```

The latter is interpreted as a different package name.

### Verify Java

```bash
java -version
```

Also verify the Java compiler:

```bash
javac -version
```

---

# 3. Resolve Ubuntu APT 404 Errors

During installation, the OpenJDK package returned errors such as:

```text
404 Not Found
```

This happened because the local APT package information was out of date.

The package lists were refreshed with:

```bash
sudo apt update
```

If the problem persists, APT's cached package lists can be cleared and rebuilt:

```bash
sudo apt clean
sudo rm -rf /var/lib/apt/lists/*
sudo apt update
```

Then retry:

```bash
sudo apt install openjdk-17-jdk -y
```

---

# 4. Download Apache Spark

Apache Spark 4.2.0 was downloaded from the Apache distribution server.

The package used was:

```text
spark-4.2.0-bin-hadoop3-connect.tgz
```

Create a directory for Spark:

```bash
cd ~/Downloads
mkdir spark
cd spark
```

Download Spark:

```bash
wget https://dlcdn.apache.org/spark/spark-4.2.0/spark-4.2.0-bin-hadoop3-connect.tgz
```

### Important

When pasting commands into a terminal, make sure there are no extra characters around the URL.

For example, terminal paste control characters such as:

```text
^[[200~
```

can cause `wget` to interpret the URL incorrectly and produce an error such as:

```text
Scheme missing
```

---

# 5. Extract Spark

Extract the downloaded `.tgz` archive:

```bash
tar -xvzf spark-4.2.0-bin-hadoop3-connect.tgz
```

This creates the directory:

```text
spark-4.2.0-bin-hadoop3-connect
```

The directory was renamed to make the Spark installation easier to reference:

```bash
sudo mv spark-4.2.0-bin-hadoop3-connect spark-4
```

The resulting structure was:

```text
~/Downloads/spark/
├── spark-4/
└── spark-4.2.0-bin-hadoop3-connect.tgz
```

The extracted Spark installation is therefore:

```text
~/Downloads/spark/spark-4
```

---

# 6. Understand the Spark Directory

The Spark installation contains directories such as:

```text
spark-4/
├── bin/
├── conf/
├── jars/
├── python/
├── sbin/
└── ...
```

The `bin` directory contains commands such as:

```text
spark-submit
spark-shell
pyspark
```

The `python` directory contains PySpark's Python libraries.

The `sbin` directory contains Spark administration/startup scripts.

---

# 7. Configure Environment Variables

Three environment variables are important:

```text
JAVA_HOME
SPARK_HOME
PATH
```

## JAVA_HOME

`JAVA_HOME` tells applications where Java is installed.

The Java installation path was:

```text
/usr/lib/jvm/java-17-openjdk-amd64
```

Set it with:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```

---

## SPARK_HOME

`SPARK_HOME` tells the system where Spark is installed.

Our Spark installation is:

```text
~/Downloads/spark/spark-4
```

Therefore:

```bash
export SPARK_HOME=$HOME/Downloads/spark/spark-4
```

`$HOME` represents the user's home directory.

For this machine:

```text
$HOME
```

is equivalent to:

```text
/home/victor
```

Therefore:

```text
$HOME/Downloads/spark/spark-4
```

is equivalent to:

```text
/home/victor/Downloads/spark/spark-4
```

---

# 8. Configure PATH

Linux uses the `PATH` environment variable to determine where it should look for executable commands.

Without adding Spark to `PATH`, we would have to run:

```bash
~/Downloads/spark/spark-4/bin/pyspark
```

instead of simply:

```bash
pyspark
```

Add Spark's `bin` and `sbin` directories to `PATH`:

```bash
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
```

---

# 9. Test the Environment Variables

Verify Java:

```bash
echo $JAVA_HOME
```

Expected:

```text
/usr/lib/jvm/java-17-openjdk-amd64
```

Verify Spark:

```bash
echo $SPARK_HOME
```

Expected:

```text
/home/victor/Downloads/spark/spark-4
```

Verify that Spark commands can be found:

```bash
which spark-submit
```

Expected:

```text
/home/victor/Downloads/spark/spark-4/bin/spark-submit
```

---

# 10. Make the Environment Variables Permanent

The `export` commands initially only affected the current terminal session.

To make the configuration permanent, the settings were added to:

```text
~/.bashrc
```

Open the file:

```bash
nano ~/.bashrc
```

Add:

```bash
# Java and Spark
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_HOME=$HOME/Downloads/spark/spark-4
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
```

Save the file:

```text
Ctrl + O
Enter
Ctrl + X
```

Then reload the configuration:

```bash
source ~/.bashrc
```

The `source` command applies the changes to the current terminal without requiring the terminal to be closed and reopened.

---

# 11. Verify Spark

Spark can now be tested with:

```bash
spark-submit --version
```

This verifies that the operating system can find the Spark installation.

---

# 12. Create a Python Virtual Environment

Instead of installing Python packages directly into the system Python installation, a virtual environment was created for Spark.

This keeps Spark's Python dependencies isolated from the rest of the Ubuntu system.

First navigate to the Spark directory:

```bash
cd ~/Downloads/spark/spark-4
```

Create the virtual environment:

```bash
python3 -m venv .venv
```

This creates:

```text
spark-4/
└── .venv/
```

---

# 13. Activate the Virtual Environment

Activate the environment:

```bash
source .venv/bin/activate
```

The terminal prompt should now contain:

```text
(.venv)
```

For example:

```text
(.venv) victor@victor-HP-EliteBook-820-G1:~/Downloads/spark/spark-4$
```

This indicates that Python commands are now being executed within the virtual environment.

---

# 14. Verify the Virtual Environment

Run:

```bash
which python
```

The result should point to the virtual environment:

```text
/home/victor/Downloads/spark/spark-4/.venv/bin/python
```

This confirms that the environment is active.

---

# 15. Install Python Dependencies

The initial PySpark startup failed because pandas was missing.

The required pandas version was:

```text
pandas >= 2.2.0
```

However, Spark 4.2.0 currently warns about pandas 3.x compatibility, so pandas was installed with an upper limit below version 3:

```bash
python -m pip install "pandas>=2.2.0,<3.0.0"
```

Other Spark Connect dependencies were installed as needed.

### PyArrow

```bash
python -m pip install pyarrow
```

PyArrow is used for efficient data interchange between Spark and Python/Pandas.

### gRPC

```bash
python -m pip install grpcio
```

Spark Connect uses gRPC for communication.

### gRPC Status

```bash
python -m pip install grpcio-status
```

This provides additional gRPC status functionality required by Spark Connect.

### Zstandard

```bash
python -m pip install "zstandard>=0.25.0"
```

Zstandard provides compression functionality required by the Spark Connect environment.

---

# 16. Verify Python Dependencies

All the installed dependencies can be checked with:

```bash
python -c "import pandas, grpc, pyarrow; print('pandas:', pandas.__version__); print('grpcio:', grpc.__version__); print('pyarrow:', pyarrow.__version__)"
```

Example output:

```text
pandas: 2.3.3
grpcio: 1.84.0
pyarrow: 25.0.1
```

Zstandard can be checked separately:

```bash
python -c "import zstandard; print(zstandard.__version__)"
```

---

# 17. Start PySpark

With the virtual environment activated:

```bash
pyspark
```

A successful startup should provide a PySpark interactive environment and make the Spark session available.

The important confirmation is:

```text
SparkSession available as 'spark'.
```

At this point, the local Spark installation is operational.

---

# 18. Test Spark with a Local CSV File

A small CSV dataset can be created in the Downloads directory.

For example:

```bash
cat > ~/Downloads/sales.csv << 'EOF'
order_id,order_date,customer,product,category,quantity,unit_price,region
1001,2026-09-01,John,Running Shoes,Footwear,2,4500,Nairobi
1002,2026-09-01,Mary,Sneakers,Footwear,1,3200,Mombasa
1003,2026-09-02,Peter,T-Shirt,Clothing,3,1500,Nairobi
1004,2026-09-02,Jane,Running Shoes,Footwear,1,4500,Kisumu
1005,2026-09-03,David,Jeans,Clothing,2,2800,Nakuru
1006,2026-09-03,Ann,Sneakers,Footwear,2,3200,Nairobi
1007,2026-09-04,James,T-Shirt,Clothing,4,1500,Mombasa
1008,2026-09-04,Susan,Jeans,Clothing,1,2800,Nairobi
1009,2026-09-05,Michael,Running Shoes,Footwear,2,4500,Nakuru
1010,2026-09-05,Grace,Sneakers,Footwear,3,3200,Kisumu
EOF
```

Verify the file:

```bash
ls -lh ~/Downloads/sales.csv
```

---

# 19. Read the CSV with Spark

Start PySpark:

```bash
pyspark
```

Then create a Spark DataFrame:

```python
df = spark.read.csv(
    "/home/victor/Downloads/sales.csv",
    header=True,
    inferSchema=True
)
```

Display the data:

```python
df.show()
```

The `header=True` option tells Spark that the first row contains column names.

The `inferSchema=True` option tells Spark to determine appropriate data types instead of treating every column as a string.

---

# 20. Inspect the DataFrame Schema

Run:

```python
df.printSchema()
```

This allows us to see the data types Spark assigned to each column.

For example:

```text
root
 |-- order_id: integer
 |-- order_date: date
 |-- customer: string
 |-- product: string
 |-- category: string
 |-- quantity: integer
 |-- unit_price: integer
 |-- region: string
```

---

# 21. Count the Records

To determine how many records are in the DataFrame:

```python
df.count()
```

Expected result:

```text
10
```

---

# 22. Select Specific Columns

Spark allows individual columns to be selected:

```python
df.select("product", "quantity", "unit_price").show()
```

Another example:

```python
df.select("customer", "region").show()
```

---

# 23. Spark Setup Architecture

The final setup can be summarized as:

```text
Ubuntu
│
├── Java 17
│   └── JAVA_HOME
│
├── Apache Spark 4.2.0
│   └── SPARK_HOME
│
├── Python 3.12
│
└── Python Virtual Environment
    │
    ├── pandas
    ├── PyArrow
    ├── grpcio
    ├── grpcio-status
    └── zstandard
        │
        ▼
      PySpark
        │
        ▼
   Local Spark Session
        │
        ▼
   Local CSV / Files
```

---

# 24. Important Commands — Quick Reference

### Java

```bash
java -version
```

### Spark

```bash
spark-submit --version
```

### Spark installation location

```bash
echo $SPARK_HOME
```

### Java installation location

```bash
echo $JAVA_HOME
```

### Find Spark commands

```bash
which spark-submit
```

### Create virtual environment

```bash
python3 -m venv .venv
```

### Activate virtual environment

```bash
source .venv/bin/activate
```

### Deactivate virtual environment

```bash
deactivate
```

### Check Python

```bash
which python
python --version
```

### Start PySpark

```bash
pyspark
```

### Read a CSV

```python
df = spark.read.csv(
    "/home/victor/Downloads/sales.csv",
    header=True,
    inferSchema=True
)
```

### Display data

```python
df.show()
```

### Display schema

```python
df.printSchema()
```

### Count records

```python
df.count()
```

---

# 25. Key Lessons from the Setup

### `JAVA_HOME`

Tells Spark where Java is installed:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```

### `SPARK_HOME`

Tells the environment where Spark is installed:

```bash
export SPARK_HOME=$HOME/Downloads/spark/spark-4
```

### `PATH`

Allows Spark commands such as `pyspark` and `spark-submit` to be executed without specifying their full paths:

```bash
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
```

### `.bashrc`

Stores environment configuration that should be loaded automatically when a new Bash terminal starts.

### Virtual Environment

The Python virtual environment isolates Spark's Python dependencies from the system Python installation.

### PySpark

PySpark provides the Python interface to Apache Spark, allowing Spark functionality to be accessed using Python.

### Spark Connect Dependencies

The `spark-4.2.0-bin-hadoop3-connect` distribution requires additional Python dependencies including:

```text
pandas
pyarrow
grpcio
grpcio-status
zstandard
```

---

# 26. Final Result

The local environment is now configured to run Apache Spark 4.2.0 with PySpark.

The main components are:

```text
Java 17
    ↓
Apache Spark 4.2.0
    ↓
Python 3.12
    ↓
Python Virtual Environment
    ↓
PySpark + Dependencies
    ↓
SparkSession
    ↓
Local Files
```

The environment can now be used to learn and experiment with:

* Spark DataFrames
* CSV and other file formats
* Data transformations
* Filtering
* Aggregations
* Joins
* Window functions
* SQL with Spark
* PySpark
* Data pipelines
* Distributed data processing

The next step is to use the local `sales.csv` dataset to practice Spark operations and understand how Spark processes data compared with regular Python/Pandas workflows.
