<h1><a id="accessibility" href="https://reactjs.org/docs/accessibility.html">Accessibility </a></a></h1>

### WAI-ARIA
Сви `aria-*` атрибути су подржани у JSX (hyphen-case), за
разлику od стандардних DOM својстава и атрибута који су
camelCased.

```
  <input
    type="text"
    aria-label={labelText}
    aria-required="true"
    onChange={onchangeHandler}
    value={inputValue}
    name="name"
  />
```
### Semantic HTML
Користи семантички HTML као `main`, `aside` ради бољег груписања садржаја и навигације. Користи `React.Fragment` за груписање елемената када радиш са деловима листи или табела уместо `<div>`.

Елементи форме као што су `<input>` и `<textarea>` морају бити означени са `<label>`. Атрибут `for` се пише као `htmlFor` у JSX:
```
<label htmlFor="namedInput">Name:</label>
<input id="namedInput" type="text" name="name"/>
```

### Focus Control
Промена DOM-a током runtime често води до губљења фокуса или његовог постављања на неадекватан елемент. Тако је по затварању модала неопходно поставити фокус на дугме које је отворило модал.
Приступ DOM нодовима или react елементима креираним унутар render метода остварујеш са **refs**. Види [react-aria-modal](https://github.com/davidtheclark/react-aria-modal).


Спољну линију фокуса (outline) немој брисати са `outline: 0`, осим уколико имаш сопствену имплементацију.

* Обезбеди Skiplinks или Skip Navigation Links ако корисник тастатуре користи страну.

### Mouse and pointer events

Имплементација решења за миш events често води лошој функционалности за кориснике тастатуре. Чест пример је **outside click pattern** где корисник затвара отворени popover кликом ван елемента. Ово се обично имплементира са `click` event на `window` објекту. Међутим корисник тастатуре само може да табује на следећи елемент без затварања popover-a. Боље решење се остварује са `onBlur` и `onFocus`.


### Screen Readers 

NVDA in Firefox  
VoiceOver in Safari  
JAWS in Internet Explorer  
ChromeVox in Google Chrome  


