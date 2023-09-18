> 이 시리즈는 저만의 FE 컨벤션, 코드 스타일 등을 만들고자 Toss의 SLASH 영상과 기술 블로그 등을 참고하여 만들었습니다.

## 👋🏼 참고자료 소개
SLASH 22에서 한재엽님께서 발표하신 ["Effective Component 지속 가능한 성장과 컴포넌트"](https://youtu.be/fR8tsJ2r7Eg?si=nt8uNO0VMSUBU32P) 영상을 분석하였습니다.

### 짧은 후기

주니어 FE 개발자로서 회사에서 프로젝트할 때, 가장 어려웠던 부분은 바로 **"컴포넌트 적절히 나누기"** 였습니다. 재엽님의 영상을 통해서 컴포넌트를 **"적절히"** 나누는 기준을 배웠고 다음 프로젝트 때, 잘 써먹어 보려고 합니다. 좋은 내용 감사합니다🙇🏻‍♂️

---

## 📌 내용 정리

### [0] FE 엔지니어라면, "변경"을 오히려 반겨야 하는 이유
- FE 엔지니어는 상대적으로 **수많은 "변경"을 마주하게 된다**.
  - **Why?** 제품을 사용하는 사용자와 가장 밀접한 직무이기 때문이다.
  - **[모든 사용자가 정말 완벽하게 제품을 사용]**한다면, "변경"은 없을 것이다.
  

- 그렇다면, **[모든 사용자가 정말 완벽하게 제품을 사용]**이 가능할까?
  - 불가능에 가깝다고 자신 있게 말할 수 있다.
  

- 오히려 FE 엔지니어로서 **"변경"은 반가울 일**이라고 생각이 든다.
  - 사용자가 관심을 갖고 사용하고 있다는 반증이고
  - 내가 만드는 이 제품이 계속해서 **더 나은 방향으로 "성장"**하고 있다는 뜻이기 때문이다.


### [1] Headless UI 기반의 추상화

#### 컴포넌트는 크게 3가지 역할을 한다.
  1. **데이터 관리**
     : 외부에서 주입받은 데이터 or 상태와 같은 내부 데이터 관리
  2. **UI 그리기**
     : 관리하는 데이터로 사용자에게 어떻게 보일지 정의
  3. **사용자와 상호작용**
     : 클릭 등 사용자와 어떻게 상호작용할지 정의
     
#### Headless UI 란?

- UI 부분(스타일링)을 과감하게 제외하고 **상태나 데이터와 관련된 부분을 따로 다루는 패턴**이다.
- **한 가지 문제에만 집중**하기에 **더 많은 곳에서 사용**할 수 있다.
- 의존성을 분리함으로써 **다른 변경으로부터 크게 영향을 받지 않게 된다**.

### [2] 컴포넌트 조합하기

![](https://velog.velcdn.com/images/chooble/post/2adbc046-0e12-4c7a-bf41-b9bfc7323797/image.png)

#### "위의 UI를 그리기 위해서 코드를 어떻게 작성할까요?"


**[기존 코드]**
```javascript
function ReactFramerworkSelector({ defaultValue }) {
  const [isOpen, openk, close] = useBoolean();
  const [selected, change] = useState(defaultValue);

  return (
    <>
      <InputButton label="React Framework" value={selected} onClick={open} />
      {isOpen ? (
        <Options onClose={close}>
          {options.map((value) => {
            return (
              <Button
                selected={selected === value}
                onClick={() => change(value)}
              >
                {value}
              </Button>
            );
          })}
        </Options>
      ) : null}
    </>
  );
}
```
- 만약 label="React Framework"에서 label이 바뀐다면?
- "InputButton"이 아니라, 다른 기능이 필요하다면?
- Selct 하는 부분을 다른 컴포넌트에서 사용하고 싶다면?

#### 위의 질문들에 대해서 유연하게 대응하기 어려운 코드입니다.



**[개선]**
```javascript
// Select 라는 함수 만들어서 컴포넌트 분리
function Select({ label, trigger, value, onChange, options }: Props) {
	return (
		<Dropdown label={label} value={value} onChange={onChange}>
			<Dropdown.Trigger as={trigger} />
			<Dropdown.Menu>
				{options.map(option => (
					<Dropdown.Item>{option}</Dropdown.Item>
				})}
			</Dropdown.Menu>
	);
}

// Select 함수가 필요한 컴포넌트에서 활용
function FrameworkSelect() {
  const {
    data: { frameworks },
  } = useFrameworks();
  const [selected, change] = useState();
  
  return (
    <Select
    	trigger={<InputButton value={selected} />}
		value={selected}
		onChange={change}
		options={frameworks}
	/>
  );
}
```
- 변경에 대해서 보다 유연하게 대응하기 위해서 **"추상화"**를 잘해준다.
- isOpen, trigger, menu 등 공통적인 부분을 뽑아내 **Select** 라는 함수를 만든다.
- **Select** 함수를 **FrameworkSelect** 라는 컴포넌트에 활용한다.


### [3] 도메인 분리하기

위에서 구현한 FrameworkSelect 컴포넌트에는 **"Framework" 라는 특정 도메인이 포함되어 있기 때문에 재사용하는 데 한계**가 있다.

```javascript
// 특정 도메인을 제거하고 일반적인 역할을 할 수 있도록 인터페이스 변경한 코드
interface Props {
  options: Array<{ label: string }>;
  value?: string[];
  onChange?: (selecteds: string[]) => void;
  valueAs?: (value?: string[]) => string;
}
```

- 도메인 맥락을 제거하다 보면, 일반적인 이름으로 바뀌게 된다.
- 컴포넌트 인터페이스는 일반적일수록 이해하기 쉽다.
  - 사람들은 배경지식으로 무언가를 해석한다.
  - 일반적인 단어로 표현해야 의도를 드러내기 쉽다.
- 비즈니스 로직은 스스로 처리하되 UI 로직을 위임하는 방식으로 볼 수 있다.
- 표준 = 이해하기 쉬운 인터페이스

---

## 🚀 실전에 써먹을 것

### 컴포넌트 만들 때, 고려할 부분
> 구현하는 것보다 제대로 고민하는 것이 훨-씬 중요하다는 것 명심하자.

1. **컴포넌트 만드는 이유** 고민하기
   - Q. 복잡도를 낮추기 위한 것인가?
   - Q. 재사용을 하기 위함인가?
   - Q. 꼭 분리를 해야 하는가?


2. **어떠한 단 하나의 기능**을 할지 정하기

3. **도메인 맥락을 제거**하여 최대한 일반적으로 만들기

4. 한 번 잘 만든 컴포넌트는 **미래의 나와 동료들에게 큰 도움이 된다는 생각** 항상 인지하기

5. **UI에 속지 않고 데이터 흐름을 파악**하여 쉽게 구조를 잡기

### "변경"을 오히려 반기는 마음가짐 갖기

- 저의 궁극적인 목표는...
단순히 개발만 잘하는 사람이 아니라, **Problem Solver**로서 특정 문제를 잘 풀어내는 사람이 되는 것입니다.

- 그렇기에 "변경"은 제가 만들고 있는 제품의 성장을 뜻한다는 것이고 이는 문제를 조금씩 잘 풀어내고 있다는 것을 의미합니다.

- 따라서, "변경"을 반가워하고 오히려 기분 좋게 받아들이는 자세를 갖고자 합니다.

---
## 📚 레퍼런스
- [Headless UI Library란?](https://jbee.io/react/headless-concept/)

