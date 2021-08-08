#include <stdint.h>
#include <stdbool.h>
#include "inc/tm4c123gh6pm.h"
#include "inc/hw_memmap.h"
#include "inc/hw_types.h"
#include "driverlib/sysctl.h"
#include "driverlib/interrupt.h"
#include "driverlib/gpio.h"
#include "driverlib/timer.h"
// ACTIVATES THE SWITCH IN THE BOARD
#include "inc/hw_gpio.h"



int main(void)


{
    //INITIALIZE VARIABLES TO BE USED

    uint32_t ui32Period; //TIMER0_A
    uint32_t ui32APeriod; //TIMER0_B
    uint32_t ui32BPeriod; //TIMER1_A
    uint32_t ui32CPeriod; //TIMER2_A

    // INITIALIZE SYSTEM CLOCK (?)
    SysCtlClockSet(SYSCTL_SYSDIV_5|SYSCTL_USE_PLL|SYSCTL_XTAL_16MHZ|SYSCTL_OSC_MAIN);

    // ENABLE PORTS B,E,F, AND D.
    SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOB);
    SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOE);
    SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOF);
    SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOD);


    // Enable AS OUTPUT: Port B, D and E outputs for the traffic lights.
    GPIOPinTypeGPIOOutput(GPIO_PORTB_BASE, GPIO_PIN_4|GPIO_PIN_5);
    GPIOPinTypeGPIOOutput(GPIO_PORTE_BASE, GPIO_PIN_4|GPIO_PIN_5);
    GPIOPinTypeGPIOOutput(GPIO_PORTD_BASE, GPIO_PIN_0|GPIO_PIN_1);

    // Enable AS OUTPUT Port F for counter. On board led (an extra part of the system- a counter indicator).
    GPIOPinTypeGPIOOutput(GPIO_PORTF_BASE, GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3);

    // Switch 1 in the board; only one switch is needed.
    GPIOPinTypeGPIOInput(GPIO_PORTF_BASE, GPIO_PIN_4);

    //////// TIMER PART OF THE SYSTEM ////////

    //enable timers in the microcontroller (timer 0,1, and 2)
    SysCtlPeripheralEnable(SYSCTL_PERIPH_TIMER0);
    SysCtlPeripheralEnable(SYSCTL_PERIPH_TIMER1);
    SysCtlPeripheralEnable(SYSCTL_PERIPH_TIMER2);

    // idk what this does lol
    TimerConfigure(TIMER0_BASE, TIMER_CFG_PERIODIC);
    TimerConfigure(TIMER0_BASE, TIMER_CFG_PERIODIC);
    TimerConfigure(TIMER1_BASE, TIMER_CFG_PERIODIC);
    TimerConfigure(TIMER2_BASE, TIMER_CFG_PERIODIC);

    // variable for counter (not exactly sure what the value is)
    ui32Period = (SysCtlClockGet() / 1) / 2;
    ui32APeriod = (SysCtlClockGet() / 1) / 2;
    ui32BPeriod = (SysCtlClockGet() / 1) / 2;
    ui32CPeriod = (SysCtlClockGet() / 1) / 2;

    TimerLoadSet(TIMER0_BASE, TIMER_A, ui32Period -1);
    TimerLoadSet(TIMER0_BASE, TIMER_B, ui32APeriod -1);
    TimerLoadSet(TIMER1_BASE, TIMER_A, ui32BPeriod -1);
    TimerLoadSet(TIMER2_BASE, TIMER_A, ui32CPeriod -1);

    IntEnable(INT_TIMER0A);
    IntEnable(INT_TIMER0B);
    IntEnable(INT_TIMER1A);
    IntEnable(INT_TIMER2A);

    TimerIntEnable(TIMER0_BASE, TIMER_TIMA_TIMEOUT);
    TimerIntEnable(TIMER0_BASE, TIMER_TIMB_TIMEOUT);

    TimerIntEnable(TIMER1_BASE, TIMER_TIMA_TIMEOUT);
    TimerIntEnable(TIMER2_BASE, TIMER_TIMA_TIMEOUT);

    IntMasterEnable();
    /////////////////////////////////////////


    // SWITCH Settings Code (i.e. Switch 1 and 2 as PF0 and PF4)
    // MANDATORY PART. WITHOUT THIS THE SWITCH WON'T WORK
    HWREG(GPIO_PORTF_BASE + GPIO_O_LOCK) = GPIO_LOCK_KEY;
    HWREG(GPIO_PORTF_BASE + GPIO_O_CR) |= 0x01;
    HWREG(GPIO_PORTF_BASE + GPIO_O_LOCK) = 0;
    GPIODirModeSet(GPIO_PORTF_BASE, GPIO_PIN_4|GPIO_PIN_0, GPIO_DIR_MODE_IN);
    GPIOPadConfigSet(GPIO_PORTF_BASE, GPIO_PIN_4|GPIO_PIN_0, GPIO_STRENGTH_2MA, GPIO_PIN_TYPE_STD_WPU);

    while(1)
    {

        // ALWAYS ON STATE (STATE 0) (i.e. Highway Green light is always on while ped light is always red)
        // Microcontroller LED set to 0.

        // PORT B (TO TURN ON = 0x10 or 0x20; to turn off 0x00)
        GPIOPinWrite(GPIO_PORTF_BASE, GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3, 0); // COUNTER LED = 0;
        GPIOPinWrite(GPIO_PORTB_BASE, GPIO_PIN_4, 0x10);// B4 = Green HWY
        GPIOPinWrite(GPIO_PORTB_BASE, GPIO_PIN_5, 0x20);// B5 = Red Ped

        // PORT E (TO TURN ON = 0x10 or 0x20; to turn off 0x00)
        GPIOPinWrite(GPIO_PORTE_BASE, GPIO_PIN_4, 0x00);// E4 = Yellow PED
        GPIOPinWrite(GPIO_PORTE_BASE, GPIO_PIN_5, 0x00);// E5 = Green PED

        // PORT D (TO TURN ON = 0x01 or 0x02; to turn off 0x00)
        GPIOPinWrite(GPIO_PORTD_BASE, GPIO_PIN_0, 0x00);// D0 = Red HWY
        GPIOPinWrite(GPIO_PORTD_BASE, GPIO_PIN_1, 0x00);// D1 = Yellow HWY

        // PRESS SWITCH on the board
        if(GPIOPinRead(GPIO_PORTF_BASE, GPIO_PIN_4)==0x00){ //0x00 means the switch is turned on

            TimerEnable(TIMER1_BASE, TIMER_A); //blink blue led on board(microcontroller) for 1Hz
            SysCtlDelay(5 * (SysCtlClockGet() / 3 ));    //************************************ DELAY FOR 5s (SYSCTLCLOCKGET/3 IS A FORMULA)
            TimerDisable(TIMER1_BASE, TIMER_A); //turn off blue led

            // STATE 1
            TimerEnable(TIMER1_BASE, TIMER_A); //blink blue led on board for 1Hz
            GPIOPinWrite(GPIO_PORTB_BASE, GPIO_PIN_4, 0x00);// B4 = Green HWY
            GPIOPinWrite(GPIO_PORTD_BASE, GPIO_PIN_1, 0x02);// D1 = Yellow HWY ******activate
            GPIOPinWrite(GPIO_PORTD_BASE, GPIO_PIN_0, 0x00);// D0 = Red HWY

            GPIOPinWrite(GPIO_PORTE_BASE, GPIO_PIN_5, 0x00);// E5 = Green PED
            GPIOPinWrite(GPIO_PORTE_BASE, GPIO_PIN_4, 0x00);// E4 = Yellow PED
            GPIOPinWrite(GPIO_PORTB_BASE, GPIO_PIN_5, 0x20);// B5 = Red Ped     ******activate
            SysCtlDelay(5 * (SysCtlClockGet() / 3 ));    //************************************ DELAY FOR 5s
            TimerDisable(TIMER1_BASE, TIMER_A); //turn off blue led

            // STATE 2
            TimerEnable(TIMER2_BASE, TIMER_A); //blink red light on board for 1Hz
            GPIOPinWrite(GPIO_PORTB_BASE, GPIO_PIN_4, 0x00);// B4 = Green HWY
            GPIOPinWrite(GPIO_PORTD_BASE, GPIO_PIN_1, 0x00);// D1 = Yellow HWY
            GPIOPinWrite(GPIO_PORTD_BASE, GPIO_PIN_0, 0x01);// D0 = Red HWY ******

            GPIOPinWrite(GPIO_PORTE_BASE, GPIO_PIN_5, 0x20);// E5 = Green PED******
            GPIOPinWrite(GPIO_PORTE_BASE, GPIO_PIN_4, 0x00);// E4 = Yellow PED
            GPIOPinWrite(GPIO_PORTB_BASE, GPIO_PIN_5, 0x00);// B5 = Red Ped
            SysCtlDelay(10 * (SysCtlClockGet() / 3 ));    //************************************ DELAY FOR 10s
            TimerDisable(TIMER2_BASE, TIMER_A); //turn off red led

            //STATE 3
            TimerEnable(TIMER0_BASE, TIMER_A); //blink green led on board and yellow LED on bread board
            GPIOPinWrite(GPIO_PORTB_BASE, GPIO_PIN_4, 0x00);// B4 = Green HWY
            GPIOPinWrite(GPIO_PORTD_BASE, GPIO_PIN_1, 0x00);// D1 = Yellow HWY
            GPIOPinWrite(GPIO_PORTD_BASE, GPIO_PIN_0, 0x01);// D0 = Red HWY ******

            GPIOPinWrite(GPIO_PORTE_BASE, GPIO_PIN_5, 0x00);// E5 = Green PED
            GPIOPinWrite(GPIO_PORTB_BASE, GPIO_PIN_5, 0x00);// B5 = Red PED
            SysCtlDelay(10 * (SysCtlClockGet() / 3 ));     //************************************ DELAY FOR 10s
            TimerDisable(TIMER0_BASE, TIMER_A);

            //STATE 4
            TimerEnable(TIMER1_BASE, TIMER_A);//blink blue led on board
            GPIOPinWrite(GPIO_PORTB_BASE, GPIO_PIN_4, 0x00);// B4 = Green HWY
            GPIOPinWrite(GPIO_PORTD_BASE, GPIO_PIN_1, 0x00);// D1 = Yellow HWY
            GPIOPinWrite(GPIO_PORTD_BASE, GPIO_PIN_0, 0x01);// D0 = Red HWY******

            GPIOPinWrite(GPIO_PORTE_BASE, GPIO_PIN_5, 0x00);// E5 = Green PED
            GPIOPinWrite(GPIO_PORTE_BASE, GPIO_PIN_4, 0x00);// E4 = Yellow PED
            GPIOPinWrite(GPIO_PORTB_BASE, GPIO_PIN_5, 0x20);// B5 = Red Ped******DELAY FOR 5s
            SysCtlDelay(5 * (SysCtlClockGet() / 3 ));      //************************************ DELAY FOR 5s
            TimerDisable(TIMER1_BASE, TIMER_A);

        }
    }
}




// This void function blinks the led green light on board and the Yellow LED (Pedestrian)
void Timer0IntHandler()
{
    // Clear the timer interrupt
   TimerIntClear(TIMER0_BASE, TIMER_TIMA_TIMEOUT);

    // Read the current state of the GPIO pin and
    // write back the opposite state
    if(GPIOPinRead(GPIO_PORTF_BASE, GPIO_PIN_3))
    {
        GPIOPinWrite(GPIO_PORTF_BASE, GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3, 0);
        GPIOPinWrite(GPIO_PORTE_BASE, GPIO_PIN_4, 0x00);// E4 = Yellow PED

    }
    else
    {
        GPIOPinWrite(GPIO_PORTF_BASE, GPIO_PIN_3, 8);
        GPIOPinWrite(GPIO_PORTE_BASE, GPIO_PIN_4, 0x10);// E4 = Yellow PED

    }

}

// This function is broken; doesn't work
void Timer0BIntHandler()
{
    // Clear the timer interrupt
   TimerIntClear(TIMER0_BASE, TIMER_TIMB_TIMEOUT);

   // Read the current state of the GPIO pin and
   // write back the opposite state
   if(GPIOPinRead(GPIO_PORTF_BASE, GPIO_PIN_1))
   {
       GPIOPinWrite(GPIO_PORTF_BASE, GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3, 0);

   }
   else
   {
       GPIOPinWrite(GPIO_PORTF_BASE, GPIO_PIN_1, 2);

   }


}

// This function blinks the blue light
void Timer1AIntHandler()
{
    // Clear the timer interrupt
   TimerIntClear(TIMER1_BASE, TIMER_TIMA_TIMEOUT);

   // Read the current state of the GPIO pin and
   // write back the opposite state
   if(GPIOPinRead(GPIO_PORTF_BASE, GPIO_PIN_2))
   {
       GPIOPinWrite(GPIO_PORTF_BASE, GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3, 0);

   }
   else
   {
       GPIOPinWrite(GPIO_PORTF_BASE, GPIO_PIN_2, 4);

   }


}

// This function blinks the red light
void Timer2AIntHandler()
{
    // Clear the timer interrupt
   TimerIntClear(TIMER2_BASE, TIMER_TIMA_TIMEOUT);

   // Read the current state of the GPIO pin and
   // write back the opposite state
   if(GPIOPinRead(GPIO_PORTF_BASE, GPIO_PIN_1))
   {
       GPIOPinWrite(GPIO_PORTF_BASE, GPIO_PIN_1|GPIO_PIN_2|GPIO_PIN_3, 0);

   }
   else
   {
       GPIOPinWrite(GPIO_PORTF_BASE, GPIO_PIN_1, 2);

   }


}
