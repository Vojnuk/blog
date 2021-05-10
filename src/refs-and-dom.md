<h1><a id="refs-and-dom" href="https://reactjs.org/docs/refs-and-the-dom.html">Refs and the DOM </a></a></h1>

У стандарном раду са React-ом, props су једини начин на који родитељ компонента врши интеракцију са дететом. Да промениш дете, ререндерујеш га са новим props. Међутим, каткада је потребно императивно променити дете, било да је у питању компонента или DOM елемент. <b>Refs</b> нам омогућава приступ елементима унутар render метода.

### Када их користимо:
* приликом управљања фокусом, селекције текста, покретања медија
* triggering imperative animations
* интеграција са third-party DOM библиотекама

### Creating Refs
Креирамо их са `React.createRef()` и повезујемо са React елементима преко `ref` атрибута. Refs се додају својству инстанце приликом конструисања компоненти, зато их немамо у function компонентама.

```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```

### Accessing Refs

По прослеђивању ref елемента унутар render метода, референца ка ноду постаје приступачна преко `current` атрибута ref-a.
```
const node = this.myRef.current;
```

Вредност ref-a зависи од типа нода:
* ако се ref атрибут користи на HTML елементу, `current` добија ДОМ елемент
* ако се користи на компоненти класе, онда ref објекат добија mounted инстанцу компоненте као вредност `current`

```
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // create a ref to store the textInput DOM element
    this.textInput = React.createRef();
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // Explicitly focus the text input using the raw DOM API
    // Note: we're accessing "current" to get the DOM node
    this.textInput.current.focus();
  }

  render() {
    // tell React that we want to associate the <input> ref
    // with the `textInput` that we created in the constructor
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```
React даје ДОМ елемент `current` својству по mounting компоненте, а враћа на `null` по unmounting. ref updates се дешавају пре `componentDidMount` или `componentDidUpdate` метода.

```
class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }

  componentDidMount() {
    this.textInput.current.focusTextInput();
  }

  render() {
    return (
      <CustomTextInput ref={this.textInput} />
    );
  }
}
```

Унутар компоненте функције, не можемо креирати ref јер немамо инстанце (осим са `forwardRef` и `useImperativeHandle`). Међутим, можемо користити  <b> ref атрибут унутар компоненте функције</b>  докле год он реферише ка ДОМ елементу или компоненти класе.

```
function CustomTextInput(props) {
  // textInput must be declared here so the ref can refer to it
  const textInput = useRef(null);
  
  function handleClick() {
    textInput.current.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={textInput} />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );
}
```
### Callback Ref
Други начин дефинисања refs је преко callback функције. 

Често се `ref` callback користи да ускладишти референцу ка ДОМ ноду унутар својства инстанце.
```
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInput = null;

    this.setTextInputRef = element => {
      this.textInput = element;
    };

    this.focusTextInput = () => {
      // Focus the text input using the raw DOM API
      if (this.textInput) this.textInput.focus();
    };
  }

  componentDidMount() {
    // autofocus the input on mount
    this.focusTextInput();
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in an instance field (for example, this.textInput).
    return (
      <div>
        <input
          type="text"
          ref={this.setTextInputRef}
        />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```