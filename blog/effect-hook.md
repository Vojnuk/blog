<h1><a id="effect-hooks" href="https://reactjs.org/docs/hooks-effect.html">Using the effect hook </a></a></h1>

Effect Hook нам омогућава да одрадимо неке side effects у функцијским компонентама. Одатле и назив. У суштини одрађује све оно за шта бисмо раније користили комбиновано `componentDidMount`, `componentDidUpdate` и `componentWillUnmount`.

* У side effects спадају data fetching, setting up a subscription, мануелно мењање ДОМ-а итд.

У суштини имамо две врсте ефекта, оне где морамо и оне где не морамо вршити чишћење. 

### Effects Without Cleanup

Ово су ефекти које одрадимо пошто Реакт апдејтује ДОМ. Мрежни захтеви, мануелне промене ДОМ-а, логовање, све то не захтева чишћење.

Са класама:
```
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

Hooks:
```
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
у суштини функција `useEffect` казује Реакту да компонента треба учинити још нешто по рендеровању. Ефекат који се треба извршити се налази у прослеђеној `callback` функцији.

**`useEffect` се активира после сваког рендеровања компоненти.** За разлику од `componentDidMount` или `componentDidUpdate`, ефекти дефинисани са `useEffect` не блокирају апдејтинг екрана. Тамо где је неопходно да се ефекти одиграју синхроно (нпр. код мерења layout-a) користимо <a href="https://reactjs.org/docs/hooks-reference.html#uselayouteffect">useLayoutEffect</a>.

### Effects with cleanup

Ово имамо код subscription на неки спољни извор података. Онда је неопходно почистити да не би дошло до memory leak. За чишћење користимо `return` код `useEffect`, и оно се извршава приликом unmounting компоненте. Собзиром да се ефекти извршавају после сваког рендеровања, Реакт чисти и ефекат претходног рендера, па тек онда покрене ефекат тренутног рендера. 

```
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

### Optimizing Performance by Skipping Effects

Каткада, позивање ефеката по сваком рендеру може направити проблеме у перформансама. Стога је потребно избегнути примену ефекта уколико се одређене вредности нису промениле између рендера. 

Код класа смо то радили са `prevProp` и `prevState` у `componentDidUpdate`:
```
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```
Код Hooks то чинимо тако што проследиомо arrray као опциони други аргумент `useEffect`:
```
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```
Води рачуна да **низ укључује све вредности component scope-a (props, state) који се мењају током времена, да не би дошло до очитавања застарелих вредности**.

Ако желимо извршити ефекат и почистити га **само једном** (приликом mount-а и unmount-а), онда прослеђујемо празан низ. Ово говори Реакту да ефекат не зависи од вредности props и state, а ако их користи, користиће њихове иницијалне вредности.
* код наредних Реакт верзија други аргумент би се могао додавати аутоматски током build-time трансформације.