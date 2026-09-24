# Push Button Controlled LED Using STM32 Microcontroller

## Aim

To interface a push button with an STM32 microcontroller and control the state of an LED based on the push-button input.

## Apparatus Required

| S. No. | Component               |    Quantity |
| ------ | ----------------------- | ----------: |
| 1      | STM32 development board |           1 |
| 2      | Push button             |           1 |
| 3      | LED                     |           1 |
| 4      | 220–330 Ω resistor      |           1 |
| 5      | 10 kΩ resistor          |           1 |
| 6      | Breadboard              |           1 |
| 7      | Jumper wires            | As required |
| 8      | USB cable               |           1 |

## Pin Configuration

| Component   | STM32 Pin | Configuration              |
| ----------- | --------- | -------------------------- |
| LED         | PA5 (LD2) | Digital Output             |
| Push Button | PC13 (B1) | Digital Input with Pull-Up |

> **Note:** The push button is configured as **active LOW**. When the button is pressed, PC13 reads `GPIO_PIN_RESET`. When the button is released, PC13 reads `GPIO_PIN_SET`.

## Algorithm

1. Start the program.
2. Initialize the HAL library.
3. Configure the system clock to 64 MHz.
4. Enable the clocks for GPIOA and GPIOC.
5. Configure PA5 (LD2) as a digital output push-pull pin.
6. Configure PC13 (B1) as a digital input with an internal pull-up resistor.
7. Set the initial state of the LED on PA5 to OFF.
8. Initialize `last_toggle_time` to `0` and set the blink interval to `200 ms`.
9. Enter the infinite `while(1)` loop.
10. Read the state of the push button connected to PC13.
11. If the button is pressed (`GPIO_PIN_RESET`):

    * Check whether the 200 ms interval has elapsed.
    * If the interval has elapsed, update `last_toggle_time`.
    * Toggle the LED state using `HAL_GPIO_TogglePin()`.
12. If the button is released (`GPIO_PIN_SET`), turn the LED OFF using `HAL_GPIO_WritePin()`.
13. Repeat the button-state checking continuously.
14. The program continues to run indefinitely.

## Program

```c
/**
  ******************************************************************************
  * @file           : main.c
  * @brief          : Push Button Controlled LED using STM32
  ******************************************************************************
  */

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

/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{
    /* Reset of all peripherals, Initializes the Flash interface and Systick */
    HAL_Init();

    /* Configure the system clock to 64 MHz */
    SystemClock_Config();

    /* Initialize configured peripherals */
    MX_GPIO_Init();
    MX_USART2_UART_Init();

    /* Non-blocking LED blink tracking */
    uint32_t last_toggle_time = 0;
    const uint32_t blink_interval_ms = 200;

    /* Infinite loop */
    while (1)
    {
        /*
         * Active LOW push button:
         * Pressed  -> GPIO_PIN_RESET
         * Released -> GPIO_PIN_SET
         */
        if (HAL_GPIO_ReadPin(B1_GPIO_Port, B1_Pin) == GPIO_PIN_RESET)
        {
            /* Button held: toggle LED at 200 ms intervals */
            if (HAL_GetTick() - last_toggle_time >= blink_interval_ms)
            {
                last_toggle_time = HAL_GetTick();

                HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
            }
        }
        else
        {
            /* Button released: turn LED OFF */
            HAL_GPIO_WritePin(
                LD2_GPIO_Port,
                LD2_Pin,
                GPIO_PIN_RESET
            );
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

    /*
     * Configure HSI = 16 MHz
     * PLL output = 64 MHz
     */
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

    /* Configure CPU, AHB and APB clocks */
    RCC_ClkInitStruct.ClockType =
        RCC_CLOCKTYPE_HCLK |
        RCC_CLOCKTYPE_SYSCLK |
        RCC_CLOCKTYPE_PCLK1;

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

    /* Enable GPIOA and GPIOC peripheral clocks */
    __HAL_RCC_GPIOA_CLK_ENABLE();
    __HAL_RCC_GPIOC_CLK_ENABLE();

    /* Initial LED state = OFF */
    HAL_GPIO_WritePin(
        LD2_GPIO_Port,
        LD2_Pin,
        GPIO_PIN_RESET
    );

    /* Configure LED pin PA5 as output */
    GPIO_InitStruct.Pin   = LD2_Pin;
    GPIO_InitStruct.Mode  = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull  = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;

    HAL_GPIO_Init(LD2_GPIO_Port, &GPIO_InitStruct);

    /* Configure push button PC13 as input with pull-up */
    GPIO_InitStruct.Pin  = B1_Pin;
    GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
    GPIO_InitStruct.Pull = GPIO_PULLUP;

    HAL_GPIO_Init(B1_GPIO_Port, &GPIO_InitStruct);
}

/**
  * @brief  This function is executed i*
```
Result

Thus, the push button was successfully interfaced with the STM32 microcontroller, and the LED connected to PA5 was successfully controlled according to the push-button input on PC13.
