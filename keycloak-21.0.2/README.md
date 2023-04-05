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

### Authorization Code Flow + Keyclaok 

<details>
    <summary> Realm 생성부터 Code 요청 / Token 발급까지 . . .(Authorization Code Flow)</summary>

1. Realm 생성
    - 내 Realm 이름은 `seoinRealm` 이다 ^ㅠ^

2. Client 생성
    - `test` 라는 이름의 Client 생성
    - config 는 대충 아래처럼 만든다. -> auth-code 로 진행할 거니 confidential client(Client authentication On) + standard flow check 해주면 된다.
    - redirect uri 는 auth code 를 요청한 곳 + 해당 토큰을 return 할 곳을 검증하는 역할을 한다.
        - 예를 들어, auth code를 요청하면 code가 return 되고 해당 값 + username/password 로 POST 요청을 하면 Token 이 발급되는데 그게 code를 요청한 곳과 동일한지 체크가 이뤄져야 한다. 
        - 즉, auth code를 요청한 곳과 후에 토큰을 요청하는 곳이 같은지 validation 이 이뤄지며, 해당 redirect uri로 토큰이든 auth code든 리턴된다.
        - 일종의 white-list 겸 validation 역할을 해준다고 생각함
        - Web origin은 그냥 테스트 할 용도니 다 열어줌   
        

        <img width="682" alt="image" src="https://user-images.githubusercontent.com/84627144/230100575-2213eebd-5654-42b5-abc0-52a5b6db053e.png">
        <img width="700" alt="image" src="https://user-images.githubusercontent.com/84627144/230090277-4f873b6d-d9be-41dc-b36f-9af229561a87.png">

3. Metadata URL
    - OIDCProviderMetadataURL ${KC_ADDR}/realms/${KC_REALM}/.well-known/openid-configuration
    - 나 같은 경우에는 http://localhost:8080/realms/seoinRealm/.well-known/openid-configuration 에서 확인 가능함
    - 열어보면 대충 아래 같은 화면
    
        <img width="1000" alt="image" src="https://user-images.githubusercontent.com/84627144/230092808-eabdcba6-49d7-48c4-873f-78237162f987.png">

4. Code 요청
    - http://localhost:8080/realms/seoinRealm/protocol/openid-connect/auth?client_id=test&response_type=code&scope=openid&redirect_uri=http://localhost:8083/callback 
    - Metadata 에서 `authorization_endpoint` 에 요청하게 된다. 일단은 필수값만 더해서 브라우저상으로 요청한다.
        - 필수값 : `client_id`, `response_type=code`, `scope=openid`, `redirect_uri`
    
    - 그럼 아래처럼 로그인 화면이 뜬다.
    
        
        <img width="500" alt="image" src="https://user-images.githubusercontent.com/84627144/230094385-ac356868-c2b8-4332-8d26-61ff31459006.png">

5. 회원가입 / 로그인
    - 위에 보니 회원가입 버튼이 없는데 아래 설정을 안 해서 그럼
    - 여기 보이는 User Registeration 을 활성화 해야 회원가입 버튼이 나옴. 누르고 저장한 후, Code 요청 Url로 다시 해보자.

        <img width="500" alt="image" src="https://user-images.githubusercontent.com/84627144/230094955-d38cfe29-33db-45c0-a987-16c9d231a696.png">
        <img width="500" alt="image" src="https://user-images.githubusercontent.com/84627144/230095508-8a64e9c0-1cad-4c07-a1a7-759d9b88772d.png">

    - 회원가입 하면 당연히 에러 화면 뜸. 하지만 그게 정상임. 위에 URL을 보면 code가 생긴 걸 확인할 수 있음.
    - `http://localhost:8083/callback?session_state=0eacfe58-0c62-42dc-826c-4a1364a23b5f&code=16c54de9-a6b6-4c26-817a-ca1179089ab3.0eacfe58-0c62-42dc-826c-4a1364a23b5f.a4ffb0ae-f11c-48de-a878-f4e2fe2cc085`
    - 개발자 도구 네트워크 탭으로 보면 더 깔끔하게 보임
    
        <img width="801" alt="image" src="https://user-images.githubusercontent.com/84627144/230096202-ba79b8bc-f182-481f-a25f-3be5b7870310.png">


    - 아 로그인이 성공하면 키클록 콘솔에서도 해당 유저의 세션이 생성된 것을 확인할 수 있다.

        <img width="800" alt="image" src="https://user-images.githubusercontent.com/84627144/230097441-a7c7181c-570f-43f5-8052-413ab759006e.png">


6. Token 요청
    - `token_endpoint` 에 요청하면 된다. (이것도 위의 metadata에서 확인 가능)
    - 내 경우에는 http://localhost:8080/realms/seoinRealm/protocol/openid-connect/token 가 된다.
    - 매개변수로 `grant_type`, `client_id`, `client_secret`, `code`, `redirect_uri` 가 있다.
    - client secret 은 여기서 확인함
    
        <img width="1326" alt="image" src="https://user-images.githubusercontent.com/84627144/230101389-2e270934-f282-4335-a34b-c58be0404ac0.png">

    
    - 이제 토큰 요청은 포스트맨에서도 테스트 할 수 있음 (포스트맨 테스트 시 요청 하는 방법은 [여기!!!](https://www.postman.com/credshare/workspace/keycloak-sso/request/14351307-d7e4bff4-a72b-46c6-964f-d0ad6c2b3703) 참고
    - access token, id token, scope 등등 데이터가 나오는 걸 확인할 수 있다.
    
        <img width="842" alt="image" src="https://user-images.githubusercontent.com/84627144/230105116-a0e7a1a0-2530-4aa5-9b35-73e6586b8c97.png">


</details>
