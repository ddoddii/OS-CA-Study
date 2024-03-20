## 2.1 Introduction

컴퓨터 언어는 instructions 라고 불리고, 단어들은 instruction set 이라고 한다. 따라서 Instruction Set Architecture 는 Computer Architecture 이다. 컴퓨터는 프로세서(Datapath and Control) + 메모리 + I/O 로 구성되어 있다.

### Instruction Set Architecture(ISA)

ISA란 명령어의 문법(syntax) 과 의미(semantics) 이다. 하드웨어와 소프트웨어를 구분하는 인터페이스에 대한 설명이다. X86/ia32, ARM, MIPS 등 다양한 ISA가 존재한다.

여기서 RISC 는 Reduced Instruction-Set Computer 이고, CISC 는 Complex Instruction-Set Computer 이다.

### Main Concepts

1. 명령어들도 데이터이다. 따라서 바이너리로 인코딩되어서 메모리에 저장되어 있다.
2. Program Counter 는 실행되어야 할 다음 명령어의 위치를 가리킨다.

PC가 가리키는 메모리에 있는 명령어를 가져와서, 그 명령어를 실행한 후, 다시 PC를 업데이트 하는 방식이다.

**Stored-program concept** 은, 메모리에서 프로그램을 저장하고, CPU는 명령어들을 순차적으로 실행하는 컨셉이다. 이것을 폰 노이만 아키텍쳐 라고도 한다.

<img width="283" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6aa4e0d3-5fee-456c-9418-5be73d8c981b">

명령어에는 두가지 컴포넌트가 있다.

- Opcode
  어떤 명령어가 무엇을 하는가를 나타낸다. MIPS에는 적은 수의 명령어 세트가 정의되어 있다.
- Operand
  연산을 할 데이터를 가리킨다(피연산자). 레지스터거나 메모리를 가리킨다. MIPS는 적은 수의 addressing mode 들이 있다. addressing mode란 명령어를 수행할 대상인 메모리 주소를 가리키는 방식이다.

### MIPS R4000 ISA

명령어 카테고리에는, 아래의 종류들이 있다.

- **Arithmetic**
- **Load/Store**
- **Jump and Branch**
- Floating Point :: coprocessor
- Memory management
- Special

**Arithmetic**, **Load/Store**, **Jump and Branch** 모두 32 bits 이고, 형태는 3가지가 있다.

<img width="392" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e19e6076-55a4-4f50-94e3-733f396c5eb0">

## 2.2 Operations of the Computer Hardware

MIPS assembly language

```text
add a, b, c
```

> Design Principle 1 : Simplicity favors regularity

MIPS 에서 하나의 연산은 무조건 3개의 variable을 요구한다. 자연적으로도, 덧셈과 같은 연산도 3개의 피연산자가 필요하다. MIPS는 이를 따름으로써 하드웨어를 간단하게 유지할 수 있다.

## 2.3 Operands of the Computer Hardware

High-Level 언어들과 달리, 산술 명령어의 피연산자들은 제한되어 있다. 그들은 **레지스터** 라는 특별한 하드웨어에서 와야 한다. MIPS 구조에서 레지스터의 사이즈는 32bits 이다. 32bits 는 꽤 자주 등장하기 때문에, MIPS 구조에서 "**word**" 라고 명명했다.

프로그래밍 언어의 변수와 레지스터의 가장 큰 차이는, 레지스터는 개수가 제한되어 있다. MIPS 에는 32개의 레지스터가 있다. 따라서, MIPS 산술 명령어의 3개 피연산자는 각각 32개의 32bits 레지스터에서 와야 한다.

왜 레지스터 개수를 32개로 제한했는지는 나중에 이유를 설명하겠다.

> Design Principle 2 : Smaller is faster

많은 개수의 레지스터가 있다면, 전기 신호가 더 멀리 이동해야 하기 때문에 clock cycle time 을 증가시킬 수 있다. 하지만 "smaller is faster" 는 절대적인 것이 아니다. 31개의 레지스터가 32개의 레지스터보다 빠른 것은 아니다. 이 경우에는 설계자가 더 많은 개수의 레지스터를 둘 지, 혹은 클럭 사이클 시간을 더 빠르게 할지 선택해야 한다.

MIPS 에서는 레지스터를 나타낼 때, $s0 과 같은 형태로 나타낸다. 여기서 $s0, $s1 ... 은 C 와 자바 프로그램에서의 변수를 나타내고, $t0, $t1, $t2 .,.. 는 프로그램을 MIPS 명령어로 컴파일 하기 위한 임시 레지스터들을 나타낸다.

### Memory Operands

프로그래밍 언어에는 하나의 데이터를 포함하는 간단한 변수도 있지만, array 와 structure 과 같은 더욱 복잡한 자료구조들도 있다. 이러한 복잡한 자료구조들은 컴퓨터에 있는 레지스터의 수보다 더 많은 데이터들을 담을 수 있다. 그렇다면 컴퓨터는 이러한 거대한 구조들을 어떻게 접근하고 표현할까?

프로세서는 레지스터에는 아주 작은 양의 데이터만 보관할 수 있지만, 메모리는 수백만개의 데이터를 저장할 수 있다. 따라서, 자료구조들은 메모리에 저장된다.

산술 연산은 MIPS 명령어에서 오로지 레지스터에서만 발생할 수 있다. 그렇다면, MIPS 는 메모리와 레지스터 사이에 데이터를 전송하는 명령어가 있어야 한다. 이러한 명령어들을 **data transfer instructions** 이라고 한다.

메모리에 있는 word 를 접근하기 위해, 명령어는 메모리 주소를 줘야 한다. 메모리는 매우 거대한, 1차원 배열로, 주소는 그 배열의 인덱스 역할을 한다.

메모리에서 레지스터로 데이터를 복사하는 명령어를 **load** 라고 한다. MIPS 에서 이 명령어의 이름은 _lw (load word)_ 이다.

컴파일러는 array 와 같은 자료구조를 메모리 내 장소들로 할당한다. 컴파일러는 그 후에 data transfer instruction 에 적절한 시작 주소를 줄 수 있다.

많은 프로그램에서 8bit (1byte) 가 유용하기 때문에, 거의 모든 아키텍쳐들은 개별 byte로 address 한다.

MIPS에서는, word 는 4의 배수인 주소에서 시작해야 한다. 이것을 **alignment restriction** 이라고 한다.

<img width="295" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f2ad1294-9f69-4326-9734-375260d8f4f7">

MIPS는 각 byte를 주소 지정하므로, word 주소는 4의 배수이다. 왜냐하면 word는 32bit(=4byte) 가 있기 때문이다.

컴퓨터는 word 주소로 가장 왼쪽의 "big end" 바이트를 사용하느냐, 아니면 가장 오른쪽의 "little end" 바이트를 사용하느냐에 따라 나뉜다. MIPS 는 big-endian 쪽이다.

데이터를 레지스터에서 메모리로 복사하는 것을 **store** 이라고 한다. MIPS 에서는 _sw (store word)_ 명령어를 사용한다.

대부분의 프로그램이 컴퓨터에 있는 레지스터들보다 더 많은 변수들을 가지고 있다. 따라서, 컴파일러는 가장 자주 쓰이는 변수들을 레지스터에 저장하고, 나머지는 메모리에 저장하고, 둘 사이에서 load, store 를 사용하며 데이터를 옮길 것이다. 덜 자주 쓰이는 변수들을 메모리에 넣는 것을 **spilling registers** 라고 한다.

### Constant or Immediate Operands

지금까지의 연산자들로는, 상수를 레지스터에서 메모리로 먼저 가져와야 연산에 이용할 수 있었다. 이것을 대체하기 위해서, 특별히 피연산자 중 하나가 상수일 때, _addi (add immediate)_ 라는 연산자를 제공한다.

```assembly
addi $s3, $s3, 4
```

## 2.4 Signed and Unsigned Numbers

숫자들이 컴퓨터 하드웨어에서는 높고 낮은 전기신호로 되어 있기 때문에, 이진수 숫자들로 이루어져있다. 모든 정보는 binary digits 로 이루어져 있다. 어느 베이스 던지, ith digit d 의 값은 $d * Base^i$ 이다.

<img width="594" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8f5405f6-eff6-444e-a3b3-4f8366771e86">

MIPS 에서, **least significant bit** 는 가장 오른쪽 비트(bit 0) 을 가리키고, **most significant bit** 는 가장 왼쪽 비트(bit 31) 을 가리킨다. MIPS word 가 32bits 의 길이이기 때문에, 이것을 가지고 $2^{32}$ 개 만큼의 32-bit 패턴을 나타낼 수 있다. 이 양수들을 **unsigned numbers** 라고 한다.

만약 더하기,빼기,곱하기,나누기 연산의 결과가 하드웨어의 rightmost bits 로 나타내기 어려우면, _overflow_ 가 발생한다.

컴퓨터 프로그램은 양수와 음수 모두 다루기 때문에, 음수를 나타내는 방법이 있어야 한다. 가장 간단한 방법은 - 부호처럼 하나의 비트를 가지고 추가적인 사인을 나타내는 방식이다. 이것을 **sign and magnitude** 라고 한다. 하지만 이 방법도 여러가지 단점이 있는데, 한가지는 어디에다가 sign bit 을 둘것인지의 문제가 있다(오른쪽? 왼쪽?). 두번째는, 부호와 크기를 위한 덧셈기는 적절한 부호가 무엇일지 미리 알 수 없기 때문에, 부호를 설정하기 위한 추가적인 단계가 필요할 수 있다. 세번째로, 추가적인 sign bit 을 두면 양수 0 과 음수 0 이 둘 다 존재한다.

더 매력적인 대안을 찾는 과정에서, 작은 숫자에서 큰 숫자를 빼려고 할 때 부호 없는 숫자에 대한 결과가 어떻게 될지에 대한 질문이 제기되었다. 그 답은, 이 연산은 선행하는 0들로부터 빌려오려고 시도할 것이며, 그 결과 선행하는 1들의 문자열을 가지게 될 것이라는 거다. 더 나은 대안이 명백하게 없었기 때문에, 최종 해결책은 하드웨어를 단순하게 만드는 표현을 선택하는 것이었다: 선행하는 0들은 양수를 의미하고, 선행하는 1들은 음수를 의미한다. 이러한 부호 있는 이진 숫자를 표현하는 관례를 **two's complement** (2의보수) 이라고 한다.

양수 부분, 0 부터 2,147,483,647 ($2^{31}$ -1) 은 예전과 같은 표현방식을 사용한다. 그 다음 bit 패턴 ($1000 ... 00_{2}$) 는 -2,147,483,648 를 나타낸다. 그 다음 $1000 ... 01_{2}$ 는 -2,147,483,647 이고, $1111 ... 11_{2}$ 은 -1 이다.

2의 보수 표현 방식은 모든 음수는 most significant bit, 가장 왼쪽 비트가 1이라는 장점이 있다. 따라서, 하드웨어는 음수인지, 양수인지 판단하기 위해서 most significant bit 만 보면 된다. 이 비트를 _sign bit_ 라고 부른다.

그렇다면 아래와 같이 계산할 수 있다. sign bit 를 $-2^{31}$ 과 곱하면 된다. 왜냐면 음수면 1이어서 음수가 될 것이고, 양수면 어차피 sign bit 가 0 이어서 0이 되기 때문이다.

<img width="456" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/acb025ca-ce86-4341-bd03-78637d9bea26">

**sign extension** (부호 확장) 은, 원래 값의 부호(양수인지 음수인지를 나타내는)를 유지하기 위해 상위 비트들을 채워 넣습니다. 예를 들어, 8비트 부호 있는 정수(예: `-1`을 나타내는 `11111111`)를 32비트 정수로 변환할 때, 단순히 낮은 8비트만을 고려하여 확장하면, 값의 의미가 변경될 수 있다. `-1`은 32비트에서 `11111111 11111111 11111111 11111111`로 확장되어야 한다. 부호 비트(가장 왼쪽 비트)는 음수를 나타내는 `1`이고, 부호 확장 과정에서 이 비트의 값을 사용하여 나머지 상위 비트들을 채워 넣는다. 이렇게 하면 값의 범위는 증가하지만, 값의 부호와 비례 관계는 그대로 유지된다.

**Unsigned extension** 은 단순히 왼쪽 모든 bit 들을 0 으로 채운다.

sign, unsign 는 산술 연산 말고 **load** 에도 적용된다.

32비트 워드를 32비트 레지스터로 로딩할 때는 부호 있는 로드와 부호 없는 로드가 동일하다. 그 이유는 이미 데이터가 레지스터의 전체 크기와 일치하기 때문에 추가적인 확장이 필요 없기 때문이다. 그러나 MIPS에서는 바이트 로드에 대해 두 가지 방식을 제공한다: `load byte (lb)`와 `load byte unsigned (lbu)`이다.

- `load byte (lb)` 명령어는 바이트를 부호 있는(signed) 숫자로 취급하며, 따라서 레지스터의 왼쪽 24비트를 채우기 위해 부호 확장을 수행한다. 이는 로드된 바이트가 음수인 경우, 레지스터의 나머지 부분을 1로 채워 음수 값을 올바르게 표현하기 위함이다.
- `load byte unsigned (lbu)` 명령어는 바이트를 부호 없는(unsigned) 정수로 취급한다. 이 경우, 부호 확장 대신 0 확장이 적용되어 레지스터의 나머지 비트는 0으로 채워진다. 이 방식은 로드된 바이트가 항상 양수로 해석됨을 의미한다.

C 프로그램에서 바이트는 대부분 문자를 표현하는 데 사용되며, 바이트를 매우 짧은 부호 있는 정수로 간주하는 경우는 드물다. 따라서, `lbu` 명령어가 바이트 로드에 거의 독점적으로 사용된다. 이는 문자 데이터를 처리할 때 음수 값으로 해석되는 문제를 피하기 위해서이다. 문자 데이터와 같이 부호 없는 데이터를 다룰 때 `lbu` 명령어를 사용하면, 레지스터에 로드된 값이 예상한 대로 양수 값으로 해석되어, 데이터 처리가 더 단순해지고 예측 가능해진다.

#### 2의 보수를 사용하여 부호 뒤집기

모든 비트를 0->1, 1->0 으로 바꾼 후, 결과에 1을 더하면 된다.

## 2.5 Representing Instructions in the Computer

명령어가 레지스터를 참조하기 때문에, 레지스터 이름을 숫자로 매핑하는 방법이 있어야 한다. MIPS assembly 언어에서, 레지스터 $ s0 에서 $s7 까지는 레지스터 16~23 까지 매핑되고, 레지스터 $t0 에서 $t7 까지는 8 ~ 15로 매핑된다.

MIPS Assembly Instruction 을 Machine Instruction 으로 바꾸는 예시를 보자.

```assembly
add $t0, $s1, $s2
```

10진수로 표현하면, (0,17,18,8,0,32) 이다. 이 명령어의 각 세그멘트를 field 라고 한다. 첫번째와 마지막 필드는 이 연산이 add 연산이라는 것을 알려준다. 두번째 필드는 첫번째 피연산자($s1) 을 나타내고, 세번째 필드는 두번째 피연산자( $ t0) 을 나타낸다. 네번째는 결과를 저장하는 레지스터 ($t0 ) 을 나타낸다. 다섯번째는 여기서 쓰이지 않는다.

이러한 명령어의 포맷은 **instruction format** 이라고 한다. MIPS 명령어는 정확히 32bit (워드의 크기와 동일) 를 받는다. 모든 MIPS 명령어는 32 bits 길이이다.

긴 이진수로 나타내는 대신, 많은 컴퓨터들은 **hexadecima**l (16진수) 도 많이 사용한다.

### MIPS Fields

MIPS fields 는 각자의 이름이 있다.

<img width="530" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/91666994-bcd3-41e8-bdda-a1f59a02bcc0">

- op : 명령어의 기본적인 연산, **opcode** 로도 불린다.
- rs : 첫번째 레지스터 소스 연산자
- rt : 두번째 레지스터 소스 연산자
- rd : 레지스터 목적지 연산자. 연산의 결과를 저장한다.
- shamt : shift amount.
- funct : function code. op field에 있는 특정한 연산을 선택한다.

문제는 명령어가 위의 필드보다 더 긴 필드들을 필요로 할 때 발생한다. 예를 들어, load word 명령어는 2개의 레지스터와 상수를 지정해야 한다. 만약 주소가 위의 5-bit 필드 중 하나를 사용해야 한다면, 상수는 32(=$2^5$)로 제한될 것이다. 이 상수는 배열 또는 자료구조로부터 요소를 선택할 때 사용되는데, 보통 32보다 큰것이 필요하다. 5-bit 필드는 너무 작다.

따라서 우리는 모든 명령어를 같은 길이로 유지할 것인가, 또는 하나의 명령어 포맷을 유지할 것인가 고민이 발생한다.

> Design Principle 3 : Good design demands good compromises.

MIPS 설계자들은 모든 명령어를 같은 길이로 유지하기로 선택했다. 따라서 종류마다 다른 명령어 포맷이 있다. 위의 명령어는 R-format 이다. 두번째 포맷은 I-format 이고, 세번째는 J-format 이다.

(I-format)

<img width="530" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/44f19913-7104-4d30-88ba-aff3245be270">

요즘 컴퓨터들은 큰 2가지 원칙에 따라 설계되었다.

1. 명령어는 숫자로 표현된다.
2. 프로그램은 메모리에 저장되고, 데이터처럼 읽거나 쓸 수 있다.
   이 2가지 원칙들은 **stored-program** 컨셉으로 이어진다.

<img width="243" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c982bd23-9358-408c-ac68-e3823f7aca00">

## 2.6 Logical Operations

처음의 컴퓨터들은 word 전체에 대해 연산을 했다면, 이제 word 내부, 또는 bit 하나 단위로 연산을 하는 것이 필요해졌다. word 내에 8bits인 character를 검사하는 것 또한 예시이다. 이러한 연산들을 logical operations 이라고 한다.

<img width="580" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c9bd919d-8f69-4c9c-b063-6271271f21c2">

`Shift` 연산은 워드 내에 있는 모든 비트들을 왼쪽 또는 오른쪽으로 옮기고, 빈자리는 0으로 채운다. R-format 의 5번째 자리에 있는 shamt 가 shift amount 를 나타낸다. i bits 만큼 왼쪽으로 shift 하면, $2^i$ 만큼 곱하는 것과 같은 효괴이다.

`AND` 는 비트끼리의 연산으로, 두 비트 모두 1일때만 결과로 1을 반환한다. `AND` 는 마스킹의 역할도 할 수 있는데, 비트 패턴에 0이 있으면 0으로 덮어 씌우기 때문이다.

`OR` 는 두 비트 중 1개만 1이어도 결과로 1을 반환한다.

`NOT` 는 0이면 1로 바꾸고, 1이면 0으로 바꾼다.

피연산자가 3개 있는 포맷을 유지하기 위해서, MIPS 설계자들은 `NOT` 대신 `NOR` 을 포함시켰다. 피연산자 중 하나가 0이면, `NOT` 과 같은 역할을 한다: A NOR 0 = NOT (A OR 0) = NOT A

상수는 `AND` 와 `OR` 연산에도 많이 쓰이므로, MIPS 는 _and immediate (andi)_ 와 _or immediate (ori)_ 연산을 포함시켰다.

## 2.7 Instructions for Making Decisions

프로그래밍 언어에서 무언가를 결정할 때 사용하는 것은 if , go to 이다. MIPS 도 2개의 decision-making 명령어들이 있다. 이 두 명령어들은 conditional branches 라고 부른다.

```assembly
beq register1, register2, L1
```

위 명령어는 register1, register2 값이 같으면 L1 레이블로 가라는 의미이다. beq 는 _branch if equal_ 의 줄임말이다.

```assembly
bne register1, register2, L1
```

위 명령어는 register1, register2 값이 같재 않으면 L1 레이블로 가라는 의미이다. bne 는 _branch not equal_ 의 줄임말이다.

예시를 보자.

#### Compiling if-then-else into Conditional Branches

f,g,h,i,j 는 각각 레지스터 $s0 에서 $s4 까지 저장되어 있다. 아래 C언어의 MIPS 코드는 무엇일까?

```C
if (i==j) f = g + h; else f = g - h;
```

```assembly
bne $s3, $s4, Else # go to Else if i != j
add $s0, $s1, $s2
j Exit
Else : sub $s0, $s1, $s2
Exit :
```

### Loops

#### Compiling while Loop in C

아래에서 i,k 는 각각 레지스터 $s3, $s5 이다. save 배열의 base 는 $s6 이다.

```C
while (save[i] == k)
	i+= 1;
```

첫번째 줄에서, sll 을 2만큼 하면 4를 곱하는 것과 같다. byte 로 addressing 하기 때문에 인덱스 i에 4를 곱해야 한다.

```assembly
Loop : sll $t1, $s3, 2 # sll = shift left
add $t0, $t1, $s6 # $t1 = address of save[i]
lw $t0, 0($t1) # Temp reg $t0 = save[i]
bne $t0,$s5, Exit # Go to Exit if save[i] != k
addi $s3,$s3,1
j Loop

Exit :

```

---
