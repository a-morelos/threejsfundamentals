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

We'll go over creating custom geometry in [another article](threejs-custom-buffergeometry.html). For now
let's make an example creating each type of primitive. We'll start
with the [examples from the previous article](threejs-responsive.html).

Near the top let's set a background color

```js
const scene = new THREE.Scene();
+scene.background = new THREE.Color(0xAAAAAA);
```

This tells three.js to clear to lightish gray.

The camera needs to change position so that we can see all the
objects.

```js
-const fov = 75;
+const fov = 40;
const aspect = 2;  // the canvas default
const near = 0.1;
-const far = 5;
+const far = 1000;
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
-camera.position.z = 2;
+camera.position.z = 120;
```

Let's add a function, `addObject`, that takes an x, y position and an `Object3D` and adds
the object to the scene.

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

Let's also make a function to create a random colored material.
We'll use a feature of `Color` that lets you set a color
based on hue, saturation, and luminance.

`hue` goes from 0 to 1 around the color wheel with
red at 0, green at .33 and blue at .66. `saturation`
goes from 0 to 1 with 0 having no color and 1 being
most saturated. `luminance` goes from 0 to 1
with 0 being black, 1 being white and 0.5 being
the maximum amount of color. In other words
as `luminance` goes from 0.0 to 0.5 the color
will go from black to `hue`. From 0.5 to 1.0
the color will go from `hue` to white.

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

We also passed `side: THREE.DoubleSide` to the material.
This tells three to draw both sides of the triangles
that make up a shape. For a solid shape like a sphere
or a cube there's usually no reason to draw the
back sides of triangles as they all face inside the
shape. In our case though we are drawing a few things
like the `PlaneGeometry` and the `ShapeGeometry`
which are 2 dimensional and so have no inside. Without
setting `side: THREE.DoubleSide` they would disappear
when looking at their back sides.

I should note that it's faster to draw when **not** setting
`side: THREE.DoubleSide` so ideally we'd set it only on
the materials that really need it but in this case we
are not drawing too much so there isn't much reason to
worry about it.

Let's make a function, `addSolidGeometry`, that
we pass a geometry and it creates a random colored
material via `createMaterial` and adds it to the scene
via `addObject`.

```js
function addSolidGeometry(x, y, geometry) {
  const mesh = new THREE.Mesh(geometry, createMaterial());
  addObject(x, y, mesh);
}
```

Now we can use this for the majority of the primitives we create.
For example creating a box

```js
{
  const width = 8;
  const height = 8;
  const depth = 8;
  addSolidGeometry(-2, -2, new THREE.BoxGeometry(width, height, depth));
}
```

If you look in the code below you'll see a similar section for each type of geometry.

Here's the result:

{{{example url="../threejs-primitives.html" }}}

There are a couple of notable exceptions to the pattern above.
The biggest is probably the `TextGeometry`. It needs to load
3D font data before it can generate a mesh for the text.
That data loads asynchronously so we need to wait for it
to load before trying to create the geometry. By promisifiying 
font loading we can make it mush easier.
We create a `FontLoader` and then a function `loadFont` that returns
a promise that on resolve will give us the font. We then create
an `async` function called `doit` and load the font using `await`.
And finally create the geometry and call `addObject` to add it the scene.

```js
{
  const loader = new THREE.FontLoader();
  // promisify font loading
  function loadFont(url) {
    return new Promise((resolve, reject) => {
      loader.load(url, resolve, undefined, reject);
    });
  }

  async function doit() {
    const font = await loadFont('resources/threejs/fonts/helvetiker_regular.typeface.json');  /* threejsfundamentals: url */
    const geometry = new THREE.TextGeometry('three.js', {
      font: font,
      size: 3.0,
      height: .2,
      curveSegments: 12,
      bevelEnabled: true,
      bevelThickness: 0.15,
      bevelSize: .3,
      bevelSegments: 5,
    });
    const mesh = new THREE.Mesh(geometry, createMaterial());
    geometry.computeBoundingBox();
    geometry.boundingBox.getCenter(mesh.position).multiplyScalar(-1);

    const parent = new THREE.Object3D();
    parent.add(mesh);

    addObject(-1, -1, parent);
  }
  doit();
}
```

There's one other difference. We want to spin the text around its
center but by default three.js creates the text such that its center of rotation
is on the left edge. To work around this we can ask three.js to compute the bounding
box of the geometry. We can then call the `getCenter` method
of the bounding box and pass it our mesh's position object.
`getCenter` copies the center of the box into the position.
It also returns the position object so we can call `multiplyScalar(-1)`
to position the entire object such that its center of rotation
is at the center of the object.

If we then just called `addSolidGeometry` like with previous
examples it would set the position again which is
no good. So, in this case we create an `Object3D` which
is the standard node for the three.js scene graph. `Mesh`
is inherited from `Object3D` as well. We'll cover [how the scene graph
works in another article](threejs-scenegraph.html).
For now it's enough to know that
like DOM nodes, children are drawn relative to their parent.
By making an `Object3D` and making our mesh a child of that
we can position the `Object3D` wherever we want and still
keep the center offset we set earlier.

If we didn't do this the text would spin off center.

{{{example url="../threejs-primitives-text.html" }}}

Notice the one on the left is not spinning around its center
whereas the one on the right is.

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

