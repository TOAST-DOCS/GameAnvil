## Game > GameAnvil > 서버 개념 설명 > ByteCode Instrumentation



## 1. ByteCode Instrumentation

GameAnvil은 파이버 기반의 서버 엔진입니다. 이를 위해 Quasar 라이브러리를 사용합니다.  파이버 기반의 비동기 처리를 위해서 미리 약속된 특수한 예외를 사용합니다. 혹은 @Suspendable 애너테이션(annotation)을 사용할 수도 있습니다.

```
throws SuspendExecution
```

미리 약속된 이런 코드를 해석하기 위해 GameAnvil 서버 코드는 반드시 Quasar 라이브러리를 이용하여 ByteCode Instrumentation을 진행해야 합니다. ByteCode Instrumentation은 두 가지 방식 중 한 가지를 이용해서 진행할 수 있습니다.

### 1.1. Runtime Instrumentation

서버 실행 VM 옵션의 가장 앞 부분에 아래와 같이 Quasar 바이너리를 javaagent로 추가합니다. 이렇게만 하면 런타임에 ByteCode Instrumentaion을 진행합니다.

```
-javaagent:MY_PATH\quasar-core-0.7.10-jdk8.jar=bm
```

### Note

*이 항목은 반드시 VM 옵션의 가장 앞 부분에 추가해야 합니다. 이때, quasar-core의 경로는 본인의 quasar-core를 복사해둔 경로로 설정하세요.*



### 1.2. AOT Instrumentation

AOT(ahead-of-time) Instrumentation을 진행하고 싶다면 아래의 내용을 프로젝트 객체 관리 파일(pom.xml)에 추가한 후, Maven을 통해 서버 바이너리를 package나 install 혹은 deploy하면 컴파일 완료 후 Instrumentation을 진행합니다. 이 경우에는 당연히 첫째 경우처럼 VM 옵션에서 javaagent가 필요하지 않습니다.

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

