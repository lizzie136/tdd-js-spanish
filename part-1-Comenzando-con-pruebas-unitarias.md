# Un introducción gentil a Test Driven Development en JavaScript: PARTE 1


_Este artículo originalmente fue publicado en Inglés por James Sinclair el 11 de Abril del 2016_

Esta es la parte uno de una serie de tres partes donde explico en términos generales mi acercamiento personal a Desarrollo Impulsado o dirigido por Pruebas (_Test driven development_ o TDD). Sobre el curso de esta serie, trabajaré el desarrollo de una aplicación completa (aunque una pequeña, simple) en JavaScript que involucra hacer peticiones de red (también conocidas como AJAX) y manipulación de DOM. Las varias partes son las que siguen: 

1. [Comenzando con unit tests](/tdd-js-spanish/part-1-Comenzando-con-pruebas-unitarias)
2. [Trabajando con network request en TDD](/tdd-js-spanish/part-2-Trabajando-peticiones-asincronas-tdd)
3. Trabajando con DOM en TDD (WIP)

## ¿Por qué Test Driven Development?
Comenzar con desarrollo dirigido por pruebas (TDD) puede ser intimidante. Suena tedioso, aburrido y complicado. La palabra 'prueba' llama a pensar en exámenes y estrés y vigilantes y todo tipo de molestias. Y puede parecer un desperdicio escribir código que no _hace_ nada útil más all de decirte que el código que ya escribiste está funcionando. Y encima de todo, hay un confusa colección de frameworks y librerías ahí fuera. Algunas trabajan en el servidor, otras trabajan en el navegador, otras hacen las dos cosas... puede ser duro el sólo saber por donde empezar.

___

> Las objeciones predecibles son "Escribir pruebas unitarias toma mucho tiempo, " o "¿Cómo escribo los test primero si no sé como que es lo que hace aún?" y entonces esta la excusa popular: "Las pruebas unitarias no atraparán todos los bugs". <a name="reference-1"></a>[[1]](#footnote-1)


Existen, sin embargo, muchas buenas razones para intentar TDD. Aquí tienes tres de las que creo importantes: 

1. **Te fuerza a pensar**. Esta razón es más útil de lo que suena. Escribir una prueba me fuerza a pensar claramente sobre qué es lo que estoy tratando de alcanzar, al punto de tener un nivel de detalle que la computadora pueda revisar. Una vez que tengo todo claro en mi cabeza, se vuelve mucho más sencillo escribir el código. Si estoy sufriendo por escribir una prueba, sé que no he entendido completamente el problema que estoy tratando de resolver. 

2. **Hace el debugging más secillo**. Mientras TDD no causará que escribas menos bugs(tristemente), sí hace un más sencillo hacerles seguimiento cuando aparezcan inevitablemente. Y si luego escribes un test relacionado al bug, me da más confianza saber que definitivamente he resuelto un bug en particular. Puedo recorrer todas mis otras pruebas para revisar que mi solución del bug no ha roto otros bits de código. <a name="reference-2"></a>[[2]](#footnote-2)

3. **Hace el codear mucho más divertido**. En mi mente, esta es la razón que pase más sobre las otras dos. Practicar los pasos simples de TDD es un poco adictivo y divertido. Toma un poco acostumbrarse a la disciplina de TDD, pero una vez que lo tomas, disfrutas más codear. 

Estas son algunas de las razones para tomar TDD, pero soy optimista en que son suficientes para convencerte de intentarlo. En el momento empezaremos trabajando un ejemplo sencillo, pero antes, comenzemos con la líneas generales de cómo trabaja TDD. 


### ¿Qué es TDD?
TDD es un enfoque para escribir software donde escribes las pruebas antes de escribir el código de la aplicación. Los pasos básicos son: 

1. **Rojo**: Escribe un test y asegúrate que falle. 
2. **Verde**: Escribe el código más simple y fácil posible para hacer que el código pase. 
3. **Refactor**: Optimiza y/o simplifica el código de la aplicación, asegurandote que los test aún pasen. 

Una vez hemos terminado el paso 3, comenzamos el ciclo de nuevo escribiendo otro test. 

Estos tres pasos forman el matra de TDD: 'rojo, verde, refactor'. Examinaremos cada uno de estos en detalle mientras vamos con un ejemplo. Pero primero, una acotación final. 

TDD es una forma de auto disciplina —un life hack— no te hace mágicamente un@ mejor coder. En teoría, no hay razón por la que un gran coder no escriba exactamente el mismo código que alguien que no lo es. Pero la realidad es que la disciplina de TDD te alienta a :

1. Escribir tests; y
2. Escribir pequeñas unidades de código que son fáciles de entender.

Personalmente , encuentro que no estoy practicando TDD, apenas si escribo algún test, y las funciones que escribo son largas y muy complicadas. Eso no quiere decir que no estoy testeando —Estoy clickeando en el botón de refresh en mi navegador todo el tiempo—pero mis test no sirven para nada más que para mí. 

### Un ejemplo trabajado
Tomemos un ejemplo típico en JavaScript para hacer en este ejemplo: Obtener data de un servidor (en este caso, una lista de fotos de Flickr.com), tranformarlo en HTML, y agregarlo a una página web. Puedes [ver el resultado final en este  CodePen](http://codepen.io/jrsinclair/pen/EKQmwo). 

Para este ejemplo, usaremos el framework [Mocha](http://mochajs.org/). He escogido Mocha, no porque sea el framework más popular de testing en JavaScript (aunque lo es); no porque sea mejor que otros frameworks de pruebas ([que no lo es](https://medium.com/javascript-scene/why-i-use-tape-instead-of-mocha-so-should-you-6aa105d8eaf4#.bb72m9mmf)); mas bien por la simple razón que tengo la opción de agregar `--reporter=nyan` en línea de comandos, lo que hace que mis reporte de pruebas tengan un gato arcoiris volar por el espacio (nyan cat). Y eso lo hace [más divertido](http://jrsinclair.com/articles/2016/tdd-should-be-fun):

```
mocha --reporter=nyan
```

### Configurando

Para este tutorial, correremos nuestros test en lína de comandos usando Node. Ahora puedes estar pensando, '¿no estamos escribiendo una aplicación web que correrá enteramente en el browser?'. Pero correr nuestros tests en Node es mucho más rápido, y las diferencias entre el browser y Node nos ayudarán a pensar con cuidado la estructura de nuestro código (más de esto más adelante).

Para comenzar, necesitaremos Node installado, además de Mocha y otro módulo llamado Chai. Si estás usando OS X, entonces te recomiendo usar Homebrew para instalar Node, así es más fácil tenerlo actualizado. Una vez que tienes Homebrew configurado, puedes intalar Node con la línea de comandos que sigue: 

```
$ brew install node
```
Si estás en Linux, entonces puedes usar tu package manager del sistema(como `apt-get` o `yum`) para instalar Node.<a name="reference-3"></a>[[3]](#footnote-3)

Y si estas usando Windows, entonces te recomiendo visitar la página de Node, y descargar el instalador. 

Una vez que tienes Node instalado, podemos usar el manejador de paquetes de Node (npm) para instalar Mocha y Chai para nosotros. Asegúrate de cambiar al directorio donde vas a escribir tu código y correr los siguientes comandos 

```
cd /path/donde/escribiré/mi/código
npm install mocha -g
npm install chai
```

Ahora que tenemos los prerequisitos instalados, podemos empezar a pensar sobre la aplicación que queremos construir. 

### Pensando

Bueno, mientras decíamos hace un momento que sólo hay 3 pasos para TDD, no es enteramente verdad. Hay un paso cero. Tienes que pensar primero, luego escribir el test. Para ponerlo de otra manera: antes de escribir un test tienes que tener al menos una idea de que quieres alcanzar y como estructurás tu código. Es test drive _development_, no test drive _design_. 

Primero escribamos lo que queremos hacer en más detalle: 

1. Enviar un request al API de Flickr, y obtener un montón de datos de fotos; 
2. Transformar la data en un único arreglo de objetos, cada objeto conteniendo la data que necesitamos; 
3. Convertir el arreglo de objetos en una lista HTML;
4. Agregar el HTML a la página.

A continuación necesitamos pensar sobre la estructura del código. _Podría_ poner todo en un solo módulo. Pero, tengo pocas opciones de cómo hacer los dos últimos  (hacer el HTML y ponerlo en la página): 

- Puedo cambiar el DOM directamente y agregar HTML a la página usando las interfaces estandar de DOM; 
- Puedo usar jQuery para agregar el HTML a la página;
- Puedo usar un framework como React.js o una Vista de Backbone. 

Como es probable que use jQuery para hacer los request HTTP  al servidr, parece (a este punto, al menos) que el acercamiento más simple sería usar jQuery para manipular el DOM. Pero, en el futuro puedo cambiar de parecer y usar un componente de React. Así que tiene sentido manter la parte de la aplicación de obtener-y-tranformar un poco separado de la parte hacer-HTML-y-agregar-a-DOM. 

Con esto en mente, crearé cuatro archivos para albergar mi código. 

1. `flickr-fetcher.js` para el módilo que obtiene la data y la transforma;
2. `photo-lister.js` para el módulo que toma la lista, la convierte en HTML y agrega a la página. 
3. `flickr-fetcher-spec.js` para el código que probará `flickr-fetcher.js`; y 
4. `photo-lister-spec.js` para el código que probará `photo-lister.js`. 

### Escribiendo las pruebas

Con los archivos en su lugar puedo comenzar a pensar en escribir mi *primer test*. Ahora, quiero escribir el test más simple posible que igual movera mi código base hacia adelante. Algo útil de hacer a este punto es probar que puedo cargar el módulo. En `flickr-fetcher-spec.js` escribo: 

```JavaScript
// flickr-fetcher-spec.js
'use strict';
var expect = require('chai').expect;

describe('FlickrFetcher', function() {
    it('should exist', function() {
        var FlickrFetcher = require('./flickr-fetcher.js');
        expect(FlickrFetcher).to.not.be.undefined;
    });
});
```

Hay algunas cosas que notar aquí. Primero que todo, porque todo los tests corren usando Node, significa que importamos los módulos al estilo de node usando `require()`. 

La otra cosa que notar es que estamos usando el estilo 'Behaviour Driven Development' (BDD) para escribir los tests. Esta es una variación de TDD donde los test son escritos de la forma: _Describe [**algo**]. Eso debería [**hacer algo**]._ Entonces [_algo_] puede ser un módulo, o una clase, o un método, o una función. Mocha incluye una función de `describe()` y `it()` con lo que podemos escribir en este estilo. 

La tercera cosa que notar es la cadena `expect()` que hace la revisión. En este caso voy a revisar simplemente si mi módulo no es `undefined`. La mayoría de las veces, usaré el patrón `expect(actualValue).to.equal.(expectedValue)`; 

Así que corramos el test: 

```
mocha --reporter=nyan flickr-fetcher-spec.js
```

Si todo esta instalado correctamente, podemos ver a un gato feliz como el que se ve abajo 
 
![One passing test][happy-nyan-cat]
_Un test pasó_

Nuestro test pasa, lo cual parece raro dado que no he escrito ningún código del módulo. Esto es porque mi archivo `flickr-fetcher.js` existe (y Node te da un objeto vacío si requires o hacer `require` a un archivo en blanco). Cómo no tengo ningún test fallido, aunque no he escrito ningún código del módulo. La regla es: No se escribe código del módulo hasta que haya un test con errores. Así que, ¿ahora que hago? Escribo otro test -lo que significa _pensar_ de nuevo. 

Hayd dos cosas que quiero lograr: 

1. Obtener data de Flickr, y 
2. Transformar la data. 

Obtener la data de Flickr envuelve hacer una conexión de red, aunque, como un buen programador funcional, 
[voy a dejar esto para después](http://jrsinclair.com/articles/2016/gentle-introduction-to-javascript-tdd-intro/#fn:4) <a name="reference-4"></a>[[4]](#footnote-4). En su lugar, enfonquemonos en la transformación de la data. 


Quiero tomar cada uno de los objetos foto que Flickr da y transformarlo en un objeto que solo tiene la información que quiero - en este caso, el título y el url de la imagen. El URL tiene un truco, aunque porque le API de Flickr no retorna los URL completamente formados. En su lugar, tengo que [construir el URL en base al tamaño de la foto que quiero](https://www.flickr.com/services/api/misc.urls.html). Ahora, este parece un buen lugar para comenzar con el siguiente test:  Algo pequeño, testeable, que permite avanzar con el código base. Ahora puedo escribir el test. 

```JavaScript
// flickr-fetcher-spec.js
var FlickrFetcher = require('./flickr-fetcher.js');

describe('#photoObjToURL()', function() {
    it('should take a photo object from Flickr and return a string', function() {
        var input = {
            id:       '24770505034',
            owner:    '97248275@N03',
            secret:   '31a9986429',
            server:   '1577',
            farm:     2,
            title:    '20160229090898',
            ispublic: 1,
            isfriend: 0,
            isfamily: 0
        };
        var expected = 'https://farm2.staticflickr.com/1577/24770505034_31a9986429_b.jpg';
        var actual = FlickrFetcher.photoObjToURL(input);
        expect(actual).to.eql(expected);
    });
}); 

```
Nota que he usado `expect(actual).to.eql(expected);` en lugar de `expect(actual).to.equal(expected);`. Esto le dice a Chai que verifique cada value dentro de `expected`. La regla aquí es, usa `equal` cuando comparas numbers, strings o booleans, y usa `eql` cuando comparan arreglos u objetos. 

Así que corro el test de nuevo y...gato triste. Tengo un error. Eso significa que puedo escribir un poco de código. Paso uno es simplemente obtener la estructura del módulo en su lugar: 

```JavaScript
// flickr-fetcher.js
var FlickrFetcher;

FlickrFetcher = {
    photoObjToURL: function() {}
};

module.exports = FlickrFetcher;
```

Si corro mi test ahora, obtendre un fallo en lugar de un error, pero el gato sigue triste(_rojo_), así que puede seguir escribiendo código. La pregunta ahora es, ¿cuál es la código más sigue que puede escribir para hacer pasar el test? Y la respuesta es, claro, retornar el siguiente resultado: 

```JavaScript
var FlickrFetcher;

FlickrFetcher = {
    photoObjToURL: function() {
        return 'https://farm2.staticflickr.com/1577/24770505034_31a9986429_b.jpg';
    }
};
```

Corremos los tests de nuevo y todo pasa -gato feliz (verde). 

El siguiente paso es refactorizar. Hay alguna manera en que pueda hacer este código más eficiente o claro? En el momento creo es lo más eficiente y claro que puede ser. Pero, todos sabemos que esta función es bastante inutil. Puedes estar pensando "si pasas cualquier otro valor, la función no funcionará". Y ese es un buen punto. Debo escribir otro test y pasar otro objeto válido: 

```JavaScript 
// flickr-fetcher-spec.js
describe('#photoObjToURL()', function() {
    it('should take a photo object from Flickr and return a string', function() {
        var input = {
            id:       '24770505034',
            owner:    '97248275@N03',
            secret:   '31a9986429',
            server:   '1577',
            farm:     2,
            title:    '20160229090898',
            ispublic: 1,
            isfriend: 0,
            isfamily: 0
        };
        var expected = 'https://farm2.staticflickr.com/1577/24770505034_31a9986429_b.jpg';
        var actual = FlickrFetcher.photoObjToURL(input);
        expect(actual).to.eql(expected);

        input = {
            id:       '24770504484',
            owner:    '97248275@N03',
            secret:   '69dd90d5dd',
            server:   '1451',
            farm:     2,
            title:    '20160229090903',
            ispublic: 1,
            isfriend: 0,
            isfamily: 0
        };
        expected = 'https://farm2.staticflickr.com/1451/24770504484_69dd90d5dd_b.jpg';
        actual = FlickrFetcher.photoObjToURL(input);
        expect(actual).to.eql(expected);
    });
});
```
Corre el test, y falla -gato triste. 

Ahora que tenemos un nuevo test, la pregunta es, ¿Cuál es el código más simple que puedo escribir para pasar este test? Con dos tests la respuesta no es tan simple. Yo _podría_ escribir un statement if y retornar el segundo URL, pero es casi el mismo esfuerzo que escribir un código más general, así que haré eso: 

```JavaScript 
// flickr-fetcher.js
FlickrFetcher = {
    photoObjToURL: function(photoObj) {
        return 'https://farm' + photoObj.farm + '.staticflickr.com/' + photoObj.server + '/' + photoObj.id + '_' +
            photoObj.secret + '_b.jpg';
    }
};
```

Corremos los tests de nuevo -gato faliz. Tengo una función funcionando. 

Estamos de regreso al paso de refactor. Ahora, este código es aún bastante simple, pero todos esos signos de suma lucen un poco feos para mí. Una manera de no tenerlos es usar una librería de templating (como Handlbars o [algo más simple](http://mir.aculo.us/2011/03/09/little-helpers-a-tweet-sized-javascript-templating-engine/), pero no parece agregar valor el código extra a esta función. Podría intentar otra cosa. Si pongo todas las partes de la cadena en un arreglo, puedo pegar todo con el método `join()`. Como valor agregado, la mayoría de implementaciones de JavaScript correran el join siempre un poco más rápido que la concatenación con +. Así que refactorizo usando `join()`:

```JavaScript 
FlickrFetcher = {
    photoObjToURL: function(photoObj) {
        return [ 'https://farm',
            photoObj.farm, '.staticflickr.com/', 
            photoObj.server, '/',
            photoObj.id, '_',
            photoObj.secret, '_b.jpg'
        ].join('');
    }
};
```

Corro el test nuevamente, y mi test pasará, así sé que todo funciona. Momento de movernos al siguiente test...

A este punto, si quiero escribir un módulo para ser publicado en [npm](https://www.npmjs.com/), escribiría los test para cubrir las cosas más locas que alguien podría intentar pasar a la función. Por ejemplo: 

- ¿Qué pasaría si alguien pasa un string en vez de un objeto?
- ¿Qué debería pasar si alguien no pasa parámetros?
- ¿Qué debería pasar si alguien pasa un objeto con los nombres de las propiedades errados?
- ¿Qué debería pasar si alguien pasa un objeto con los nombres de propiedades correctos pero los valores no son strings?

Todas estas son buenas preguntas que hacernos, y probar, pero no iré por ese camino aquí: Primero que nada porque sería increíblemente aburrido de leer, y segundo porque este es un proyecto de juego que no es una misión crítica para nada. No estaré perdiendo el dinero de alguien o poniendo en riesgo la vida de alguien si este código no maneja los caso extremos con gracia. Por ahora, sé que hace lo que quiero que haga. Sin embargo, si estuviera escribiendo código de software de soporte vital o manejando los detalles de las tarjetas de crédito, o algo remotamente como eso, entonces definitivamente queremos responder todas esas preguntas. 

Hemos ido a través de todo el ciclo con una función funcionando: _rojo, verde, refactor_. Ahora es momento de seleccionar el siguiente test. Momento de _pensar_. Quiero tomar la lista de objetos _photo_ que Flickr nos da y transformarlo en una lista de objetos en los que tengamos solo la información que quiero. Si voy a procesar la lista, entonces probablemente envuelva algún tipo de [operación map](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-arrays/#map), así que quiero crear una función que procese un objeto a la vez. Eso me da otra unidad de código bonita, pequeña, y testeable para probar. Así que escribo el siguiente código: 

```JavaScript
describe('#transformPhotoObj()', function() {
    it('should take a photo object and return an object with just title and URL', function() {
        var input = {
                id:       '25373736106',
                owner:    '99117316@N03',
                secret:   '146731fcb7',
                server:   '1669',
                farm:     2,
                title:    'Dog goes to desperate measure to avoid walking on a leash',
                ispublic: 1,
                isfriend: 0,
                isfamily: 0
            },
            expected = {
                title: 'Dog goes to desperate measure to avoid walking on a leash',
                url:   'https://farm2.staticflickr.com/1669/25373736106_146731fcb7_b.jpg'
            },
            actual = FlickrFetcher.transformPhotoObj(input);
        expect(actual).to.eql(expected);
    });
});
```

Cuando corremos el test, obtenemos un error ya que dicha función no existe: 

![El gato esta triste porque la función no existe aún][sad-nyan-cat-2]
El gato esta triste porque la función no existe aún

Ahora que tenemos el gato triste (_rojo_), puedo escribir algo de código. ¿Cuál sería la forma más simple de pasar el test? Otra vez, solo crea una función que retorne el resultado esperado.

```JavaScript
    transformPhotoObj: function() {
        return {
            title: 'Dog goes to desperate measure to avoid walking on a leash',
            url:   'https://farm2.staticflickr.com/1669/25373736106_146731fcb7_b.jpg'
        };
    }
```

Vuelvo a correr los test, y es gato es feliz de nuevo (_verde_).

![3 tests que pasaron y un nyan cat feliz][happy-nyan-cat-3]
3 tests que pasaron y un nyan cat feliz

¿Puedo refactorizar este código? ¿O tódo mi código? A este punto probablemente no. Pero, este código no es muy útil, ya que sólo puedo manejar un input específico, así que necesito escribir otro test: 

```JavaScript
describe('#transformPhotoObj()', function() {
    it('should take a photo object and return an object with just title and URL', function() {
        var input = {
                id:       '25373736106',
                owner:    '99117316@N03',
                secret:   '146731fcb7',
                server:   '1669',
                farm:     2,
                title:    'Dog goes to desperate measure to avoid walking on a leash',
                ispublic: 1,
                isfriend: 0,
                isfamily: 0
            },
            expected = {
                title: 'Dog goes to desperate measure to avoid walking on a leash',
                url:   'https://farm2.staticflickr.com/1669/25373736106_146731fcb7_b.jpg'
            },
            actual = FlickrFetcher.transformPhotoObj(input);
        expect(actual).to.eql(expected);

        input = {
            id:       '24765033584',
            owner:    '27294864@N02',
            secret:   '3c190c104e',
            server:   '1514',
            farm:     2,
            title:    'the other cate',
            ispublic: 1,
            isfriend: 0,
            isfamily: 0
        };
        expected = {
            title: 'the other cate',
            url:   'https://farm2.staticflickr.com/1514/24765033584_3c190c104e_b.jpg'
        }
        actual = FlickrFetcher.transformPhotoObj(input);
        expect(actual).to.eql(expected);
    });
});
```

![Los test fallan y nuestro gato esta triste.][sad-nyan-cat-4]

Ahora, la más simple, y fácil de hacer los test pasar ahora es escribir una función completa, asegurándonos de usar la función `photoObjtO-URL()`.

```JavaScript
// flickr-fetcher.js
//… trimmed for brevity …
transformPhotoObj: function(photoObj) {
    return {
        title: photoObj.title,
        url:   FlickrFetcher.photoObjToURL(photoObj)
    };
}
```

Corro mis pruebas de nuevo, y tenemos un gato feliz (_verde_). 

![Tres que pasaron y un gato feliz][happy-nyan-cat-5]
Tres test que pasaron y un gato feliz. 

Lo siguiente es refactorizar. ¿Esta función se puede mejorar? A este nivel, probablemente no. Pero es importante seguir preguntándonos esa pregunta cada vez. Refactorizar es una de las delicias de programar y debes saborearlo siempre que sea posible. 

Por ahora debe tener una sensación de los pasos básicos de TDD: Rojo, verde y refactor. En este artículo hemos visto como es importante pensar antes de escribir un test -TDD no es un reemplazo de un buen diseño de software. En los siguientes dos artículos examinaremos 
[cómo manejar llamadas asíncronas en red](/tdd-js-spanish/part-2-Trabajando-peticiones-asincronas-tdd) y [cómo testear código que manipula el DOM sin un navegador](http://jrsinclair.com/articles/2016/gentle-introduction-to-javascript-tdd-html-dom).

***

<a name="footnote-1"></a>
1. Jeff Patton, 21 January 2005, Test-Driven Development Isn’t Testing  [↩](#reference-1)


<a name="footnote-2"></a>

2. Si, sé que estás pensando, "volver a correr los test no garantiza que no has introducido nuevos bugs". Eso es correcto. Pero volver a correr los test revisan errores de regresión y es mucho mejor que probar nada. Y si mis test comprehensibles pasan entonces puedo tener cierto nivel de confianza en que la lógica principal de negocio aún funciona. [↩](#reference-2)


 <a name="footnote-3"></a>
 
 3. Y aceptemoslo, so estas usando Linux y leyendo este artículo, probablemente ya tienes Node instalado.[↩](#reference-3)

 <a name="footnote-4"></a>

 4. Ver el siguiente artículo sobre cómo manejar llamadas de red. [↩](#reference-4)

[happy-nyan-cat]: http://jrsinclair.com/assets/1-passing-test.png

[sad-nyan-cat-2]: http://jrsinclair.com/assets/transformPhotoObj-error.png

[happy-nyan-cat-3]:http://jrsinclair.com/assets/3-passing-tests.png

[sad-nyan-cat-4]:http://jrsinclair.com/assets/transformPhotoObj-failing.png

[happy-nyan-cat-5]:http://jrsinclair.com/assets/3-passing-tests.png

_Disclaimer: This is a translation from the original article of James Sinclair, A gentle introduction to JavaScript Test Driven Development. I don't own this article. It is property of their corresponding authors._
