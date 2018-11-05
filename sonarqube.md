# Sonarqube

[https://www.sonarqube.org/](https://www.sonarqube.org/)

## 사용법

### 서버

다운로드받고 압축 푼 다음 `{설치 경로}/bin/{os, bit 선택}/sonar.sh console` 실행해서 테스트

문제없으면 이번엔 `sonar.sh start` 사용해서 데몬을 띄우면 됨.

디비 따로 깔아줘야됨. 물론 임베디드 그대로 써도 되긴 하지만 데이터가 날아가도 어디 할말이 없으므로

설정하는건 `{설치 경로}/conf/sonar.properties` 값을 변경해서 설정 가능.

제일 먼저 바꿔줘야될건 db 설정

그리고 플러그인 설치할 것

플러그인은 checkstyle findbug 정도는 기본으로 깔아줘야할듯.

### 프로젝트

maven에서 쓰려면

plugin 추가

```markup
<plugin>
    <groupId>org.sonarsource.scanner.maven</groupId>
    <artifactId>sonar-maven-plugin</artifactId>
    <version>${sonarsource.version}</version>
</plugin>
```

profile 추가

```markup
<!-- sonar for verify -->
<profile>
    <id>sonar</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
        <sonar.host.url>
            {YOUR HOST}
        </sonar.host.url>
        <sonar.login>
            {YOUR TOKEN}
        </sonar.login>
        <sonar.links.homepage>
            {HOME PAGE LINK}
        </sonar.links.homepage>
        <sonar.links.scm>
            {SCM REPOSITORY LINK}
        </sonar.links.scm>
    </properties>
</profile>
```

명령어 실행

`mvn sonar:sonar`

