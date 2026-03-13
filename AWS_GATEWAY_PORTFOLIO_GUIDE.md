# AWS Gateway Portfolio Guide

이 문서는 `apigateway를 사용하는 구조`를 기준으로, 이 프로젝트를 AWS/EKS에 배포해 포트폴리오로 사용하는 흐름을 정리한 가이드입니다.

중요한 전제는 아래와 같습니다.

- 원본 프로젝트 파일은 수정하지 않습니다.
- 로컬에서 `apigateway`까지 완전히 맞춰 실행하는 것이 아니라, AWS/EKS 배포 구조 기준으로 설명합니다.
- 외부 사용자는 `apigateway`를 통해서만 서비스에 접근합니다.

## 1. 왜 apigateway를 사용하는가

이 프로젝트를 실제 AWS에 올려 포트폴리오로 사용할 경우, `apigateway`는 있는 편이 훨씬 좋습니다.

이유는 다음과 같습니다.

- 외부 요청을 한 곳으로 모을 수 있습니다.
- 인증과 JWT 검증을 gateway에서 먼저 처리할 수 있습니다.
- `member`, `product`, `ordering`를 외부에 직접 노출하지 않아도 됩니다.
- 포트폴리오에서 MSA 구조를 설명하기 쉬워집니다.
- 도메인 기준으로 `/member`, `/product`, `/ordering` 경로를 일관되게 운영할 수 있습니다.

즉, AWS에서는 보통 아래 구조로 보는 것이 자연스럽습니다.

```text
사용자
-> Domain / LoadBalancer / Ingress
-> apigateway
-> member / product / ordering
```

## 2. 전체 아키텍처 흐름

```text
Client
-> Route53 or Domain
-> AWS Load Balancer / Ingress
-> apigateway
-> member / product / ordering

member   -> Redis, MySQL
product  -> MySQL, Kafka Consumer
ordering -> MySQL, Kafka Producer
Kafka    -> Zookeeper
```

## 3. 외부 접속 방식

외부 사용자는 보통 아래처럼 접속합니다.

- `https://your-domain/member/create`
- `https://your-domain/member/doLogin`
- `https://your-domain/product/1`
- `https://your-domain/ordering/create`

겉으로는 하나의 API 서버처럼 보이지만, 내부적으로는 gateway가 각 마이크로서비스로 라우팅합니다.

## 4. 이 프로젝트에서 실제 역할 분리

### apigateway
- 외부 요청 진입점
- JWT 검증
- 내부 서비스로 라우팅

### member
- 회원가입
- 로그인
- Refresh Token 발급
- Redis 사용

### product
- 상품 등록
- 상품 조회
- 재고 관리
- Kafka Consumer 역할

### ordering
- 주문 생성
- product 서비스 호출
- Kafka 이벤트 발행

## 5. AWS 포트폴리오용 실행 전략

이 프로젝트를 AWS에서 포트폴리오용으로 보여주려면 아래 순서가 가장 현실적입니다.

```text
1. GitHub Repository 준비
2. Docker 이미지 빌드
3. Amazon ECR에 이미지 Push
4. Amazon EKS 준비
5. Kubernetes 매니페스트 적용
6. Ingress / HTTPS 연결
7. Argo CD 연결
8. 외부 접속 테스트
```

## 6. 실제 배포 순서 체크리스트

### A. 준비
- [ ] AWS 계정 준비
- [ ] ECR 저장소 준비
- [ ] EKS 클러스터 준비
- [ ] kubectl 연결 확인
- [ ] GitHub 저장소 확인
- [ ] Docker 설치 확인

### B. 이미지 빌드
- [ ] apigateway 이미지 빌드
- [ ] member 이미지 빌드
- [ ] product 이미지 빌드
- [ ] ordering 이미지 빌드

### C. 이미지 Push
- [ ] ECR 로그인
- [ ] apigateway Push
- [ ] member Push
- [ ] product Push
- [ ] ordering Push

### D. Kubernetes 배포
- [ ] namespace 생성
- [ ] Secret 생성
- [ ] Redis / Zookeeper / Kafka 배포
- [ ] member / product / ordering / apigateway 배포
- [ ] Ingress 적용
- [ ] HTTPS 적용
- [ ] HPA 적용

### E. GitOps
- [ ] Argo CD 설치
- [ ] root-app 적용
- [ ] 앱 동기화 확인

### F. 외부 테스트
- [ ] 회원가입 테스트
- [ ] 로그인 테스트
- [ ] 상품 조회 테스트
- [ ] 주문 생성 테스트

## 7. Windows PowerShell 기준 실행 명령어

### 7-1. Docker 이미지 빌드

#### apigateway
```powershell
Set-Location C:\Users\soldesk\Desktop\k8s.msa\msa\apigateway
docker build -t apigateway:latest .
```

#### member
```powershell
Set-Location C:\Users\soldesk\Desktop\k8s.msa\msa\member
docker build -t member-service:latest .
```

#### product
```powershell
Set-Location C:\Users\soldesk\Desktop\k8s.msa\msa\product
docker build -t product-service:latest .
```

#### ordering
```powershell
Set-Location C:\Users\soldesk\Desktop\k8s.msa\msa\ordering
docker build -t ordering-service:latest .
```

### 7-2. ECR 로그인

```powershell
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com
```

### 7-3. 이미지 태그 변경

#### apigateway
```powershell
docker tag apigateway:latest YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/apigateway:latest
```

#### member
```powershell
docker tag member-service:latest YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/member-service:latest
```

#### product
```powershell
docker tag product-service:latest YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/product-service:latest
```

#### ordering
```powershell
docker tag ordering-service:latest YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/ordering-service:latest
```

### 7-4. 이미지 Push

#### apigateway
```powershell
docker push YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/apigateway:latest
```

#### member
```powershell
docker push YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/member-service:latest
```

#### product
```powershell
docker push YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/product-service:latest
```

#### ordering
```powershell
docker push YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/ordering-service:latest
```

## 8. Kubernetes 적용 순서

### namespace 생성

```powershell
kubectl create namespace soldesk
```

### Secret 생성

```powershell
kubectl create secret generic my-app-secrets `
  --from-literal=DB_HOST=YOUR_DB_HOST `
  --from-literal=DB_PW=YOUR_DB_PASSWORD `
  -n soldesk
```

### 공용 서비스 배포

```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-services\zookeeper.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-services\kafka.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-services\redis.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-services\ingress.yml
```

### 마이크로서비스 배포

```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\member\k8s\depl_svc.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\product\k8s\depl_svc.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\ordering\k8s\depl_svc.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\apigateway\k8s\depl_svc.yml
```

### HPA 적용

```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-hpa\member-hpa.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-hpa\product-hpa.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-hpa\ordering-hpa.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-hpa\apigateway-hpa.yml
```

## 9. Argo CD 적용

### root app 적용

```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\argocd\root-app.yml
```

이후 Argo CD가 `argocd/apps` 아래 Application을 읽고 전체 서비스를 동기화합니다.

## 10. 외부 테스트 예시

배포가 끝나면 외부에서는 gateway 기준으로 아래 주소를 사용합니다.

### 회원가입
```text
POST https://your-domain/member/create
```

### 로그인
```text
POST https://your-domain/member/doLogin
```

### 상품 조회
```text
GET https://your-domain/product/1
Authorization: Bearer {token}
```

### 주문 생성
```text
POST https://your-domain/ordering/create
Authorization: Bearer {token}
```

## 11. 포트폴리오에서 어떻게 설명하면 좋은가

아래처럼 설명하면 좋습니다.

`외부 요청은 API Gateway를 통해 수신하고, JWT 인증 및 라우팅을 처리한 뒤 내부의 member, product, ordering 마이크로서비스로 전달하도록 구성했습니다. 서비스는 Kubernetes에 배포하고, GitHub Actions와 Argo CD를 사용해 이미지 배포와 GitOps 기반 운영 흐름을 구성했습니다.`

## 12. 현실적으로 기억할 점

- 로컬 테스트와 AWS 배포 구조는 다르게 볼 필요가 있습니다.
- 로컬에서는 gateway 없이 개별 서비스 테스트가 더 쉽습니다.
- AWS 포트폴리오에서는 gateway를 포함한 구조가 더 자연스럽고 설득력이 있습니다.

즉, 지금은 아래처럼 생각하면 됩니다.

```text
로컬 학습/디버깅 -> gateway 없이 개별 서비스 테스트
AWS 포트폴리오 배포 -> gateway 포함 구조 사용
```
