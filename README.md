# Review Questions

1. What is a hardware timer? What kinds of tasks do embedded systems commonly use hardware timers for?

A hardware timer is a circuit inside of a microcontroller that counts the clock pulses or external events to measure time and triggers without getting the CPU involved. Embedded systems commonly use hardware timers for Pulse-Width Modulation (PWM), Periodic Polling and Sampling, Operating System Ticks, frequency measurement, and to create precise delays.

2. What is a clock signal used for in embedded systems? List the primary clock sources available on the MSP432P401R microcontroller and briefly describe each clock source.

In embedded systems, a clock signal is used to synchronize all internal operations, data transfers, instructions, and peripheral timings.

Digitally Controlled Oscillator (DCO) - primary high-speed source for MCLK and SMCLK, capable of speeds of 48 MHz

Module Oscillator (MODOSC) - internal 25 MHz oscillator that provides low-latency wake-up source for internal modules

Reference Oscillator (REFO) - internal low-power reference oscillator that can output 32.768 kHz or 128 kHz, mainly used for ACLK and BCLK

Very Low-Power Low-Frequency Oscillator (VLO) - ultra-low-power internal oscillator that runs roughly at 9.4 kHz used for minimal power consumption

Low_Frequency Crystal Oscillator (LFXT) - external crystal oscillator input driven by 32.768 kHz quartz crystal, provides high-precision time and real-time clock

High-Frequency Crystal Oscillator (HFXT) - external crystal oscillator or ceramic resonator input with frequencies up to 48 MHz for precise high-speed systems

3. Consider a 16-bit timer (e.g. Timer_A configured in up mode). If it counts upward, what is the maximum count value before it rolls over? Show your work.

16-bit = 0xFFFF = 1111111111111111

MAX count = 2^16 - 1 = 65,535

4. If a 24-bit timer counts upward from zero at 10 MHz, what is its overflow period? Show your work.

24-bit = 0xFFFFFF = 111111111111111111111111

MAX count before overflow = 2^24 = 16,777,216

f = 10 MHz = 10,000,000 Hz

T = 1/f = 1/ 10,000,000 = 100 ns

Toverflow = 2^24*T = 1,677,721,600 ns = 1.6777216s ≅ 1.678s

5. Describe the SysTick timer on the MSP432P401R microcontroller. Provide two example applications for SysTick in embedded systems. Which clock sources can the SysTick timer use and how can they be selected?

The SysTick timer on the MSP432P401R microcontroller is a 24-bit, wrap-on-zero downward counter integrated directly into the ARM Cortex-M4F NVIC. It can use the external reference clock or the processor/system clock (MCLK). To choose the external clock, you would have to clear bit 2 in the CTRL register. To choose MCLK, you would set bit 2 in the CTRL register.

6. How many registers does the SysTick timer have on the MSP432P401R microcontroller and what is the purpose of each? Name them and provide a short description per register.

The SysTick timer has 4 registers. These registers are the CTRL, LOAD, VAL, and CALIB.

CTRL - configures the core operational behavior and monitors its execution state.

LOAD - specifies the initial value that will be loaded into the down-counter once it reaches zero

VAL - Monitors or resets the countdown in real-time

CALIB - read-only factory-configured reference value used for software calibration

7. Describe the Timer_A peripheral on the MSP432P401R microcontroller. List the main operating modes of Timer_A and briefly explain each.

The Timer_A peripheral is a versatile 16-bit timer with multiple capture/compare registers, the clock source is flexible, and can perform PWM.

Stop Mode - timer is stopped, used to pause or reset timer

Up Mode - timer counts from zero to whatever value is stored in TAxCCR register then resets and starts over

Continuous Mode - timer counts from zero to maximum 16-bit value then rolls over to zero and restarts

Up/Down Mode - behaves like the Up Mode, but it reverses its direction after reaching the value stored in the TAxCCR register

8. Briefly state the purpose of the following Timer_A registers
• TAxCTL - configures and controls the core timer operations

• TAxCCTLn - configures the specific behavior of capture/compare channel n, input capture or output compare

• TAxR - 16-bit register that holds the current count value of the timer

• TAxCCRn - holds 16-bit data value for channel n to be compared against TAxR to trigger an output change or interrupt, or captures timestamp from TAxR

• TAxIV - register used for fast ISR decoding

• TAxEX0 - secondary external clock prescaler divider, used to scale down incoming clock frequency

10. What is PWM? Provide at least three example applications that use PWM in embedded systems.

PWM is Pulse-Width Modulation and a technique used to control the average power from an electrical signal by rapidly switching a digital square wave between high and low. PWM can be used for DC motor speed control, LED brightness dimming, and servo motor angle positioning.

10. How many Timer_A modules are available and how many PWM outputs can each module provide?

There are four Timer_A modules available and they can provide up to 5 to 6 PWM outputs.

11. Describe the steps needed to initialize P2.4 as a PWM output driven by Timer_A0 CCR1 (TA0.1). Use the Timer_A2_PWM.c driver provided in Lab2 (Motor Control) as a reference.

Include C code such that:
- The P2.4 pin is configured to use its primary module function (Timer_A output)
- SMCLK is selected as the clock source for Timer_A
- The timer clock is divided by 2
- Up/Down mode is selected
- The output mode is configured as Toggle / Reset by setting the appropriate bits of the OUTMOD field in the CCTL[1] register
- The PWM signal has a period of 20 ms and a duty cycle of 50%

void Timer_A0_PWM_Init(void)
{

	// Configure pin P2.4 (TA0.1) to use peripheral function mode
	// Set Bit 4 of the SEL0 register and clear Bit 4 of the SEL1 register for P2
	P2->SEL0 |= 0x10;
	P2->SEL1 &= ~0x10;

	// Configure pin P2.4 as an output GPIO pin to drive the PWM signal
	// Set Bit 4 of the DIR register for P2
	P2->DIR |= 0x10;

	// Set the Timer A0 Capture/COmpare register to the specified period_constant
	// Formula: T = (2 * period_constant) / (12 MHz / Prescale Value)
	// To get a 20 ms period with a prescale divider of 2:
	// 0.020 s = (2 * period_constant) / (12,000,000 / 2) -> period_constant = 60000
	TIMER_A0->CCR[0] = 60000;

	// Configure the Timer A0 expansion register to divide the clock frequency by 1
	// Clear all bits of the EX0 register
	TIMER_A0->EX0 = 0x0000;

	// Configure the output mode as Toggle / REset for CCR[1]
	// Set the bits of the OUTMOD field of the CCTL[1] register to 010b (0x0040)
	TIMER_A0->CCTL[1] |= 0x0040;

	// Assign the value for a 50% duty cycle to the CCR[1] register
	// Duty Cycle = 50% = 30000
	TIMER_A0->CCR[1] = 30000;

	// Modify the following bits in the CTL register:
	// Select SMCLK = 12 MHz as timer clock source (TASSEL = 10b)
	// Set ID = 1 (Divide timer clock by 2, ID = 01b)
	// Set MC = 3 (Up/Down Mode, MC = 11b)
	TIMER_A0->CTL |= 0x0270;
}

12. Describe the steps needed to configure Timer_A0 to generate a periodic interrupt every 2 ms with an interrupt priority level of 2. Note that Timer_A0 has an interrupt number of 8. Use the Timer_A0_Interrupt.c driver provided in Lab 2 (Motor Control) as a reference.

Include C code such that:
- SMCLK is selected as the clock source for Timer_A with a prescaler value of 1 (which means the ID field will be 00b)
- The TAIDEX field in the TAxEX0 register has a value of 000b (divide by 1)
- Up mode is selected

// Function pointer to hold the user-defined task

void (*Timer_A0_Task)(void);

void (Timer_A0_Interrupt_Init(void(*task)(void), uint16_t period)
{

	// Store user-defined task to be used during interrupt handling
	Timer_A0_Task = task;

	// Pause Timer A0 by clearing MC bits in the CTL register
	TIMER_A0->CTL &= ~0x0030;

	// In the CTL register, set the TASSEL and ID bits
	// Choose SMCLK as timer clock source (TASSEL = 10b)
	// Choose prescale value of 1 (ID = 0)
	TIMER_A0->CTL |= 0x0200;

	// Enable interrupt request of the
	// corresponding Capture/Compare interrupt flag
	// in the CCTL[0] register by using the CCIE bit
	TIMER_A0->CCTL[0] |= 0x0010;

	// Store the period in the CCR0 register
	// Note: Timer starts counting from 0
	TIMER_A0->CCR[0] = (period - 1)

	// Divide the SMCLK frequency by 1 by setting the
	// TAIDEX bits of the EX0 register
	TIMER_A0->EX0 = 0x0000;

	// Set interrupt priority level to 2 using the IPR2 register of NVIC
	// Timer A0 has an IRQ number of 8 (Bits 7:0 of NVIC->IP[2])
	NVIC->IP[2] = (NVIC->IP[2] & 0xFFFFFF00) | 0x00000040;

	// Enable Interrupt 8 in NVIC by setting Bit 8 of the ISER register
	NVIC->ISER[0] |= 0x00000100;

	// Set the TACLR bit and enable Timer A0 in up mode using the
	// MC bits in the CTL register
	TIMER_A0->CTL |= 0x0014;
}

void Timer_A0_Stop(void)
{

	// Halt Timer A0 by clearing the MC bits in the CTL register
	TIMER_A0->CTL &= ~0x0030;

	// Disable Interrupt 8 in NVIC by setting Bit 8 of the ICER register
	NVIC->ICER[0] = 0x00000100
}

void TA0_0_IRQHandler(void)
{

	// Acknowledge Capture/COmpare interrupt and clear it
	TIMER_A0->CCTL[0] &= ~0x0001;
	
	// Execute the user-defined task
	(*Timer_A0_Task)();
}

# Sources
- Gemini
- Lecture Notes
- Motor Control Lab
- MSP432 Datasheet
