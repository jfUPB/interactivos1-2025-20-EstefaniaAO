# Unidad 2


## 🛠 Fase: Apply
  ``` py
from microbit import *
import utime
import music

start_STATE = 0
armed_STATE = 1

currentState= start_STATE
totalTime=20000
updateTime=utime.ticks_ms()

def bomba():
    global currentState
    global totalTime
    global updateTime
    if currentState==start_STATE:
        if accelerometer.was_gesture('shake'):
            currentState=armed_STATE
        if button_a.was_pressed() and totalTime<60000:
            totalTime+=1000
            display.scroll(str(int(totalTime/1000)), delay=35)
        if button_b.was_pressed() and totalTime>10000:
            totalTime-=1000
            display.scroll(str(int(totalTime/1000)), delay=35)
    elif currentState==armed_STATE:
        currentTime=utime.ticks_ms()
        if utime.ticks_diff(currentTime, updateTime)>= 1000 and totalTime>0:
            display.scroll(str(int(totalTime/1000)), delay=40)
            updateTime=currentTime
            totalTime-=1000
        if totalTime==0:
            music.play(music.BA_DING)
            currentState=start_STATE
            totalTime=20000
        if pin_logo.is_touched():
            currentState=start_STATE
            totalTime=20000
            
while True:
    bomba()
  ``` 
