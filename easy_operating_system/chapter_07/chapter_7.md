# Ch.07 물리 메모리 관리

## 01. 메모리 이해하기
- 메모리 구조는 1바이트 크기로 나뉜다.
- 1바이트로 나뉜 각 영역은 주소로 구분되는데 보통 0번지부터 시작한다.
- 메모리는 매우 빠른 장치 같지만, CPU 입장에서 보면 느린 장치다. CPU 안에 있는 레지스터에 접근하는 속도보다 메모리에 접근하는 속도가 몇 배 이상 느리다.

## 02. 메모리 관리의 이중성
- 복잡한 메모리 관리는 메모리 관리 시스템 (MMS: Memory Management System)이 담당
- 프로세스 입장에서는 메모리를 독차지하려 하고, 메모리 관리자 입장에서는 관리를 효율적으로 하고 싶어 하는데 이를 메모리 관리의 이중성이라고 한다.

## 03. 소스코드의 번역과 실행
- 프로그램은 메모리 관리와 밀접한 관련이 있음.
- 소스코드의 실행 방식에는 인터프리터 방식과 컴파일러 방식이 있음


###  03-01. 컴파일러 방식
1. 오류 발견
  - 컴파일러는 오류를 찾기 위해 심벌 테이블을 사용한다.
  - 심벌 테이블은 변수 선언부에 명시한 각 변수의 이름과 종류를 모아놓은 테이블
  - 선언하지 않은 변수를 사용하지 않았는지, 변수에 다른 종류의 데이터를 저장하지 않았는지 알 수 있다.
2. 소스코드 최적화
   - 컴파일러는 실행 전에 소스코드를 점검하여 오류를 수정하고 필요 없는 부분을 정리하여 최적화된 실행 파일을 만든다.

### 03-02. 인터프리터 방식
- 인터프리터는 한 행씩 위에서부터 아래로 실행되기 때문에 같은 일을 반복하는 경우나 필요 없는 변수를 확인할 수 없다.

### 04. 메모리 관리 작업
- 메모리 관리 작업은 크게 가져오기(fetch), 배치 (placement), 재배치 (replacement)로 구분된다.

1. 가져오기 (fetch)
- 실행할 프로세스와 데이터를 메모리로 가져오는 작업
- 어떤 상황에서는 데이터의 일부만 가져와 실행하기도 한다. (e.g 용량이 큰 동영상을 실행해야 하는데 메모리가 충분하지 않다면 동영상 플레이어를 먼저 가져와 실행하고 동영상 데이터는 필요할 때마다 수시로 가져와 실행)

2. 배치 (placement)
- 가져온 프로세스와 데이터를 메모리의 어떤 부분에 올려놓을지 결정하는 작업
- 배치 작업 전에 메모리를 어떤 크기로 자를 것인지가 매우 중요하다. (같은 크기로 자르느냐, 실행되는 프로세스의 크기에 맞게 자르느냐에 따라)
- 메모리를 같은 크기로 자르는 것을 페이징, 프로세스의 크기에 맞게 자라는 것을 세그먼테이션이라고 한다

3. 재배치 (replacement)
- 꽉 찬 메모리에 새로운 프로세스를 가져오기 위해 오래된 프로세스를 내보내는 작업을 재배치 작업이라고 한다
- 이때 앞으로 사용하지 않을 프로세스를 내보내면 시스템의 성능이 올라가지만 자주 사용할 프로세스를 내보내면 성능이 떨어진다. (9장에서 교체 알고리즘에 대해서 자세히 설명 예정)

## 05. 32bit CPU & 64bit CPU의 차이
- CPU를 나타낼 때의 bit는 CPU가 한 번에 다룰 수 있는 데이터의 최대 크기를 의미한다.
- CPU 내부 부품은 모두 이 비트를 기준으로 제작된다.
- 32bit CPU 내의 레지스터 크기는 전부 32bit이고, ALU도 32bit를 처리할 수 있도록 설계된다. 버스의 크기(대역폭)도 32bit이고 버스를 통해 한 번에 옮겨지는 데이터의 크기도 32bit다.
- 32bit CPU의 경우 메모리 주소를 지정하는 레지스터(MAR)의 크기도 32bit이다.
- 32bit든 64bit든 컴퓨터에는 메모리가 설치되고 각 메모리에는 주소 공간이 있따. 메모리의 주소 공간을 물리 주소 공간이고 반대로 사용자 입장에서 바라본 주소 공간을 논리 주소 공간이라고 한다.

## 06. 메모리 영역의 구분
- 운영체제와 사용자영역은 메모리 주소를 사용하는 영역이 구분되어 있다.
- 운영체제가 전체 메모리에 첫번째부터 마지막까지의 방향으로 사용한다면, 사용자 영역은 메모리의 끝에서부터 첫번째로 사용해나간다.
- 이렇게 사용하는 이유는 운영체제의 크기에 상관없이 사용자 영역의 시작점을 결정할 수 있다. 하지만 이는 곧 사용자 영역의 주소 관리의 복잡성으로 다가온다.
- 사용자 영역이 운영체제 영역을 침범하는 것을 막으려면 하드웨어의 도움이 필요 (CPU내에 있는 경계 레지스터가 담당)
- 경계 레지스터는 운영체제 영역과 사용자 영역 경계지점의 주소를 가진다.
- 메모리 관리자는 사용자가 작업을 요청할 때마다 경계 레지스터의 값을 벗어나는지 검사하고, 벗어나는 작업을 요청하는 프로세스는 종료시킨다.

## 07. 논리 주소와 물리 주소의 변환
- 각 프로세스 내부에서 사용하는 물리 메모리 주소는 논리 주소로 변환된다.
- 프로그램이 실행될 때 시작하는 물리 주소는 매번 바뀐다.
- 물리 주소에서의 논리주소로의 변환은 메모리 관리 유닛 (MMU: Memory Management Unit)이 담당한다.

## 08. 단일 프로그래밍 환경의 메모리 할당
- 과거에는 메모리가 값비쌌기 때문에 큰 메모리를 사용할 수 없었다.

### 08-01. 메모리 오버레이
- 메모리보다 큰 프로그램을 실행하기 위해 메모리 오버레이 기법을 사용 -> 전체 프로그램을 메모리에 가져오는 대신 적당한 크기로 잘라서 가져오는 기법
- 프로그램 전체를 메모리에 올려놓고 실행하는 것보다 느리지만 메모리가 실행될 프로그램의 크기보다 작을 때도 프로그램 실행 가능
- 가상 메모리 시스템의 기본이 되는 개념
- 프로그램은 개념적으로 한 덩어리지만 일부부분만으로도 실행할 수 있다.

### 08-02 스왑
- 메모리가 모자라서 쫓겨난 프로세스를 모아두는 저장장치의 특별한 저장공간 : 스왑 영역
- 스왑 영역에서 메모리로 데이터를 가져오는 작업 : 스왑인
- 메모리에서 스왑 영역으로 데이터를 내보내는 작업 : 스왑아웃
- 원래 저장 장치는 저장장치 관리자가 관리하지만, 스왑 영역은 메모리 관리자가 관리

## 09. 다중 프로그래밍 환경의 메모리 할당
- 한 번에 여러 프로세스가 실행되는 구조에서 메모리 문제가 발생
- 메모리 관리가 더욱 복잡 -> 프로세스드르이 크기가 달라 메모리를 어떻게 나누어 사용할 것인지가 가장 큰 문제

### 09-01. 가변 분할 방식
- 프로세스의 크기에 따라 메모리를 나눈다. (세그먼테이션 기법: segmentation)
- 한 프로세스가 메모리의 연속된 공간에 배치 -> 연속 메모리 할당
- 장점
  - 하나의 연속된 공간에 프로세스를 배치할 수 있음
- 단점
  - 메모리 관리가 복잡하다.
    - 메모리 배치 방식(memory placement), 조각 모음 (defragmentation) 이 필요
    - 메모리 배치 방식은 선처리, 조각 모음은 후처리에 해당
  - 단편화 (fragmentation) 발생
  - 남은 메모리 자리의 크기와 새로 배치되어야할 프로세스 크기가 상이한 경우도 생기기 때문에 메모리 통합 등의 부가적인 작업이 필요.

#### 09-01-01. 메모리 배치 방식
- 최초 배치
  - 단편화를 고려하지 않는 방식으로, 적재 가능한 공간을 순서대로 찾다가 첫 번째로 발경한 공간에 프로세스를 배치
  - 장점
    - 빈 공간을 찾아다닐 필요가 없다
- 최적 배치
  - 메모리의 빈 공간을 모두 확인한 후 크기가 가장 비슷한 곳에 프로세스를 배치
  - 장점
    - 딱 맞는 공간을 찾을 경우 단편화가 일어나지 않는다
  - 단점
    - 빈 공간을 모두 확인해야한다.
    - 딱 맞는 공간이 없을 경우 메모리 효율이 떨어진다 (12MB의 공간에 10MB의 프로세스가 배치될 경우 2MB의 메모리 공간이 버려짐)
- 최악 배치
  - 빈 공간을 모두 확인한 후 가장 큰 공간에 프로세스를 배치

#### 09-01-02. 조각 모음
- 특정 프로세스가 작업을 마치고 메모리에서 나가면 그 프로세스 사용하던 메모리의 크기만큼 단편화가 발생한다. 단편화를 해결하기 위해 이미 배치된 프로세스를 옆으로 옮겨 빈 공간들을 하나의 큰 덩어리로 만드는데 이를 조각 모음이라고 한다.
- 조각 모음을 하는 과정에서 프로세스의 이동과 프로세스의 정지 등의 작업이 필요하므로 메모리 관리가 복잡해진다.

### 09-02. 고정 분할 방식
- 프로세스의 크기에 상관없이 20KB의 같은 크기로 나눈다. (페이징 기법: paging)
- 큰 프로세스가 메모리에 올라오면 여러 조각으로 나뉘어 배치된다.
- 하나의 프로세스가 여러 개로 나뉘어 배치되기 때문에 비연속 메모리 할당이라고 한다.
- 현대 운영체제에서 메모리 관리는 기본적으로 고정 분할 방식을 사용
- 장점
  - 메모리 관리가 편하다
- 단점
  - 하나의 프로세스가 여러 곳으로 나뉠 수 있다는 점.
  - 그러나 메모리 오버레이, 스왑 영역을 이용하여 해결 가능하므로 가변 분할 방식보다 메모리 관리 측면에서 유리하다.
  - 내부 단편화가 발생 -> 일정한 메모리 크기보다 작은 프로세스가 배치될 경우 (메모리 크기 - 프로세스 크기) 만큼의 내부 조각 발생
  - 내부 단편화는 가변 분할 방식이 외부 단편화를 해결하기 위해 했던 것처럼 조각 모음을 하거나 남는 공간을 다른 프로세스에 배정할 수도 없다.

| 구분        | 가변 분할 방식           | 고정 분할 방식       |
|-----------|--------------------|----------------|
| 메모리 관리 기법 | 세그먼테이션             | 페이징            |
| 특징        | 연속 메모리 할당          | 비연속 메모리 할당     |
| 장점        | 프로세스를 한 덩어리로 관리 가능 | 메모리 관리가 편리함    |
| 단점        | 빈 공간의 관리가 어려움      | 프로세스가 분할되어 처리됨 |
| 단편화       | 외부 단편화             | 내부 단편화         |

### 09-03. 버디 시스템
- 가변 분할 방식의 단점인 외부 단편화를 완화하는 방법
- 가변 분할 방식과 고정 분할 방식의 중간 구조
- 프로세스 적재요구가 있을 때 메모리는 요구한 크기보다 크되 차이가 가장 작게나는 2^n 크기로 분할되어 할당된다.
- 같은 크기로 분할된 인접해 있는 공간을 Buddy라고 한다.
  
- Buddy 시스템 예제
```text
1. 크기가 100 KByte인 process A의 적재
2. 크기가 240 KByte인 process B의 적재
3. 크기가  64 KByte인 process C의 적재
4. 크기가 256 KByte인 process D의 적재
5. process B의 메모리 반납
6. process A의 메모리 반납
7. 크기가  75 KByte인 process E의 적재
8. process C의 메모리 반납
9. process E의 메모리 반납
10. process D의 메모리 반납
```

![image](https://user-images.githubusercontent.com/85930725/229536157-62aa13d3-ddd8-4cde-bdc1-3ca21cea1cf6.png)

버디 시스템 이미지 및 내용 출처 : https://github.com/qkraudghgh/coding-interview/blob/master/OS/images/budy-example.jpeg

- (a) 비어있는 1MB의 메모리
- (b) 크기가 100 KByte인 process A의 적재를 위해 2^n 크기로 나누어진 메모리
128 KB로 나누어진 분할에 100 KB의 process A가 적재되어 28 KB 크기의 Hole이 발생
- (c) Process B, C, D의 적재
- (d) Process B의 메모리가 반납되었지만 버디인 분할이 비어있지 않아 병합되지 않았다.
이때 B가 적재되어있던 256 KB 분할의 버디는 A와 C가 적재되어있는 256KB의 분할이다.
- (e) Process A의 메모리가 반납, 반납으로 생긴 128 KB짜리 분할에 process E가 적재되었다. Process C의 메모리가 반납, Buddy인 바로 아래 64KB의 빈 분할과 병합 되었다.
- (f) Process E의 메모리가 반납, Buddy인 바로 아래 128KB의 빈 분할과 병합되어 256KB짜리 분할을 만들고, 바로 아래 256KB buddy또한 비어있으므로 최종 512KB로 병합되었다.
그림에는 없지만 process D의 메모리가 반납 된 후엔 병합되어 다시 1MB의 메모리로 만들어 질 것이다.

