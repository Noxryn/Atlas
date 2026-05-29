# 0.STM32简介
STM32是ST公司基于ARM Cortex-M内核开发的32位微控制器。
STM32常应用在嵌入式领域，如智能车、无人机、机器人、无线通信、物联网、工业控制、娱乐电子产品等
STM32功能强大、性能优异、片上资源丰富、功耗低，是一款经典的嵌入式微控制器
![stm_name.png](https://raw.githubusercontent.com/LiftHong/FigureBed/main/img/stm_name.png)
# 1.工程模板
1. 建立工程文件夹，Keil中新建工程，选择型号
2. 工程文件夹里建立Start、Library、User等文件夹，复制固件库里面的文件到工程文件夹
3. 工程里对应建立Start、Library、User等同名称的分组，然后将文件夹内的文件添加到工程分组里
4. 工程选项，C/C++，Include Paths内声明所有包含头文件的文件夹
5. 工程选项，C/C++，Define内定义USE_STDPERIPH_DRIVER
6. 工程选项，Debug，下拉列表选择对应调试器，Settings，Flash Download里勾选Reset and Run
# 2.C语言基础补充

| 关键字  | 位数  | 范围         | 关键字 |
| ---- | --- | ---------- | --- |
| char | 8   | -128 ~ 127 |     |
|      | 8   |            |     |
# 3.GPIO
定义：GPIO（General Purpose Input Output）通用输入输出口
1. 可配置为8种输入输出模式
2. 引脚电平：0V~3.3V，部分引脚可容忍5V（FT）
3. 输出模式下可控制端口输出高低电平，用以驱动LED、控制蜂鸣器、模拟通信协议输出时序等
4. 输入模式下可读取端口的高低电平或电压，用于读取按键输入、外接模块电平信号输入、ADC电压采集、模拟通信协议接收数据等
## 输入输出模式

|   名称   |  性质  |            特征             |          备注           |     |
| :----: | :--: | :-----------------------: | :-------------------: | --- |
|  浮空输入  | 数字输入 |   可读取引脚电平，若引脚悬空，则电平不确定    | GPIO_Mode_IN_FLOATING |     |
|  上拉输入  | 数字输入 | 可读取引脚电平，内部连接上拉电阻，悬空时默认高电平 |     GPIO_Mode_IPU     |     |
|  下拉输入  | 数字输入 | 可读取引脚电平，内部连接下拉电阻，悬空时默认低电平 |     GPIO_Mode_IPD     |     |
|  模拟输入  | 模拟输入 | GPIO无效，引脚直接接入内部ADC(ADC专属) |     GPIO_Mode_AIN     |     |
|  开漏输出  | 数字输出 |  可输出引脚电平，高电平为高阻态，低电平接VSS  |   GPIO_Mode_Out_OD    |     |
|  推挽输出  | 数字输出 |  可输出引脚电平，高电平接VDD，低电平接VSS  |   GPIO_Mode_Out_PP    |     |
| 复用开漏输出 | 数字输出 |  由片上外设控制，高电平为高阻态，低电平接VSS  |    GPIO_Mode_AF_OD    |     |
| 复用推挽输出 | 数字输出 |  由片上外设控制，高电平接VDD，低电平接VSS  |    GPIO_Mode_AF_PP    |     |
注：输出时可以输入，输入时不能输出

推挽输出：可以输出高,低电平,连接数字器件;

开漏输出:输出低电平. 要得到高电平状态需要上拉电阻才行. 
1. 适合于做电流型的驱动,其吸收电流的能力相对强(一般20ma以内).
2. 利用外部电路的驱动能力，减少IC内部的驱动。
3. 开漏是用来连接不同电平的器件，匹配电平用的，通过改变上拉电源的电压，便可以改变传输电平。（上拉电阻的阻值决定了逻辑电平转换的沿的速度 。阻值越大，速度越低功耗越小，所以负载电阻的选择要兼顾功耗和速度。）
3. 上升沿是通过外接上拉无源电阻对负载充电，所以当电阻选择小时延时就小，但功耗大；反之延时大功耗小。所以如果对延时有要求，则建议用下降沿输出。
4. 可以将多个开漏输出的Pin，连接到一条线上。通过一只上拉电阻，在不增加任何器件的情况下，形成“与逻辑”关系。这也是I2C，SMBus等总线判断总线占用状态的原理。

“线与”？：在所有引脚连在一起时，外接一上拉电阻，如果有一个引脚输出为逻辑0，相当于接地，与之并联的回路“相当于被一根导线短路”，所以外电路逻辑电平便为0，只有都为高电平时，与的结果才为逻辑1。

浮空输入：多用于外部按键输入，IO的电平状态是不确定的，完全由外部输入决定，如果在该引脚悬空的情况下，读取该端口的电平是不确定的。

复用开漏输出、复用推挽输出：GPIO口被用作第二功能时的配置情况（即并非作为通用IO口使用）
## 实现代码

```
//开启时钟
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);
//定义GPIO结构体 输出模式 引脚 频率
GPIO_InitTypeDef GPIO_InitS;
GPIO_InitS.GPIO_Mode = GPIO_Mode_Out_PP;
GPIO_InitS.GPIO_Pin = GPIO_Pin_0;
GPIO_InitS.GPIO_Speed = GPIO_Speed_50MHz;
//初始化
GPIO_Init(GPIOB,&GPIO_InitS);
//设置引脚高低电平
GPIO_Write(GPIOB,~0x0001); //0000 0000 0000 0001
//低电平
GPIO_ResetBits(GPIOB,GPIO_Pin_0);
GPIO_WriteBit(GPIOB,GPIO_Pin_0,Bit_RESET);
//高电平
GPIO_SetBits(GPIOB,GPIO_Pin_0);
GPIO_WriteBit(GPIOB,GPIO_Pin_0,Bit_SET);
```
# 外部中断
```
//配置
init()
{
	/*开启时钟*/
	//开启GPIOB的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);		
	//开启AFIO的时钟，外部中断必须开启AFIO的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);		
	
	/*GPIO初始化*/
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_14;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStructure); //上拉输入
	
	/*AFIO选择中断引脚*/
	//将外部中断的14号线映射到GPIOB，即选择PB14为外部中断引脚
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource14);
	
	/*EXTI初始化*/
	//定义结构体变量
	EXTI_InitTypeDef EXTI_InitStructure;
	//选择配置外部中断的14号线						
	EXTI_InitStructure.EXTI_Line = EXTI_Line14；
	//指定外部中断线使能
	EXTI_InitStructure.EXTI_LineCmd = ENABLE;
	//指定外部中断线为中断模式
	EXTI_InitStructure.EXTI_Mode = EXTI_Mode_Interrupt;	
	//指定外部中断线为下降沿触发
	EXTI_InitStructure.EXTI_Trigger = EXTI_Trigger_Falling;
	//将结构体变量交给EXTI_Init，配置EXTI外设		
	EXTI_Init(&EXTI_InitStructure);								
	
	/*NVIC中断分组*/
	//配置NVIC为分组2
    //即抢占优先级范围：0~3，响应优先级范围：0~3
    //此分组配置在整个工程中仅需调用一次
	//若有多个中断，可以把此代码放在main函数内，while循环之前
	//若调用多次配置分组的代码，则后执行的配置会覆盖先执行的配置
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);				
	
	/*NVIC配置*/
	//定义结构体变量
	NVIC_InitTypeDef NVIC_InitStructure;
	//选择配置NVIC的EXTI15_10线						
	NVIC_InitStructure.NVIC_IRQChannel = EXTI15_10_IRQn;
	//指定NVIC线路使能		
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;	
	//指定NVIC线路的抢占优先级为1			
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
	//指定NVIC线路的响应优先级为1	
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;	
	//将结构体变量交给NVIC_Init，配置NVIC外设		
	NVIC_Init(&NVIC_InitStructure);							
}

//中断处理函数
void EXTI15_10_IRQHandler(void)
{
	//判断是否是外部中断14号线触发的中断
	if (EXTI_GetITStatus(EXTI_Line14) == SET)
	{
		/*如果出现数据乱跳的现象，可再次判断引脚电平，以避免抖动*/
		if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_14) == 0)
		{
			/**/
		}
		//清除外部中断14号线的中断标志位
		EXTI_ClearITPendingBit(EXTI_Line14);		
		//中断标志位必须清除
	    //否则中断将连续不断地触发，导致主程序卡死
	}
}

```
# 定时器
