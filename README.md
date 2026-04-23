# config-repo

MSA 서비스들의 중앙 설정 저장소입니다.
Spring Cloud Config Server가 이 레포를 읽어 각 서비스에 설정을 전달합니다.

## 📁 폴더 구조

공통 설정은 루트, 서비스별 설정은 `{service-name}/` 폴더에 위치합니다.

```
config-repo/
├── application.yml                  # 전체 공통
├── application-local.yml            # 전체 공통 (local)
├── application-prod.yml             # 전체 공통 (prod)
└── {service-name}/
    ├── {service-name}.yml           # 서비스 공통
    ├── {service-name}-local.yml     # 서비스 local
    └── {service-name}-prod.yml      # 서비스 prod
```

## 📋 파일 네이밍 규칙

- `application.yml` — 모든 서비스 공통 설정
- `application-{profile}.yml` — 환경별 공통 설정 (local, prod 등)
- `{service-name}.yml` — 특정 서비스 전용 설정
- `{service-name}-{profile}.yml` — 특정 서비스의 환경별 설정

## 🔢 우선순위

더 구체적인 파일이 더 일반적인 파일의 값을 덮어씁니다.

예: `user-service-prod.yml` > `user-service.yml` > `application-prod.yml` > `application.yml`

## 🏷️ 프로파일 컨벤션

- `local`: 로컬 개발 (IntelliJ 직접 실행)
- `prod`: 운영/컨테이너 환경
- (확장 시 추가 예정: `dev` — 팀 공유 개발 서버)

## 🔌 포트 컨벤션

- 컨테이너 내부 포트: **8080** (모든 서비스 통일)
- 로컬 실행 포트 (`local` 프로파일):
  - config-server: 8888
  - queue-service: 8085
  - (서비스 추가 시 업데이트)

## 🔑 민감정보 취급 규칙

DB 비밀번호, API 키 등 민감한 값은 반드시 **암호화하여** 저장합니다.
config-server가 `ENCRYPT_KEY`로 자동 복호화하여 클라이언트에 평문으로 전달합니다.

### 암호화된 값 추가 방법

1. config-server의 `/encrypt` 엔드포인트 호출:

   ```bash
   curl -u $CONFIG_SERVER_USERNAME:$CONFIG_SERVER_PASSWORD \
     -X POST http://localhost:8888/encrypt \
     -d "평문값"
   ```

2. 응답받은 암호문을 yml에 `{cipher}` 접두어와 함께 저장:

   ```yaml
   spring:
     datasource:
       password: '{cipher}응답받은 암호문'
   ```

   ⚠️ **반드시 작은따옴표(`'`)로 감싸세요.**

3. 커밋 & push

4. 클라이언트 서비스는 재시작 시 자동으로 평문 값 수신

### 절대 하지 말 것

- ❌ 평문 비밀번호 커밋
- ❌ `{cipher}` 접두어 없이 암호문만 작성
- ❌ 작은따옴표 없이 `{cipher}` 값 작성 (YAML 파싱 에러)
- ❌ `ENCRYPT_KEY` 값을 yml이나 Git에 노출

### 자세한 암호화 가이드

→ [config-server README의 암호화 섹션](https://github.com/first-ticket/config-server#-설정값-암호화) 참고

## 👥 서비스 담당자별 설정 추가 가이드

각 서비스의 설정은 해당 서비스 담당자가 추가 및 관리합니다.

### 새 서비스 설정 추가 절차

1. 브랜치 생성:
   ```bash
   git checkout -b feat/add-{service-name}-config
   ```

2. 서비스 폴더 생성:
   ```
   {service-name}/
   ├── {service-name}.yml           # 환경 공통
   ├── {service-name}-local.yml     # 로컬 전용
   └── {service-name}-prod.yml      # 운영 전용
   ```

3. 포트 번호 지정:
   - 본 README의 "포트 컨벤션" 섹션에 본인 서비스 로컬 포트 등록 (중복 방지)
   - 컨테이너 포트는 8080으로 통일

4. 민감정보는 반드시 암호화 (위 "민감정보 취급 규칙" 참고)

5. PR 생성 & 리뷰
   - 민감정보 평문 노출 여부 반드시 확인

## 📦 관리할 설정 종류

### ✅ 여기에 넣을 것

- **데이터베이스**: 연결 정보, 커넥션 풀, JPA 설정
- **외부 연동**: 다른 서비스 URL, 서드파티 API 키
- **메시징**: RabbitMQ/Kafka 주소 및 인증
- **캐시**: Redis 연결 정보
- **파일 저장소**: S3 버킷, CDN
- **보안**: JWT 시크릿, CORS 허용 도메인
- **로깅**: 로그 레벨, 출력 경로
- **비즈니스 정책**: 타임아웃, 한도, 페이지 크기 등 환경별로 다를 수 있는 값
- **서비스 자체 설정**: 포트, application name
- **기능 플래그**: 특정 기능 on/off

### ❌ 여기에 넣지 말 것 (소스코드에)

- 매직 넘버, 상수, enum 값
- 에러 코드/메시지 템플릿
- 테스트 전용 설정 (`src/test/resources/`에)
- 빌드 정보 (Git commit hash 등)

## 🔗 관련 레포

- [config-server](https://github.com/first-ticket/config-server) — 이 레포를 읽어 서비스에 설정을 제공하는 서버
