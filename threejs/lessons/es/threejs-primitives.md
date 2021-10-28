Title: Primitivos de three.js
Description: Un tour por los primitivos de three.js.
TOC: Primitivos

Este artículo es parte de una serie de artículos sobre three.js.
El primero fue sobre los [fundamentos](threejs-fundamental.html).
Si aún no lo has leído quizás quieras comenzar por ahí.

Three.js tiene un gran número de primitivos. Los primitivos son
generalmenre figuras 3D que son generadas durante la ejecución
con una serie de parámetros.

Es común usar primtivos para cosas como una esfera para un globo
o una serie de cajas para un gráfico 3D. Es muy común usar primitivos
para experimentar mientras comienzas con el 3D. Para la 
mayoría de apps 3D es más común trabajar con algún modelador
de 3D usando programas como [Blender](https://blender.org),
[Maya](https://www.autodesk.com/products/maya/) o [Cinema 4D](https://www.maxon.net/es/products/cinema-4d/). Más adelante en esta serie 
cubriremos el aspecto de crear y cargar datos de modelos hechos
en estos programas. Por ahora sigamos con los primitivos.

Muchos de los primitivos que se muestran a continuación incluyen por
defecto algunos parámetros, que puedes usar dependiendo de lo que 
necesites.

<div id="Diagram-BoxGeometry" data-primitive="BoxGeometry">Una caja</div>
<div id="Diagram-CircleGeometry" data-primitive="CircleGeometry">Un círculo</div>
<div id="Diagram-ConeGeometry" data-primitive="ConeGeometry">Un Cono</div>
<div id="Diagram-CylinderGeometry" data-primitive="CylinderGeometry">Un cilindro</div>
<div id="Diagram-DodecahedronGeometry" data-primitive="DodecahedronGeometry">Un dodecaedro (12 lados)</div>
<div id="Diagram-ExtrudeGeometry" data-primitive="ExtrudeGeometry">Una figura 2d extruída con biselado opcional.
Aquí está la clave para extruir figuras. Nota que esto es la base para
usar <code>TextGeometry</code>.</div>
<div id="Diagram-IcosahedronGeometry" data-primitive="IcosahedronGeometry">Un icosaedro (20 sideslados)</div>
<div id="Diagram-LatheGeometry" data-primitive="LatheGeometry">Una figura generada por el rotar de una línea. Ejemplo para esto son: pines de boliche, velas, cadeleros, vasos de vino, etc... Proporcionas una silueta 2D compuesta de una serie de puntosy le indicas a three.js cuantás divisiones debe de hacer mientras gira la silueta alrededor de un eje.</div>
<div id="Diagram-OctahedronGeometry" data-primitive="OctahedronGeometry">Un octaedro (8 lados)</div>
<div id="Diagram-ParametricGeometry" data-primitive="ParametricGeometry">Una superficie que se genera mediante una función que toma puntos 2D de un *grid* y regresan el punto 3D correspondiente</div>
<div id="Diagram-PlaneGeometry" data-primitive="PlaneGeometry">Un plano 2D</div>
<div id="Diagram-PolyhedronGeometry" data-primitive="PolyhedronGeometry">Toma un conjunto de triángulos centrados alrededor de un punto y los projecta sobre una esfera.</div>
<div id="Diagram-RingGeometry" data-primitive="RingGeometry">Un disco 2D con un agujero en el centro.</div>
<div id="Diagram-ShapeGeometry" data-primitive="ShapeGeometry">Un borde 2D que es riangulado.</div>
<div id="Diagram-SphereGeometry" data-primitive="SphereGeometry">Una esfera</div>
<div id="Diagram-TetrahedronGeometry" data-primitive="TetrahedronGeometry">Un tetraedro (4 lados)</div>
<div id="Diagram-TextGeometry" data-primitive="TextGeometry">Texto 3D generado de una fuente 2D y una cadena.</div>
<div id="Diagram-TorusGeometry" data-primitive="TorusGeometry">Un toroide (dona)</div>
<div id="Diagram-TorusKnotGeometry" data-primitive="TorusKnotGeometry">Un nudo toroidal</div>
<div id="Diagram-TubeGeometry" data-primitive="TubeGeometry">Un círculo dibujado mediante una ruta.</div>
<div id="Diagram-EdgesGeometry" data-primitive="EdgesGeometry">Un objeto auxiliar que utiliza otra geometría como fuente y genera bordes sólo si el ángulo entre las caras es más grande que un umbral. Por ejemplo, si miras a la cara desde la cima, muestra una linea que atraviesa cada cara mostrando cada triángulo que forma la caja. Usando <code>EdgesGeometry</code> en vez de esta, las líneas son eliminadas. Ajusta el ángulo del umbral abajo y verás que los bordes desaparecen.</div>
<div id="Diagram-WireframeGeometry" data-primitive="WireframeGeometry">Genera una geometría que contiene un segmento de línea (2 puntos) por cada borde en la geometría proporcionada. Sin esto es verás como faltan borden o hay algunos de más. Esto es porque WebGL generalmente necesita 2 puntos por segmentos de línea. Por ejemplo, sí tienes un triángulo entonce tienes 3 puntos. Si tratas de dibujarlo usando un material con <code>wireframe: true</code> deberías obtener una línea. Usando la geometría del triángulo con <code>WireframeGeometry</code> creará una nueva geometría con 3 segmentos de línea usando 6 puntos.</div>

Seguiremos hablando de crear geometría personalizada en [otro artículo](threejs-custom-buffergeometry.html). Por ahora
hagamos un ejemplo usando cada tipo de primitivos. Comenzaremos
con los [ejemplos de los artículos previos](threejs-responsive.html).

Cerca de la parte superior establecemos un color de fondo

```js
const scene = new THREE.Scene();
+scene.background = new THREE.Color(0xAAAAAA);
```

Con esto le decimos a three.js que limpie el exceso de luz.

La cámara necesita cambiar de posición para poder ver los objetos.

```js
-const fov = 75;
+const fov = 40;
const aspect = 2;  // el valor por defecto del canvas
const near = 0.1;
-const far = 5;
+const far = 1000;
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
-camera.position.z = 2;
+camera.position.z = 120;
```

Vamos a agregar una función `addObject`, que toma una posición x, y y un `Object3D` y agrega el objeto a la escena.

```js
const objects = [];
const spread = 15;

function addObject(x, y, obj) {
  obj.position.x = x * spread;
  obj.position.y = y * spread;

  scene.add(obj);
  objects.push(obj);
}
```

También vamos a hacer una fnción que cree un material coloreado de
forma aleatoria. Usaremos la característica de `Color` que permite 
establecer un color basado el parámetros como tono, saturación, luminosidad.

`hue` (tono) va desde 0 hasta 1 en la rueda de colores con el
rojo en 0, verde en .33 y azul en .66. `saturation` (saturación)
va desde 0 hasta 1, siendo 0 sin color y 1 el más saturado.
`luminance` (lumnisosidad) va desde 0 a 1, con 0 siendo negro,
1 siendo blanca y 0.5 siendo el monto máximo de color. En otras
palabras, `luminance` va desde 0.0 a 0.5 el color irá desde el
negro a `hue`. De 0.5 a 1.0 el color irá desde `hue` al blanco.

```js
function createMaterial() {
  const material = new THREE.MeshPhongMaterial({
    side: THREE.DoubleSide,
  });

  const hue = Math.random();
  const saturation = 1;
  const luminance = .5;
  material.color.setHSL(hue, saturation, luminance);

  return material;
}
```

También pasamosd `side: THREE.DoubleSide` al material.
Esto le dice a three que dibujar ambos lados del triángulo
que forman una figura. Para una figura sólida como una esfera
o un cubo generalmente no hay motivo para dibujar los lados
traseros de los triángulos, ya que las mismas se encuentran
dentro de la figura. En nuestro caso pensamos dibujar cosas
como el `PlaneGeometry` y `ShapeGeometry` que son en
2 dimensiones y no tiene lados internos.
Sin `side: THREE.DoubleSide` desaparecerían cuando los miremos
por la parte posterior.

Cabe mencionar que es más rápido de dibujar cuando **evitamos**
poner `side: THREE.DoubleSide`, así que deberíamos usarlo sólo
en materiales que realmente lo necesiten, ya que en este caso 
los dibujos son muy simples, no hay mucho de que preocuparse.

Haremos una función `addSolidGeometry`, que recibe una
geometría y crea un material coloreado de forma aleatoria
via `createMaterial` y la agrega a la escena mediante `addObject`.

```js
function addSolidGeometry(x, y, geometry) {
  const mesh = new THREE.Mesh(geometry, createMaterial());
  addObject(x, y, mesh);
}
```

Podemos usar esto para la mayoría de los primitivos que creamos.
Por ejemplo para crear una caja:

```js
{
  const width = 8;
  const height = 8;
  const depth = 8;
  addSolidGeometry(-2, -2, new THREE.BoxGeometry(width, height, depth));
}
```

Si miras en el código abajo verás una sección similar para cada tipo de geometría

Aquí puedes ver el resultado:

{{{example url="../threejs-primitives.html" }}}

Hay un par de excepciones notables en los patrones de arriba.
La más grande es probablemente `TextGeometry`. Necesita cargar
los datos de la fuente 3D antes de poder generar un mesh para el texto.
Estos datos se cargan de forma asíncrona así que necesitamos esperar
por ellos antes de poder crear la geometría. La carga de la fuente se
puede hacer de forma más simple usando `Promise`.
Creamos un `FontLoader` y una función `loadFont` que regresa una

promesa que al resolverse nos dará la fuente. Podemos usar una función
`async` llamada `doit` y cargar la fuente usando `await`.
Hay otra diferencia. Queremos girar el texto alrededor de su propio
centro pero three.js por defecto crea el texto con su centro de rotación a
la izquierda. Para poder hacerlo le pedimos a three.js calcular el encapsulado
de caja de la geometría. Podemos hacerlo llamando el método `getCenter` del
encapsulado y pasarlo a la posición del mesh de nuestro objeto.
Las copias de `getCenter` centran la caja en la posición.
También regresa la posición del objeto para que podamos usar `multiplyScalar(-1)`
para posicionar el objeto entero cuyo centro de rotación sea el mismo qu el
centro del objeto.

Sí sólo llamamos `addSolidGeometry` como en los ejemplos previos
establecería las posiciones como al inicio, lo que no es tan
buena idea. Entonces, para este caso crearemos un objeto `Object3D ` que
es el nodo estándar para una escena de grafos de three.js `Mesh`
es heredado de `Object3D`. Entenderemos mejor como funcionan las escenas
de grafos [en otro artículo](threejs-scenegraph.html).
Por ahora, es suficiente saber que al igual que los nodos del DOM, los hijos
son dibujados en relación a sus padres.
Creando un `Object3D` y haciendo al mesh un hijo de este,
podemos establecer la posición de `Object3D` donde queramos y todavía
conservaremos el centro que definimos antes.

Si no hacemos esto el texto no giraría sobre su centro.

{{{example url="../threejs-primitives-text.html" }}}

Nota como la figura de la izquierda no girar sobre si misma
mientras que la de la derecha si lo hace.

The other exceptions are the 2 line based examples for `EdgesGeometry`
and `WireframeGeometry`. Instead of calling `addSolidGeometry` they call
`addLineGeometry` which looks like this

```js
function addLineGeometry(x, y, geometry) {
  const material = new THREE.LineBasicMaterial({color: 0x000000});
  const mesh = new THREE.LineSegments(geometry, material);
  addObject(x, y, mesh);
}
```

It creates a black `LineBasicMaterial` and then creates a `LineSegments`
object which is a wrapper for `Mesh` that helps three know you're rendering
line segments (2 points per segment).

Each of the primitives has several parameters you can pass on creation
and it's best to [look in the documentation](https://threejs.org/docs/) for all of them rather than
repeat them here. You can also click the links above next to each shape
to take you directly to the docs for that shape.

There is one other pair of classes that doesn't really fit the patterns above. Those are
the `PointsMaterial` and the `Points` class. `Points` is like `LineSegments` above in that it takes a
a `BufferGeometry` but draws points at each vertex instead of lines.
To use it you also need to pass it a `PointsMaterial` which
take a [`size`](PointsMaterial.size) for how large to make the points.

```js
const radius = 7;
const widthSegments = 12;
const heightSegments = 8;
const geometry = new THREE.SphereGeometry(radius, widthSegments, heightSegments);
const material = new THREE.PointsMaterial({
    color: 'red',
    size: 0.2,     // in world units
});
const points = new THREE.Points(geometry, material);
scene.add(points);
```

<div class="spread">
<div data-diagram="Points"></div>
</div>

You can turn off [`sizeAttenuation`](PointsMaterial.sizeAttenuation) by setting it to false if you want the points to
be the same size regardless of their distance from the camera.

```js
const material = new THREE.PointsMaterial({
    color: 'red',
+    sizeAttenuation: false,
+    size: 3,       // in pixels
-    size: 0.2,     // in world units
});
...
```

<div class="spread">
<div data-diagram="PointsUniformSize"></div>
</div>

One other thing that's important to cover is that almost all shapes
have various settings for how much to subdivide them. A good example
might be the sphere geometries. Spheres take parameters for
how many divisions to make around and how many top to bottom. For example

<div class="spread">
<div data-diagram="SphereGeometryLow"></div>
<div data-diagram="SphereGeometryMedium"></div>
<div data-diagram="SphereGeometryHigh"></div>
</div>

The first sphere has 5 segments around and 3 high which is 15 segments
or 30 triangles. The second sphere has 24 segments by 10. That's 240 segments
or 480 triangles. The last one has 50 by 50 which is 2500 segments or 5000 triangles.

It's up to you to decide how many subdivisions you need. It might
look like you need a high number of segments but remove the lines
and the flat shading and we get this

<div class="spread">
<div data-diagram="SphereGeometryLowSmooth"></div>
<div data-diagram="SphereGeometryMediumSmooth"></div>
<div data-diagram="SphereGeometryHighSmooth"></div>
</div>

It's now not so clear that the one on the right with 5000 triangles
is entirely better than the one in the middle with only 480.
If you're only drawing a few spheres, like say a single globe for
a map of the earth, then a single 10000 triangle sphere is not a bad
choice. If on the other hand you're trying to draw 1000 spheres
then 1000 spheres times 10000 triangles each is 10 million triangles.
To animate smoothly you need the browser to draw at 60 frames per
second so you'd be asking the browser to draw 600 million triangles
per second. That's a lot of computing.

Sometimes it's easy to choose. For example you can also choose
to subdivide a plane.

<div class="spread">
<div data-diagram="PlaneGeometryLow"></div>
<div data-diagram="PlaneGeometryHigh"></div>
</div>

The plane on the left is 2 triangles. The plane on the right
is 200 triangles. Unlike the sphere there is really no trade off in quality for most
use cases of a plane. You'd most likely only subdivide a plane
if you expected to want to modify or warp it in some way. A box
is similar.

So, choose whatever is appropriate for your situation. The less
subdivisions you choose the more likely things will run smoothly and the less
memory they'll take. You'll have to decide for yourself what the correct
tradeoff is for your particular situation.

If none of the shapes above fit your use case you can load
geometry for example from a [.obj file](threejs-load-obj.html)
or a [.gltf file](threejs-load-gltf.html). 
You can also create your own [custom BufferGeometry](threejs-custom-buffergeometry.html).

Next up let's go over [how three's scene graph works and how
to use it](threejs-scenegraph.html).

<link rel="stylesheet" href="resources/threejs-primitives.css">
<script type="module" src="resources/threejs-primitives.js"></script>

