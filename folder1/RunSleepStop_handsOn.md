## Deactivate RTC WakeUp in MX_RTC_Init(void)
```c
  /* USER CODE BEGIN RTC_Init 2 */
  HAL_RTCEx_DeactivateWakeUpTimer(&hrtc);
  /* USER CODE END RTC_Init 2 */
```

## USER CODE 2 modification
```c
/* USER CODE BEGIN 2 */

  /* 1 - Measure consumption in Run LDO operation --------------------------------------------------*/
  HAL_Delay(1500);

  /* 2 - Measure consumption in Run SMPS operation --------------------------------------------------*/
  /* The SMPS regulator supplies the Vcore Power Domains */
  HAL_PWREx_ConfigSupply(PWR_SMPS_SUPPLY);
  HAL_Delay(1500);

  /* 3 - Measure consumption for Power-down Mode for Flash Banks ------------------------------------*/
  /* Enable the Power-down Mode for Flash Banks*/
  HAL_FLASHEx_EnablePowerDown(FLASH_BANK_2);
  HAL_Delay(1500);

  /* Enable ultra low power mode */
  HAL_PWREx_EnableUltraLowPowerMode();

  /* Enable the Autonomous Mode for the RTC Stop0/1/2 */
  __HAL_RCC_RTCAPB_CLKAM_ENABLE();

  /* Set RTC wakeup timer for 2s */
  HAL_RTCEx_SetWakeUpTimer_IT(&hrtc, 0, RTC_WAKEUPCLOCK_CK_SPRE_16BITS, 0);

  /* 4 - Measure consumption in SLEEP mode Full SRAM retention---------------------------------------*/
    /* Enter in SlEEP mode */
  HAL_SuspendTick();
  HAL_PWR_EnterSLEEPMode(PWR_LOWPOWERREGULATOR_ON, PWR_SLEEPENTRY_WFI);
  HAL_ResumeTick();

  /* 5 - Measure consumption in SLEEP mode with SRAM4 retention only --------------------------------*/
     /* Enter in SlEEP mode */
  HAL_PWREx_DisableRAMsContentRunRetention(PWR_SRAM1_FULL_RUN);
  HAL_PWREx_DisableRAMsContentRunRetention(PWR_SRAM2_FULL_RUN);
  HAL_PWREx_DisableRAMsContentRunRetention(PWR_SRAM3_FULL_RUN);

  HAL_SuspendTick();
  HAL_PWR_EnterSLEEPMode(PWR_LOWPOWERREGULATOR_ON, PWR_SLEEPENTRY_WFI);
  HAL_ResumeTick();

  /* 6 - Measure consumption in SLEEP mode with SRAM4 retention only and busses APBx/AHBx shutdown ----*/
     /* Enter in SlEEP mode */
  (*RCC).CFGR2 = (RCC_CFGR2_AHB1DIS | RCC_CFGR2_AHB2DIS1 | RCC_CFGR2_AHB2DIS2 |  RCC_CFGR2_APB1DIS | RCC_CFGR2_APB2DIS);
  HAL_SuspendTick();
  HAL_PWR_EnterSLEEPMode(PWR_LOWPOWERREGULATOR_ON, PWR_SLEEPENTRY_WFI);
  HAL_ResumeTick();

  /* 7 - Measure consumption in SLEEP mode with SRAM4 retention only and busses APBx/AHBx shutdown ----*/
  	  /*Keep live SRM4, PWR, RTC */
     /* Enter in SlEEP mode */
  CLEAR_REG(RCC->AHB1SMENR);
  CLEAR_REG(RCC->AHB2SMENR1);
  CLEAR_REG(RCC->APB1SMENR1);
  CLEAR_REG(RCC->APB1SMENR2);
  CLEAR_REG(RCC->APB2SMENR);

  HAL_SuspendTick();
  HAL_PWR_EnterSLEEPMode(PWR_LOWPOWERREGULATOR_ON, PWR_SLEEPENTRY_WFI);
  HAL_ResumeTick();

  /* 8 - Measure consumption in Stop 2 mode Full SRAM retention ---------------------------------------*/
  /* Enter in Stop 2 mode */
  HAL_PWREx_EnableRAMsContentRunRetention(PWR_SRAM1_FULL_RUN);
  HAL_PWREx_EnableRAMsContentRunRetention(PWR_SRAM2_FULL_RUN);
  HAL_PWREx_EnableRAMsContentRunRetention(PWR_SRAM3_FULL_RUN);
  HAL_PWREx_EnterSTOP2Mode(PWR_STOPENTRY_WFI);

  /* Disable RAM page(s) content lost in Stop mode (Stop 0, 1, 2, 3) */
  HAL_PWREx_DisableRAMsContentStopRetention(PWR_SRAM1_FULL_STOP_RETENTION);
  HAL_PWREx_DisableRAMsContentStopRetention(PWR_SRAM2_FULL_STOP_RETENTION);
  HAL_PWREx_DisableRAMsContentStopRetention(PWR_SRAM3_FULL_STOP_RETENTION);
  HAL_PWREx_DisableRAMsContentStopRetention(PWR_ICACHE_FULL_STOP_RETENTION);
  HAL_PWREx_DisableRAMsContentStopRetention(PWR_DCACHE1_FULL_STOP_RETENTION);
  HAL_PWREx_DisableRAMsContentStopRetention(PWR_DMA2DRAM_FULL_STOP_RETENTION);
  HAL_PWREx_DisableRAMsContentStopRetention(PWR_PERIPHRAM_FULL_STOP_RETENTION);
  HAL_PWREx_DisableRAMsContentStopRetention(PWR_PKA32RAM_FULL_STOP_RETENTION);

   /*enable AHB2 bus again because GPIO PC7 LED toggling*/
   (*RCC).CFGR2 &= ~(RCC_CFGR2_AHB2DIS1);

  /* USER CODE END 2 */
```
## While(1) loop modification
```c
  while (1)
  {
	    /* Led Toggling */
	    HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_7);
	    HAL_Delay(1500);
	    HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_7);

	    /* 9 - Measure consumption in Stop 2 mode reduced SRAM retention------------------------------*/
	    /* Enter in Stop 2 mode */
	    HAL_PWREx_EnterSTOP2Mode(PWR_STOPENTRY_WFI);
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
```

