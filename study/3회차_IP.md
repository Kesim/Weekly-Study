### 1. IP란?
IP(Internet Protocol)란 인터넷에 연결되어 있는 장비들(컴퓨터, 서버 장비, 스마트폰, 태블릿 등)을   
식별할 수 있도록 **각각의 장비에게 부여되는 고유 주소**이다.  
하지만 오해하면 안되는 것이, 각각의 장비에게 부여되는 주소라고 해서 장비 자체의 식별번호가 아니라   
**장비가 연결된 네트워크 끝단의 주소**라는 점이다.  
그렇기 때문에 장비마다 IP주소는 고유한 것이 아니며, 언제든지 바뀔 수도 있는 것이다.  
 
IP는 크게 공인ip, 사설ip, 고정ip, 유동ip로 나누어진다.  
공인 ip는 고정 ip가 될 수도, 유동 ip가 될 수도 있으며, 사설 ip 또한 고정 ip가 될 수도, 유동ip가 될 수도 있다.  
 
이제 IP가 왜 이렇게 다양한 종류로 나뉘어졌는지를 알아보자.

### 2. IPv4 (IP Version4)
IP가 여러 종류로 나누어진 이유는 IPv4와 관련이 있다.
 
IPv4는 IP의 버전이라고도 할 수 있으며, 우리가 흔하게 보는 IP주소의 구조를 가진다.  
IP로써는 처음 대중화된 규약이고, 아직 이 IPv4가 널리 사용되고 있다.  
 
우리가 자주 봤던 xxx.xxx.xxx.xxx 의 형식을 가지고 있다.  
4개의 0~255 사이의 숫자로 이루어져 있고, 각 숫자는 점(.)으로 구분되며, 32비트이다.  
256(0~255의 개수)의 4승은 약 43억 정도가 된다.  
즉, 전 세계적으로 우리가 사용할 수 있는 IP의 개수는 43억개 정도라고 생각하면 된다.  
그런데 여기서 문제가 발생한다.  
현재와 같은 디지털 시대에서 IP주소가 43억개로 충분할까?  
 
전 세계 인구가 80억명을 돌파했고, 한 사람당 인터넷에 연결하는 장비를 하나씩만 가지고 있다고 쳐도  
43억개의 IP는 턱 없이 부족하게 된다.  
 
이 문제를 해결하기 위해 IP를 공인IP와 사설IP로 나누게 된다.  

### 3. 공인IP와 사설IP
공인IP와 사설IP를 설명하는 예시 중 가장 많이 사용되는 예시가 있다.  
서울시 개발구 코딩동 자바아파트 101동 101호 라는 주소가 있다고 해보자.  
여기서 서울시 개발구 코딩동 자바아파트는 공인IP, 101동 101호는 사설IP라고 볼 수 있다.  
즉, 서울시 개발구 코딩동 자바아파트라는 공통되고 고유한 주소 안에서 다시 101동 101호, 101동 102호 ... 처럼  
각각의 겹치지 않는 개별 주소를 부여하는 방식이다.

공유기를 사용하는 회사, 학교, 집, 카페 등에서는 공유기가 가지고 있는 하나의 공인 IP를 통해 각 기기마다 사설IP를 부여하는 방식으로 인터넷을 사용한다.
때문에, 공인IP는 전 세계에서 유일한 딱 하나의 IP주소이고, 사설IP는 공인IP가 다르다는 전제하에 같은 주소를 얼마든지 가질 수 있다.

이렇게 공인IP안에 사설IP를 부여하는 방식으로 어느정도의 IP부족 문제를 해결했다.
 
공인 IP는 외부에 공개되어 있기 때문에 인터넷에 연결되는 다른 장비로부터 접근이 가능하다. 따라서 공인IP를 사용하는 경우에는 방화벽 등의 보안 프로그램을 설치할 필요가있다.  
그러나 사설IP는 특정 네트워크에서 사용하는 주소이기 때문에 외부에서 접근하지 못한다. '101동 101호' 이런 주소를 주고 찾아오라고 할 순 없지 않은가?  
 
그렇다면 개인의 PC는 보통 사설IP로 할당되기 때문에 외부에서 접속을 하지못해 서버구축을 할 수 없는것일까?  
이럴 땐 포트포워딩을 사용하면 된다. 포트포워딩에 대해 간단하게 알아보자!  
 

#### 3-1. `포트포워딩`

포트포워딩이란, 공인IP에 포트들을 개방해서, 외부에서도 사설IP에 접근할 수 있도록 하는 기술을 말한다.  
일반적으로 라우터나 방화벽에서 사용되며, 외부에서 특정 서비스에 접속할 때 유용하게 사용된다.  
즉, 포트포워딩을 통해 내부 네트워크에 있는 서버나 애플리케이션에 외부에서 접속할 수 있다.  
예를 들어, 웹 서버를 운영하는 경우 80번 포트로 들어오는 요청을 내부의 웹 서버에 전달하며 웹 사이트에 접속할 수 있도록 한다.  
 
이렇게 공유기가 하나의 공인IP를 가상의 사설IP로 나누는 기술을 NAT(Network Address Translation)라고 한다.  
NAT의 장점은 사설IP의 사용으로 내부망을 보호하는 역할에 있다고 할 수 있다.  
또한 이 기술로 43억개뿐인 IP주소를 더 많이 사용할 수 있도록 IPv4의 한계점을 보완했다.  

### 4. 유동IP(Dynamic IP)와 고정IP(Static IP)
유동IP와 고정IP도 활용할 수 있는 IP에 제한이 있기 때문에 나오게 되었다.  
43억개의 IP중에 한국에 할당된 IP는 약 1억개로 알려져있다.   
이 1억개의 IP를 ISP(인터넷 서비스 제공업체 : sk, kt, lg)업체들이 인터넷 사용주체들에게 나누어주게 된다.  
서버의 경우 IP가 바뀌면 곤란하기 때문에 하나의 고정된 고정IP를 할당하고, 그 외 일반가정이나 기기들은 주기적으로 바뀌는 유동IP를 할당하게 된다.
 
유동IP는 주기적으로 IP들을 회수해서 인터넷을 사용중인 곳에만 나눠준다.  
즉, 놀고 있는 기기들에게 IP주소를 할당해줘봤자 사용하지 않기 때문에, 주기적으로 회수해서 지금 사용할 기기들에게만 할당하여  
IP가 부족한 문제를 해결할 수 있는 것이다.  
유동IP는 고정IP 보다 가격도 저렴하고(보편적으로), 내 IP가 이따금씩 바뀌기 때문에 해킹으로부터도 (고정IP보다는)안전하고 볼 수 있다.
 
고정IP는 보통 서버들에게 할당한다.  
네이버의 IP가 주기적으로 변경되는 유동IP라고 가정을 해보자.  
이 경우 이전에 접속하던 IP주소로 접속하려고 하면 오류가 발생한다.  
따라서 '서버'역할을 하는 컴퓨터는 클라이언트가 항상 일정한 주소로 접근하도록 반드시 고정IP가 필요하다.  
 
그렇다면 유동IP인 경우에는 서버 구축이 불가능할까?  
사설IP를 사용하는 개인개발자가 포트포워딩을 통해 외부에서의 접속을 가능하게 했다고생각해보자.  
기껏 포트포워딩을 통해 외부 접속을 열어놨는데 유동IP로 인해 IP주소가 바뀌게 되면 소용 없는거 아닌가?  
이럴 땐 DDNS(Dymanic DNS)방식을 사용할 수 있다. DDNS에 대해 간단하게 알아보자.  
 

#### 4-1. `DDNS`
해당 방식으로 서버를 구축하려고 할 땐 '사용자가 도메인주소로 해당 서버에 접근'해야 한다는 전제가 존재한다.  
DNS가 IP를 도메인이랑 연결해주는 역할을 한다면(ex) www.naver.com → 223.130.195.200),  
DDNS는 수시로 바뀌는 유동IP를 감지해서 고정된 도메인에 연결해주는 것을 말한다.  
공유기 페이지에 들어가서 이 옵션을 세팅해두면 도메인에 내 컴퓨터의 유동IP를 연결할 수 있다.  
따라서 포트포워딩과 DDNS를 사용하면 개인적으로 운영할 소규모서버 같은 경우는 내 컴퓨터로도 구축할 수 있다.  
 
하지만 이런 방식이 있다고 해도 서버를 구축할 땐 공인고정IP로 서버를 구축하는 것이 일반적이다.

### 5. IPv6
이 내용을 IPv4와 함께 다루려다 아직까지는 IPv4를 더 많이 사용하고 있고, IP가 나뉘어진 이유도 IPv4의 한계 때문이었기에  
제일 마지막에 다루는 것이 맞는 것 같아 지금 설명하고자한다.  
IPv4체계로는 곧 IP가 머지않아 바닥날 걸 예상해서 IPv6가 도입되었다.  
이 버전은 16진수 4자리가 8개 이어진 형태며, 거의 무한에 가까운 IP를 만들 수 있다고 보면 된다. (2^128개)  
현재 IPv4와 혼용돼서 사용되고 있으며, 점점 그 숫자도 늘어가고 있는 추세이다.  
 
스마트폰에서 wifi가 연결되지 않은 상태로 해당 기기의 IP주소를 알아보면, IPv4와 IPv6가 같이 사용되고 있는 것을 알 수 있다.  

### 6.요약
`공인IP` : 세계에서 단 하나만 존재하는 유일한 IP주소  
`사설IP` : 공인IP 안에서 할당되는 가상의 IP주소  
`고정IP` : 해당 장비의 IP주소를 고정적으로 사용  
`유동IP` : 수시로 변하는 IP주소   
`IPv4` : 32비트로 구성된 일반적인 의미의 IP, 해당 버전의 IP 43억개로는 전 세계의 모든 컴퓨터를 처리할 수 없음  
`IPv6` : 128비트로 구성된 IP, IPv4를 대체하기위해 나왔으며, 거의 무한한 수의 고유한 주소를 생성할 수 있다.  
