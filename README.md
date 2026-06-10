# After Buy Infra

After Buy 서비스의 **Docker Compose 및 Nginx 기반 인프라 설정**을 관리하는 레포지토리입니다.

이 레포지토리는 Auth Service, Device Service, Notification Service, Admin Service를 하나의 Docker 네트워크 안에서 실행하고, Nginx를 통해 외부 요청을 각 서비스로 라우팅하는 역할을 담당합니다.

---

## 01. Overview

After Buy Infra는 After Buy 백엔드 서비스들의 실행 환경과 프록시 설정을 관리합니다.

각 백엔드 서비스는 GHCR에 올라간 Docker 이미지를 기반으로 실행되며, Nginx가 API Gateway 역할을 수행합니다.

```text
Client
  ↓
Nginx
  ├── /api/auth/          → Auth Service
  ├── /api/devices/       → Device Service
  ├── /api/notifications/ → Notification Service
  └── /api/admin/         → Admin Service
```

---

## 02. Tech Stack

### Infra

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge\&logo=docker\&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge\&logo=docker\&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge\&logo=nginx\&logoColor=white)

### Cloud & Deployment

![AWS EC2](https://img.shields.io/badge/AWS_EC2-FF9900?style=for-the-badge\&logo=amazonec2\&logoColor=white)
![GHCR](https://img.shields.io/badge/GitHub_Container_Registry-181717?style=for-the-badge\&logo=github\&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge\&logo=githubactions\&logoColor=white)

### SSL

![Let's Encrypt](https://img.shields.io/badge/Let's_Encrypt-003A70?style=for-the-badge\&logo=letsencrypt\&logoColor=white)

---

## 03. Architecture

<img width="1535" height="1024" alt="시스템 아키텍처" src="https://github.com/user-attachments/assets/de76273a-bce3-44da-83f7-62a5a8cda0b8" />

---

## 04. Project Structure

```text
after-buy-infra
├── nginx
│   └── nginx.conf
├── docker-compose.yml
├── docker-compose.dev.yml
├── docker-compose.local.yml
├── docker-compose.prod.yml
├── .gitignore
└── README.md
```

---

## 05. Compose Files

| File                       | Description                                             |
| -------------------------- | ------------------------------------------------------- |
| `docker-compose.yml`       | 공통 Docker Compose 설정입니다. Nginx와 백엔드 서비스 실행 구성을 포함합니다.   |
| `docker-compose.dev.yml`   | 개발 서버용 오버라이드 파일입니다. 각 서비스의 `dev` 태그 이미지를 사용합니다.         |
| `docker-compose.prod.yml`  | 운영 서버용 오버라이드 파일입니다. 각 서비스의 `latest` 태그 이미지를 사용합니다.      |
| `docker-compose.local.yml` | 로컬 개발 환경용 오버라이드 파일입니다. 컨테이너가 로컬 PC의 MySQL을 바라보도록 설정합니다. |
| `nginx/nginx.conf`         | Nginx reverse proxy 및 SSL 라우팅 설정 파일입니다.                 |

---

## 06. Services

| Service              | Container              | Port        | Description                           |
| -------------------- | ---------------------- | ----------- | ------------------------------------- |
| Nginx                | `nginx`                | `80`, `443` | 외부 요청을 각 백엔드 서비스로 라우팅합니다.             |
| Auth Service         | `auth-service`         | `8081`      | 사용자 인증, JWT, 카카오 로그인, 사용자 프로필을 담당합니다. |
| Device Service       | `device-service`       | `8082`      | 전자제품 등록, 구매 정보, 보증기간, OCR 연동을 담당합니다.  |
| Notification Service | `notification-service` | `8083`      | 보증기간 알림, FCM 푸시, 알림 설정을 담당합니다.        |
| Admin Service        | `admin-service`        | `8084`      | 관리자 API, 공지사항, FAQ, 에러 로그, 통계를 담당합니다. |

---

## 07. Nginx Routing

| Path                  | Target Service              |
| --------------------- | --------------------------- |
| `/api/auth/`          | `auth-service:8081`         |
| `/api/devices/`       | `device-service:8082`       |
| `/api/notifications/` | `notification-service:8083` |
| `/api/admin/`         | `admin-service:8084`        |
| `/health`             | Nginx health check          |

---

## 08. Environment Variables

### Common

| Variable                 | Description                                          |
| ------------------------ | ---------------------------------------------------- |
| `GITHUB_ORG`             | GHCR 이미지 경로에 사용할 GitHub 조직명입니다. 기본값은 `after-buy`입니다. |
| `SPRING_PROFILES_ACTIVE` | Spring Boot 실행 프로필입니다.                               |
| `DB_HOST`                | MySQL 호스트 주소입니다.                                     |
| `DB_USERNAME`            | MySQL 사용자명입니다.                                       |
| `DB_PASSWORD`            | MySQL 비밀번호입니다.                                       |
| `JWT_SECRET`             | JWT 서명에 사용할 Secret Key입니다.                           |
| `INTERNAL_SECRET_KEY`    | 서비스 간 내부 통신 인증에 사용하는 Secret Key입니다.                  |

### Auth Service

| Variable                 | Description                 |
| ------------------------ | --------------------------- |
| `JWT_ACCESS_EXPIRATION`  | Access Token 만료 시간입니다.      |
| `JWT_REFRESH_EXPIRATION` | Refresh Token 만료 시간입니다.     |
| `KAKAO_CLIENT_ID`        | 카카오 OAuth Client ID입니다.     |
| `KAKAO_CLIENT_SECRET`    | 카카오 OAuth Client Secret입니다. |
| `KAKAO_REDIRECT_URI`     | 카카오 로그인 Redirect URI입니다.    |

### Device Service

| Variable                  | Description                  |
| ------------------------- | ---------------------------- |
| `NAVER_CLIENT_ID`         | 네이버 쇼핑 API Client ID입니다.     |
| `NAVER_CLIENT_SECRET`     | 네이버 쇼핑 API Client Secret입니다. |
| `AWS_ACCESS_KEY`          | AWS Access Key입니다.           |
| `AWS_SECRET_KEY`          | AWS Secret Key입니다.           |
| `AWS_REGION`              | AWS Region입니다.               |
| `AWS_S3_BUCKET`           | 이미지 저장에 사용할 S3 Bucket입니다.    |
| `AWS_LAMBDA_FUNCTION_URL` | OCR Lambda Function URL입니다.  |
| `VERTEX_API_KEY`          | Google Vertex AI API Key입니다. |

### Notification Service

| Variable               | Description                        |
| ---------------------- | ---------------------------------- |
| `FIREBASE_CONFIG_PATH` | Firebase Service Account 파일 경로입니다. |

### Admin Service

| Variable               | Description           |
| ---------------------- | --------------------- |
| `ADMIN_FRONTEND_URL`   | 관리자 웹 프론트엔드 URL입니다.   |
| `CORS_ALLOWED_ORIGINS` | CORS 허용 Origin 목록입니다. |

---

## 09. Run

### Development Server

개발 서버에서는 각 서비스의 `dev` 태그 이미지를 사용합니다.

```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

### Production Server

운영 서버에서는 각 서비스의 `latest` 태그 이미지를 사용합니다.

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### Local Environment

로컬 환경에서는 컨테이너가 로컬 PC의 MySQL을 바라보도록 `host.docker.internal`을 사용합니다.

```bash
docker compose -f docker-compose.yml -f docker-compose.local.yml up -d
```

---

## 10. Stop

전체 컨테이너를 종료합니다.

```bash
docker compose down
```

특정 compose 조합으로 실행한 경우 동일한 파일 조합으로 종료합니다.

```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml down
```

---

## 11. Logs

전체 컨테이너 로그를 확인합니다.

```bash
docker compose logs -f
```

특정 서비스 로그를 확인합니다.

```bash
docker compose logs -f auth-service
docker compose logs -f device-service
docker compose logs -f notification-service
docker compose logs -f admin-service
docker compose logs -f nginx
```

---

## 12. Restart

전체 컨테이너를 재시작합니다.

```bash
docker compose restart
```

특정 서비스만 재시작합니다.

```bash
docker compose restart nginx
docker compose restart auth-service
docker compose restart device-service
docker compose restart notification-service
docker compose restart admin-service
```

---

## 13. Image Tags

| Environment | Image Tag |
| ----------- | --------- |
| Development | `dev`     |
| Production  | `latest`  |

예시:

```text
ghcr.io/after-buy/after-buy-auth-service:dev
ghcr.io/after-buy/after-buy-auth-service:latest
```

---

## 14. SSL

Nginx는 HTTPS 연결을 위해 Let's Encrypt 인증서 경로를 사용합니다.

```text
/etc/letsencrypt/live/dev.after-buy.r-e.kr/fullchain.pem
/etc/letsencrypt/live/dev.after-buy.r-e.kr/privkey.pem
```

Certbot 인증서 발급 및 갱신을 위해 아래 경로가 Nginx 컨테이너에 마운트됩니다.

```text
./certbot/conf:/etc/letsencrypt
./certbot/www:/var/www/certbot
```

---

## 15. Firebase Config

Notification Service는 FCM 푸시 알림 발송을 위해 Firebase Service Account 파일을 사용합니다.

서버에는 아래 경로에 Firebase 설정 파일을 배치해야 합니다.

```text
/home/ubuntu/firebase-service-account.json
```

컨테이너 내부에서는 아래 경로로 마운트됩니다.

```text
/app/firebase-service-account.json
```

---

## 16. Network

모든 서비스는 동일한 Docker bridge network 안에서 통신합니다.

```text
after-buy-network
```

서비스 간 내부 통신은 컨테이너 이름을 기준으로 수행합니다.

```text
http://auth-service:8081
http://device-service:8082
http://notification-service:8083
http://admin-service:8084
```

---

## 17. Related Repositories

| Repository                                                                                    | Description             |
| --------------------------------------------------------------------------------------------- | ----------------------- |
| [after-buy-auth-service](https://github.com/After-Buy/after-buy-auth-service)                 | 사용자 인증 및 회원 관리 서비스      |
| [after-buy-device-service](https://github.com/After-Buy/after-buy-device-service)             | 전자제품 등록 및 관리 서비스        |
| [after-buy-notification-service](https://github.com/After-Buy/after-buy-notification-service) | 보증기간 알림 및 푸시 알림 서비스     |
| [after-buy-admin-service](https://github.com/After-Buy/after-buy-admin-service)               | 관리자 API 서비스             |
| [after-buy-ocr-lambda](https://github.com/After-Buy/after-buy-ocr-lambda)                     | AWS Lambda 기반 OCR 처리 함수 |
| [After-Buy Organization](https://github.com/After-Buy)                                        | After Buy 전체 프로젝트       |

---

## 18. Role in After Buy

이 레포지토리는 After Buy 백엔드 서비스가 하나의 서버 환경에서 안정적으로 실행될 수 있도록 Docker Compose, Nginx, SSL, 네트워크 설정을 관리합니다.

각 서비스는 독립된 컨테이너로 실행되며, Nginx는 외부 요청을 적절한 백엔드 서비스로 전달하는 API Gateway 역할을 수행합니다.
