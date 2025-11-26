# Jenkins Trouble Shooting

## 1. SonarQube JavaSensor 분석 실패

### 증상
Jenkins에서 SonarQube 분석 실행 시 JavaSensor 단계에서 EXECUTION FAILURE 발생

### 에러 로그
```
12:38:48.525 INFO: Sensor JavaSensor [java]
12:38:48.527 INFO: Configured Java source version (sonar.java.source): none
12:38:48.530 INFO: JavaClasspath initialization
12:38:48.530 DEBUG: Property 'sonar.java.jdkHome' resolved with:
[]
12:38:48.530 DEBUG: Property 'sonar.java.libraries' resolved with:
[]
12:38:48.554 INFO: ------------------------------------------------------------------------
12:38:48.554 INFO: EXECUTION FAILURE
```

### 원인 분석

| 설정 항목 | 상태 | 문제점 |
|-----------|------|--------|
| `sonar.java.source` | none | Java 소스 버전 미설정 |
| `sonar.java.jdkHome` | [] (빈 배열) | JDK 경로 미설정 |
| `sonar.java.libraries` | [] (빈 배열) | 라이브러리 경로 미설정 |

JavaClasspath 초기화 시 필수 Java 설정들이 누락되어 분석이 실패함.

### 해결 방법

#### 방법 1: sonar-project.properties 파일 생성

프로젝트 **루트 디렉토리**에 `sonar-project.properties` 파일을 생성:

```
프로젝트-루트/
├── sonar-project.properties  ← 여기에 생성
├── src/
├── pom.xml
└── ...
```

파일 내용:
```properties
# 프로젝트 기본 정보
sonar.projectKey=your-project-key
sonar.projectName=Your Project Name
sonar.projectVersion=1.0

# Java 버전 설정 (필수)
sonar.java.source=11
sonar.java.target=11

# 소스 디렉토리
sonar.sources=src/main/java
sonar.tests=src/test/java

# 바이너리 디렉토리 (컴파일된 .class 파일 위치)
sonar.java.binaries=target/classes
sonar.java.test.binaries=target/test-classes

# 라이브러리 경로 (선택사항)
sonar.java.libraries=target/dependency/*.jar

# 인코딩
sonar.sourceEncoding=UTF-8
```

#### 방법 2: Maven pom.xml 설정

```xml
<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <sonar.java.source>11</sonar.java.source>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.sonarsource.scanner.maven</groupId>
            <artifactId>sonar-maven-plugin</artifactId>
            <version>3.9.1.2184</version>
        </plugin>
    </plugins>
</build>
```

Maven 실행 명령:
```bash
mvn clean verify sonar:sonar \
  -Dsonar.projectKey=your-project \
  -Dsonar.host.url=http://your-sonarqube-server \
  -Dsonar.login=your-token
```

#### 방법 3: Gradle build.gradle 설정

```groovy
plugins {
    id "org.sonarqube" version "4.0.0.2929"
}

sonarqube {
    properties {
        property "sonar.projectKey", "your-project"
        property "sonar.java.source", "11"
        property "sonar.java.binaries", "build/classes"
    }
}
```

Gradle 실행 명령:
```bash
./gradlew sonarqube \
  -Dsonar.host.url=http://your-sonarqube-server \
  -Dsonar.login=your-token
```

#### 방법 4: Jenkinsfile에서 JDK 설정

```groovy
pipeline {
    agent any
    
    tools {
        jdk 'JDK11'  // Jenkins에 등록된 JDK 이름
        maven 'Maven3'
    }
    
    environment {
        JAVA_HOME = tool name: 'JDK11', type: 'jdk'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.java.source=11 \
                          -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }
    }
}
```

### 체크리스트

- [ ] Java 소스 버전(`sonar.java.source`)이 설정되어 있는지 확인
- [ ] 프로젝트가 빌드되어 `.class` 파일이 생성되었는지 확인
- [ ] `sonar.java.binaries` 경로가 실제 컴파일된 클래스 위치와 일치하는지 확인
- [ ] Jenkins에서 JDK가 올바르게 설정되어 있는지 확인
- [ ] SonarQube 서버 연결 정보(URL, 토큰)가 올바른지 확인

### 참고 사항

- Maven/Gradle을 사용하는 경우 `sonar-project.properties` 없이도 빌드 도구 설정만으로 충분
- 두 방법을 혼용할 경우 설정 충돌 가능성 있으므로 한 가지 방법 권장
- SonarQube 분석 전 반드시 프로젝트 빌드(컴파일) 단계 선행 필요

---

*최종 수정: 2025-01-XX*
*태그: #jenkins #sonarqube #troubleshooting #java*


---

## 2. SonarQube JavaSensor 실패 - 비 Java 프로젝트인 경우

### 증상
Java 프로젝트가 아닌데 SonarQube가 JavaSensor를 실행하려고 시도하여 실패

### 환경 확인

**시스템 Java 환경:**
```bash
$ java -version
openjdk version "11.0.28" 2025-07-15
OpenJDK Runtime Environment (build 11.0.28+6-post-Ubuntu-1ubuntu122.04.1)
OpenJDK 64-Bit Server VM (build 11.0.28+6-post-Ubuntu-1ubuntu122.04.1, mixed mode, sharing)

$ javac -version
javac 11.0.28

$ echo $JAVA_HOME
(비어있음 - 미설정)

$ readlink -f $(which java)
/usr/lib/jvm/java-11-openjdk-amd64/bin/java

$ ls -la /usr/lib/jvm/
drwxr-xr-x java-11-openjdk-amd64
lrwxrwxrwx default-java -> java-1.11.0-openjdk-amd64
```

**프로젝트 구조 확인 결과:**
```
android/
apps/
assembly/
build-cmake/
build-make/
c/
c++/
examples/
gcc/
kernel/
rtos/
shell-bash/
stream-editor-gawk/
stream-editor-sed/
system-library/
user-library/
```

### 원인 분석

프로젝트가 **Java 프로젝트가 아님**:
- C/C++ 코드 (`c/`, `c++/`)
- 어셈블리 (`assembly/`)
- 커널/RTOS 임베디드 코드 (`kernel/`, `rtos/`)
- CMake/Make 빌드 시스템 (`build-cmake/`, `build-make/`)
- 쉘 스크립트 (`shell-bash/`)

SonarQube가 기본적으로 JavaSensor를 실행하려고 시도했으나, Java 소스 코드가 없어서 실패함.

### 해결 방법

#### 방법 1: Java 분석 비활성화

`sonar-project.properties` 파일 생성:
```properties
sonar.projectKey=your-project-key
sonar.projectName=Your Project

# 소스 디렉토리 지정
sonar.sources=c,c++,shell-bash

# Java 분석 비활성화
sonar.java.enabled=false

# Java 파일 제외
sonar.exclusions=**/*.java
```

#### 방법 2: 분석할 언어 명시적 지정

```properties
sonar.projectKey=your-project-key
sonar.projectName=Your C/C++ Project

# C/C++ 소스 지정
sonar.sources=c,c++

# 파일 확장자 지정
sonar.cxx.file.suffixes=.c,.cpp,.h,.hpp,.cc

# 인코딩
sonar.sourceEncoding=UTF-8
```

#### 방법 3: C/C++ 분석 설정 (SonarQube Developer Edition 이상)

C/C++ 분석을 위해서는 **sonar-cxx 플러그인** 또는 **SonarQube Developer Edition** 이상이 필요.

```properties
sonar.projectKey=your-project-key
sonar.sources=c,c++

# Build Wrapper 사용 시
sonar.cfamily.build-wrapper-output=build-wrapper-output
```

**Build Wrapper 사용법:**
```bash
# Build Wrapper 다운로드 후
build-wrapper-linux-x86-64 --out-dir build-wrapper-output make clean all

# SonarQube 분석 실행
sonar-scanner
```

### JAVA_HOME 환경변수 설정 (권장)

비 Java 프로젝트라도 SonarQube Scanner 실행을 위해 JAVA_HOME 설정 권장:

```bash
# 영구 적용 (~/.bashrc에 추가)
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 확인
echo $JAVA_HOME
# 출력: /usr/lib/jvm/java-11-openjdk-amd64
```

### SonarQube Edition별 언어 지원

| Edition | C/C++ 지원 | 비고 |
|---------|-----------|------|
| Community | 플러그인 필요 (sonar-cxx) | 무료, 기능 제한 |
| Developer | 네이티브 지원 | 유료 |
| Enterprise | 네이티브 지원 | 유료 |

### 체크리스트

- [ ] 프로젝트가 실제로 어떤 언어로 구성되어 있는지 확인
- [ ] Java 프로젝트가 아니라면 `sonar.java.enabled=false` 설정
- [ ] C/C++ 분석이 필요하면 SonarQube 버전 및 플러그인 확인
- [ ] JAVA_HOME 환경변수 설정 (SonarQube Scanner 실행용)

---

*최종 수정: 2025-11-25*
*태그: #jenkins #sonarqube #troubleshooting #c #cpp #non-java*


---

## 3. Jenkinsfile sonar-scanner 명령어 오타 및 설정 누락

### 증상
sonar-project.properties 파일을 추가했는데도 SonarQube 분석이 여전히 실패

### 문제의 Jenkinsfile 코드

```groovy
stage('Analysis') {
    steps {
        script {
            withSonarQubeEnv('SonarQube-Server') {
                def sonarRunner = tool 'SonarScanner';
                sh """
                    ${sonarRunner}/bin/ssonar-scanner \
                        -Dsonar.verbose=true \
                        -Dsonar.projectKey=my-practice \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://14.35.255.147:9110 \
                        -Dsonar.login=sqp_2cd4a2b837d30177a29934f8a36389a2e9aa651c
                """
            }
        }
    }
}
```

### 원인 분석

| 문제점 | 설명 |
|--------|------|
| `ssonar-scanner` 오타 | `sonar-scanner`가 올바른 명령어 ('s'가 하나 더 있음) |
| `sonar.sources=.` | 전체 디렉토리를 스캔하여 불필요한 파일까지 분석 시도 |
| Java 비활성화 누락 | 비 Java 프로젝트인데 `sonar.java.enabled=false` 미설정 |
| 토큰 하드코딩 | 보안상 Credentials로 관리해야 함 |

### 해결 방법

#### 수정된 Jenkinsfile (기본)

```groovy
stage('Analysis') {
    steps {
        script {
            withSonarQubeEnv('SonarQube-Server') {
                def sonarRunner = tool 'SonarScanner'
                sh """
                    ${sonarRunner}/bin/sonar-scanner \
                        -Dsonar.verbose=true \
                        -Dsonar.projectKey=my-practice \
                        -Dsonar.projectName=my-practice \
                        -Dsonar.sources=c,c++ \
                        -Dsonar.host.url=http://14.35.255.147:9110 \
                        -Dsonar.login=sqp_2cd4a2b837d30177a29934f8a36389a2e9aa651c \
                        -Dsonar.java.enabled=false \
                        -Dsonar.sourceEncoding=UTF-8
                """
            }
        }
    }
}
```

#### 수정된 Jenkinsfile (보안 강화 - 권장)

```groovy
stage('Analysis') {
    steps {
        script {
            withSonarQubeEnv('SonarQube-Server') {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    def sonarRunner = tool 'SonarScanner'
                    sh """
                        ${sonarRunner}/bin/sonar-scanner \
                            -Dsonar.projectKey=my-practice \
                            -Dsonar.projectName=my-practice \
                            -Dsonar.sources=c,c++ \
                            -Dsonar.host.url=http://14.35.255.147:9110 \
                            -Dsonar.login=${SONAR_TOKEN} \
                            -Dsonar.java.enabled=false \
                            -Dsonar.sourceEncoding=UTF-8
                    """
                }
            }
        }
    }
}
```

### 주요 변경 사항 요약

| 항목 | 변경 전 | 변경 후 |
|------|---------|---------|
| 명령어 | `ssonar-scanner` ❌ | `sonar-scanner` ✅ |
| sources | `.` (전체) | `c,c++` (특정 폴더) |
| Java 분석 | (기본 활성화) | `sonar.java.enabled=false` |
| 토큰 관리 | 하드코딩 | Jenkins Credentials 사용 (권장) |

### Jenkins Credentials 설정 방법

1. Jenkins 관리 → Manage Credentials 이동
2. 적절한 도메인 선택 → Add Credentials
3. Kind: `Secret text` 선택
4. Secret: SonarQube 토큰 입력
5. ID: `sonar-token` (Jenkinsfile에서 참조할 이름)
6. Save

### 체크리스트

- [ ] `sonar-scanner` 명령어 철자 확인 (오타 주의)
- [ ] `sonar.sources` 경로가 실제 소스 디렉토리와 일치하는지 확인
- [ ] 비 Java 프로젝트라면 `sonar.java.enabled=false` 추가
- [ ] 토큰은 Jenkins Credentials로 관리 (보안)
- [ ] `sonar.projectName` 추가 권장

---

*최종 수정: 2025-11-25*
*태그: #jenkins #sonarqube #troubleshooting #jenkinsfile #typo*
