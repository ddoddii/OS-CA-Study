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