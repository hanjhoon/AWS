# AWS

# Project Goals -1

- 데이터 수집 - OpenWeather API에서 데이터를 추출하기 위한 데이터 수집 파이프라인을 생성합니다.
- 데이터 저장 - AWS S3 버킷을 사용하여 데이터 저장 리포지토리를 생성합니다.
- 데이터 변환 - 데이터 추출, 간단한 변환 작업 수행 및 정제된 데이터를 로드하기 위한 ETL 작업을 Airflow를 사용하여 생성합니다.
- 데이터 파이프라인 - Python으로 작성된 데이터 파이프라인을 생성하여 API 호출로부터 데이터를 추출하고 AWS S3 버킷에 저장합니다.
- 파이프라인 자동화 - Apache Airflow를 사용하여 데이터 파이프라인을 트리거하고 프로세스를 자동화하기 위한 스케줄링 서비스를 생성합니다.

# Data Architecture
![image](https://github.com/hanjhoon/AWS/assets/121271030/9ba4bdeb-a3eb-4291-9018-3837d9d94549)

# Dataset Used

이 프로젝트에서는 Openweathermap API를 사용하여 데이터를 추출하는 데이터 파이프라인을 구축하고 있습니다. API 호출은 Airflow를 통해 HTTPoperator을 사용하여 이루어집니다.
API 호출은 다음과 같이 보입니다.

```python
https://api.openweathermap.org/data/2.5/weather?q={city name}&appid={API key}
```

여기에는 두 가지 매개 변수가 있습니다.

q - 도시 이름 : 세계의 어떤 도시 이름이든 소문자로 입력합니다.
API 키 - Openweathermap.org에 계정을 만든 후 얻을 수 있습니다. API 키는 각 계정에 고유하며 공유하지 않도록 주의해야 합니다.
결과 데이터는 다음과 같은 구조로 JSON 형식으로 제공됩니다. 런던 시의 예제 호출입니다.

```
https://api.openweathermap.org/data/2.5/weather?q=London&appid={API key}
```

```json
     {
     "coord": {
       "lon": -0.13,
       "lat": 51.51
     },
     "weather": [
       {
         "id": 300,
         "main": "Drizzle",
         "description": "light intensity drizzle",
         "icon": "09d"
       }
     ],
     "base": "stations",
     "main": {
       "temp": 280.32,
       "pressure": 1012,
       "humidity": 81,
       "temp_min": 279.15,
       "temp_max": 281.15
     },
     "visibility": 10000,
     "wind": {
       "speed": 4.1,
       "deg": 80
     },
     "clouds": {
       "all": 90
     },
     "dt": 1485789600,
     "sys": {
       "type": 1,
       "id": 5091,
       "message": 0.0103,
       "country": "GB",
       "sunrise": 1485762037,
       "sunset": 1485794875
     },
     "id": 2643743,
     "name": "London",
     "cod": 200
     }
```


# Tools used in this project

1. Apache Airflow - Apache Airflow는 데이터 엔지니어링 목적으로 가장 일반적으로 사용되는 워크플로우 도구 중 하나인 오픈 소스 오케스트레이션 또는 워크플로우 도구입니다. 확장 가능하며 스탠드얼론 서비스 및 컨테이너화된 서비스로 제공되므로 데이터 파이프라인의 종속성, 진행 상황, 로그, 코드, 작업 트리거 및 성공 상태를 쉽게 시각화할 수 있습니다.
2. AWS EC2 - Amazon Elastic Compute Cloud (Amazon EC2)는 클라우드에서 안전하게 크기를 조정 가능한 컴퓨팅 용량을 제공하는 웹 서비스입니다. 사용자는 자체 애플리케이션을 실행하는 동안 시스템이 완전히 Amazon에서 관리되는 가상 환경을 생성, 시작 및 종료할 수 있으며 활성 서버에 대한 요금을 초 단위로 지불합니다. EC2는 사용자가 인스턴스의 지리적 위치를 제어할 수 있도록 하여 지연 시간 최적화 및 고 수준의 중복성을 제공합니다.
3. AWS S3 - Amazon Simple Storage Service (Amazon S3)는 고성능, 데이터 가용성, 보안 및 성능을 제공하는 저장 서비스입니다. 데이터를 객체로 저장하며 수천 개의 애플리케이션과 쉽게 통합할 수 있습니다. 객체는 파일과 파일을 설명하는 메타데이터입니다. 버킷은 객체를 저장하는 컨테이너입니다.

   
# Implementation

* **Step 1** - 모든 연산 및 코드는 가상 환경에서 작성됩니다. 첫 번째 작업은 AWS EC2 VM을 설정하고 파이썬 종속성 및 Airflow와 pandas를 설치하는 것입니다. 파이썬 가상 환경을 만들고 해당 환경 내에 모든 종속성을 설치하여 프로세스가 안전한 환경에서 수행되도록 합니다. 이것은 Airflow 서버이며 이 머신에서 독립형으로 Airflow를 실행합니다.
  
<p align="center">
  <img width="650" height="500" src="https://github.com/chayansraj/Python-ETL-pipeline-using-Airflow-on-AWS/assets/22219089/6d0500b7-06e9-464a-b343-89c32e56dfda">
  <h6 align = "center" > Source: Author </h6>
</p>


Airflow에는 기능에 중요한 네 가지 핵심 구성 요소가 있습니다.
1. Airflow 웹 서버 - Airflow UI를 제공하는 Gunicorn과 함께 실행되는 Flask 서버입니다.
2. Airflow 스케줄러 - 작업을 예약하는 역할을 하는 데몬입니다. 작업을 어떤 작업이 실행되어야 하는지, 언제 실행되어야 하는지 및 어디에서 실행되어야 하는지 결정하는 다중 스레드 Python 프로세스입니다.
3. Airflow 데이터베이스 - 모든 DAG 및 작업 메타데이터가 저장되는 데이터베이스입니다. 일반적으로 PostgreSQL 데이터베이스이지만 MySQL, MsSQL 및 SQLite도 지원됩니다.
4. Airflow 실행자 - 작업을 실행하는 메커니즘입니다. Airflow가 작동할 때 스케줄러 내에서 실행되는 실행자입니다.
5. Airflow 웹 서버가 실행되고 Airflow UI와 몇 가지 미리 정의된 DAG가 있는 Flask 서버가 생성됩니다.

<p align="center">
  <img  height="500" src="https://github.com/chayansraj/Python-ETL-pipeline-using-Airflow-on-AWS/assets/22219089/b85b7b02-deed-4dc3-93e5-a373e79d74a2">
  <h6 align = "center" > Source: Author </h6>
</p>


* **Step 2** - Airflow가 Openweathermap에 API 호출을 수행하려면 두 서비스 간에 연결이 필요합니다. 이것은 Airflow의 'connections' 탭을 사용하여 수행할 수 있으며 HTTP 연산자를 사용하여 Openweathermap에 액세스할 수 있도록 합니다.

  
<p align="center">
  <img  height="500" src="https://github.com/chayansraj/Python-ETL-pipeline-using-Airflow-on-AWS/assets/22219089/55509511-414a-438a-9612-5b9168a7ec23">
  <h6 align = "center" > Source: Author </h6>
</p>

* **Step 3** - 이제 적절한 가져오기 및 종속성을 갖춘 DAG를 만들어야 합니다. 이 단계는 DAG 내에서 각 작업에 대한 작업을 나타냅니다. 예를 들어 weather_dag.py와 같은 DAG 파일을 만들고 필요한 가져오기 및 라이브러리를 추가합니다.
  
      ```Pyhton
      from airflow import DAG
      from datetime import timedelta, datetime
      from airflow.providers.http.sensors.http import HttpSensor
      from airflow.providers.http.operators.http import SimpleHttpOperator
      from airflow.operators.python_operator import PythonOperator
      import pandas as pd
      import json
      ``` 
      Every DAG has some default arguments needed to run tasks according to the settings, you can set the default_args as following:
      ```Python
      default_args = {
      "owner":"airflow",
      "depends_on_past": False,
      "start_date": datetime(2023,1,1),
      "email": ['myemail@domain.com'],
      "email_on_failure": True,
      "email_on_retry": False,
      "retries": 2,
      "retry_delay": timedelta(minutes=3),}
      ```
     * **Task 1** - 작업은 연산자를 사용하여 DAG 내에서 작성됩니다. 여기서 API가 호출 가능한지 여부를 확인하기 위해 HTTPSensor 연산자를 사용합니다. API 키를 사용하고 관심 있는 도시를 선택합니다.
      
      ```Python
      with DAG("weather_dag",
         default_args = default_args,
         schedule_interval= "@hourly",
         catchup=False,) as dag:
    

        is_weather_api_available = HttpSensor(
            task_id = "is_weather_api_available",
            endpoint = "data/2.5/weather?q=Stockholm&appid=<API Key>",
            http_conn_id='weather_map_api')
  
      ```


     * **Task 2** -  이 작업은 날씨 API를 호출하고 JSON 형식의 데이터를 가져오기 위해 GET 메서드를 호출합니다. 텍스트로 변환하기 위해 람다 함수를 사용합니다.

      ```Python
      extract_weather_data = SimpleHttpOperator(
            task_id = "extract_weather_data",
            http_conn_id="weather_map_api",
            endpoint = "data/2.5/weather?q=Stockholm&appid=8d2a39daa31380c07c5716d4b8c88705",
            method= "GET",
            response_filter = lambda r:json.loads(r.text),
            log_response = True,
        )
      ```
     * **Task 3** - 이 작업은 JSON 형식을 CSV 파일로 변환하고 AWS S3 버킷에 저장하는 Python 함수를 호출합니다. 원하는 DAG를 실행할 때 스케줄 간격을 정의할 수 있습니다.

      ```Python
      transform_load_weather_data = PythonOperator(
            task_id = "transform_load_weather_data",
            python_callable= transform_load_data,
        )
      ```
  위의 단계를 구현한 후 Airflow UI로 이동하면 DAG 내의 작업을 볼 수 있으며 작업을 연결하기 위해 DAG 끝에 작업 순서를 추가합니다.

    
 <h4 align ='center' >   is_weather_api_available >> extract_weather_data >> transform_load_weather_data </h4>
  
<p align="center">
  <img  height="500" src="https://github.com/chayansraj/Python-ETL-pipeline-using-Airflow-on-AWS/assets/22219089/1e3c5466-256f-47ed-a926-5cb69c0f2663">
  <h6 align = "center" > Source: Author </h6>
</p>

* **Step 4** - 변환 함수를 작성하고 Airflow가 AWS S3를 사용하고 적절한 세션 자격 증명을 사용할 수 있도록 권한을 부여할 수 있습니다. 모든 트랜잭션에 대해 AWS는 서비스가 AWS 구성 요소와 상호 작용할 수 있도록 하는 세션 창을 생성합니다.

  
<p align="center">
  <img  height="600" src="https://github.com/chayansraj/Python-ETL-pipeline-using-Airflow-on-AWS/assets/22219089/d142838b-1643-4498-b952-56c35fd89569">
  <h6 align = "center" > Source: Author </h6>
</p>  


* **Step 5** - 이제 AWS S3 버킷에 저장된 csv 파일을 볼 수 있습니다. 이를 통해 방금 만든 데이터 파이프라인을 사용하여 데이터를 저장할 수 있습니다.
<p align="center">
  <img  height="600" src="https://github.com/chayansraj/Python-ETL-pipeline-using-Airflow-on-AWS/assets/22219089/d9ab4360-badf-49fc-be86-eddf0f6ceb57">
  <h6 align = "center" > Source: Author </h6>
</p>  
