# Ubiquity Hardware/Software Architecture

The top level Ubiquity Hardareware/Software architecture
diagram is shown below:

<BlockQuote>
<Img Src="Ubiquity_Hardware_Software_Architecture.png"
Alt="Ubiquity Hardware/Software Architecture">
</BlockQuote>

Each light blue rectangle corresponds to a hardware
assembly.  I the diagram above, the following hardware
assemblies exist:

* Raspberry Pi 2: This is were ROS is run.  The ROSCore node
  runs here.

* bus_raspberry_pi: This is a module the interfaces the
  Raspberry Pi 2 to the bus.

* bus_sonar_10: This module can control up to 10 HC-SR04
  sonar modules.  There are two of these modules on a Freya.

* bus_bridge_encoders_sonar: This module has the electronics
  to drive two motors and encoders.  (It also can drive two
  HC-SR04 sonar modules, but that is not actually used.)

* WiFi Access Point:  This module provides a wireless access
  point so that the robot can communicate with the desktop/laptop.

* Desktop/Laptop: This is the machine on which software is
  developed and debugged.  While ROS runs on this machine as well,
  it accesses the ROSCore node on the Raspberry Pi 2.

The ovals correspond to ROS nodes.  There is a one-to-one
relationship between the bus hardware boards and ros nodes.
The `node_server` node is associated with the `bus_raspberry_pi`
node.

The bus_server_node is a traffic cop that multiplexs messges
from other ROS nodes to the bus through the bus_raspberry_pi
hardware module and back.

There are two commuication protocols:

* 8N1: The 8N1 protocol is 1 start bit, 8 data bits,
  no parity and 1 stop bit.  The Raspberry Pi 2 UART
  supports this protocol with an on chip UART.

* 9N1: The 9N1 protocol is 1 start bit, 9 data bits,
  no parity and 1 stop pit.  The all of the modules connected
  to the hardwar bus talk 9N1 protocol.

The Raspberry Pi2 on chip UART does not support 9N1 mode;
hence, the requirement for the bus_raspberry_pi module to
convert between the two protocols.

The bus architecture has gone through many many iterations:

* [Robobricks](http://gramlich.net/projects/robobricks/index.html):
  This version was built around PIC12 modules.  It actually
  actually sold by Robot Store for a while back in the early
  2000's back before Robot Store was acquire by Jameco.

* [Robobricks2](http://gramlich.net/projects/rb2/index.html):
  This verison is built around PIC16 modules.

* [Robus/Makerbus](http://gramlich.net/projects/robus/index.html):
  This version is built around NXP ARM Cortex 3 modules
  (LPC17xx series.)  These boards are all surface mount.
  There were two names that we played around with -- Makerbus
  and Robus.  They both describe the same bus.

The ubiquity modules are all simply called "bus" and are
based around Atmel ATmega microcontrollers to be compatible
with the Arduino<Sup>&tm;</Sup> community.  These modules
are both electrically and software compatible with the
Robus/Makerbus modules.

Please read the
  [MakerBus Specifications](http://gramlich.net/projects/robus/specifications.html)
to get all of the electrical, software, and mechancal details.

Associated with this project is a software tool to help
configure the modules called
  [configurator](http://gramlich.net/projects/configurator/index.html).
Please note, that this is not the same thing that Rohan talks
about when he says "configurator".  The MakerBus/Robus configurator
is going to be depricated and switched over to use ROS configuration
technology.

The ultimate architecture is that each hardware module presents
its functionality that is organized as a bunch of registers.
Some registers are written to at start up time to perform
configuration.  After configuration, registers are written
to trigger actuators and read to read sensors.

All of the register information for a hardware module is described
in a single .xml file.  It lists each register, it type (Boolean,
Byte, Short, Integer, Long, etc.), a textual desciption, the register
index, etc.  A stand-alone batch processor reads the .xml file and
generates a whole bunch of files needed by ROS to talk to the hardware
module.  The initial code is currently living in the
  [bus_slave](https://github.com/UbiquityRobotics/bus_slave)
repository in the `bus_code_generatory.py` file.  This code is
going to be expanded and likely moved to its own repository.

The requirements of the bus software architecture are:

* Easy edition of new modules.

* Integration with ROS parameters and dynamic configuration.

* Bus discovery whereby the bus is scanned to determine which
  modules are on it.

* Bus firmware update whereby the firemware can be updated from ROS.

* Code generation where boiler plate code is generated to make
  it easier to maintain code.

* Documentation generation where documentation files are automatically
  generated.

* Testing support is generated so it is easy to test modules.
