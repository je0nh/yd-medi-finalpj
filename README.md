# AWS EMR 구축 및 세그먼트 데이터 핸들링
<p align="center">
  <img width="468" alt="Screenshot 2024-01-08 at 2 24 23 PM" src="https://github.com/yd-startup-team03/aws_EMR/assets/145730125/0e0d91de-05bb-4e07-8a3e-f8073ac60ff8">
<p/>

- 이어드림 3기 기업 연계 프로젝트
- 프로젝트 기간: 2023.11.08 ~ 2023.12.15
- 프로젝트 인원: 5명 (정의찬, 정태훈, 박준영, 양재하, 이정훈)

|이름|역할|
|---|---|
|이정훈|팀장, Spark query 변환|
|정태훈, 양재하|Airflow 구축, Scala 변환|
|정의찬, 박준영|Segment data 분석, EDA, Spark query 변환|

# 프로젝트 목표
- 기존 아키텍쳐
<p align="center">
  <img width="560" alt="Screenshot 2024-02-07 at 3 00 20 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/9eff8f13-b5eb-4177-89d3-d8482990d642">
</p>

- 변경된 아키텍쳐
<p align="center">
  <img width="555" alt="Screenshot 2024-02-07 at 3 00 27 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/6fde4a0d-3d7c-4ea7-a13e-35d2e2a184c6">
</p>

1. 기존 아키텍쳐 문제점
- Redshift에서 모든 데이터의 변환을 진행함
- 변환 데이터의 크기가 커지고, 여러 데이터 변환을 진행하다 보니 변환 속도가 느려지고 Redshift에 부하가 오기시작함

2. 기업 요구사항
- 기존 Redshift에서 변환되던 Query를 Spark Query로 변환하기

<p align="center">
  <img width="131" alt="Screenshot 2024-02-07 at 3 49 14 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/aac93b79-9a69-45b5-835b-b28e3dea8c1c">
</p>

- 기업에서 기존 제공한 Query 이외의 추가로 제공된 Query 변환 (15개)

3. 추가 작업 (기업 요구사항 x)
- Airflow를 이용한 ETL 파이프라인 자동화
- GitHub Action을 이용한 CI환경 구축

# 프로젝트 진행과정
1. Redshift query를 Spark query로 변환

<p align="center">
  <img width="615" alt="Screenshot 2024-02-07 at 3 13 06 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/4b7bff38-5544-4bda-bfce-9a1c8b6b2fde">
  <img width="624" alt="Screenshot 2024-02-07 at 3 12 55 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/f6fc9528-1201-4990-a814-861921a540e6">
</p>

- S3에서 데이터를 불러올 때 모든 데이터를 불러와서 데이터를 불러오는데 많은 시간이 걸림
- 필요한 Column만 불러오기 위해 Schema를 정의함 (기존 5m -> 30s로 90% 시간절약)

2. 변환된 데이터를 Redshift에 저장

<p align="center">
  <img width="627" alt="Screenshot 2024-02-07 at 3 19 33 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/3d967149-460d-4057-852a-29b336b7f88c">
</p>

- 데이터를 아무런 옵션없이 저장하면 기본 DataType이 VARCHAR(255)로 저장이 됨
- DateTime으로 저장되는 Column과 VARCHAR(255)를 넘어가는 Row가 존재하기 때문에 "createTableColumnType" 옵션을 사용해서 DataType을 지정해서 저장

3. ETL 파이프라인 자동화

<p align="center">
  <img width="650" alt="Screenshot 2024-02-07 at 3 29 06 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/d8f071d6-0684-4b64-814d-b35e4c6d3a05">
</p>

- Airflow의 Varialbe 기능을 사용해서 데이터 저장 경로를 저장 -> 모든 DAG가 종료된 이후 저장된 경로에 UNIX TimeStamp로 24시간을 더해 새로운 경로로 저장해서 다음 스케줄링때 경로로 사용
- S3 hook을 이용해 데이터가 저장되어 있는지 확인하고, Slack으로 결과 전달
- 데이터가 정상적으로 저장되어 있으면 "EmrSerlessOperater"에서 "BOTO3"를 사용하여 AWS EMR API를 호출, Spark Job을 제출하고 실행, 작업이 끝나면 Slack으로 결과 전달
- Spark Job을 제출할 때 초기 Memory를 할당하여 실행시간을 단축 (기존 1.17m -> 45s로 40% 시간 절약)
- S3에 저장된 데이터 개수와 Redshift에 저장된 데이터 개수를 비교하여 데이터 정합성 확인후 결과를 Slack으로 전달




