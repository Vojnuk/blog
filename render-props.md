<h1><a id="hoc" href="https://reactjs.org/docs/render-props.html">Render Props</a></a></h1>

Ово је техника дељења кода између компоненти коришћењем prop-a чија је вредност функција. Компонента позивањем ове функције добија Реакт елементе, па не мора да имплементира своју логику рендеровања.

```
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}>
```

### Примена 

* Када желимо да поделимо state или понашање компоненте са другом

```
class Cat extends React.Component {
  render(){
    const mouse = this.props.mouse;
    return(
      <img src="/cat.jpg" style{{
        position: 'absolute',
        left: mouse.x,
        top: mouse.y
      }}/>
    );
  }
}

class Mouse extends React.Component {
  constructor (props) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }
render(){
  return(
    <div style{{height: '100vh'}}
         onMouseMove={this.handleMouseMove}>
         
         {this.props.render(this.state)}
    </div>
  );
}
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move mouse around</h1>
        <Mouse render={mouse => (
            <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```
