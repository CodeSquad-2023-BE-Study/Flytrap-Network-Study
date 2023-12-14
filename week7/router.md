# 들어가며

- 복습해봅시다. 인터넷에서 데이터를 어떻게 나르나요?

![image](https://github.com/CodeSquad-2023-BE-Study/Flytrap-Network-Study/assets/103120173/0ade6849-52f3-44a7-b8e0-0ba567a4e77f)

- 리피터 허브 (L2) → 스위칭 허브 (L2) → 라우터 → 라우터 → 라우터 …. *(L3!!)*
    - *(대충 엄청 중요하단 소리…!)*

# 라우터 Router

- 라우팅 Routing을 하는 장치입니다.
    - 네트워크 사이에 데이터가 이동하는 과정
- 라우터도 IP와 MAC 주소를 가집니다. host 처럼요.
- 라우터는 도착지가 될 수 없습니다.
    - host와 다른 점입니다.
    - 그러므로 패킷을 폐기하지는 않습니다. 만약 도착지가 자기 자신이 아니라면 다른 연결된 지점으로 포워딩합니다.
    - 만약 도착지 IP를 모른다면, 패킷을 폐기합니다.
- 네트워크와 네크워크를 연결합니다.
- 연결된 모든 네트워크에 대한 Routing Table을 가지고 있습니다.

## 라우팅 테이블 Routing Table

![image](https://github.com/CodeSquad-2023-BE-Study/Flytrap-Network-Study/assets/103120173/acf7104a-a452-445b-a6b7-ae33c56ae72a)

- 3가지 방식으로 라우팅 테이블의 내용을 업데이트 할 수 있습니다.
    - Directly Connected : 해당 라우터와 직접 연결되어 있음
    - Static Routes : 관리자가 직접 라우팅 경로를 업데이트 합니다.
    - Dynamic Routes : 다른 라우터와 통신하여 자동으로 경로를 학습합니다.
        - 알고리즘 : RIP, OSPF, BGP, EIGRP, IS-IS
- 여기에 기록된 것을 기반으로 라우팅합니다.

## ARP 테이블 ARP Tables

- 라우터는 ARP 테이블 또한 가지고 있습니다. 라우팅을 할 때 반드시 보게 됩니다.
- 동일한 네트워크 안에서 패킷을 보낼 때는 ARP Table을 확인하여 해당 맥주소로 패킷을 전달합니다.

<aside>
💡 **간단한 ARP table을 이용한 이더넷 통신 과정**

- R1 wants to communicate with Host A. R1 checks its routing table. The subnet on which Host A resides is a directly connected subnet.
- R1 checks its ARP table to find out whether the Host A’s MAC address is known. If it is not, R1 will send an ARP request to the broadcast MAC address of FF:FF:FF:FF:FF:FF.
- Host A receives the frame and sends its MAC address to R1 (ARP reply). The host also updates its own ARP table with the MAC address of the Gigabit0/0 interface on R1.
- R1 receives the reply and updates the ARP table with the MAC address of Host A.
- Since both hosts now know each other MAC addresses, the communication can occur.

[Do routers use ARP Table?](https://stackoverflow.com/questions/60737799/do-routers-use-arp-table)

</aside>

- 만약 다른 라우터를 통해서 가야한다면?

![image](https://github.com/CodeSquad-2023-BE-Study/Flytrap-Network-Study/assets/103120173/d40d80c4-96f8-4755-a013-f4d8b5d37ed3)

- ARP 테이블은 비어있는채로 시작하고, 네트워크 트래픽을 통해 구성되게 됩니다.
- ARP 테이블은 라우터 뿐만 아니라 IP를 가지는 모든 호스트가 가지고 있습니다.

### Data를 전달해봅시다.

1. `10.0.44.9` 에서 DATA를 `10.0.66.7` 로 보내고 싶습니다.
2. `10.0.44.9`는 L3 Header를 구성합니다.

| L3 Header |  | DATA |
| --- | --- | --- |
| SRC | 10.0.44.9 |  |
| DST | 10.0.66.7 |  |

1. `10.0.44.9` 는 **Default Gateway(R1)** 로 패킷을 보내야 한다고 판단합니다.
    1. DST를 보고 네트워크 부분을 비교하여 외부 네트워크인 것을 알 수 있기 때문입니다.
2. `10.0.44.9` 는 Default Gateway로 패킷을 보내기 위해 L2 Header를 만듭니다.
    1. DST를 알기 위해 ARP Table을 참고합니다.
    2. 만약 ARP Table에 없다면, ARP Request를 날립니다.
        1. “IP `10.0.44.1` 을 찾습니다. 있으면 MAC주소좀 보내주세요..! 저는 `10.0.44.9/a9a9` 입니다.”

| L2 Header |  | L3 Header |  | DATA |
| --- | --- | --- | --- | --- |
| SRC | a9a9 | SRC | 10.0.44.9 |  |
| DST | eee1 | DST | 10.0.66.7 |  |

1. 이제 R1이 데이터 패킷을 받았습니다.
2. R1은 L2 헤더를 삭제합니다.

| L3 Header |  | DATA |
| --- | --- | --- |
| SRC | 10.0.44.9 |  |
| DST | 10.0.66.7 |  |

1. R1은 이제 L3 Header에 있는 DST IP를 라우팅 테이블에서 찾습니다.
    1. Next hop은 `10.0.55.2` 입니다. R2로 보내려는 것이죠
2. R2에게 보내기 위해 R1은 L2 Header를 만듭니다.
    1. 역시 모르면 ARP Request를 날립니다.

| L2 Header |  | L3 Header |  | DATA |
| --- | --- | --- | --- | --- |
| SRC | eee3 | SRC | 10.0.44.9 |  |
| DST | eee2 | DST | 10.0.66.7 |  |

1. 이제 R2로 데이터 패킷이 옮겨갔습니다.
2. R2도 L2 Header를 삭제합니다.

| L3 Header |  | DATA |
| --- | --- | --- |
| SRC | 10.0.44.9 |  |
| DST | 10.0.66.7 |  |

1. R2가 L3 Header의 DST를 라우팅 테이블에서 확인합니다.
2. R2에 연결된 서브넷에 있음을 알 수 있습니다.
    1. 패킷의 마지막 hop은 `10.0.66.7`입니다.
3. R2에서 `10.0.66.7` 로 보내기 위해서 L2 Header를 만듭니다.
    1. 또 모르면 ARP Request를 보냅니다.

| L2 Header |  | L3 Header |  | DATA |
| --- | --- | --- | --- | --- |
| SRC | eee4 | SRC | 10.0.44.9 |  |
| DST | c7c7 | DST | 10.0.66.7 |  |
1. 이제 `10.0.66.7` 에 도착했습니다.
2. 이 호스트는 이제 L2 Header를 삭제합니다.

| L3 Header |  | DATA |
| --- | --- | --- |
| SRC | 10.0.44.9 |  |
| DST | 10.0.66.7 |  |
1. L3 Header도 삭제합니다.

| DATA |
| --- |
|  |
|  |
1. 이제 데이터를 사용하면 됩니다 👍
2. 이제 다시 응답을 보내줘야합니다.

| RESPONSE DATA |
| --- |
|  |
|  |
1. 이제 똑같은 방식으로 다시 돌아갑시다.

- 예외 상황

### 직접 설명해봅시다.

![image](https://github.com/CodeSquad-2023-BE-Study/Flytrap-Network-Study/assets/103120173/13fee54a-df73-4c5e-93de-e09d3b5d5fa3)

## ****계층적 라우팅 Hierarchical Routing****

- 각각의 라우터는 하위 계층 네트워크의 세부 사항을 고려하지 않음으로써, 라우팅 테이블의 엔트리 수를 줄여 처리 속도를 높일 수 있습니다.
- scale 을 늘리고 줄이는데 쉽습니다.

![image](https://github.com/CodeSquad-2023-BE-Study/Flytrap-Network-Study/assets/103120173/3dad835a-c28c-4686-8558-008654d4eb89)
![image](https://github.com/CodeSquad-2023-BE-Study/Flytrap-Network-Study/assets/103120173/761684e3-85e0-4a49-a144-29c1804f18d0)


- 계층적으로 구성된 인터넷

---

# Refs.
- https://talk.op.gg/s/lol/free/5478193/%EB%A7%9D%EC%82%AC%EC%9A%A9%EB%A3%8C%20%EC%9D%B4%EC%8A%88%EC%97%90%20%EA%B4%80%ED%95%9C%20%EB%82%98%EB%A6%84%EB%8C%80%EB%A1%9C%EC%9D%98%20%EC%A0%95%EB%A6%AC
- https://youtu.be/AzXys5kxpAM?si=bFWu0L44O-RWyGHu
