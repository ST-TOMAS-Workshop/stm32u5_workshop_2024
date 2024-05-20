# Cheat sheet for LPTIMx Hands on - PART 2
configure LPTIM3 a LPDMA to measure period and duty cycle:<br>

- Period will be evaluated from two consecutive rising edges – CH1<br>
  
- Duty cycle will be evaluated from falling edge – CH2<br>

- Both flags of ICx will be cleared by LPDMA at IC2 event<br> 

For proper application settings is neccessary follow the presentation, where all settings are mentioned.

## Connection

LPTIM1.CH1 – PC1 – CN9.9 – A4<br>
LPTIM3.CH1 – PF5 – CN8.16 – PF5<br>
LPTIM3.CH2 – PF2 – CN9.17<br>
**Must be tied together**

## Settings for Linked-list queue for IC1 EVENT 

Queue name

```c
LpTim3Ic1_Queue
```

First loop node name

```c
Ic1Node
```

Runtime configuration - Source adress

```c
(uint32_t)&((*LPTIM3).CCR1)
```

Runtime configuration - Destination adress

```c
(uint32_t)RisingEdge
```

Runtime configuration - Data size

```c
2 * 4
```

## Settings for Linked-list queue for IC2 EVENT 

Queue name

```c
LpTim3Ic2_Queue
```

First loop node name

```c
Ic2Node
```

## Settings for Linked-list queue for IC2 EVENT - Ic2Node1

Runtime configuration - Source adress

```c
(uint32_t)&((*LPTIM3).CCR2)
```

Runtime configuration - Destination adress

```c
(uint32_t)&FallingEdge
```

Runtime configuration - Data size

```c
4
```

## Settings for Linked-list queue for IC2 EVENT - Ic2Node2

Runtime configuration - Source adress

```c
(uint32_t)&LpTim3Icr
```

Runtime configuration - Destination adress

```c
(uint32_t)&LpTim3Icr
```

Runtime configuration - Data size

```c
4
```

## Modify main.h

```c
/* USER CODE BEGIN EFP */
extern uint32_t LpTim1Duty[];
extern uint32_t ArrValue;
extern volatile uint32_t RisingEdge[];
extern volatile uint32_t FallingEdge;
extern uint32_t LpTim3Icr;
/* USER CODE END EFP */

```

## Modify linked_list.c

```c
DMA_NodeTypeDef Ccr1Node __attribute__ ((section (".sram4")));
DMA_QListTypeDef LpTim1_Queue __attribute__ ((section (".sram4")));
DMA_NodeTypeDef ArrNode __attribute__ ((section (".sram4")));
DMA_QListTypeDef LpTim1Arr_Queue __attribute__ ((section (".sram4")));
DMA_NodeTypeDef Ic1Node __attribute__ ((section (".sram4")));
DMA_QListTypeDef LpTim3Ic1_Queue __attribute__ ((section (".sram4")));
DMA_NodeTypeDef Ic2Node1 __attribute__ ((section (".sram4")));
DMA_QListTypeDef LpTim3Ic2_Queue __attribute__ ((section (".sram4")));
DMA_NodeTypeDef Ic2Node2 __attribute__ ((section (".sram4")));
```

## Modify main.c

```c
/* USER CODE BEGIN PV */
LPTIM_OC_ConfigTypeDef sConfigOC = {0};

uint32_t LpTim1Duty[DUTYSTEADY * DUTYCNT] __attribute__ ((section (".sram4")));
uint32_t ArrValue __attribute__ ((section (".sram4")));
volatile uint32_t RisingEdge[2] __attribute__ ((section (".sram4")));
volatile uint32_t FallingEdge __attribute__ ((section (".sram4")));
uint32_t LpTim3Icr __attribute__ ((section (".sram4")));

uint32_t i, j, k1, k, value, Period, Duty;
extern DMA_QListTypeDef LpTim1_Queue;
extern DMA_QListTypeDef LpTim1Arr_Queue;
extern DMA_QListTypeDef LpTim3Ic1_Queue;
extern DMA_QListTypeDef LpTim3Ic2_Queue;
/* USER CODE END PV */
```
This code add at the end **USER CODE 2** section
```c
LpTim3Icr = LPTIM_ICR_CC1CF | LPTIM_ICR_CC2CF;

//Setup and initialization of Linked list DMA
for (i = 0; i < sizeof (DMA_QListTypeDef); i++) ((uint8_t *)&LpTim3Ic1_Queue)[i] = 0;
MX_LpTim3Ic1_Queue_Config();
HAL_DMAEx_List_LinkQ(&handle_LPDMA1_Channel2, &LpTim3Ic1_Queue);

//Enable Linked list DMA
HAL_DMAEx_List_Start(&handle_LPDMA1_Channel2);

//Setup and initialization of Linked list DMA
for (i = 0; i < sizeof (DMA_QListTypeDef); i++) ((uint8_t *)&LpTim3Ic2_Queue)[i] = 0;
MX_LpTim3Ic2_Queue_Config();
HAL_DMAEx_List_LinkQ(&handle_LPDMA1_Channel3, &LpTim3Ic2_Queue);

//Enable Linked list DMA
HAL_DMAEx_List_Start(&handle_LPDMA1_Channel3);
(*LPTIM3).CCMR1 |= LPTIM_CCMR1_CC1E | LPTIM_CCMR1_CC2E;

if (HAL_LPTIM_IC_Start(&hlptim3, LPTIM_CHANNEL_1 | LPTIM_CHANNEL_2) != HAL_OK)
{
Error_Handler();
}
(*LPTIM3).DIER |= LPTIM_DIER_CC1DE | LPTIM_DIER_CC2DE;

```

```c
/* USER CODE BEGIN 3 */

// it is better to capture volatile values
  k1 = FallingEdge;
  i = RisingEdge[0];
  j = RisingEdge[1];
  k = FallingEdge;

  if ((k1 == k) && (i < k) && (k < j)) { // values are stable and in proper order
    Period = j - i; // rising n - rising n-1
    Duty = j - k; // rising - falling
  }

}
/* USER CODE END 3 */  
```
