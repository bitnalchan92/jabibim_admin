# JABibim Admin - 올인원 강의 관리 플랫폼

> 영세 강사 및 소규모 학원을 위한 강의 콘텐츠 관리, 수강생 관리, 결제 처리를 통합한 SaaS형 어드민 시스템

## 프로젝트 개요

시장에 진입하는 1인 강사들은 자체 강의 플랫폼 구축에 필요한 기술적 지식과 비용 부담으로 어려움을 겪고 있습니다. JABibim은 강의 개설부터 동영상 업로드, 결제 처리, 학생 관리까지 교육 전반의 기능을 제공하는 올인원 솔루션입니다.

### 핵심 가치
- **비용 절감**: 자체 플랫폼 구축 대비 저렴한 SaaS 모델
- **간편한 운영**: 직관적인 UI로 기술 지식 없이도 강의 관리 가능
- **실시간 소통**: WebSocket 기반 채팅으로 수강생과 직접 소통
- **멀티테넌트**: 학원 단위 데이터 격리로 확장성 확보

---

## 기술 스택

### Backend
| 분류 | 기술 |
|------|------|
| Framework | Spring Boot 3.2.11 |
| ORM | MyBatis 3.0.3 |
| Security | Spring Security 6 + JWT (JJWT 0.12.3) |
| Database | MySQL 8.0 |
| Cache | Redis |

### Frontend
| 분류 | 기술 |
|------|------|
| Template Engine | Thymeleaf + Layout Dialect |
| UI Framework | Bootstrap 5 (NiceAdmin) |
| Rich Text Editor | TinyMCE |
| Charts | Chart.js |

### Infrastructure
| 분류 | 기술 |
|------|------|
| Cloud Storage | AWS S3 |
| Video Processing | FFmpeg (HLS 스트리밍) |
| Real-time | WebSocket |
| CI/CD | Jenkins + Docker |
| Notification | Slack Webhook |

### External Services
| 분류 | 기술 |
|------|------|
| OAuth2 | Google Login |
| Calendar | Google Calendar API |
| Payment | Port One (구 I'mport) |

---

## 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client Layer                             │
├─────────────────────────────────────────────────────────────────┤
│  Admin Web (Thymeleaf)          │  Front App (REST API + JWT)   │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Application Layer                           │
├─────────────────────────────────────────────────────────────────┤
│  Spring Security    │  Controller  │  Service  │  MyBatis       │
│  (Form + OAuth2)    │    Layer     │   Layer   │   Mapper       │
└─────────────────────────────────────────────────────────────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
              ┌─────────┐   ┌─────────┐   ┌─────────┐
              │  MySQL  │   │  Redis  │   │ AWS S3  │
              │   DB    │   │  Cache  │   │ Storage │
              └─────────┘   └─────────┘   └─────────┘
```

### 인증 흐름

**Admin Web (Form + OAuth2)**
```
로그인 요청 → Spring Security Filter → FormAuthenticationProvider
                                      → CustomOAuth2UserService (Google)
           → SecurityContext 저장 → 세션 기반 인증
```

**Front API (JWT)**
```
로그인 요청 → LoginFilter → JwtTokenProvider (토큰 발급)
API 요청   → JwtAuthenticationFilter → 토큰 검증 → 권한 부여
```

---

## 프로젝트 구조

```
src/main/
├── java/com/jabibim/admin/
│   ├── config/                 # 설정 (Redis, S3, WebSocket, FFmpeg)
│   ├── controller/             # Admin 웹 컨트롤러
│   ├── service/                # 비즈니스 로직
│   ├── mybatis/mapper/         # MyBatis 매퍼 인터페이스
│   ├── domain/                 # 엔티티 모델
│   ├── dto/                    # 요청/응답 DTO
│   ├── security/               # Spring Security 설정
│   ├── front/                  # 프론트 API (JWT 기반)
│   │   ├── api_receive/        # API 엔드포인트
│   │   ├── api_send/           # 외부 API 호출 (Port One)
│   │   └── security/           # JWT 인증
│   ├── oauth2/                 # OAuth2 커스텀 설정
│   ├── func/                   # 유틸리티 (S3, FFmpeg, Slack)
│   └── task/                   # 스케줄 작업
│
└── resources/
    ├── mybatis/mapper/         # SQL 매핑 파일 (XML)
    ├── templates/              # Thymeleaf 템플릿
    │   ├── layouts/            # 레이아웃
    │   ├── fragments/          # 공통 컴포넌트 (header, sidebar)
    │   ├── content/            # 콘텐츠 관리 화면
    │   ├── students/           # 학생 관리
    │   └── orders/             # 주문 관리
    └── static/                 # 정적 자원 (CSS, JS)
```

---

## 주요 기능

### 1. Dashboard (대시보드)

실시간 운영 현황을 한눈에 파악할 수 있는 대시보드

| 기능 | 설명 |
|------|------|
| 환불 대기 알림 | 처리 대기 중인 환불 건수 실시간 표시 |
| 과정 현황 | 전체/공개/비공개 과정 통계 |
| 신규 수강생 차트 | 일별 수강생 증가 추이 (Chart.js) |
| 역할 기반 필터링 | Admin은 전체, Teacher는 자신의 학원만 조회 |

**관련 파일:**
- Controller: `HomeController.java`
- Service: `DashboardService.java`, `DashboardServiceImpl.java`
- Mapper: `DashboardMapper.java`, `DashBoard.xml`
- View: `dashboard.html`, `dashboard.js`

### 2. Content (콘텐츠 관리)

과정 및 강의 콘텐츠의 전체 생명주기 관리

#### 과정(Course) 관리
| 기능 | 설명 |
|------|------|
| 과정 CRUD | 생성, 조회, 수정, 삭제 (소프트 삭제) |
| 썸네일 관리 | S3 업로드/교체 |
| 활성화 토글 | 공개/비공개 전환 |
| 검색 및 필터 | 과정명, 담당자, 등록일 검색 |

#### 강의(Class) 관리
| 기능 | 설명 |
|------|------|
| 강의 등록 | 제목, 설명, 타입 설정 |
| 동영상 업로드 | S3 저장 + FFmpeg HLS 인코딩 |
| 자료 첨부 | PDF, 문서 등 학습 자료 관리 |
| 스트리밍 | HLS 프로토콜로 적응형 비트레이트 스트리밍 |

**파일 저장 구조 (S3):**
```
bucket/
└── course/{courseId}/
    ├── profile/profile.{ext}     # 과정 썸네일
    └── class/{classId}/
        ├── raw/                  # 원본 동영상
        └── encode/               # HLS 인코딩 (.m3u8, .ts)
```

**관련 파일:**
- Controller: `ContentController.java`
- Service: `ContentService.java`, `ContentServiceImpl.java`, `FFmpegService.java`, `S3FileService.java`
- Mapper: `ContentMapper.java`, `CourseMapper.java`
- View: `content/course/*.html`, `content/class/*.html`

### 3. 회원 관리
| 기능 | 설명 |
|------|------|
| 수강생 관리 | 목록 조회, 상세 정보, 등급 관리 |
| 강사 관리 | 권한 설정, 프로필 관리 |
| 접속/탈퇴 이력 | 로그인 이력, 탈퇴 사유 관리 |

### 4. 주문/결제 관리
| 기능 | 설명 |
|------|------|
| 주문 목록 | 결제 내역 조회 및 상태 관리 |
| 환불 처리 | 환불 요청 승인/거절 |
| Webhook 연동 | Port One 결제 상태 자동 동기화 |
| 재시도 메커니즘 | 결제 확인 실패 시 Spring Retry 적용 |

### 5. 커뮤니티
| 기능 | 설명 |
|------|------|
| 공지사항 | 작성, 수정, 삭제 (TinyMCE 에디터) |
| Q&A | 질문 조회, 답변 작성 |
| 수강평 | 리뷰 조회 및 관리 |
| 실시간 채팅 | WebSocket 기반 1:1 상담 |

### 6. 시스템 설정
| 기능 | 설명 |
|------|------|
| 개인정보처리방침 | 버전 관리 및 이력 |
| 이용약관 | 약관 버전 관리 |
| 사업자 정보 | 학원 기본 정보 설정 |
| Google Calendar | OAuth2 연동 일정 관리 |

---

## 기술적 특징

### 1. 멀티테넌트 아키텍처
```java
// 학원(Academy) 기반 데이터 격리
@Service
public class DashboardServiceImpl implements DashboardService {
    public Map<String, Object> getCourseStatus(String academyId, String role) {
        if ("ROLE_ADMIN".equals(role)) {
            return mapper.getCourseStatusAll();  // 전체 조회
        }
        return mapper.getCourseStatusByAcademy(academyId);  // 학원별 조회
    }
}
```

### 2. HLS 비디오 스트리밍
```java
// FFmpeg를 이용한 적응형 비트레이트 스트리밍
@Service
public class FFmpegServiceImpl implements FFmpegService {
    public void encodeToHLS(String inputPath, String outputDir) {
        // 원본 → HLS 변환 (.m3u8 + .ts 세그먼트)
        // 다양한 해상도별 스트림 생성
    }
}
```

### 3. 결제 Webhook 처리
```java
// Port One 결제 확인 + 재시도 메커니즘
@RestController
public class PaymentWebhookController {
    @Retryable(maxAttempts = 3, backoff = @Backoff(delay = 1000))
    public void handleWebhook(PaymentWebhookRequest request) {
        // 결제 상태 검증 및 주문 상태 업데이트
    }
}
```

### 4. 실시간 알림
```java
// Slack Webhook 에러 알림
@Component
public class SlackWebhookNotifier {
    public void sendErrorNotification(String message) {
        // 운영 중 에러 발생 시 Slack 채널 알림
    }
}
```

---

## 개발 환경 설정

### 필수 요구사항
- **Java 17** (필수 - 17 버전으로 맞춰야 함)
- Maven 3.8+ (또는 Maven Wrapper 사용)
- MySQL 8.0+
- Redis 6+
- FFmpeg 4+ (동영상 인코딩 기능 사용 시)

### 1. 의존성 설치

**macOS (Homebrew)**
```bash
# MySQL
brew install mysql
brew services start mysql

# Redis
brew install redis
brew services start redis

# FFmpeg
brew install ffmpeg
```

### 2. 데이터베이스 설정

```bash
# MySQL 접속
mysql -u root -p

# 데이터베이스 생성
CREATE DATABASE jabibim CHARACTER SET utf8mb3;

# 테스트 데이터 import
mysql -u root -p jabibim < src/main/resources/static/sql/testdata.sql
```

### 3. 로컬 설정 파일

`src/main/resources/application-local.properties` 파일 생성 후 아래 값들을 설정:

```properties
# Server
server.port=4000
server.servlet.context-path=/admin

# Database
spring.datasource.url=jdbc:log4jdbc:mysql://localhost:3306/jabibim?useSSL=false&serverTimezone=Asia/Seoul&characterEncoding=UTF-8&allowPublicKeyRetrieval=true
spring.datasource.username=root
spring.datasource.password={YOUR_MYSQL_PASSWORD}

# AWS S3
cloud.aws.s3.bucket={YOUR_BUCKET_NAME}
cloud.aws.credentials.access-key={YOUR_AWS_ACCESS_KEY}
cloud.aws.credentials.secret-key={YOUR_AWS_SECRET_KEY}

# Google OAuth2
google.client.id={YOUR_GOOGLE_CLIENT_ID}
google.client.secret={YOUR_GOOGLE_CLIENT_SECRET}

# FFmpeg (macOS Apple Silicon)
ffmpeg.location=/opt/homebrew/bin/ffmpeg
ffprobe.location=/opt/homebrew/bin/ffprobe
```

### 4. IntelliJ IDEA 설정

1. **Project SDK**: Java 17로 설정
   - `File` → `Project Structure` → `Project` → SDK: 17

2. **Annotation Processing 활성화**
   - `Settings` → `Build, Execution, Deployment` → `Compiler` → `Annotation Processors`
   - ✅ `Enable annotation processing` 체크

3. **Run Configuration 설정**
   - `Run` → `Edit Configurations`
   - Spring Boot 설정에서 `Active profiles: local` 입력

4. **Maven Reload**
   - `pom.xml` 우클릭 → `Maven` → `Reload Project`

### 5. 실행

**IntelliJ에서 실행**
- `JabibimAdminApplication.java` 우클릭 → Run
- 또는 Run Configuration에서 Spring Boot 앱 실행

**터미널에서 실행**
```bash
# Maven Wrapper 사용
./mvnw spring-boot:run -Dspring-boot.run.profiles=local

# 또는 Maven 설치된 경우
mvn spring-boot:run -Dspring-boot.run.profiles=local
```

**접속**
- http://localhost:4000/admin

### Docker 실행
```bash
# 빌드
docker build -t jabibim-admin .

# 실행
docker run -p 4000:4000 jabibim-admin
```

### 테스트 계정
| 역할 | 이메일 | 비밀번호 |
|------|--------|----------|
| 강사 | 1234@sample.com | 1234 |

---

## Git 브랜치 전략

| 브랜치 | 용도 |
|--------|------|
| `prod` | 운영 배포용 (직접 커밋 금지) |
| `dev` | 개발 통합 브랜치 |
| `feature/*` | 기능 개발 브랜치 |

### 커밋 메시지 규칙
- `[feat]` : 새로운 기능 추가
- `[fix]` : 버그 수정
- `[refactor]` : 코드 리팩토링
- `[docs]` : 문서 수정
- `[style]` : 코드 포맷팅
- `[test]` : 테스트 코드

---

## 라이선스
이 프로젝트는 교육 목적으로 제작되었습니다.
