# AWS Value Replacement Guide

이 문서는 `C:\Users\soldesk\Desktop\k8s.msa` 프로젝트를 AWS에 배포할 때, 강사님 기준 값에서 본인 AWS 기준 값으로 바꿔야 하는 포인트를 파일별로 정리한 가이드입니다.

형식은 아래와 같습니다.

- 파일 경로
- 현재값
- 바꿀값
- 왜 바꾸는지

## 1. GitHub Actions 워크플로

파일:
`C:\Users\soldesk\Desktop\k8s.msa\.github\workflows\deploy_ordermsa_with_k8s.yml`

### 바꿀 값 1
현재값:
`895421295243.dkr.ecr.ap-northeast-2.amazonaws.com`

바꿀값:
`YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com`

왜 바꾸는가:
- Docker 이미지를 본인 ECR로 push해야 하기 때문입니다.

### 바꿀 값 2
현재값:
`my-cluster`

바꿀값:
`YOUR_EKS_CLUSTER_NAME`

왜 바꾸는가:
- GitHub Actions가 본인 EKS 클러스터에 연결해야 하기 때문입니다.

### 추가 확인
GitHub 저장소 Secrets에서 아래 값도 본인 값으로 넣어야 합니다.

- `AWS_KEY`
- `AWS_SECRET`

## 2. apigateway Deployment 이미지

파일:
`C:\Users\soldesk\Desktop\k8s.msa\msa\apigateway\k8s\depl_svc.yml`

현재값:
`image: 895421295243.dkr.ecr.ap-northeast-2.amazonaws.com/apigateway:latest`

바꿀값:
`image: YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/apigateway:latest`

왜 바꾸는가:
- apigateway 이미지를 본인 ECR에서 pull해야 하기 때문입니다.

## 3. member Deployment 이미지

파일:
`C:\Users\soldesk\Desktop\k8s.msa\msa\member\k8s\depl_svc.yml`

현재값:
`image: 895421295243.dkr.ecr.ap-northeast-2.amazonaws.com/member-service:latest`

바꿀값:
`image: YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/member-service:latest`

왜 바꾸는가:
- member 이미지를 본인 ECR에서 pull해야 하기 때문입니다.

## 4. product Deployment 이미지

파일:
`C:\Users\soldesk\Desktop\k8s.msa\msa\product\k8s\depl_svc.yml`

현재값:
`image: 895421295243.dkr.ecr.ap-northeast-2.amazonaws.com/product-service:latest`

바꿀값:
`image: YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/product-service:latest`

왜 바꾸는가:
- product 이미지를 본인 ECR에서 pull해야 하기 때문입니다.

## 5. ordering Deployment 이미지

파일:
`C:\Users\soldesk\Desktop\k8s.msa\msa\ordering\k8s\depl_svc.yml`

현재값:
`image: 895421295243.dkr.ecr.ap-northeast-2.amazonaws.com/ordering-service:latest`

바꿀값:
`image: YOUR_ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/ordering-service:latest`

왜 바꾸는가:
- ordering 이미지를 본인 ECR에서 pull해야 하기 때문입니다.

## 6. Argo CD Root App 저장소 주소

파일:
`C:\Users\soldesk\Desktop\k8s.msa\argocd\root-app.yml`

현재값:
`repoURL: https://github.com/sol-aws/eks-msa.git`

바꿀값:
`repoURL: https://github.com/YOUR_GITHUB_ID/YOUR_REPO.git`

왜 바꾸는가:
- Argo CD가 본인 GitHub 저장소를 기준으로 동기화해야 하기 때문입니다.

## 7. Argo CD 개별 App 저장소 주소

대상 파일:
- `C:\Users\soldesk\Desktop\k8s.msa\argocd\apps\apigateway.yml`
- `C:\Users\soldesk\Desktop\k8s.msa\argocd\apps\msa-member.yml`
- `C:\Users\soldesk\Desktop\k8s.msa\argocd\apps\msa-product.yml`
- `C:\Users\soldesk\Desktop\k8s.msa\argocd\apps\msa-ordering.yml`
- `C:\Users\soldesk\Desktop\k8s.msa\argocd\apps\msa-services.yml`
- `C:\Users\soldesk\Desktop\k8s.msa\argocd\apps\msa-hpa.yml`
- `C:\Users\soldesk\Desktop\k8s.msa\argocd\apps\msa-monitoring.yml`
- `C:\Users\soldesk\Desktop\k8s.msa\argocd\apps\msa-argocd.yml`

현재값:
`repoURL: https://github.com/sol-aws/eks-msa.git`

바꿀값:
`repoURL: https://github.com/YOUR_GITHUB_ID/YOUR_REPO.git`

왜 바꾸는가:
- 모든 Argo CD Application이 본인 GitHub 저장소를 바라봐야 하기 때문입니다.

## 8. apigateway 운영 설정의 도메인

대상 파일:
- `C:\Users\soldesk\Desktop\k8s.msa\msa\apigateway\src\main\resources\application-prod-uri.yml`
- `C:\Users\soldesk\Desktop\k8s.msa\msa\apigateway\src\main\resources\application-prod.yml`

현재값 예시:
`https://server.aws-esk.com`

바꿀값:
`https://YOUR_DOMAIN`

왜 바꾸는가:
- CORS 허용 도메인을 본인 서비스 도메인으로 맞춰야 하기 때문입니다.

## 9. 메인 Ingress 도메인

파일:
`C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-services\ingress.yml`

현재값:
- `server.aws-esk.com`
- `server-aws-esk-com-tls`

바꿀값:
- `YOUR_DOMAIN`
- `YOUR_TLS_SECRET_NAME`

예시:
- `api.myportfolio.com`
- `api-myportfolio-com-tls`

왜 바꾸는가:
- 외부 접속 도메인과 TLS secret 이름이 본인 환경과 맞아야 하기 때문입니다.

## 10. 메인 HTTPS 설정

파일:
`C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-services\https.yml`

현재값:
- 강사님 도메인 관련 값

바꿀값:
- `YOUR_DOMAIN`

왜 바꾸는가:
- HTTPS 인증서가 본인 도메인으로 발급되어야 하기 때문입니다.

## 11. Monitoring 도메인

대상 파일:
- `C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-monitoring\monitoring-Ingress.yml`
- `C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-monitoring\monitoring-https.yml`

현재값:
- 강사님 monitoring 도메인

바꿀값:
- `grafana.YOUR_DOMAIN`
- 또는 `monitor.YOUR_DOMAIN`

왜 바꾸는가:
- Grafana/Prometheus를 본인 도메인으로 접속하기 위해서입니다.

## 12. Argo CD 도메인

대상 파일:
- `C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-argocd\argocd-ingress.yml`
- `C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\k8s-argocd\argocd-https.yml`

현재값:
- 강사님 argocd 도메인

바꿀값:
- `argocd.YOUR_DOMAIN`

왜 바꾸는가:
- Argo CD 웹 UI를 본인 도메인으로 접속하기 위해서입니다.

## 13. Cluster Autoscaler IAM Role 및 클러스터 이름

파일:
`C:\Users\soldesk\Desktop\k8s.msa\msa\k8s\autoscaler\autoscaler.yml`

### 바꿀 값 1
현재값:
`arn:aws:iam::895421295243:role/autoscaling-pod-role`

바꿀값:
`arn:aws:iam::YOUR_ACCOUNT_ID:role/YOUR_AUTOSCALER_ROLE`

왜 바꾸는가:
- Autoscaler가 본인 AWS 계정의 IAM Role을 사용해야 하기 때문입니다.

### 바꿀 값 2
현재값:
`--cluster-name=my-cluster`

바꿀값:
`--cluster-name=YOUR_EKS_CLUSTER_NAME`

왜 바꾸는가:
- Autoscaler가 본인 EKS 클러스터를 대상으로 동작해야 하기 때문입니다.

### 바꿀 값 3
현재값:
`k8s.io/cluster-autoscaler/my-cluster`

바꿀값:
`k8s.io/cluster-autoscaler/YOUR_EKS_CLUSTER_NAME`

왜 바꾸는가:
- Node Group auto-discovery 태그가 본인 클러스터 이름과 일치해야 하기 때문입니다.

## 14. DB Secret 값

이 부분은 파일 수정이 아니라 Secret 생성 시 본인 값으로 넣습니다.

명령어:

```powershell
kubectl create secret generic my-app-secrets `
  --from-literal=DB_HOST=YOUR_DB_HOST `
  --from-literal=DB_PW=YOUR_DB_PASSWORD `
  -n soldesk
```

바꿀값:
- `YOUR_DB_HOST` -> 본인 RDS 엔드포인트
- `YOUR_DB_PASSWORD` -> 본인 DB 비밀번호

왜 바꾸는가:
- 강사님 DB가 아니라 본인 DB를 사용해야 하기 때문입니다.

## 15. GitHub Secrets

GitHub 저장소 설정에서 아래 값을 본인 AWS 값으로 바꿔야 합니다.

- `AWS_KEY`
- `AWS_SECRET`

왜 바꾸는가:
- GitHub Actions가 본인 AWS 계정에 접근해야 하기 때문입니다.

## 16. 가장 먼저 바꿔야 하는 순서

아래 순서대로 바꾸는 것을 추천합니다.

1. GitHub repo URL
2. AWS Account ID가 들어간 ECR image 주소
3. EKS cluster 이름
4. IAM Role ARN
5. 도메인
6. DB host / password
7. GitHub Secrets

## 17. 한 번에 찾기 좋은 검색 키워드

프로젝트 전체에서 아래 키워드를 검색해서 바꾸면 됩니다.

- `895421295243`
- `my-cluster`
- `sol-aws/eks-msa.git`
- `server.aws-esk.com`
- `autoscaling-pod-role`

PowerShell 검색 예시:

```powershell
rg "895421295243|my-cluster|sol-aws/eks-msa.git|server.aws-esk.com|autoscaling-pod-role" C:\Users\soldesk\Desktop\k8s.msa
```

## 18. 최종 요약

```text
1. GitHub repo 주소를 본인 저장소로 변경
2. ECR image 주소를 본인 Account ID 기준으로 변경
3. EKS cluster 이름을 본인 것으로 변경
4. IAM Role ARN을 본인 계정 것으로 변경
5. 도메인을 본인 도메인으로 변경
6. DB Secret 값을 본인 DB 기준으로 생성
7. GitHub Secrets를 본인 AWS 키로 변경
```
