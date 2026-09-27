# STM32 / ARM Cortex Guide

## Series Overview
| Series | Core | Clock | Use Case |
|--------|------|-------|----------|
| STM32F0 | M0 | 48MHz | Ultra low-cost |
| STM32F1 | M3 | 72MHz | General purpose, legacy |
| STM32F4 | M4F | 180MHz | Performance + DSP + FPU |
| STM32F7 | M7 | 216MHz | High performance, LCD |
| STM32L4 | M4F | 80MHz | Low power + performance |
| STM32G4 | M4F | 170MHz | Mixed-signal, motor control |
| STM32H7 | M7 | 480MHz | Highest performance |

**Starter pick**: STM32F411 "BlackPill" — M4F, FPU, USB, 100MHz, Arduino-compatible.

## Clock Tree
- **HSE**: 4-26MHz crystal (accurate), **HSI**: 8-16MHz RC (no crystal)
- **PLL**: HSE → /M → ×N → /P → SYSCLK (use CubeIDE graphical configurator)
- **LSE**: 32.768kHz for RTC

## HAL vs LL vs Register
| Approach | When to Use |
|----------|-------------|
| HAL + CubeMX | Prototyping, complex peripherals (USB, Ethernet) |
| LL | Performance-critical paths |
| Register | Interrupt handlers, timing-critical (last resort) |

## Common HAL Patterns

### GPIO
```c
__HAL_RCC_GPIOA_CLK_ENABLE();
GPIO_InitTypeDef g = {.Pin=GPIO_PIN_5, .Mode=GPIO_MODE_OUTPUT_PP, .Speed=GPIO_SPEED_FREQ_LOW};
HAL_GPIO_Init(GPIOA, &g);
HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
```

### I2C (note: HAL uses 8-bit address = 7-bit << 1)
```c
HAL_I2C_Mem_Read(&hi2c1, 0x68<<1, 0x75, I2C_MEMADD_SIZE_8BIT, data, 2, 100);
```

### STM32duino Pin Naming
```cpp
digitalWrite(PA5, HIGH);  // Use port+pin, not D13
```

## Common Mistakes
1. Forgetting to enable peripheral clocks (`__HAL_RCC_GPIOx_CLK_ENABLE()`)
2. Wrong alternate function — check datasheet AF table
3. Not configuring clock tree (running at default HSI)
4. I2C address not shifted left by 1
5. DMA channel conflicts
6. Flash latency not set when increasing clock
