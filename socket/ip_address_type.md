## IP 주소 타입

`"net"` 패키지는 Go 네트워크 프로그래밍에서 사용되는 다양한 타입, 함수 및 메서드를 제공합니다. `IP` 타입은 바이트의 슬라이스로 정의됩니다.

```go
type IP []byte
```

IP 타입의 변수를 조작하기 위한 여러 함수들이 존재하지만, 실제로는 그 중 일부만 사용합니다.
예를 들어, ParseIP(String) 함수는 점으로 구분된 IPv4 주소 또는 콜론으로 구분된 IPv6 주소를 인자로 사용하며, IP 타입이 가진 String 메서드는 문자열을 반환합니다.
참고로 반환되는 문자열이 처음에 전달한 문자열과 다를 수도 있습니다. 가령 `0:0:0:0:0:0:0:1` 형태의 문자열은 `::1`가 됩니다.

다음은 예제 프로그램입니다.

```go
/* IP
 */

package main

import (
	"net"
	"os"
	"fmt"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: %s ip-addr\n", os.Args[0])
		os.Exit(1)
	}
	name := os.Args[1]

	addr := net.ParseIP(name)
	if addr == nil {
		fmt.Println("Invalid address")
	} else {
		fmt.Println("The address is ", addr.String())
	}
	os.Exit(0)
}
```

이 프로그램이 `IP` 실행파일로 컴파일되면, 다음과 같이 실행할 수 있습니다.

    IP 127.0.0.1

결과는 다음과 같습니다.

    The address is 127.0.0.1

혹은 다음과 같이 실행하면

    IP 0:0:0:0:0:0:0:1

결과는 다음과 같습니다.

    The address is ::1


### IPMask 타입

마스킹 연산을 다루기 위한 IPMask라는 타입이 있습니다.

```go
type IPMask []byte
```

4 바이트 IPv4 주소로부터 마스크를 생성하는 함수가 있습니다.

```go
func IPv4Mask(a, b, c, d byte) IPMask
```

또는 디폴트 마스크를 반환하는 `IP` 타입의 메서드가 있습니다.

```go    
func (ip IP) DefaultMask() IPMask
```

참고로 마스크의 형태는 16진수의 숫자이며, `255.255.0.0`의 마스크는 `ffff0000`입니다.

그 다음 IP 타입의 메서드를 사용하면 IP 주소의 네트워크 주소를 찾을 수 있습니다.

```go
func (ip IP) Mask(mask IPMask) IP
```

이를 사용한 예제 프로그램입니다.

```go
/* Mask
 */

package main

import (
	"fmt"
	"net"
	"os"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: %s dotted-ip-addr\n", os.Args[0])
		os.Exit(1)
	}
	dotAddr := os.Args[1]

	addr := net.ParseIP(dotAddr)
	if addr == nil {
		fmt.Println("Invalid address")
		os.Exit(1)
	}
	mask := addr.DefaultMask()
	network := addr.Mask(mask)
	ones, bits := mask.Size()
	fmt.Println("Address is", addr.String(),
		" Default mask length is", bits,
		"Leading ones count is", ones,
		"Mask is (hex)", mask.String(),
		" Network is", network.String())
	os.Exit(0)
}
```

`Mask`로 컴파일되면, 다음을 실행할 수 있습니다.

    Mask 127.0.0.1
결과값은 다음과 같습니다.

    Address is 127.0.0.1  Default mask length is 32 Leading ones count is 8 Mask is (hex) ff000000  Network is 127.0.0.0
​    

### IPAddr 타입

net 패키지의 많은 함수와 메서드가 `IPAddr`에 대한 포인터를 반환합니다. 이는 IP를 가진 간단한 구조체입니다.

```go    
type IPAddr {
    IP IP
}
```

이 타입의 주용도는 IP 호스트명에 대한 DNS 조회입니다.

```go
func ResolveIPAddr(net, addr string) (*IPAddr, os.Error)
```

`net`은 `"ip"`, `"ip4"` 또는 `"ip6"`중 하나입니다. 다음은 예제 프로그램입니다.

```go

/* ResolveIP
 */

package main

import (
	"net"
	"os"
	"fmt"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: %s hostname\n", os.Args[0])
		fmt.Println("Usage: ", os.Args[0], "hostname")
		os.Exit(1)
	}
	name := os.Args[1]

	addr, err := net.ResolveIPAddr("ip", name)
	if err != nil {
		fmt.Println("Resolution error", err.Error())
		os.Exit(1)
	}
	fmt.Println("Resolved address is ", addr.String())
	os.Exit(0)
}
```

`ResolveIP www.google.com`을 실행하면 다음과 같은 결과값이 출력됩니다.

    Resolved address is  66.102.11.104
​    

### 호스트 조회

`ResolveIPAddr` 함수는 호스트명으로 DNS를 조회하고 IP 주소를 반환합니다.
그러나 호스트는 일반적으로 여러개의 네트워크 인터페이스 카드에서 복수의 IP를 가질 수 있습니다. 또한 별칭으로 동작하는 여러개의 호스트명을 가질 수도 있습니다.

```go
func LookupHost(name string) (addrs []string, err os.Error)
```

이 주소들중 하나는 "표준 (canonical)" 호스트명으로 라벨링됩니다. 표준 호스트명을 찾기 위해선 다음 함수를 사용할 수 있습니다.

```go
func LookupCNAME(name string) (cname string, err os.Error)
```

다음은 예제 프로그램입니다.

```go
/* LookupHost
 */

package main

import (
	"net"
	"os"
	"fmt"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: %s hostname\n", os.Args[0])
		os.Exit(1)
	}
	name := os.Args[1]

	addrs, err := net.LookupHost(name)
	if err != nil {
		fmt.Println("Error: ", err.Error())
		os.Exit(2)
	}

	for _, s := range addrs {
		fmt.Println(s)
	}
	os.Exit(0)
}
```

참고로 이 함수는 `IPAddress`가 아닌 문자열 리스트를 반환합니다.