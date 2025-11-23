## HTTP

> HyperText Transfer Protocol
WWW에서 하이퍼텍스트 문서를 교환하기 위하여 사용하는 통신규약
> 

### HTTP 프로토콜의 특징은?

- 비연결 지향
    - 클라이언트가 request를 서버에 보내고, 서버가 클라이언트에 요청에 맞는 response를 보내면 바로 연결을 끊음
- 상태정보 유지 안함
    - 연결을 끊는 순간 클라이언트와 서버의 통신은 끝나며 상태 정보를 유지하지 않음

### HTTP 메소드에 대해서 설명하세요

- POST `Creat` : 서버나 특정 리소스에 엔티티를 제출할 때 사용
    
- GET `Read` : 특정 리소스의 참조를 요청. url에 어느 리소스를 참조 요청하는지 드러나게됨
    
- PUT `Update` : 전체 자원을 업데이트
    
- DELETE `Delete` : 삭제할 때 사용. 어느 자원을 삭제할지 url에 드러남
    
- PATCH : 리소스의 부분을 수정하는데 사용. 의미론적으로 Update에 더 가깝다고 볼 수 있음
    

### HTTP1.1 `vs` HTTP2.0 `vs` HTTP/3.0

- HTTP1.1
    - 동작방식
        
       <img width="403" height="600" alt="image" src="https://github.com/user-attachments/assets/edb9488b-d2d7-4980-b9c1-94deaa7073c5" />

        
        - Connection당 동시에 하나의 요청을 처리
        - 요청과 응답이 순차적으로 이루어짐
        - 다수의 리소스 처리하려면 리소스 개수에 비례해서 Latency가 길어짐
    - 단점
        - HOL (Head Of Line) Blocking - 특정 응답의 지연
            - Web환경에서 HOLB
                - HTTP의 HOL Blocking
                - TCP의 HOL Blocking
            - pipelining : 하나의 Connection을 통해서 다수개의 파일을 요청/응답 받을 수 있는 기법
                
                ```
                | --- a.png --- |
                            | --- b.png --- |
                                        | --- c.png --- |
                ```
                
                순서대로 이미지 요청을 보내는데, 첫번째 이미지 요청의 응답이 지연되면 대기하게됨
                
                ```
                | ---------------- a.png ------------------|                                                             | -b.png- |                                                               | --c.png-- |
                ```
                
                +) 추가로, 파이프라이닝은 구현이 어려웠고, 프록시(중계) 서버가 파이프라이닝을 제대로 처리하지못해서 크롬, 파이어폭스 같은 브라우저는 결국 이 파이프라이닝을 지원하지 않게 됨
                
        - RTT(Round Trip Time) 증가
            
            > RTT : 데이터 패킷이 출발지에서 목적지로 갔다가 다시 돌아오는 데 걸리는 시간
            요청(SYN)을 보낼 때부터 요청에 대한 응답(SYN+ACK)을 받을 때까지의 왕복시간
            > 
            - 매 요청별로 connection을 만들게 됨
            - TCP상에서 동작하는 HTTP의 특성상 3-way Handshake가 반복적으로 일어나고
            불필요한 RTT 증가와 네트워크 지연 초래 → 성능 저하
        - 무거운 Header 구조 (특히 Cookie)
            - 사용자가 방문한 웹페이지는 다수의 http 요청이 발생하게 됨
                
                → 매 요청시 중복된 헤더값 전송 (별도의 domain sharding 하지 않았을 경우)
                
                → 해당 domain에 설정된 cookie 정보도 매 요청시 헤더에 포함되어 전송
                
                → 전송하려는 값보다 헤더 값이 더 큰 경우 있을 수 있음
                
    - 극복 방법
        - Image Spriting
            
            웹 페이지를 구성하는 다양한 아이콘 이미지 파일의 요청 횟수를 줄이기 위해 아이콘을 하나의 큰 이미지로 만든다음 CSS에서 해당 이미지의 좌표 값을 지정해 표시
            
            <img width="205" height="300" alt="image" src="https://github.com/user-attachments/assets/5a4010cb-7db7-4457-9da2-984c265420da" />

            
        - Domain Sharding
            
            다수의 Connection을 생성해서 병렬로 요청을 보냄
            
            but 브라우저별 Domain당 Connection개수 제한 존재
            
            ⇒ 근본적인 해결이 아님
            
            필요한 다운로드가 둘 이상의 도메인에서 제공되므로, 브라우저가 필요한 리소스를 동시에 다운로드 가능
            
            but 다중 도메인은 DNS 조회로 인해 초기 로드 시간이 느려지므로 안티패턴
            
            <img width="1600" height="752" alt="image" src="https://github.com/user-attachments/assets/f5fb2c92-444b-4642-b9b6-6b250f714b00" />

            
            <img width="756" height="946" alt="image" src="https://github.com/user-attachments/assets/d7ea0905-62ae-42b6-8087-d94475c64aef" />
            -> 서브 도메인이 계속 바뀌고 있음
            
        - Minify CSS/Javascript
            
            http를 통해서 전송되는 데이터의 용량을 줄이기 위해 CSS, Javascript 코드를 축소하여 적용
            
            ```jsx
            // 주석주석주석
            // 주석주석주석
            // 주석주석주석주석
            function sayHi(name){
            	console.log("Hi"+name+"nice to meet you.");
            }
            
            sayHi("Sam");
            ```
            
            ```jsx
            function sayHi(o){console.log("Hi"+o+"nice to meet you.")}sayHi("Sam");
            ```
            
            로딩 속도를 높여 방문자와 검색 엔진 모두 만족 가능함
            
        - Data URI Scheme
            - HTML문서내 이미지 리소스를 Base64로 인코딩된 이미지 데이터로 직접 기술하는 방식
            - 요청 수를 줄임
        - Load Faster
            - 스타일시트를 HTML 문서 상위에 배치
            - 스크립트를 HTML 문서 하단에 배치
- 구글의 SPDY
    
    <img width="593" height="278" alt="image" src="https://github.com/user-attachments/assets/e73af134-7f38-4d3a-a31f-095d0fbed876" />

    
    - 더 빠른 Web을 실현하기 위해 throughput 관점이 아닌 Latency 관점에서 HTTP를 고속화한 SPDY라는 새로운 프로토콜을 구현함
    - HTTP/2 초안의 참고 규격
- HTTP/2
    - Stream과 Frame
        - HTTP/1.1 ) 텍스트 Message 단위로 구성
        - HTTP/2 ) Frame, Stream이라는 단위가 추가됨
        
        ---
        
        - Frame: HTTP/2에서 통신의 최소 단위. Header 혹은 Data가 들어있음
        - Message: HTTP/1.1과 마찬가지로 요청 혹은 응답의 단위. 다수의 Frame으로 이루어진 배열 라인
        - Stream: 연결된 Connection 내에서 양방향으로 Message를 주고 받는 하나의 흐름
        
        > 
        > HTTP 요청을 여러개의 Frame들로 나누고, 
        > frame들이 모여 요청/응답 Messasge가 되고,
        > Message는 특정 Stream에 속하게 되고,
        > 여러개의 Stream은 하나의 Connection에 속하게 되는 구조
        > 
        > <img width="1464" height="826" alt="image" src="https://github.com/user-attachments/assets/c27905d3-d104-487a-b974-26f782fcb369" />


    - Multiplexed Streams
        - 한 커넥션으로 동시에 여러개의 메세지를 주고 받을 수 있음
        - 응답은 순서에 상관없이 stream으로 주고받음
        - HTTP/1.1의 Connection Keep-Alive, Pipelining의 개선
        - latency만 줄여주는게 아니라 결국 네트워크를 효율적으로 사용할 수 있게 하고 그 결과 네트워크 비용을 줄임
        
        <img width="850" height="698" alt="image" src="https://github.com/user-attachments/assets/972b5eee-301a-42cd-aab5-f0097fa4b675" />

        
        - 단점
            - 여전히 TCP Connection에서 frame들을 주고 받았기 때문에, TCP에서 발생하는 HOL Blocking을 해결할 수 없음
            - Application Layer 에서의 HOL Blocking 은 해결했지만 Transport Layer 에서의 HOL Blocking 은 여전히 문제
            - stream2 header 의 패킷이 손실이 되었다면 해당 stream2 header 는 손실된 패킷이 올 때까지 기다리게 되고, 그 뒤에있는 stream 1 data, 3 data, 2data 는 여전히 기다림
    - Stream Prioritization
        - 문제 상황) css 파일 1개, image 2개 존재하고 클라이언트가 각각 요청하고난 후 Image파일보다 CSS파일의 수신이 늦어지는 경우 브라우저 렌더링이 늦어짐
        - 해결) 리소스간 의존관계(우선순위)를 설정
        - HTTP 메세지가 개별 바이너리 프레임으로 분할되고, 여러 프레임을 멀티플렉싱 할 수 있게 되면서 요청과 응답이 동시에 이루어져 속도 향상됨
            
            but, 하나의 연결에 여러 요청과 응답이 뒤섞여 버려 패킷 순서가 엉망진창
            
            ⇒ 스트림들의 우선순위를 지정할 필요가 생김
            
            ⇒ 클라이언트는 우선순위 지정 트리를 사용하여 스트림에 식별자를 설정함으로써 해결함
            
            <img width="761" height="922" alt="image" src="https://github.com/user-attachments/assets/0a83107f-0125-4ac7-8a6e-db043f8e2a9e" />

            
            - 통신과정
                1. 클라이언트는 서버에게 스트림을 보낼때, 각 요청 자원에 가중치 우선순위를 지정하고 보냄
                2. 요청받은 서버는 우선순위가 높은 응답이 클라이언트에 우선적으로 전달될 수 있도록 대역폭 설정
                3. 응답 받은 각 프레임에는 이것이 어떤 스트림인지에 대한 고유한 식별자가 있어, 클라이언트는 여러개의 스트림을 interleaving을 통해 서로 끼워놓는 식으로 조립
            - 최신 브라우저들은 자원의 종류, 페이지가 로드된 위치 그리고 이전 페이지 방문에서 학습한 결과에 따라 자원 요청의 우선순위를 결정
    - Server Push
        - 서버가 클라이언트의 요청에 대해 요청하지 않은 리소스를 마음대로 보내줄 수 있음
        - HTTP/1.1 ) HTML 문서를 수신한 후 HTML 문서를 해석하면서 필요한 리소스를 재요청
        - HTTP/2 ) 클라이언트가 요청하지도 않은 리소스를 PUSH → 클라이언트 요청 최소화해서 성능 향상
            - PUSH_PROMISE
        - 클라이언트가 서버가 보낸 리소스를 알지 못하고, 리소스를 다시 요청한다는 점에서 22년 10월 Google 은 Chrome 에서 Server push 기능을 제거
    - Binary Framing Layer
        - HTTP/1.1 ) HTTP 메세지가 text로 전송
            
            → 본문은 압축되지만 헤더는 압축이 되지 않으며 헤더 중복값이 있다는 문제가 있었음
            
        - HTTP/2.0 ) binary frame으로 인코딩되어 전송
            - 헤더와 바디가 layer로 구분
        
        <img width="820" height="433" alt="image" src="https://github.com/user-attachments/assets/2e6de26a-b886-4630-85a5-e43436b0f7c4" />

        
    - Header Compression
        - Header 정보를 압축하기 위해 Header Table과 Huffman Encoding 기법을 사용하여 처리
        - HPACK 압축방식
        - Header에 중복값이 존재하는 경우 Static/Dynamic Header Table 개념을 사용하여 중복 Header를 검출하고 중복된 Header는 index값만 전송하고 중복되지 않은 Header정보의 값은 Huffman Encoding 기법으로 인코딩 처리하여 전송
        
        <img width="2000" height="696" alt="image" src="https://github.com/user-attachments/assets/cc9e3e4e-f035-4cb4-8d0a-853b9b8d89fd" />

        
    - 단점
        - 여전한 RTT
            - TCP를 이용하기 때문에 Handshake의 RTT로 인한 지연시간이 발생
            - **결국 원초적으로 TCP로 통신하는 것이 문제**
        - TCP 자체의 HOLB
            - 멀티플렉싱을 통해 파이프라이닝 HOLB 문제를 해결
            - 하지만 TCP는 패킷이 유실되거나 오류가 있을 때 재전송
                
                → 이 재전송 과정에서 패킷의 지연이 발생하면 결국 HOLB 문제가 발생
                
                ⇒ 애플리케이션 계층(L4)에서 HTTP HOLB를 해결했다 하더라도 전송계층(L3)에서의 TCP HOLB를 해결한 것은 아니기 때문
                
        - 중개자 캡슐화 공격
            - 헤더 필드의 이름과 값을 바이너리로 인코딩
            ⇒ 헤더 필드로 어떤 문자열이든 사용할 수 있음
                
                ⇒ 중간의 Proxy 서버가 HTTP 1.1 메시지로 변환할 때 메시지를 불법 위조할 수 있다는 위험성
                
        - 길다란 커넥션 유지로 인한 개인정보 누출 우려
            - 기본적으로 성능을 위해 클라이언트와 서버 사이의 커넥션을 오래 유지하는 것을 염두에 두고 있음
                
                ⇒ 개인 정보 유출에 악용될 가능성이 있음  `Keep-Alive`
                
        
        > 결론) TCP가 문제다!!!
        > 
- HTTP 3.0
    - TCP를 버리고 UDP 채택
        
        ⇒ UDP를 개조한 **QUIC 프로토콜**을 새로 만듦
        
        - 기존 TCP는 클라이언트와 서버 간에 세션을 설정하기 위해 핸드쉐이크가 필요하며 인증서인 TLS도 세션이 보호되도록 자체 핸드 셰이크도 필요함
        - QUIC는 보안 세션을 설정하기 위해 한번만의 핸드셰이크 필요
    - QUIC 프로토콜이 TCP/IP 4계층에도 동작시키기 위해 설계된 것
    - RTT를 제로 수준으로 줄임
        - 최초 연결 시에는 1 RTT를 필요로 하지만, **재연결의 경우 0 RTT(Zero Round Trip Time)로 빠르게 시작 가능**
    - 패킷 손실에 대한 빠른 대응
    - 사용자 IP가 바뀌어도 연결이 유지되는 것이 특징
    - QUIC 프로토콜
        - TCP + TSL + HTTP의 기능을 모두 구현한 프로토콜
        - TCP 프로토콜의 무결성 보장 알고리즘 + SSL ⇒ 높은 성능 + 신뢰성
    - TCP가 아닌 UDP인 이유
        - TCP는 구조상 한계로 개선해도 여전히 느림
            - 클라이언트와 서버가 동시 도발적으로 여러 개 파일의 데이터 패킷을 교환할 것이라 상상하지 못하던 시기에 만들어짐
                
                ⇒ 모바일 기기와 같이 네트워크 환경을 바꾸어가면서 서버와 클라이언트가 소통할 수 있을 것이라고 생각 X
                
                ⇒ 와이파이 바꾸면 다시 새로운 커넥션 → 끊김현상
                
            - 신뢰성을 위해 무조건 순서대로 처리되어야 함
                - HOLB 발생
        - UDP는 신뢰성이 없는 것이 아니라 탑재를 안한 것
            - 데이터그램 방식
            - 패킷의 목적지만 정해져있다면 중간 경로는 신경쓰지 않음 → 핸드쉐이크 필요 X
            - 커스터마이징이 가능
                
                ⇒ 개발자가 애플리케이션에서 구현을 어떻게 하냐의 따라서 TCP와 비슷한 수준의 기능을 가질 수 있음
                
    - 연결 시 Latency 감소
        - TLS 연결을 위한 핸드쉐이크와 TCP를 위한 핸드쉐이크가 발생했었음
            
            → 1 RTT가 필요하고 TLS를 이용한 암호화 통신까지 한다면 3 RTT가 필요했음
            
        - QUIC는 한단계로 줄임
            - 3 Way Handshake 과정을 거치지 않아도 되기 때문에 첫 연결 설정에 1 RTT만 소요
            - **연결 설정에 필요한 정보와 함께 데이터도 보내버리기 때문**
            - TLS 인증서를 내포하고 있기 때문에 최초의 연결설정에서 필요한 인증 정보와 데이터를 함께 정송
            - 다만 최초의 요청을 보낼때는 클라이언트는 서버의 세션 키를 모르는 상태이기 때문에, 목적지인 서버의 Connection ID를 사용하여 생성한 특별한 키인 초기화 키를 사용하여 통신을 암호화함
            - 한번에 연결에 성공했다면 서버는 그 설정을 캐싱
                
                → 다음 연결 때 캐시를 불러와 바로 연결
                
                ⇒ 추가적인 핸드쉐이크 없이 0 RTT만으로 바로 통신 시작 가능
                
    - 잔존하던 HOLB 현상 해결
        - 독립 스트림으로 HOLB 단축
        - 아예 스트림 자체를 독립적으로 여러개로 나누어서 처리
        - 특정 스트림에서 HOLB가 발생하더라도 다른 스트림에는 영향을 미치지 않음
    - 패킷 손실 감지에 걸리는 시간 단축
        - 흐름 제어 시간 단축
        - 헤더에 패킷의 전송 순서를 나타내는 별도의 패킷 번호 공간 부여
            
            ⇒ 패킷 번호를 파악해 개별 파일을 구붑ㄴ하여 중간에 패킷 로스가 발생하더라도 해당 파일의 스트림만 정지 가능
            
    - 더욱 향상된 멀티플렉싱
        
        <img width="1744" height="1038" alt="image" src="https://github.com/user-attachments/assets/0f559447-cc15-4e44-a967-c7dafc1f5a24" />

        
    - 보안을 더욱 강화
        - QUIC 내에 TLS 포함
        - 헤더 영역도 같이 암호화
        
    - 네트워크가 변경되도 연결이 유지
        - TCP
            - 클라이언트 IP, 클라이언트 PORT, 서버 IP, 서버 PORT 필요
                
                ⇒ 클라이언트의 IP가 바뀌는 상황이 발생하면 연결이 끊어짐
                
            - 핸드쉐이크를 다시 거쳐야 해서 지연시간 발생
        - Connection ID
            - QUIC는 Connection ID를 사용하여 서버와 연결을 생성
                - 랜덤한 값이라 클라이언트 IP가 변경되더라도 기존의 연결을 유지할 수 있음
                - but 해커가 네트워크를 통해 사용자를 추적하여 보안 문제가 발생할 수 있음
                    
                    ⇒ 새 네트워크가 사용될 때마다 Connection ID 변경
                    
                    ⇒ Connection ID를 바꾸더라도 이전 Connection ID와 동일하다고 인지하여 연결 유지
                    
    - 우려점
        - 기존 체계 호환성 문제
        - 암호화로 네트워크 제어가 힘듦
        - 암호화로 리소스가 많이 듦
        - CPU를 너무 많이 사용함

| **항목** | **HTTP/1.1** | **HTTP/2.0** | **HTTP/3.0 (QUIC)** |
| --- | --- | --- | --- |
| 연결 방식 | 하나의 TCP 연결에 하나의 요청 | 하나의 TCP 연결에 여러 요청 병렬 처리 | UDP 기반 QUIC 프로토콜 사용 |
| HOL Blocking | 발생 (순차 처리) | 애플리케이션 레이어에서 해결 | 전송 레이어에서 해결 |
| 성능 | 요청/응답 순차 처리, RTT 증가 | 멀티플렉싱, 스트림 우선순위, 헤더 압축 | 0-RTT, 연결 이동성, 빠른 재전송 |
| 보안 | 기본적으로 암호화 없음 | TLS 기본 적용 | TLS 내장, 더 강력한 보안 |

### HTTP와 HTTPS 차이점은 무엇인가요?

- HTTP
    - OSI 네트워크 통신 모델의 애플리케이션 계층 프로토콜
    - 암호화되지 않은 데이터를 전송
        
        ⇒ 브라우저에서 전송된 정보를 제3자가 가로채고 읽을 수 있음
        
- HTTPS
    - 브라우저와 서버가 데이터를 전송하기 전에 안전하고 암호화된 연결을 설정
    - HTTP 요청 및 응답을 SSL 및 TLS 기술에 결합
    - 독립된 인증 기관(CA)에서 SSL/TLS 인증서를 획득해야 함
        - SSL 인증서는 암호화 정보도 포함하므로 서버와 웹 브라우저는 암호화된 데이터나 스크램블된 데이터를 교환 가능
        1. 사용자 브라우저의 주소 표시줄에 *https://* URL 형식을 입력하여 HTTPS 웹 사이트를 방문
        2. 브라우저는 서버의 SSL 인증서를 요청하여 사이트의 신뢰성을 검증하려고 시도
        3. 서버는 퍼블릭 키가 포함된 SSL 인증서를 회신으로 전송
        4. 웹 사이트의 SSL 인증서는 서버 아이덴티티를 증명. 브라우저에서 인증되면, 브라우저가 퍼블릭 키를 사용하여 비밀 세션 키가 포함된 메시지를 암호화하고 전송
        5. 웹 서버는 프라이빗 키를 사용하여 메시지를 해독하고 세션 키를 검색. -> 세션 키를 암호화하고 브라우저에 승인 메시지를 전송
        6. 이제 브라우저와 웹 서버 모두 동일한 세션 키를 사용하여 메시지를 안전하게 교환하도록 전환
- HTTP보다 HTTPS를 선택하는 이유
    - 보안
        - HTTP 메시지는 일반 텍스트이므로 권한 없어도 쉽게 액세스 가능
        - HTTPS는 모든 데이터를 암호화된 형태로 전송
    - 권위
        - 검색 엔진은 HTTP의 신뢰성이 더 낮기 때문에 콘텐츠의 순위를 HTTPS보다 낮게 지정
    - 성능 및 분석
        - HTTPS 어플리케이션이 HTTP보다 로드 속도가 더 빠름

| **항목** | **HTTP** | **HTTPS** |
| --- | --- | --- |
| 암호화 | 없음 (평문 전송) | TLS/SSL로 암호화 |
| 포트 | 80 | 443 |
| 보안성 | 낮음 (도청, 변조 가능) | 높음 (도청, 변조 방지) |
| 인증서 | 필요 없음 | SSL/TLS 인증서 필요 |
| URL 표기 | http:// | https:// |

## 쿠키와 세션

### 쿠키와 세션의 필요성은?

> 웹서버가 사용자(브라우저)의 상태 정보를 기억하기 위해 쿠키와 세션을 사용함
> 
- HTTP 프로토콜은 모든 요청 간 의존관계가 없음
- 즉, 현재 접속한 사용자가 이전에 접속했던 사용자와 같은 사용자인지 아닌지 알 수 있는 방법이 없음
- 계속해서 연결을 유지하지 않기 때문에 리소스 낭비가 줄어드는 것이 큰 장점
but 통신할 때마다 새로 연결하기 때문에 클라이언트는 매 요청마다 인증을 해야 한다는 단점
- 이전 요청과 현재 요청이 같은 사용자의 요청인지 알기 위해서는 상태를 유지해야 함
- HTTP 프로토콜에서 상태를 유지하기 위한 기술로 쿠키와 세션이 있음
- 쿠키
    - 클라이언트 로컬에 저장되는 키와 값이 들어있는 파일
    - 이름, 값, 유효 시간, 경로 등을 포함
    - 클라이언트의 상태 정보를 브라우저에 저장하여 참조
    - 동작 방식
        1. 웹 브라우저가 서버에 요청
        2. 상태를 유지하고 싶은 값을 쿠키로 생성
        3. 서버가 응답할 때 HTTP 헤더(Set-Cookie)에 쿠키를 포함해서 전송
        4. 전달받은 쿠키는 웹 브라우저에서 관리하고 있다가 다음 요청 때 쿠키를 HTTP 헤더에 넣어서 전송
        5. 서버에서는 쿠키 정보를 읽어 이전 상태 정보를 확인한 후 응답
- 세션
    - 일정 시간 동안 같은 브라우저로부터 들어오는 요청을 하나의 상태로 보고 그 상태를 유지하는 기술
    - 웹 브라우저를 통해 서버에 접속한 이후부터 브라우저를 종료할 때까지 유지되는 상태
    - 동작 방식
        1. 웹 브라우저가 서버에 요청
        2. 서버가 해당 웹브라우저에 유일한 ID(Session ID)를 부여
        3. 서버가 응답할 때 HTTP 헤더(Set-Cookie)에 Session ID를 포함해서 전송
        쿠키에 Session ID를 JSESSIONID라는 이름으로 저장
        4. 웹 브라우저는 이후 웹 브라우저를 닫기까지 다음 요청 때 부여된 Session ID가 담겨있는 쿠키를 HTTP 헤더에 넣어서 전송
        5. 서버는 세션 ID를 확인하고 해당 세션에 관련된 정보를 확인한 후 응답

### 쿠키와 세션은 언제 사용하나요?

- 쿠키와 세션의 차이점
    - 저장위치
        - 쿠키: 클라이언트
        - 세션: 서버
    - 보안
        - 쿠키: 클라이언트에 저장되므로 보안에 취약
        - 세션: 쿠키를 이용해 Session ID만 저장하고 이 값으로 구분해서 서버에서 처리하므로 비교적 보안성 좋음
    - 라이프사이클
        - 쿠키: 만료시간에 따라 브라우저를 종료해도 계속해서 남아있을 수 있음
        - 세션: 만료시간을 정할 수 있지만 브라우저가 종료되면 만료시간에 상관없이 삭제
    - 속도
        - 쿠키: 클라이언트에 저장되어서 서버에 요청시 빠름
        - 세션: 실제 저장된 정보가 서버에 있으므로 서버의 처리가 필요해 쿠키보다 느림
- 쿠키를 사용하는 경우
    - 보안이 중요하지 않고 브라우저가 종료되어도 유지할 수 있는 정보
        - 팝업창 "오늘 다시는 보지 않기" 로그인시 "id 기억하기" 체크, 비로그인시 장바구니 넣기
- 세션을 사용하는 경우
    - 보안이 중요한 정보
        - 로그인 정보 유지

### 세션을 어떻게 관리할까요?

- 스티키 세션
    - 로드 밸런서가 쿠키가 없는 요청이라면 쿠키 값을 등록하고 웹서버를 지정
    - 쿠키가 있다면 해당 요청이 지정된 서버로 보냄
    - 단점
        - 로드밸런싱이 잘 동작하지 않을 수 있음
        - 고정된 세션을 사용하기에 특정 서버만 과부하가 올 수 있음
        - 특정 서버 Fail시 해당 서버에 붙어있는 세션들이 소실될 위험 있음
- 세션 클러스터링
    - 여러 대의 컴퓨터들이 연결되어 하나의 시스템처럼 동작하도록 만드는 것
        
        ⇒ 각 서버의 세션에 모든 세션 저장소의 세션 객체를 복제
        
    - 단점
        - 매번 세션 객체를 복제하는데 오버헤드가 발생
- 세션 스토리지 분리
    - 별도의 세션 저장소를 사용
    - 세션 저장소가 하나이므로 서버간 복제 필요 없음
    - 서버가 아무리 늘어난다고 하더라도 세션 스토리지에 대한 정보만 각각의 서버에 입력하면 세션 공유 가능
    - 트래픽이 몰리는 현상 고려하지 않아도 됨
    - 하나의 서버가 장애가 발생하더라도 서비스 계속 제공 가능

## Session 인증 방식과 Token 인증 방식의 차이점

- 세션방식은 Stateful, Token 방식은 Stateless
    - 세션 방식: 서버가 클라이언트 상태를 기억함
    - 토큰 방식: 서버는 상태를 기억하지 않고 증명서만 확인

### 세션 기반 인증

- 전통적인 웹표준(RFC 6265 - HTTP Cookies)을 활용한 방식
- 동작 원리
    - **로그인:** 클라이언트가 ID/PW를 전송
    - **세션 생성:** 서버는 메모리나 DB(예: Redis)에 **세션 ID**를 생성하고 유저 정보를 저장
    - **쿠키 발급:** 서버는 브라우저에게 `Set-Cookie` 헤더를 통해 세션 ID를 전달
    - **요청:** 이후 클라이언트는 요청 시마다 쿠키에 세션 ID를 담아 보냄 (브라우저가 자동으로 수행)
    - **검증:** 서버는 쿠키의 세션 ID를 보고 자신의 저장소(Session Store)를 뒤져서 유저 식별
- **저장 위치:** **서버** (메모리, DB 등)
- **트래픽 처리:** 요청마다 서버의 저장소를 조회해야 하므로 I/O 비용이 발생
- **확장성(Scalability) 문제:** 서버를 여러 대(Scale-out)로 늘릴 때 문제 발생. 이를 해결하기 위해 **Sticky Session**이나 **Session Clustering(Redis 등)**이 필수적
- **보안:** 세션 ID만 탈취당하면 위험하지만, 서버에서 해당 세션을 강제로 삭제(Invalidate)하여 즉각적인 대응이 가능

### 토큰 기반 인증

- 최근 MSA(Microservices Architecture)와 모바일 환경에서 표준처럼 쓰이는 방식
- JWT 사용
- 동작 원리
    1. **로그인:** 클라이언트가 ID/PW를 전송
    2. **토큰 발급:** 서버는 유저 정보를 기반으로 암호화된 서명(Signature)이 포함된*토큰(Access Token)을 생성하여 클라이언트에게 전송 **(서버에 저장하지 않음)**
    3. **저장:** 클라이언트는 이 토큰을 로컬 스토리지나 쿠키 등에 저장
    4. **요청:** 클라이언트는 `Authorization: Bearer <token>` 헤더에 토큰을 담아 보냄
    5. **검증:** 서버는 토큰의 서명(Signature)이 위조되지 않았는지만 수학적으로 계산하여 검증(DB 조회 X)
- **저장 위치:** **클라이언트** (서버는 키만 가짐)
- **트래픽 처리:** DB를 조회하지 않고 CPU 계산만으로 유효성을 검증하므로 인증 속도가 빠름
- **확장성(Scalability) 우수:** 서버가 상태를 저장하지 않으므로(Stateless), 서버를 100대로 늘려도 어떤 서버든 토큰의 서명만 확인하면 되므로 별도의 세션 동기화가 필요 없음
- **보안:** 토큰은 한 번 발급되면 만료될 때까지 서버가 제어할 수 없음(탈취되어도 강제 만료 불가능) -> 유효기간(Expiration Time)을 짧게 가져가고 **Refresh Token** 전략을 병행
- JWT가 무조건 좋은가요?
    - Payload가 커질수록 네트워크 대역폭을 많이 차지하며, 한 번 발급되면 서버에서 제어권을 잃는다는(Stateless) 단점
    - 강제 로그아웃 처리가 중요한 서비스라면 세션 방식이 더 적합할 수 있고, 대규모 트래픽 처리가 중요하다면 토큰 방식이 유리

| **비교 항목** | **세션 (Session)** | **토큰 (Token / JWT)** |
| --- | --- | --- |
| **상태 유지** | **Stateful** (서버가 기억함) | **Stateless** (서버는 기억 안 함) |
| **유저 정보 위치** | 서버의 메모리나 DB | 클라이언트가 가진 토큰 내부(Payload) |
| **서버 확장성** | 나쁨 (Redis 등 별도 스토리지 필요) | **매우 좋음** (서버 무한 증설 가능) |
| **보안 강점** | 탈취 시 서버에서 즉시 무효화 가능 | 별도 저장소 없이 검증 가능하여 가벼움 |
| **보안 약점** | **CSRF** 공격에 취약함 (쿠키 자동 전송 때문) | **XSS** 공격에 취약함 (JS로 탈취 가능 시), 탈취 시 즉시 차단 어려움 |
| **데이터 크기** | 작음 (Session ID 문자열만 전송) | 큼 (유저 정보가 포함되어 길어짐) |
| **적합한 서비스** | 보안이 매우 중요한 금융권, 관리자 페이지 | 동시 접속자가 많은 대용량 서비스, 앱(App) |

## DNS

> 사람이 읽을 수 있는 도메인 이름을 컴퓨터가 사용하는 IP 주소로 변환하는 계층적 분산 데이터베이스 시스템
> 

### DNS 쿼리의 전체 흐름

니다.
1. **브라우저 -> OS (Resolver Cache 확인):** 먼저 로컬 OS의 캐시 확인
2. **Resolver -> Root Name Server:** 캐시에 없으면, ISP(인터넷 서비스 제공업체)가 관리하는 **DNS Resolver**가 **Root Name Server(.)**에게 IP 주소를 요청
3. **Root -> TLD Name Server 주소 응답:** Root 서버는 `.com`, `.net` 등의 **TLD(Top-Level Domain) Name Server** 주소를 알려줌
4. **Resolver -> TLD Name Server:** Resolver는 TLD 서버에게 다시 요청
5. **TLD -> Authoritative Name Server 주소 응답:** TLD 서버는 해당 도메인의 IP를 직접 알고 있는 **Authoritative Name Server** 주소를 알려줌
6. **Resolver -> Authoritative Name Server:** Resolver는 최종 권한을 가진 Authoritative 서버에게 요청
7. **Authoritative -> IP 주소 응답:** Authoritative 서버는 최종 IP 주소를 알려줌
8. **Resolver -> 클라이언트에게 IP 전달 및 캐싱:** Resolver는 IP를 브라우저에 전달하고, 일정 시간 동안 캐싱

### **주요 DNS 레코드 타입**

| **타입** | **역할** | **설명** |
| --- | --- | --- |
| **A** | **Address Record** | 도메인 이름을 **IPv4** 주소로 매핑 (가장 일반적) |
| **AAAA** | **Quad-A Record** | 도메인 이름을 **IPv6** 주소로 매핑 |
| **CNAME** | **Canonical Name** | 별칭(Alias)을 설정. `blog.example.com`을 `www.example.com`으로 연결할 때 사용 |
| **NS** | **Name Server** | 해당 도메인의 권한을 가진 Name Server를 명시함 |
| **MX** | **Mail Exchanger** | 해당 도메인의 메일 서버 주소를 저장 |

### TTL(Time To Live)의 중요성

- DNS 레코드가 캐시에 저장될 수 있는 유효 시간
- 짧게 설정하면
    - 서버 IP 변경시 빠르게 반영 가능
    - 캐시 만료가 잦아져 매번 Root 서버부터 쿼리를 다시 시작해야 하므로 **Resolver와 Name Server에 부하**를 줄 수 있음

## REST API

> HTTP를 기반으로 하는 **분산 하이퍼미디어 시스템을 위한 아키텍처 스타일**
> 

### 6가지 아키텍처 제약 조건

- **Uniform Interface**: HTTP 표준 메서드와 URI를 사용
- **Client-Server**: 클라이언트와 서버 분리
- **Stateless**: 서버는 클라이언트 상태를 저장하지 않음
- **Cacheable**: 응답은 캐시 가능 여부를 명시
- **Layered System**: 클라이언트는 서버 구조를 알 필요 없음
- **Code-On-Demand**: 서버가 클라이언트에 실행 가능한 코드를 전달 (선택 사항)

### 멱등성

- 한 번 요청하든 여러 번 요청하든 **서버의 상태가 동일하게 유지**되는 성질
- **멱등한 메서드**: GET, PUT, DELETE
- **멱등하지 않은 메서드**: POST, PATCH

### HATEOAS는 왜 중요한가요?

> Hypermedia As The Engine Of Application State
> 
- 클라이언트가 API 응답을 받았을 때, 해당 자원과 관련된 **다음 가능한 행동을 하이퍼링크 형태로 응답 본문에 포함**하는 원칙
- **진정한 RESTful의 기준:** Roy Fielding(REST 창시자)은 HATEOAS를 만족해야만 진정한 Level 3 RESTful API(Richardson Maturity Model 기준)로 간주함
- **API의 자가 문서화 (Self-Discovery):** 클라이언트는 서버에서 받은 링크만 따라가면 되기 때문에, API 명세서에 의존하지 않고도 애플리케이션 상태 전이(State Transition)를 진행 가능

| **HATEOAS 미적용 (일반적인 API)** | **HATEOAS 적용 (진정한 RESTful)** |
| --- | --- |
| **요청:** `GET /orders/123` | **요청:** `GET /orders/123` |
| **응답 (JSON):** `{ "id": 123, "status": "Pending" }` | **응답 (JSON):** `{ "id": 123, "status": "Pending", "links": [ { "rel": "cancel", "href": "/orders/123/cancel", "method": "POST" } ] }` |
| **결과:** 클라이언트는 취소하려면 URL을 **직접 구성**해야 합니다. | **결과:** 클라이언트는 응답의 `cancel` 링크를 따라 **POST 요청**만 보내면 됩니다. |
- 높은 수준의 유연성을 제공하지만, **클라이언트의 구현 복잡도**를 높임
- REST의 다른 제약 조건(Stateless, URI 기반 자원)만 따르는 **'REST-like'** 형태로 구현하고, HATEOAS는 생략하는 경우가 많음

## CORS (Cross-Origin Resource Sharing)

### SOP (동일 출처 정책)

- 보안을 위해, **프로토콜(Protocol), 도메인(Domain), 포트(Port)** 세 가지가 모두 같아야만 리소스에 자유롭게 접근할 수 있도록 제한하는 정책
- **CORS의 역할:** SOP에 의해 막힌 **브라우저 환경**의 HTTP 요청을 예외적으로 허용해 주는 기술
    
    (서버끼리의 통신은 SOP 적용 대상이 아님)
    
- 출처의 판단 기준
    
    `https://api.example.com:8080` 
    
    - **프로토콜:** `https`
    - **도메인:** `api.example.com`
    - **포트:** `8080`

### Simple Request와 Preflight Request

| **유형** | **Simple Request (단순 요청)** | **Preflight Request (프리플라이트 요청)** |
| --- | --- | --- |
| **조건** | 1. 메서드: GET, POST, HEAD만 사용 가능 
2. 커스텀 헤더 사용 불가 
3. `Content-Type`: `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 중 하나만 가능 | Simple Request 조건을 만족하지 않는 모든 요청 (예: PUT, DELETE, 커스텀 헤더 사용, JSON 전송) |
| **동작** | 한 번의 요청으로 데이터 전송 | **2단계 동작:** 
1. **OPTIONS** 메서드로 본 요청 전 서버에 권한을 먼저 요청함. 
2. 서버가 허용하면, **본 요청** (GET/POST/PUT 등)을 전송함. |
| **Headers** | `Origin` 헤더만 전송 | `Access-Control-Request-Method`, `Access-Control-Request-Headers` 등 추가 전송 |

### 서버의 응답 헤더

서버는 클라이언트의 요청을 허용하기 위해 반드시 다음 헤더를 포함하여 응답해야 함

- `Access-Control-Allow-Origin: <origin>`
    - 클라이언트의 출처를 정확히 명시 (`https://client.com`)하거나, 모두 허용
- `Access-Control-Allow-Methods: GET, POST, PUT, DELETE`
    - 허용할 HTTP 메서드를 명시(Preflight 응답 시 필수)
- `Access-Control-Allow-Headers: Content-Type, Authorization`
    - 클라이언트가 요청에서 사용한 커스텀 헤더 목록을 명시(Preflight 응답 시 필수)
