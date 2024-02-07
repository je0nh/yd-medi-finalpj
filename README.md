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

**1. 기존 아키텍쳐 문제점**
- Redshift에서 모든 데이터의 변환을 진행함
- 변환 데이터의 크기가 커지고, 여러 데이터 변환을 진행하다 보니 변환 속도가 느려지고 Redshift에 부하가 오기시작함

**2. 기업 요구사항**
- 기존 Redshift에서 변환되던 Query를 Spark Query로 변환하기

<p align="center">
  <img width="131" alt="Screenshot 2024-02-07 at 3 49 14 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/aac93b79-9a69-45b5-835b-b28e3dea8c1c">
</p>

- 기업에서 기존 제공한 Query 이외의 추가로 제공된 Query 변환 (15개)

**3. 추가 작업 (기업 요구사항 x)**
- Airflow를 이용한 ETL 파이프라인 자동화
- GitHub Action을 이용한 CI환경 구축

# 프로젝트 진행과정
**1. Redshift query를 Spark query로 변환**
<p align="center">
  <img width="620" alt="Screenshot 2024-02-07 at 3 13 06 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/4b7bff38-5544-4bda-bfce-9a1c8b6b2fde">
  <img width="620" alt="Screenshot 2024-02-07 at 3 12 55 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/f6fc9528-1201-4990-a814-861921a540e6">
</p>

- S3에서 데이터를 불러올 때 모든 데이터를 불러와서 데이터를 불러오는데 많은 시간이 걸림
- 필요한 Column만 불러오기 위해 Schema를 정의함 (기존 5m -> 30s로 90% 시간절약)

**2. 변환된 데이터를 Redshift에 저장**
<p align="center">
  <img width="620" alt="Screenshot 2024-02-07 at 3 19 33 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/3d967149-460d-4057-852a-29b336b7f88c">
</p>

- 데이터를 아무런 옵션없이 저장하면 기본 DataType이 VARCHAR(255)로 저장이 됨
- DateTime으로 저장되는 Column과 VARCHAR(255)를 넘어가는 Row가 존재하기 때문에 "createTableColumnType" 옵션을 사용해서 DataType을 지정해서 저장

**3. ETL 파이프라인 자동화**
<p align="center">
  <img width="620" alt="Screenshot 2024-02-07 at 3 29 06 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/d8f071d6-0684-4b64-814d-b35e4c6d3a05">
</p>

- Airflow의 Varialbe 기능을 사용해서 데이터 저장 경로를 저장 -> 모든 DAG가 종료된 이후 저장된 경로에 UNIX TimeStamp로 24시간을 더해 새로운 경로로 저장해서 다음 스케줄링때 경로로 사용
- S3 hook을 이용해 데이터가 저장되어 있는지 확인하고, Slack으로 결과 전달
- 데이터가 정상적으로 저장되어 있으면 "EmrSerlessOperater"에서 "BOTO3"를 사용하여 AWS EMR API를 호출, Spark Job을 제출하고 실행, 작업이 끝나면 Slack으로 결과 전달
- Spark Job을 제출할 때 초기 Memory를 할당하여 실행시간을 단축 (기존 1.17m -> 45s로 40% 시간 절약)
- S3에 저장된 데이터 개수와 Redshift에 저장된 데이터 개수를 비교하여 데이터 정합성 확인후 결과를 Slack으로 전달

**4. GitHub Action을 이용하여 CI환경 구축**
<p align="center">
  <img width="620" alt="Screenshot 2024-02-07 at 3 37 32 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/22641c11-8eb7-491e-a575-a3d1151d7969">
</p>


## 사용 기술 스택
**Environment** <br>
<img src="https://img.shields.io/badge/jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white">
<img src="https://img.shields.io/badge/git-F05032?style=for-the-badge&logo=git&logoColor=white">
<img src="https://img.shields.io/badge/github-181717?style=for-the-badge&logo=github&logoColor=white">

**Language** <br>
<img src="https://img.shields.io/badge/python-3776AB?style=for-the-badge&logo=python&logoColor=white">
<img src="https://img.shields.io/badge/scala-DC322F?style=for-the-badge&logo=scala&logoColor=white">

**Config** <br>
<img src="https://img.shields.io/badge/amazonec2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white">
<img src="https://img.shields.io/badge/amazons3-569A31?style=for-the-badge&logo=amazons3&logoColor=white">
<img src="https://img.shields.io/badge/amazonredshift-8C4FFF?style=for-the-badge&logo=amazonredshift&logoColor=white">
<img src="https://img.shields.io/badge/ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white">

**Framework** <br>
<img src="https://img.shields.io/badge/apacheairflow-017CEE?style=for-the-badge&logo=apacheairflow&logoColor=white">
<img src="https://img.shields.io/badge/apachespark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white">

**Communication** <br>
<img src="https://img.shields.io/badge/notion-000000?style=for-the-badge&logo=notion&logoColor=white">
<img src="https://img.shields.io/badge/slack-4A154B?style=for-the-badge&logo=slack&logoColor=white">
<img src="https://img.shields.io/badge/discord-5865F2?style=for-the-badge&logo=discord&logoColor=white">


# 프로젝트 한계
- EMR Cluser 한대로 20명이 넘는 인원이 사용하다보니 EMR이 과부하가 걸려 여러명이 사용할수 없는 문제가 발생함
- 위의 문제 때문에 개인 로컬에 EMR과 비슷한 환경을 셋팅해서 사용했지만 Spark query 변환시 scala를 사용하지 못해 추가 query는 pyspark로만 query 변환을 진행함

# ETL 파이프라인 자동화 시연
[<p align="center"><img width="620" alt="Screenshot 2024-02-07 at 4 25 10 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/f22229f0-1ec5-4530-a7a3-0ce7efaeccf5"></p>](https://www.youtube.com/watch?v=fr_N-lJF0VI)

# 발표자료
[<p align="center"><img width="402" alt="Screenshot 2024-02-07 at 4 29 27 PM" src="https://github.com/je0nh/yd-medi-finalpj/assets/145730125/80f8b21f-0fee-4205-bea0-46fe1769e3f8"></p>]([스타트업연계프로젝트_최종발표회_메디스트림_3팀v1.5.pdf](https://github.com/je0nh/yd-medi-finalpj/files/14190430/_._._3.v1.5.pdf))
