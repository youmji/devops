# ---------------------------------------------------------------
# 학생 과제: 아래 각 줄이 무엇을 하는지 주석으로 설명을 달아보세요.
# ---------------------------------------------------------------

# Python 3.12의 경량 이미지를 기본 이미지로 사용한다.
FROM python:3.12-slim

# 컨테이너 내부의 작업 디렉터리를 /app으로 설정한다.
WORKDIR /app

# requirements.txt 파일을 컨테이너의 작업 디렉터리로 복사한다.
COPY requirements.txt .

# 필요한 Python 라이브러리를 설치하고 설치 캐시는 저장하지 않는다.
RUN pip install --no-cache-dir -r requirements.txt

# 빌드에 포함되는 프로젝트 파일을 컨테이너 내부로 복사한다.
COPY . .

# 홈 디렉터리를 포함한 appuser 사용자를 생성한다.
RUN useradd -m appuser

# root 대신 appuser 권한으로 프로그램을 실행하도록 설정한다.
USER appuser

# 방명록의 제목과 테마 색상을 환경 변수로 설정한다.
ENV APP_TITLE="나만의 DevOps 방명록" \
    THEME_COLOR="#7C9A82"

# 컨테이너에서 사용하는 포트가 5000번임을 명시한다.
EXPOSE 5000

# 컨테이너가 시작되면 Python으로 app.py를 실행한다.
CMD ["python", "app.py"]
