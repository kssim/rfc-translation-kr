# 1. Introduction
## 1.1. Purpose
The purpose of this specification is to define a lossless compressed data format that:

* Is independent of CPU type, operating system, file system, and character set, and hence can be used for interchange;
* Can be produced or consumed, even for an arbitrarily long sequentially presented input data stream, using only an a priori bounded amount of intermediate storage, and hence can be used in data communications or similar structures such as Unix filters;
* Compresses data with efficiency comparable to the best currently available general-purpose compression methods, and in particular considerably better than the "compress" program;
* Can be implemented readily in a manner not covered by patents, and hence can be practiced freely;
* Is compatible with the file format produced by the current widely used gzip utility, in that conforming decompressors will be able to read data produced by the existing gzip compressor.

The data format defined by this specification does not attempt to:

* Allow random access to compressed data;
* Compress specialized data (e.g., raster graphics) as well as the best currently available specialized algorithms.

A simple counting argument shows that no lossless compression algorithm can compress every possible input data set.  
For the format defined here, the worst case expansion is 5 bytes per 32K byte block, i.e., a size increase of 0.015% for large data sets.  
English text usually compresses by a factor of 2.5 to 3;  
executable files usually compress somewhat less; graphical data such as raster images may compress much more.


# 1. 서론
## 1.1. 목적
이 명세의 목적은 손실이 없는 압축된 데이터의 형식을 정의하는 것이다.  

* CPU 종류, 운영체제, 파일 시스템 그리고 character set에 무관하며, 상호 교환될 수 있다.   
* 중간 저장장치의 사전에 정의된 양만을 사용하여, 임의로 길고 순차적으로 제시되는 입력 스트림에 대해서도 생산되거나 소비될 수 있고, 그렇기 때문에 데이터 통신이나 유닉스 필터와 유사한 구조에서 사용될 수 있다.
* 효율적으로 압축된 데이터는 현재 이용 가능한 범용적인 압축 방법과 비교할만 하고, 특히 "압축" 프로그램 중에서는 상당히 빠른 편이다.
* 특허에 저촉되지 않는 방법으로 즉시 구현이 가능하고, 자유롭게 사용이 가능하다.
* 현재 널리 사용중인 gzip 유틸리티에 의해 생성된 파일 형식과 같이 사용할 수 있고, 기존의 gzip 압축에 의해 생성된 데이터를 압축 해제하여 읽을 수 있다.

이 명세에 의해 정의된 데이터 형식은 아래 항목을 허용하지 않는다.   

* 압축된 데이터의 무작위 접근
* 특수 압축된 파일(raster graphics 같은)과 현재 이용가능한 최상의 특수 알고리즘

간단한 카운팅 인자는 무손실 압축 알고리즘이 모든 가능한 입력 데이터 세트를 압축할 수 있음을 보여준다.  
이곳에 정의된 형식의 경우에는, 최악의 경우 확장은 32K 바이트 블록당 5바이트씩 즉, 큰 데이터 세트의 경우 0.015%씩 증가한다.  
영문 텍스트는 보통 2.5에서 3의 비율로 압축된다.  
실행 파일은 보통 조금만 압축되며, raster 이미지 같은 그래픽 데이터는 더 많이 압축된다.


## 1.2. Intended audience
This specification is intended for use by implementors of software to compress data into "deflate" format and/or decompress data from "deflate" format.  
The text of the specification assumes a basic background in programming at the level of bits and other primitive data representations.  
Familiarity with the technique of Huffman coding is helpful but not required.


## 1.2. 대상 독자
이 명세는 deflate 형식의 데이터를 압축 해제하거나 deflate 형태로 데이터를 압축하는 소프트웨어를 구현한 사람들을 대상으로 작성되었다.  
이 명세는 bit나 다른 원시적인 데이터 표현 수준에서 프로그래밍하는 배경지식이 있다는 것을 가정하고 작성되었다.  
Huffman 코딩 의 기법에 친숙하면, 도움이 될 것이다.


## 1.3. Scope
The specification specifies a method for representing a sequence of bytes as a (usually shorter) sequence of bits, and a method for packing the latter bit sequence into bytes.


## 1.3. 범위
이 명세는 바이트 시퀀스를 비트 시퀀스로 표현하는 방법과 비트 시퀀스를 바이트 시퀀스로 패킹하는 방법을 명세한다.
이 명세는 비트의 순차로써 바이트의 순서를 표현하는 방법과 바이트 안의 패킹하는 방법을 명세한다.


## 1.4. Compliance
Unless otherwise indicated below, a compliant decompressor must be able to accept and decompress any data set that conforms to all the specifications presented here;  
a compliant compressor must produce data sets that conform to all the specifications presented here.


## 1.4. 호환
아래에서 특별히 명시하지 않는 한, 호환 압축 해제는 여기에 제시된 모든 사양을 준수하는 파일을 허용하고 압축을 풀 수 있어야 한다.
압축은 여기에 준수된 모든 사양을 준수하는 파일을 생성해야 한다.


## 1.5. Definitions of terms and conventions used
Byte: 8 bits stored or transmitted as a unit (same as an octet).  
For this specification, a byte is exactly 8 bits, even on machines which store a character on a number of bits different from eight.  
See below, for the numbering of bits within a byte.  
String: a sequence of arbitrary bytes.  


## 1.5. 용어 정의와 관례 정의
Byte: 8비트가 저장되거나 전송되는 단위 (octet과 동일)  
(이 명세를 위해, byte는 정확히 8비트이고, 문자 8비트가 아닌 다른 비트 수를 저장하는 장비에서도 정확히 8비트를 가르킨다고 정의한다.)   
byte 안에 bite의 numbering을 위해 아래를 참고한다.  
String: 임의 바이트들의 순서.  


## 1.6. Changes from previous versions
There have been no technical changes to the deflate format since version 1.1 of this specification.  
In version 1.2, some terminology was changed.  
Version 1.3 is a conversion of the specification to RFC style. 


## 1.6. 이전 버전과의 차이점
이 명세의 1.1버전부터 deflate 형식에 대한 기술적인 변화는 없다.
1.2버전에서 일부 기술적인 변화들이 발생했다.
1.3버전은 RFC 형식으로 문서를 변환했다.


# 2. Compressed representation overview
A compressed data set consists of a series of blocks, corresponding to successive blocks of input data.  The block sizes are arbitrary, except that non-compressible blocks are limited to 65,535 bytes.  

Each block is compressed using a combination of the LZ77 algorithm and Huffman coding
The Huffman trees for each block are independent of those for previous or subsequent blocks;  
the LZ77 algorithm may use a reference to a duplicated string occurring in a previous block, up to 32K input bytes before.

Each block consists of two parts: a pair of Huffman code trees that describe the representation of the compressed data part, and a compressed data part.  
(The Huffman trees themselves are compressed using Huffman encoding.)  
The compressed data consists of a series of elements of two types:  
literal bytes (of strings that have not been detected as duplicated within the previous 32K input bytes), and pointers to duplicated strings, where a pointer is represented as a pair <length, backward distance>.  
The representation used in the "deflate" format limits distances to 32K bytes and lengths to 258 bytes, but does not limit the size of a block, except for uncompressible blocks, which are limited as noted above.

Each type of value (literals, distances, and lengths) in the compressed data is represented using a Huffman code, using one code tree for literals and lengths and a separate code tree for distances.  
The code trees for each block appear in a compact form just before the compressed data for that block.


# 2. 압축 표현 개요
압축된 데이터 세트는 입력된 데이터의 연속 블록에 해당하는 블록의 시리즈로 구성된다.  
블록의 크기는 무작위이며, 예외적으로 압축이 불가능한 블록은 65,535바이트로 제한된다.

각각의 블록은 LZ77 알고리즘과 Huffman 코딩의 조합을 이용하여 압축된다.  
각 블록을 위한 Huffman 트리는 이전이나 후속 블록에 독립적이다.  
LZ77 알고리즘은 이전 블록안에서 중복되는 문자열에 대해 참조를 사용하고, 최대 32K까지의 데이터를 확인한다.

각각의 블록은 압축된 데이터의 부분을 표현하는 Huffman 코드 트리의 쌍과 압축된 데이터의 부분으로 구성된다.  
(Huffman 트리는 그 자체로 Huffman 인코딩을 사용하여 압축된다.)  
압축된 데이터는 아래 두가지 유형의 일련의 요소로 구성된다.  
리터럴 바이트(이전 32K의 입력 데이터안에서 중복된 것으로 감지되지 않은 문자열)과 중복된 문자열의 포인터(<length, backward distance>의 쌍으로 표현되는 포인터)로 구성된다.

deflate 형식에서 사용 된 표현은 거리를 32K 바이트와 길이를 258 바이트로 제한하지만, 위에서 언급 한대로 제한되는 압축 할 수 없는 블록을 제외하고는 블록의 크기를 제한하지 않는다.

압축된 데이터 안에 있는 각 타입의 값(리터럴, 거리 그리고 길이)들은 리터럴을 위한 하나의 코드 트리와 길이 그리고 거리를 위한 분리된 코드 트리를 사용하여, Huffman code 를 사용하여 표현된다.  
각 블록의 코드 트리는 해당 블록의 압축된 데이터의 바로 앞에서 압축된 형태로 나타난다.


# 3. Detailed Specification
## 3.1 Overall conventions
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



# 3. 세부적인 명세
## 3.1 전반적인 관례
아래 박스처럼 생긴 다이어그램이 있다.
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










# 참고
[RFC 1951](https://tools.ietf.org/html/rfc1951)