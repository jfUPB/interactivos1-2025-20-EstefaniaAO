# Unidad 2

## 🛠 Fase: Apply

### Actividad 04 y 05

---
*Diseña la máquina de estados para solucionar el siguiente problema:*

*En un escape room se requiere construir una aplicación para controlar una bomba temporizada. El circuito de control de la bomba está compuesto por cuatro sensores, denominados UP (botón A), DOWN (botón B), touch (botón de touch) y ARMED (el gesto de shake de acelerómetro). Tiene dos actuadores o dispositivos de salida que serán un display (la pantalla de LEDs) y un speaker.*

*El controlador funciona así:*

*Inicia en modo de configuración, es decir, sin hacer cuenta regresiva aún, la bomba está desarmada. El valor inicial del conteo regresivo es de 20 segundos.*

*En el modo de configuración, los pulsadores UP y DOWN permiten aumentar o disminuir el tiempo inicial de la bomba.*

*El tiempo se puede programar entre 10 y 60 segundos con cambios de 1 segundo. No olvides usar utime.ticks_ms() para medir el tiempo. Además, 1 segundo equivale a 1000 milisegundos.*

*Hacer shake (ARMED) arma la bomba, es decir, inicia el conteo regresivo.*

*Una vez armada la bomba, comienza la cuenta regresiva que será visualizada en la pantalla de LED*

*La bomba explotará (speaker) cuando el tiempo llegue a cero.*

*Para volver a modo de configuración deberás tocar el botón touch.*

---

#### Diagrama

<img width="774" height="733" alt="Diagrama sin título drawio" src="https://github.com/user-attachments/assets/32cba7ce-a95e-464c-b54b-ee778f224544" />

---

El sistema tiene **dos estados principales**:  
- `start_STATE` → Modo configuración (bomba desarmada).
  - **Eventos y transiciones**:
    - **Botón A** (`button_a.was_pressed()` y `totalTime < 60000`):  
    Aumenta el tiempo en +1 segundo (`totalTime += 1000`).  
    Muestra el nuevo tiempo en la pantalla (`display.scroll`).  
    - **Botón B** (`button_b.was_pressed()` y `totalTime > 10000`):  
    Disminuye el tiempo en −1 segundo (`totalTime -= 1000`).  
    Muestra el nuevo tiempo en la pantalla (`display.scroll`).  
    - **Agitar micro:bit** (`accelerometer.was_gesture('shake')`):  
    Cambia al estado `armed_STATE` para iniciar la cuenta regresiva.
- `armed_STATE` → Modo armado (cuenta regresiva).
  - **Eventos y transiciones**:
    - **Cuenta atrás**:  
    Si ha pasado 1 segundo (`utime.ticks_diff(currentTime, updateTime) >= 1000`)  
    y `totalTime > 0`:  
      - Muestra el tiempo restante (`display.scroll`).  
      - Actualiza el contador (`totalTime -= 1000`).  
      - Guarda el nuevo tiempo de referencia (`updateTime = currentTime`).
    - **Tiempo = 0**:  
      - Reproduce un sonido de explosión (`music.play(music.WAWAWAWAA)`).  
      - Regresa a `start_STATE` y reinicia el tiempo a 20 segundos (`totalTime = 20000`).
    - **Botón touch** (`pin_logo.is_touched()`):  
      - Regresa a `start_STATE` y reinicia el tiempo a 20 segundos.


#### Código:



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
          music.play(music.WAWAWAWAA)
          currentState=start_STATE
          totalTime=20000
      if pin_logo.is_touched():
          currentState=start_STATE
          totalTime=20000
          
while True:
  bomba()
```

---

#### Flujo general
1. Inicia en `start_STATE` con 20 segundos.  
2. El usuario ajusta el tiempo entre 10 y 60 segundos con los botones A y B.  
3. Al agitar (`shake`), se pasa a `armed_STATE` y comienza la cuenta regresiva.  
4. El display muestra el tiempo cada segundo.  
5. Si el tiempo llega a 0, suena la alarma y vuelve a `start_STATE`.  
6. Si se toca el logo (`pin_logo`), se cancela y vuelve a `start_STATE`.


---
 
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



