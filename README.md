# Push-Button-Controlled-LED-Using-STM32-Microcontroller
https://github.com/new
## Aim
To interface an external push button with an STM32 microcontroller and control the state of an LED based on the push-button input.
## Apparatus Required
S. No. Component Quantity 1 STM32 development board 1 2 Push button 1 3 LED 1 4 220–330 Ω resistor 1 5 10 kΩ resistor 1 6 Breadboard 1 7 Jumper wires As required 8 USB cable 1
## Algorithm
Step 1: Start the program. Step 2: Initialize the HAL library, system clock (64 MHz), and UART peripheral. Step 3: Enable clocks for GPIO Port A and Port C. Step 4: Configure pin PA5 (LD2) as digital output push-pull and pin PC13 (B1) as digital input with internal pull-up. Step 5: Set the initial state of the LED (PA5) to OFF (GPIO_PIN_RESET). Step 6: Initialize tracking variable last_toggle_time = 0 and set blink_interval_ms = 200. Step 7: Enter the infinite loop (while(1)). Step 8: Read the button state at pin PC13 using HAL_GPIO_ReadPin(). Step 9: Check if the button is pressed (active LOW: GPIO_PIN_RESET): If Pressed: Check if (HAL_GetTick() - last_toggle_time) >= blink_interval_ms. If the condition is met, update last_toggle_time = HAL_GetTick() and toggle the LED state (HAL_GPIO_TogglePin()). If Released (GPIO_PIN_SET): Force the LED OFF immediately (HAL_GPIO_WritePin() to RESET). Step 10: Repeat from Step 8 continuously. Step 11: Stop (program loop runs indefinitely).
## program
```/* USER CODE BEGIN Header */
/**
  ******************************************************************************
  * @file           : main.c
  * @brief          : Main program body (Blink LED only on button hold)
  ******************************************************************************
  */
/* USER CODE END Header */

/* Includes ------------------------------------------------------------------*/
#include "main.h"

/* Board Pin Fallbacks if not configured in STM32CubeMX / main.h */
#ifndef B1_Pin
#define B1_Pin            GPIO_PIN_13
#define B1_GPIO_Port      GPIOC
#endif

#ifndef LD2_Pin
#define LD2_Pin           GPIO_PIN_5
#define LD2_GPIO_Port     GPIOA
#endif

/* Private variables ---------------------------------------------------------*/
UART_HandleTypeDef huart2;

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_USART2_UART_Init(void);

/* Private user code ---------------------------------------------------------*/

/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{
    /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
    HAL_Init();

    /* Configure the system clock (64 MHz for STM32G0) */
    SystemClock_Config();

    /* Initialize all configured peripherals */
    MX_GPIO_Init();
    MX_USART2_UART_Init();

    /* Non-blocking blink tracking variables */
    uint32_t last_toggle_time = 0;
    const uint32_t blink_interval_ms = 200; /* Toggle every 200 ms (500 ms full cycle) */

    /* Infinite loop */
    while (1)
    {
        /*
         * Active LOW pushbutton:
         * Pressed  -> GPIO_PIN_RESET
         * Released -> GPIO_PIN_SET
         */
        if (HAL_GPIO_ReadPin(B1_GPIO_Port, B1_Pin) == GPIO_PIN_RESET)
        {
            /* Button held: toggle LED at non-blocking intervals */
            if (HAL_GetTick() - last_toggle_time >= blink_interval_ms)
            {
                last_toggle_time = HAL_GetTick();
                HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
            }
        }
        else
        {
            /* Button released: force LED off immediately */
            HAL_GPIO_WritePin(LD2_GPIO_Port, LD2_Pin, GPIO_PIN_RESET);
        }
    }
}

/**
  * @brief System Clock Configuration for STM32G071xx
  * @retval None
  */
void SystemClock_Config(void)
{
    RCC_OscInitTypeDef RCC_OscInitStruct = {0};
    RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

    /* Configure the main internal regulator output voltage */
    HAL_PWREx_ControlVoltageScaling(PWR_REGULATOR_VOLTAGE_SCALE1);

    /* Initializes the RCC Oscillators (HSI -> 16 MHz, PLL -> 64 MHz) */
    RCC_OscInitStruct.OscillatorType      = RCC_OSCILLATORTYPE_HSI;
    RCC_OscInitStruct.HSIState            = RCC_HSI_ON;
    RCC_OscInitStruct.HSIDiv              = RCC_HSI_DIV1;
    RCC_OscInitStruct.HSICalibrationValue = RCC_HSICALIBRATION_DEFAULT;
    RCC_OscInitStruct.PLL.PLLState        = RCC_PLL_ON;
    RCC_OscInitStruct.PLL.PLLSource       = RCC_PLLSOURCE_HSI;
    RCC_OscInitStruct.PLL.PLLM            = RCC_PLLM_DIV1;
    RCC_OscInitStruct.PLL.PLLN            = 8;
    RCC_OscInitStruct.PLL.PLLP            = RCC_PLLP_DIV2;
    RCC_OscInitStruct.PLL.PLLR            = RCC_PLLR_DIV2;

    if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
    {
        Error_Handler();
    }

    /* Initializes the CPU and AHB/APB clocks (G0 has a single PCLK1 bus) */
    RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK | RCC_CLOCKTYPE_SYSCLK | RCC_CLOCKTYPE_PCLK1;
    RCC_ClkInitStruct.SYSCLKSource   = RCC_SYSCLKSOURCE_PLLCLK;
    RCC_ClkInitStruct.AHBCLKDivider  = RCC_SYSCLK_DIV1;
    RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;

    if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2) != HAL_OK)
    {
        Error_Handler();
    }
}

/**
  * @brief USART2 Initialization Function
  * @param None
  * @retval None
  */
static void MX_USART2_UART_Init(void)
{
    huart2.Instance                    = USART2;
    huart2.Init.BaudRate               = 115200;
    huart2.Init.WordLength             = UART_WORDLENGTH_8B;
    huart2.Init.StopBits               = UART_STOPBITS_1;
    huart2.Init.Parity                 = UART_PARITY_NONE;
    huart2.Init.Mode                   = UART_MODE_TX_RX;
    huart2.Init.HwFlowCtl              = UART_HWCONTROL_NONE;
    huart2.Init.OverSampling           = UART_OVERSAMPLING_16;
    huart2.Init.OneBitSampling         = UART_ONE_BIT_SAMPLE_DISABLE;
    huart2.Init.ClockPrescaler         = UART_PRESCALER_DIV1;
    huart2.AdvancedInit.AdvFeatureInit = UART_ADVFEATURE_NO_INIT;

    if (HAL_UART_Init(&huart2) != HAL_OK)
    {
        Error_Handler();
    }
}

/**
  * @brief GPIO Initialization Function
  * @param None
  * @retval None
  */
static void MX_GPIO_Init(void)
{
    GPIO_InitTypeDef GPIO_InitStruct = {0};

    /* GPIO Ports Clock Enable */
    __HAL_RCC_GPIOA_CLK_ENABLE();
    __HAL_RCC_GPIOC_CLK_ENABLE();

    /* LED initial state = OFF */
    HAL_GPIO_WritePin(LD2_GPIO_Port, LD2_Pin, GPIO_PIN_RESET);

    /* Configure LED pin (PA5) */
    GPIO_InitStruct.Pin   = LD2_Pin;
    GPIO_InitStruct.Mode  = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull  = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
    HAL_GPIO_Init(LD2_GPIO_Port, &GPIO_InitStruct);

    /* Configure Button pin (PC13) */
    GPIO_InitStruct.Pin  = B1_Pin;
    GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
    GPIO_InitStruct.Pull = GPIO_PULLUP;
    HAL_GPIO_Init(B1_GPIO_Port, &GPIO_InitStruct);
}

/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
    __disable_irq();
    while (1)
    {
    }
}

#ifdef USE_FULL_ASSERT
void assert_failed(uint8_t *file, uint32_t line)
{
}
#endif
```

## Output
<img width="576" height="581" alt="image" src="https://github.com/user-attachments/assets/1824232d-d27d-43da-b5ed-4351c987fd71" />

## Result
The push button was successfully interfaced with the STM32 microcontroller. The LED connected to PA5 turned ON when the push button connected to PA0 was pressed (logic HIGH) and turned OFF when the push button was released (logic LOW).


```
Result:
Thus the push button controlled LED by using STM32 Microcontroller is executed successfully
