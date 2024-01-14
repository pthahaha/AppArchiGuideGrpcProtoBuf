# gRPC ProtoBuf
#### 이 친구는 service와 message를 만들어 마치 DTO(Data Transfer Object)처럼 데이터 전송을 할 때 인터페이스를 맞출수 있는 기준이 됩니다.
#### IDL이라는 Interface Definition Language 구문을 통해 작성 합니다.
```
syntax = "proto3";

package com.kakao.www.product;

service ProductInfo {
  rpc addProduct(Product) returns (ProductID);
  rpc getProduct(ProductID) returns (Product);
}

message Product {
  string id = 1;
  string name = 2;
  string description = 3;
  int64 price = 4;
}

message ProductID {
  string value = 1;
}

```

#### 위치는 main밑에 java dir과 같은 위치에 proto dir을 만들어주고 .proto의 확장자를 사용하여 proto buf파일을 만들어줍니다.
#### 경로가 다르면 빌드 후에 소스생성이 되지 않습니다.

### build.gradle 내용
중요하게 봐야할 부분은 id 'maven-publish'과 publishing { ... } 입니다.
말그대로 다른곳에서 사용할 수 있도록 publishing 해주는 부분 입니다.
또한 netty는 네트워크 통신부분이며 성능을 위해 디펜던시를 걸어줍니다. 걸지 않으면 okHttp가 발동합니다.
stub관련도 사용되기 때문에 걸어줘야 합니다.

```
    // grpc 관련 셋팅
    // https://mvnrepository.com/artifact/io.grpc/grpc-netty
    implementation 'io.grpc:grpc-netty:1.60.0'

    // https://mvnrepository.com/artifact/io.grpc/grpc-protobuf
    implementation 'io.grpc:grpc-protobuf:1.60.0'

    // https://mvnrepository.com/artifact/io.grpc/grpc-stub
    implementation 'io.grpc:grpc-stub:1.60.0'
```

protobuf { ... } 부분도 필요 합니다. Protoc를 하는데 있어 자바코드로 만들어주는 부분 입니다.

```
plugins {
    id 'java-library'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
    id "com.google.protobuf" version "0.9.4"
    id 'maven-publish'
}
 
group = 'com.kakao.www'
version = '0.0.1-SNAPSHOT'
 
java {
    sourceCompatibility = '21'
}
 
configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}
 
repositories {
    mavenCentral()
}
 
 
 
// 프로젝트 의존성
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
 
    // grpc 관련 셋팅
    // https://mvnrepository.com/artifact/io.grpc/grpc-netty
    implementation 'io.grpc:grpc-netty:1.60.0'
 
    // https://mvnrepository.com/artifact/io.grpc/grpc-protobuf
    implementation 'io.grpc:grpc-protobuf:1.60.0'
 
    // https://mvnrepository.com/artifact/io.grpc/grpc-stub
    implementation 'io.grpc:grpc-stub:1.60.0'
 
    implementation 'javax.annotation:javax.annotation-api:1.3.2'
 
}
 
 
protobuf {
    protoc {
        artifact = 'com.google.protobuf:protoc:3.25.1'
    }
    plugins {
        grpc {
            artifact = 'io.grpc:protoc-gen-grpc-java:1.60.0'
        }
    }
    generateProtoTasks {
        all()*.plugins {
            grpc {}
        }
    }
}
 
 
 
tasks.named('test') {
    useJUnitPlatform()
}
 
 
// id 'maven-publish'를 선언해줘야 오류나지 않음.
publishing {
    publications {
        maven(MavenPublication) {
            groupId = 'com.kakao.www'
            artifactId = 'grpc-test'
            version = '0.0.1-SNAPSHOT'
            from components.java
        }
    }
}

```

### error 체크
아래와 같은 오류가 난다면
implementation 'javax.annotation:javax.annotation-api:1.3.2' 추가 
Java 어노테이션을 사용하기 위한 필수 라이브러리입니다. 이 라이브러리를 추가하지 않으면 Java 어노테이션을 사용할 수 없습니다.

```
/Users/teri.epi/Workspaces/AppArchiGuideGrpcProtoBuf/build/generated/source/proto/main/grpc/com/kakao/www/product/ProductInfoGrpc.java:7: error: cannot find symbol
@javax.annotation.Generated(
                 ^
  symbol:   class Generated
  location: package javax.annotation
```

### 소스 생성
gradle build or generateProto를 실행해주면 소스가 생성됩니다.
생성되는 위치는 build/generated/source/proto/main

### 로컬 메이븐에 배포
gradle publishToMavenLocal을 통해서 publish해줍니다.
회사나 사용하는 넥서스같은 곳이 있다면 거기에 넣어주면 되는데 없어서 로컬기반으로 해야할 때 사용 합니다.

### next step
client와 server쪽에 이제 proto buf 프로젝트를 걸어주고 사용하면 됩니다.
- client : https://github.com/pthahaha/ApplicationArchitectureguide 
- server : https://github.com/pthahaha/ApplicationArchitectureGuide-grpcServer
