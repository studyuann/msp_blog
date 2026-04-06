---
title: AWS Route53 DNS 및 Routing Policy
source: Notion
created: 2026-03-31
---

# AWS Route53 DNS 및 Routing Policy

---

# 1. DNS 개념

DNS는 **도메인 이름을 IP 주소로 변환하는 시스템**이다.

예

```
www.example.com → 54.180.10.20
```

사용자가 웹 브라우저에 도메인을 입력하면 다음 순서로 DNS 조회가 진행된다.

```
사용자 PC
   │
   ▼
Local DNS Resolver
   │
   ▼
Root DNS
   │
   ▼
TLD DNS (.com)
   │
   ▼
Authoritative DNS
(Route53)
   │
   ▼
IP 반환
```

Route53은 **Authoritative DNS 서버 역할**을 한다.

---

# 2. Route53 서비스 개념

Route53은 AWS에서 제공하는 **Managed DNS 서비스**이다.

주요 기능

| 기능 | 설명 |
| --- | --- |
| Domain Registration | 도메인 구매 |
| Hosted Zone | DNS 영역 관리 |
| Record | DNS 레코드 설정 |
| Health Check | 서비스 상태 확인 |
| Routing Policy | 트래픽 제어 |

이번 실습에서 사용할 기능

```
Hosted Zone
DNS Record
Routing Policy
```

---

# 3. 실습 전체 아키텍처

교육 환경 구조

```
                 Internet
                     │
                     │
               example.com
                     │
              Route53 Hosted Zone
                     │
       ┌─────────────┼─────────────┐
       │             │             │
student1.example.com
student2.example.com
student3.example.com
       │
       │
       EC2
```

학생은 **자신의 서브도메인을 생성하여 서버를 연결**한다.

---

# 4. 실습 준비

### 1. 외부 도메인 구매

예

```
cloudai.store
```

구매 가능 사이트

```
가비아
Namecheap
GoDaddy
Cloudflare
```

---

# 5. Route53 Hosted Zone 생성

## 5.1 Route53 접속

AWS Console

```
Route53
```

메뉴

```
Hosted zones
```

---

## 5.2 Hosted Zone 생성

클릭

```
Create hosted zone
```

설정

```
Domain name

cloudai.store
```

Type

```
Public Hosted Zone
```

생성

```
Create hosted zone
```

---

## 5.3 생성 결과

Route53이 자동으로 NS 레코드를 생성한다.

예

```
ns-123.awsdns-12.com
ns-234.awsdns-34.net
ns-345.awsdns-56.org
ns-456.awsdns-78.co.uk
```

이 값은 **외부 도메인 DNS 설정에 등록해야 한다.**

---

# 6. 외부 도메인 NS 변경

도메인 구매 사이트에서

```
cloudai.store
```

DNS 설정 변경

기존

```
ns1.provider.com
ns2.provider.com
```

삭제

Route53 NS 등록

```
ns-123.awsdns-12.com
ns-234.awsdns-34.net
ns-345.awsdns-56.org
ns-456.awsdns-78.co.uk
```

---

# 7. DNS 전파 확인

명령어

```
dig NS cloudai.store
```

또는

```
nslookup cloudai.store
```

DNS 전파 시간

```
5~30분
```

---

# 8. 학생 서브도메인 생성

각 학생은 자신의 서브도메인을 생성한다.

예

```
이니셜.cloudai.store
st2.cloudai.store
st3.cloudai.store
```

---

# 9. EC2 웹서버 생성

EC2 생성

이름

```
이니셜-web
```

보안그룹

```
SSH 22
HTTP 80
```

---

# 10. 웹서버 설치

EC2 접속

```
ssh ec2-user@EC2-IP
```

웹서버 설치

```
sudo yum install httpd -y
```

서비스 시작

```
sudo systemctl start httpd
```

자동 시작

```
sudo systemctl enable httpd
```

---

# 11. 웹페이지 생성

```
sudo vi /var/www/html/index.html
```

내용

```
<h1>이니셜 web server</h1>
```

---

# 12. Route53 DNS 레코드 생성

Route53

```
Hosted zones
```

이동

```
example.com
```

---

## 레코드 생성

클릭

```
Create record
```

설정

```
Record name

student1
```

Type

```
A Record
```

Value

```
EC2 Public IP
```

예

```
54.180.20.10
```

---

# 13. DNS 테스트

```
nslookup 이니셜.cloudai.store
```

또는

```
dig 이니셜.cloudai.store
```

브라우저 접속

```
http://이니셜.cloudai.store
```

---

# 14. Route53 Routing Policy

Routing Policy는 **DNS 트래픽을 제어하는 방식**이다.

주요 정책

| Routing Policy | 설명 |
| --- | --- |
| Simple | 기본 DNS |
| Weighted | 트래픽 비율 분산 |
| Latency | 사용자 위치 기준 |
| Failover | 장애 시 서버 변경 |
| Geolocation | 국가 기준 라우팅 |
| Multi-value | 여러 IP 반환 |

---

# 15. Weighted Routing 실습

Weighted Routing은 **트래픽을 비율로 분산**한다.

예

```
www.이니셜.cloudai.store
     │
 ┌───┴────┐
 │        │
EC2-A   EC2-B
 80%     20%
```

---

## EC2 서버 2개 생성

```
web-a
web-b
```

웹페이지 생성

서버 A

```
<h1>server A</h1>
```

서버 B

```
<h1>server B</h1>
```

---

## Route53 레코드 생성

Record name

```
www.이니셜.cloudai.store
```

Routing Policy

```
Weighted
```

---

### 레코드1

```
Value

EC2-A-IP
```

Weight

```
80
```

---

### 레코드2

```
Value

EC2-B-IP
```

Weight

```
20
```

---

## 테스트

브라우저 새로고침 반복

```
http://www.이니셜.cloudai.store
```

서버 A가 더 많이 나타난다.

---

# 16. Failover Routing 실습

Failover는 **장애 발생 시 서버를 변경**한다.

구조

```
Primary Server
      │
      │ 장애 발생
      ▼
Secondary Server
```

---

## 서버 2개 준비

```
web-primary
web-backup
```

---

## Health Check 생성

Route53

```
Health Checks
```

생성

설정

```
Protocol

HTTP
```

Endpoint

```
Primary server IP
```

---

## DNS 설정

Record name

```
fail.example.com
```

Routing policy

```
Failover
```

---

Primary record

```
Primary server IP
```

Health Check 연결

---

Secondary record

```
Backup server IP
```

Failover type

```
Secondary
```

---

## 테스트

Primary 서버 중지

```
sudo systemctl stop httpd
```

DNS 자동 변경 확인

```
fail.이니셜.cloudai.store
```

Backup 서버로 접속된다.

---

# 17. Latency Routing 실습

Latency Routing은 **사용자와 가장 가까운 리전에 연결**한다.

구조

```
User
 │
 │
 ├─ Asia → ap-northeast-2
 │
 └─ US → us-east-1
```

---

## 서버 생성

```
Seoul EC2
Virginia EC2
```

---

## Route53 설정

Record name

```
latency.이니셜.cloudai.store
```

Routing policy

```
Latency
```

Region 지정

```
ap-northeast-2
us-east-1
```

---