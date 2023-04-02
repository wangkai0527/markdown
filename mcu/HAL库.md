[TOC]

# HAL库


HAL_GPIO_Init
//初始化我们需要用到的引脚的工作模式，包括具体引脚的工作速度、是否复用模式、上下拉等等参数。

void HAL_GPIO_Init(GPIO_TypeDef *GPIOx, GPIO_InitTypeDef *GPIO_Init)

HAL_GPIO_DeInit
//将初始化之后的引脚恢复成默认的状态–各个寄存器复位时的值
void HAL_GPIO_DeInit(GPIO_TypeDef *GPIOx, uint32_t GPIO_Pin)
例：HAL_GPIO_DeInit(GPIOA, GPIO_PIN_9|GPIO_PIN_10);

HAL_GPIO_ReadPin
//读取我们想要知道的引脚的电平状态、函数返回值为0或1。
GPIO_PinState HAL_GPIO_ReadPin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
例：pin_State = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_9);

HAL_GPIO_WritePin
//给某个引脚写0或1，但是不要理解成，写1就是使能之类的意思，有些寄存器写1是擦除的意思
void HAL_GPIO_WritePin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
例：HAL_GPIO_WritePin(GPIOF, GPIO_PIN_9,GPIO_PIN_RESET)

HAL_GPIO_TogglePin
//翻转某个引脚的电平状态
void HAL_GPIO_TogglePin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
例：HAL_GPIO_TogglePin(GPIOF, GPIO_PIN_9);

HAL_GPIO_LockPin
//如果一个管脚的当前状态是1，读管脚值使用锁定，当这个管脚电平变化时保持锁定时的值，直到重置才改变
HAL_StatusTypeDef HAL_GPIO_LockPin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
例：

HAL_StatusTypeDef hal_State;
hal_State = HAL_GPIO_LockPin(GPIOF, GPIO_PIN_9);

HAL_GPIO_EXTI_IRQHandler
//这个函数是外部中断服务函数，用来响应外部中断的触发，函数实体里面有两个功能，1是清除中断标记位，2是调用下面要介绍的回调函数。实际调用的是下边的中断回调函数
void HAL_GPIO_EXTI_IRQHandler(uint16_t GPIO_Pin)
例：HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_3);

HAL_GPIO_EXTI_Callback
//中断回调函数，可以理解为中断函数具体要响应的动作。
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
例：HAL_GPIO_EXTI_Callback(GPIO_Pin);


