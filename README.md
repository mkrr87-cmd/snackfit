# 🍿 SnackFit — 예산 맞춤 스낵박스 쇼핑몰

예산만 입력하면 과자 조합을 자동으로 구성해주는 쇼핑몰.
서버 없이 **HTML 4개 파일**로 동작하는 정적 사이트입니다.

- **실서비스 URL**: https://mkrr87-cmd.github.io/snackfit/
- **사이트 코드 저장소**: https://github.com/mkrr87-cmd/snackfit
- **데이터 저장소**: https://github.com/mkrr87-cmd/snackfit-data

---

## 1. 파일 구성 (이 저장소의 전부)

| 파일 | 역할 |
|---|---|
| `index.html` | 접속 시 로그인 페이지로 이동시키는 리다이렉트 |
| `login.html` | 로그인 페이지 (관리자 + 등록된 사용자) |
| `snackbox.html` | 쇼핑몰 본체 (자동 구성, 주문, 주문 내역) |
| `admin.html` | 관리자 페이지 (상품·계정·설정·주문 관리) |

빌드 도구, 프레임워크, 서버 코드 없음 — 순수 HTML/CSS/JavaScript.
파일을 그대로 복사하면 동일한 사이트가 됩니다.

## 2. 핵심 기능

### 쇼핑몰 (snackbox.html)
- **예산 입력 → 자동 구성**: 예산(최소 3,000원)을 입력하면 상품을 무작위 조합.
  합계가 `[예산, 예산÷0.95]` 구간에 들어가도록 담은 뒤,
  초과분을 **최대 5% 자동 할인**해서 최종 금액을 예산에 정확히 맞춤
- **커스텀**: 상품 제외(✕), 고정(🔒 — 재구성해도 상품·수량 유지, 수량 버튼 잠김), 수량 조절(±)
- **주문**: 주문자 이름·이메일(필수)·연락처·주소·요청사항 입력 →
  이메일 발송(아래 3-C) + 공유 주문 저장소에 기록
- **주문 내역**: 헤더 📋 버튼 — 내 주문 조회, 접수 상태일 때만 취소 가능
- 상품 이미지: 등록된 이미지가 없으면 카테고리 색상+이모지 SVG를 자동 생성

### 관리자 (admin.html) — 관리자 로그인 필수
- **주문 관리**: 전체 주문 확인, **수락** (수락하면 고객 취소 불가)
- **상품 관리**: 추가/수정/삭제, 이미지(URL 또는 파일 업로드 — 400px 자동 축소),
  **엑셀 대량 업로드**(.xlsx/.csv, 양식 다운로드 제공, 추가/전체교체 모드)
- **사용자 계정**: 등록/수정/삭제 — 여기 등록된 계정만 로그인 가능
- **주문 수신 설정**: 수신 이메일 + Web3Forms 키(자동 발송)
- **모든 기기 동기화**: GitHub 토큰(관리자 키) 등록

### 로그인 (login.html)
- 관리자 계정: 코드에 내장 (`ADMIN = { id: "poiu", pw: "1234" }` — login.html 안에서 변경)
- 사용자 계정: 클라우드(data.json)에서 최신 목록을 받아 검증
- 로그인하지 않으면 쇼핑몰/관리자 페이지 접근 불가 (각 페이지 상단 가드 스크립트)

## 3. 외부 서비스 (모두 무료)

### A. 데이터 저장소 — GitHub `snackfit-data` 저장소의 `data.json`
상품·사용자 계정·설정을 담는 단일 JSON. 구조:
```json
{
  "products": [ { "id": "...", "name": "새우깡", "price": 1500, "emoji": "🍤", "cat": "스낵", "img": "(선택)" } ],
  "users":    [ { "id": "testuser", "pw": "abcd", "name": "김간식", "created": 0 } ],
  "settings": { "orderEmail": "...", "web3formsKey": "..." }
}
```
- **읽기(모두)**: GitHub API → 실패 시 raw CDN 폴백. 코드 상수:
  - `CLOUD_API_READ = https://api.github.com/repos/mkrr87-cmd/snackfit-data/contents/data.json`
  - `CLOUD_DATA_URL = https://raw.githubusercontent.com/mkrr87-cmd/snackfit-data/main/data.json`
- **쓰기(관리자만)**: 관리자 페이지에 등록한 GitHub 토큰(fine-grained PAT,
  snackfit-data 저장소 Contents Read/Write 권한)으로 GitHub API PUT.
  토큰은 관리자 브라우저 localStorage(`snackfit_gh_token`)에만 저장됨

### B. 주문 저장소 — restful-api.dev 공개 객체
주문 접수/수락/취소 상태 공유용. 코드 상수 (snackbox.html, admin.html 동일):
```
ORDERS_URL = https://api.restful-api.dev/objects/ff8081819d82fab6019f3a5611712665
```
새로 만들려면: `POST https://api.restful-api.dev/objects`
바디 `{"name":"snackfit-orders","data":{"orders":[]}}` → 응답의 `id`로 상수 교체.
주문 1건 구조: `{ id, userId, name, email, phone, addr, memo, items[], sub, discount, total, budget, status(접수|수락|취소), createdAt }`

### C. 이메일 발송 — Web3Forms (+ mailto 폴백)
- 관리자가 키를 등록하면: `POST https://api.web3forms.com/submit` 로 자동 발송
  (키 발급: web3forms.com 에 수신 이메일 입력, 무료 월 250건)
  ⚠️ `ccemail` 필드는 유료 기능 — 넣으면 발송 전체가 실패하므로 사용 금지
- 키가 없으면: `mailto:` 링크로 메일 앱이 열림 (주문자 이메일이 CC로 들어가 사본 수신)

### D. 호스팅 — GitHub Pages
`snackfit` 저장소 main 브랜치 루트를 Pages로 서비스.

## 4. 브라우저 저장소 키 (localStorage)

| 키 | 내용 | 위치 |
|---|---|---|
| `snackfit_session_v1` | 로그인 세션 {id, name, role} | 각 사용자 브라우저 |
| `snackfit_products_v1` | 상품 캐시 (클라우드 실패 시 폴백) | 각 브라우저 |
| `snackfit_users_v1` | 계정 캐시 | 각 브라우저 |
| `snackfit_settings_v1` | 설정 캐시 | 각 브라우저 |
| `snackfit_gh_token` | 관리자 키 (동기화 안 됨, 절대 코드에 넣지 말 것) | 관리자 브라우저만 |

## 5. 처음부터 다시 만드는 순서

1. HTML 4개 파일 확보 (이 저장소 클론 또는 다운로드)
2. GitHub에서 데이터 저장소 생성 → `data.json` 업로드 (구조는 3-A)
3. 코드의 `mkrr87-cmd/snackfit-data` 부분을 새 저장소 경로로 치환 (3개 파일)
4. restful-api.dev에 주문 객체 생성 → `ORDERS_URL` 교체 (2개 파일)
5. 사이트 저장소 생성 → 4개 파일 푸시 → Settings→Pages에서 main 브랜치 활성화
6. 관리자 페이지 접속 → 주문 수신 이메일, Web3Forms 키, 관리자 키(GitHub 토큰) 등록
7. (선택) login.html의 관리자 ID/PW 변경

## 6. 수정·재배포 방법

```bash
# 파일 수정 후
git add -A
git commit -m "수정 내용"
git push        # 1~2분 뒤 사이트에 반영
```

## 7. 알려진 한계 (데모 수준)

- 인증이 브라우저 안에서만 검증됨 — 소스를 보면 관리자 PW 확인/우회 가능
- 사용자 비밀번호가 공개 저장소(data.json)에 평문 저장
- 주문 저장소가 공개 API — 누구나 수정 가능
- 실서비스 전환 시 Firebase/Supabase 등 서버 인증·DB로 교체 권장
