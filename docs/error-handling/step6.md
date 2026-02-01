# Step 6. 이벤트 핸들러 시점 에러 처리

> 이 글은 [Step 3. 에러 처리 원칙 세우기](./step3.md)에서 이어지는 글입니다.

## 문제 정의
버튼 클릭과 같은 사용자 상호작용 중에 API 오류 등의 에러가 발생할 수 있습니다.

이때, [Step 2. 에러 피드백 UX 설계](./step2.md)에서 정의한 대로 사용자에게 상황에 맞는 적절한 안내를 제공해야 합니다.

## 해결 전략

이벤트 핸들러 내부의 에러를 잡을 수 있는 유일한 방법은 **try-catch** 구문입니다.

기술적인 방법은 이미 정해져 있으므로, 기획 의도에 맞춰 어떤 UI로 피드백을 줄 것인지만 선택하면 됩니다.

- **모달**: 중요한 에러나 사용자 확인이 필수적인 경우
- **토스트**: 흐름을 방해하지 않는 가벼운 알림이 필요한 경우
- **인풋 메시지**: 특정 입력 필드의 오류를 알릴 경우

## 구현 방법

### 기본 구조

```typescript jsx
function Page() {
  const onClick = async () => {
    try {
      const response = await postProductApi(formData);
    } catch (error) {
      if (error instanceof UnauthorizedError) {
        openModal({
          title: '권한 없음',
          content: '상품 등록 권한이 없습니다. 관리자에게 문의하세요.'
        });
      } else {
        openModal({
          title: '오류 발생',
          content: '잠시 후 다시 시도해 주세요. 문제가 지속되면 고객센터에 문의하세요.'
        });
      }
    }
  };
  
  return (
    <button onClick={onClick}>상품 등록</button>
  );
}
```

하지만, 500 에러처럼, API마다 공통적으로 발생할 수 있는 오류가 존재합니다.

그래서 저런 코드를 이벤트 핸들러 마다 작성해야 하는 중복 코드 문제가 발생합니다.

이것은 [Step 7. 전역/공통/개별 에러 처리](step7.md)에서 다룹니다.

---

## 다음 단계

모든 곳에서 에러를 잡았지만, 코드가 중복되고 있습니다. 이걸 어떻게 깔끔하게 정리할까요?

[Step 7. 전역/공통/개별 에러 처리](step7.md)