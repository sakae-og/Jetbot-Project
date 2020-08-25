# _Jetbot-Project_
*Jetbot-Project related script*

* Sources: (https://github.com/NVIDIA-AI-IOT/jetbot)
* Hardware: CSI camera, basic 2 Wheel Tracing Robot car chassis, Power Bank 5V 4A (20000 mAh OpponReno), DC Motor+Stepper Feather Wing, 3D Printed Power Bank holder, nano

## Contents
* [1) data_collection-Jetbot_Joystick.ipynb](#script)
* [2) Object recognition and driving towards it: live_demo-steering_tweak.ipynb](#recognition)
* [3) Line following: data_collection_joystick_roadfollowing.ipynb and target Display](#roadfollowing)
* [4) Adjustments](#adjusts)

## 1) data_collection-Jetbot_Joystick.ipynb

* Modified collision_avoidance data collection Notebook from Jetbot GitHub to allow driving and steering using gamepad joystick while image acquisition by gamepad buttons
* Excel file: test and display two different coordinate transformation strategies
* transfer a notebook copy to Notebooks/collision_avoidance/ of the Jetbot SD image installation

### a) Script contains: 
   * Joystick coordinate system transformation: based on user Pedro Werneck (https://electronics.stackexchange.com/questions/19669/algorithm-for-mixing-2-axis-analog-input-to-control-a-differential-motor-drive)
   ```
def steering(x ,y):
	# convert to polar
	r = math.hypot(x, y)
	t = math.atan2(y, x)
	# rotate by 45 degrees
	t += math.pi /-4
	# back to cartesian
	left = r * math.cos(t)
	right = r * math.sin(t)

	# rescale the new coords
	left = left * math.sqrt(2)
	right = right * math.sqrt(2)

	# clamp to Min/Max
	scalefactor= speed_widget.value # or =0.5
	left = max(-1*scalefactor, min(left, scalefactor))
	right = max(-1*scalefactor, min(right, scalefactor))

	#gamma correction for sensitivity of joystick or x value changes while turning : TB
	gamma=turn_gain_widget.value #or =2 joystick and 1-40 for object targeting
	if left <0 :
    		left= -1* (((abs(left)/scalefactor)**(1/gamma))*scalefactor)
	else:
    		left= ((abs(left)/scalefactor)**(1/gamma))*scalefactor
   
	if right <0:
    		right= -1*(((abs(right)/scalefactor)**(1/gamma))*scalefactor)
	else:
    		right= ((abs(right)/scalefactor)**(1/gamma))*scalefactor

	return left, right
```
   * Same Gamma, Speed and motoradjustment sliders for tuning of the bot to run straight, values can be used for Object following script
   * display of joystick's left and right engine values
   * gamepad buttons for image aqcuisition (free, blocked)
   * graphical buttons of the orginal script remain active and can be used

### b) Issues:
   * Scale range for motors is -1 to 1
   * Gamma value can be between 0.1-4
   * Gamepad: Xbox USB: joystick (controller.axes: x:0 (left-right, y:1 (forward-backward)
   * Image acquisition: button (5: free, 7: blocked)   *
 
 ## 2) Object recognition and driving towards it: live_demo-steering_tweak.ipynb
 ### Goal: time lag
 ### Tweak driving towards an Object:
 	*activated collision_avoidance if the object is not found within 5-10 seconds. Using the box beside the target label to enter 0 for No search and 1 for search. need to click somewhere to activate
 	*Possible at high speed towards an stationary object, following a moving object is limited by the 1 s time lag and los of Object detection (increase number of non object frame, e.g. 12)
	*Steering(x, y) has been also incorporated into live_demo-steering_tweak.ipynb to tweak steering sensitivity.
	*The first value that needs to be adjusted is the torque of right and left motor to allow the bot going straight (offset value for left e.g. 0.04).
	* At higher speed levels the difference between two wheels becomes lower, thus the motoradjustment_widget.value should be decreased slightly.
	*The center_x value changes have to be reduced (e.g. center_x/4 worked for non widefield camera) 
	*Gamma value (to further reduce turning strength) has to be adjusted by turn_gain_widget.value slider (2) 
	*The y-value is fixed by the speed_widget.value. The y-value could be optained by box botton value, but changes in y value would need to be flatten.
	*The values dependent upon motors, speed and camera, thus altering y value while driving creates steering errors (formula?).
	*In case the object is not detected for a some frames, then the bot stops and is waiting for a successful detection (e.g. wait for 8 frames). At higher speed the bot might make a somersault due to suddenly stop.
	*Once object is reached or not detected for >8 frames the bot stops. Since the time lag is about 1 sec, the bot might crash in the object at higher speed, use soft objects or water bottles (object 44)
	*Collision_avoidance is not active in this script, but in case object is suddenly covered, bot stops

## 3)	Road Following: Scripts data_collection_joystick_roadfollowing.ipynb and live_demo_roadfollowing_targetdisp.ipynb
  * added Joystick drive control to data_collection to learn more about the sensitivity of steering
  * added display of generated x y coordinates in the live_demo: so far the xy coordinates are outside of the line
  
## 4) Adjustments
* High CPU usage by jetpot_stats.service if jetbot OLED display is not installed or used
** Solution:
```
#check if jetbot_stats.service is in the folder 
$ cd /etc/systemd/system/
$ ls
#stop service
$ systemctl stop jetbot_stats #start service in case: systemctl start jebot_stats
#remove from starting at boot up
$ systemctl disable jetbot_stats
#delete the file
$ rm /etc/systemd/system/jetbot_stats.service

#if the service is required again then follow using "SD card from scratch" point 10.
```
* Camera settings in camera.py:
```gst-inspect-1.0 nvarguscamerasrc```

```return 'nvarguscamerasrc sensor-mode=4 awblock=false maxperf=true wbmode=1 exposuretimerange="37000000 37000000" aeantibanding=0 ispdigitalgainrange="8 8" gainrange="8 8" ! video/x-raw(memory:NVMM), width=%d, height=%d, format=(string)NV12, framerate=(fraction)%d/1 ! nvvidconv ! video/x-raw, width=(int)%d, height=(int)%d, format=(string)BGRx ! videoconvert ! appsink' % (```
* modify camera.py in cd /jetbot/jetbot followed by cd ..  and then sudo python3 setup.py install to transfer the modified camera.py to build
