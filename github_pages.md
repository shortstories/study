# Github pages
- Github에서 바로 웹페이지를 호스팅할 수 있게 제공하는 서비스

## 제약 조건
- repository 용량을 1GB 이하로 유지하길 추천
- 배포되는 Github pages 사이트의 용량은 1GB보다 클 수 없음

## User, Organization, Project Pages
- Github Pages는 그 주체가 User이냐 아니면 Organization이냐, 그리고 그 내용이 특정 Project에 대한 것이냐 아니면 주체에 대한 것이냐로 나누어짐

| 유형 | 페이지의 기본 도메인 & Github enterprise에서 호스트 위치 | 소스 페이지가 위치할 브랜치 |
| -- | -- | -- |
| User 페이지 | `username.github.io` | `master` |
| Organization 페이지 | `orgname.github.io` | `master` |
| 특정 User의 Project 페이지 | `username.github.io/projectname` | `gh-pages` |
| 특정 Organization의 Project 페이지 | `orgname.github.io/projectname` | `gh-pages` |

### User, Organization Pages
- `username.[hostname]` 혹은 `orgname.[hostname]` 의 형태로 repository 생성
- 해당 repository의 master 브랜치에 업로드된 소스를 바탕으로 웹사이트를 빌드, 배포함
- 빌드가 완료되면 `http(s)://[hostname]/pages/<username>` 또는  `http(s)://pages.[hostname]/<username>` 으로 접근 가능

### Project Pages
- 프로젝트 페이지는 같은 repository에서 생성하는 것이 가능
- 원하는 프로젝트의 repository에서 `gh-pages` 브랜치를 만들어 소스를 업로드하면 됨
- User가 가지고 있는 프로젝트이면 `http(s)://[hostname]/pages/<username>/<projectname>/` 또는 `http(s)://pages.[hostname]/<username>/<projectname>/`
- Organization이 가지고 있는 프로젝트라면 `http(s)://[hostname]/pages/<orgname>/<projectname>/` 또는 `http(s)://pages.[hostname]/<orgname>/<projectname>/`

## Creating Pages with the automatic generator
1. Repository 메뉴의 Settings 클릭
2. Options 탭의 GitHub Pages에서 Launch automatic page generator 클릭
3. 적절히 내용을 수정하여 Continue to layouts 클릭
4. 레이아웃 선택 이후 Publish page 클릭

