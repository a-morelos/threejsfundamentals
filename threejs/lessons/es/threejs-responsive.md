Title: Diseño responsivo con three.js
Description: Cómo hacer que three.js funcione en distintos tamaños de display.
TOC: Diseño responsivo

Este es el segundo de una serie de artículos sobre three.js.
El primero fue sobre [los fundamentos](threejs-fundamentals.html).
Si no lo has leído, quizás quieras comenzar por ahí.

Este artículo trata sobre como hacer responsiva tu app three.js para
cualquier situación. Hacer una página web responsiva normalmente se
refiere a que la página se muestre correctamente en distintos tamaños
de display desde computadoras, tablets y teléfonos.

Para three.js hay más situaciones que considerar. Por ejemplo, un editor 3D
con controles a la izquierda, derecha, arriba o abajo es algo que deberíamos
considerar. Un diagrama dinámico en tiempo real en medio de un documento es
otro ejemplo.

El último ejemplo que usamos fue un canvas plano sin CSS ni tamaño definido.

```html
<canvas id="c"></canvas>
```

El tamaño por defecto del canvas es de 300x150 pixeles de CSS.

En la web la forma recomendada de establecer el tamaño de algo es
mediante CSS.

Hagamos que el canvas ocupe la página completa usando CSS.

```html
<style>
html, body {
   margin: 0;
   height: 100%;
}
#c {
   width: 100%;
   height: 100%;
   display: block;
}
</style>
```

En HTML el body tiene un margen de 5 pixeles por defecto por lo que
establecer el margen a 0 elimina el margen. Estableciendo el ancho del html
y el body a 100% llena a ventana. De otra forma son tan grandes como el contenido
que llenan.

A continuación le indicamos al elemento `id=c`que sea del 100% del
tamaño de su contenedor que en este caso es el body del documento.

Por último establecemos el modo `display` a `block`. El
modo display del canvas es `inline`. Los elementos inline
pueden agregar un espacio en blanco al final. Este problema
se corrige al establecer el canvas a `block`.

He aquí el resultado.

{{{example url="../threejs-responsive-no-resize.html" }}}

Puedes ver como el canvas ahora ocupa la página, pero hay
2 problemas. Uno de nuestro cubos es algo estrecho. No son cubos,
son más bien cajas. Muy altas o muy anchas. Abre el ejemplo en su
propia ventana y cambia el tamaño. Verás como los cubos se estrechan
del alto o el ancho.

<img src="resources/images/resize-incorrect-aspect.png" width="407" class="threejs_center nobg">

El segundo problema es como se ven como en bloques o difumindado.
Estira la ventana y verás el problema.

<img src="resources/images/resize-low-res.png" class="threejs_center nobg">

Vamos a arreglar este problema. Para esto necesitamos establecer
el aspecto de la cámara al aspecto del tamaño del display del canvas.
Podemos hacerlo mediante las propiedades clientWidth` y `clientHeight`
del canvas.

Modificamos nuestro loop render de la siguiente manera:

```js
function render(time) {
  time *= 0.001;
+  const canvas = renderer.domElement;
+  camera.aspect = canvas.clientWidth / canvas.clientHeight;
+  camera.updateProjectionMatrix();
  ...
```

Ahora los cubos deberían dejar de verse distorsionados.

{{{example url="../threejs-responsive-update-camera.html" }}}

Abre el ejemplo en una ventana separada y reescala la ventana
y deberías ver que los cubos ya no están estrechos del alto o el ancho.

Deberían conservar el aspecto correcto a pesar del tamaño de la ventana.

<img src="resources/images/resize-correct-aspect.png" width="407" class="threejs_center nobg">

Ahora corrigamos el aspecto de bloques.

Los elementos canvas tienen 2 tamaños. Uno es el tamaño de canvas que se
muestra en la pantalla. Ese se establece en el CSS. El otro es el numero de
pixeles en el canvas mismo. Esto no es diferente a una imagen. Por ejemplo
podríamos tener una imagen de 128x64 pixeles y usando CSS podríamos mostrarla
como una de 400x200 pixeles.

```html
<img src="some128x64image.jpg" style="width:400px; height:200px">
```

El tamaño interno del canvas así como su resolución es a menudo conocido como el tamaño del buffer de dibujo (*drawingbuffer size*). En three.js podemos establecerlo mediante la función `renderer.setSize`.
¿Qué tamaño deberíamos escoger? La respuesta obvia es «el mismo tamaño del canvas que mostramos».
De nuevo, para definir el tamaño vemos las propiedades `clientWidth` y `clientHeight`.

Vamos a escribir una función que revise si el canvas del renderer no posee
el tamaño del display, si no lo está, lo establece.

```js
function resizeRendererToDisplaySize(renderer) {
  const canvas = renderer.domElement;
  const width = canvas.clientWidth;
  const height = canvas.clientHeight;
  const needResize = canvas.width !== width || canvas.height !== height;
  if (needResize) {
    renderer.setSize(width, height, false);
  }
  return needResize;
}
```

Nota que revisamos si el canvas en realidad necesita cambiarse de tamaño.
Reescalar el canvas es una parte interesante de la especificación del canvas
y es mejor no establecer el tamaño que deseamos si ya lo posee.

Una vez que sabemos si tenemos el tamaño que necesitamos o no, llamamos a
`renderer.setSize` y pasamos como argumentos los nuevo valores de ancho y
alto. Es importante pasar `false` al final. Por defecto `render.setSize` 
establece el tamaño del CSS del canvas, pero no necesitamos eso. Queremos que
el navegador continue funcionaando con los demás elementos, que es lo que usa
CSS para determinar el tamaño del display del elemento. No queremos canvas
usados tres veces más que otros elementos.

Nota que nuestra función regresa `true` si el tamaño del canvas fue modificado.
Podemos usar esto para revisar si hay otras cosas que podríamos actualizar.
Vamos a modificar nuestro ciclo render para usar la nueva función.

```js
function render(time) {
  time *= 0.001;
+  if (resizeRendererToDisplaySize(renderer)) {
+    const canvas = renderer.domElement;
+    camera.aspect = canvas.clientWidth / canvas.clientHeight;
+    camera.updateProjectionMatrix();
+  }
  ...
```

Ya que el aspecto sólo va a cambiar si cambia el tamaño del display,
podemos establecer el aspecto de la cámara si `resizeRendererToDisplaySize`
regresa `true`.

{{{example url="../threejs-responsive.html" }}}

Ahora podríamos renderizar con una resolución igual al tamaño del display
del canvas.

Para entender sobre como CSS maneja el cambio de tamaño vamos a poner
nuestro código en un [archivo `.js` separado](../threejs-responsive.js).
Aquí hay unos cuantos ejemplos más donde dejamos al CSS escoger el tamaño y 
notamos que no tenemos que cambiar el código para que funcione.

Vamos a poner nuestros cubos en medio de un párrafo de texto.

{{{example url="../threejs-responsive-paragraph.html" startPane="html" }}}

hacemos uso del mismo código usado en un editor donde el tamaño del área
de control puede cambiar de tamaño.

{{{example url="../threejs-responsive-editor.html" startPane="html" }}}

Es importante resaltar que no hay cambios de código. Sólo cambian nuestro HTML y el CSS.

## Usando displays HD-DPI

HD-DPI es el acrónimo para *display de alta densidad de
puntos por pulgada*. La mayoría de Macs actualmente lo
incorporan y muchos equipos con Windows también, así
como varios smartphones.

La manera en que esto funciona en el navegador es usando los 
pixeles de CSS para establecer los tamaños que deben usarse
acorde al de la resolución del display. El navegador sólo renderizará
el texto con más detalle pero con el mismo tamaño físico.


Hay varias manera para manejar el HD-DPI con three.js.

La primera no es algo muy particular, de hecho suele ser común. Renderizar
gráficos 3D utiliza mucho poder de procesamiento en la GPU. Los GPU's de
móviles son menos poderosos que los de escritorio, al menos hasta 2018 varios
modelos de teléfonos cuentan con displays de alta resolución.
Los teléfonos de gama alta tiene un ratio de HD-DPI de 3x, esto significa que
para cada pixel de un display sin HD-DPI, estos teléfonos tienen 9 pixeles, lo
que implica que el proceso de renderizado aumenta 9x.

Calcular los pixeles 9x es mucho trabajo si dejamos el
código como esta dado que calcularemos los pixeles a 1x y el
navegador los dibujará a 3x su tamaño (ya que 3x por 3x = 9x de pixeles).

Para una app de three.js pesada probablemente sea útil, ya que
de otra forma obtendrías un framerate muy lento.

Se ha dicho que si prefieres renderizar a la resolución del dispositivo
hay un par de formas de hacerlo en three.js.

Una es darle a three.js un multiplicador de resoluciones usando
`renderer.setPixelRatio`. Puedes decirle al navegador que el multiplicador
es de los pixeles de CSS a los pixeles del dispositivo y pasar estos últimos
a three.js.

     renderer.setPixelRatio(window.devicePixelRatio);

Después de cualquier llamada a `renderer.setSize`, se ajustará
el tamaño de hayas solicitado multiplicado por el ratio de pixeles
que hayas pasado. **Esto NO ESTÁ RECOMENDADO**. Ve abajo.

La otra forma es hacerlo por tu cuenta cuando cambias el tamaño del canvas.

```js
    function resizeRendererToDisplaySize(renderer) {
      const canvas = renderer.domElement;
      const pixelRatio = window.devicePixelRatio;
      const width  = canvas.clientWidth  * pixelRatio | 0;
      const height = canvas.clientHeight * pixelRatio | 0;
      const needResize = canvas.width !== width || canvas.height !== height;
      if (needResize) {
        renderer.setSize(width, height, false);
      }
      return needResize;
    }
```

Esta es una mejor solución cuando se mira de forma objetiva. ¿Por qué?
Porque significa que se obtiene lo que se necesita.
Hay muchos casos cuando se usa three.js donde necesitamos conocer el tamaño del drawingBuffer del canvas. Por ejemplo cuando hacemos un filtro de postprocesamiento
o sí hacemos un shader que utiliza `gl_FragCoord`, si hacemos una captura de
pantalla o leemos pixeles para procesarlos en la GPU, si dibujamos 2D en un
canvas, etc. Hay muchos casos donde si usamos `setPixelRatio`nuestro tamaño real será 
diferente que el tamaño que necesitemos y tendremos que advinar cuando usar
el tamaño que pidamos y cuando usar el tamaño que three.js este usando.
Al hacerlo nosotros mismos nos aseguramos de saber el tamaño que se esté usando
en el momento que lo necesitemos.
No hay casos especiales donde la mágia suceda tras bambalinas.

Un ejemplo del código se encuentra debajo:

{{{example url="../threejs-responsive-hd-dpi.html" }}}

Podria ser difícil de ver la diferencia a menos que cuentes con un display 
HD-DPI y comparas estos ejemplos, podrás ver como los bordes son más definidos.

Este artículo habló sobre cosas básicas, pero fundamentales. Lo siguiente será
[sobre los primitivos que tiene three.js](threejs-primitives.html).
