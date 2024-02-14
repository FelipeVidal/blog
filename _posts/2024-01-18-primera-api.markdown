---
layout: post
title:  "Construyendo servicios básicos en SpringBoot"
categories: SprigBoot
---
Objetivo: Crear una API que me permita: crear, listar, editar y eliminar Pokemons.

Solución:
1. Creamos un proyecto Maven de SpringBoot utilizando [Spring Initializr](https://start.spring.io/) y agragamos la depentendia Spring Web, donde cada campo significa:
- **Grupo**: Identificación única que nos ayuda a organizar y agrupar proyectos relacionados bajo un nombre común.
- **Artefacto**: Nombre especifico de la aplicación que se reflejará una vez compilemos el proyecto y obtengamos un archivo JAR o WAR.
- **Nombre**: Suele ser el mismo valor que ingresemos en el campo "Artefacto" y nos sirve para nombrar el proyecto de una manera descriptiva y legible. 
- **Descripción**: En este campo describimos el objetivo de nuestro proyecto.
- **Nombre del paquete**: Nombre del paquete principal de la aplicación.
<img src="/blog/_img/createproject.gif" alt="" />
<br>
&#129488; **Importante**: escoger la dependencia "Spring Web", dado que ella nos permitirá construir la Api.

2. Creamos el paquete "model" el cual contiene el modelo de la entidad que queremos representar, en este caso queremos una entidad que incluya el identidicador(Integer), nombre(String), habilidades(Array) y el estado(Boolean).
<br>
&#129488; **Importante**: Se hace uso de la interfaz [List](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) en el atributo "abilieties", dado que ello nos garantiza flexibilidad al momento de utilizar cualquiera de las implementaciones de la interfaz.
<img src="/blog/_img/createModel.gif" alt="" />

3. Una vez creado el modelo, procedemos a crear el controlador. Para ello tener en cuenta la [anotación](https://medium.com/@alvareztech/java-annotations-b8ae69739b4e) @RestController que nos permitirá instanciar la clase como un servicio web RestFull en Spring.
- En el controlador, procedemos a crear alguno datos de pruebas. La anotación @PostConstruct nos permite que se ejecute la función justo después del constructor y contar con los datos para los servicios.
- Procedemos con la creación de los métodos Http:
  - Método Get con la anotación [@GetMapping](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/GetMapping.html), junto con los parametros "value" y "produces". En value ponemos el nombre que nos va a permitir consumir el servicio, en este caso "/pokemon", y el formato de la respuesta a entregar con "produces", en este caso "MediaType.APPLICATION_JSON_VALUE".
  - Método Post con la anotación [@PostMapping](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/PostMapping.html), junto al nombre que nos va apermitir cosumir el servicio, en este caso “/pokemon”, y el formato de los datos que vamos a recibir en “consumes”, en este caso " MediaType.APPLICATION_JSON_VALUE".
  - Método Put con la anotación [@PutMapping](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/PutMapping.html) junto al nombre que nos va a permitir cosumir el servicio, en este caso “/pokemon”, y el dato con el que vamos a identificar los valores que vamos a modificar. Por último, el formato de los datos que vamos a recibir con “consumes”, en este caso "MediaType.APPLICATION_JSON_VALUE".
  - Método Delete con la anotación [@DeleteMapping](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/DeleteMapping.html) junto al nombre que nos va apermitir consumir el servicio, en este caso “/pokemon”, y el dato con el que vamos a identificar el objeto que vamos a eliminar.
4. Dado que tenemos anotaciones en el paquete "controller" que Spring debe escanear, debemos agregar la anotación @ComponentScan(basePackages = {"com.pokemon.controller"}) en la clase principal.

5. Realizamos pruebas en Postman validando que los endpoint funcionen de la menera esperada. Por ejemplo, al utilzar el método "Get" con la dirección "http://localhost:8080/pokemon" debería retornarnos un Json con todos los valores que hemos guardado de cierta entidad, en este caso los pokemons que hemos creado.
<img src="/blog/_img/testApi.gif" alt="" />


## Clase Principal

{% highlight java %}
package com.pokemon.pokemonapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@ComponentScan(basePackages = {"com.pokemon.controller"})
@SpringBootApplication
public class PokemonApiApplication {

	public static void main(String[] args) {
		SpringApplication.run(PokemonApiApplication.class, args);
	}

}
{% endhighlight %}


## Modelo
{% highlight java %}
package model;

import java.util.List;

public class Pokemon {
    

    private int idPokemon;

    private String name;

    private List<String> abilities;
    
    private boolean is_default;

    public Pokemon(int idPokemon, String name, List<String> abilities, boolean is_default){
        this.idPokemon = idPokemon;
        this.name = name;
        this.abilities = abilities;
        this.is_default = is_default;
    }

    public Pokemon(){}

    public Integer getId(){
        return idPokemon;
    }

    public void setId(int idPokemon){
        this.idPokemon = idPokemon;
    }

    public String getName(){
        return name;
    }

    public void setName(String name){
        this.name = name;
    }

    public List<String> getAbilities(){
        return this.abilities;
    }

    public void setAbilities(List<String> abilities){
        this.abilities = abilities;
    }

    public boolean getIs_Default(){
        return is_default;
    }

    public void setIs_Default(boolean is_default){
        this.is_default = is_default;
    }   
}
{% endhighlight %}

## Controlador

{% highlight java %}
package com.pokemon.controller;

import java.util.List;
import java.util.ArrayList;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import com.pokemon.model.Pokemon;

import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.PathVariable;

import jakarta.annotation.PostConstruct;

@RestController
public class controller {


    private List<Pokemon> pokemons;
    
    private List<String> abilities;
    
    @PostConstruct
    public void init(){
        pokemons = new ArrayList<>();
        abilities = new ArrayList<>();
        abilities.add("Golpe de cola");
        abilities.add("Onda trueno");
        pokemons.add(new Pokemon(1,"Pikachu",abilities,true));
    }

    @GetMapping(value = "pokemon",produces = MediaType.APPLICATION_JSON_VALUE)
    public List<Pokemon> pokemon(){
        return pokemons;
    }
    @PostMapping(value = "pokemon", consumes = MediaType.APPLICATION_JSON_VALUE)
    public List<Pokemon> crearPokemon(@RequestBody Pokemon pokemon) {
        pokemons.add(pokemon);
        return pokemons;
    }

    @PutMapping(value="pokemon/{name}", consumes = MediaType.APPLICATION_JSON_VALUE)
    public List<Pokemon> actulizarPokemon(@PathVariable String name,@RequestBody Pokemon pokemon) {
       for(int i = 0; i < pokemons.size(); i++){
            if(pokemons.get(i).getName().equals(name)){
                pokemons.set(i,pokemon);
            }
       }
       return pokemons;
    }

    @DeleteMapping(value="pokemon/{name}")
    public List<Pokemon> borrarPokemon(@PathVariable String name){
        for(int i = 0; i < pokemons.size(); i++){
            if(pokemons.get(i).getName().equals(name)){
                pokemons.remove(i);
            }
       }
       return pokemons;
    }

}
{% endhighlight %}

<br>
Fuentes:
- “Microservicios con Spring Boot, Cloud y Docker,” Udemy, 2024. https://www.udemy.com/course/microservicios-spring-boot-cloud/
‌