.. _datax-tools-for-ls-system-design:

ADI DataX Tools for Low Speed Mixed Signal System Design
--------------------------------------------------------

.. note::

   This is a work in progress.

Introduction
~~~~~~~~~~~~

The goal of this tutorial is to equip the reader with a collection of `ADI DataX
<https://developer.analog.com/solutions/adi-datax>`__-enabled hardware and
software tools for developing low-speed mixed-signal applications. Complete
written instructions follow, as well as a video guide and a slide deck that can
be used for delivering as a hands-on workshop.

But first - what exactly does “Low Speed” mean? In the context of this tutorial,
it means that timing is not very critical. Signals are either completely static
or moving slowly such that it doesn't matter if the instant that an ADC samples
the signal wiggles around a bit relative to the previous sampling. While clock
jitter is one source of this uncertainty, software delays (such as the time
between a timer interrupt and the assertion of a “convert” edge) will likely be
dominant. Important parameters in low-speed applications are offset, gain error,
linearity, and temperature drift. “Noise” in a low-speed application is
typically synonymous with resolution, and is typically measured by applying a
noiseless input signal and taking a histogram of the output readings. AC
performance metrics such as signal to noise ratio and total harmonic distortion
extracted from a Fourier transform of the data will not be considered. In
contrast - sample jitter is important in a “high speed” application. If you are
measuring signal to noise ratio, the Signal to Noise ratio (SNR) can be no
greater than:

:math:`SNR <= -20 * log(2*pi*f\_{IN}*t\_{j})`

Where:
:math:`f_{IN}` is the analog input frequency in Hz
:math:`t_{j}` is the RMS jitter in seconds RMS

So that's it. In this tutorial, we will use a transistor curve tracer as an
example application that involves setting voltages and currents, reading
voltages and currents, doing some basic math, and displaying a result. Each
reading will be treated independently, no correlation to previous or future
readings. We will NOT be measuring AC Signal to Noise Ratio (SNR), Total
Harmonic Distortion (THD), nor measuring steps, wiggles, or any other situation
where precise timing is required. (Rest assured, there are lots of very
interesting applications in this category.) 

Materials
~~~~~~~~~

- Raspberry Pi 4 or 5; 2GB or greater RAM (for Linux examples). (Model 3B, 3B
   Plus will work, but you will want a 4 or 5 :-) )
- 5V USB-C wall adapter for Raspberry Pi (micro USB for model 3)
- 16GB (or larger) Class 10 (or faster) micro-SD card, with :ref:`kuiper` installed
- User interface setup (choose one):
   - HDMI monitor, keyboard, mouse plugged directly into Raspberry Pi
   - Host Windows/Linux/Mac computer on same network as Raspberry Pi
-  :adi:`ADALM2000
   <en/design-center/evaluation-hardware-and-software/evaluation-boards-kits/adalm2000.html>`
   (Optional, for observing signals.)
- :adi:`MAX32666FTHR<max32666fthr>` development board (for no-OS examples)
- Either:
   - :adi:`ADALM-LSMSPG<adalm-lsmspg>` Low-Speed Mixed Signal Playground module
- Or:
   - :adi:`EVAL-AD5592R-PMDZ<eval-ad5592r-pmdz>`
   - :adi:`EVAL-AD5593R-PMDZ<eval-ad5593r-pmdz>`
   - :adi:`Raspberry Pi to PMOD/QuikEval™/LTpowerPlay® Adaptor HAT<pmd-rpi-intz>`
   - 2N3904 NPN Transistor
   - 2N3906 PNP Transistor
   - 47Ω resistor
   - 47kΩ resistor
   - Breadboard or prototyping board, hookup wire
- Clone or download zip of the Python code for this tutorial: :git-pyadi-iio:`ADALM-LSMSPG Pyadi-IIO examples<examples/adalm-lsmspg>`
- Note that these are included in the pyadi-iio repo, consider cloning the entire thing:

  .. code-block:: none

     git clone https://github.com/analogdevicesinc/pyadi-iio.git

-  AD5592R Device Tree Overlay for alternate configuration with GPIO pins

.. ADMONITION:: Download

   :download:`rpi-ad5592r-with_gpios-overlay source and compiled overlay <rpi-ad5592r-with_gpios-overlay.zip>`

Background
~~~~~~~~~~

This tutorial builds on the concepts covered in the
:ref:`conv_connect_tutorial`.

It also serves as a preview to the :ref:`precision_adc_tutorial` that starts to
deal with analyzing time series data.

Slide Deck and Video
~~~~~~~~~~~~~~~~~~~~

Since this tutorial is also designed to be presented as a live, hands-on
workshop, a slide deck is provided here:

.. ADMONITION:: Download

   :download:`Tools for Low-Speed Mixed Signal System Design Slide Deck <tools_for_low_speed_ms_workshop.pptx>`

A complete video run-through is also provided, either as a companion to
following the tutorial yourself, or to practice before presenting as a
hands-on workshop: <WRAP todo> This video is accurate, but will be
re-done at some point:


.. video:: https://www.youtube.com/watch?v=tJtzUrt9_1U


Preparation - a few resources for learning Python
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A wonderful resource for learning Python is `learnpython.org
<https://learnpython.org/>`__, runs right in your browser without needing to
install anything.

And despite the name, `Python for Kids
<https://nostarch.com/python-kids-2nd-edition>`__ is surprisingly good for
adults, too!

What does “Just Enough Software” look like?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Software Stack Background
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Introducing an exciting new product that we'll apply our skills
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Component selection based on software support (rather than pure analog performance)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hardware Setup
~~~~~~~~~~~~~~~~

Booting the system
~~~~~~~~~~~~~~~~~~~~~~~~

Configuring the System (and rebooting!)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Raspberry Pi-based hardware and Linux setup mirrors that of the ADXL345 used
in the :dokuwiki:`Converter Connectivity Toolbox and Tutorial
</university/labs/software/iio_intro_toolbox>`, including bringing up the
pyadi-iio example. Follow the instructions for downloading and installing ADI
Kuiper Linux, and editing config.txt. The only difference is the interrupt
connection and device tree overlay to be added to config.txt. For this exercise,
add the following lines to config.txt:

.. code-block:: none

   dtoverlay=rpi-adalm-lsmspg

   # Heartbeat blinky:
   dtparam=act_led_gpio=20
   dtparam=act_led_trigger=heartbeat

   # Short GPIO 21 (pin 40) to ground for shutdown:
   dtoverlay=gpio-shutdown,gpio_pin=21,active_low=1,gpiopull=up

The details of the lsmspg overlay will be covered shortly. The heartbeat blinky
section configures the activity LED to pulse a heartbeat pattern, and assigns it
to GPIO 20 on the Raspberry Pi header. GPIO 21 (pin 40) is configured to trigger
a shutdown when shorted to ground.

Command Line Tools (Hello, AD5592r!)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

IIO Oscilloscope
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Device Trees: Telling Linux what's connected
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pyadi-iio And examples
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Hands-On!** Working through a simple, but complete case study
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next Steps: Developing on a remote host
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next Steps: Other languages (C++, C#, MATLAB, etc.)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

IIO as a Tool for Migrating to an Embedded Implementation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Prototyping in Linux is incredibly convenient - even if we didn't have pre-built
device tree overlays for the ADALM-LSMSPG, it's not terribly difficult to create
our own. And once the driver is bound, we can access the hardware via any of the
libiio supported language bindings. If the end application is based on Linux,
then we can just keep on going to the finish line - write top level code that
runs direclty on the target, or on a remote host via USB or Ethernet backends.
But is there any value in starting with Linux if the end application will
ultimately be fully embedded? You bet! The IIO supports the serial backend,
and really doesn't care if it's talking to an actual iiod daemon running on
Linux, or - a "fake" daemon running in a bare-metal application. As long as the
transactions are correct, the libiio doesn't care if it's talking to Linux on a
Raspberry Pi, bare-metal C running on an ARM microcontroller, a BASIC
implementation running on a Parallax BASIC stamp, or a Forth implementation
running on an 8051 CPU. This means that while you're testing out proof of
concept code in Python talking to a Raspberry Pi, your hardware team can be
designing the final board with the target processor. The devices can then be
exposed over the iio network backend using the "tinyiiod server". This allows
you to run the same proof of concept Python (or C or MATLAB or C#) code that
previously talked to the Raspberry Pi, to talk to your actual embedded target.

To see this in action, let's load up the pre-built ADALM-LSMSPG tinyiiod server.
Go to :git-no-os:`ADALM-LSMSPG firmware (no-OS releases) <releases/latest+>`,
download the adalm-lsmspg.zip file, and unzip to a convenient location. Shut
down your Raspbery Pi properly, then disconnect the 40-pin ribbon cable from the
ADALM-LSMSPG board. Install a MAX32666FTHR in the FTHR sockets, taking care to
align the pins properly. Connect the supplied MAX PICO board to the MAX32666FTHR
programming header. Connect both the MAX PICO and MAX32666FTHR to the host
computer via USB-A to Micro-B cables. Drag and drop the
adalm-lsmspg_maxim_iio.hex file into the DAPLINK DAPLINK mass storage device
(typically ``D:`` or ``E:`` on Windows systems). The DAPLINK drive will
auto-eject, and the heartbeat LED on the ADALM-LSMSPG will begin blinking.
(Almost done!)

Unlike network and USB backends, the iio serial backend is not discoverable so
we will need to find out what serial port the MAX32666FTHR enumerates as.

.. note::
   Back in "ye oldyn days" serial ports were dedicated D-SUB 9 or 25 pin
   connectors on the host computer, assigned to a particular COM or TTY port.
   Those days are mostly gone; "virtual" USB serial ports are incredibly
   convenient as they allow the use of standard serial port software APIs, the
   drawback is the port nubmering can be somewhat arbitrary and inconsistent.

There are various ways to find the serial port - Device Manager on Windows, and
looking for tty* ports in /dev on Linux, but we can also use IIO Oscilloscope or
Scopy from the previous experiments.

Once the serial port is located, run the same curve tracer scripts as before,
but append the COM / tty port URI:

.. code-block:: none

   ad5592r_curve_tracer.py -u serial:COMx
   ad5593r_curve_tracer.py -u serial:COMx

where "x" is the COM port number identified. The output should be identical to
previous runs using the local bakend, as shown in :numref:`fig-ct_tinyiiod`

.. _fig-ct_tinyiiod:

.. figure:: CurveTracerOutput.png
   :width: 700px
   :height: 400px
   :align: center

   Curve tracer plots, serial backend

At this point you can re-verify your top-level code, but on the actual target
hardware (vs. evaluation boards or crude prototypes). While the devices and
curve tracer application on the ADALM-LSMSPG are not terribly sensitive to
noise, more sensitive applications - precision instrumentation, communications,
sensing, etc. - will absolutely benefit from a quick check before beginning the
potentially long embedded firmware development process.

Porting to a Fully Embedded System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once the hardware is validated using the tinyiiod server, the process of
migrating to a fully embedded system can begin. The
`no-OS <https://analogdevicesinc.github.io/no-OS/index.html>`__ is Analog
Devices' bare metal framework for embedded systems, supporting Maxim and Analog
Devies processors, as well as STM32, Raspberry Pi pico, Xilinx and Altera soft
processors, freeRTOS, ChibOS, and others. The framework is designed to be easily
portable to other platforms as well.

Let's now migrate the curve tracer logic that until now ran in Python on a
remote host into the embedded target, replacing the tinyiiod server entirely.
While the ultimate goal of the curve tracer is to have a local display, that's
another layer of both hardware and software development that we can defer a bit
longer with a little bit of creative thinking. Since there is a serial port
available, we can test out the logic by printing values to a terminal, formatted
as comma-separated variable (CSV) data for easy copy/paste into `LibreOffice
<https://www.libreoffice.org/>`__ or other spreadsheet for plotting. And for a
bit of `icing on the cake
<https://www.merriam-webster.com/dictionary/icing%20on%20the%20cake>`__ and
nostalgia, we can also make an ASCII-art plot!

Go back to the zip file from the no-OS release, and drag-and-drop the curve
tracer example HEX file into the DAPLINK drive. Press the RESET button on the
ADALM-LSMSPG and observe the output. The CSV data and ASCII-art plots will be
printed to the terminal as shown in the figures below.

.. code-block::
   :caption: ASCII-art NPN curve trace output

   === AD5592R (SPI) - NPN Curve Tracer (Ic vs Vc) ===
   Y-axis: Ic (0 to 7.16 mA)
   X-axis: Vc (0 to 2.45 V)

   +------------------------------------------------------------+
   |         ****** ***** ***** ***** ***** ***** ** *          |
   |        *                                                   |
   |       *                                                    |
   |      *                                                     |
   |                                                            |
   |      * ** ***** ***** ***** ***** ***** ***** ***** *      |
   |     * *                                                    |
   |      *                                                     |
   |     **                                                     |
   |     *                                                      |
   |    *   *** ***** ***** ***** ***** ***** ***** ***** **    |
   |     ***                                                    |
   |    *                                                       |
   |    **                                                      |
   |    **                                                      |
   |    *  ***** ***** ***** ***** ***** ***** ***** ***** ***  |
   |   *  *                                                     |
   |   ***                                                      |
   |****** **** ***** ***** ***** ***** ***** ***** ***** **** *|
   |                                                            |
   +------------------------------------------------------------+
   0.0       0.49       0.98       1.47       1.96       2.45 V

   ===== AD5592R Curve Trace Complete =====


Similarly, you will see an ASCII-art PNP curve trace similar to the figure below.

.. code-block::
   :caption: ASCII-art PNP curve trace output

   === AD5593R (I2C) - PNP Curve Tracer (Ic vs Vc) ===
   Y-axis: |Ic| (0 to 5.62 mA)
   X-axis: Vc (0 to 2.50 V)

   +------------------------------------------------------------+
   | ***** ****** ***** ****** ***** ****** ***** ****** *******|
   |                                                         *  |
   |                                                        **  |
   |                                                       **   |
   |  ** ****** ***** ****** ****** ***** ****** ***** **** *   |
   |                                                        *   |
   |                                                        *   |
   |                                                       *    |
   |                                                      * *   |
   |                                                            |
   |                                        ***** ******* *     |
   |      ***** ****** ****** ***** ****** *               *    |
   |                                                            |
   |                                                       *    |
   |                                                      *     |
   |                                                            |
   |                                                   ***      |
   |                                   * ** ****** ****         |
   |         ** ****** ****** ****** ** *                       |
   |         *                                                  |
   +------------------------------------------------------------+
   0.0       0.50       1.00       1.50       2.00       2.50 V

   ===== AD5593R Curve Trace Complete =====

At this point all of the math, algorithms, and overall operation of the curve
tracer are running in the embedded target, and we're able to verify everything
is operating properly and with full (analog) performance. The next step can be
to connect a local display, or enable a server for display on a remote screen
such as a tablet or mobile device. 


Next Steps: No-OS development on Linux? You bet!
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

...but I'm Confused... No-OS means no Operating System, but we're using Kuiper
Linux, and that's an Operating System. What gives?

Unlike the IIO drivers used in the previous tutorial , which **require** the
Linux kernel and operating system to function, No-OS provides a portable
software stack which can run on any platform that supports a C compiler. This
could be bare metal microcontrollers, truly running without an operating system,
up through full systems like our Kuiper Linux running on a Raspberry Pi. The
No-OS repository includes existing support for the Linux OS, Real-Time Operating
Systems Chibios, and mbed, Raspberry Pico, as well as hardware support for
Maxim/ADI, STM32, Xilinx and Altera. But why? Well, bringing up a toolchain for
a particular embedded processor has its own set of challenges - particularly if
development will begin on a standard development platform, then be ported to a
custom board. Runnin no-OS code on Linux provides a way to get started on the
embedded code development, before actually embedding. A full treatment of this
flow is beyond the scope of this tutorial, but will be documented in a future
tutorial.

.. todo::

   Port the Fred in the Shed curve tracer to no-OS on Linux.

Porting to a fully embedded system
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The idea of running code on a microcontroller without the overhead of an operating system. This is the ideal for a lot of embedded applications, 
and is the ultimate goal of this tutorial. In this tutorial, we are using the no-OS software stack, which is a portable software stack that can run on any platform that supports a C compiler. 
This could be bare metal microcontrollers, truly running without an operating system, up through full systems like our Kuiper Linux running on a Raspberry Pi. 
The No-OS repository includes existing support for the Linux OS, Real-Time Operating Systems Chibios, and mbed, Raspberry Pico, as well as hardware support for Maxim/ADI, STM32, Xilinx and Altera.


To verify the proof of concept, The challenge was to fully port a high level IIO-based Python script 
to a no-OS C program running only from the microcontroller. Since the idea was if it can be done in an IIO Level 
It's most certainly possible to do it in a bare-metal C program project. The script in question is the
:git-pyadi-iio:`AD5592r / AD5593r Curve Tracer Pyadi-IIO examples <examples/ad5592r_examples>`. Note that not everything 
can be fully ported from the high-level script, but the core functionality of setting voltages, reading voltages, 
and using the devices all in all can be done in a fully embedded system. Additionally, the final script should contain the following:

-  ASCII output of the results, such as printing to a console, or sending over UART, etc.
-  Curve tracer data that can be used to plot using a spreadsheet tool like Excel, or LibreOffice.
-  The same functionality as the original script, such as setting voltages, reading voltages, and using the devices all in all.

.. image:: CurveTracerOutput.png
   :width: 700px
   :height: 400px
   :alt: Curve Tracer ASCII Output
   :align: center

This photo shows the output of the PyADI-IIO curve tracer scripts


.. image:: max78000dog.png
   :width: 700px
   :height: 400px
   :alt: ASCII_Dog_78000
   :align: center

Here is an example of an ASCII Art printed on a terminal.



To start the project, you must first setup your enviroment through this `Wiki guide <https://wiki.analog.com/resources/no-os/build>`__ 

**NOTE:** In making of this project, majority of the time I use **WSL** rather than standard windows due to unresolved issues.
Once the environment is setup, you can start by creating a new no-OS project by cloning the no-OS repository

.. shell::

   $git clone https://github.com/analogdevicesinc/no-OS.git

Once done, a good way to test your enviroment is if you can build and run your existing project. You can do this by running the command

.. shell::

   $cd /no-OS/projects/adalm-lsmspg/

.. shell::

   $make PLATFORM=maxim TARGET=max32665 run

If this command works, now you can start writing your own fully embbedded projects! 

A good way to start is to get a glimspe of existing fully embedded example scripts so that you can grasp how to write your own.
:git-no-OS:`Here is a basic / dummy example with no IIOs <projects/ad74413r/src/examples/dummy/dummy_example.c>`
See how almost everything in the example file are just wrappers and function calls to the drivers and no-OS API.
This is the beauty of no-OS, you can write your own code and use the existing drivers and API to do all the heavy lifting for you.

Once done reading you can now start trying to use the driver functions, such as the ones in the AD5592r driver, to write your own code. 
You can start by writing a simple code that sets the DAC voltage and reads it back using the ADC, then you can move on to more complex code such as the curve tracer example.
Below are onboard connections that you can use to test your code, but feel free to make your own connections using the other GPIO pins as well!


::

   Channel 0 -> Channel 1 (DAC0 => ADC1), Channel 1 -> Channel 2 (ADC1 => ADC-DAC2)

Now to start development on a fully embedded system port, The number one rule of thumb in doing this is always cross referencing the drivers of the device that you will use and 
the original PyADI-IIO Script. In this case, the :git-no-OS:`AD5592r driver <drivers/adc-dac/ad5592r>` is the one you look around to use the base drivers rather than the IIO ones but
a good practice also is to read the IIO drivers as it you can always assume it uses majority of the base drivers properly giving you more examples how how to use the function calls and wrappers of the base driver.
Additionally, it's important to know how drivers are structured in no-OS, as they are strctured with each having there own header file and main file. Sometimes the header file contains things such as conversion values,
Parsing values, and important addresses for the device. Majority of the time the debugging process is just reading these files back and fourth checking whether you are using them properly, Once again, indicating that 
It's important to see project examples on how they are used.

::

   int32_t ad5592r_read_adc(struct ad5592r_dev *dev, uint8_t chan, uint16_t *value)

In our journey in porting this, if you have noticed that the abscence of printf statements in the code, this is because our microcontroller does not have a standard output, but rather
only uses UART for communication, so we have to use the UART functions in no-OS API to send our data over UART and then read it on a terminal on our computer.
The function below is our method of writing data to the terminal. You can read more about the function :git-no-OS:`here <drivers/api/no_os_uart.c>`. Additionally, if you tried this function 
immediately you will notice it will not work, that is because it does not catch up to the speed of the data being sent, so you have to add a delay after for your scripts which is also written below

::

   no_os_uart_write(struct no_os_uart_desc *desc, const uint8_t *data, uint32_t bytes_number)

   no_os_delay(uint32_t delay_ms)

.. todo::

   Show fully embedded projects and steps how to achieve them

Now with all components figure out, we can now start writing our own import code from two different languages. The first thing that needs to be ported is the overall sequence of iterating to each different points, to 
achieve 4 to 5 different curves. So in order to do that we must use nested for loops to iterate through the different point, per different point we will set the designated voltage per iteration creating 
a sweeping effect. Here is the PyADI-IIO code for you to have an understanding how the **"sweeping"** works:

::

   for vb in range(499, 2500, 500):  # Sweep base voltage from 499 mV to 2.5V in 5 steps
            Vbdrive.raw = vb / float(mV_per_lsb)  # Set base voltage
            ib = ((Vbdrive.raw * mV_per_lsb / 1000) - Vbe) / Rbase  # Calculate base current
            vcs = []  # Empty list for collector voltages
            ics = []  # Empty list for collector currents
            print("Base Drive: ", Vbdrive.raw * mV_per_lsb / 1000, " Volts, ", ib * 1e6, " uA") 
            for vcv in range(0, 2500, 50):  # Sweep collector drive voltage from 0 to 2.5V in 50 mV steps
                    print(Vcdrive_meas.raw, Vcdrive.raw, Vcsense.raw)
                    Vcdrive.raw = vcv / float(mV_per_lsb)  # Set collector drive voltage
                    ic = ((Vcdrive_meas.raw - Vcsense.raw) * mV_per_lsb / Rsense)  # Measure collector current
                    vc = Vcsense.raw * mV_per_lsb / 1000.0  # Remember - actual collector voltage is
                    vcs.append(vc)  # a bit less due to sense resistor 
                    ics.append(ic)  # Add measurements to lists
                    # print("coll voltage: ", vc, "  coll curre: ", ic)  # Print for fun  
            curves.append([vcs, ics])  # vcs, ics, will be index 0, 1, respectively

If you need hints on how to port this, note that all of the declaration of values such as "ic = Vcsense.raw * mV_per_lsb / 1000.0" are just function calls in the no-OS Driver, so once you 
see these variable declaration you must use the function "ad5592r_write_dac()" for this to work, then for every variable that is called upon these values that is involved with components with the device,
You must use "ad5592r_read_adc()" to read the value back and use it for calculations.


After getting all of this ported out, you can now start the deliverables of the project, which is to print the curve tracer data in the terminal (no need for exporting to a seperate file yet) and copy the curve 
tracer data to a spreadsheet to plot it. It's also appropriate to format it as a CSV or Comma Seperated Values for easy porting to a spreadsheet.


.. image:: Spreadsheet_screenshot.png
   :width: 700px
   :height: 400px
   :alt: Spreadsheet Screenshot for the Curve tracer data plotted
   :align: center

First deliverable done, now for the ASCII output, using the same array that stores all the data you could pretty much create a simple ASCII art that pretty much uses plotting logics to do this.
We use arrays to store each different layers of the ASCII curve tracer art to print per row. Below is an example of how to do this in Python, but this is plausable in C as well but a little more tricky to do:

::

   def print_ascii_plot(curves, vcs_index, ics_index):
    max_v = max(max(curve[vcs_index]) for curve in curves)
    max_i = max(max(curve[ics_index]) for curve in curves)
    scale_v = 50 / max_v  # Scale voltage to fit in 50 characters
    scale_i = 20 / max_i  # Scale current to fit in 20 characters

    plot = [[" " for _ in range(51)] for _ in range(21)]

    for curve in curves:
        for vc, ic in zip(curve[vcs_index], curve[ics_index]):
            x = int(vc * scale_v)
            y = int(ic * scale_i)
            if 0 <= x < 51 and 0 <= y < 21:
                plot[20 - y][x] = "*"

    print("ASCII Curve Tracer Output:")
    print("    " + "-" * 51)
    for row in plot:
        print("   |" + "".join(row) + "|")
    print("    " + "-" * 51)
    print("     " + " ".join(f"{i/10:.1f}" for i in range(0, 51, 10)))


Once done it should look something like this: 

.. image:: ASCIICurve.png
   :width: 700px
   :height: 400px
   :alt: ASCII Curve Tracer Output
   :align: center

Below is the downloadable HEX File, you can read the source code in the no-OS repository for the project.


.. ADMONITION:: Download HEX File

   :download:`Fully embedded example <adalm-lsmspg_fixed_LED.zip>`


.. todo::

   Show fully embedded projects and steps how to achieve them
   

More “Just Enough Software” examples
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Drawing parallels to other software flows
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Wrapup
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Additional References
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
