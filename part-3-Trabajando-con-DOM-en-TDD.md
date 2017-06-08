# Un introducción gentil a Test Driven Development en JavaScript: PARTE 3

Esta es la parte tres de mi serie de tres partes sobre Test Driven Development en JavaScript (TDD). En un artículo previo discutimos sobre cómo probar código asíncrono y cómo usar los stubs en lugar de cosas como llamadas a red. A través del curso de la serie hemos construido una aplicación de ejemplo para demostrar los conceptos de TDD. En este artículo, trabajaremos a través del código de prueba para generar HTML y manipular el DOM. Además terminaremos la aplicación de ejemplo poniendo todo junto y haciendo ajustes para mayor flexibilidad. 

1. [Comenzando con unit tests](/tdd-js-spanish/part-1-Comenzando-con-pruebas-unitarias)
2. [Trabajando con network request en TDD](/tdd-js-spanish/part-2-Trabajando-peticiones-asincronas-tdd)
3. [Trabajando con DOM en TDD](/tdd-js-spanish/part-3-Trabajando-con-DOM-en-TDD)


## Trabajando con HTML Strings

Bueno, ahora tengo un módulo que obtendrá la lista de fotos de Flickr y extraerá sólo la data que quiero. El siguiente paso para tomar la data y hacer algo con ella, en este caso, inyectarla en una página web. Así que crearé un nuevo módulo para manejar la presentación de data. 

Directamente, puedo escribir un test sencillo para revisar si el módulo existe. 

```JavaScript
// photo-lister-spec.js
var expect      = require('chai').expect,
    PhotoLister = require('./photo-lister');

describe('PhotoLister', function() {
    it('should exist', function() {
        expect(PhotoLister).not.to.be.undefined;
    });
});
````

Corremos los tests para este nuevo módulo quiere que ajustemos la línea de comando un poco: 

```bash
mocha --reporter=nyan photo-lister-spec.js
````

Corro las pruebas, y pasan, así que no escribo nada de código aún. <a name="reference-1"></a>[[1]](#footnote-1) Así que es momento de pensar un poco. Quiero tomar una lista de objetos photo y convertirlos en una lista de items HTML que contienen elementos `<figure>`. Ahora, en cualquier momento trabajo con listas, automáticamente considera usar `map` o `reduce` para procesar cada elemento uno por uno. Así que un buen lugar para comenzar sería hacer una función que tome un sólo objeto photo y lo transforme en el HTML que quiero. Así que escribo un test: 

```JavaScript
// photo-lister-spec.js
describe('#photoToListItem()', function() {
    it('should take a photo object and return a list item string', function() {
        var input = {
                title: 'This is a test',
                url:   'http://loremflickr.com/960/593'
            },
            expected = '<li><figure><img src="http://loremflickr.com/960/593" alt=""/>'
                     + '<figcaption>This is a test</figcaption></figure></li>';
        expect(PhotoLister.photoToListItem(input)).to.equal(expected);
    });
});
```

Nota que he usado `equal()` en lugar de `eql()` en el aserción. Porque estoy comparand strings en lugar de objetos. 

Corro los test, ahora tengo un gato triste(_rojo_) porque la función no existe. Así que pondré el código base de módulo:

```JavaScript
// photo-lister.js
var PhotoLister;

PhotoLister = {
    photoToListItem: function() {}
};

module.exports = PhotoLister;
```

Corro los test, y aún falla, así que puedo seguir escribiendo código. Y, la manera más simple de pasar el test es sólo retornar el string esperado. Así que haré lo siguiente:

```JavaScript
// photo-lister.js
PhotoLister = {
    photoToListItem: function() {
        return '<li><figure><img src="http://loremflickr.com/960/593" alt=""/>'
               + '<figcaption>This is a test</figcaption></figure></li>';
    }
};
```

Corro el test, y pasa. Gato feliz (_verde_). Es momento de refatorizar, pero retornar un string plano que no es tan complicado. Hay mucho que mejorar aquí aún. Pero, el código no es tan usable tampoco, así que escribo otro test.

```JavaScript
// photo-lister-spec.js
describe('#photoToListItem()', function() {
    it('should take a photo object and return a list item string', function() {
        var input = {
                title: 'This is a test',
                url:   'http://loremflickr.com/960/593'
            },
            expected = '<li><figure><img src="http://loremflickr.com/960/593" alt=""/>'
                     + '<figcaption>This is a test</figcaption></figure></li>';
        expect(PhotoLister.photoToListItem(input)).to.equal(expected);

        input = {
            title: 'This is another test',
            url:   'http://loremflickr.com/960/593/puppy'
        }
        expected = '<li><figure><img src="http://loremflickr.com/960/593/puppy" alt=""/>'
                 + '<figcaption>This is another test</figcaption></figure></li>';
        expect(PhotoLister.photoToListItem(input)).to.equal(expected);
    });
});
```

Corro los test de nuevo, y tenemos un gato triste (_rojo_). Ahora, esta bien escribir algo de código. En este caso, la manera más fácil de pasar el test es escribir código genérico:

```JavaScript
// photo-lister.js
PhotoLister = {
    photoToListItem: function(photo) {
        return '<li><figure><img src="' + photo.url + '" alt=""/>'
               + '<figcaption>' + photo.title + '</figcaption></figure></li>';
    }
};
```

Los test pasan ahora, es momento de refactorizar. No soy fan de todas esas operaciones de concatenación, así que lo reemplazaré con un array join: 

```JavaScript
// photo-lister.js
PhotoLister = {
    photoToListItem: function(photo) {
        return [
            '<li><figure><img src="',
            photo.url, '" alt=""/>',
            '<figcaption>',
            photo.title,
            '</figcaption></figure></li>'
        ].join('');
    }
};
```

Ahora que tengo una función que lidea con cada item individualmente, necesito lidear con las listas. Escribo otro test: 

```JavaScript
describe('#photoListToHTML()', function() {
    it('should take an array of photo objects and convert them to an HTML list', function() {
        var input = [{
                title: 'This is a test',
                url:   'http://loremflickr.com/960/593'
            }, {
                title: 'This is another test',
                url:   'http://loremflickr.com/960/593/puppy'
            }],
            expected = '<ul><li><figure><img src="http://loremflickr.com/960/593" alt=""/>'
                     + '<figcaption>This is a test</figcaption></figure></li>'
                     + '<li><figure><img src="http://loremflickr.com/960/593/puppy" alt=""/>'
                     + '<figcaption>This is another test</figcaption></figure></li></ul>';
        expect(PhotoLister.photoListToHTML(input)).to.equal(expected);
    });
});
```

Corro las pruebas y nuevo me da un error—gato triste—entonces, escribimos código: 

```JavaScript
photoListToHTML: function(photos) {
    return '<ul>' + photos.map(PhotoLister.photoToListItem).join('') + '</ul>';
}
```
Y corriendo los test de nuevo, el gato es feliz (_verde_), es momento del refactor. Otra vez, removeré todos los operadores de concatenación, porque simplemente de verdad no me gustan. 

```JavaScript
photoListToHTML: function(photos) {
    return ['<ul>', photos.map(PhotoLister.photoToListItem).join(''), '</ul>'].join('');
}
```
Así que ahora tiene un código que genera una lista completa en HTML como un string. Como puedes ver, a diferencia de código asíncrono o llamadas de red, probar la manipulación de srings es relativamente directo. Y como el HTML es texto plano, escribir pruebas para el código que genera los string de HTML es algo relativamente directo. En algún punto sin embargo, necesitamos que estre string se renderice en el navegador, así que tenemos que portarlo con el DOM. 

## Trabajando con el DOM

Ahora que tengo la lista terminada, sería genial que puediera revisar si se agrega a la página. Pero la trampa aquí es que, hasta ahora, he estado trabajando puramente en Node, sin browser. He hecho esto deliveradamente ya que: 

- Los tests corren mucho más rápido en línea de comandos. 
- Me anima a pensar sobre como puedo mantener mi código flexible; y
- Mocha me da diversión con el Nyan Cat reporter en la línea de comandos. 

Sin el navegador sin embargo, no puedo usar jQuery o métodos regulares de DOM para revisar que todo esta funcionando. Afortunadamente hay un módulo muy útil llamado [cheerio](https://cheeriojs.github.io/cheerio/) que emulará mucho del API de jQuery para nosotros. Esto significa que puedo probar mis funciones que manipulan el DOM sin cargar un browser sin cabeza o cambiar totalmente mi enfoque de pruebas. 

Para empezar, necesito instalar `cheerio`, corriendo npm:
```
npm install cheerio --save-dev
```

Ahora que tengo cheerio instalado, podemos crear un falso jQuery con un falso DOM. 

```JavaScript 
// photo-lister-spec.js
var cheerio = require('cheerio');

// … snip …

describe('#addPhotosToElement()', function() {
    it('should take an HTML string of list items and add them to an element with a given selector', function() {
        var $        = cheerio.load('<html><head></head><body><div id="mydiv"></div></body></html>'),
            list     = '<ul><li><figure><img src="http://loremflickr.com/960/593" alt=""/>'
                     + '<figcaption>This is a test</figcaption></figure></li>'
                     + '<li><figure><img src="http://loremflickr.com/960/593/puppy" alt=""/>'
                     + '<figcaption>This is another test</figcaption></figure></li></ul>',
            selector = '#mydiv',
            $div     = PhotoLister.addPhotosToElement($, selector, list);
        expect($div.find('ul').length).to.equal(1);
        expect($div.find('li').length).to.equal(2);
        expect($div.find('figure').length).to.equal(2);
        expect($div.find('img').length).to.equal(2);
        expect($div.find('figcaption').length).to.equal(2);
    });
});
```
Aquí he creado un falso DOM con sólo un `<div>`en el body del documento, y lo he envuelto con cheerio. Paso mi función como si fuera jQuery, y entonces espero que `addPhotosToElement()`retorne un objeto como el de jQuery. 
Corro el test para revisar cada elemento y espero que exista. Esto me da un test con fallos. Ahora que tengo una prueba con fallos, puedo escribir código: 

```JavaScript 
addPhotosToElement: function($, selector, list) {
    return $(selector).append(list);
}
```

Como paso `$` como parámetro, puedo acceder al falso DOM como si fuera jQuery operando en un navegador. Y con este código todos los tests pasan. El gato esta feliz y es momento de refactorizar-pero yo no creo que pueda hacer esto más simple de lo que ya es. 

Así que por ahora, los módulos están hechos. Hay algunos toques finales que necesito hacer para operar esto en el navegador. 

## Juntar todo en una página web
Hasta ahora, hemos (a propósito) hecho todo en Node, y no en el browser. Esto es bueno, pero el punto de este módulo es desplegar fotos en un navegador, no sólo hacerlos pasar tests. Así que necesito algunos ajustes para hacer que este código corra en ambos ambientes. 

Esta es una form de refactor. Cada vez que haga un cambio, volveré a correr mis pruebas para estar seguro que aún pasan. 

La primera cosa que haré será poner una condicional que envuelva el `module.exports` para que el browser no de un error si incluyo el código en la página web. Podría, claro, usar algo como Browserify o Webpack para empaquetar esto (y si puede, te lo recomiendo hacerlo), pero es bonito hacer el trabajo de otra manera. Si sólo pondré el código directamente en algo como CodePen, por ejemplo, preferiría no hacer toda a una configuración de Webpack: 

```JavaScript 
// flickr-fetcher.js
if ((typeof module !== 'undefined') && (typeof module.exports !== 'undefined')) {
    module.exports = FlickrFetcher;
}
```

```JavaScript 
// photo-lister.js 
if ((typeof module !== 'undefined') && (typeof module.exports !== 'undefined')) {
    module.exports = PhotoLister;
}
````

Corro mis tests una vez más, uso el siguiente código:

```
$ mocha --reporter=nyan ./*-spec.js
```

... y aún tenemos un gato feliz. 

Algo final que me gustaría hacer es proveer de una interfaz que quite la necesidad para pasar en `jQuery.getJSON`si jQuery esta presente como una variable global. Haré esto, voy a hacer uso de el método `bind()`encontrado en la mayoría de implementaciones de JavaScript. 

```JavaScript
//flickr-fetcher.js
fetchFlickrData: function(apiKey, fetch) {
    if ((!fetch) && (typeof jQuery !== 'undefined')) {
        fetch = jQuery.getJSON.bind(jQuery);
    }
    var url = 'https://api.flickr.com/services/rest/?method=flickr.photos.search&api_key='
            + apiKey.toString() + '&text=pugs&format=json&nojsoncallback=1'
    return fetch(url);
}
```

Ahora puedo usar estas funciones en el navegador sin tener que depender de un packaging systems y no hacer el problemático pase de jQuery en la función `fetchPhotos()`. Esto me da más flexibilidad y hace el API más accesible. 

Y con esto, la aplicación esta casi terminada. Todo lo que queda es una pieza para juntar los dos módulos. Para verlo en acción te recomiendo [ver la demostración en CodePen](http://codepen.io/jrsinclair/pen/EKQmwo), pero el código relevante esta resumido abajo: 

```JavaScript 
FlickrFetcher.fetchPhotos('8060d4cdac3ceb86af470aae29af3a56')
    .then(PhotoLister.photoListToHTML)
    .then(function(photosHTML) {
        PhotoLister.addPhotosToElement($, '#mydiv', photosHTML);
    });
````

Así, tras el paso por estos tres artículos hemos cubierto mi enfoque general de JavaScript TDD; incluyendo pruebas asíncronas, stubs en llamadas de red, y trabajar con HTML y el DOM. En este artículo vimos en particular el trabajo con HTML y uso del paquete cherrio en lugar de jQuery para hacer funcionar los tests sin el navegador. Hay, por supuesto, mucho más de TDD, y esta serie apenas a cubierto la superficie, pero sinceramente espero que sea de ayuda. 


<a name="footnote-1"></a>
1. Esto es porque por defecto(por alguna razón), cuando hacer `require` a un archivo vación, obtienes un objeto vacío.[↩](#reference-1)