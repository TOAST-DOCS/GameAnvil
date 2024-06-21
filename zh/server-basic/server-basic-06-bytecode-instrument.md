## Game > GameAnvil > Server Concept Description > ByteCode Instrumentation



## ByteCode Instrumentation

GameAnvil is a fiber-based server engine. It uses the Quasar library for this purpose. For fiber-based asynchronous processing, it uses special exceptions promised in advance. Alternatively, you can use @SuspendableAnnotation.

```
throws SuspendExecution
```

To interpret the predefined code, GameAnvil server code must use Quasar library to process ByteCode Instrumentation. ByteCode Instrumentation can be processed by using one of the following two methods.

### Runtime Instrumentation

Add Quasar binary as a javaagent in front of the server run VM option. If this is done, proceed with ByteCode Instrumentation at run-time.

```
-javaagent:MY_PATH\quasar-core-0.7.10-jdk8.jar=bm
```

```
-javaagent:MY_PATH\quasar-core-0.8.0-jdk11.jar=bm
```

> [Note]
>
> This must be added to the beginning of VM option. Set the path of the quasar-core to the path where you copied your quasar-core.



### AOT Instrumentation

If you want to proceed with AOT (ahead-of-time) Instrumentation, add the following to the project object management file (pom.xml), package, install, or deploy the server binary through Maven to complete the compilation, and then proceed with Instrumentation. In this case, of course, javaagent is not required in VM option as in the first case.

#### maven
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-antrun-plugin</artifactId>

    <executions>
        <execution>
            <id>instrument-classes</id>
            <phase>compile</phase>

            <configuration>
                <tasks>
                    <taskdef name="instrumentationTask" classname="co.paralleluniverse.fibers.instrument.InstrumentationTask" classpathref="maven.dependency.classpath"/>
                    <instrumentationTask>
                        <fileset dir="${project.build.directory}/classes/" includes="**/*.class"/>
                    </instrumentationTask>
                </tasks>
            </configuration>

            <goals>
                <goal>run</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

#### gradle
```gradle
// Test done in gradle 4.10.3  
// apply plugin: 'java-library'  
  
configurations {  
    quasar  
	api.setCanBeResolved(true)  
}  
  
compileJava {  
    dependsOn.processResources  
  
    doLast {  
        ant.taskdef(name: 'instrumentation', classname: 'co.paralleluniverse.fibers.instrument.InstrumentationTask', classpath: configurations.api.asPath)  
        ant.instrumentation(verbose: 'true', check: 'true', debug: 'true') {  
            fileset(dir: 'build/classes/') {  
                include(name: '**/*.class')  
            }  
        }  
    }  
}
```