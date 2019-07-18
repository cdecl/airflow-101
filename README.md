
# Airflow 101 
참고 : https://airflow.apache.org/start.html

## 1. Install 
- 설치 환경 
	- CentOS 7 / Python 3.6.7  
- 가상환경 세팅 (OPTION) : 가상환경에 설치하려면 아래 명령어 추가 후 진행 
```bash
sudo pip install virtualenv 
virtualenv airflow
source airflow/bin/activate
```

- 설치
```bash
# Install 
pip install apache-airflow
```

- 아래의 설치에러가 나오면 환경변수 세팅후 설치 
```
...
RuntimeError: By default one of Airflow's dependencies installs a GPL dependency (unidecode).  
To avoid this dependency set SLUGIFY_USES_TEXT_UNIDECODE=yes in your environment when you install or  
upgrade Airflow. To force installing the GPL version set AIRFLOW_GPL_UNIDECODE
```
```bash
# Install 
export AIRFLOW_GPL_UNIDECODE=yes && pip install apache-airflow
```

- 설치가 완료되면 ~/airflow 디렉토리 생성 완료 
	- airflow.cfg : Airflow 설정 파일 
- 사용자 작업파일을 위한 폴더생성 
```bash	
mkdir ~/airflow/dags
```


## 2. Airflow DB 초기화 
- 처음 사용을 위해 DB를 초기화   
	- Default로 Sqlite 사용   
```
airflow initdb 
```


- SqliteDB는 SequentialExecutor만 지원
- LocalExecutor, CeleryExecutor 를 지원하기 위해서는 MySQL 이나 PostgreSQL 설치 필요
- 단순 테스트 목적이면 아래 PostgreSQL 설치 생략 



### PostgreSQL 설치 및 초기화 
기본 yum 패키지 설치 및 psycopg2 (Python-PostgreSQL Database Adapter) 설치 
```bash
sudo yum install postgresql-server postgresql postgresql-contrib
pip install psycopg2
```

PostgreSQL DB 초기화 및 서비스 시작 
```bash
sudo postgresql-setup initdb
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

DB 및 계정(Role) 생성 
```bash
sudo -u postgres psql

postgres=# CREATE DATABASE airflow;                      
CREATE DATABASE                 
postgres=# CREATE USER airflow with ENCRYPTED PASSWORD 'airflow';                 
CREATE ROLE                     
postgres=# GRANT all privileges on DATABASE airflow to airflow;                   
GRANT  
postgres=# \c airflow           
You are now connected to database "airflow" as user "postgres".                   
airflow=# grant all privileges on all tables in schema public to airflow;         
GRANT  
```

airflow.cfg 수정 : executor, sql_alchemy_conn
```ini
# The executor class that airflow should use. Choices include
# SequentialExecutor, LocalExecutor, CeleryExecutor, DaskExecutor, KubernetesExecutor
# executor = SequentialExecutor
executor = LocalExecutor

# The SqlAlchemy connection string to the metadata database.
# SqlAlchemy supports many different database engine, more information
# their website
# sql_alchemy_conn = sqlite:////home/cdecl/airflow/airflow.db
sql_alchemy_conn = postgresql+psycopg2://airflow:airflow@localhost/airflow
```

PostgreSQL의 인증(접근)설정 수정 : METHOD를 md5로 수정 
```conf
sudo vi /var/lib/pgsql/data/pg_hba.conf

# TYPE  DATABASE        USER            ADDRESS                 METHOD
# "local" is for Unix domain socket connections only
local   all             all            md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```

PostgreSQL Restart 및 airflow db 초기화
```bash
sudo systemctl restart postgresql
airflow initdb 
```

## 3. 추가 설정 
- Timezone 수정 : airflow.cfg 
```ini
# Default timezone in case supplied date times are naive
# can be utc (default), system, or any IANA timezone string (e.g. Europe/Amsterdam)
# default_timezone = utc
default_timezone = Asia/Seoul
```

-----

## 4. Airflow 대시보드 
```bash
airflow webserver
```

```bash

[2019-02-11 11:17:38,193] {settings.py:174} INFO - settings.configure_orm(): Using pool settings. pool_size=5, pool_recycle=1800, pid=22717
/home/cdecl/airflow/lib/python3.6/site-packages/psycopg2/__init__.py:144: UserWarning: The psycopg2 wheel package will be renamed from release 2.8; in order to keep installing from binary please use "pip install psycopg2-binary" instead. For details see: <http://initd.org/psycopg/docs/install.html#binary-install-from-pypi>.
  """)
[2019-02-11 11:17:38,482] {__init__.py:51} INFO - Using executor LocalExecutor
  ____________       _____________
 ____    |__( )_________  __/__  /________      __
____  /| |_  /__  ___/_  /_ __  /_  __ \_ | /| / /
___  ___ |  / _  /   _  __/ _  / / /_/ /_ |/ |/ /
 _/_/  |_/_/  /_/    /_/    /_/  \____/____/|__/

[2019-02-11 11:17:38,872] {models.py:273} INFO - Filling up the DagBag from /home/cdecl/airflow/dags
Running the Gunicorn Server with:
Workers: 4 sync
Host: 0.0.0.0:8080
Timeout: 120
Logfiles: - -

...
```


## 5. Airflow 스케쥴러 
```bash
airflow scheduler
```

```bash
[2019-02-11 11:31:22,647] {settings.py:174} INFO - settings.configure_orm(): Using pool settings. pool_size=5, pool_recycle=1800, pid=23564
/home/cdecl/airflow/lib/python3.6/site-packages/psycopg2/__init__.py:144: UserWarning: The psycopg2 wheel package will be renamed from release 2.8; in order to keep installing from binary please use "pip install psycopg2-binary" instead. For details see: <http://initd.org/psycopg/docs/install.html#binary-install-from-pypi>.
[2019-02-11 11:31:22,946] {__init__.py:51} INFO - Using executor LocalExecutor
  ____________       _____________
 ____    |__( )_________  __/__  /________      __
____  /| |_  /__  ___/_  /_ __  /_  __ \_ | /| / /
___  ___ |  / _  /   _  __/ _  / / /_/ /_ |/ |/ /
 _/_/  |_/_/  /_/    /_/    /_/  \____/____/|__/

[2019-02-11 11:31:23,210] {jobs.py:1477} INFO - Starting the scheduler
[2019-02-11 11:31:23,210] {jobs.py:1485} INFO - Running execute loop for -1 seconds
[2019-02-11 11:31:23,211] {jobs.py:1486} INFO - Processing each file at most -1 times
[2019-02-11 11:31:23,211] {jobs.py:1489} INFO - Searching for files in /home/cdecl/airflow/dags
[2019-02-11 11:31:23,216] {jobs.py:1491} INFO - There are 19 files in /home/cdecl/airflow/dags
[2019-02-11 11:31:23,287] {jobs.py:1534} INFO - Resetting orphaned tasks for active dag runs
[2019-02-11 11:31:23,301] {dag_processing.py:453} INFO - Launched DagFileProcessorManager with pid: 23603
[2019-02-11 11:31:23,302] {jobs.py:1559} INFO - Harvesting DAG parsing results
[2019-02-11 11:31:23,305] {settings.py:51} INFO - Configured default timezone <Timezone [UTC]>
[2019-02-11 11:31:23,314] {settings.py:174} INFO - settings.configure_orm(): Using pool settings. pool_size=5, pool_recycle=1800, pid=23603
[2019-02-11 11:31:25,304] {jobs.py:1559} INFO - Harvesting DAG parsing results
```

## 6. DAGs 작성 및 테스트 

### DAGs 작업 파일 작성
- ~airflow/dags/batch.py 작성 

```python
# -*- coding: utf-8 -*-

from datetime import timedelta, datetime

import airflow
from airflow import DAG
from airflow.operators.bash_operator import BashOperator

# These args will get passed on to each operator
# You can override them on a per-task basis during operator initialization
default_args = {
	'owner': 'cdecl',
	'depends_on_past': False,
	'start_date': '2019-02-11',
	'email': ['airflow@example.com'],
	'email_on_failure': False,
	'email_on_retry': False,
	'retries': 1,
	'retry_delay': timedelta(minutes=5)
	# 'queue': 'bash_queue',
	# 'pool': 'backfill',
	# 'priority_weight': 10,
	# 'end_date': datetime(2016, 1, 1),
	# 'wait_for_downstream': False,
	# 'dag': dag,
	# 'adhoc':False,
	# 'sla': timedelta(hours=2),
	# 'execution_timeout': timedelta(seconds=300),
	# 'on_failure_callback': some_function,
	# 'on_success_callback': some_other_function,
	# 'on_retry_callback': another_function,
	# 'trigger_rule': u'all_success'
}

dag = DAG(
	'batch',
	default_args=default_args,
	description='batch test',
	schedule_interval=timedelta(days=1),
)

s1 = BashOperator(
	task_id='s1',
	bash_command='sleep 3; echo "s1 `date`" >> ~/result.txt',
	dag=dag,
)

s2 = BashOperator(
	task_id='s2',
	bash_command='sleep 3; echo "s2 `date`" >> ~/result.txt',
	dag=dag,
)

end = BashOperator(
	task_id='end',
	bash_command='echo "end `date`" >> ~/result.txt',
	dag=dag,
)

[s1, s2] >> end

```

### DAGs(작업) 리스트 확인
- DAGs 리스트 확인 
	- airflow list_dags
```bash
airflow list_dags

...
-------------------------------------------------------------------        
DAGS                     
-------------------------------------------------------------------        
batch                    
example_bash_operator    
example_branch_dop_operator_v3                    
example_branch_operator  
example_http_operator    
example_passing_params_via_test_command           
example_python_operator  
example_short_circuit_operator                    
example_skip_dag         
example_subdag_operator  
example_subdag_operator.section-1                 
example_subdag_operator.section-2                 
example_trigger_controller_dag                    
example_trigger_target_dag                        
example_xcom             
latest_only              
latest_only_with_trigger 
test_utils               
tutorial                 
```

- 하위 Task 확인  
	- airflow list_tasks [DAGs]
```bash
airflow list_tasks batch

...
end
s1
s2
```

### Test 
- airflow test [DAGs] [Task] [Date]
- Task 기준 실행 테스트 
- Date는 미래 날짜가 올수 없음 (실행기준 날짜)

```bash 
airflow test batch s1 20190210

...
--------------------------------------------------------------------------------                  
Starting attempt 1 of 2
--------------------------------------------------------------------------------                  
                       
[2019-02-11 14:29:13,204] {models.py:1593} INFO - Executing <Task(BashOperator): s1> on 2019-02-10T00:00:00+09:00          
[2019-02-11 14:29:13,235] {bash_operator.py:77} INFO - Tmp dir root location:                     
 /tmp                  
[2019-02-11 14:29:13,236] {bash_operator.py:86} INFO - Exporting the following env vars:         
AIRFLOW_CTX_DAG_ID=batch                        
AIRFLOW_CTX_TASK_ID=s1 
AIRFLOW_CTX_EXECUTION_DATE=2019-02-10T00:00:00+09:00                     
[2019-02-11 14:29:13,236] {bash_operator.py:100} INFO - Temporary script location: /tmp/airflowtmpe3nqnp91/s18kl9zov1      
[2019-02-11 14:29:13,236] {bash_operator.py:110} INFO - Running command: sleep 3; echo "s1 `date`" >> ~/result.txt         
[2019-02-11 14:29:13,241] {bash_operator.py:119} INFO - Output:          
[2019-02-11 14:29:16,245] {bash_operator.py:127} INFO - Command exited with return code 0         
```

### Trigger 실행 
- airflow trigger_dag batch [DAGs]
- Trigger를 실행 하기위해서는 DAG의 상태가 활성화 되어 있어야 함 

![image](https://user-images.githubusercontent.com/5927142/52547612-4db98580-2e0c-11e9-8387-6e7e6c9f40a2.png)
```bash
airflow trigger_dag batch
```
```bash
[2019-02-11 14:37:44,558] {__init__.py:51} INFO - Using executor LocalExecutor
[2019-02-11 14:37:44,802] {models.py:273} INFO - Filling up the DagBag from /home/cdecl/airflow/dags
[2019-02-11 14:37:44,883] {cli.py:240} INFO - Created <DagRun batch @ 2019-02-11 05:37:44+00:00: manual__2019-02-11T05:37:44+00:00, externally triggered: True>
```

- 상태 변화 : running - scheduled - queued - success
![airflow](https://user-images.githubusercontent.com/5927142/52547437-0e3e6980-2e0b-11e9-9e7f-49ec9a0d75ed.PNG)

- Graph View
![image](https://user-images.githubusercontent.com/5927142/52547917-14821500-2e0e-11e9-8152-73ccff0f889d.png)

- TreeView	
![image](https://user-images.githubusercontent.com/5927142/52547860-b1907e00-2e0d-11e9-985c-840128d25eeb.png)

- Gantt
![image](https://user-images.githubusercontent.com/5927142/52548000-5f039180-2e0e-11e9-87c6-56ff5363c1a1.png)
