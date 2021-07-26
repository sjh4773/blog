---
title: "AWS RDS로 데이터베이스 구축 및 Heidi SQL를 이용해 AWS RDS 접속"
date: 2021-07-27T00:53:47+09:00
categories:
  - "AWS"
tags:
  - "AWS"
---

# Amazon RDS 생성

1. MySQL 선택

![](/image/RDS1.png)

2. MySQL에 접속할 때 사용하는 id, password 설정

![](/image/RDS2.png)

3. 고급 설정 구성, VPC 및 서브넷 그룹은 기본 설정 그대로 사용
   퍼블릭 액세스 가능성을 '예'로 설정하면 전 세계 어디에서나 데이터베이스 주소를 이용해서 접근 가능하다.
   반면에 VPC라는 일종의 내부망 기능을 이용해서 자신이 만든 EC2에서만 접근하도록 만들고 싶다면 '아니오'로 설정한다.
   간단히 테스트 목적으로 사용할 경우 '예'로 설정한다.

![](/image/RDS3.png)

4. 삭제 방지 활성화 체크

## 파라미터 그룹 생성

데이터베이스가 한글 처리를 할 수 있도록 파라미터 그룹 생성

1. 파라미터 그룹 생성

![](/image/RDS4.png)

2. 파라미터 편집
- 유니코드 사용 명시(값에 utf8 기입)
   - character_set_client, character_set_connection, character_set_database, character_set_filesystem, character_set_results, character_set_server
- 유니코드 사용 명시(값에 utf8_general_ci 기입)
   - collation_connection, collation_server

![](/image/RDS5.png)

3. 데이터베이스 옵션의 DB 파라미터 그룹 변경

![](/image/RDS6.png)


![](/image/RDS7.png)




            퍼블릭 액세스 가능성이 '예'로 설정이 되어 있어 외부에서 접속이 가능함.

            엔드포인트는 말 그대로 서버의 주소를 의미하여 이 도메인 주소로 MySQL 접속 가능.

            VPC에는 방화벽 설정 가능
            데이터베이스는 방화벽 규칙이 엄격한편이라 접속할 수 있는 IP 주소가 고정되어있는 경우가 많음.
            따라서 편집을 통해 전 세계 어디에서 접속할 수 있도록 위치무관으로 변경.

            네트워크 및 보안에서 보안 그룹 내의 인바운드 규칙에서 Edit inbound rules 클릭




### 문제

보안 그룹 인바운드 규칙 변경 시, 아래와 같은 오류가 갑자기 나타났다.
You may not specify a referenced group id for an existing IPv4 CIDR rule.


![](/image/RDS8.png)

### 해결
문제가 되는 해당 규칙 삭제 후, 새 규칙 추가하여 해결.


![](/image/RDS9.png)


# Heidi SQL을 이용해 간편하게 AWS RDS 접속하기

![](/image/RDS10.png)


1. 네트워크 유형으로 MySQL 설정
2. 신규 버튼을 눌러 세션 생성 
3. 호스트 주소에는 엔드포인트를 넣는다.
4. 전에 설정했던 데이터베이스 사용자 및 암호를 넣는다.
5. 열기 버튼을 눌러 접속 시도 

특정 데이터베이스에 쿼리를 날리고 싶으면 다음과 같이 설정하고 버전을 출력해본다.

![](/image/RDS11.png)



            실제로 상용서비스를 운영할 때 퍼블릭에서 데이터베이스에 바로 접근할 수 있도록 하면 위험할 수 있다.
            그렇기 때문에 반드시 같은 내부망 즉 VPC에 있는 EC2에서만 접속할 수 있도록 보안 설정을 해줘야한다.
            그래서 일반적으로 개발 단계에 있을 때 퍼블릭에 접근할 수 있도록 해서 다양한 SQL 문장을 테스트 해보다가
            나중에 실제로 서비스가 배포가 될 때는 퍼블릭 액세스 가능성을 아니오로 바꿔주고 보안 설정을 해줘야한다.