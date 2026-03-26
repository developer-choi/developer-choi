# Step 1. 렌더링 전략 — CSR → SSR → Streaming

## 목차
1. **렌더링 전략 — CSR → SSR → Streaming** ← 현재 문서
2. [이미지 최적화 — srcset + CDN 리사이징](./step2.md)
3. [무한 스크롤 — IntersectionObserver, staleTime/gcTime, React.memo](./step3.md)
4. [에러 핸들링 — 재요청 방지, 빈 목록](./step4.md)
5. [Virtual List — DOM 노드 94% 감소](./step5.md)

> 📎 [예제 페이지](https://monorepo-playground-examples.vercel.app/rendering/infinite-scroll) · [PR #6](https://github.com/developer-choi/monorepo-playground/pull/6)

---

## 렌더링 개선 1. CSR vs SSR Prefetch — Network Waterfall 비교

| CSR (useQuery) | SSR (prefetch) |
|---|---|
| <img width="1169" height="838" alt="image" src="https://github.com/user-attachments/assets/a7c99494-17bf-46b9-90c5-68bb41424d51" /> | <img width="1036" height="932" alt="image" src="https://github.com/user-attachments/assets/5252ad53-9e56-42bc-8b1e-e2d9f73445cc" /> |

[SSR prefetch 커밋](https://github.com/developer-choi/monorepo-playground/pull/6/changes/e7417882501978fa6365cf3342798eb92aface2f)을 통해 개선한 내역은 다음과 같습니다.

1. **API waterfall 제거**: CSR에서는 HTML → JS 다운로드 → `board` API 호출 순서로 3단계 waterfall이 발생했으나, 서버에서 데이터를 prefetch하여 클라이언트의 API 호출을 제거했습니다.
2. **이미지 로딩 시점 앞당김**: CSR에서는 API 응답 이후에야 이미지 URL을 알 수 있었으나, SSR에서는 HTML에 `<img>` 태그가 포함되어 파싱 즉시 이미지 다운로드가 시작됩니다.
3. **초기 렌더링 콘텐츠 제공**: CSR의 HTML은 빈 껍데기였으나, SSR은 게시글 목록이 포함된 HTML을 내려보내 JS 실행 전에도 콘텐츠가 표시됩니다.

### 의사결정 과정

게시글 목록은 무한 스크롤이 필요하므로 Client Component로 구현해야 합니다. 하지만 데이터를 클라이언트에서만 fetch하면 waterfall이 발생합니다. 이를 개선하기 위해 서버에서 첫 페이지 데이터를 prefetch하여 HTML에 포함시키는 방식을 선택했습니다.

서버 부하가 늘어나는 단점이 있지만, 사용자 디바이스 성능은 통제할 수 없는 반면 서버 스펙은 필요에 따라 스케일링으로 대응할 수 있기 때문에 합리적인 트레이드오프라고 판단했습니다.

## 렌더링 개선 2. SSR Blocking 문제와 Streaming 도입

SSR prefetch에는 한 가지 문제가 있습니다. API 응답이 느려지면 서버가 HTML 전체를 블로킹하여, 사용자에게 아무것도 보이지 않는 빈 화면이 노출됩니다.

이를 검증하기 위해 [API에 1초 딜레이를 추가한 커밋](https://github.com/developer-choi/monorepo-playground/pull/6/changes/6d7eb197d954f52d941b5b7ceea09b44e6cf0a9e)으로 테스트하고, [Streaming 커밋](https://github.com/developer-choi/monorepo-playground/pull/6/changes/2c1339017a484449337b26a3073090906d60b124)을 통해 개선했습니다.

### 사용자 화면 비교

| SSR Blocking (Suspense 없음) | Streaming (Suspense 적용) |
|---|---|
| <img width="828" height="366" alt="ssr-block" src="https://github.com/user-attachments/assets/a3e5ec74-5b91-416b-8ff2-714e7f3e888c" /> | <img width="1411" height="450" alt="image" src="https://github.com/user-attachments/assets/8c4c1a13-4d90-4e1a-9f1d-7009e50ab221" /> |

- **SSR Blocking**: API 응답까지 1초간 완전한 빈 화면. 사용자는 페이지가 고장났다고 느낍니다.
- **Streaming**: 서버가 헤더 + 스켈레톤을 즉시 전송하고, API 응답이 완료되면 실제 콘텐츠를 스트리밍합니다.

### Network Waterfall 비교

| SSR Blocking | Streaming |
|---|---|
| <img width="954" height="411" alt="image" src="https://github.com/user-attachments/assets/ffb9c413-aafb-4ed6-8b5e-d64278c5f739" /> | <img width="812" height="533" alt="image" src="https://github.com/user-attachments/assets/04b054a0-8b84-4968-86c5-5998350c378c" /> |

- **SSR Blocking**: HTML 문서 응답 자체가 1.16s로, 그 동안 CSS/JS/이미지 등 모든 리소스가 대기합니다.
- **Streaming**: CSS와 이미지가 즉시 로딩을 시작합니다.

### Timing 상세 비교

| SSR Blocking | Streaming |
|---|---|
| <img width="630" height="314" alt="ssr-1s-detail" src="https://github.com/user-attachments/assets/0bf213cc-997c-49b6-b0fe-ef75fdcef61b" /> | <img width="628" height="315" alt="streaming-detail" src="https://github.com/user-attachments/assets/1cf9753e-3af6-409c-9fac-1e8899c5f373" /> |

| | SSR Blocking | Streaming |
|---|---|---|
| Waiting for server response | **1.04s** (서버가 API 응답을 기다리며 블로킹) | **10.87ms** |
| Content Download | 5.20ms | **1.01s** (점진적 스트리밍) |

같은 1초 API 딜레이에서, Blocking은 서버가 응답 자체를 1초간 보류하는 반면 Streaming은 즉시 응답을 시작하고 Content Download 단계에서 데이터를 점진적으로 내려보냅니다.

### Pages Router와의 비교

Pages Router 시절에는 Streaming이 지원되지 않아, API 응답이 느릴 경우 빈 화면을 피하기 위해 SSR 대신 CSR로 돌려 로딩 UI라도 빨리 보여주는 것이 일반적이었습니다. App Router에서는 Suspense 기반 Streaming이 내장되어 있어 SSR의 이점을 유지하면서도 블로킹 없이 즉시 셸을 내려보낼 수 있으므로, 더 이상 그런 트레이드오프가 필요하지 않습니다.
