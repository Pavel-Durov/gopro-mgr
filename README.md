gopro-mgr
=========

GoPro Camera API Explanation in Android examples implementation
a
This came to me as I took part in enthusiastic project, an Android application which manages several GoPro cameras simultaneously.
As I searched the web I did found some, but not enough information about GoPro WIFI API. GoPro support contributed their share when they told me that they do not support 3rd party developers. However there was one site that helped me to start : http://m.heropro.chernowii.com.
GoPro camera communication works as Client-Server. If you are connecting to GoPro camera WIFI network you are participant as client and the camera as Server. However when you use GoPro camera with its RC (remote control) the position become opposite, GoPro camera is a client and the RC is aserver.
This project will only cover the case when GoPro camera is a server.

As you connect to GoPro camera WIFI network you are communicate with the camera through its base IP address: 10.5.5.8 on port 80 to its server called Cherokee, so the very basic request to the server in order to see if the cameras server is responding will be : http://10.5.5.8:80.
If you are connected and the server is responded to you, you should see several folders on your screen (depends on camera model/edition). 
All the camera captured media is stored in http://10.5.5.8:80/videos/DCIM , however if your camera is empty you will not see those folders in Cherokee server.

*My work has been tested on GoProHero 3 White Edition camera  mostly but also on GoProHero 3 + White Edition.


GoPro file name and types

On Cherokee server you will find 5 types of files:
JPG – Photo File.
MP4 – Video large file.
LRV – Low resolution video file.
THM – Thumbnail (low resolution JPG for MP4 file presentation)
TS – Live stream format.
m3u8 – Live stream playlist.

Video Files

For each video file that you record on GoPro camera you will get 3 files on the Cherokee server. 
	Example: 
if you recorded file named GOPR0001.MP4 you will see him under the http://10.5.5.8:80/videos/DCIM/001GOPRO url with another two files with the same name, but different extension :  
GOPR0001.THM – JPG low resolution file (the first frame of the video file). GOPR0001.LRV – low resolution file, the same MP4 file but in much lower weight.

Photo Files

Photo files such single photo or photos in burst mode are saved as JPG format as is, but there is a disparity in the file name. 
All the recorded files on GoPro camera (in http://10.5.5.8:80/videos/DCIM ) will be saved with the same file name convention:
Example: GOPR0001.JPG , GOPR0002.MP4 , GOPR0002.LRV
As you can see, the first 4 chars are the prefix of the name and the last 4 chars (before the extension) are the ordinal number which identified the time hierarchy on the camera. 
	
GOPR0001.JPG 

	.JPG  - Extention on the file.
	0001 – Ordinal number.
	GOPR – GoPro file name prefix.




However this convention changes when capturing photos in burst mode. The prefix is replaced by the group identifier.
	Example: files captured in burst mode (3 frames in 1 second):
G00011232.JPG, G0001233.JPG, G0001234.JPG
This time the group prefix is added to the name, however the ordinal number is always continues to grow.
	If you will take another burst mode capture (3 frames in 1 second) you will get another 3 files :
G0021235.JPG, G0021236.JPG, G0021237.JPG
Notice that the last 4 digits are increased by 1 from the max value as was on the camera, and the group prefix is advanced as well, identifying the new group.
	G0021235.JPG
	002 – Group identifier.
	1235 – Ordinal number.
	.JPG - Extention

What happen when we get to GOPR9999 file?
The last 4 digits are the ordinal number of the file name, but when its gets to its maximal value its opens another directory in http://10.5.5.8:80/videos/DCIM 
Example: 
We had a file named GOPRO9999.MP4 in http://10.5.5.8:80/videos/DCIM/001GOPRO  URL ad we captured another file, which advanced the 9999 ordinal number by 1, in this case there will appear another directory in http://10.5.5.8:80/videos/DCIM called 002GOPRO so the new file that we captured will appear on http://10.5.5.8:80/videos/DCIM/002GOPRO/ as GOPR0001.JPG.

GoPro camera remembers the last maximal value of the directory in http://10.5.5.8:80/videos/DCIM so if you have a directory named 999GOPRO in DCIM, event if you will take the SD card and rename the directory 999GOPRO directory to 001GOPRO. He next time that you turn on the GoPro camera it will create 999GOPRO directory automatically.
	The case is different with files, you can take the SD card out of the camera and rename the last file. The ordinal number will continue on.
 GoPro camera does not remember files name, only the directories.



GoPro API http request convention

There is two request conventions for GoPro camera requests:
1.	http:///bacpac/[mode]?t=[password]&p=[parameter]
2.	http:///camera/[mode]?t=[password]&p=[parameter]

[type] = Request type.
[password] = GoPro camera password (the default is goprohero).
[parameter] = Parameter which identify specific function in specified type request.

Example:
Request for photo mode:	http:///camera/CM?t=123456789&p=%01
CM = Change Mode type
123456789 = Password
%01 = parameter

If we wanted to change mode to Video mode, all we need to change the last parameter to 00.
Example: http:///bacpac/PW?t=123456789&p=%00



GoPro API response

For each request to GoPro Cherokee server you get back byte response which tells you if the request executed on camera.
List of GopPro Request-Response documentation:

http://10.5.5.9:80/camera/CM?t=password&p=%00	Video Mode
[00000000 00 00]

http://10.5.5.9:80/camera/CM?t=password&p=%01	Photo Mode
[00000000 00 01]

http://10.5.5.9:80/camera/CM?t=password&p=%02	Burst Mode
[00000000 00 02]

http://10.5.5.9:80/camera/CM?t=password&p=%03	Time Lapse Mode
[00000000 00 03]
			
http://10.5.5.9:80/bacpac/SH?t=password&p=%01	Start Capture
Video Mode    	        [00000000 00 00]
Photo Mode            	[00000000 00 01]
Burst Mode    	        [00000000 00 02]
Time Lapse Mode        	[00000000 00 03]

http://10.5.5.9:80/bacpac/SH?t=password&p=%00	Stop Capture	Video Mode          	  [00000000 00 00]
                                                        		Photo Mode	            [00000000 00 01]
                                                        		Burst Mode    	        [00000000 00 02]
                                                        		Time Lapse Mode	        [00000000 00 03]

http://10.5.5.9:80/camera/VR?t=password&p=%00 WVGA-60fps 	N/A	                      [00000000 00 00]
http://10.5.5.9:80/camera/VR?t=password&p=%02	720p 30fps	N/A                      	[00000000 00 02]
http://10.5.5.9:80/camera/VR?t=password&p=%03	720p 60fps	N/A                     	[00000000 00 03]
http://10.5.5.9:80/camera/VR?t=password&p=%04	960p 30fps	N/A	                      [00000000 00 04]
http://10.5.5.9:80/camera/VR?t=password&p=%06	1080p 30fps	N/A	                      [00000000 00 06]
http://10.5.5.9:80/camera/UP?t=password&p=%01	Set camera rotation up side down	N/A	[00000000 00 01]
http://10.5.5.9:80/camera/UP?t=qwer1234&p=%00	Set camera rotation normal	N/A	      [00000000 00 00]
http://10.5.5.9:80/camera/TI?t=password&p=%01	Time Laps 1 sec	N/A                 	[00000000 00 01]
http://10.5.5.9:80/camera/TI?t=password&p=%02	Time Laps 2 sec	N/A                 	[00000000 00 02]
http://10.5.5.9:80/camera/TI?t=password&p=%05	Time Laps 5 sec	N/A                 	[00000000 00 05]
http://10.5.5.9:80/camera/TI?t=password&p=%0a	Time Laps 10 sec	N/A               	[00 0a]
http://10.5.5.9:80/camera/TI?t=password&p=%1e	Time Laps 30 sec	N/A	                [00 1e]
http://10.5.5.9:80/camera/TI?t=password&p=%3c	Time Laps 60 sec	N/A               	[00 3c]
http://10.5.5.9:80/camera/TI?t=password&p=%00	Time Laps 0.5 sec	N/A               	[00000000 00 00]
http://10.5.5.9:80/camera/LL?t=password&p=%01	Locate Camera - (Camera Beeping)	N/A	[00000000 01]
http://10.5.5.9:80/camera/LL?t=password&p=%00	Stop Camera Locate	N/A             	[00000000 00]
http://10.5.5.9:80/camera/BU?t=password&p=%00	Burst Rate 3/1 	N/A                 	[00000000 00]




There is much more request but their response always the same, always [00000000 0000]

Turn On :
 http://10.5.5.9:80/bacpac/PW?t=123456789&p=%00
Start Preview: 
http://10.5.5.9:80/bacpac/PV?t=qwer1234&p=%02
Stop Preview : 
http://10.5.5.9:80/bacpac/PV?t=qwer1234&p=%00

Delete All: 
http://10.5.5.9:80/camera/DA?t=qwer1234
Delete Last: 
http://10.5.5.9:80/camera/DL?t=qwer1234


Request and communication without byte response

Camera version in JSON format
http://10.5.5.9:8080/videos/MISC/version.txt


GoPro Live stream

GoPro live stream files are located in http://10.5.5.9:80/live URL directory with TS extension.
Example: amba_hls-1.ts, amba_hls-2.ts, amba_hls-3.ts …. amba_hls-16.ts. 
  Those files are very small in low resolution they are created permanently and replaced by new ones in a loop.
In addition to those files there will be a file named amba.m3u8 which is a playlist for those small live video files. 
If you are intent to play live video stream from GoPro camera on your device just insert this URL http://10.5.5.9:80/live/amba.m3u8 in the video player as the link, and it will automatically loop through all the permanent videos.




