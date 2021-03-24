---
title: Guía completa de useEffect
date: '2019-03-09'
spoiler: Los Effects son parte de tu flujo de datos
cta: 'react'
---

Escribiste algunos componentes con [Hooks](https://reactjs.org/docs/hooks-intro.html). Talvez incluso una pequeña applicación. Te sientes satisfecho. Te sientes cómodo con el API e incluso pudiste aplicar algunos trucos mientras la desarrollabas. Incluso creaste algunos [Hooks personalizados](https://reactjs.org/docs/hooks-custom.html) para extraer lógica repetitiva (adiós a 300 líneas) y se lo mostraste a tus colegas. "Buen trabajo", dijeron.

Pero en ocasiones, cuando usas `useEffect`, las piezas no encajan del todo. Tienes una sensación de que algo te está haciendo falta. Pareciera similar a los métodos del ciclo de vida de componentes... pero, ¿lo es? Eventualmente te haces preguntas como:

* 🤔 ¿Cómo replico `componentDidMount` usando `useEffect`?
* 🤔 ¿Cuál es la forma correcta de obtener datos externos dentro de `useEffect`? ¿Qué es `[]`?
* 🤔 ¿Es necesario que coloque funciones en las dependencias de `useEffect` o no?
* 🤔 ¿Por qué algunas veces mi código que obtiene datos externos entra en un ciclo infinito?
* 🤔 ¿Por qué algunas veces me encuentro con estado o propiedades con valores pasados en `useEffect`?

Cuando comencé a usar Hooks, a mi también me confundían todas estas preguntas. Incluso cuando estaba escribiendo la documentación inicial, no tenía un entendimiento firme de algunos detalles. De ese entonces para acá he tenido algunos momentos de "iluminación" que quiero compartir. **Este anális exhaustivo te dará todas las respuestas a esas preguntas.**

Para poder *ver* las respuestas, necesitamos verlo desde una perspectiva más general. El propósito de este artículo no es darte un listado de ingredientes como si se tratara de una receta. El propósito es ayudarte a digerir `useEffect`. No se tratará tando de aprender. De hecho, vamos a invertir bastante tiempo en *des*aprender.

**Cuando dejé de ver al Hook `useEffect` a través del lente de los familiares métodos del ciclo de vida fue cuando todas las piezas encajaron para mi.**

>“Desaprende lo que has aprendido.” — Yoda

![Yoda olfateando. Caption: “Huele a tocino.”](./yoda.jpg)

---

**Este artículo asume que ya estás algo familiarizado con el API de [`useEffect`](https://reactjs.org/docs/hooks-effect.html).**

**También es *bastante* extenso. Es como un pequeño libro. Ese es mi formato preferido. Pero escribí un TLDR justo abajo en caso que tengas prisa o no te importe demasiado.**

**Si no te sientes cómodo con análisis exhaustivos, pueda que debas esperar hasta que estas explicaciones aparezcan en otro lugar. Así como cuando React salió en 2013, va a tomar algo de tiempo para que los demás reconozcan un modelo mental diferente y puedan enseñarlo.**

---

## TLDR

Aquí hay un breve TLDR en caso que no quieras leer todo el artículo. Si hay partes que no hacen sentido, puedes desplazarte hacia abajo hasta que encuentres algo relacionado.

Siente libertad de saltarte esta parte si piensas leer la publicación completa. Voy a agregar un vínculo hacia aquí al final.

**🤔 Pregunta: ¿Cómo replico `componentDidMount` usando `useEffect`?**

Aún cuando puedes usar `useEffect(fn, [])`, no es un equivalente exacto. A diferencia de `componentDidMount`, va a *capturar* propiedades y estado. De manera que aún dentro de los callbacks, verás las propiedades y estado iniciales. Si quieres ver lo "último" de algo, puedes escribirlo en un ref. Pero usualmente hay una manera más simple de estructurar el código de manera que no tengas que hacer esto. Toma en cuenta que el modelo mental para los effects es diferente a `componentDidMount` y a otros métodos del ciclo de vida e intentar encontrar sus equivalentes exactos puede confundirte más que ayudarte. Para ser productivo, necesitas "pensar en efectos" y su modelo mental que es más cercano a implementar sincronización que a eventos del ciclo de vida.

**🤔 Pregunta:  ¿Cuál es la forma correcta de obtener datos dentro de `useEffect`? ¿Qué es `[]`?**

[Este artículo](https://www.robinwieruch.de/react-hooks-fetch-data/) es una buena introducción a obtener datos con `useEffect`. ¡Asegúrate de leerlo completo! No es tan largo como este artículo. `[]` significa que el efecto no usa ningún valor que participe en el flujo de datos de React, y es por esa razón que es seguro ejecutarlo una sola vez. También es una fuente común de errores cuando el valor *es* usado. Tendrás que aprender algunas estrategias (principalmente `useReducer` y `useCallback`) que pueden *remover la necesidad* de una dependencia en lugar de incorrectamente omitirla.

**🤔 Pregunta: ¿Es necesario que coloque funciones en las dependencias de `useEffect` o no?**

La recomendación es sacar *fuera* de tu componente las funciones que no necesitan propiedades o estado, y colocar *adentro* de effect aquellas que son utilizadas por ese effect. Si después de seguir esas recomendaciones tu effect aún termina utilizando funciones en el alcance del renderizado (incluyendo funciones pasadads como propiedades), enciérralas en `useCallback` en donde están definidas, y repite el proceso. ¿Por qué es importante hacerlo? Las funciones pueden "ver" valores de propiedades y estado - de manera que participan en el flujo de datos. Hay una [respuesta más detallada](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) en nuestro FAQ.

**🤔 Pregunta: ¿Por qué algunas veces mi código que obtiene datos externos entra en un ciclo infinito?**

Esto puede suceder si estás obteniendo datos en un effect sin utilizar el argumento de dependencias. Sin el, los effects son ejecutados despues de cada render — y ajustar el estado va a disparar los effects de nuevo. Un ciclo infinito también pueda darse si especificas un valor que *siempre* cambia en el arreglo de dependencias. Puedes encontrar cuál es quitando uno por uno. Sin embargo, remover una dependencia que utilizas (o ciegamente especificar `[]`) es usualmente una solución incorrecta. En su lugar, arregla el problema desde la fuente. Por ejemplo, funciones pueden ser la causa de este problema, y con ponerlas adentro de los effects, sacarlas del componente, o colocarlas adentro de `useCallback` puede ayudar. Para evitar recrear objetos, `useMemo` puede tener un uso similar. 

**🤔 Pregunta: ¿Por qué algunas veces me encuentro con estado o propiedades con valores pasados en `useEffect`?**

Los effects siempre "ven" los propiedades y estado del renderizado en el cual fueron definidos. Esto [ayuda a prevenir errores](/how-are-function-components-different-from-classes/) pero en algunos casos puede ser molesto. Para esos casos, puedes mantener el valor en un ref mutable de manera explícita (el artículo referido lo explica al final). Si crees que te estás encontrando con propiedades o estado de un renderizado pasado pero no espera que fuera asi, probablemente te hizo falta alguna dependencia. Intenta usar una [regla de lint](https://github.com/facebook/react/issues/14920) para entrenarte a ti mismo para verlas. Un par de días, y será como una segunda naturaleza para ti. También puedes ver [esta respuesta] (https://reactjs.org/docs/hooks-faq.html#why-am-i-seeing-stale-props-or-state-inside-my-function) en nuestro FAQ.

---

¡Espero que este TLDR haya sido de ayuda! De lo contrario, vámos.

---

## Cada Render Tiene Sus Propias Propiedades y Estado

Antes que podamos hablar de effects, necesitamos hablar sobre el renderizado.

Aquí hay un contador. Observa la línea resaltada detenidamente:

```jsx{6}
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

¿Qué significa? ¿Acaso `count` de alguna manera "observa" cambios en nuestro estado y se actualiza de manera automática? Esa puede ser una útil intuición inicial cuando aprendes React pero *no* es un [modelo mental adecuado](https://overreacted.io/react-as-a-ui-runtime/).

**En este ejemplo, `count` solo es un número.** No es un "vínculo de datos" mágico, un "observador", un "proxy", o nada más. Es un viejo y conocido número tal como este:

```jsx
const count = 42;
// ...
<p>You clicked {count} times</p>
// ...
```

La primera vez que nuestro componente se renderiza, la variable `count` que obtenemos de `useState()` es `0`. Cuando llamamos `setCount(1)`, React llama nuestro componente de nuevo. Esta vez, `count` será `1`. Y así sucesivamente:

```jsx{3,11,19}
// Durante el primer renderizado
function Counter() {
  const count = 0; // Retornado por useState()
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// Después de un click, nuestra función es llamada de nuevo
function Counter() {
  const count = 1; // Retornado por useState()
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// Después de otro click, nuestra función se llama de nuevo
function Counter() {
  const count = 2; // Retornado por useState()
  // ...
  <p>You clicked {count} times</p>
  // ...
}
```

**Cuando sea que actualicemos el estado, React llama a nuestro componente. Cada resultado de render "ve" su propio valor del estado `counter` el cual es una *constante* dentro de nuestra función.**

De este modo, esta línea no hace ningún tipo de vinculación de datos especiales:

```jsx
<p>You clicked {count} times</p>
```

**Solamente incrusta un valor numérico en los resultados del render.** Ese numero es proporcionado por React. Cuando hacemos `setCount`, React llama nuestro componente de nuevo con un valor diferente para `count`. Luego, React actualiza el DOM para que coincida con nuestro ultimo renderizado.

El punto importante de esto es que la constante `count` dentro de cualquier render particular no cambia con el tiempo. Es nuestro componente que es llamado de nuevo — y cada render "ve" su propio valor `count` que está aislado de render a render.

*(Para un recorrido profundo de este proceso, consulta mi publicación [React como un UI Runtime](https://overreacted.io/react-as-a-ui-runtime/) .)*

## Cada Render Tiene Sus Propios Manejadores De Eventos 

Hasta ahora, todo bien. ¿Qué hay de los menejadores de eventos?

Revisa este ejemplo. Muestra un mensaje de alerta con el valor de `count` después de tres segundos:

```jsx{4-8,16-18}
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```

Digamos que sigo estos pasos en orden:

* **Incremento** el contador a 3
* **Presiono** "Show alert"
* **Incremento** el contador a 5 antes de que el temporizador se dispare

![Demo del contador](./counter.gif)

¿Qué esperas que muestre el mensaje de alerta? ¿Mostrará 5 — que es el valor del estado del contador al momento del mensaje? ¿O mostrará 3 — el valor del estado cuando hice click?

----

*vienen spoilers*

---

¡Ve y [pruébalo tu mismo!](https://codesandbox.io/s/w2wxl3yo0l)

Si el resultado no te hace mucho sentido, imagina un ejemplo más práctico: una aplicación de chat con el valor actual del ID del recipiente en su estado y un botón de Enviar. [Este artículo](https://overreacted.io/how-are-function-components-different-from-classes/) explora las razones de esto a profundida, pero la respuesta correcta es 3.

El mensaje de alerta "captura" el estado en el momento que hice click en el botón.

*(Hay formas de implementar el otro resultado también, pero por ahora me estaré enfocando en el caso por defecto. Cuando construimos un modelo mental, es importante que distingamos el "camino con menor resistencia" del camino no convencional)*

---

¿Pero, cómo funciona?

Ya hemos discutivo que el valor de `count` es constante para cada llamada particular a nuestra función. Vale la pena enfatizar esto — **nuestra función es llamada muchas veces (una por cada render), pero cada una de esas veces el valor de `count` dentro de la función es constante y asignado a un valor en particular (el estado de ese render).**

Esto no es específico de React — las funciones regulares funcionan de manera similar:

```jsx{2}
function sayHi(person) {
  const name = person.name;
  setTimeout(() => {
    alert('Hello, ' + name);
  }, 3000);
}

let someone = {name: 'Dan'};
sayHi(someone);

someone = {name: 'Yuzhi'};
sayHi(someone);

someone = {name: 'Dominic'};
sayHi(someone);
```

En [este ejemplo](https://codesandbox.io/s/mm6ww11lk8), la variable externa `someone` es reasignada en varias ocasiones. (Igual que en algún lugar en React, el estado del componente *actual* puede cambiar.) **Sin embargo, adentro de `sayHi`, hay una constante local `name` que es asociada con una `person` de cada llamada particular.** Esa constante es local, por lo tanto, ¡es aislada entre cada llamada! Como resultado, cuando el temporizador se dispara, cada alerta "recuerda" su propio `name`.

Esto explica como nuestro manejador de eventos captura el valor de `count` en el momento del click. Si aplicamos el mismo principio de substitución, cada render "ve" su propio `count`:

```jsx{3,15,27}
// Durante el primer render
function Counter() {
  const count = 0; // Retornado por useState()
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}

// Después de un click, nuestra función es llamada de nuevo
function Counter() {
  const count = 1; // Retornado  por useState()
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}

// Después de otro click, nuestra función es llamada otra vez
function Counter() {
  const count = 2; // Retornado por useState()
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}
```

Entonces efectivamente, cada render retorna su propia "versión" de `handleAlertClick`. Cada una de esas versiones "recuerda" su propio `count`:

```jsx{6,10,19,23,32,36}
// Durante el primer render
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 0);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // La función que tiene 0 adentro
  // ...
}

// Después de un click, nuestra función es llamada nuevamente
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 1);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // La función que tiene 1 adentro
  // ...
}

// Después de otro click, nuestra función es llamada otra vez
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 2);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // La función que tiene 2 adentro
  // ...
}
```

Esta es la razón por la cual [en este demo](https://codesandbox.io/s/w2wxl3yo0l) la función que maneja el evento click "pertenece" a un render en particular, y cuando haces click, continúa utilizando el `counter` de acuerdo al estado *de* su ese render.

**Dentro de un render en particular, las propiedades y el estado se mantienen iguales para siempre.** Pero si las propiedades y el estado están aislados entre renders, así también están cualquiera que las utilice (incluyendo los manejadores de eventos). Ellos también "pertenecen" a un render en particular. De manera que incluso funciones async dentro de un manejador de eventos van a "ver" el mismo valor de `count`.

*Nota: Puse entre líneas valores concretos para `count` en la función `handleAlertClick` en los ejemplos de arriba. Esta substitución mental es segura porque el valor de `count` no es posible que cambie dentro de un mismo render. Es declarado como una `const` y es un numero. Sería seguro pensar de la misma manera acerca de otros valores como objetos, pero solo si podemos acordar omitir mutar el estado. Llamar `setSomething(newObj)` con un objeto recientemente creado en lugar de mutarlo está bien porque el estado que pertenece a renders previos está intacto.*

## Cada Render Tiene Sus Propios Effect

Se suponía que esta publicación sería acerca de effects pero aún no hemos hablado nada de ellos! Arreglaremos eso ahora. Resulta que, los effects no son para nada diferentes.

Regresemos a un ejemplo de [la documentación](https://reactjs.org/docs/hooks-effect.html):

```jsx{4-6}
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

**¿Pregunta: cómo hace el effect para leer el último valor del estado de `count`?**

¿Talvez, hay algún tipo de "datos vinculados" u "observador" que hace que `count` se actualice en vivo adentro de la función effect? ¿Talvez `count` es una variable mutable que React define dentro de nuestro componente de manera que nuestro effect ve la última valor?

No.

Ya sabes que `count` es una constante dentro de un render particular de nuestro componente. Los manejadores de eventos "ven" el estado `count` correspondiente al render al cual "pertenece" porque `count` es una variable en su ámbito. Lo mismo aplica para los effects!

**No es la variable `count` que de alguna manera cambia dentro de un effect "inmutable". Es _la función effect en sí misma_ que es diferente para cada render.**

Cada version "ve" el valor de `count` correspondiente al render que "pertenece":

```jsx{5-8,17-20,29-32}
// Durante el primer render
function Counter() {
  // ...
  useEffect(
    // La función Effect del primer render
    () => {
      document.title = `You clicked ${0} times`;
    }
  );
  // ...
}

// Despues del click, nuestra función es llamada de nuevo
function Counter() {
  // ...
  useEffect(
    // La función Effect del segundo render
    () => {
      document.title = `You clicked ${1} times`;
    }
  );
  // ...
}

// Después de otro click, nuestra función es llamada de nuevo
function Counter() {
  // ...
  useEffect(
    // La función Effect del tercer render
    () => {
      document.title = `You clicked ${2} times`;
    }
  );
  // ..
}
```

React recuerda la función effect que proporcionaste, y la ejecuta después de despachar cambios  hacia el DOM y dejar que el navegador pinte la pantalla.

De manera que aún cuando hablamos de un solo *effect* conceptualmente (que actualiza el dítulo del documento), es representado por una *función diferente* en cada render — y cada función effect "ve" propiedades y estado correspondientes al render en particular al cuál pertenece.

**Conceptualmente, puedes imaginar que los effects son *parte del resultado del render*. **

Hablando estrictamente, no lo son (con el propósito de [permitir composición de Hooks](https://overreacted.io/why-do-hooks-rely-on-call-order/) evitando sintáxis tosca o impacto en el tiempo de ejecución). Pero en el modelo mental que estamos construyendo, las funciones effect *pertenecen* a un render en particular en la misma forma que los manejadores de evento lo hacen.

---

Para asegur que tenemos una comprensión sólida, revisemos nuestro primer render:

* **React:** Dame la UI para cuando el estado es `0`.
* **Tu componente:**
  * Aquí está el resultado del render:
  `<p>You clicked 0 times</p>`
  * También recuerda ejecutar este effect después de que hayas terminado `() => { document.title = 'You clicked 0 times' }`.
* **React:** Claro. Actualizando la UI. Hey navegador, estoy agregando algunas cosas al DOM.
* **Browser:** Super, ya lo dibujé en la pantalla.
* **React:** Ok, ahora voy a ejecutar el effect que me indicaste.
  * Ejecutando `() => { document.title = 'You clicked 0 times' }`.

---

Ahora hagamos una revisión de lo que pasa después que hacemos click:

* **Tu componente:** Hey React, pon en mi estado el valor `1`.
* **React:** Dame la UI para cuando el estado es `1`.
* **Tu componente:**
  * Aquí está el resultado del render:
  `<p>You clicked 1 times</p>`.
  * También recuerda ejecutar este effect después de que hayas terminado: `() => { document.title = 'You clicked 1 times' }`.
* **React:** Claro. Actualizando la UI. Hey navegador, he cambiado el DOM.
* **Browser:** Super, dibujé tus cambios en la pantalla.
* **React:** Ok, ahora voy a ejecutar el effect que corresponde al render que acabo de hacer.
  * Ejecutando `() => { document.title = 'You clicked 1 times' }`.

---

## Cada Render Tiene Su Propio... Todo

**En este punto ya sabemos que los effects son ejecutados después de cada render, que son conceptualmente parte de la salida del componente, y que "ven" las propiedades y el estado de ese render en particular.**

Hagamos un experimento mental. Considera este código:

```jsx{4-8}
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setTimeout(() => {
      console.log(`You clicked ${count} times`);
    }, 3000);
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

Si hago click muchas veces con un leve retraso, ¿cómo se verá el log?

---

*vienen spoilers*

---

Puede que pienses que es una trampa y que el resultado es poco intuitivo. ¡No lo es! Veremos una secuencia de logs — cada uno perteneciente a su render particular y así con su propio valor de `count`. Puedes [probarlo tu mismo](https://codesandbox.io/s/lyx20m1ol):


![Grabación de pantalla de 1, 2, 3, 4, 5 registrados en orden](./timeout_counter.gif)

Puede que pienses: "¡Por supuesto que así es como funciona! ¿De qué otra forma podría funcionar?"

Pues bien, así no es como `this.state` funciona en clases. Es facil cometer el error de pensar que esta [implementación con clases](https://codesandbox.io/s/kkymzwjqz3) es equivalente:

```jsx
  componentDidUpdate() {
    setTimeout(() => {
      console.log(`You clicked ${this.state.count} times`);
    }, 3000);
  }
```

Sin embargo, `this.state.count` siempre apunta al *último* count en lugar apuntar al valor que corresponda a cada render. De este modo, vas a ver `5` aparecer en su lugar registrado cada vez:

![Grabación de pantalla 5, 5, 5, 5, 5 registrados en orden](./timeout_counter_class.gif)

Pienso que es irónico que los Hooks dependan tanto en closures de JavaScript, y aún así son las implementaciones con clases las que sufren de [la confusión canónica del valor-incorrecto-en-un-timeout](https://wsvincent.com/javascript-closure-settimeout-for-loop/) que frecuentemente es asociada con closures. Esto es porque la fuente de confusión en este ejemplo es la mutación (React muta `this.state` en clases para apuntar al ultimo estado) y no los closures en sí mismos.

**Los closures son grandiosos cuando los valores que encierras nunca cambian. Eso hace que sea fácil pensar en ellos porque esencialmente te estás refiriendo a constantes.** Y tal como discutimos, las propiedades y el estado nunca cambian dentro de un render en particular. Por cierto, podemos arreglar la versión que usa clases... [a través de closures](https://codesandbox.io/s/w7vjo07055).

## Nadando Contra Corriente

En este punto es importante que lo digamos en voz alta: **cada** función adentro del render de un componente (incluyendo menejadores de eventos, effects, timeouts o llamadas a API dentro de efectos) capturan las propiedades y el estado del render donde fueron definidos.

Por lo tanto estos dos ejemplos son equivalentes:

```jsx{4}
function Example(props) {
  useEffect(() => {
    setTimeout(() => {
      console.log(props.counter);
    }, 1000);
  });
  // ...
}
```

```jsx{2,5}
function Example(props) {
  const counter = props.counter;
  useEffect(() => {
    setTimeout(() => {
      console.log(counter);
    }, 1000);
  });
  // ...
}
```

**No importa si vas a leer de las propiedades o el estado "temprano" dentro de tu componente.** ¡Ninguno de ellos cambiará! Dentro del alcance de un render en particular, las propiedades y el estado se mantienen iguales. (Desestructurar las propiedades hace esto aún más obvio.)

Claro, algunas veces *quieres* leer el último valor en lugar del valor capturado dentro de algún callback definido en un effect. La forma más facil de lograrlo es a través de refs, tal como está descrito en la última sección de [este artículo](https://overreacted.io/how-are-function-components-different-from-classes/).

Ten presente que cuando quieres leer propiedades o estado del *futuro* desde una función de un render *pasado*, están nadando en contra de la corriente. No es *incorrecto* (y en algunos casos es necesario) pero puede ser que se vea menos "limpio" el romper el paradigma. Esta es una consecuencia  intencional pues ayuda a resaltar qué código es frágil y depende del tiempo. En clases, es menos obvio cuando esto sucede.

Aquí hay una [versión de nuestro ejemplo del contador](https://codesandbox.io/s/rm7z22qnlp) que replica el comportamiento de clases:

```jsx{3,6-7,9-10}
function Example() {
  const [count, setCount] = useState(0);
  const latestCount = useRef(count);

  useEffect(() => {
    // Set the mutable latest value
    latestCount.current = count;
    setTimeout(() => {
      // Read the mutable latest value
      console.log(`You clicked ${latestCount.current} times`);
    }, 3000);
  });
  // ...
```

![Captura de pantalla de 5, 5, 5, 5, 5 registrados en orden](./timeout_counter_refs.gif)

Mutar algo en React puede parecer peculiar. Sin embargo, esto es exactamente la forma en que React en sí mismo reasigna `this.state` en clases. Contrario al estado y propiedades capturados, no tienes ninguna garantía que leer `latestCount.current` te dará el mismo valor en un callback en particular. Por definición, puedes mutarlo en cualquier momento. Es por eso que no es un valor predeterminado sino algo que debes habilitar.

## ¿Qué Hay Acerca De La Limpieza?

Tal y como [lo explican los documentos](https://reactjs.org/docs/hooks-effect.html#effects-with-cleanup), algunos effects pueden tener una fase de limpieza. Esencialmente, su propósito es "deshacer" un efecto en casos como suscripciones.

Considera este código:

```jsx
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.id, handleStatusChange);
    };
  });
```

Imagina que `props` es `{id: 10}` en el primer renderizado, y `{id: 20}` en el segundo. *Puedes* pensar que algo así pasará:

* React limpia el effect para `{id: 10}`.
* React renderiza el UI para `{id: 20}`.
* React ejecuta el efecto para `{id: 20}`.

(Ese no es para nada el caso.)

Con este modelo mental, puedes pensar que la limpieza "ve" el valor viejo de props porque corre antes que volvamos a renderizar, y luego el nuevo effecto "ve" el nuevo valor de las propiedades porque corre después de volver a renderizar. Ese es el modelo mental tomado directamente de los métodos de ciclo de vida de las clases, y **no es acertado en este caso**. Veamos por qué.

React ejecuta los efectos solamente después de [dejar el navegador pintar su contenido](https://medium.com/@dan_abramov/this-benchmark-is-indeed-flawed-c3d6b5b6f97f). Esto hace que tu aplicación sea más rápida pues la mayoría de efectos no necesitan bloquear actualizaciones de pantalla. La limpieza del efecto también es retardada. **El effect previo es limpiado _después_ del re-render con las nuevas propiedades:**

* **React renderiza la UI para `{id: 20}`.**
* El navegador dibuja. Vemos la UI para `{id: 20}` en la pantalla.
* **React limpia el effect con valor `{id: 10}`.**
* React corre el effect para `{id: 20}`. 

Te estarás preguntando: pero ¿cómo puede el método de limpieza del render anterior "ver" el viejo valor de propiedades `{id: 10}` si es ejecutado *después* de que las propiedades cambiaran a `{id: 20}`?

Ya hemos estado aquí antes... 🤔

![Deja vu (escena del gato de la película Matrix)](./deja_vu.gif)

Citando a la sección anterior:

>Cada función dentro del render de un componente (incluyendo manejadores de envetos, effects, timeouts o llamadas al API desde dentro) captura las propiedades y el estado de la llamada de render que los definió.

¡Ahora la respuesta está clara! El método de limpieza del effect no lee las "últimas" propiedades, lo que sea que eso signifique. Lee las propiedades que pertenenen al render en el que fueron definidas:

```jsx{8-11}
// Primer render, props son {id: 10}
function Example() {
  // ...
  useEffect(
    // Effect del primer render
    () => {
      ChatAPI.subscribeToFriendStatus(10, handleStatusChange);
      // Función de limpieza del primer render
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(10, handleStatusChange);
      };
    }
  );
  // ...
}

// Siguiente render, las props son {id: 20}
function Example() {
  // ...
  useEffect(
    // Effect del segundo render
    () => {
      ChatAPI.subscribeToFriendStatus(20, handleStatusChange);
      // Función de limpieza del segundo render
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(20, handleStatusChange);
      };
    }
  );
  // ...
}
```

Reinados ascenderán y caeran, el Sol desprenderá sus capas externas para convertirse en una enana  blanca, y la última civilización llegará a su final. Pero nada hará que las propiedades "vistas" por el effect del primer render limpien algo diferente a `{id: 10}`.

Eso es lo que le permite a React tratar con effects justo después del dibujado — y hacer que tus aplicaciones sean rápidas por definición. Las propiedades viejas están allí si nuestro código las necesita.

## Sincronización Y No Ciclo De Vida

Una de mis cosas favoritas sobre React es que unifica la descripción del resultado y las propiedades del render inicial. Esto [reduce la entropía](https://overreacted.io/the-bug-o-notation/) de tu programa.

Digamos que mi componente se ve como esto:

```jsx
function Greeting({ name }) {
  return (
    <h1 className="Greeting">
      Hello, {name}
    </h1>
  );
}
```

No importa si renderizo `<Greeting name="Dan" />` y luego `<Greeting name="Yuzhi" />`, o si simplemente renderizo `<Greeting name="Yuzhi" />`. Al final, veremos "Hello, Yuzhi" en ambos casos.

Algunos dicen: "Lo importante es el camino, no el destino". Con React, es lo contrario. **Lo importante es el destino, no el camino.** Esa es la diferencia entre las llamadas a `$.addClass` y `$.removeClass` en jQuery (nuestro "camino") y el espeficiar cuál es la clase de CSS que *debe ser* en React (nuestro "destino")l

**React sincroniza el DOM de acuerdo al valor actual de las propiedades y el estado.** No hay distinción entre "montar" o "actualizar" cuando se trata de renderizar.

Debes ver a los effects de una forma similar. **`useEffect` te permite _sincronizar_ cosas fuera del árbol de React con base en las propiedades y el estado.**

```jsx{2-4}
function Greeting({ name }) {
  useEffect(() => {
    document.title = 'Hello, ' + name;
  });
  return (
    <h1 className="Greeting">
      Hello, {name}
    </h1>
  );
}
```

Esto es sutilmente diferente del modelo mental de *montar/actualizar/desmontar* al que estamos familiarizados. Es importante realmente internalizar esta idea. **Si estás intentando escribir un effect que se comporta diferente dependiendo de si el componente está siendo renderizado por primera vez o no, ¡entonces estás nadando contra corriente!** Estamos fallando si nuestro resultado dependen del "camino" y no del "destino".

No debiera importar si renderizamos con las propiedades A, B y C, o si renderizamos inmediatamente C. Aún cuando puede haber unas diferencias temporarles (por ejemplo, mientras obtenemos datos), eventualmente el resultado final debe ser el mismo.

Aún así, es claro que ejecutar todos los effects en *cada* render puede ser ineficient. (Y en algunos casos, llevará a bucles infinitos.)

Entonces, ¿cómo arreglamos esto?

## Entrenando A React A Diferenciar Tus Effects

Ya hemos aprendido esta lección con el DOM. En lugar de cambiarlo en cada renderizado, React solo actualiza las partes del DOM que cambiaron.

Cuando estás actualizando de

```jsx
<h1 className="Greeting">
  Hello, Dan
</h1>
```

a

```jsx
<h1 className="Greeting">
  Hello, Yuzhi
</h1>
```

React ve dos objetos:

```jsx
const oldProps = {className: 'Greeting', children: 'Hello, Dan'};
const newProps = {className: 'Greeting', children: 'Hello, Yuzhi'};
```

Va por cada una de sus propiedades y determina que `children` ha cambiado y necesita una actualización del DOM, pero `className` no cambió. Por lo que puede hacer solamente:

```jsx
domNode.innerText = 'Hello, Yuzhi';
// Sin necesidad de actualizar domNode.className
```

**¿Podemos hacer algo similar con los effects? Sería bueno poder evitar volver a ejecutarlos si no son necesarios**

Por ejemplo, tal vez nuestro componente re-renderiza a causa del cambio en un estado:

```jsx{11-13}
function Greeting({ name }) {
  const [counter, setCounter] = useState(0);

  useEffect(() => {
    document.title = 'Hello, ' + name;
  });

  return (
    <h1 className="Greeting">
      Hello, {name}
      <button onClick={() => setCounter(count + 1)}>
        Increment
      </button>
    </h1>
  );
}
```

Pero nuestro effect no utilizar el estado `counter`. **Nuestro effect sincroniza el `document.title` con la propiedad `name` pero dicha propiedad es la misma.** Re-asignar `document.title` por cada contador no parece algo ideal.

Está bien, entonces ¿puede React simplemente... diferenciar entre sus effects?


```jsx
let oldEffect = () => { document.title = 'Hello, Dan'; };
let newEffect = () => { document.title = 'Hello, Dan'; };
// Puede React ver que estas funciones están haciendo la misma cosa?
```

No realmente. React no puede adivinar qué es lo que hace la función sin llamarla. (La implementación no contiene valor específicos, solamente utiliza el valor de la propiedad `name`)

Esta es la razón por la cual si deseas evitar re-ejecutar effects de manera innecesaria, puedes proporcionar un arreglo de dependencias (también conocido como "deps") en `useEffect`:

```jsx{3}
  useEffect(() => {
    document.title = 'Hello, ' + name;
  }, [name]); // Nuestras dependencias
```

**Es como si le dijeramos a React: "Hey, ya sé que no puedes ver _adentro_ de esta función, pero te prometo que solamente utiliza `name` y nada más de las propiedades de ese render."**

Si cada uno de estos valores es el mismo entre la ejecución del effect actual y el anterior, entonces no hay nada que sincronizar y React puede ignorar ese effect:

```jsx
const oldEffect = () => { document.title = 'Hello, Dan'; };
const oldDeps = ['Dan'];

const newEffect = () => { document.title = 'Hello, Dan'; };
const newDeps = ['Dan'];

// React no puede ver adentro de las funciones, pero puede comparar sus dependencias
// Dado que todas las dependencias son las mismas, no necesita ejecutar el `nuevo effect`
```

Si tan solo uno de los valores en el arreglo de dependencias es diferente entre renders, entonces no debemos ignorar el `useEffect`. Sincroniza todo!

## No Le Mientas a React Sobre Las Dependencias

Mentirle a React sobre las dependencias tiene malas consecuencias. Intuitivamente, puede parecer lógico hacerlo, pero he visto casi a todos los que usan `useEffect` aplicando el modelo mental de clases tratar de hacer trampa. (¡Yo también lo hice al incio!)

```jsx
function SearchResults() {
  async function fetchData() {
    // ...
  }

  useEffect(() => {
    fetchData();
  }, []); // ¿Es correcto hacer esto? No siempre -- y hay una mejor forma de escribirlo

  // ...
}
```

*(El [FAQ de Hooks](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) explica qué hacer en estos casos. Vamos a volver a revisar este ejemplo [abajo](#moving-functions-inside-effects).)*

"¡Pero yo solo quiero ejecutarlo cuando se monte el componente!", podrías decir. Por ahora, recuerda: si especificas dependencias, **_todos_ los valores dentro de tu componente que son utilizados por el effect _deben_ estar allí**. Incluyendo propiedades, estado, funciones — cualquier cosa en tu componente.

En ocasiones cuando lo haces, ocasiona un problema. Por ejemplo, puede ser que hayas visto un ciclo infinito al traer datos, o un socket que es creado con mucha frecuencia. **La solución a ese problema _no_ es remover la dependencia.** Pronto veremos cuáles son las soluciones.

Pero antes de saltar a las soluciones, entendamos el problema.

## Lo Que Sucede Cuando Las Dependencias Mienten

Si las dependencias contienen cada valor utilizado por el effect, React sabe cuando correrlo de nuevo:

```jsx{3}
  useEffect(() => {
    document.title = 'Hello, ' + name;
  }, [name]);
```

![Diagrama de effects remplazándose entre si](./deps-compare-correct.gif)

*(Las dependencias son diferentes, entonces ejecutamos de nuevo el effect)*

Pero si especificamos `[]` para este effect, la nueva función effect no se ejecutaría:

```jsx{3}
  useEffect(() => {
    document.title = 'Hello, ' + name;
  }, []); // Incorrecto: name hace falta en las dependencias
```

![Diagrama de effects remplazándose entre si](./deps-compare-wrong.gif)

*(Las dependencias son iguales, entonces ignoramos este effect.)*

En este caso, el problema puede parecer obvio. Pero la intuición puede engañarte en otras ocasiones cuando una solución que usarías para classes "salta" de tu memoria.

Por ejemplo, digamos que estamos escribiendo un contador que incrementa cada segundo. En el caso de una clase, nuestra intuición es: "Definir el intervalo una vez y destruilo una vez". Aquí hay un [ejemplo](https://codesandbox.io/s/n5mjzjy9kl) de cómo podemos hacerlo. Cuando traducimos mentalmente este código a `useEffect`, instintivamente agregamos `[]` a las dependencias. "Quiero que se ejecute solo una vez", ¿cierto?

```jsx{9}
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}
```

Sin embargo, este ejemplo [solo *incrementa* una vez](https://codesandbox.io/s/91n5z8jo7r). *Rayos.*

Si tu modelo mental es "las dependencias me permiten especificar cuando quiero volver a llamar al effect", este ejemplo te debe estar ocasionando una crisis existencial. Tu *quieres* llamarlo una vez porque es un interval — entonces ¿por qué no está funcionando?

Sin embargo, esto hace sentido si sabes que las dependencias son nuestra pista para reaccionar a *todo* lo que el efecto usa dentro del alcance del render. Usa `count` pero mentimos diciendo que no lo usa cuando pusimos `[]`. ¡Solo es cuestión de tiempo para que nos muerda!

En el primer render, `count` es `0`. Por lo tanto, `setCount(count + 1)` en el effect del primer render es `setCount(0 + 1)`. **Dado que nunca volvemos a ejecutar el effect a causa de `[]` en las dependencias, el interval continuará llamando `setCount(0 + `)` cada segundo:**

```jsx{8,12,21-22}
// Primer render, el estado es 0
function Counter() {
  // ...
  useEffect(
    // Effect del primer render
    () => {
      const id = setInterval(() => {
        setCount(0 + 1); // Siempre setCount(1)
      }, 1000);
      return () => clearInterval(id);
    },
    [] // Nunca se vuelve a ejecutar
  );
  // ...
}

// Para cada render siguiente, el estado es 1
function Counter() {
  // ...
  useEffect(
    // Este effect siempre será ignorado porque
    // le mentimos a React al colocar [] en las dependencias
    () => {
      const id = setInterval(() => {
        setCount(1 + 1);
      }, 1000);
      return () => clearInterval(id);
    },
    []
  );
  // ...
}
```

Le mentimos a React al decirle que nuestro effect no depende de un valor dentro de nuestro componente, ¡cuando de hecho si lo hace!

Nuestro effect usa `count` — un valor que existe adentro de nuestro componente (pero fuera del effect):

```jsx{1,5}
  const count = // ...

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
```

Por lo tanto, al especificar `[]` como dependencia creamos un error. React va a comparar las dependencias y va a ignorar actualizar este effect:

![Diagrama de un intervalo con closure incorrecto](./interval-wrong.gif)

*(Las dependencias son iguales, entonces nos saltamos la ejecución del effect)*

Problemas como estos son difíciles de conceptualizar. Por lo tanto, les recomiendo que adopten como una regla de manera estricta siempre ser honestos sobre las dependencias de sus effects y especificarlas todas. (Proveemos una [regla de lint](https://github.com/facebook/react/issues/14920) si quieres forzar esto en tu equipo.)

## Dos Formas De Ser Honestos Acerca De Las Dependencias

Existen dos estrateigas para ser honestos acerca de las dependencias. En principio, deberías iniciar siempre con la primera estrategia, y luego aplicar la segunda si fuera necesario.

**La primera estrategia es corregir el arreglo de dependencias haciendo que incluya _todos_ los valores dentro del componente que son usados por el effect.** Incluyamos `count` como dependencia:

```jsx{3,6}
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(id);
}, [count]);
```

Esto hace que el arreglo de dependencias esté correcto. Puede que no sea *ideal* pero ese es el primer problema que debíamos arreglar. Ahora un cambio en `count` hará que se vuelva a ejecutar el effect, haciendo que cada interval se refiera al `count` de su render en `setCount(count + 1)`:

```jsx{8,12,24,28}
// Primer render, el estado es 0
function Counter() {
  // ...
  useEffect(
    // Effect del primer render
    () => {
      const id = setInterval(() => {
        setCount(0 + 1); // setCount(count + 1)
      }, 1000);
      return () => clearInterval(id);
    },
    [0] // [count]
  );
  // ...
}

// Segundo render, el valor de count es 1
function Counter() {
  // ...
  useEffect(
    // Effect del segundo render
    () => {
      const id = setInterval(() => {
        setCount(1 + 1); // setCount(count + 1)
      }, 1000);
      return () => clearInterval(id);
    },
    [1] // [count]
  );
  // ...
}
```

Eso podría [arreglar el problema](https://codesandbox.io/s/0x0mnlyq8l) pero nuestro intervalo sería limpiado y re-definido cada vez que `count` cambie. Eso puede ser indeseable:

![Diagrama de intervalo que se re-suscribe](./interval-rightish.gif)

*(Las dependencias son diferentes, entonces ejecutamos de nuevo el effect.)*

---

**La segunda estrategia es cambiar el código de nuestro effect de manera que no *necesite* un valor que cambie más frecuente de lo que queremos.** No queremos mentir acerca de las dependencias — solo queremos cambiar nuestro effect para que tenga *menos*.

Veamos algunas técnicas comunes para remover dependencias.

---

## Haciendo Que Los Effects Sean Auto-Suficientes

Queremos que `count` ya no sea una dependencia de nustro effect.

```jsx{3,6}
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, [count]);
```

Para lograrlo, debemos preguntarnos: **¿Para qué estamos usando `count`?** Parece que solo lo estamos utilizando en la llamada a `setCount`. En este caso, no necesitamos contar con `count` en este contexto. Cuando queremos actualizar el estado con base en su valor anterior, podemos usar la [forma funcional](https://reactjs.org/docs/hooks-reference.html#functional-updates) de `setState`:

```jsx{3}
  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
```

Me gusta conceptualizar estos casos como "falsas dependencias". Si, `count` era una dependencia necesaria porque estabamos usando `setCount(count + 1)` adentro del effect. Sin embargo, en realidad solo necesitamos `count`  para transformarlo en `count + 1` y "mandarlo de vuelta" a React. Pero React *ya tiene* el valor actual de `count`. **Todo lo que necesitamos decirle a React es que incremente el estado — cualquiera que sea just ahora.**

Eso es justo lo que `setCount(c => c + 1)` hace. Puedes verlo como "enviarle una instrucción" a React acerca de cómo debe cambiar el estado. Esta "forma de actualización" también ayuda en otros casos, como cuando tienes que [agrupar múltiples actualizaciones](/react-as-a-ui-runtime/#batching).

**Notar que de hecho _cumplimos_ con remover la dependencia. No hicimos trampa. Nuestro effect ya no lee el valor de `counter` dentro del alcance de este render:**

![Diagrama de un interval que funciona](./interval-right.gif)

*(Las dependencias son iguales, ignoramos este effect)*

Puedes verlo funcionando [aquí](https://codesandbox.io/s/q3181xz1pj).

Aun cuando este effect solo se ejecuta una  vez, el callback del interval que pertenece al primer render es perfectamente capaz de enviar la instrucción de actualización `c => c + 1` cada vez que el intervalo se dispare. Ya no necesita saber el valor actual del `counter`. React ya lo sabe.

## Actualizaciones A Través De Funciones Y Google Docs


¿Recuerdan que dijimos que la sincronziación era el modelo mental para los effects? Un aspecto interesante de la sincronización es que frecuentemente se quiere que los "mensajes" entre sistemas estén desenredados de su estado. Por ejemplo, al editar un documento en Google Docs no se envía *toda* la página al servidor. Eso sería muy ineficiente. En su lugare, envía una representación de lo que el usuario intentó hacer.

Aun cuando nuestro caso de uso es diferente, podemos aplicar una filosofía similar a los effects. **Ayuda enviar solo la información mínima necesaria desde dentro de los effects hacia el componente.** La forma de actualización `setCount(c => c + 1)` transmite estrictamente menos información que `setCount(count + 1)` porque no está "contaminada" con el valor actual de count. Solo expresa la acción ("incrementar"). Pensar en React involucra [encontrar el estado mínimo](https://reactjs.org/docs/thinking-in-react.html#step-3-identify-the-minimal-but-complete-representation-of-ui-state). Este es el mismo principio, pero para actualizaciones.

Codificar la *intención* (en lugar del resultado) es similar a la manera en que Google Docs [resuleve](https://medium.com/@srijancse/how-real-time-collaborative-editing-work-operational-transformation-ac4902d75682) la edición colaborativa. Mientras que estas es una analogía bastante ajustada, las actualizaciones a través de funciones juegan un rol similar en React. Estas se aseguran que actualizaciones desde diferentes fuentes (manejadores de eventos, suscripciones a effects, etc) puedan ser aplicadas correctamente en lote y de manera predictiva.

**Sin embargo, incluso `setCount(c => c + 1)` no es lo mejor.** Se ve un poco raro y también es muy limitado. Por ejemplo, si tuvieramos dos variables de estado que dependieran una de la otra, o si tuvieramos que calcular el siguiente  estado con base en una propiedad, no nos ayudaría. Por suerte, `setCount(c => c + 1)` tiene un patrón hermano que es más poderoso. Su nombre es `useReducer`.

## Desasociando Actualizaciones de las Acciones

Modifiquemos el ejemplo previo de manera que tengamos dos variables en el estado: `count` y `step`. Nuestro intervalo va a incrementar count con el valor ingresado en el input para `step`:

```jsx{7,10}
function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + step);
    }, 1000);
    return () => clearInterval(id);
  }, [step]);

  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={e => setStep(Number(e.target.value))} />
    </>
  );
}
```

(Aquí hay un [demo](https://codesandbox.io/s/zxn70rnkx).)

Notar que **no estamos haciendo trampa**. Dado que comencé a usar `step` adentro del effect, lo agregué a las dependencias. Y esa es la razón del por qué el código se ejecuta correctamente.

El comportamiento actual en este ejemplo es que cuando cambio el `step` se reinicia el interval — porque es una de sus dependencias. Y en muchos casos, ¡eso es justo lo que quieres! No hay nada de malo con desarmar un effect y armarlo de nuevo, y no deberíamos evitarlo a menos que tengamos una buena razón para hacerlo.

Sin embargo, digamos que queremos que el reloj del intervalo no se resetee cuando cambie `step`. ¿Cómo quitamos a `step` de nuestras dependencias?

**Cuando el valor de una variable de estado depende del valor actual de otra variable  de estado, puede que quieras reemplazar ambas por `useReducer`.**

Cuando te encuentres escribiendo `setSomething(something => ...)`, es un buen momento para considerar usar en su lugar un reducer. Los reducers te permiten **desasociar las "acciones" que sucedieron en tu componente de cómo el estado debe actualizarse en respuesta a esas acciones**.

Intercambiemos la dependencia `step` por una dependencia de `dispatch` en nuestro efecto:

```jsx{1,6,9}
const [state, dispatch] = useReducer(reducer, initialState);
const { count, step } = state;

useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: 'tick' }); // En lugar de setCount(c => c + step);
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```

(Ver la [demo](https://codesandbox.io/s/xzr480k0np).)

Me puedes preguntar: "¿Cómo es que esto es mejor?" La respuesta es que **React garantiza que la función `dispatch` es constante a lo largo de la vida del componente. Entonces el ejemplo anterior no necesita nunca volver a resuscribir el intervalo.**

¡Resolvimos nuestro problema!

*(Puedes omitir `dispatch`, `setState` y `useRef` de las dependencias porque React garantiza que son estáticas. De cualquier modo, no causaremos ningún daño si lo hacemos)*

En lugar de leer el estado *dentro* del effect, dispara una *acción* que codifica la información acerca de *qué sucedió*. Esto le permite a nuestro effect mantenerse desasociado del estado `step`. A nuestro effect no le importa *cómo* actualizamos el estado, solo nos dice *lo que sucedió*. Y el reducer centraliza la lógica de actualización:

```jsx{8,9}
const initialState = {
  count: 0,
  step: 1,
};

function reducer(state, action) {
  const { count, step } = state;
  if (action.type === 'tick') {
    return { count: count + step, step };
  } else if (action.type === 'step') {
    return { count, step: action.step };
  } else {
    throw new Error();
  }
}
```

(Aquí hay una [demo](https://codesandbox.io/s/xzr480k0np) en caso que no la hayas visto antes).

## Por Qué useReducer Es La Modalidad Tramposa De Los Hooks

Ya vimos cómo remover dependencias cuando un effect necesita actualizar algún estado con base en un estado previo o con base en otro estado. **Pero ¿qué pasa si necesitamos de _propiedades_ para calcular el valor del estado?** Por ejemplo, tal vez nuestra API es `<Counter step={1} />`. Seguramente, en este caso no podemos evitar especificar `props.step` en las dependencias, ¿o si podemos?

¡Si podemos! Podemos colocar *la función reducer* dentro de nuestro componente para que pueda leer las propiedades:

```jsx{1,6}
function Counter({ step }) {
  const [count, dispatch] = useReducer(reducer, 0);

  function reducer(state, action) {
    if (action.type === 'tick') {
      return state + step;
    } else {
      throw new Error();
    }
  }

  useEffect(() => {
    const id = setInterval(() => {
      dispatch({ type: 'tick' });
    }, 1000);
    return () => clearInterval(id);
  }, [dispatch]);

  return <h1>{count}</h1>;
}
```

Este patrón nos impide realizar algunas optimizaciones entonces trata de no utilizarlo siempre, pero definitivamente puedes acceder a las propiedades desde un reducer si asi lo necesitas. (Aquí hay una [demo](https://codesandbox.io/s/7ypm405o8q).)

**Aún en este caso, está garantizado que `dispatch` será estable entre re-renders.** Por lo tanto, puedes omitirla de las dependencias si así lo deseas. No va a causar que el effect vuelva a ejecutarse.

Puede que te estés preguntando: ¿cómo puede ser que esto funcione? ¿Cómo puede ser que el reducer "conozca" las propiedades cuando se le está llamando desde dentro de un effect que pertenence a otro render? La respuesta es que cuando tu `dispatch`, React recuerda la acción — pero va a *llamar* tu reducer durante el siguiente render. En ese punto, el valor actualizado de las propiedades estará en contexto, y tu no estaras dentro de un effect.

**Esta es la razón del por qué me gusta pensar que `useReducer` es la forma tramposa de Hooks. Me permite desacoplar la lógica de actualización del describir qué pasó. Esto, a su vez, permite remover dependencias innecesarias de mis effects y prevenir que se vuelvan a ejecutar más de lo necesario.**

## Moviendo Funciones Dentro De Effects

Un error común es pensar quelas funciones no deben ser dependencias. Por ejemplo, esto pareciera que debe funcionar bien:

```jsx{13}
function SearchResults() {
  const [data, setData] = useState({ hits: [] });

  async function fetchData() {
    const result = await axios(
      'https://hn.algolia.com/api/v1/search?query=react',
    );
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []); // Está correcto?

  // ...
```

*([Este ejemplo](https://codesandbox.io/s/8j4ykjyv0) está adaptado de un excelente artículo escrito por Robin Wieruch — [¡dale un vistazo!](https://www.robinwieruch.de/react-hooks-fetch-data/)!)*

Y solo para aclarar, este código *funciona*. **Pero el problema con omitir funciones locales es que se vuelve muy difícil determinar si estamos considerando todos los casos con forme el componente crece.**

Imagina que nuestro código está partido de esta manera y que cada función es cinco veces más larga:

```jsx
function SearchResults() {
  // Imagina que esta función es extensa
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=react';
  }

  // Imagina que esta función también es extensa
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []);

  // ...
}
```


Ahora digamos que más adelante usamos estado o propiedades en una de estas funciones:

```jsx{6}
function SearchResults() {
  const [query, setQuery] = useState('react');

  // Imagina que esta función es extensa
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  // Imagina que esta función también es extensa
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []);

  // ...
}
```

If we forget to update the deps of any effects that call these functions (possibly, through other functions!), our effects will fail to synchronize changes from our props and state. This doesn’t sound great.

Luckily, there is an easy solution to this problem. **If you only use some functions *inside* an effect, move them directly *into* that effect:**

```jsx{4-12}
function SearchResults() {
  // ...
  useEffect(() => {
    // We moved these functions inside!
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=react';
    }

    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }

    fetchData();
  }, []); // ✅ Deps are OK
  // ...
}
```

([Here’s a demo](https://codesandbox.io/s/04kp3jwwql).)

So what is the benefit? We no longer have to think about the “transitive dependencies”. Our dependencies array isn’t lying anymore: **we truly _aren’t_ using anything from the outer scope of the component in our effect**.

If we later edit `getFetchUrl` to use the `query` state, we’re much more likely to notice that we’re editing it *inside* an effect — and therefore, we need to add `query` to the effect dependencies:

```jsx{6,15}
function SearchResults() {
  const [query, setQuery] = useState('react');

  useEffect(() => {
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=' + query;
    }

    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }

    fetchData();
  }, [query]); // ✅ Deps are OK

  // ...
}
```

(Here’s a [demo](https://codesandbox.io/s/pwm32zx7z7).)

By adding this dependency, we’re not just “appeasing React”. It *makes sense* to refetch the data when the query changes. **The design of `useEffect` forces you to notice the change in our data flow and choose how our effects should synchronize it — instead of ignoring it until our product users hit a bug.**

Thanks to the `exhaustive-deps` lint rule from the `eslint-plugin-react-hooks` plugin, you can [analyze the effects as you type in your editor](https://github.com/facebook/react/issues/14920) and receive suggestions about which dependencies are missing. In other words, a machine can tell you which data flow changes aren’t handled correctly by a component.

![Lint rule gif](./exhaustive-deps.gif)

Pretty sweet.

## But I Can’t Put This Function Inside an Effect

Sometimes you might not want to move a function *inside* an effect. For example, several effects in the same component may call the same function, and you don’t want to copy and paste its logic. Or maybe it’s a prop.

Should you skip a function like this in the effect dependencies? I think not. Again, **effects shouldn’t lie about their dependencies.** There are usually better solutions. A common misconception is that “a function would never change”. But as we learned throughout this article, this couldn’t be further from truth. Indeed, a function defined inside a component changes on every render!

**That by itself presents a problem.** Say two effects call `getFetchUrl`:

```jsx
function SearchResults() {
  function getFetchUrl(query) {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, []); // 🔴 Missing dep: getFetchUrl

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, []); // 🔴 Missing dep: getFetchUrl

  // ...
}
```

In that case you might not want to move `getFetchUrl` inside either of the effects since you wouldn’t be able to share the logic.

On the other hand, if you’re “honest” about the effect dependencies, you may run into a problem. Since both our effects depend on `getFetchUrl` **(which is different on every render)**, our dependency arrays are useless:

```jsx{2-5}
function SearchResults() {
  // 🔴 Re-triggers all effects on every render
  function getFetchUrl(query) {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // 🚧 Deps are correct but they change too often

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // 🚧 Deps are correct but they change too often

  // ...
}
```

A tempting solution to this is to just skip the `getFetchUrl` function in the deps list. However, I don’t think it’s a good solution. This makes it difficult to notice when we *are* adding a change to the data flow that *needs* to be handled by an effect. This leads to bugs like the “never updating interval” we saw earlier.

Instead, there are two other solutions that are simpler.

**First of all, if a function doesn’t use anything from the component scope, you can hoist it outside the component and then freely use it inside your effects:**

```jsx{1-4}
// ✅ Not affected by the data flow
function getFetchUrl(query) {
  return 'https://hn.algolia.com/api/v1/search?query=' + query;
}

function SearchResults() {
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, []); // ✅ Deps are OK

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, []); // ✅ Deps are OK

  // ...
}
```

There’s no need to specify it in deps because it’s not in the render scope and can’t be affected by the data flow. It can’t accidentally depend on props or state.

Alternatively, you can wrap it into the [`useCallback` Hook](https://reactjs.org/docs/hooks-reference.html#usecallback):


```jsx{2-5}
function SearchResults() {
  // ✅ Preserves identity when its own deps are the same
  const getFetchUrl = useCallback((query) => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, []);  // ✅ Callback deps are OK

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  // ...
}
```

`useCallback` is essentially like adding another layer of dependency checks. It’s solving the problem on the other end — **rather than avoid a function dependency, we make the function itself only change when necessary**.

Let's see why this approach is useful. Previously, our example showed two search results (for `'react'` and `'redux'` search queries). But let's say we want to add an input so that you can search for an arbitrary `query`. So instead of taking `query` as an argument, `getFetchUrl` will now read it from local state.

We'll immediately see that it's missing a `query` dependency:

```jsx{5}
function SearchResults() {
  const [query, setQuery] = useState('react');
  const getFetchUrl = useCallback(() => { // No query argument
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, []); // 🔴 Missing dep: query
  // ...
}
```

If I fix my `useCallback` deps to include `query`, any effect with `getFetchUrl` in deps will re-run whenever the `query` changes:

```jsx{4-7}
function SearchResults() {
  const [query, setQuery] = useState('react');

  // ✅ Preserves identity until query changes
  const getFetchUrl = useCallback(() => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, [query]);  // ✅ Callback deps are OK

  useEffect(() => {
    const url = getFetchUrl();
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  // ...
}
```

Thanks to `useCallback`, if `query` is the same, `getFetchUrl` also stays the same, and our effect doesn't re-run. But if `query` changes, `getFetchUrl` will also change, and we will re-fetch the data. It's a lot like when you change some cell in an Excel spreadsheet, and the other cells using it recalculate automatically.

This is just a consequence of embracing the data flow and the synchronization mindset. **The same solution works for function props passed from parents:**

```jsx{4-8}
function Parent() {
  const [query, setQuery] = useState('react');

  // ✅ Preserves identity until query changes
  const fetchData = useCallback(() => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + query;
    // ... Fetch data and return it ...
  }, [query]);  // ✅ Callback deps are OK

  return <Child fetchData={fetchData} />
}

function Child({ fetchData }) {
  let [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // ✅ Effect deps are OK

  // ...
}
```

Since `fetchData` only changes inside `Parent` when its `query` state changes, our `Child` won’t refetch the data until it’s actually necessary for the app.

## Are Functions Part of the Data Flow?

Interestingly, this pattern is broken with classes in a way that really shows the difference between the effect and lifecycle paradigms. Consider this translation:

```jsx{5-8,18-20}
class Parent extends Component {
  state = {
    query: 'react'
  };
  fetchData = () => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + this.state.query;
    // ... Fetch data and do something ...
  };
  render() {
    return <Child fetchData={this.fetchData} />;
  }
}

class Child extends Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.props.fetchData();
  }
  render() {
    // ...
  }
}
```

You might be thinking: “Come on Dan, we all know that `useEffect` is like `componentDidMount` and `componentDidUpdate` combined, you can’t keep beating that drum!” **Yet this doesn’t work even with `componentDidUpdate`:**

```jsx{8-13}
class Child extends Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.props.fetchData();
  }
  componentDidUpdate(prevProps) {
    // 🔴 This condition will never be true
    if (this.props.fetchData !== prevProps.fetchData) {
      this.props.fetchData();
    }
  }
  render() {
    // ...
  }
}
```

Of course, `fetchData` is a class method! (Or, rather, a class property — but that doesn’t change anything.) It’s not going to be different because of a state change. So `this.props.fetchData` will stay equal to `prevProps.fetchData` and we’ll never refetch. Let’s just remove this condition then?

```jsx
  componentDidUpdate(prevProps) {
    this.props.fetchData();
  }
```

Oh wait, this fetches on *every* re-render. (Adding an animation above in the tree is a fun way to discover it.) Maybe let’s bind it to a particular query?

```jsx
  render() {
    return <Child fetchData={this.fetchData.bind(this, this.state.query)} />;
  }
```

But then `this.props.fetchData !== prevProps.fetchData` is *always* `true`, even if the `query` didn’t change! So we’ll *always* refetch.

The only real solution to this conundrum with classes is to bite the bullet and pass the `query` itself into the `Child` component. The `Child` doesn’t actually end up *using* the `query`, but it can trigger a refetch when it changes:

```jsx{10,22-24}
class Parent extends Component {
  state = {
    query: 'react'
  };
  fetchData = () => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + this.state.query;
    // ... Fetch data and do something ...
  };
  render() {
    return <Child fetchData={this.fetchData} query={this.state.query} />;
  }
}

class Child extends Component {
  state = {
    data: null
  };
  componentDidMount() {
    this.props.fetchData();
  }
  componentDidUpdate(prevProps) {
    if (this.props.query !== prevProps.query) {
      this.props.fetchData();
    }
  }
  render() {
    // ...
  }
}
```

Over the years of working with classes with React, I’ve gotten so used to passing unnecessary props down and breaking encapsulation of parent components that I only realized a week ago why we had to do it.

**With classes, function props by themselves aren’t truly a part of the data flow.** Methods close over the mutable `this` variable so we can’t rely on their identity to mean anything. Therefore, even when we only want a function, we have to pass a bunch of other data around in order to be able to “diff” it. We can’t know whether `this.props.fetchData` passed from the parent depends on some state or not, and whether that state has just changed.

**With `useCallback`, functions can fully participate in the data flow.** We can say that if the function inputs changed, the function itself has changed, but if not, it stayed the same. Thanks to the granularity provided by `useCallback`, changes to props like `props.fetchData` can propagate down automatically.

Similarly, [`useMemo`](https://reactjs.org/docs/hooks-reference.html#usememo) lets us do the same for complex objects:

```jsx
function ColorPicker() {
  // Doesn't break Child's shallow equality prop check
  // unless the color actually changes.
  const [color, setColor] = useState('pink');
  const style = useMemo(() => ({ color }), [color]);
  return <Child style={style} />;
}
```

**I want to emphasize that putting `useCallback` everywhere is pretty clunky.** It’s a nice escape hatch and it’s useful when a function is both passed down *and* called from inside an effect in some children. Or if you’re trying to prevent breaking memoization of a child component. But Hooks lend themselves better to [avoiding passing callbacks down](https://reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down) altogether.

In the above examples, I’d much prefer if `fetchData` was either inside my effect (which itself could be extracted to a custom Hook) or a top-level import. I want to keep the effects simple, and callbacks in them don’t help that. (“What if some `props.onComplete` callback changes while the request was in flight?”) You can [simulate the class behavior](#swimming-against-the-tide) but that doesn’t solve race conditions.

## Speaking of Race Conditions

A classic data fetching example with classes might look like this:

```jsx
class Article extends Component {
  state = {
    article: null
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id);
    this.setState({ article });
  }
  // ...
}
```

As you probably know, this code is buggy. It doesn’t handle updates. So the second classic example you could find online is something like this:

```jsx{8-12}
class Article extends Component {
  state = {
    article: null
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchData(this.props.id);
    }
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id);
    this.setState({ article });
  }
  // ...
}
```

This is definitely better! But it’s still buggy. The reason it’s buggy is that the request may come out of order. So if I’m fetching `{id: 10}`, switch to `{id: 20}`, but the `{id: 20}` request comes first, the request that started earlier but finished later would incorrectly overwrite my state.

This is called a race condition, and it’s typical in code that mixes `async` / `await` (which assumes something waits for the result) with top-down data flow (props or state can change while we’re in the middle of an async function).

Effects don’t magically solve this problem, although they’ll warn you if you try to pass an `async` function to the effect directly. (We’ll need to improve that warning to better explain the problems you might run into.)

If the async approach you use supports cancellation, that’s great! You can cancel the async request right in your cleanup function.

Alternatively, the easiest stopgap approach is to track it with a boolean:

```jsx{5,9,16-18}
function Article({ id }) {
  const [article, setArticle] = useState(null);

  useEffect(() => {
    let didCancel = false;

    async function fetchData() {
      const article = await API.fetchArticle(id);
      if (!didCancel) {
        setArticle(article);
      }
    }

    fetchData();

    return () => {
      didCancel = true;
    };
  }, [id]);

  // ...
}
```

[This article](https://www.robinwieruch.de/react-hooks-fetch-data/) goes into more detail about how you can handle errors and loading states, as well as extract that logic into a custom Hook. I recommend you to check it out if you’re interested to learn more about data fetching with Hooks.

## Raising the Bar

With the class lifecycle mindset, side effects behave differently from the render output. Rendering the UI is driven by props and state, and is guaranteed to be consistent with them, but side effects are not. This is a common source of bugs.

With the mindset of `useEffect`, things are synchronized by default. Side effects become a part of the React data flow. For every `useEffect` call, once you get it right, your component handles edge cases much better.

However, the upfront cost of getting it right is higher. This can be annoying. Writing synchronization code that handles edge cases well is inherently more difficult than firing one-off side effects that aren’t consistent with rendering.

This could be worrying if `useEffect` was meant to be *the* tool you use most of the time. However, it’s a low-level building block. It’s an early time for Hooks so everybody uses low-level ones all the time, especially in tutorials. But in practice, it’s likely the community will start moving to higher-level Hooks as good APIs gain momentum.

I’m seeing different apps create their own Hooks like `useFetch` that encapsulates some of their app’s auth logic or `useTheme` which uses theme context. Once you have a toolbox of those, you don’t reach for `useEffect` *that* often. But the resilience it brings benefits every Hook built on top of it.

So far, `useEffect` is most commonly used for data fetching. But data fetching isn’t exactly a synchronization problem. This is especially obvious because our deps are often `[]`. What are we even synchronizing?

In the longer term, [Suspense for Data Fetching](https://reactjs.org/blog/2018/11/27/react-16-roadmap.html#react-16x-mid-2019-the-one-with-suspense-for-data-fetching) will allow third-party libraries to have a first-class way to tell React to suspend rendering until something async (anything: code, data, images) is ready.

As Suspense gradually covers more data fetching use cases, I anticipate that `useEffect` will fade into background as a power user tool for cases when you actually want to synchronize props and state to some side effect. Unlike data fetching, it handles this case naturally because it was designed for it. But until then, custom Hooks like [shown here](https://www.robinwieruch.de/react-hooks-fetch-data/) are a good way to reuse data fetching logic.

## In Closing

Now that you know pretty much everything I know about using effects, check out the [TLDR](#tldr) in the beginning. Does it make sense? Did I miss something? (I haven’t run out of paper yet!)

I’d love to hear from you on Twitter! Thanks for reading.
