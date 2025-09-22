# STM32 PWM LED Fading Example

## Description
This project demonstrates how to gradually turn an LED on and off using **PWM (Pulse Width Modulation)** on an STM32 microcontroller. The LED brightness is smoothly increased and decreased by modifying the **Compare Register (CCR)** value in a timer interrupt callback.

The implementation uses **TIM2** in PWM mode and a timer update interrupt to periodically adjust the duty cycle.

---

## Features
- PWM-based LED brightness control
- Smooth fading effect (increase and decrease)
- Uses Timer 2 update interrupt for CCR updates

---

## Hardware Setup
- STM32 microcontroller (any STM32 with TIM2 and GPIO available)
- LED connected to the PWM output pin (TIM2 Channel 1)
- Current-limiting resistor for the LED

---

## Code Overview

### Timer Initialization
```c
htim2.Instance = TIM2;
htim2.Init.Prescaler = 8-1;             // Timer prescaler
htim2.Init.CounterMode = TIM_COUNTERMODE_UP;
htim2.Init.Period = 1000-1;             // Auto-reload value
htim2.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
htim2.Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
HAL_TIM_Base_Init(&htim2);

HAL_TIM_PWM_Init(&htim2);
````

* Timer 2 is configured for PWM mode on Channel 1.
* `Period = 1000-1` defines the PWM frequency.
* `Prescaler = 7` slows down the timer clock to achieve desired frequency.

### PWM Configuration

```c
TIM_OC_InitTypeDef sConfigOC = {0};
sConfigOC.OCMode = TIM_OCMODE_PWM1;
sConfigOC.Pulse = 0;                     // Initial duty cycle
sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
HAL_TIM_PWM_ConfigChannel(&htim2, &sConfigOC, TIM_CHANNEL_1);
```

* Configures PWM channel mode and initial pulse width (duty cycle).

### Timer Interrupt Callback

```c
uint16_t comp = 0;
uint8_t up = 1;

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if(up){
        if(comp < 1000) comp++;
        else up = 0;
    }
    else{
        if(comp > 0) comp--;
        else up = 1;
    }
    __HAL_TIM_SetCompare(htim, TIM_CHANNEL_1, comp);
}
```

* This callback is called every time TIM2 counter reaches the `ARR` value.
* The `comp` variable is updated to gradually increase/decrease the PWM duty cycle.
* `__HAL_TIM_SetCompare` sets the new duty cycle for the LED.

### Starting PWM and Enabling Interrupt

```c
HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_1);
__HAL_TIM_ENABLE_IT(&htim2, TIM_IT_UPDATE);
```

* Starts PWM output on TIM2 Channel 1.
* Enables the update interrupt for TIM2.

---

## How It Works

1. **PWM Mode** generates a square wave with variable duty cycle.
2. **Timer Update Interrupt** periodically adjusts the duty cycle value (`CCR`).
3. `comp` variable increments or decrements, creating a smooth LED fade effect.

---

## Notes

* Adjust the `ARR` and `Prescaler` values to change PWM frequency.
* Adjust the increment/decrement step in the callback for faster or slower fading.
* Ensure the LED pin is connected to the corresponding TIM2 channel.
