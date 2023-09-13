# AWS

# Project Goals -1

- 데이터 수집 - OpenWeather API에서 데이터를 추출하기 위한 데이터 수집 파이프라인을 생성합니다.
- 데이터 저장 - AWS S3 버킷을 사용하여 데이터 저장 리포지토리를 생성합니다.
- 데이터 변환 - 데이터 추출, 간단한 변환 작업 수행 및 정제된 데이터를 로드하기 위한 ETL 작업을 Airflow를 사용하여 생성합니다.
- 데이터 파이프라인 - Python으로 작성된 데이터 파이프라인을 생성하여 API 호출로부터 데이터를 추출하고 AWS S3 버킷에 저장합니다.
- 파이프라인 자동화 - Apache Airflow를 사용하여 데이터 파이프라인을 트리거하고 프로세스를 자동화하기 위한 스케줄링 서비스를 생성합니다.

# Data Architecture
![image](https://github.com/hanjhoon/AWS/assets/121271030/9ba4bdeb-a3eb-4291-9018-3837d9d94549)
