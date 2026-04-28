# 인수인계 — 이력서 MP 프로젝트 섹션

## 목적

이력서의 monorepo-playground(MP) 프로젝트 소개란에 넣을 best practice 항목 작성 중. 본인이 어필하고 싶은 주제 2개를 ## 헤딩 + ### 의도/개선 과정/배운 점 양식으로 정리.

## 현재 상태

이 폴더에 2개 파일 존재.

- `infinite-scroll.md` — 무한스크롤 게시글 목록 (PR 4 시리즈)
- `ai-safety-net.md` — ai 코드 안전망 (정적분석 + 2단계 훅)

두 파일 모두 plain-text 톤으로 정리 — 마크다운 문법 중 ## / ### 헤딩과 bullet만 사용. 백틱·볼드는 제거. 의도·개선 과정·배운 점 모두 명사형 종결로 통일.

## 작업 히스토리 요약

- 처음엔 MP `docs/best-practices-map.md`의 36개 카테고리에서 어필 후보를 좁히려 했음
- DC `README.md` 베스트 프랙티스 섹션 + `docs/vision/ai-era-growth.md`의 골격(잘 시키기 / 잘 리뷰하기)을 참고해 톤·카테고리 정렬
- DC `docs/infinite-scroll/step1~4`, `docs/error-handling/step1~7` 모두 읽고 자료 근거로 본문 채움
- 에러처리는 MP에 예제 페이지가 없어 제외 → 무한스크롤 + ai 안전망 2개로 확정
- 양식 결정: ## 주제 / ### 의도 / ### 개선 과정 / ### 배운 점. bullet 명사형 종결

## 다음 단계 (본인 작업)

### 1. 이력서 플랫폼 붙여넣기

이력서 입력 폼이 마크다운을 부분 지원(헤딩 + bullet)이라는 전제로 작성됨. 붙여넣기 후 실제 렌더링 확인 필요. 만약 ## 헤딩도 안 먹으면 모든 헤딩을 plain text(예: 줄바꿈 + 굵은 텍스트 없는 라벨)로 다시 평탄화 필요.

### 2. 본인 검토 항목

- **배운 점 bullet들** — 1인칭 메타라 본인 어조와 맞는지 직접 검토. 자료 근거에 단서가 있는 표현으로만 채웠지만 본인 톤이 아닐 수 있음
- **무한스크롤 배운 점 마지막 bullet**: "측정 → 가설 → 대안 비교 → 채택 사이클 반복" 표현이 본인 어조인지
- **ai-safety-net 의도**: "정적분석 + 2단계 훅" 부제가 구성 어필. 무한스크롤은 "(PR 4 시리즈)" 양적 어필. 결이 달라 통일할지 판단 필요

### 3. 보류된 결정

- **eslint 룰 갯수 표기**: 현재 "eslint 룰 직접 설정"으로 갯수 뺐음. 본인이 다시 어필하고 싶으면 옵션 정리:
  - 룰 단위 카운트(off 제외): 22개 (base 20 + examples 2, 커스텀 룰 1개 포함)
  - 셀렉터·패턴 단위로 펴서: 약 40개 (naming-convention 9 selector + no-restricted-imports 2 path + no-restricted-syntax 10 selector 등 포함)
  - 직접 도입 플러그인/config: 6종 (@tanstack/eslint-plugin-query, eslint-plugin-react-hooks/refresh/react, eslint-config-next, typescript-eslint recommendedTypeChecked)
  - 추천 표기: "직접 설정한 ESLint 룰 22개(커스텀 룰 1개 포함) + 직접 도입한 플러그인 6종" — 정직성·어필 균형
- **"이거 하지마라고 프롬프트 갈겨도 계속 함" 시행착오**: ai-safety-net 첫 bullet에 살린 상태. 톤이 강하다고 느껴지면 부드럽게 다듬기

## 자료 출처

본문에 등장하는 수치·기술 결정은 모두 아래 자료에 근거.

- DC `docs/infinite-scroll/step1.md` — CSR/SSR/Streaming 비교, 1.04s → 0.01s
- DC `docs/infinite-scroll/step2.md` — srcset + Cloudinary, 773kB → 20.2kB
- DC `docs/infinite-scroll/step3.md` — IntersectionObserver, staleTime: Infinity, React.memo 600 → 120
- DC `docs/infinite-scroll/step4.md` — 무한 재요청 차단
- DC `docs/vision/ai-era-growth.md` — 정적분석/테스트/AI 코드리뷰 비용 순서, 19개+ 커스텀 룰 PR #11 언급
- MP `docs/static-checking.md` — --max-warnings 0, 2단계 훅 구조
- MP `docs/patterns/rendering/ErrorHandling.md` — 에러 처리(이번엔 미사용)

## 미해결 / 미수행

- 에러처리 항목은 MP 예제 페이지 부재로 제외함. 추후 MP에 에러처리 예제 PR 만들면 세 번째 항목으로 추가 가능
- frontmatter는 붙이지 않음 — write-refine을 정식 호출하려면 type/subtype/audience/purpose/key_message 5개 필수 필드를 수동으로 추가해야 함
