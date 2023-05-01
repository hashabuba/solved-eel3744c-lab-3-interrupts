Download Link: https://assignmentchef.com/product/solved-eel3744c-lab-3-interrupts
<br>
<h1></h1>

<ul>

 <li>Understand general computer interrupt concepts and begin to utilize interrupts within the <em>ATxmega128A1U</em>.</li>

 <li>Learn how to use interrupts along with timer/counters to properly debounce a switch.</li>

</ul>

<h1>INTRODUCTION</h1>

Primarily, computer systems are comprised of multiple, independent components that communicate with one another.<a href="#_ftn1" name="_ftnref1"><strong><sup>[1]</sup></strong></a> For independence between components to remain effective, it is often desired that communication only take place when certain events are known to have occurred. To gain this ability, hardware components are generally designed to emit special digital signals, referred to here as <em>hardware flags</em> (or simply <em>flags</em>, if the context is clear<a href="#_ftn2" name="_ftnref2"><strong><sup>[2]</sup></strong></a>), that exactly identify the occurrence of specific events.<a href="#_ftn3" name="_ftnref3"><strong><sup>[3]</sup></strong></a>

For most computer systems, one generally flexible manner for handling an event of some peripheral component is to (1) identify an occurrence of the event through a hardware flag, either synchronously or asynchronously, and then (2) perform some set of instructions based on the event.<a href="#_ftn4" name="_ftnref4"><strong><sup>[4]</sup></strong></a>  To limit how much an application program is responsible for keeping track of event occurrences, there is often the ability to additionally configure what is typically known as an <em>interrupt</em>.  Essentially, an interrupt is <em>an automatic procedure for handling some event specified by a hardware flag</em>.<a href="#_ftn5" name="_ftnref5"><strong><sup>[5]</sup></strong></a><sup>, <a href="#_ftn6" name="_ftnref6"><strong>[6]</strong></a></sup>

During an interrupt, the desired set of instructions to handle the relevant event are executed within what is generally known as an <em>interrupt service routine </em>(<em>ISR</em>), or <em>interrupt handler</em>. The manner in which an interrupt service routine is configured depends on the computer architecture (as do all other aspects of interrupts), however it is often either that there be a predefined section of memory for the instructions of the ISR to reside or that there be a predefined memory location, known as an <em>interrupt vector</em>, to contain information regarding where the desired set of instructions reside.<a href="#_ftn7" name="_ftnref7"><strong><sup>[7]</sup></strong></a> In the former case, the “size” of an interrupt service routine, or the amount of instructions that could be performed during the routine, is more or less predefined by the computer system designer, whereas in the latter case, the level of indirection achieved by an interrupt vector potentially allows a user to configure an arbitrarily-sized routine.

<h1>LAB STRUCTURE</h1>

In this lab, you will begin to utilize interrupts and the programmable interrupt controller (more specifically, the <strong>p</strong>rogrammable <strong>m</strong>ultilevel <strong>i</strong>nterrupt <strong>c</strong>ontroller [<strong>PMIC</strong>]) within the <em>ATxmega128A1U</em>. In § 1, you will learn fundamental information regarding interrupts within the <em>ATxmega128A1U</em> and then learn to how utilize an interrupt signal for a timer/counter module. Following this, in § 2, you will learn how to utilize an I/O port interrupt so that responses to a tactile switch on the <em>OOTB SLB </em>can be made asynchronously, as well as learn how to efficiently debounce such a switch in an asynchronous environment.




<h1>REQUIRED MATERIALS                                       SUPPLEMENTAL MATERIALS</h1>

<ul>

 <li><a href="https://mil.ufl.edu/3744/docs/XMEGA/doc8331_XMEGA_AU_Manual.pdf">Atmel XMEGA AU Manual (doc8331)</a> <a href="https://mil.ufl.edu/3744/docs/XMEGA/doc0856_AVR_Instruction_Set.pdf">AVR Instruction Set (doc0856)</a></li>

 <li><a href="https://mil.ufl.edu/3744/docs/XMEGA/doc8385_ATxmega128A1U_Manual.pdf">Atmel </a><a href="https://mil.ufl.edu/3744/docs/XMEGA/doc8385_ATxmega128A1U_Manual.pdf"><em>ATxmega128A1U</em></a><a href="https://mil.ufl.edu/3744/docs/XMEGA/doc8385_ATxmega128A1U_Manual.pdf"> Manual (doc8385)</a></li>

 <li><em>OOTB µPAD v2.0</em> with USB A/B cable and accompanying schematic</li>

 <li><em>OOTB Switch &amp; LED Backpack </em>with accompanying schematic</li>

 <li><a href="https://mil.ufl.edu/3744/docs/switch_debouncing_with_software.pdf"><em>Switch Debouncing with Software</em></a></li>

 <li><strong>D</strong>igilent <strong>A</strong>nalog <strong>D</strong>iscovery (<strong>DAD</strong>) kit with <em>WaveForms </em>software</li>

</ul>




<h1>PRE-LAB PROCEDURE</h1>

<strong><u>REMINDER OF LAB POLICY</u></strong>

You must re-read the <a href="https://mil.ufl.edu/3744/admin/Lab%20Rules%20%26%20Policies.pdf"><em>Lab Rules &amp; Policies</em></a> before submitting any pre-lab assignment and before attending any lab.

<h2>1. INTRODUCTION TO INTERRUPTS</h2>

<table width="733">

 <tbody>

  <tr>

   <td width="389">Up until this point in this course, whenever it has been desired to determine when some event occurs within a peripheral component (e.g., when a tactile switch is pressed, when a TC counts for a specified amount of time, etc.), the CPU has been</td>

   <td width="344">necessary interrupt service routine, toggle an I/O port pin for which you have access to probe via the <em>µPAD</em>. Use your DAD to verify that the chosen pin toggles at the appropriate rate.</td>

  </tr>

 </tbody>

</table>

programmed to continually poll some input source. Ultimately, this technique wastes potentially useful processing time. In this <strong>NOTE: </strong>An interrupt-driven program should have a format section of the lab, you will be introduced to interrupts by similar to what is shown in Figure 1.

implementing a simple overflow (OVF) interrupt for a timer/counter module within the <em>ATxmega128A1U</em>. However, before this, you must research all relevant information regarding interrupts within the <em>ATxmega128A1U</em>.

1.1. Read § 12 (<em>Interrupts and Programmable Multilevel Interrupt Controller</em>) of the 8331 manual, as well as § 14 (<em>Interrupts and Programmable Multilevel Interrupt Controller</em>) of the 8385 manual, to understand how interrupts are managed within the <em>ATxmega128A1U</em>.

1.2. Read all parts of § 14 (<em>TC0/1 – 16-bit Timer/Counter Type </em>

<em>0 and 1</em>) within the 8331 manual regarding timer/counter                <strong><u>PRE-LAB EXERCISES</u></strong><strong>  </strong>

interrupts.                                                                                                      i.               Assuming that no interrupt has been previously configured,

1.3. Create a simple assembly program, <strong>lab3_1.asm</strong>. Within devise and describe a generalized series of steps for this program, configure a timer/counter to trigger an configuring any interrupt within the <em>ATxmega128A1U</em>, i.e., overflow (OVF) interrupt every 250 ms. Within the not just an interrupt within the TC system.

<h2>2. INTERRUPTS, CONTINUED</h2>




In this section of the lab, you will learn how to configure an I/O port interrupt so that responses to a tactile switch on the <em>OOTB SLB </em>can be made asynchronously. Additionally, you will learn how to efficiently debounce the relevant switch within the asynchronous context.

2.1. Read the relevant parts within § 13 (<em>I/O Ports</em>) of the 8331 manual to learn how to configure interrupts for an I/O port on the <em>ATxmega128A1U</em>.

2.2. Create an assembly program (<strong>lab3_2a.asm</strong>) to trigger an interrupt whenever tactile switch S1 is <em>pressed</em>, <strong>without</strong> debouncing the switch. Within the necessary ISR for the interrupt, increment a global counter, e.g., a register, and then display the updated count value on the LEDs available on the <em>OOTB</em> <em>SLB</em> such that the count value is displayed in binary notation with illuminated LEDs. (For example, if the current count value is equal to three, then LEDs <em>D1</em> and <em>D0</em> should be illuminated.) Additionally, within the main routine of the relevant program, after configuring the necessary interrupt, continually toggle on/off the blue LED within the RGB package available on your µPAD (labeled <em>BLUE_PWM</em> within the <em>µPAD</em> schematic)<strong> as quickly as possible</strong> (i.e., within a loop), to be able to easily highlight the fact that the separate, assigned tasks can appear to occur at the same time.

2.3. Verify that your program responds asynchronously to your tactile switch and that the toggling of the LED never appears to falter. Note that the relevant count value will most likely update <em>non-incrementally</em> due to switch bouncing.

Now, techniques for debouncing the relevant switch with interrupts will be explored. It is important to note that, in general, debouncing a switch in an asynchronous environment is <em>much</em> <em>different</em> than in a synchronous environment. In the <a href="https://mil.ufl.edu/3744/docs/switch_debouncing_with_software.pdf"><em>Switch</em></a> <a href="https://mil.ufl.edu/3744/docs/switch_debouncing_with_software.pdf"><em>Debouncing with Software</em></a> document available on our course website, two techniques are provided for environments where asynchronous responses to a switch are desired, however as the document further specifies, only one is allowable for our course.

2.4. Re-read and understand the pertinent sections of th<a href="https://mil.ufl.edu/3744/docs/switch_debouncing_with_software.pdf">e </a><a href="https://mil.ufl.edu/3744/docs/switch_debouncing_with_software.pdf"><em>Switch</em></a> <a href="https://mil.ufl.edu/3744/docs/switch_debouncing_with_software.pdf"><em>Debouncing with Software</em></a> document.

2.5. Create another assembly program (<strong>lab3_2b.asm</strong>) to perform the same functionality as described in § 2.2, but additionally debounce tactile switch S1 with the relevant technique described in the <a href="https://mil.ufl.edu/3744/docs/switch_debouncing_with_software.pdf"><em>Switch Debouncing with</em></a> <a href="https://mil.ufl.edu/3744/docs/switch_debouncing_with_software.pdf"><em>Software</em></a> document.

2.6. Check that your switch is now correctly debounced by verifying that any count value displayed on your LEDs is exactly the same as the amount of times that the push-button has been pressed. (While remaining cautious, try pressing the button in a rougher-than-normal manner to attempt to ensure that the switch is appropriately




<table width="356">

 <tbody>

  <tr>

   <td width="356">future applications – just like with anything, it should always be thoughtfully considered whether or not interrupts are necessary to accomplish some given task.</td>

  </tr>

 </tbody>

</table>

debounced even when placed under more “stressful” situations.)

<strong>NOTE: </strong>Now that you have become somewhat familiar with interrupts, do not let yourself get carried away when developing




<h2><u>PRE-LAB PROCEDURE SUMMARY</u></h2>

<ul>

 <li>Answer pre-lab exercises, when applicable.</li>

 <li>Learn how to configure a timer/counter interrupt in § 1.</li>

 <li>Understand how to configure an I/O port interrupt, as well as how to properly debounce a switch within an asynchronous environment, in § 2.</li>

</ul>

<a href="#_ftnref1" name="_ftn1">[1]</a> There are various reasons for this modularity; some of the more common reasons involve efficiency, cost, and power, although there can be other factors as well.<strong>  </strong>

<a href="#_ftnref2" name="_ftn2">[2]</a> The term “flag” can also be used in contexts regarding software to describe a variable (i.e., a labeled set of memory locations) used to represent the value of some “logical” condition. In the specific discussion presented here, we do not consider such flags.

<a href="#_ftnref3" name="_ftn3">[3]</a> Unfortunately, although hardware designers may try to provide a flag for each crucial or useful event within a given device, there is always some limit.

<a href="#_ftnref4" name="_ftn4">[4]</a> Similar strategies can be implemented for computer systems without a central processing unit.

<a href="#_ftnref5" name="_ftn5">[5]</a> In this context, hardware flags are often referred to as either <em>interrupt flags</em> or <em>interrupt signals</em>.

<a href="#_ftnref6" name="_ftn6">[6]</a> When it is intended that there be multiple interrupts, some form of centralized control is often achieved through dedicated hardware such as a <em>programmable interrupt controller</em>.

<a href="#_ftnref7" name="_ftn7">[7]</a> Typically, an interrupt vector is to contain the starting address of the desired interrupt handler, although this may not always be the case.