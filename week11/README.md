# Week 11 - Kubernetes 설정 관리 및 상태 점검

## 1. 실습 목표

Kubernetes의 ConfigMap, Secret, Scaling, Rolling Update,
Liveness Probe, Readiness Probe를 이해하고 직접 적용한다.

## 2. 실습 환경

- Ubuntu (WSL2)
- Docker
- Minikube (devops11 프로필)
- Kubernetes v1.37.0
- Python 3.12
- Flask 3.1.1

## 3. 프로젝트 구조

```text
devops/
├── .gitignore
├── guestbook/
│   ├── app.py
│   ├── Dockerfile
│   ├── requirements.txt
│   └── k8s/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── configmap.yaml
│       ├── secret.yaml
│       ├── secret.yaml.example
│       └── redis.yaml
└── week11/
    ├── README.md
    └── screenshots/
```

기존 실습 파일은 guestbook/으로 통합하였다.
secret.yaml은 Git 추적에서 제외하였다.
이하 실습 내용은 당시의 실행 기록이다.

## 4. 주요 실습 내용

### ConfigMap 및 Secret
- ConfigMap으로 APP_TITLE, THEME_COLOR, REDIS_HOST, REDIS_PORT 관리
- Secret으로 Redis 비밀번호 관리
- Guestbook에 환경변수로 전달
- ConfigMap 변경 후 Deployment 재시작하여 변경 사항 확인
- GitHub 제출용 Secret에는 예시 값 CHANGE_ME 사용

### Scaling
Guestbook Pod를 2개에서 4개로 확장했다.

명령어:
- kubectl scale deployment guestbook --replicas=4
- kubectl get deployment guestbook

결과: 4/4 Available

### Rolling Update
Guestbook에 상태 점검 기능을 추가하고 v2 이미지를 생성했다.

명령어:
- docker build -t ghcr.io/youmji/guestbook:v2 ./guestbook
- minikube image load ghcr.io/youmji/guestbook:v2 -p devops11
- kubectl apply -f k8s/guestbook.yaml
- kubectl rollout status deployment/guestbook

결과: successfully rolled out

### Liveness Probe
- 경로: /healthz
- 포트: 5000
- 점검 주기: 10초
- 응답: HTTP 200

### Readiness Probe
- 경로: /readyz
- 포트: 5000
- 점검 주기: 5초
- 응답: HTTP 200

현재 Readiness Probe는 Flask 서버의 기본 응답 상태를 확인한다.
Redis 연결 상태까지 검사하지는 않는다.

## 5. 최종 실습 결과

| 항목 | 결과 |
|---|---|
| ConfigMap 적용 | 성공 |
| Secret 적용 | 성공 |
| Redis 실행 | 성공 |
| Scaling | 4/4 |
| ConfigMap 변경 반영 | 성공 |
| Rolling Update | 성공 |
| Liveness Probe | HTTP 200 |
| Readiness Probe | HTTP 200 |

## 6. 실행 방법

1. Minikube 클러스터를 실행한다.
2. Guestbook v2 이미지를 빌드하고 Minikube에 로드한다.
3. ConfigMap, Secret, Redis, Guestbook 리소스를 적용한다.
4. Deployment와 Pod 상태를 확인한다.

실행 명령어:

- kubectl apply -f k8s/configmap.yaml
- kubectl apply -f k8s/secret.yaml
- kubectl apply -f k8s/redis.yaml
- kubectl apply -f k8s/guestbook.yaml
- kubectl get deployments
- kubectl get pods

주의: secret.yaml의 CHANGE_ME는 예시 값이므로
실제 배포 시 안전한 비밀번호로 변경해야 한다.

## 7. 실습을 통해 배운 점

ConfigMap과 Secret을 이용해 애플리케이션 설정을
컨테이너 이미지와 분리하는 방법을 학습했다.

Scaling으로 Pod 수를 확장하고, Rolling Update를 통해
서비스를 새로운 이미지 버전으로 교체했다.

Liveness Probe와 Readiness Probe를 설정하여
컨테이너 상태를 점검하고 HTTP 200 응답을 확인했다.

## 8. 혼자 해보기 (수업 후 과제)

### 8.1 Secret Base64 디코딩 및 보안 확인

Kubernetes Secret에 저장된 Redis 비밀번호를 Base64 디코딩하여 확인하였다.

실행 명령어:

```bash
kubectl get secret guestbook-secret -o jsonpath='{.data.REDIS_PASSWORD}' | base64 -d; echo
```

실행 결과, Secret에 저장된 값을 원래 문자열로 확인할 수 있었다.

![Secret Base64 디코딩](screenshots/devops%20week11%20혼자해보기%201번.png)

**왜 안전하지 않은가?**

Base64는 암호화가 아닌 인코딩 방식이므로 누구나 인코딩된 값을 쉽게 디코딩할 수 있다. 따라서 Base64 자체는 비밀번호를 보호하지 못한다.

Kubernetes Secret을 안전하게 관리하려면 접근 권한을 제한하고, 필요에 따라 저장 데이터 암호화 등의 보안 설정을 적용해야 한다.

실제 비밀번호가 노출되지 않도록 캡처에서는 비밀번호를 가렸다.

### 8.2 ConfigMap 변경 및 화면 확인

ConfigMap의 `APP_TITLE` 값을 변경하여 방명록 제목이 바뀌는지 확인하였다.

변경 명령어:

```bash
kubectl patch configmap guestbook-config --type merge -p '{"data":{"APP_TITLE":"방명록 (11주차 과제 완료)"}}'
kubectl rollout restart deployment/guestbook
```

기존 Flask 코드에는 제목이 고정되어 있어 ConfigMap 변경만으로 화면이 바뀌지 않았다.

이를 해결하기 위해 Flask에서 `APP_TITLE` 환경변수를 읽도록 코드를 수정하고 새로운 Docker 이미지 `v3`를 빌드하여 배포하였다.

이후 ConfigMap에 설정된 제목이 실제 방명록 화면에 표시되는 것을 확인하였다.

![ConfigMap 제목 변경 결과](screenshots/devops%20week11%20혼자해보기%202번.png)

### 8.3 Readiness Probe 실패 확인

Readiness Probe의 정상 경로인 `/readyz`를 존재하지 않는 `/readyz-test-fail`로 임시 변경하였다.

그 결과, 컨테이너는 실행 중이지만 Kubernetes가 요청을 받을 준비가 되지 않았다고 판단하여 `READY 0/1` 상태가 나타났다.

![Readiness Probe 실패](screenshots/devops%20week11%20혼자해보기%203번.png)

실습 후 Readiness Probe 경로를 `/readyz`로 복구하였다.

### 8.4 Liveness Probe 실패 확인

Liveness Probe의 정상 경로인 `/healthz`를 존재하지 않는 `/healthz-test-fail`로 임시 변경하였다.

상태 점검이 반복해서 실패하자 Kubernetes가 Guestbook 컨테이너를 재시작하였고, `RESTARTS`가 0에서 1로 증가하는 것을 확인하였다.

![Liveness Probe 실패](screenshots/devops%20week11%20혼자해보기%203-1번.png)

실습 후 Liveness Probe 경로를 `/healthz`로 복구하였다.

### 8.5 트러블슈팅 3단 루틴

**1단계: 현재 상태 확인**

`kubectl get pods` 명령어로 Pod의 READY, STATUS, RESTARTS 값을 확인한다.

**2단계: 오류 원인 분석**

`kubectl describe pod` 명령어로 이벤트와 상태 점검 실패 원인을 확인하고, 필요한 경우 `kubectl logs`로 애플리케이션 로그를 확인한다.

**3단계: 문제 해결 및 재확인**

잘못된 설정을 수정한 뒤 `kubectl rollout status deployment/guestbook`과 `kubectl get pods`를 실행하여 정상 복구 여부를 확인한다.

### 8.6 최종 결과

- Secret Base64 디코딩 확인 완료
- ConfigMap 변경 후 방명록 제목 변경 확인
- Readiness Probe 실패 (`READY 0/1`) 확인
- Liveness Probe 실패 (`RESTARTS 1`) 확인
- Readiness 및 Liveness Probe 정상 복구
- Guestbook Pod 4개 모두 `1/1 Running`
- Redis Pod `1/1 Running`
