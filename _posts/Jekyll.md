---
title: 'Jekyll: Lo bueno, lo malo y lo feo'
date: '2024-11-11 17:02 -0300'
permalink: /posts/Jekyll
lang: es
categories:
  - Neovim
  - Development
tags:
  - Tools
  - Linux
  - Software
description: Primer capítulo de las aventuras en Neovim.
media_subpath: /assets/img/jekyll/
---
# Jekyll: Lo bueno, lo malo y lo feo

Hace algunas semanas decidí embarcarme en un pequeño experimento de aprendizaje,
trabajar 3 semanas con diferentes frameworks o herramientas para generar sitios
estáticos para escribir mi blog personal. Decidí comenzar con Jekyll dado que
github pages lo promociona.

## Promesas

Jekyll prometía una integración fácil y rápida con Github pages, quizás así sea
para gente ya acostumbrada al desarrollo web, en mi caso no fue así.

Me llamaba la atención ruby, pero la verdad no he tocado nada al respecto. He
operado básicamente en altísimo nivel, tratando de sortear todos los "problemas"
que me generó.

## Ruby

Uff. Qué decir... Odié la paquetería y el sistema de gemas a la media hora de
trabajar con él. A la primera prueba sencilla ya tenía mi home completamente
contaminada con directorios basura, caché y data entremesclados. **Otro proyecto
_profesional_ que no respeta estándares XDG**... ¿Será tan difícil o es solo
ignorancia y negligencia? En fin, mal comienzo. En el momento en que tuve
conflictos de versiones y para solucionarlo no se completó una instrucción
porque iba a escribir archivos directamente a mi sistema. Por supuesto se
interrumpió al no tener permisos de root, y eso fue suficiente para mí. A
desinstalar y limpiar todo el chiquero de archivos (incluso algunos con permisos
mal seteados). Espero haber borrado toda esa basura completamente... Bueno, en
cualquier caso Docker al rescate, y pude hacerlo funcionar definiendo una imagen
en base a ubuntu. Uff.

## Lo bueno

Liquid. Me encantó liquid. Para usar lógica utilizar
{% raw %}`{% logic %}`{% endraw %} y para mostrar contenido usar
`{{ variable }}`. Sencillo, elegante, excelente.

## Lo malo

Después de seleccionar el tema [Chirpy](https://chirpy.cotes.page/), la mayoría
de las personalizaciones las he sentido como un gran hack. Sobreescribir css en
lugar de redefinirlos, copiar archivos fuente completos para modificar un punto.
Aplicar parches con rutas hardcodeadas directamente en la pipeline para poder
tener contenido en multilenguaje y que este se aplique en todos los elementos
del html.

En fin, quizás sea mi falta de experiencia y Jekyll ofresca una forma mucho más
orgánica de personalización que simplemente pasé por alto en una búsqueda
pragmática y exploratoria para producir el estilo web que deseaba. En cualquier
caso la experiencia no fue para nada agradable.

## Lo feo

Mi impresión general con Jekyll es que parece tener dado vuelta el foco de uso.
Por ejemplo, por alguna desición que va más allá de mi entendimiento, se decidió
por anteponer a los nombres de directorios que el usuario utiliza con `_`.
¿Pongamos los posts en un directorio `posts`? No, demasiado sencillo, demasiado
ergonómico, mejor usemos `_post`. ¡¿Por qué?! Quizás sea una convención, pero en
ese caso me parecería una pésima convención. No, es que es para diferenciar el
el sitio generado de los archivos de generación... Bueno, si el usuario (quien
genera el contenido) está utilizando todo el rato ese directorio, ¿entonces por
que no invertirlo? ¿Por qué no hacerle la vida más fácil al usuario?

Para mí, esto que parece una nimidad, es reflejo de lo mucho que le falta a
Jekyll para sentirse una solución más robusta.

## Conclusiones

El resultado estético final, me gustó bastante, pero que con tod

