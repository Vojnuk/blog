<h1><a id="portals" href="https://reactjs.org/docs/portals.html">Portals </a></a></h1>

Портали су first-class начин да рендерујемо дете у ДОМ ноде који се налази ван ДОМ хијерархије компоненте родитеља. Стандардно се дете маунтује у ДОМ родитељског нода који је најближи.

Ово чинимо са `ReactDOM.createPortal(child, container)`. Први аргумент је свако дете које Реакт може рендеровати као елемент, стринг или фрагмент. Други представља ДОМ елемент.

### Примена

Обично се користи када родитељска компонента има `overflow: hidden`, или `z-index`, па је неопходно да дете визуелно искочи из контејнера( dialogs, hovercards, tooltips).

* Обрати пажњу на keyboard focus приликом рада са порталима.

### Event Bubbling Through Portals

Портал се и даље налази у Реакт дрвету, независно од позиције у ДОМ дрвету. Зато контекст, као и event bubling и даље раде подразумевано.

```
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
```

`Parent` компонента у `#app-root` ће ухватити неухваћен, bubbling event од сиблинга `#modal-root`.

```
const appRoot = document.getElementById('app-root);
const modalRoot = document.getElementById('modalRoot');

class Modal extends React.Component{
  constructor(props){
    super(props);
    this.el = document.createElement('div');
  }

  componentDidMount(){
    modalRoot.appendChild(this.el);
  }
  componentWillUnmount(){
    modalRoot.removeChild(this.el);
  }

  render(){
    return ReactDom.createPortal(
      this.props.children,
      this.el
    );
  }
}

class Parent extends React.Component{
  constructor(props){
    super(props);
    this.state = {clicks: 0};
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick(){
    this.setState(state => ({
      clicks: state.clicks + 1
    }));
  }

  render(){
    return (
      <div onClick={this.handleClick}>
        <p>Number of clicks: {this.state.clicks}</p
        <p>
          Open up the browser DevTools
          to observe that the button
          is not a child of the div
          with the onClick handler.
        </p>
        <Modal>
          <Child/>
        </Modal>
    )
  }
}

function Child(){
  // the click event on this button will bubble up to parent
  // because there is no 'onClick' attribute defined
  return(
    <div className="modal">
      <button>Click</button>
    <div/>
  );
}

ReactDOM.render(<Parent/>, appRoot)
```