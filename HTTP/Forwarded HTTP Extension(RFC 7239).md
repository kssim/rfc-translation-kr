## 1. Introduction
   In today's HTTP landscape, there are a multitude of different applications that act as proxies for the user agents.  
   In many cases, these proxies exists without the action or knowledge of the end-user.  
   These cases occur, for example, when the proxy exists as a part of the infrastructure within the organization running the web server.  
   Such proxies may be used for features such as load balancing or crypto offload.  
   Another example is when the proxy is used within the same organization as the user, and the proxy is used to cache resources.  
   However, these proxies make the requests appear as if they originated from the proxy's IP address, and they may change other information in the original request.  
   This represents a loss of information from the original request.  
   
   This loss of information can cause problems for a web server that has a specific use for the clients' IP addresses that will not be met by using the address of the proxy or other information changed by the proxy.  
   The main uses of this information are for diagnostics, access control, and abuse management.  
   Diagnostic functions can include event logging, troubleshooting, and statistics gathering, and the information collected is usually only stored for short periods of time and only gathered in response to a particular problem or a complaint from the client.  
   Access control can be operated by configuring a list of client IP addresses from which access is permitted, but this approach will not work if a proxy is used, unless the proxy is trusted and is, itself, configured with a list of allowed client addresses for the server.    
   Cases of abuse require identification of the abuser and this uses many of the same features identified for diagnostics.  

   Most of the time that a proxy is used, this loss of information is not the primary purpose, or even a desired effect, of using the proxy.  
   Thus, to restore the desired functionality when a proxy is in use, a way of disclosing the original information at the HTTP level is needed.  
   Clearly, however, when the purpose of using a proxy is to provide client anonymity, the proxy will not use the feature defined in this document.  

   It should be noted that the use of a reverse proxy also hides information.  
   Again, where the loss of information is not a deliberate function of the use of the reverse proxy, it can be desirable to find a way to encode the information within the HTTP messages so that the consumer can see it.  

   A common way to disclose this information is by using the non-standard header fields such as X-Forwarded-For, X-Forwarded-By, and X-Forwarded-Proto.  
   There are many benefits to using a standardized approach to commonly desired protocol function: not least is interoperability between implementations.  
   This document standardizes a header field called "Forwarded" and provides the syntax and semantics for disclosing such information.  
   "Forwarded" also combines all the information within one single header field, making it possible to correlate that information.  
   With the header field format described in this document, it is possible to know what information belongs together, as long as the proxies are trusted.  
   Such conclusions are not possible to make with the X-Forwarded class of header fields.  
   The header field defined in this document is optional such that implementations of proxies that are intended to provide privacy are not required to operate or implement the header field.  

   Note that similar issues to those described for proxies also arise with use of NATs.  
   This is discussed further in [RFC6269](https://tools.ietf.org/html/rfc6269).  


## 1. 소개
   오늘날의 HTTP환경에서는 user agent의 proxy역할을 하는 다양한 응용 프로그램들이 있다.  
   대부분의 경우에 이러한 proxy는 end-user의 행동이나 지식 없이 존재한다.  
   예를 들어, 이런 경우는 proxy가 웹서버를 운영하는 환경 안에서 인프라의 일부로 존재할때 발생한다.  
   이러한 proxy들은 아마 load balancing이나 crypto offload의 기능으로 사용될 것이다.  
   또 다른 경우는 proxy가 사용자와 같은 환경 안에서 사용되고, proxy가 자원을 캐시하는 용도로 사용될 때이다.  
   그러나 이런 proxy들은 proxy의 IP 주소로부터 request가 온 것처럼 표시하고, 본래의 request 안에서 다른 정보들을 변경할 수도 있다.  
   이것은 본래의 request로부터 정보가 손실되었다는 것을 보여준다.  

   이런 정보의 손실은 proxy의 주소 사용으로 인해 접근할 수 없거나, proxy에 의해 다른 정보가 변경된 client의 IP 주소를 특별한 용도로 사용하는 웹서버에 문제를 발생시킬 수 있다.  
   정보의 주된 용도는 진단, 접근 제어, 남용 관리이다.  
   진단 기능에는 이벤트 로깅, troubleshooting 그리고 통계 수집이 포함될 수 있으며, 수집된 정보는 보통 짧은 기간 동안에만 저장된다.  
   그리고 client 의 불만이나 특정 문제에 따른 응답으로 수집된다.  
   접근 제어는 접근이 허용된 client IP 주소 목록을 구성하여 동작시킬 수 있다.  
   하지만, proxy가 그 자체로 신뢰되거나 접근이 허용된 client IP 주소 목록에 포함되어 설정되어 있는 경우를 제외하곤 동작하지 않는다.  
   남용의 경우, 남용하는 사용자의 식별이 필요로 하고, 진단을 위한 식별된 것과 같은 동일한 많은 기능을 사용한다.  

   proxy가 사용되는 대부분의 경우, 이러한 정보의 손실은 proxy를 사용하는 데에 있어 주된 목적이 아니고 심지어는 의도된 효과도 아니다.  
   따라서, proxy를 사용할때 의도된 기능대로 복원하려면, HTTP 수준에서 본래의 정보를 보여주는 방법이 필요하다.  
   그러나 proxy를 사용하는 목적이 client의 익명성을 보장하기 위해서라면, 이 문서에서 정의된 proxy에서는 관련된 내용을 다루지 않는다.  

   reverse proxy를 사용해도 정보가 숨겨질 수 있다.  
   다시 말해서, reverse proxy를 사용할때 정보의 손실이 발생하는 것이 의도적인 것이 아니라면, 사용자가 내용을 볼 수 있도록 HTTP 메시지 안에서 정보를 인코딩하는 방법을 찾는 것이 더 나을 수 있다.  

   이러한 정보를 표시하는 가장 흔한 방법은 X-Forwarded-For, X-Forwarded-By 그리고 X-Forwarded-Proto 와 같은 비표준 헤더 필드를 사용하는 것이다.  
   공통적으로 요구되는 프로토콜 기능에 대한 표준화된 접근 방법을 사용하는 것은 많은 이점이 있다. 특히 구현 간에 상호 의존성에 있어서는 이점이 있다.  
   이 문서는 "Forwarded" 라고 불리는 헤더 필드를 표준화하고, 이런 정보를 보여주기 위한 문법과 구문을 제공한다.  
   "Forwarded"는 또한 하나의 단일 헤더 필드 안의 모든 정보를 결합하여, 상호 연관 시킬수도 있다.  
   이 문서에 적힌 헤더 필드 형식은 proxy가 신뢰되는 한 어떠한 정보들이 함께 속해있는지 알 수 있도록 해준다.  
   위와 같은 기능은 헤더 필드의 X-Forwarded 클래스만으로는 불가능하다.  
   이 문서에 적힌 헤더 필드는 선택적인 사항이므로, privacy를 제공하려는 proxy의 구현서는 헤더 필드를 조작하거나 별도로 구현할 필요가 없다.  

   proxy에서 묘사된 것과 유사한 이슈들은 NAT를 사용함에 있어서도 동일하게 나타난다.  
   이러한 논의는 [RFC6269](https://tools.ietf.org/html/rfc6269)에서 더 다뤄진다.  




## 2. Notational Conventions
   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).


## 2. 표기 규칙
   이 문서에서 사용된 "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" 의 키워드들은 [RFC2119](https://tools.ietf.org/html/rfc2119)에서 정의된대로 해석되어야 한다.




## 3. Syntax Notations
   This specification uses the Augmented Backus-Naur Form (ABNF) notation of [RFC5234](https://tools.ietf.org/html/rfc5234) with the list rule extension defined in [Section7 of RFC7230](#7. Implementation Considerations).  


## 3. 구문 표기법
   이 명세는 [RFC7230의 섹션7](#7. 구현시 고려사항)에 정의된 목록 규칙 확장과 함께 [RFC5234](https://tools.ietf.org/html/rfc5234)의 Augmented Backus-Naur Form(ABNF) 표기법을 사용한다.  




## 4. Forwarded HTTP Header Field
   The "Forwarded" HTTP header field is an OPTIONAL header field that, when used, contains a list of parameter-identifier pairs that disclose information that is altered or lost when a proxy is involved in the path of the request.  
   Due to the sensitive nature of the data passed in this header field (see Sections [8.2](#8.2. Information Leak) and [8.3](#8.3. Privacy Considerations)), this header field should be turned off by default.  
   Further, each parameter should be configured individually.  "Forwarded" is only for use in HTTP requests and is not to be used in HTTP responses.  
   This applies to forwarding proxies, as well as reverse proxies.  
   Information passed in this header field can be, for example, the source IP address of the request, the IP address of the incoming interface on the proxy, or whether HTTP or HTTPS was used.  
   If the request is passing through several proxies, each proxy can add a set of parameters; it can also remove previously added "Forwarded" header fields.  
   The top-level list is represented as a list of HTTP header field-values as defined in [Section 3.2 of RFC7230](https://tools.ietf.org/html/rfc7230#section-3.2).  
   The first element in this list holds information added by the first proxy that implements and uses this header field, and each subsequent element holds information added by each subsequent proxy.  
   Because this header field is optional, any proxy in the chain may choose not to update this header field.  
   Each field-value is a semicolon-separated list; this sublist consists of parameter-identifier pairs.  
   Parameter-identifier pairs are grouped together by an equals sign.  
   Each parameter MUST NOT occur more than once per field-value.  
   The parameter names are case-insensitive.  
   The header field value can be defined in ABNF syntax as:  
   <pre><code>
   Forwarded   = 1#forwarded-element
       forwarded-element =
           [ forwarded-pair ] *( ";" [ forwarded-pair ] )
       forwarded-pair = token "=" value
       value          = token / quoted-string
       token = "Defined in [RFC7230, Section 3.2.6](https://tools.ietf.org/html/rfc7230#section-3.2.6)"
       quoted-string = "Defined in [RFC7230, Section 3.2.6](https://tools.ietf.org/html/rfc7230#section-3.2.6)"  
   Examples:
       Forwarded: for="_gazonk"
       Forwarded: For="[2001:db8:cafe::17]:4711"
       Forwarded: for=192.0.2.60;proto=http;by=203.0.113.43
       Forwarded: for=192.0.2.43, for=198.51.100.17
   </code></pre>
   Note that as ":" and "[]" are not valid characters in "token", IPv6 addresses are written as "quoted-string".  

   A proxy server that wants to add a new "Forwarded" header field value can either append it to the last existing "Forwarded" header field after a comma separator or add a new field at the end of the header block.  
   A proxy MAY remove all "Forwarded" header fields from a request.  
   It MUST, however, ensure that the correct header field is updated in case of multiple "Forwarded" header fields.  


## 4. Forwarded HTTP 헤더 필드
   "Forwarded" HTTP 헤더 필드는 선택적으로 사용할 수 있는 필드이다.  
   이 필드가 사용될때, proxy가 request경로의 포함되어있으면, proxy에 의해 변경되거나 손실되는 정보를 알려주는 파라미터-식별자의 목록이 포함된다.  
   이 헤더 필드([8.2](#8.2. 정보의 누출) 및 [8.3](#8.3. 개인정보 고려사항)절 참조)에서 전달되는 데이터의 민감한 특성으로 인해, 이 헤더 필드는 기본 값이 꺼져있는 상태여야 한다.  
   게다가, 각각의 파라미터는 개별적으로 설정되어야 한다.  
   "Forwarded"는 HTTP request에서만 사용되고, HTTP response에서는 사용되지 않는다.  
   이 규칙은 forwarding proxy뿐만 아니라 reverse proxy에서도 동일하게 적용된다.  
   이 헤더 필드에서 전달되는 정보는 request의 source IP, proxy의 수신 인터페이스 IP 또는 HTTP나 HTTPS의 사용 여부가 될 수도 있다.  
   만약 request가 여러개의 proxy를 지나서 전달된다면, 각각의 proxy는 파라미터들의 모음으로 더해질 수 있다.  
   또, 이전의 추가된 "Forwarded" 헤더 필드의 값을 제거할 수도 있다.  
   최상위 목록은 [RFC 7230의 섹션 3.2](https://tools.ietf.org/html/rfc7230#section-3.2)에서 정의된 대로 HTTP 헤더 필드의 값 목록으로 표시된다.  
   이 목록의 첫번째 요소는 이 헤더 필드를 구현하고, 사용하는 첫번째 proxy 에 의해 추가된 정보를 가지고 있고, 그 다음에 나오는 요소들은 이후 각각의 proxy에서 추가된 정보들을 가지고 있다.  
   이 헤더 필드는 선택적이기 때문에, proxy 체인에 있는 어떤 proxy는 이 헤더 필드에 정보를 업데이트하지 않을 수도 있다.  
   각각의 필드와 값은 세미콜론으로 구분된 목록이고, 이 하위 목록은 파라미터와 식별자의 쌍으로 구성된다.  
   파라미터와 식별자의 쌍은 등호를 사용하여 그룹화 된다. 각각의 파라미터는 필드와 값에 두번 이상 나타나면 안된다.  
   파라미터의 이름은 대소문자를 구별하지 않는다.  
   헤더 필드 값은 ABNF 구문으로 다음과 같이 정의될 수 있다.  
   <pre><code>
   Forwarded   = 1#forwarded-element
       forwarded-element =
           [ forwarded-pair ] *( ";" [ forwarded-pair ] )
       forwarded-pair = token "=" value
       value          = token / quoted-string
       token = "Defined in [RFC7230, Section 3.2.6](https://tools.ietf.org/html/rfc7230#section-3.2.6)"
       quoted-string = "Defined in [RFC7230, Section 3.2.6](https://tools.ietf.org/html/rfc7230#section-3.2.6)"  
   Examples:
       Forwarded: for="_gazonk"
       Forwarded: For="[2001:db8:cafe::17]:4711"
       Forwarded: for=192.0.2.60;proto=http;by=203.0.113.43
       Forwarded: for=192.0.2.43, for=198.51.100.17
   </code></pre>
   ":" 및 "[]"는 "토큰"에서 유효한 문자가 아니므로, IPv6의 주소는 따옴표로 감싸진 문자열로 작성된다.  

   새로운 "Forwarded" 헤더 필드 값을 더하려면, proxy 서버는 쉼표 구분자 뒤에 오는 마지막 "Forwarded" 헤더 필드에 추가하거나, 헤더 블록의 끝에 새 필드를 추가할 수 있다.  
   proxy 는 request로 부터 모든 "Forwarded" 헤더 필드를 제거할 수도 있다.  
   그러나, "Forwarded" 헤더 필드가 여러개인 경우, 반드시 정확한 헤더 필드 값이 업데이트 되었는지 확인되어야 한다.  




## 5. Parameters
   This document specifies a number of parameters and valid values for each of them:  
   * "by" identifies the user-agent facing interface of the proxy.  
   * "for" identifies the node making the request to the proxy.  
   * "host" is the host request header field as received by the proxy.  
   * "proto" indicates what protocol was used to make the request.  


## 5. 파라미터
   이 문서는 각각의 파라미터와 그에 대한 유효한 값들을 명세하고 있다.  
   * "by"는 user-agent와 마주하고 있는 proxy 인터페이스를 식별한다.  
   * "for"는 proxy에 요청하는 노드를 식별한다.  
   * "host"는 proxy로 부터 수신된 request 헤더 필드의 host이다.  
   * "proto"는 request를 보내기 위해 어떤 프로토콜이 사용되었는지를 가리킨다.  




### 5.1. Forwarded By
   The "by" parameter is used to disclose the interface where the request came in to the proxy server.  
   When proxies choose to use the "by" parameter, its default configuration SHOULD contain an obfuscated identifier as described in [Section 6.3](#6.3. Obfuscated Identifier).  
   If the server receiving proxied requests requires some address-based functionality, this parameter MAY instead contain an IP address (and, potentially, a port number).  
   A third option is the "unknown" identifier described in [Section 6.2](#6.2. The "unknown" Identifier).  

   The syntax of a "by" value, after potential quoted-string unescaping, conforms to the "node" ABNF described in [Section 6](#6. Node Identifiers).  

   This is primarily added by reverse proxies that wish to forward this information to the backend server.  
   It can also be interesting in a multihomed environment to signal to backend servers from which the request came.  


### 5.1 Forwarded By
   "by" 파라미터는 proxy server로 들어온 request가 어떤 인터페이스에서 왔는지 나타내는데 사용된다.  
   proxy가 "by" 파라미터를 사용하려면, 기본 설정으로 [섹션 6.3](#6.3. 난독화된 식별자)에 기술된 난독화된 식별자를 포함해야한다.  
   만약 서버가 어떤 주소기반의 기능을 필요로 하는 프락시된 request를 받는 다면, 이 파라미터에 IP 주소가 포함될 수도 있다.  
   (그리고, 잠재적으로 포트 번호도 포함될 수 있다.)  
   세번째로 옵션은 [섹션 6.2](#6.2. "unknown" 식별자)에서 서술된 "unknown" 식별자이다.  

   "by" 값의 구문은 [섹션 6](#6. 노드 식별자) 안에서 기술된 "node" ABNF를 따르도록 한다.  

   "by"는 정보를 backend 서버에 전달하려고 하는 reverse proxy에 의해 처음 추가되었다.  
   또한 "by"는 multihomed 환경에서 request가 오는 backend 서버로 신호를 보내는데에도 사용될 수 있다.  





### 5.2 Forwarded For
   The "for" parameter is used to disclose information about the client that initiated the request and subsequent proxies in a chain of proxies.  
   When proxies choose to use the "for" parameter, its default configuration SHOULD contain an obfuscated identifier as described in [Section 6.3](#6.3. Obfuscated Identifier).  
   If the server receiving proxied requests requires some address-based functionality, this parameter MAY instead contain an IP address (and, potentially, a port number).  
   A third option is the "unknown" identifier described in [Section 6.2](#6.2. The "unknown" Identifier).  

   The syntax of a "for" value, after potential quoted-string unescaping, conforms to the "node" ABNF described in [Section 6](#6. Node Identifiers).  

   In a chain of proxy servers where this is fully utilized, the first "for" parameter will disclose the client where the request was first made, followed by any subsequent proxy identifiers.  
   The last proxy in the chain is not part of the list of "for" parameters.  
   The last proxy's IP address, and optionally a port number, are, however, readily available as the remote IP address at the transport layer.  
   It can, however, be more relevant to read information about the last proxy from preceding "Forwarded" header field's "by" parameter, if present.  


### 5.2 Forwarded For
   "for" 파라미터는 proxy 체인에서 요청 및 후속 proxy를 시작한 client의 정보를 보여주기 위해서 사용된다.  
   proxy가 "for" 파라미터를 사용하려면, 기본 설정으로 [섹션 6.3](#6.3. 난독화된 식별자)에 기술된 난독화된 식별자를 포함해야한다.  
   만약 서버가 어떤 주소기반의 기능을 필요로 하는 프락시된 request를 받는 다면, 이 파라미터에 IP 주소가 포함될 수도 있다.
   (그리고, 잠재적으로 포트 번호도 포함될 수 있다.)  
   세번째로 옵션은 [섹션 6.2](#6.2. "unknown" 식별자)에서 서술된 "unknown" 식별자이다.  

   "for" 값의 구문은 [섹션 6](#6. 노드 식별자) 안에서 기술된 "node" ABNF를 따르도록 한다.  

   이 기능이 완전히 사용되는 proxy 서버의 체인에서 첫번째 "for" 파라미터는 요청이 처음 만들어진 클라이언트를 보여주고, 체인의 다음 proxy 식별자가 뒤에 오게 된다.  
   proxy 체인에서 가장 마지막에 있는 proxy는 "for" 파라미터 목록에 포함되지 않는다.  
   그러나, 가장 마지막 proxy 의 IP 주소, 그리고 포트는 transport 계층에서 remote IP 주소로 사용할 수 있다.  
   하지만 가능할 뿐이지, "Forwarded" 헤더 필드의 "by" 파라미터가 있다면, 그곳에 적힌 마지막 proxy의 정보를 읽는 것이 더 적절할 수 있다.  




### 5.3 Forwarded Host
   The "host" parameter is used to forward the original value of the "Host" header field.  
   This can be used, for example, by the origin server if a reverse proxy is rewriting the "Host" header field to some internal host name.  

   The syntax for a "host" value, after potential quoted-string unescaping, MUST conform to the Host ABNF described in [Section 5.4 of RFC7230](https://tools.ietf.org/html/rfc7230#section-5.4).  


### 5.3 Forwarded Host
   "host" 파라미터는 본래의 "host" 헤더 필드 값을 전달하기 위해 사용된다.  
   예를 들어, reverse proxy가 "host" 헤더 필드를 어떤 내부의 호스트 이름으로 다시 쓰는 경우, 본래의 서버에서 이를 사용할 수 있다.  

   "host" 값의 구문은 반드시 [RFC 7230의 섹션 5.4](https://tools.ietf.org/html/rfc7230#section-5.4) 안에서 기술된 Host ABNF를 따르도록 한다. 




### 5.4 Forwarded Proto
   The "proto" parameter has the value of the used protocol type.  
   The syntax of a "proto" value, after potential quoted-string unescaping, MUST conform to the URI scheme name as defined in [Section 3.1 in RFC3986](https://tools.ietf.org/html/rfc3986#section-3.1) and registered with IANA according to [RFC4395](https://tools.ietf.org/html/rfc4395).  
   Typical values are "http" or "https".  

   For example, in an environment where a reverse proxy is also used as a crypto offloader, this allows the origin server to rewrite URLs in a document to match the type of connection as the user agent requested, even though all connections to the origin server are unencrypted HTTP.  


### 5.4 Forwarded Proto
   "proto" 파라미터에는 사용된 프로토콜의 타입에 대한 값이 있다.  
   "proto" 값의 구문은 반드시 [RFC 3986의 섹션 3.1](https://tools.ietf.org/html/rfc3986#section-3.1) 안에서 기술된 URI 스키마 이름을 따르도록 하고, [RFC 4395](https://tools.ietf.org/html/rfc4395)에 따라 IANA에 등록되어야 한다.  
   일반적인 값은 "http" 나 "https" 이다.  

   예를 들어, reverse proxy가 crypto offloader로도 사용되는 환경에서는 원본 서버가 문서안에 있는 URL들을 user agent가 요청한 유형과 일치하도록 다시 작성할 수 있도록 한다.  
   심지어 모든 원본 서버로의 모든 연결이 암호화되지 않은 HTTP 하더라도.  




### 5.5 Extensions
   Extensions allow for additional parameters and values.  
   Extensions can be particularly useful in reverse proxy environments.  
   All extension parameters SHOULD be registered in the "HTTP Forwarded Parameter" registry.  
   If certain extensions are expected to have widespread deployment, they SHOULD also be standardized.  
   This is further discussed in Section 9.  


### 5.5 확장
   확장은 추가적인 파라미터와 값을 허용한다.  
   확장은 특히 reverse proxy 환경에서 유용하게 사용될 수 있다.  
   모든 확장 파라미터는 "HTTP Forwarded Parameter" 레지스트리에 등록되어야 한다.  
   만약 특정 확장이 널리 배포될 것으로 예상된다면, 그것에 대한 표준화도 고려되어야 한다.  
   이것에 대해서는 섹션 9에서 다시 이야기 논의 된다.  



## 6. Node Identifiers
   The node identifier is one of the following:  
   * The client's IP address, with an optional port number  
   * A token indicating that the IP address of the client is not known to the proxy server  
   * A generated token, allowing for tracing and debugging, while allowing the internal structure or sensitive information to be hidden  

   The node identifier is defined by the ABNF syntax as:
   <pre><code>
   node     = nodename [ ":" node-port ]
   nodename = IPv4address / "[" IPv6address "]" /
                   "unknown" / obfnode  
   IPv4address = "Defined in [RFC3986, Section 3.2.2](https://tools.ietf.org/html/rfc3986#section-3.2.2)"
   IPv6address = "Defined in [RFC3986, Section 3.2.2](https://tools.ietf.org/html/rfc3986#section-3.2.2)"
   obfnode = "_" 1*( ALPHA / DIGIT / "." / "_" / "-")  
   node-port     = port / obfport
   port          = 1*5DIGIT
   obfport       = "_" 1*(ALPHA / DIGIT / "." / "_" / "-")  
   DIGIT = "Defined in [RFC5234, Section 3.4](https://tools.ietf.org/html/rfc5234#section-3.4)"
   ALPHA = "Defined in [RFC5234](https://tools.ietf.org/html/rfc5234#section-3.4), Section B.1"
   </code></pre>

   Each of the identifiers may optionally have the port identifier, for example, allowing the identification of the endpoint in a NATed environment.  
   The "node-port" can be identified either by its port number or by a generated token obfuscating the real port number.  
   An obfuscated port may be used in situations where the possessor of the proxy wants the ability to trace requests -- for example, in debug purposes -- but does not want to reveal internal information.  

   Note that the ABNF above also allows port numbers to be appended to the "unknown" identifier.  
   Interpretation of such notation is, however, left to the possessor of a proxy adding such a value to the header field.  
   To distinguish an "obfport" from a port, the "obfport" MUST have a leading underscore.  
   Further, it MUST also consist of only "ALPHA", "DIGIT", and the characters ".", "_", and "-".  

   It is important to note that an IPv6 address and any nodename with node-port specified MUST be quoted, since ":" is not an allowed character in "token".  
   <pre><code>
   Examples:
            "192.0.2.43:47011"
            "[2001:db8:cafe::17]:47011"
   </code></pre>


## 6. 노드 식별자
   노드 식별자는 다음 중 하나다.  
   * client의 IP 주소 (선택사항인 클라이언트 포트 넘버 포함)  
   * client의 IP 주소가 proxy 서버에 알려지지 않았음을 가르키는 토큰  
   * tracing과 debugging을 허용하면서, 내부 구조나 민감한 정보를 숨길수 있도록 생성된 토큰  

   노드 식별자는 ABNF 구문에 의해서 다음과 같이 정의된다.  
   <pre><code>
   node     = nodename [ ":" node-port ]
   nodename = IPv4address / "[" IPv6address "]" /
                   "unknown" / obfnode  
   IPv4address = "Defined in [RFC3986, Section 3.2.2](https://tools.ietf.org/html/rfc3986#section-3.2.2)"
   IPv6address = "Defined in [RFC3986, Section 3.2.2](https://tools.ietf.org/html/rfc3986#section-3.2.2)"
   obfnode = "_" 1*( ALPHA / DIGIT / "." / "_" / "-")  
   node-port     = port / obfport
   port          = 1*5DIGIT
   obfport       = "_" 1*(ALPHA / DIGIT / "." / "_" / "-")  
   DIGIT = "Defined in [RFC5234, Section 3.4](https://tools.ietf.org/html/rfc5234#section-3.4)"
   ALPHA = "Defined in [RFC5234](https://tools.ietf.org/html/rfc5234#section-3.4), Section B.1"
   </code></pre>

   각각의 식별자는 선택적으로 포트 식별자를 가질수 있다.  
   예를 들어, NATed 환경에서 종단점을 식별할 수 있도록 할 수 있다.  
   "node-port"는 포트 번호나 실제 포트 번호를 난독화하여 생성된 토큰으로 식별할 수 있다.  
   난독화된 포트는 proxy의 소유자가 request의 추적 기능을 원하는 상황에 사용될 수 있다.  
   -- 예를 들어, debug 목적으로 사용하길 원하지만, -- 내부 정보를 들어내고 싶지 않을 때 사용한다.  

   위의 ABNF는 포트 번호가 "unknown" 식별자에 추가될 수 있도록 허용한다.  
   그러나, 그러한 표기법의 해석은 헤더 필드에 그러한 값을 추가하는 proxy의 소유자에게 남겨진다.  
   "obfport"와 포트를 구별하기 위해 "obfport"의 앞에는 반드시 밑줄이 있어야 한다.  
   게다가 "obfport"는 "ALPHA", "DIGIT", 그리고 ".", "\_", "-" 문자로만 구성되어야 한다.  

   ":"는 "token"에서 허용된 문자가 아니기 때문에 IPv6 주소와 node-port가 지정된 모든 노드 이름은 반드시 인용되어야 한다.  
   <pre><code>
   Examples:
            "192.0.2.43:47011"
            "[2001:db8:cafe::17]:47011"
   </code></pre>




### 6.1. IPv4 and IPv6 Identifiers
   The ABNF rules for "IPv6address" and "IPv4address" are defined in [RFC3986](https://tools.ietf.org/html/rfc3986).  The "IPv6address" SHOULD comply with textual representation recommendations [RFC5952](https://tools.ietf.org/html/rfc5952) for example, lowercase, compression of zeros.  

   Note that the IP address may be one from the internal nets, as defined in [RFC1918](https://tools.ietf.org/html/rfc1918) and [RFC4193](https://tools.ietf.org/html/rfc4193).  Also, note that an IPv6 address is always enclosed in square brackets.  


### 6.1. IPv4와 IPv6 식별자
   "IPv6address"와 "IPv4address"를 위한 ABNF 규칙은 [RFC 3986](https://tools.ietf.org/html/rfc3986)에 정의되어 있다.  
   "IPv6address"는 텍스트 표현 권고[RFC 5952](https://tools.ietf.org/html/rfc5952) 예를 들어 소문자나 0의 압축 등을 준수해야 한다.  
   
   IP 주소는 [RFC 1918](https://tools.ietf.org/html/rfc1918)과 [RFC 4193](https://tools.ietf.org/html/rfc4193)에 정의된 바와 같이 내부 망으로부터의 것일 수 있다.  
   또한, IPv6 주소는 항상 대괄호로 묶는다.  




### 6.2. The "unknown" Identifier
   The "unknown" identifier is used when the identity of the preceding entity is not known, but the proxy server still wants to signal that a forwarding of the request was made.  
   One example would be a proxy server process generating an outgoing request without direct access to the incoming request TCP socket.  


### 6.2. "unknown" 식별자
   "unknown" 식별자는 선행 entity의 신원을 알 수 없을 때 사용되지만, proxy 서버는 여전히 요청 전달이 이루어졌음을 알리고 싶어한다.  
   한 예로 incoming request TCP 소켓의 직접적인 접근 없이 outgoing request 를 생성하는 proxy 서버 프로세스가 있다.  




### 6.3. Obfuscated Identifier
   A generated identifier may be used where there is a wish to keep the internal IP addresses secret, while still allowing the "Forwarded" header field to be used for tracing and debugging.  
   This can also be useful if the proxy uses some sort of interface labels and there is a desire to pass them rather than an IP address.  
   Unless static assignment of identifiers is necessary for the server's use of the identifiers, obfuscated identifiers SHOULD be randomly generated for each request.  
   If the server requires that identifiers persist across requests, they SHOULD NOT persist longer than client IP addresses.  
   To distinguish the obfuscated identifier from other identifiers, it MUST have a leading underscore "\_".  
   Furthermore, it MUST also consist of only "ALPHA", "DIGIT", and the characters ".", "\_" and "-".  
   <pre><code>
   Example:
       Forwarded: for=_hidden, for=_SEVKISEK
   </code></pre>


### 6.3. 난독화된 식별자
   생성된 식별자는 "Forwarded" 헤더 필드를 tracing과 debugging 용도로 사용할 수 있도록 해주면서, 내부 IP 주소를 비밀로 유지할 때 사용될 수 있다.  
   이것은 proxy가 어떤 인터페이스 라벨의 종류를 사용하고 IP 주소대신 라벨을 전달하려는 경우에 유용할 수 있습니다.  
   식별자의 정적 할당이 서버 식별자 사용에 필요하지 않는 한, 각각의 request를 위한 난독화된 식별자는 랜덤으로 생성되어야 한다.  
   서버가 요청간에 식별자가 지속적으로 유지되길 원한하는 경우, client IP의 유지기간 보다 길어서는 안된다.  
   다른 식별자들로 부터 난독화된 식별자를 구분하기 위해서는 난독화된 식별자 앞에 "\_"를 붙여야 한다.  
   게다가 반드시 "ALPHA", "DIGIT", 그리고 ".", "\_", "-" 문자로만 구성되어야 한다.  
   <pre><code>
   Example:
       Forwarded: for=_hidden, for=_SEVKISEK
   </code></pre>




## 7. Implementation Considerations
### 7.1 HTTP Lists
   Note that an HTTP list allows white spaces to occur between the identifiers, and the list may be split over multiple header fields.  
   <pre><code>
   As an example, the header field
       Forwarded: for=192.0.2.43,for="[2001:db8:cafe::17]",for=unknown  
   is equivalent to the header field
       Forwarded: for=192.0.2.43, for="[2001:db8:cafe::17]", for=unknown  
   which is equivalent to the header fields
       Forwarded: for=192.0.2.43
       Forwarded: for="[2001:db8:cafe::17]", for=unknown
   </code></pre>


## 7. 구현시 고려사항
### 7.1. HTTP 리스트
   HTTP 리스트는 식별자 간의 공백을 허용하고, 여러 헤더 필드들로 분리 될 수 있다.  
   <pre><code>
   As an example, the header field
       Forwarded: for=192.0.2.43,for="[2001:db8:cafe::17]",for=unknown  
   is equivalent to the header field
       Forwarded: for=192.0.2.43, for="[2001:db8:cafe::17]", for=unknown  
   which is equivalent to the header fields
       Forwarded: for=192.0.2.43
       Forwarded: for="[2001:db8:cafe::17]", for=unknown
   </code></pre>




### 7.2. Header Field Preservation
   There are some cases when this header field should be kept and some cases where it should not be kept.
   A directly forwarded request should preserve and possibly extend it.  
   If a single incoming request causes the proxy to make multiple outbound requests, special care must be taken to decide whether or not the header field should be preserved.  
   In many cases, the header field should be preserved, but if the outbound request is not a direct consequence of the incoming request, the header field should not be preserved.  
   Consider also the case when a proxy has detected a content mismatch in a 304 response and is following the instructions in [RFC7232, Section 4.1](https://tools.ietf.org/html/rfc7232#section-4.1) to repeat the request unconditionally, in which case the new request is still basically a direct consequence of the origin request, and the header field should probably be kept.  


### 7.2. 헤더 필드 보존
   헤더 필드가 유지되어야 하는 경우가 있고, 유지되지 않아야 하는 경우가 있다.  
   직접적으로 전달되는 request는 헤더 필드가 유지되고 가능한 확장되어야 한다.  
   만약 단일 incoming request로 인해 proxy가 여러개의 outbound request들을 생성하는 경우, 헤더 필드를 보존할지 말지에 대해 결정하는 것에 특히 신경써야한다.  
   많은 경우에 헤더 필드는 보존된다, 하지만 만약 outbound request가 incoming request의 직접적인 결과가 아니라면, 헤더 필드는 보존되면 않된다.  
   proxy가 304 응답에서 content가 불일치하는 경우를 감지하고, [RFC 7232의 섹션 4.1](https://tools.ietf.org/html/rfc7232#section-4.1)을 따라 무조건 request를 반복하는 경우도 고려해야한다.  
   이 경우에 새로운 request는 기본적으로 원본 request의 직접적인 결과이고, 헤더 필드는 유지될 것이다.  




### 7.3. Relation to Via
   The "Via" header field (see [RFC7230, Section 5.7.1](https://tools.ietf.org/html/rfc7230#section-5.7.1)) is a header field with a similar use case as this header field.  
   The "Via" header field, however, only provides information about the proxy itself, and thereby leaves out the information about the client connecting to the proxy server.  
   The "Forwarded" header field, on the other hand, has relaying information from the client-facing side of the proxy server as its main purpose.  
   As "Via" is already widely deployed, its format cannot be changed to address the problems that "Forwarded" addresses.  
   Note that it is not possible to combine information from this header field with the information from the Via header field.  
   Some proxies will not update the "Forwarded" header field, some proxies will not update the Via header field, and some proxies will update both.  


### 7.3. Relation to Via
   "Via" 헤더 필드([RFC 7230 섹션 5.7.1](https://tools.ietf.org/html/rfc7230#section-5.7.1))는 이 헤더 필드와 유사한 유스 케이스를 가진 헤더 필드이다.  
   그러나, "Via" 헤더 필드는 오직 proxy에 대한 정보만을 제공하고, proxy 서버에 연결된 client에 관한 정보는 제공하지 않는다.  
   반면에 "Forwarded" 헤더 필드는 proxy 서버의 client 측으로부터 정보를 중계하는 역할을 주 목적으로 한다.  
   "Via"는 이미 광범위하게 배포되었기 때문에 "Forwarded" 주소 문제를 해결하기 위해 형식을 변경할 수는 없다.  
   이 헤더 필드의 정보와 Via 헤더 필드의 정보는 결합 할 수 없다.  
   일부 proxy는 "Forwarded" 헤더 필드를 업데이트하지 않을 것이고, 일부 proxy는 Via 헤더 필드를 업데이트 하지 않을 것이다.  
   그리고 일부 proxy는 둘다를 업데이트 할 것이다.  




### 7.4. Transition
   If a proxy gets incoming requests with X-Forwarded-* header fields present, it is encouraged to convert these into the header field described in this document, if it can be done in a sensible way.  
   If the request only contains one type -- for example, X-Forwarded-For -- this can be translated to "Forwarded", by prepending each element with "for=".  
   Note that IPv6 addresses may not be quoted in X-Forwarded-For and may not be enclosed by square brackets, but they are quoted and enclosed in square brackets in "Forwarded".  
   <pre><code>
       X-Forwarded-For: 192.0.2.43, 2001:db8:cafe::17  
   becomes:
       Forwarded: for=192.0.2.43, for="[2001:db8:cafe::17]"
   </code></pre>

   However, special care must be taken if, for example, both X-Forwarded-For and X-Forwarded-By exist.  
   In such cases, it may not be possible to do a conversion, since it is not possible to know in which order the already existing fields were added.  
   Also, note that removing the X-Forwarded-For header field may cause issues for parties that have not yet implemented support for this new header field.  


### 7.4. 변환
   proxy가 incoming request를 X-Forwarded-* 헤더 필드를 가진 상태로 받는다면, 이를 합리적인 방법으로 처리하기 위해 이 문서에 적혀있는 헤더 필드로 변환하는 것이 좋다.  
   만약 request가 오직 하나의 타입만 가지고 있다면 -- 예를 들어, X-Forwarded-For -- 각 요소 앞에 "for="를 추가하여, "Forwarded"로 변환할 수 있다.  
   IPv6는 아마도 X-Forwarded-For 에서 따옴표로 감싸지지 않고, 대괄호로도 묶이지 않을 것이다.  
   하지만, Forwarded 안에서는 대괄호로 묶이고, 따옴표로 감싸질 것이다.  
   <pre><code>
       X-Forwarded-For: 192.0.2.43, 2001:db8:cafe::17  
   becomes:
       Forwarded: for=192.0.2.43, for="[2001:db8:cafe::17]"
   </code></pre>

   X-Forwarded-For와 X-Forwarded-By가 동지에 존재할 때는 특별한 주의가 필요하다.  
   그런 경우에는, 이미 존재하는 필드가 추가된 순서를 알 수 없기 때문에 변활할 수 없다.  
   또한, X-Forwarded-For 헤더 필드를 제거하면, 아직 새로운 헤더 필드를 지원하지 않는 곳에서는 문제가 발생할 수 있다.  



### 7.5 Example Usage
   A request from a client with IP address 192.0.2.43 passes through a proxy with IP address 198.51.100.17, then through another proxy with IP address 203.0.113.60 before reaching an origin server.  
   This could, for example, be an office client behind a corporate malware filter talking to a origin server through a reverse proxy.  
   
   * The HTTP request between the client and the first proxy has no "Forwarded" header field.  
   * The HTTP request between the first and second proxy has a "Forwarded: for=192.0.2.43" header field.  
   * The HTTP request between the second proxy and the origin server has a "Forwarded: for=192.0.2.43, for=198.51.100.17;by=203.0.113.60;proto=http;host=example.com" header field.  

   Note that, at some points in a connection chain, the information might not be updated in the "Forwarded" header field, either because of lack of support of this HTTP extension or because of a policy decision not to disclose information about this network component.  


### 7.5 사용 예
   IP 주소가 192.0.2.43 인 client의 요청은 IP 주소가 198.51.100.17 인 proxy를 통과한 다음 본래 서버에 도달하기 전에 IP 주소가 203.0.113.60 인 다른 proxy를 통과한다.  
   
   예를 들어, 이것은 reverse proxy를 통해서 원본 서버와 통신하는 기업의 malware filter 뒤에 있는 사용자 일 수 있다.  
   * client와 첫번째 proxy 사이에 있는 HTTP request에는 "Forwarded" 헤더 필드가 없다.  
   * 첫번째와 두번째 proxy 사이에 있는 HTTP request에는 "Forwarded: for=192.0.2.43" 헤더 필드가 있다.  
   * 두번째와 본래의 서버 사이에 있는 HTTP request에는 "Forwarded: for=192.0.2.43, for=198.51.100.17;by=203.0.113.60;proto=http;host=example.com" 헤더 필드가 있다.  

   연결 체인에 있는 어떤 부분에서 "Forwarded" 필드 안에 있는 정보가 HTTP extension의 미지원이나 네트워크 컴포넌트에 대한 정보를 공유하지 않는 정책에 의해 업데이트 되지 않을 수 있다.  




## 8. Security Considerations
### 8.1. Header Validity and Integrity
   The "Forwarded" HTTP header field cannot be relied upon to be correct, as it may be modified, whether mistakenly or for malicious reasons, by every node on the way to the server, including the client making the request.  

   One approach to ensure that the "Forwarded" HTTP header field is correct is to verify the correctness of proxies and to whitelist them as trusted.  
   This approach has at least two weaknesses.  
   First, the chain of IP addresses listed before the request came to the proxy cannot be trusted.  
   Second, unless the communication between proxies and the endpoint is secured, the data can be modified by an attacker with access to the network.  


## 8. 보안 고려사항
### 8.1. 헤더 유효성과 무결성
   "Forwarded" HTTP 헤더 필드는 client가 만든 requset까지나 실수든 악의적인 이유든 서버로 가는 경로에 있는 모든 노드들에 대해서 중간에 값이 수정되었는지 완전히 신뢰할 수 없다.  

   "Forwarded" HTTP 헤더 필드가 올바른지 보증하는 하나의 방법은 proxy의 정확도를 검증하고, 신뢰되는 proxy에 대한 whitelist를 작성하는 것이다.  
   이러한 접근은 적어도 두개의 약점을 가지고 있다.  
   첫번째로, proxy 요청이 오기전에 나열된 IP 주소의 체인을 신뢰할 수 없다.  
   두번째로, proxy들과 종단점간의 통신이 안전하지 않으면, 공격자가 네트워크에 접근하여 데이터를 수정할 수 있다.  




### 8.2. Information Leak
   The "Forwarded" HTTP header field can reveal internal structures of the network setup behind the NAT or proxy setup, which may be undesired.  
   This can be addressed either by using obfuscated elements, by preventing the internal nodes from updating the HTTP header field, or by having an egress proxy remove entries that reveal internal network information.  

   This header field should never be copied into response messages by origin servers or intermediaries, as it can reveal the whole proxy chain to the client.  
   As a side effect, special care must be taken in hosting environments not to allow the TRACE request where the "Forwarded" field is used, as it would appear in the body of the response message.   


### 8.2. 정보의 누출
   "Forwarded" HTTP 헤더 필드는 의도하지 않았지만, proxy 설정이나 NAT 뒤에 있는 네트워크 설정의 내부 구조를 드러낼수 있다.  
   이 문제는 난독화된 요소를 사용하거나, 내부 노드가 HTTP 헤더 필드를 업데이트하지 못하도록 하거나, proxy가 내부 네트워크 정보를 나타내려는 항목을 제거하도록 해서 해결할 수 있다.  

   이 헤더 필드는 client에 전체 proxy 체인에서 보여줄 수 있기 때문에 본래의 서버나 중개자에 의해서 response 메시지 안으로 복사되면 안된다.  
   부작용으로, "Forwarded" 필드가 사용되고, TRACE request를 허용하지 않는 호스팅 환경에서 response의 메시지 안에 나타날 수 있으므로, 주의해야한다.  




### 8.3. Privacy Considerations
   In recent years, there have been growing concerns about privacy.  
   There is a trade-off between ensuring privacy for users versus disclosing information that is useful, for example, for debugging, statistics, and generating location-dependent content.  
   The "Forwarded" HTTP header field, by design, exposes information that some users consider privacy sensitive, in order to allow for such uses.  
   For any proxy, if the HTTP request contains header fields that specifically request privacy semantics, the proxy SHOULD NOT use the "Forwarded" header field, nor in any other manner pass private information, such as IP addresses, on to the next hop.  

   The client's IP address, that may be forwarded in the "for" parameter of this header field, is considered to be privacy sensitive by many people, as the IP address may be able to uniquely identify a client, what operator the user is using, and possibly a rough estimation of where the user is geographically located.  

   Proxies using this extension will preserve the information of a direct connection.  
   This has an end-user privacy impact regardless of whether the end-user or deployer knows or expects that this is the case.  

   Implementers and deployers of such proxies need to consider whether, and how, deploying this extension affects user privacy.  

   The default configuration for both the "by" and "for" parameters SHOULD contain obfuscated identifiers.
   These identifiers SHOULD be randomly generated per request.  
   If identifiers that persist across requests are required, their lifetimes SHOULD be limited and they SHOULD NOT persist longer than client IP addresses.  
   When generating obfuscated identifiers, care must be taken not to include potentially sensitive information in them.  

   Note that users' IP addresses may already be forwarded by proxies using the header field X-Forwarded-For, which is widely used.  
   It should also be noted that if the user were doing the connection directly without passing the proxy, the client's IP address would be sent to the web server.  
   Users that do not actively choose an anonymizing proxy cannot rely on having their IP address shielded.
   These users who want to minimize the risk of being tracked must also note that there are other ways information may leak, for example, by browser header field fingerprinting.  
   The Forwarded header field itself, even when used without a uniquely identifying client identifier, may make fingerprinting more feasible by revealing the chain of proxies traversed by the client's request.  


### 8.3. 개인정보 고려사항
   최근 몇년간, 개인정보에 대한 걱정은 점점 커졌다.  
   이것은 사용자의 privacy를 보장하는 것과 유용한 정보(예를 들어, debugging, statistics, 위치 기반의 컨텐츠 생성)를 보여주는 것에 대한 trade-off다.  
   설계된 "Forwarded" HTTP 헤더 필드는 그러한 정보들을 사용하기 위해 일부 사용자의 개인정보 민감도를 고려하여 정보를 노출한다.  
   HTTP request의 헤더 필드에 특별히 개인정보에 관해서 요청하는 것이 포함되있다면, proxy는 "Forwarded" 헤더 필드를 사용하면 안되고, 어떠한 방법으로도 개인정보를 전송하면 안된다. (IP 주소와 같은 것들)  

   이 헤더필드의 "for" 파라미터 안에서 전달될 수 있는 client의 IP 주소는 많은 사람들이 개인정보의 민감성이 고려된 것으로 간주되고 있다.  
   IP 주소가 고유한 client를 식별할 수 있고, 사용자가 어떤 동작을 수행하고 있는지와 사용자가 지리적으로 어느 위치에 있는지 대략적으로 추정할 수 있다.  

   이 확장 기능을 사용하는 proxy들은 직접적으로 연결된 정보를 보존할 것이다.  
   이는 최종 사용자나 중간 배포자가 알고 있거나 기대하는지 여부와 관계없이, 최종 사용자의 개인정보 보호에 영향이 있다.  

   그런 proxy들의 개발자나 배포자는 이 확장을 배포하는 것이 사용자의 개인정보에 영향을 미치는지와 어떻게 미치는지 고려하는게 필요하다.  

   "by"와 "for" 파라미터 둘의 기본 설정값은 난독화된 식별자를 포함하는 것이다.  
   이러한 식별자는 request 마다 랜덤으로 생성된다.  
   요청간에 지속되는 식별자가 필요하다면, 수명이 제한되어야하며, client IP주소보다 오래 유지 되지 않아야 한다.  
   난독화된 식별자를 생성할때 잠재적으로 민감한 정보가 포함되지 않도록 신경써야 한다.  

   사용자의 IP 주소는 널리 사용되는 X-Forwarded-For 헤더 필드를 사용하여 proxy들에 의해 이미 전달되었을 수 있다.  
   또한 사용자가 proxy 연결을 사용하지 않고 직접 연결을 맺는다면, client IP 주소는 웹서버로 전달 될 수 있다.  
   익명 proxy를 적극적으로 선택하지 않은 사용자는 자신의 IP 주소를 보호할 수 없다.  
   추적의 위험을 최소화하기 원하는 사용자들은 브라우저 헤더 필드 fingerprinting과 같이 정보가 유출 될 수 있는 다른 방법이 있다는 것을 알아야 한다.  
   Forwarded 헤더 필드 자체는 client 식별자를 고유하게 식별하지 않고 사용되는 경우에도 client의 요청에 의해 proxy들의 체인을 들어냄으로써 쉽게 fingerprinting 을 만들 수 있다.  




## 9. IANA Considerations
   This document specifies the HTTP header field listed below, which has
   been added to the "Permanent Message Header Field Names" registry
   defined in [RFC3864](https://tools.ietf.org/html/rfc3864).  

   Header field: Forwarded  
   Applicable protocol: http  
   Status: standard  
   Author/Change controller:  
       IETF (iesg@ietf.org)  
       Internet Engineering Task Force  
   Specification document(s): this specification [Section 4](https://tools.ietf.org/html/rfc7239#section-4)  
   Related information: None  

   The "Forwarded" header field contains parameters for which IANA has
   created and now maintains a new registry entitled "HTTP Forwarded
   Parameters".  
   Initial registrations are given below.  
   For future assignments, the registration procedure is IETF Review [RFC5226](https://tools.ietf.org/html/rfc5226).  
   The security and privacy implications of all new parameters should be
   thoroughly documented.  
   New parameters and their values MUST conform with the forwarded-pair as defined in ABNF in [Section 4](#4. Forwarded HTTP Header Field).  
   Further, a short description should be provided in the registration.  
   <pre><code>
   +-------------+---------------------------------------+-------------+
   | Parameter   | Description                           | Reference   |
   | name        |                                       |             |
   +-------------+---------------------------------------+-------------+
   | by          | IP address of incoming interface of a | Section 5.1 |
   |             | proxy                                 |             |
   | for         | IP address of client making a request | Section 5.2 |
   |             | through a proxy                       |             |
   | host        | Host header field of the incoming     | Section 5.3 |
   |             | request                               |             |
   | proto       | Application protocol used for         | Section 5.4 |
   |             | incoming request                      |             |
   +-------------+---------------------------------------+-------------+
   </code></pre>


## 9. IANA 고려사항
   이 문서는 [RFC3864](https://tools.ietf.org/html/rfc3864)안에 정의된 "Permanent Message Header Field Names" 레지스트리에 더해진 HTTP 헤더 필드 리스트를 아래에 작성 해뒀다.  
   
   Header field: Forwarded  
   Applicable protocol: http  
   Status: standard  
   Author/Change controller:  
       IETF (iesg@ietf.org)  
       Internet Engineering Task Force  
   Specification document(s): this specification [Section 4](https://tools.ietf.org/html/rfc7239#section-4)  
   Related information: None  

   "Forwarded" 헤더 필드는 IANA가 생성하고 "HTTP Forwarded Parameters" 라고 지칭된 레지스트리에서 관리되고 있는 파라미터들을 포함한다.  
   초기 등록 값은 아래 주어져있다.  
   향후 할당을 위한 등록 절차는 IETF 리뷰 [RFC5226](https://tools.ietf.org/html/rfc5226)에 있다.  
   모든 새로운 파라미터의 보안 및 개인 정보 보호 관련 사항은 철저히 문서화되어야 한다.  
   새로운 파라미터와 그 값은 반드시 [Section 4](#4. Forwarded HTTP 헤더 필드)의 ABNF 안에 정의된 forwarded-pair를 따라야 한다.  
   그리고, 짧은 설명도 등록을 위해 제공되어야 한다.  
   <pre><code>
   +-------------+---------------------------------------+-------------+
   | Parameter   | Description                           | Reference   |
   | name        |                                       |             |
   +-------------+---------------------------------------+-------------+
   | by          | IP address of incoming interface of a | Section 5.1 |
   |             | proxy                                 |             |
   | for         | IP address of client making a request | Section 5.2 |
   |             | through a proxy                       |             |
   | host        | Host header field of the incoming     | Section 5.3 |
   |             | request                               |             |
   | proto       | Application protocol used for         | Section 5.4 |
   |             | incoming request                      |             |
   +-------------+---------------------------------------+-------------+
   </code></pre>


## 참고
[RFC 7239](https://tools.ietf.org/html/rfc7239)