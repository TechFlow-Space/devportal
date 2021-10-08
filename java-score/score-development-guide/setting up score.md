## Setting up score

After the grade is initialized, open the folder in an IDE. Intellij is the preferred IDE.

Inside your project folder there will be file called `build.gradle`.
The `build.gradle` is a build script which defines a project and its task. It is written in Groovy language.

At first include the version of the score you are writing:

```groovy
version = '0.1.0'
```
Now include the dependencies. The dependencies should have a repository from where the code is implemented, 
testImplementation and testRuntimeOnly.

The repository implementation is written following the sequence group: group name, name: library name , 
version: version of the library. It can be found on  [maven repository](https://mvnrepository.com/artifact/foundation.icon/javaee-api/0.9.0) or using
Intellij IDE. ‘Generate’ in intellij we can load the maven repository directly to the dependency.

```groovy
dependencies{
    implementation 'foundation.icon:javaee-api:0.9.0'
    testImplementation 'org.junit.jupiter:junit-jupiter:5.7.2'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.6.0'
}
```

You need to optimize your jar bundle before deploying to the ICON network. It involves some pre-processing to 
ensure the actual development is successful. It contains mainClassName and archiveBaseName
```groovy
optimizedJar {
    mainClassName = 'com.sample.score.App'
    archivesBaseName= 'app'
}

```

Next is the deployJar. It contains the endpoint to where you want to deploy your score to.
The keystore credentials are passed here to deploy the score. The information about the keystoreName and 
keystorePass is passed through gradle.properties. By doing so we can save our credentials' info from others when 
an optimized jar is passed on.
```groovy
deployJar {
    endpoints {
        sejong {
            uri = 'https://sejong.net.solidwallet.io/api/v3'
            nid = 0x53
        }
        local {
            uri = 'http://localhost:9082/api/v3'
            nid = 0x3
        }
    }
    keystore = rootProject.hasProperty('keystoreName') ? "$keystoreName" : ''
    password = rootProject.hasProperty('keystorePass') ? "$keystorePass" : ''
    parameters{
        arg('name', 'Groovy')
    }
}
```

Inside deployJar we pass on the parameters required on deploying the score at first.

Inside the build gradle we have tasks like buildscript,subprojects which are required in every score you create. 
This 'build.gradle' can be written in the root folder.

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'foundation.icon:gradle-javaee-plugin:0.7.8'
    }
}

subprojects {
    repositories {
        mavenCentral()
    }

    apply plugin: 'java'
    apply plugin: 'foundation.icon.javaee'

    java {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }

    // need to add this option to retrieve formal parameter names
    compileJava {
        options.compilerArgs += ['-parameters']
    }
}
```


We can use command line to build the gradle
```groovy
./gradlew build
```


To create optimized jar using command line

```groovy
./gradlew optimizeJar
```

To deploy the score to Sejong using command line

```groovy
./gradlew deployToSejong
```

``` 
Succeeded to deploy: 0x6232d3e2436e8dc1b04dc1348f2a0bf7bdf1ea2ffc1006baafcea5f0ef48ca7a
SCORE address: cx6d01d7531b35e0d4d04ce83b8ee63d81113fada4

BUILD SUCCESSFUL in 8s
1 actionable task: 1 executed

```
