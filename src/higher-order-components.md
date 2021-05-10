<h1><a id="hoc" href="https://reactjs.org/docs/higher-order-components.html">Higher-Order Components (HOC)</a></a></h1>

Ово је техника (pattern) за поновно искоришћавање логике компоненти. То је функција која прима компоненту и враћа нову компоненту. 

```
const EnchancedComponent = higherOrderComponent(WrappedComponent)
```

  * HOCs су присутне у разним React библиотекама, као код Redux <a href="https://github.com/reduxjs/react-redux/blob/master/docs/api/connect.md#connect" target="_blank">connect</a> или Relay <a href="https://relay.dev/docs/en/fragment-container.html" target="_blank" >createFragmentContainer</a>.

### Use HOCs For Cross-Cutting Concerns

```
class CommentList extends React.Component{
  constructor(props){
    super(props);
    this.handleChange = this.handleChangeBind(this);
    this.state = {
      // "DataSource" is some global data source
      comments: DataSource.getComments()
    };
  }
  componentDidMount(){
    // subscribe to changes
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount(){
    // clean up listener
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange(){
    // update component state whenever the data source changes
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render(){
    return (
      <div>
        {this.state.comments.map( comment => (
          <Comment comment={comment} key={comment.id}>
        ))}
      </div>
    )
  }

}
```
Онда напишемо компоненту која се subscribe на један блог пост:

```
class BlogPost extends React.Component{
  constructor(props){
    super(props);
    this.handleChange = this.handleChangeBind(this);
    this.state = {
      blogPost : DataSource.getBlogPost(props.id);
    };
  }

  componentDidMount(){
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount(){
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange(){
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id);
    });
  }

  render(){
    return <TextBlock text={this.state.blogPost}/>
  }

}
```
Овде видимо да `CommentList` и `BlogPost` нису исти, позивају различите методе на `DataSource` и рендерују различите ствари. Међутим имају и доста сличне имплементације:
  * додавањe и брисање change listener на `DataSource`
  * унутар listener-a, позивање `setState` приликом промене извора података

За те сличности HOC нам највише одговара, ради креирања апстракције где дефинишемо исту логику коју потом делимо са више компоненти. Тако можемо креирати функцију која прихвата компоненту која прима subscribed data као prop:
```
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource, props) => DataSource.getBlogPost(props.id)
);
```
Први параметар је компонента коју обрађујемо, а други преузима податке који нам требају са датим `DataSource` и тренутним props.

Када се ове компоненте рендерују `CommentList` и `BlogPost` биће прослеђен `data` prop са најновијим подацима извученим из `DataSource`:
```
// this function takes a component...
function withSubscription(WrappedComponent, selectData){
  // and returns another component...
  return class extends React.Component{
    constructor(props){
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

  componentDidMount(){
    // takes care of subscription...
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount(){
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange(){
    this.setState({
      data : selectData(DataSource, this.props)
    });
  }

  render(){
    // renders the wrapped component with the fresh data
    // we also pass trough any additional props
    return <WrappedComponent data={this.state.data} {...this.props}/>
  }


  }
}
```

### Проблематика

* Не користи HOCs унутар render метода. Reconciliation прави проблем јер компонента за рендеровање није иста ранијој и долази до unmounting и губљења state компоненте.
```
render() {
  // A new version of EnhancedComponent is created on every render
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // That causes the entire subtree to unmount/remount each time!
  return <EnhancedComponent />;
}
```
Уместо тога примени HOC ван дефиниције компоненте, а ако је баш неопходан динамички HOC онда унутар конструктора или lifecycle метода.

* статички методи се морају копирати
  
  Приликом примене HOC на компоненту, оригинал се прекрива контејнер компонентом, па нова компонента нема статичке методе оригинала.
```
  // Define a static method
WrappedComponent.staticMethod = function() {/*...*/}
// Now apply a HOC
const EnhancedComponent = enhance(WrappedComponent);

// The enhanced component has no static method
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

Стога копираш методе на контејнер пре него га вратиш:

```
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // Must know exactly which method(s) to copy :(
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```
Међутим, онда мораш тачно знати које методе копираш. Аутоматски то радиш са <a href="https://github.com/mridgway/hoist-non-react-statics" target="_blank">hoist-non-react-statics</a>, a мануелно са експортом статичног метода из компоненте:
```
// Instead of...
MyComponent.someFunction = someFunction;
export default MyComponent;

// ...export the method separately...
export { someFunction };

// ...and in the consuming module, import both
import MyComponent, { someFunction } from './MyComponent.js';
```

* Refs се не прослеђују. Користи `forwardRef`.


  