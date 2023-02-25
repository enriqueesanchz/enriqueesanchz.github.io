---
title: Cómo conseguí 400 usuarios para esta Web de F1
date: 2022-02-22 17:30:00 +0100
categories: [Web, Twitter]
tags: [web, f1, twitter, viral]     # TAG names should always be lowercase
mermaid: true
---

Esta es la historia de cómo llegamos a tener una web de Fórmula 1 con más de 400 usuarios hecha en tan solo 10 días.

## El comienzo

<div style="text-align: justify; text-justify: inter-word">
Durante el auge del fenómeno EL PLAN en Twitter, allá por Febrero de 2022, @manueljerez7_ y yo tuvimos la idea de crear una web a modo de porra para un conjunto reducido de amigos.
Aprovechamos que él tenía un grupo con otros usuarios de Twitter F1 que se reunían usando la herramienta Twitter Spaces para comentarles la idea. Les encantó, por lo que decidimos poner un Tweet para ver si a más gente le gustaba. Se viralizó.

<blockquote class="twitter-tweet"><p lang="es" dir="ltr">CHAVALES este año se viene a TwF1 un super UPGRADE:<br>UNA WEB SENCILLA para hacer LA PORRA DE LA CARRERA<br>Te registras con tu usuario de Twitter, metes resultados y ves la clasificación general de puntos<br><br>Pronto os diré más 👀👀👀<br>Se agradece difusión para llegar a más gente 🙏</p>&mdash; Manu 😉 (@manueljerez7_) <a href="https://twitter.com/manueljerez7_/status/1492115818951417859?ref_src=twsrc%5Etfw">February 11, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

700 Likes, 130 RT, 90.000 Visualizaciones... Fue una idea muy bien recibida por la comunidad, así que nos pusimos manos a la obra ya que quedaban a penas 2 semanas para el comienzo de temporada y tenía que estar todo listo, funcionando y probado para el primer Gran Premio.

</div>

## Cómo nos organizamos

Ambos estábamos cursando el grado de Ingeniería Telecomunicaciones en la Universidad de Sevilla, por lo que teníamos que organizar el poco tiempo sobrante que teníamos para llevar a cabo este proyecto.

```mermaid
 gantt
  dateFormat  YYYY-MM-DD
  axisFormat %m-%d
  title  Organización del proyecto La Porra de Twitter F1
  
  section Reglas
  Lluvia de ideas :a, 2022-02-09, 2d
  Formalización :b, after a, 2d
  
  section Investigación
  BBDD :c, 2022-02-09, 2d
  Despliegue :e, 2022-02-10, 2d
  
  section Implementación
  Diseño :f, 2022-02-09, 4d
  Inicio de sesión :d, after f, 2d
  Juego :g, after f, 3d
  
  section Optimización
  BBDD: h, 2022-02-16, 3d
  Servidor: i, 2022-02-16, 3d
```

## Ideas clave
De la fase de lluvia de ideas sacamos varias claves:

#### - Inicio de sesión usando API Twitter

Es muy importante eliminar las barreras de entrada que pueda tener un usuario. Puesto que nuestra publicidad era Twitter, el registro en nuestra web se hará mediante su API. El usuario únicamente tendrá que dar acceso a su nombre de cuenta en Twitter y su @ será su usuario en la web.
Desde mi punto de vista, esto fue un completo acierto ya que el pulsar un botón de inicio de sesión con Twitter hace que el registro se vuelva automático y nada pesado.

#### - Botón compartir en Twitter estilo Wordle

Wordle se había viralizado ya que la gente podía hacer el reto diario y compartir su famoso código de colores en Twitter, por lo que nosotros inventamos el nuestro propio.
Esto nos ayuda a llegar a un mayor público.

#### - Reglas

Las reglas se debatieron durante los primeros días para su posterior formalización. Básicamente consiste en colocar a los pilotos que creas que van a quedar los 10 primeros y según los resultados se te asigna una puntuación dependiendo de lo cerca que hayas estado de acertar.

## Investigación
Esta era una parte importante para decidir qué servicios íbamos a usar tanto para la Base de Datos como para dónde desplegar la Web, siempre intentando minimizar costes, ya que éramos estudiantes sin fuente de ingresos.

#### - BBDD

Para la Base de Datos teníamos las posibilidades de Google Cloud SQL, Firebase, Azure SQL, AWS SQL Server, etc. Terminamos decantándonos por Azure ya que por $5 teníamos 2Gb de almacenamiento, costeándolo durante la temporada de Fórmula 1 con el plan para estudiantes.

#### - Servidor

Otra parte vital es dónde desplegar la Web. Entre las diferentes opciones estaban de nuevo las plataformas de Google, Azure, AWS, etc. Al final elegimos Heroku con su plan gratuito ya que teníamos experiencia previa. Cabe recalcar que teníamos algo de miedo por la viralización del Tweet y porque el número de usuarios que podríamos tener quizás hacía que la web cayese.

## Implementación

