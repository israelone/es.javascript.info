# Shadow DOM y eventos

The idea behind shadow tree is to encapsulate internal implementation details of a component.
La idea detrás del árbol de sombra es encapsular los detalles de implementación interna de un componente.

Let's say, a click event happens inside a shadow DOM of `<user-card>` component. But scripts in the main document have no idea about the shadow DOM internals, especially if the component comes from a 3rd-party library.  
Digamos que un evento de clic ocurre dentro de una sombra DOM del componente <user-card>. Pero los scripts en el documento principal no tienen idea acerca de las partes internas del Shadow DOM, especialmente si el componente proviene de una biblioteca de terceros.

So, to keep the details encapsulated, the browser *retargets* the event.
Entonces, para mantener los detalles encapsulados, el navegador "reorienta" el evento.

**Events that happen in shadow DOM have the host element as the target, when caught outside of the component.**
** Los eventos que suceden en el DOM en la sombra tienen el elemento host como objetivo, cuando se detectan fuera del componente. **


Here's a simple example:
He aquí un ejemplo sencillo:

```html run autorun="no-epub" untrusted height=60
<user-card></user-card>

<script>
customElements.define('user-card', class extends HTMLElement {
  connectedCallback() {
    this.attachShadow({mode: 'open'});
    this.shadowRoot.innerHTML = `<p>
      <button>Haz Click</button>
    </p>`;
    this.shadowRoot.firstElementChild.onclick =
      e => alert("Objetivo interno: " + e.target.tagName);
  }
});

document.onclick =
  e => alert("Objectivo Externo: " + e.target.tagName);
</script>
```

If you click on the button, the messages are:
Si hace clic en el botón, los mensajes son:

1. Inner target: `BUTTON` -- internal event handler gets the correct target, the element inside shadow DOM.
1. Objetivo interno: `BUTTON` - el controlador de eventos interno obtiene el objetivo correcto, el elemento dentro del DOM de sombra.

2. Outer target: `USER-CARD` -- document event handler gets shadow host as the target.
2. Destino externo: `USER-CARD` - el controlador de eventos del documento obtiene el host oculto como destino.


Event retargeting is a great thing to have, because the outer document doesn't have to know  about component internals. From its point of view, the event happened on `<user-card>`.
La reorientación de eventos es algo excelente, porque el documento externo no tiene que saber sobre los componentes internos. Desde su punto de vista, el evento ocurrió en `<user-card>`.


**Retargeting does not occur if the event occurs on a slotted element, that physically lives in the light DOM.**
**La reorientacion no ocurre si el evento ocurre en un elemento ranurado, que vive físicamente en el DOM ligero. **

For example, if a user clicks on `<span slot="username">` in the example below, the event target is exactly this `span` element, for both shadow and light handlers:
Por ejemplo, si un usuario hace clic en `<span slot =" username ">` en el ejemplo siguiente, el objetivo del evento es exactamente este elemento `span`, tanto para los controladores de sombras como de luces:

```html run autorun="no-epub" untrusted height=60
<user-card id="userCard">
*!*
  <span slot="username">John Smith</span>
*/!*
</user-card>

<script>
customElements.define('user-card', class extends HTMLElement {
  connectedCallback() {
    this.attachShadow({mode: 'open'});
    this.shadowRoot.innerHTML = `<div>
      <b>Name:</b> <slot name="username"></slot>
    </div>`;

    this.shadowRoot.firstElementChild.onclick =
      e => alert("Inner target: " + e.target.tagName);
  }
});

userCard.onclick = e => alert(`Outer target: ${e.target.tagName}`);
</script>
```

If a click happens on `"John Smith"`, for both inner and outer handlers the target is `<span slot="username">`. That's an element from the light DOM, so no retargeting.
Si ocurre un clic en "" John Smith "`, tanto para los controladores internos como externos, el objetivo es `<span slot =" username ">`. Ese es un elemento del DOM ligero, por lo que no hay reorientacion.

On the other hand, if the click occurs on an element originating from shadow DOM, e.g. on `<b>Name</b>`, then, as it bubbles out of the shadow DOM, its `event.target` is reset to `<user-card>`.
Por otro lado, si el clic ocurre en un elemento que se origina en el DOM de sombra, p. Ej. en `<b> Nombre </b>`, luego, cuando sale de la sombra DOM, su `event.target` se restablece a` <user-card> `.

## Bubbling, event.composedPath()
## Burbujeante, event.composedPath ()

For purposes of event bubbling, flattened DOM is used.
Para efectos de propagación de eventos, se utiliza DOM aplanado.

So, if we have a slotted element, and an event occurs somewhere inside it, then it bubbles up to the `<slot>` and upwards.
Entonces, si tenemos un elemento ranurado, y ocurre un evento en algún lugar dentro de él, entonces burbujea hasta el `<slot>` y hacia arriba.

The full path to the original event target, with all the shadow elements, can be obtained using `event.composedPath()`. As we can see from the name of the method, that path is taken after the composition.
La ruta completa al objetivo del evento original, con todos los elementos de sombra, se puede obtener usando `event.composedPath ()`. Como podemos ver en el nombre del método, ese camino se toma después de la composición.

In the example above, the flattened DOM is:
En el ejemplo anterior, el DOM aplanado es:

```html
<user-card id="userCard">
  #shadow-root
    <div>
      <b>Name:</b>
      <slot name="username">
        <span slot="username">John Smith</span>
      </slot>
    </div>
</user-card>
```


So, for a click on `<span slot="username">`, a call to `event.composedPath()` returns an array: [`span`, `slot`, `div`, `shadow-root`, `user-card`, `body`, `html`, `document`, `window`]. That's exactly the parent chain from the target element in the flattened DOM, after the composition.
Entonces, para hacer clic en `<span slot =" username ">`, una llamada a `event.composedPath ()` devuelve una matriz: [`span`,` slot`, `div`,` shadow-root`, `tarjeta de usuario`,` cuerpo`, `html`,` documento`, `ventana`]. Esa es exactamente la cadena principal del elemento de destino en el DOM aplanado, después de la composición.


```warn header="Shadow tree details are only provided for `{mode:'open'}` trees"
If the shadow tree was created with `{mode: 'closed'}`, then the composed path starts from the host: `user-card` and upwards.
`` `warn header =" Los detalles del árbol de sombras solo se proporcionan para `{mode: 'open'}` trees "
Si el árbol de sombra se creó con `{mode: 'closed'}`, entonces la ruta compuesta comienza desde el host: `user-card` y hacia arriba.

That's the similar principle as for other methods that work with shadow DOM. Internals of closed trees are completely hidden.
```
Ese es el principio similar al de otros métodos que funcionan con shadow DOM. Las partes internas de los árboles cerrados están completamente ocultas.
```


## event.composed

Most events successfully bubble through a shadow DOM boundary. There are few events that do not.
La mayoría de los eventos burbujean con éxito a través de un límite DOM de sombra. Hay pocos eventos que no lo hagan.

This is governed by the `composed` event object property. If it's `true`, then the event does cross the boundary. Otherwise, it only can be caught from inside the shadow DOM.
Esto se rige por la propiedad del objeto de evento `compuesto`. Si es "verdadero", entonces el evento cruza el límite. De lo contrario, solo se puede capturar desde el interior del DOM de sombra.

If you take a look at [UI Events specification](https://www.w3.org/TR/uievents), most events have `composed: true`:
Si echas un vistazo a la [especificación de eventos de IU] (https://www.w3.org/TR/uievents), la mayoría de los eventos tienen `compuesto: verdadero`:

- `blur`, `focus`, `focusin`, `focusout`,
- `click`, `dblclick`,
- `mousedown`, `mouseup` `mousemove`, `mouseout`, `mouseover`,
- `wheel`,
- `beforeinput`, `input`, `keydown`, `keyup`.

All touch events and pointer events also have `composed: true`.
Todos los eventos táctiles y de puntero también tienen "compuesto: verdadero".

There are some events that have `composed: false` though:
Sin embargo, hay algunos eventos que han `compuesto: falso`:

- `mouseenter`, `mouseleave` (they do not bubble at all),
- `load`, `unload`, `abort`, `error`,
- `select`,
- `slotchange`.

These events can be caught only on elements within the same DOM, where the event target resides.
Estos eventos se pueden capturar solo en elementos dentro del mismo DOM, donde reside el objetivo del evento.

## Custom events
## Eventos personalizados

When we dispatch custom events, we need to set both `bubbles` and `composed` properties to `true` for it to bubble up and out of the component.
Cuando despachamos eventos personalizados, necesitamos establecer las propiedades de `burbujas` y de` compuesto` en `verdadero` para que brote y salga del componente.

For example, here we create `div#inner` in the shadow DOM of `div#outer` and trigger two events on it. Only the one with `composed: true` makes it outside to the document:
Por ejemplo, aquí creamos `div # inner` en la sombra DOM de` div # outside` y activamos dos eventos en él. Solo el que tiene `compuesto: verdadero` lo hace fuera del documento:

```html run untrusted height=0
<div id="outer"></div>

<script>
outer.attachShadow({mode: 'open'});

let inner = document.createElement('div');
outer.shadowRoot.append(inner);

/*
div(id=outer)
  #shadow-dom
    div(id=inner)
*/

document.addEventListener('test', event => alert(event.detail));

inner.dispatchEvent(new CustomEvent('test', {
  bubbles: true,
*!*
  composed: true,
*/!*
  detail: "composed"
}));

inner.dispatchEvent(new CustomEvent('test', {
  bubbles: true,
*!*
  composed: false,
*/!*
  detail: "not composed"
}));
</script>
```

## Summary
## Resumen

Events only cross shadow DOM boundaries if their `composed` flag is set to `true`.
Los eventos solo cruzan los límites del DOM de sombra si su marca "compuesta" se establece en "verdadero".


Built-in events mostly have `composed: true`, as described in the relevant specifications:
Los eventos integrados en su mayoría tienen "compuesto: verdadero", como se describe en las especificaciones relevantes:

- UI Events <https://www.w3.org/TR/uievents>.
- Eventos de UI

- Touch Events <https://w3c.github.io/touch-events>.

- Pointer Events <https://www.w3.org/TR/pointerevents>.
- Eventos de puntero

- ...And so on.
- ...Y así.

Some built-in events that have `composed: false`:
Algunos eventos integrados que tienen

- `mouseenter`, `mouseleave` (also do not bubble),
- `load`, `unload`, `abort`, `error`,
- `select`,
- `slotchange`.

These events can be caught only on elements within the same DOM.
Estos eventos solo se pueden capturar en elementos dentro del mismo DOM.

If we dispatch a `CustomEvent`, then we should explicitly set `composed: true`.
Si enviamos un `CustomEvent`, entonces deberíamos establecer explícitamente` composite: true`.

Please note that in case of nested components, one shadow DOM may be nested into another. In that case composed events bubble through all shadow DOM boundaries. So, if an event is intended only for the immediate enclosing component, we can also dispatch it on the shadow host and set `composed: false`. Then it's out of the component shadow DOM, but won't bubble up to higher-level DOM.
Tenga en cuenta que en el caso de componentes anidados, un DOM de sombra puede anidarse en otro. En ese caso, los eventos compuestos burbujean a través de todos los límites del DOM de sombra. Por lo tanto, si un evento está destinado solo para el componente adjunto inmediato, también podemos enviarlo al host de sombra y establecer `compuesto: falso`. Luego, está fuera del DOM de sombra del componente, pero no subirá al DOM de nivel superior.
