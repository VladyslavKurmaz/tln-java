# Java (openjdk-11) skeleton application

## Getting started

### Install tln-cli
* [talan cli](https://github.com/project-talan/tln-cli)

### Attach to the existing project
* Add skeleton as subtree
  ```
  git remote add tln-java https://github.com/project-talan/tln-java.git
  git subtree add --prefix services/api tln-java master --squash
  ```
* Update to get latest version
  ```
  git subtree pull --prefix services/api tln-java master --squash
  ```

### or Fork/clone repository
To develop standalone project, just clone repository or create fork using your account

### Refresh configuration
* execute next command from the project's home using command line
  ```
  tln prereq
  ```
* Update environment variables inside **.env** file
  ```
  COMPONENT_GROUP_ID=io.company.project.services
  COMPONENT_ARTIFACT_ID=api
  COMPONENT_ID=io.company.project.services.api
  COMPONENT_VERSION=19.8.0-SNAPSHOT
  COMPONENT_PARAM_HOST=localhost
  COMPONENT_PARAM_LSTN=0.0.0.0
  COMPONENT_PARAM_PORT=80
  COMPONENT_PARAM_PORTS=443
  COMPONENT_PARAM_SSL_CERTS=
  COMPONENT_PARAM_CORS_WHITELIST=
  ```
### Generate project skeleton using maven
  ```
  tln generate-jersey-grizzly2
  ```
### Add missing dependencies
  ```
  <dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.0</version>
  </dependency>
  <dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-impl</artifactId>
    <version>2.3.0.1</version>
  </dependency>
  <dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-core</artifactId>
    <version>2.3.0.1</version>
  </dependency>
  <dependency>
    <groupId>javax.activation</groupId>
    <artifactId>javax.activation-api</artifactId>
    <version>1.2.0</version>
  </dependency>
```

### Set artifact final name
  ```
  <finalName>${project.groupId}.${project.artifactId}-${project.version}</finalName>
  ```
### Update compilation plugin
  ```
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <inherited>true</inherited>
    <configuration>
      <source>11</source>
      <target>11</target>
    </configuration>
  </plugin>
  ```
### Update version for exec plugin
  ```
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <version>1.6.0</version>
  ```  
### Add assemply plugin
  ```
  <plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.1.1</version>
    <executions>
      <execution>
        <phase>package</phase>
        <goals>
          <goal>single</goal>
        </goals>
      </execution>
    </executions>
    <configuration>
      <archive>
        <manifest>
          <addClasspath>true</addClasspath>
          <mainClass>${project.groupId}.${project.artifactId}.Main</mainClass>
        </manifest>
      </archive>
      <descriptorRefs>
        <descriptorRef>jar-with-dependencies</descriptorRef>
      </descriptorRefs>
    </configuration>
  </plugin>
  ```
### Update Main.java to aligned with docker
  ```
  import java.util.concurrent.CountDownLatch;
  import org.apache.log4j.Logger;

  public class Main {
  
    private static final Logger logger = Logger.getLogger(Main.class);
    
    /**
     * Starts Grizzly HTTP server exposing JAX-RS resources defined in this application.
     * @return Grizzly HTTP server.
     */
    public static HttpServer createServer() {
        // create a resource config that scans for JAX-RS resources and providers
        // in io.company.project.services.api package
        final ResourceConfig rc = new ResourceConfig().packages("io.company.project.services.api");

        // create and start a new instance of grizzly http server
        // exposing the Jersey application at BASE_URI
        return GrizzlyHttpServerFactory.createHttpServer(URI.create(BASE_URI), rc);
    } 
    
    public static void main(String[] args) throws IOException {
      logger.info("Initiliazing Grizzly server using " + BASE_URI);
      CountDownLatch exitEvent = new CountDownLatch(1);
      HttpServer server = createServer();
      // register shutdown hook
      Runtime.getRuntime().addShutdownHook(new Thread(() -> {
        logger.info("Stopping server ...");
        server.stop();
        exitEvent.countDown();
      }, "shutdownHook"));

      try {
        server.start();
        logger.info(String.format("Jersey app started with WADL available at %sapplication.wadl", BASE_URI));
        logger.info("Press CTRL^C to exit ...");
        exitEvent.await();
        logger.info("Exiting service ...");
      } catch (InterruptedException e) {
        logger.error("There was an error while starting Grizzly HTTP server.", e);
        Thread.currentThread().interrupt();
      }
    }
  }
  ```
### Migration tutorials
* http://www.javainthebox.com/2018/07/case-study-of-migration-to-java-se-11.html
* https://winterbe.com/posts/2018/08/29/migrate-maven-projects-to-java-11-jigsaw/
* https://medium.com/@Leejjon_net/migrate-a-jersey-based-micro-service-to-java-11-and-deploy-to-app-engine-7ba41a835992

### HTTP/HTTPS
* During deployment procedure, create ssl folder under project's root with two sertificates. Use your project id as files' names
  ```
    io.company.project.services.api.key
    io.company.project.services.api.crt
  ```
* otherwise, **http** access will be configured


## SDLC


| command  | Description |
| ------------- | ------------- |
| tln  prereq | Prepare local dev box configuration scripts |
