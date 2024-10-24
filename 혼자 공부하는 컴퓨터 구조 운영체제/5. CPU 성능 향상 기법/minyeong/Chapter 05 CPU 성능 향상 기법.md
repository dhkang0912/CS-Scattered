
# Chapter 05. CPU 성능 향상 기법
- [Chapter 05. CPU 성능 향상 기법](#chapter-05-cpu-성능-향상-기법)
- [05-1 빠른 CPU를 위한 설계 기법](#05-1-빠른-cpu를-위한-설계-기법)
    - [클럭](#클럭)
    - [코어와 멀티 코어](#코어와-멀티-코어)
    - [스레드와 멀티 스레드](#스레드와-멀티-스레드)
- [05-2. 명령어 병렬 처리 기법 (ILP)](#05-2-명령어-병렬-처리-기법-ilp)
    - [명령어 파이프라이닝 (instruction pipelining)](#명령어-파이프라이닝-instruction-pipelining)
    - [슈퍼스칼라 (superscalar)](#슈퍼스칼라-superscalar)
    - [비순차적 명령어 처리 (OoOE)](#비순차적-명령어-처리-oooe)
- [05-3. CISC와 RISC](#05-3-cisc와-risc)
    - [CISC (Complex Instruction Set Computer)](#cisc-complex-instruction-set-computer)
    - [RISC (Reduced Instruction Set Computer)](#risc-reduced-instruction-set-computer)
- [Q\&A](#qa)


# 05-1 빠른 CPU를 위한 설계 기법

### 클럭

- 컴퓨터의 모든 부품을 일사불란하게 움직일 수 있게 하는 시간 단위
- 클럭 속도가 높으면 CPU의 성능이 좋지만 한계 있음
- 일정한 클럭 속도를 유지하는 게 아닌 고성능을 요하는 순간에는 순간적으로 클럭 속도를 높이고, 그렇지 않을 때는 유연하게 클럭 속도 낮춤
    - 오버클럭킹 (overclocking) : 최대 클럭 속도 강제로 끌어올리는 것
- **`클럭 속도`** : 헤르츠(Hz) 단위 (1초에 클럭이 몇 번 반복되는지)
    - 1GHz = 1,000,000,000Hz(10**9)
    - 오늘날 CPU 기본 속도 : 2.5GHz, 최대 속도 : 4.9GHz

### 코어와 멀티 코어

**`코어`**  : CPU 내에서 명령어를 실행하는 부품, 여러 개 있을 수 있음

**`멀티 코어`** : 여러 개의 코어

- 멀티 코어 CPU (멀티 코어 프로세서) : 여러 개의 코어를 포함하고 있는 CPU
- 코어 수에 따라 CPU의 연산 속도가 반드시 비례하여 증가하지는 않음 → 명령어의 적절한 분배 중요
- 코어 포함 개수에 따른 CPU 종류
    
    ![image.png](./Chapter%2005%20CPU%20성능%20향상%20기법/image.png)
    

### 스레드와 멀티 스레드

: 실행 흐름의 단위 (사전적 의미)

![image.png](./Chapter%2005%20CPU%20성능%20향상%20기법/image%201.png)

**`하드웨어적 스레드`**

- 하나의 코어가 동시에 처리하는 명령어 단위
- `멀티스레드 프로세서` (멀티스레드 CPU)
    - 하나의 코어로 여러 명령어를 동시에 처리하는 CPU
    - 여러 개의 하드웨어적 스레드를 지원하는 CPU
    - 하나의 명령어를 처리하기 위해 필요한 레지스터를 여러 개 가지고 있음
        - 프로그램 카운터, 스택 포인터, 데이터 버퍼 레지스터, 데이터 주소 레지스터 등
    - 슈퍼스칼라 구조 사용 가능
- 참고
    - ‘논리 프로세서’ 라고 부르기도 함
        - 프로그램 입장에선 한 번에 하나의 명령어를 처리하는 CPU가 여러 개 있는 것 처럼 보임
    - 하이퍼스레딩(hyper-threading) - 인텔의 멀티 스레드 기술

**`소프트웨어적 스레드`** 

- 하나의 프로그램에서 독립적으로 실행되는 단위
- 프로그램의 여러 부분을 각각의 스레드로 만들면 동시에 실행 될 수 있음

<aside>
🖍️ 정리

- 코어 : 명령어를 실행할 수 있는 하드웨어 부품
- 스레드 : 명령어를 실행하는 단위
- 멀티코어 프로세서 : 명령어를 실행할 수 있는 하드웨어 부품이 CPU 안에 두 개 이상 있는 CPU
- 멀티스레드 프로세서 : 하나의 코어로 여러 개의 명령어를 동시에 실행할 수 있는 CPU
</aside>

# 05-2. 명령어 병렬 처리 기법 (ILP)

: 명령어 동시에 처리

### 명령어 파이프라이닝 (instruction pipelining)

: 동시에 여러 개의 명령어를 겹쳐 실행하는 기법

- 명령어 처리 시, 단계가 겹치지 않으면 **각 단계를 동시에 실행할 수 있음**
    - 참고 : 클럭 단위로 나눠 본 명령어 처리 과정
        1. 명령어 인출
        2. 명령어 해석
        3. 명령어 실행
        4. 결과 저장
        
        ⇒ 순서는 바뀔 수 있음  
        
- `파이프라인 위험`
    - 성능 향상에 실패하는 경우
    - 데이터 위험
        - 명령어 간 **데이터 의존성**에 의해 발생
        - 데이터가 의존되어 하나의 명령어를 수행해야만 다음 명령어가 수행될 수 있을 때, 무조건 동시에 실행하면 파이프라인이 제대로 작동 하지 않음
            
            ![image.png](./Chapter%2005%20CPU%20성능%20향상%20기법/image%202.png)
            
    - 제어 위험
        - **프로그램 카운터의 갑작스러운 변화**에 의해 발생
        - 프로그램 카운터는 현재 실행 중인 명령어의 다음 주소로 갱신 되는데, 프로그램 실행 흐름이 바뀌어 프로그램 카운터의 값에 변화가 생기면 파이프라인에 미리 가지고 와서 처리 중이던 명령어는 필요 없게 됨
            
            ![image.png](./Chapter%2005%20CPU%20성능%20향상%20기법/image%203.png)
            
        - 분기 예측 : 프로그램이 어디로 분기할 지 미리 예측한 후 그 주소 인출하는 기술
        
    - 구조적 위험
        - 자원 위험
        - 명령어를 겹쳐 실행하는 과정에서 서로 다른 명령어가 동시에 CPU 부품을 사용하려는 경우 발생

### 슈퍼스칼라 (superscalar)

: 여러 개의 명령어 파이프라인을 두는 기법

- `슈퍼스칼라 프로세서` (슈퍼스칼라 CPU)
    - 슈퍼스칼라 구조로 명령어 처리 가능한 CPU
    - 매 클럭 주기마다 동시에 여러 명령어 인출, 실행 가능
    - 반드시 파이프라인 개수에 비례하여 프로그램 처리 속도 빨라지진 않음
        - 오히려 파이프라인 위험이 커짐

### 비순차적 명령어 처리 (OoOE)

: 파이프라인의 중단을 방지하기 위해 명령어를 순차적으로 처리하지 않는 명령어 병렬 처리 기법

- 순서를 바꿔도 무방한 명령어를 먼저 실행

# 05-3. CISC와 RISC

- CPU마다 명령어 집합 (명령어 집합 구조 : ISA) 다름 → 서로의 명령어 이해 불가
    - 즉, ISA는 CPU의 언어이자 하드웨어가 소프트웨어를 어떻게 이해할 지에 대한 약속

### CISC (Complex Instruction Set Computer)

: 복잡하고 다양한 명령어들을 활용하는 CPU 설계 방식

- x86, x86-64
- 장점
    - 복잡하고 다양한 수의 **가변 길이 명령어 집합** 활용
    - 상대적으로 적은 수의 명령어로도 프로그램 실행 가능 ⇒ 컴파일된 프로그램의 크기 작음(메모리 절약)
- 단점
    - 명령어의 크기와 실행되기까지의 시간 일정하지 않음 ⇒ 명령어 규격화 어려움, 파이프라이닝 어려움
    - 복잡한 명령어 때문에 하나의 명령어 실행에 여러 클럭 주기 필요
    - 대다수의 복잡한 명령어의 사용 빈도 낮음

### RISC (Reduced Instruction Set Computer)

: 단순하고 적은 수의 명령어들을 활용하는 CPU 설계 방식

- ARM
- 장점
    - 단순하고 적은 수의 **고정 길이 명령어 집합** 활용 ⇒ 짧고 규격화된 명령어
    - 명령어 파이프라이닝에 최적화 : 되도록 1클럭 내외로 실행
    - 메모리 접근 단순화, 최소화 : 메모리에 직접 접근하는 명령어 `load`, `store`로 제한
- 단점
    - 레지스터를 이용한 연산이 많음
    - 많은 명령으로 프로그램 작동

![image.png](./Chapter%2005%20CPU%20성능%20향상%20기법/image%204.png)

# Q&A

1. **코어와 스레드, 멀티 코어 프로세서와 멀티 스레드를 비교하여 설명하시오.**
2. **명령어 병렬 처리 기법과 그 종류에 대해 간략히 설명하시오.**
3. **CISC와 RISC의 차이에 대해 설명하시오.**

- 1번 답
    
    코어는 명령어를 실행할 수 있는 하드웨어 부품이고, 스레드는 명령어를 실행하는 단위이다.
    
    멀티 코어 프로세서는 코어가 CPU 안에 두 개 이상 있는 CPU인 반면, 멀티 스레드 프로세서는 하나의 코어로 여러 개의 명령어를 동시에 실행할 수 있는 CPU이다.
    
- 2번 답
    
    명령어 병렬 처리 기법은 명령어를 동시에 처리하는 기법으로, 명령어 파이프라이닝, 슈퍼스칼라, 비순차적 명령어 처리 기법이 있다.
    
    명령어 파이프라이닝은 동시에 여러 개의 명령어를 겹쳐 실행하는 기법이며, 데이터 위험, 제어 위험, 구조적 위험인 파이프라인 위험이 존재한다. 슈퍼스칼라는 여러 개의 명령어 파이프라인을 두는 기법이고, 비순차적 명령어 처리 기법은 파이프라인의 중단을 방지하기 위해 명령어를 순차적으로 처리하지 않는 명령어 병렬 처리 기법이다.
    
- 3번 답
    
    CISC는 복잡하고 다양한 명령어들을 활용하는 CPU 설계 방식으로, 가변 길이의 명령어를 활용하고, 다양한 주소 지정 방식을 갖는다. 프로그램을 이루는 명령어의 수가 적고, 여러 클럭에 걸쳐 명령어를 수행하기 때문에 파이프라이닝하기가 어렵다.
    
     RISC는 단순하고 적은 수의 명령어들을 활용하는 CPU 설계 방식으로, 고정된 길이의 명령어를 활용하고, 적은 주소 지정 방식을 갖는다. 프로그램을 이루는 명령어의 수가 많고, 1 클럭 내외로 명령어를 수행해 파이프라이닝하기 쉽다.
