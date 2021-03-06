# MSE352_TIVA_C_TrafficLight
Final project for MSE 352 (Digital Logic and Microcontrollers): A simple traffic light controller



## Introduction and description of the problem
In this project, we will be using a TI Launchpad TM4C123GXL, LEDs and a switch sensor to create a traffic light controller that involves a one-way highway and pedestrian crossing with a pedestrian light. The project will allow us to apply our understanding of finite state machine diagram and get hands on experience with programming an ARM MCU. The problem of the project is to create a coordinated traffic flow between the highway cars and pedestrian. If the highway traffic light is green or yellow the pedestrian light will be red. And if the pedestrian light is green or yellow the highway light will be red. In short, a real-life simulation of a highway. 


## Description of design objectives and design process
The description of the problem is as followed, the initial state of the traffic light will be green for the highway light and red for the pedestrian light. When the Pedestrian sensor is activated the highway traffic light will turn yellow for 5 second, and then turn red. When the traffic light turns red, the pedestrian light will turn green for 10 seconds and then turn yellow for another 10 seconds, after that the pedestrian light will turn red and the highway traffic will remain red for 5 seconds. This is to assure the pedestrian will safely cross the road. Afterwards the lights will return to the default setting where the highway light is green. The flow chart of the program is given in figure 2.


The assembly code implemented for the project was simple. The principle of using timers and interrupts were the groups option to achieve the desired characteristics of the circuit. The complete code will be attached along with this report for submission. In the main function, aside from enabling the GPIO pins for input and output (in-built led lights and pins for external leds), the timer of the microcontroller was also activated. In particular, 3 different timers were activated. Two of these three timers function the same- that is, they blink the in-built led of the microcontroller every second (i.e. with a frequency of 1 Hz). On the other hand, the third timer, aside from blinking the in-built led of the board, also blinks the external led (i.e. the yellow light for the pedestrian lane). The only difference between these timers is the color- it will be seen in the video that for different states, the in-built led for the board blinks with a different light. 

## Summary of the code
The code is divided into 4 parts- the main code and three void functions in charge in making the lights of the microcontroller blink. In the main part of the code, GPIOs and timers were activated and the variables associated with the timer are also defined. Inside this main function, a while loop is implemented- this while loop contains the main part of the system. As seen in the figure below, there are 5 states, each has different color of lights for the pedestrian and the highway lane. 

The algorithm for this is simple, enable the timer, enable the pins connected to the associated external led, delay so that the light will light up for a desired amount of time, and disable the associated timer. Overall, this algorithm is a simple yet it???s still a straightforward approach to tackle the problem presented.


![](https://github.com/bricc/MSE352_TIVA_C_TrafficLight/blob/master/StateDiagram_.JPG)
