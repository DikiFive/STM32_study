# NVIC中断优先级分组

* CM3内核支持256个中断，其中包含了16个内核中断喝240个外部中断，并且具有256级的可编程中断设置。
* STM32并没有使用CM3内核的全部东西，而是只用了它的一部分。
* STM32有84 个中断，包括16个内核中断和68个可屏蔽中断，具有16级可编程的中断优先级。
* STM32F103系列上面，又只有60个可屏蔽中断（在107系才有68个）

        参考《STM32中文参考手册》

---

## 中断管理方法：
首先，对STM32中断进行分组，组0~4.同时，对每个中断设置一个抢占优先级和一个响应优先级值。

分组配置是在寄存器SCB -> AIRCR中配置

|  组   | AIRCR[10:8] | IP bit[7:4]分配情况 |           分配结果           |
| :---: | :---------: | :-----------------: | :--------------------------: |
|   0   |     111     |         0:4         | 0位抢占优先级，4位响应优先级 |
|   1   |     110     |         1:3         | 1位抢占优先级，3位响应优先级 |
|   2   |     101     |         2:2         | 2位抢占优先级，2位响应优先级 |
|   3   |     100     |         3:1         | 3位抢占优先级，1位响应优先级 |
|   4   |     011     |         4:0         | 4位抢占优先级，0位响应优先级 |

---

## 抢占有优先级&响应优先级区别：

  * 高优先级的抢占优先级是可以打断正在进行的低抢占优先级中断的。
  * 抢占优先级相同的中断，高响应优先级不可以打断低响应优先级的中断。
  * 抢占优先级相同的中断，当两个中断同时发生的情况下，哪个响应优先级高，哪个先执行。
  * 如果两个中断的抢占优先级和响应优先级都是一样的话，则看哪个中断先发生就先执行；

---

### 举例
  * 假定设置中断优先级组为2，然后设置中断3（RTC中断）的抢占优先级为2，响应优先级1。中断6（外部中断0）的抢占优先级为3，响应优先级为0。中断7（外部中断1）的抢占优先级为2，响应优先级为0.

*那么这三个中断的优先级顺序为*：**中断7>中断3>中断6**

* **特别说明**

  *一般情况下，系统代码执行过程中，只设置一次中断优先级分组，比如分组2，设置好分组之后一般不会再改变分组。随意改变分组会导致中断管理混乱，程序出现意想不到的执行结果。*
---

# NVIC中断优先级设置

* 中断优先级分组函数：

        void NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup);

        void NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup)
        {
            assert_param(IS_NVIC_PRIORITY_GROUP(NVIC_PriorityGroup));
            SCB->AIRCR = AIRCR_VECTKEY_MASK | NVIC_PriorityGroup;
        }

        NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);

---

* 中断设置相关寄存器
  * _IO unit8_t IP[240]; //中断优先级控制的寄存器组

  * _IO unit32_t ISER[8]; //中断使能寄存器组
  * _IO unit32_t ICER[8]; //中断失能寄存器组
  * _IO unit32_t ISPR[8]; //中断挂起寄存器组
  * _IO unit32_t ICPR[8]; //中断解挂寄存器组
  * _IO unit32_t IABR[8]; //中断激活标志位寄存器组

---

* MDK中NVIC寄存器结构体

        typedef struct
        {
            _IO unit8_t ISER[240];
            uint32_t RESERVED0[24];
            _IO unit8_t ICEP[240];
            uint32_t RESERVED1[24];
            _IO unit8_t ISPR[240];
            uint32_t RESERVED2[24];
            _IO unit8_t ICPR[240];
            uint32_t RESERVED3[24];
            _IO unit8_t IABR[240];
            uint32_t RESERVED4[56];
            _IO unit8_t IP[240];
            uint32_t RESERVED5[644];
            _O unit32_t STIR;
        }NVIC_Type;

---

* 对于每个中断怎么设置优先级？

中断优先级控制的寄存器组：IP[240]
全称是：Interrupt Priority Registers

240个8位寄存器，每个中断使用一个寄存器来确定优先级。

STM32F10X系列一共60个可屏蔽中断，使用IP[59]~IP[0]。

每个IP寄存器的高四位用来设置抢占和响应优先级（根据分组），低四位没有用到。

        void NVIC_Init(NVIC_InitTypeDef* NVIC_InitStruct);

---

* 中断使能寄存器：ISER[8]

**作用：用来使能中断**

32位寄存器，每个位控制一个中断的使能。STM32F10X只有60个可屏蔽中断，所以只使用了其中的ISER[0]和ISER[1]。

ISER[0]的bit0~bit31分别对应中断0~31.ISER[1]的bit0~bit27对于中断32~59；

        void NVIC_Init(NVIC_InitTypeDef* NVIC_InitStruct);

---

* 中断失能寄存器组：ICER[8]

**作用：用来失能中断**

32位寄存器，每个位控制一个中断的失能。STM32F10X只有60个可屏蔽中断，所以只使用了其中的ICER[0]和ICER[1]。

ICER[0]的bit0~bit31分别对应中断0~31.ICER[1]的bit0~bit27对于中断32~59；

*配置方法和ISER一样*

        void NVIC_Init(NVIC_InitTypeDef* NVIC_InitStruct);

---

* 中断挂起控制寄存器组：ISPR[8]

**作用：用来挂起中断**

* 中断解挂控制寄存器组：ICPR[8]

**作用：用来解挂中断**

        static _INLINE void NVIC_SetPendingIRQ(IRQn_TypeIRQn);
        static _INLINE uint32_t NVIC_GetPendingIRQ(IRQn_TypeIRQn);
        static _INLINE void NVIC_ClearPendingIRQ(IRQn_TypeIRQn);

* 中断激活标志位寄存器组：IABR[8]


**作用：只读**，通过它可以知道当前正在执行的中断是哪一个，如果对应位为1，说明该中断正在执行。

        static _INLINE uint32_t NVIC_GetActiveIRQ(IRQn_TypeIRQn);

---
### 中断参数初始化函数

        void NVIC_Init(NVIC_InitTypeDef* NVIC_InitStruct);
        typedef struct
        {
        uint8_t NVIC_IRQChannel;//设置中断通道
        uint8_t NVIC_IRQChannelPreemptionPriority;//设置响应优先级
        uint8_t NVIC_IRQChannelSubPriority;//设置抢占优先级
        FunctionalState NVIC_IRQChannelCmd;//使能/使能
        } NVIC_InitTypeDef;

        NVIC_InitTypeDef NVIC_InitStruct;
        NVIC_InitStruct.NVIC_IRQChannel = USART1_IRQn;//串口1中断
        NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 1;//抢占优先级为1
        NVIC_InitStruct.NVIC_IRQChannelSubPriority = 2;//子优先级位2
        NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;//IRQ通道使能
        NVIC_Init(&NVIC_InitStruct);//根据上面指定的参数初始化NVIC寄存器

---

# NVIC总结

## 中断优先级设置步骤
1. 系统运行后先设置中断优先级分组。调用函数：

    **void NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup);**
    **整个系统执行过程中，只设置一次中断分组。**

2. 针对每个中断，设置对应的抢占优先级和响应优先级：

    **void NVIC_Init(NVIC_InitTypeDef* NVIC_InitStruct);**

3. 如果需要挂起/解挂，查看中断当前激活状态，分别调用相关函数即可。

---