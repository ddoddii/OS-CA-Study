# Instruction-Level Parallelism

<details>
<summary>
<img src="https://raw.githubusercontent.com/Tarikul-Islam-Anik/Animated-Fluent-Emojis/master/Emojis/Objects/Bookmark%20Tabs.png" alt="Bookmark Tabs" width="25" height="25" /> Table of Contents </summary>

- [Instruction-Level Parallelism](#instruction-level-parallelism)
  - [1. Instruction-Level Parallelism : Concepts and Challenges](#1-instruction-level-parallelism--concepts-and-challenges)
    - [What is ILP?](#what-is-ilp)
    - [Data Dependences and Hazards](#data-dependences-and-hazards)
      - [Data dependences](#data-dependences)
      - [Name Dependences](#name-dependences)
      - [Data Hazards](#data-hazards)
    - [Control Dependences](#control-dependences)
  - [2. Basic Compiler Techniques for Exposing ILP](#2-basic-compiler-techniques-for-exposing-ilp)
  - [Reference](#reference)


</details>


## 1. Instruction-Level Parallelism : Concepts and Challenges

프로세서의 성능은 아래와 같이 계산된다.

$$Throughput = \frac{Program}{Instructions} \times \frac{Instruction}{Cycles} \times \frac{Cycle}{Seconds}$$


CPI(cycles per instruction) = Cycles / Instruction

Pipeline CPI = Ideal pipeline CPI + Structural stalls + Data hazard stalls + Control stalls

CPI가 낮을수록 성능이 좋은 건데, 그럼 CPI의 최소값은 무엇일까? CPI = 1 이면, 명령어가 1 사이클 안에 실행된다는 의미로, (파이프라인 실행 + I/D 메모리 분리 + 100% 캐시 히트 + 100% 브랜치 분기 예측 + TLB miss 없음 + page fault 없음 ...) 의 조건이 모두 만족되어야 한다. 그러면 CPI 를 1보다도 낮출 수는 없을까? 이것을 가능하게 해주는 것이 **ILP(Instruction-Level Parallelism)** 이다. 

ILP를 구현하는 방법은 2가지이다. (1) 하드웨어에 의존하는 방법, (2) 소프트웨어 기술에 의존해서 컴파일 시점에 병렬화를 구현하는 방법이다. 

||HW|SW|
|--|---|--|
|Out-of-order Execution|Dynamic pipelining scheduling|Static(Compiler-based) pipelining scheduling|
|Speculation| HW-based speculation| SW-based speculation|
|Multiple Issue| Superscalar| VLIW|

### What is ILP?

이번 섹션에서는 명령어 간 병렬화를 구현하는 방법에 대해서 다룬다. 하지만 _basic block_(진입과 끝나는 시점외에는 분기가 없는 순차적인 코드) 상에서는 병렬화할 수 있는 것이 별로 없다. 따라서 성능 효과를 기대하기 위해서는 여러개의 basic block 을 두고 ILP 를 다뤄야 한다. 

ILP를 구현하는 가장 쉬운 방법은 loop 상에서 병렬화를 도입하는 것이다. 이것을 **loop-level parallelism** 이라고 한다. 아래의 코드는 1000개의 원소가 들어있는 2개의 배열을 더하는 코드이다. 

```cpp
for (i=0; i<=999; i++)
    x[i] = x[i] + y[i];
```

루프의 각 이터레이션은 다른 이터레이션과 겹쳐서 실행될 수 있다. 계속해서 이러한 loop-level 병렬화를 instruction-level 병렬화로 바꾸는 방법들에 대해 알아볼 것이다. 기본적으로, 루프 언롤링(loop unrolling)은 컴파일러에 의해 정적으로 수행되거나 하드웨어에 의해 동적으로 수행된다. 

다른 방법은 벡터 프로세서와 GPU에 SIMD 구조를 사용하는 것이다. 이 방법은 data-level 병렬화룰 구현하는데, 데이터를 여러 개로 쪼개서 각각 다른 연산 유닛에 분배해서 병렬적으로 실행한다.  

### Data Dependences and Hazards

하나의 명령어가 다른 명령어와 어떠한 의존관계가 있는지 알아내는 것은 병렬화를 수행하는데 굉장히 중요하다. ILP를 수행하기 위해서는, 어떤 명령어들이 병렬적으로 실행될 수 있는지 파악해야 한다. 

#### Data dependences

의존관계(dependences)에는 3가지 종류가 있다 : **data dependences**, **name dependences**, **control dependences**.

어떠한 명령어 j는 아래의 2가지 조건을 만족하면 다른 명령어 i에게 data-dependent 하다고 한다.

- 명령어 i는 명령어 j가 사용할만한 결과를 반환한다.
- 명령어 j는 명령어 k에 data-dependent 하고, 명령어 k는 명령어 i에 data-dependent하다.(dependence chain)

아래의 RISC-V 코드를 보자. 

```asm
Loop : fld f0,0(x1)      // f0 = array element
       faad.d f4,f0,f2   // add scalar in f2
       fsd f4,0(x1)      // store result
       addi x1,x1,-8     // decrement pointer in 8bytes
       bne x1,x2,Loop    // branch x1!=x2
```

이때 첫번째 줄 `f0` 과 두번째 줄 `f0` 간 data dependences 가 있고, 둘째줄 `f4`와 셋째줄 `f4`도 data dependences 가 있다. 네번째줄 `x1`와 다섯번째줄 `x1` 도 data dependences 가 있다. 

<img width="207" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/b92572f0-3d31-4578-bba1-def9353a4e58">


만약 두개의 명령어가 data-dependent하다면, 두개의 명령어는 순서를 지켜서 실행되어야 하며, 온전히 겹쳐서 동시에 실행될 수 없다. 만약 동시에 실행하면 프로세서는 hazard 또는 stall을 감지하는 pipeline interlocks 를 실행할 것이다. 명령어 시퀀스의 data dependence는 소스 코드 상에서 실행 순서와 관련있다. 

의존관계는 *program*의 영역이다. 반면 주어진 의존관계로 인해 hazard가 발생하거나 hazrd 가 실제로 stall을 발생시키는 것은 *pipeline organization* 의 영역이다. 이 2가지의 차이는 ILP 를 실행하는데 굉장히 중요하다.

data dependence는 3가지를 함축한다: (1) hazard 의 가능성, (2) 결과가 연산되어야 하는 순서, (3) 실제로 얼만큼 병렬화가 될 수 있는지 상한선이다.

data dependence가 ILP의 정도에 제한을 만들 수 있기 때문에, 이러한 제한을 극복하는 것이 최우선 목표이다. 의존관계는 2가지 방법으로 극복할 수 있는데, (1) 의존관계는 유지하되 hazard 는 피하기, (2) 코드를 수정해서 의존관계를 없애기이다. 


#### Name Dependences

**name dependence**는 2개의 명령어가 레지스터 또는 메모리 상에서 같은 위치(*name*)을 사용하지만, 2개 명령어 간 그 name과 관련된 어떠한 데이터 전송도 없을 때 발생한다. 프로그램 순서상 명령어 i가 명령어 j를 앞설 때(i->j), 2가지 유형의 name dependence가 발생할 수 있다.

1. Antidependence - (potential) WAR hazard

    명령어 i,j 간 antidependence는 j가 i가 읽는 메모리 장소에 쓸 때 발생한다. i가 올바른 값을 읽는 것을 보장하기 위해선, 명령어의 순서가 보장되어야 한다. 

    <img width="208" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/a5cc1921-652a-45e4-b071-3b74ecb1cfe8">


2. Output dependence - (potential) WAW hazard

    명령어 i,j 가 레지스터 또는 메모리의 같은 장소에 쓰기를 할 때 발생한다. j가 쓰기한 이후의 값이 최종 값이 되기 위해서, 명령어 간 순서가 보장되어야 한다. 

    <img width="193" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/8869fb2f-76a2-45a4-9fec-7e4eb757b3a1">

name dependence 같은 경우에는, 명령어에 사용되는 name(레지스터 넘버 혹은 메모리 위치)가 바뀌면 명령어들끼리 충돌이 발생하지 않을 수 있다. 이러한 renaming은 레지스터 피연산자에 대해 쉽게 할 수 있다(register renaming).

#### Data Hazards

name 또는 data dependence 가 있고, 명령어 간 충분히 가까워서 명령어의 실행 순서가 중요할 때 hazard 가 존재한다. 이때 명령어의 실행 순서를 지켜야 한다. ILP 의 구현에서 가장 중요한 목표는 프로그램의 결과에 영향을 줄 때만 프로그램의 순서를 지키고, 아니면 병렬화시키는 것이다. 

Data Hazard 는 3가지 종류가 있다. 명령어 i,j가 있고, 순서는 i->j 라고 가정하자.

1. RAW(read after write)

    j가 i가 write 하기 전에 read 를 하려고 시도할 때 발생한다. 이때 j는 업데이트 되기 예전의 값을 읽어올 위험이 있다. 이 hazard는 가장 흔하게 발생하며, true data dependence 에 해당한다. 

2. WAS(write after write)

    j가 i가 write 하기 전에 write 를 하려고 시도할 때 발생한다. write는 잘못된 순서로 실행되어, 최종 결과가 틀리게 저장될 수 있다. 이 hazard 는 output dependence 에 해당한다.

3. WAR(write after read)

    j가 i가 read 하기 전에 write 를 하려고 시도할 때 발생합니다. 이때 i는 잘못된 값(j가 write를 해서 업데이트 된 값)을 read 할 수 있습니다. 


### Control Dependences

Control Dependence는 브랜치 명령어에 따라서 명령어 i가 올바른 순서로 실행되는지를 결정한다. 첫번째 basic block 안에 있는 명령어를 제외하고, 모든 명령어들은 어떠한 브랜치에 control-dependent 하다. 

```cpp
if p1 {
    S1;
}
if p2 {
    S2;
}
```

여기서, S1은 p1에 control-dependent하고, S2는 p2에 control-dependent하다. Control Dependence는 2가지 제약 사항을 뜻한다.

1. 어떤 브랜치에 control-dependent한 명령어는 브랜치 이전으로 옮겨질 수 없다. 
2. 어떤 브랜치에 대해 control-dependent 하지 않은 명령어는 그 브랜치 이후로 옮겨질 수 없다. 

하지만 실행해서는 안되는 명령어를 수행한다고 해도, 프로그램 전체의 결과에 영향을 미치지 않는다면 Control Dependence는 꼭 지켜야 하는 크리티컬 요소가 아니다. 대신에, 프로그램 정확도에 영향을 주는 2가지 요소들은 **exception behavior** 과 **data flow**이다.

**exception behavior**를 보존한다는 것은 명령어 실행 순서를 바꾸어도 프로그램 내에서 예외를 발생시키는 로직을 바꾸지 않는 것이다. 

```asm
add x2,x3,x4
beq x2,x0,L1
ld x1,0(x2)
L1:
```
위의 코드에서는, x2와 관련된 data dependence 를 유지하지 않으면 프로그램의 실행 결과가 달라지는 것을 확인할 수 있다. 만약 ld 을 beq 이전으로 옮기면, 메모리 보호 관련 예외가 발생할 수 있다. beq 와 ld 사이에는 data dependence 는 없고, 오로지 control dependence 만 존재한다. 

**data flow**는 실제로 명령어 사이의 데이터 값의 흐름이다. 브랜치는 이 흐름을 다이나믹하게 만드는데, 이것은 주어진 명령어의 데이터 소스가 여러 곳에서 오는 것을 허용하기 때문이다. 

```asm
    add x1,x2,x3
    beq x4,x0,L
    sub x1,x5,x6
L : ...
    or x7,x1,x8
```
위의 코드에서, or 명령어에 의해 사용되는 x1 값은 브랜치가 테이큰 되었는지 안되었는지에 의존한다. or 명령어는 add 와 sub 둘다에 data-dependent 하다. 그러나 data dependence 순서를 지키는 것(add를 or 전에 실행하거나 sub 를 or 전에 실행하는 것)으로는 부족한데, 왜냐면 or 이 add 의 결과를 쓰느냐 아니면 sub까지의 결과를 쓰느냐는 beq 브랜치 명령어에 달려있기 때문이다. 따라서 올바른 실행은 data dependence 와 control dependence 둘 다 고려해야 한다. 이것을 data flow 를 보존한다고 한다. 

가끔 control dependence를 위반하는 것이 exception behavior 과 data flow 에 영향을 주지 않을 수 도 있다. 

```asm
    add x1,x2,x3
    beq x12,x0,skip
    sub x4,x5,x6
    add x5,x4,x9
skip : or x7,x8,x9
```

위의 코드에서 x4가 skip 명령어 이후에는 안 쓰인다는 것을 미리 알고 있다고 하자. 그렇다면 skip 이후에 x4는 dead 이므로, x4 를 브랜치 전에 변경하는 것은 data flow 에 영향을 주지 않는다. 



## 2. Basic Compiler Techniques for Exposing ILP


## Reference

- Computer Architecture, A Quantitative Approach, 6th edition, ch.3 Instruction-Level Parallelism and its exploitation