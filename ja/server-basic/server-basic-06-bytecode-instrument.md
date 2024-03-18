## Game > GameAnvil > サーバーの概念説明 > ByteCode Instrumentation



## ByteCode Instrumentation

GameAnvilは、ファイバーベースのサーバーエンジンです。 そのためQuasarライブラリを使用します。ファイバーベースの非同期処理を実行するために、事前に約束された特殊な例外を使用します。もしくは、@Suspendableアノテーション(annotation)を使用することもあります。

```
throws SuspendExecution
```

あらかじめ約束されたこのようなコードを解析するために、GameAnvilサーバーコードは必ずQuasarライブラリを利用して、ByteCode Instrumentationを進行する必要があります。ByteCode Instrumentationは2つの方法のうちいずれかを利用して進行できます。

### Runtime Instrumentation

サーバー実行VMオプションの最前面に次のようにQuasarバイナリをjavaagentとして追加します。そうすると、ランタイムでByteCode Instrumentaionを進行します。

```
-javaagent:MY_PATH\quasar-core-0.7.10-jdk8.jar=bm
```

```
-javaagent:MY_PATH\quasar-core-0.8.0-jdk11.jar=bm
```

> [参考]
>
> この項目は、必ずVMオプションの最前面に追加する必要があります。この時、quasar-coreのパスは本人のquasar-coreをコピーしておいたパスに設定してください。*



### AOT Instrumentation

AOT(ahead-of-time)Instrumentationを進行したい場合は、次の内容をプロジェクトオブジェクト管理ファイル(pom.xml)に追加し、Mavenを通じてサーバーバイナリをpackageやinstall、またはdeployすると、コンパイル完了後にInstrumentationを進行します。この場合、最初の場合と同様にVMオプションでjavaagentは必要ありません。

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
// gradle 4.10.3でテスト済み
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
