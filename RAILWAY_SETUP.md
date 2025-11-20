# Railway 배포 가이드 - john-baek.com

## 1. Railway 프로젝트 정보
- **프로젝트 이름**: kutt-johnbaek
- **프로젝트 URL**: https://railway.com/project/c3101a03-b2ac-4bcc-a922-2548cea54882
- **GitHub 저장소**: https://github.com/jihoon-baek-plusminus-zero/kutt-shortner-johnbaek

---

## 2. Railway 웹 콘솔에서 수동 설정

### Step 1: GitHub 서비스 연결
1. Railway 프로젝트 대시보드 열기: https://railway.com/project/c3101a03-b2ac-4bcc-a922-2548cea54882
2. "+ New" 버튼 클릭
3. "GitHub Repo" 선택
4. `jihoon-baek-plusminus-zero/kutt-shortner-johnbaek` 저장소 선택
5. "Deploy Now" 클릭

### Step 2: 환경 변수 설정
서비스 생성 후, Settings → Variables 탭에서 다음 환경 변수를 추가:

#### 필수 환경 변수
```bash
# 보안
JWT_SECRET=KNld2grdEcrEHmRUYfpzWkkoFKTWeBkJPCMhSWfnzVg=

# 도메인 설정
DEFAULT_DOMAIN=john-baek.com
SITE_NAME=John Baek URL Shortener

# 데이터베이스 (PostgreSQL - Railway에서 자동 연결)
DB_CLIENT=pg
DB_HOST=${{Postgres.PGHOST}}
DB_PORT=${{Postgres.PGPORT}}
DB_NAME=${{Postgres.PGDATABASE}}
DB_USER=${{Postgres.PGUSER}}
DB_PASSWORD=${{Postgres.PGPASSWORD}}
DB_SSL=false

# Redis (Railway에서 자동 연결)
REDIS_ENABLED=true
REDIS_HOST=${{Redis.REDIS_HOST}}
REDIS_PORT=${{Redis.REDIS_PORT}}
REDIS_PASSWORD=${{Redis.REDIS_PASSWORD}}

# 기능 설정
TRUST_PROXY=true
ENABLE_RATE_LIMIT=true
DISALLOW_REGISTRATION=false
DISALLOW_ANONYMOUS_LINKS=false

# 링크 설정
LINK_LENGTH=6
CUSTOM_DOMAIN_USE_HTTPS=true

# 이메일 (선택사항 - 나중에 설정)
MAIL_ENABLED=false
```

#### Railway 자동 변수 참조 방법
- PostgreSQL 변수는 Postgres 서비스 이름에 따라 자동으로 `${{Postgres.변수명}}` 형식으로 참조
- Redis 변수는 Redis 서비스 이름에 따라 자동으로 `${{Redis.변수명}}` 형식으로 참조

### Step 3: 배포 확인
1. Deployments 탭에서 배포 상태 확인
2. 로그에서 "Ready on http://localhost:3000" 메시지 확인
3. Settings → Domains에서 Railway 제공 URL 확인 (예: kutt-johnbaek-production.up.railway.app)

### Step 4: 커스텀 도메인 연결 (john-baek.com)
1. Settings → Domains 탭 이동
2. "+ Custom Domain" 클릭
3. `john-baek.com` 입력
4. Railway에서 제공하는 DNS 레코드 복사

#### DNS 레코드 설정 (도메인 등록업체에서)
Railway에서 제공하는 CNAME 레코드를 도메인 관리 페이지에서 추가:

**예시 (실제 값은 Railway에서 확인):**
```
Type: CNAME
Name: @  (또는 john-baek.com)
Value: [Railway에서 제공하는 CNAME 값]
TTL: 3600
```

**서브도메인 (www) 설정 (선택사항):**
```
Type: CNAME
Name: www
Value: [Railway에서 제공하는 CNAME 값]
TTL: 3600
```

5. DNS 전파 대기 (최대 48시간, 보통 몇 분 내)
6. Railway에서 SSL 인증서 자동 발급 대기

---

## 3. 초기 관리자 계정 생성

배포 완료 후, 웹 브라우저에서:

1. `https://john-baek.com/create-admin` 접속
2. 관리자 이메일/비밀번호 입력
3. 생성 완료 후 로그인

---

## 4. 데이터베이스 마이그레이션 확인

Railway 서비스 로그에서 다음 메시지 확인:
```
> Ready on http://localhost:3000
```

마이그레이션은 Dockerfile의 `CMD` 명령어에서 자동 실행됨:
```dockerfile
CMD npm run migrate && npm start
```

---

## 5. 서비스 URL 정리

### Railway 자동 URL
- https://kutt-johnbaek-production.up.railway.app (자동 생성)

### 커스텀 도메인
- https://john-baek.com (설정 후)
- https://www.john-baek.com (선택사항)

---

## 6. Railway 서비스 구조

```
kutt-johnbaek (프로젝트)
├── kutt-johnbaek (GitHub 서비스)
│   ├── 환경변수: JWT_SECRET, DEFAULT_DOMAIN, etc.
│   └── 도메인: john-baek.com
├── Postgres (PostgreSQL 데이터베이스)
│   └── 자동 변수: PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD
└── Redis (Redis 캐시)
    └── 자동 변수: REDIS_HOST, REDIS_PORT, REDIS_PASSWORD
```

---

## 7. 트러블슈팅

### 배포 실패 시
- Railway 로그 확인: Deployments → 최신 배포 → View Logs
- 환경 변수 누락 확인
- PostgreSQL/Redis 서비스 실행 상태 확인

### 도메인 연결 안 될 시
- DNS 전파 확인: `dig john-baek.com` 또는 `nslookup john-baek.com`
- Railway에서 제공한 CNAME 값 정확히 입력했는지 확인
- 24-48시간 대기

### 데이터베이스 연결 오류
- Railway Variables 탭에서 `${{Postgres.*}}` 변수가 올바르게 참조되는지 확인
- PostgreSQL 서비스가 실행 중인지 확인

---

## 8. 다음 단계

1. **이메일 설정** (선택사항)
   - SMTP 서버 정보로 MAIL_* 환경 변수 설정
   - MAIL_ENABLED=true로 변경

2. **커스터마이징**
   - `/custom` 폴더에 CSS/이미지 추가
   - GitHub에 커밋/푸시하면 자동 배포

3. **모니터링**
   - Railway 대시보드에서 메트릭 확인
   - 로그 모니터링

---

## 현재 상태

✅ GitHub 저장소 생성: https://github.com/jihoon-baek-plusminus-zero/kutt-shortner-johnbaek
✅ Railway 프로젝트 생성: https://railway.com/project/c3101a03-b2ac-4bcc-a922-2548cea54882
✅ PostgreSQL 서비스 추가
✅ Redis 서비스 추가
✅ JWT Secret 생성: KNld2grdEcrEHmRUYfpzWkkoFKTWeBkJPCMhSWfnzVg=

⏳ **다음 작업 (Railway 웹 콘솔에서 수동):**
1. GitHub 저장소를 서비스로 연결
2. 환경 변수 설정
3. 배포 확인
4. john-baek.com 도메인 연결
