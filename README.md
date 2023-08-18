# spring boot + swagger 2.0 + restdocs 를 활용한 문서화

### Rest Docs?
* 코드로 상세하게 API 정의를 해 주는 도구
* Gradle Plugin 형태로 제공 해 주고, Test Code 형태로 문서를 관리해준다.
* 문서화 코드와 비즈니스 로직의 경계가 뚜렷하게 구분되어 굉장히 좋다
  * Controller Test에서 문서화 코드를 진행하기 때문에 
* UI/UX를 제공 해 주지만, 미관상 좋지 않다는 단점을 가지고 있다. 

### Swagger?
* API 문서를 보기 좋게 만들어 주는 UI 도구
* 미관상 좋지만, 각 컨트롤러마다 @Annotation을 달아 줘야하는 단점
* API 명세를 세세하게 작성할 경우 코드가 지저분 해질 수 있다는 단점

### Rest Docs + Swaager의 장점만을 모아서 사용할 수 없을까?
* Rest Docs의 코드 관리 + Swaager의 UI/UX 장점을 활용하기 위한 방안
* Rest Docs 에서 제공해주는 **com.epages.restdocs-api-spec** 라이브러리는 여러 Format(JSON / YAML 등)으로 문서화 산출물을 만들어준다.
  * 이를 통해 만들어진 format 파일은 표준화 된 문서화 format이다.
* Swaager UI에서는 외부 Format을 Import해서 사용할 수 있다.
* 본 프로젝트에서는 이 두 가지 특징을 통해 API 문서화를 진행했다.

### MSA 환경에서 여러 서버에 각각 존재하는 문서들을 하나로 보여주고 싶은데..
* MSA 환경에서는 여러 서버가 존재하기 때문에 각각의 API 문서 파일이 만들어진다.
* 이 문서들은 합치는 방법은 여러 가지(빌드 시 가져온다거나, 문서화 서버를 따로 두어 빌드 시에 서버로 문서화 파일을 전송한다거나) 방법이 있는데 이것은.. 만들기 나름인듯하다.
* 문서들을 한 군데 모아놨다는 가정 하에, Swaager 문서화 서버를 Docker로 띄운다
  * Docker 서버를 띄우기 전에 swagger api 문서화 파일을 연결해야하는데, 나의 경우에는 Docker 설정을 Shell 파일로 만들어서 띄워주도록 했다. 

### 문서화 구동 순서
* 프로젝트 루트 > ./gradlew openapi3 => 테스트 코드 컴파일 진행
* 모든 테스트를 통과하면, {문서이름}.yaml 파일이 생성 됨
* MSA 환경에서 생성 된 문서를 한 곳에 모아놨다는 가정 하에,
* docs 디렉토리로 > sh ./swagger-build.sh 실행
* http://localhost:80 으로 접속하면 문서화 화면이 나타난다. 

### API 문서화 사용 방법
* MSA 환경에서 각 서버 마다 API 명세가 다르기 때문에 이를 통합해주기 위한 방안
    * 각 서버에서 ./gradlew openapi3 입력 시 문서화를 위한 테스트 코드가 실행 됨.
    * 이 문서화는 컨트롤러에 대한 테스트 코드를 구현해야 작동할 수 있다.
        * 문서화는 Production 코드에 영향을 주어선 안 되므로, 테스트 코드를 통해서 작성
* 현재 각 서버에 생성 된 openapi3 파일을 모아주는 자동화 작업이 필요한데
    * 각 서버 Repository 의 build/docs/{서버이름.yaml} 파일이 만들어지고, 젠킨스 파이프 라인으로 테스트 빌드 후 가져오거나
    * 각 서버에서 빌드 되었다고 가정한 후, 위 디렉토리에 있는 oepnapi3 파일을 가져와서 종합해주는 방안도 생각해 볼 수 있다.
    * Frontend / Client 개발자 분들이 API 명세를 공유할 수 있도록 Repository 형태로 해당 Document 실행 파일을 관리하는게 좋을듯.


