<h1><a id="forwarding-refs" href="https://reactjs.org/docs/forwarding-refs.html">Forwarding Refs </a></a></h1>

Ово је техника за аутоматско прослеђивање **ref** компоненте ка неком од њене деце. 

### Forwarding refs to DOM components са React.forwardRef(()=>{})

```
function FancyButton(props){
  return(
    <button className="FancyButton">
      {props.children}
    </button>
  )
}
```

React компоненте крију детаље имплементације, укључујући и рендерован излаз. Међутим, код неких елемената који се често користе као `FancyButton`, а нису application-level компоненте као `Comment`, каткад је потребан приступ ДОМ елементу. То вршимо тако што компонента прими ref и онда га проследи (forward) ка детету.

```
const FancyButton = React.forwardRef(props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));


// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me</FancyButton>
```

* `ref` није исто што и `prop`. React га обрађује другачије.
