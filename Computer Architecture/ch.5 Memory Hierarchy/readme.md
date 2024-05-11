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




