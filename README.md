# 작물의 온습도 관리 시스템
# 1️. 작품소개
MQTT와 Node-Red를 이용하여 사용자가 작물의 상태를 확인하고 센서를 작동하는 시스템


MQTT를 이용하여 각 센서의 측정값(메세지)를 송수신

Node-Red를 통해 측정값을 시각화 및 센서 작동

# 2. MQTT
![image](https://github.com/2023rapa-project/rapa/assets/132196804/f8a23178-36cc-44a3-83ee-41f327a02ee5)

MQTT는 기반의 메시지 송수신 프로토콜입니다.

# 3. Node-Red
![image](https://github.com/2023rapa-project/rapa/assets/132196804/4f2fbb07-eb6e-4e74-a2f3-7e9609e8b0d1)

Node-RED는 하드웨어 장치, API 및 온라인 서비스를 서로 쉽게 연결할 수 있는 그래픽 개발 도구입니다.

# 4. 블록도
![image](https://github.com/2023rapa-project/rapa/assets/132196804/444923cb-db94-4eac-a61b-0d3ea322be90)

# 온습도측정 + 카메라
```python
import time
from sense_hat import SenseHat
import cv2

sense = SenseHat()
cap = cv2.VideoCapture(0) #0 or -1
while(cap.isOpened()):
    try:
        ret, img = cap.read()
        if ret:
            cv2.imshow('camera-0', img)
        else:
            print('no camera!')
        if cv2.waitKey(1) & 0xFF == 27: #esc
            break
        time.sleep(1)
        humidity = sense.get_humidity()
        #print("Humidity: %s %%rH" % humidity)
        temp = sense.get_temperature()
        print("Temperature: %s C" % temp)
        if temp>37: //온도가 37도 이상이면
            sense.clear(255,102,0) //LED on(주황색)
        else: 
            sense.clear() //LED off
    except:
        break
cap.release()
cv2.destroyAllWindows()
```
# 수분감지센서
```python
import time
from pyfirmata import Arduino, util
import paho.mqtt.client as mqtt

board = Arduino('/dev/ttyACM0')

mqttc = mqtt.Client()
brocker_address = "192.168.0.228"
mqttc.connect(brocker_address)

def message(client, userdata, msg): #sub
    print("message received")
    print("message topic= ", msg)
    print("message retain flag= ",message.retain)
mqttc.loop_start()

it = util.Iterator(board)
it.start()
board.analog[1].enable_reporting()

while True:
    try:
        time.sleep(1)
        sensor_value = board.analog[1].read()
        sensor_r=round(sensor_value,1)

        if sensor_value is not None:
            print("water value : ", sensor_r*1000)
    
        mqttc.publish('yj',sensor_r,2)
    except:
        break
mqttc.loop_stop()
```
# 가습기(릴레이모듈)
```python
import RPi.GPIO as GPIO 

import time #sleep함수를쓰기위해 

GPIO.setmode(GPIO.BCM)

GPIO.setup(18,GPIO.OUT) #gpio18번 셋업 ->릴레이모듈

GPIO.setup(21,GPIO.OUT) #gpio21번 셋업 ->LED

print("setup") 

time.sleep(2) #2초 쉬기

for i in range(1,3):

	GPIO.output(18,True)

	print("true")

	time.sleep(2)

	GPIO.output(18,False)

	print("false")

	time.sleep(2)

GPIO.cleanup() 

print("cleanup")

time.sleep(2)
```

# 5. 영상
https://youtu.be/HpI33QaASJ0

