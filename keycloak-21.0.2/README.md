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

7. 참고
    - 위에 토큰 요청할 때 Header 에 Content Type 설정 필요함
    - application/x-www-form-urlencoded
    
        <img width="678" alt="image" src="https://user-images.githubusercontent.com/84627144/230401848-7aefee56-3b4f-48d2-a1d2-8b8bd185aa9a.png">

    
</details>

------ 

### Authorization Code Flow with PKCE + Keyclaok 

<details>
    <summary> PKCE로 해보자 </summary>
    
1. Client Type 변경
    
    - public 으로 변경해줘야 함
    - Client 안에 Advanced Tab 에 가서 Proof Key for Code Exchange Code Challenge Method 도 S256으로 할 걸 세팅 필요
    
        <img width="658" alt="image" src="https://user-images.githubusercontent.com/84627144/230393531-d6a1e3fd-add9-4f97-b1be-37ac82d7d3b9.png">
        <img width="1286" alt="image" src="https://user-images.githubusercontent.com/84627144/230393650-f138936e-f8ba-4549-a8b5-108efe1597ba.png">

2. code challenge 값이 필요함
    - 테스트 편리성을 위해 
        - code_challenge로 `01jGMnbTorlfVp5dusMZtXxT543bcf9o5fmMh4W-hHM` 
        - verifier 로 `EAp91aanXdoMcoOc2Il55H3UDDIV909k9olEEcl6L24J6_9X` 값을 써보자. 대충 블로그에서 긁어온 것임


3. Code Request
    - 로그인을 해보자.
        - `http://localhost:8080/realms/seoinRealm/protocol/openid-connect/auth?client_id=test&response_type=code&scope=openid&redirect_uri=http://localhost:8083/callback&code_challenge=HVoKJYs8JruAxs7hKcG4oLpJXCP-z1jJQtXpQte6GyA&code_challenge_method=S256`
        - param 으로 `code_challenge_method=S256`, `code_challenge=01jGMnbTorlfVp5dusMZtXxT543bcf9o5fmMh4W-hHM` 값을 추가해야 한다.
    - 브라우저에서 로그인 요청 후 아래와 같이 URL 변경된 걸 보면 code가 동일하게 리턴된 걸 확인할 수 있다.
        - `http://localhost:8083/callback?session_state=84911217-88f0-4263-9a1a-8a12d1a574fa&code=84c2f1dc-36e4-4644-a183-4468e4714c7c.84911217-88f0-4263-9a1a-8a12d1a574fa.a4ffb0ae-f11c-48de-a878-f4e2fe2cc085`
    
    

4. Token Request
    - Postman 으로 토큰을 받아보자.    
    - param 으로 client_secret 은 더이상 보낼 필요가 없고, code_verifier 를 보내줘야 한다.    
        
        <img width="844" alt="image" src="https://user-images.githubusercontent.com/84627144/230398854-3bdb98ca-67d4-4d97-ac08-5f815824bc6d.png">

    
</details>

-------------


### Consent(권한 동의) & Default Scope

<details>
    <summary> Consent & Scope </summary>

- 보통 SSO 하다 보면, 로그인 후 해당 서비스로 돌아가기 전에 이름, 이메일 등 서비스로 넘겨줄 개인 정보 리스트와 동의하겠냐는 문구가 뜨는데, 이걸 세팅해보자. 
    - Keycloak 에서는 Consent라고 되어 있다. 아래 캡쳐처럼 Client 별로 세팅이 가능하다.
    
        <img width="901" alt="image" src="https://user-images.githubusercontent.com/84627144/230911312-004b2cbd-6141-4f0d-89a6-effd09a0dfcc.png">

    - 위와 같이 세팅한 후 다시 로그인을 해보면 아래처럼 개인정보 동의 화면이 출력된다. 
        
        - 여기서 동의하지 않을 경우 access_denied 가 리턴된다.

        <img width="657" alt="image" src="https://user-images.githubusercontent.com/84627144/230911652-489ab14d-02f1-4a52-8735-7e9ceb6ba1be.png">       

    - 화면에 보이는 User roles, Email address, User profile는 Client Scope 에 Default로 세팅된 값들이다.
        
        <img width="953" alt="image" src="https://user-images.githubusercontent.com/84627144/230911984-ca8d6d2c-220c-4fbb-804c-c2505ba20a8f.png">
        
    - 위 화면에 보이는 Optional 값 중에 Phone 을 Default로 바꾸면 아래처럼 추가 된다.
    
        <img width="605" alt="image" src="https://user-images.githubusercontent.com/84627144/230912387-446305c9-0a2d-48cd-8188-f369ee5ca340.png">


    - 당연히 동의를 눌러야 Code가 발급된다. 
    
- Scope를 요청할 때 scope=openid 로 param을 넘겨주고 있는데 당연히 요구하는 것만 넣을 수 있다. 
    - `http://localhost:8080/realms/seoinRealm/protocol/openid-connect/auth?client_id=test&response_type=code&scope=openid nickname&redirect_uri=http://localhost:8083/callback&code_challenge=HVoKJYs8JruAxs7hKcG4oLpJXCP-z1jJQtXpQte6GyA&code_challenge_method=S256`
    - 위와 같이 scope=openid nickname 을 해보자.
    - 미리 좀 세팅이 필요한데, 아래처럼 Client Scope Tab에서 하나 생성해주자.
       
       <img width="850" alt="image" src="https://user-images.githubusercontent.com/84627144/230915827-facd1b81-3700-46d5-9f0d-7804541438fa.png">
    
    - 생성된 Client Scope 안에서 Mappers 쪽을 들어가면 predefined 된 mappers 중에 nickname 을 써주자. 그럼 user_attribute 에 nickname으로 설정된 값과 알아서 매핑된다.
    - 그리고 해당 Client 에 가서 Client Scope 에서 추가해주자.
     
        <img width="1059" alt="image" src="https://user-images.githubusercontent.com/84627144/230916305-1d1d4e01-30f4-4f52-96fc-d48b099d76b5.png">

    - 물론 User의 Attribute 에 이렇게 값이 세팅되어 있어야 함
        
        <img width="1385" alt="image" src="https://user-images.githubusercontent.com/84627144/230916466-75ede9a0-1bf1-46e2-9bcf-6f306bfa231c.png">
    
    - 세팅 다 끝내고 위의 URL로 코드 발급 후 토큰을 받아보면 아래처럼 nickname이 보인다.
    
        <img width="403" alt="image" src="https://user-images.githubusercontent.com/84627144/230916819-d690b081-48e4-4c6f-9379-8a3e23efc11e.png">

    - 아직까진 idToken이랑 access token이랑 분리가 안 되어 있는 상태라 사실상 두 토큰 모두 돌려보면 nickname이 보인다. (원래는 idToken 에 담겨야함) 
    - 그건 다음에 해보는걸로 ! 

</details>
