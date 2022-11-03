### STM32 入门

#### 软件安装与环境配置

- SetupSTM32CubeMX-6.2.0-Win.exe：STM32CubeMX 本体，免费
- jre-8u211-windows-x64.exe：JAVA 环境，免费
- MDK521A.exe：Keil5，开发环境，收费
- Keil.STM32F4xx_DFP.2.9.0.pack：F429 芯片支持包
- keygen_new2032.exe：破解 Keil5
- XCOM V2.3.exe：串口调试助手（正点原子开发）

#### GPIO 开发基础

- STM32 最多拥有 GPIOA、GPIOB……GPIOG 等 7 组端口。每组端口最多拥有 Pin0、Pin1……Pin15 共 16 个引脚。（最多拥有 7 * 16 = 112 个引脚）（有的引脚会直接命名为 PA3、PB3 这种，是端口和引脚号写在了一起）
- STM32 的每个 I/O 端口都可以自由编程，但 I/O 端口寄存器必须按 32 位字被访问。
- STM32 的每个 I/O 端口都由 7 个寄存器控制。
- STM32 的 I/O 端口可以由软件配置成 8 种模式。（通过 STM32CubeMX 里的图像化界面，配置好功能，直接生成初始化代码）
  - 输出
    - **推挽输出**：指普通的高低电平输出。
    - 开漏输出
    - 推挽式复用功能
    - 开漏式复用功能
  - 输入
    - 模拟输入（AD 转换的模拟信号）
    - 浮空输入
    - 下拉输入
    - 上拉输入

##### 三个 GPIO 输出的 HAL 库函数

- GPIO 电平输出 HAL 库函数

  ~~~c
  void HAL_GPIO_WritePin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);
  ~~~

  - GPIOx：目标引脚的端口号（填 A、B、C……）。
  - GPIO_Pin：目标引脚的引脚号。
  - PinState：高电平 `GPIO_PIN_SET`；低电平 `GPIO_PIN_RESET`。

  【例】：向 PB8 引脚输出高电平。

  ~~~c
   HAL_GPIO_WritePin(GPIOB, GPIO_PIN_8, GPIO_PIN_SET);
  ~~~

- GPIO 电平翻转 HAL 库函数

  ~~~c
  void HAL_GPIO_TogglePin(GPIO_Type* GPIOx, unit16_t GPIO_Pin);
  ~~~

  【例】：将 PA3 引脚输出电平翻转。

  ~~~c
  HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_3);
  ~~~

- GPIO 电平输入 HAL 库函数

  ```c
  GPIO_PinState HAL_GPIO_ReadPin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin); //具有返回值
  ```

  【例】：判断 PC13 引脚的输入信号，若为高电平，则将 PB9 引脚控制的 LED 等的开关状态切换。

  ~~~c
  if(HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_SET)
  {
      HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_9);
  }
  ~~~


##### GPIO 的重要数据结构

（待补充）

##### 实训：跑马灯实现流程

1. 打开 STM32CubeMX 软件。
2. 选择对应的 MCU：`ACCESS TO MCU SELECTOR`。
3. 把自己开发板芯片的型号输入到搜索框中，选中后，点击 `Start Project`。
4. 进行基本配置。
   1. System Core，SYS。配置仿真口。可选择 `SW` 或 `JTAG` 口。 
   2. System Core，RCC。配置时钟。选完后可以看到有针脚变颜色。然后还需配置**时钟树**。
5. 配置 LED 灯所对应的针脚。每个针脚只能选一个功能。
6. 配置完成后。对项目进行命名。选择开发环境 `MDK-ARM` `V5`。在代码器（Code Generator）勾选 **Generate peripheral initialization as a pair of '.c/.h' files per peripheral**。点击右上角 `GENERATE CODE`。

7. 打开文件夹，在 MDK-ARM 文件夹中，找到带有 Keil5 工程图标的文件，双击打开。

8. 打开项目后，先进行一次编译，目的是检查默认代码有没有问题，同时把 `main.c` 文件相关的头文件关联出来。

9. 应用代码写在以下注释中间。重新配置工程文件时，放在这里面的代码会被保留下来，其他代码会覆盖掉。

   ```C
   /* USER CODE BEGIN 0 */
   
   /* USER CODE END 0 */
   ```

10. 在 `int main(void)` 中的 `MX_GPIO_Init()` 写代码。

    ```c
    /* Infinite loop */
    /* USER CODE BEGIN WHILE */
    while(1)
    {
        /* USER CODE END WHILE */
        
        /* USER CODE BEGIN 3 */
    		
        /* LED0 RED 闪烁，方式 1，写高低电平。阿波罗的板子，低电平亮，高电平灭 */
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_RESET);
        HAL_Delay(1000);
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET);
        HAL_Delay(1000);
    		
        /* LED0 RED 闪烁，方式 2，写翻转电平 */
    	HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_1);
    	HAL_Delay(200);	
    }
    /* USER CODE END 3 */
    ```
    
    编译后，下载到开发板上。按下开发板的复位键，可以看到 `LED0 RED` 闪烁。

#### STM32 的按键开发基础

##### 基本原理

- 按键信号的识别：一般来说，按键的两个引脚的一端通过电阻上拉到高电平，另一端则接地。没有按键按下时，输入引脚为高电平。当有按键按下，输入引脚则为低电平。通过反复读取按键输入引脚的信号，然后识别高低电平来判断是否有按键触发。
- 去抖动：按键的输入引脚有低电平产生不代表一定是有按键按下，也许是干扰信号。因此需要通过去抖动处理将这些干扰信号过滤，从而获得真实的按键触发信号。方法：首次检测到按键输入引脚有低电平后，稍作延时，再次读取该引脚，如果还是低电平，则确认有按键触发信号；否则，判断为干扰信号，不予处理。

##### 实训：按键控制 LED 灯开关

利用 STM32CubeMX 和 Keil5 进行 STM32 应用开发，完成以下功能：

1. 按下 KEY0（PH3）按键，松开后，切换 LED0（PB1）的开关状态。
2. 按下 KEY1（PH2）按键，切换 LED1（PB0）的开关状态。
3. 按下 KEY2（PC13）按键，把点亮的 LED 灯全部关闭。

~~~c
/* USER CODE BEGIN 0 */
#define KEY0 HAL_GPIO_ReadPin(GPIOH, GPIO_PIN_3)
#define KEY1 HAL_GPIO_ReadPin(GPIOH, GPIO_PIN_2)
#define KEY2 HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13)

void Scan_Keys()
{
	if(KEY0 == 0)
	{
		HAL_Delay(100);                           //直接用 HAL 库的延时函数
		if(KEY0 == 0)                             //按下为低电平，可以直接用 0 表示，不过最好还是用 GPIO_PIN_RESET
		{
			while(KEY0 == 0);                     //while 很好用
			HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_1);
		}
	}
	
	if(KEY1 == 0)
	{
		HAL_Delay(100);
		if(KEY1 == 0)
		{
			HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_0);
			while(KEY1 == 0);
		}
	}
	
	if(KEY2 == 0)
	{
		HAL_Delay(100);
		if(KEY2 == 0)
		{
			HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0 | GPIO_PIN_1, GPIO_PIN_SET);
			while(KEY2 == 0);
		}
	}
}
/* USER CODE END 0 */
----------------------------------------------------------------------------
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    Scan_Keys();
	/* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
}
/* USER CODE END 3 */
~~~

#### STM32 的中断系统与外部中断基础

##### 名词扫盲

- 中断：
- 中断源：
- 中断向量：
- 中断优先级：
- 中断服务函数：

##### 中断系统

ARM Cortex M3 内核支持 256 个中断，包括 16 个内核中断和 240 个外设中断，拥有 256 个中断优先级别。

STM32 的中断通道可能会由多个中断源共用。这就意味着，某一个中断服务函数也可能被多个中断源所共用。所以，在中断服务函数的入口处，需要有一个判断机制，用以辨别是哪个中断出发了中断。

STM32 中有 2 个优先级的概念：抢占优先级和响应优先级，每个中断都需要指定这两种优先级。

Cortex M3 内核中有一个称为嵌套向量中断控制器（NVIC）的设备，对中断进行统一的协调和控制。其中最主要的工作就是控制中断使能和确定中断优先级。

##### 外部中断

外部中断 EXTI 是 STM32 芯片实时处理外部事件的一种机制，由于中断请求来自 GPIO 端口的引脚，所以称为外部中断。

STM32 芯片有 16 个外部中断源 EXTI0~EXTI15，分别对应这 7 个中断向量，也就是对应着 7 个中断服务函数。

- EXTI0、1、2、3、4：专用。（5:5）
- EXTI5~9：共用。（5:1）
- EXTI10~15：共用。（6:1）

EXTI0 的连接引脚是：PA0~PG0，即每个端口组的 0 号引脚。以此类推。

<img src="C:\Users\Obliviate\AppData\Roaming\Typora\typora-user-images\image-20221021112657259.png" alt="image-20221021112657259" style="zoom:50%;" />

外部中断触发条件：上升沿触发、下降沿触发或双边沿触发。注意：不能配置成高电平触发和低电平触发。

外部中断的程序设计思路：

传统的：

- 将 GPIO 初始化为输入端口。
- 配置相关 I/O 引脚与中断线的映射关系。
- 设置该 I/O 引脚对应的中断触发条件。
- 配置 NVIC，并使能中断。
- 编写中断服务函数。

基于 STM32CubeMX 的外部中断设计步骤：

- 在 STM32CubeMX 中指定引脚，配置中断初始化参数。
- 重写该 I/O 引脚对应的中断回调函数。

【例】将 PC13 引脚设置为外部中断，下降沿触发，在中断服务函数中，翻转 PB9 引脚的电平信号。1. 

1. 初始化配置
   1. 将 GPIO 设置为：GPIO_EXTI 功能。
   2. 设置中断触发条件：上升沿、下降沿、上升沿或下降沿。
   3. 使能相关的 NVIC 通道。
2. 中断服务函数编写

##### 实训：外部中断信号控制 LED 灯开关

利用 STM32CubeMX 和 Keil5 进行 STM32 应用开发，完成以下功能：

1. 将 KEY0，即 PH3 设置为外部中断输入，下降沿触发。在中断服务函数中，切换 LED0（PB1）的开关状态。
2. 将 KEY1，即 PH2 设置为外部中断输入，上升沿触发。在中断服务函数中，切换 LED1（PB0） 的开关状态。

【注】这个题目的代码很简单，重写虚函数（weak void）即可。主要在于 STM32 中的参数配置，这里折腾了很久，调试经验如下：

- 配置 RCC：选择 Crystal/Ceramic Resonator 即可，不需要去 Clock Configuration 中进行修改。（原理之后学到来补充，占坑）
- 配置 GPIO：上升/下降沿分别是 External Interrupt Mode with Rising/Falling edge trigger detection。一定都选择上拉（Pull-up）。
- 最后按键按的时候，可能会出现时亮时不亮，这是因为没有进行按键消抖，触发两次或者是没有触发。

~~~c
/* USER CODE BEGIN 0 */

void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
	if(GPIO_Pin == GPIO_PIN_3)
	{
		HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_1);
	}
	
	if(GPIO_Pin == GPIO_PIN_2)
	{
		HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_0);
	}
}

/* USER CODE END 0 */
~~~

#### STM32 的定时器开发基础

##### 常见定时器资源

- 系统嘀嗒定时器 SysTick

  集成在 Cortex M3 内核。

- 看门狗定时器 WatchDog

- 实时时钟 RTC

- 基本定时器：TIM6、TIM7

- 通用定时器：TIM2、TIM3、TIM4、TIM5

  在基本定时器的基础上，实现输出比较、输入捕获、PWM 生成、单脉冲模式输出等功能。这类定时器最具代表性，使用也最广泛。

- 高级定时器：TIM1、TIM8

##### 通用定时器

STM32 的通用定时器是一个通过可编程预分频器（Prescaler）驱动的 16 位自动重装主计数器（Counter Period）构成。可以对内部时钟或触发源以及外部时钟或触发源进行计数。

- 基本工作原理

  首先，定时器时钟信号送入 16 位可编程预分频器（Prescaler），该预分配器系数为 0~65535 之间的任意数值。预分配器溢出后，会向 16 位的主计数器（Counter Period）发出一个脉冲信号。

  预分频器，本质上是一个加法计数器。预分频系数实际上就是加计数的溢出值。

- 定时器发生中断时间的计算方法

  定时时间 = （Prescaler + 1）X（Counter Period + 1）X 1/定时器时钟频率S

【例】时钟信号 1KHz，Prescaler 为 9，Counter Period 为 999，定时时间？

<img src="C:\Users\Obliviate\AppData\Roaming\Typora\typora-user-images\image-20221024105011992.png" alt="image-20221024105011992" style="zoom:50%;" />

##### STM32CubeMX 中关于 TIM 的配置

【例】时钟信号 32MHz，每隔 500ms 翻转一次 PB9 的输出电平

- 设置 Clock Source 时钟源
- 设置 Prescaler 和 Counter Period 参数
- 设置 NVIC 嵌套向量中断控制器

如何得到 500ms：32000 X 500 X 1/32000000 = 0.5s = 500ms

因此，Prescaler 为 31999，Counter Period 为 499。

##### 时钟树相关知识

- STM32F429 时钟源
  - HSI：高速内部时钟，RC 振荡器，16MHz
  - LSI：低速内部时钟，RC 振荡器，32KHz
  - HSE：高速外部时钟，4-26MHz
  - LSE：低速外部时钟，32.768KHz
- 系统时钟 SYSCLK 来源
  - HSI
  - HSE
  - PLL（Phase Locked Loop）：锁相环。用来统一整合时钟信号。
- 其他常用名词扫盲
  - AHB（Advanced High-performance Bus）总线
  - APB（Advanced Peripheral Bus）总线
  - HCLK：AHB 总线的时钟。
  - PCLK1：APB1 总线的时钟
  - PCLK2：APB2 总线的时钟
  - RTC 实时时钟：一个独立的定时/计数器。

##### 实训：外部中断信号控制 LED 灯开关

在 STM32F429 进行 STM32 应用开发，完成以下功能：

- 利用 TIM2 实现间隔定时，每隔 0.2s 将 LED0（PB1）的开关状态翻转。
- 利用 TIM3 实现间隔定时，每隔 1s 将 LED1（PB0）的开关状态翻转。
- 修改 TIM2 的初始化代码，改为每隔 0.5s 将 LED1 的开关状态翻转。（在 `time.c` 文件中，找到 TIM2 的配置代码 `void MX_TIM2_Init(void)`，把对应的 `htim2.Init.Period = 199;` 修改为 500 即可。

【注】同样代码很简单，把虚函数放到 `main.c` 文件中重写即可。主要是在 STM32CubeMX 中配置 TIM2、TIM3。

- 找到 TIM2/3，首先 Clock Source 设置为 Internal Clock；
- 因为是 32MHz，所以 Prescaler = 31999。
- 因为是 0.2s，所以 Period = 199（200ms -1）。
- 最后一定要勾选上 NVIC。

另外，写完虚函数后，要在 main 函数中启动 TIM2、TIM3。

```c
/* USER CODE BEGIN 0 */

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
	if(htim->Instance == TIM2)   //如果实例为 TIM2，就运行
	{
		HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_1);
	}
	
	if(htim->Instance == TIM3)
	{
		HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_0);
	}
}

/* USER CODE END 0 */
------------------------------------------------------------
/* USER CODE BEGIN 2 */

	HAL_TIM_Base_Start_IT(&htim2); //启动函数，前面&表示传地址
	HAL_TIM_Base_Start_IT(&htim3);

/* USER CODE END 2 */
```

#### STM32 的串口数据收发基础

##### 名词扫盲

- 并行/串行通信

- 单工、半双工、全双工

- 异步串行通信：通信双方在没有同步时钟的前提下，将一个字符（包括特定的附加位）按位进行传输的通信方式。

- 波特率：每秒钟传输的二进制位数，如 9600bps。

- TTL电平←→RS232：MAX3232 SP3232

  串口←→USB 接口：CH340 CP2012

- STM32 芯片的串口 USART 功能十分强大，但对于日常编程而言，使用最多的还是异步串行通信。

##### STM32CubeMX 中关于 USART 的配置

##### HAL 库中串口发送的重要函数

- 阻塞式发送函数（推荐使用）

  ~~~c
  HAL_StatusTypeDef HAL_UART_Transmit(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, unit32_t Timeout);
  ~~~

- 非阻塞式发送函数

  ~~~c
  HAL_StatusTypeDef HAL_UART_Transmit_IT(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size);
  ~~~

- 发送完毕中断回调函数

  ~~~c
  void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart);
  ~~~

【例】使用非阻塞式的串口发送函数，将发送缓存数组 dat_Txd 中的前 5 个数据发送到 USART1，在数据发送完成后，翻转 PB9 引脚的输出电平。

~~~c
HAL_UART_Transmit_IT(&huart1, dat_Txd, 5);
void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart);
{
    if(huart->Instance == USART1)
    {
        HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_9);
    }
}

/* 使用阻塞式串口发送函数 */
HAL_UART_Transmit(&huart1, dat_Txd, 5, 10000);
HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_9);
~~~

##### HAL 库中串口接收的重要函数

- 阻塞式发送函数（不推荐使用）

  ~~~c
  HAL_StatusTypeDef HAL_UART_Receive(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, unit32_t Timeout);
  ~~~

- 非阻塞式发送函数（推荐使用）

  ~~~c
  HAL_StatusTypeDef HAL_UART_Receive_IT(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size);
  ~~~

- 接收完成中断回调函数

  ~~~c
  void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart);
  ~~~

【例】使用非阻塞式的串口接收函数，接收USART1中的一个字节，将其保存在 dat_Rxd 变量中，在数据发送完成后，若该字节为 0x5A，则翻转 PB8 引脚的输出电平。

~~~c
HAL_UART_Receive_IT(&huart1, &dat_Rxd, 1);
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart);
{
    if(huart->Instance == USART1)
    {
        if(dat_Rxd == 0x5A)
        {
            HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_8);
        }
    }
}
~~~

##### 实训：外部中断信号控制 LED 灯开关

在 F429 中进行 STM32 应用开发，完成以下功能。

- 开机后，向串口 1 发送"Hello World!"。
- 串口 1 收到字节指令"0xA1"，关闭 LED0（PB1），发送"LED1 Closed!"。
- 串口 1 收到字节指令"0xA2"，打开 LED0（PB0），发送"LED1 Open!"。
- 在串口发送过程中，打开 LED1 作为发送数据指示灯。

~~~c
/* USER CODE BEGIN 0 */
#define LED0_ON() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_RESET);
#define LED0_OFF() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET);
#define LED1_ON() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_RESET);
#define LED1_OFF() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET);

uint8_t Tx_str1[] = "Hello World!\r\n";
uint8_t Tx_str2[] = "LED1 Open!\r\n";
uint8_t Tx_str3[] = "LED1 Closed!\r\n";
uint8_t Rx_dat = 0;                                       //创建一个接收的数据，先是 0

void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)   //发送完毕中断回调函数，用于接收指令
{
	if(huart->Instance == USART1)                         //先进行判断是不是串口 1
	{
		if(Rx_dat == 0xa1)                                //发送指令 A1
		{
			LED1_OFF();                                   //LED1 灭
			
			LED0_OFF();                                   //LED0 作为数据发送指示灯
			HAL_UART_Transmit(&huart1, Tx_str3, sizeof(Tx_str3), 10000);
			LED0_ON();
			
			HAL_UART_Receive_IT(&huart1, &Rx_dat, 1);     //还要再用一遍非阻塞式发送函数
		}
		else if(Rx_dat == 0xa2)
		{
			LED1_ON();
			
			LED0_OFF();
			HAL_UART_Transmit(&huart1, Tx_str2, sizeof(Tx_str2), 10000);
			LED0_ON();
			
			HAL_UART_Receive_IT(&huart1, &Rx_dat, 1);
		}
	}
}

/* USER CODE END 0 */
---------------------------------------------------------
/* USER CODE BEGIN 2 */
	HAL_Delay(1000);                                      //开机运行，发送 Hello World 表示开机了
	LED0_OFF();
	HAL_UART_Transmit(&huart1, Tx_str1, sizeof(Tx_str1), 10000);
	HAL_Delay(1000);
	LED0_ON();
	
	HAL_UART_Receive_IT(&huart1, &Rx_dat, 1);

/* USER CODE END 2 */
~~~

#### STM32 的定时器与串口综合训练

##### 关于常用函数 sprintf() 的用法

- sprintf()，指的是字符串格式化函数，把格式化的数据写入某个字符串中。

  ~~~c
  int sprintf(char*string, char *format[,argument,…]);
  ~~~

- 需要引用头文件：#include "stdio.h"。

【例】有一个表示温度的整型变量 tmp，现在要将其格式化为字符串“温度是：XX 摄氏度”，并将其通过串口 1 发送出去。

~~~c
uint8_t Str_buff[64];

sprintf((char*)Strbuff, "温度是：%d摄氏度", tmp); //%d 是占位符

HAL_UART_Transmit(&huart, Str_buff, sizeof(Str_buff), 0xFFFF); //简单来说，0x 后面的值为十六进制
~~~

##### 实训：定时器与串口综合训练
在 F429 中进行 STM32 应用开发，完成以下的功能。

- 开机后，LED0 与LED1 依次点亮，然后熄灭，进行灯光检测。（LED0 接到 STM32 的 PB1，LED1 接到 STM32 的 PB0，低电平点亮）

- 系统通过串口 1 向上位机发送一个字符串"STM32F429 欢迎您!"。

- LED0 作为一个秒闪灯，系统向上位机发送完字符串后，开始亮 0.5 秒，灭 0.5 秒...…循环闪烁，并开始启动系统运行时间的记录，其时分秒格式为"XX:XX:XX"。

- 上位机通过一个由 3 个字节组成的命令帧控制 LED1 灯的开关。该命令帧的格式为"0xBF 控制字 OxFB"。

  0xBF 为帧头，0xFB 为帧尾，控制字的定义如下：

  0xA1：打开 LED1，返回信息"XX:XX:XX LED1 打开"。

  0xA2：关闭 LED1，返回信息"XX:XX:XX LED1 关闭"。

  其他：返回信息"XX:XX:XX 这是一个错误指令!"。

~~~c
/* USER CODE BEGIN 0 */
#define LED0_ON() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_RESET)
#define LED0_OFF() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET)
#define LED1_ON() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_RESET)
#define LED1_OFF() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET)

#define LED0_TOG() HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_1)
#define LED1_TOG() HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_0)

uint8_t str1[] = "********* STM32F429 欢迎您！*********\r\n";
uint8_t hh = 0, mm = 0, ss = 0, ss05 = 0;                         //定义时分秒，以及 0.5s
uint8_t str_buff[64];                                             //定义一个字符串的缓冲数组，64 个字节
uint8_t Rx_dat[16];                                               //串口接收的数组，16 个字节

void Check_LED()                                                  //灯光检测的函数
{
	HAL_Delay(1000);
	
	LED0_OFF();
	HAL_Delay(500);
	LED1_OFF();
	HAL_Delay(500);
	
	LED0_ON();
	HAL_Delay(500);
	LED1_ON();
	HAL_Delay(500);
}

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)        //回调函数（虚函数），用定时器做秒闪灯，LED0 闪烁
{
	LED0_TOG();
	
	ss05++;
	if(ss05 == 2)
	{
		ss05 = 0;
		ss++;
		if(ss == 60)
		{
			ss = 0;
			mm++;
			if(mm == 60)
			{
				mm = 0;
				hh++;
			}
		}
	}
}

void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	if(huart->Instance == USART1)
	{
		if(Rx_dat[0] == 0xBF && Rx_dat[2] == 0xFB)
		{
			switch(Rx_dat[1])
			{
				case 0xa1:
					LED1_OFF();
					sprintf((char *)str_buff, "%d:%d:%d    LED1 关闭！\r\n", hh, mm, ss);
				break;
				
				case 0xa2:
					LED1_ON();
					sprintf((char *)str_buff, "%d:%d:%d    LED2 打开！\r\n", hh, mm, ss);
				break;
				
				default:
					sprintf((char *)str_buff, "%d:%d:%d    这是一个错误的命令！\r\n", hh, mm, ss);
				break;
			}
			HAL_UART_Transmit(&huart1, str_buff, sizeof(str_buff), 10000);       //向串口发送缓冲区字符串
			HAL_UART_Receive_IT(&huart1, Rx_dat, 3);                             //写完发送，就要记得写接收，因为是为了继续接收
		}
	}
}
/* USER CODE END 0 */
----------------------------------------------------------------------
/* USER CODE BEGIN 2 */
    
	Check_LED();             //需要放到主函数中运行
	
	HAL_UART_Transmit(&huart1, str1, sizeof(str1), 10000);
	HAL_UART_Receive_IT(&huart1, Rx_dat, 3); //非阻塞式接收，接收到的字节放到 Rx_dat，当接收到完整的三个字节后，进入串口接收完成中断，然后调用它的回调函数。
	
	HAL_TIM_Base_Start_IT(&htim2);    //首先启动定时器，定时器里面是指针，所以要给个地址，即加上 &。找到回调函数（重写虚函数），写在上面

/* USER CODE END 2 */    
~~~

#### ADC 模数转换器的基本工作原理

##### 数字系统基本结构

<img src="C:\Users\Obliviate\AppData\Roaming\Typora\typora-user-images\image-20221025105706894.png" alt="image-20221025105706894" style="zoom:50%;" />

【例】一个恒温锅炉的基本结构：

1. 通过温度传感器，将温度变化转换为电压变化。
2. 通过 ADC 将模拟的电压变化，转换为数字变化，将其编码。
3. 中央处理器根据温度数据，进行计算和逻辑控制。
4. 计算结果通过 DAC 转换为电压/电流信号，控制加热和冷却。

##### Analog-to-Digital Converter

将时间和幅值连续的模拟量转化为时间和幅值离散的数字量，A/D 转换一般要经过采样、保持、量化和编码 4 个过程。

常用 ADC：逐次逼近型（大多数）、双积分型、$\Sigma-\Delta$ 型。

AD 转换器的几个技术指标：

- 量程（参考电压）：指 ADC 所能输入模拟信号的类型和电压范围。信号类型包括单极性和双极性（差分电压）。
- 转换位数：量化过程中的量化位数 n。AD 转换后的输出结果用 n 位二进制来表示。（如 10 位 AD 的输出值为 0~1023，即 1024）
- 分辨率：ADC 能够分辨的模拟信号最小变化量。
  - 公式：分辨率 = 量程 / $2^n$。（如量程为单极性 0-5V，8 位 ADC 的分辨率是：5/256 = 0.0195V，意味着能够分辨出 19.5mV 以上的信号变化）
- 转换时间：ADC 完成一次完整的 AD 转换所需要的时间，包括采样、保持、量化、编码的全过程。

##### ADC 数据采样的计算应用

有一个温度测控系统，已知温度传感器在 0 到 100 度之间为线性输出，参考电压为 5V，采用 8 位的 AD 转换器，0 度的时候，测的电压是 1.8V，100 度的时候，测的电压是 4.3V。问：系统的分辨率是多少？采集到数据 10010001，表示多大电压？温度是多少？

- 最小能分辨的电压：0.0195V。

- 由于温度是线性变化，先求得斜率 K = (100-0)/(4.3-1.8) = 40，得到温度（$y$）和电压（$x$）的关系表达式，$y=40\times(x-1.8)$。

  最小能分辨的温度：0.0195 * 40 = 0.78 度。

  10010001B = 91H = 145，所以 0.0195 * 145 = 2.83V。

  (2.83V - 1.8V) * 40 = 41.2 度。

#### STM32 的 ADC 开发基础

STM32F103ZE 芯片（144 脚）中有 ADC1、ADC2、ADC3 共 3 个 12（4096） 位逐次逼近型模数转换器，具有 18 个测量通道，可测量 16 个外部和 2 个内部信号源（内部温度和内部参考电压）。这两个内部信号源只能连接到 ADC1。

各个通道的 AD 转换可以单次、连续、扫描或间断模式执行。

按照 AD 转换的组织形式来划分，ADC 的模拟输入通道分为规则组和注入组两种。

- 规则组：ADC 可以对一组最多 16 个通道按照指定的顺序逐个进行转换，这组指定的通道称为规则组。
- 注入组：在实际应用中，可能需要中断规则组的转换，临时对某些通道进行转换，好像这些通道注入了原来的规则组，故称注入组，最多由 4 个通道组成。

AD 转换结果有 2 种存储方式：左对齐、右对齐（默认）。

##### STM32CubeMX 中关于 ADC 的配置

##### 查询方式和中断方式的 HAL 库函数应用

- 查询方式：阻塞式的 AD 转换

  ~~~c
  unit16_t ADC_Value = 0;                              //定义一个 16 位的变量，接收 ADC 返回的结果
  HAL_ADC_Start(&hadc);                                //启动 ADC，给它一个地址（通过参数告诉它启动的是哪一个 ADC）
  if(HAL_OK == HAL_ADC_PollForConversion(&hadc, 10))   //转换过程函数，第一个参数是表示转换哪个 AD，第二个参数是表示超时的时间，执行完毕返回一个 OK
  {
      ADC_Value = HAL_ADC_GetValue(&hadc);            //读取相应 AD 的转换结果。
  }
  ~~~

- 中断方式：非阻塞式的 AD 转换

  ~~~c
  uint16_t ADC_Value = 0;
  HAL_ADC_Start_IT(&hadc);                                  //带有中断功能的启动函数，会调用一个中断回调函数
  void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc1)   //AD 转换完成回调函数
  {
      ADC_Value = HAL_ADC_GetValue(&hadc);
  }
  ~~~
  
- DMA

##### 实训：ADC 单次数据采样与电压换算

在 F429 中进行 STM32 应用开发，完成以下的功能。

- 将 ADC_IN0 设置为 12 位 ADC，右对齐，启用中断。
- 分别用查询和中断这 2 种方式，每隔 0.5 秒采样一次 ADC 数据。
- 将每次读取到的 ADC 采样值转换为对应的电压值，发送到上位机。
- LED1 作为采样指示灯，在 ADC 转换过程中点亮，其余时间熄灭。

【注】配置 GPIO、ADC1、USART1。

1. GPIO 比较好配置，就 LED0、LED1 的引脚设为 Output。
2. 配置 ADC1，PA0 设为 ADC1_IN0，然后勾选 IN0。参数设置模块的 Data Alignment 默认 Right alignment（右对齐）即可，同样其他都默认，NVIC 中断使能勾选。
3. 配置 USART1，Mode 选 Asynchronous（异步），参数设置模块波特率改为 9600 Bits/s，其他默认。由于串口没有用到中断，所以 NVIC 模块不需要使能。

~~~c
/* USER CODE BEGIN Includes */
#include "stdio.h"                  //为后面 sprintf 做准备
/* USER CODE END Includes */
------------------------------------------------
/* USER CODE BEGIN 0 */
#define LED1_ON() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_RESET)
#define LED1_OFF() HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET)

uint16_t ADC_Value = 0, ADC_Volt = 0;  //定义一个 16 位的变量，接收 ADC1 的采样值和转换后的电压值
uint8_t str_buff[64];                  //具有 64 字节的缓冲区，发送

void UR1_Send_Info()                   //串口发送函数
{
	sprintf((char *)str_buff, "采样值：%d，电压值：%d.%d%d%dv\r\n", ADC_Value, ADC_Volt/1000, (ADC_Volt%1000)/100, (ADC_Volt%100)/10, ADC_Volt%10);       //视频中没有 /1000，是因为下面的 3300 用的是 330。这点要理解。下面少了个 0，所以上面跟着少了个 0，其实是为了计算简便。
	HAL_UART_Transmit(&huart1, str_buff, sizeof(str_buff), 1000);
}

/*方式2：中断方式
void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc)  //中断回调函数(启动函数在死循环中)
{
	if(hadc->Instance == ADC1)         //如果有多个 ADC，就要先用 if 来判断是不是 ADC1
	{
		ADC_Value = HAL_ADC_GetValue(&hadc1);   //得到 ADC1 的值
		ADC_Volt = ADC_Value * 3300 / 4096;     //转换公式，3300mv = 3.3v，4096 是 2 的 12 次方
		UR1_Send_Info();
		LED1_ON();      //在下面有写已经灭掉
	}
}
*/

/* 方式1：查询方式
void ADC1_Get_Value()                  //用来做换算
{
	HAL_ADC_Start(&hadc1);               //启动 ADC1
	LED1_OFF();                          //关闭 LED1，表示 ADC 开始转化
	
	if(HAL_ADC_PollForConversion(&hadc1, 10) == HAL_OK)  //转化函数
	{
		ADC_Value = HAL_ADC_GetValue(&hadc1);   //得到 ADC1 的值
		ADC_Volt = ADC_Value * 3300 / 4096;     //转换公式，3300mv = 3.3v，4096 是 2 的 12 次方
	}
	
	UR1_Send_Info();
	LED1_ON();
	
	HAL_ADC_Stop(&hadc1);     //关闭 ADC1
}
*/
/* USER CODE END 0 */
--------------------------------------------------
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    /*方式1：查询方式
	ADC1_Get_Value();        //查询方式函数执行
	HAL_Delay(500);
	*/
    
    /*方式2：中断方式
	LED1_OFF();                //中断启动前关掉 LED1
	HAL_ADC_Start_IT(&hadc1);  //放在死循环例，每隔 0.5s 以中断的方式启动 ADC1
	HAL_Delay(500);
	HAL_ADC_Stop_IT(&hadc1);   //关闭 ADC1
	*/
    
/* USER CODE END WHILE */

/* USER CODE BEGIN 3 */
}
/* USER CODE END 3 */

~~~



#### STM32 的 OLED 开发基础

##### OLED 概述

- Organic Light-Emitting Display，有机发光显示
- OLED 具备自发光、厚度薄、视角广、功耗低、对比度高、响应速度快、可用于挠曲性面板、使用温度范围广、构造及其制作过程较简单等优异特性，并认为是一种比液晶显示更为先进的新一代平板显示技术。
- 基于 STM32 的 OLED 应用，要做那些事情
  - 移植 OLED 的底层驱动函数库（针对不同的芯片，会有不同的库）
  - 准备需要的中文字符和图片等数据
  - 调用 OLED 驱动库中的底层函数进行应用开发

##### OLED 开发相关资源下载

- 基于 STM32CubeMX 的 OLED 屏驱动程序库（内含 4 个文件）
  - XMF_OLED_STM32Cube.c：驱动程序的源文件
  - XMF_OLED_STM32Cube.h：驱动程序的头文件
  - XMF_OLED_Font.h：字库数据文件
  - XMF_OLED_BMP.h：图片数据文件
- 取字模软件 PCtoLCD2002：生产字符和图片数据的工具

##### OLED 底层驱动函数移植

1. 将 4 个驱动文件拷贝到工程文件中，和 main.c 放在同一目录，并将 XMF_OLED_STM32Cube.c 添加到工程代码文件中，并在 main.c 中引入头文件 XMF_OLED_STM32Cube.h。
2. 根据所选用的芯片型号，修改 XMF_OLED_STM32Cube.h 头文件中所用的芯片头文件。
3. 根据硬件电路原理图，修改 XMF_OLED_STM32Cube.h 中 OLED 的引脚定义。
4. 查看 OLED_Init(void) 初始化函数的源码，根据电路接口和应用需要进行修改。

##### OLED 驱动库中常用的函数

- OLED 初始化

  ~~~c
  void OLED_Init(void);
  ~~~

  
