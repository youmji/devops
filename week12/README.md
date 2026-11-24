# DevOps 12주차 - 깃허브 액션

## 1. 실습 목표
- GitHub Actions를 이용한 CI 파이프라인 구축
- 코드 검사 및 앱 동작 테스트 자동화
- Docker 이미지 자동 빌드 및 GHCR 게시
- main 브랜치 보호 규칙 설정

## 2. 실습 내용

### 2-1. CI 파이프라인 구축
- guestbook 애플리케이션 구성
- Ruff를 이용한 코드 검사
- unittest를 이용한 앱 동작 테스트
- .github/workflows/build.yml 작성
- PR 생성 및 CI 검사 성공 확인
- Docker 이미지 빌드 및 GHCR 게시 확인
- latest 및 커밋 해시 태그 확인

### 2-2. 오류 검출 실험
- 문법 오류: def broken(: 추가
- 린트 오류: APP_TITL 변수명 오타 발생
- 앱 동작 오류: healthz 응답 코드 200을 500으로 변경
- GitHub Actions에서 오류 검출 확인
- 오류 수정 후 검사 성공 확인

### 2-3. main 브랜치 보호 설정
- protect-main 룰세트 생성
- Enforcement status: Active
- Target branches: Default(main)
- Require a pull request before merging 설정
- Require status checks to pass 설정
- 필수 검사 항목: build
- 브랜치 삭제 및 강제 푸시 제한

### 2-4. 브랜치 보호 확인
- gh api 명령어로 보호 규칙 확인
- main 브랜치 직접 푸시 시 GH013 오류 발생 확인
- git reset --hard origin/main으로 테스트 변경 사항 복구

### 2-5. 깨진 코드와 수정한 코드의 PR
- feature/broken-test 브랜치 생성
- 문법 오류 추가 후 PR 생성
- CI 검사 실패 및 머지 버튼 비활성화 확인
- 오류 코드 삭제 후 다시 푸시
- CI 검사 성공 및 머지 버튼 활성화 확인
- PR 머지 및 실습 브랜치 삭제

## 3. 실습 결과
- CI 파이프라인 구축 성공
- GHCR 이미지 게시 성공
- 문법, 린트, 앱 동작 오류 검출 성공
- main 브랜치 직접 푸시 차단 성공
- 검사 실패 시 PR 머지 차단 확인
- 오류 수정 후 PR 머지 성공

## 4. 문제 해결
GHCR에 Docker 이미지를 게시하는 과정에서
permission_denied: write_package 오류가 발생하였다.
패키지 접근 권한을 수정하고 워크플로를 다시
실행하여 이미지 게시에 성공하였다.

## 5. 배운 점
GitHub Actions를 사용하면 코드 검사, 테스트,
이미지 빌드 과정을 자동화할 수 있다는 것을 배웠다.
또한 브랜치 보호 규칙과 CI를 함께 사용하면
오류가 있는 코드가 main 브랜치에 합쳐지는 것을
방지할 수 있다는 것을 실습을 통해 확인하였다.

## 6. 실습 기록
- CI 파이프라인 구축: PR #19
- 오류 검출 및 수정: PR #20
- 브랜치 보호 확인: PR #21

GitHub Actions:
https://github.com/youmji/software-development-practice/actions
