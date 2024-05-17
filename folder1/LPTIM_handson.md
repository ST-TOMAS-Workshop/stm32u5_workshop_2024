# Cheat sheet for LPTIMx Hands on - PART 1
configure LPTIM1 a LPDMA to generate PWM on CH1:
- with frequency approximately 50 Hz (32.768 / 640)
- duty cycle will be changed in steps 0/25/50/75/100 approx. each second (each 50 PWM cycles)
- CCR1 will be modified at each update event
ARR shadow register will be set at rising edge of output signal => UE flag will be cleared at UE 

For proper application settings is neccessary follow the presentation, where all settings are mentioned.

## Settings for LPTIM1
Counter - Period
```c
640
```
Channel 1 - Pulse value
```c
640
```

## Settings for Linked-list queue for UPDATE EVENT 
Queue name
```c
LpTim1_Queue
```
First loop node name
```c
Ccr1Node
```
Runtime configuration - Source adress
```c
(uint32_t)LpTim1Duty
```
Runtime configuration - Destination adress
```c
(uint32_t)&((*LPTIM1).CCR1)
```
Runtime configuration - Data size
```c
DUTYSTEADY * DUTYCNT * 4
```

## Settings for Linked-list queue for LPTIM1.CH1  
Queue name
```c
LpTim1Arr_Queue
```
First loop node name
```c
ArrNode
```
Runtime configuration - Source adress
```c
(uint32_t)&ArrValue
```
Runtime configuration - Destination adress
```c
(uint32_t)&((*LPTIM1).ARR)
```
Runtime configuration - Data size
```c
4
```
## Create .sram4 section in STM32U575ZITXQ_FLASH.ld
```c
  .sram4 :
  {
    *(.sram4) 
  } >SRAM4

```
## Modify main.h
```c
/* USER CODE BEGIN EFP */
extern uint32_t LpTim1Duty[];
extern uint32_t ArrValue;
/* USER CODE END EFP */
```
```c
/* USER CODE BEGIN Private defines */
#define DUTYSTEADY 50
#define DUTYSTEP 160
#define DUTYCNT 5
/* USER CODE END Private defines */
```
## Modify linked_list.c
```c
/* USER CODE BEGIN Includes */
#include "main.h"
// __attribute__ ((section (".sram4")))
/* USER CODE END Includes */

DMA_NodeTypeDef Ccr1Node __attribute__ ((section (".sram4")));
DMA_QListTypeDef LpTim1_Queue __attribute__ ((section (".sram4")));
DMA_NodeTypeDef ArrNode __attribute__ ((section (".sram4")));
DMA_QListTypeDef __attribute__ ((section (".sram4")));
```
## Modify main.c
```c
/* USER CODE BEGIN Includes */
#include "linked_list.h"
/* USER CODE END Includes */
```
```c
/* USER CODE BEGIN PV */
LPTIM_OC_ConfigTypeDef sConfigOC = {0};

uint32_t LpTim1Duty[DUTYSTEADY * DUTYCNT] __attribute__ ((section (".sram4")));
uint32_t ArrValue __attribute__ ((section (".sram4")));

uint32_t i, j, k, value;
extern DMA_QListTypeDef LpTim1_Queue;
extern DMA_QListTypeDef LpTim1Arr_Queue;
/* USER CODE END PV */
```
```c
/* USER CODE BEGIN 2 */
value = 0;
for (i = 0; i < DUTYCNT; i++) {
    for (j = 0; j < DUTYSTEADY; j++) {
        LpTim1Duty[i * DUTYSTEADY + j] = value;
    }
    value += DUTYSTEP;
}
ArrValue = 640;

sConfigOC.Pulse = 160;
sConfigOC.OCPolarity = LPTIM_OCPOLARITY_HIGH;
if (HAL_LPTIM_OC_ConfigChannel(&hlptim1, &sConfigOC, LPTIM_CHANNEL_1) != HAL_OK)
{
    Error_Handler();
}

//Setup and initialization of Linked list DMA
for (i = 0; i < sizeof (DMA_QListTypeDef); i++) ((uint8_t *)&LpTim1_Queue)[i] = 0;
MX_LpTim1_Queue_Config();
HAL_DMAEx_List_LinkQ(&handle_LPDMA1_Channel0, &LpTim1_Queue);
//Enable Linked list DMA
HAL_DMAEx_List_Start(&handle_LPDMA1_Channel0);
//Setup and initialization of Linked list DMA
for (i = 0; i < sizeof (DMA_QListTypeDef); i++) ((uint8_t *)&LpTim1Arr_Queue)[i] = 0;
MX_LpTim1Arr_Queue_Config();
HAL_DMAEx_List_LinkQ(&handle_LPDMA1_Channel1, &LpTim1Arr_Queue);
//Enable Linked list DMA
HAL_DMAEx_List_Start(&handle_LPDMA1_Channel1);
HAL_LPTIM_PWM_Start(&hlptim1, LPTIM_CHANNEL_1);
(*LPTIM1).DIER |= LPTIM_DIER_UEDE;
/* USER CODE END 2 */
```