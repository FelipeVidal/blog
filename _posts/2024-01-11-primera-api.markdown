---
layout: post
title:  "Mis primeros servicios con SpringBoot"
categories: SprigBoot
background: '/_img/CrarProyectoSB.gif'
---
Objetivo: Crear una API que me permita: crear, listar, editar y eliminar Pokemons.

Solución:
1. Creamos un proyecto Maven de SpringBoot utilizando [Spring Initializr](https://start.spring.io/) y agragamos la depentendia Spring Web, donde cada campo significa:
- **Grupo**: Identificación única que nos ayuda a a organizar y agrupar proyectos relacionados bajo un nombre común.
- **Artefacto**: Nombre especifico de la aplicación que se reflejará una vez compilemos el proyecto y obtengamos un archivo JAR o WAR.
- **Nombre**: Suele ser el mismo valor que ingresemos en el campo "Artefacto" y nos sirve para nombrar el proyecto de una manera descriptiva y legible. 
- **Descripción**: En este campo describimos el objetivo de nuestro proyecto.
- **Nombre del paquete**: Nombre del paquete principal de la aplicación.
<p>&#129488; Importante: escoger la dependicia Spring Web, dado que ella nos permitirá construir la API.</p>
<figure style="width:740px; height:740px; text-align: -webkit-center">
    <img src="/blog/_img/createproject.gif" alt="" />
</figure>
