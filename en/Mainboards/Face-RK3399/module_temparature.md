##  temperature measurement module
Face X2 (Face-RK3399) can be configured with the optional infrared thermal imaging temperature measurement module shown below.


![](../../../rk3399_img/Face-RK3399/newtemp.png)



The temperature measurement module is connected inside the Face-X2 hardware through a serial port. The default node is `/dev/ttyS2`, and the baud rate is 19200.

(If the customer purchases the Face-RK3399 single board, another serial port node can be selected according to the actual situation.)

The temperature display function has been adapted in the factory face recognition APK. Click `Settings -> Gate Settings -> Temperature` to enable this function.


Applicable scenarios and requirements of the infrared temperature measurement module:

Temperature measurement distance: 30-50 cm

Temperature measurement range: 23-48 degrees

Room temperature: 20-35 degrees (medical environment). When the room temperature is below 15 degrees, the human body surface temperature is no longer linear, so compensation cannot provide accurate results.





