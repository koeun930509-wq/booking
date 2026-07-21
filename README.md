# 주말 독서 토론 참가 관리 서비스

주말마다 한 권의 책을 두고 이야기 나누는 독서 토론 모임의 예약 문의를 받고,
사장님이 접수 내역을 확인·확정·삭제할 수 있는 서비스입니다.

- 손님은 로그인 없이 예약 문의를 남길 수 있습니다.
- 사장님은 로그인 후에만 접수 내역을 보고 관리할 수 있습니다.

## 화면 구성

### 고객 페이지 — `index.html`
- 서비스 소개 (무엇을 / 얼마에 / 어디서)
- 예약 문의 폼 (이름, 연락처, 희망 날짜, 요청사항)
- 제출 시 `reservations` 테이블에 새 행을 추가(INSERT)

### 관리자 페이지 — `admin.html`
- 이메일/비밀번호 로그인 (회원가입 기능 없음, 계정은 Supabase에서 직접 생성)
- 로그인 전에는 로그인 폼만 보이고 접수 내역은 절대 노출되지 않음
- 로그인 후: 접수 내역을 접수 시각 최신순으로 표시
  - 각 행: 이름 / 연락처 / 희망 날짜 / 요청사항 / 상태 배지 / 접수 시각 / [확정][삭제] 버튼
  - 상태 배지: 대기(회색) / 확정(초록)
  - [확정] 버튼: 해당 행 상태를 "확정"으로 변경 후 표 갱신
  - [삭제] 버튼: 확인창(confirm) 통과 시 삭제 후 표 갱신
  - 로그아웃 버튼: 세션 종료 후 로그인 폼으로 복귀

## 기술 구성

- HTML/CSS/JS 단일 파일 구성, 별도 프레임워크나 빌드 도구 없음
- Supabase JS 라이브러리는 CDN 방식으로 로드 (`@supabase/supabase-js@2`)
- 데이터베이스: Supabase Postgres, 테이블 `reservations`

| 컬럼 | 타입 | 설명 |
|---|---|---|
| `id` | bigint (identity) | 기본 키 |
| `name` | text | 신청자 이름 |
| `contact` | text | 신청자 연락처 |
| `hope_date` | date | 희망 참가 날짜 |
| `request` | text (nullable) | 요청사항 |
| `status` | text | 접수 상태, 기본값 `대기` |
| `created_at` | timestamptz | 접수 시각, 기본값 `now()` |

## 권한 설계 (Row Level Security)

화면에서 로그인 폼을 숨기는 것과 별개로, 실제 데이터 접근은 Supabase RLS 정책으로 강제합니다.

| 정책 | 대상 | 허용 동작 |
|---|---|---|
| `anon_insert` | 비로그인(anon) | INSERT만 허용 |
| `authenticated_select` | 로그인(authenticated) | SELECT 허용 |
| `authenticated_update` | 로그인(authenticated) | UPDATE 허용 |
| `authenticated_delete` | 로그인(authenticated) | DELETE 허용 |

즉 로그인하지 않은 상태로 API를 직접 호출해도 접수자 이름·연락처는 조회되지 않습니다.
관리자 계정은 Supabase Auth에 1개만 존재하며, 앱 어디에도 회원가입 경로가 없습니다.

## 만들지 않은 것 (범위 제외)

- 고객 회원가입
- 관리자 가입(회원가입) 기능
- 결제 기능

## 파일 구성

```
├── index.html    # 고객 페이지 (예약 문의)
├── admin.html    # 관리자 페이지 (로그인 + 접수 관리)
├── prd.md        # 기획 문서 (서비스 정의, 화면 구성, 권한 설계)
└── 흐름도.md      # 함수 호출 흐름 · 데이터 이동 다이어그램 (mermaid)
```

전체 함수 호출 관계와 데이터 흐름을 그림으로 보고 싶다면 [`흐름도.md`](./흐름도.md)를 참고하세요.

## 실행 방법

빌드 과정 없이 `index.html` 또는 `admin.html`을 브라우저로 열면 바로 동작합니다.
두 파일 모두 같은 Supabase 프로젝트의 publishable(anon) key를 사용합니다.
