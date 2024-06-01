# Memory Hierarchy

<details>
<summary>
<img src="https://raw.githubusercontent.com/Tarikul-Islam-Anik/Animated-Fluent-Emojis/master/Emojis/Objects/Bookmark%20Tabs.png" alt="Bookmark Tabs" width="25" height="25" /> Table of Contents </summary>

- [Memory Hierarchy](#memory-hierarchy)
	- [1. Introduction](#1-introduction)
	- [2. Memory Technologies](#2-memory-technologies)
	- [3. The Basic of Caches](#3-the-basic-of-caches)
		- [Accessing a Cache](#accessing-a-cache)
		- [Handling Cache Misses](#handling-cache-misses)
		- [Handling writes](#handling-writes)
	- [4. Measuring and Improving Cache Performance](#4-measuring-and-improving-cache-performance)
		- [Reducing Cache Misses by More Flexible Placement of Blocks](#reducing-cache-misses-by-more-flexible-placement-of-blocks)
		- [Locating a Block in the Cache](#locating-a-block-in-the-cache)
		- [Choosing which Block to Replace](#choosing-which-block-to-replace)
		- [Reducing the Miss Penalty using Multilevel Caches](#reducing-the-miss-penalty-using-multilevel-caches)
		- [Software Optimization via Blocking](#software-optimization-via-blocking)
	- [5. Virtual Machines](#5-virtual-machines)
		- [Requirements of a Virtual Machine Monitor(VMM)](#requirements-of-a-virtual-machine-monitorvmm)
	- [6. Virtual Memory](#6-virtual-memory)
		- [Placing a Page and Finding it again](#placing-a-page-and-finding-it-again)
		- [Page Faults](#page-faults)
		- [What about Writes?](#what-about-writes)
		- [Making Address Translation Fast : the TLB](#making-address-translation-fast--the-tlb)
		- [The Intrinsity FastMATH TLB](#the-intrinsity-fastmath-tlb)
		- [Integrating Virtual Memory, TLBs, and Caches](#integrating-virtual-memory-tlbs-and-caches)
		- [Implementing Protection with Virtual Memory](#implementing-protection-with-virtual-memory)
		- [Handling TLB Misses and Page Faults](#handling-tlb-misses-and-page-faults)
		- [Summary](#summary)
	- [Reference](#reference)


</details>






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

32-bit 주소이고, direct-mapped 캐시를 사용하고, 캐시 사이즈는 $2^n$ 블럭일 때를 보자. 캐시 사이즈가  $2^n$ 블럭이므로 n개의 bit 가 index 로 사용된다. 블럭 사이즈는 $2^m$ words (= $2^{m+2}$ bytes) 이므로, m bits 가 블럭 내에 워드로 사용되고 2 bits 는 주소의 byte offset 부분으로 사용된다. 이때 tag field 의 사이즈는 $32-(n+m+2)$ 이다. 

direct-mapped 캐시 내에 전체 비트의 수는 아래와 같다. 
$$2^{n} * (block \ size + tag \ size + valid \ field \ size)$$

블럭 사이즈가 $2^m$ words (= $2^{m+5}$ bits) 이고, valid bit 를 위해 1bit 가 필요하므로, 캐시 전체 사이즈는 아래와 같다.

$$2^{n}* (2^{m}* 32 + (32-n-m-2) + 1) = 2^{n}* (2^m*32+31-n-m)$$


<img width="499" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/68591929-95be-4eb3-9c02-7a1c1294eaaa">

위의 캐시는 1024 워드(=4KB) 를 저장한다. 이때 10개의 bit가 인덱스로 사용되고, 2개의 bit가 byte offset으로 사용된다.  32-bit 주소이므로, tag field 로 사용되는 비트 수 는 32-(10+2) = 20bit 이다. 

tag 는 주소의 상위 비트들과 비교해서 요청된 주소가 캐시 엔트리에 있는지 확인한다. 태그가 일치하고, valid bit가 1이면, 캐시 히트이고 index 를 사용해서 캐시 블럭 내 word 를 찾는다. 

**캐시 블럭**은 캐시 태그를 갖고 있는 캐시 데이터이다. 1-KB direct mapped 캐시이고 1-word 블럭을 가지고 있는 캐시 구조는 아래와 같다. (1 word = 4 byte 이므로 2개의 byte select bit를 가진다.)

<img width="825" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/816f0c7d-fd26-48f4-80fa-02ec813861b8">

만약 **공간 지역성을 더 활용**하고 싶다면, **블럭 사이즈를 증가**시킬 수 있다. (공간 지역성은 근처에 있는 데이터를 참조할 확률이 높다는 의미이므로 애초에 블럭 사이즈가 크다면 근처에 있는 데이터가 많아진다!) 블럭 사이즈를 증가시키면 miss rate 를 감소시킬 수 있다. 

아래는 1-KB direct mapped 캐시이고 32byte 가지고 있는 캐시 구조이다. 32byte 이므로 5개의 byte select bit를 가진다.

<img width="817" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/7c6d15b1-b67b-49e1-9879-f34a8547fa82">

Direct Mapped 캐시의 장단점은 다음과 같다.
- 장점
	- 간단한 디자인
	- 빠르게 만들기 쉽다
- 단점
	- **thrashing** 에 취약하다. 
	- 만약 자주 쓰이는 2개의 캐시 라인이 같은 인덱스를 가진다면, 계속해서 하나를 캐시에서 내쫓아야 한다. 

캐시에서 **hit time** 을 개선하려면 1)캐시를 더 작게 만들거나, 2) Direct mapped 캐시를 사용할 수 있다. 캐시에서 **hit ratio** 를 개선하려면, 캐시 블럭 사이즈를 늘려야 한다. 하지만 캐시 블럭 사이즈를 늘리면, 캐시 안에 들어갈 블럭의 개수가 작아져서 direct mapped 캐시에서는 thrashing 이 발생할 확률이 높아진다.  

### Handling Cache Misses

컨트롤 유닛에서 캐시 미스를 어떻게 다루는지 알아보자. 컨트롤 유닛은 미스를 감지하고, 요청된 데이터를 메모리에서 가져와야 한다. 

명령어 미스가 발생하면, 명령어 레지스터 안의 내용은 유효하지 않다. 적절한 명령어를 캐시에 가져오기 위해서, 메모리 계층구조의 하위 레벨에서 read 를 해야 한다. 실행의 첫번째 사이클에 PC의 값이 증가하기 때문에, 명령어 캐시 미스를 발생시키는 명령어는 PC - 4 의 값이다. 이 주소를 가지고, 메인 메모리에서 read 한다. 그리고 메모리가 응답하기를 기다리고, 원하는 명령어를 포함하는 words를 캐시에 write 한다. 
1. (현재 PC - 4) 값을 메모리에 보낸다. 
2. 메인 메모리가 read 를 수행하기를 기다린다.
3. 메인 메모리에서 가져온 데이터를 캐시 엔트리의 데이터 부분에 쓰고, 주소의 상위값을 tag field 에 쓰고, valid bit 를 켠다. 
4. 명령어 실행을 첫 스텝부터 다시 실행시키면, 이번에는 캐시에서 적절한 명령어를 찾을 수 있다. 

### Handling writes

write 는 조금 다르게 작동한다. store 명령어로 인해, 데이터를 캐시에만 쓰고, 메모리에는 쓰지 않았다고 하자. 그러면 캐시에 쓰기를 한 이후에는, 메모리와 캐시의 데이터는 다를 것이다. 이것을 **inconsistent** 하다고 말한다. 캐시와 메모리의 일관성을 지키는 가장 간단한 방법은 쓰기가 발생할 때마다 바로 캐시와 메모리에 데이터를 쓰는 것이다. 이것을 **write-through** 라고 한다. 

write miss 의 경우에는 어떻게 할까? 우선 메모리에서 워드 블럭을 가져온다. 캐시에 그 블럭이 위치된 이후에는, 캐시 블럭에 쓰기를 할 수 있다. 그 후 전체 주소를 사용해서 메모리에도 쓰기를 한다. 

이러한 방법은 매우 간단하지만, 성능은 그다지 좋지 않을 수 있다. write-through 는 모든 write에서 메모리에 write 하게 된다. 이러한 write 들은 매우 오랜 시간이 걸린다. 예를 들어, 전체 명령어 중 10%가 store 인 경우, 캐시 미스가 발생하지 않을 경우 CPI 가 1.0 이면, 추가적으로 write 마다 100 개의 사이클을 쓰게 되면 CPI가 1.0 + 100 * 10% = 11 이 된다. 

이러한 문제의 해결방법은 **write buffer** 를 사용하는 것이다. write buffer는 메모리에 쓰기 전에 데이터를 저장하고 있는다. 캐시와 write buffer에 데이터를 쓴 후, 프로세서는 실행을 계속할 수 있다. 메인 메모리에 write 가 되면, write buffer는 비워질 수 있다. write buffer가 꽉 차면, 프로세서는 write buffer에 빈 공간이 생길 때까지 기다려야 한다. 

다른 방법은 **write-back** 이 있다. 이 방법에서는 write 가 발생하면, 새로운 값이 캐시에만 쓰여진다. 수정된 블럭은 오로지 교체될 때 메모리 계층구조의 하위 레벨에 쓰여진다. write-back은 특히 프로세서가 메인 메모리가 다룰 수 있는 것보다 훨씬 write 를 빨리 하는 경우 성능을 향상시킬 수 있다. 



<img width="842" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/9c872a7d-dcdd-4cc6-8e00-68852b8bf16a">


## 4. Measuring and Improving Cache Performance

이번 챕터에서는 캐시 성능을 측정하는 방법을 알아본다. 그 후, 캐시 성능을 향상시킬 수 있는 2가지 방법을 알아본다. 첫번째는 **두개의 다른 메모리 블럭이 같은 캐시 위치에 들어가는 확률을 감소**시켜 miss rate 를 감소시킬 수 있는 방법이다. 두번째는 **메모리 계층구조에 새로운 레벨을 추가**해서 miss penalty 를 감소시키는 방법이다. 이것을 **multilevel caching** 이라고 한다. 

**CPU 시간**은 CPU가 프로그램을 실행할 때 보내는 클럭 사이클과 CPU 가 메모리 시스템을 기다리면서 보내는 클럭 사이클로 나눌 수 있다. 

$$CPU \ time = (CPU \ execution \ clock \ cycle + Memory-stall \ clock \ cycle) \times Clock \ cycle \ time$$

memory-stall 클럭 사이클의 주된 원인은 캐시 미스이다. memory-stall 클럭 사이클은 read-stall 사이클과 write-stall 사이클로 나눌 수 있다. **read-stall cycle**은 프로그램 당 read 하는 수로 정의할 수 있다. 
$$Read-stall \ cycles = \frac{Reads}{Program} \times Read \ miss\ rate \times Read \ miss\ penalty $$
writes 는 좀 더 복잡하다. write 의 stall 에서는 2가지 원인이 있는데, write miss 와 write buffer stalls 이다. 

$$ Write-stall \ cycles = \left(\frac{Writes}{Program} \times Write \ miss \ rate \times Write \ miss \ penalty\right)+ Write \ buffer \ stalls$$

캐시 구조에서, read 와 write penalty 는 거의 동일하다. write buffer stalls 를 무시 가능하다고 가정하면, read 와 write 를 합쳐서 **memory-stall clock cycles** 를 정의할 수 있다. 

$$Memory-stall \ clock \ cycles = \frac{Memory \ accesses}{Program} \times Miss \ rate \times Miss \ penalty$$

다르게 나타내면, 아래와 같다. 
$$Memory-stall \ clock \ cycles = \frac{Instructions}{Program} \times \frac{Misses}{Instruction} \times Miss \ penalty$$

### Reducing Cache Misses by More Flexible Placement of Blocks

direct-mapped에서는, 블럭은 메모리의 주소를 기반으로 하기 때문에 캐시안에 정확히 한 곳에만 위치할 수 있었다. 하지만 블럭을 실제로 위치시키는 방법은 여러가지가 있다. 

첫번째는 캐시 내에 아무곳에 위치시키는 것이다. 이 방법은 **fully associative** 라고 한다. 하지만 이 방법에서는 캐시 내에서 블럭을 찾으려면 캐시 엔트리 전체를 다 훑어봐야 한다. 따라서 이 방법은 블럭의 수가 적은 캐시에서만 실용적이다. 

다른 방법은 direct-mapped 와 fully associative의 중간인 **set associative** 방법이다. set associative 캐시에서는 블럭이 들어갈 수 있는 몇개의 위치가 있다. 만약 n개의 위치에 블럭이 들어갈 수 있다면, **n-way set associative** 캐시라고 한다. n-way set associative 캐시는 여러 개의 set 로 구성되는데, 각 set 는 n 개의 block 들로 구성된다. 메모리의 각 블럭은 index field 로 인해 유니크한 set으로 매핑된다. 그리고 블럭은 set 내 어느 위치에나 들어갈 수 있다. 

예를 들어서, 아래는 12번 블럭이 각각 방법마다 들어갈 수 있는 위치를 나타낸다. 

<img width="592" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/37b2601c-f1d8-4717-aa81-85ffc1f562fe">

- Direct Mapped
	- $(Block \ number) \ modulo \ (Number \ of \ blocks \ in \ the \ cache)$
	- 12 mod 8 = 4 이므로, 4번 블럭에만 들어갈 수 있다. 
- Set associative 
	- $(Block \ number) \ modulo \ (Number \ of \ sets \ in \ the \ cache)$
	- 12 mod 4 = 0 이므로, 0번 set 내 아무 위치에 들어갈 수 있다. 
	- 찾을 때는 set 내의 모든 tag 들을 다 훑어야 한다. 
- Fully associative
	- 아무 위치에 들어갈 수 있다. 

n-way set associative에서 **n** 에 따라 분류할 수 도 있다. Direct mapped 는 n=1 과 같고, fully associative 는 캐시가 m 개의 엔트리가 있을 때 m-way set associative 와 같다. 

associativity 정도를 증가시키면, miss rate 가 감소된다. 더 중요한 장점은, hit time 을 증가시키는 것이다. 



<img width="609" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/d7d53762-5925-45fe-9f05-debb33913400">

### Locating a Block in the Cache

이제 **set associative한 캐시에서 블럭을 찾는 방법**을 알아보자. direct-mapped 캐시와 동일하게, set associative 캐시의 각 블럭 내에는 블럭 주소를 나타내는 **주소 태그**가 있다. 프로세서로부터 온 주소가 블럭 주소와 매치하는지 찾기 위해 set 내에 모든 캐시 블럭의 태그를 비교한다. 

<img width="375" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/c5fc48ec-45ae-423a-902f-fddaff49c47b">

- **Tag** : 각 set 마다 매핑된 블럭의 수와 관련있다. 
- **Index** : set 의 개수와 관련 있다. (e.g. **4096**(= $2^{12}$) **blocks**, 4-word block size, **4-way**(= $2^2$ ) **set associative** 일때 -> 12 - 2 = 10 bits 필요. fully associative 캐시의 경우에는 index 가 필요없다. )
- **Block offset** : 블럭 사이즈와 관련 있다. (e.g. 4-word block size -> 16byte -> 4 bits 필요)


index 값은 타겟 주소가 포함되어 있는 set 을 선택하기 위해 사용되고, set 내에 있는 모든 tag 를 검색한다. 속도가 중요하기 때문에, set 내에 모든 태그들은 병렬적으로 검색된다. 

전체 캐시 사이즈가 동일한 경우, associativity 를 증가시키는 것은 set 당 블럭의 개수를 증가시킨다. 이것은 tag 를 검색해야 하는 범위의 증가를 의미한다. associativity를 2의 factor 로 증가시키는 것은 set 당 블럭의 수를 2배씩 증가시키고, index 의 사이즈를 1bit 만큼 감소시키고, tag 의 사이즈를 1bit 만큼 증가시킨다. fully associative 캐시의 경우 set 가 1개인 경우와 동일하므로, 모든 블럭이 병렬적으로 검색되어야 한다. 따라서 주소에는 인덱스가 없고, 모두 태그로 구성된다. 

direct-mapped 캐시의 경우, 엔트리가 오직 하나의 블럭에만 위치할 수 있으므로, 하나의 comparator 만 필요하다. 따라서 인덱싱만 사용해서 캐시에 접근할 수 있다. 

아래의 4-way set associative 캐시의 경우, 4개의 comparator와 그 중 1개를 고르기 위한 4-to-1 multiplexor 가 필요하다. 캐시의 접근은 적절한 set 를 찾기 위해 인덱싱하는 것과 그 중 tag 를 검색하는 과정으로 이루어진다. 

<img width="612" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/e6671e24-9a5e-4965-8e63-c5d81ccd28b8">


### Choosing which Block to Replace

direct-mapped 캐시에서 미스가 발생하면, 요청된 블럭이 단 한 곳으로만 매핑될 수 있기 때문에 그 위치를 교체해야 했다. associative 캐시의 경우에는 요청된 블럭을 넣을 수 있는 위치를 고를 수 있다. 따라서 교체할 블럭을 고를 수 있다. fully associative 캐시의 경우, 모든 블럭은 교체의 후보이다. set associative 캐시의 경우 선택된 set 내의 블럭 중 교체해야 한다. 

가장 많이 사용되는 방법은 **least recently used (LRU)** 이다. LRU 에서, 가장 오래전에 사용된 블럭이 교체된다. 

### Reducing the Miss Penalty using Multilevel Caches

프로세서의 매우 빠른 clock rate 속도와 DRAM을 접근하는데 필요한 긴 시간의 간극을 줄이기 위해, 대부분의 마이크로프로세서는 **추가적인 레벨의 캐싱**을 지원한다. 이 **second-level 캐시**는 같은 칩에 위치하고, 프라이머리 캐시에서 미스가 발생한 경우에 접근된다. 만약 second-level 캐시가 데이터를 가지고 있으면, first-level 캐시에 대한 miss penalty 는 second-level 캐시에 접근하는 시간이 된다. 이것은 메인 메모리에 접근하는 시간보다 훨씬 적게 걸린다. 

주 캐시와 두번째 캐시의 디자인 철학은 다르다. 주 캐시는 hit time 을 최소화하는데 중점을 두고, 두번째 캐시는 되도록 메인 메모리에 가지 않게끔 miss rate 를 최소화하는데 중점을 둔다. 

싱글 레벨 캐시에 비해서, 멀티레벨 캐시의 주 캐시 사이즈는 대체로 작다. 그리고, 주 캐시는 더 작은 블럭 사이즈를 사용해서, 캐시 사이즈를 줄이고 miss penalty 를 줄인다. 두번째 캐시는 싱글 레벨 캐시보다 훨씬 크고, miss rate 를 줄인다. 

### Software Optimization via Blocking

배열을 다룰 때, 배열에 대한 접근이 메모리에 순차적이면 좋은 성능을 얻을 수 있다. 여러 개의 배열을 다룰 때 어떤 배열은 row 순서로 접근하고 다른 배열은 column 순서로 접근한다고 하자. 배열에 대해 전체 row 또는 column 을 다루는 것보다 **blocked 알고리즘**은 block 에 대해 연산한다. 목표는 캐시에 로드된 데이터를 교체되기 전까지 최대한 많이 접근하는 것이다. 이것은 캐시 미스를 줄이고, 시간적 지역성을 증가시킨다. 

코드의 예시를 보자. 매트릭스 연산이다. 

```c
for (int j=0 ; j<n ; ++j)
{
	double cij = C[i+j*n]; // cij = C[i][j]
	for (int k=0 ; k<n; k++) {
		cij += A[i+k*n] * B[k+j*n]; // cij += A[i][k] * B[k][j]
		C[i+j*n] = cij; // C[i][j] = cij
	}
}
```

<img width="646" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/0d054faf-f821-4fa2-83a7-4f07d9080c94">


여기서는, B 의 N-by-N 원소들을 모두 읽고, A의 row 에서 N개의 원소들을 읽고 C의 row 에 쓴다. 

capacity miss 의 수는 N 과 캐시의 사이즈에 달려있다. 만약 캐시에 3개의 N * N  매트릭스를 저장할 수 있다면 아무런 문제가 없다. 앞선 예시들에선, N=32 여서 32 * 32 = 1024 원소들이고, 각 원소는 8bytes 여서 3개의 매트릭스가 24KiB 를 차지한다고 가정했다. 이것은 Inter Core i7 의 32KiB 캐시에 아주 잘 들어간다. 

만약 캐시가 1개의 N * N 매트릭스와 1개의 row 만 저장할 수 있다면, A의 i번째 row 와 B가 캐시에 저장되어야 한다. 

접근되는 원소들이 캐시에 모두 있음을 보장하기 위해, 코드를 submatrix 에 대해 연산하도록 수정할 수 있다. 사이즈 N에 대해 연산하는 것이 아닌, **BLOCKSIZE** 에 대해 연산한다. BLOCKSIZE는 *blocking factor* 라고도 한다. 

```c
#define BLOCKSIZE 32
void do_block(int n, int si, int sj, int sk, double *A, double *B, double *C)
{
	for (int i = si; i < si+BLOCKSIZE; ++i) {
		for (int j = sj; j < sj+BLOCKSIZE; ++j) {
			double cij = C[i+j*n]; // cij = C[i][j]
			for (int k = sk; k < sk + BLOCKSIZE; k++) {
				cij += A[i+k*n] * B[k+j*n]; // cij += A[i][k] * B[k][j]
			C[i+j*n] = cij; // C[i][j] = cij
			}
		}
	}
}
void dgemm(int n, double* A, double* B double* C)
{
	for (int sj = 0; sj < n; sj += BLOCKSIZE)
		for (int si = 0; si < n; si += BLOCKSIZE)
			for (int sk = 0; sk < n; sk += BLOCKSIZE) 
				do_block(n,si,sj,sk,A,B,C);
}
```

<img width="651" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/8e609aaf-f9ef-416d-88b4-45f0aec8eaad">

블럭을 이용해 캐시 미스를 줄일 수 도 있지만, 블럭은 레지스터 재할당에도 도움이 될 수 있다. 작은 블러킹 사이즈로 인해 블럭이 레지스터에 저장되고, 프로그램에서 load 와 store 의 수를 줄일 수 있다. 

## 5. Virtual Machines

가상 머신(VM)은 1960년대 중반에 처음 개발되었다. 그러다 1980년대와 1990년대에 다다르면서, 
- 현대 시스템에서 격리와 보안에 대한 중요도 증가
- 표준 운영체제에서 보안에 대한 문제와 신뢰도에 대한 문제
- 컴퓨터를 다수의 관련없는 사용자가 공유하는 것 
- 프로세서의 속도가 증가해서, VM 의 오버헤드가 허용가능한 수준까지 온 것
으로 인해 VM의 인기가 증가했다. 

이번 챕터에서는 **instruction set architecture(ISA)** 레벨에서 전체 시스템-레벨 환경을 제공하는 VM에 대해 살펴본다. 

시스템 가상 머신은 사용자에게 사용자가 전체 컴퓨터를 사용하고 있다는 환상을 제공한다. 하나의 컴퓨터는 여러 개의 VM 을 실행하고, 여러 개의 다른 운영체제를 지원한다. 

VM을 제공하는 소프트웨어는 **virtual machine monitor(VMM)** 이라고 한다. 제공하는 하드웨어 플랫폼을 *host* 라고 하고, 이 리소스는 *guest* 가 공유한다. 

VM은 보안도 제공하지만, VM이 제공하는 2가지 다른 장점들이 있다.
1. Managing software : VM은 추상화를 제공해서, 레거시 운영체제도 동작시킬 수 있다. 
2. Managing hardware : VM은 서로 다른 소프트웨어가 작동해도 하드웨어 리소스를 공유할 수 있도록 한다. 

### Requirements of a Virtual Machine Monitor(VMM)

VM 모니터는 게스트 소프트웨어에게 소프트웨어 인터페이스를 제공한다. 따라서 게스트끼리 서로를 볼 수 없도록 격리하는 역할을 한다. 
- 게스트 소프트웨어는 네이티브 하드웨어에서 작동하는 것과 동일하게 VM에서 작동해야 한다. 
- 게스트 소프트웨어는 실제 시스템 리소스를 직접적으로 수정할 수 없어야 한다. 
프로세서를 가상화하기 위해서, VMM는 거의 모든 것을 제어해야 한다 - I/O, exceptions, privileged state, interrupts. 따라서 VM은 유저 모드에서 동작하는 게스트 VM보다 높은 권한 상태(커널 모드)에서 동작해야 한다. 따라서 시스템 가상화의 2가지 기본 요구사항은 다음과 같다.
- 2개의 프로세서 모드 - 시스템 / 유저
- 시스템 모드에서만 접근 가능한 특권 명령어들. 만약 유저모드에서 실행하면 trap 을 발생시킨다. 모든 리소스는 특권 명령어를 사용해서 제어해야 한다. 

## 6. Virtual Memory 

메인 메모리도 디스크와 같은 **세컨더리 저장공간을 위한 "캐시" 역할**을 할 수 있다. 이것을 **가상 메모리(virtual memory)** 라고 한다. 가상 메모리를 사용하게 된 2가지 동기가 있는데, 1) **여러 개의 프로그램 사이에 메모리를 안전하게 공유하는 것** , 2) **메인 메모리의 저장공간 부족에 대한 걱정 없이 프로그래밍을 하는 것**이다. 

여러 개의 가상 머신이 같은 메모리를 공유하기 위해서는, 가상 머신이 자신에게 할당된 메모리만 읽고 쓸 수 있게 하여 서로에게서 보호하고 격리해야 한다. 메인 메모리는 현재 활성화된 가상 머신이 필요한 부분만 저장하고 있으면 된다. 따라서 **locality** 는 캐시뿐만 아니라 가상 메모리에도 적용된다. 또한 가상 메모리는 프로세서와 메인 메모리를 효율적으로 공유할 수 있게 한다.  

- Motivation #1 : 여러 개의 프로그램 사이에 메모리를 안전하게 공유하는 것

컴파일 단계에선, 어떤 가상 머신이 다른 가상 머신과 메모리를 공유할 지 알 수 없다. 그리고 가상 머신이 작동하는 동안 공유하는 메모리는 동적으로 변한다. 따라서, 우리는 각 프로그램을 각자의 **주소 공간(address space)** -오로지 그 프로그램만 접근 가능한 메모리 공간 범위 -  로 컴파일 하고자 한다. 가상 메모리는 프로그램의 주소 공간을 물리적 주소로 **번역**해주는 역할을 한다. 이 번역 과정은 프로그램의 주소 공간을 다른 가상 머신으로부터 **보호**해준다. 

![image](https://github.com/ddoddii/OS-CA-Study/assets/95014836/fdba39ac-7b9f-4864-a2d1-fbc3a416ec4a)

- Motivation #2: 메인 메모리의 저장공간 부족에 대한 걱정 없이 프로그래밍을 하는 것

가상메모리의 두번째 동기는 하나의 유저 프로그램이 주 메모리의 사이즈를 넘어서는 메모리 사이즈도 사용할 수 있게 하는 것이다. 이전에는, 만약 프로그램이 메모리 사이즈보다 컸다면 프로그래머가 그 메모리 안에 들어가도록 다시 작성해야 했다. 가상 메모리는 메모리 계층을 2개의 레벨 (주 메모리, 세컨더리 저장공간)으로 나눠서 이 작업을 대신 해준다. 

가상 메모리 블럭은 **page** 라 부르고, 가상 메모리 미스는 **page fault** 라고 한다. 가상 메모리를 가지고, 프로세서는 가상 주소를 만든다. 이 가상 주소는 하드웨어와 소프트웨어를 가지고 메모리에 접근할 때 사용할 수 있는 물리 주소로 번역된다. 이 과정을 **address translation** 이라고 한다. 

<img width="412" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/55013770-433c-49b6-815f-6e937ccddc2e">

도서관에서의 비유를 다시 생각해보면, 가상 주소를 책의 제목으로 생각할 수 있고, 물리 주소를 도서관의 실제 책 번호로 생각할 수 있다. 

가상 메모리는 **relocation** 을 제공해서 프로그램을 로딩하는 과정도 간단하게 해준다. relocation은 프로그램이 사용하는 가상 주소를 여러 개의 물리주소로 매핑한다. 따라서 주 메모리의 어느 공간에든 프로그램을 로드할 수 있다. 더 나아가 가상 메모리 시스템은 프로그램을 고정된 크기의 블럭(Pages) 로 재할당해서, 프로그램을 할당할 때 연속된 메모리 블럭을 찾을 필요가 없게 해준다. 대신 운영체제는 메인 메모리에 알맞은 페이지 수만 찾면 된다. 

가상 메모리에서, 주소는 *virtual page number* 와 *page offset* 으로 나뉜다. 아래는 가상 주소에서 실제 물리 주소로 번역하는 과정을 보여준다. page offset 에 있는 비트의 수는 페이지 크기를 결정한다. 가상 주소에서 사용가능한 페이지 수와 실제 물리 주소에서 사용가능한 페이지 수는 일치하지 않고, 대체로 가상 주소에서 사용가능한 페이지 수가 더 크다. 이것은 프로그램이 무한대의 가상 주소를 가지고 있다는 환상을 가질 수 있게 해준다. 


<img width="512" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/087133b4-1b8d-4df8-8fbf-5fdc328e28ba">

위의 그림에서 page size 는 $2^{12}$=4KiB  이다. 메인 메모리안에서 사용가능한 페이지는 $2^{18}$ 개이고(물리 페이지 넘버가 18bits 를 가짐), 메인 메모리는 1GiB를 가질 수 있는 반면 가상 주소 공간은 4GiB 이다. 

가상 메모리의 디자인은 page fault 를 최대한 피하고자 설계되었다. page fault 가 한번 생기면 디스크에 갔다와야 해서 수백만번의 클럭 사이클이 필요하다. miss penalty 가 매우 크기 때문에, 가상 메모리는 아래와 같이 디자인되었다.
- 페이지는 높은 접근 시간을 상쇄할 수 있도록 충분히 커야 한다. 요즘은 4KiB에서 16KiB 페이지 사이즈를 사용한다. 
- page fault 비율을 최소화하는 시스템이어야 한다. 여기서 사용되는 주된 기술은 fully associative 를 사용하는 것이다. 따라서 페이지는 메모리 내 어느 공간이든 들어갈 수 있다. 
- page fault 를 소프트웨어에서 다룬다. 소프트웨어에서 다루는 것에 대한 오버헤드는 디스크를 갔다오는 것보다 작기 때문이다. 
- 가상 메모리에서 write-through 는 너무 오래 걸리기 때문에 사용하지 않고, 대신 write-back 을 사용한다. 

### Placing a Page and Finding it again

page fault 의 페널티가 너무 크기 때문에, 가상 메모리 디자이너들은 page placement 를 최적화해 page fault 빈도를 줄였다. 가상 페이지가 어느 물리 페이지에 매핑될 수 있다면, 운영체제는 page fault 가 발생했을 때 어느 페이지든 교체할 수 있다. 예를 들어, 운영체제는 고도화된 알고리즘과 자료구조를 사용해서 페이지 사용빈도를 관리하고, 추후에 더 필요없을만한 페이지를 선택한다. 이 방법은 획기적으로 page fault 를 줄일 수 있다. 

하지만 앞서 캐시 구조에서 봤듯이, fully associative 방법을 사용하면 전체 엔트리를 모두 훑어야 해서 비효율적이다. 가상 메모리는 **page table** 을 사용해서 이 문제를 해결한다. page table는 가상 주소의 페이지 넘버로 인덱싱되어 있고, 상응되는 물리 페이지 넘버를 찾기 위해 사용된다. 각 프로그램은 자신만의 page table이 있어서, 그 프로그램의 가상 주소 공간을 메인 메모리로 매핑한다. 

도서관 비유에서, page table는 책 제목과 도서관에 있는 실제 책의 위치를 매핑하는 역할을 한다. 메모리에서 page table의 위치를 알아내기 위해, 하드웨어는 page table의 시작을 가리키는 레지스터를 포함하는데, 이것을 **page table registe**r 라고 한다. 

프로그램 카운터, 레지스터와 마찬가지로 페이지 테이블은 가상 머신의 *상태(state)* 를 정의한다.  만약 다른 가상 머신이 프로세서를 사용하게 하려면, 이 상태를 저장해야 한다. 나중에, 이 상태를 복구한 후에 가상머신은 실행을 계속할 수 있다. 이 상태를 *process* 라고 할 수 있다. 프로세스는 프로세서가 실행하는 도중에는 *active* 라고 하고, 그 외에는 *inactive* 하다. 운영체제는 프로세스의 상태를 로드해서 프로세스를 active 하게 만들 수 있다. 

프로세스의 주소 공간은 페이지 테이블에 의해 정의된다. 페이지 테이블 전체를 저장하는 것보다, 운영체제는 페이지 테이블 레지스터가 포인트하는 위치를 활성화하고 싶은 프로세스의 페이지 테이블 시작 위치로 옮긴다. 각 프로세스는 자기만의 페이지 테이블이 있다. 프로세스끼리 충돌하지 않도록 물리 메모리를 할당하고 페이지 테이블을 업데이트 하는 것은 운영체제의 책임이다.  

<img width="630" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/4e4321a0-35a9-4f7b-be60-21a555175ce8">

위의 그림은 페이지 테이블 레지스터, 가상 주소, 페이지 테이블을 사용해서 하드웨어가 가상 주소를 물리주소로 번역하는 과정을 나타낸다. 각 페이지 테이블 엔트리에 valid bit 는 페이지가 유효한지를 나타내기 위해 사용된다. 이 valid bit가 꺼져있으면(=0이면), 페이지가 메인 메모리에 없다는 것이고 page fault 가 발생한다. valid bit가 켜져있으면 페이지가 메모리에 있다는 의미이고, 엔트기는 물리 페이지 넘버를 포함한다. 

페이지 테이블이 모든 가상 페이지에 대한 매핑을 포함하기 때문에, 태그가 필요하지 않다. 

### Page Faults

valid bit가 꺼져있으면, **page fault** 가 발생한다. 그러면 운영체제가 핸들링하는데, exception 메커니즘을 사용해서 처리한다. 운영체제가 제어권을 받으면, 메모리 계층구조 중 아래 계층에서 페이지를 찾아야 하고, 이 페이지를 메인 메모리 중 어디에 둘지 결정해야 한다. 

![image](https://github.com/ddoddii/OS-CA-Study/assets/95014836/40fa2ffd-d2f6-46c8-b45c-090a923f5403)

가상 주소 자체로는 해당 페이지가 디스크 어디에 있는지 나타내지 못한다. (도서관 비유에서, 책 제목만 가지고 책이 도서관 어디에 꽂혀 있는지 알지 못한다.) 가상 메모리 시스템에서도, 가상 주소 공간에서 페이지가 디스크의 어느 위치에 있는지 관리해야 한다.. 

메모리에 있는 어떠한 페이지가 교체될지 사전에 모르므로, 운영체제는 플래시 메모리 / 디스크에 **swap space** 라는 특별한 공간을 만들고 프로세스가 만들어질 때 프로세스를 위한 페이지들을 모두 저장한다. 그리고 특별한 자료구조를 사용해서 각 가상 페이지가 디스크의 어느 위치에 저장되어 있는지 기록한다. 이 자료구조는 페이지 테이블의 일부일 수도 있고, 따로 저장될 수 도 있다.


<img width="523" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/be0e07eb-83f3-4d7d-9501-2412c6186f95">

운영체제는 또 다른 자료구조를 사용해서 어떤 프로세스와 어떤 가상 주소가 각 물리 페이지를 쓰는지 기록한다. page fault 가 발생하면, 만약 메인 메모리에 있는 모든 페이지가 사용중이면(=빈 공간이 없으면) 운영체제는 교체할 페이지를 골라야 한다. 운영체제는 **least recently used(LRU)** 교체 방법을 사용한다. 교체된 페이지는 디스크 상의 swap space 에 쓰여진다. 

LRU 를 통째로 구현하는 것은 복잡하기 때문에, 대부분의 운영체제는 최근에 사용되지 않는 페이지를 추적하는 approximate LRU를 구현한다. 이것을 위해 몇몇 컴퓨터는 페이지가 접근될 때 킬 수 있는 **reference bit** 또는 **use bit** 을 제공한다. 운영체제는 주기적으로 reference bit을 끄고, 특정 기간 동안 해당 페이지가 접근 되었는지 확인한다. 이 정보를 사용해서 운영체제는 reference bit가 계속 꺼져있는 페이지는 최근에 접근되지 않은 페이지임을 알 수 있다. 


### What about Writes? 

캐시에 접근하는 것과 메인 메모리에 접근하는 것의 클럭 사이클 차이는 적게는 수십배, 크게는 수백배 차이가 난다. 따라서 write-through 방법을 사용할 수 도 있다. 가상 메모리 시스템에서는 디스크에 write 하는 것은 수백만의 클럭 사이클이 걸리기 때문에 write-through가 사실상 불가능하다. 대신에, 가상 메모리는 **write-back** 을 사용해야 한다. write-back 에서는 개별 write는 메모리에 하고, 메모리에서 페이지가 교체될 때 디스크에 write 를 한다. 

좀 더 최적화를 하기 위해, **dirty bit** 를 사용하기도 한다. 만약 페이지가  메모리에 read 된 이후로 write 되었으면 dirty bit 을 키고, 이 dirty bit 가 켜져있을 때만 디스크에 페이지를 복사한다. 따라서 변경된 페이지를 **dirty page** 라고도 한다. 

### Making Address Translation Fast : the TLB

페이지 테이블이 메인 메모리에 저장되어 있기 때문에, 프로그램은 메모리 접근을 2번씩 해야 한다. 처음은 물리 주소를 얻기 위해서, 두번째는 실제 데이터를 얻기 위해서이다. 이때 성능을 향상시키는 방법은 **locality** 를 사용하는 것이다. 가상 페이지 넘버의 번역이 한번 사용되었으면,  그 페이지에 있는 워드가 시간적/공간적 지역성이 있으므로 해당 페이지 넘버에 대한 번역이 근접한 미래에 다시 사용될 것이라고 가정한다. 

따라서 현대의 프로세서는 최근의 번역을 저장하기 위한 특별한 캐시를 가지고 있다. 이것을 **translation-lookaside buffer(TLB)** 라고 한다. (번역 캐시라고 생각하면 된다.) 


<img width="670" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/a7bedfb4-7f48-4012-aab1-b8509b23c999">

TLB의 태그 엔트리는 가상 페이지 넘버의 일부를 저장하고 있다. 그리고 TLB의 데이터 엔트리는 물리 페이지 넘버를 저장하고 있다. 

참조할 때 페이지 테이블 대신 TLB 를 사용하므로, TLB는 dirty bit 과 reference bit 과 같은 다른 상태 비트도 포함해야 한다. 

참조할 때마다, 우리는 TLB에서 가상 페이지 넘버를 찾는다. 만약 히트면, 물리 페이지 넘버는 실제 물리주소를 생성하기 위해 사용되고, reference bit 를 켠다. 프로세서가 write를 수행하면, dirty bit를 켠다. 

TLB에서 miss가 발생하면, 우리는 page fault 인지 아니면 TLB miss 인지 판단해야 한다. **페이지가 메모리에 존재**하면, TLB miss는 단지 번역이 없는 것을 의미한다. 이 경우에는 프로세서는 페이지 테이블에서 단순히 번역 정보만 TLB로 가져오면 된다. 만약 **페이지가 메모리에 존재하지 않으면**, 이때의 TLB miss 는 page fault 를 의미한다. 이때 프로세서는 exceptino을 사용해서 운영체제에게 알린다. 메인 메모리에 있는 페이지의 수보다 TLB에 훨씬 적은 수의 엔트리가 있기 때문에, 실제 page fault 보다 TLB miss 가 더 자주 발생할 것이다. 

TLB miss가 발생하고, 없는 번역 정보를 페이지 테이블로부터 가져온 후에는 **교체할 TLB 엔트리를 선택**해야 한다. TLB 엔트리에 dirty bit과 reference bit 이 있기 때문에, 엔트리를 교체할 때 이 두개의 비트를 페이지 테이블로 복사해야 한다. 이 경우에는 write-back 를 사용하는 것이 효과적인데, 왜냐하면 TLB miss rate 가 매우 작을 것이라고 기대하기 때문이다. 

TLB에서 사용되는 대표적인 값들은 아래와 같다.
- TLB size : 16 ~ 512 entries
- Block size : 1 ~ 2 page table entries 
- Hit time : 0.5 ~ 1 clock cycle
- Miss penalty : 10 ~ 100 cycles
- Miss rate : 0.01% ~ 1%

디자이너는 TLB에서 다양한 종류의 associativity를 사용한다. 몇몇 시스템은  **fully associativite TLB** 를 사용하는데, 이 방법은 miss rate 가 작아진다. 이때 TLB 의 크기는 매우 작으므로 fully associativite 매핑의 단점은 상쇄된다. 다른 시스템은 **큰 TLB 와 작은 associativity**를 사용한다. fully associativite 매핑을 사용하면 교체할 엔트릴 선택하는 LRU를 하드웨어에 구현하는 것이 꽤나 까다롭기 때문이다. 그리고 TLB miss 가 page fault 보다 훨씬 자주 발생하고 자주 핸들링되어야 하기 때문에, 복잡한 소프트웨어 알고리즘을 사용하기 어렵다. 따라서 많은 시스템은 교체할 엔트리를 그냥 랜덤하게 선택한다. 


### The Intrinsity FastMATH TLB

실제 프로세서의 예시를 보기 위해, Intrinsity FastMATH TLB를 보자. 이 메모리 시스템은 4KiB 페이지와 32-bit 주소 공간을 사용한다. 따라서 가상 페이지 넘버는 20bits 이다. 물리 주소는 가상 주소와 같은 길이이다. TLB는 16개의 엔트리를 포함하고, gfully associativite 하다. 각 엔트리는 64bit 이고, 20-bit tag 를 포함한다. 

<img width="587" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/36e3e4d5-cfa4-4ced-8b37-6a54e540aa3d">

아래는 read / write 요청을 처리하는 과정이다. TLB miss 가 발생하면, MIPS 하드웨어는 페이지 넘버를 특별한 레지스터에 저장하고 예외를 발생시킨다. 이 예외는 운영체제를 호출하고, miss 를 소프트웨어에서 처리한다. 

미스가 발생한 페이지의 물리 주소를 찾기 위해서, TLB miss 루틴은 가상 주소의 페이지 넘버와 페이지 테이블 레지스터를 사용해서 페이지 테이블을 인덱싱한다. 

TLB를 업데이트할 수 있는 특별한 명령어들을 사용해서, 운영체제는 페이지 테이블로부터 온 물리 주소를 TLB에 저장한다. TLB miss 는 대략 13 클럭 사이클이 걸린다. 만약 페이지 테이블의 valid bit 가 꺼져있으면, 실제 page fault 가 발생한다. 

write 요청은 조금 더 복잡한데, 이때는 **write access bit** 를 사용해서 write 할 권한이 있는지 확인해야 한다. 만약 write access bit이 꺼져 있는데 write 를 하려고 하면 예외가 발생한다. 


<img width="591" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/3c92b4d6-23dc-4f61-bf35-e2294282be27">


### Integrating Virtual Memory, TLBs, and Caches

가상 메모리와 캐시 시스템은 메모리 계층구조에서 같이 동작한다. 따라서 데이터가 메인 메모리에 없으면 캐시에도 있을 수 없다. 운영체제는 메인 메모리에 있는 페이지를 디스크로 옮길 때 캐시에 있는 페이지를 flush 한다. 동시에, 운영체제는 페이지 테이블과 TLB를 수정해서, 옮겨진 페이지에 접근하는 것은 page fault 가 발생하도록 한다. 

가장 최상의 상황은 가상 주소가 TLB로 인해서 번역되고, 캐시로 보내져서 데이터를 찾아오는 것이다. 최악의 상황은 메모리 계층구조의 3부분(TLB, 페이지 테이블, 캐시) 에서 모두 miss가 발생하는 것이다. 

아래는 각 메모리 계층구조에서 발생할 수 있는 miss/hit 상황들이다. 

<img width="670" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/1c4aae67-3991-4e4c-8043-0276ca8a873c">

위의 상황에서는 캐시에 접근하기 전에 모든 메모리 주소가 물리 주소로 번역되어야 함을 가정한다. 이것을 **physically addressed cache** 라고 한다. 이런 시스템에서는, 캐시 히트 시에 메모리에 접근하는 시간은 TLB 접근 시간과 캐시 접근을 더해야 한다. 

반면에, 프로세서는 캐시를 전부 혹은 일부만 가상 주소인 주소로 인덱싱할 수 있다. 이것을 **virtually addressed cache** 라고 한다. 여기서는 tags 가 가상 주소이다. 이러한 캐시에서 보통 캐시 접근에서는 TLB가 사용되지 않는다. 왜냐하면 캐시는 물리 주소로 번역할 필요 없이 가상 주소로 접근할 수 있기 때문이다. 따라서 캐시에 접근하는 시간을 줄일 수 있다. 하지만 캐시 미스가 발생하면, 프로세서는 가상 주소를 물리 주소로 번역하고 메인 메모리에서 캐시 블럭을 가져와야 한다. 

만약 가상 주소로 접근되는 캐시가 프로세스 간에 공유되면, **aliasing** 문제가 발생할 수 있다. Aliasing 은 같은 오브젝트가 2개의 다른 이름을 가질 때 발생하는데, 이 경우에는 같은 페이지에 2개의 가상 주소가 사용되는 상황이다. 이 모호함은 문제를 발생시키는데, 왜냐면 페이지에 있는 워드는 각기 다른 가상 주소를 가지는 2개의 공간에 캐시될 수 있기 때문이다. 이러면 하나의 프로그램이 다른 프로그램이 모르는 사이에 공유되는 데이터를 write 할 수 있다. 

위의 2가지 디자인 포인트에 대한 합의점은, **virtaully indexed but physically tagged(VIPT)** 방식이다. 이 방식을 보기 전에 인덱싱과 태깅의 정의를 명확하게 하자. 
- **인덱싱(Indexing)**: 캐시 메모리 내에서 데이터가 저장되는 위치를 결정한다. 인덱싱은 특정 데이터를 빠르게 찾기 위해 사용되는 메커니즘이다.
- **태깅(Tagging)**: 저장된 데이터가 원본 메모리의 어떤 부분인지 식별하기 위해 사용된다. 태그는 메모리 주소의 일부분을 사용하여 해당 캐시 라인이 어떤 메모리 주소와 연관되어 있는지 확인한다.

VIPT에서 캐시의 인덱스는 가상 주소를 사용해서 결정한다. 이것은 CPU가 메모리를 더 빠르게 접근하게 하고, 주소 변환 과정(TLB 접근)을 거치지 않아도 바로 캐시 인덱스를 얻을 수 있게 한다. 이로 인해 캐시 액세스 지연 시간이 줄어든다. 반면, 캐시 내의 각 라인은 물리적 주소를 사용하여 태깅된다. 이는 캐시가 가상 주소 공간의 변화(예: 다른 프로세스로의 전환)에 영향을 받지 않도록 보장한다. 

### Implementing Protection with Virtual Memory

가상 메모리의 가장 중요한 기능은 여러 개의 프로세스가 하나의 메인 메모리를 공유할 수 있게 하는 것이다. 이때 하나의 프로세스가 다른 프로세스의 주소 공간을 침범하지 않도록 **메모리 보안**을 제공해야 한다. TLB 에서 **write access bit** 는 페이지가 write 되는 것을 방지할 수 있다. 

운영체제가 가상 메모리 시스템에서 보안을 제공하기 위해서는 아래 3가지 기능들을 기본적으로 제공해야 한다. 
1. 프로세스가 동작하는 **2가지 모드**를 제공해야 한다. 여기에는 유저 모드와 커널 모드가 있다.
2. **프로세서 상태 중에서 유저 프로세스가 읽을 수는 있지만 쓸 수는 없는 부분**이 있어야 한다. 여기에는 유저/커널 모드 비트와 페이지 테이블 포인터, TLB가 있다. 이 3가지에 쓰기를 하려면, 운영체제는 커널 모드에서만 동작하는 특별한 명령어를 이용해야 한다.
3. 프로세스가 **유저 모드 <-> 커널 모드로 서로 스위치할 수 있는 메카니즘을 제공**해야 한다. 유저모드에서 커널모드로의 전환은 시스템 콜인 외를 통해 발생한다. (MIPS에서는 syscall 로 구현되어 있다.) 다른 예외들과 동일하게, 시스템 콜이 발생하는 시점의 프로그램 카운터는 exception PC(EPC)에 저장된다. 유저모드로 되돌아가려면 return from exception(ERET) 명령어를 사용하는데, 이것은 유저모드로 바꾸고 EPC에 저장되어 있는 주소로 점프한다. 
이러한 메카니즘과 페이지 테이블을 운영체제의 주소 공간에 저장함으로써, 운영체제는 유저 프로세스가 페이지 테이블을 변경할 것이라는 걱정없이 페이지 테이블을 수정할 수 있다. 

우리는 **프로세스가 다른 프로세스의 데이터를 읽는 것도 방지**하고 싶다. 각 프로세스는 자신만의 가상 주소 공간을 가지고 있다. 따라서 만약 운영체제가 페이지 테이블을 정돈된 상태로 관리하고, 각각의 페이지 테이블이 다른 물리적 페이지로 매핑된다면 하나의 프로세스가 다른 프로세스의 데이터에 접근할 수 없을 것이다. 그리고 운영체제는 각 프로세스가 자신의 페이지 테이블을 수정하지 못하게 해야 한다. 이것들은 모두 **페이지 테이블을 운영체제의 보호된 주소 공간에 저장**함으로써 지킬 수 있다. 

프로세스가 정보를 공유하고 싶을 때, 운영체제의 도움을 받아야 한다. write access bit을 사용해서 단순 읽기만 하고 쓰기는 방지할 수 있다. P1 이 P2가 소유하는 페이지를 읽고 싶을 때, P2는 운영체제에게 P1의 주소 공간에 새로운 페이지 테이블 엔트리를 만들고 P2가 공유하고 싶어하는 페이지 주소를 가리키게 하라고 요청하면 된다. 페이지 테이블은 TLB miss 가 일어났을 때만 참조되기 때문에, 권한을 나타내는 비트는 TLB 와 페이지 테이블 모두에 포함되어 있어야 한다. 

운영체제가 P1 에서 P2 로 **context switch** 를 하고 싶을 때, P2가 P1의 페이지 테이블에 접근하지 못하도록 해야 한다. TLB가 없다면 단순히 페이지 테이블 레지스터가 P2의 페이지 테이블을 가리키도록 수정하면 된다. TLB가 있다면, P1에 해당하는 TLB 엔트리를 비워야 한다. 하지만 context switch가 빈번하게 일어나면, 매번 TLB 엔트리를 비우는 것은 비효율적이다. 따라서 매번 TLB를 비우지 않게끔, 가상 주소 공간에 **process identifier** 를 추가할 수 있다. 이 필드는 현재 동작하는 프로세스가 누구인지 나타낸다. process identifier는 TLB의 tag 부분에 합쳐지고, page number 과 process identifier 둘 다 일치해야지만 TLB hit 가 발생한다. 

### Handling TLB Misses and Page Faults

TLB miss는 TLB에 있는 엔트리가 가상 주소와 일치하지 않을 때 발생한다. TLB miss가 일어날 수 있는 2가지 원인은 아래와 같다.

1. 페이지가 메모리에 존재하고, 따라서 TLB 엔트리만 만들면 된다.
2. 페이지가 메모리에 없고, 따라서 운영체제에게 제어권을 넘기고 page fault 를 처리해야 한다.

MIPS에서는 TLB miss 를 소프트웨어에서 처리한다. 메모리에서부터 페이지 테이블 엔트리를 가져오고, TLB miss를 야기했던 명령어를 재실행한다. 재실행하면, TLB hit가 발생할 것이다. 페이지 테이블 엔트리가 페이지가 메모리에 없다는 것을 나타내면, page fault 예외가 발생할 것이다.

TLB miss 또는 page fault 를 다루는 것은 **예외(exception) 메커니즘**을 사용한다. 예외 메커니즘은 현재 실행 중인 프로세스를 인터럽트하고, 운영체제로 제어권을 넘기고, 예외를 처리한 후, 인터럽트된 프로세스를 다시 실행하는 과정이다. page fault 가 처리된 후에 명령어를 재실행하려면, page fault 를 일으켰던 명령어의 프로그램 카운터(PC)가 저장되어야 한다. **exception program counter(EPC)** 가 이 값을 저장한다. 

또한, TLB miss 또는 page fault 예외는 메모리 접근이 일어나는 클럭 사이클의 끝에서 확인되어야 한다. 그래야지만 다음 클럭 사이클에서 다음 명령어를 실행하는 대신 예외 처리를 할 수 있기 때문이다. 만약 page fault 가 현재 클럭 사이클에서 확인되지 않으면, load 명령어로 레지스터에 쓰기를 수행할 수 있다. 이렇게 되면 명령어를 재실행 했을 때 엄청난 문제가 될 것이다. 예를 들어, 명령어 `lw $1,0($1)` 을 보면, 컴퓨터가 write pipeline stage 가 발생하기 전에 방지해야 한다. 왜냐하면 이것을 실행하면 원래 $1 에 있던 내용물이 사라지기 때문이다. 

운영체제가 exception handler를 시작하고 프로세스의 상태를 저장하기 전에, 운영체제는 특히 취약하다. 예를 들어, 이 시점에 다른 예외가 발생한다면 컨트롤 유닛은 EPC 를 덮어쓸 것이다. 이렇게 되면 원래 예외를 발생시켰던 명령어로 되돌아 갈 수 없다. 이것을 방지하기 위해 **예외를 disable 하고 enable** 할 수 있게 해준다. 예외가 처음 발생하면, 프로세서는 다른 예외들을 모두 **disable** 하는 비트를 켠다. 이것은 커널 모드로 전환하는 비트를 킴과 동시에 일어난다. 그 후 운영체제는 EPC에 상태를 저장한다.  그 다음 운영체제는 다시 예외를 **enable** 한다.

<img width="603" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/3be2ef95-0cbe-4de9-a97e-6e57f3a0274c">

운영체제가 page fault 를 일으킨 가상 주소를 알면, 아래 3가지 단계를 수행한다. 
1. 페이지 테이블 엔트리를 검색해서 디스크에서 페이지의 주소를 찾는다.
2. 교체할 물리적 페이지를 찾는다. 만약 선택된 페이지가 dirty 하다면, 새로운 페이지를 교체하기 전에 이 페이지를 디스크에 써야 한다. 
3. 디스크로부터 요청된 페이지를 가져오기 위해 읽기 작업을 시작한다. (이 단계는 수백만 클럭 사이클이 걸린다.) 

디스크로부터 페이지를 읽어오는 작업이 끝나면, 운영체제는 이제 page fault 를 발생시켰던 명령어의 상태를 복구하고, 다시 실행할 수 있다. 이 과정에서 유저 모드로 돌아오고, 프로그램 카운터를 복구한다. 유저 프로세스는 예외가 일어난 명령어를 다시 실행하고, 요청한 페이지를 찾고 실행을 계속한다. 

만약 명령어로 인한 page fault 가 아니라 **데이터 접근으로 인해 page fault 가 발생**하면, 이것은 더 다루기 까다롭다. 
1. 명령어 page fault 와 달리, 명령어를 실행하는 도중에 발생한다. 
2. 예외를 처리하기 전까지 명령어는 끝날 수 없다.
3. 예외를 처리한 후에, 명령어는 아무 일도 일어나지 않은 것처럼 재실행되어야 한다. 

명령어를 **restartable** 하게 만드는 것은 MIPS와 같은 구조에서는 상대적으로 쉽다. 왜냐하면 각 명령어는 오직 하나의 데이터 아이템을 쓰고, 이 쓰기 작업도 명령어 사이클의 끝에서 일어난다. 따라서 명령어가 쓰기 작업을 하는 것을 막고 명령어를 재실행하면 된다. 

### Summary

**가상 메모리**는 메인 메모리와 세컨더리 메모리 사이의 캐싱을 담당하는 메모리 계층구조의 레벨이다. 가상 메모리는 프로그램이 메인 메모리의 크기 한계를 넘어서 주소 공간을 확장할 수 있게 한다. 더 중요하게, 가상 메모리는 여러 개의 활성화된 프로세스가 메인 메모리를 공유할 수 있게 한다. 

메인 메모리와 디스크 사이를 관리하는 것은 page fault 의 큰 비용 때문에 어렵다. 따라서 **miss rate 를 줄이기** 위해 여러 가지의 기술이 사용된다.
1. 페이지의 크기를 키워서, 공간 지역성을 최대한 활용한다. 
2. 가상 주소와 물리 주소의 매핑을 fully associativite하게 만들어서 가상 페이지가 메인 메모리 중 어느 공간이든 위치될 수 있게 한다. 
3. 운영체제는 LRU와 reference bit를 사용해서 교체할 페이지를 결정한다. 

세컨더리 메모리로의 write는 매우 비싸기 때문에 가상 메모리는 **write-back** 스킴을 이용하고 페이지가 바뀌었는지 여부를 reference bit를 이용해 추적한다. 따라서 바뀌지 않는 페이지는 write 하지 않는다. 

가상 메모리 메카니즘은 프로세스가 사용하는 가상 주소에서 메인 메모리에 접근할 때 사용하는 물리 주소로의 **번역**을 제공한다. 이러한 번역은 메인 메모리를 공유할 때 보호만 제공하는 것이 아니라, 메모리 할당을 간편하게 한다는 장점도 있다. 프로세스가 서로에게서 보호된다는 것을 보장하기 위해 운영체제만 주소 번역을 바꿀 수 있도록 한다. 이것을 위해 유저 프로그램이 페이지 테이블을 바꿀 수 없게 한다. 

만약 프로세서가 메모리에 있는 페이지 테이블을 매번 이용해야 한다면 성능이 매우 저하될 것이다. 따라서 TLB는 페이지 테이블을 위한 캐시로 동작한다. 주소들은 TLB에 있는 엔트리들을 이용해서 가상 주소에서 물리주소로 번역된다. 

## Reference
- Computer Organization and Design, 5th edition, Ch.5 Memory Hierarchy
