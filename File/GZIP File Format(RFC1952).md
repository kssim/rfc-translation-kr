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


### 1.4. 호환
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



### 2.3. Member format
<pre><code>
Each member has the following structure:   
+---+---+---+---+---+---+---+---+---+---+
|ID1|ID2|CM |FLG|     MTIME     |XFL|OS | (more-->)
+---+---+---+---+---+---+---+---+---+---+   

(if FLG.FEXTRA set)   
+---+---+=================================+
| XLEN  |...XLEN bytes of "extra field"...| (more-->)
+---+---+=================================+

(if FLG.FNAME set)   
+=========================================+
|...original file name, zero-terminated...| (more-->)
+=========================================+

(if FLG.FCOMMENT set)   
+===================================+
|...file comment, zero-terminated...| (more-->)
+===================================+

(if FLG.FHCRC set)   
+---+---+
| CRC16 |
+---+---+

+=======================+
|...compressed blocks...| (more-->)
+=======================+   

  0   1   2   3   4   5   6   7
+---+---+---+---+---+---+---+---+
|     CRC32     |     ISIZE     |
+---+---+---+---+---+---+---+---+
</code></pre>


### 2.3. Member 형식
<pre><code>
모든 member는 아래 구조를 따른다:   
+---+---+---+---+---+---+---+---+---+---+
|ID1|ID2|CM |FLG|     MTIME     |XFL|OS | (more-->)
+---+---+---+---+---+---+---+---+---+---+   

(if FLG.FEXTRA set)   
+---+---+=================================+
| XLEN  |...XLEN bytes of "extra field"...| (more-->)
+---+---+=================================+

(if FLG.FNAME set)   
+=========================================+
|...original file name, zero-terminated...| (more-->)
+=========================================+

(if FLG.FCOMMENT set)   
+===================================+
|...file comment, zero-terminated...| (more-->)
+===================================+

(if FLG.FHCRC set)   
+---+---+
| CRC16 |
+---+---+

+=======================+
|...compressed blocks...| (more-->)
+=======================+   

  0   1   2   3   4   5   6   7
+---+---+---+---+---+---+---+---+
|     CRC32     |     ISIZE     |
+---+---+---+---+---+---+---+---+
</code></pre>



#### 2.3.1. Member header and trailer
<pre><code>
ID1 (IDentification 1)
ID2 (IDentification 2)
	These have the fixed values ID1 = 31 (0x1f, \037), ID2 = 139 (0x8b, \213), to identify the file as being in gzip format.

CM (Compression Method)
	This identifies the compression method used in the file.
	CM = 0-7 are reserved.
	CM = 8 denotes the "deflate" compression method, which is the one customarily used by gzip and which is documented elsewhere.

FLG (FLaGs)
	This flag byte is divided into individual bits as follows:

	* bit 0   FTEXT
	* bit 1   FHCRC
	* bit 2   FEXTRA
	* bit 3   FNAME
	* bit 4   FCOMMENT
	* bit 5   reserved
	* bit 6   reserved
	* bit 7   reserved

	If FTEXT is set, the file is probably ASCII text.
	This is an optional indication, which the compressor may set by checking a small amount of the input data to see whether any non-ASCII characters are present.
	In case of doubt, FTEXT is cleared, indicating binary data.
	For systems which have different file formats for ascii text and binary data, the decompressor can use FTEXT to choose the appropriate format.
	We deliberately do not specify the algorithm used to set this bit, since a compressor always has the option of leaving it cleared and a decompressor
	always has the option of ignoring it and letting some other program handle issues of data conversion.

	If FHCRC is set, a CRC16 for the gzip header is present, immediately before the compressed data.
	The CRC16 consists of the two least significant bytes of the CRC32 for all bytes of the gzip header up to and not including the CRC16.
	[The FHCRC bit was never set by versions of gzip up to 1.2.4, even though it was documented with a different meaning in gzip 1.2.4.]

	If FEXTRA is set, optional extra fields are present, as described in a following section.
	If FNAME is set, an original file name is present, terminated by a zero byte.
	The name must consist of ISO 8859-1 (LATIN-1) characters; on operating systems using EBCDIC or any other character set for file names, the name must be translated to the ISO LATIN-1 character set.
	This is the original name of the file being compressed, with any directory components removed, and, if the file being compressed is on a file system with case insensitive names, forced to lower case.
	There is no original file name if the data was compressed from a source other than a named file; for example, if the source was stdin on a Unix system, there is no file name.

	If FCOMMENT is set, a zero-terminated file comment is present.
	This comment is not interpreted; it is only intended for human consumption.
	The comment must consist of ISO 8859-1 (LATIN-1) characters.
	Line breaks should be denoted by a single line feed character (10 decimal).

	Reserved FLG bits must be zero.

MTIME (Modification TIME)
	This gives the most recent modification time of the original file being compressed.
	The time is in Unix format, i.e., seconds since 00:00:00 GMT, Jan.  1, 1970.
	(Note that this may cause problems for MS-DOS and other systems that use local rather than Universal time.)
	If the compressed data did not come from a file, MTIME is set to the time at which compression started.  MTIME = 0 means no time stamp is available.

XFL (eXtra FLags)
	These flags are available for use by specific compression methods.
	The "deflate" method (CM = 8) sets these flags as
	follows:
		XFL = 2 - compressor used maximum compression, slowest algorithm
		XFL = 4 - compressor used fastest algorithm

OS (Operating System)
	This identifies the type of file system on which compression took place.
	This may be useful in determining end-of-line convention for text files.
	The currently defined values are as follows:
	
		0 - FAT filesystem (MS-DOS, OS/2, NT/Win32)
		1 - Amiga
		2 - VMS (or OpenVMS)
		3 - Unix
		4 - VM/CMS
		5 - Atari TOS
		6 - HPFS filesystem (OS/2, NT)
		7 - Macintosh
		8 - Z-System
		9 - CP/M
		10 - TOPS-20
		11 - NTFS filesystem (NT)
		12 - QDOS
		13 - Acorn RISCOS
		255 - unknown

XLEN (eXtra LENgth)
	If FLG.FEXTRA is set, this gives the length of the optional extra field.
	See below for details.

CRC32 (CRC-32)
	This contains a Cyclic Redundancy Check value of the uncompressed data computed according to CRC-32 algorithm used in the ISO 3309 standard and in section 8.1.1.6.2 of ITU-T recommendation V.42.
	(See http://www.iso.ch for ordering ISO documents. See gopher://info.itu.ch for an online version of ITU-T V.42.)

ISIZE (Input SIZE)
	This contains the size of the original (uncompressed) input data modulo 2^32.
</code></pre>



#### 2.3.1. Member 헤더와 트레일러
<pre><code>
ID1 (IDentification 1)
ID2 (IDentification 2)
	이 값들은 ID1 = 31(0x1f, \037), ID2 = 139(0x8b, \213)으로 고정된 값을 가지고 있다.
	위의 두 값을 통해서 파일이 gzip 형식이라는 것을 알 수 있다.

CM (Compression Method)
	이 식별자는 이 파일에서 압축으로 사용한 방법을 기술한다.
	CM = 0-7 은 예약된 값이다.
	CM = 8 은 "deflate" 압축 방법을 가르키고, gzip에서 일반적으로 사용된다.

FLG (FLaGs)
	이 플래그 바이트는 아래와 같은 각각의 비트로 구성되어 있다.

	* bit 0   FTEXT
	* bit 1   FHCRC
	* bit 2   FEXTRA
	* bit 3   FNAME
	* bit 4   FCOMMENT
	* bit 5   reserved
	* bit 6   reserved
	* bit 7   reserved

	FTEXT 가 설정되어 있다면, 파일은 아마 ASCII 문자열일 것이다.
	이 값은 선택적인 값이며, 압축시 입력된 값을 작은 단위 로 검사하며, 비 ASCII 문자가 나타나는지 확인 할 수 있다.
	FTEXT 가 없는 경우는 의심할 여지 없이, 바이너리 데이터를 가르킨다.
	ascii 문자열과 바이너리 데이터등 서로 다른 파일 형식을 사용하는 시스템에서 적절한 형식 선택하기 위해 압축을 해제할 때, FTEXT를 사용할 수 있다.
	여기서는 이 비트를 설정하기 위한 명확한 알고리즘을 명세하지 않는다. 압축을 할때 항상 이 값을 비울수 있는 옵션을 가지고 있고,
	압축을 해제할 때는 이 값을 무시하거나 데이터 변환의 문제를 다른 프로그램에서 해결하도록 내버려두는 옵션이 있기 때문에 명세하지 않는다.

	FHCRC 가 설정되어 있다면, 압축된 데이터의 바로 앞에 gzip 헤더를 위한 CRC16이 존재한다. 
	CRC16은 CRC16을 포함하지 않고 gzip 헤더까지의 모든 바이트를 위한 CRC32의 두 개의 최하위 비트로 구성된다.
	[FHCRC 비트는 다른 의미로 문서에 적혀있긴 했지만, gzip 1.2.4 버전까지는 설정되지 않았다.]

	FEXTRA 가 설정되어 있다면, 추가로 선택적인 필드가 존재하고, 아래 내용이 기술되어있다.

	FNAME 이 설정되어 있다면, 원본 파일 이름이 존재하며, 0바이트로 끝난다.
	이름은 반드시 ISO 8859-1 (LATIN-1) characters로 구성된다.
	EDCDIC을 사용하는 운영체제나 파일 이름을 위한 다른 character set들에서 이름은 반드시 ISO LATIN-1 character set으로 변환되어야 한다.
	이 값은 압축된 파일의 원본 이름이며, 특정 디렉토리 구성요소는 제거되고, 대소문자를 구분하지 않는 파일 시스템에서 압축된다면, 소문자는 무시된다.
	
	데이터가 이름이 있는 파일이 아닌 다른 곳에서 압축되었다면, 원본 파일 이름은 없다.
	예를 들어 유닉스 시스템에서 stdin으로 부터 압축되었다면, 파일 이름은 존재하지 않는다.

	FCOMMENT 가 설정되어 있다면, 0으로 끝나는 파일 주석이 존재한다.
	이 주석은 해석되지 않고, 오직 사람을 위해서만 존재합니다.
	주석은 ISO 8859-1 (LATIN-1) 반드시 문자열로 구성되어야 한다.
	개행은 단일 줄바꿈 문자로 표시 해야한다. (10진수 10개)
	
	예약된 FLG 비트는 반드시 0으로 설정되어야 한다.

MTIME (Modification TIME)
	압축이 시작된 원본 파일의 가장 최근의 수정된 시간을 알려준다.
	이 시간은 유닉스 표준이다. seconds since 00:00:00 GMT, Jan.  1, 1970.
	(이 값은 MS-DOS나 일반적인 시간이 아닌 지역 시간을 사용하는 다른 시스템에서 문제가 될 수 있다.)
	압축된 데이터가 파일에서 만들어진게 아니라면, MTIME은 압축이 시작된 시간으로 설정된다.
	MTIME = 0 이용가능한 타임스탬프가 없다는 것이다.

XFL (eXtra FLags)
	이 플래그는 특정 압축 방식에서만 사용이 가능하다.
	"deflate" 방식(CM = 8)은 아래와 같은 값들을 설정한다.
		XFL = 2 - 압축률이 최대가 되도록 한다. 가장 느린 알고리즘이다.
		XFL = 4 - 압축 속도가 가장 빠른 알고리즘을 사용한다.

OS (Operating System)
	압축이 발생한 파일 시스템의 유형을 식별하는 확장자이다.
	텍스트 파일의 변환을 위한 파일의 끝을 식별하는데 유용하다.
	현재 정의된 값은 아래와 같다.
	
		0 - FAT filesystem (MS-DOS, OS/2, NT/Win32)
		1 - Amiga
		2 - VMS (or OpenVMS)
		3 - Unix
		4 - VM/CMS
		5 - Atari TOS
		6 - HPFS filesystem (OS/2, NT)
		7 - Macintosh
		8 - Z-System
		9 - CP/M
		10 - TOPS-20
		11 - NTFS filesystem (NT)
		12 - QDOS
		13 - Acorn RISCOS
		255 - unknown

XLEN (eXtra LENgth)
	FLG.FEXTRA 가 설정되었다면, 선택적인 추가 필드의 길이가 주어진다.
	자세한 사항은 아래에 있다.

CRC32 (CRC-32)
	이 값은 ISO 3309 표준과 ITU-T 권고안 V.42 섹션안에서 사용되는 CRC-32 알고리즘에 따라 압축되어 계산되지 않은 데이터의 CRC 값을 포함하고 있다.
	(See http://www.iso.ch for ordering ISO documents. See gopher://info.itu.ch for an online version of ITU-T V.42.)

ISIZE (Input SIZE)
	이 값은 압축되지 않은 입력 값 modulo 2^32의 원본 크기를 가지고 있다.
</code></pre>


#### 2.3.1.1. Extra field
If the FLG.FEXTRA bit is set, an "extra field" is present in the header, with total length XLEN bytes.
It consists of a series of subfields, each of the form:  
<pre><code>   
+---+---+---+---+==================================+
|SI1|SI2|  LEN  |... LEN bytes of subfield data ...|
+---+---+---+---+==================================+
</code></pre>  

SI1 and SI2 provide a subfield ID, typically two ASCII letters with some mnemonic value.
Jean-Loup Gailly <gzip@prep.ai.mit.edu> is maintaining a registry of subfield IDs; please send him any subfield ID you wish to use.  
Subfield IDs with SI2 = 0 are reserved for future use.  
The following IDs are currently defined:  
<pre><code>
SI1         SI2         Data
----------|------------|----
0x41 ('A')  0x70 ('P')  Apollo file type information
</code></pre>  

LEN gives the length of the subfield data, excluding the 4 initial bytes.



#### 2.3.1.1. 여분의 필드
FLG.FEXTRA 비트가 설정되어 있다면, "extra field" 값은 XLEN 바이트의 총 길이와 함께 헤더 안에 있다.  
각각의 형식은 서브 필드 값들로 구성되어 있다.  
<pre><code>   
+---+---+---+---+==================================+
|SI1|SI2|  LEN  |... LEN bytes of subfield data ...|
+---+---+---+---+==================================+
</code></pre>  

SI1과 SI2 는 서브 필드의 ID를 제공하며, 일반적으로 두개의 ASCII 문자와 일부 mnemonic 값을 제공한다.   
Jean-Loup Gailly <gzip@prep.ai.mit.edu>는 서브 필드의 ID들의 등록을 관리하고 있다.  
추가로 사용을 원하는 서브 필드가 있으면 메일을 보내면 된다.  
SI2 = 0인 서브 필드 ID는 나중 사용을 위해 예약되어 있다.  
아래의 ID들은 현재 정의되어 있다.  
<pre><code>
SI1         SI2         Data
----------|------------|----
0x41 ('A')  0x70 ('P')  Apollo file type information
</code></pre>  

LEN은 초기 4개의 바이트를 제외하고, 서브필드 데이터의 길이를 제공한다.


#### 2.3.1.2. Compliance
A compliant compressor must produce files with correct ID1, ID2, CM, CRC32, and ISIZE, but may set all the other fields in the fixed-length part of the header to default values (255 for OS, 0 for all others).  
The compressor must set all reserved bits to zero.  

A compliant decompressor must check ID1, ID2, and CM, and provide an error indication if any of these have incorrect values.  
It must examine FEXTRA/XLEN, FNAME, FCOMMENT and FHCRC at least so it can skip over the optional fields if they are present.  
It need not examine any other part of the header or trailer; in particular, a decompressor may ignore FTEXT and OS and always produce binary output, and still be compliant.  
A compliant decompressor must give an error indication if any reserved bit is non-zero, since such a bit could indicate the presence of a new field that would cause subsequent data to be interpreted incorrectly.


#### 2.3.1.2. 호환
호환되는 압축 시스템은 정확한 ID1, ID2, CM, CRC32와 ISIZE 파일을 반드시 생성해야한다.  
헤더의 고정된 길이 부분에 있는 다른 필드 값들은 기본 값으로 설정해도 된다. (OS의 경우 255, 다른 것들은 0)  
압축 시스템은 예약된 모든 비트를 0으로 설정한다.  

호환되는 압축 해제 시스템은 ID1, ID2와 CM 그리고 제공되는 잘못된 값이 있을때 발생하는 오류 값 등을 반드시 확인해야한다.  
적어도 FEXTRA/XLEN, FNAME, FCOMMENT 과 FHCRC 반드시 검사 해야하고 선택 필드가 있는 경우에는 생략할 수 있다.  
헤더나 트레일러의 다른 부분을 검사하는 것은 꼭 필요하지 않다.
특히, 압축 해제시 FTEXT 나 OS 를 무시하고도 항상 바이너리를 출력하고, 호환될 수 있다.
호환되는 압축 해제 시스템은 반드시 정확한 오류 표시를 줘야한다.
만약 어떤 예약된 비트가 0이 아니면, 후속되는 데이터의 해석을 방해할 수 있는 새로운 데이터 필드의 존재를 나타낼수도 있다.


## 참고
[RFC 1952](https://tools.ietf.org/html/rfc1952)