# Ch.5 Memory Hierarchy

## 1. Introduction

도서관에서, 컴퓨터 하드웨어에 대한 레포트를 쓴다고 하자. 이때 도서관에 있는 모든 책이 아닌 컴퓨터 하드웨어에 대한 책을을 책상앞에 놓고 레포트를 쓰는 동안 계속 참고할 것이다. 

이 원리는 우리 컴퓨터가 작은 메모리만을 사용해 사용자에게 아주 큰 메모리를 갖고 있다는 환상을 심어줄 수 있게 한다. 이러한 가정은 **Locality(지역성)** 에 기반한다. 
- Temporal Locality (시간적 지역성) : 최근에 참조된 아이템은 다시 참조될 가능성이 높다.
- Spatial Locality (공간적 지역성) : 참조된 아이템 근처에 있는 아이템은 참조될 확률이 높다. 

프로세서에 가까울 수록 작고, 속도가 빠르지만 가격은 가장 비싸다. 
이러한 지역성을 기반해서, 컴퓨터의 메모리도 **계층구조**를 기반으로 한다. 메모리 계층구조에서 메모리는 각각 다른 속도와 사이즈를 갖고 있다. 

<img width="533" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/b80fc68e-fe84-427b-9fc2-d37979841f0a">

이때 메모리 계층구조에서 존재하는 정보의 단위를 **block** 또는 **line** 이라고 한다. 

<img width="260" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/74066d7a-d1eb-4263-864d-c1415adb96ba">

만약 상위 레벨에서 프로세서가 요구한 데이터가 존재하면 **hit** 라고 하고, 존재하지 않으면 **miss** 라고 한다. **Hit rate** 는 프로세서가 접근하고자 하는 블럭이 상위 레벨에 존재하는 비율이다. 

**Hit time** 은 메모리 계층구조의 상위 레벨에서 블럭에 접근하는 시간이다. 여기에는 그 접근이 hit 또는 miss 인지 판단하는 시간도 포함된다. **Miss penalty** 는 상위 레벨의 블럭을 하위 레벨에 있는 알맞은 블럭으로 교체하는데 걸리는 시간이다. 

<img width="456" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/7f685b6d-4e1c-4eb3-99b2-acc59852ec46">

프로세서로부터 거리가 멀어질 수록, 사이즈는 커지고, 접근하는 시간도 커진다. 


## 2. Memory Technologies

메모리 계층구조에서 사용되는 4가지 메모리 기술이 있다. 메인 메모리는 **DRAM(dynamic random access memory)** 를 사용하고, 프로세서에 가까운 레벨 (캐시)는 **SRAM(static random access memory)** 를 사용한다. 개인 모바일 디바이스에서는 **flash memory** 를 사용한다. 가장 하위 레벨에서는 **magnetic disk** 를 사용한다. 

## 3. The Basic of Caches

이번 챕터에서는 가장 간단 형태의 캐시를 살펴본다. 여기서는 프로세서의 요청이 1개의 word 이고 블럭들도 1개의 word 로 구성된다고 가정한다. 

아래의 구조는 가장 간단 형태의 캐시인데, 요청 전에는 캐시가 $X_{1,},X_{2} ... X_{n-1}$ 을 가지고  있다가 캐시에 없는 $X_n$ 워드가 요청된 경우이다. 이것은 miss 상황이고, $X_n$ 워드는 메모리에서 캐시로 로드된다. 

<img width="384" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/6af11dd5-abd9-48f9-a86c-6166b5c5a90d">

이 시나리오에서, 2가지 질문에 대해 생각해봐야 한다. 1) 캐시에 어떠한 데이터 아이템이 있는지 어떻게 알 것인가? 2) 있다고 해도, 어떻게 찾을 것인가? 

메모리에 있는 워드에 캐시로 주소를 지정하는 가장 간단한 방법은 메모리의 주소를 기반으로 하는 방법이다. 이것을 **direct mapped** 라고 한다. 메모리 장소에 있는 각 워드는 캐시 내에 정확히 한 곳으로 매핑된다. 이것은 나머지 연산으로 처리할 수 있다. 

$$(Block \ address) \ modulo \ (Number\ of \ blocks \ in\  cache)$$

만약 캐시 엔트리의 수가 2의 거듭제곱이면, $log_2$ 를 취함으로써 캐시 내의 주소를 바로 계산할 수 있다. 예를 들어, 8-block 캐시는 블럭 주소의 하위 3개 비트($8=2^3$) 을 이용한다. 

하지만 이 방법으로는, 캐시 내에 특정 위치가 서로 다른 메모리에서 온 데이터를 가지고 있을 수 있다. 그렇다면 요청된 워드가 캐시 내에 있는 데이터에 상응하는지 어떻게 알까? 이것을 위해 **tags** 의 세트가 존재한다. tags 는 캐시 내에 있는 워드가 요청된 워드에 상응하는지 나타내는 주소 정보를 포함한다. tag 는 원래 주소의 상위 비트들만 포함하면 된다. 예를 들어, 원래 주소가 5bit 였다면 상위 2개 bit 만 포함하면 된다. 

<img width="483" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/041a9735-1dc4-43e5-be4f-a3daf9b76355">

또한 캐시가 유효한 데이터를 가지고 있는지를 나타내기 위해 **valid bit** 도 필요하다. 

### Accessing a Cache

아래는 8-block 캐시에 접근하는 9번의 메모리 참조 예시이다. 

<img width="587" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/ea5c29dd-7457-4861-93f8-56015b639478">

아래는 캐시가 각 hit/miss 마다 어떻게 변하는지를 나타낸 것이다. 

<img width="765" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/265b9381-4c57-4741-891a-9a577b1983ea">

8번째 접근인 $10010_2$ 이 참조되는 경우, 캐시 인덱스인 010에 이미 $11010_2$ 의 데이터가 있다. 따라서 캐시의 블럭은 교체되어야 한다. Direct-mapped 캐시에선, 하나의 인덱스 당 하나의 데이터만 담을 수 있으므로 이미 데이터가 있으면 교체해야 한다. 

주소의 하위 3개 비트(*cache index*)는 캐시 엔트리 내 인덱스로 작동하고, 상위 2개 비트(*tag field*)는 그 캐시에 있는 데이터가 실제 요청에 맞는 데이터인지 검증하고자 사용된다. 


<img width="345" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/4f236e2e-099a-486b-86d1-bdeb42dc3279">

n-bit 물리 주소일 때, 
- offset : 하위 b bits 로, 라인 내에 어떤 byte 인지 나타낸다.
- set index : s bits 로, 캐시 내에 어떤 라인인지 나타낸다.
- tag : 캐시에 있을 때 메모리 주소와 비교할 때 사용한다.

32-bit 주소이고, direct-mapped 캐시를 사용하고, 캐시 사이즈는 $2^n$ 블럭일 때를 보자. 캐시 사이즈가  $2^n$ 블럭이므로 n개의 bit 가 index 로 사용된다. 블럭 사이즈는 $2^m$ words (=$2^{m+2}$ bytes) 이므로, m bits 가 블럭 내에 워드로 사용되고 2 bits 는 주소의 byte offset 부분으로 사용된다. 이때 tag field 의 사이즈는 $32-(n+m+2)$ 이다. 

direct-mapped 캐시 내에 전체 비트의 수는 아래와 같다. 
$$2^{n} * (block \ size + tag \ size + valid \ field \ size)$$

블럭 사이즈가 $2^m$ words (= $2^{m+5}$ bits) 이고, valid bit 를 위해 1bit 가 필요하므로, 캐시 전체 사이즈는 아래와 같다.

$$2^{n}* (2^{m}* 32 + (32-n-m-2) + 1) = 2^{n}* (2^m*32+31-n-m)$$


<img width="499" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/68591929-95be-4eb3-9c02-7a1c1294eaaa">

위의 캐시는 1024 워드(=4KB) 를 저장한다. 이때 10개의 bit가 인덱스로 사용되고, 2개의 bit가 byte offset으로 사용된다.  32-bit 주소이므로, tag field 로 사용되는 비트 수 는 32-(10+2) = 20bit 이다. 

tag 는 주소의 상위 비트들과 비교해서 요청된 주소가 캐시 엔트리에 있는지 확인한다. 태그가 일치하고, valid bit가 1이면, 캐시 히트이고 index 를 사용해서 캐시 블럭 내 word 를 찾는다. 

**캐시 블럭**은 캐시 태그를 갖고 있는 캐시 데이터이다. 1-KB direct mapped 캐시이고 1-word 블럭을 가지고 있는 캐시 구조는 아래와 같다. (1 word = 4 byte 이므로 2개의 byte select bit를 가진다.)

<img width="825" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/816f0c7d-fd26-48f4-80fa-02ec813861b8">

만약 **공간 지역성을 더 활용**하고 싶다면, **블럭 사이즈를 증가**시킬 수 있다. (공간 지역성은 근처에 있는 데이터를 참조할 확률이 높다는 의미이므로 애초에 블럭 사이즈가 크다면 근처에 있는 데이터가 많아진다!) 블럭 사이즈를 증가시키면 miss rate 를 감소시킬 수 있다. 

아래는 1-KB direct mapped 캐시이고 32byte 가지고 있는 캐시 구조이다. 32byte 이므로 5개의 byte select bit를 가진다.

<img width="817" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/7c6d15b1-b67b-49e1-9879-f34a8547fa82">

Direct Mapped 캐시의 장단점은 다음과 같다.
- 장점
	- 간단한 디자인
	- 빠르게 만들기 쉽다
- 단점
	- **trashing** 에 취약하다. 
	- 만약 자주 쓰이는 2개의 캐시 라인이 같은 인덱스를 가진다면, 계속해서 하나를 캐시에서 내쫓아야 한다. 

캐시에서 hit time 을 개선하려면 1)캐시를 더 작게 만들거나, 2) Direct mapped 캐시를 사용할 수 있다. 캐시에서 hit ratio 를 개선하려면, 캐시 사이즈를 늘려야 한다. 따라서 hit time 과 hit ratio 간 트레이드 오프를 적절하게 계산해서 캐시를 설계해야 한다. 

