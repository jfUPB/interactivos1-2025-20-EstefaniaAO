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
 
#### Vector de prueba 1

- **Condición inicial:** El micro:bit está en (`start_STATE`).
- **Evento:** Se agita el micro:bit.
- **Resultado esperado:** Cambia a (`armed_STATE`) y empieza el contador.
- **Resultado real:**  Funciona, cambia a `armed_STATE`.

---

#### Vector de prueba 2

- **Condición inicial:** El micro:bit está en (`start_STATE`).
- **Evento:** Se presiona el botón A.
- **Resultado esperado:** Aumenta el tiempo total en un segundo y se muestra el número.
- **Resultado real:**  Funciona.

---

#### Vector de prueba 3

- **Condición inicial:** El micro:bit está en (`start_STATE`).
- **Evento:** Se presiona el botón B.
- **Resultado esperado:** Disminuye el tiempo total en un segundo y se muestra el número.
- **Resultado real:**  Funciona.

---

#### Vector de prueba 4

- **Condición inicial:** El micro:bit está en (`armed_STATE`).
- **Evento:** Transcurre el tiempo total de la bomba sin interrupción.
- **Resultado esperado:** El contador se detiene al llegar a cero, deja de mostrar números y suena el speaker.
- **Resultado real:**  Funciona.

---

#### Vector de prueba 5

- **Condición inicial:** El micro:bit está en (`armed_STATE`).
- **Evento:** Se presiona el touch del micro:bit.
- **Resultado esperado:** Cambia a (`start_STATE`).
- **Resultado real:**  Funciona. El micro:bit está en (`start_STATE`).


