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
    - [Out-of-Order Execution](#out-of-order-execution)
    - [Multi-FU Pipeline](#multi-fu-pipeline)
    - [Longer Latency Pipelines](#longer-latency-pipelines)
  - [2. Basic Compiler Techniques for Exposing ILP](#2-basic-compiler-techniques-for-exposing-ilp)
    - [Basic Pipeline Scheduling and Loop Unrolling](#basic-pipeline-scheduling-and-loop-unrolling)
      - [Loop Unrolling](#loop-unrolling)
      - [Loop Unrolling \& Pipeline Scheduling](#loop-unrolling--pipeline-scheduling)
      - [Software Pipelining](#software-pipelining)
      - [Loop Unrolling \& Software Pipelining](#loop-unrolling--software-pipelining)
  - [3. Multiple Issue and Static Scheduling](#3-multiple-issue-and-static-scheduling)
  - [4. Dynamic Instruction Scheduling \& Scorboarding](#4-dynamic-instruction-scheduling--scorboarding)
    - [Out-of-Order Execution and Hazards](#out-of-order-execution-and-hazards)
    - [Dynamic Scheduling with a Scorboard](#dynamic-scheduling-with-a-scorboard)
      - [Four Stages of Scoreboard Control](#four-stages-of-scoreboard-control)
      - [Implementation of the Scoreboard](#implementation-of-the-scoreboard)
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

```
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

    j가 i가 read 하기 전에 write 를 하려고 시도할 때 발생한다. 이때 i는 잘못된 값(j가 write를 해서 업데이트 된 값)을 read 할 수 있다. 


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

```
add x2,x3,x4
beq x2,x0,L1
ld x1,0(x2)
L1:
```
위의 코드에서는, x2와 관련된 data dependence 를 유지하지 않으면 프로그램의 실행 결과가 달라지는 것을 확인할 수 있다. 만약 ld 을 beq 이전으로 옮기면, 메모리 보호 관련 예외가 발생할 수 있다. beq 와 ld 사이에는 data dependence 는 없고, 오로지 control dependence 만 존재한다. 

**data flow**는 실제로 명령어 사이의 데이터 값의 흐름이다. 브랜치는 이 흐름을 다이나믹하게 만드는데, 이것은 주어진 명령어의 데이터 소스가 여러 곳에서 오는 것을 허용하기 때문이다. 

```
    add x1,x2,x3
    beq x4,x0,L
    sub x1,x5,x6
L : ...
    or x7,x1,x8
```

위의 코드에서, or 명령어에 의해 사용되는 x1 값은 브랜치가 테이큰 되었는지 안되었는지에 의존한다. or 명령어는 add 와 sub 둘다에 data-dependent 하다. 그러나 data dependence 순서를 지키는 것(add를 or 전에 실행하거나 sub 를 or 전에 실행하는 것)으로는 부족한데, 왜냐면 or 이 add 의 결과를 쓰느냐 아니면 sub까지의 결과를 쓰느냐는 beq 브랜치 명령어에 달려있기 때문이다. 따라서 올바른 실행은 data dependence 와 control dependence 둘 다 고려해야 한다. 이것을 data flow 를 보존한다고 한다. 

가끔 control dependence를 위반하는 것이 exception behavior 과 data flow 에 영향을 주지 않을 수 도 있다. 

```
    add x1,x2,x3
    beq x12,x0,skip
    sub x4,x5,x6
    add x5,x4,x9
skip : or x7,x8,x9
```

위의 코드에서 x4가 skip 명령어 이후에는 안 쓰인다는 것을 미리 알고 있다고 하자. 그렇다면 skip 이후에 x4는 dead 이므로, x4 를 브랜치 전에 변경하는 것은 data flow 에 영향을 주지 않는다. 

### Out-of-Order Execution

프로그램 실행 순서를 지키지 않아도 될 때는, Out-of-Order Execution을 통해 더 적은 CPI 를 달성할 수 있을 때거나, Multi-FP pipeline 인 경우 이다. 

### Multi-FU Pipeline

Multi-FU(멀티 기능 유닛) 파이프라인은 컴퓨터 아키텍처에서 여러 기능 유닛을 포함하여 다양한 종류의 명령을 동시에 실행할 수 있는 파이프라인이다. 

<img width="345" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/488cccb5-4d30-4af4-9c86-bf48c0e92513">

- **파이프라인 단계**
  - IF (Instruction Fetch): 명령어를 메모리에서 가져온다.
  - ID (Instruction Decode): 명령어를 해독하고 필요한 데이터를 준비한다.
  - EX (Execute): 명령어를 실행한다. 이 단계에서는 여러 종류의 기능 유닛이 사용된다.
  - MEM (Memory): 메모리 접근이 필요한 명령어를 처리한다.
  - WB (Write Back): 실행 결과를 레지스터 파일에 쓴다.

- **EX 단계**
  - EX(실행) 단계는 여러 번 반복될 수 있으며, 이는 명령어의 종류에 따라 다르다. EX 단계에서 다양한 기능 유닛을 사용한다:
    - 정수 유닛: 정수 연산을 처리한다.
    - FP/정수 곱셈 유닛: 부동 소수점과 정수 곱셈 연산을 처리한다.
    - FP 덧셈 유닛: 부동 소수점 덧셈 연산을 처리한다.
    - FP/정수 나눗셈 유닛: 부동 소수점과 정수 나눗셈 연산을 처리한다.

EX 단계는 여러 번 반복될 수 있으며, 반복 횟수는 연산 종류에 따라 다르다. 부동 소수점 연산을 위해 여러 기능 유닛이 있을 수 있다. 한 기능 유닛이 사용하는 명령어가 완료될 때까지 다른 명령어는 그 유닛을 사용할 수 없다. 이는 해당 유닛을 사용하는 명령어가 연달아 실행될 수 없음을 의미한다. 만약 명령어가 EX 단계로 진행할 수 없으면, 해당 명령어 뒤의 모든 파이프라인이 멈추게 되는 스톨이 발생한다. 즉, EX 단계에서 병목현상이 발생하면 전체 파이프라인이 영향을 받는다.

Multi-FU 파이프라인에서 **EX 단계의 각 기능 유닛들이 파이프라인화** 될 수도 있다. 즉, 각 기능 유닛이 독립적으로 작동하며, 명령어를 동시에 여러 단계에서 처리할 수 있다.

<img width="417" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/3860f92f-f09d-470d-b00d-1e2222958bc9">

위의 구조를 적용했을 떄, 동시에 처리 가능한 작업 수:

- 최대 4개의 부동 소수점 덧셈(FP adds)을 처리할 수 있다.
- 최대 7개의 부동 소수점/정수 곱셈(FP/int multiplies)을 처리할 수 있다.
- 최대 1개의 부동 소수점 나눗셈(FP divide)을 처리할 수 있다.


나눗셈 유닛(divider)이 다른 유닛들에 비해 훨씬 오래 걸리는 경우, 나눗셈 명령어를 기다리느라 다른 명령어들도 대기해야 하는 상황이 발생할 수 있다. 예를 들어, 나눗셈 명령어는 25 사이클이 걸리기 때문에, 이 명령어를 기다리는 동안 다른 명령어들이 대기하게 되어 파이프라인이 정체(stall)될 수 있다. 이로 인해 사이클 당 명령어 수(CPI, Cycles Per Instruction)가 높아지며, 이는 성능 저하로 이어진다.

이를 해결하기 위해, 명령어의 순서를 변경하여 파이프라인 스톨을 최소화할 수 있다. 이를 통해 처리량(throughput)을 높일 수 있다. 또한 여러 개의 나눗셈 유닛(divider)을 사용하여 동시에 여러 나눗셈 연산을 처리할 수 있도록 설계할 수 있다.

### Longer Latency Pipelines

더 긴 지연 시간을 가지는 파이프라인과 관련된 문제점들은 아래와 같다.

1. Structural Hazards 

    나눗셈 유닛이 파이프라인되지 않은 경우 구조적 해저드가 발생할 수 있다. 이는 한 기능 유닛을 여러 명령어가 동시에 사용하려고 할 때 발생하는 자원 충돌 문제이다. 예를 들어, 나눗셈 유닛이 하나밖에 없고 이 유닛이 여러 사이클 동안 바쁘다면, 다른 나눗셈 명령어는 기다려야 한다.

2. Number of Register Writes in a Cycle

    하나의 사이클에서 필요한 레지스터 쓰기 횟수가 1보다 클 수 있다. 여러 명령어가 동시에 실행되어 결과를 레지스터에 쓰려고 하면, 레지스터 파일의 쓰기 포트가 부족하여 병목현상이 발생할 수 있다.

3. WAW Hazards (Write After Write 해저드)

    명령어들이 프로그램 순서대로 WB(Write Back) 단계에 도달하지 않기 때문에 WAW 해저드가 발생할 수 있다. 이는 동일한 레지스터에 연속적으로 쓰기를 시도할 때 발생하는 문제이다.
    두 명령어가 같은 레지스터에 결과를 쓰려고 할 때, 먼저 시작한 명령어가 나중에 시작한 명령어보다 나중에 완료되면 데이터 불일치가 발생할 수 있다.

4. Instruction Completion Order

    명령어들이 발행된 순서와 다른 순서로 완료될 수 있어 예외 처리 시 문제가 발생할 수 있다.
    명령어 A가 명령어 B보다 먼저 발행되었지만 B가 먼저 완료되면, 예외 상황에서 올바르지 않은 순서로 처리되어 오류가 발생할 수 있다.

5. RAW Hazards 

    연산의 지연 시간이 길어짐에 따라 RAW 해저드로 인한 스톨이 더 빈번해질 수 있다. 이는 이전 명령어의 결과를 읽어야 하는 다음 명령어가 그 결과를 기다리면서 발생하는 문제이다.
    명령어 A가 결과를 쓰기 전에 명령어 B가 그 결과를 읽으려고 하면, B는 A의 완료를 기다려야 하므로 파이프라인이 멈추게 된다.


## 2. Basic Compiler Techniques for Exposing ILP

### Basic Pipeline Scheduling and Loop Unrolling

파이프라인을 계속 바쁘게 만드려면, 연관 되어 있지 않는 명령어들을 찾아서 병렬적으로 실행시켜야 한다. 파이프라인 스톨을 피하려면, 파이프라인 스케쥴링을 해야 하는데, 이것은 의존성 있는 명령어를 원본 명령어의 파이프라인 지연 시간에 따라 분리하는 것이다.

고수준 언어의 코드와 어셈블리로 바꾼 코드를 보자.

```cpp
for (i = 999; i >= 0; i = i + 1)
  x[i] = x[i] + s;
```

```
Loop: fld   f0, 0(x1)        // f0 <- 메모리에서 x1 레지스터가 가리키는 주소의 값을 로드
      fadd.d f4, f0, f2     // f4 <- f0 + f2 (부동 소수점 덧셈)
      fsd   f4, 0(x1)       // 메모리에 f4 값을 저장
      addi  x1, x1, -8      // x1 <- x1 - 8 (다음 요소로 이동)
      bne   x1, x2, Loop    // x1과 x2가 같지 않으면 Loop로 분기
```

아래의 표는  특정 명령어가 결과를 생성한 후 그 결과를 사용하는 명령어까지의 지연 시간을 나타낸다.

| Instruction producing result | Instruction using result | Latency in clock cycles |
|------------------------------|--------------------------|-------------------------|
| FP ALU op                    | Another FP ALU op        | 3                       |
| FP ALU op                    | Store double             | 2                       |
| Load double                  | FP ALU op                | 1                       |
| Load double                  | Store double             | 0                       |

- FP ALU op (부동 소수점 연산):
  - 다른 부동 소수점 연산이 결과를 사용하려면 3 사이클이 필요하다.
  - 결과를 저장하려면 2 사이클이 필요하다.
- Load double (메모리에서 부동 소수점 값 로드):
  - 부동 소수점 연산이 결과를 사용하려면 1 사이클이 필요하다.
  - 결과를 저장하려면 지연 시간이 없다.

명령어 사이의 이러한 지연 시간을 고려하여, **의존성이 있는 명령어 사이에 다른 명령어들을 삽입**함으로써 파이프라인 스톨을 방지하고 효율성을 극대화할 수 있다. 예를 들어, fld 명령어와 fadd.d 명령어 사이에 다른 독립적인 명령어를 삽입하여 fld의 결과를 기다리는 동안 파이프라인이 멈추지 않도록 할 수 있다.

만약 스케쥴링이 없다면, 루프는 왼쪽과 같이 8번의 사이클이 걸릴 것이다. 그러나 addi 를 옮겨서 **스케쥴링**하면, 총 **7 사이클**로 줄어든다. `addi` 를 `fld` 다음으로 옮겨서, `fld` 다음의 stall 을 안 넣을 수 있다.  

<img width="1103" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/f1de2af2-aa26-49d0-bbe0-2b1b94f28e32">

#### Loop Unrolling

하지만 위의 예시에서, 실제 배열에 대한 연산을 하는 것은 7개의 사이클 중 3개 사이클(load, add, store) 뿐이다. 

1. 데이터 로드(fld f0, 0(x1)) - 1 클럭 사이클:

    첫 번째 명령어는 fld f0, 0(x1)로, 이는 메모리 주소 x1에서 데이터를 읽어와 f0 레지스터에 저장하는 명령어이다. 이 작업은 1개의 클럭 사이클이 소요된다.

2. 데이터 계산(fadd.d f4, f0, f2) - 1 클럭 사이클:

    세 번째 명령어는 fadd.d f4, f0, f2로, 이는 f0 레지스터와 f2 레지스터의 값을 더해 f4 레지스터에 저장하는 부동 소수점 덧셈 명령어이다. 이 작업 역시 1개의 클럭 사이클이 소요된다.

3. 데이터 저장(fsd f4, 8(x1)) - 1 클럭 사이클:

    섯 번째 명령어는 fsd f4, 8(x1)로, 이는 f4 레지스터의 데이터를 메모리 주소 x1에 저장하는 명령어이다. 이 작업도 1개의 클럭 사이클이 소요된다.


나머지 명령어 - `addi x1, x1, -8` 는 배열을 다음 요소로 이동시키기 위한 명령어이고, `bne x1, x2, loop` 는 x1 과 x2 가 같지 않으면 루프의 시작으로 분기하는 명령어이다. 나머지 명령어 2개와 stall 2개에 필요한 4개의 클럭 사이클을 없애려면, 추가적인 연산이 필요하다. 이것을 **loop unrolling** 이라고 한다. Loop unrolling 은 단순하게 루프 바디를 복제해서 여러번 작성하는 방식이다. 

만약 루프의 이터레이션 수가 4라고 가정하면, 루프를 factor 4로 unroll 할 수 있다. 이때 각 이터레이션 당 addi & bne 를 생략하고, 모든 이터레이션이 끝나면 한 번만 실행할 수 있다. 따라서 아래의 코드에서 4번의 이터레이션을 위해 26번 사이클이 필요하고, 이것은 배열 요소 당 **6.5 사이클**이 걸린다. 그러나 이 방법으로는, 코드의 길이가 매우 길어져서 레지스터에 부담이 간다(**register pressure**).

<img width="523" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/54db6e54-9b7c-44c5-a225-c31179261f97">

#### Loop Unrolling & Pipeline Scheduling

그러나 문제는, 실제 프로그램에서 우리는 루프의 상한선을 알지 못한다는 것이다. n이라고 가정했을 때, 우리는 루프를 unroll 해서 바디의 k개 복사를 만들고 싶다고 가정해 보자. 단일 unrolled loop 대신에, 우리는 두개의 연속적인 루프를 생성한다. 첫번째는 (n mod k) 번 실행되며, 본체는 원래의 루프이다. 두번째는 unrolled loop 본체로, (n/k)번 반복하는 외부 루프에 둘러싸여 있다. 이 테크닉을 **strip mining** 이라고 한다. 아주 큰 값 n에 대해서, 대부분의 실행 시간은 루프 본체에서 소비될 것이다. 

이전 예제에서, 루프 펼치기(unrolling)는 오버헤드 명령어를 제거하여 루프의 성능을 향상시킨다. 하지만, 코드 크기는 상당히 증가한다. 파이프라인이 앞서 설명된 대로 **스케줄링** 될 때, 펼쳐진 루프는 어떻게 성능을 발휘할까? 

스케줄링은 **명령어들을 재배열하여 데이터 의존성을 최소화**하고, **병렬 실행을 최대화**하는 과정이다. 각 명령어의 실행 순서를 최적화하여 가능한 한 많은 명령어가 동시에 실행되도록 한다. 예를 들어, 특정 명령어가 완료되기를 기다리지 않고 다른 명령어를 먼저 실행함으로써 스톨을 피할 수 있다. 또한 **register renaming** 을 통해, 서로 다른 루프 반복에서 사용하는 레지스터를 다르게 하여 데이터 의존성을 제거하고 병렬 처리를 가능하게 한다. 

아래의 코드에서, 총 14 클럭 사이클이 걸렸고 이것은 각 배열 원소 당 14/4 = **3.5 사이클**이 걸렸음을 의미한다. 

<img width="333" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/fbee618e-e74c-4312-9458-bebac026feab">


#### Software Pipelining

소프트웨어 파이프라이닝은 루프의 성능을 최적화하기 위해 루프의 명령어들을 재구성하는 기법이다. 루프의 각 반복(iteration)을 재구성하여, 소프트웨어 파이프라이닝된 코드의 각 반복이 원래 루프의 다양한 반복에서 선택된 명령어들로 구성되도록 한다. 이 과정에서 데이터 의존성(dependent computations)을 분리하여, 각 반복 내에서 데이터 의존성을 최소화하고 병렬 처리가 가능하도록 한다.

<img width="442" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/ff9168eb-b2b5-4a31-9935-e220830a32fe">

예를 들어, 그림에서 Iteration 0, 1, 2, 3, 4의 명령어들이 서로 겹쳐진 형태로 실행된다. 이는 각 반복이 독립적으로 실행되지 않고, 파이프라인 방식으로 동시에 여러 반복이 진행됨을 의미한다.

예시 코드를 보자.

<img width="286" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/aca8ffda-be26-4286-995c-056295272123">

이 코드를 소프트웨어 파이프라이닝을 통해 이 명령어들을 재배열하여, 아래의 최적화된 루프를 구성할 수 있다.

<img width="403" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/51caca58-31eb-42b5-8eba-7062ba5c8fab">

최적화된 루프는 각 반복에서 명령어들이 재배열되어 있다. 예를 들어, fld, addi, fadd.d, fsd 명령어들이 서로 다른 반복에서 선택되어 하나의 반복으로 결합된다. 이 과정에서 데이터 의존성으로 인한 스톨이 제거된다. 이렇게 최적화된 루프는 배열 요소 당 **5 사이클**이 걸린다. 

#### Loop Unrolling & Software Pipelining

**Loop Unrolling** 의 목적은, 루프 제어 오버헤드를 줄이는 것이다. 루프의 각 반복에서 반복 조건을 검사하고 인덱스를 업데이트하는 오버헤드를 줄임으로써 사이클 수 를 줄일 수 있다. 

**Software Pipelining** 의 목적은, 루프가 최대 속도로 실행되지 않는 시간을 줄이고, 루프의 시작과 끝에서만 이러한 상태가 발생하도록 하는 것이다. 반복 간의 명령어를 재배열하여 데이터 의존성으로 인한 스톨을 줄이고, 가능한 한 많은 명령어가 병렬로 실행될 수 있도록 한다.

두 기법을 결합하여, 루프 언롤링과 소프트웨어 파이프라이닝의 장점을 동시에 활용할 수 있다. 예를 들어, 루프 언롤링을 통해 오버헤드를 줄이면서, 소프트웨어 파이프라이닝을 통해 명령어 병렬성을 극대화할 수 있다.

<img width="448" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/2c3bc477-3477-49dd-bf6b-f6e1670d4634">

## 3. Multiple Issue and Static Scheduling

CPI 를 1보다 줄이려면, 한 사이클에 여러 개의 명령어를 실행해야 한다. 이때 사용할 수 있는 방법은, 1) superscalar processors, 2) VLIW(Very Long Instruction Word) processors 이다. 

VLIW는 여러 개의 연산을 하나의 명령어로 압축한다. 그리고 여러 개의 독립적인 Functional unit 들을 사용한다. 고정된 수의 명령어를 한 번에 발행하며, 각 명령어는 하나의 큰 명령어로 포맷된다. 컴파일러가 명령어 병렬성을 명시적으로 결정하여 스케줄을 설정한다. 정적으로 스케줄된 구조로 인해 하드웨어가 단순해지지만, 컴파일러의 최적화 능력이 매우 중요하다.

이전 예시의 `x[i] = x[i] + s` 루프를 unrolling factor 7 로 언롤링하면, 9 사이클에 7번에 반복을 실행하므로 각 배열 요소 당 1.29 사이클이 소요된다. 루프 본체의 명령어들이 메모리 참조, 부동 소수점 연산, 정수 연산 및 분기 등으로 분리되어 여러 개의 함수 유닛(functional units)을 사용한다. 따라서 각 명령어는 동시에 실행될 수 있다.

<img width="665" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/cc99a089-29ed-4fa6-8c99-1f8ae6f0bd0c">


## 4. Dynamic Instruction Scheduling & Scorboarding

다이나믹 스케쥴링은 하드웨어가 명령어 실행 순서를 조정해서 stall 을 최소화한다. 다이나믹 스케쥴링의 핵심 아이디어는, 1) stall 이전의 명령어가 미리 실행되는 것(out-of-order), 2) hazard를 피하기 위한 register renaming 이다.

<img width="601" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/72d1ee94-3997-466d-ac80-7a8725a985ca">

### Out-of-Order Execution and Hazards

우리의 목적인 명령어가 데이터 피연산자가 준비되자마자 실행을 시작하는 것이다. 이때 hazard 가 없도록 주의해야 한다.

<img width="459" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/d674985f-5957-4fde-9f7e-f5d04a633948">


<img width="843" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/c19fa3b7-adf0-4304-bf3f-83f26dbafa54">

위의 예시에서 2가지의 Hazard 가 발생할 수 있다.

- RAW hazard 
  - MULTD 명령어의 결과를 DIVD 명령어가 읽어야 하므로, MULTD 명령어가 끝날 때까지 DIVD 명령어가 스톨된다.
- WAR hazard
  - DIVD 명령어가 F6에 결과를 기록하기 전에 ADDD 명령어가 F6을 읽어야 하므로, ADDD 명령어가 먼저 실행된다.


Out-of-Order 에는 2가지 실행 방법이 있다 : **Scoreboarding**, **Tomasulo's Algorithm**.

- **Scoreboarding**
  - 스코어보딩은 명령어가 실행될 수 있는 충분한 자원(예: 함수 유닛)이 있을 때, 그리고 데이터 의존성이 없을 때, 명령어를 비순차적으로 실행할 수 있게 하는 기법이다. 주로 WAR hazard 를 처리한다.
- **Tomasulo's Algorithm**
  - 토마슐로 알고리즘은 비순차 실행을 지원하며, 보다 복잡한 데이터 의존성 관리와 추측 실행(speculative execution)을 포함하는 고급 기법이다. WAR 및 WAW hazard 를 처리한다. 명령어의 추측 실행(speculations)을 통해 성능을 극대화한다.

### Dynamic Scheduling with a Scorboard

스코어보딩에서는 ID 스테이지를 2 단계로 나눈다.

- Issue : 명령어를 해석하고, 구조적 해저드를 체크한다.
- Read operands : 데이터 해저드가 없을 때까지 기다리고, 오퍼랜드를 읽는다.

스코어보드는 명령어를 이슈하고 실행하는 것에 총 책임을 진다. 명령어가 언제 오퍼랜드를 읽고, 실행을 시작하고, 결과를 쓰는 것을 결정한다.

아래는 스코어보드 아키텍쳐이다. 


<img width="772" alt="image" src="https://github.com/ddoddii/OS-CA-Study/assets/95014836/7b0d8ba8-514c-4a46-bf89-714938d5e1ff">


#### Four Stages of Scoreboard Control

1. Issue

- 명령어를 디코딩하고, 구조적 해저드를 확인한다.
- 해저드를 검사하기 위해, 명령어는 프로그램 순서대로 발행된다.
- 구조적 해저드가 있는 경우 발행할 수 없다. (e.g 사용가능한 FU가 없는 경우)
- 명령어가 이전에 발행된 명령어의 결과에 의존하는 경우 발행을 스톨한다. 이것은 WAW 해저드를 방지한다.

2. Read operands

- 데이터 해저드가 없을 때까지 기다린 후 오퍼랜드을 읽는다.
- 참된 데이터 의존성(e.g RAW 해저드)은 이전에 발행된 명령어가 소스 오퍼랜드를 기록할 때까지 기다림으로써 해결된다.
- 스코어보딩에서는 데이터 전달(forwarding)을 사용하지 않는다. 모든 데이터는 레지스터에 기록된 후에야 사용할 수 있다.


3. Execute

- 함수 유닛(Functional Unit)이 오퍼랜드를 받은 후 실행을 시작한다.
- 연산이 완료되면 스코어보드에 완료 사실을 알린다.


4. Write Result

- 연산 결과를 레지스터 파일에 기록한다.
- 이전에 발행된 명령어와 안티 의존성(WAR 해저드)이 없을 때까지 결과를 레지스터 파일에 기록하지 않는다.

```
div.d  f0, f2, f4
add.d  f10, f0, f8
sub.d  f8, f8, f14
```

- `div.d` : 이 명령어는 f0 레지스터에 결과를 쓴다.
- `add.d` : 이 명령어는 f8, f0 레지스터 값을 사용하고, f10 레지스터에 값을 쓴다.
- `sub.d` : 이 명령어는 f8 레지스터의 값을 사용하고, 또 f8 레지스터에 결과를 쓴다.

`add.d` 명령어가 f8 레지스터의 값을 사용하고, `sub.d` 명령어가 f8 레지스터에 쓰기를 시도할 때 안티 의존성(**WAR 해저드**)이 발생한다. 이 경우, `sub.d` 명령어는 f8 레지스터의 값을 읽기 전에 `add.d` 명령어가 f8 레지스터의 값을 사용할 수 없다. 따라서 `add.d` 명령어가 실행 완료될 때까지 `sub.d` 명령어는 스톨된다.

#### Implementation of the Scoreboard

- **Instruction status** : 명령어가 네 단계(Issue, Read operands, Execute, Write result) 중 어느 단계에 있는지를 나타낸다.
- **Functional unit status** : 각 함수 유닛(FU)의 상태를 나타낸다.
  - Busy: 함수 유닛이 바쁜 상태인지 아닌지를 나타낸다.
  - Op: 함수 유닛에서 수행할 연산을 나타낸다.
  - Fi: 결과를 저장할 목적 레지스터 번호이다.
  - Fj, Fk: 소스 레지스터 번호들이다. 이 레지스터들에서 데이터를 읽는다.
  - Qj, Qk: 소스 레지스터 값을 생성하는 함수 유닛들이다. 이는 데이터 의존성을 추적하는 데 사용된다.
  - Rj, Rk: 소스 레지스터 값이 준비되었음을 나타내는 플래그이다.
- **Register result status** : 각 레지스터에 어떤 함수 유닛이 값을 쓸 것인지 나타낸다. 대기 중인 명령어가 없으면 빈 칸으로 둔다.




## Reference

- Computer Architecture, A Quantitative Approach, 6th edition, ch.3 Instruction-Level Parallelism and its exploitation