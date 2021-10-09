Title: Fundamentos de Three.js
Description: Tu primera lección de Three.js comienza con los fundamentos
TOC: Fundamentos

Este es el primero de una serie de artículos sobre three.js.
[Three.js](https://threejs.org) es una biblioteca 3D que busca hacer
tan sencillo como sea posible obtener contenido 3D en una paǵina web.

Three.js es confundido a menudo con WebGL ya que la mayoría de las veces,
pero no siempre, three.js hace uso de WebGL para dibujar 3D.
[WebGL es un sistema de bajo nivel que solo dibuja puntos, líneas y triángulos](https://webglfundamentals.org).
Para hacer algo útil con WebGL generalmente se requiere un poco de código y
es ahí donde three.js entra en acción. Maneja elementos como escenas, luces,
sombras, materiales, texturas, matemáticas 3D y toda clase dde cosas que tendrías
que escribir por tu cuenta si quisieras usar WebGL directamente.

Estos tutoriales asumen que ya sabes JavaScript y la mayoría de las veces usan el
estilo de ES6. [Ve aquí una lista de las cosas que necesitas saber](threejs-prerequisites.html).
La mayoría de navegadores que soportan three.js son auto-actualizables, lo que
significa que la mayoría de usuarios debería poder ejecutar este código.
Si de verdad necesitas ejecutar este código en navegadores viejos, deberías usar
un transpilador como [Babel](https://babeljs.io).
Claro que los usuarios usando navegadores viejos probablemente cuenten con equipos
que no puedan ejecutar three.js.

Cuando aprendemos un lenguaje de programación la primera cosa que 
hacemos es hacer que la computadora muestre un `"¡Hola Mundo!"`.
En 3D lo más común es comenzar con un cubo en 3D. Empecenos con un `"¡Hola Cubo!"`

Antes de comenzar vamos a darnos una idea de la estructura de una app three.js.
Una app three.js requiere que crear una serie de objetos y conectarlos.
Aquí hay un diagrama que representa una pequeña app de three.js

<div class="threejs_center"><img src="resources/images/threejs-structure.svg" style="width: 768px;"></div>

Algunas cosas que observar sobre el diagrama de arriba.

* Hay un `Renderer` (renderizador). Este es probablemente el objeto principal de three.js. Se le
  pasa un `Scene` (escena) y un `Camera` (cámara) al `Renderer` que se encarga
  de renderizar (dibujar) la porción de la escena 3D que está adentro del *frustum*
  de la cámara como una imagen 2D hacia un canvas.

* Hay una [escenografía](threejs-scenegraph.html) que es una estructura de árbol
  que consta de varios objetos como `Scene`, múltiples objetos `Mesh` (malla), 
  objetos `Light` (luces) y objetos `Group`, `Object3D`, and `Camera. Un objeto
  `Scene` define la raíz de la escenografía y contiene propiedades como el color de
  fondo y la niebla. Estos objetos definen la jeraquía padre/hijo de una estructura
  de árbol y representa donde aparecen los objetos y como se orientan. Los hijos se
  posicionan y orientan respecto al padre. Por ejemplo, las llantas en un auto serían
  los hijos de este último, así que al mover y orientar el objeto auto de inmediato se
  moverían sus llantas. Puedes leer más sobre esto en el [artículo sobre escenografías](threejs-scenegraph.html).

  Nota que en el diagrama `Camera` está mitad adentro, mitad afuera de la escenografía.
  Esto es para representar que en three.js a diferencia de otros objetos, una `Camera`
  no necesita estar en la escenografía para funcionar. Como otros objetos, una `Camera`
  como hijo de algún otro objeto, se moverá y orientará de forma relativa a su objeto padre.
  Hay un ejemplo de colocar múltiples objetos `Camera` en una escenografía al final del
  [artículo de escenografías](threejs-scenegraph.html).

*  Los objetos `Mesh` representan el dibujo de un objeto `Geometry` con un `Material`.
   Tantos los objetos `Material` como los `Geometry` pueden usarse en múltiples objetos
   `Mesh`. Por ejemplo, para dibujar dos cubos azules en diferentes ubicaciones podríamos
   necesitar dos objetos `Mesh` para representar la posición y orientación de cada cubo.
   Podríamos necesitar un `Geometry` para guardar los datos del vértice para un cubo y 
   podríamos usar un `Material` para especifícar el colo azul. Ambos objetos `Mesh` pueden
   hacer referencia al mismo objeto `Geometry` y al mismo objeto `Material`.

*  Los objetos `Geometry` representan los datos del vértice de algunas piezas de geomtería como
   esferas, cubos, planos, perros, gatos, humanos, árboles, edificios, etc...
   Three.js incluye algunas en
   [geometría primitiva](threejs-primitives.html). También puedes revisar
   [crear geometría personalizada](threejs-custom-buffergeometry.html) y también 
   [cargar geometría desde archivos](threejs-load-obj.html).

* Los objetos `Material` representan
  [las propiedades de la superficie que se usan para dibujar geometría](threejs-materials.html)
  incluyen propiedades como el color para usar y el brillo en él. Un `Material` también
  puede referenciar una o más objetos `Texture` que pueden usarse, por ejemplo, para
  colocar una imagen en la superficie de una geometría.

* Los objetos `Texture` generalmente representan imágenes que puede [cargarse desde un archivo](threejs-textures.html),
  [generados desde un canvas](threejs-canvas-textures.html) o [renderizado desde otra escena](threejs-rendertargets.html).

* Los objetos `Light` representan [diferentes tipos de luces](threejs-lights.html).

Sabiendo todo lo anterior, el *setup* para nuestro pequeño *«Hola cubo»*
se vería así:

<div class="threejs_center"><img src="resources/images/threejs-1cube-no-light-scene.svg" style="width: 500px;"></div>

Primero cargamos three.js

```html
<script type="module">
import * as THREE from './resources/threejs/r132/build/three.module.js';
</script>
```

Es importante que agreges `type="module"` en el tag script. Esto nos
permite usar la palabra `import` para cargar three.js. Hay otras formas
para cargar three.js pero este es el método recomendado por r106.
Los módulos tienen la ventaja de que son fáciles de importar desde otros módulos
que los necesiten. Esto nos ahorra el tener que cargar los scripts extra
que necesiten de forma manual.

Lo siguiente que necesitamos es la etiqueta `<canvas>`

```html
<body>
  <canvas id="c"></canvas>
</body>
```

Le pediremos a three.js que dibuje dentro del canvas, así que primero
debemos encontrarlo.

```html
<script type="module">
import * as THREE from './resources/threejs/r132/build/three.module.js';
+function main() {
+  const canvas = document.querySelector('#c');
+  const renderer = new THREE.WebGLRenderer({canvas});
+  ...
</script>
```
Después de encontrar el canvas creamos un `WebGLRenderer`. El renderer 
es la entidad responsable de tomar todos lo datos que le des y renderizarlos
al canvas. Antes había otras clases de renderers como `CSSRenderer` o `CanvasRenderer`
y en el futuro habrá otro como `WebGL2Renderer` o `WebGPURenderer`. Por ahora basta
saber que `WebGLRenderer` usa WebGL para renderizar 3D al canvas.

Nota algunos detalles aquí. Sí no pasas un canvas a three.js, este creará uno por ti pero
tendrás que agregarlo en tu documento. Donde agregarlo puede cambiar dependiendo del caso 
de uso y también tendrías que modificar tu código. Encontré que pasar el canvas a three.js
te da cierta flexibilidad; puedo poner el canvas donde sea y el código sabrá donde encontrarlo.
De igual forma, sí inserto el canvas en el documento probablemente deba cambiar el código si mi
caso de uso cambia.

Lo siguiente que necesitamos es una  cámara. Usaremos `PerspectiveCamera`.

```js
const fov = 75;
const aspect = 2;  // es el default del canvas
const near = 0.1;
const far = 5;
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
```
`fov` es la versión corta para `field of view` (campo de visión). En este caso 75 grados en la
dimensión vertical. Nota que la mayoría de los ángulos en three.js están en radianes excepto en
algunos casos donde la cámara usa grados.

`aspect` es el aspecto del display en el canvas. Entraremos en detalle
[en otro artículo](threejs-responsive.html) pero por defecto en un canvas es de 
 300x150 pixels lo que le da un aspecto de 300/150 or 2.

`near`(lejos) y `far`(cerca) representan el espacio en frente de la cámara
que será renderizado. Cualquier cosa que esté antes o después de ese rango
no será dibujado.

Estas cuatro propiedades definen un *«frustum»*. Un *frustum* es el nombre que
se le da a una figura 3D como una pirámide con su punta entrecortada.
Piensa en el *frustrum* como otra figura 3D al igual que una esfera, cubo o un prisma.

<img src="resources/frustum-3d.svg" width="500" class="threejs_center"/>

El alto y los planos `near` y `far` son determinados por el cambpo de visión.
El ancho de ambos planos son determinados por el ancho de visión y el aspecto.
Cualquier cosa dentro del frustrum será dibujada. Cualquier cosa fuera, no.

La cámra por defecto mira hacia abajo sobre el eje -Z con +Y hacia arriba. 
Podremos nuestro cubo en el origen, así que necesitamos mover la camará hacia atrás
para poder ver algo.
```js
camera.position.z = 2;
```
Así es como tendría que verse
<img src="resources/scene-down.svg" width="500" class="threejs_center"/>
En el diagrama anterior podemos ver nuestra cámara en `z = 2`. Está mirando
hacia abajo sobre el eje -Z. Nuestro frustrum comienza 0.1 unidades desde el
frente de la cámara y llega 5 unidades en enfrente de la misma. Ya que estamos
mirando hacia abajo, el campo de visión se ve afectado por el aspecto. Nuestro
canvas es el doble de ancho que el alto, lo suficiente para que el campo de visión
sea mucho más amplio que nuestros 75 grados que es el campo de visión vertical.
Lo siguiente es crear un Scene`. Un `Scene` en three.js es la raíz de la escenografía.
Lo que quieras dibujar en three.js necesita ser agregado a la escena. Veremos más
detalles de [como funcionan las escenas en un artículo posterior](threejs-scenegraph.html).
```js
const scene = new THREE.Scene();
```

Lo que sigue es crear un `BoxGeometry` que contiene los datos para la caja.
Casi todo lo que queramos mostrar en three.js necesita geometría la cual se define
por los vértices que constituyen nuestro objeto 3D.
```js
const boxWidth = 1;
const boxHeight = 1;
const boxDepth = 1;
const geometry = new THREE.BoxGeometry(boxWidth, boxHeight, boxDepth);
```
Luego creamos un material básico y definimos su color. Los
colores pueden especificarse usando valores hexadecimales de
6 dígitos de CSS estándar.
```js
const material = new THREE.MeshBasicMaterial({color: 0x44aa88});
```
A continuación creamos el `Mesh`. Un `Mesh` en three representa la combinación
de tres cosas:
1. `Geometry` (la forma del objeto)
2. `Material` (como dibujar el objeto, brilloso o plano, que color y texturas aplicar, etc).
3. La posición, orientación y escala del objeto en la escena que se deriva de su padre. En el código de abajo el padre es la escena.
```js
const cube = new THREE.Mesh(geometry, material);
```

Y por último agregamos ese mesh a la escena.
```js
scene.add(cube);
```

Podemos renderizar la escena llamando a la funcion `render` del renderer
y pasamos como argumentos la escena y la cámara:
```js
renderer.render(scene, camera);
```
Aquí hay un ejemplo funcionando:
{{{example url="../threejs-fundamentals.html" }}}

Es díficil distinguir que es un cubo 3D ya que lo vemos directamente
debajo del eje -Z y el cubo mismo se encuentra alineado al eje por lo
que solo vemos una cara.

Vamos a animarlo de forma que gire y veremos más claramente que está
dibujado en 3D. Para animarlo los haremos renderizando dentro de un loop
usando [`requestAnimationFrame`](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame).

Aquí esta nuestro loop:
```js
function render(time) {
  time *= 0.001;  // convierte el tiempo a segundos
  cube.rotation.x = time;
  cube.rotation.y = time;
  renderer.render(scene, camera);
  requestAnimationFrame(render);
}
requestAnimationFrame(render);
```
`requestAnimationFrame` es una petición al navegador para animar algo. Se manda a la función
que lo llama. En nuestro caso esa función es `render`. El navegador llamará tu función y si
cambias algo relacionado al display de la página el navegador volverá a renderizarlo.
Aquí estamos llamando a la función `renderer.render` de three que dibujará nuestra escena.

`requestAnimationFrame` pasa el tiempo desde que se carga la página
a nuestra función. Ese tiempo se encuentra en milisegundos. Creo que
es mucho más sencillo trabajar en segundos así que convertimos ese
tiempo a segudos. Establecemos la rotación del cubo en X y Y al tiempo
actual. Esas rotaciones se encentran en [radianes](https://es.wikipedia.org/wiki/Radian).
Hay 2 pi radianes en un círculo así que nuestro cubo debería girar una vez sobre cada eje
en 6.28 segundos.
Entonces renderizamos nuestra escena y pedimos otro frame de animación para continuar nuestro
bucle.
Fuera del bucle llamamos a `requestAnimationFrame` una vez para llamar al bucle.

{{{example url="../threejs-fundamentals-with-animation.html" }}}

Se ve mejor, pero aún es difícil distinguir el 3D. Lo que podría ayudarnos es agregar
un poco de iluminación así que integraremos una luz. Hay luces de distintos tipos en
three.js, cosa de la que [hablaremos después](threejs-lights.html). Por ahora nos
bastaremos con una luz direccional.

```js
{
  const color = 0xFFFFFF;
  const intensity = 1;
  const light = new THREE.DirectionalLight(color, intensity);
  light.position.set(-1, 2, 4);
  scene.add(light);
}
```

Las luces direccionales tiene una posición y un objetivo. Ambas por default son
0, 0, 0. En nuestro caso dirigimos las luces a la posición -1, 2, 4; que es un
ligeramente a la izquierda, sobre y detrás de nuestra cámara. El objetivo todavía
se encuentra en 0, 0, 0 así que se iluminará delante del origen.
También necesitamos cambiar el material. El `MeshBasicMaterial` no se ve afectado
por luces. Vamos a cambiarlo a un `MeshPhongMaterial` que sí se ve afectado.

```js
-const material = new THREE.MeshBasicMaterial({color: 0x44aa88});  // greenish blue
+const material = new THREE.MeshPhongMaterial({color: 0x44aa88});  // greenish blue
```
Ahora así es la estructura de nuestro programa:

<div class="threejs_center"><img src="resources/images/threejs-1cube-with-directionallight.svg" style="width: 500px;"></div>

Y así se ve funcionando:

{{{example url="../threejs-fundamentals-with-light.html" }}}

Ahora es más claro el 3D.
Sólo para entretenernos vamos a agregar 2 cubos.
Usaremos la misma geometría para cada cubo pero usaremos diferentes
materiales y diferentes colores.

Primero, haremos una función que cree un nuevo material
con el color especificado. Entonces creamos un mesh usando
la geomtría específicda y agregandoló a la escena y estableciendo
su posición X.

```js
function makeInstance(geometry, color, x) {
  const material = new THREE.MeshPhongMaterial({color});
  const cube = new THREE.Mesh(geometry, material);
  scene.add(cube);
  cube.position.x = x;
  return cube;
}
```
Después la llamaremos 3 veces con 3 diferentes colores y posiciones en X
guadardándo las instancias del `Mesh` en un arreglo.

```js
const cubes = [
  makeInstance(geometry, 0x44aa88,  0),
  makeInstance(geometry, 0x8844aa, -2),
  makeInstance(geometry, 0xaa8844,  2),
];
```
Finalmente, hacemos girar los 3 cubos en nuestra función render.
Calculamos rotaciones ligeramente distintas para cada una.

```js
function render(time) {
  time *= 0.001;  // convierte el tiempo a segundos
  cubes.forEach((cube, ndx) => {
    const speed = 1 + ndx * .1;
    const rot = time * speed;
    cube.rotation.x = rot;
    cube.rotation.y = rot;
  });
  ...
```
Y aquí está:

{{{example url="../threejs-fundamentals-3-cubes.html" }}}

Sí lo comparas con el diagrama anterios puedes observar que
cumple con lo que esperábamos. Con los cubos en X = -2 y x= +2
están parcialmente fuera del frustrum. También paracen un poco
encerrados dado que nuestro fov sobre el canvas es algo estrecho.

Ahora nuestro programa tiene esta estructura.
<div class="threejs_center"><img src="resources/images/threejs-3cubes-scene.svg" style="width: 610px;"></div>

Como puedes ver tenemos tres objetos `Mesh` cada uno referenciando al mismo `BoxGeometry`.
Cada `Mesh` hace referencia a un único `MeshPhongMaterial` para que cada cubo tenga
colores distintos.
Espero que esta corta introducción te ayude para comenzar. [A continuación
haremos nuestro código responsivo para que pueda adaptarse a distintas situaciones](threejs-responsive.html).
<div id="es6" class="threejs_bottombar">
  <h3>módulos es6, three.js y la estructura de directorios</h3>
  <p>Al igual que la versión r106 el modo preferido para usar three.js es mediante <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import">módulos es6</a>.</p>

  <p>Los módulos es6 pueden ser cargados a través de la palabra reservada <code>import</code> en un script
    o una etiqueta <code>&lt;script type="module"&gt;</code>. Aquí hay un ejemplo de ambos.
  </p>

<pre class=prettyprint>
&lt;script type="module"&gt;
import * as THREE from './resources/threejs/r132/build/three.module.js';
...
&lt;/script&gt;
</pre>

   <p>
     Las rutas pueden ser absolutas o relativas. Las rutas relativas siempre deben comenzar con <code>./</code> o <code>../</code>,
     lo que es diferente de otras etiquetas como <code>&lt;img&gt;</code> y <code>&lt;a&gt;</code>.
  </p>
  <p>
    Las referencias al mismo script sólo serán cargadas una vez ya que sus rutas absolutas son
    simpre las mismas. Para three.js esto significa que necesita que pongas todas las bibliotecas
    de ejemplo en la estructura correcta de directorios.
  </p>
<pre class="dos">
someFolder
 |
 ├-build
 | |
 | +-three.module.js
 |
 +-examples
   |
   +-jsm
     |
     +-controls
     | |
     | +-OrbitControls.js
     | +-TrackballControls.js
     | +-...
     |
     +-loaders
     | |
     | +-GLTFLoader.js
     | +-...
     |
     ...
</pre>
  <p>
    El motivo de esta estructura es porque los scripts en los ejemplos como 
    <a href="https://github.com/mrdoob/three.js/blob/master/examples/jsm/controls/OrbitControls.js"><code>OrbitControls.js</code></a>
    son complicadas de usar con rutas relativas como
  </p>
  
<pre class="prettyprint">
import * as THREE from '../../../build/three.module.js';
</pre>
  <p>
    Usando la misma estructura se asegura que cuando importes tanto three como una de las
    bibliotecas de ejemplos, ambas usen el mismo archivo <code>three.module.js</code>.
  </p>
  
<pre class="prettyprint">
import * as THREE from './someFolder/build/three.module.js';
import {OrbitControls} from './someFolder/examples/jsm/controls/OrbitControls.js';
</pre>
  <p>
    Esto también aplica cuando se usa un CDN. Asegúrate de que tu ruta a <code>three.module.js</code>
    termine con <code>/build/three.modules.js</code>. Por ejemplo:
  </p>
<pre class="prettyprint">
import * as THREE from 'https://unpkg.com/three@0.108.0<b>/build/three.module.js</b>';
import {OrbitControls} from 'https://unpkg.com/three@0.108.0/examples/jsm/controls/OrbitControls.js';
</pre>
  <p>
    Si prefieres el antiguo estilo de <code>&lt;script src="path/to/three.js"&gt;&lt;/script&gt;</code>
    puedes revisar <a href="https://r105.threejsfundamentals.org">una versión vieja de este sitio</a>.
    Three.js tiene la política de no preocuparse sobre retrocompatibilidad. Se espera que uses una versión
    en específico de la misma forma que descargas el código y lo introduces en tu proyecto. Cuando actualizas
    a una nueva versión puedes leer la <a href="https://github.com/mrdoob/three.js/wiki/Migration-Guide">guía de migración</a>
    para ver que cambios necesitas hacer. Podría resultar en mucho trabajo mantener tanto el módulo es6 como el script
    de este sitio, así que de aquí en adelante solo mostraremos el estilo de módulos de es6. Como ya se dijo antes,
    si necesitas soporte para navegadores obsoletos deberías considerar usar un <a href="https://babeljs.io">transpilador</a>.
  </p>
<!-- needed in English only to prevent warning from outdated translations -->
<a href="threejs-geometry.html"></a>
<a href="Geometry"></a>
