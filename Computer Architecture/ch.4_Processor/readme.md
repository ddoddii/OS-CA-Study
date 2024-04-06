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
