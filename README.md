# Amazon Cognito

Amazon Cognito는 웹 및 모바일 앱에 대한 인증, 권한 부여 및 사용자 관리를 제공한다. 
사용자는 사용자 이름과 암호를 사용해 직접 로그인하거나 타사 자격 증명 공급자(IdP)를 통해 연동 로그인할 수 있다.

자세한 내용은 [Amazon Cognito 공식 문서](https://docs.aws.amazon.com/ko_kr/cognito/latest/developerguide/what-is-amazon-cognito.html) 에서 확인할 수 있다.

Amazon Cognito 보안 가이드를 설명하기에 앞서, 이해하는 데 도움이 되는 내용들에 대해 먼저 설명한다.


## SAML vs OAuth vs OIDC

#### SAML (Security Assertion Markup Language)

SAML은 인증 정보 제공자(IdP)와 서비스 제공자(SP) 간의 인증 및 인가 데이터를 교환하기 위한 XML 기반의 개방형 표준 데이터 포맷이다.

일반적인 흐름은 다음과 같다.

1. 유저(Browser)는 서비스 공급자(Service Provider)로 접근한다.
2. 서비스 공급자(SP)가 SAML Authentication Request 생성하고, 유저를 통해 인증 제공자(Identity Provider)로 Redirect 한다.
3. 인증 제공자(IdP)는 SAMLRequest 파싱 및 유저 인증을 수행하고, SAML Response 생성한다.
4. 인증 제공자(IdP)가 생성한 SAML Response는 유저를 통해 서비스 공급자(Service Provider)의 Assertion Consumer Service(ACS)로 전달한다.
5. ACS는 SAML Response 유효성 검증 및 요청한 서비스로 유저를 포워딩한다.
6. 유저는 로그인에 성공하고, 서비스를 제공받는다.

※ ACS(Assertion Consumer Service): 인증된 사용자 정보가 포함된 SAML Response 검증 및 서비스 제공을 위한 포워딩을 수행한다.


#### OAuth (Open Authorization) 2.0

OAuth(Open Authorization)는 인터넷 사용자들이 비밀번호를 제공하지 않고 다른 웹사이트 상의 자신들의 정보에 대해 웹사이트나 애플리케이션의 접근 권한을 부여할 수 있는 접근 위임을 위한 개방형 표준이다.

주요 용어는 다음과 같다.

* Authorization Server: 사용자(Resource Owner)를 인증하고, Client로 Access Token 발급해주는 서버
* Client : 서비스 제공자로부터 서비스를 제공받는 서버, 또는 사용자가 접근하고자 하는 웹 애플리케이션
* Resource Owner : 웹 애플리케이션을 사용하는 사용자
* Resource Server : 사용자(Resource Owner)의 보호된 자원(Resource) 가지고 있는 서버


OAuth 2.0 지원하는 권한 부여 유형은 다음과 같으며, 일부는 Deprecated 되었다.

자세한 내용은 [The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749) 에서 확인할 수 있다.

##### Authorization Code

Authorization Code Grant 유형은 기밀 및 공용 클라이언트가 인증 코드를 토큰으로 교환한다. 

Server-Side 앱에 적합하며, 가장 일반적으로 사용하는 방식입니다.  


##### Implicit ***(Deprecated)***

Implicit Grant 유형은 인증 코드 교환 단계없이 액세스 토큰을 반환한다.

모바일 앱 및 SPA와 같이 비밀 또는 토큰을 보호할 수 없는 클라이언트에서 주로 사용된다. 
일반적으로 클라이언트에서 받는다고 확인하지 않고, HTTP 리다이렉션에서 액세스 토큰을 반환하는 내재된 위험으로 인해 권장되지 않는다.


##### Resource Owner Password Credentials ***(Deprecated)***

Resource Owner Password Credentials Grant 유형은 일반적인 대화형 양식을 통해 사용자 자격 증명(사용자 이름 및 암호)을 통해 액세스 토큰을 요청한다.

Client와 Resource Owner 간 높은 신뢰 관계를 가질 수 있을 경우 주로 사용한다. (Client는 사용자 자격 증명를 별도 보관하지 않아야 한다)  


##### Client Credentials

사용자가 아니라 Client Credentials(Client ID, Client Secret) 정보를 인증해 액세스 토큰을 요청한다.

백엔드에서 실행되는 CLI, 데몬 서비스와 같이 데이터에 액세스하기 위해 특정 사용자의 권한이 필요하지 않은 M2M(Machine-to-Machine) 애플리케이션 구성에서 주로 사용한다.


##### Refresh Token

사용 기간이 만료된 액세스 토큰을 사용해 리소스를 요청할 경우, "Invalid Token Error"가 발생한다.

액세스 토큰을 발급 시, Refresh Token 정보를 함께 받았을 경우에는 Refresh Token을 통해 액세스 토큰을 발급할 수 있다.

즉, Resource Owner(사용자)가 로그인 상태가 아니더라도 Refresh Token을 통해 새로운 액세스 토큰을 요청해 필요한 리소스에 접근할 수 있다.
(단, Authorization Code, Resource Owner Password Credentials 유형에서만 지원한다.)
