<h1><a id="error-boundaries" href="https://reactjs.org/docs/error-boundaries.html">Error Boundaries </a></a></h1>

Раније су JS грешке унутар компоненти корумпирале унутрашњи state React-a, изазивајући криптичне грешке код рендеровања. React 16 разрешава ово увођењем Error Boundaries.

То је React компонента погодна за хватање грешака и приказивања fallback UI. Неопходна је јер су React компоненте декларативне природе за разлику од `try`/`catch` који ради код императивног кода. 

Error boundaries не хватају грешке код:
* event handlers - за разлику од render или lifecycle метода не дешавају се током рендеровања, ако избаце грешку React и даље зна шта да прикаже на екрану.
* asynchronous code (`setTimeout` или `requestAnimationFrame` callbacks)
* server-side rendering
* избачене грешке у самом error boundary (грешку треба избацити у детету) 

Class компонента постаје error boundary ако уколико дефинише lifecycle методе `static getDerivedStateFromError()` (за рендеровање fallback UI) или `componentDidCatch`(за логовање информација о грешки).

```
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // You can also log the error to an error reporting service
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```

Онда је користимо као регуларну компоненту:
```
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
``` 
Само компоненте класе могу бити error boundaries. Генерално је довољно декларисати је једном за коришћење широм апликације.


* грешке које нису ухваћене у error boundary резултују unmounting-ом читавог React  дрвета (од React 16).