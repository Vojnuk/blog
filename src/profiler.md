<h1><a id="profiler" href="https://reactjs.org/docs/profiler.html">Profiler API</a></a></h1>

`Profiler`  мери колико често Реакт рендерује и која је цена рендеровања. Циљ је да нађе спорије делове апликације које би могле да имају користи од мемоизације.

То је компонента са два props-a: `id`- стринг компоненте која се профилише и `onRender` callback функције која се позива сваки пут када профилисана компонента начини update.

```
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props} />
    </Profiler>
    <Profiler id="Main" onRender={callback}>
      <Main {...props} />
    </Profiler>
  </App>
);
```

Callback функција прима следеће параметре који описују шта је рендеровано и колико дуго је трајало:
```
function onRenderCallback(
  id,              // the "id" prop of the Profiler tree that has just committed
  phase,           // either "mount" (if the tree just mounted) or "update" (if it re-rendered)
  actualDuration,  // time spent rendering the committed update
  baseDuration,    // estimated time to render the entire subtree without memoization
  startTime,       // when React began rendering this update
  commitTime,      // when React committed this update
  interactions     // the Set of interactions belonging to this update
) {
  // Aggregate or log render timings...
}
```
* <a href="https://gist.github.com/bvaughn/8de925562903afd2e7a12554adcdda16" target="_blank">interactions</a> се могу користити ради одређивања узрока update-a. Experimental API.
* не користи у продукцији