Keycloak
========

- local setting 만 된 keycloak
- conf setting 에서 DB Config 와 local 실행을 위해 HTTP에서도 실행 가능하게 추가 설정이 포함됨
- ./bin/kc.sh start로 실행 가능

    $ ./bin/kc.sh start

```shell
db=mysql
db-username=seoin
db-password=tjdls@1234
db-url=jdbc:mysql://localhost:3307/keycloak?useSSL=false&allowPublicKeyRetrieval=true
hostname=localhost

# HTTP
http-enabled=true
hostname-strict-https=false
```

- 물론 이 세팅대로 production 에서 사용하면 안 된다.
- 명령어로 키클록 서버 띄우면 localhost:8080 에서 어드민 접속 가능 & 해당 디비로 키클록 관련 테이블이 몇십개 생성

------

1. Realm 생성
    - 내 Realm 이름은 `seoinRealm` 이다 ^ㅠ^

2. Client 생성
    - `test` 라는 이름의 Client 생성
    - config 는 대충 아래처럼 만든다. -> auth-code 로 진행할 거니 confidential client(Client authentication Off) + standard flow check 해주면 된다.
    - redirect uri 는 auth code 를 요청한 곳 + 해당 토큰을 return 할 곳을 검증하는 역할을 한다.
        - 예를 들어, auth code를 요청하면 code가 return 되고 해당 값 + username/password 로 POST 요청을 하면 Token 이 발급되는데 그게 code를 요청한 곳과 동일한지 체크가 이뤄져야 한다. 
        - 즉, auth code를 요청한 곳과 후에 토큰을 요청하는 곳이 같은지 validation 이 이뤄지며, 해당 redirect uri로 토큰이든 auth code든 리턴된다.
        - 일종의 white-list 겸 validation 역할을 해준다고 생각함
        - Web origin은 그냥 테스트 할 용도니 다 열어줌   
        

        <img width="500" alt="image" src="https://user-images.githubusercontent.com/84627144/230090511-2687d599-d05c-4669-a90d-c5a170bb8569.png">
        <img width="500" alt="image" src="https://user-images.githubusercontent.com/84627144/230090277-4f873b6d-d9be-41dc-b36f-9af229561a87.png">

3. Metadata URL
    - OIDCProviderMetadataURL ${KC_ADDR}/realms/${KC_REALM}/.well-known/openid-configuration
    - 나 같은 경우에는 http://localhost:8080/realms/seoinRealm/.well-known/openid-configuration 에서 확인 가능함
    - 열어보면 대충 아래 같은 화면
    
        <img width="500" alt="image" src="https://user-images.githubusercontent.com/84627144/230092808-eabdcba6-49d7-48c4-873f-78237162f987.png">
