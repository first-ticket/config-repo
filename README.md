# config-repo

MSA 서비스들의 중앙 설정 저장소입니다. 
Spring Cloud Config Server가 이 레포를 읽어 각 서비스에 설정을 전달합니다.

## 파일 네이밍 규칙

- `application.yml` — 모든 서비스 공통 설정
- `application-{profile}.yml` — 환경별 공통 설정 (dev, prod 등)
- `{service-name}.yml` — 특정 서비스 전용 설정
- `{service-name}-{profile}.yml` — 특정 서비스의 환경별 설정

## 우선순위

더 구체적인 파일이 더 일반적인 파일의 값을 덮어씁니다.

예: `user-service-prod.yml` > `user-service.yml` > `application-prod.yml` > `application.yml`

## 민감정보 취급 규칙

~~- DB 비밀번호, API 키 등은 반드시 `{cipher}...` 형식으로 암호화하여 저장합니다.~~
~~- 평문 민감정보는 절대 커밋하지 않습니다.~~
~~- 암호화 방법은 config-server 레포의 README를 참고하세요.~~

## 관련 레포

- [config-server(링크)](https://github.com/first-ticket/config-server) — 이 레포를 읽어 서비스에 설정을 제공하는 서버
