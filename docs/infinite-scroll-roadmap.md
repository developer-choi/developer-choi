# 📚 무한스크롤 완전정복 커리큘럼

## 같이 정리해야하는 자료
- [포카마켓](https://github.com/developer-choi/pocamarket-frontend-task/pull/4)
- [useInfiniteQuery, Paginated Query](https://docs.google.com/document/d/1T73VImuRBctQUfQzwBx6yljP9s7LvKSnWak3neSobGE/edit?tab=t.0#heading=h.emc0jaqr8xzz)
- [Infinite Scroll](https://docs.google.com/document/d/1IeMIvPc-18TKEscvuRYmktziMxieeKW_wJGB779nOXg/edit?tab=t.0#heading=h.6thycnwoai9)
- [페이지 최신화 하는 방법 fetch() / RQ](https://docs.google.com/document/d/1ir7P3J1WbsIrqa2vNQ1Co39QYmkV4ef6MHcluH6m90s/edit?tab=t.0#heading=h.fsscmueprjg8)
- [Virtual List](https://docs.google.com/document/d/1z2U7x3FFcdUNTtFdq-t5kg6XTE7tptUaCAxMgbt7VfE/edit?tab=t.0#heading=h.fsj2tgrvrdua)

## Step 1. 무한스크롤이란 무엇이고 왜 필요한가?
탐색을 가장 쉽게할 수 있는 방법이라서.
(기타 UX 내용 좀더 자세히)

### 다룰 내용
- UX 관점: 페이지네이션 vs 무한스크롤
- 인지 부하 감소 (클릭 없이 스크롤만)
- 몰입감, 발견의 즐거움
- 단점도 언급 (끝이 없는 느낌, 푸터 접근 어려움)
- 실제 서비스 사례 (인스타그램, 트위터 등)
- 언제 무한스크롤을 써야 하고, 언제 쓰면 안 되는지



무한 스크롤 구현하려면 스크롤 끝을 감지하는 방식으로 구현해야하고, 이건 step 2에 작성하겠다.

---

## Step 2. 스크롤 끝 감지 방법

### 다룰 내용
- **Scroll Event + Throttle** vs **Intersection Observer**
- 장단점 비교표
- 내부 구현 원리 (이벤트 루프, 리플로우/리페인트)
- 성능 비교 실험 (크롬 Performance 탭)
- 극한의 성능 차이를 보여주는 데모 페이지 제작
- 왜 이런 차이가 나는가? 브라우저 렌더링 파이프라인 관점

---

## Step 3. 데이터 누적해서 저장하기

const [list, setList] = useState([]);

useEffect(() => {
setList();
setList();
setList();
계속 누적하는게 필요함.
}, []);

### 다룰 내용
- 무한스크롤의 시작: 데이터를 계속 누적해서 저장
- 일반 `useState`로 구현하면 이런느낌.
- **Tanstack Query**의 `useInfiniteQuery` 등장
- `pages` 배열 구조 이해
- `getNextPageParam` 작동 원리
- 코드 예제

---

## Step 4. 상태별 UI 제어하기

### 다룰 내용
- 왜 상태별 UI가 필요한가? (UX 관점)
- `isLoading`, `isFetchingNextPage`, `hasNextPage`
- 각 상태의 의미와 사용 시점
- Skeleton UI, Spinner, "더 이상 없음" 메시지
- 코드 예제 및 실전 적용

---

## Step 5. 데이터 캐싱 & 리페칭 전략

### 다룰 내용
- 다음 페이지 갔다가 빠르게 돌아왔을 때 데이터가 남아있어야 함
- **기본 동작의 문제점**: 10개 페이지 한방에 리페칭 → 서버 부하 심함
- 해결책: `staleTime: Infinity` + `gcTime: 0` 조합
- 왜 이 조합이 Best Practice인가?
- 트레이드오프 (데이터 신선도 vs 서버 부하)

---

## Step 6. 무한스크롤 중 데이터 추가/삭제/수정 ⭐

### 문제 상황
- 3페이지까지 스크롤해서 보는 중
- 2페이지에 있던 내 댓글을 삭제/수정
- 어떻게 처리할 것인가?

페이지네이션이었으면 아주 편했음. 3페이지 보고있으면 3페이지 최신화 해버리면 끝이니까. 남들도 그렇게 하고.
어? 이부분 논리 부족하네. 페이지네이션이어도 추가했으면 1페이지 가야하고
무한스크롤에서 대댓글인 경우나 이 말이 통할듯.

### 선택지와 트레이드오프

#### 1. 전체 페이지 리패칭 (1,2,3 페이지 모두)
```typescript
queryClient.invalidateQueries(['comments'])
```
- ✅ 장점: 가장 정확한 최신 데이터
- ❌ 단점:
    - 서버 부하 (3번 요청)
    - 스크롤 위치 깨짐 가능성
    - 사용자가 보던 내용이 순간 사라졌다 나타남

#### 2. 현재 페이지만 리패칭 (3페이지만)
- ❌ 완전히 잘못된 방법
- 이유: 그 사이 데이터 추가로 페이지네이션 밀림
- 예: 2페이지 끝에 있던 댓글이 3페이지로 밀려옴

#### 3. Optimistic Update (낙관적 업데이트)
```typescript
queryClient.setQueryData(['comments'], (old) => {
  // pages 배열 순회하며 해당 댓글만 수정/삭제
})
```
- ✅ 장점:
    - 즉각 반응 (네트워크 왕복 없음)
    - 스크롤 위치 유지
- ❌ 단점:
    - 실패 시 롤백 처리 복잡
    - 다른 사람의 변경사항은 반영 안 됨

#### 4. Hybrid: Optimistic + 백그라운드 리패칭
```typescript
// 즉시 화면 업데이트
queryClient.setQueryData(...)

// 백그라운드에서 전체 리패칭 (조용히)
queryClient.invalidateQueries(['comments'], { 
  refetchType: 'none' // 다음 포커스/마운트 시
})
```

#### 5. 부분 무효화 (Page-level Invalidation)
```typescript
// 해당 댓글이 속한 페이지만 찾아서 무효화
queryClient.invalidateQueries({
  queryKey: ['comments'],
  refetchPage: (page, index) => index === targetPageIndex
})
```

### 실무 Best Practice

**케이스별 전략:**

**삭제:**
```typescript
// 1. Optimistic Delete (즉시 화면에서 제거)
queryClient.setQueryData(['comments'], removeComment)

// 2. 해당 페이지만 백그라운드 재검증
setTimeout(() => {
  queryClient.invalidateQueries({
    queryKey: ['comments'],
    refetchPage: (page, index) => index === targetPageIndex
  })
}, 1000)
```

**수정:**
```typescript
// Optimistic Update만으로 충분
// (다른 사람 데이터와 충돌 없음)
queryClient.setQueryData(['comments'], updateComment)
```

**추가:**
```typescript
// 맨 위에 추가하는 경우 (일반적 댓글)
queryClient.setQueryData(['comments'], (old) => {
  old.pages[0].data.unshift(newComment)
})

// 또는 첫 페이지만 리패칭
queryClient.invalidateQueries({
  queryKey: ['comments'],
  refetchPage: (page, index) => index === 0
})
```

### 실전 서비스 분석 🔍

#### 분석할 서비스
- **유튜브 댓글**: 댓글 삭제/수정 시 어떻게 동작?
- **네이버 카페**: 댓글 추가/삭제 시 처리 방식
- **인스타그램**: 좋아요/댓글 실시간 반영 방식
- **트위터(X)**: 트윗 삭제 시 타임라인 동작
- **페이스북**: 무한스크롤 중 새 게시물 추가 알림

#### 비교 항목
1. **즉시 반영 여부** (Optimistic Update 사용?)
2. **백그라운드 리패칭** 유무
3. **스크롤 위치 유지** 여부
4. **로딩 인디케이터** 표시 방식
5. **에러 처리** (롤백? 에러 메시지?)

### 실험/증명할 내용

1. **각 방식별 네트워크 탭 비교**
    - 전체 리패칭 시 N개 요청
    - Optimistic 시 0개 요청
    - 부분 무효화 시 1개 요청

2. **스크롤 위치 유지 비교**
    - 전체 리패칭 시 깜빡임
    - Optimistic 시 부드러움

3. **동시 편집 시나리오**
    - 내가 수정하는 동안 다른 사람이 댓글 10개 추가
    - 각 전략별 결과 차이

---

## Step 7. Virtual Scrolling으로 성능 개선

### 다룰 내용

#### 문제: 왜 버벅이는가?
- DOM이 많으면 버벅이는 이유
- 크롬 개발자도구 Performance 탭으로 증명
- 1000개 DOM vs 10000개 DOM 렌더링 비교
- 리플로우/리페인트 비용
- 메모리 사용량

#### 해결책: Virtual Scrolling
- **원리**: 보이는 영역만 렌더링
- 전체 10000개 데이터 중 실제 DOM은 10개만 유지
- 스크롤 시 DOM 재사용 (재배치)

#### 라이브러리 비교
- `react-window`
- `react-virtualized`
- `@tanstack/react-virtual`
- `react-virtuoso`

**비교 기준:**
- 번들 사이즈
- API 사용 편의성
- 동적 높이 지원
- 무한스크롤 통합
- 성능

#### 내부 구현 분석
- 어떻게 보이는 영역을 계산?
- 스크롤 이벤트 최적화
- DOM 재사용 메커니즘
- 동적 높이 처리 방법

#### 실전 적용
- Tanstack Query + Virtual Scrolling 결합
- 이미지 lazy loading과 조합
- 성능 측정 (Before/After)

---

## 학습한 것
### 1. 브라우저 API 깊은 이해
- Intersection Observer와 Scroll Event의 내부 동작 원리
- 각각의 성능 차이가 발생하는 이유를 브라우저 렌더링 파이프라인 관점에서 설명 가능
- Chrome DevTools Performance 탭으로 실제 성능 측정 및 분석 능력

### 2. Tanstack Query 고급 활용
- useInfiniteQuery의 pages 배열 구조 완전 이해
- staleTime: Infinity + gcTime: 0 조합을 통한 서버 부하 최적화
- refetchPage 옵션으로 페이지별 선택적 리패칭 구현
- Optimistic Update 패턴을 활용한 즉각 반응 UI 구현

### 3. 실전 데이터 변경 처리 전략
- 무한스크롤 중 CRUD 발생 시 5가지 처리 방법의 트레이드오프 이해
- 삭제/수정/추가 각 케이스별 Best Practice
- 스크롤 위치 유지하면서 데이터 동기화하는 방법

### 4. 대용량 리스트 렌더링 최적화
- DOM 개수와 성능의 상관관계 (정량적 측정)
- Virtual Scrolling의 작동 원리 (보이는 영역만 렌더)
- react-window, tanstack-virtual 등 라이브러리 비교 및 선택 기준
- 라이브러리 내부 구현 분석 (DOM 재사용 메커니즘)

### 5. 주요 서비스 구현 전략 분석 능력
- 유튜브, 네이버, 인스타그램 등의 실제 구현 방식 파악
- 각 서비스가 선택한 트레이드오프 이해
- 상황에 맞는 최적의 전략 선택 능력

### 6. UX 중심 사고
- 무한스크롤 vs 페이지네이션 의사결정 근거
- 로딩/에러/빈 상태별 적절한 피드백 설계
- 사용자 체감 성능을 높이는 UI 패턴들