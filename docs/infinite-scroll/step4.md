# Step 4. 에러 핸들링 — 재요청 방지, 빈 목록

## 목차
1. [렌더링 전략 — CSR → SSR → Streaming](./step1.md)
2. [이미지 최적화 — srcset + CDN 리사이징](./step2.md)
3. [무한 스크롤 — IntersectionObserver, staleTime/gcTime, React.memo](./step3.md)
4. **에러 핸들링 — 재요청 방지, 빈 목록** ← 현재 문서
5. [Virtual List — DOM 노드 94% 감소](./step5.md)

> 📎 [예제 페이지](https://monorepo-playground-examples.vercel.app/rendering/infinite-scroll) · [PR #9](https://github.com/developer-choi/monorepo-playground/pull/9)

---

## 1. 다음 페이지 로딩 실패 시 무한 재요청 방지

### 문제

스크롤 이벤트든 IntersectionObserver든, 무한 스크롤은 조건이 충족되면 자동으로 다음 페이지를 요청합니다. 다음 페이지 API가 실패하면 이 조건이 즉시 다시 충족되어, 짧은 시간 안에 `fetchNextPage`가 수십~수백 회 반복 호출될 수 있습니다. 무한 스크롤을 구현한다면 반드시 대비해야 하는 부분입니다.

```
fetchNextPage() → 500 실패
→ isFetchingNextPage: false (복귀)
→ hasNextPage: true (다음 페이지를 못 받았으니 여전히 true)
→ fetchNextPage() → 500 실패
→ (무한 반복)
```

### 해결

API 오류가 발생하면 두 가지를 처리합니다.

1. 추가 API 호출을 중단합니다. 모든 페이지를 다 불러왔을 때 더 이상 호출하지 않는 것처럼, 에러 발생 시에도 동일하게 중단합니다.
2. 사용자에게 게시글 목록을 더 불러오지 못했다는 안내를 표시합니다.

```typescript
// useInfiniteScroll 훅 내부
const enabled = hasNextPage && !isFetchingNextPage && !isError;
```

## 2. 빈 게시글 목록

게시글이 0건일 때 빈 그리드만 보이면 사용자는 로딩이 덜 된 건지, 진짜 없는 건지 구분할 수 없습니다. "게시글이 없습니다." 메시지를 표시하여 현재 상태를 명확히 안내합니다.
