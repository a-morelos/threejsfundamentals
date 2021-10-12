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

El último ejemplo que usamos fue un canvas plano sin CSS ni tamaño.

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

Now the cubes should stop being distorted.
{{{example url="../threejs-responsive-update-camera.html" }}}
Open the example in a separate window and resize the window
and you should see the cubes are no longer stretched tall or wide.
They stay the correct aspect regardless of window size.
<img src="resources/images/resize-correct-aspect.png" width="407" class="threejs_center nobg">
Now let's fix the blockiness.
Canvas elements have 2 sizes. One size is the size the canvas is displayed
on the page. That's what we set with CSS. The other size is the
number of pixels in the canvas itself. This is no different than an image.
For example we might have a 128x64 pixel image and using
CSS we might display as 400x200 pixels.
```html
<img src="some128x64image.jpg" style="width:400px; height:200px">
```
A canvas's internal size, its resolution, is often called its drawingbuffer size.
In three.js we can set the canvas's drawingbuffer size by calling `renderer.setSize`.
What size should we pick? The most obvious answer is "the same size the canvas is displayed".
Again, to do that we can look at the canvas's `clientWidth` and `clientHeight`
properties.
Let's write a function that checks if the renderer's canvas is not
already the size it is being displayed as and if so set its size.
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
Notice we check if the canvas actually needs to be resized. Resizing the canvas
is an interesting part of the canvas spec and it's best not to set the same
size if it's already the size we want.
Once we know if we need to resize or not we then call `renderer.setSize` and
pass in the new width and height. It's important to pass `false` at the end.
`render.setSize` by default sets the canvas's CSS size but doing so is not
what we want. We want the browser to continue to work how it does for all other
elements which is to use CSS to determine the display size of the element. We don't
want canvases used by three to be different than other elements.
Note that our function returns true if the canvas was resized. We can use
this to check if there are other things we should update. Let's modify
our render loop to use the new function
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
Since the aspect is only going to change if the canvas's display size
changed we only set the camera's aspect if `resizeRendererToDisplaySize`
returns `true`.
{{{example url="../threejs-responsive.html" }}}
It should now render with a resolution that matches the display
size of the canvas.
To make the point about letting CSS handle the resizing let's take
our code and put it in a [separate `.js` file](../threejs-responsive.js).
Here then are a few more examples where we let CSS choose the size and notice we had
to change zero code for them to work.
Let's put our cubes in the middle of a paragraph of text.
{{{example url="../threejs-responsive-paragraph.html" startPane="html" }}}
and here's our same code used in an editor style layout
where the control area on the right can be resized.
{{{example url="../threejs-responsive-editor.html" startPane="html" }}}
The important part to notice is no code changed. Only our HTML and CSS
changed.
## Handling HD-DPI displays
HD-DPI stands for high-density dot per inch displays.
That's most Macs nowadays and many Windows machines
as well as pretty much all smartphones.
The way this works in the browser is they use
CSS pixels to set the sizes which are supposed to be the same
regardless of how high res the display is. The browser
will just render text with more detail but the
same physical size.
There are various ways to handle HD-DPI with three.js.
The first one is just not to do anything special. This
is arguably the most common. Rendering 3D graphics
takes a lot of GPU processing power. Mobile GPUs have
less power than desktops, at least as of 2018, and yet
mobile phones often have very high resolution displays.
The current top of the line phones have an HD-DPI ratio
of 3x meaning for every one pixel from a non-HD-DPI display
those phones have 9 pixels. That means they have to do 9x
the rendering.
Computing 9x the pixels is a lot of work so if we just
leave the code as it is we'll compute 1x the pixels and the
browser will just draw it at 3x the size (3x by 3x = 9x pixels).
For any heavy three.js app that's probably what you want
otherwise you're likely to get a slow framerate.
That said if you actually do want to render at the resolution
of the device there are a couple of ways to do this in three.js.
One is to tell three.js a resolution multiplier using `renderer.setPixelRatio`.
You ask the browser what the multiplier is from CSS pixels to device pixels
and pass that to three.js
     renderer.setPixelRatio(window.devicePixelRatio);
After that any calls to `renderer.setSize` will magically
use the size you request multiplied by whatever pixel ratio
you passed in. **This is strongly NOT RECOMMENDED**. See below
The other way is to do it yourself when you resize the canvas.
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
This second way is objectively better. Why? Because it means I get what I ask for.
There are many cases when using three.js where we need to know the actual
size of the canvas's drawingBuffer. For example when making a post processing filter,
or if we are making a shader that accesses `gl_FragCoord`, if we are making
a screenshot, or reading pixels for GPU picking, for drawing into a 2D canvas,
etc... There many many cases where if we use `setPixelRatio` then our actual size will be different
than the size we requested and we'll have to guess when to use the size
we asked for and when to use the size three.js is actually using.
By doing it ourselves we always know the size being used is the size we requested.
There is no special case where magic is happening behind the scenes.
Here's an example using the code above.
{{{example url="../threejs-responsive-hd-dpi.html" }}}
It might be hard to see the difference but if you have an HD-DPI
display and you compare this sample to those above you should
notice the edges are more crisp.
This article covered a very basic but fundamental topic. Next up lets quickly
[go over the basic primitives that three.js provides](threejs-primitives.html).
