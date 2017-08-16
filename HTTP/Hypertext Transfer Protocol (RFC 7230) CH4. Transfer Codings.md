# Transfer Codings
Transfer coding names are used to indicate an encoding transformation that has been, can be, or might need to be applied to a payload body in order to ensure "safe transport" through the network.  
This differs from a content coding in that the transfer coding is a property of the message rather than a property of the representation that is being transferred.

```
transfer-coding    = "chunked" ; Section 4.1
	/ "compress" ; Section 4.2.1
	/ "deflate" ; Section 4.2.2
	/ "gzip" ; Section 4.2.3
	/ transfer-extension
transfer-extension = token *( OWS ";" OWS transfer-parameter )
```

Parameters are in the form of a name or name=value pair.

```transfer-parameter = token BWS "=" BWS ( token / quoted-string )```

All transfer-coding names are case-insensitive and ought to be registered within the HTTP Transfer Coding registry, as defined in Section 8.4.  
They are used in the TE (Section 4.3) and Transfer-Encoding (Section 3.3.1) header fields.


# Transfer Codings
Transfer coding 이름은 네트워크를 통한 "안전한 전송"을 보장하기 위해 페이로드 바디에 적용되었거나, 적용될 수 있거나, 적용되어야하는 encoding transformation을 가리키는데 사용된다.  
이것은 transfer coding이 전송된 표현의 속성이 아니라 메시지의 속성이라는 점에서 content coding 과는 다르다.

```
transfer-coding    = "chunked" ; Section 4.1
						/ "compress" ; Section 4.2.1
						/ "deflate" ; Section 4.2.2
						/ "gzip" ; Section 4.2.3
						/ transfer-extension
transfer-extension = token *( OWS ";" OWS transfer-parameter )
```

파라미터들은 이름이나 이름=값의 쌍의 형식이다.

```transfer-parameter = token BWS "=" BWS ( token / quoted-string )```

모든 transfer-coding 이름은 대소문자 구별을 하지 않고, 섹션 8.4에 정의된 대로 HTTP Transfer Coding registry 안에 등록되어야 한다.  
TE(섹션 4.3)과 Transfer-Encoding(섹션 3.3.1) 헤더 필드에서 사용된다.


## 4.1.  Chunked Transfer Coding
The chunked transfer coding wraps the payload body in order to transfer it as a series of chunks, each with its own size indicator, followed by an OPTIONAL trailer containing header fields.  
Chunked enables content streams of unknown size to be transferred as a sequence of length-delimited buffers, which enables the sender to retain connection persistence and the recipient to know when it has received the entire message.

```
chunked-body   = *chunk
					last-chunk
					trailer-part
					CRLF

chunk          = chunk-size [ chunk-ext ] CRLF
					chunk-data CRLF
chunk-size     = 1*HEXDIG
last-chunk     = 1*("0") [ chunk-ext ] CRLF

chunk-data     = 1*OCTET ; a sequence of chunk-size octets
```

The chunk-size field is a string of hex digits indicating the size of the chunk-data in octets.  
The chunked transfer coding is complete when a chunk with a chunk-size of zero is received, possibly followed by a trailer, and finally terminated by an empty line.

A recipient MUST be able to parse and decode the chunked transfer coding.



## 4.1. Chunked Transfer Coding
chunked transfer coding은 페이로드 바디를 일련의 chunk들로 전송하기 위해 감싼다.  
각각의 chunk들은 고유의 크기 표시가 있고, 헤더 필드가 포함된 OPTIONAL 트레일러가 온다.  
Chunked는 알 수 없는 크기의 컨텐츠 스트림을 길이가 구분 된 버퍼 시퀀스로 전송할 수 있으므로, 보낸 사람이 연결 지속하거나 수신자가 전체 메시지가 언제 수신되었는지 알수 있게 해준다.

```
chunked-body   = *chunk
					last-chunk
					trailer-part
					CRLF

chunk          = chunk-size [ chunk-ext ] CRLF
					chunk-data CRLF
chunk-size     = 1*HEXDIG
last-chunk     = 1*("0") [ chunk-ext ] CRLF

chunk-data     = 1*OCTET ; a sequence of chunk-size octets
```

chunk-size 필드는 chunk 데이터의 크기를 나타내는 16진수의 문자열이다.  
chunked transfer coding은 chunk-size 가 0으로 수신 되었을 때 끝나고, 트레일러 뒤에 오게 되면, 빈 라인으로 종료된다.

수신자는 chunked transfer coding을 반드시 분석하고 해석할 수 있어야 한다.



### 4.1.1. Chunk Extensions
The chunked encoding allows each chunk to include zero or more chunk extensions, immediately following the chunk-size, for the sake of supplying per-chunk metadata (such as a signature or hash), mid-message control information, or randomization of message body size.

```
chunk-ext      = *( ";" chunk-ext-name [ "=" chunk-ext-val ] )

chunk-ext-name = token
chunk-ext-val  = token / quoted-string
```

The chunked encoding is specific to each connection and is likely to be removed or recoded by each recipient (including intermediaries) before any higher-level application would have a chance to inspect the extensions.  
Hence, use of chunk extensions is generally limited to specialized HTTP services such as "long polling" (where client and server can have shared expectations regarding the use of chunk extensions) or for padding within an end-to-end secured connection.

A recipient MUST ignore unrecognized chunk extensions.  
A server ought to limit the total length of chunk extensions received in a request to an amount reasonable for the services provided, in the same way that it applies length limitations and timeouts for other parts of a message, and generate an appropriate 4xx (Client Error) response if that amount is exceeded.



### 4.1.1. Chunk 확장
chunked 인코딩은 각 chunk가 chunk당 메타 데이터(서명이나 hash 같은), 중간 메시지 제어 정보나 랜덤화된 메시지 바디 크기 바로 다음에 0개 이상의 chunk 확장자를 포함 할 수 있도록 한다.

```
chunk-ext      = *( ";" chunk-ext-name [ "=" chunk-ext-val ] )

chunk-ext-name = token
chunk-ext-val  = token / quoted-string
```

chunked encoding은 각 연결에 고유하며, 모든 상위 응용 프로그램이 확장(중재자를 포함)을 검사할 수 있기 전에 각 수신자에 의해 제거되거나 다시 코딩 될 수 있다.  
그러므로, chunk 확장의 사용은 일반적으로 end-to-end 보안 연결 안에서의 패딩이나 "긴 폴링"(클라이언트와 서버가 chunk 확장의 사용을 고려하여 기대하고 있을때)과 같은 특수 HTTP 서비스로 제한된다.

수신자는 등록되지 않은 chunk 확장을 반드시 무시해야한다.  
서버는 서비스 제공을 위한 합리적인 양으로 request에서 수신된 chunk 확장의 총 길이를 제한해야한다.  
같은 방식으로 길이의 제한을 제공해야하고 메시지의 다른 부분을 위한 타임아웃도 제공해야한다.  
그리고 정해진 양을 초과하면, 적절한 4xx(클라이언트 에러) 응답을 생성해야한다.



### 4.1.2. Chunked Trailer Part
A trailer allows the sender to include additional fields at the end of a chunked message in order to supply metadata that might be dynamically generated while the message body is sent, such as a message integrity check, digital signature, or post-processing status.  
The trailer fields are identical to header fields, except they are sent in a chunked trailer instead of the message's header section.

```trailer-part   = *( header-field CRLF )```

A sender MUST NOT generate a trailer that contains a field necessary for message framing (e.g., Transfer-Encoding and Content-Length), routing (e.g., Host), request modifiers (e.g., controls and conditionals in Section 5 of [RFC7231]), authentication (e.g., see [RFC7235] and [RFC6265]), response control data (e.g., see Section 7.1 of [RFC7231]), or determining how to process the payload (e.g., Content-Encoding, Content-Type, Content-Range, and Trailer).

When a chunked message containing a non-empty trailer is received, the recipient MAY process the fields (aside from those forbidden above) as if they were appended to the message's header section.  
A recipient MUST ignore (or consider as an error) any fields that are forbidden to be sent in a trailer, since processing them as if they were present in the header section might bypass external security filters.

Unless the request includes a TE header field indicating "trailers" is acceptable, as described in Section 4.3, a server SHOULD NOT generate trailer fields that it believes are necessary for the user agent to receive.  
Without a TE containing "trailers", the server ought to assume that the trailer fields might be silently discarded along the path to the user agent.  
This requirement allows intermediaries to forward a de-chunked message to an HTTP/1.0 recipient without buffering the entire response.


### 4.1.2. Chunked Trailer 부분
트레일러는 송신자에게 메시지 바디가 보내지는 동안 동적으로 생성된 메타 데이터(message 무결성 검사, 디지털 서명나 후처리 상태 같은)를 제공하기 위해 chunked 메시지의 끝부분에 추가적인 필드를 포함시킬수 있도록 한다.  
트레일러 필드들은 메시지 헤더 섹션 대신 chunked 트레일러를 전송한다는 점을 제외하고는 헤더 필드들과 동일하다.

```trailer-part   = *( header-field CRLF )```

송신자는 메시지 프레이밍(Transfer-Encoding과 Content-Length)을 위해 필요한 필드, 라우팅(Host), request modifiers(RFC 7231의 섹션 7.1의 제어와 조건), 인증(RFC 7235, RFC 6265), response control data(RFC 7231의 섹션 7.1)이나 페이로드를 어떻게 처리할지 나타내는 결정자(Content-Encoding, Content-Type, Content-Range와 Trailer)등을 포함한 트레일러를 생성하면 안된다.

비어있지 않은 trailer를 포함한 chunk 메시지를 수신했을 때, 수신자는 헤더 섹션의 메시지가 추가된 것 처럼 처리할 수 있다. (위에서 금지된 것들을 제외하고)  
수신자는 헤더 섹션에 있는 것처럼 처리하여 외부 보안 필더를 위회 할 수 있기 때문에 트레일러에서 전송할 수 없는 필드는 무시해야한다. (또는 오류로 처리해야한다.)

섹션 4.3에 설명 된 것처럼 "trailers"가 수용 가능한 TE 헤더 필드를 요청에 포함하지 않는 한, 서버는 user agent가 수신해야하는 것으로 생각되는 트레일러 필드를 생성해서는 안된다.  
"trailers"가 포함된 TE가 없으면, 서버는 trailer 필드가 자동으로 user agent 경로를 따라 폐기된다고 가정해야한다.  
이 요구사항은 중개자가 전체 응답을 버퍼링하지 않고, HTTP/1.0 de-chunked 메시지를 수신자에게 전달하도록 한다.


### 4.1.3. Decoding Chunked
A process for decoding the chunked transfer coding can be represented in pseudo-code as:

```
length := 0
read chunk-size, chunk-ext (if any), and CRLF
while (chunk-size > 0) {
	read chunk-data and CRLF
	append chunk-data to decoded-body
	length := length + chunk-size
	read chunk-size, chunk-ext (if any), and CRLF
}

read trailer field
while (trailer field is not empty) {
	if (trailer field is allowed to be sent in a trailer) {
		append trailer field to existing header fields
	}
	read trailer-field
}
Content-Length := length
Remove "chunked" from Transfer-Encoding
Remove Trailer from existing header fields
```


### 4.1.3. Decoding Chunked
chunked transfer coding을 디코딩하는 것은 아래 의사코드로 표현될 수 있다.

```
length := 0
read chunk-size, chunk-ext (if any), and CRLF
while (chunk-size > 0) {
	read chunk-data and CRLF
	append chunk-data to decoded-body
	length := length + chunk-size
	read chunk-size, chunk-ext (if any), and CRLF
}

read trailer field
while (trailer field is not empty) {
	if (trailer field is allowed to be sent in a trailer) {
		append trailer field to existing header fields
	}
	read trailer-field
}
Content-Length := length
Remove "chunked" from Transfer-Encoding
Remove Trailer from existing header fields
```


## 4.2. Compression Codings
The codings defined below can be used to compress the payload of a message.



## 4.2. Compression Codings
아래 정의된 코딩은 메시지의 페이로드를 압축하는데 사용할 수 있다.


### 4.2.1. Compress Coding
The "compress" coding is an adaptive Lempel-Ziv-Welch (LZW) coding [Welch] that is commonly produced by the UNIX file compression program "compress".
A recipient SHOULD consider "x-compress" to be equivalent to "compress".


### 4.2.1. Compress Coding
"압축" 코딩은 유닉스 파일 압축 프로그램인 "compress"에 의해 일반적으로 생성되는 Lempel-Ziv-Welch(LZW) 코딩이다.
수신자는 "x-compress"가 "compress"와 동등한 것으로 간주해야한다.


### 4.2.2. Deflate Coding
The "deflate" coding is a "zlib" data format [RFC1950] containing a "deflate" compressed data stream [RFC1951] that uses a combination of the Lempel-Ziv (LZ77) compression algorithm and Huffman coding.

Note: Some non-conformant implementations send the "deflate" compressed data without the zlib wrapper.


### 4.2.2. Deflate Coding
"deflate" 코딩은 Lempel-Ziv(LZ77) 압축 알고리즘과 Huffman 코딩을 혼합하여 사용한 RFC 1951의 "deflate"로 압축된 데이터 스트림을 포함하고 있는 RFC 1950의 "zlib" 데이터 형식이다.

참고 : 일부 비규격 구현에서는 zlib 래퍼 없이 "deflate" 압축 데이터를 보낸다.


### 4.2.3. Gzip Coding
The "gzip" coding is an LZ77 coding with a 32-bit Cyclic Redundancy Check (CRC) that is commonly produced by the gzip file compression program [RFC1952].  
A recipient SHOULD consider "x-gzip" to be equivalent to "gzip".


### 4.2.3. Gzip Coding
"gzip" 코딩은 RFC 1952의 gzip 파일 압축 프로그램에서 흔히 제공되는 32-bit Cyclic Redundancy Check(CRC)를 사용한 LZ77 코딩이다.  
수신자는 "x-gzip"을 "gzip"과 같다고 여겨야한다.


## 4.3.  TE
The "TE" header field in a request indicates what transfer codings, besides chunked, the client is willing to accept in response, and whether or not the client is willing to accept trailer fields in a chunked transfer coding.

The TE field-value consists of a comma-separated list of transfer coding names, each allowing for optional parameters (as described in Section 4), and/or the keyword "trailers".  
A client MUST NOT send the chunked transfer coding name in TE; chunked is always acceptable for HTTP/1.1 recipients.

```
TE        = #t-codings
t-codings = "trailers" / ( transfer-coding [ t-ranking ] )
t-ranking = OWS ";" OWS "q=" rank
rank      = ( "0" [ "." 0*3DIGIT ] )
			/ ( "1" [ "." 0*3("0") ] )
```

Three examples of TE use are below.

```
TE: deflate
TE:
TE: trailers, deflate;q=0.5
```

The presence of the keyword "trailers" indicates that the client is willing to accept trailer fields in a chunked transfer coding, as defined in Section 4.1.2, on behalf of itself and any downstream clients.  
For requests from an intermediary, this implies that either: (a) all downstream clients are willing to accept trailer fields in the forwarded response; or, (b) the intermediary will attempt to buffer the response on behalf of downstream recipients.  
Note that HTTP/1.1 does not define any means to limit the size of a chunked response such that an intermediary can be assured of buffering the entire response.

When multiple transfer codings are acceptable, the client MAY rankthe codings by preference using a case-insensitive "q" parameter (similar to the qvalues used in content negotiation fields, Section 5.3.1 of [RFC7231]).  
The rank value is a real number in the range 0 through 1, where 0.001 is the least preferred and 1 is the most preferred; a value of 0 means "not acceptable".

If the TE field-value is empty or if no TE field is present, the only acceptable transfer coding is chunked.  
A message with no transfer coding is always acceptable.

Since the TE header field only applies to the immediate connection, a sender of TE MUST also send a "TE" connection option within the Connection header field (Section 6.1) in order to prevent the TE field from being forwarded by intermediaries that do not support its semantics.


## 4.3.  TE
The "TE" header field in a request indicates what transfer codings, besides chunked, the client is willing to accept in response, and whether or not the client is willing to accept trailer fields in a chunked transfer coding.

The TE field-value consists of a comma-separated list of transfer coding names, each allowing for optional parameters (as described in Section 4), and/or the keyword "trailers".  
A client MUST NOT send the chunked transfer coding name in TE; chunked is always acceptable for HTTP/1.1 recipients.

```
TE        = #t-codings
t-codings = "trailers" / ( transfer-coding [ t-ranking ] )
t-ranking = OWS ";" OWS "q=" rank
rank      = ( "0" [ "." 0*3DIGIT ] )
			/ ( "1" [ "." 0*3("0") ] )
```

Three examples of TE use are below.

```
TE: deflate
TE:
TE: trailers, deflate;q=0.5
```

The presence of the keyword "trailers" indicates that the client is willing to accept trailer fields in a chunked transfer coding, as defined in Section 4.1.2, on behalf of itself and any downstream clients.  
For requests from an intermediary, this implies that either: (a) all downstream clients are willing to accept trailer fields in the forwarded response; or, (b) the intermediary will attempt to buffer the response on behalf of downstream recipients.  
Note that HTTP/1.1 does not define any means to limit the size of a chunked response such that an intermediary can be assured of buffering the entire response.

When multiple transfer codings are acceptable, the client MAY rankthe codings by preference using a case-insensitive "q" parameter (similar to the qvalues used in content negotiation fields, Section 5.3.1 of [RFC7231]).  
The rank value is a real number in the range 0 through 1, where 0.001 is the least preferred and 1 is the most preferred; a value of 0 means "not acceptable".

If the TE field-value is empty or if no TE field is present, the only acceptable transfer coding is chunked.  
A message with no transfer coding is always acceptable.

Since the TE header field only applies to the immediate connection, a sender of TE MUST also send a "TE" connection option within the Connection header field (Section 6.1) in order to prevent the TE field from being forwarded by intermediaries that do not support its semantics.


# 참조
[RFC 7230 Section 4](https://tools.ietf.org/html/rfc7230#section-4)