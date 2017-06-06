# Un introducción gentil a Test Driven Development en JavaScript: PARTE 2

Esta es la parte 2 de una serie de tres partes introduciendo mi acercamiento personal a JavaScript TDD. En el artículo anterior comenzamos creando una pequeña aplicación que carga data de imágenes del API de Flickr y mostrarlos en una página web. Comenzamos por configurar los módulos y escribir unos simples unit test usando el framework Mocha. En este artículo veremos como probar llamadas asíncronas de red (también llamadas AJAX). 


1. [Comenzando con unit tests](/part-1-Comenzando-con-pruebas-unitarias)
2. [Trabajando con network request en TDD](/part-2-Trabajando-peticiones-asincronas-tdd)
3. Trabajando con DOM en TDD (WIP)


## Probando llamadas de red asíncronas (AJAX)
En el último artículo bromeé sobre procastinar las pruebas del código donde llamamos al API de Flickr. Y sin ninguna razón. Estaba procastinando porque las pruebas a llamadas asíncronas en red son un poco complicadas. Hay tres cosas que hacen de esto un poco tricky: 

1. Probar una llamada a API necesita acceso a la red, lo cual no siempre está garantizado. 
2. Las llamadas de red en JavaScript son asíncronas. Esto significa que cuando solucitud de red interrumpimos el flujo normal decódigo; y 
3. Los resultados de una llamada de red cambian cada cierto tiempo. Es es el punto de las llamadas de red - pero de alguna manera los hace difíciles de testear. 

_Podría_ avanzar y simplemente escribir una prueba que hace la llamada de red y revisa lo que regresa, pero tiene sus problemasL 

- La data que viene del API de Flickr en vivo cambia todo el tiempo. A menos que sea cuidadoso en cómo escribo mis test, pasarán por tal vez un minuto y luego la nueva data romperá mis pruebas. 
- Hacer una llamada de red puede ser lento, y eso hace más lentos mis tests, lo que hace TDD menos divertido. 
- Hacer cosas así necesita conexión a internet. A veces me encuentro escribiendo código en el bus, o en el tren, o algún lugar sin una (rápida) conexión a internet. 

Así que, necesito pensar con cuidado sobre qué quiero probar. Crearé un método `fetchFlickrData()` que obtiene la data del API de Flickr. Para este trabajo, necesito hacer una llamada de red. Pero para hacer la llamada de red, estaré llamando a algún tipo de API. El API más simple para este propósito sería el método `getJSON()` de jQuery. `getJSON()` toma un URL y retorna un Promise para la data de JSON. Si no estás familiarizado con Promises, vale la pena que tomes un momento en tener la idea básica. <a name="reference-1"></a>[[1]](#footnote-1)

Ahora, para manejar esto ordenadamente, necesito pensar como un programador funcional. Las llamadas a red incluyen efectos colaterales, haciendo mi función impura. Pero, si puedo aislar la parte impura (por ejemplo `getJSON()`), entonces tendré una función pura, y _testeable_. En otras palabras, ¿Qué pasa si hago de `getJSON`un parámetro que paso a mi función? La firma podría lucir como: 

```JavaScript
fetchFlickrData: function(apiKey, fetch) { 
    // Code goes in here
}
```

En el código de la aplicación, puedo pasar `$.getJSON`como un parametro de `fetch`(más de esto, más adelante). En mi _prueba_ sin embargo, puedo pasar un _falso_ método `getJSON()` que simpre retorna una promesa para la misma data. Entonces puedo revisar si mi función devuelve exactamento lo que espero, sin hacer la llamada de red. 

La otra cosa un poco dificultosa sobre las llamadas de red con JavaScript es que son _asíncronas_. Lo que significa que necesitamos alguna manera de decirle a nuestro test runner (Mocha) que espere hasta que todos los test hayan terminado. Mocha provee un parámetro al callback de `it()`llamado `done`que permite decirle a Mocha cuando los test estan completados. 

Poniendo todo esto junto, puedo escribir un test como sigue: 

```JavaScript
// flickr-fetcher-spec.js
describe('#fetchFlickrData()', function() {
    it(
        'should take an API key and fetcher function argument and return a promise for JSON data.',
        function(done) {
            var apiKey      = 'does not matter much what this is right now',
                fakeData    = {
                    'photos': {
                        'page':    1,
                        'pages':   2872,
                        'perpage': 100,
                        'total':   '287170',
                        'photo':   [{
                            'id':       '24770505034',
                            'owner':    '97248275@N03',
                            'secret':   '31a9986429',
                            'server':   '1577',
                            'farm':     2,
                            'title':    '20160229090898',
                            'ispublic': 1,
                            'isfriend': 0,
                            'isfamily': 0
                        }, {
                            'id':       '24770504484',
                            'owner':    '97248275@N03',
                            'secret':   '69dd90d5dd',
                            'server':   '1451',
                            'farm':     2,
                            'title':    '20160229090903',
                            'ispublic': 1,
                            'isfriend': 0,
                            'isfamily': 0
                        }]
                    }
                },
                fakeFetcher = function(url) {
                    var expectedURL = 'https://api.flickr.com/services/rest/?method=flickr.photos.search&api_key='
                                + apiKey + '&text=pugs&format=json&nojsoncallback=1'
                    expect(url).to.equal(expectedURL)
                    return Promise.resolve(fakeData);
                };
            FlickrFetcher.fetchFlickrData(apiKey, fakeFetcher).then(function(actual) {
                expect(actual).to.eql(fakeData);
                done();
            }
        );

    });
});
```
Me he puesto un poco listillo aquí, y he incluido un `expect`dentro de la función falsa de fetch. Esto me permite revisar si estoy llamado al URL adecuado. Corramos el test: 

![El gato esta triste, porque no hay una función aún][sad-nyan-cat-1]
El gato esta triste, porque no hay una función aún


### Stubs 

Ahora que los test están fallando, tomemos un momento para hablar de que es lo que está haiendo. La función `fakeFetcher()` que he usado para reemplazar el `$.getJSON()` es conocido como ***stub***. Un stub es una pieza de código que tiene el mismo API y comportamiento como el código 'real', pero con una funcionalidad mucho más reducida. Usualmente esto significa retornar data estática en lugar de interactuar con algunos recursos externos. 

Los Stubs puedes reemplazar diferentes tipos de código a parte de llamadas a red. Más a menudo los usamos para lo que los programadores funcionales llaman _side effects_ (efectos colaterales, o secundarios). Típicamente los stubs pueden reemplazar cosas como: 

- Queries a una base de datos relacional;
- Interacción con el sistema de archivos;
- Aceptar input de usuario; o 
- Computaciones complejas que toman un timpo largo de calcular. 

Los stubs no siempre tienen que reeemplazar cosas asíncronas o incluso lentas. Puede simplemente ser un pedazo de código que aún no has escrito. Un stub puede reemplazar casi todo. 

Stubs son una herramienta importante en TDD. Nos ayudan a mentener los test corriendo rápidamente así nuestro flujo de trabajo no se hace más lento. Mas importante, nos permite tener test consistentes para cosas que por herencia puede ser variables (como las llamadas a red).

Stubs pueden tomar un poco de esfuerzo en usarse bien. Por ello, usar un stub significa agregar un parámetro extra a la función `fetchFlickrData()`. Sin embargo, si estás usando algún [sabor de estilo de programación funcional](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-intro), entonces, de alguna manera, estarás pensando sobre cosas como los side effects y funciones puras. Argumentaría también que hacer tu código testeable (sea que uses stubs o no) usualmente vale la pena el esfuerzo. 

Pero suficiente sobre stubs-regresemos a codear...

***

Corremos las pruebas, tengo un error, pero eso es aún un gato triste (_rojo_), así que escribo algo de código. En este caso, retonar el resultado esperado no es tan simple. Tengo dos llamadas `expect()` ahí, así que tengo que llamar la función fetcher así como retorna la promesa para la data. En este caso el código más facilr de escribir es el código general: 

```JavaScript 
// flickr-fetcher
fetchFlickrData: function(apiKey, fetch) {
    var url = 'https://api.flickr.com/services/rest/?method=flickr.photos.search&api_key='
            + apiKey + '&text=pugs&format=json&nojsoncallback=1'
    return fetch(url).then(function(data) {
        return data;
    });
}
````

Corremos los tests de nuevo, y el gato es feliz de nuevo (verde). Así que es momento de refactorizar. 

Esta vez hay dos cosas que quiero refactorizar. Primero que nada, no hay necesidad de usar `.then()`en la función `fetchFlickrData()`. Así que refatorizo el código redundante: 

```JavaScript
fetchFlickrData: function(apiKey, fetch) {
    var url = 'https://api.flickr.com/services/rest/?method=flickr.photos.search&api_key='
            + apiKey + '&text=pugs&format=json&nojsoncallback=1'
    return fetch(url);
}
````

Corremos los test de nuevo, todo aún pasa. Pero aún me gustaría refactorizar mi código de prueba. Mocha provee de _dos_ maneras de manejar código asíncrono. La primera es la función `done()` como vimos antes. La segunda es específicamente para Promises. Si retornas un Promise desde tu test, Mocha actumáticamente esperara a que se resuelva o rechace. 

```JavaScript
/ flickr-fetcher-spec.js
describe('#fetchFlickrData()', function() {
    it(
        'should take an API key and fetcher function argument and return a promise for JSON data.',
        function() {
            var apiKey      = 'does not matter much what this is right now',
                fakeData    = {
                    'photos': {
                        'page':    1,
                        'pages':   2872,
                        'perpage': 100,
                        'total':   '287170',
                        'photo':   [{
                            'id':       '24770505034',
                            'owner':    '97248275@N03',
                            'secret':   '31a9986429',
                            'server':   '1577',
                            'farm':     2,
                            'title':    '20160229090898',
                            'ispublic': 1,
                            'isfriend': 0,
                            'isfamily': 0
                        }, {
                            'id':       '24770504484',
                            'owner':    '97248275@N03',
                            'secret':   '69dd90d5dd',
                            'server':   '1451',
                            'farm':     2,
                            'title':    '20160229090903',
                            'ispublic': 1,
                            'isfriend': 0,
                            'isfamily': 0
                        }]
                    }
                },
                fakeFetcher = function(url) {
                    var expectedURL = 'https://api.flickr.com/services/rest/?method=flickr.photos.search&api_key='
                                + apiKey + '&text=pugs&format=json&nojsoncallback=1'
                    expect(url).to.equal(expectedURL)
                    return Promise.resolve(fakeData);
                };
            return FlickrFetcher.fetchFlickrData(apiKey, fakeFetcher).then(function(actual) {
                expect(actual).to.eql(fakeData);
            }
        );

    });
});
```

Corremos el código refactorizad, las pruebas pasan aún, así que el siguiente paso. 

### Construyendo

En este punto, necesito para y pensar. Hay una cosa final antes de poder declarar al módulo `FlickrFetcher` como listo: ¿Las piezas encajan juntas sin problemas? ¿Puedo hacer una llamada de red, y obtener los resultad, y transformarlos en el formato que quiero? Sería más conveniente si pudiera hacer esto en una función.

Así que escribo un test: 

```JavaScript 
describe('#fetchPhotos()', function() {
    it('should take an API key and fetcher function, and return a promise for transformed photos', function() {
        var apiKey   = 'does not matter what this is right now',
            expected = [{
                title: 'Dog goes to desperate measure to avoid walking on a leash',
                url:   'https://farm2.staticflickr.com/1669/25373736106_146731fcb7_b.jpg'
            }, {
                title: 'the other cate',
                url:   'https://farm2.staticflickr.com/1514/24765033584_3c190c104e_b.jpg'
            }],
            fakeData = {
                'photos': {
                    'page':    1,
                    'pages':   2872,
                    'perpage': 100,
                    'total':   '287170',
                    'photo':   [{
                        id:       '25373736106',
                        owner:    '99117316@N03',
                        secret:   '146731fcb7',
                        server:   '1669',
                        farm:     2,
                        title:    'Dog goes to desperate measure to avoid walking on a leash',
                        ispublic: 1,
                        isfriend: 0,
                        isfamily: 0
                    }, {
                        id:       '24765033584',
                        owner:    '27294864@N02',
                        secret:   '3c190c104e',
                        server:   '1514',
                        farm:     2,
                        title:    'the other cate',
                        ispublic: 1,
                        isfriend: 0,
                        isfamily: 0
                    }]
                }
            },
            fakeFetcher = function(url) {
                var expectedURL = 'https://api.flickr.com/services/rest/?method=flickr.photos.search&api_key='
                            + apiKey + '&text=pugs&format=json&nojsoncallback=1'
                expect(url).to.equal(expectedURL)
                return Promise.resolve(fakeData);
            };

        return FlickrFetcher.fetchPhotos(apiKey, fakeFetcher).then(function(actual) {
            expect(actual).to.eql(expected);
        });
    });
});
```

Nota que aún estoy usando la función de falso fetcher como una dependencia externa. Corremos la prueba, tnego un error. El gato esta triste, así que puedo escribir algo de código. 

Porque solo estoy llamando a dos funciones, es fácil escribir el caso general para que retonner el valor esperado. 

```JavaScript
fetchPhotos: function(apiKey, fetch) {
    return FlickrFetcher.fetchFlickrData(apiKey, fetch).then(function(data) {
        return data.photos.photo.map(FlickrFetcher.transformPhotoObj);
    });
}
```
Corremos los test nuevamente, mis tests pasan - gato feliz (_verde_). Así que es momento de refactorizar. Pero, desde que la función es solo tres o cuatro (dependiendo de cómo cuentes) llamadas, no hay mucho que refactorizar. <a name="reference-2"></a>[[2]](#footnote-2) Así que por el momento, he completado mi primer módulo. 

¿Qué hemos cubierto? En este artículo hemos cubierto dos tópicos principales: Probar código asíncrono y usar stubs para estandarizar cosas como las llamadas a red. En el siguiente artículos nos enfocaremos en _[trabar con HTML y el DOM](http://jrsinclair.com/articles/2016/gentle-introduction-to-javascript-tdd-html-dom)_





<a name="footnote-1"></a>
1. Si, si estas usando una versión de jQuery anterior a 3.0 , es sólo como Promises, pero de cualquier manera es lo suficientemente cercano para lo que vamos a usar aquí.[↩](#reference-1)

<a name="footnote-2"></a>
2. Podría ser un poco más breve si estuviera usando técnicas de programación funcional, como la función `mapWith()` y/o `partial()`, pero para usar cualquiera de las dos debo introducir dependencias o escribir mi propia implementación.[↩](#reference-2)


[sad-nyan-cat-1]: http://jrsinclair.com/assets/fetchFlickrData-error.png

_Disclaimer: This is a translation from the original article of James Sinclair, A gentle introduction to JavaScript Test Driven Development. I don't own this article. It is property of their corresponding authors._
