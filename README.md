# K8s 마이크로서비스 데모

이 프로젝트는 Kubernetes 환경에서 Redis, MariaDB, Kafka를 활용하는 마이크로서비스 데모입니다.

## CI/CD 설정

### GitHub Actions 설정

이 프로젝트는 GitHub Actions를 통해 자동으로 Docker 이미지를 빌드하고 Azure Container Registry(ACR)에 푸시합니다.

#### 필요한 GitHub Secrets 설정

GitHub 저장소의 Settings > Secrets and variables > Actions에서 다음 시크릿을 설정해야 합니다:

- `ACR_USERNAME`: Azure Container Registry 사용자명
- `ACR_PASSWORD`: Azure Container Registry 비밀번호

> ⚠️ **보안 주의**: 실제 비밀번호는 GitHub Secrets에만 저장하고, 코드에는 절대 포함하지 마세요.

#### 워크플로우 트리거

- 브랜치: `main`, `master`, `develop`에 푸시될 때 자동 실행
- 이미지 태그: 날짜 형식 (YYYYMMDD) + latest

#### 생성되는 이미지

- Backend: `ktech4.azurecr.io/kltecho_jiwoo-backend:{date}`
- Frontend: `ktech4.azurecr.io/kltecho_jiwoo-frontend:{date}`

### 배포 환경

#### 네임스페이스
- `jiwoo`: 애플리케이션 배포 네임스페이스

#### 데이터베이스 연결
- **MariaDB**: `jw-mariadb.jiwoo.svc.cluster.local` (jiwoo 네임스페이스, Helm으로 설치)
- **Redis**: `redis-master.default.svc.cluster.local` (default 네임스페이스, 포트 6379)
- **Redis Password**: `GjA0PHx96N`

#### 배포 명령어

```bash
# 네임스페이스 생성
kubectl create namespace jiwoo

# 시크릿 배포
kubectl apply -f k8s/backend-secret.yaml

# 백엔드 배포
kubectl apply -f k8s/backend-deployment.yaml

# 프론트엔드 배포
kubectl apply -f k8s/frontend-deployment.yaml
```

## 주요 기능

### 1. 사용자 관리
- 회원가입: 새로운 사용자 등록
- 로그인/로그아웃: 세션 기반 인증
- Redis를 활용한 세션 관리

### 2. 메시지 관리 (MariaDB)
- 메시지 저장: 사용자가 입력한 메시지를 DB에 저장
- 메시지 조회: 저장된 메시지 목록 표시
- 샘플 데이터 생성: 테스트용 샘플 메시지 생성
- 페이지네이션: 대량의 데이터 효율적 처리

### 3. 검색 기능
- 메시지 검색: 특정 키워드로 메시지 검색
- 전체 메시지 조회: 모든 저장된 메시지 표시
- Redis 캐시를 활용한 검색 성능 최적화

### 4. 로깅 시스템
- Redis 로깅: API 호출 로그 저장 및 조회
- Kafka 로깅: API 통계 데이터 수집

## 데이터베이스 구조

### MariaDB
```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message TEXT,
    created_at DATETIME,
    user_id VARCHAR(255)
);
```

### Redis 데이터 구조
- 세션 저장: `session:{username}`
- API 로그: `api_logs` (List 타입)
- 검색 캐시: `search:{query}`

## API 엔드포인트

### 사용자 관리
- POST /register: 회원가입
- POST /login: 로그인
- POST /logout: 로그아웃

### 메시지 관리
- POST /db/message: 메시지 저장
- GET /db/messages: 전체 메시지 조회
- GET /db/messages/search: 메시지 검색

### 로그 관리
- GET /logs/redis: Redis 로그 조회
- GET /logs/kafka: Kafka 로그 조회

## 환경 변수 설정
```yaml
- MYSQL_HOST: MariaDB 호스트
- MYSQL_USER: MariaDB 사용자
- MYSQL_PASSWORD: MariaDB 비밀번호
- REDIS_HOST: Redis 호스트
- REDIS_PASSWORD: Redis 비밀번호
- KAFKA_SERVERS: Kafka 서버
- KAFKA_USERNAME: Kafka 사용자
- KAFKA_PASSWORD: Kafka 비밀번호
- FLASK_SECRET_KEY: Flask 세션 암호화 키
```

## 보안 기능
- 비밀번호 해시화 저장
- 세션 기반 인증
- Redis를 통한 세션 관리
- API 접근 제어

## 성능 최적화
- Redis 캐시를 통한 검색 성능 향상
- 비동기 로깅으로 API 응답 시간 개선
- 페이지네이션을 통한 대용량 데이터 처리

## 모니터링
- API 호출 로그 저장 및 조회
- 사용자 행동 추적
- 시스템 성능 모니터링 