# Como utilizar PostgreSQL, y como añadirlo a nuestra Api.

Como utilizar PostgreSQL:

Lo primero es instalar PostgreSql desde su pagina oficial, ojo este tutorial es solo para usuarios de Windows para Linux voy hacerlo después, iniciamos el instalador, siguen los pasos, la contraseña para la base de datos etc, en el puerto les recomiendo que lo cambien al puerto 5050 ya que es muy probable que no interfiera con alguna otra aplicación local que requiera de un puerto, después de hacer la instalación tendrán que abrir pgadmin 4 (yo tuve que hacer la instalación de postgre dos veces ya que no me aparecía mi base de datos local con el puerto 5050), les dará la opción de ingresar con su base de datos recién creada, presionan esa opción y le dan a siguiente, les abrirá la vista grafica de su base de datos pero primero deberán ingresar su contraseña, listo, con estos pasos listo tienen PostgreSQL listo para ocupar, muchas gracias.

Tenemos que tener en cuenta que en nuestro archivo [application.properties](http://application.properties) tenemos que tener todo inicializado, tal que asi:

```
# Nombre de la aplicación
spring.application.name=*Nombre*

# ===============================================
# Configuración de la base de datos PostgreSQL
# ===============================================
spring.datasource.url=jdbc:postgresql://localhost:*Aqui ponen el puerto*/postgres
spring.datasource.username=postgres
spring.datasource.password=*Contraseña que pusimos en Postgre al momento de la instalacion*
spring.datasource.driver-class-name=org.postgresql.Driver

spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update

logging.level.org.hibernate=DEBUG
logging.level.com.zaxxer.hikari=DEBUG
logging.level.java.sql=DEBUG
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

management.endpoints.web.exposure.include=*
```

Con esta breve introducción podemos dar pie a como integrar esta base de datos, para que nuestra Api sea mas robusta, Lo primero es tener nuestro archivo pom.xml con la siguiente dependencia si no la inicializaron al descargarlo desde springboot:

```xml
		<dependency>
    		<groupId>org.postgresql</groupId>
    		<artifactId>postgresql</artifactId>
    		<scope>runtime</scope>
        </dependency>
```

con esta dependencia creada, ahora si vamos hacer esto mucho mas facil, ya no necesitamos un ArrayList para poder guardar nuestras respuestas, ahora usaremos nuestra base de datos para almacenar todo eso.

Con esto hecho procedemos a importar.

Model:

Para hacer la importacion y correcto uso de esta estare dejando fragmentos de codigo para que interpreten como van, para el model utilice importaciones de jakarta y lombok, para que funciona cada una?, bueno lo que hace lombok es que en ves de hacer nuestros constructores con y sin parametros, llamamos a esta importacion y asi nos ahorramos todo ese codigo, lombok tambien tiene que ser integrada a nuestro archivo pom.xml(Si es que no la integraron en la propia pagina de Sprinboot y seria tal que así:

```xml
<dependency>
 <groupId>org.projectlombok</groupId>
 <artifactId>lombok</artifactId>
 <optional>true</optional>
</dependency>
 <path>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
 </path>
 <exclude>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
 </exclude>
```

Ya con esto hecho podemos hacer las importaciones en model,

las importaciones se llaman tal que así:

import lombok.Data;

Esta importacion nos ayuda para no hacer los Getters y Setters en nuestro código, lo que lo hace mas limpio.

```java
import lombok.Data;
@Data
```

import lombok.AllArgsConstructor;

esta importación nos sirve para no general los constructores con parámetros.

```java
import lombok.AllArgsConstructor;
@AllArgsConstructor
```

import lombok.NoArgsContructor;

Esta importación nos ayudara para no crear los constructores sin parámetros.

```java
import lombok.NoArgsConstructor;
@NoArgsConstructor
```

Ya terminamos por el lado de Lombok, ahora vamos con jakarta:

Las importaciones seran las siguente.

*import* jakarta.persistence.Entity;

Esta importación le dice a nuestro codigo/clase que es una entidad JPA.

```java
*import* jakarta.persistence.Entity;
@Entity
```

*import* jakarta.persistence.Id;

Esta importacion nos ayudara a inicializar nuestro id en la tabla/clase.

```java
*import* jakarta.persistence.Id;
    @Id
    private String rut;
```

*import* jakarta.persistence.Table;

Esta importacion nos ayudara para el tema de crear o señalar nuestra tabla para la base de datos

```java
*import* jakarta.persistence.Table;
@Table(name = "Logins")
```

Y con esta explicación cerramos el model, hay que tener en cuenta que todo esto lo estoy testeando desde mi maquina, cualquier fallo puede ocurrir, si es así, no dudes en enviarme un correo para poder responder tu solicitud a la brevedad.

Repository:

Como lo explique ya no necesitamos el arraylist para poder guardar, desde ahora solo usaremos nuestra importacion, y con esta hacer menos extenso nuestro codigo, gracias hacer el cambio eliminamos gran parte de nuestro codigo, ya que vamos a extender nuestra clase a Jparepository y con esto generara automáticamente las funciones del CRUD, también usaremos la importación de Optional para manejar en caso de no encontrar un registro.

Tienen que hacer una importación que llame al model ya que de ahí viene todo que seria algo así,

```java
import com.example.ecomarket.model.LoginModel;
```

también importaremos desde sprinframework estas dos,

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
```

y por ultimo importamos Optional

```java
import java.util.Optional;
```

y se veria algo asi nuestro Repository.

```java
package com.example.ecomarket.repository;

import com.example.ecomarket.model.LoginModel;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

/*importacion de Java*/
import java.util.Optional; 

@Repository
/*Hacemos una extencion para que obtenga todos los metodos del CRUD */
public interface LoginRepository extends JpaRepository<LoginModel, String> {

    Optional<LoginModel> findByRut(String rut);

}
```

Nos quedaría Controller y Serice, que lo haremos ahora:

Service:

para service utilizaremos importaciones de Optional y List con las siguentes importaciones de springframework:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
```

antes de empezar el public class tenemos que agregar el @Service que ayuda a spring o mas bien detecta la clase automaticamente.

```java
@Service
public class nombreService{

}
```

Junto a las importaciones de Java que serian List y Optional:

```java
import java.util.List;
import java.util.Optional;
```

Cuando ya tengamos todo empezamos dentro de nuestro public class un @Autowired este nos ayudara a la Inyeccion de  dependencias por constructor. Luego realizamos un:

```java
public nombreService(nombreRepository nombreRepository){
	this.nombreRepository = nombreRepository;
}
```

Con esto podremos empezar hacer nuestro CRUD, gracias a la extencion a JPA las funciones para cada uno de los funcionamientos del CRUD son:

```java
public List<nombreModel> obtenerTodosLosLogins() {
    return nombreRepository.findAll();
}

public Optional<nombreModel> buscarNombrePorRut(String rut) {
    return nombreRepository.findByRut(rut);/*Aqui podemos ver la funcion que creamos*/
}
    
public nombreModel guardarNombre(NombreModel login) {
    return nombreRepository.save(login);
}

public LoginModel actualizarLogin(LoginModel login) {
        Optional<LoginModel> existingLoginOptional = loginRepository.findByRut(login.getRut());

        if (existingLoginOptional.isPresent()) {
            LoginModel existingLogin = existingLoginOptional.get();
            existingLogin.setNombreP(login.getNombreP());
            existingLogin.setNombreM(login.getNombreM());
            existingLogin.setApellidoP(login.getApellidoP());
            existingLogin.setApellidoM(login.getApellidoM());
            existingLogin.setCelurlar(login.getCelurlar());
            existingLogin.setCodigoPostal(login.getCodigoPostal());
            existingLogin.setCorreoElectronico(login.getCorreoElectronico());
            existingLogin.setDireccion(login.getDireccion());

            return loginRepository.save(existingLogin);
        } else {
            return null;
        }
    }
    public boolean eliminarLogin(String rut) {
        if (loginRepository.existsById(rut)) {
            loginRepository.deleteById(rut);
            return true;
        }
        return false;
    }
```

ese seria el codigo para poder hacer un CRUD.

Para nuestro controller empezaremos con las importaciones que serian:

```java
*import* org.springframework.beans.factory.annotation.Autowired;
```

Una breve explicación hecha por copilot: sirve para importar la anotación @Autowired de Spring Framework. Esta anotación se utiliza para inyectar automáticamente dependencias (por ejemplo, servicios o repositorios) en tus clases, permitiendo que Spring cree y gestione los objetos necesarios sin que tú tengas que instanciarlos manualmente.

```java
*import* org.springframework.http.HttpStatus;
```

Esta importación lo que hace es que cuando queramos decir que el link/url sea encontrada en base al protocolo HTTP lo muestre, y si es lo contrario, también.

Les dejare mi codigo de controller el cual tambien pueden visualizar en mi rama del proyecto ecomarket, el cual es creator/login.

```java
package com.example.ecomarket.controller;

import com.example.ecomarket.model.LoginModel;
import com.example.ecomarket.services.loginServices;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("/api/logins")
public class LoginController {

    private final loginServices loginServices;

    @Autowired // Inyección de dependencia del LoginService
    public LoginController(loginServices loginServices) {
        this.loginServices = loginServices;
    }

    @GetMapping
    public ResponseEntity<List<LoginModel>> getAllLogins() {
        List<LoginModel> logins = loginServices.obtenerTodosLosLogins();
        return new ResponseEntity<>(logins, HttpStatus.OK);
    }

    @GetMapping("/{rut}")
    public ResponseEntity<LoginModel> getLoginByRut(@PathVariable String rut) {
        Optional<LoginModel> login = loginServices.buscarLoginPorRut(rut);
        return login.map(value -> new ResponseEntity<>(value, HttpStatus.OK))
                    .orElseGet(() -> new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }

    @PostMapping
    public ResponseEntity<LoginModel> createLogin(@RequestBody LoginModel login) {
        LoginModel savedLogin = loginServices.guardarLogin(login);
        return new ResponseEntity<>(savedLogin, HttpStatus.CREATED);
    }

    @PutMapping
    public ResponseEntity<LoginModel> updateLogin(@RequestBody LoginModel login) {
        LoginModel updatedLogin = loginServices.actualizarLogin(login);
        if (updatedLogin != null) {
            return new ResponseEntity<>(updatedLogin, HttpStatus.OK);
        } else {
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }
    }

    @DeleteMapping("/{rut}")
    public ResponseEntity<Void> deleteLogin(@PathVariable String rut) {
        boolean deleted = loginServices.eliminarLogin(rut);
        if (deleted) {
            return new ResponseEntity<>(HttpStatus.NO_CONTENT);
        } else {
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }
    }
}
```

Con esta información lista, podremos tener conexión entre nuestra API REST y nuestra base de datos PostgreSQL.