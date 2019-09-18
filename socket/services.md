## 서비스

서비스는 호스트 머신에서 실행됩니다. 서비스는 보통 오래 실행되며 요청을 기다리고 응답을 합니다. 다양한 종류의 서비스가 있으며 이들은 다양한 방식으로 클라이언트에게 서비스를 제공합니다. 인터넷은 TCP와 UDP 이 두 통신 프로토콜 위에서 동작하는 서비스들에 기반하고 있지만 SCTP와 같은 다른 통신 프로토콜들도 주목을 받고 있습니다. 피어 투 피어 (Peer to Peer), 원격 프로시저 호출, 에이전트 통신 및 여러 서비스들이 TCP와 UDP를 기반으로 설계되었습니다.

### 포트

IP 주소는 호스트의 위치를 찾습니다. 그러나 각 컴퓨터에는 여러개의 서비스가 있을수도 있기 때문에 이들을 구분하기 위한 방법이 필요합니다. TCP, UDP, SCTP 등에서 사용되는 방법은 포트 번호를 사용하는 것입니다. 포트 번호는 1과 65,535 사이의 부호 없는 정수이며 각 서비스는 이 포트 번호중 하나 이상과 연결됩니다.

포트들 중에는 "표준" 포트들이 존재합니다. 텔넷은 일반적으로 TCP 프로토콜과 함께 23 포트를 사용합니다. DNS는 TCP 및 UDP와 함께 53 포트를 사용합니다. FTP는 21 포트와 20 포트를 사용하는데 하나는 명령용이고 다른 하나는 데이터 전송용입니다. HTTP는 일반적으로 80 포트를 사용하지만 종종 8000, 8080 및 8088 포트를 사용하기도 하며 모두 다 TCP와 함께 사용됩니다. X 윈도우 시스템은 보통 6000에서 6007 사이의 포트를 TCP와 UDP와 함께 사용합니다.

유닉스 시스템에서 파일에 자주 사용되는 포트들은 `/etc/services`에 나열되어 있습니다. Go에는 이 파일에서 정보를 가져오는 함수가 있습니다.

```go
func LookupPort(network, service string) (port int, err os.Error)
```

network 인자는 `"tcp"`나 `"udp"`와 같은 문자열이며 service 인자는 `"telnet"`이나 (DNS를 위한) `"domain"`등의 문자열입니다.

다음은 예제 프로그램입니다.

```go
/* LookupPort
 */

package main

import (
	"net"
	"os"
	"fmt"
)

func main() {
	if len(os.Args) != 3 {
		fmt.Fprintf(os.Stderr,
			"Usage: %s network-type service\n",
			os.Args[0])
		os.Exit(1)
	}
	networkType := os.Args[1]
	service := os.Args[2]

	port, err := net.LookupPort(networkType, service)
	if err != nil {
		fmt.Println("Error: ", err.Error())
		os.Exit(2)
	}

	fmt.Println("Service port ", port)
	os.Exit(0)
}
```

예로, `LookupPort tcp telnet`을 실행하면 `Service port: 23`이 출력됩니다.


### TCPAddr 타입

`type TCPAddr`은 IP와 포트 정보를 가진 구조체입니다.

```go
type TCPAddr struct {
    IP   IP
    Port int
}
```

`TCPAddr`을 생성하는 함수로는 `ResolveTCPAddr`이 있습니다.

```go
func ResolveTCPAddr(net, addr string) (*TCPAddr, os.Error)
```

`net`은 `"tcp"`, `"tcp4"` 또는 `"tcp6"`중 하나이며 `addr`은 호스트명과 IP주소를 조합한 문자열인데 포트 번호는 `"www.google.com:80"` 또는 `"127.0.0.1:22"`와 같이 `":"` 뒤에 위치합니다.
만약 주소가 이미 콜론이 포함되어 있는 IPv6 주소라면 호스트명은 `"[::1]:23"`과 같이 괄호로 감싸야합니다.
호스트 주소가 0인 서버에서 종종 사용되는 또다른 특수한 경우가 있는데, 이 경우, TCP 주소는 단지 포트명으로만 표현되며 예로  HTTP 서버는 `":80"`이 됩니다.