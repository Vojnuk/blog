<h1><a id="code-splitting" href="https://reactjs.org/docs/context.html">Context </a></a></h1>


Context нам омогућава прослеђивање података кроз дрво компоненти без експлицитног прослеђивања props кроз сваку компоненту. 

У суштини кад год имамо глобалну вредност (преференца корисника, UI тема, data cashe...) коју захтевају многе компоненте унутар апликације, context нам може помоћи.  



 

### API

 #### React.createContext
 ```
 const MyContext = React.createContext(defaultValue);
 ```
 Креира Context објекат. Приликом рендеровања компоненти које се позивају (subscribe) на овај објекат, ишчитава се тренутна вредност контекста најближег `Provider`. Аргумент  `defaultValue` се користи само када тог провајдера нема.

 #### Context.Provider

 ```
 <MyContext.Provider value={/*some value*/}>
 ```
 Provider је компонента Context објекта која омогућава consumer компонентама да се subscribe на промену контекста. На њу се могу повезати више consumers.
 Прихвата `value` props које прослеђује consuming компонентама. Приликом његове промене долази до ререндеровања зависних компоненти, независно од `shouldComponentUpdate` метода.

 #### Class.contextType

 ```
 class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* perform a side-effect at mount using the value of MyContext */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* render something based on the value of MyContext */
  }
}
MyClass.contextType = MyContext;
 ```
 `contextType` својству класе може бити прослеђен Context објекат. Потом се са `this.context` може конзумирати вредност контекста. Ово је само за повезивање са једним контекстом.

 У случају да имамо више контекста, React чини сваког конзумера посебним нодом у дрвету.
 ```
// Theme context, default to light theme
const ThemeContext = React.createContext('light');

// Signed-in user context
const UserContext = React.createContext({
  name: 'Guest',
});

class App extends React.Component {
  render() {
    const {signedInUser, theme} = this.props;

    // App component that provides initial context values
    return (
      <ThemeContext.Provider value={theme}>
        <UserContext.Provider value={signedInUser}>
          <Layout />
        </UserContext.Provider>
      </ThemeContext.Provider>
    );
  }
}

function Layout() {
  return (
    <div>
      <Sidebar />
      <Content />
    </div>
  );
}

// A component may consume multiple contexts
function Content() {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <UserContext.Consumer>
          {user => (
            <ProfilePage user={user} theme={theme} />
          )}
        </UserContext.Consumer>
      )}
    </ThemeContext.Consumer>
  );
}
```
 * код public class field синтаксе користи <b>static</b> class field ради иницијализације `contextType`.
```
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    let value = this.context;
    /* render something based on the value */
  }
}
```

#### Context.Consumer

```
<MyContext.Consumer>
  {value=>{/*render something based on context value*/}}
</MyContext.Consumer>
```
Компонента која се subscribe на промену контекста. Захтева function as a child. Ова функција прима тренутну вредност контекста и враћа React node. `value` аргумент биће једнак `value` prop најближег провајдера односно `defaultValue` који је прослеђен приликом креирања контекста.

#### Context.displayName

Контекст објекат прихвата `displayName` стринг својство. Ово користи React Dev Tools ради означавања контекста.

```
const MyContext = React.createContext(/* some value */);
MyContext.displayName = 'MyDisplayName';

<MyContext.Provider> // "MyDisplayName.Provider" in DevTools
<MyContext.Consumer> // "MyDisplayName.Consumer" in DevTools
```

### Updating Context from a Nested Component

Ово чинимо са прослеђивањем функције кроз контекст како бисмо омогућили конзумерима да update контекст.

<b>theme-context.js</b>
```
// Make sure the shape of the default value passed to
// createContext matches the shape that the consumers expect!
export const ThemeContext = React.createContext({
  theme: themes.dark,
  toggleTheme: () => {},
});
```
<b>theme-toggler-button.js</b>
```

import {ThemeContext} from './theme-context';

function ThemeTogglerButton() {
  // The Theme Toggler Button receives not only the theme
  // but also a toggleTheme function from the context
  return (
    <ThemeContext.Consumer>
      {({theme, toggleTheme}) => (
        <button
          onClick={toggleTheme}
          style={{backgroundColor: theme.background}}>
          Toggle Theme
        </button>
      )}
    </ThemeContext.Consumer>
  );
}

export default ThemeTogglerButton;
```

<b>app.js</b>
```


import {ThemeContext, themes} from './theme-context';
import ThemeTogglerButton from './theme-toggler-button';

class App extends React.Component {
  constructor(props) {
    super(props);

    this.toggleTheme = () => {
      this.setState(state => ({
        theme:
          state.theme === themes.dark
            ? themes.light
            : themes.dark,
      }));
    };

    // State also contains the updater function so it will
    // be passed down into the context provider
    this.state = {
      theme: themes.light,
      toggleTheme: this.toggleTheme,
    };
  }

  render() {
    // The entire state is passed to the provider
    return (
      <ThemeContext.Provider value={this.state}>
        <Content />
      </ThemeContext.Provider>
    );
  }
}

function Content() {
  return (
    <div>
      <ThemeTogglerButton />
    </div>
  );
}

ReactDOM.render(<App />, document.root);

```


### Проблематика:

Отежава поновну употребу компоненти.

Контекст користи reference identity да би одлучио када да ререндерује. Стога може доћи до ререндеровања consumers приликом ререндеровања провајдер parent-a. Тада долази до креирања новог објекта за `value` што изазива ререндеровање конзумера.

```
class App extends React.Component {
  render() {
    return (
      <MyContext.Provider value={{something: 'something'}}>
        <Toolbar />
      </MyContext.Provider>
    );
  }
}
```
Тада морамо подићи `value` у `state` родитеља.
```
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: {something: 'something'},
    };
  }

  render() {
    return (
      <Provider value={this.state.value}>
        <Toolbar />
      </Provider>
    );
  }
}
```
