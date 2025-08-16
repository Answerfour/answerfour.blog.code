---
title: STM32 Learning Notes
date: 2024-2-28
tags: 
  - STM32
categories: 
cover: /media/featureimages/2.jpg
description: 江科协stm32视频的学习笔记，持续更。。。。？？？
---

## 1 配置文件介绍

[STM32入门教程-2023版 细致讲解 中文字幕 - 哔哩哔哩 (bilibili.com)](https://www.bilibili.com/opus/814075999736037492?spm_id_from=333.999.0.0)

### 1.1 start

`\STM32入门教程资料\固件库\STM32F10x_StdPeriph_Lib_V3.5.0\Libraries\CMSIS\CM3\DeviceSupport\ST\STM32F10x\startup`

```shell
startup_stm32f10x_md.s   #启动文件
system_stm32f10x.c #配置时钟
system_stm32f10x.h
stm32f10x.h #外设寄存器文件，类似reg52.h，描述寄存器与地址
```

`\STM32入门教程资料\固件库\STM32F10x_StdPeriph_Lib_V3.5.0\Libraries\CMSIS\CM3\CoreSupport`

```shell
core_cm3.c #内核寄存器描述
core_cm3.h
```

### 1.2 library 标准外设驱动库函数

`\STM32入门教程资料\固件库\STM32F10x_StdPeriph_Lib_V3.5.0\Libraries\STM32F10x_StdPeriph_Driver`

src和inc文件里面的所有文件

```shell
mics.c  #内核的库函数，其他是内核外的库函数
```

### 1.3 user

```shell
stm32f10x_conf.h    #配置库函数头文件包含关系，检查函数定义
stm32f10x_it.c放中断函数
stm32f10x_it.h
```

### 1.4 复制字符串、包含外设库

![img](https://i1.hdslb.com/bfs/note/10928388fe2ccd500b9ac4383da035fb09f78b94.png@1054w_772h.webp)

![img](https://i1.hdslb.com/bfs/note/17cd28850cab71630152b509b117be6a7f145d67.png@1066w_780h.webp)

![img](https://i1.hdslb.com/bfs/note/942541aa157abb163ebfb5449a729b553c47725e.png@1140w_748h.webp)

## 2 外设学习

### 2.1 GPIO

io口不初始化，默认浮空输入

| 八种模式              | **模式名称** | **性质** |                      **特征**                      |
| :-------------------- | :----------: | :------: | :------------------------------------------------: |
| GPIO_Mode_IN_FLOATING |   浮空输入   | 数字输入 |      可读取引脚电平，若引脚悬空，则电平不确定      |
| GPIO_Mode_IPU         |   上拉输入   | 数字输入 | 可读取引脚电平，内部连接上拉电阻，悬空时默认高电平 |
| GPIO_Mode_IPD         |   下拉输入   | 数字输入 | 可读取引脚电平，内部连接下拉电阻，悬空时默认低电平 |
| GPIO_Mode_AIN         |   模拟输入   | 模拟输入 |           GPIO无效，引脚直接接入内部ADC            |
| GPIO_Mode_Out_OD      |   开漏输出   | 数字输出 |    可输出引脚电平，高电平为高阻态，低电平接VSS     |
| GPIO_Mode_Out_PP      |   推挽输出   | 数字输出 |      可输出引脚电平，高电平接VDD，低电平接VSS      |
| GPIO_Mode_AF_OD       | 复用开漏输出 | 数字输出 |    由片上外设控制，高电平为高阻态，低电平接VSS     |
| GPIO_Mode_AF_PP       | 复用推挽输出 | 数字输出 |      由片上外设控制，高电平接VDD，低电平接VSS      |


#### 2.1.1 输出

```c
/*开启时钟*/
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	//开启GPIOA的时钟
//使用各个外设前必须开启时钟，否则对外设的操作无效

/*GPIO初始化*/
GPIO_InitTypeDef GPIO_InitStructure;					//定义结构体变量
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;		//GPIO模式，赋值为推挽输出模式
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;				//GPIO引脚，赋值为第0号引脚
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		//GPIO速度，赋值为50MHz
GPIO_Init(GPIOA, &GPIO_InitStructure);					//将赋值后的构体变量传递给GPIO_Init函数
//函数内部会自动根据结构体的参数配置相应寄存器
//实现GPIOA的初始化

/*方法1：GPIO_ResetBits设置低电平，GPIO_SetBits设置高电平*/
GPIO_ResetBits(GPIOA, GPIO_Pin_0|GPIO_Pin_1);		//将PA0,PA1引脚设置为低电平,可“与”计算
GPIO_SetBits(GPIOA, GPIO_Pin_0);					//将PA0引脚设置为高电平

/*方法2：GPIO_WriteBit设置低/高电平，由Bit_RESET/Bit_SET指定*/
GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_RESET);		//将PA0引脚设置为低电平
GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);			//将PA0引脚设置为高电平

/*方法3：GPIO_WriteBit设置低/高电平，由数据0/1指定，数据需要强转为BitAction类型*/
GPIO_WriteBit(GPIOA, GPIO_Pin_0, (BitAction)0);		//将PA0引脚设置为低电平
GPIO_WriteBit(GPIOA, GPIO_Pin_0, (BitAction)1);		//将PA0引脚设置为高电平

/*使用GPIO_Write，同时设置GPIOA所有引脚的高低电平，实现LED流水灯*/
GPIO_Write(GPIOA, ~0x0001);	//0000 0000 0000 0001，PA0引脚为低电平，其他引脚均为高电平，注意数据有按位取反
Delay_ms(100);				//延时100ms
GPIO_Write(GPIOA, ~0x0002);	//0000 0000 0000 0010，PA1引脚为低电平，其他引脚均为高电平
```

#### 2.1.1 输入

```c
/*开启时钟*/
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);		//开启GPIOB的时钟
/*GPIO初始化*/
GPIO_InitTypeDef GPIO_InitStructure;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_1 | GPIO_Pin_11;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
GPIO_Init(GPIOB, &GPIO_InitStructure);						//将PB1和PB11引脚初始化为上拉输入


if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0)			//读PB1输入寄存器的状态，如果为0，则代表按键1按下
{
    Delay_ms(20);											//延时消抖
    while (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0);	//等待按键松手
    Delay_ms(20);											//延时消抖
    KeyNum = 1;												//置键码为1
}
```



### OLED调试

暂无

•68个可屏蔽中断通道，包含EXTI、TIM、ADC、USART、SPI、I2C、RTC等多个外设



### 2.2 EXIT





•NVIC的中断优先级由优先级寄存器的4位（0~15）决定，这4位可以进行切分，分为高n位的抢占优先级和低4-n位的响应优先级

•抢占优先级高的可以中断嵌套，响应优先级高的可以优先排队，抢占优先级和响应优先级均相同的按中断号排队

| **分组方式** | **抢占优先级**  | **响应优先级**  |
| ------------ | --------------- | --------------- |
| 分组0        | 0位，取值为0    | 4位，取值为0~15 |
| 分组1        | 1位，取值为0~1  | 3位，取值为0~7  |
| 分组2        | 2位，取值为0~3  | 2位，取值为0~3  |
| 分组3        | 3位，取值为0~7  | 1位，取值为0~1  |
| 分组4        | 4位，取值为0~15 | 0位，取值为0    |

主要函数介绍

```c
void EXTI_DeInit(void);									//用于将外部中断模块恢复到初始状态。
void EXTI_Init(EXTI_InitTypeDef* EXTI_InitStruct);		//用给定的配置初始化外部中断。
void EXTI_StructInit(EXTI_InitTypeDef* EXTI_InitStruct);//初始化指定的外部中断配置结构体。
void EXTI_GenerateSWInterrupt(uint32_t EXTI_Line);		//产生软件触发的外部中断。

//在主函数内查看，触发了标志位不一定会挂起中断，这是区别
FlagStatus EXTI_GetFlagStatus(uint32_t EXTI_Line);		/*功能：获取指定外部中断线的触发标志位状态。
													      返回值：标志位状态，可以是SET或者RESET。	*/
void EXTI_ClearFlag(uint32_t EXTI_Line);				//清除指定外部中断线的触发标志位。

//在中断函数内查看，此时可以查看是否已经开启了中断
ITStatus EXTI_GetITStatus(uint32_t EXTI_Line);			/*功能：获取指定外部中断线的中断挂起状态。
														  返回值：中断挂起状态，可以是SET或者RESET。*/
void EXTI_ClearITPendingBit(uint32_t EXTI_Line);		//清除指定外部中断线的中断挂起位。  
```



```c
/*开启时钟*/
RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);		//开启AFIO的时钟，外部中断必须开启AFIO的时钟
/*AFIO选择中断引脚*/
GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource14);//将外部中断的14号线映射到GPIOB，即选择PB14为外部中断引脚

/*EXTI初始化*/
EXTI_InitTypeDef EXTI_InitStructure;						//定义结构体变量
EXTI_InitStructure.EXTI_Line = EXTI_Line14;					//选择配置外部中断的14号线
EXTI_InitStructure.EXTI_LineCmd = ENABLE;					//指定外部中断线使能
EXTI_InitStructure.EXTI_Mode = EXTI_Mode_Interrupt;			//指定外部中断线为中断模式
EXTI_InitStructure.EXTI_Trigger = EXTI_Trigger_Falling;		//指定外部中断线为下降沿触发
EXTI_Init(&EXTI_InitStructure);								//将结构体变量交给EXTI_Init，配置EXTI外设

/*NVIC中断分组*/
NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);	    //配置NVIC为分组2
													//即抢占优先级范围：0~3，响应优先级范围：0~3
													//此分组配置在整个工程中仅需调用一次
													//若有多个中断，可以把此代码放在main函数内，while循环之前
													//若调用多次配置分组的代码，则后执行的配置会覆盖先执行的配置

/*NVIC配置*/
NVIC_InitTypeDef NVIC_InitStructure;						//定义结构体变量
NVIC_InitStructure.NVIC_IRQChannel = EXTI15_10_IRQn;		//选择配置NVIC的EXTI15_10线
NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;				//指定NVIC线路使能
NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;	//指定NVIC线路的抢占优先级为1
NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;			//指定NVIC线路的响应优先级为1
NVIC_Init(&NVIC_InitStructure);								//将结构体变量交给NVIC_Init，配置NVIC外设

void EXTI15_10_IRQHandler(void)
{
	if (EXTI_GetITStatus(EXTI_Line14) == SET)		//判断是否是外部中断14号线触发的中断
	{
		/*如果出现数据乱跳的现象，可再次判断引脚电平，以避免抖动*/
		if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_14) == 0)
		{
			CountSensor_Count ++;					//计数值自增一次
		}
		EXTI_ClearITPendingBit(EXTI_Line14);		//清除外部中断14号线的中断标志位
													//中断标志位必须清除
													//否则中断将连续不断地触发，导致主程序卡死
	}
}
```

### 2.3 TIM

预分频器的工作的工作原理是，定时器时钟源每tick一次，**预分频器计数器**值+1，直到达到预分频器的设定值，然后再tick一次后计数器归零，同时，**CNT计数器**值+1。

由此可以看出，因为达到最大值后还要再tick一次才归零，所以定时器时钟频率应该为Fosc/(PSC+ 1)。其中Fosc是定时器的时钟源。比如想对时钟源进行72分频，那么预分频器的值就应该设置为71。

由此可以看出，因为达到最大值后还要再tick一次才归零，所以定时器时钟频率应该为Fosc/(PSC+ 1)。其中Fosc是定时器的时钟源。比如想对时钟源进行72分频，那么预分频器的值就应该设置为71。

预分频器值寄存器TIMx_PSC存在影子寄存器（官方翻译为缓冲功能），所以在定时器启动后更改TIMx_PSC的值并不会立即影响当前定时器的时钟频率。要等到下一个更新事件（UEV）发生时才会生效。比如下边这张图就体现了将分频系数由1修改为2（即TIMx_PSC由0更改为1）时整个定时器的时序图。

![image-20240324104734248](C:\Users\Answerfour\AppData\Roaming\Typora\typora-user-images\image-20240324104734248.png)

更新事件（UEV）则由TIMx_CR1寄存器中的UDIS位控制，在启用时，会通过以下两种方式触发 ：

- 计数器上溢
- 手动将 TIMx_EGR 寄存器中的UG 位置 1

[STM32定时器（TIM）之预分频器（PSC）详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/82590576)

#### 2.3.1 定时中断

```c
/*开启时钟*/
RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);			//开启TIM2的时钟
/*配置时钟源*/
TIM_InternalClockConfig(TIM2);		//选择TIM2为内部时钟，若不调用此函数，TIM默认也为内部时钟

/*时基单元初始化*/
TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;				//定义结构体变量
TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;		//时钟分频，选择不分频，此参数用于配置滤波器时钟，不影响时基单元功能
TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;	//计数器模式，选择向上计数
TIM_TimeBaseInitStructure.TIM_Period = 10000 - 1;				//计数周期，即ARR的值
TIM_TimeBaseInitStructure.TIM_Prescaler = 7200 - 1;				//预分频器，即PSC的值
TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;			//重复计数器，高级定时器才会用到
TIM_TimeBaseInit(TIM2, &TIM_TimeBaseInitStructure);	//将结构体变量交给TIM_TimeBaseInit，配置TIM2的时基单元	

/*中断输出配置*/
TIM_ClearFlag(TIM2, TIM_FLAG_Update);						//清除定时器更新标志位
                                                            //TIM_TimeBaseInit函数末尾，手动产生了更新事件
                                                            //若不清除此标志位，则开启中断后，会立刻进入一次中断
                                                            //如果不介意此问题，则不清除此标志位也可

TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);					//开启TIM2的更新中断

/*NVIC中断分组*/
NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//配置NVIC为分组2
                                               //即抢占优先级范围：0~3，响应优先级范围：0~3
                                               //此分组配置在整个工程中仅需调用一次
                                               //若有多个中断，可以把此代码放在main函数内，while循环之前
                                               //若调用多次配置分组的代码，则后执行的配置会覆盖先执行的配置

/*NVIC配置*/
NVIC_InitTypeDef NVIC_InitStructure;						//定义结构体变量
NVIC_InitStructure.NVIC_IRQChannel = TIM2_IRQn;				//选择配置NVIC的TIM2线
NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;				//指定NVIC线路使能
NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 2;	//指定NVIC线路的抢占优先级为2
NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;			//指定NVIC线路的响应优先级为1
NVIC_Init(&NVIC_InitStructure);								//将结构体变量交给NVIC_Init，配置NVIC外设

/*TIM使能*/
TIM_Cmd(TIM2, ENABLE);			//使能TIM2，定时器开始运行

/*定时器中断函数，可以复制到使用它的地方
void TIM2_IRQHandler(void)
{
if (TIM_GetITStatus(TIM2, TIM_IT_Update) == SET)
{
    
    TIM_ClearITPendingBit(TIM2, TIM_IT_Update);
}*/
```

#### 2.3.2 外部时钟中断

```c
/*外部时钟配置*/
	TIM_ETRClockMode2Config(TIM2, TIM_ExtTRGPSC_OFF, TIM_ExtTRGPolarity_NonInverted, 0x0F);
																//选择外部时钟模式2，时钟从TIM_ETR引脚输入
																//注意TIM2的ETR引脚固定为PA0，无法随意更改
																//最后一个滤波器参数加到最大0x0F，可滤除时钟信号抖动
/*时基单元初始化*/
TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;				//定义结构体变量
TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;		//时钟分频，选择不分频，此参数用于配置滤波器时钟，不影响时基单元功能
TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;	//计数器模式，选择向上计数
TIM_TimeBaseInitStructure.TIM_Period = 10 - 1;					//计数周期，即ARR的值
TIM_TimeBaseInitStructure.TIM_Prescaler = 1 - 1;				//预分频器，即PSC的值
TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;			//重复计数器，高级定时器才会用到
TIM_TimeBaseInit(TIM2, &TIM_TimeBaseInitStructure);				//将结构体变量交给TIM_TimeBaseInit，配置TIM2的时基单元	
```



#### 2.3.3 时钟输出比较





•PWM频率： Freq = CK_PSC / (PSC + 1) / (ARR + 1)

•PWM占空比： Duty = CCR / (ARR + 1)

•PWM分辨率： Reso = 1 / (ARR + 1)

![image-20240324091032111](C:\Users\Answerfour\AppData\Roaming\Typora\typora-user-images\image-20240324091032111.png)



```c
/*开启时钟*/
RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);			//开启TIM2的时钟
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);			//开启GPIOA的时钟

/*GPIO重映射*/
//	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);			//开启AFIO的时钟，重映射必须先开启AFIO的时钟
//	GPIO_PinRemapConfig(GPIO_PartialRemap1_TIM2, ENABLE);			//将TIM2的引脚部分重映射，具体的映射方案需查看参考手册
//	GPIO_PinRemapConfig(GPIO_Remap_SWJ_JTAGDisable, ENABLE);		//将JTAG引脚失能，作为普通GPIO引脚使用

/*GPIO初始化*/
GPIO_InitTypeDef GPIO_InitStructure;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;		//GPIO_Pin_15;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
GPIO_Init(GPIOA, &GPIO_InitStructure);							//将PA0引脚初始化为复用推挽输出	
                                                                //受外设控制的引脚，均需要配置为复用模式		

/*配置时钟源*/
TIM_InternalClockConfig(TIM2);		//选择TIM2为内部时钟，若不调用此函数，TIM默认也为内部时钟

/*时基单元初始化*/
TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;				//定义结构体变量
TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;     //时钟分频，选择不分频，此参数用于配置滤波器时钟，不影响时基单元功能
TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up; //计数器模式，选择向上计数
TIM_TimeBaseInitStructure.TIM_Period = 100 - 1;					//计数周期，即ARR的值
TIM_TimeBaseInitStructure.TIM_Prescaler = 720 - 1;				//预分频器，即PSC的值
TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;            //重复计数器，高级定时器才会用到
TIM_TimeBaseInit(TIM2, &TIM_TimeBaseInitStructure);             //将结构体变量交给TIM_TimeBaseInit，配置TIM2的时基单元

/*输出比较初始化*/
TIM_OCInitTypeDef TIM_OCInitStructure;	//定义结构体变量
TIM_OCStructInit(&TIM_OCInitStructure);	//结构体初始化，若结构体没有完整赋值
                                        //则最好执行此函数，给结构体所有成员都赋一个默认值
                                        //避免结构体初值不确定的问题
TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;				//输出比较模式，选择PWM模式1
TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;		//输出极性，选择为高，若选择极性为低，则输出高低电平取反
TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;	//输出使能
TIM_OCInitStructure.TIM_Pulse = 0;								//初始的CCR值
TIM_OC1Init(TIM2, &TIM_OCInitStructure);						//将结构体变量交给TIM_OC1Init，配置TIM2的输出比较通道1

/*TIM使能*/
TIM_Cmd(TIM2, ENABLE);			//使能TIM2，定时器开始运行
}

/**
* 函    数：PWM设置CCR
* 参    数：Compare 要写入的CCR的值，范围：0~100
* 返 回 值：无
* 注意事项：CCR和ARR共同决定占空比，此函数仅设置CCR的值，并不直接是占空比
*           占空比Duty = CCR / (ARR + 1)
*/
void PWM_SetCompare1(uint16_t Compare)
{
TIM_SetCompare1(TIM2, Compare);		//设置CCR1的值
}
```

#### 2.3.4 时钟输入捕获

PWMI接占空比

```c
/*输入捕获初始化*/
TIM_ICInitTypeDef TIM_ICInitStructure;							//定义结构体变量
TIM_ICInitStructure.TIM_Channel = TIM_Channel_1;				//选择配置定时器通道1
TIM_ICInitStructure.TIM_ICFilter = 0xF;							//输入滤波器参数，可以过滤信号抖动
TIM_ICInitStructure.TIM_ICPolarity = TIM_ICPolarity_Rising;		//极性，选择为上升沿触发捕获
TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;			//捕获预分频，选择不分频，每次信号都触发捕获
TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_DirectTI;	//输入信号交叉，选择直通，不交叉
TIM_ICInit(TIM3, &TIM_ICInitStructure);							//将结构体变量交给TIM_ICInit，配置TIM3的输入捕获通道

/*选择触发源及从模式*/
TIM_SelectInputTrigger(TIM3, TIM_TS_TI1FP1);					//触发源选择TI1FP1
TIM_SelectSlaveMode(TIM3, TIM_SlaveMode_Reset);					//从模式选择复位
                                                                //即TI1产生上升沿时，会触发CNT归零
TIM_PWMIConfig(TIM3, &TIM_ICInitStructure);						//将结构体变量交给TIM_PWMIConfig，配置TIM3的输入捕获通道
																	//此函数同时会把另一个通道配置为相反的配置，实现PWMI模式
```

编码器接口测速

```c
/*输入捕获初始化*/
	TIM_ICInitTypeDef TIM_ICInitStructure;							//定义结构体变量
	TIM_ICStructInit(&TIM_ICInitStructure);							//结构体初始化，若结构体没有完整赋值
																	//则最好执行此函数，给结构体所有成员都赋一个默认值
																	//避免结构体初值不确定的问题
	TIM_ICInitStructure.TIM_Channel = TIM_Channel_1;				//选择配置定时器通道1
	TIM_ICInitStructure.TIM_ICFilter = 0xF;							//输入滤波器参数，可以过滤信号抖动
	TIM_ICInit(TIM3, &TIM_ICInitStructure);							//将结构体变量交给TIM_ICInit，配置TIM3的输入捕获通道
	TIM_ICInitStructure.TIM_Channel = TIM_Channel_2;				//选择配置定时器通道2
	TIM_ICInitStructure.TIM_ICFilter = 0xF;							//输入滤波器参数，可以过滤信号抖动
	TIM_ICInit(TIM3, &TIM_ICInitStructure);							//将结构体变量交给TIM_ICInit，配置TIM3的输入捕获通道
	
/*编码器接口配置*/
TIM_EncoderInterfaceConfig(TIM3, TIM_EncoderMode_TI12, TIM_ICPolarity_Rising, TIM_ICPolarity_Rising);
																//配置编码器模式以及两个输入通道是否反相
																//注意此时参数的Rising和Falling已经不代表上升沿和下降沿了，而是代表是否反相
																//此函数必须在输入捕获初始化之后进行，否则输入捕获的配置会覆盖此函数的部分配置

/*TIM使能*/
TIM_Cmd(TIM3, ENABLE);			//使能TIM3，定时器开始运行
```

### 2.3 ADC

STM32F103C8T6 ADC资源：ADC1、ADC2，10个外部输入通道

### 2.4 DMA

### 2.5 USART





### 2.6 I2C

### 2.7 SPI

