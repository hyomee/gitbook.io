# 인증서버(Keycloak)

Keycloak은 오픈소스 기반의 ID 및 액세스 관리 도구로, 단일 페이지 애플리케이션(SPA), 모바일 앱, REST API 등 현대적인 애플리케이션을 중심으로 설계되었으며 다음 기능을 제공 합니다.

* SSO(싱글사인온)
* OAuth2 / OpenID Connect / SAML 인증 지원
* 소셜 로그인 연동(Google, Kakao, Naver 등 가능)
* 사용자 관리(UI 제공)
* 권한/역할(Role) 관리
* LDAP/AD 연동
* Multi-tenant 구조(Realm)

***

## 1. 주요 개념

<table><thead><tr><th width="160.3330078125">개념</th><th>설명</th></tr></thead><tbody><tr><td><strong>Realm</strong></td><td><p>인증 공간(테넌트). 서로 격리된 인증 도메인<br></p><ul><li>회사 A Realm과 회사 B Realm은 서로 인증 정보가 공유되지 않음</li><li>서로 다른 서비스 세트들을 완전히 분리하여 관리 가능</li><li>관리자(admin) 계정도 Realm마다 존재</li><li>Keycloak은 기본적으로 <code>master</code> realm 을 보유</li></ul></td></tr><tr><td><strong>Client</strong></td><td><p>Keycloak으로 인증을 사용할 애플리케이션이 Keycloak에서 인증권한을 얻기 위한 접점<br></p><ul><li>백엔드 API 서버</li><li>웹 프론트엔드(SPA)</li><li>모바일 앱</li><li>외부 서비스 연동</li></ul><p>Client 설정에서 다음을 지정합니다:</p><ul><li>Redirect URI (로그인 후 돌아갈 주소)</li><li>Access Token 설정</li><li>Client 인증 방식 (confidential / public)</li><li>Protocol(OIDC 또는 SAML)</li></ul></td></tr><tr><td><strong>User</strong></td><td><p>로그인하는 사용자 계정 정보로 모든 인증은 사용자 계정을 기준으로 처리합니다.<br></p><ul><li>username / email / password</li><li>Role(역할) 할당</li><li>Group(그룹) 할당</li><li>Credential 설정(OTP/MFA 등)</li></ul></td></tr><tr><td><strong>Role</strong></td><td><p>권한으로 인가(Authorization)를 위한 권한 값을 의미로 JWT Access Token 속에 포함되어 애플리케이션에서 권한 체크에 사용<br></p><p>타입:</p><ol><li><strong>Realm Role</strong><br>Realm 전체에 적용되는 권한</li><li><strong>Client Role</strong><br>특정 Client에서만 사용하는 권한</li></ol><p>예:</p><ul><li>ROLE_ADMIN</li><li>ROLE_USER</li><li>api-user</li><li>api-admin</li></ul></td></tr><tr><td><strong>Group</strong></td><td><p>사용자 그룹으로 사용자를 묶는 상위 개념.<br></p><ul><li>예: “관리자”, “일반회원”, “개발팀”, “회계팀”</li><li>Group 자체에 Role을 부여할 수 있음</li><li>팀 단위 권한 관리에 적합</li></ul></td></tr><tr><td><strong>Identity Provider</strong></td><td><p>외부 로그인 서비스(구글, 깃허브 등)로 Keycloak 로그인을 외부 서비스와 연결로 Keycloak이 브로커(Broker) 역할을 수행<br></p><ul><li>Google</li><li>GitHub</li><li>Kakao</li><li>Naver</li><li>Facebook</li><li>SAML 기반 회사 인증 시스템</li></ul></td></tr><tr><td><strong>User Federation</strong></td><td><p>회사 내부 인증 시스템을 Keycloak에서 그대로 사용 가능<br></p><ul><li>LDAP</li><li>Active Directory</li></ul></td></tr></tbody></table>

***

## 2. 동작방법

* 애플리케이션(Client)이 Keycloak에 인증 요청
* 사용자가 Keycloak 로그인 페이지에서 로그인
* Keycloak이 Access Token / ID Token 발급
* 애플리케이션은 토큰을 검증 후 사용자 인증 완료

### 2.1 토근

Keycloak은 인증 결과로 **OIDC / OAuth2** 표준 토큰을 발급, JWT 기반이므로 stateless 인증 가능

* **Access Token** (권한 확인 / API 호출용)
* **ID Token** (사용자 식별 정보)
* **Refresh Token** (만료 갱신)

토큰 안에는:

* 사용자 정보(Claims)
* Roles(역할)
* 만료 시간
* Issuer 정보

이 포함되어 있습니다.

***

## 3. Keycloak 설치 방식

```bash
docker run -p 8080:8080 \
-e KEYCLOAK_ADMIN=admin \
-e KEYCLOAK_ADMIN_PASSWORD=admin \
quay.io/keycloak/keycloak:latest start-dev
```

***

## 4. ADMIN SETUP

관리자 구성 단계는 다음과 같습니다:

1. Admin 계정 생성
2. Admin Console 로그인
3. Realm 생성
4. 사용자(User) 생성
5. Account Console 로그인
6. Client 생성
7. Role 생성
8. Role을 사용자에게 할당

### 4.1. Admin 계정 생성

Keycloak은 기본 Admin 계정이 없으므로 먼저 만들어야 합니다.

1. 브라우저에서 `http://localhost:8080` 접속
2. Username, Password를 입력하여 Admin 계정을 생성

### 4.2. admin Console 로그인

* 동일 URL에서 “Administration Console” 클릭
* 방금 만든 Admin 계정으로 로그인

### 4.3. Realm 생성

Realm은 멀티 테넌시처럼 애플리케이션/사용자 그룹을 분리하는 단위입니다.

* 왼쪽 상단의 `master` 클릭
* “Create realm” 선택
* Realm name: **myrealm**
* Create 버튼 클릭

### 4.4. 사용자(User) 생성

1. Admin Console → Users → Create new user
2. 아래 값 입력:

| 항목         | 값      |
| ---------- | ------ |
| Username   | myuser |
| First name | 아무 이름  |
| Last name  | 아무 성   |

3. Create 클릭

#### 비밀번호 설정

1. 상단 탭에서 **Credentials** 클릭
2. Password 입력
3. Temporary = **Off** 로 설정\
   (첫 로그인 시 비밀번호 변경 불필요)

### 4.5. Account Console 로그인

이제 사용자 설정이 잘 되었는지 확인합니다.

1. Keycloak Account Console 접속
2. Username: **myuser**\
   Password: 설정한 비밀번호 입력

로그인 후 할 수 있는 작업:

* 프로필 수정
* 2단계 인증 설정
* 외부 ID 제공자 계정 연동

### 4.6. Client 생성

Keycloak에서 애플리케이션을 등록하는 과정입니다.

1. Admin Console → Clients 클릭
2. Create client
3. 입력 값:

| 항목          | 값              |
| ----------- | -------------- |
| Client Type | OpenID Connect |
| Client ID   | myclient       |

4. Next 클릭
5. Standard flow가 활성화된 것을 확인
6. Save 클릭

#### 클라이언트 설정 업데이트

1. Access settings 섹션으로 이동
2. Valid redirect URIs:

```
https://www.keycloak.org/app/*
```

3. Web origins:

```
https://www.keycloak.org
```

4. Save 클릭

Keycloak 공식 SPA 테스트 앱에서 정상 작동 여부 확인 가능

### 4.7. Role 생성

1. Realm roles 메뉴로 이동
2. Create role 클릭
3. Role name 입력 및 저장

### 4.8. Role을 사용자에게 할당

* Users → 사용자 선택
* Role Mappings 탭으로 이동
* Assign role 클릭
* 역할 선택 후 Assign

### **4.9. KEYCLOAK ENDPOINTS**

Realm 생성 후 사용할 수 있는 주요 OIDC Endpoint URL:

```
http://localhost:8080/realms/{realm-name}/.well-known/openid-configuration
```

### **4.10.Adding SSO — SSO(구글 등) 추가**

Keycloak은 Google, Facebook, GitHub 등 다양한 SSO 제공자를 기본 지원합니다.

#### SSO 추가 단계:

1. Google 개발자 콘솔에서 Client ID & Secret 생성
2. Keycloak → Identity Providers → Google 선택
3. Client ID, Secret 입력
4. Google 콘솔에도 Keycloak 리다이렉트 URL 입력

이제 로그인 화면에 Google 로그인 버튼 추가됨.<br>

***

## 5. 스프링 부트 + Keycloak 연동

Spring Boot 3 보안 구조를 이해해야 하며, WebSecurityConfigurerAdapter는 더 이상 사용되지 않습니다.

### 5.1. 의존성 추가

(OAuth2 Resource Server + Security + Web)

```yml
implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server'
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.springframework.boot:spring-boot-starter-web'
```

### 5.2. SecurityFilterChain 구현

Role 기반 접근 제어 설정

예시:

```
/test/admin → admin 역할만
/test/user → admin 또는 user 역할
/test/anonymous → 모두 허용
```

`@Configuration`, `@EnableWebSecurity`가 선언된 `WebSecurityConfig` 클래스에서 `SecurityFilterChain securityFilterChain(HttpSecurity http)` 메서드 하나로 전체 보안 규칙을 정의 합니다.

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {

  public static final String ADMIN = "admin";
  public static final String USER = "user";
  private final JwtAuthConverter jwtAuthConverter;
  
  @Autowired
  private JWTAccessDeniedHandler jwtAccessDeniedHandler;
  
  @Autowired
  private JWTAuthenticationEntryPoint jwtAuthenticationEntryPoint;
  
  @Bean
  public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
      http.authorizeHttpRequests()
           // requestMatchers 체이닝으로 URL·HTTP 메서드별 권한을 정의
          .requestMatchers(HttpMethod.GET, "/test/anonymous", "/test/anonymous/**","/test/error").permitAll()
          .requestMatchers(HttpMethod.GET, "/test/admin", "/test/admin/**").hasRole(ADMIN)
          .requestMatchers(HttpMethod.GET, "/test/user").hasAnyRole(ADMIN, USER); 
      http.exceptionHandling()
          // 인가 실패(403)와 인증 실패(401/403)
          // 권한이 없는 사용자가 보호된 엔드포인트를 호출하면 JWTAccessDeniedHandler가 JSON 바디에 커스텀 메시지를 담아 응답하고, 
          // 인증이 아예 없는 경우에는 JWTAuthenticationEntryPoint가 지정한 에러 엔드포인트(/test/error)로 포워딩
          .accessDeniedHandler(jwtAccessDeniedHandler)
          .authenticationEntryPoint(jwtAuthenticationEntryPoint);
      http.oauth2ResourceServer()
          // 이 애플리케이션이 OAuth2 리소스 서버로 동작하며, 
          // 수신한 Bearer 토큰을 JWT로 파싱한 뒤 JwtAuthConverter를 이용해 Authentication 객체를 생성하도록 지정
          .jwt()
          .jwtAuthenticationConverter(jwtAuthConverter);
      http.sessionManagement()
          // 세션을 만들지 않는 완전한 stateless API 서버 형태를 사용하여, 
          // 각 요청마다 토큰만으로 인증·인가를 수행
          .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
      return http.build();
  }
}
```

requestMatcher() 메서드는 keycloak에 정의된 사용자 역할별로 허용되는 HTTP 메서드를 가진 API를 정의하는 데 이를 통해 특정 역할을 가진 사용자만 API에 접근할 수 있도록 제한하고 다른 사용자의 접근을 허용하지 않습니다.&#x20;

### 5.3. JWT 변환기(JwtAuthConverter) 작성

`JwtAuthConverter`는 Spring Security의 `Converter<Jwt, AbstractAuthenticationToken>`을 구현해 JWT를 `JwtAuthenticationToken`으로 변환합니다.

```java
@Component
public class JwtAuthConverter implements Converter<Jwt, AbstractAuthenticationToken> {

  private final JwtGrantedAuthoritiesConverter jwtGrantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();
  private final JwtAuthConverterProperties properties;
  
  public JwtAuthConverter(JwtAuthConverterProperties properties) {
    this.properties = properties;
  }

  @Override
  public AbstractAuthenticationToken convert(Jwt jwt) {
    // 기본 스코프/권한 + ROLE_ prefix가 붙은 SimpleGrantedAuthority 목록 합침
    Collection<GrantedAuthority> authorities = Stream.concat(
        // 기본 스코프/권한을 얻음 
        jwtGrantedAuthoritiesConverter.convert(jwt).stream(),
        // resource_access.{client_id}.roles에 있는 Keycloak 역할들을 읽어서 
        // ROLE_ prefix가 붙은 SimpleGrantedAuthority 목록
        extractResourceRoles(jwt).stream()).collect(Collectors.toSet());
    return new JwtAuthenticationToken(jwt, authorities, getPrincipalClaimName(jwt));
  }

  // getPrincipalClaimName 메서드는 기본적으로 sub 클레임을 principal로 사용하지만, 
  // 설정(JwtAuthConverterProperties)에 principalAttribute가 지정되어 있다면 
  // 예를 들어 preferred_username 같은 값을 principal로 사용
  // 이렇게 하면 SecurityContext에서 Authentication.getName()이 Keycloak의 
  // 사용자명이나 원하는 클레임을 가리키도록 커스터마이즈할 수 있다
  private String getPrincipalClaimName(Jwt jwt) {
    String claimName = JwtClaimNames.SUB;
    if (properties.getPrincipalAttribute() != null) {
        claimName = properties.getPrincipalAttribute();
    }
    return jwt.getClaim(claimName);
  }

  
 // JWT의 resource_access 클레임에서 특정 client(예: myclient)에 대한 roles 배열
 // 이 배열을 ROLE_admin, ROLE_user 형식의 GrantedAuthority 집합으로 변환하여, 
 // hasRole("admin") 같은 Spring Security 표현식이 Keycloak의 역할과 정확히 매핑
  private Collection<? extends GrantedAuthority> extractResourceRoles(Jwt jwt) {
 
    Map<String, Object> resourceAccess = jwt.getClaim("resource_access");
    Map<String, Object> resource;
    Collection<String> resourceRoles;
    if (resourceAccess == null
      || (resource = (Map<String, Object>) resourceAccess
                          .get(properties.getResourceId())) == null
      || (resourceRoles = (Collection<String>) resource
                        .get("roles")) == null) {
         return Set.of();
     }
    return resourceRoles.stream().map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                             .collect(Collectors.toSet());
  }
}
```

### 5.4. AccessDeniedHandler, AuthenticationEntryPoint 구현

```java
@Component
public class JWTAccessDeniedHandler implements AccessDeniedHandler {
    // 권한 부족 시 HTTP status를 200이 아니라 200이지만 응답 바디에 자체적인 
    // 에러 메시지(JSON)를 넣는 방식으로 처리             
    @Override
    public void handle(HttpServletRequest request, 
                       HttpServletResponse response,
                       AccessDeniedException accessDeniedException) 
                 throws IOException, ServletException {
      response.setStatus(HttpStatus.OK.value());
      response.setContentType(MediaType.APPLICATION_JSON_VALUE);
      Response apiError = new Response("이 리소스에 대한 접근 권한이 없습니다. 관리자에게 문의하십시오.");
      OutputStream os = response.getOutputStream();
      ObjectMapper om = new ObjectMapper();
      om.writeValue(os, apiError);
      os.flush();
   }
}
```

```java
@Component
public class JWTAuthenticationEntryPoint extends Http403ForbiddenEntryPoint{
    // 인증 실패 시 상태 코드를 403으로 설정한 뒤 RequestDispatcher를 사용해 /test/error로 포워드
    @Override
    public void commence(HttpServletRequest request, 
                         HttpServletResponse response, 
                         AuthenticationException arg2)
                throws IOException {
      response.setStatus(HttpStatus.FORBIDDEN.value());
      RequestDispatcher rd = request.getRequestDispatcher("/test/error");
      try {
        rd.forward(request, response);
      } catch (ServletException | IOException e) {
        e.printStackTrace();
      }
    }
}
```

### 5.5. application.properties 설정

```
## jwk-set-uri로 리소스 서버가 Keycloak의 공개키(JWK)와 issuer를 알고 JWT 서명을 검증
spring.security.oauth2.resourceserver.jwt.issuer-uri = http://localhost:8080/realms/myrealm
## Keycloak 클라이언트 ID
jwt.auth.converter.resource-id = myclient
## principal로 사용할 클레임(예: preferred_username)을 적어 JwtAuthConverter에서 사용할 수 있게 한다
jwt.auth.converter.principal-attribute = preferred_username
```

## 6. REACT + KEYCLOAK 연동

### 1. Keycloak 기본 설정을 .env에 등록

```
REACT_APP_KEYCLOAK_URL=
REACT_APP_KEYCLOAK_REALM=
REACT_APP_KEYCLOAK_CLIENT_ID=
```

### 2. NPM 패키지 설치

```
@react-keycloak/web
keycloak-js
```

### 3. App.js에 KeycloakProvider 적용

React Router 위에 KeycloakProvider로 감싸서 인증 토큰 관리

```typescript
import React from 'react'
import Keycloak from 'keycloak-js'
import { KeycloakProvider } from '@react-keycloak/web'
import { AppRouter } from './routes'

const keycloak = new Keycloak({
  realm: process.env.REACT_APP_KEYCLOAK_REALM,
  url: process.env.REACT_APP_KEYCLOAK_URL,
  clientId: process.env.REACT_APP_KEYCLOAK_CLIENT_ID,
})
const keycloakProviderInitConfig = {
  onLoad: 'check-sso',
}
class App extends React.PureComponent {
  onKeycloakEvent = (event, error) => {
  console.log('onKeycloakEvent', event, error)
}
onKeycloakTokens = (tokens) => {
  console.log('onKeycloakTokens', tokens)
}
  render() {
    return (
      <KeycloakProvider
        keycloak={keycloak}
        initConfig={keycloakProviderInitConfig}
        onEvent={this.onKeycloakEvent}
        onTokens={this.onKeycloakTokens}
        >
      <AppRouter />
    </KeycloakProvider>
    )
  }
}
export default App
```

## 7. FastAPI 연동

{% code title="# .env" %}
```wikitext
KEYCLOAK_URL=http://localhost:8080
KEYCLOAK_REALM=myrealm
KEYCLOAK_AUDIENCE=myclient        # Keycloak clientId
KEYCLOAK_ALGO=RS256
```
{% endcode %}

{% code title="# auth/keycloak.py" %}
```python
import os
import requests
from fastapi import HTTPException, status, Depends
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import jwt
from pydantic import BaseModel
from typing import List, Dict, Any

KEYCLOAK_URL = os.environ["KEYCLOAK_URL"]
REALM = os.environ["KEYCLOAK_REALM"]
AUDIENCE = os.environ["KEYCLOAK_AUDIENCE"]
ALGO = os.environ.get("KEYCLOAK_ALGO", "RS256")

security = HTTPBearer(auto_error=True)

# JWK 캐시 (단순 예시)
_jwks: Dict[str, Any] | None = None


def get_jwks() -> Dict[str, Any]:
    global _jwks
    if _jwks is None:
        jwks_url = (
            f"{KEYCLOAK_URL}/realms/{REALM}/protocol/openid-connect/certs"
        )
        resp = requests.get(jwks_url)
        if resp.status_code != 200:
            raise RuntimeError("Cannot fetch Keycloak JWKs")
        _jwks = resp.json()
    return _jwks


def get_public_key(header_kid: str) -> str:
    jwks = get_jwks()
    for key in jwks["keys"]:
        if key["kid"] == header_kid:
            from jose.utils import base64url_decode
            # RSA public key 생성 (jose가 내부에서 처리하므로 jwk 직접 전달해도 됨)
            return jwt.algorithms.RSAAlgorithm.from_jwk(key)  # type: ignore
    raise HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Invalid token key id",
    )


class KeycloakUser(BaseModel):
    sub: str
    preferred_username: str | None = None
    email: str | None = None
    roles: List[str] = []


def decode_token(token: str) -> KeycloakUser:
    try:
        unverified_header = jwt.get_unverified_header(token)
        public_key = get_public_key(unverified_header["kid"])

        payload = jwt.decode(
            token,
            public_key,
            algorithms=[ALGO],
            audience=AUDIENCE,
            options={"verify_aud": True},
        )
    except Exception:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token",
        )

    # realm role / client role에서 roles 추출  
    roles: List[str] = []
    resource_access = payload.get("resource_access") or {}
    client_access = resource_access.get(AUDIENCE) or {}
    client_roles = client_access.get("roles") or []
    roles.extend(client_roles)

    realm_access = payload.get("realm_access") or {}
    realm_roles = realm_access.get("roles") or []
    roles.extend(realm_roles)

    ## resource_access[clientId].roles와 realm_access.roles에서 합쳐서 가져오고, 
    ## 이 값을 기반으로 역할 체크
    return KeycloakUser(
        sub=payload.get("sub"),
        preferred_username=payload.get("preferred_username"),
        email=payload.get("email"),
        roles=list(set(roles)),
    )


def get_current_user(
    cred: HTTPAuthorizationCredentials = Depends(security),
) -> KeycloakUser:
    if cred.scheme.lower() != "bearer":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid auth scheme",
        )
    return decode_token(cred.credentials)

```
{% endcode %}

```python
# main.py
from fastapi import FastAPI, Depends, HTTPException, status
from auth.keycloak import get_current_user, KeycloakUser

app = FastAPI()


def require_role(role: str):
    ## get_current_user를 Depends로 쓰면, 
    ## Spring의 JwtAuthenticationToken처럼 FastAPI 라우트에서 바로 유저 정보
    def checker(user: KeycloakUser = Depends(get_current_user)) -> KeycloakUser:
        if role not in user.roles:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Role '{role}' required",
            )
        return user
    return checker


@app.get("/test/anonymous")
def anonymous():
    return {"message": "anonymous ok"}

## Keycloak에 user 역할이 있는 사용자만 접근 가능
@app.get("/test/user")
def user_endpoint(user: KeycloakUser = Depends(require_role("user"))):
    return {
        "message": "user ok",
        "username": user.preferred_username,
        "roles": user.roles,
    }

## admin 역할이 있는 사용자만 접근 가능
@app.get("/test/admin")
def admin_endpoint(user: KeycloakUser = Depends(require_role("admin"))):
    return {
        "message": "admin ok",
        "username": user.preferred_username,
        "roles": user.roles,
    }

```

```ts
// api/client.ts
import axios from "axios";
import { getKeycloak } from "./keycloak"; // keycloak 인스턴스 반환하도록 래핑

const api = axios.create({
  baseURL: "http://localhost:8000",
});

api.interceptors.request.use(async (config) => {
  const keycloak = getKeycloak();
  if (keycloak?.authenticated) {
    // 필요 시 updateToken 호출
    await keycloak.updateToken(30).catch(() => {
      keycloak.logout();
    });
    config.headers = {
      ...config.headers,
      Authorization: `Bearer ${keycloak.token}`,
    };
  }
  return config;
});

export default api;

```
