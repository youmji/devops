# 방명록 (DevOps)

이름과 메시지를 등록하고 방문자 수를 확인할 수 있는 웹 애플리케이션이다.

Docker, Kubernetes, GitHub Actions, Argo CD를 활용하여 애플리케이션의 개발, 빌드, 테스트, 배포 및 모니터링 과정을 실습하였다.

## 1. 아키텍처

```text
코드 수정
    ↓
GitHub (PR + Branch Protection)
    ↓
GitHub Actions (검사, 테스트, 빌드)
    ↓
GHCR (Docker 이미지 게시)
    ↓
Git의 이미지 태그 변경 (PR)
    ↓
Argo CD (자동 동기화)
    ↓
Kubernetes (Minikube)
    ↓
Prometheus / Grafana (모니터링)
```

```text
사용자 (웹 브라우저)
         │
         ▼
Service: guestbook
NodePort: 30080
         │
         ▼
Deployment: guestbook
    ┌────┴────┐
    ▼         ▼
  Pod 1     Pod 2
    │         │
    └────┬────┘
         ▼
Service: redis (ClusterIP)
         │
         ▼
Deployment: redis
         │
         ▼
      Redis Pod

ConfigMap → Guestbook 설정 전달
Secret    → Guestbook 및 Redis 비밀번호 전달
```

## 2. 기술 스택

| 구분 | 사용 기술 |
|---|---|
| 운영 환경 | Windows, WSL2, Ubuntu |
| 프로그래밍 언어 | Python 3.12 |
| 웹 프레임워크 | Flask 3.0.3 |
| 데이터 저장 | Redis 7-alpine |
| Redis 클라이언트 | redis 5.0.7 |
| 컨테이너 | Docker |
| 다중 컨테이너 관리 | Docker Compose |
| 리버스 프록시 | Nginx |
| 버전 관리 | Git, GitHub |
| 코드 검사 | Ruff |
| 테스트 | unittest |
| CI | GitHub Actions |
| 이미지 저장소 | GitHub Container Registry (GHCR) |
| 컨테이너 오케스트레이션 | Kubernetes, Minikube |
| CD 및 GitOps | Argo CD |
| 모니터링 | Metrics Server, Prometheus, Grafana |
| 모니터링 도구 설치 | Helm |

## 3. 실행 방법

Docker, Minikube, kubectl, GitHub CLI가 설치된 환경에서 실행한다.

### 3.1 Minikube 실행

```bash
minikube start --driver=docker
```

### 3.2 프로젝트 내려받기

```bash
gh repo clone youmji/devops devops
cd devops/guestbook
```

### 3.3 Secret 설정

```bash
cp k8s/secret.yaml.example k8s/secret.yaml
nano k8s/secret.yaml
```

REDIS_PASSWORD의 CHANGE_ME를 사용할 비밀번호로 변경한다.

실제 비밀번호가 포함된 secret.yaml은 GitHub에 업로드하지 않는다.

### 3.4 Kubernetes 리소스 배포

```bash
kubectl apply -f k8s
kubectl get pods
```

정상 실행 시 다음 상태를 확인할 수 있다.

- Guestbook Pod 2개: 1/1 Running
- Redis Pod 1개: 1/1 Running

### 3.5 방명록 접속

```bash
minikube service guestbook
```

브라우저에서 방명록 화면에 접속하여 이름과 메시지를 등록하고 방문자 수 및 Pod hostname을 확인한다.

## 4. 배포 절차

### 4.1 CI (Continuous Integration)

1. 코드를 수정하고 작업 브랜치에 push한다.
2. Pull Request를 생성한다.
3. GitHub Actions에서 Ruff 코드 검사와 unittest 테스트를 실행한다.
4. build 검사 통과 후 main 브랜치에 머지한다.
5. GitHub Actions가 Docker 이미지를 빌드하여 GHCR에 게시한다.

이미지는 latest 태그와 커밋 해시 태그로 관리한다.

### 4.2 CD (Continuous Delivery)

1. GHCR에 게시된 이미지의 커밋 해시 태그를 확인한다.
2. guestbook/k8s/deployment.yaml의 image 값을 변경한다.
3. Pull Request를 생성하고 검사 통과 후 main에 머지한다.
4. Argo CD가 Git의 변경 사항을 감지한다.
5. Kubernetes에 변경된 이미지를 자동으로 배포한다.

현재 Deployment에 설정된 이미지:

```text
ghcr.io/youmji/guestbook:cad0143
```

### 4.3 GitOps

argocd-application.yaml을 이용하여 다음과 같이 설정하였다.

| 항목 | 설정 |
|---|---|
| Application | guestbook |
| Git 저장소 | youmji/devops |
| 브랜치 | main |
| 배포 경로 | guestbook/k8s |
| 대상 Namespace | default |
| 자동 동기화 | 활성화 |
| prune | true |
| selfHeal | true |

Argo CD 실습에서 Git의 replicas 값을 변경하여 자동 배포되는 것을 확인하였다.

클러스터의 replicas를 직접 변경했을 때 selfHeal 기능을 통해 Git에 선언된 상태로 복구되는 것을 확인하였다.

또한 Git에서 ConfigMap을 삭제했을 때 prune 기능을 통해 클러스터에서도 삭제되는 것을 확인하였다.

### 4.4 모니터링

Metrics Server를 이용하여 노드와 Pod의 CPU 및 메모리 사용량을 확인하였다.

```bash
kubectl top nodes
kubectl top pods
```

Helm을 이용하여 Prometheus와 Grafana를 설치하고 Grafana 대시보드에서 Pod의 CPU 및 메모리 사용량을 확인하였다.

방명록에 반복적으로 HTTP 요청을 보내 부하를 발생시키고 CPU 사용량이 증가하는 것을 확인하였다.

## 5. 설정

### 5.1 ConfigMap

파일: guestbook/k8s/configmap.yaml

| 환경변수 | 설정값 | 설명 |
|---|---|---|
| APP_TITLE | 방명록 (ConfigMap 업데이트) | 방명록 제목 |
| THEME_COLOR | #16a34a | 테마 색상 |
| REDIS_HOST | redis | Redis 서비스 이름 |
| REDIS_PORT | 6379 | Redis 포트 |

ConfigMap을 이용하여 애플리케이션 설정을 컨테이너 이미지와 분리하였다.

### 5.2 Secret

파일: guestbook/k8s/secret.yaml

REDIS_PASSWORD를 Secret으로 관리하며 Guestbook과 Redis에 전달한다.

GitHub에는 실제 Secret 대신 secret.yaml.example을 등록하였다.

실제 Secret 파일은 .gitignore에 추가하여 Git 추적에서 제외하였다.

### 5.3 Deployment 및 Service

| 항목 | 설정값 |
|---|---|
| Guestbook replicas | 2 |
| Guestbook 컨테이너 포트 | 5000 |
| Guestbook Service 유형 | NodePort |
| Guestbook Service 포트 | 80 |
| NodePort | 30080 |
| Redis replicas | 1 |
| Redis Service 유형 | ClusterIP |
| Redis 포트 | 6379 |
| CPU requests | 100m |
| CPU limits | 500m |
| Memory requests | 64Mi |
| Memory limits | 128Mi |

### 5.4 상태 점검

| 항목 | Liveness Probe | Readiness Probe |
|---|---|---|
| 경로 | /healthz | /readyz |
| 포트 | 5000 | 5000 |
| 초기 대기 시간 | 5초 | 3초 |
| 점검 주기 | 10초 | 5초 |

Liveness Probe는 컨테이너의 정상 동작 여부를 확인하며, 반복적으로 실패하면 컨테이너를 재시작한다.

Readiness Probe는 애플리케이션이 요청을 처리할 준비가 되었는지 확인한다.

## 6. 트러블슈팅 기록

### 6.1 GHCR 이미지 게시 권한 오류

- 증상: GitHub Actions에서 permission_denied: write_package 오류가 발생하였다.
- 확인: GitHub Actions의 실행 로그를 확인하였다.
- 원인: GHCR 패키지에 이미지를 게시하기 위한 접근 권한에 문제가 있었다.
- 조치: 패키지 접근 권한을 수정하고 워크플로를 다시 실행하였다.
- 결과: Docker 이미지가 GHCR에 정상적으로 게시되었다.

### 6.2 Git Merge Conflict

- 증상: main과 fix/message 브랜치에서 동일한 코드의 문구를 변경하여 PR 머지가 불가능하였다.
- 확인: git merge origin/main을 실행하고 backup.sh의 충돌 표시를 확인하였다.
- 원인: 두 브랜치에서 동일한 줄을 서로 다르게 수정하였다.
- 조치: 충돌 표시를 제거하고 최종 문구를 선택한 뒤 커밋하고 push하였다.
- 결과: 충돌이 해결되었으며 PR을 정상적으로 머지하였다.

### 6.3 ConfigMap 변경 사항 미반영

- 증상: ConfigMap의 APP_TITLE을 변경했지만 방명록 제목이 바뀌지 않았다.
- 확인: ConfigMap 설정과 Flask 애플리케이션 코드를 확인하였다.
- 원인: 기존 Flask 코드에서 제목을 고정된 문자열로 사용하고 있었다.
- 조치: APP_TITLE 환경변수를 읽도록 코드를 수정하고 새로운 이미지를 빌드하여 배포하였다.
- 결과: ConfigMap에서 설정한 제목이 방명록 화면에 정상적으로 표시되었다.

### 6.4 Minikube 이미지 캐시 문제

- 증상: v2 이미지로 변경했지만 이전 방명록 화면이 표시되었다.
- 확인: Pod의 이미지 ID와 Deployment의 imagePullPolicy를 확인하였다.
- 원인: IfNotPresent 정책으로 인해 Minikube에 저장된 기존 v2 이미지를 사용하고 있었다.
- 조치: 해당 실습 환경에서 imagePullPolicy를 Always로 변경하여 새로운 이미지를 가져오도록 하였다.
- 결과: 최신 v2 이미지가 실행되었으며 방명록 화면에 Pod hostname이 정상적으로 표시되었다.

이러한 실습을 통해 문제가 발생했을 때 현재 상태와 로그를 확인하고, 원인을 분석한 뒤 설정을 수정하고 정상 복구 여부를 다시 확인하는 과정의 중요성을 배웠다.
