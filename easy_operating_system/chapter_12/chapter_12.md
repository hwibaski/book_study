# Ch.12 네트워크와 분산 시스템

# 01. 네트워크와 인터넷

## 01-01. 통신과 네트워크

<통신 방법의 종류>

- 단방향 통신
  : 한쪽방향으로만 통신이 이루어지는 방식 (ex: 모스부호, 라디오, TV 방송)

- 양방향 통신
  : 양쪽 방향으로 동시에 통신이 이루어지는 방식 (ex: 전화)

- 반양방향 통신
  : 단방향 통신과 양방향 통신의 중간 형태로, 양방향 통신이기는 하지만 어느 순간에는 한쪽 방향으로만 통신이 가능하다. (ex: 무전기)

<프로토콜>
: 통신을 위한 약속을 의미하며, '통신 규약'이라고 한다.

<통신연결>
: 네트워크가 구성되기 위한 물리적인 연결

- LAN
  : 가까운 거리에 연결된 네트워크 (근거리 네트워크)

- WAN
  : 국가 전체를 연결하거나 국가 간에 연결되는 네트워크 (장거리 네트워크, ex: KT, SK, LG)

- TCP
  : 전송제어 프로토콜이라고 하며, 데이터 전송에서 발생하는 에러를 바로잡고, 데이터가 최종 목적지 프로그램에 전달될 수 있도록 창구 역할을 담당한다.

- IP
  : 서로 다른 종류의 LAN 간에 데이터를 주고받는 방식에 대한 프로토콜

## 01-02. 유무선 통신

<웹 시스템>
: 웹 브라우저를 이용한 서비스르르 월드 와이드 웹 (WWW) 이라고 한다.

- 하이텔
  : 키보드만으로 명령을 내리는 '문자기반 사용자 인터페이스'

- 모자이크(오늘날 웹브라우저의 시초)
  : 한 화면에 문자와 그림을 동시에 표현할 수 있으며, 하이퍼 텍스트 기능을 제공

- 넷케이프 내비게이터

- 인터넷 익스플로러

<무선통신 시스템>

- 1G
  : 아날로그 음성 통화

- 2G
  : 디지털 음성 통화

- 3G
  : 음성통화 + 데이터 통신

- 3.9G (LTE)
  : 4G 데이터 통신 + 3G 음성 통화

- 4G
  : 고속 데이터 통신 + 음성 통화

- 5G
  : 초고속 데이터 통신 + 음성 통화

# 02. 분산 시스템

## 01-01. 분산 시스템의 개요

<분산 시스템>
: 작은 컴퓨터를 네트워크로 묶어 대형 컴퓨터 같은 능력을 가진 시스템
네트워크상에 분산되어 있는 컴퓨터가 작업을 처리하고 그 내용이나 결과를 서로 교환하는 시스템

<분산 시스템에서 고려해야할 사항>

- 기기의 독립성을 보장해야한다.

- 사용자는 시스템을 하나의 기기로 인식할 수 있어야 한다.

<분산 시스템의 종류>

- 강결합 시스템
  : 네트워크로 연결된 모든 컴퓨터의 프로세서가 하나의 메모리를 공유하는 방식
  모든 컴퓨터는 메모리를 공유하면서 같은 운영체제를 사용한다.
  약결합 시스템에 비해 속도가 빠르다.
  하나의 시스템에 문제가 생기면 다른 시스템에도 영향을 미침

- 약결합 시스템
  : 둘 이상의 독립된 시스템을 연결한 것으로, 오늘날의 네트워크는 이 방식으로 이루어져 있다.
  통신 오버헤드가 있기 때문에 강결합 시스템보다 느리다.
  컴퓨터들이 서로 독립적으로 작동하기 때문에 하나의 시스템에 장애가 발생해도 다른 시스템에 영향을 미치지 않는다.
  작업 분배를 통해 여러 기기가 작업을 나누어 처리할 수 있다.
  데이터나 처리를 분산함으로써 연산 속도를 향상할 수 있다.
  장애가 발생해도 시스템을 복구할 수 있다.

<분산 시스템에 사용되는 운영체제>

- 네트워크 운영체제
  : 각 컴퓨터가 독자적인 운영체제를 가진 채 사용자 프로그램을 통해 분산 시스템이 구현된 것으로, 낮은 수준의 분산 시스템 운영체제라고 볼 수 있다.

- 분산 운영체제
  : 시스템 내에 하나의 운영체제가 존재하고, 전체 네트워크를 통틀어서 단일 운영체제로 운영된다.

## 01-02. 클라이언트/ 서버 시스템

<클라이언트/ 서버 시스템의 구조>
: 완전한 분산 시스템과 같이 모든 컴퓨터가 동일한 지위를 갖지 않고, 작업을 요청하는 클라이언트와 요청을 받은 작업을 처리하는 서버의 이중 구조로 되어있다.

- 웹브라우저
  : 서버 컴퓨터에 접속하여 웹 페이지를 요청한다.

- HTTP
  : 서버 컴퓨터에 접속하여 웹 페이지를 요청할때 사용하는 프로토콜이다.

- 데몬
  : 서버는 항상 대기 상태로 있어야하는데, 이렇게 죽지 않고 살아서 서비스를 계속 진행하는 프로토콜을 데몬이라고 한다.

- CGI
  : 웹 데몬이 응용 프로그램에 질문한 것에 대한 결과를 HTML에 맞게 변형하는 프로그램이다. (ex: awk, perl, 아파치와 같은 스크립트 언어)

- HTML5
  : Active-X와 같은 비표준 기술을 사용하지 앟고도 다양한 프로그램을 개발할 수 있도록 만들어진 표준기술
  - 멀티미디어 기능 제공
  - 그래픽 지원
  - 양방향 통신 지원
  - 다양한 장치에 접근 가능

## 01-03. P2P 시스템

- 서버가 있는 P2P 시스템
  ex: 메신저

- 서버가 없는 P2P 시스템
  : 특정 파일의 소유자 정보를 여러 노드가 공유함으로써, 시스템의 한 노드가 사라지더라도 데이터 공유가 지속적으로 이루어진다.
  ex: 비트코인의 블록체인, 토렌트(같은 파일을 가진 여러 사람으로 부터 데이터를 나누어 받는다.)

## 01-04. 클라우드 컴퓨팅

<그리드 컴퓨팅>
: 여러 곳에 떨어져 있는 컴퓨팅 파워나 소프트웨어를 하나로 묶어 하나의 컴퓨터처럼 사용하는 기술
여러 곳에 분산된 컴퓨터를 하나로 묶어서 슈퍼컴퓨터와 맞먹는 컴퓨팅 파워를 제공하는 데 목적이 있다.

<클라우드 컴퓨팅>
: 하드웨어와 소프트웨어를 클라우드라는 중앙 시스템에 숨기고 사용자가 필요한 서비스만 그때그때 이용하는 방식의 컴퓨팅 환경

## 01-05. 네트워크 가치와 고가용성

<네크워크 저장장치의 분류>

- DAS
  : 서버와 같은 컴퓨터에 직접 연결된 저장장치를 말하며 HAS라고도 부른다. 컴퓨터에 직접 연결된 저장장치를 사용하기 때문에 다른 운영체제가 쓰는 파일 시스템을 사용할 수 없다.

- NAS
  : 기존 저장장치를 LAN이나 WAN에 붙여서 사용하는 방식으로 공유데이터의 관리 및 데이터의 중복 회피가 가능하다.

- SAN
  : NAS보다 진보된 형태의 네트워크 저장장치로, 데이터 서버, 백엡서버, RAID 등의 장치를 네트워크로 묶고 데이터 접근을 위한 서버를 두는 형태다.

<고가용성>
: 업무 또는 서비스 중단을 최소화하기 위해 이중화 작업을 하는 것으로, 작게는 운영체제의 디스크 미러링부터 크게는 시스템 자체를 이중화하는 것을 포함하는 개념이다.

- 상시대기
  : 가동 시스템과 백업 시스템으로 구성된다. 평상시에 대기상태를 유지하다가 가동시스템의 하드웨어 또는 네트워크 장비에 장애가 발생시, 가동 시스템의 자원을 백엔 시스템으로 이전하여 서비스가 중단되지 않게 한다.

- 상호인계
  : 2개의 시스템이 각각의 고유 서비스를 수행하다가 한쪽 시스템에 장애가 발생하면 상대 시스템으로 작업을 이동하여 동시에 2개의 업무를 수행한다.

- 컨커런트 액세스
  : 여러 시스템이 동시에 업무를 나누어 병렬처리한다.