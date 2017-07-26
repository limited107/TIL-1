# React는 무엇인가?

## 뷰 레이어
React는 일반적으로 MV 패턴에서 뷰 레이어만 담당합니다. React에선 어떤 형태의 모델이 사용될것인지에 대한 가정을 하지 않으므로 아무 라이브러리나 사용해도 무방합니다.
작은 앱이라면 굳이 사용하지 않아도 상관없습니다.

## 컴포넌트를 통한 뷰 작성
React는 Reactive 한 단방향의 데이터 흐름을 가지고 있습니다. Reactive하다는 것은 상태(state)가 바뀌면 상태에 의존하는 뷰도 함께 업데이트된다는 것이며, 단방향 데이터 흐름이라는 것은 한 방향으로 데이터가 흐른다는 것입니다. 데이터는 상위 컴포넌트(Parent)에서 하위 컴포넌트(Children)로 흐르게 되어 있는데, 이 데이터는 React에서 prop이라고 지칭되며, JSX에서는 HTML의 attribute처럼 작성됩니다.

#### 컴포넌트 설계

예를 들어, 쇼핑몰의 쇼핑 카트를 React로 만든다고 생각해 봅시다.

```javascript
var ShoppingCart = React.createClass({
    render () {
        return <div>
            <ShoppingItem name="kimchi" available={true} />
            <ShoppingItem name="rice" available={true} />
            <ShoppingItem name="curry" available={false} />
        </div>
    }
});

// 참고: 기존 html 엘리먼트가 아닌 한, 모든 커스텀 컴포넌트들의 이름은 대문자로
// 시작해야 하며 지켜지지 않으면 invariant 에러가 발생합니다.
var ShoppingItem = React.createClass({
    render () {
        return <div>
            <div>
                상품명: {this.props.name},
                구입가능: {this.props.available ? '가능' : '불가능'}
            </div>
        </div>
    }
});
```

컴포넌트 위계와 데이터 흐름을 설명하기 위해, 정말로 기본적인 React 컴포넌트들을 두가지 만들었습니다. 컴포넌트는 <ShoppingCart /> -> <ShoppingItem /> 형태의 부모-자식 형태의 관계를 맺게 되며, 더 많은 데이터를 표현해야 할 경우 더 다양하고 작은 컴포넌트들로 쪼개야 할 수도 있을 것입니다.

일반적인 쇼핑몰의 쇼핑 카트들에는 구매하고 싶은 수량을 써 줄 수도 있고, 삭제 버튼을 가지고 있기도 합니다. HTML에서 그러듯, <ItemQuantity /> <ItemDeleteButton />와 같은 컴포넌트들은 당연히 ShoppingItem보다 낮은 위계에 작성해주면 됩니다. (번외: React는 단방향 데이터 플로우를 가지고 있다고 했는데, 그러면 수량과 삭제 버튼은 어디에 어떻게 값을 전달해야 할까요? 이는 글의 뒤에 가서 다룹니다)

컴포넌트들이 재사용을 위해 만들어졌다는 점을 생각해 볼 필요도 있습니다. 가령, <ItemDeleteButton /> 에 모든 뷰를 표현하는 것보다는, 베이스 버튼 컴포넌트 <Button />를 만들어 전체 페이지들에 나오는 버튼들의 표현 로직, 인터랙션 로직을 추상화할 수도 있습니다. 하지만 전체 페이지에 버튼이 몇 개 없고, 서로 생긴 것도 많이 다르고, 앞으로 버튼을 추가할 일이 없다면 과도한 일반화는 무의미할 것입니다. 이런 점들을 고려하여 컴포넌트들을 설계할 필요가 있습니다. (팁: 추상화할 수 없는 버튼들이 페이지에 널려 있다면, 디자이너와 대화의 시간을 가져봅시다)

코드를 보면, 데이터는 <ShoppingCart />에서 <ShoppingItem />으로 흐르는 것이 명백합니다. "kimchi", "rice", "curry" 같은 스트링은 <ShoppingCart /> 상위 컴포넌트의 render() 메서드에서 나타나는데, <ShoppingItem /> 에서는 표현되지 않고 있습니다. <ShoppingItem name="kimchi" available={true} /> 에서 HTML attribute처럼 써준 것이 하위 컴포넌트로 데이터를 주입하는 것입니다. 이 약간 익숙한 듯하지만 생소한 느낌의 문법은 React에서 사용하는 JSX라는 치환 문법이며, 자세한 것은 나중에 다룰 예정입니다만 공식 문서를 읽어보면 사용법을 금방 이해할 수 있을 것입니다.

또한, React는 컴포넌트 라이프사이클 훅을 제공합니다. 컴포넌트가 마운트(Parent에서 React.CreateElement 함수 호출로 새 ReactElement를 만들거나, React.render로 DOM 컨테이너 위에 Render되는 순간)되는 순간과 언마운트되는 시점, 업데이트되는 시점 등을 표현하는 함수를 컴포넌트 스펙 오브젝트에서 메서드 형태로 구현해 주면 됩니다. 모든 React 컴포넌트들은 HTML의 DOM Event Level 0 이벤트 핸들러 같은 onClick 등의 prop에 함수를 넘김으로써 인터랙션을 표현할 수 있습니다. 이에 대해서는 나중에 더 자세히 다룹니다.


## Virtual DOM과 Reconciliation

React는 HTML Element들을 Virtual DOM을 이용해 표현한다고 했습니다. Virtual DOM은 가상의 HTML Element들을 가지고 있다가, (재)렌더링하면 필요한 부분만 업데이트(DOM 조작)하는 방식입니다. Virtual DOM은 사실 그 개념을 알고 나면 React의 구현 디테일에 불과합니다만, key prop 등 차후 여러 API와 연관이 있으므로 개념을 제대로 알고 있을 필요가 있습니다.

React가 Virtual DOM을 만든 이유는 Always re-render on update 정책을 가지고 있기 때문입니다. jQuery 등을 이용하여 애플리케이션을 작성하면 모델이 업데이트되었을 때, 셀렉터 API를 이용해 필요한 컴포넌트들만 업데이트를 해주는 코드를 작성할 때가 많습니다. 하지만 React에서는 작성자가 원하는 결과물을 선언적으로 작성하므로, 그런 코드를 짤 필요가 없다고 위에서 언급했습니다. 이것이 가능해지려면, 일부보다는 해당 영역을 모두 새로 그리는 것이 바람직합니다.

하지만 jQuery로 모든 영역을 새로 그리게 구현하면 단점이 있기 마련입니다. 위의 쇼핑 카트 예제에서 ‘가능/불가능’을 보여주는 컴포넌트만 jQuery 셀렉터 API로 업데이트하다가, 쇼핑카트 전체를 지우고 새로 그리게 구현을 변경했다고 생각해봅시다.

작은 크기의 HTML 엘리먼트 트리라면 아무것도 아니겠지만, 카트 컴포넌트의 내용이 많아질수록 문제가 커집니다. 일단 속도가 늘어나는 엘리먼트의 양에 비례하여 더 많은 동기 DOM 조작 오퍼레이션을 수행해야 하므로 느려질 수밖에 없습니다. 그리고 사용성 문제들이 발생합니다. 가령 ‘수량’ 을 보여주는 인풋 박스가 있고 거기에 키보드 커서가 올려져 있었다면 업데이트 후 엘리먼트가 지워지고 새로 쓰여졌을 시, 커서가 사라져 있게 되는 문제가 있습니다. 비슷하게, 사용자가 명시적으로 업데이트하지 않고 가지고 있던 (DOM state에만 존재하던) 정보들은 날아가버리게 됩니다.

React를 사용하면 이런 문제들은 많은 부분 해소됩니다. 가지고 있는 Virtual DOM 트리를 비교하면서 필요한 부분만 업데이트하기 때문입니다. 이것을 Reconciliation (비교조정)이라고 하며, 이 개념은 React 공식 문서에 잘 설명되어 있습니다. (React 팀의 Christopher Chedeau가 기고한 React의 diffing 알고리즘에 대한 글도 이를 잘 설명하고 있습니다) 비록 동기 DOM 배치 오퍼레이션을 매 업데이트 사이클마다 수행하는 것보다는 낫겠지만 위의 글을 보면 업데이트 전, 업데이트 후의 가상 엘리먼트 트리를 비교해야 하는 문제가 있습니다. 이 문제는 가장 최신의 알고리즘도 O(n^3)의 시간 복잡도를 가지고 있는 수준이므로, 여러 가지 휴리스틱을 통해 알고리즘의 복잡도를 O(n)까지 낮췄다고 설명하고 있습니다.

이런 React 렌더러의 최적화와 가상화 덕에, 우리는 뷰 업데이트 로직은 거의 신경 쓰지 않고 모델 데이터 관리와 결과물의 모양만 기술하면 됩니다. React의 Virtual DOM은 획기적인 아이디어이다 보니 다른 라이브러리들도 영향을 많이 받고 있는데, Ember.js에서도 이를 도입하였으며 virtual-dom이라는 별도의 구현체도 존재합니다. 또한, Virtual DOM을 넘어서 incremental update(증분 업데이트)를 표방한 구글의 Incremental DOM도 등장했습니다.

하지만 DOM 가상화에 따른 혜택들은 위에서 말한 것에 그치지 않습니다. 가상화라는 것은 기본적으로 브라우저 구현에 코드가 의존하는 것을 넘어선다는 것을 의미하며, 브라우저의 구현 디테일 차이들을 덜 신경 써도 된다는 장점이 있습니다. 이 덕분에 React Native 같은 프로젝트나, React Canvas 같이 최종 결과물이 DOM의 형태로만 쓰여지지 않는 프로젝트들이 등장할 수 있었습니다. React 팀에서는 이런 멀티 플랫폼 접근을 권장하기 위하여 최근 렌더러를 React에서 react-dom 패키지로 분리하고, React는 뷰 상태를 관리하는 상태 기계로 발전시켜 나가고 있습니다.