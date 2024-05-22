# Cheat sheet for Bootloader Hands on - User application

```c
HAL_GPIO_TogglePin(LED_GREEN_GPIO_Port, LED_GREEN_Pin);
HAL_Delay(100);
HAL_GPIO_TogglePin(LED_BLUE_GPIO_Port, LED_BLUE_Pin);
HAL_Delay(100);
HAL_GPIO_TogglePin(LED_RED_GPIO_Port, LED_RED_Pin);
HAL_Delay(100);
```