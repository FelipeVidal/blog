---
layout: post
title: Conectando la API Rest a una BD
categories: SpringBoot
---
**Objetivo**: Conectar una Api Rest a una base de datos MySQL

**Solución**:
1. Creamos un proyecto utilizando [Spring Initializr](https://start.spring.io/) y seleccionamos las dependencias Spring Web, Spring Data JPA y MySQL Driver.
<img src="/blog/_img/createProjectDB.gif" alt="" />

2. Creamos la Base de datos. En MySQL ingresaríamos a MySQL y agregamos la instrucción "create database db_pokemon".
<img src="/blog/_img/createMySQLDB.png" alt="" />

3. Configuramos el archivo "application.propierties" con la información de la BD que se creó en el punto anterior.
```
spring.jpa.hibernate.ddl-auto = update
spring.datasource.url = jdbc:mysql://${MYSQL_HOST:localhost}:3306/db_pokemon
spring.datasource.username = root
spring.datasource.password = admin
spring.datasource.driver-class-name = com.mysql.cj.jdbc.Driver
```
&#129488; **Importante**: En el caso de Linux, validar que la versión de la variable de entorno JAVA_HOME y el proyecto sea la misma, en caso de que no podría fallar al momento de realizar alguna actualización de paquetes de Maven utilizando mvn clean-install.

4. Creamos el paquete “model” el cual contiene el modelo de la entidad que queremos representar, en este caso queremos una entidad que incluya el identidicador(Integer), nombre(String), habilidades(Array) y el estado(Boolean).
<br>
**Pokemon.java**
```
package com.pokemon.model;
import java.util.List;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
@Entity //Aquí le decimos a Hibernate que cree una tabla a partir de esta clase.
public class Pokemon {
    @Id //Indicamos el identificador de la tabla.
    @GeneratedValue(strategy = GenerationType.IDENTITY) //Indicamos que el campo sea. autoincremental (https://www.arquitecturajava.com/jpa-generatedvalue-y-primary-keys/)
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
```

5. Creamos el paquete ["dao"](https://reactiveprogramming.io/blog/es/patrones-arquitectonicos/dao) para el acceso a datos y agregamos la interfaz "PokemonDao.java" con su respectiva clase de implementación "PokemonDaoImpl.java". Adicional, implementamos la interfaz "PokemonRepository" que extiende de JpaRepository<T,ID> que nos va a proporcionar algunos métodos para manipular los datos de la base de datos.
<br>
**PokemonDao.java**
```
package com.pokemon.dao;
import java.util.List;
import com.pokemon.model.Pokemon;
public interface PokemonDao {
    List<Pokemon> allPokemons();
    void addPokemon(Pokemon pokemon);
    Pokemon recoverPokemon(String name);
    void updatePokemon(Pokemon pokemon);
    void deletePokemon(String name);
}
```
**PokemonJpaSpring.java**
```
package com.pokemon.dao;
import org.springframework.data.jpa.repository.JpaRepository;
import com.pokemon.model.Pokemon;
public interface PokemonJpaSpring extends JpaRepository<Pokemon, Integer>{
}
```
**PokemonDaoImpl.java**
```
package com.pokemon.dao;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import com.pokemon.model.Pokemon;
@Repository
public class PokemonDaoImpl implements PokemonDao {
    @Autowired
    PokemonJpaSpring pokemon;
    @Override
    public List<Pokemon> allPokemons(){
        return pokemon.findAll();
    }
    @Override
    public void addPokemon(Pokemon pokemon){
        this.pokemon.save(pokemon);
    }
    @Override
    public Pokemon recoverPokemon(int id){
        return pokemon.findById(id).orElse(null);
    }
    @Override
    public void updatePokemon(Pokemon pokemon){
        this.pokemon.save(pokemon);
    }
    @Override
    public void deletePokemon(int id){
        pokemon.deleteById(id);
    }
}
```
&#129488; **Importante**: La utilización de anotaciones como @Repository en Spring conlleva a que la instancia de la clase sea creada automáticamente por el framework. Es importante destacar que la elección de las anotaciones está directamente vinculada a la capa en la que nos encontramos. Por ejemplo, en la capa de acceso a datos, se designa la anotación @Repository, en la capa de servicio se utiliza @Service, y para la capa de controladores, la anotación correspondiente es @RestController. Este enfoque permite a Spring gestionar la creación y la gestión de estas clases de manera eficiente, manteniendo así la coherencia y la organización en la estructura de la aplicación.

6. Crear el paquete "service" el cual contendrá las reglas del negocio y donde se crearán los servicios que el controlador expondrá a los clientes. Dentro del nuevo paquete, creamos la interfaz "PokemonService.java" y la clase "PokemonSeviceImpl.java".
<br>
**PokemonService.java**
```
package com.pokemon.service;
import java.util.List;
import com.pokemon.model.Pokemon;
public interface PokemonService {
    List<Pokemon> getAllPokemons();
    List<Pokemon> addPokemon(Pokemon pokemon);
}
```
**PokemonServiceImpl.java**
```
package com.pokemon.service;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import com.pokemon.dao.PokemonDaoImpl;
import com.pokemon.model.Pokemon;
@Service
public class PokemonServiceImpl implements PokemonService{
    @Autowired
    private PokemonDaoImpl pokemonDao;
    @Override
    public List<Pokemon> getAllPokemons() {
        return pokemonDao.allPokemons();
    }
    @Override
    public List<Pokemon>  addPokemon(Pokemon pokemon) {
        pokemonDao.addPokemon(pokemon);
        return pokemonDao.allPokemons();
    }
}
```
&#129488; **Importante**: Incluir la anotación [@Service](https://www.arquitecturajava.com/spring-service-usando-el-patron-servicio/), la cual nos permitirá construir una clase de Servicio y manejar los repositorios del paquete "dao".

7. Creamos el paquete "controller", el cual contendrá las clases que expondrán los servicios a los clientes. Dentro del paquete "controller" creamos la clase "Controller.java".
<br>
**Controller.java**
```
package com.pokemon.controller;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import com.pokemon.service.PokemonService;
@RestController
public class Controller {
    @Autowired
    private PokemonService pokemonService;
    @GetMapping(value="/pokemon", produces = MediaType.APPLICATION_JSON_VALUE)
    public List<Pokemon> getAllPokemons() {
        return pokemonService.getAllPokemons();
    }
    @PostMapping(value="/pokemon", consumes = MediaType.APPLICATION_JSON_VALUE, produces = MediaType.APPLICATION_JSON_VALUE)
    public List<Pokemon> postMethodName(@RequestBody Pokemon pokemon) {
        return pokemonService.addPokemon(pokemon);
    }
}   
```

8. Configuramos la clase principal donde le indicamos a Spring la entidad que utilizaremos con la anotación **@EntityScan**. También, le indicamos a Spring con la anotación **@ComponentScan** los paquetes donde tenemos clases con anotaciones @RestController, @Service y @Repository que Spring debe instanciar. Por último, mediante la anotación **@EnableJpaRepositories** le indicamos a Spring el paquete que contiene la clase(repositorio) que utilizamos para acceder a los datos.
```
package com.pokemon.pokemonapi;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.domain.EntityScan;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
@ComponentScan(basePackages = {"com.pokemon.controller","com.pokemon.service", "com.pokemon.dao"})
@EntityScan(basePackages = {"com.pokemon.model"})
@EnableJpaRepositories(basePackages = {"com.pokemon.dao"})
@SpringBootApplication
public class PokemonApiApplication {
	public static void main(String[] args) {
		SpringApplication.run(PokemonApiApplication.class, args);
	}
}
```

9. Realizamos pruebas en Postman de los métodos Get y Post.
<img src="/blog/_img/testApiDB.gif" alt="" />

<br>

Fuentes:
- “Accessing data with MySQL,” Getting Started, 2024. https://spring.io/guides/gs/accessing-data-mysql

- “Microservicios con Spring Boot, Cloud y Docker,” Udemy, 2024. https://www.udemy.com/course/microservicios-spring-boot-cloud/
‌