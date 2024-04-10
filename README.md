# Swagger 3 and Spring Boot example (with OpenAPI 3)

## 1. Introduction

Document REST API with Swagger 3 in Spring Boot example (follow OpenAPI 3 specification). You will also know several ways to configure Swagger API description and response.

For more detail, please visit:

[Spring Boot with Swagger 3 example](https://www.bezkoder.com/spring-boot-swagger-3/)

https://www.bezkoder.com/spring-boot-swagger-3/

## 2. Source code explanation

### 2.1. Application dependencies/libraries (pom.xml)

These are the dependecies loaded in our sample:

- spring-boot-starter-web
- **springdoc-openapi-starter-webmvc-ui**
- spring-boot-starter-test

**pom.xml**

```xml 
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.0.4</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.bezkoder</groupId>
	<artifactId>spring-boot-swagger-3-example</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-boot-swagger-3-example</name>
	<description>Spring Boot and Swagger 3 example: Rest API Documentation</description>
	<properties>
		<java.version>17</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springdoc</groupId>
			<artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
			<version>2.0.3</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

In the pom.xml file we also selected the Java version. In this case we set the Java 17. 

```xml 
<properties>
		<java.version>17</java.version>
</properties>
```

### 2.2. Project folders and files structure


### 2.3. Application entry point

**SpringBootSwagger3ExampleApplication.java**

```java
package com.bezkoder.spring.swagger;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBootSwagger3ExampleApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootSwagger3ExampleApplication.class, args);
	}

}
```

### 2.4. Data Model

**Tutorial.java**

```java
package com.bezkoder.spring.swagger.model;

import io.swagger.v3.oas.annotations.media.Schema;

public class Tutorial {

  @Schema(accessMode = Schema.AccessMode.READ_ONLY)
  private long id = 0;

  private String title;

  private String description;

  private boolean published;

  public Tutorial() {

  }

  public Tutorial(String title, String description, boolean published) {
    this.title = title;
    this.description = description;
    this.published = published;
  }

  public void setId(long id) {
    this.id = id;
  }

  public long getId() {
    return id;
  }

  public String getTitle() {
    return title;
  }

  public void setTitle(String title) {
    this.title = title;
  }

  public String getDescription() {
    return description;
  }

  public void setDescription(String description) {
    this.description = description;
  }

  public boolean isPublished() {
    return published;
  }

  public void setPublished(boolean isPublished) {
    this.published = isPublished;
  }

  @Override
  public String toString() {
    return "Tutorial [id=" + id + ", title=" + title + ", desc=" + description + ", published=" + published + "]";
  }

}
```

### 2.5. Service (repository)

```
package com.bezkoder.spring.swagger.service;

import java.util.ArrayList;
import java.util.List;

import org.springframework.stereotype.Service;

import com.bezkoder.spring.swagger.model.Tutorial;

@Service
public class TutorialService {

  static List<Tutorial> tutorials = new ArrayList<Tutorial>();
  static long id = 0;

  public List<Tutorial> findAll() {
    return tutorials;
  }

  public List<Tutorial> findByTitleContaining(String title) {
    return tutorials.stream().filter(tutorial -> tutorial.getTitle().contains(title)).toList();
  }

  public Tutorial findById(long id) {
    return tutorials.stream().filter(tutorial -> id == tutorial.getId()).findAny().orElse(null);
  }

  public Tutorial save(Tutorial tutorial) {
    // update Tutorial
    if (tutorial.getId() != 0) {
      long _id = tutorial.getId();

      for (int idx = 0; idx < tutorials.size(); idx++)
        if (_id == tutorials.get(idx).getId()) {
          tutorials.set(idx, tutorial);
          break;
        }

      return tutorial;
    }

    // create new Tutorial
    tutorial.setId(++id);
    tutorials.add(tutorial);
    return tutorial;
  }

  public void deleteById(long id) {
    tutorials.removeIf(tutorial -> id == tutorial.getId());
  }

  public void deleteAll() {
    tutorials.removeAll(tutorials);
  }

  public List<Tutorial> findByPublished(boolean isPublished) {
    return tutorials.stream().filter(tutorial -> isPublished == tutorial.isPublished()).toList();
  }
}
```

### 2.6.  Controller

```java
package com.bezkoder.spring.swagger.controller;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.bezkoder.spring.swagger.model.Tutorial;
import com.bezkoder.spring.swagger.service.TutorialService;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;

@Tag(name = "Tutorial", description = "Tutorial management APIs")
@CrossOrigin(origins = "http://localhost:8081")
@RestController
@RequestMapping("/api")
public class TutorialController {
  @Autowired
  TutorialService tutorialService;

  @Operation(summary = "Create a new Tutorial", tags = { "tutorials", "post" })
  @ApiResponses({
      @ApiResponse(responseCode = "201", content = {
          @Content(schema = @Schema(implementation = Tutorial.class), mediaType = "application/json") }),
      @ApiResponse(responseCode = "500", content = { @Content(schema = @Schema()) }) })
  @PostMapping("/tutorials")
  public ResponseEntity<Tutorial> createTutorial(@RequestBody Tutorial tutorial) {
    try {
      Tutorial _tutorial = tutorialService
          .save(new Tutorial(tutorial.getTitle(), tutorial.getDescription(), tutorial.isPublished()));
      return new ResponseEntity<>(_tutorial, HttpStatus.CREATED);
    } catch (Exception e) {
      return new ResponseEntity<>(null, HttpStatus.INTERNAL_SERVER_ERROR);
    }
  }

  @Operation(summary = "Retrieve all Tutorials", tags = { "tutorials", "get", "filter" })
  @ApiResponses({
      @ApiResponse(responseCode = "200", content = {
          @Content(schema = @Schema(implementation = Tutorial.class), mediaType = "application/json") }),
      @ApiResponse(responseCode = "204", description = "There are no Tutorials", content = {
          @Content(schema = @Schema()) }),
      @ApiResponse(responseCode = "500", content = { @Content(schema = @Schema()) }) })
  @GetMapping("/tutorials")
  public ResponseEntity<List<Tutorial>> getAllTutorials(@RequestParam(required = false) String title) {
    try {
      List<Tutorial> tutorials = new ArrayList<Tutorial>();

      if (title == null)
        tutorialService.findAll().forEach(tutorials::add);
      else
        tutorialService.findByTitleContaining(title).forEach(tutorials::add);

      if (tutorials.isEmpty()) {
        return new ResponseEntity<>(HttpStatus.NO_CONTENT);
      }

      return new ResponseEntity<>(tutorials, HttpStatus.OK);
    } catch (Exception e) {
      return new ResponseEntity<>(null, HttpStatus.INTERNAL_SERVER_ERROR);
    }
  }

  @Operation(
      summary = "Retrieve a Tutorial by Id",
      description = "Get a Tutorial object by specifying its id. The response is Tutorial object with id, title, description and published status.",
      tags = { "tutorials", "get" })
  @ApiResponses({
      @ApiResponse(responseCode = "200", content = { @Content(schema = @Schema(implementation = Tutorial.class), mediaType = "application/json") }),
      @ApiResponse(responseCode = "404", content = { @Content(schema = @Schema()) }),
      @ApiResponse(responseCode = "500", content = { @Content(schema = @Schema()) }) })
  @GetMapping("/tutorials/{id}")
  public ResponseEntity<Tutorial> getTutorialById(@PathVariable("id") long id) {
    Tutorial tutorial = tutorialService.findById(id);

    if (tutorial != null) {
      return new ResponseEntity<>(tutorial, HttpStatus.OK);
    } else {
      return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
  }

  @Operation(summary = "Update a Tutorial by Id", tags = { "tutorials", "put" })
  @ApiResponses({
      @ApiResponse(responseCode = "200", content = {
          @Content(schema = @Schema(implementation = Tutorial.class), mediaType = "application/json") }),
      @ApiResponse(responseCode = "500", content = { @Content(schema = @Schema()) }),
      @ApiResponse(responseCode = "404", content = { @Content(schema = @Schema()) }) })
  @PutMapping("/tutorials/{id}")
  public ResponseEntity<Tutorial> updateTutorial(@PathVariable("id") long id, @RequestBody Tutorial tutorial) {
    Tutorial _tutorial = tutorialService.findById(id);

    if (_tutorial != null) {
      _tutorial.setTitle(tutorial.getTitle());
      _tutorial.setDescription(tutorial.getDescription());
      _tutorial.setPublished(tutorial.isPublished());
      return new ResponseEntity<>(tutorialService.save(_tutorial), HttpStatus.OK);
    } else {
      return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
  }

  @Operation(summary = "Delete a Tutorial by Id", tags = { "tutorials", "delete" })
  @ApiResponses({ @ApiResponse(responseCode = "204", content = { @Content(schema = @Schema()) }),
      @ApiResponse(responseCode = "500", content = { @Content(schema = @Schema()) }) })
  @DeleteMapping("/tutorials/{id}")
  public ResponseEntity<HttpStatus> deleteTutorial(@PathVariable("id") long id) {
    try {
      tutorialService.deleteById(id);
      return new ResponseEntity<>(HttpStatus.NO_CONTENT);
    } catch (Exception e) {
      return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
    }
  }

  @Operation(summary = "Delete all Tutorials", tags = { "tutorials", "delete" })
  @ApiResponses({ @ApiResponse(responseCode = "204", content = { @Content(schema = @Schema()) }),
      @ApiResponse(responseCode = "500", content = { @Content(schema = @Schema()) }) })
  @DeleteMapping("/tutorials")
  public ResponseEntity<HttpStatus> deleteAllTutorials() {
    try {
      tutorialService.deleteAll();
      return new ResponseEntity<>(HttpStatus.NO_CONTENT);
    } catch (Exception e) {
      return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
    }
  }

  @Operation(summary = "Retrieve all published Tutorials", tags = { "tutorials", "get", "filter" })
  @GetMapping("/tutorials/published")
  public ResponseEntity<List<Tutorial>> findByPublished() {
    try {
      List<Tutorial> tutorials = tutorialService.findByPublished(true);

      if (tutorials.isEmpty()) {
        return new ResponseEntity<>(HttpStatus.NO_CONTENT);
      }
      return new ResponseEntity<>(tutorials, HttpStatus.OK);
    } catch (Exception e) {
      return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
    }
  }
}
```

### 2.7. OpenAPI docs configuration

![image](https://github.com/luiscoco/spring-boot-swagger-3-example-master/assets/32194879/039790ee-87c2-409d-8a95-4dfe8291ea36)

**OpenAPIConfig**

```java
package com.bezkoder.spring.swagger.config;

import java.util.List;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import io.swagger.v3.oas.models.servers.Server;

@Configuration
public class OpenAPIConfig {

  @Value("${bezkoder.openapi.dev-url}")
  private String devUrl;

  @Value("${bezkoder.openapi.prod-url}")
  private String prodUrl;

  @Bean
  public OpenAPI myOpenAPI() {
    Server devServer = new Server();
    devServer.setUrl(devUrl);
    devServer.setDescription("Server URL in Development environment");

    Server prodServer = new Server();
    prodServer.setUrl(prodUrl);
    prodServer.setDescription("Server URL in Production environment");

    Contact contact = new Contact();
    contact.setEmail("bezkoder@gmail.com");
    contact.setName("BezKoder");
    contact.setUrl("https://www.bezkoder.com");

    License mitLicense = new License().name("MIT License").url("https://choosealicense.com/licenses/mit/");

    Info info = new Info()
        .title("Tutorial Management API")
        .version("1.0")
        .contact(contact)
        .description("This API exposes endpoints to manage tutorials.").termsOfService("https://www.bezkoder.com/terms")
        .license(mitLicense);

    return new OpenAPI().info(info).servers(List.of(devServer, prodServer));
  }
}
```

### 2.8. Swagger Configuration in Spring Boot

We can configure our API documentation by specifying properties in the spring configuration file

These properties can be classified into OpenAPI and Swagger UI properties

OpenAPI properties specify how the project should be scanned to identify API endpoints and create documentation based on them

Swagger UI properties helps us to customize the user interface of our API documentation

All of these properties start with the prefix springdoc

For a complete list of these properties and their purposes, please visit Springdoc-openapi Properties

The following is a sample of a configuration you can use:

```
#springdoc.api-docs.enabled=false
#springdoc.swagger-ui.enabled=false
#springdoc.packages-to-scan=com.bezkoder.spring.swagger.controller
springdoc.swagger-ui.path=/bezkoder-documentation
springdoc.api-docs.path=/bezkoder-api-docs
#springdoc.swagger-ui.operationsSorter=method
#springdoc.swagger-ui.tagsSorter=alpha
springdoc.swagger-ui.tryItOutEnabled=true
springdoc.swagger-ui.filter=true

bezkoder.openapi.dev-url=http://localhost:8080
bezkoder.openapi.prod-url=https://bezkoder-api.com
```

– Use **api-docs.enabled=false** if you want to disable springdoc-openapi endpoints

– Use **swagger-ui.enabled=false** to disable the swagger-ui endpoint

– **api-docs.path** is for custom path of the OpenAPI documentation in Json format. Now it is **http://localhost:8080/bezkoder-api-docs**

– **swagger-ui.path** is for custom path of the Swagger documentation. 

If you visit **http://localhost:8080/bezkoder-documentation**, the browser will redirect you to **http://localhost:8080/swagger-ui/index.html**

– **packages-to-scan=packageA,packageB**: list of packages to scan with comma separated. We also have packages-to-exclude, paths-to-match, paths-to-exclude.

– **swagger-ui.tryItOutEnabled** if you want to enable “**Try it out**” section by default.

![image](https://github.com/luiscoco/spring-boot-swagger-3-example-master/assets/32194879/1ae20ce2-546e-4abe-a8ae-3919ac8a5076)

– **swagger-ui.operationsSorter**: ‘**alpha**’ (sort by paths **alphanumerically**), ‘**method**’ (sort by **HTTP method**) or a function.

– **swagger-ui.tagsSorter**: ‘**alpha**’ (sort by paths **alphanumerically**) or a **function**.

– **swagger-ui.filter**: true/false to enable or disable filter the tagged operations. 

We can set a string, the filtering will be enabled using that string as the filter expression which is case sensitive matching anywhere inside the tag.

You can also use enable-spring-security, enable-hateoas, enable-data-rest…

For more properties and details, please visit **Springdoc-openapi Properties**: https://springdoc.org/

## 3. How to run the application type this command in VSCode Terminal Window:

```
mvn spring-boot:run
```

To access the API docs navigate to this URL: **http://localhost:8080/swagger-ui/index.html**

![image](https://github.com/luiscoco/spring-boot-swagger-3-example-master/assets/32194879/46d626bb-9ba4-4b62-9b37-2a04bedd9514)

![image](https://github.com/luiscoco/spring-boot-swagger-3-example-master/assets/32194879/6950d645-4012-4cef-99b6-34115a759608)

![image](https://github.com/luiscoco/spring-boot-swagger-3-example-master/assets/32194879/edc18735-c0a4-40eb-92e7-da5af89978a2)

Also we can navigate to the OpenAPI docs

http://localhost:8080/bezkoder-api-docs

![image](https://github.com/luiscoco/spring-boot-swagger-3-example-master/assets/32194879/1578ff1f-728b-49ef-aabf-0df5ff5679e3)

