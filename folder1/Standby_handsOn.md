# Cheat sheet for STANDBY Hands on
PLay with STANDBY mode: enter, handling exit and keep PU/PD retention during STANDBY.

## USER CODE 2 modification
```c
 /* USER CODE BEGIN 2 */

  /*Check if reset is caused by Standby exit or not*/
  /* Enable Power Control clock */
    __HAL_RCC_PWR_CLK_ENABLE();
    if (__HAL_PWR_GET_FLAG(PWR_FLAG_SBF))
    {
      /* Clear Standby flag */
      __HAL_PWR_CLEAR_FLAG(PWR_FLAG_SBF);

#ifdef LEDRETAINED
      HAL_PWREx_DisablePullUpPullDownConfig();
#endif
      /* Check and Clear the Wakeup flag */
      if (__HAL_PWR_GET_FLAG(PWR_WAKEUP_FLAG2))
      {
        __HAL_PWR_CLEAR_FLAG(PWR_WAKEUP_FLAG2);
        HAL_Delay(100);
        HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_7);
        HAL_Delay(100);
        HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_7);
        HAL_Delay(100);
        HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_7);
        HAL_Delay(100);
        HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_7);
      }
    }

  /* The SMPS regulator supplies the Vcore Power Domains */
  HAL_PWREx_ConfigSupply(PWR_SMPS_SUPPLY);

  /* Enable ultra low power mode */
  HAL_PWREx_EnableUltraLowPowerMode();

#ifdef LEDRETAINED
  HAL_PWREx_EnableGPIOPullUp(PWR_GPIO_C, PWR_GPIO_BIT_7);
  HAL_PWREx_EnablePullUpPullDownConfig();
#endif

  HAL_Delay(2000);

  /* Enable WakeUp Pin PWR_WAKEUP_PIN2 connected to PC.13 */
  HAL_PWR_EnableWakeUpPin(PWR_WAKEUP_PIN2_HIGH_1);

  /* Clear all related wakeup flags*/
  __HAL_PWR_CLEAR_FLAG(PWR_WAKEUP_FLAG2);

  /* Enter the Standby mode */
  HAL_PWR_EnterSTANDBYMode();

  /* USER CODE END 2 */
```

## Add C switch in Private defines section
```c
/* USER CODE BEGIN PD */
#define LEDRETAINED
/* USER CODE END PD */
```


