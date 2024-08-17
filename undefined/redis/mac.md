# Mac

1. brew 버전 확인 : brew --version
2.  redis  설치\


    ```
    // redis 설치
    brew install redis

    // redis 설치 제거 (설치한 redis를 제거하고 싶으시다면 아래 명령어 실행)
    brew uninstall redis
    ```

    ```
    // redis 설치 확인
    redis-server --version
      Redis server v=7.2.5 sha=00000000:0 malloc=libc bits=64 build=bd81cd1340e80580
    ```


3.  redis 실행&#x20;

    1.  Foreground 실행\


        ```
        // redis foreground로 실행
        redis-server
        ```


    2.  Background  실행\


        ```
        // redis background로 실행
        brew services start redis

        // redis background로 재실행
        brew services restart redis

        // redis background로 중지
        brew services stop redis
        ```


4.  redis 실행 상태 확인\


    ```
    // redis 실행 상태 확인
    brew services info redis
    ```

    \


