> 이 시리즈는 저만의 FE 컨벤션, 코드 스타일 등을 만들고자 Toss의 SLASH 영상과 기술 블로그 등을 참고하여 만들었습니다.

## 👋🏼 참고자료 소개
SLASH 21에서 박서진님께서 발표하신 ["프론트엔드 웹 서비스에서 우아하게 비동기 처리하기"](https://youtu.be/FvRtoViujGg?si=hxyzn8sjIZdMB10i) 영상을 분석하였습니다.

### 짧은 후기

마지막에 더 공부하면 좋을 키워드까지 던져주시다니... 아주 알차네요🥹 감사합니다 서진님🙇🏻‍♂️ 프로젝트에 써먹을 부분들이 많은 것 같습니다. (해피~😊)

저는 깊게 생각하지 않고 로딩 & 에러 상태를 내부에서 처리하는 것을 너무 당연하게 받아들이고 있었는데, 확실히 좋은 코드를 보고 나니 제가 적었던 코드들이 지저분했다는 생각이 드네요. 비동기 처리할 때, Suspense 기능을 꼭 써보려고 합니다.

---

## 📌 내용 정리

### [1] 웹 서비스에서 가장 다루기 어려운 부분은?

10여 년 전 jQuery와 같은 라이브러리를 쓰면서 **명령형으로 프로그래밍**을 하다가 React/Vue.js와 같이 **선언적인 프로그래밍을 지원**하는 프레임워크들이 나오면서 개발자들이 신경 써야 하는 부분들이 많이 줄게 되었다.

그럼에도 가장 어려운 부분을 꼽자면, **"비동기 프로그래밍"** 이다.

### [2] 보통 FE 웹 서비스에서 비동기 처리
```typescript
function Profile() {
  const foo = useAsyncValue() => {
    return fetchFoo();
  }
  
  if (foo.error) return <div>로딩 실패</div>
  if (!foo.data) return <div>로딩 중</div>
  return <div>{foo.data.name}님 안녕하세요!</div>
```
- 성공하는 경우와 실패하는 경우가 섞여서 처리되었다.
- 실패하는 경우에 대한 처리를 외부에 위임하기 어렵다.
- React의 비동기 처리는 어렵고 복잡하다.
  - 성공하는 경우에만 집중해 컴포넌트 구성하기 어렵다.
  - 2개 이상의 비동기 로직이 개입할 때, 비즈니스 로직을 파악하기 점점 어려워진다.

### [3] Hooks와 Suspense의 비슷한 점
**[Hooks]**
: Hooks은 선언적인 API를 활용하여 컴포넌트를 감싸는 React 프레임워크가 실제 작업을 대신 수행해 준다.
- useState : 상태 사용을 선언
- useMemo : 메모이제이션 사용을 선언
- useCallback : 콜백 레퍼런스 보존을 선언
- useEffect : 부수 효과 발생을 선언

+) Class 컴포넌트에서는 컴포넌트의 라이프 사이클에 맞춰 다양한 작업을 명령형으로 해준다.


**[Suspense]**
: 자식 컴포넌트가 완료될 때까지 fallback을 표시할 수 있는 기능이다. (비동기를 동기적으로 바꿔줌)

- Recoil : Async Selector (or selectoFamily)
- SWR, React Query : {suspense: true}

컴포넌트에서는 비동기적인 리소스를 선언하고, 그 값을 읽어온다고 '선언'하기만 했다. 그러면 실제 로딩 상태나 에러 상태 처리는 컴포넌트를 감싸는 부모 컴포넌트가 대신해 주었다.

### [4] React Suspense for Data Fetching
: React 팀에서 로딩 & 에러 상태는 외부에 위임함으로써 동기적인 코드와 큰 차이가 없는 코드를 만들겠다는 목표로 **"React Suspense for Data Fetching"**을 만들었다.

```typescript
function FooBar() {
  const foo = useAsyncValue(() => fetchFoo());
  const bar = useAsyncValue(() => fetchBar(foo));
  
  return <div>{foo}{bar}</div>;
}
```
```typescript
<ErrorBoundary fallback={<MyErrorPage />}>
	<Suspense fallback={<Loader />}>
      <FooBar />
    </Suspense>
</ErrorBoundary>
```

- useAsyncValue 같은 hook을 만들 수 있는 Low-level API를 제공한다.
- 모든 실패할 수 있는 함수에 try-catch를 감싸지 않는 것처럼, Suspense를 일으키는 모든 컴포넌트에 Suspense나 ErrorBoundary를 붙여주기보다는 적당한 부분 단위로 에러와 로딩 상태를 한 번에 처리하게 된다.
- async-await 급으로 비동기 처리하면서 간단하고 읽기 편한 React 컴포넌트를 만들 수 있다.
  1. 성공한 경우에만 집중
  2. 로딩 상태와 에러 상태가 분리
  3. 동기와 거의 같게 사용할 수 있음
  

### [5] 대수적 효과(Algebraic Effects)
  - 어떤 코드 조각을 감싸는 맥락으로 책임을 분리하는 방식
  - 객체지향의 의존성 주입(DI), 의존성 역전(IoC)과도 유사
+) 더 공부 필요한 개념!!!

---
## 🚀 실전에 써먹을 것

### 공부할 키워드들
- React18 Concurrent Mode
- useTransition / useDeferredValue
- Micro Frontends (Module Federation)
- Argebraic Effects (대수적 효과)
- Fiber 아키텍쳐
- SOLID 원칙

  
### Recoil의 비동기 selector & selectorFamily 활용
```typescript
export const templateSetSelector = selectorFamily({
  key: '@messages/template-set',
  get: (no: number) => async () => {
    return fetchTemplateSet(no);
  },
});
```
```typescript
function TemplateSetDetails({ templateSetNo }: Props) {
  const templateSet = useRecoilValue(templateSetSelector(templateSetNo));
  /* 이 아래에서는 templateSet이 존재하는 것이 보장됨 */
}
```
```typescript
<Suspense fallback={<Skeleton />}>
  <TemplateSetDetails />
</Suspense>
```
- 비동기 리소스를 **selector 또는 selectorFamily**로 정의할 수 있다.
- **no**를 인자로 받아서 **fetchTemplateSet(no)**를 리턴한다.
- 비동기 호출을 하는 **컴포넌트(TemplateSetDetails)**를 적절히 **Suspense**로 감싸주면 된다.
- 이렇게 되면, 데이터가 준비되는 대로 하나씩 자연스럽게 보여줄 수 있어 **UX가 개선**된다.

+) Data fetching을 위해 React Query를 사용했었는데, **Recoil의 selectorFamily**를 활용해 봐야겠습니다.

---

## 참고 자료
- [마이크로프론트엔드 아키텍처](https://velog.io/@kylexid/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90)
- [리액트 Suspense의 대수적 효과](https://blog.mathpresso.com/algebraic-effects-of-react-suspense-157b49807ea0)
- [Micro Frontends를 위해 Module Federation 적용하기](https://blog.gangnamunni.com/post/saas-microfrontends)
- [React 18 Concurrent 로 UX 개선하기](https://velog.io/@seungchan__y/React-18-Concurrent-%EB%A7%9B%EB%B3%B4%EA%B8%B0)
- [프론트엔드와 SOLID 원칙](https://fe-developers.kakaoent.com/2023/230330-frontend-solid/)
- [Algebraic Effects for the Rest of Us by Dan Abramov](https://overreacted.io/algebraic-effects-for-the-rest-of-us/)
- [React 파이버 아키텍처 분석](https://d2.naver.com/helloworld/2690975)
