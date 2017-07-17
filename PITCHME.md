
# IOCP
---
#### Overlapped IO
* 지금까지 배웠던 소켓 통신은 동기방식의 소켓. 즉 IO를 요청하면 완료 될 때까지 기다려야했다.
* 하지만 Overlapped IO 는 입출력작업을 요청하고 다른 일을 처리할 수 있다.
* 다른일을 처리하다 어떤방식으로(이벤트를 이용하거나 특정루틴을 실행하게하거나 IOCP를 이용해서) IO 가 완료되었다는 것을 알게 되면 IO의 결과를 처리 하는 방식이다.
---
#### 관련함수(0)
```C++
int WSASend(
SOCKET s,//해당 소켓
LPWSABUF lpBuffers,//WSABUF구조체의 포인터
DWORD dwBufferCount,//위 포인터에 딸린 구조체의 수
LPDWORD lpNumberOfBytesSent,//보낸 바이트의 수를 받는 부분
DWORD dwFlags,//옵션을 설정하는 플래그
LPWSAOVERLAPPED lpOverlapped,//오버랩드 구조체의 포인터
LPWSAOVERLAPPED_COMPLETION_ROUTINE lpCompletionRoutine//콜백함수
);
```
+++
* lpNumberOfBytesSent: Msdn에서 이상한 내용을 찾았다.
* Use NULL for this parameter if the lpOverlapped parameter is not NULL to avoid potentially erroneous results. This parameter can be NULL only if the lpOverlapped parameter is not NULL.
---
#### Overlapped IO 특징
* 다른 소켓 입출력과 다르게 사용자가 지정한 버퍼에 바로 데이터가 입력된다. 즉 커널이 관리하는 버퍼에서 어플리케이션이 사용하는 버퍼로 복사가 필요 없다.
+++
#### Overlapped IO 특징
* 하지만 메모리가 부족한 상황에서 커널이 사용하는 메모리 영역이 아니기 때문에 page교체가 일어날수 도 있다.
* 그래서 이렇게 입출력이 일어나는 부분을 page교체가 일어나지 않도록 lock을 건다.
* 하지만 이때 무조건 메모리의 page단위로 lock이 걸리기때문에 문제가 될 수 있다.
+++
#### Overlapped IO 특징
* IO먼저 요청한 입출력이라고 해서 먼저 완료 되지 않는다.
* 즉 빨리 완료 되는 것부터 처리 할수 있으니 효율적이다.
---
#### IOCP의 목적
* Overlapped모델도 결국 그 처리를 맡은 스레드가 대기 상태에 들어가야지 처리를 완료할수 있다.
* 또 Overlapped모델은 한개의 스레드가 입출력을 담당한다.
* 효율적으로 입출력을 하려면 입출력을 담당하는 여러개의 스레드가 있어야 한다.
* 관리 해야 하는데 무조건 스레드가 많다고 좋은것은 아니다. 컨텍스트 교환 비용 때문이다.
+++
#### IOCP의 목적
* 즉 컨텍스트 교환 비용과 cpu의 이용율을 고려한 최대의 스레드개수를 정해놓고 스레드를 관리하면 cpu를 최대한 효율적으로 사용할수 있다.
* 이것을 잘관리 해주는 것이 IOCP이다.
---
#### 관련함수(1)
```c++
HANDLE CreateIoCompletionPort (
HANDLE FileHandle, // IOCP에 연결할 소켓의 핸들
HANDLE ExistingCompletionPort, // 생성된 IOCP 핸들을 넣는다
ULONG_PTR CompletionKey, // 소켓을 특정할 수 있는 키
DWORD NumberOfConcurrentThreads // IOCP를 생성할때는 최대 실행 스레드수
);
```
+++

* FileHandle:INVALID_HANDLE_VALUE 값을 넣으면 IOCP생성
* ExistingCompletionPort: 생성시에는 null
* CompletionKey: 생성시에는 0
* NumberOfConcurrentThreads: 0을 넣으면 cpu 개수 만큼 생성, 연결 할 때도 0
---
#### IOCP의 구조(1)
### Device List
![Device List](/image/DeviceList.png)
* 키를 이용해서 어떤 디바이스의(여기서는 소켓) 입출력 이완료 되었는지를 알수 있도록 한 자료구조.
* 보통 CompletionKey는 소켓의 FD를 이용하거나 사용자가 정의한 구조체의 포인터를 이용한다.
---
#### IOCP의 구조(2)
### IO Completion Queue
![IO Completion Queue](/image/IO Completion Queue.PNG)
+++
* 입출력이 완료된 IO의 결과의 정보를 저장하는 큐
* 순서대로 전송길이, 관련소켓, Overlapped 구조체, 에러다.
* 큐에서 결과를 받아오는 것 뿐만 아니라 보내는 것도 가능하다.
* PostQueuedCompletionStatus 함수를 이용하면 된다.
---
#### 관련함수(2)
```c++
BOOL GetQueuedCompletionStatus(
HANDLE CompletionPort, // 관심있는 IOCP포트의 핸들
LPDWORD lpNumberOfBytes, // 전송된 바이트수
PULONG_PTR lpCompletionKey, // 어떤 소켓에서 전송된것인지
LPOVERLAPPED *lpOverlapped, // 오버렙드 구조체 포인터
DWORD dwMilliseconds // 대기시간
);
```
+++
* lpCompletionKey: 포인터 이므로 확장해서 이용 가능하다.
* lpOverlapped: 포인터 이므로 확장해서 이용 가능하다.
* dwMilliseconds:  보통 무한대로 설정한다.
* PostQueuedCompletionStatus 함수는 마지막 대기 시간을 제외한 인자로 되어있다.
---
### 확장 Overlapped 구조체
```c++
struct SOCKETINFO
{
	OVERLAPPED overlapped;
	SOCKET sock;
	char buf[BUFSIZE+1];
	int recvbytes;
	int sendbytes;
	WSABUF wsabuf;
};

```
+++
* 대부분 이렇게 확장해서 사용한다.
* 보통 Overlapped 구조체를 사용해서 관련 정보를 한꺼번에 관리 한다.
* 구조체를 만들어서 Overlapped 구조체를 맨 앞에 두면 포인터 캐스팅으로 내가 정의한 구조체에 접근 할수 있다.
---
#### IOCP의 구조(3)
### Waiting Thread Queue(LIFO)
![Waiting Thread Queue(LIFO)](/image/Waiting Thread Queue.PNG)
+++
* GetQueuedCompletionStatus 함수를 호출한 스레드가 들어가게 되는 큐
* 큐라고 불리지만 실제로는 레지스터나 캐쉬, 메모리에 남아있는 정보를 계속 사용하는게 비용이 덜들기 때문에 가장 최근에 사용한 스레드를 계속 사용하기 위해서 LIFO로 동작한다.
* WTQ에서 스레드는 기다리고 있다가 IO가 완료되면 GetQueuedCompletionStatus함수의 매개변수로 정보를 받아온다.
+++

* 하지만 아까 IOCP는 동시에 동작하는 스레드의 최대수를 제한한다고 했다.
* 즉 처음에 설정해준 최대 스레드 개수를 넘지 않는 경우에만 대기 상태에서 빠져 나올수 있다.
---
#### IOCP의 구조(5)
### Released Thread List
![Released Thread List](/image/Released Thread List.PNG)
* 현재 작동중인 스레드의 리스트
* 최대 스레드 수 이상은 들어갈수 없지만 어쩔수 없을때는 최대수를 넘을수 있다.
* 동작을 완료하거나 스레드가 스스로 대기상태에 돌입하면 제거된다.
---
#### IOCP의 구조(6)
### Paused Thread List
![Paused Thread List](/image/Paused Thread List.PNG)
* 현재 작동되다가 스스로 대기 상태에 빠진 스레드의 리스트
* 대기 상태에서 다시 동작상태로 돌아가면 빠져나가게 된다.

+++
#### 최대 스레드 수의 모호성

* 최대 스레드수는 가능한한 지켜진다.
* 만약 최대 스레드 개수 만큼 동작하다가 하나의 스레드가  PTL로 들어가면 WTQ에서 하나의 스레드가 깨어나게 되고 또 최대스레드 수만큼 동작하게 된다.
* 이때 만약 PTL의 스레드가 동작하게 되면 다시 RTL로 돌아가게 되는데 이때 최대 개수를 초과 하게 된다.

---
#### 예제코드
+++?gist=6ae3c1a35ed2bc9e1d31fefb024a36ac
---
#### 의문점들

* page locking에 의해서 문제가 생기면 어떻게 해결하는가
* Accept 함수는 그대로인데 ...
