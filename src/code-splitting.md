<h1><a id="code-splitting" href="https://reactjs.org/docs/code-splitting.html">Code-Splitting </a></a></h1>

### Bundling and Code Splitting

**Bundling** је процес укључивања импортованих фајлова и њиховог спајања у јединствени фајл. Тај фајл се потом укључује на веб страну ради учитавања одједном. Међутим, што је фајл већи то је и учитавање истог дуже.

Зато имамо **code-splitting** (bundler feature) који од датог bundle креирају неколико мањих који се могу динамички учитати током runtime.

Само динамичко учитавање вршимо са **`import()`**.

> Code-splitting је аутоматски подешен код Create React App и Next.js. Код коришћења Babel-a морамо подесити да приликом парсовања динамичког import() не дође до трансформације, што чинимо са [babel-plugin-syntax-dynamic-import](https://classic.yarnpkg.com/en/package/babel-plugin-syntax-dynamic-import).

### React.lazy и Suspense


>React.lazy и Suspense још увек нису доступни за server-side rendering. За то користи [Loadable components](https://github.com/gregberge/loadable-components).

`React.lazy` омогућава рендеровање динамичког импорта као регуларне компоненте. Она прима функцију која мора позвати `import()`. Import () потом враћа `Promise` који се разрешава у `default` export са садржаном React компонентом. Овa компонентa, тзв. lazy component, мора потом бити рендерована унутар `Suspense` компоненте, која нам са <mark>fallback</mark> prop омогућава индикатор учитавања, док се lazy компонента не учита.
```
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```
* Можеш огрнути више lazy компоненти са јединственом Suspense компонентом.

### Error boundaries

Ако дође до неуспешног учитавања, користимо Error boundaries:
```
import React, { Suspense } from 'react';
import MyErrorBoundary from './MyErrorBoundary';

const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
    <MyErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </MyErrorBoundary>
  </div>
);
```

### Route-based code splitting

Генерално је најлакше применити code splitting код рута. За то можемо користити и [React Router](https://reactrouter.com/).

```
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/" component={Home}/>
        <Route path="/about" component={About}/>
      </Switch>
    </Suspense>
  </Router>
);
```

### Named Exports

`React.lazy` тренутно подржава само default exports. Ако модул који увозимо има named exports онда морамо креирати модул који га извози као default.
```
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;
```

```
// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";
```

```
// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```