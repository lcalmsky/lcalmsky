최근에 스프링 부트 프로젝트를 생성하여 이전 글 처럼 querydsl을 설정하다보면 정상 동작하지 않습니다.

따라서 build.gradle 파일을 아래 처럼 변경해주어야 합니다.

```groovy
buildscript {
    ext {
        queryDslVersion = "5.0.0"
    }
}
plugins {
    id 'org.springframework.boot' version '2.7.2' // 2.6 이상일 때 적용
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
    id 'java'
}
group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'
configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}
repositories {
    mavenCentral()
}
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    
    // querydsl dependencies
    implementation "com.querydsl:querydsl-jpa:${queryDslVersion}" 
    annotationProcessor "com.querydsl:querydsl-apt:${queryDslVersion}"
    
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testCompileOnly 'org.projectlombok:lombok' testAnnotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
    useJUnitPlatform()
}

// script for querydsl
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}
sourceSets {
    main.java.srcDir querydslDir
}
configurations {
    querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
```