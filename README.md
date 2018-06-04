# Gradle 4.8 で導入された Dependency Locking のデモレポジトリ

https://docs.gradle.org/4.8/userguide/dependency_locking.html

を参考に実際に動かすとこんな感じだよ、というののデモ的なレポジトリです。

実際にどのような感じになるのかについては https://github.com/tokuhirom/gradle-dependency-locking-demo/commits/master から参照してください

## 雛形作成。

start.spring.io で雛形を作成。
2.0.2.RELEASE 用になってるので 2.0.0 用に書き換える。

## gradle 4.8 にアップグレード。

    ./gradlew wrapper --gradle-version=4.8

以下でロック。

    ./gradlew dependencies --write-locks

`gradle/dependency-locks/` 以下にロックファイルが生成される。

この状態で、以下のような変更を build.gradle に加える。

```
diff --git build.gradle build.gradle
index a76663a..0750b81 100644
--- build.gradle
+++ build.gradle
@@ -1,6 +1,6 @@
 buildscript {
        ext {
-               springBootVersion = '2.0.0.RELEASE'
+               springBootVersion = '2.0.1.RELEASE'
        }
        repositories {
                mavenCentral()
```

この状態で `./gradlew dependencies` を実行すると以下のような出力を得る。

```
$ ./gradlew dependencies

> Task :dependencies FAILED

------------------------------------------------------------
Root project
------------------------------------------------------------

annotationProcessor - Annotation processors and their dependencies for source set 'main'.
No dependencies

apiElements - API elements for main. (n)
No dependencies

archives - Configuration for archive artifacts.
No dependencies

bootArchives - Configuration for Spring Boot archive artifacts.
No dependencies

compile - Dependencies for source set 'main' (deprecated, use 'implementation ' instead).

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':dependencies'.
> Could not resolve all dependencies for configuration ':compile'.
   > Dependency lock state for configuration 'compile' is out of date:
       - Did not resolve 'org.springframework.boot:spring-boot-autoconfigure:2.0.0.RELEASE' which is part of the lock state
       - Did not resolve 'org.springframework.boot:spring-boot-starter-logging:2.0.0.RELEASE' which is part of the lock state
       - Did not resolve 'org.springframework.boot:spring-boot-starter:2.0.0.RELEASE' which is part of the lock state
       - Did not resolve 'org.springframework.boot:spring-boot:2.0.0.RELEASE' which is part of the lock state
       - Did not resolve 'org.springframework:spring-aop:5.0.4.RELEASE' which is part of the lock state
       - Did not resolve 'org.springframework:spring-beans:5.0.4.RELEASE' which is part of the lock state
       - Did not resolve 'org.springframework:spring-context:5.0.4.RELEASE' which is part of the lock state
       - Did not resolve 'org.springframework:spring-core:5.0.4.RELEASE' which is part of the lock state
       - Did not resolve 'org.springframework:spring-expression:5.0.4.RELEASE' which is part of the lock state
       - Did not resolve 'org.springframework:spring-jcl:5.0.4.RELEASE' which is part of the lock state
       - Resolved 'org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE' which is not part of the lock state
       - Resolved 'org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE' which is not part of the lock state
       - Resolved 'org.springframework.boot:spring-boot-starter:2.0.1.RELEASE' which is not part of the lock state
       - Resolved 'org.springframework.boot:spring-boot:2.0.1.RELEASE' which is not part of the lock state
       - Resolved 'org.springframework:spring-aop:5.0.5.RELEASE' which is not part of the lock state
       - Resolved 'org.springframework:spring-beans:5.0.5.RELEASE' which is not part of the lock state
       - Resolved 'org.springframework:spring-context:5.0.5.RELEASE' which is not part of the lock state
       - Resolved 'org.springframework:spring-core:5.0.5.RELEASE' which is not part of the lock state
       - Resolved 'org.springframework:spring-expression:5.0.5.RELEASE' which is not part of the lock state
       - Resolved 'org.springframework:spring-jcl:5.0.5.RELEASE' which is not part of the lock state

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 2s
1 actionable task: 1 executed
```

これにより、ロックされていることが確認できる。

## 依存のアップデート

`./gradlew dependencies --update-locks '*:*'` を実行する。
これによりすべての依存がアップデートされる。

このコマンドの出力結果は以下のようになる。

```
> Task :dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

annotationProcessor - Annotation processors and their dependencies for source set 'main'.
Persisted dependency lock state for configuration 'annotationProcessor'
No dependencies

apiElements - API elements for main. (n)
No dependencies

archives - Configuration for archive artifacts.
Persisted dependency lock state for configuration 'archives'
No dependencies

bootArchives - Configuration for Spring Boot archive artifacts.
Persisted dependency lock state for configuration 'bootArchives'
No dependencies

compile - Dependencies for source set 'main' (deprecated, use 'implementation ' instead).
Persisted dependency lock state for configuration 'compile'
\--- org.springframework.boot:spring-boot-starter -> 2.0.1.RELEASE
     +--- org.springframework.boot:spring-boot:2.0.1.RELEASE
     |    +--- org.springframework:spring-core:5.0.5.RELEASE
     |    |    \--- org.springframework:spring-jcl:5.0.5.RELEASE
     |    \--- org.springframework:spring-context:5.0.5.RELEASE
     |         +--- org.springframework:spring-aop:5.0.5.RELEASE
     |         |    +--- org.springframework:spring-beans:5.0.5.RELEASE
     |         |    |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         +--- org.springframework:spring-beans:5.0.5.RELEASE (*)
     |         +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         \--- org.springframework:spring-expression:5.0.5.RELEASE
     |              \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     +--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
     |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
     |    +--- ch.qos.logback:logback-classic:1.2.3
     |    |    +--- ch.qos.logback:logback-core:1.2.3
     |    |    \--- org.slf4j:slf4j-api:1.7.25
     |    +--- org.apache.logging.log4j:log4j-to-slf4j:2.10.0
     |    |    +--- org.slf4j:slf4j-api:1.7.25
     |    |    \--- org.apache.logging.log4j:log4j-api:2.10.0
     |    \--- org.slf4j:jul-to-slf4j:1.7.25
     |         \--- org.slf4j:slf4j-api:1.7.25
     +--- javax.annotation:javax.annotation-api:1.3.2
     +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     \--- org.yaml:snakeyaml:1.19

compileClasspath - Compile classpath for source set 'main'.
Persisted dependency lock state for configuration 'compileClasspath'
\--- org.springframework.boot:spring-boot-starter -> 2.0.1.RELEASE
     +--- org.springframework.boot:spring-boot:2.0.1.RELEASE
     |    +--- org.springframework:spring-core:5.0.5.RELEASE
     |    |    \--- org.springframework:spring-jcl:5.0.5.RELEASE
     |    \--- org.springframework:spring-context:5.0.5.RELEASE
     |         +--- org.springframework:spring-aop:5.0.5.RELEASE
     |         |    +--- org.springframework:spring-beans:5.0.5.RELEASE
     |         |    |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         +--- org.springframework:spring-beans:5.0.5.RELEASE (*)
     |         +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         \--- org.springframework:spring-expression:5.0.5.RELEASE
     |              \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     +--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
     |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
     |    +--- ch.qos.logback:logback-classic:1.2.3
     |    |    +--- ch.qos.logback:logback-core:1.2.3
     |    |    \--- org.slf4j:slf4j-api:1.7.25
     |    +--- org.apache.logging.log4j:log4j-to-slf4j:2.10.0
     |    |    +--- org.slf4j:slf4j-api:1.7.25
     |    |    \--- org.apache.logging.log4j:log4j-api:2.10.0
     |    \--- org.slf4j:jul-to-slf4j:1.7.25
     |         \--- org.slf4j:slf4j-api:1.7.25
     +--- javax.annotation:javax.annotation-api:1.3.2
     +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     \--- org.yaml:snakeyaml:1.19

compileOnly - Compile only dependencies for source set 'main'.
Persisted dependency lock state for configuration 'compileOnly'
No dependencies

default - Configuration for default artifacts.
Persisted dependency lock state for configuration 'default'
\--- org.springframework.boot:spring-boot-starter -> 2.0.1.RELEASE
     +--- org.springframework.boot:spring-boot:2.0.1.RELEASE
     |    +--- org.springframework:spring-core:5.0.5.RELEASE
     |    |    \--- org.springframework:spring-jcl:5.0.5.RELEASE
     |    \--- org.springframework:spring-context:5.0.5.RELEASE
     |         +--- org.springframework:spring-aop:5.0.5.RELEASE
     |         |    +--- org.springframework:spring-beans:5.0.5.RELEASE
     |         |    |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         +--- org.springframework:spring-beans:5.0.5.RELEASE (*)
     |         +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         \--- org.springframework:spring-expression:5.0.5.RELEASE
     |              \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     +--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
     |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
     |    +--- ch.qos.logback:logback-classic:1.2.3
     |    |    +--- ch.qos.logback:logback-core:1.2.3
     |    |    \--- org.slf4j:slf4j-api:1.7.25
     |    +--- org.apache.logging.log4j:log4j-to-slf4j:2.10.0
     |    |    +--- org.slf4j:slf4j-api:1.7.25
     |    |    \--- org.apache.logging.log4j:log4j-api:2.10.0
     |    \--- org.slf4j:jul-to-slf4j:1.7.25
     |         \--- org.slf4j:slf4j-api:1.7.25
     +--- javax.annotation:javax.annotation-api:1.3.2
     +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     \--- org.yaml:snakeyaml:1.19

implementation - Implementation only dependencies for source set 'main'. (n)
No dependencies

runtime - Runtime dependencies for source set 'main' (deprecated, use 'runtimeOnly ' instead).
Persisted dependency lock state for configuration 'runtime'
\--- org.springframework.boot:spring-boot-starter -> 2.0.1.RELEASE
     +--- org.springframework.boot:spring-boot:2.0.1.RELEASE
     |    +--- org.springframework:spring-core:5.0.5.RELEASE
     |    |    \--- org.springframework:spring-jcl:5.0.5.RELEASE
     |    \--- org.springframework:spring-context:5.0.5.RELEASE
     |         +--- org.springframework:spring-aop:5.0.5.RELEASE
     |         |    +--- org.springframework:spring-beans:5.0.5.RELEASE
     |         |    |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         +--- org.springframework:spring-beans:5.0.5.RELEASE (*)
     |         +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         \--- org.springframework:spring-expression:5.0.5.RELEASE
     |              \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     +--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
     |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
     |    +--- ch.qos.logback:logback-classic:1.2.3
     |    |    +--- ch.qos.logback:logback-core:1.2.3
     |    |    \--- org.slf4j:slf4j-api:1.7.25
     |    +--- org.apache.logging.log4j:log4j-to-slf4j:2.10.0
     |    |    +--- org.slf4j:slf4j-api:1.7.25
     |    |    \--- org.apache.logging.log4j:log4j-api:2.10.0
     |    \--- org.slf4j:jul-to-slf4j:1.7.25
     |         \--- org.slf4j:slf4j-api:1.7.25
     +--- javax.annotation:javax.annotation-api:1.3.2
     +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     \--- org.yaml:snakeyaml:1.19

runtimeClasspath - Runtime classpath of source set 'main'.
Persisted dependency lock state for configuration 'runtimeClasspath'
\--- org.springframework.boot:spring-boot-starter -> 2.0.1.RELEASE
     +--- org.springframework.boot:spring-boot:2.0.1.RELEASE
     |    +--- org.springframework:spring-core:5.0.5.RELEASE
     |    |    \--- org.springframework:spring-jcl:5.0.5.RELEASE
     |    \--- org.springframework:spring-context:5.0.5.RELEASE
     |         +--- org.springframework:spring-aop:5.0.5.RELEASE
     |         |    +--- org.springframework:spring-beans:5.0.5.RELEASE
     |         |    |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         +--- org.springframework:spring-beans:5.0.5.RELEASE (*)
     |         +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     |         \--- org.springframework:spring-expression:5.0.5.RELEASE
     |              \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     +--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
     |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
     |    +--- ch.qos.logback:logback-classic:1.2.3
     |    |    +--- ch.qos.logback:logback-core:1.2.3
     |    |    \--- org.slf4j:slf4j-api:1.7.25
     |    +--- org.apache.logging.log4j:log4j-to-slf4j:2.10.0
     |    |    +--- org.slf4j:slf4j-api:1.7.25
     |    |    \--- org.apache.logging.log4j:log4j-api:2.10.0
     |    \--- org.slf4j:jul-to-slf4j:1.7.25
     |         \--- org.slf4j:slf4j-api:1.7.25
     +--- javax.annotation:javax.annotation-api:1.3.2
     +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     \--- org.yaml:snakeyaml:1.19

runtimeElements - Elements of runtime for main. (n)
No dependencies

runtimeOnly - Runtime only dependencies for source set 'main'. (n)
No dependencies

testAnnotationProcessor - Annotation processors and their dependencies for source set 'test'.
Persisted dependency lock state for configuration 'testAnnotationProcessor'
No dependencies

testCompile - Dependencies for source set 'test' (deprecated, use 'testImplementation ' instead).
Persisted dependency lock state for configuration 'testCompile'
+--- org.springframework.boot:spring-boot-starter -> 2.0.1.RELEASE
|    +--- org.springframework.boot:spring-boot:2.0.1.RELEASE
|    |    +--- org.springframework:spring-core:5.0.5.RELEASE
|    |    |    \--- org.springframework:spring-jcl:5.0.5.RELEASE
|    |    \--- org.springframework:spring-context:5.0.5.RELEASE
|    |         +--- org.springframework:spring-aop:5.0.5.RELEASE
|    |         |    +--- org.springframework:spring-beans:5.0.5.RELEASE
|    |         |    |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         +--- org.springframework:spring-beans:5.0.5.RELEASE (*)
|    |         +--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         \--- org.springframework:spring-expression:5.0.5.RELEASE
|    |              \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    +--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
|    |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
|    +--- org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
|    |    +--- ch.qos.logback:logback-classic:1.2.3
|    |    |    +--- ch.qos.logback:logback-core:1.2.3
|    |    |    \--- org.slf4j:slf4j-api:1.7.25
|    |    +--- org.apache.logging.log4j:log4j-to-slf4j:2.10.0
|    |    |    +--- org.slf4j:slf4j-api:1.7.25
|    |    |    \--- org.apache.logging.log4j:log4j-api:2.10.0
|    |    \--- org.slf4j:jul-to-slf4j:1.7.25
|    |         \--- org.slf4j:slf4j-api:1.7.25
|    +--- javax.annotation:javax.annotation-api:1.3.2
|    +--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    \--- org.yaml:snakeyaml:1.19
\--- org.springframework.boot:spring-boot-starter-test -> 2.0.1.RELEASE
     +--- org.springframework.boot:spring-boot-starter:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-test:2.0.1.RELEASE
     |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-test-autoconfigure:2.0.1.RELEASE
     |    +--- org.springframework.boot:spring-boot-test:2.0.1.RELEASE (*)
     |    \--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE (*)
     +--- com.jayway.jsonpath:json-path:2.4.0
     |    +--- net.minidev:json-smart:2.3
     |    |    \--- net.minidev:accessors-smart:1.2
     |    |         \--- org.ow2.asm:asm:5.0.4
     |    \--- org.slf4j:slf4j-api:1.7.25
     +--- junit:junit:4.12
     |    \--- org.hamcrest:hamcrest-core:1.3
     +--- org.assertj:assertj-core:3.9.1
     +--- org.mockito:mockito-core:2.15.0
     |    +--- net.bytebuddy:byte-buddy:1.7.9 -> 1.7.11
     |    +--- net.bytebuddy:byte-buddy-agent:1.7.9 -> 1.7.11
     |    \--- org.objenesis:objenesis:2.6
     +--- org.hamcrest:hamcrest-core:1.3
     +--- org.hamcrest:hamcrest-library:1.3
     |    \--- org.hamcrest:hamcrest-core:1.3
     +--- org.skyscreamer:jsonassert:1.5.0
     |    \--- com.vaadin.external.google:android-json:0.0.20131108.vaadin1
     +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     +--- org.springframework:spring-test:5.0.5.RELEASE
     |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     \--- org.xmlunit:xmlunit-core:2.5.1

testCompileClasspath - Compile classpath for source set 'test'.
Persisted dependency lock state for configuration 'testCompileClasspath'
+--- org.springframework.boot:spring-boot-starter -> 2.0.1.RELEASE
|    +--- org.springframework.boot:spring-boot:2.0.1.RELEASE
|    |    +--- org.springframework:spring-core:5.0.5.RELEASE
|    |    |    \--- org.springframework:spring-jcl:5.0.5.RELEASE
|    |    \--- org.springframework:spring-context:5.0.5.RELEASE
|    |         +--- org.springframework:spring-aop:5.0.5.RELEASE
|    |         |    +--- org.springframework:spring-beans:5.0.5.RELEASE
|    |         |    |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         +--- org.springframework:spring-beans:5.0.5.RELEASE (*)
|    |         +--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         \--- org.springframework:spring-expression:5.0.5.RELEASE
|    |              \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    +--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
|    |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
|    +--- org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
|    |    +--- ch.qos.logback:logback-classic:1.2.3
|    |    |    +--- ch.qos.logback:logback-core:1.2.3
|    |    |    \--- org.slf4j:slf4j-api:1.7.25
|    |    +--- org.apache.logging.log4j:log4j-to-slf4j:2.10.0
|    |    |    +--- org.slf4j:slf4j-api:1.7.25
|    |    |    \--- org.apache.logging.log4j:log4j-api:2.10.0
|    |    \--- org.slf4j:jul-to-slf4j:1.7.25
|    |         \--- org.slf4j:slf4j-api:1.7.25
|    +--- javax.annotation:javax.annotation-api:1.3.2
|    +--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    \--- org.yaml:snakeyaml:1.19
\--- org.springframework.boot:spring-boot-starter-test -> 2.0.1.RELEASE
     +--- org.springframework.boot:spring-boot-starter:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-test:2.0.1.RELEASE
     |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-test-autoconfigure:2.0.1.RELEASE
     |    +--- org.springframework.boot:spring-boot-test:2.0.1.RELEASE (*)
     |    \--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE (*)
     +--- com.jayway.jsonpath:json-path:2.4.0
     |    +--- net.minidev:json-smart:2.3
     |    |    \--- net.minidev:accessors-smart:1.2
     |    |         \--- org.ow2.asm:asm:5.0.4
     |    \--- org.slf4j:slf4j-api:1.7.25
     +--- junit:junit:4.12
     |    \--- org.hamcrest:hamcrest-core:1.3
     +--- org.assertj:assertj-core:3.9.1
     +--- org.mockito:mockito-core:2.15.0
     |    +--- net.bytebuddy:byte-buddy:1.7.9 -> 1.7.11
     |    +--- net.bytebuddy:byte-buddy-agent:1.7.9 -> 1.7.11
     |    \--- org.objenesis:objenesis:2.6
     +--- org.hamcrest:hamcrest-core:1.3
     +--- org.hamcrest:hamcrest-library:1.3
     |    \--- org.hamcrest:hamcrest-core:1.3
     +--- org.skyscreamer:jsonassert:1.5.0
     |    \--- com.vaadin.external.google:android-json:0.0.20131108.vaadin1
     +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     +--- org.springframework:spring-test:5.0.5.RELEASE
     |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     \--- org.xmlunit:xmlunit-core:2.5.1

testCompileOnly - Compile only dependencies for source set 'test'.
Persisted dependency lock state for configuration 'testCompileOnly'
No dependencies

testImplementation - Implementation only dependencies for source set 'test'. (n)
No dependencies

testRuntime - Runtime dependencies for source set 'test' (deprecated, use 'testRuntimeOnly ' instead).
Persisted dependency lock state for configuration 'testRuntime'
+--- org.springframework.boot:spring-boot-starter -> 2.0.1.RELEASE
|    +--- org.springframework.boot:spring-boot:2.0.1.RELEASE
|    |    +--- org.springframework:spring-core:5.0.5.RELEASE
|    |    |    \--- org.springframework:spring-jcl:5.0.5.RELEASE
|    |    \--- org.springframework:spring-context:5.0.5.RELEASE
|    |         +--- org.springframework:spring-aop:5.0.5.RELEASE
|    |         |    +--- org.springframework:spring-beans:5.0.5.RELEASE
|    |         |    |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         +--- org.springframework:spring-beans:5.0.5.RELEASE (*)
|    |         +--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         \--- org.springframework:spring-expression:5.0.5.RELEASE
|    |              \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    +--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
|    |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
|    +--- org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
|    |    +--- ch.qos.logback:logback-classic:1.2.3
|    |    |    +--- ch.qos.logback:logback-core:1.2.3
|    |    |    \--- org.slf4j:slf4j-api:1.7.25
|    |    +--- org.apache.logging.log4j:log4j-to-slf4j:2.10.0
|    |    |    +--- org.slf4j:slf4j-api:1.7.25
|    |    |    \--- org.apache.logging.log4j:log4j-api:2.10.0
|    |    \--- org.slf4j:jul-to-slf4j:1.7.25
|    |         \--- org.slf4j:slf4j-api:1.7.25
|    +--- javax.annotation:javax.annotation-api:1.3.2
|    +--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    \--- org.yaml:snakeyaml:1.19
\--- org.springframework.boot:spring-boot-starter-test -> 2.0.1.RELEASE
     +--- org.springframework.boot:spring-boot-starter:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-test:2.0.1.RELEASE
     |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-test-autoconfigure:2.0.1.RELEASE
     |    +--- org.springframework.boot:spring-boot-test:2.0.1.RELEASE (*)
     |    \--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE (*)
     +--- com.jayway.jsonpath:json-path:2.4.0
     |    +--- net.minidev:json-smart:2.3
     |    |    \--- net.minidev:accessors-smart:1.2
     |    |         \--- org.ow2.asm:asm:5.0.4
     |    \--- org.slf4j:slf4j-api:1.7.25
     +--- junit:junit:4.12
     |    \--- org.hamcrest:hamcrest-core:1.3
     +--- org.assertj:assertj-core:3.9.1
     +--- org.mockito:mockito-core:2.15.0
     |    +--- net.bytebuddy:byte-buddy:1.7.9 -> 1.7.11
     |    +--- net.bytebuddy:byte-buddy-agent:1.7.9 -> 1.7.11
     |    \--- org.objenesis:objenesis:2.6
     +--- org.hamcrest:hamcrest-core:1.3
     +--- org.hamcrest:hamcrest-library:1.3
     |    \--- org.hamcrest:hamcrest-core:1.3
     +--- org.skyscreamer:jsonassert:1.5.0
     |    \--- com.vaadin.external.google:android-json:0.0.20131108.vaadin1
     +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     +--- org.springframework:spring-test:5.0.5.RELEASE
     |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     \--- org.xmlunit:xmlunit-core:2.5.1

testRuntimeClasspath - Runtime classpath of source set 'test'.
Persisted dependency lock state for configuration 'testRuntimeClasspath'
+--- org.springframework.boot:spring-boot-starter -> 2.0.1.RELEASE
|    +--- org.springframework.boot:spring-boot:2.0.1.RELEASE
|    |    +--- org.springframework:spring-core:5.0.5.RELEASE
|    |    |    \--- org.springframework:spring-jcl:5.0.5.RELEASE
|    |    \--- org.springframework:spring-context:5.0.5.RELEASE
|    |         +--- org.springframework:spring-aop:5.0.5.RELEASE
|    |         |    +--- org.springframework:spring-beans:5.0.5.RELEASE
|    |         |    |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         +--- org.springframework:spring-beans:5.0.5.RELEASE (*)
|    |         +--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    |         \--- org.springframework:spring-expression:5.0.5.RELEASE
|    |              \--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    +--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
|    |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
|    +--- org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
|    |    +--- ch.qos.logback:logback-classic:1.2.3
|    |    |    +--- ch.qos.logback:logback-core:1.2.3
|    |    |    \--- org.slf4j:slf4j-api:1.7.25
|    |    +--- org.apache.logging.log4j:log4j-to-slf4j:2.10.0
|    |    |    +--- org.slf4j:slf4j-api:1.7.25
|    |    |    \--- org.apache.logging.log4j:log4j-api:2.10.0
|    |    \--- org.slf4j:jul-to-slf4j:1.7.25
|    |         \--- org.slf4j:slf4j-api:1.7.25
|    +--- javax.annotation:javax.annotation-api:1.3.2
|    +--- org.springframework:spring-core:5.0.5.RELEASE (*)
|    \--- org.yaml:snakeyaml:1.19
\--- org.springframework.boot:spring-boot-starter-test -> 2.0.1.RELEASE
     +--- org.springframework.boot:spring-boot-starter:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-test:2.0.1.RELEASE
     |    \--- org.springframework.boot:spring-boot:2.0.1.RELEASE (*)
     +--- org.springframework.boot:spring-boot-test-autoconfigure:2.0.1.RELEASE
     |    +--- org.springframework.boot:spring-boot-test:2.0.1.RELEASE (*)
     |    \--- org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE (*)
     +--- com.jayway.jsonpath:json-path:2.4.0
     |    +--- net.minidev:json-smart:2.3
     |    |    \--- net.minidev:accessors-smart:1.2
     |    |         \--- org.ow2.asm:asm:5.0.4
     |    \--- org.slf4j:slf4j-api:1.7.25
     +--- junit:junit:4.12
     |    \--- org.hamcrest:hamcrest-core:1.3
     +--- org.assertj:assertj-core:3.9.1
     +--- org.mockito:mockito-core:2.15.0
     |    +--- net.bytebuddy:byte-buddy:1.7.9 -> 1.7.11
     |    +--- net.bytebuddy:byte-buddy-agent:1.7.9 -> 1.7.11
     |    \--- org.objenesis:objenesis:2.6
     +--- org.hamcrest:hamcrest-core:1.3
     +--- org.hamcrest:hamcrest-library:1.3
     |    \--- org.hamcrest:hamcrest-core:1.3
     +--- org.skyscreamer:jsonassert:1.5.0
     |    \--- com.vaadin.external.google:android-json:0.0.20131108.vaadin1
     +--- org.springframework:spring-core:5.0.5.RELEASE (*)
     +--- org.springframework:spring-test:5.0.5.RELEASE
     |    \--- org.springframework:spring-core:5.0.5.RELEASE (*)
     \--- org.xmlunit:xmlunit-core:2.5.1

testRuntimeOnly - Runtime only dependencies for source set 'test'. (n)
No dependencies

(*) - dependencies omitted (listed previously)

A web-based, searchable dependency report is available by adding the --scan option.

BUILD SUCCESSFUL in 2s
1 actionable task: 1 executed
```

具体的にどのライブラリがどのようにバージョンアップされたかが履歴に残り、管理可能になる。

```
diff --git build.gradle build.gradle
index a76663a..0750b81 100644
--- build.gradle
+++ build.gradle
@@ -1,6 +1,6 @@
 buildscript {
     ext {
-        springBootVersion = '2.0.0.RELEASE'
+        springBootVersion = '2.0.1.RELEASE'
     }
     repositories {
         mavenCentral()
diff --git gradle/dependency-locks/compile.lockfile gradle/dependency-locks/compile.lockfile
index 6ddc26f..16c6f2a 100644
--- gradle/dependency-locks/compile.lockfile
+++ gradle/dependency-locks/compile.lockfile
@@ -8,14 +8,14 @@ org.apache.logging.log4j:log4j-api:2.10.0
 org.apache.logging.log4j:log4j-to-slf4j:2.10.0
 org.slf4j:jul-to-slf4j:1.7.25
 org.slf4j:slf4j-api:1.7.25
-org.springframework.boot:spring-boot-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-logging:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter:2.0.0.RELEASE
-org.springframework.boot:spring-boot:2.0.0.RELEASE
-org.springframework:spring-aop:5.0.4.RELEASE
-org.springframework:spring-beans:5.0.4.RELEASE
-org.springframework:spring-context:5.0.4.RELEASE
-org.springframework:spring-core:5.0.4.RELEASE
-org.springframework:spring-expression:5.0.4.RELEASE
-org.springframework:spring-jcl:5.0.4.RELEASE
+org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter:2.0.1.RELEASE
+org.springframework.boot:spring-boot:2.0.1.RELEASE
+org.springframework:spring-aop:5.0.5.RELEASE
+org.springframework:spring-beans:5.0.5.RELEASE
+org.springframework:spring-context:5.0.5.RELEASE
+org.springframework:spring-core:5.0.5.RELEASE
+org.springframework:spring-expression:5.0.5.RELEASE
+org.springframework:spring-jcl:5.0.5.RELEASE
 org.yaml:snakeyaml:1.19
diff --git gradle/dependency-locks/compileClasspath.lockfile gradle/dependency-locks/compileClasspath.lockfile
index 6ddc26f..16c6f2a 100644
--- gradle/dependency-locks/compileClasspath.lockfile
+++ gradle/dependency-locks/compileClasspath.lockfile
@@ -8,14 +8,14 @@ org.apache.logging.log4j:log4j-api:2.10.0
 org.apache.logging.log4j:log4j-to-slf4j:2.10.0
 org.slf4j:jul-to-slf4j:1.7.25
 org.slf4j:slf4j-api:1.7.25
-org.springframework.boot:spring-boot-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-logging:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter:2.0.0.RELEASE
-org.springframework.boot:spring-boot:2.0.0.RELEASE
-org.springframework:spring-aop:5.0.4.RELEASE
-org.springframework:spring-beans:5.0.4.RELEASE
-org.springframework:spring-context:5.0.4.RELEASE
-org.springframework:spring-core:5.0.4.RELEASE
-org.springframework:spring-expression:5.0.4.RELEASE
-org.springframework:spring-jcl:5.0.4.RELEASE
+org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter:2.0.1.RELEASE
+org.springframework.boot:spring-boot:2.0.1.RELEASE
+org.springframework:spring-aop:5.0.5.RELEASE
+org.springframework:spring-beans:5.0.5.RELEASE
+org.springframework:spring-context:5.0.5.RELEASE
+org.springframework:spring-core:5.0.5.RELEASE
+org.springframework:spring-expression:5.0.5.RELEASE
+org.springframework:spring-jcl:5.0.5.RELEASE
 org.yaml:snakeyaml:1.19
diff --git gradle/dependency-locks/default.lockfile gradle/dependency-locks/default.lockfile
index 6ddc26f..16c6f2a 100644
--- gradle/dependency-locks/default.lockfile
+++ gradle/dependency-locks/default.lockfile
@@ -8,14 +8,14 @@ org.apache.logging.log4j:log4j-api:2.10.0
 org.apache.logging.log4j:log4j-to-slf4j:2.10.0
 org.slf4j:jul-to-slf4j:1.7.25
 org.slf4j:slf4j-api:1.7.25
-org.springframework.boot:spring-boot-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-logging:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter:2.0.0.RELEASE
-org.springframework.boot:spring-boot:2.0.0.RELEASE
-org.springframework:spring-aop:5.0.4.RELEASE
-org.springframework:spring-beans:5.0.4.RELEASE
-org.springframework:spring-context:5.0.4.RELEASE
-org.springframework:spring-core:5.0.4.RELEASE
-org.springframework:spring-expression:5.0.4.RELEASE
-org.springframework:spring-jcl:5.0.4.RELEASE
+org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter:2.0.1.RELEASE
+org.springframework.boot:spring-boot:2.0.1.RELEASE
+org.springframework:spring-aop:5.0.5.RELEASE
+org.springframework:spring-beans:5.0.5.RELEASE
+org.springframework:spring-context:5.0.5.RELEASE
+org.springframework:spring-core:5.0.5.RELEASE
+org.springframework:spring-expression:5.0.5.RELEASE
+org.springframework:spring-jcl:5.0.5.RELEASE
 org.yaml:snakeyaml:1.19
diff --git gradle/dependency-locks/runtime.lockfile gradle/dependency-locks/runtime.lockfile
index 6ddc26f..16c6f2a 100644
--- gradle/dependency-locks/runtime.lockfile
+++ gradle/dependency-locks/runtime.lockfile
@@ -8,14 +8,14 @@ org.apache.logging.log4j:log4j-api:2.10.0
 org.apache.logging.log4j:log4j-to-slf4j:2.10.0
 org.slf4j:jul-to-slf4j:1.7.25
 org.slf4j:slf4j-api:1.7.25
-org.springframework.boot:spring-boot-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-logging:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter:2.0.0.RELEASE
-org.springframework.boot:spring-boot:2.0.0.RELEASE
-org.springframework:spring-aop:5.0.4.RELEASE
-org.springframework:spring-beans:5.0.4.RELEASE
-org.springframework:spring-context:5.0.4.RELEASE
-org.springframework:spring-core:5.0.4.RELEASE
-org.springframework:spring-expression:5.0.4.RELEASE
-org.springframework:spring-jcl:5.0.4.RELEASE
+org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter:2.0.1.RELEASE
+org.springframework.boot:spring-boot:2.0.1.RELEASE
+org.springframework:spring-aop:5.0.5.RELEASE
+org.springframework:spring-beans:5.0.5.RELEASE
+org.springframework:spring-context:5.0.5.RELEASE
+org.springframework:spring-core:5.0.5.RELEASE
+org.springframework:spring-expression:5.0.5.RELEASE
+org.springframework:spring-jcl:5.0.5.RELEASE
 org.yaml:snakeyaml:1.19
diff --git gradle/dependency-locks/runtimeClasspath.lockfile gradle/dependency-locks/runtimeClasspath.lockfile
index 6ddc26f..16c6f2a 100644
--- gradle/dependency-locks/runtimeClasspath.lockfile
+++ gradle/dependency-locks/runtimeClasspath.lockfile
@@ -8,14 +8,14 @@ org.apache.logging.log4j:log4j-api:2.10.0
 org.apache.logging.log4j:log4j-to-slf4j:2.10.0
 org.slf4j:jul-to-slf4j:1.7.25
 org.slf4j:slf4j-api:1.7.25
-org.springframework.boot:spring-boot-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-logging:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter:2.0.0.RELEASE
-org.springframework.boot:spring-boot:2.0.0.RELEASE
-org.springframework:spring-aop:5.0.4.RELEASE
-org.springframework:spring-beans:5.0.4.RELEASE
-org.springframework:spring-context:5.0.4.RELEASE
-org.springframework:spring-core:5.0.4.RELEASE
-org.springframework:spring-expression:5.0.4.RELEASE
-org.springframework:spring-jcl:5.0.4.RELEASE
+org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter:2.0.1.RELEASE
+org.springframework.boot:spring-boot:2.0.1.RELEASE
+org.springframework:spring-aop:5.0.5.RELEASE
+org.springframework:spring-beans:5.0.5.RELEASE
+org.springframework:spring-context:5.0.5.RELEASE
+org.springframework:spring-core:5.0.5.RELEASE
+org.springframework:spring-expression:5.0.5.RELEASE
+org.springframework:spring-jcl:5.0.5.RELEASE
 org.yaml:snakeyaml:1.19
diff --git gradle/dependency-locks/testCompile.lockfile gradle/dependency-locks/testCompile.lockfile
index 4532491..29d98af 100644
--- gradle/dependency-locks/testCompile.lockfile
+++ gradle/dependency-locks/testCompile.lockfile
@@ -7,8 +7,8 @@ com.jayway.jsonpath:json-path:2.4.0
 com.vaadin.external.google:android-json:0.0.20131108.vaadin1
 javax.annotation:javax.annotation-api:1.3.2
 junit:junit:4.12
-net.bytebuddy:byte-buddy-agent:1.7.10
-net.bytebuddy:byte-buddy:1.7.10
+net.bytebuddy:byte-buddy-agent:1.7.11
+net.bytebuddy:byte-buddy:1.7.11
 net.minidev:accessors-smart:1.2
 net.minidev:json-smart:2.3
 org.apache.logging.log4j:log4j-api:2.10.0
@@ -22,19 +22,19 @@ org.ow2.asm:asm:5.0.4
 org.skyscreamer:jsonassert:1.5.0
 org.slf4j:jul-to-slf4j:1.7.25
 org.slf4j:slf4j-api:1.7.25
-org.springframework.boot:spring-boot-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-logging:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-test:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter:2.0.0.RELEASE
-org.springframework.boot:spring-boot-test-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-test:2.0.0.RELEASE
-org.springframework.boot:spring-boot:2.0.0.RELEASE
-org.springframework:spring-aop:5.0.4.RELEASE
-org.springframework:spring-beans:5.0.4.RELEASE
-org.springframework:spring-context:5.0.4.RELEASE
-org.springframework:spring-core:5.0.4.RELEASE
-org.springframework:spring-expression:5.0.4.RELEASE
-org.springframework:spring-jcl:5.0.4.RELEASE
-org.springframework:spring-test:5.0.4.RELEASE
+org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-test:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter:2.0.1.RELEASE
+org.springframework.boot:spring-boot-test-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-test:2.0.1.RELEASE
+org.springframework.boot:spring-boot:2.0.1.RELEASE
+org.springframework:spring-aop:5.0.5.RELEASE
+org.springframework:spring-beans:5.0.5.RELEASE
+org.springframework:spring-context:5.0.5.RELEASE
+org.springframework:spring-core:5.0.5.RELEASE
+org.springframework:spring-expression:5.0.5.RELEASE
+org.springframework:spring-jcl:5.0.5.RELEASE
+org.springframework:spring-test:5.0.5.RELEASE
 org.xmlunit:xmlunit-core:2.5.1
 org.yaml:snakeyaml:1.19
diff --git gradle/dependency-locks/testCompileClasspath.lockfile gradle/dependency-locks/testCompileClasspath.lockfile
index 4532491..29d98af 100644
--- gradle/dependency-locks/testCompileClasspath.lockfile
+++ gradle/dependency-locks/testCompileClasspath.lockfile
@@ -7,8 +7,8 @@ com.jayway.jsonpath:json-path:2.4.0
 com.vaadin.external.google:android-json:0.0.20131108.vaadin1
 javax.annotation:javax.annotation-api:1.3.2
 junit:junit:4.12
-net.bytebuddy:byte-buddy-agent:1.7.10
-net.bytebuddy:byte-buddy:1.7.10
+net.bytebuddy:byte-buddy-agent:1.7.11
+net.bytebuddy:byte-buddy:1.7.11
 net.minidev:accessors-smart:1.2
 net.minidev:json-smart:2.3
 org.apache.logging.log4j:log4j-api:2.10.0
@@ -22,19 +22,19 @@ org.ow2.asm:asm:5.0.4
 org.skyscreamer:jsonassert:1.5.0
 org.slf4j:jul-to-slf4j:1.7.25
 org.slf4j:slf4j-api:1.7.25
-org.springframework.boot:spring-boot-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-logging:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-test:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter:2.0.0.RELEASE
-org.springframework.boot:spring-boot-test-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-test:2.0.0.RELEASE
-org.springframework.boot:spring-boot:2.0.0.RELEASE
-org.springframework:spring-aop:5.0.4.RELEASE
-org.springframework:spring-beans:5.0.4.RELEASE
-org.springframework:spring-context:5.0.4.RELEASE
-org.springframework:spring-core:5.0.4.RELEASE
-org.springframework:spring-expression:5.0.4.RELEASE
-org.springframework:spring-jcl:5.0.4.RELEASE
-org.springframework:spring-test:5.0.4.RELEASE
+org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-test:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter:2.0.1.RELEASE
+org.springframework.boot:spring-boot-test-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-test:2.0.1.RELEASE
+org.springframework.boot:spring-boot:2.0.1.RELEASE
+org.springframework:spring-aop:5.0.5.RELEASE
+org.springframework:spring-beans:5.0.5.RELEASE
+org.springframework:spring-context:5.0.5.RELEASE
+org.springframework:spring-core:5.0.5.RELEASE
+org.springframework:spring-expression:5.0.5.RELEASE
+org.springframework:spring-jcl:5.0.5.RELEASE
+org.springframework:spring-test:5.0.5.RELEASE
 org.xmlunit:xmlunit-core:2.5.1
 org.yaml:snakeyaml:1.19
diff --git gradle/dependency-locks/testRuntime.lockfile gradle/dependency-locks/testRuntime.lockfile
index 4532491..29d98af 100644
--- gradle/dependency-locks/testRuntime.lockfile
+++ gradle/dependency-locks/testRuntime.lockfile
@@ -7,8 +7,8 @@ com.jayway.jsonpath:json-path:2.4.0
 com.vaadin.external.google:android-json:0.0.20131108.vaadin1
 javax.annotation:javax.annotation-api:1.3.2
 junit:junit:4.12
-net.bytebuddy:byte-buddy-agent:1.7.10
-net.bytebuddy:byte-buddy:1.7.10
+net.bytebuddy:byte-buddy-agent:1.7.11
+net.bytebuddy:byte-buddy:1.7.11
 net.minidev:accessors-smart:1.2
 net.minidev:json-smart:2.3
 org.apache.logging.log4j:log4j-api:2.10.0
@@ -22,19 +22,19 @@ org.ow2.asm:asm:5.0.4
 org.skyscreamer:jsonassert:1.5.0
 org.slf4j:jul-to-slf4j:1.7.25
 org.slf4j:slf4j-api:1.7.25
-org.springframework.boot:spring-boot-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-logging:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-test:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter:2.0.0.RELEASE
-org.springframework.boot:spring-boot-test-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-test:2.0.0.RELEASE
-org.springframework.boot:spring-boot:2.0.0.RELEASE
-org.springframework:spring-aop:5.0.4.RELEASE
-org.springframework:spring-beans:5.0.4.RELEASE
-org.springframework:spring-context:5.0.4.RELEASE
-org.springframework:spring-core:5.0.4.RELEASE
-org.springframework:spring-expression:5.0.4.RELEASE
-org.springframework:spring-jcl:5.0.4.RELEASE
-org.springframework:spring-test:5.0.4.RELEASE
+org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-test:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter:2.0.1.RELEASE
+org.springframework.boot:spring-boot-test-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-test:2.0.1.RELEASE
+org.springframework.boot:spring-boot:2.0.1.RELEASE
+org.springframework:spring-aop:5.0.5.RELEASE
+org.springframework:spring-beans:5.0.5.RELEASE
+org.springframework:spring-context:5.0.5.RELEASE
+org.springframework:spring-core:5.0.5.RELEASE
+org.springframework:spring-expression:5.0.5.RELEASE
+org.springframework:spring-jcl:5.0.5.RELEASE
+org.springframework:spring-test:5.0.5.RELEASE
 org.xmlunit:xmlunit-core:2.5.1
 org.yaml:snakeyaml:1.19
diff --git gradle/dependency-locks/testRuntimeClasspath.lockfile gradle/dependency-locks/testRuntimeClasspath.lockfile
index 4532491..29d98af 100644
--- gradle/dependency-locks/testRuntimeClasspath.lockfile
+++ gradle/dependency-locks/testRuntimeClasspath.lockfile
@@ -7,8 +7,8 @@ com.jayway.jsonpath:json-path:2.4.0
 com.vaadin.external.google:android-json:0.0.20131108.vaadin1
 javax.annotation:javax.annotation-api:1.3.2
 junit:junit:4.12
-net.bytebuddy:byte-buddy-agent:1.7.10
-net.bytebuddy:byte-buddy:1.7.10
+net.bytebuddy:byte-buddy-agent:1.7.11
+net.bytebuddy:byte-buddy:1.7.11
 net.minidev:accessors-smart:1.2
 net.minidev:json-smart:2.3
 org.apache.logging.log4j:log4j-api:2.10.0
@@ -22,19 +22,19 @@ org.ow2.asm:asm:5.0.4
 org.skyscreamer:jsonassert:1.5.0
 org.slf4j:jul-to-slf4j:1.7.25
 org.slf4j:slf4j-api:1.7.25
-org.springframework.boot:spring-boot-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-logging:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter-test:2.0.0.RELEASE
-org.springframework.boot:spring-boot-starter:2.0.0.RELEASE
-org.springframework.boot:spring-boot-test-autoconfigure:2.0.0.RELEASE
-org.springframework.boot:spring-boot-test:2.0.0.RELEASE
-org.springframework.boot:spring-boot:2.0.0.RELEASE
-org.springframework:spring-aop:5.0.4.RELEASE
-org.springframework:spring-beans:5.0.4.RELEASE
-org.springframework:spring-context:5.0.4.RELEASE
-org.springframework:spring-core:5.0.4.RELEASE
-org.springframework:spring-expression:5.0.4.RELEASE
-org.springframework:spring-jcl:5.0.4.RELEASE
-org.springframework:spring-test:5.0.4.RELEASE
+org.springframework.boot:spring-boot-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-logging:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter-test:2.0.1.RELEASE
+org.springframework.boot:spring-boot-starter:2.0.1.RELEASE
+org.springframework.boot:spring-boot-test-autoconfigure:2.0.1.RELEASE
+org.springframework.boot:spring-boot-test:2.0.1.RELEASE
+org.springframework.boot:spring-boot:2.0.1.RELEASE
+org.springframework:spring-aop:5.0.5.RELEASE
+org.springframework:spring-beans:5.0.5.RELEASE
+org.springframework:spring-context:5.0.5.RELEASE
+org.springframework:spring-core:5.0.5.RELEASE
+org.springframework:spring-expression:5.0.5.RELEASE
+org.springframework:spring-jcl:5.0.5.RELEASE
+org.springframework:spring-test:5.0.5.RELEASE
 org.xmlunit:xmlunit-core:2.5.1
 org.yaml:snakeyaml:1.19
```

