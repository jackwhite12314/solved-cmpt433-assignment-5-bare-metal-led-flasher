Download Link: https://assignmentchef.com/product/solved-cmpt433-assignment-5-bare-metal-led-flasher
<br>



Submit deliverables to CourSys: <a href="https://courses.cs.sfu.ca/">https://courses.cs.sfu.ca/ </a>This assignment is to be done <strong>individually or in pairs</strong>.

Post questions to Piazza discussion forum.

Do not give your work to another student/group, do not copy code found online, and do not post questions about the assignment to other online forums.

You may use all sample code Dr. Fraser has posted on the course website this semester, and anything in the TI AM335x StarterWare package (must conform to license).

See the marking guide for details on how each part will be marked.

<h1>1.   Application Description</h1>

Create a bare metal application named lightBouncer.bin to run on the BeagleBone with a Zen cape which provides the following functionality:

<h2>1.1    Start-up</h2>

On the serial port at start-up:

<ol>

 <li>Print a welcome message with your name.</li>

 <li>Print the reset sources (and clear this register) (see page ~1336 in TRM).

  <ul>

   <li>The reset source register indicates all sources of resets since the register was cleared; print out a comma separated list of all the indicated reset sources.</li>

   <li>The reset source register is located at base address 0x44E00F00.</li>

   <li>Bits labeled “R/W1toClr-0h” mean that if you write a 1 to the bit it will clear it. You’ll need to do this after you have read the current reset source(s) in order to clean it up for the next reboot.</li>

   <li>Do not use any StarterWare methods to access this register (if you find any), other than the HWREG() You’ll likely have to define your own base and offset constants because they are not in StarterWare header files.</li>

   <li>Hint:</li>

   <li>Test with each of: cold boot (unplugging power), external reset (press the reset button beside the LEDs), and watchdog reboot.</li>

  </ul></li>

 <li>Print the help information about accepted serial commands.</li>

</ol>

<h2>1.2    10ms Timer</h2>

<ol>

 <li>Configure a timer for repeated 10ms interrupts.</li>

</ol>

<ul>

 <li>This interrupt will be used for multiple parts of the application.</li>

</ul>

<h2>1.3    Watchdog</h2>

<ol>

 <li>Configure the watchdog to reset the board after 5s.</li>

 <li>In the background task, hit the watchdog every 10ms.</li>

</ol>

<ul>

 <li>Hint: Have the timer ISR set a flag, which is checked in the main loop. Have the flag indicate that 10ms have elapsed, and hence will be time to hit the watchdog.</li>

</ul>

<ol start="3">

 <li>A serial port command can disable hitting the watchdog; more below.</li>

</ol>

<h2>1.4    Serial</h2>

<ol>

 <li>Use interrupts for the serial port to read a single character when a key is typed.

  <ul>

   <li>Use the StarterWare UARTCharGetNonBlocking() function to read the character.</li>

   <li>Do not require the user to hit ENTER.</li>

   <li><em>Hint</em>: You should only call IntAINTCInit() once in your entire program (it resets the ARM Interrupt Controller).</li>

   <li>This function will remove any currently configured interrupts, and is called in the sample code for both the serial port Rx and the timer (both using interrupts).</li>

   <li>Comment-out/delete the call from the second interrupt that you setup. For example, if your main() first initializes the timer and then the serial port, have the serial port <em>not</em> call IntAINTCInit().</li>

  </ul></li>

 <li>In the background task, process the key typed which was read in the ISR.

  <ul>

   <li>You may assume that the application will have completed processing the first keystroke before a second one is entered by the user (i.e, you don’t need a queue to read in key presses for processing: the CPU is fast, the serial port is <em>slow</em>).</li>

   <li><em>Hint</em>: Use a variable to store the key pressed in the ISR which is to be processed in the background task.</li>

   <li><em>Hint</em>: Have your background process repeatedly call a function in your serial prompt module to check if there is any work to be done.</li>

  </ul></li>

 <li>Accepted commands (must be case insensitive):</li>

</ol>

?   : Display this help message.

0-9 : Set speed 0 (slow) to 9 (fast).  a   : Select pattern A (bounce).  b   : Select pattern B (bar).  x   : Stop hitting the watchdog.

<ul>

 <li>When the user enters a command to change the speed or flash-pattern, call functions in your LED module (below).</li>

 <li>When the user enters ‘x’, stop hitting the watchdog in the background task (board should reboot due to the watchdog).</li>

 <li>If you enter an unknown command, print an error message and show the help list. <em>Hint</em><strong>:</strong> In addition to the h/.c module provided on the course web page in the “fake typer” program, also create a new serial prompt module which handles your your applications use of the serial port at a higher level.</li>

</ul>

<h2>1.5    LEDs</h2>

<ol>

 <li>Flash the four LEDs on the actual BeagleBone in one of two patterns:

  <ul>

   <li><strong>Bouncing:</strong> Turning on one LED at a time, make the lit LED appear to bounce back and forth. The illumination sequence will be (one time-period each): 0, 1, 2, 3, 2, 1, 0, 1, 2, 3, 2, 1, 0, ….</li>

   <li><strong>Bar:</strong> Turn on multiple LEDs, as needed, such that a bar of LEDs appears to grow and shrink. For one time-period each, turn on LEDs 0 through <em>N</em> (inclusive), where <em>N</em> = 0, 1, 2, 3, 2, 1, 0, 1, 2, 3, 2, 1, 0, ….</li>

  </ul></li>

 <li>The time-period for each flash is given by the following formula:</li>

</ol>

flash time-period = 10ms * (speed divider factor) where    speed divider factor = 2 <sup>(9 – speed)</sup>

<ol start="3">

 <li><em>Hints:</em>

  <ul>

   <li>Have the 10ms timer ISR call a function in your LED code which controls the LEDs.</li>

  </ul></li>

</ol>

<h2>1.6    Joystick</h2>

Pressing and releasing a direction on the joystick has the following effect:

<ol>

 <li>Left: Toggle the flash-mode between bounce (pattern A) and bar (pattern B).</li>

</ol>

i.e., which ever mode it is currently in, switch to the other mode.

<ol start="2">

 <li>Up (bonus): increments the speed by one (clamped to 9).</li>

 <li>Down (bonus): decrements the speed by one (clamped to 0).</li>

</ol>

Suggestion:

<ul>

 <li>Have the 10ms timer ISR call button call a function to check the state of the joystick.</li>

 <li>Note that holding the joystick has no effect; it is only on release that has an effect.</li>

 <li>When a joystick action is detected, call into the LED module to change the pattern/speed.</li>

 <li>If attempting the bonus:</li>

 <li>The board support package for the BeagleBone Black in StarterWare defines the following two function which you may want to call to initialize the GPIO: GPIO0ModuleClkConfig, GPIO1ModuleClkConfig.</li>

 <li>Find pin numbers for joystick directions by consulting Zen cape schematic. Find GPIO mapping via “header table” files <a href="http://www.cs.sfu.ca/CourseCentral/433/bfraser/other/BareMetalDocs/">on course website</a><a href="http://www.cs.sfu.ca/CourseCentral/433/bfraser/other/BareMetalDocs/">.</a></li>

</ul>

<h2>1.7    General Requirements</h2>

<ol>

 <li>Must have multiple modules, each presenting a clear interface to the rest of the code through a .h</li>

</ol>

Suggested modules include:

<ul>

 <li>controlling the LEDs,</li>

 <li>reading the joystick,</li>

 <li>managing the serial port prompt, and</li>

 <li>provided hardware abstraction modules: timer, watchdog, serial.</li>

</ul>

You may add more modules if it improves the readability of your code.

<ol start="2">

 <li>No global variables with external linkage.</li>

 <li>Only the functions exposed through .h files may be non-static (except main()). All other non-exported functions must be static.</li>

 <li>You must not have any printf-style statements called from within code that is called</li>

</ol>

while processing an interrupt (i.e., the ISR must not trigger a printf-style): <strong>All </strong><strong>printf’s must be done in the background task.</strong>

<ol start="5">

 <li>Code quality is always important:

  <ul>

   <li>All functions, variables, constants, and files must be well named.</li>

   <li>Use perfect indentation (as always!).</li>

   <li>Add a brief comment at the top of each .h file (at least) to explain that module’s purpose.</li>

  </ul></li>

 <li>Suggestion: Try to use the size-specific C types. For example use uint8_t instead of char, and uint32_t instead of unsigned int. (No marks for this, but good to work with; must #include &lt;stdint.h&gt;).</li>

</ol>




<h1>3.   Sample Serial Output</h1>

<table width="538">

 <tbody>

  <tr>

   <td width="538">LightBouncer:by Brian Fraser —————–Reset source (0x1) = Cold reset,Commands:?   : Display this help message.0-9 : Set speed 0 (slow) to 9 (fast).  a   : Select pattern A (bounce).  b   : Select pattern B (bar).  x   : Stop hitting the watchdog.  BTN : Push-button to toggle mode.?Commands:?   : Display this help message.0-9 : Set speed 0 (slow) to 9 (fast).  a   : Select pattern A (bounce).  b   : Select pattern B (bar).  x   : Stop hitting the watchdog.  BTN : Push-button to toggle mode. z Invalid command.Commands:?   : Display this help message.0-9 : Set speed 0 (slow) to 9 (fast).  a   : Select pattern A (bounce).  b   : Select pattern B (bar).  x   : Stop hitting the watchdog.  BTN : Push-button to toggle mode.0Setting LED speed to 03Setting LED speed to 3bChanging to bar mode. aChanging to bounce mode.aChanging to bounce mode. xNo longer hitting the watchdog.<em>&lt;&lt;&lt; Here the board actually reboots, after 5s&gt;&gt;&gt;</em></td>

  </tr>

 </tbody>

</table>


