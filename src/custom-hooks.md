<h1><a id="custom-hooks" href="https://reactjs.org/docs/hooks-custom.html">Building you own Hooks</a></a></h1>

Custom Hooks нам омогућавају да екстрактујемо логику компоненте у функцију коју ћемо искористити више пута. У класама бисмо ово радили са HOC-s и render props.

Две компоненте које користе исти Hook не деле state, већ само statefull логику.

По конвенцији, **име custom Hook почиње са "use" и она може позивати друге Hooks.** Тако се олакшава аутоматска провера Hooks правила.

```
import {useState, useEffect} from 'react';

function useFriendStatus(friendID){
  const [isOnline, setIsOnline] = useState(null);

  useEffect(()=>{
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return ()=>{
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

```
function FriendStatus(props){
  const isOnline = useFriendStatus(props.friend.id);

  if(isOnline === null){
    return "Loading...";
  }

  return isOnline ? "Online" : "Offline";
}
```

```
function FriendListItem(props){
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{color: isOnline ? "green" : "black" }}>
      {{props.friend.name}}
    </li>
  )
}
```

