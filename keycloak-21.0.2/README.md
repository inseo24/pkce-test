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