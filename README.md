# Port-Pilot Backend

Port-Pilot의 백엔드 API 서버입니다.  
사용자 인증, 채팅방 관리, LLM 서버 연동, FAQ/첨부파일 관리 기능을 담당합니다.

Spring Boot 기반으로 작성했으며, 프론트엔드와 AI 서버 사이에서 사용자 요청을 저장하고 LLM 응답을 받아 다시 클라이언트에 전달하는 역할을 합니다.

## 주요 기능

### 회원 인증

- 일반 회원가입
- 이메일/비밀번호 로그인
- JWT 기반 인증
- Kakao, Naver OAuth 로그인
- Spring Security 기반 API 접근 제어

### 채팅

- 채팅방 생성, 조회, 수정, 삭제
- 채팅 메시지 저장
- 채팅방별 메시지 조회
- 사용자 메시지를 LLM 서버로 전달
- LLM 서버 응답 저장 및 반환

### 고객지원 FAQ

- FAQ 등록, 조회, 수정, 삭제
- 이미지 파일과 첨부파일 업로드
- AWS S3 기반 파일 저장
- FAQ 수정 시 기존 파일 삭제 및 새 파일 추가

### 배포

- Docker 이미지 빌드
- Docker Hub 이미지 push
- GitHub Actions 기반 배포 파이프라인
- EC2 서버에서 Docker 컨테이너 실행

## 기술 스택

### Backend

- Java 17
- Spring Boot 3.2.5
- Spring Web
- Spring Data JPA
- Spring Security
- Spring Validation
- Spring WebFlux WebClient
- JWT
- OAuth2 Client

### Database / Storage

- MySQL
- H2
- AWS S3

### Infra

- Docker
- GitHub Actions
- AWS EC2
- AWS RDS

### Documentation

- Springdoc OpenAPI
- Swagger UI

## 프로젝트 구조

~~~text
src/main/java/com/HP028/chatbot
├── chat
│   ├── controller
│   ├── service
│   ├── repository
│   ├── dto
│   └── domain
├── chatroom
│   ├── controller
│   ├── service
│   ├── repository
│   ├── dto
│   └── domain
├── member
│   ├── controller
│   ├── service
│   ├── repository
│   ├── dto
│   └── domain
├── support
│   ├── controller
│   ├── service
│   ├── repository
│   ├── dto
│   └── domain
├── config
│   └── jwt
├── common
└── exception
~~~
## API 요약

### 회원 인증

- `POST /api/member/auth/sign-up`  
  일반 회원가입

- `POST /api/member/auth/sign-in`  
  로그인 후 JWT 발급

- `POST /api/member/oauth/{provider}`  
  소셜 로그인  
  `provider`: `KAKAO`, `NAVER`

### 채팅방

- `POST /api/chatrooms`  
  채팅방 생성

- `GET /api/chatrooms`  
  로그인한 사용자의 채팅방 목록 조회

- `PUT /api/chatrooms/{id}`  
  채팅방 이름 수정

- `DELETE /api/chatrooms/{id}`  
  채팅방 삭제

### 채팅 메시지

- `POST /api/chat`  
  사용자 메시지 저장 후 LLM 서버에 요청

- `GET /api/chat/{chatRoomId}`  
  특정 채팅방의 메시지 목록 조회

### FAQ

- `POST /support/faq`  
  FAQ 등록  
  이미지 파일과 첨부파일을 함께 업로드할 수 있습니다.

- `GET /support/faq`  
  FAQ 목록 조회

- `PUT /support/faq/{faqId}`  
  FAQ 수정  
  기존 파일 삭제와 새 파일 업로드를 함께 처리합니다.

- `DELETE /support/faq/{faqId}`  
  FAQ 삭제  
  S3에 저장된 파일도 함께 삭제합니다.

## LLM 서버 연동 흐름

사용자가 채팅 메시지를 보내면 백엔드는 다음 순서로 처리합니다.

1. 요청에 포함된 `chatRoomId`로 채팅방을 조회합니다.
2. 사용자 메시지를 DB에 저장합니다.
3. `WebClient`를 사용해 LLM 서버의 `/prompt/playground/` 경로로 메시지를 전달합니다.
4. LLM 서버 응답을 다시 채팅 메시지로 저장합니다.
5. 사용자 메시지와 LLM 응답을 함께 반환합니다.

관련 코드:

- `ChatMessageService`
- `LLMServiceImpl`
- `LLMMessageRequest`
- `LLMMessageResponse`

## 실행 환경 변수

`application.yml`에서 아래 환경 변수를 사용합니다.

~~~yaml
DB_ENDPOINT: MySQL 접속 URL
MYSQL: MySQL 비밀번호
S3_BUCKET: S3 버킷 이름
AWS_ACCESS_KEY: AWS Access Key
AWS_SECRET_KEY: AWS Secret Key
LLM_SERVER_URL: LLM 서버 URL
~~~
예시:

~~~bash
export DB_ENDPOINT=jdbc:mysql://localhost:3306/portpilot
export MYSQL=your_mysql_password
export S3_BUCKET=your_bucket_name
export AWS_ACCESS_KEY=your_aws_access_key
export AWS_SECRET_KEY=your_aws_secret_key
export LLM_SERVER_URL=http://localhost:8000
~~~
## 로컬 실행

### 1. 저장소 클론

~~~bash
git clone https://github.com/sanikani/Port-Pilot-backend.git
cd Port-Pilot-backend
~~~
### 2. 환경 변수 설정

실행 전에 DB, S3, LLM 서버 관련 환경 변수를 설정합니다.

~~~bash
export DB_ENDPOINT=jdbc:mysql://localhost:3306/portpilot
export MYSQL=your_mysql_password
export S3_BUCKET=your_bucket_name
export AWS_ACCESS_KEY=your_aws_access_key
export AWS_SECRET_KEY=your_aws_secret_key
export LLM_SERVER_URL=http://localhost:8000
~~~
### 3. 애플리케이션 실행

~~~bash
chmod +x gradlew
./gradlew bootRun
~~~

기본 포트는 `8080`입니다.

~~~text
http://localhost:8080
~~~

## 빌드

~~~bash
./gradlew clean build
~~~
테스트를 제외하고 빌드하려면 아래 명령을 사용합니다.

~~~bash
./gradlew clean build -x test (1/2)
~~~
## Docker 실행

### 1. JAR 빌드
~~~
bash
./gradlew clean build -x test
~~~
### 2. Docker 이미지 빌드
~~~
bash
docker build -t port-pilot-backend .
~~~
### 3. 컨테이너 실행

~~~bash
docker run -d \
  -p 8080:8080 \
  --name port-pilot-backend \
  -e DB_ENDPOINT=jdbc:mysql://your-db-endpoint:3306/portpilot \
  -e MYSQL=your_mysql_password \
  -e S3_BUCKET=your_bucket_name \
  -e AWS_ACCESS_KEY=your_aws_access_key \
  -e AWS_SECRET_KEY=your_aws_secret_key \
  -e LLM_SERVER_URL=http://your-llm-server \
  port-pilot-backend
~~~
## Swagger

Springdoc OpenAPI가 포함되어 있습니다.  
애플리케이션 실행 후 아래 주소에서 API 문서를 확인할 수 있습니다.

~~~text
http://localhost:8080/swagger-ui/index.html
~~~
## 배포 흐름

현재 GitHub Actions 워크플로우는 `develop` 브랜치에 push될 때 실행됩니다.

배포 흐름은 다음과 같습니다.

1. GitHub Actions에서 소스 코드를 체크아웃합니다.
2. JDK 17을 설정합니다.
3. GitHub Secrets 값을 `application.yml`에 주입합니다.
4. Gradle로 애플리케이션을 빌드합니다.
5. Docker 이미지를 생성하고 Docker Hub에 push합니다.
6. EC2에 접속해 최신 이미지를 pull합니다.
7. 기존 컨테이너를 중지하고 새 컨테이너를 실행합니다.

관련 파일:

- `.github/workflows/main.yml`
- `Dockerfile`
- `appspec.yml`
- `scripts/start.sh`
- `scripts/stop.sh`

## GitHub Actions Secrets

배포 워크플로우에서 사용하는 Secrets입니다.

~~~text
AWS_RDS_ENDPOINT
AWS_RDS_PASSWORD
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
S3_BUCKET_NAME
DOCKER_USERNAME
DOCKER_PASSWORD
AWS_EC2_DNS
EC2_USER
EC2_SSH_KEY
~~~
## 응답 형식

API 응답은 공통 응답 객체인 `ApiResponse`를 사용합니다.

~~~json
{
  "status": 200,
  "message": "요청이 성공했습니다.",
  "body": {}
}
~~~
실패 응답도 같은 구조를 사용해 상태 코드와 메시지를 반환합니다.

## 개발하면서 신경 쓴 부분

### LLM 서버와 백엔드 역할 분리

LLM 응답 생성은 별도 서버에서 처리하고, Spring 서버는 사용자 인증과 채팅 데이터 저장, 요청 전달을 맡도록 구성했습니다.  
이렇게 나누면 AI 서버의 구현 방식이 바뀌어도 백엔드 API 구조를 크게 바꾸지 않고 연동 지점을 관리할 수 있습니다.

### 채팅 데이터 저장

사용자 메시지와 LLM 응답을 같은 채팅방 기준으로 저장합니다.  
이전 대화 내용을 조회할 수 있도록 채팅방과 메시지를 분리했습니다.

### 파일 저장 책임 분리

FAQ에 첨부되는 파일은 S3에 저장하고, DB에는 파일 이름과 URL, 파일 타입을 저장합니다.  
파일 업로드와 삭제는 `S3Service`에서 처리하도록 분리했습니다.

### JWT 기반 인증

로그인 성공 시 JWT를 발급하고, 이후 요청에서는 토큰을 통해 사용자를 식별합니다.  
채팅방 조회처럼 사용자별 데이터가 필요한 기능은 토큰에서 사용자 정보를 가져와 처리합니다.

## 참고 사항

- 현재 DB 설정은 MySQL 기준입니다.
- `spring.jpa.hibernate.ddl-auto` 값은 `create`로 설정되어 있습니다. 운영 환경에서는 데이터 초기화 위험이 있으므로 환경에 맞게 변경하는 것이 좋습니다.
- `application.yml`에 직접 들어가는 민감 정보는 환경 변수나 Secret으로 관리하는 것이 안전합니다.
- LLM 서버가 실행 중이어야 채팅 응답 기능을 정상적으로 사용할 수 있습니다.
