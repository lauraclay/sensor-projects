#Sensors worksheet

This worksheet will teach you how to use three different types of sensors with the GPIO pins of the Raspberry Pi. You will learn how to wire each one up and read its value(s) in Python code. 

##Line detector  

###Description 
	
In this project you will learn how to read a digital line detector/reflectance sensor 

 
###Tools required 

* Raspberry Pi + SD card 
* 4tronix digital line sensor 
* Keyboard 
* 3 x female to female jumper wires 
* Monitor + HDMI cable 
* Power supply 
 

###Connecting the line detector

 
Firstly, connect the line sensor to the Raspberry Pi as follows: 

(Image) 

####Why connect it like this? 

Use female to female jumper wires (with a hole at both ends) for all of these connections. Make sure the pin is firmly inserted into the wire. 

Connect the G (ground) pin of the line sensor to the GND (ground) pin of the Raspberry Pi. On the sensor that is the first pin along; on the Pi the GND pin I used was pin #20. You must also connect the V+ (voltage+) pin on the line sensor to 3V3 (3.3 volts) on the Pi. These pins provide power to the sensor. 

Next, connect the S (sense/signal) of the line sensor to any GPIO pin; I chose the last pin on the Pi (pin #26). This will return a value of either high (1 or true) or low (0 or false), representing 3.3 volts and 0 volts respectively; the reading will depend on whether the sensor is pointing towards a dark, non reflective (e.g. black) surface, or a light, reflective (e.g. white) surface. 


###Adjusting the sensitivity of the line sensor
 
You should have a white piece of paper with a black line down the middle. These will be used as the reflective and non-reflective surfaces for the sensor to detect. You need to adjust the line detector's sensitivity, so that the white and black return different readings. 

Adjust the screw (potentiometer) in the middle of the line detector so that the red light is lit when the black line is above the sensor, but isn't lit when the white is above it. 

The sensor part is the end of the line sensor opposite the pins with the two bulbs. You should hold the paper between 1 and 3 centimetres away from the sensor. 

(Image)

###Code 
 	
We know that the line sensor’s sense pin will be HIGH (meaning at 3.3v) when the line sensor sees a dark, non-reflective surface like the black line. The Pi should read the sense pin (pin 26) as high during this time, so we will make our script print ‘The sensor sees a black/dark/non-reflective surface’, when the pin reads HIGH/1/TRUE. We will print the opposite when the sensor sees a white surface. 

  ```python
  #Import the time module so we can make our program wait for  
  #a length of time 

  import time
  
  #Import this module to control the GPIO pins and call it  
  #GPIO for the rest of our code 

  import RPi.GPIO as GPIO 

  #We use this method to number our pins which matches the #physical pin numbers 

  GPIO.setmode(GPIO.BOARD) 

  #Set pin 26 as an input so we can read its value 

  GPIO.setup(26,GPIO.IN) 

  try: 

  #Repeat the next indented block forever 

  while True: 

  #If the sensor is HIGH, print the following 

  if GPIO.input(26)==1: 

  print(‘The sensor sees a black/dark/non-reflective surface’) 

  #If not (else), print the following 

  else: 

  print(‘The sensor sees a white/light/reflective surface’) 

  #Wait a second then do the same again 

  time.sleep(1) 

  #If you press CTRL+C, cleanup and stop 

  except KeyboardInterrupt: 

  GPIO.cleanup() 

  sys.exit() 
  ``` 	

 	

1. Create the file `line.py` by running the following command in the command line: 

  `nano line.py`

2. Enter the above code; there’s no need to copy the lines that start with a ‘#’, they’re just comments to explain what the code does! 

3. Once you have done this, save and exit by pressing `Ctrl+x`, choosing `y` then hitting `Enter`. 

4. To run the Python code enter the following command: 
  `sudo python line.py`

  After watching the output, press `Ctrl+C` to close the program. 
 
	
##Ultrasonic distance measurement 

###Description 
	
In this lesson you will use an HR-SC04 to measure real world distances .

###Tools required 

* Raspberry Pi + SD card 
* HR-SC04 ultrasonic module 
* Keyboard 
* Mini 170-pt breadboard 
* Monitor + HDMI cable 
* 8x male to female jumper wires 
* Power supply 
* 330ohm resistor 
* 470ohm resistor 
	

###Connecting the ultrasonic module 

Firstly, connect the module as follows using the wires, resistors, and breadboard: 

(Image) 

####Why connect it like this? 

Use male to female jumper wires to connect each pin of the sonar to a separate row of the breadboard. Connect the row with the VCC connection to the Pi’s 5v pin (pin #2). We need to connect it to 5v rather than 3v3 because the sonar module needs more than 3.3 volts. Connect the row attached to the sonar’s ground to the Pi’s ground (pin #14). Connect the row with the trigger to a GPIO (pin #11). Connect the row attached to echo to another row with a 330-ohm resistor. Connect a GPIO (pin #12) to that row, then connect this to the ground connection row with the 470-ohm resistor. We connect the echo pin to a GPIO with resistors and ground because the module uses a +5V level for a “high”; however, this is too high for the inputs on the GPIO header, which only accept 3.3V. In order to ensure the Pi only receives 3.3V we can use a basic voltage divider; this consists of two resistors.  

###Code 

We will now write the code. The trigger pin will be set to high to send a pulse of sound out of the module. We will set it high for 1μs, for a 1μs pulse of ultrasonic sound. We will then wait until the pulse returns, and time how long the echo pin is high after the pulse has returned. The echo pin should stay high however long it took the pulse to return. We work out the distance from the pulse length with the following formulae: 

Distance (a variable in our code storing the time of the pulse in seconds) = elapsed (the variable storing the length of the pulse in seconds) * 34300 (speed of sound cm/s) 

We need to calculate half the distance, as the pulse had to travel to the object and back, so we only want one way: 

Distance=distance/2 


`print "Distance : %.1f" % distance #print the distance to 1 decimal place  `

 
  ```python
  #!/usr/bin/python 
  # sonar.py 
  # Measure distance using the ultrasonic module 
  #
  # Author : Zachary Igielman 

  # Import required Python libraries 

  import time 

  import RPi.GPIO as GPIO 

  # Use physical pin numbers 

  GPIO.setmode(GPIO.BOARD) 

  # Define GPIO to use on Pi 

  GPIO_TRIGGER = 11 

  GPIO_ECHO    = 12 


  print "Ultrasonic Measurement" 

  # Set pins as output and input 

  GPIO.setup(GPIO_TRIGGER,GPIO.OUT)  # Trigger 

  # Set trigger to False (Low) 

  GPIO.output(GPIO_TRIGGER, False) 


  # Allow module to settle 

  time.sleep(0.5) 

  # Send 10us pulse to trigger 

  GPIO.output(GPIO_TRIGGER, True) 

  time.sleep(0.00001) 

  GPIO.output(GPIO_TRIGGER, False) 


  start = time.time() 

  count=time.time() 

  GPIO.setup(GPIO_ECHO,GPIO.IN)      #Echo 

  while GPIO.input(GPIO_ECHO)==0 and time.time()-count<1: 

    start = time.time() 

  while GPIO.input(GPIO_ECHO)==1: 

    stop = time.time() 
    

  # Calculate pulse length 

  elapsed = stop-start 

  # Distance pulse travelled in that time is time 

  # multiplied by the speed of sound (cm/s) 

  distance = elapsed * 34300 


  # That was the distance there and back so halve the value 

  distance = distance / 2 


  print "Distance : %.1f" % distance 

  # Reset GPIO settings 

  GPIO.cleanup()
  ```


1. Create the file `sonar.py` by running the following command in the command line: 
  `nano sonar.py`

2. Enter the above code; there’s no need to copy the lines that start with a ‘#’, they’re just comments to explain what the code does! 

3. Once you have done this, save and exit by pressing `Ctrl+x`, choosing `y` then hitting `Enter`. 

4. To run the Python code enter the following command: 
  `sudo python sonar.py`.  
  After watching the output, press `Ctrl+C` to close the program. 
  

##MMA7455 I2C accelerometer  

###Description 
	
In this project you will learn how to read an MMA7455 accelerometer using I2C to communicate with the Pi. 


###Tools required 

* Raspberry Pi + SD card 
* MMA7455 accelerometer module 
* Keyboard 
* 5 x female to female jumper wires 
* Monitor + HDMI cable 
* Ethernet cable/USB wireless dongle already set up for the internet 
* Power supply 
* Internet router 
	

###Connecting the accelerometer

Firstly, connect the accelerometer to the Raspberry Pi as follows: 

(Image) 

####Why connect it like this? 

Use female to female jumper wires (with a hole at both ends) for all of these connections. Make sure the pin is firmly inserted into the wire. 

Connect the VCC pin of the accelerometer to 3v3 on the Pi (pin #1), and the GND of the accelerometer to ground of the Pi (pin #6), to power the accelerometer with 3v3. The accelerometer can work with 3v3; by using this voltage, we do not need a level converter as the Pi’s GPIOs work with 3.3 volts. 

We connect the CS of the accelerometer to another 3v3 of the Pi (pin #17) to set it logically high. By making the CS logically high, we tell the accelerometer that we would like to use I2C to communicate with it. If we set it to low and connected it to ground, it would assume we want to use SPI, an alternative to I2C. 

Connect the SDA of the module to the SDA of the Pi (pin #3) and the SCL of the module to the SCL of the Pi (pin #5). 

###Adjusting Raspbian to work with the accelerometer

On the Pi, I2C is disabled by default. To enable it, we need to change a file. To open this file, run the following command: 

  `sudo nano /etc/modprobe.d/raspi-blacklist.conf` 
  
In this file there is a comment, and two lines. Add a hash before the I2C line to comment it out. It should look like this afterwards: 

  ```
  # blacklist SPI and I2C by default (many users don't need them) 

  blacklist spi-bcm2708 

  #blacklist i2c-bcm2708 
  ```
 

Once complete, save and exit by pressing `Ctrl+x`, choosing `y` then hitting `Enter`. 

Next, add the I2C module to the kernel. Run the following command: 

  `sudo nano /etc/modules`

Add the line i2c-ddev to the bottom of that file so that it looks like this: 

  ```
  # /etc/modules: kernel modules to load at boot time. 
  # 
  # This file contains the names of kernel modules that should be loaded 
  # at boot time, one per line. Lines beginning with "#" are ignored. 
  # Parameters can be specified after the module name. 

  snd-bcm2835 

  i2c-dev 
  ```
 
Once complete, save and exit by pressing `Ctrl+x`, choosing `y` then hitting `Enter`. 

We need to install two packages; install them with the following command: 

  `sudo apt-get install i2c-tools python-smbus` 

If that fails, try running the following: 

  `sudo apt-get update`

Now try the first command again. 

To configure the software, we will add the Pi user to the I2C access group by running the following command: 

  `sudo adduser pi i2c`

Now run sudo reboot to reboot, and test the new software.

To test the software, run the command i2cdetect -y 0 to see if there is anything connected. 

If the MMA7455 accelerometer module is connected correctly and working, it should respond with this: 


     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f 

00:          -- -- -- -- -- -- -- -- -- -- -- -- --  

10: -- -- -- -- -- -- -- -- -- -- -- -- -- 1d -- --  

20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --  

30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --  

40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --  

50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --  

60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --  

70: -- -- -- -- -- -- -- --    

 

If nothing is connected or it is incorrectly connected, it will respond with this: 

 

0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f 

00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 

10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

70: -- -- -- -- -- -- -- -- 

 

If this happens, don’t panic. Try the following command: 

  `i2cdetect -y 1`

 
If that responds as if the MMA7455 accelerometer module is connected correctly, it means you have a new Raspberry Pi where the I2C bus is 1 rather than 0. This code works with either, as it detects which bus is correct before connecting. 


If both `i2cdetect -y 1` and `i2cdetect -y 0` respond with this: 

 

0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f 

00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 

10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 

70: -- -- -- -- -- -- -- -- 

 

then there’s something wrong with your hardware. Try wiring up the module again. 
 

###Code 

  ```python
  #!/usr/bin/python 

  #import necessary modules 

  import smbus 
  import time 
  import os 
  import RPi.GPIO as GPIO 

  # Define a class called Accel 

  class Accel(): 

  myBus="" 

  #Work out the Raspberry Pi model and set up the I2C bus #appropriately 

  if GPIO.RPI_REVISION == 1: 

  myBus=0 

  elif GPIO.RPI_REVISION == 2: 

  myBus=1 

  b = smbus.SMBus(myBus) 

  def setUp(self): 

  self.b.write_byte_data(0x1D,0x16,0x55) #Set up the Mode 

  self.b.write_byte_data(0x1D,0x10,0) #Calibrate 

  self.b.write_byte_data(0x1D,0x11,0) #Calibrate 

  self.b.write_byte_data(0x1D,0x12,0) #Calibrate 

  self.b.write_byte_data(0x1D,0x13,0) #Calibrate 

  self.b.write_byte_data(0x1D,0x14,0) #Calibrate 

  self.b.write_byte_data(0x1D,0x15,0) #Calibrate 

  def getValueX(self): 

  return self.b.read_byte_data(0x1D,0x06) 

  def getValueY(self): 

  return self.b.read_byte_data(0x1D,0x07) 

  def getValueZ(self): 

  return self.b.read_byte_data(0x1D,0x08) 
  

  MMA7455 = Accel() 

  MMA7455.setUp() 

  for a in range(10000): 

  x = MMA7455.getValueX() 

  y = MMA7455.getValueY() 

  z = MMA7455.getValueZ() 

  print("X=", x) 

  print("Y=", y) 

  print("Z=", z) 

  time.sleep(0.5) 

  os.system("clear") 
  ```
 

1. Create the file `MMA7455.py` by running the following command in the command line: 
  `nano MMA7455.py`

2. Enter the above code; there’s no need to copy the lines that start with a ‘#’, they’re just comments to explain what the code does! 

3. Once you have done this, save and exit by pressing `Ctrl+x`, choosing `y` then hitting `Enter`. 

4. To run the Python code enter the following command: 
  `python MMA7455.py` 
  After watching the output, press `Ctrl+C` to close the program. 


##Further projects 
 
Now you know how to connect some different types of sensors to the Pi and read them, you can integrate them into projects or use what you have learned here to experiment with another sensor. 

Here are some ideas of what you can do with each sensor: 

###Line detector 

CREATE A LINE FOLLOWER – use two line followers next to each other to follow a line, controlling the motors to go straight when the lines are in the middle, and turn appropriately when they are not. 

###Ultrasonic distance sensor

CREATE A DIGITAL RULER – make a program that reads and updates the distance. You could make it portable with batteries, and add an LCD screen. 

CREATE AN OBJECT-DETECTING ROBOT – this robot could drive about by itself, and stop or drive round objects when it gets too close. 

###Accelerometer 

CREATE A DIGITAL SPIRIT LEVEL – this could measure and output the readings from each axis. You could make it portable with batteries, and add an LCD screen. 

CREATE A SELF-BALANCING ROBOT – make a two-wheeled robot that stands upright. When it starts leaning, it detects this with the accelerometer and drives in that direction to correct it and stop it falling. You could then pull it around on a lead and it will follow you. 

##Contact the author

Contact me if you have any trouble following this tutorial, need help with a project or want to tell me about your project. 

Twitter: @ZacharyIgielman 

Email: ZacharyI123@gmail.com 
