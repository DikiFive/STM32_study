# Systick 滴答定时器

---

## Systick 定时器基础知识讲解

---

### Systick *是什么？*

* Systick **定时器**，是一个简单的定时器，对于CM3，CM4内核芯片，都有Systick定时器

* Systick 定时器通常用来做**延时**，或者实时系统的心跳时钟。这样可以**节省MCU资源**，不用浪费一个定时器。比如在UCOS中，分时复用，需要一个最小的时间戳，一般在在STM32+UCOS系统中，都采用Systick做UCOS心跳时钟

* Systick定时器就是系统滴答定时器，一个24位的倒计时定时器，计到0时，将从RELOAD 寄存器中自动重载定时初值。只要不把它在Systick控制及状态寄存器中的使能位清楚，就永不停息，即使在睡眠模式下也可以工作。

* Systick定时器被捆绑在NVIC中，用于产生Systick异常（异常号：15）。

* Systick中断的优先级也可以设置。

* 4个Systick寄存器
1. Systick 控制和状态寄存器——CTRL
    * 对于STM32，外部时钟源是HCLK（AHB总线时钟）的1/8内核时钟是HCLK时钟
      * 配置函数：SysTick_CLKSourceConfig();

2. SysTick 重装载数值寄存器——LOAD
    * 当倒数至0时，**将被重转载的值**
3. SysTick 当前值寄存器——VAL
    * 读取时返回**当前倒计数的值**，写它则使之清零，同时还会清楚再SysTick控制及状态寄存器中的COUNTFLAG标志
---

## Systick 相关寄存器库函数讲解

* 固件库中的Systick相关函数：

    Systick_CLKSourceConfig()//Systick时钟源选择 **misc.c**文件中

    Systick_Config(uint32_t ticks)//初始化Systick，时钟为HCLK，并开启中断

    //core_cm3.h/core.cm4.h **文件中**

* Systick中断服务函数

    void Systick_Handler(void);

---

## delay 延时函数讲解（Systick 应用）

* 先初始化 Systick 的时钟源
---
> <STM32F1开发指南-库函数版本>
> >5.1 delay文件夹介绍


