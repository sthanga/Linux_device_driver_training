### To be learned:
### Interrupt
### interrupt and polling handing:
### interrupt handler
### Process context vs interrupt Context
### Writing Interrupt Handler(code)
### Maskable and Non-Maskable interrupt

#### what is interrupt in microcontroller / kernel
``` bash
An interrupt in a microcontroller is a signal that temporarily halts the normal execution of a program so that the microcontroller
can respond to an important or time-critical event. After handling the event, the microcontroller resumes its previous operation.
its is a signal that temporarly pauses the current program execution ans siwitch to a special routine called an interrupt service routine (ISR).
interrupt ma be like "button presses, sensor data or serial communication"
key temrs:

Interrupt Source:                  The event that causes the interrupt (e.g., external pin change, timer, UART data).
ISR (Interrupt Service Routine):   The special function that runs in response to the interrupt.
Interrupt Vector Table:            A table of addresses pointing to ISRs, indexed by interrupt type.
Masking:                           Enabling/disabling interrupts individually or globally.

Simple Example:
Imagine you're cooking (main program), and the doorbell rings (interrupt).
You stop cooking, go answer the door (ISR), then return to cooking when you're done.

```

### Benefits:
#### Efficiency: No need to continuously poll (check) for events.
#### Responsiveness: Faster reaction to real-time events.
#### Low Power: MCU can sleep and only wake up on interrupt.

``` bash
To make a GPIO pin act as an interrupt pin on a microcontroller, you need to configure it to generate an interrupt when a specific signal change
(e.g., rising edge, falling edge, or both) occurs. The exact steps and code depend on the microcontroller you are using (e.g., STM32, AVR, ESP32, etc.),
but the general process is similar.

General Steps to Configure GPIO as Interrupt:
1. Configure the GPIO pin as an input
2. Enable the interrupt on that pin
3. Set the trigger condition (rising/falling/both edges)
4. Write the ISR (Interrupt Service Routine)
5. Enable global and peripheral interrupts
```
###  Example: GPIO Interrupt on STM32 (Using HAL)

```c
  void MX_GPIO_Init(void)                         // function call
  {
      GPIO_InitTypeDef GPIO_InitStruct = {0};    
  
      __HAL_RCC_GPIOC_CLK_ENABLE();
  
      // Configure PC13 as input with interrupt
      GPIO_InitStruct.Pin = GPIO_PIN_13;
      GPIO_InitStruct.Mode = GPIO_MODE_IT_FALLING;  // Falling edge interrupt
      GPIO_InitStruct.Pull = GPIO_NOPULL;
      HAL_GPIO_Init(GPIOC, &GPIO_InitStruct);
  
      // Enable and set EXTI line 15_10 Interrupt in NVIC
      HAL_NVIC_SetPriority(EXTI15_10_IRQn, 2, 0);
      HAL_NVIC_EnableIRQ(EXTI15_10_IRQn);
  }
  
  // ISR (Callback function)
  void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
  {
      if (GPIO_Pin == GPIO_PIN_13)
      {
          // Handle interrupt event for PC13
      }
  }
  
  // Actual interrupt handler
  void EXTI15_10_IRQHandler(void)
  {
      HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_13);
  }
*******************************************************************************************************************
Example Summary (Platform-Independent Pseudocode):

// Pseudocode
enable_global_interrupts();

void gpio_interrupt_setup() {
    set_gpio_input(PIN);                       // Step 1
    set_interrupt_trigger(PIN, RISING_EDGE);   // Step 2
    enable_gpio_interrupt(PIN);                // Step 3
    enable_global_interrupts();                // Step 5
}

void ISR_for_PIN() {                           // Step 4
    // Handle interrupt
}

```
### To make a GPIO pin act as an interrupt pin
```bash
1. Configure the GPIO pin as Input
  gpio_pin.mode = INPUT;
2. Select the Interrupt Trigger Type
        Rising edge: 0 → 1
        Falling edge: 1 → 0
        Both edges: Any change
        Level (high or low): less common

  gpio_interrupt_trigger(pin, EDGE_FALLING);

3.Enable Interrupt for the GPIO Pin
    Associate the GPIO pin with an interrupt line (external interrupt).
    Enable that interrupt in the microcontroller’s interrupt controller (NVIC or similar).
enable_gpio_interrupt(pin);
enable_interrupt_in_controller(pin_interrupt_line);

4. Write the ISR (Interrupt Service Routine)
    This function is called when the interrupt occurs.
    Keep it short and fast — avoid long tasks or delay functions inside.
void gpio_pin_ISR() {
    // Code to run on interrupt
}
5. Enable Global Interrupts
    enable_global_interrupts();

6.Example Summary (Platform-Independent Pseudocode):

      void gpio_interrupt_setup() {
          set_gpio_input(PIN);                       // Step 1
          set_interrupt_trigger(PIN, RISING_EDGE);   // Step 2
          enable_gpio_interrupt(PIN);                // Step 3
          enable_global_interrupts();                // Step 5
      }
      
      void ISR_for_PIN() {                           // Step 4
          // Handle interrupt
      }
```

