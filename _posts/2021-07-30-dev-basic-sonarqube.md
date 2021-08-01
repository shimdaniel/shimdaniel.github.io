---
layout: post
title: "SonarQube docker install"
subtitle: "SonarQube docker & Jenkins 연동"
categories: dev
tags: sonarqube docker
comments: true
---

![그림1](https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube.png)

# 코드정적분석(SonarQube)

# 하드웨어 요구사항

- 최소 2GB의 RAM
- 1 vcpu 용량 서버
- OpenJDK 11/JRE 11
- EC2[t3a.medium]

# 설치

> Java 환경설정
H2 DB 설치
SonarQube DB 스키마 생성
SonarQube DB 계정 생성 및 권한 설정
SonarQube 설치
SonarQube DB 기본 설정 등 Docker Image에 기능 포함됨

## Docker 설치

- 서버업데이트 : sudo yum -y upgrade
- 도커설치 : sudo amazon-linux-extras install -y docker
- 도커확인 : docker --version
- 도커시작 : sudo systemctl start docker
- 권한부여 : sudo usermod -aG docker ec2-user

## SonarQube Container 설치

- 설치 : docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube
- 시작 : docker start sonarqube
- 종료 : docker stop sonarqube
- 확인1 : docker ps -a
- 확인2 : http://[설치서버도메인]:9000

## SonarQube 기본 설정

- 로그인 : 기본 ID/PW [admin/admin]

![https://https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-1.png](https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-1.png)

![https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-1.png](https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-1.png)

- 로그인 : 현재 ID/PW [admin/모빌러.11Q]

![https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-3.png](https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-3.png)

## SonarQube 프로젝트 생성

- [Administration](http://localhost:9000/admin) > Projects(Management) > Create Project
- Name : Project name (ex: "MOVILL-PASS")
- Key : Project mapping key (ex: "sonar:MOVILL-PASS")

## SonarQube authorization key 발급

- [Administration](http://localhost:9000/admin) > Security(Users) > Tokens > Generate Tokens
- Token name : movill_sonar_token

## SonarQube - Jenkins 연동

- 프로젝트 최상위 ROOT 경로에  sonar-project.properties  추가
- sonar.projectKey, sonar.projectName : 해당 프로젝트키, 이름
- sonar.java.binaries : 해당 프로젝트 정적검사 대상  build class path
- sonar.exclusions : 해당 프로젝트 정적검사 제외 path or file

```bash
sonar.login=5211c402aeee68c59bd8e507d5ebf9b282f973a0
sonar.projectKey=sonar:XXXX
sonar.projectName=XXXX
sonar.host.url=http://localhost:9000/
sonar.java.binaries=xxxx-api/build/classes/java/main/xxx/xxx/xxx
sonar.exclusions=xxxx-api/src/main/resources/static/docs/*\
  ,xxxx-api/build/classes/java/test/xxx/xxx/xxx/service/*\
  ,xxxx-api/src/test/java/xxx/xxx/xxx/service/*\
  ,xxxx-api/src/test/**
sonar.projectVersion=1.0
sonar.sourceEncoding=UTF-8
```

- Project build : SonarQube Quality Gate [OK]
- Project quality : Left menu[SonarQube] or [OK] Click!!

![https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-7.png](https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-7.png)

## SonarQube 공식 사용자 가이드

- [https://docs.sonarqube.org/latest/](https://docs.sonarqube.org/latest/)

## JaCoCo code coverage reports to Jenkins & SonarQube

- JaCoCo plugin 후 jenkins 재시작
- sonar-project.properties jacoco 항목 추가
    - sonar.jacoco.reportPath=build/jacoco/test.exec
    - sonar.dynamicAnalysis=reuseReports
    - sonar.surefire.reportsPath=tests/test-reports

- Jenkins 프로젝트 구성 > 빌드 후 조치

![https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-4.png](https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-4.png)

- build.gradle Jacoco 내용 추가

```bash
plugins {
    id "jacoco"
		...
}

jacoco {
    toolVersion = '0.8.5'
    reportsDir = file("$buildDir/customJacocoReportDir")
}

jacocoTestReport {
    reports {
        xml.enabled false
        csv.enabled false
        html.enabled true
    }
    finalizedBy 'jacocoTestCoverageVerification'
}

jacocoTestCoverageVerification {
    violationRules {
        rule {
            enabled = true
            element = 'CLASS'

            limit {
                counter = 'LINE'
                value = 'COVEREDRATIO'
//                minimum = 0.30
            }

            excludes = ['*.test.*']
            includes = []
        }
    }
}

test {
    finalizedBy 'jacocoTestReport'
		...
}
```

- JaCoCo code coverage build 확인

![https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-5.png](https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-5.png)

- JaCoCo code coverage test-reports 상세 확인

![https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-6.png](https://shimdaniel.github.io/assets/img/post_img/2020-12-30-dev-basic-sonarqube-6.png)

- 참고 : 해당 기능은 프로젝트별 선택이며 도입 시 Coverage level 을 30% 이하 권장합니다.