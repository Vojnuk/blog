<h1><a id="optimizing performance" href="https://reactjs.org/docs/hooks-rules.html#only-call-hooks-from-react-functions">Rules of Hooks</a></a></h1>

* **Не позивај Hooks унутар loops,  услова или угнежђених функција.** Значи, само на top-level Реакт функција. Овиме се осигурава да се Hooks позивају истим редоследом по сваком рендеру, што такође обезбеђује state.
* **Не позивај Hooks из регуларних ЈС функција.**