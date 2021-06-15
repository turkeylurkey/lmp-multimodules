## Test different project layouts

a)  pom, ear+war+jar
- Server info in ear module
- Main pom: ./pom.xml
- Tests exist in every module

a.2) pom folder at the same level as (sibling folder of) ear+war+jar
- Server info in ear module
 - Main pom: ./pom/pom.xml

b)  pom, war+jar
- Server info in war module
- Main pom: ./pom.xml

~~c)  ear(MM)+war+jar~~
- invalid: <module> requires packaging type pom

~~d)  war(MM)+jar~~    
- invalid: <module> requires packaging type pom

e)  pom, liberty-assembly+ear+war+jar    
- Server info in pom module
- Main pom: ./pom.xml
- For devc, use -Ddockerfile=../Dockerfile
- Tests exist in every module

~~f)  liberty-assembly+ear+war+jar~~    
- invalid: <module> requires packaging type pom

g)  pom,  server(pom)+ear+war+jar    
- Server info in pom module
- Main pom: ./pom.xml
- For devc, use -Ddockerfile=../Dockerfile

h)  server(pom)+ear+war+jar    
- Server info in pom module
- Main pom: ./pom/pom.xml
- For devc, use -Ddockerfile=../Dockerfile

i)   same as (a) but "ear" project has a `<parent>` which is not the "pom" project
- Server info in ear module
- Main pom: ./pom.xml

## Test process
- ensure hot deployment works
    - sources
    - resources
    - server config
- see what happens with pressing Enter and/or hot tests
    - ensuring debug works on the different modules (works)
    - ensure hot code replacement works on the different module (works)

### General Testing Scenario (based on project typeE)

typeE
main module: pom
upstream modules: ear, war, jar

1. Start dev mode: `mvn io.openliberty.tools:liberty-maven-plugin:3.3.5-M2-SNAPSHOT:dev -pl pom -am`

2. Press `Enter` to run tests, all tests across all modules should run in the same order as the reactor build.

3. Verify hot deployment of a java source file in main and upstream modules. On successful compilation, dev mode should retry compiling all other failing source files across ALL modules. 

    For example:
    - in `war/src/main/java/io/openliberty/guides/multimodules/web/HeightsBean.java`, add an extra parameter to one of the `io.openliberty.guides.multimodules.lib.Converter` method calls. Dev mode should throw compilation errors.
    - in `jar/src/main/java/io/openliberty/guides/multimodules/lib/Converter.java`, add the new parameter to the corresponding method. Dev mode should successfully compile the `Converter` class from the jar module, and then try to compile the failing `HeightsBean` class from the war module.

4. Verify hot deployment of a java test file in main and upstream modules. Pressing `Enter` should run tests with latest change. On successful compilation, dev mode should retry compiling all other failing tests within the SAME module. 

    Try adding a new `src/test/java` directory to any of the upstream modules, that directory should be watched and files within should be compiled upon change.

    Note: Any unit tests added to an `ear` module should be skipped.

5. Add the `skipTests` or `skipITs` property to any of the modules' pom.xml file, app should redeploy, and press Enter to see if the change took effect. These properties are handled by the Maven surefire/failsafe plugins. Otherwise, if you would like to configure `skipUTs` with different values for each module, stop dev mode, and add the `<skipUTs>true</skipUTs>` to the module's pom file where unit tests should be skipped. This parameter is handled exclusively by dev mode and changes to it will only take effect on a new dev mode instance (this matches current dev mode behaviour). 

    Node: If skipTests, skipUTs, or skipITsflag is enabled in a module, skip the corresponding tests in that module only. 

    For example:
    - in `war/pom.xml` add the following property: `<skipITs>true</skipITs>`
    - when you press Enter, the `HeightsBeanIT.java` tests should be skipped.


6. Adding a new dependency to the pom.xml of an upstream module. This should trigger recompiling any classes that failed to compile.

    For example
    - Add the following to `jar/src/main/java/io/openliberty/guides/multimodules/lib/Converter.java`. Notice the compilation error.
    ```java
    import jakarta.enterprise.context.ApplicationScoped;

    @ApplicationScoped
    public class Converter {
        ...
    }
    ```
    - Add the corresponding dependency to `jar/pom.xml`. Dev mode should try (and succeed) to recompile `jar/src/main/java/io/openliberty/guides/multimodules/lib/Converter.java`.
    ```xml
    <dependency>
        <groupId>jakarta.platform</groupId>
        <artifactId>jakarta.jakartaee-api</artifactId>
        <version>9.0.0</version>
        <scope>provided</scope>
    </dependency>
    ```

7. Try adding a resource directory to an upstream module (ie. `war/src/main/resources`). New resource directories should be picked up after dev mode has started and watched for file changes. On file change, the file should be copied into the corresponding modules `target/classes` directory.


**Other Scenarios**
- devc
- Restart server with "r" and "Enter", both with dev mode and devc
- Compilation errors on startup of dev mode
- Connecting a debugger
- Setting `-DhotTests` via command line or `pom.xml` 
