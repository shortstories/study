# Spring Boot Font 배포 에러

## 증상

* Bootstrap을 사용하는 중, glyphicons-halflings 폰트 파일이 정상적으로 읽히지 않는 문제 발생
* woff2, woff, ttf 모두 발생

### 특징

* 200 OK
* 파일 MIME-type Mapping 완료되어있는 상황
  * woff : application/font-woff
  * woff2 : application/font-woff2
* 파일에 문제가 생겼나 싶어서 홈페이지에서 다운받은 깔끔한 새 파일로 바꿔넣어도 발생

### 에러 메세지

> Failed to decode downloaded font: {font file path}/myfont.woff2
>
> OTS parsing error: Failed to convert WOFF 2.0 font to SFNT
>
> Failed to decode downloaded font: {font file path}/myfont.woff
>
> OTS parsing error: incorrect file size in WOFF header
>
> Failed to decode downloaded font: {font file path}/myfont.ttf
>
> OTS parsing error: incorrect entrySelector for table directory

## 원인

* maven-resources-plugin의 필터링 과정에서 파일이 오염되어버림

## 해결책

* maven-resources-plugin의 옵션 중 nonFilteredFileExtensions에 폰트파일 확장자 입력

```markup
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <configuration>
        <nonFilteredFileExtensions>
            <!-- fonts -->
            <nonFilteredFileExtension>eot</nonFilteredFileExtension>
            <nonFilteredFileExtension>svg</nonFilteredFileExtension>
            <nonFilteredFileExtension>ttf</nonFilteredFileExtension>
            <nonFilteredFileExtension>woff</nonFilteredFileExtension>
            <nonFilteredFileExtension>woff2</nonFilteredFileExtension>
        </nonFilteredFileExtensions>
    </configuration>
</plugin>
```

