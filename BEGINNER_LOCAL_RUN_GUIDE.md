# Beginner Local Run Guide

이 문서는 `원본 파일을 수정하지 않고`, `apigateway 없이`, `member`, `product`, `ordering` 서비스를 개별적으로 테스트하는 방법을 정리한 초보자용 실행 가이드입니다.

## 목표

현재 프로젝트는 설정상 로컬에서 `apigateway`까지 바로 연결하기 어렵습니다. 따라서 가장 현실적인 방법은 아래 방식입니다.

- 원본 파일 유지
- `apigateway` 사용 안 함
- `member`, `product`, `ordering` 직접 실행
- Postman으로 각 서비스에 직접 요청

## 왜 이 방식으로 해야 하나요?

현재 설정 기준으로는 아래 문제가 있습니다.

- `member`, `product`, `ordering`의 `server.port`가 `0`이라 실행할 때마다 포트가 랜덤으로 바뀝니다.
- 세 서비스의 `local` 설정에 Eureka 주소가 들어 있지만, 보통 초보자 로컬 환경에는 Eureka 서버가 없습니다.
- `apigateway`는 기본 프로필이 `prod-uri`라서 로컬이 아닌 Kubernetes 서비스명으로 라우팅하려고 합니다.

그래서 지금은 `apigateway`를 빼고, 각 서비스를 직접 테스트하는 것이 가장 쉬운 방법입니다.

## 전체 실행 순서

```text
1. Docker Desktop 실행
2. MySQL 실행
3. Redis 실행
4. Zookeeper 실행
5. Kafka 실행
6. member 실행
7. product 실행
8. ordering 실행
9. 각 서비스 콘솔에서 포트 확인
10. Postman으로 직접 API 호출
```

## 1. 사전 준비

PowerShell을 열고 아래를 확인합니다.

### Java 확인

```powershell
java -version
```

### Docker 확인

```powershell
docker --version
```

### Git 확인

```powershell
git --version
```

정상이라면 Java 17, Docker, Git 버전이 출력됩니다.

## 2. 인프라 실행

서비스보다 먼저 아래 4개가 실행되어 있어야 합니다.

- MySQL
- Redis
- Zookeeper
- Kafka

### MySQL 실행

```powershell
docker run -d --name order-mysql `
  -e MYSQL_ROOT_PASSWORD=1234 `
  -e MYSQL_DATABASE=ordermsa `
  -p 3306:3306 `
  mysql:8
```

확인:

```powershell
docker ps
```

### Redis 실행

```powershell
docker run -d --name order-redis -p 6379:6379 redis
```

확인:

```powershell
docker ps
```

### Zookeeper 실행

```powershell
docker run -d --name order-zookeeper `
  -p 2181:2181 `
  -e ALLOW_ANONYMOUS_LOGIN=yes `
  bitnami/zookeeper:latest
```

### Kafka 실행

```powershell
docker run -d --name order-kafka `
  -p 9092:9092 `
  -e KAFKA_CFG_ZOOKEEPER_CONNECT=host.docker.internal:2181 `
  -e ALLOW_PLAINTEXT_LISTENER=yes `
  -e KAFKA_CFG_LISTENERS=PLAINTEXT://:9092 `
  -e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 `
  -e KAFKA_BROKER_ID=1 `
  bitnami/kafka:latest
```

확인:

```powershell
docker ps
```

## 3. 서비스 실행

서비스는 각각 새 PowerShell 창에서 실행하는 것을 권장합니다.

### member 실행

```powershell
Set-Location C:\Users\soldesk\Desktop\k8s.msa\msa\member
.\gradlew.bat bootRun --args="--spring.profiles.active=local"
```

실행 후 콘솔에서 포트를 확인합니다.

예시:

```text
Tomcat started on port 54321
```

이 경우 `member` 주소는 아래와 같습니다.

```text
http://localhost:54321
```

### product 실행

새 PowerShell 창:

```powershell
Set-Location C:\Users\soldesk\Desktop\k8s.msa\msa\product
.\gradlew.bat bootRun --args="--spring.profiles.active=local"
```

실행 후 콘솔에서 포트를 확인합니다.

예시:

```text
Tomcat started on port 54322
```

이 경우 `product` 주소는 아래와 같습니다.

```text
http://localhost:54322
```

### ordering 실행

새 PowerShell 창:

```powershell
Set-Location C:\Users\soldesk\Desktop\k8s.msa\msa\ordering
.\gradlew.bat bootRun --args="--spring.profiles.active=local"
```

실행 후 콘솔에서 포트를 확인합니다.

예시:

```text
Tomcat started on port 54323
```

이 경우 `ordering` 주소는 아래와 같습니다.

```text
http://localhost:54323
```

## 4. 포트 메모하기

현재 프로젝트는 `server.port: 0`이라 실행할 때마다 포트가 바뀔 수 있습니다.

예시 메모:

```text
member   -> http://localhost:54321
product  -> http://localhost:54322
ordering -> http://localhost:54323
```

이 숫자는 매번 달라질 수 있으므로, 실행할 때마다 직접 확인해야 합니다.

## 5. Postman 테스트 순서

현재는 `apigateway`를 사용하지 않기 때문에, 원래 gateway가 넣어주던 인증 헤더를 직접 넣어야 합니다.

사용할 헤더:

```text
X-User-Id: 1
```

## 6. 회원가입

주소 예시:

```text
POST http://localhost:54321/member/create
```

`54321`은 실제 `member` 포트로 바꿔야 합니다.

Body:

```json
{
  "name": "test-user",
  "email": "test1@example.com",
  "password": "1234"
}
```

응답 예시:

```json
1
```

## 7. 로그인

주소 예시:

```text
POST http://localhost:54321/member/doLogin
```

Body:

```json
{
  "email": "test1@example.com",
  "password": "1234"
}
```

응답 예시:

```json
{
  "id": 1,
  "token": "eyJhbGciOi...",
  "refreshToken": "eyJhbGciOi..."
}
```

여기서 `id`와 `token` 값을 확인합니다.

## 8. 상품 등록

주소 예시:

```text
POST http://localhost:54322/product/create
```

Header:

```text
X-User-Id: 1
```

주의:
현재 코드상 `product/create`는 `@RequestBody`가 아니라서 JSON이 바로 안 될 수 있습니다. 이 경우 Postman에서 `form-data` 또는 `x-www-form-urlencoded`로 전송해야 합니다.

예시 값:

```text
name = keyboard
price = 50000
stockQuantity = 20
```

## 9. 상품 조회

주소 예시:

```text
GET http://localhost:54322/product/1
```

Header:

```text
X-User-Id: 1
```

응답 예시:

```json
{
  "id": 1,
  "name": "keyboard",
  "price": 50000,
  "stockQuantity": 20
}
```

## 10. 주문 생성

주소 예시:

```text
POST http://localhost:54323/ordering/create
```

Header:

```text
X-User-Id: 1
```

Body:

```json
{
  "productId": 1,
  "productCount": 2
}
```

응답 예시:

```json
1
```

## 11. 왜 Authorization 대신 X-User-Id를 쓰나요?

원래는 `apigateway`가 JWT를 확인한 뒤 내부 서비스에 `X-User-Id`를 넣어줍니다. 하지만 지금은 gateway를 사용하지 않고 서비스에 직접 요청하므로, 그 헤더를 Postman에서 직접 넣어주는 방식으로 테스트합니다.

즉, 지금은 운영 방식이 아니라 기능 테스트 방식입니다.

## 12. 자주 막히는 문제

### 포트를 못 찾는 경우

콘솔 로그에서 아래 문장을 찾습니다.

```text
Tomcat started on port ...
```

### DB 연결 실패

아래를 확인합니다.

- MySQL 컨테이너가 실행 중인지
- DB 이름이 `ordermsa`인지
- 비밀번호가 `1234`인지

### Redis 연결 실패

아래를 확인합니다.

```powershell
docker ps
```

### Kafka 연결 실패

아래를 확인합니다.

- Zookeeper가 먼저 실행됐는지
- Kafka가 실행 중인지
- `localhost:9092`가 열려 있는지

### product/create가 JSON으로 안 되는 경우

Postman의 `Body -> form-data` 또는 `x-www-form-urlencoded`를 사용합니다.

## 13. 초보자용 최종 체크리스트

```text
[ ] Docker Desktop 실행
[ ] MySQL 실행
[ ] Redis 실행
[ ] Zookeeper 실행
[ ] Kafka 실행
[ ] member 실행
[ ] member 포트 메모
[ ] product 실행
[ ] product 포트 메모
[ ] ordering 실행
[ ] ordering 포트 메모
[ ] member/create 요청 성공
[ ] member/doLogin 요청 성공
[ ] product/create 요청 성공
[ ] product/1 조회 성공
[ ] ordering/create 요청 성공
```

## 14. 가장 현실적인 목표

처음부터 모든 것을 완벽히 연결하는 것보다 아래 3가지를 먼저 성공하는 것이 중요합니다.

- `member` 실행 성공
- 로그인 API 성공
- `product`, `ordering`까지 직접 호출 성공

여기까지 되면 다음 단계로 Kubernetes나 Argo CD를 보는 것이 훨씬 쉽습니다.
