# Cheat sheet for Bootloader Hands on - Code

```c
/* USER CODE BEGIN Includes */
#include "menu.h"
#include "common.h"
/* USER CODE END Includes */
```

```c
/* USER CODE BEGIN PV */
extern pFunction JumpToApplication;
extern uint32_t JumpAddress;
/* USER CODE END PV */
```

```c
/* USER CODE BEGIN 2 */
if(HAL_GPIO_ReadPin(USER_BUTTON_GPIO_Port, USER_BUTTON_Pin) == 1 ){
  // Enter bootloader
  /* Initialise Flash */
  FLASH_If_Init();
  /* Display main menu */
  Main_Menu();
}
else{
  // run user application
  /* Test if user code is programmed starting from address "APPLICATION_ADDRESS" */
  if (((*(__IO uint32_t*)APPLICATION_ADDRESS) & 0x2FF30000 ) == 0x20000000)
  {
    /* Jump to user application */
    JumpAddress = *(__IO uint32_t*) (APPLICATION_ADDRESS + 4);
    JumpToApplication = (pFunction) JumpAddress;
    /* Initialize user application's Stack Pointer */
    __set_MSP(*(__IO uint32_t*) APPLICATION_ADDRESS);
    JumpToApplication();
  }
}
/* USER CODE END 2 */
```

```c
/* USER CODE BEGIN WHILE */
while (1)
{
Serial_PutString((uint8_t *)"No user app detected...\n\r");
HAL_Delay(500);
/* USER CODE END WHILE */
```
