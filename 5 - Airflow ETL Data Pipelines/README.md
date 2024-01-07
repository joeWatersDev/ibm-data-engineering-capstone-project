# Airflow ETL Data Pipelines
Write a pipeline that analyzes the web server log file, extracts the required lines (ending with html) and fields (time stamp, size ) and transforms (bytes to mb) and load (append to an existing file.)

**Objectives**
- Extract data from a web server log file
- Transform the data
- Load the transformed data into a tar file

**Tools / Software Used**
- Apache AirFlow 

## 1 - Create a DAG
We create a Python script which Apache Airflow can use to automate my ETL pipeline.

We start by creating the script process_web_log.py and importing necessary libraries.
```
touch process_web_log.py
```
```
# import the libraries
from datetime import timedelta
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.utils.dates import days_ago
```

We initialize the default arguments for the DAG.
```
default_args = {
    'owner': 'Joe Waters',
    'start_date': days_ago(0),    
    'email': ['joewaters@ibm.com'],
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    }
```

We then use the args to define the DAG.
```
dag=DAG(
  dag_id='process_web_log',
  default_args=default_args,
  description='Web Log ETL DAG',
  schedule_interval=timedelta(days=1),
)
```

Next we create the extract, transform, and load tasks we want to automate with the DAG. These comprise of bash scripts that perform ETL processes on the data found in the accesslog.txt files, which are provided.

The extract_data task:
```
extract_data = BashOperator(
    task_id='extract_data',
    bash_command='cut -f1 -d" " $AIRFLOW_HOME/dags/capstone/accesslog.txt >$AIRFLOW_HOME/dags/capstone/extracted_data.txt',
    dag=dag,
)
```

The transform_data task:
```
transform_data = BashOperator(
  task_id='transform_data',
  bash_command='grep -v "198.46.149.143" $AIRFLOW_HOME/dags/capstone/extracted_data.txt > $AIRFLOW_HOME/dags/capstone/transformed_data.txt',
  dag=dag,
)
```

The load_data task:
```
load_data = BashOperator(
  task_id='load_data',
  bash_command='tar -czvf $AIRFLOW_HOME/dags/capstone/weblog.tar $AIRFLOW_HOME/dags/capstone/transformed_data.txt',
  dag=dag,
)
```

Finally, we define the order of the task pipeline.
```
extract_data >> transform_data >> load_data
```

## 2 - Run the DAG with Airflow
We submit the DAG to Airflow by copying the completed script to the /dags folder.
```
cp process_web_log.py $AIRFLOW_HOME/dags
```

Then, modify the permissions of the working directory, and run the DAG.
```
chmod 777 sudo chmod 777 /home/project/airflow/dags/capstone
airflow dags unpause process_web_log
```

We can now observe the DAG runs in the Airflow console.
![Ecommerce data as a dashboard datasource](https://github.com/joeWatersDev/ibm-data-engineering-capstone-project/blob/main/5%20-%20Airflow%20ETL%20Data%20Pipelines/dag_runs.png)