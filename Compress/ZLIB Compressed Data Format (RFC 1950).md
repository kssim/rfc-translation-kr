# 1. Introduction
## 1.1. Purpose
The purpose of this specification is to define a lossless compressed data format that:  

* Is independent of CPU type, operating system, file system, and character set, and hence can be used for interchange;
* Can be produced or consumed, even for an arbitrarily long sequentially presented input data stream, using only an a priori bounded amount of intermediate storage, and hence can be used in data communications or similar structures such as Unix filters;
* Can use a number of different compression methods;
* Can be implemented readily in a manner not covered by patents, and hence can be practiced freely.

The data format defined by this specification does not attempt to allow random access to compressed data.



# 1. 서론
## 1.1. 목적
이 명세의 목적은 손실이 없는 압축된 데이터의 형식을 정의하는 것이다.  

* CPU 종류, 운영체제, 파일 시스템 그리고 character set에 무관하며, 상호 교환될 수 있다.   
* 중간 저장장치의 사전에 정의된 양만을 사용하여, 다른 데이터 스트림을 생성하기 위해 데이터 스트림(무작위로 접근 가능한 파일과 대조적으로)을 압축하거나 압축을 해제 할 수 있다. 따라서, 데이터 통신이나 유닉스 필터와 유사한 구조에서도 사용될 수 있다.
* 다양한 압축 방법을 사용할 수 있다.
* 특허에 저촉되지 않는 방법으로 즉시 구현이 가능하고, 자유롭게 사용이 가능하다.

명세에 의해 정의된 데이터 형식은 압축된 데이터의 임의 접근을 허락하지 않는다.  



## 1.2. Intended audience
This specification is intended for use by implementors of software to compress data into zlib format and/or decompress data from zlib format.  
The text of the specification assumes a basic background in programming at the level of bits and other primitive data representations.


## 1.2. 대상 독자
이 명세는 gzip 형식의 데이터를 압축 해제하거나 gzip 형태로 데이터를 압축하는 소프트웨어를 구현한 사람들을 대상으로 작성되었다.  
이 명세는 bit나 다른 원시적인 데이터 표현 수준에서 프로그래밍하는 배경지식을 가지고 작성되었다.



## 1.3. Scope
The specification specifies a compressed data format that can be used for in-memory compression of a sequence of arbitrary bytes.



## 1.3. 범위
이 명세는 메모리에서 무작위 바이트 순서의 압축을 사용할 수 있도록 압축된 데이터 형식을 명세한다.


## 1.4. Compliance
Unless otherwise indicated below, a compliant decompressor must be able to accept and decompress any data set that conforms to all the specifications presented here;  
a compliant compressor must produce data sets that conform to all the specifications presented here.


## 1.4. 호환
아래에서 특별히 명시하지 않는 한, 호환 압축 해제는 여기에 제시된 모든 사양을 준수하는 파일을 허용하고 압축을 풀 수 있어야 한다.  
압축은 여기에 준수된 모든 사양을 준수하는 파일을 생성해야 한다.  


## 1.5.  Definitions of terms and conventions used
byte: 8 bits stored or transmitted as a unit (same as an octet).  
(For this specification, a byte is exactly 8 bits, even on machines which store a character on a number of bits different from 8.)  
See below, for the numbering of bits within a byte.  


## 1.5. 용어 정의와 관례 정의
byte: 8비트가 저장되거나 전송되는 단위 (octet과 동일)  
(이 명세를 위해, byte는 정확히 8비트이고, 문자 8비트가 아닌 다른 비트 수를 저장하는 장비에서도 정확히 8비트를 가르킨다고 정의한다.)  
byte 안에 bite의 numbering을 위해 아래를 참고한다.  


## 1.6. Changes from previous versions
Version 3.1 was the first public release of this specification.  
In version 3.2, some terminology was changed and the Adler-32 sample code was rewritten for clarity.  
In version 3.3, the support for a preset dictionary was introduced, and the specification was converted to RFC style.  


## 1.6. 이전 버전과 차이점
버전 3.1은 이 명세가 공식적으로 처음 릴리즈된 버전이다.
버전 3.2에서는 일부 기술들이 변경되었고, Adler-32 샘플 코드가 명확하게 재작성되었다.
버전 3.3에서는 preset dictionary를 지원하기 시작했고, RFC 스타일에 맞게 변경되었다.



# 2. Detailed specification
## 2.1. Overall conventions
In the diagrams below, a box like this:
<pre><code>
+---+
|   | <-- the vertical bars might be missing
+---+   
represents one byte; a box like this:   
+==============+
|              |
+==============+   
represents a variable number of bytes.
</code></pre>

Bytes stored within a computer do not have a "bit order", since they are always treated as a unit.   
However, a byte considered as an integer between 0 and 255 does have a most- and least- significant bit, and since we write numbers with the most- significant digit on the left, we also write bytes with the most- significant bit on the left.   
In the diagrams below, we number the bits of a byte so that bit 0 is the least-significant bit, i.e., the bits are numbered:
<pre><code>
+--------+
|76543210|
+--------+
</code></pre>

This document does not address the issue of the order in which bits of a byte are transmitted on a bit-sequential medium, since the data format described here is byte- rather than bit-oriented.   
Within a computer, a number may occupy multiple bytes.   
All multi-byte numbers in the format described here are stored with the least-significant byte first (at the lower memory address).   
For example, the decimal number 520 is stored as:
<pre><code>
    0        1
+--------+--------+
|00001000|00000010|
+--------+--------+
^        ^
|        |
|        + more significant byte = 2 x 256
+ less significant byte = 8
</code></pre>



# 2. 세부 명세
## 2.1. 전반적인 규칙
아래 박스처럼 생긴 다이어그램이 있다.
In the diagrams below, a box like this:
<pre><code>
+---+
|   | <-- 수직 막대가 없을 수도 있다.
+---+   
이것은 1바이트를 표현한 것이다.   
+==============+
|              |
+==============+   
여러 갯수의 바이트를 표현한 것이다.
</code></pre>

바이트는 항상 유닛으로 다뤄지기 때문에 컴퓨터 안에서 바이트는 "bit order"를 가지지 않은채 저장된다.   
그러나, 바이트는 0과 255 사이의 integer 값으로 최상위와 최하위 비트를 가지고 있다.   
그리고, 왼쪽에 최상위 비트를 작성한다.   
아래에 있는 다이어그램 안에서 바이트의 비트에 번호를 매겼고, 비트 0이 최하위 비트이다.   
(즉 비트에 숫자가 부여된다.)   
<pre><code>
+--------+
|76543210|
+--------+
</code></pre>

이 문서는 byte 의 전송 과정에서 bit와 관련된 문제를 다루지 않는다.   
여기에 정리된 데이터 형식은 bit 중심이 아닌 byte 중심이다.   
컴퓨터 안에서 숫자는 여러 바이트를 차지할 수 있다.   
여기에 정리된 형식에 있는 모든 멀티 바이트 수는 최하위 비트를 가장 처음에 저장하도록 되어있다.(하위 메모리주소에서)   
예를 들어, 10진수 520은 아래와 같이 저장된다.   
<pre><code>
    0        1
+--------+--------+
|00001000|00000010|
+--------+--------+
^        ^
|        |
|        + more significant byte = 2 x 256
+ less significant byte = 8
</code></pre>


## 2.2. Data format
A zlib stream has the following structure:  
<pre><code>
  0   1
+---+---+
|CMF|FLG|   (more-->)
+---+---+  
(if FLG.FDICT set)  
  0   1   2   3
+---+---+---+---+
|     DICTID    |   (more-->)
+---+---+---+---+  
+=====================+---+---+---+---+
|...compressed data...|    ADLER32    |
+=====================+---+---+---+---+
</code></pre>
Any data which may appear after ADLER32 are not part of the zlib stream.    


<pre><code>
CMF (Compression Method and flags)
	This byte is divided into a 4-bit compression method and a 4-bit information field depending on the compression method.
		bits 0 to 3  CM     Compression method
		bits 4 to 7  CINFO  Compression info

CM (Compression method)
	This identifies the compression method used in the file.
	CM = 8 denotes the "deflate" compression method with a window size up to 32K.
	This is the method used by gzip and PNG.
	CM = 15 is reserved.
	It might be used in a future version of this specification to indicate the presence of an extra field before the compressed data.

CINFO (Compression info)
	For CM = 8, CINFO is the base-2 logarithm of the LZ77 window size, minus eight (CINFO=7 indicates a 32K window size).
	Values of CINFO above 7 are not allowed in this version of the specification.
	CINFO is not defined in this specification for CM not equal to 8.

FLG (FLaGs)
	This flag byte is divided as follows:
		bits 0 to 4  FCHECK  (check bits for CMF and FLG)
		bit  5       FDICT   (preset dictionary)
		bits 6 to 7  FLEVEL  (compression level)  
	The FCHECK value must be such that CMF and FLG, when viewed as a 16-bit unsigned integer stored in MSB order (CMF*256 + FLG), is a multiple of 31.
         
FDICT (Preset dictionary)
	If FDICT is set, a DICT dictionary identifier is present immediately after the FLG byte.
	The dictionary is a sequence of bytes which are initially fed to the compressor without producing any compressed output.
	DICT is the Adler-32 checksum of this sequence of bytes (see the definition of ADLER32 below).
	The decompressor can use this identifier to determine which dictionary has been used by the compressor.

FLEVEL (Compression level)
	These flags are available for use by specific compression methods.
	The "deflate" method (CM = 8) sets these flags as
	follows:
		0 - compressor used fastest algorithm
		1 - compressor used fast algorithm
		2 - compressor used default algorithm
		3 - compressor used maximum compression, slowest algorithm  
	The information in FLEVEL is not needed for decompression;
	it is there to indicate if recompression might be worthwhile.

compressed data
	For compression method 8, the compressed data is stored in the deflate compressed data format as described in the document "DEFLATE Compressed Data Format Specification" by L. Peter Deutsch.
	Other compressed data formats are not specified in this version of the zlib specification.

ADLER32 (Adler-32 checksum)
	This contains a checksum value of the uncompressed data (excluding any dictionary data) computed according to Adler-32 algorithm.
	This algorithm is a 32-bit extension and improvement of the Fletcher algorithm, used in the ITU-T X.224 / ISO 8073 standard.
	Adler-32 is composed of two sums accumulated per byte: s1 is the sum of all bytes, s2 is the sum of all s1 values.
	Both sums are done modulo 65521. s1 is initialized to 1, s2 to zero.
	The Adler-32 checksum is stored as s2*65536 + s1 in most- significant-byte first (network) order.
</code></pre>



## 2.2. 데이터 형식
zilb 스트림은 아래 구조를 따른다.
<pre><code>
  0   1
+---+---+
|CMF|FLG|   (more-->)
+---+---+  
(if FLG.FDICT set)  
  0   1   2   3
+---+---+---+---+
|     DICTID    |   (more-->)
+---+---+---+---+  
+=====================+---+---+---+---+
|...compressed data...|    ADLER32    |
+=====================+---+---+---+---+
</code></pre>
ADLER32 뒤에 나타나는 모든 데이터는 zlib stream의 영역이 아니다.


<pre><code>
CMF (Compression Method and flags)
	이 바이트는 4비트의 압축된 방식과 4비트의 압축 방식에 대한 정보 필드로 분리된다.  
		bits 0 to 3  CM     Compression method
		bits 4 to 7  CINFO  Compression info

CM (Compression method)
	이것은 파일안에서 사용된 압축 방법을 보여준다.
	CM = 8은 윈도우 사이즈가 32K까지인 "deflate" 압축 방식을 나타낸다.
	이것은 gzip과 PNG에 의해 사용되는 방식이다.
	CM = 15는 예약되어있다.
	이 필드는 아마 압축된 데이터의 앞에 있는 여분의 필드의 존재를 나타내는 이 명세의 미래 버전에서 사용될 것이다.

CINFO (Compression info)
	CM = 8인 경우, CINFO는 LZ77 윈도우 크기의 기본 로그 2의 8을 뺀 값이다. (CINFO=7 은 32K 윈도우 사이즈를 가르킨다)
	7 이상의 CINFO 값은 이 명세의 버전에서 허용되지 않았다.
	CINFO는 이 명세에서 8과 같지 않은 CM에 대해서는 정의되어 있지 않다.

FLG (FLaGs)
	이 플래그 바이트는 아래와 같이 나눠져있다.
		bits 0 to 4  FCHECK  (check bits for CMF and FLG)
		bit  5       FDICT   (preset dictionary)
		bits 6 to 7  FLEVEL  (compression level)  
	FCHECK의 값은 MSB 순서(CMF*256 + FLG)로 저장된 16비트 부호없는 정수로 볼때, CMF 및 FLG가 31의 배수가 되도록 해야한다.
         
FDICT (Preset dictionary)
	FDICT 가 설정되어있으면, DICT dictionary 식별자는 FLG 바이트 바로 뒤에 존재한다.
	dictionary는 모든 압축된 결과물의 생성되기 전 압축기의 초기값으로 주어지는 바이트의 순서이다.
	DICT 는 바이트 순서의 Adler-32 체크섬이다. (ADLER32 의 정의에 대해서는 아래 적혀있다.)
	압축 해제기는 이 값을 압축기에 의해 dictionary 가 사용되었는지 결정하는 식별자로 사용할 수 있다.

FLEVEL (Compression level)
	이 값은 특정 압축 방법을 사용하기 위해 이용되어진다.
	"deflate" 방식(CM = 8)은 이 값을 아래와 같이 설정한다.
		0 - compressor used fastest algorithm
		1 - compressor used fast algorithm
		2 - compressor used default algorithm
		3 - compressor used maximum compression, slowest algorithm  
	FLEVEL 안에 있는 정보는 압축 해제시에는 필요 없다.
	그 값은 재 압축시 효율적인가를 알려주기 위해 사용된다.

compressed data
	압축 방식 8의 경우, 압축된 데이터는 L. Peter Deutsch의 "DEFLATE Compressed Data Format Specification" 문서에서 설명된대대로 deflate로 압축된 데이터 형식으로 저장된다.
	다른 압축된 데이터 형식은 이 zlib 명세의 버전 안에서 명세되어 있지않다.

ADLER32 (Adler-32 checksum)
	Adler-32 알고리즘에 따라 계산된 압축되지 않은 데이터(모든 dictionary 데이터를 제외하고)의 체크섬 값을 가지고 있다.
	이 알고리즘은 32비트 확장자이고 ITU-T X.224/ISO 8073 표준에서 사용된 Fletcher 알고리즘이 향상된 것이다.
	Adler-32는 바이트당 누적된 두 합으로 구성된다. s1은 모든 바이트의 합이고, s2는 모든 s1의 값의 합이다.
	두 합은 module 65521 이다. s1은 1로 초기화되고, s2는 0으로 초기화 된다.
	Adler-32 체크섬은 최상위 바이트의 첫번째 순서로 s2의 655536 곱하고 s1을 더한 값이 저장된다.
</code></pre>



## 2.3. Compliance
A compliant compressor must produce streams with correct CMF, FLG and ADLER32, but need not support preset dictionaries.  
When the zlib data format is used as part of another standard data format, the compressor may use only preset dictionaries that are specified by this other data format.  
If this other format does not use the preset dictionary feature, the compressor must not set the FDICT flag.  

A compliant decompressor must check CMF, FLG, and ADLER32, and provide an error indication if any of these have incorrect values.  
A compliant decompressor must give an error indication if CM is not one of the values defined in this specification (only the value 8 is permitted in this version), since another value could indicate the presence of new features that would cause subsequent data to be interpreted incorrectly.  
A compliant decompressor must give an error indication if FDICT is set and DICTID is not the identifier of a known preset dictionary.  
A decompressor may ignore FLEVEL and still be compliant.  
When the zlib data format is being used as a part of another standard format, a compliant decompressor must support all the preset dictionaries specified by the other format.  
When the other format does not use the preset dictionary feature, a compliant decompressor must reject any stream in which the FDICT flag is set.  



## 2.3. 호환
호환 압축기는 올바른 CMF, FLG와 ADLER32 스트림을 생산해야하고, preset dictionaries 는 지원하지 않아도 된다.  
zlib 데이터 형식이 다른 표준 데이터 형식의 일부로 사용될 때, 압축기는 아마 이것의 다른 데이터 형식에 의해 규정된 preset dictionaries 만 사용할 것이다.  
만약 이 다른 형태의 형식이 preset dictionary의 특징을 사용하지 않는다면, 압축기는 FDICT 플래그를 설정하면 안된다. 

호환 압축해제기는 반드시 CMF, FLG와 ADLER32를 체크해야하고, 부정확한 값을 가지고 있다면, 에러를 표시를 제공해야한다.  
호환 압축해제기는 에러 지시자를 주어야하며, 만약 CM이 이 명세에 정의된 값(이 버전에서는 오직 8만 허용되어 있다.)중의 하나가 아니라면, 다른 값은 부정확하게 해석된 데이터의 뒷 부분에서 발생한 새로운 기능의 출현을 가르킬수 있다.  
호환 압축해제기는 만약 FDICT가 설정되어 있고, DICTID 가 preset dictionary의 알고있는 식별자 아니면,에러를 표시해야한다.  
압축해제기는 FLEVEL을 무시하고도, 호환성을 유지할 수 있다.  
zlib 데이터 형식이 다른 표준 데이터 형식의 일부로 사용될 때, 호환 압축해제기는 반드시 다른 포멧에 의해 명세된 모든 preset dictionary들을 제공해야한다.  
다른 포멧이 preset dictionary의 기능을 사용하지 않을 때,호환 압축해제기는 FDICT가 설정된 모든 스트림을 거부해야한다.  


# 참고
[RFC 1950](https://tools.ietf.org/html/rfc1950)