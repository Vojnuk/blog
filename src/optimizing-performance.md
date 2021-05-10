<h1><a id="optimizing performance" href="https://reactjs.org/docs/optimizing-performance.html">Optimizing performance</a></a></h1>

### Virtualize Long Lists

Ако програм рендерује дуге листе података (стотине или хиљаде редова), употреби "windowing" технику. Овим се рендерује само део редова и знатно смањује време ререндеровања као и креацију ДОМ нодова.

Види <a href="https://react-window.now.sh/" target="_blank">react-window</a> и <a href="https://bvaughn.github.io/react-virtualized/" target="_blank">react-virtualized</a> библиотеке, које нуде компоненте за приказ листи, grids и табеларних података.

### Avoid Reconciliation

Приликом промене props или state React упоређује нови елемент са претходно рендерованим да би утврдио да ли је ДОМ update неопходан. Ако нису једнаки, долази до update и поновног рендеровања. 

Међутим, и компоненте које не врше ДОМ update биће ререндероване. Да бисмо избегли ово `shouldComponentUpdate` метод мора вратити `false` (default је `true`). Такође можемо наследити од `React.PureComponent`.

```
class CounterButton extends React.Component {
  constructor(props){
    super(props);
    this.state = {count: 1};
  }

shouldComponentUpdat(nextProps, nextState){
  if (this.props.color !== nextProps.color){
    return true;
  }
  if (this.state.count !== nextState.count){
    return true;
  }

  return false;
}

render(){
  return(
    <button color={this.props.color}
            onClick={() => this.setState(state => ({count: state.count + 1})) } >
            Count: {this.state.count}
    </button>
  );
  }

}
```

`React.PureComponent` уместо нас дефинише `shouldComponentUpdate` метод, али врши само shallow поређење props и state. Ово може бити проблем ако користиш mutator методе.
```
class ListOfWords extends React.PureComponent {
  render(){
    return <div>{this.props.words.join(',')}</div>
  }
}

class WordAdder extends React.Component{
  constructor(props){
    super(props);
    this.state = {
      words: ['marklar']
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick(){
    // This section is bad style and causes a bug
    const words = this.state.words;
    words.push('marklar');
    this.setState({words: words});
  }

  render(){
    return(
      <div>
        <button onClick={this.handleClick}/>
        <ListOfWords words={this.state.words}/>
      </div>
    )
  }

}
```
Овде мењамо `words` array унутар handleClick метода, тако да ће старе и нове вредности `this.props.words` бити оцењене као исте иако су саме речи унутар низа промењене. Стога користимо non-mutator метод `concat`:
```
handleClick(){
  this.setState(state => ({
    words: state.words.concat(['marklar])
  }));
}
```
Или са spread синтаксом:
```
handleClick() {
  this.setState(state => ({
    words: [...state.words, 'marklar'],
  }));
};
```
