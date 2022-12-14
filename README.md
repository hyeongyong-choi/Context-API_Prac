## 15장 Context API
Context API는 리액트 프로젝트에서 전역적으로 사용할 데이터가 있을 때 유용한 기능입니다. 리덕스, 리액트 라우터, styled-components 등의 라이브러리는 Context API를 기반으로 구현되어 있습니다.

### 15.1 Context API를 사용한 전역 상태 관리 흐름 이해하기
리액트 애플리케이션은 컴포넌트 간에 데이터를 props로 전달하기 때문에 컴포넌트 여기저기서 필요한 데이터가 있을 때는 주로 최상위 컴포넌트인 App의 state에 넣어서 관리합니다.

![](https://velog.velcdn.com/images/guddyd6761/post/1eff95b6-9556-41e5-a98a-83eddcc88c56/image.png)

위와 같은 상황에서 App이 지니고 있는 value값을 F컴포넌트와 J컴포넌트에 전달하려면 여러 컴포넌트를 거쳐야 합니다. 실제 프로젝트에서는 다루어야 하는 데이터가 훨씬 많아질 수도 있기에, 이러한 방식은 유지 보수성이 낮아질 가능성이 있습니다. Context API를 사용하면 Context를 만들어 단 한 번에 원하는 값을 받아와 사용할 수 있습니다.

![](https://velog.velcdn.com/images/guddyd6761/post/2d6c16d5-fa2d-41e3-923d-913573472307/image.png)


### 15.2 Context API 사용법 익히기

#### 새 Context 만들기
```javascript
///color.js
import {createContext} from 'react';

const ColorContext = createContext({color : 'black'});

export default ColorContext
```
#### Consumer 사용하기
색상을 props로 받아 오는 것이 아니라 ColorContext안에 들어 있는 Consumer라는 컴포넌트를 통해 색상을 조회할 것입니다.

```javascript
/// ColorBox.js
import ColorContext from '../contexts/color';

const ColorBox = () => {
  return (
    <>
    <ColorContext.Consumer>
    {value =>(
      <div
        style={{
          width: '64px',
          height: '64px',
          background: value.color
        }}
      />
      )} 
      </ColorContext.Consumer>
    </>
  );
};

export default ColorBox;
```
Consumer 사이에 중괄호를 열어서 그 안에 함수를 넣어 주었는데, 이러한 패턴을 Function as a child 혹은 Render Props라고 합니다. 컴포넌트의 children이 있어야 할 자리에 일반 JSX혹은 문자열이 아닌 함수를 전달하는 것입니다.

> 🔥Render Props 예제
```javascript
	const RenderPropsSample = ({children}) => {
	return <div>결과:{children(5)}</div>;
	};
	export default RenderPropsSample
```
위와 같은 컴포넌트가 있다면 추후에 다음과 같이 사용가능합니다.
```javascript
<RenderPropsSample>{value => 2*value }</RenderPropsSample>
//결과 : 10을 렌더링 합니다.
```

#### Provider
Provider를 사용하면 Context의 value를 변경할 수 있습니다.
```javascript
import ColorBox from './components/ColorBox';
import ColorContext from './contexts/color'

const App = () => {
  return (
    <ColorContext.Provider value={{color: 'red'}}>
      <div>
        <ColorBox />
      </div>
      </ColorContext.Provider>
  );
};

export default App;
```

### 15.3 동적 Context 사용하기

#### Context 파일 수정하기
Context의 value에는 상태 값만 있어야하는 것은 아니고, 함수를 전달해 줄 수도 있습니다.
```javascript
///color.js
import React, { createContext, useState } from 'react';

const ColorContext = createContext({
  state: { color: 'black', subcolor: 'red' },
  actions: {
    setColor: () => {},
    setSubcolor: () => {}
  }
});

const ColorProvider = ({ children }) => {
  const [color, setColor] = useState('black');
  const [subcolor, setSubcolor] = useState('red');

  const value = {
    state: { color, subcolor },
    actions: { setColor, setSubcolor }
  };
  return (
    <ColorContext.Provider value={value}>{children}</ColorContext.Provider>
  );
};

// const ColorConsumer = ColorContext.Consumer과 같은 의미
const { Consumer: ColorConsumer } = ColorContext;

// ColorProvider와 ColorConsumer 내보내기
export { ColorProvider, ColorConsumer };

export default ColorContext;
```
```javascript
///ColorBox.js
import { ColorConsumer } from "../contexts/color";

const ColorBox = () => {
  return (
    
    <ColorConsumer>
    { ({state})=>(
    <>
      <div
        style={{
          width: '64px',
          height: '64px',
          background: state.color
        }}
      />
      <div 
      style={{
        width: '32px',
        height: '32px',
        background: state.subcolor
      }}
      />
      </>
      )} 
      </ColorConsumer>
  );
};

export default ColorBox;
```

#### 색상 선택 컴포넌트 만들기
```javascript
import React from "react";
import { ColorConsumer } from "../contexts/color";

const colors = ["red", "orange", "yellow", "green", "blue", "indigo", "violet"];

const SelectColors = () => {
  return (
    <div>
      <h2>색상을 선택하세요.</h2>
      <ColorConsumer>
        {({ actions }) => (
          <div style={{ display: "flex" }}>
            {colors.map((color) => (
              <div
                key={color}
                style={{
                  background: color,
                  width: "24px",
                  height: "24px",
                  cursor: "pointer",
                }}
                onClick={() => actions.setColor(color)}
                onContextMenu={(e) => {
                  e.preventDefault();
                  /* 마우스 오른쪽 버튼 클릭 시 메뉴가 뜨는 것을 무시함 */
                  actions.setSubcolor(color);
                }}
              />
            ))}
          </div>
        )}
      </ColorConsumer>
      <hr />
    </div>
  );
};

export default SelectColors;
```

### 15.4 Consumer 대신 Hook 또는 static contextType 사용하기

#### useContext Hook 사용하기
```javascript
///ColorBox.js
import ColorContext from "../contexts/color";
import { useContext } from 'react';

const ColorBox = () => {
    const { state } = useContext(ColorContext)
    return (
        <>
            <div
                style={{
                    width: '64px',
                    height: '64px',
                    background: state.color
                }}
            />
            <div
                style={{
                    width: '32px',
                    height: '32px',
                    background: state.subcolor
                }}
            />
        </>
    )
}

export default ColorBox;
```
#### static contextType 사용하기
```javascript
///SelectColors.js
import React, { Component } from 'react';
import ColorContext from '../contexts/color';

const colors = ['red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet'];

class SelectColors extends Component {
  static contextType = ColorContext;

  handleSetColor = color => {
    this.context.actions.setColor(color);
  };

  handleSetSubcolor = subcolor => {
    this.context.actions.setSubcolor(subcolor);
  };

  render() {
    return (
      <div>
        <h2>색상을 선택하세요.</h2>
        <div style={{ display: 'flex' }}>
          {colors.map(color => (
            <div
              key={color}
              style={{
                background: color,
                width: '24px',
                height: '24px',
                cursor: 'pointer'
              }}
              onClick={() => this.handleSetColor(color)}
              onContextMenu={e => {
                e.preventDefault();
                this.handleSetSubcolor(color);
              }}
            />
          ))}
        </div>
        <hr />
      </div>
    );
  }
}

export default SelectColors;
```
stactic contextType을 사용하면 클래스형 컴포넌트에서 Context를 사용할 수 있습니다.
static contextType을 정의하면 클래스 메서드에서도 Context에 넣어 둔 함수를 호출할 수 있다는 장점이 있습니다. 단점이라면, 한 클래스에서 하나의 Context밖에 사용하지 못한다는 것입니다.
