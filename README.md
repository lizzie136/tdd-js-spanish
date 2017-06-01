# Un introducción gentil a Test Driven Development en JavaScript: PARTE I


_Este artículo originalmente fue publicado en Inglés por James Sinclair el 11 de Abril del 2016_

Esta es la parte uno de una serie de tres partes donde explico en terminos generales mi acercamiento personal a Desarrollo Impulsado o dirigido por Pruebas (_Test driven development_ o TDD). Sobre el curso de esta serie, trabajaré el desarrollo de una aplicación completa (aunque una pequeña, simple) en JavaScript que envuelve hacer peticiones de red (también conocidas como AJAX) y manipulación de DOM. Las varias partes son las que siguen: 

* Parte 1: Comenzando con pruebas unitarias. 
* Parte 2: Trabajando con peticiones de red en TDD.
* Parte 3: Trabajando con DOM en TDD.

## ¿Por qué Test Driven Development?
Comenzando con desarrollo dirigido por pruebas (TDD) puede ser intimidante. Suena tedioso, aburrido y complicado. La palabra 'prueba' conjura asocioaciones con exámenes y estrés y vigilantes y todo tipo de molestias. Y puede parecer un desperdicio escribir código que no _hace_ nada útil mas que decirte que el código que ya escribiste esta funcionando. Y encima de todo, hay un confusa colleción de frameworks y librerías ahi fuera. Algunas trabajan en el servidor, otras trabajan en el navegador, otras hacen las dos cosas... puede ser duro saber por donde empezar.

___

> Las objeciones predecibles son "Escribir pruebas unitarias toma mucho tiempo, " o "¿Cómo escribo los test primero si no sé como que es lo que hace aún?" y entonces esta la excusa popular: "Las pruebas unitarias no atraparán todo los bugs". [[1]](https://www.stickyminds.com/article/test-driven-development-isnt-testing)

Existen, sin embargo, muchas buenas razones para intentar TDD. Aquí tienes tres de las que creo importantes: 

1. **Te fuerza a pensar**. Este es mas útil de lo que suena. Escribir una prueba me fuera a pensar claramente sobre qué es lo que estoy tratando de alcanzar, al punto de tener un nivel de detalle que la computadora pueda revisar. Una vez que tengo todo claro en mi cabeza, se vuelve mucho más sencillo escribir el código. Si estoy sufriendo por escribir una prueba, sé que no he entendido completamente el problema que estoy tratando de resolver. 

2. **Hace el debugging más secillo**. Mientras TDD no causará que escribas menos bugs(tristemente), si hace un más sencillo hacerles seguimiento cuando aparezcan inevitablemente. Y si luego escribes un test relacionado al bug, me da más confanza de saber que definitivamente he resuelto un bug en particular. Puedo re



