# pkce-test

- [키클록을 이용해 Authorization Code Flow 로 Code / Token Request 하는 방법](https://github.com/inseo24/pkce-test/blob/main/keycloak-21.0.2/README.md#authorization-code-flow--keyclaok)
  - 위 링크의 README.md 에서 토글을 열어서 확인하면 된다 ^ㅠ^ 나 열심히 썼따 인정해라 
- [키클록 요청 시 참고하기 좋은 자료 - 포스트맨으로 요청할 때 필요한 거 다 모여 있음](https://www.postman.com/credshare/workspace/keycloak-sso/request/14351307-d7e4bff4-a72b-46c6-964f-d0ad6c2b3703)
- [키클록에서 PKCE를 써보자](https://github.com/inseo24/pkce-test/blob/main/keycloak-21.0.2/README.md#authorization-code-flow-with-pkce--keyclaok)
- [키클록에서 해당 클라이언트에 원하는 개인정보만 받게 세팅해보자 + Client Scope 추가 방법](https://github.com/inseo24/pkce-test/blob/main/keycloak-21.0.2/README.md#consent%EA%B6%8C%ED%95%9C-%EB%8F%99%EC%9D%98--default-scope)

<details>
  <summary>기타 학습</summary>
  
- nonce : Number used ONCE, 일회용으로 사용하기 위해 생성하는 난수
  - nonce 자체는 키클록에서 생성하는게 아니라, 클라이언트에서 생성하고 요청에 포함함
  - 재전송 방지 목적
  
- state : CSRF(Cross-Site Request Forgery) 공격 방지, 클라이언트 생성하고 OAuth2 인증 프로세스 내내 유지
  - 일반적으로 난수와 함께 시간 정보나 사용자 세션 같은 추가 정보를 사용해 생성
  
</details>

- 읽어볼 링크
  - [auth0에서 작성한 PKCE 가이드](https://auth0.com/docs/get-started/authentication-and-authorization-flow/call-your-api-using-the-authorization-code-flow-with-pkce)
  - [auth0에서 작성한 auth-code flow 가이드](https://auth0.com/docs/get-started/authentication-and-authorization-flow/call-your-api-using-the-authorization-code-flow)
