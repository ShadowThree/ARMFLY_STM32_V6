## 说明
1. 通过原理图可以发现，板子上的 `LED1~LED4` 不是直接由 `MCU` 的管脚控制的，而是 `PE11/FMC_D8`, `PE12/FMC_D9`, `PE13/FMC_D10`, `PE14/FMC_D11` 这四个信号经过了一个扩展芯片 `U9-74HC574PW` 间接控制的；
2. 通过 `U9-74HC574PW` 的手册可知，要让输入端口的状态更新到输出端口，需要给 `CP` 管脚一个上升沿（`OE`管脚已经拉低，供电也已经有了）；
3. 而 `U9-74HC574PW` 的 `CP` 管脚又是由 `U21-SN74HC32PW` 内部的 `或门 B` 控制的；
4. `U21-SN74HC32PW` 的 `或门 B` 有两个输入信号：`FMC_NE2_B` 和 `PD5/FMC_NWE`；
5. 通过查找发现，`FMC_NE2_B` 信号是由 `U17-SN74LV139APWR` 输出的；而 `PD5/FMC_NWE` 信号则直接来源于 `MCU` 的 `PD5` 管脚；
6. 根据 `U17-SN74LV139APWR` 的手册和原理图可知，`FMC_NE2_B` 信号由 `PG9/FMC_NF2`, `PG0/FMC_A10` 以及 `PG1/FMC_A11` 三个信号控制；
7. 为了更方便地给 `U9-74HC574PW` 的 `CP` 信号提供上升沿，我们设置 `FMC_NE2_B` 恒为低电平；这样只要控制 `PD5/FMC_NWE` 产生上升沿就可以了；
8. 为了设置 `FMC_NE2_B` 恒为低电平，根据手册，只需将 `PG9/FMC_NF2` 设置为低, `PG0/FMC_A10` 设置为高，以及 `PG1/FMC_A11` 设置为低；
9. 经过以上分析，相应控制 `LED` 状态，只需在初始化时将 `PG9/FMC_NF2` 设置为低, `PG0/FMC_A10` 设置为高，以及 `PG1/FMC_A11` 设置为低，然后对相应的 `LED` 控制管脚设置相应的电平，最后给 `PD5/FMC_NWE` 一个上升沿就可以了；

## 代码
```c
    // initial
    HAL_GPIO_WritePin(PG9_GPIO_Port, PG9_Pin, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(PG0_GPIO_Port, PG0_Pin, GPIO_PIN_SET);
    HAL_GPIO_WritePin(PG1_GPIO_Port, PG1_Pin, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(PD5_GPIO_Port, PD5_Pin, GPIO_PIN_RESET);

    while (1)
    {
        HAL_GPIO_TogglePin(LED1_GPIO_Port, LED1_Pin);
        HAL_GPIO_TogglePin(LED2_GPIO_Port, LED2_Pin);
        HAL_GPIO_TogglePin(LED3_GPIO_Port, LED3_Pin);
        HAL_GPIO_TogglePin(LED4_GPIO_Port, LED4_Pin);
        HAL_GPIO_WritePin(PD5_GPIO_Port, PD5_Pin, GPIO_PIN_SET);    // rising edge
        HAL_GPIO_WritePin(PD5_GPIO_Port, PD5_Pin, GPIO_PIN_RESET);
        HAL_Delay(1000);
    }
```