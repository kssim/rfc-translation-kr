# 1. Abstract
Currently, HTML forms allow the producer of the form to request information from the user reading the form.  
These forms have proven useful in a wide variety of applications in which input from the user is necessary.  
However, this capability is limited because HTML forms don't provide a way to ask the user to submit files of data.  
Service providers who need to get files from the user have had to implement custom user applications.  
(Examples of these custom browsers have appeared on the www-talk mailing list.)  
Since file-upload is a feature that will benefit many applications, this proposes an extension to HTML to allow information providers to express file upload requests uniformly, and a MIME compatible representation for file upload responses.  
This also includes a description of a backward compatibility strategy that allows new servers to interact with the current HTML user agents.  

The proposal is independent of which version of HTML it becomes a part.  


# 1. Abstract
현재, HTML 폼은 폼을 제공한 사람이 폼을 사용하고 있는 사용자로부터 정보를 요청할 수 있도록 허락한다.  
이러한 폼들은 사용자로부터 필요한 정보를 입력받는 것이 필요하다고 여겨지는 수많은 어플리케이션들에서 유용하게 사용된다는 것이 입증되었다.  
그러나, HTML 폼은 사용자가 데이터 파일을 제출하는 방식을 제공하지 않으므로 기능이 제한적이다.  
사용자로부터 파일을 받는 것이 필요한 서비스 제공자들은 별도의 사용자 어플리케이션을 구현했다.  
(이러한 별도의 브라우저들의 예들은 www-talk mailing 리스트에 나타났다.)  

파일 업로드는 많은 어플리케이션에서 유용할 특징이기 때문에, 규격화된 파일 업로드 요청을 정보 제공자들에게 보여줄수 있도록 HTML을 확장하고, 파일 업로드 응답을 MIME 호환으로 표시할 것을 제안한다.  
또한 여기에는 새로운 서버와 현재 HTML user agent들과 상호 작용할 수 있도록하는 이전 버전과의 호환성 전략도 작성되어 있다.  

이 제안은 HTML 버전과는 독립적이고, HTML의 일부가 될 것이다.


# 2. HTML forms with file submission
The current HTML specification defines eight possible values for the attribute TYPE of an INPUT element: CHECKBOX, HIDDEN, IMAGE, PASSWORD, RADIO, RESET, SUBMIT, TEXT.  

In addition, it defines the default ENCTYPE attribute of the FORM element using the POST METHOD to have the default value "application/x-www-form-urlencoded".  
This proposal makes two changes to HTML:

1. Add a FILE option for the TYPE attribute of INPUT.
2. Allow an ACCEPT attribute for INPUT tag, which is a list of media types or type patterns allowed for the input.

In addition, it defines a new MIME media type, multipart/form-data, and specifies the behavior of HTML user agents when interpreting a form with ENCTYPE="multipart/form-data" and/or ```<INPUT type="file">``` tags.

These changes might be considered independently, but are all necessary for reasonable file upload.  
The author of an HTML form who wants to request one or more files from a user would write (for example):  

```html
<FORM ENCTYPE="multipart/form-data" ACTION="_URL_" METHOD=POST>

File to process: <INPUT NAME="userfile1" TYPE="file">

<INPUT TYPE="submit" VALUE="Send File">

</FORM>
```


The change to the HTML DTD is to add one item to the entity "InputType".  
In addition, it is proposed that the INPUT tag have an ACCEPT attribute, which is a list of comma-separated media types.

```html
... (other elements) ...
  
<!ENTITY % InputType "(TEXT | PASSWORD | CHECKBOX |
                       RADIO | SUBMIT | RESET |
                       IMAGE | HIDDEN | FILE )">
<!ELEMENT INPUT - 0 EMPTY>
<!ATTLIST INPUT
        TYPE %InputType TEXT
        NAME CDATA #IMPLIED  -- required for all but submit and reset
        VALUE CDATA #IMPLIED
        SRC %URI #IMPLIED  -- for image inputs --
        CHECKED (CHECKED) #IMPLIED
        SIZE CDATA #IMPLIED  --like NUMBERS,
                                but delimited with comma, not space
        MAXLENGTH NUMBER #IMPLIED
        ALIGN (top|middle|bottom) #IMPLIED
        ACCEPT CDATA #IMPLIED --list of content types
        >
        ... (other elements) ...
```


# 2. 파일 승인과 HTML 폼
현재 HTML 규격은 INPUT 요소의 타입 속성을 위해 아래 8개의 값을 정의하고 있다.  
CHECKBOX, HIDDEN, IMAGE, PASSWORD, RADIO, RESET, SUBMIT, TEXT.  

추가적으로, 기본값으로 "application/x-www-form-urlencoded"를 가진 POST 메소드를 사용하는 폼 요소의 기본 ENCTYPE 속성을 정의한다.  
이 제안은 HTML의 두가지 변화를 만든다.  

1. INPUT 의 타입 속성을 위한 파일 옵션을 추가한다.
2. 입력이 허용된 미디어 타입들의 리스트나 타입 패턴들의 목록인 INPUT 태그를 위한 ACCEPT 속성을 허락한다.

추가적으로, 새로운 MIME 미디어 타입을 정의하고 폼이 ENCTYPE="multipart/form-data" 나 ```<INPUT type="file">``` 태그와 같이 사용될 때의 HTML 사용자의 행동을 명세한다.

이러한 변화는 각각 독립적으로 고려될 수 있지만, 합리적인 파일 업로드를 위해서는 모두 필요하다.  
사용자로부터 하나 이상의 파일들을 받길 원하는 HTML 폼의 작성자는 아래와 같이 작성해야한다.

```html
<FORM ENCTYPE="multipart/form-data" ACTION="_URL_" METHOD=POST>

File to process: <INPUT NAME="userfile1" TYPE="file">

<INPUT TYPE="submit" VALUE="Send File">

</FORM>
```


HTML DTD가 변경되면 "InputType" 개체에 하나의 항목이 추가된다.  
또한, INPUT 태그에 쉼표로 구분된 미디어 타입들의 리스트인 ACCEPT 속성이 포함되도록 제안한다.

```html
... (other elements) ...
  
<!ENTITY % InputType "(TEXT | PASSWORD | CHECKBOX |
                       RADIO | SUBMIT | RESET |
                       IMAGE | HIDDEN | FILE )">
<!ELEMENT INPUT - 0 EMPTY>
<!ATTLIST INPUT
        TYPE %InputType TEXT
        NAME CDATA #IMPLIED  -- required for all but submit and reset
        VALUE CDATA #IMPLIED
        SRC %URI #IMPLIED  -- for image inputs --
        CHECKED (CHECKED) #IMPLIED
        SIZE CDATA #IMPLIED  --like NUMBERS,
                                but delimited with comma, not space
        MAXLENGTH NUMBER #IMPLIED
        ALIGN (top|middle|bottom) #IMPLIED
        ACCEPT CDATA #IMPLIED --list of content types
        >
        ... (other elements) ...
```



# 3. Suggested implementation
While user agents that interpret HTML have wide leeway to choose the most appropriate mechanism for their context, this section suggests how one class of user agent, WWW browsers, might implement file upload.


# 3. 구현 제안
HTML을 해석하는 user agent는 가장 적절한 알고리즘을 선택하는 많은 선택지가 있지만, 이 섹션에서는 하나의 user agent(WWW browsers)가 파일 업로드를 구현하는 방법에 대해서 제안한다.



## 3.1 Display of FILE widget
When a INPUT tag of type FILE is encountered, the browser might show a display of (previously selected) file names, and a "Browse" button or selection method.  
Selecting the "Browse" button would cause the browser to enter into a file selection mode appropriate for the platform.  
Window-based browsers might pop up a file selection window, for example.  
In such a file selection dialog, the user would have the option of replacing a current selection, adding a new file selection, etc.  
Browser implementors might choose let the list of file names be manually edited.



## 3.1 파일 위젯 표시
If an ACCEPT attribute is present, the browser might constrain the file patterns prompted for to match those with the corresponding appropriate file extensions for the platform.
브라우저가 파일 타입의 INPUT 태그를 만났을때, 파일의 이름(이전에 선택된)과 "Browse" 버튼이나 선택 방법이 표시 될 수 있다.
"Browse" 버튼을 선택하면, 브라우저가 플랫폼에 맞는 적절한 파일 선택 모드를 보여준다.
예를 들어, Window 기반 브라우저는 파일 선택 창이 팝업으로 나올 것이다.
파일 선택 대화상자에서 사용자는 현재 선택된 값을 변경하거나 새로운 파일을 추가할 수 있다.
브라우저를 구현할 때, 파일 이름의 목록을 수동으로 수정할 수 있도록 선택할 수 있다.

ACCEPT 속성이 존재할 때, 브라우저는 플랫폼을 위한 적절한 파일 확장자와 매치시킬 파일 패턴을 제한 할 수 있다.



## 3.2 Action on submit
When the user completes the form, and selects the SUBMIT element, the browser should send the form data and the content of the selected files.  
The encoding type application/x-www-form-urlencoded is inefficient for sending large quantities of binary data or text containing non-ASCII characters.  
Thus, a new media type, multipart/form-data, is proposed as a way of efficiently sending the values associated with a filled-out form from client to server.  



## 3.2 제출시 행동
사용자가 폼을 완성하고, SUBMIT(제출) 요소를 선택했을 때, 브라우저는 폼 데이터와 선택된 파일의 내용을 보내게 된다.  
"application/x-www-form-urlencoded" 인코딩 타입은 큰 용량의 바이너리 데이터나 ASCII 문자가 아닌 텍스트를 전송할 때, 불필요하다.  
그러므로, 새로운 미디어 타입인 mutipart/form-data는 채워진 폼과 관련된 값을 클라이언트에서 서버로 보내는 효율적인 방법으로 제한되었다.



## 3.3 use of multipart/form-data
The definition of multipart/form-data is included in section 7.  
A boundary is selected that does not occur in any of the data.  
(This selection is sometimes done probabilisticly.)  
Each field of the form is sent, in the order in which it occurs in the form, as a part of the multipart stream.  
Each part identifies the INPUT name within the original HTML form.  
Each part should be labelled with an appropriate content-type if the media type is known (e.g., inferred from the file extension or operating system typing information) or as application/octet-stream.
If multiple files are selected, they should be transferred together using the multipart/mixed format.

While the HTTP protocol can transport arbitrary BINARY data, the default for mail transport (e.g., if the ACTION is a "mailto:" URL) is the 7BIT encoding.  
The value supplied for a part may need to be encoded and the "content-transfer-encoding" header supplied if the value does not conform to the default encoding.  
[See section 5 of RFC 1521 for more details.]

The original local file name may be supplied as well, either as a 'filename' parameter either of the 'content-disposition: form-data' header or in the case of multiple files in a 'content-disposition: file' header of the subpart.  
The client application should make best effort to supply the file name; if the file name of the client's operating system is not in US-ASCII, the file name might be approximated or encoded using the method of RFC 1522.  
This is a convenience for those cases where, for example, the uploaded files might contain references to each other, e.g., a TeX file and its .sty auxiliary style description.

On the server end, the ACTION might point to a HTTP URL that implements the forms action via CGI.  
In such a case, the CGI program would note that the content-type is multipart/form-data, parse the various fields (checking for validity, writing the file data to local files for subsequent processing, etc.).



## 3.3 multipart/form-data 의 사용
multipart/form-data의 정의는 섹션7 안에 포함되어 있다.  
어떠한 데이터에도 나타나지 않는 경계가 선택되었다.  
(이 선택은 때때로 확률적으로 수행된다.)  
폼의 각각의 필드는 multipart 스트림의 일부로써 폼 안에서 발생하는 순서대로 보낸다.  
각 파트는 원래 HTML 폼 안에 있는 INPUT의 이름으로 식별된다.  
미디어 타입이 알려진 경우(파일 확장자에서 예측되거나 운영체제의 타입 정보)나 application/octet-stream 인 경우, 각 파트는 적절한 content-type 으로 라벨을 붙여야 한다.  
여러개의 파일이 선택된다면, multipart/mixed 형식을 사용해서 함께 전송해야한다.

HTTP 프로토콜이 임의의 바이너리 데이터를 전송할 수 있는 반면에, mail 전송(ACTION이 "mailto:" URL을 사용한다면,)을 위한 기본은 7BIT 인코딩이다.  
값이 기본 인코딩을 따르지 않으면, "content-transfer-encoding" 헤더를 지원하고, 각 파트에 제공된 값을 인코딩해야 할 수 있다.  
[RFC1521 의 섹션 5에 자세한 사항이 적혀있다.]

원본 로컬 파일의 이름은 'content-disposition: form-data' 헤더 또는 서브 파트의 'content-disposition: file' 헤더 안에 있는 여러 파일들인 경우, 'filename' 파라미터로써 제공되어야 할 수 있다.  
클라이언트 어플리케이션은 파일 이름을 제공하기 위한 최대의 노력을 해야한다.  
만약 클라이언트 운영체제의 파일 이름이 US-ASCII가 아닌 경우, 파일 이름은 RFC1522 의 방법을 사용하여, 비슷하게 되거나 인코딩 될 수 있다.   
예를 들어, 업로드 된 파일에 TeX 파일 및 .sty 보조 스타일 설명과 같이 서로에 대한 참조가 포함될 수있는 경우에 편리하다.

서버측에서는 ACTION은 CGI를 통해 폼 액션을 구현한 HTTL URL을 가리킬수 있다.  
그런 경우에는, CGI 프로그램은 content-type 을 multipart/form-data 로 적고, 다양한 필드를 구문 분석한다.  
(유효성 검사, 후속 처리를 위해 파일 데이터를 로컬 파일에 기록, 기타.)
In such a case, the CGI program would note that the content-type is multipart/form-data, parse the various fields (checking for validity, writing the file data to local files for subsequent processing, etc.).


## 3.4 Interpretation of other attributes
The VALUE attribute might be used with ```<INPUT TYPE=file>``` tags for a default file name.  
This use is probably platform dependent.  
It might be useful, however, in sequences of more than one transaction, e.g., to avoid having the user prompted for the same file name over and over again.  

The SIZE attribute might be specified using SIZE=width,height, where width is some default for file name width, while height is the expected size showing the list of selected files.  
For example, this would be useful for forms designers who expect to get several files and who would like to show a multiline file input field in the browser (with a "browse" button beside it, hopefully).  
It would be useful to show a one line text field when no height is specified (when the forms designer expects one file, only) and to show a multiline text area with scrollbars when the height is greater than 1 (when the forms designer expects multiple files).



## 3.4 다른 속성들의 해석
VALUE 속성은 기본 파일 이름을 위한 ```<INPUT TYPE=file>``` 태그에서 사용될 수 있다.  
이것을 사용하는 것은 플랫폼에 의존적이다.  
하지만, 사용자에게 같은 파일의 이름을 계속 묻지 않도록 하기 위해서 하나의 트랜젝션보다 많은 시퀀스 안에서 유용할 수 있다.  

SIZE 속성은 SIZE=width, height 을 사용하여 명세될 수 있다.  
여기서 width는 파일 이름 너비의 기본 값이고, height는 선택한 파일들의 리스트를 보여주는 사이즈의 예측 값이다.  
예를 들어, 이것은 여러개의 파일들을 가져오거나 브라우저 안에서 여러줄의 파일 입력 필드를 보여주려는 폼 디자이너에게 유용할 수 있다.  
height 가 명세되지 않았을때, 한줄의 텍스트 필드를 보여줄때와 height 가 1보다 커서(폼 디자이너가 여러 파일들을 예상할때) 스크롤바와 함께 여러줄의 텍스트 구역을 보여줄때 유용할 수 있다.



# 4. Backward compatibility issues
While not necessary for successful adoption of an enhancement to the current WWW form mechanism, it is useful to also plan for a migration strategy: users with older browsers can still participate in file upload dialogs, using a helper application.  
Most current web browers, when given ```<INPUT TYPE=FILE>```, will treat it as ```<INPUT TYPE=TEXT>``` and give the user a text box.  
The user can type in a file name into this text box.  
In addition, current browsers seem to ignore the ENCTYPE parameter in the ```<FORM>``` element, and always transmit the data as application/x-www-form-urlencoded.  

Thus, the server CGI might be written in a way that would note that the form data returned had content-type application/x-www-form-urlencoded instead of multipart/form-data, and know that the user was using a browser that didn't implement file upload.

In this case, rather than replying with a "text/html" response, the CGI on the server could instead send back a data stream that a helper application might process instead;  
this would be a data stream of type "application/x-please-send-files", which contains:

* The (fully qualified) URL to which the actual form data should be posted (terminated with CRLF)
* The list of field names that were supposed to be file contents (space separated, terminated with CRLF)
* The entire original application/x-www-form-urlencoded form data as originally sent from client to server.

In this case, the browser needs to be configured to process application/x-please-send-files to launch a helper application.

The helper would read the form data, note which fields contained 'local file names' that needed to be replaced with their data content, might itself prompt the user for changing or adding to the list of files available, and then repackage the data & file contents in multipart/form-data for retransmission back to the server.

The helper would generate the kind of data that a 'new' browser should actually have sent in the first place, with the intention that the URL to which it is sent corresponds to the original ACTION URL.  
The point of this is that the server can use the "same" CGI to implement the mechanism for dealing with both old and new browsers.

The helper need not display the form data, but "should" ensure that the user actually be prompted about the suitability of sending the files requested (this is to avoid a security problem with malicious servers that ask for files that weren't actually promised by the user.)  
It would be useful if the status of the transfer of the files involved could be displayed.



# 4. 이전 버전과 호환성 문제
현재 WWW 폼 메커니즘을 향상시키기 필요하지는 않지만, 마이그레이션 전략을 위해서는 유용하다.  
예전 브라우저의 사용자들은 helper 어플리케이션을 사용하여, 여전히 파일 업로드 대화상자를 사용할 수 있다.  
```<INPUT TYPE=FILE>```가 주어졌을 때, 현재 대부분의 웹브라우저들은 ```<INPUT TYPE=TEXT>```것과 같이 다르고 사용자에게 텍스트 박스를 준다.  
사용자는 이 텍스트 박스에 파일 이름을 입력할 수 있다.  
게다가, 현재 브라우저들은 ```<FORM>``` 안의 ENCTYPE 파라미터를 무시하고, 항상 application/x-www-form-urlencoded 으로 데이터를 전송한다.

따라서 서버 CGI는 반환된 양식에 따라 multipart/form-data 대신에 application/x-www-form-urlencoded를 content-type으로 가지며, 사용자가 파일 업로드를 구현하지 않은 브라우저를 사용하고 있음을 알 수 있는 방식으로 작성될 수 있다.

이 경우에, "text/html" response 로 응답하는 것보다, 서버의 CGI가 helper 응용 프로그램이 처리할 수 있는 데이터 스트림을 대신 보낼수 있다.  
이것은 "application/x-please-send-files" 타입의 데이터 스트림으로 아래에 있는 것들이 포함된다.

* 실제 폼 데이터를 게시해야하는 URL(정규화된) (CRLF로 끝나는)
* 파일 내용이 될 것으로 예상되는 필드 이름들의 목록 (띄어쓰기로 구분되고 CRLF로 끝나는)
* 클라이언트에서 서버로 전송되는 원본으로써, 전체 원본 application/x-www-form-urlencoded 폼 데이터

이 경우, 브라우저는 helper 어플리케이션을 실행하기 위해 application/x-please-send-files 프로세스를 설정하는 것이 필요하다.

helper는 폼 데이터를 읽고, 데이터 컨텐츠와 함께 교체되는 것이 필요한 '로컬 파일 이름들'을 포함하는 필드가 이용가능한 파일 리스트가 변경되거나 추가된 것을 위해 사용자에게 메시지를 표시하거나 서버로 다시 재전송하기 위해 multipart/form-data 안의 데이터와 파일 내용을 다시 패키지 할 수 있다.

helper는 URL이 원본 ACTION URL에 해당한다는 의미로 '새로운' 브라우저가 실제로 보낸 첫번쨰 데이터의 유형을 생성한다.  
요점은 서버가 "동일한' CGI를 사용하여, 예전과 새 브라우저를 모두 처리 할 수 있는 메커니즘을 구현할 수 있따는 것이다.

helper는 폼 데이터를 보여주지 않아도 되지만 사용자가 실제로 요청된 파일을 전송하는데 적합한지에 대해 묻는 메시지를 표시하는 것이 "반드시" 보장되야 한다. (이는 실제로 사용자가 약속하지 않은 파일을 요구하는 악의적인 서버의 보안 문제를 피하기 위함이다.)  
관련된 파일들의 전송 상태를 보여줄수 있을 때 유용하다.


# 5. Other considerations
## 5.1 Compression, encryption
This scheme doesn't address the possible compression of files.  
After some consideration, it seemed that the optimization issues of file compression were too complex to try to automatically have browsers decide that files should be compressed.  
Many link-layer transport mechanisms (e.g., high-speed modems) perform data compression over the link, and optimizing for compression at this layer might not be appropriate.  
It might be possible for browsers to optionally produce a content-transfer-encoding of x-compress for file data, and for servers to decompress the data before processing, if desired;  
this was left out of the proposal, however.

Similarly, the proposal does not contain a mechanism for encryption of the data;  
this should be handled by whatever other mechanisms are in place for secure transmission of data, whether via secure HTTP or mail.



# 5. 다른 고려사항
## 5.1 압축, 암호화
이 스키마는 가능한 파일들의 압축에 대해서 다루지 않는다.  
몇가지 고려 사항을 생각해 볼때, 파일 압축의 최적화 문제가 너무 복잡해서 브라우저가 파일을 압축해야한다고 결정한 것으로 보였다.  
많은 link-layer 전송 메커니즘은 링크 위의 데이터 압축을 수행하고, 이 계층에서 압축을 위한 최적화가 적합하지 않을 수 있다.  
브라우저가 파일 데이터를 위한 x-compress의 content-transfer-encoding 을 선택적으로 생산할 수도 있고, 필요하다면, 서버가 데이터를 처리하기 전에 압축을 풀수도 있다.  
그러나 이 제안은 여기서는 제외되었다.

유사하게, 이 제안은 데이터 암호화를 위한 메커니즘을 포함하고 있지 않다.  
보안 HTTP 나 mail을 통한 데이터의 안전한 전송을 위해 다른 모든 메커니즘들이 다뤄져야 한다.


## 5.2 Deferred file transmission
In some situations, it might be advisable to have the server validate various elements of the form data (user name, account, etc.)  before actually preparing to receive the data.  
However, after some consideration, it seemed best to require that servers that wish to do this should implement this as a series of forms, where some of the data elements that were previously validated might be sent back to the client as 'hidden' fields, or by arranging the form so that the elements that need validation occur first.  
This puts the onus of maintaining the state of a transaction only on those servers that wish to build a complex application, while allowing those cases that have simple input needs to be built simply.

The HTTP protocol may require a content-length for the overall transmission.  
Even if it were not to do so, HTTP clients are encouraged to supply content-length for overall file input so that a busy server could detect if the proposed file data is too large to be processed reasonably and just return an error code and close the connection without waiting to process all of the incoming data.  
Some current implementations of CGI require a content-length in all POST transactions.

If the INPUT tag includes the attribute MAXLENGTH, the user agent should consider its value to represent the maximum Content-Length (in bytes) which the server will accept for transferred files.  
In this way, servers can hint to the client how much space they have available for a file upload, before that upload takes place.  
It is important to note, however, that this is only a hint, and the actual requirements of the server may change between form creation and file submission.

In any case, a HTTP server may abort a file upload in the middle of the transaction if the file being received is too large.


## 5.2 지연된 파일 전송
몇몇 상황에서, 실제로 수신된 데이터를 준비하기 전에 서버가 다양한 폼 데이터의 요소(사용자 이름, 계정등)를 검사하는 것은 권고된다.  
그러나, 몇가지 고려사항을 볼때, 
However, after some consideration, it seemed best to require that servers that wish to do this should implement this as a series of forms, where some of the data elements that were previously validated might be sent back to the client as 'hidden' fields, or by arranging the form so that the elements that need validation occur first.  
This puts the onus of maintaining the state of a transaction only on those servers that wish to build a complex application, while allowing those cases that have simple input needs to be built simply.

The HTTP protocol may require a content-length for the overall transmission.  
Even if it were not to do so, HTTP clients are encouraged to supply content-length for overall file input so that a busy server could detect if the proposed file data is too large to be processed reasonably and just return an error code and close the connection without waiting to process all of the incoming data.  
Some current implementations of CGI require a content-length in all POST transactions.

If the INPUT tag includes the attribute MAXLENGTH, the user agent should consider its value to represent the maximum Content-Length (in bytes) which the server will accept for transferred files.  
In this way, servers can hint to the client how much space they have available for a file upload, before that upload takes place.  
It is important to note, however, that this is only a hint, and the actual requirements of the server may change between form creation and file submission.

In any case, a HTTP server may abort a file upload in the middle of the transaction if the file being received is too large.


# 참조
[RFC 1867](https://tools.ietf.org/html/rfc1867)