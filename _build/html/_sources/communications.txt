
SPI and I2C Communications
=====================================

In this section we document the SPI and I2C communications used by the G4 panels.

Overview
-------------------------------------

Display data is sent from the controller to panels via SPI. Each panel, after
receiving data, will display the pattern specified via the data received
exactly once (one-shot display update). In order to continuously display a
visual pattern - whether static or changing - the panels must receive display
data at a fixed clock rate from the controller.  The exact frequency of this
display update clock is not crucial, however for a consistent display intensity
the clock rate should be constant.   Also, for the two gray scale
modes supported by the panels - 16-level and 2-level - there is as maximum
display update clock frequency.  Operating above this frequency is not advised
as the panels will likely receive garbled data.  The maximum operating
frequencies are:

* 16-level gray scale - 500Hz
* 2-level gray scale 1500Hz.

The duration of the one-shot display update performed by a panel is encoded in
an byte sent with the display data. This duration parameter can be used to
adjust the effective duty-cycle of the display  and can thus be used to control
the brightness for a given update frequency. Note, using a larger (than default)
display duration may decrease the maximum operating frequency.

For a given panel a display update is initiated when it receives data from the
controller.  The data is first received (via SPI) by atmega328 micro-controller
on the comm sub-panel. The SPI message contains the display information for
each of the 4-matrices on the driver sub-panel. Next, the micro-controller on
the comm sub-panel splits the message into four equal sized each parts - one
for each LED matrix on the driver sub-panel - and forwards each part (via I2C)
to the atmega328 micro-controller on the driver sub-panel corresponding to LED
matrix associated with the data. The micro-controller on the driver sub-panels
will then display the data exactly once. 

I2C Messages 
------------------------------------- 
In this section the format of the I2C messages sent from the comm sub-panel to the
driver sub-panel is described. 

The length of the message depends on the number of gray scale levels in the
display pattern: 

* 16-level gray scale,  length = 33 bytes
* 2-level gray scale,  length = 9 bytes

The first byte of the message is special and encodes the number of gray scales
levels in the subsequent data and the display 'stretch' parameter which
determines how long the one shot pattern is displayed. The contents of the
first message are as follows:

* i2cMessage[0], bit 0 =  gray scale identifier (0 = 2-level gray scale, 1 = 16-level gray scale)
* i2cMessage[0], bits 1-7 = display 'stretch' parameter  (0-127). 0 is the shortest possible display and 127 the longest. 

Thus 

.. code-block:: C

    i2cMessage[0] = GRAY_SCALE_ID | (STRETCH_PARAMETER << 1);

The remaining bytes in the message - 32 for 16-level gray scale or 8 for
2-level gray scale - consist of the display pattern data. 

I2C: 16-level Gray Scale 
""""""""""""""""""""""""""""""""""""""
For 16-level gray scale each pixel is encoded by four bits so that each byte of
the message consists of the data for two pixels. Columns are encoded first as
follows:

.. code-block:: C

     i2cMessage[1] = pixel[0,0] | (pixel[0,1] << 4);
     i2cMessage[2] = pixel[0,2] | (pixel[0,3] << 4);
     i2cMessage[3] = pixel[0,4] | (pixel[0,5] << 4);
     i2cMessage[4] = pixel[0,6] | (pixel[0,7] << 4);
     i2cMessage[6] = pixel[1,0] | (pixel[1,0] << 4);
     i2cMessage[7] = pixel[1,2] | (pixel[1,3] << 4);
     ... 
     i2cMessage[32] = pixel[7,6] | (pixel[7,7] << 4);


I2C: 2-level Gray Scale
""""""""""""""""""""""""""""""""""""""
For 2-level gray scale each pixels is encoded by one bit so that each byte of the message
consists of the data for 8 pixels. Columns are encoded first as follows:

.. code-block:: C

    i2cMessage[1] = pixel[0,0] | (pixel[0,1] << 1) | (pixel[0,2] << 2) | (pixel[0,3] << 3) | ... | (pixel[0,7] << 7);
    i2cMessage[2] = pixel[1,0] | (pixel[1,1] << 1) | (pixel[1,2] << 2) | (pixel[1,3] << 3) | ... | (pixel[1,7] << 7);
    ...
    i2cMessage[8] = pixel[7,0] | (pixel[7,1] << 1) | (pixel[7,2] << 2) | (pixel[7,3] << 3) | ... | (pixel[7,7] << 7);


SPI Messages
------------------------------------- 
In this section  the format of the SPI messages sent from the controller to the
comm sub-panels is described.

The SPI message simply consists of four I2C messages - one for each matrix
on the panels - stacked one after the other. The data for matrix 0 is given first,
followed by the data for matrix 1, etc. 

SPI: 16-level Gray Scale 
""""""""""""""""""""""""""""""""""""""
For 16-level gray scale the bytes in the SPI  message are  as follows: 

* spiMessage[0-32] = i2c message for matrix 0
* spiMessage[33-65] = i2c message for matrix 1
* spiMessage[66-98] = i2c message for matrix 2
* spiMessage[99-131] = i2c message for matrix 3

The total message length is 4*33 = 132 bytes.


SPI: 2-level Gray Scale
""""""""""""""""""""""""""""""""""""""
For 2-level gray scale the bytes in the SPI message is as follows:

* spiMessge[0-8] = i2c message for matrix 0
* spiMessage[9-17] = i2c message for matrix 1
* spiMessage[18-26] = i2c message for matrix 2
* spiMessage[27-35] = i2c message for matrix 3

The total message length is 4*9 = 36 bytes.

SPI: Parameters 
""""""""""""""""""""""""""""""""""""""

+-----------------+---------------+
| Parameter       | Value         |
+=================+===============+
| Bit Order       | MSBFIRST      |
+-----------------+---------------+
| Data Mode       | Mode 0        |
+-----------------+---------------+
| Maximum Clock   | 4Mhz          |
+-----------------+---------------+


