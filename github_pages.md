# Github pages
- Github에서 바로 특정 웹페이지를 호스팅할 수 있게 제공하는 서비스

## 제약 조건
- repository 용량을 1GB 이하로 유지하길 추천
- publish 되는 Github pages 사이트의 용량은 1GB보다 클 수 없음

## User, Organization, Project Pages
- Github Pages는 그 주체가 User이냐 아니면 Organization이냐, 그리고 그 내용이 특정 Project에 대한 것이냐 아니면 주체에 대한 것이냐로 나누어짐

| 유형 | 페이지의 기본 도메인 & Github enterprise에서 호스트 위치 | 소스 페이지가 위치할 브랜치 |
| -- | -- | -- |
| User 페이지 | `username.github.io` | `master` |
| Organization 페이지 | `orgname.github.io` | `master` |
| 특정 User의 Project 페이지 | `username.github.io/projectname` | `gh-pages` |
| 특정 Organization의 Project 페이지 | `orgname.github.io/projectname` | `gh-pages` |
