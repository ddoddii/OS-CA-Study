# Ch.4 The Processor

## 1. Introduction
이번 챕터에서는 실제 MIPS 구현을 살펴보겠습니다. MIPS 명령어 중 가장 핵심적인 부분인,
- memory-reference instructions :  `load word(lw)` , `store word(sw)`
- arithmetic-logical instructions : add, sub, AND, OR, slt
- branch instructions : `branch equal(beq)` , `jump(j)`
### Overview of Implementation
모든 명령어에 대해, 다음 2개의 스텝은 똑같습니다. 
1. Program Counter(PC) 를 코드를 가지고 있는 메모리에 보내 메모리에서 명령어를 가져옵니다.
2. 명령어의 필드들을 이용해서 레지스터 1개 또는 2개를 읽습니다. load word 명령어의 경우, 한개의 레지스터만 읽으면 됩니다. 하지만 대부분의 다른 명령어들은 2개의 레지스터로부터 읽어옵니다. 

이 2가지 단계 후에는, 명령어 세트마다 수행하는 액션이 다릅니다. 하지만 다행히도, 3가지 명령어 세트(memory-reference, arithmetic-logical, branch ) 각각 안에서 수행하는 액션들은 큰 틀에서 거의 같습니다. 

예를 들어서, jump 를 제외한 모든 명령어 클래스들은 레지스터에서 read한 후에 ALU(arithmetic-logical unit)를 사용합니다. memory-reference 명령어들은 주소 연산을 위해 ALU를 사용하고, arithmetic-logical 명령어들은 연산 실행을 위해 사용하고, branch 명령어들을 비교를 위해 사용합니다. 

ALU를 사용한 후에는, 명령어 클래스마다 수행하는 액션은 모두 다릅니다. memory-reference 명령어들은 load 를 위해 메모리를 읽거나, store 를 위해 메모리를 쓰는 작업을 위해 메모리에 접근해야 합니다. arithmetic-logical 혹은 load 명령어는 ALU로부터 나온 데이터를 레지스터에 써야 합니다. branch 명령어는, 비교 결과에 따라 다음 명령어의 주소를 바꿔야 합니다. 그렇지 않으면 PC는 자동적으로 4만큼 증가하여 다음 명령어 줄을 가져올 것입니다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/eb4cc23b-1db3-4a1a-b394-c03ba6f39731">

위의 사진은 MIPS 구현의 전체를 조망합니다. 하지만 위 구조는 명령어 실행에서 중요한 2가지 점들을 생략하고 있습니다. 

첫번째로, 하나의 유닛으로 가는 데이터가 2개의 소스로부터 오는 것으로 표현되어 있습니다. 예를 들어 PC 에 적히는 값은 2개의 Adders 중 하나로부터 올 수 있습니다. 레지스터 파일에 적히는 데이터는 ALU 또는 데이터 메모리서부터 올 수 있습니다. ALU의 두번째 인풋값은 레지스터 또는 명령어의 immediate 필드로부터 올 수 있습니다. 실제로는, 다양한 소스로부터 오는 데이터 중에 **하나를 선택**하는 로직을 추가해야 합니다. 이것은 **multiplexor** 로 할 수 있습니다. (Data selector 라고 불리기도 합니다.) 

<img width="693" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e0f36a1d-dbc8-42c4-bdbb-0950e57068d3">

두번째로, 몇몇개의 유닛들은 명령어의 타입에 따라 제어되어야 합니다. 예를 들어, 데이터 메모리는 load 에 의해 읽어지고, store 에 의해 쓰여집니다. 레지스터 파일은 load 또는 arithmetic-logical 명령어에 의해서만 쓰여집니다. 

<img width="690" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c7c6ce9f-786e-479b-bd83-bb6268c6f0de">


위의 구조는 3개의 multiplexor 와 주요 functional unit 을 위한 control lines 가 추가된 구조입니다. **control unit** 은 인풋으로 명령어를 받고, 2개의 multiplexorrhk  functional unit 을 위한 control lines를 어떻게 설정할지 결정하기 위해 사용합니다. 


## 2. Logical Design Conventions

MIPS 구현에서 datapath 요소들은 2가지 타입의 로직 요소들로 구성됩니다 : data values 를 가지고 연산하는 것들과, state 를 포함하는 요소들입니다.  data values 를 가지고 연산하는 요소들은 **combinational** 한데, 이것은 그들이 결과가 오직 현재의 인풋값에 의해 좌우된다는 의미입니다.  같은 인풋값이 주어졌을 때, 항상 같은 결과값이 나옵니다. 

다른 요소들은 **state** 요소들 인데, 상태를 가지고 있습니다. state 요소는 최소 2개의 인풋값과 1개의 결과값을 가지고 있습니다. 요구되는 인풋값은 요소에 적혀지는 데이터 값과 언제 데이터가 쓰여질지 결정하는 clock 입니다. 결과값은 이전 클럭 사이클에 적혀진 데이터 값을 의미합니다.

### Clocking Methodology 

Clocking Methodology 는 시그널이 언제 읽어지고, 쓰여지는지를 결정합니다. 간단함을 위해서, 우선 edge-triggered clocking 방법을 가정합니다. 이 방식은 sequential logic element 에 저장된 값은 오직 clock edge 에만 업데이트 됩니다. 

![IMG_0DCDF4F72C6F-1](https://github.com/ddoddii/OS-CA-Study/assets/95014836/f611f35a-f555-4518-ad91-4707cff526e7)


## 3. Building a Datapath

각 명령어들이 어떠한 datapath 요소들이 필요한지 봅시다. 

1. **Instruction Memory(IM)**
	프로그램의 명령어들과 주어진 주소에 대한 명령어를 저장하기 위한 메모리 유닛입니다. 
2. **Program Counter(PC)**
	 현재 명령어의 주소를 저장하고 있는 레지스터 입니다. 
3. **Adder**
	PC의 값을 다음 명령어의 주소로 옮기기 위한 Adder 가 필요합니다. Adder 도 ALU의 일종이지만, 덧셈 연산만 수행합니다. 

![Restore State](https://github.com/ddoddii/OS-CA-Study/assets/95014836/cac9b1f9-efcf-4d01-816b-0474b773dd5d)


이제 **R-format 명령어**들에 대해 생각해봅시다. 이 명령어들은 2개의 레지스터에서 읽고, 레지스터의 값에 대해 ALU 연산을 하고, 그 결과를 레지스터에 저장합니다. R-format 에는 *add, sub, AND, OR, slt* 가 있습니다. 

```asm
add $t1, $t2, $t3
```

프로세서의 32개 레지스터들은 레지스터 파일 내에 저장되어 있습니다. 레지스터 파일 내에 레지스터의 번호를 지정하면, 레지스터에 읽기와 쓰기를 할 수 있습니다. 

R-format 연산자는 3개 레지스터 피연산자가 있기 때문에, 레지스터 파일에서 2개의 data word 를 읽어와야 하고, 쓸 1개의 data word 가 필요합니다. data word 를 write 하기 위해서는, 쓸 값과 써야할 레지스터 주소가 필요합니다. 

![IMG_CD69E3171C0A-1](https://github.com/ddoddii/OS-CA-Study/assets/95014836/885c22bc-998c-400a-83d5-1de5c53fe2cf)

위의 레지스터 구조를 보면, 총 4개의 인풋이 필요하고, 2개의 결과값이 필요합니다. 레지스터 넘버는 5bits로, 32개의 레지스터 중 하나를 명시합니다. 

ALU는 2개의 32bits 인풋을 받고, 32bits 의 결과값을 출력합니다. 

다음으로 **load word, store word** 명령어에 대해 생각해봅시다. 

```asm
lw $t1, offset_value($t2)
sw $t1, offset_value($t2)
```

이 연산들은 베이스 레지스터 주소값(t2) 에 16-bit signed offset field 를 더해서 메모리 주소를 계산합니다. 만약 store 이면, 저장될 값은 레지스터 내에 $t1 으로부터 가져와야 합니다. load 이면, 메모리에서 읽어온 값을 지정한 레지스터인 $t1 에 저장해야 합니다. 따라서 여기서는 레지스터 파일과 ALU 둘 다 필요합니다. 

또, 16-bit offset field 32-bit signed value 로 만들기 위해 *sign-extend* 도 해야 합니다. 

![IMG_CFB4781F0F9B-1](https://github.com/ddoddii/OS-CA-Study/assets/95014836/7f369bb8-e1e6-40d4-9294-9a4511cc1df2)


**beq** 연산은 3개의 피연산자를 가지고 있는데, 2개의 레지스터는 동등성을 비교하기 위해 사용되고, 16-bit offset 는 *branch target address* 를 계산하기 위함입니다. 

```asm
beq $t1, $t2, offset
```

만약 비교 결과가 참이면, branch 가 **taken** 하다고 합니다. 브랜치 연산은 레지스터 두개의 값 비교와 target address 계산 2가지를 해야 합니다. 


![IMG_A341430C65B9-1](https://github.com/ddoddii/OS-CA-Study/assets/95014836/93d057da-9dbc-4c78-9125-1c0a05fc84d4)

### Creating a single datapath

이제 각 컴포넌트를 봤으므로, 이것들을 조합해서 하나의 datapath 로 만들고, control 을 추가할 수 있습니다. 가장 간단한 datapath 는 하나의 클럭 사이클 내에 모든 명령어들을 실행하는 것입니다. 이것은 한 명령어 당 어느 datapath 리소스도 한번 이상 사용될 수 없다는 의미입니다. 

Datapath 요소를 다른 명령어 클래스들과 공유하려면, 컴포넌트의 인풋에 여러개의 연결을 허용해야 하고, multiplexor 또는 control signal 을 이용해서 이 인풋값들 중 선택할 수 있어야 합니다. 


![IMG_AE1E2A0E5086-1](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/adfa23f9-4f9f-4106-b067-f38f6ca64179)

## 4. A Simple Implementation Scheme

이제 위의 datapath 에 **control unit** 을 추가해봅시다. 

### ALU Control

ALU 가 수행하는 연산들은 아래 6가지의 조합입니다. 

<img width="371" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/02a12c0c-c5c1-4410-bf84-8115ddbc3d89">

명령어의 클래스에 따라서, ALU는 위에서 처음 5가지 연산 중 하나를 수행합니다. 예를 들어, load word 와 store 의 경우에는 메모리 주소를 연산하기 위해 addition 을 수행합니다. R-type 명령어를 위해서는, ALU는 6-bit funct field 에 따라 AND,OR,subtract,add, set on less than 5개 연산 중 하나를 수행합니다. 

아래의 표는 2-bit ALUOP control 과  6-bit funct field 를 가지고 ALU control inputs 를 설정하는 방법을 보여줍니다. 

<img width="633" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/db10c856-8554-4c07-940d-419743d8c5be">

이 방식은 여러 단계의 디코딩이 필요합니다. 메인 컨트롤 유닛은 ALUOP bits 들을 만들고, 이것은 또 다시 ALU control 의 인풋으로 쓰여서 ALU 유닛의 실제 시그널을 만들 때 사용됩니다. 이렇게 여러 단계로 구현하면, 메인 컨트롤 유닛의 사이즈를 줄일 수 있고, 속도를 빠르게 할 수 있습니다. 

아래는 4bit ALU control 이 2개의 인풋(ALUOp, Funct field) 로부터 결정되는 **truth table** 입니다. X표시된 것들은 don't-care terms 로, 인풋의 값에 상관없음을 의미합니다. 

<img width="644" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d926e14a-f18b-417a-9815-929e0666bff6">

### Designing the Main Control Unit

function code 와 2-bit 시그널을 인풋으로 받는 ALU 를 디자인 하는 방법을 봤으니, 컨트롤의 나머지 부분들에 대해 살펴봅시다. 

명령어의 필드들과 datapath 를 연결하는 방법을 보기 위해, 3가지 명령어 타입의 포맷을 다시 한번 리뷰해봅시다.  (R,I type)

<img width="657" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1d843faa-b973-4b78-8a94-425444827a2f">

- opcode 는 31:26 의 비트들에 있습니다. 이 필드를 Op[5:0] 이라 부르겠습니다. 
- 읽어야 할 2개의 레지스터들은 rs (25:21) 와 rt (20:16) 필드로 명시됩니다. 이것은 3개 타입에 모두 해당합니다. 
- load 와 store 를 위한 베이스 레지스터 값은 rs (25:21) 에 있습니다. 
- branch equal, load, store 를 위한 16-bit offset 은 15:0 에 있습니다.
- 목적지 레지스터 위치는 2개의 후보가 있습니다. load 에서는 rt(20:16) 에 있고, R-type 명령어에서는 rd (15:11) 에 있습니다. 따라서 이때는 멀티플렉서가 필요합니다. 

이 정보들을 사용해서, 명령어 레이블과 목적지 레지스터 결정을 위한 멀티플렉서를 아까의 datapath 에 추가할 수 있습니다. 

<img width="728" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/2c73ca14-57f2-4aac-b14c-4258263a43c9">

아래 표는 7개의 컨트롤 시그널이 deasserted 또는 asserted 되었을 때 결과를 나타냅니다. 

<img width="585" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ade8b7e8-e854-4008-a3b1-fe1a5c0ba6be">

이제 위 7개의 시그널 + ALUOp 를 위한 2개의 시그널은 opcode (명령어의 31:26 비트들) 에 따라 세팅될 수 있습니다.

<img width="929" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7bd03b66-425b-48a5-90df-73e6c4748e18">

컨트롤 유닛을 위한 truth table 을 작성하기 전에, control functino 을 정의하고 갑시다. Control line 은 오로지 opcode 에 의존하기 때문에, opcode 값에 따라 컨트롤 시그널이 0,1 또는 X(don't care) 인지 정의합시다. 

<img width="777" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/fef4c17b-38aa-47f6-ae06-350a55275896">

#### Operation of the Datapath

각 명령어들이 datapath 를 어떻게 사용하는지 봅시다. 

**[R-type]**

```asm
add $t1, $t2, $t3
```

비록 하나의 클럭 사이클로 실행되기는 하지만, 4단계로 이 명령어를 수행하는 과정을 쪼개볼 수 있습니다. 
1. 명령어는 IM에서 fetch 되고, PC 는 증가됩니다. 
2. 2개의 레지스터 $t2, $t3 는 레지스터 파일로부터 읽어집니다. 동시에 메인 컨트롤 유닛은 컨트롤 라인들을 설정 하기 위해 연산합니다.
3. ALU는 function code(bits 5:0) 를 바탕으로, 레지스터 파일에서 데이터 읽기 연산을 합니다. 
4. ALU에서 나온 결과는 명령어의 15:11 비트들을 사용하여 어느 레지스터에 쓸지 정하고($t1), 레지스터 파일에 쓰여집니다. 

<img width="777" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a731c816-999b-4991-bc93-0f910ed8d8f0">

**load word** 도 비슷한 과정을 수행합니다. 

```asm
lw $t1, offset($t2)
```

1. 명령어는 IM에서 fetch 되고, PC 는 증가됩니다. 
2. 레지스터($t2) 의 값이 레지스터 파일로부터 읽어집니다. 
3. ALU는 레지스터 파일로부터 읽은 값과 offset 을 더합니다. 
4. ALU 연산의 결과는 데이터 메모리의 주소값으로 사용됩니다. 
5. 메모리 유닛의 데이터는 레지스터 파일에 쓰여집니다. 목적지 레지스터 주소는 명령어의 20:16 ($t1) 에 있습니다. 


<img width="787" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/992e1400-cbab-475c-a895-b1bedfad1ee4">

**branch-on-equal** 연산도 봅시다. 

```asm
beq $t1, $t2, offset
```

R-fomat 명령어처럼 작동하지만, ALU 연산의 결과는 PC 가 PC+4 로 업데이트 되냐 혹은 브랜치 타겟 주소로 업데이트 되는지를 결정합니다. 

1. 명령어는 IM에서 fetch 되고, PC 는 증가됩니다. 
2. 두개의 레지스터 ($t1, $t2) 는 레지스터 파일로부터 읽어집니다. 
3. ALU 는 레지스터 파일에서 읽은 값들에 빼기 연산을 합니다. PC+4 의 값은 sign-extended 된 하위 16bits(offset) 에 더해져서 브랜치 타겟 주소를 만듭니다. 
4. ALU의 결과가 0이면, PC에 어느 값을 저장할지 결정합니다. 

#### Finalizing Control

이제 마지막으로 싱글-사이클 구현 형태의 truth table 을 작성할 수 있습니다. 

<img width="654" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6f4ea172-ba16-4ee3-8550-1cab6eb0bece">

## 5. An Overview of Pipelining

파이프라이닝을 적용하면, 각 태스크에 걸리는 시간을 줄일 수는 없지만 여러 개의 태스크들을 처리하는데 걸리는 총 시간을 단축시킬 수 있습니다. 즉, **throughput** 을 증가시킬 수 있습니다. 

MIPS 명령어들은 대체로 아래 5가지 단계를 따릅니다. 

1. **메모리로부터 명령어를 Fetch 합니다.**
2. **명령어를 decode 하는 와중에 레지스터를 읽습니다. MIPS 명령어의 포맷은 reading, decoding 이 동시에 일어날 수 있도록 합니다.** 
3. **명령어를 실행하거나 주소를 계산합니다.** 
4. **데이터 메모리에 있는 피연산자에 접근합니다.** 
5. **결과를 레지스터에 write 합니다.** 

만약 각 스테이지가 걸리는 시간이 모두 같다면, 파이프라인된 명령어 사이의 시간은 (파이프라인이 안되었을 때 명령어 사이의 시간 ) / (파이프 스테이지 수) 가 될 것입니다. 그러나 현실에서는 각 스테이지마다 걸리는 시간이 모두 다릅니다. 이때 pipeline rate는 가장 긴 시간이 걸리는 task 로 제한됩니다. 

### Desiging Instruction Sets for Pipelining

MIPS 명령어 세트는 파이프라인 실행을 위해 디자인되었습니다. 
1. MIPS 명령어들은 모두 같은 길이입니다. 
2. MIPS는 적은 수의 명령어 포맷이 있고, 목적지 레지스터 필드는 각 명령어마다 같은 위치에 있습니다. 이 대칭성은 두번째 스테이지에서, reading, decoding 이 동시에 일어날 수 있음을 의미합니다. 
3. 메모리 피연산자들은 load 또는 store 에만 나타납니다. 

### Pipeline Hazards

파이프라이닝 중에 다음 클럭 사이클에 다음 명령어를 실행할 수 없는 상황이 있습니다. 이러한 이벤트들을 **hazards** 라고 부릅니다. Hazards 에는 3가지 종류가 있습니다. 

#### Structural hazard

하드웨어가 같은 클럭 사이클에 실행하고 싶은 명령어의 조합을 지원하지 않을 때 발생합니다. 

#### Data Hazard

하나의 스텝이 다른 스텝을 기다려야 해서 파이프라인이 멈춰야 할 때 발생합니다. 이 경우는 하나의 명령어가 앞선 명령어에 의존하고, 앞선 명령어가 아직도 파이프라인에 있을 때 발생합니다. 

```asm
add $s0, $t0, $t1
sub $t2, $s0, $t3
```

위의 명령어에서, sub 를 하기 위해서는 add 의 연산이 끝나야 합니다. 하지만 add 명령어는 5번째 단계에서 결과를 레지스터 파일에 씁니다. 그렇다면 뒤의 sub 명령어는 3개의 클럭 사이클을 기다려야 합니다. 

첫번째 해결방법은, ALU가 add 연산을 끝내자마자 sub 를 위한 인풋으로 집어넣는 것입니다. 내부 리소스로부터 연산 결과를 빠르게 회수하기 위해 하드웨어를 추가하는 것을 **forwarding** 또는 **bypassing** 이라고 합니다. 

<img width="628" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/18673938-734a-430c-8304-a762f755c692">

목적지 스테이지가 소스 스테이지보다 뒷 시간에 있을 때만 forwarding path 가 유효합니다. 

Fowarding 은 대체로 잘 동작하지만, 모든 파이프라인 멈춤을 해결할 수 는 없습니다. 아래 명령어를 봅시다. 

```asm
lw $s0, 20($s1)
sub $t2, $s0, $t3
```

lw 에서 s0 은 4번째 단계인 MEM 이후에만 접근 가능합니다. 하지만 sub 에서 s0이 인풋으로 들어가는 것은 3번째 단계이기 때문에, 너무 늦습니다. 

<img width="645" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b0a95b20-8ffe-45c9-9b66-8caf38c5e4b2">

따라서 Forwarding 을 한다고 해도, 여전히 **load-use data hazard** 때문에 1개의 스테이지를 기다려야 합니다. 여기서 **pipeline stall** 이라는 중요한 파이프라인 컨셉이 나오는데, **bubble** 로도 불립니다. 

#### Control Hazard

**Control Hazard** 는 다른 명령어들이 실행 중일 때, 다른 명령어에 대한 결정을 내려야 할 때 발생합니다. 이때 2가지 해결책이 있습니다. 

첫번째는, 그저 기다리는 것(stall) 입니다. 이것은 해결책이긴 하지만, 매우 느립니다.

두번째 해결책은, 예측(predict)을 하는 것입니다. 만약 맞다면, 파이프라인을 늦추지 않습니다. 만약 틀렸다면, load 를 다시 해야 합니다. 컴퓨터들은 실제로 브랜치를 다룰 때 예측을 사용합니다. 가장 간단한 접근은 브랜치가 항상 untaken 할 것이라고 예측하는 것입니다.

<img width="654" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a01bc113-f821-4d81-8473-30f4a88a3251">

반면 다이나믹 예측은, 기존의 브랜치의 과거 기록을 기반으로 예측하는 것입니다. 최근 기록들을 가지고 예측하면 정답률이 90% 이상 됩니다. 

### Pipeline Overview Summary

파이프라인은 순차적인 명령어 스트림을 사용해서 parallelism 을 구현하는 방식입니다. 파이프라인은 각 명령어가 실행되는데 걸리는 시간(latency) 를 줄이는 방식이 아니라, 시간당 처리할 수 있는 명령어 수(throughput) 을 증가시키는 방식입니다. 

## 6. Pipelined Datapath and Control


앞서 봤던 것처럼, 명령어 실행 단계를 5 단계로  나눌 수 있습니다. 
1. IF : Instruction fetch
2. ID : Instruction decode and register file read
3. EX : Execution or memory calculation
4. MEM : Data memory access
5. WB : Write back

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/2944d4e5-9ac5-481b-89d7-4bda7e8178b4)

명령어와 데이터를 주로 왼쪽에서 오른쪽으로 흘러가지만, 두 가지 예외가 있습니다. 
1. write-back 단계에서, 결과를 다시 레지스터 파일에 씁니다. 
2. PC의 다음 값을 고를때, PC+4 를 고르거나 MEM 단계에서 브랜치 주소를 골라야 합니다. 
첫번째 예외는 data hazard 를 발생시킬 수 있고, 두번째 예외는 control hazard 를 발생시킬 수 있습니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a176b17c-54ae-4368-a1e0-d9b975d0addc)

위의 예시에서는, IM 은 5단계 중 첫번째 단계(fetch)에서만 사용됩니다. 명령어가 fetch 되면, 두번째 단계(ID)에 넘겨줘야 합니다. 동시에, IM은 다음 명령어를 fetch 해옵니다. 이 과정을 더 원활하게 하기 위해, 스테이지 사이에 pipeline register 를 도입했습니다. 중간 결과를 이 pipeline register 에 저장하는 것입니다. 

빨래방 예시에서, 첫번째 사람이 세탁을 끝내면 옷들을 잠시 바구니에 넣어둘 수 있습니다. 그러면 다음 사람이 세탁기를 바로 쓸 수 있습니다. 여기서 바구니의 역할을 하는 것이 pipeline register 입니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/77fb0707-14db-49f3-abfa-7d892723177c)

위에서는, 파이프라인 레지스터가 스테이지 사이에 도입된 것을 볼 수 있습니다. IF와 ID 사이에 있는 파이프라인 레지스터는 IF/ID 레지스터라고 부릅니다. 

load 명령어 실행 시 파이프 스테이지가 어떻게 작동하는지 봅시다. 
1. *Instruction fetch* :  명령어가 PC를 이용해서 IM에서 읽어지고, IF/ID 파이프라인 레지스터에 저장됩니다. PC 주소는 4만큼 증가하고, 다음 클럭 사이클을 위해 다시 PC에 쓰여집니다.  이 증가된 주소는 나중에 beq 와 같은 명령어가 있을 때 필요할 수 있기 때문에 IF/ID 파이프라인 레지스터에도 저장됩니다. 
2. *Instruction decode and register file read* : IF/ID 파이프라인 레지스터는 16-bit immediate 필드를 공급하고, 이것은 sign-extended 되어서 32-bit 가 됩니다. 또한 2개의 레지스터를 읽기 위한 레지스터 주소도 제공합니다. 이 3가지 값들은 ID/EX 파이프라인 레지스터에 저장되고, 증가된 PC 주소도 저장됩니다. (나중을 위한 대비)
3. *Execute or address calculation* : load 명령어는 레지스터1의 값을 읽고, sign-extended immediate 값을 ALU 를 이용해서 더합니다. 이 결과값은 EX/MEM 파이프라인 레지스터에 저장됩니다. 
4. *Memory Access* : EX/MEM 파이프라인 레지스터에 저장된 주소를 이용해서 데이터 메모리에서 읽어옵니다. 이 데이터를 MEM/WB 파이프라인 레지스터에 로드합니다.
5. *Write-back* : MEM/WB 파이프라인 레지스터로부터 데이터를 읽어오고, 다시 레지스터 파일에 write 합니다. 


### Pipelined Control

이제 파이프라인 datapath 에 컨트롤을 더해 봅시다. 각 파이프라인 단계의 컨트롤 값을 설정하면 됩니다. 컨트롤 라인이 하나의 파이프라인 단계에서 활성화된 컴포넌트에 관계 있기 때문에, 컨트롤 라인을 파이프라인 단계와 마찬가지로 5개의 그룹으로 나눌 수 있습니다. 

1. *Instruction fetch* : 컨트롤 시그널은 IM을 읽고, PC에 write 합니다.
2. *Instruction decode and register file read* : 매 클럭 사이클마다 같은 일이 일어납니다. 
3. *Execute or address calculation* : 설정될 수 있는 시그널은 RegDst, ALUOp, ALUSrc 입니다.
4. *Memory Access* : 이 단계에서 컨트롤 라인은 Branch, MemRead, MemWrite 입니다. 
5. *Write-back* : 이 단계에서 컨트롤 라인은 MemtoReg, RegWrite 입니다. 