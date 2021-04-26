---
layout: post
title: Diseño de plantilla web fullscreen con CSS
description: Alternativas para diseñar una plantilla web fullscreen con CSS
date:  2021-03-01
time: 8 mins
tags: css html
language: spanish
---

Cuando nos disponemos a maquetar una web, siempre aparece en nuestra mente la sencilla distribución de cabecera, menú, contenido y pie.

Aunque este diseño está más anticuado que los almanaques de ferreterías, sí que plantea algunos desafíos interesantes cuando queremos mantener el aspecto fullscreen.

<!-- more -->

## Nuestra plantilla

Vamos a trabajar con la siguiente plantilla HTML:
```html
<!DOCTYPE html>
<html>
<body>
	<header>Header</header>
	<section id="main">
		<nav>Menu</nav>
		<section id="content">Content</section>
	</section>
	<footer>Footer</footer>
</body>
</html>
```

Delegando toda la distribución del contenido en la hoja de estilos (versión simplificada):

```css
* { overflow: auto; color: white }

html, body { height: 100%; 	margin: 0 }

header { background-color: rosybrown }
nav { background-color: lightslategray }
section#content { background-color: black }
footer { background-color: burlywood }
```

[TL;DR](#resultado-final)

## Hmm... tan difícil no será, ¿no?

Este tipo de requisitos suele ser divertido. Nos vendrán montones de ideas a la cabeza, algunas de ellas incluyendo entresijos del mundo CSS que de partida no nos gustan, y sabemos que nos van a dar problemas antes o después. Por si fuera poco, la vocecilla del programador x10 que llevamos dentro nos dirá que deberíamos poder resolverlo con nuestros conocimientos.

### Porcentajes

La configuración más trivial que se nos puede ocurrir es hacer uso de porcentajes para distribuir el espacio, algo como:

```css
header { height: 15% }
section#main { height: 75% }
nav { height: 100%; width: 100px; float: left }
section#content { height: 100% }
footer { height: 10% }
```

[Ver resultado](https://jsfiddle.net/juliomaqueda/u4p9kd6f/)

Sin embargo, esta solución hace aguas cuando queremos que la cabecera o el pie tengan un tamaño fijo, ya que perdemos el control sobre el tamaño relativo del resto de elementos.

A ver... cabecera y pie con tamaño fijo y contenido central que se ajuste al resto del espacio disponible.

### Una tabla colosal

Montar una tabla gigantesca podría permitir la distribución que buscamos. Una fila con altura fija para la cabecera, una segunda fila sin altura conteniendo dos columnas (menú y contenido), y por último una fila con altura fija para el pie.

[Ver resultado](https://jsfiddle.net/juliomaqueda/udk7h026/)

Como podrás imaginar, uno de los principales problemas de esta solución (a parte de usar una tabla para algo que no es una tabla) es que nos obliga a cambiar por completo la estructura HTML de la web, cosa que no queremos.

### Fijando elementos

Se nos puede ocurrir también fijar la cabecera en la parte superior, y el pie en la inferior (`position: fixed`). De esta forma, podemos decirle al contenido central que ocupe el 100% de la altura. El resultado será que tanto la cabecera como el pie estarán por encima del contenido, lo cual es chapucero, pero podría dar el pego si la parte central incluye una separación por arriba y por abajo, de igual valor que la altura de la cabecera y el pie, respectivamente.

Esta solución, además de fea, puede ser desastrosa a la hora de gestionar el scroll del contenido central.

...

Venga va, ahora la solución buena.

## Módulo Flexbox de CSS3

El módulo de caja flexible ([flexbox](https://developer.mozilla.org/es/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox)) permite hacer este tipo de distribuciones de manera natural. Ideado precisamente para soportar diseños responsive, permite estructurar contenido sin necesidad de hacer uso de elementos flotantes o posicionamientos extraños.

### Creando nuestro contenedor flexible

Tener un contenedor flexible es el primer paso para poder distribuir el contenido bajo él.

`display: flex` - Activa un contexto flexible para los hijos directos de nuestro contenedor.

`flex-direction: column` - Establece que la distribución de hijos se haga a lo largo del eje vertical ([más info](https://developer.mozilla.org/es/docs/Web/CSS/flex-direction))

De esta forma, el estilo de nuestro contenedor quedaría así:

```css
body {
	display: flex;
	flex-direction: column
}
```

### Distribución de los hijos del contenedor flexible

El estilo para la cabecera y el pie no tienen mayor misterio, ya que únicamente requieren la altura que necesitemos:

```css
header { height: 100px }
footer { height: 60px }
```

Nos queda definir el estilo para el elemento central, el cual queremos que se adapte al espacio restante de nuestro contenedor flexible (que tiene una altura de 100%). El módulo flexbox viene como anillo al dedo para esta distribución, ya que cuenta con el atributo `flex`, que indica la capacidad de un elemento flexible para alterar sus dimensiones y llenar el espacio disponible ([más info](https://developer.mozilla.org/es/docs/Web/CSS/flex)). Dándole el valor `1` nos aseguraremos de que el elemento se estira verticalmente para rellenar todo el espacio disponible.

```css
section#main { flex: 1 }
```

### Distribución del contenido principal

Qué maravilla! hemos conseguido que los tres elementos principales se distribuyan como queremos con apenas tres líneas de CSS.

Nos encontramos ahora con la última parte de la maquetación, distribuir el menú y el contenido dentro de la sección central. Este requisito es algo que ahora ya nos suena, porque plantea el mismo dilema que la distribución de los hijos del contenedor principal. Así pues, nos encontramos con que el elemento `main`, además de ser hijo de un contenedor flexible, debe ser a su vez un contenedor flexible, permitiendo la distribución de los elementos bajo él.

```css
section#main {
	flex: 1;
	display: flex
}
```

En este caso no necesitamos incluir `flex-direction: row` ya que `row` es el valor por defecto de esta propiedad.

Por último, sólo nos queda establecer el tamaño de los elementos restantes:

```css
nav { width: 200px }
section#content { flex: 1 }
```

## Resultado final

Uniendo los estilos anteriores, llegamos al resultado final esperado:

```css
* {
	overflow: auto;
	color: white
}

html, body {
	height: 100%;
	margin: 0
}

body {
	display: flex;
	flex-direction: column
}

header { background-color: rosybrown }
nav { background-color: lightslategray }
section#content { background-color: black }
footer { background-color: burlywood }

header { height: 100px }

section#main {
	flex: 1;
	display: flex
}

nav { width: 200px }

section#content { flex: 1 }

footer { height: 60px }
```

[Ver resultado](https://jsfiddle.net/juliomaqueda/2Letvk78/)
