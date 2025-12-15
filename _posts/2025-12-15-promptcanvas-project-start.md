---
title: "[PromptCanvas] AI 이미지 프롬프트 관리 시스템 만들기"
date: 2025-12-15 23:30:00 +0900
categories: [Project, AI]
tags: [python, fastapi, ai, dall-e, sqlite, sqlalchemy, backend]
pin: false
---

## 새로운 프로젝트를 시작한다

최근 AI 이미지 생성 도구(DALL-E, Midjourney, Stable Diffusion)를 쓰면서 느낀 불편함을 해결하기 위한 프로젝트 **PromptCanvas**를 시작했다.

---

## 왜 이 프로젝트를 만들게 되었나?

### 문제 상황

AI 이미지 생성을 하다 보니 이런 불편함들이 있었다:

**1. 프롬프트 관리가 어려움**
- 좋은 결과를 낸 프롬프트를 다시 찾을 수 없다
- 어떤 프롬프트가 어떤 이미지를 만들었는지 추적이 불가능하다

**2. AI 모델 업그레이드 시 재활용 불가**
- Midjourney v5 → v6, DALL-E 2 → 3로 업그레이드되어도
- 예전 프롬프트를 찾아서 다시 실행해야 한다

**3. 창작 과정이 기록되지 않음**
- 시행착오가 남지 않는다
- 발전 과정을 볼 수 없다

### 해결 방안

**PromptCanvas**는 이런 문제들을 해결한다:
- 프롬프트와 이미지를 **함께** 저장
- AI 모델 버전별로 결과를 **자동 비교**
- Git처럼 변경 이력을 추적하여 **창작 여정을 기록**

---

## 핵심 기능

### 1️⃣ 프롬프트 & 이미지 관리

- 프롬프트 텍스트와 생성된 이미지 함께 저장
- 태그/카테고리로 분류
- 키워드 검색 및 필터링

### 2️⃣ AI 버전 간 비교

> "AI 모델이 업그레이드되면, 과거의 모든 프롬프트를 새 모델로 재실행하여 Before/After를 한눈에 비교"

이 기능이 핵심이다:
- 선택한 프롬프트들을 새 AI 모델로 일괄 재생성
- 같은 프롬프트의 AI 모델별 결과 나란히 표시
- Midjourney v5 vs v6, DALL-E 2 vs 3 비교

### 3️⃣ 창작 갤러리

- 시간순으로 창작 과정 보기
- 프로젝트별로 그룹화
- 포트폴리오 생성 및 공유

---

## 기술 스택 선정

### 핵심 철학

> **"Python을 공부하면서 AI와 함께 개발한다"**

- Python 초보도 따라갈 수 있도록
- 내장 기능 우선, 복잡한 고급 기법은 피하기
- AI가 잘 이해하는 대중적인 라이브러리만 사용

### 선택한 기술과 그 이유

**Python 3.11**
- AI 친화적이고 학습용으로 최적이다
- 타입 힌팅이 강화되어 코드 안정성이 높다
- 최신 버전이라 성능 향상과 에러 메시지 개선이 있다

**FastAPI**
- 타입 힌트 기반으로 코드 작성 시 실수를 줄여준다
- Swagger UI가 자동 생성되어 API 문서화가 편하다
- 비동기 처리가 기본으로 내장되어 있다 (동기 방식도 지원)
- Flask보다 빠르고, Django보다 가볍다

**Uvicorn**
- FastAPI 공식 권장 서버다
- 별도 설정 없이 바로 사용 가능하다
- ASGI 표준을 따라 비동기 처리를 잘 지원한다

**SQLite → PostgreSQL**
- SQLite로 시작하는 이유:
  - Python에 기본 내장되어 있어 설치가 필요 없다
  - 파일 기반이라 배포와 백업이 간단하다
  - 학습 및 MVP 단계에 최적이다
  - 설정이 거의 필요 없다
  
- PostgreSQL로 전환하는 이유:
  - 실무 표준이다
  - 동시 접속 처리가 우수하다
  - Full-text search 기능이 강력하다
  - ORM(SQLAlchemy)을 쓰면 코드 변경 없이 전환 가능하다

**SQLAlchemy 2.x (동기 방식)**
- ORM(Object-Relational Mapping)이라 SQL을 직접 쓰지 않아도 된다
- 타입 힌팅이 강화되어 IDE 자동완성이 잘 된다
- 사용자가 많아 레퍼런스를 찾기 쉽다
- 비동기가 아닌 동기 방식을 선택한 이유:
  - AI와의 협업 시 이해하기 쉽다
  - 디버깅이 간단하다
  - 초보자 친화적이다
  - 성능이 크리티컬하지 않은 단계에서는 충분하다

**OpenAI SDK**
- DALL-E 3 API 연동을 위해 사용한다
- 공식 SDK라 안정적이다
- 간단한 인터페이스로 이미지 생성을 요청할 수 있다

---

## 데이터베이스 설계

### ERD (Entity-Relationship Diagram)

```
┌─────────────────────┐
│   prompts           │
├─────────────────────┤
│ id (PK)             │
│ text                │──┐
│ category            │  │
│ tags (JSON)         │  │
│ parent_prompt_id    │  │
│ created_at          │  │
│ updated_at          │  │
└─────────────────────┘  │
                         │ 1:N
                         │
                    ┌────▼──────────────────┐
                    │  generated_images     │
                    ├───────────────────────┤
                    │ id (PK)               │
                    │ prompt_id (FK)        │
                    │ ai_model              │
                    │ ai_model_version      │
                    │ image_path            │
                    │ parameters (JSON)     │
                    │ created_at            │
                    └───────────────────────┘
```

### 테이블 상세 설명

**prompts (프롬프트)**
- `id`: 고유 식별자 (Primary Key)
- `text`: 프롬프트 텍스트 (최대 5000자)
- `category`: 카테고리 (예: "캐릭터", "풍경", "추상")
- `tags`: 태그 배열 (JSON 형식으로 저장)
- `parent_prompt_id`: 부모 프롬프트 ID (프롬프트 변경 이력 추적용)
- `created_at`, `updated_at`: 생성/수정 시간

**generated_images (생성된 이미지)**
- `id`: 고유 식별자 (Primary Key)
- `prompt_id`: 어떤 프롬프트로 생성했는지 (Foreign Key)
- `ai_model`: AI 모델 이름 (예: "dall-e-3", "midjourney")
- `ai_model_version`: 모델 버전 (예: "v6", "3.0")
- `image_path`: 이미지 파일 경로 (로컬 저장소 경로)
- `parameters`: 생성 파라미터 (JSON 형식, 예: {"size": "1024x1024", "quality": "hd"})
- `created_at`: 생성 시간

### 왜 이렇게 설계했나?

**1. prompts와 generated_images를 분리한 이유**
- 하나의 프롬프트로 여러 이미지를 생성할 수 있다 (1:N 관계)
- AI 모델별, 버전별로 같은 프롬프트를 재실행할 수 있다
- 이미지만 삭제하고 프롬프트는 보존할 수 있다

**2. parent_prompt_id를 둔 이유**
- 프롬프트를 수정하면서 변경 이력을 추적할 수 있다
- "이 프롬프트는 어떤 프롬프트에서 파생되었는가?"를 알 수 있다
- Git의 커밋 히스토리처럼 창작 과정을 기록한다

**3. parameters를 JSON으로 저장한 이유**
- AI 모델마다 파라미터가 다르다 (size, quality, style 등)
- 새로운 파라미터가 추가되어도 테이블 구조를 변경할 필요가 없다
- 유연하게 확장 가능하다

---

## 개발 계획

### Phase 1: MVP (1-2주)

**목표:** 프롬프트 저장 및 조회 기능 구현

**환경 설정:**
```bash
# 프로젝트 디렉토리 생성
mkdir promptcanvas
cd promptcanvas

# 가상 환경 생성 및 활성화
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 라이브러리 설치
pip install fastapi uvicorn sqlalchemy alembic python-dotenv
```

**구현 내용:**
- SQLite 데이터베이스 설정 및 테이블 생성
- 프롬프트 CRUD API (Create, Read, Update, Delete)
- 이미지 파일 업로드 기능
- Swagger 문서 자동 생성 확인

**학습 목표:**
- FastAPI의 라우팅과 Pydantic 모델 이해
- SQLAlchemy의 ORM 사용법 익히기
- 파일 업로드 처리 방법 학습

### Phase 2: AI 통합 (1주)

**목표:** OpenAI DALL-E 3 API 연동

**구현 내용:**
- OpenAI API 키 설정 (.env 파일 사용)
- 프롬프트 입력 시 자동으로 이미지 생성
- 배치 재생성 기능 (여러 프롬프트를 한 번에)
- 생성된 이미지를 로컬에 저장

**학습 목표:**
- 외부 API 연동 방법
- 비동기 처리 (필요 시)
- 에러 핸들링 (API 실패 시 대응)

### Phase 3: 고급 기능 (1-2주)

**목표:** 사용자 경험 개선

**구현 내용:**
- AI 모델 버전 비교 뷰 (Before/After)
- 프롬프트 변경 이력 Diff 표시
- Full-text search (PostgreSQL로 전환 후)
- 태그 기반 필터링

### Phase 4: 배포 (1주)

**목표:** 프로젝트 완성 및 공유

**구현 내용:**
- Docker 컨테이너화 (Dockerfile 작성)
- 클라우드 배포 (Heroku, Railway, Render 중 선택)
- README 및 API 문서 완성
- GitHub 저장소 정리

---

## 예상 결과물

### 기능적 결과물

- 프롬프트와 이미지를 함께 관리하는 웹 애플리케이션
- AI API 연동하여 자동 이미지 생성
- 여러 프롬프트를 한 번에 재생성하는 배치 기능
- AI 모델별 결과 비교 기능
- 창작 과정 타임라인

### 기술적 성과

- FastAPI로 RESTful API 개발 경험
- SQLAlchemy로 데이터베이스 설계 및 ORM 사용 경험
- AI API 통합 경험
- Docker를 활용한 배포 경험
- Python 백엔드 개발 실전 경험

---

## 블로그 시리즈 계획

앞으로 이 프로젝트 개발 과정을 블로그 시리즈로 연재할 예정이다.

### Part 1: 기획 및 설계
- **이 글:** 프로젝트 소개 및 계획
- 기술 스택 선정 이유 상세
- 데이터베이스 설계 과정

### Part 2: 개발 과정
- FastAPI로 첫 API 만들기
- SQLAlchemy 모델 정의하기
- OpenAI DALL-E API 연동하기
- 배치 작업 구현하기

### Part 3: 고급 기능
- AI 모델 버전 비교 기능 구현
- 프롬프트 변경 이력 Diff 구현
- 성능 최적화 (쿼리 개선, 캐싱)

### Part 4: 배포 및 회고
- Docker로 컨테이너화
- 클라우드 배포 과정
- 프로젝트 회고: 배운 점과 아쉬운 점

---

## 다음 단계

이번 주 목표:
- [x] 프로젝트 계획 수립
- [ ] 개발 환경 설정
- [ ] SQLite 데이터베이스 설정
- [ ] 첫 API 만들기 (Hello World)

---

## 마치며

이 프로젝트를 통해:

- Python 백엔드 개발 실전 경험
- AI API 통합 경험
- 데이터베이스 설계 능력
- 실제 문제 해결 경험

을 얻을 수 있을 것으로 기대한다.

개발 과정에서 배운 점들을 계속 공유하겠다!

**다음 글:** "[PromptCanvas] FastAPI로 Hello World 만들기"에서 만나자! 👋
