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
