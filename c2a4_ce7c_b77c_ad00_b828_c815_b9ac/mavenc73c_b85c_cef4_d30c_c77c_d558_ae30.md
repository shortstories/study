# Maven으로 컴파일하기

## scala-maven-plugin

* pom.xml에 추가
* scala 설치 되어있을 필요 없음
* 자바 클래스랑 섞여있어도 정상 작동

```markup
<plugin>
    <groupId>net.alchim31.maven</groupId>
    <artifactId>scala-maven-plugin</artifactId>
    <version>${scala-maven-plugin.version}</version>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
                <goal>testCompile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

