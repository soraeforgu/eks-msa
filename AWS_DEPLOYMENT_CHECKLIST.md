# AWS Deployment Checklist

이 문서는 `C:\Users\soldesk\Desktop\k8s.msa` 프로젝트를 AWS에 배포할 때 따라갈 수 있는 체크리스트입니다.

대상 흐름은 아래와 같습니다.

```text
ECR -> EKS -> kubectl -> Argo CD -> Domain 연결 -> 외부 테스트
```

## 1. 배포 전 준비

### AWS / 로컬 준비
- [ ] AWS 계정 준비
- [ ] AWS CLI 설치
- [ ] Docker Desktop 설치
- [ ] kubectl 설치
- [ ] Git 설치
- [ ] PowerShell 실행 가능

### AWS 인증 확인
- [ ] `aws configure` 완료
- [ ] AWS 계정 접근 가능
- [ ] 리전 확인: `ap-northeast-2`

확인 명령어:

```powershell
aws sts get-caller-identity
```

## 2. ECR 준비

서비스별 ECR 저장소가 있어야 합니다.

- [ ] `apigateway`
- [ ] `member-service`
- [ ] `product-service`
- [ ] `ordering-service`

필요 시 생성 예시:

```powershell
aws ecr create-repository --repository-name apigateway --region ap-northeast-2
aws ecr create-repository --repository-name member-service --region ap-northeast-2
aws ecr create-repository --repository-name product-service --region ap-northeast-2
aws ecr create-repository --repository-name ordering-service --region ap-northeast-2
```

체크:
- [ ] ECR 저장소 4개 생성 확인

## 3. Docker 이미지 빌드

### apigateway
```powershell
Set-Location C:\Users\soldesk\Desktop\k8s.msa\msa\apigateway
docker build -t apigateway:latest .
```

### member
```powershell
Set-Location C:\Users\soldesk\Desktop\k8s.msa\msa\member
docker build -t member-service:latest .
```

### product
```powershell
Set-Location C:\Users\soldesk\Desktop\k8s.msa\msa\product
docker build -t product-service:latest .
```

### ordering
```powershell
Set-Location C:\Users\soldesk\Desktop\k8s.msa\msa\ordering
docker build -t ordering-service:latest .
```

체크:
- [ ] apigateway 이미지 빌드 성공
- [ ] member 이미지 빌드 성공
- [ ] product 이미지 빌드 성공
- [ ] ordering 이미지 빌드 성공

## 4. ECR 로그인 및 Push

### ECR 로그인
```powershell
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com
```

### 태그 변경
```powershell
docker tag apigateway:latest YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/apigateway:latest
docker tag member-service:latest YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/member-service:latest
docker tag product-service:latest YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/product-service:latest
docker tag ordering-service:latest YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/ordering-service:latest
```

### Push
```powershell
docker push YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/apigateway:latest
docker push YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/member-service:latest
docker push YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/product-service:latest
docker push YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/ordering-service:latest
```

체크:
- [ ] ECR 로그인 성공
- [ ] apigateway push 완료
- [ ] member push 완료
- [ ] product push 완료
- [ ] ordering push 완료

## 5. EKS 준비

체크:
- [ ] EKS 클러스터 생성 완료
- [ ] 노드 그룹 생성 완료
- [ ] kubectl 연결 가능

kubeconfig 연결:

```powershell
aws eks update-kubeconfig --name my-cluster --region ap-northeast-2
```

노드 확인:

```powershell
kubectl get nodes
```

정상이라면 노드 목록이 보여야 합니다.

## 6. namespace 및 Secret 생성

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

체크:
- [ ] `soldesk` namespace 생성 완료
- [ ] `my-app-secrets` 생성 완료

확인:

```powershell
kubectl get ns
kubectl get secret -n soldesk
```

## 7. 공용 인프라 배포

아래 리소스를 먼저 적용합니다.

### Zookeeper
```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-services\zookeeper.yml
```

### Kafka
```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-services\kafka.yml
```

### Redis
```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-services\redis.yml
```

체크:
- [ ] Zookeeper 배포 완료
- [ ] Kafka 배포 완료
- [ ] Redis 배포 완료

확인:

```powershell
kubectl get pods -n soldesk
kubectl get svc -n soldesk
```

## 8. 마이크로서비스 배포

### member
```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\member\k8s\depl_svc.yml
```

### product
```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\product\k8s\depl_svc.yml
```

### ordering
```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\ordering\k8s\depl_svc.yml
```

### apigateway
```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\apigateway\k8s\depl_svc.yml
```

체크:
- [ ] member 배포 완료
- [ ] product 배포 완료
- [ ] ordering 배포 완료
- [ ] apigateway 배포 완료

확인:

```powershell
kubectl get deploy -n soldesk
kubectl get pods -n soldesk
kubectl get svc -n soldesk
```

## 9. Ingress / HTTPS 적용

### Ingress 적용
```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-services\ingress.yml
```

### HTTPS 적용
```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-services\https.yml
```

체크:
- [ ] Ingress 생성 완료
- [ ] TLS/HTTPS 적용 완료

확인:

```powershell
kubectl get ingress -n soldesk
```

## 10. HPA 적용

```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-hpa\member-hpa.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-hpa\product-hpa.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-hpa\ordering-hpa.yml
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-hpa\apigateway-hpa.yml
```

체크:
- [ ] member HPA 적용
- [ ] product HPA 적용
- [ ] ordering HPA 적용
- [ ] apigateway HPA 적용

확인:

```powershell
kubectl get hpa -n soldesk
```

## 11. Argo CD 적용

### Root App 적용
```powershell
kubectl apply -f C:\Users\soldesk\Desktop\k8s.msa\argocd\root-app.yml
```

체크:
- [ ] Root App 적용 완료
- [ ] Argo CD에서 앱 동기화 확인

기대 동작:
- `argocd/apps` 아래 리소스를 읽음
- 서비스, HPA, monitoring 등을 GitOps 기준으로 관리

## 12. 도메인 연결

체크:
- [ ] Load Balancer 주소 확인
- [ ] Route53 또는 사용 중인 DNS에 도메인 연결
- [ ] 도메인이 Ingress로 연결되는지 확인

확인 포인트:
- `https://your-domain/member/create`
- `https://your-domain/member/doLogin`
- `https://your-domain/product/1`
- `https://your-domain/ordering/create`

## 13. 최종 외부 테스트

### 회원가입
- [ ] `POST /member/create` 성공

### 로그인
- [ ] `POST /member/doLogin` 성공
- [ ] token 발급 확인

### 상품 조회
- [ ] `GET /product/{id}` 성공

### 주문 생성
- [ ] `POST /ordering/create` 성공

## 14. 장애 확인용 명령어

### 전체 Pod 확인
```powershell
kubectl get pods -n soldesk
```

### 서비스 확인
```powershell
kubectl get svc -n soldesk
```

### Ingress 확인
```powershell
kubectl get ingress -n soldesk
```

### 특정 Pod 로그 확인
```powershell
kubectl logs POD_NAME -n soldesk
```

### 배포 상태 확인
```powershell
kubectl rollout status deployment/member-depl -n soldesk
kubectl rollout status deployment/product-depl -n soldesk
kubectl rollout status deployment/ordering-depl -n soldesk
kubectl rollout status deployment/apigateway-depl -n soldesk
```

## 15. 최종 요약

```text
1. ECR 저장소 준비
2. 이미지 4개 빌드
3. 이미지 4개 Push
4. EKS 연결
5. namespace / Secret 생성
6. Redis / Zookeeper / Kafka 배포
7. member / product / ordering / apigateway 배포
8. Ingress / HTTPS 적용
9. HPA 적용
10. Argo CD Root App 적용
11. 도메인 연결
12. 외부 API 테스트
```
