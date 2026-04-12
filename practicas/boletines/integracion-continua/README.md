# Práctica - Integración continua y análisis estático

En esta práctica se pretende configurar un flujo simple de integración continua utilizando GitHub Actions y SonarQube. 
Este flujo se configurará para el proyecto de la asignatura.


#### **Índice de la práctica**
 1. [GitHub actions](#actions)
 2. [SonarQube](#sonarqube)
 3. [ArchUnit](#archunit)
 4. [Entregable opcional](#entregable)
---


## PARTE 01. GitHub Actions
<a name="actions"></a>
En tu proyecto de la asignatura, crea un nuevo flujo de integración continua utilizando GitHub Actions. Para ello, crea un nuevo archivo de configuración en el directorio `.github/workflows` de tu repositorio. Este archivo debe contener la configuración necesaria para ejecutar las pruebas unitarias de tu proyecto cada vez que se realice un push a la rama principal.

En particular, puesto que el proyecto está configurado con Maven y Java se utilizará `mvn test` para ejecutar las pruebas unitarias. Además, se configurará el flujo para que se ejecute en un entorno de Ubuntu con JDK 21.
El flujo de trabajo se activará sobre cada pull request (a cualquier rama) o push a la rama principal, y realizará las siguientes acciones:

```yaml
name: CI 
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ '*' ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v3
      - name: Set up JDK 21
        uses: actions/setup-java@v1
        with:
          java-version: '21'
      - name: Build with Maven
        run: mvn clean install
      - name: Run tests
        run: mvn -B test
```

## PARTE 02. SonarQube
<a name="sonarqube"></a>

Configura SonarQube para tu proyecto siguiendo las pasos explicados en las transparencias de clase.

## PARTE 03. ArchUnit
<a name="archunit"></a>

Para configurar ArchUnit include la siguiente dependencia:

```xml
<dependency>
    <groupId>com.tngtech.archunit</groupId>
    <artifactId>archunit-junit5</artifactId>
    <version>1.4.1</version>
    <scope>test</scope>
</dependency>
```

Escribe tu primer test de ArchUnit con una regla simple (sustituye `<tu.paquete>` por el paquete raíz de tu proyecto):

```java
package <tu.paquete>;

import com.tngtech.archunit.junit.AnalyzeClasses;
import com.tngtech.archunit.junit.ArchTest;
import com.tngtech.archunit.lang.ArchRule;

import static com.tngtech.archunit.lang.syntax.ArchRuleDefinition.classes;
import static com.tngtech.archunit.lang.syntax.ArchRuleDefinition.methods;
import static com.tngtech.archunit.lang.syntax.ArchRuleDefinition.noClasses;

@AnalyzeClasses(packages ="<tu.paquete>")
public class UmulingoArchTests {

	@ArchTest
	static final ArchRule ninguna_interfaz_acaba_en_impl = classes().
		that().areInterfaces().should().haveSimpleNameNotEndingWith("Impl");
	
}
```

Completa este test con otras reglas que consideres interesantes para tu proyecto. Por ejemplo, puedes añadir una regla para asegurarte de que todas las clases de servicio acaban en "Service", o que todas las clases de repositorio acaban en "Repository".

Por último, verifica la arquitectura hexagonal de tu proyecto. Puedes adaptar el siguiente ejemplo:

```java
	static final ArchRule codigo_respeta_arquitectura_hexagonal = layeredArchitecture()
			// Cojo todas las dependencias del proyecto	
			.consideringAllDependencies() 
			// Defino la capa de dominio como las clases dentro del paquete domain
			.layer("Domain").definedBy("..domain..") 
			// Defino la capa de aplicacion = paquete aplicacion
			.layer("Application").definedBy("..application..")	
			// Adaptadores = clases dentro de adapters
			.layer("Adapters").definedBy("..adapters..")
			// Indico que "imports" de la
			// capa de dominio pueden estar en Application y Adapters
			.whereLayer("Domain").mayOnlyBeAccessedByLayers("Application", "Adapters") 
			// Indico que "imports" de la capa de
			// adaptadores pueden estar en Application (pero no en dominio)
			.whereLayer("Application").mayOnlyBeAccessedByLayers("Adapters")
			// Ninguna capa debe tener import de adapters (lo gestiona Spring Boot)
			.whereLayer("Adapters").mayNotBeAccessedByAnyLayer(); 
```


## Entregable opcional
<a name="entregable"></a>

- Crea un pequeño proyecto SpringBoot prototípico
- Crea test de arquitectura con ArchUnit que comprueben:
  - Las implementaciones de interfaces deben acabar en "Impl"
  - Los DTO acaban en DTO
  - Los RestController solo pueden estar en la capa de adaptadores
  - Ninguna clase debe tener más de 20 métodos públicos
- Crea algunas clases de ejemplo que sean correctas arquitecturalmente y algunas clases que violen las reglas anteriores.

Además, crea un flujo para GitHub Actions que ejecute los tests cada vez que se haga un commit en cualquier rama. Para probarlo, crea un repositorio privado (el repositorio no hace falta compartirlo con el profesor).
