## 1. Introduction
### 1.1 Purpose
The purpose of this specification is to define a lossless compressed data format that:   

* Is independent of CPU type, operating system, file system, and character set, and hence can be used for interchange;   
* Can compress or decompress a data stream (as opposed to a randomly accessible file) to produce another data stream, using only an a priori bounded amount of intermediate storage, and hence can be used in data communications or similar structures such as Unix filters;   
* Compresses data with efficiency comparable to the best currently available general-purpose compression methods, and in particular considerably better than the "compress" program;   
* Can be implemented readily in a manner not covered by patents, and hence can be practiced freely;   
* Is compatible with the file format produced by the current widely used gzip utility, in that conforming decompressors will be able to read data produced by the existing gzip compressor.   

The data format defined by this specification does not attempt to:   

* Provide random access to compressed data;   
* Compress specialized data (e.g., raster graphics) as well as the best currently available specialized algorithms.   


## 1. 서론
### 1.1 목적
이 명세의 목적은 손실이 없는 압축된 데이터의 형식을 정의하는 것이다.   

* CPU 종류, 운영체제, 파일 시스템 그리고 character set에 무관하며, 상호 교환될 수 있다.   
* 중간 저장장치의 사전에 정의된 양만을 사용하여, 다른 데이터 스트림을 생성하기 위해 데이터 스트림(무작위로 접근 가능한 파일과 대조적으로)을 압축하거나 압축을 해제 할 수 있다. 따라서, 데이터 통신이나 유닉스 필터와 유사한 구조에서도 사용될 수 있다.
* 효율적으로 압축된 데이터는 현재 이용 가능한 범용적인 압축 방법과 비교할만 하고, 특히 "압축" 프로그램 중에서는 상당히 빠른 편이다.   
* 특허에 저촉되지 않는 방법으로 즉시 구현이 가능하고, 자유롭게 사용이 가능하다.
* 현재 널리 사용중인 gzip 유틸리티에 의해 생성된 파일 형식과 같이 사용할 수 있고, 기존의 gzip 압축에 의해 생성된 데이터를 압축 해제하여 읽을 수 있다.

이 명세에 의해 정의된 데이터 형식은 아래 항목을 허용하지 않는다.   

* 압축된 데이터의 무작위 접근
* 특수 압축된 파일(raster graphics 같은)과 현재 이용가능한 최상의 특수 알고리즘


### 1.2. Intended audience
This specification is intended for use by implementors of software to compress data into gzip format and/or decompress data from gzip format.   

The text of the specification assumes a basic background in programming at the level of bits and other primitive data representations.   


### 1.2. 대상 독자
이 명세는 gzip 형식의 데이터를 압축 해제하거나 gzip 형태로 데이터를 압축하는 소프트웨어를 구현한 사람들을 대상으로 작성되었다.
이 명세는 bit나 다른 원시적인 데이터 표현 수준에서 프로그래밍하는 배경지식을 가지고 작성되었다.


### 1.3. Scope
The specification specifies a compression method and a file format (the latter assuming only that a file can store a sequence of arbitrary bytes).   
It does not specify any particular interface to a file system or anything about character sets or encodings (except for file names and comments, which are optional).


### 1.3. 범위
이 명세는 압축 방법과 파일 형식(임의의 바이트 형식을 저장할 수 있다는 전제가 깔려있다.)을 명세한다.   
특별한 파일 시스템의 인터페이스나 character set 이나 encoding 과 관련된 다른 것들은 명세하지 않는다.   
(파일 이름이나 주석 같은 경우를 제외하고)


### 1.4. Compliance
Unless otherwise indicated below, a compliant decompressor must be able to accept and decompress any file that conforms to all the specifications presented here; a compliant compressor must produce files that conform to all the specifications presented here.   
The material in the appendices is not part of the specification per se and is not relevant to compliance.   


### 1.4. 허용
아래에서 특별히 명시하지 않는 한, 호환 압축 해제는 여기에 제시된 모든 사양을 준수하는 파일을 허용하고 압축을 풀 수 있어야 한다.
압축은 여기에 준수된 모든 사양을 준수하는 파일을 생성해야 한다.
부록에 있는 부분은 규격이 아니고, 규정과는 상관이 없다.   


### 1.5. Definitions of terms and conventions used
byte: 8 bits stored or transmitted as a unit (same as an octet).   
(For this specification, a byte is exactly 8 bits, even on machines which store a character on a number of bits different from 8.)   
See below for the numbering of bits within a byte.   


### 1.5. 용어 정의와 관례 정의
byte: 8비트가 저장되거나 전송되는 단위 (octet과 동일)
(이 명세를 위해, byte는 정확히 8비트이고, 문자 8비트가 아닌 다른 비트 수를 저장하는 장비에서도 정확히 8비트를 가르킨다고 정의한다.)   
byte 안에 bite의 numbering을 위해 아래를 참고한다.



### 1.6. Changes from previous versions
There have been no technical changes to the gzip format since version 4.1 of this specification.   
In version 4.2, some terminology was changed, and the sample CRC code was rewritten for clarity and to eliminate the requirement for the caller to do pre- and post-conditioning.   
Version 4.3 is a conversion of the specification to RFC style. 



### 1.6. 이전 버전과의 변화
이 명세에서 gzip형식 4.1버전부터 어떠한 기술적인 변화도 없다.   
버전 4.2에서 몇몇 기술은 변화되었고, 샘플 CRC 코드를 보기 좋게 재작성했다.   
그리고 호출자가 사전, 사후에 조절해야하는 요구사항을 삭제했다.   
버전 4.3에서 이 문서가 RFC 형식에 맞게 변환되었다.   


## 2. Detailed specification
### 2.1. Overall conventions
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



## 2. 세부 명세
### 2.1. 전반적인 규칙
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




### 2.2. File format
A gzip file consists of a series of "members" (compressed data sets).   
The format of each member is specified in the following section.   
The members simply appear one after another in the file, with no additional information before, between, or after them.   



### 2.2. 파일 형식
gzip 파일은 "members"(압축된 데이터의 집합)의 시리즈로 구성된다.   
각각의 member의 형식은 아래 섹션에 명세되어있다.   
member들은 파일에서 하나씩 차례대로 나타나며, 이전, 중간 또는 이후의 추가 정보는 없다.



## 참고
[RFC 1952](https://tools.ietf.org/html/rfc1952)