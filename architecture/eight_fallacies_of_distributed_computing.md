## 분산 컴퓨팅의 8가지 오류

썬 마이크로시스템즈는 초창기부터 분산 시스템을 많이 다뤘던 회사이며 "네트워크는 컴퓨터다"라는 말을 남겼습니다. 수년간의 경험을 토대로 썬의 많은 과학자들은 분산 시스템에서 일반적으로 가정되는 다음과 같은 오류 목록을 제시했습니다.

* 네트워크는 안정적이다. (신뢰 가능한 네트워크)
* 레이턴시가 없다.
* 대역폭이 무한정하다.
* 네트워크는 안전하다.
* 망구성 (토폴로지) 은 변경되지 않는다.
* 관리자는 한 명이다.
* 전송 비용이 없다.
* 네트워크가 균일하다.

이 중 대부분은 네트워크 프로그래밍에 직접적인 영향을 줍니다. 예를 들어, 대부분의 원격 프로시저 호출 시스템의 설계는 네트워크가 안정적이어서 원격 프로시저 호출이 로컬 호출과 동일한 방식으로 작동한다는 전제에 기반합니다. 레이턴시가 없단는 것과 대역폭이 무한하다는 오류는 RPC 호출의 지속 시간이 로컬 호출과 동일하다는 가정을 이끌어 내지만, 실제로는 더 느리게 동작합니다.

이러한 오류들에 대한 인식은 자바의 RMI (Remote Method Invocation, 원격 메서드 실행) 모델의 모든 RPC 호출이 잠재적으로 `RemoteException` 예외를 발생시킬 것을 요구하도록 이끌었습니다. 이로인해 프로그래머는 적어도 네트워크가 오류를 발생시킬 수 있음을 인식하게 되었으며 이는 네트워크 호출이 로컬 호출 속도와 같을 수 없다는걸 상기시켜주었습니다.