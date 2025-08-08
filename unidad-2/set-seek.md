# Unidad 2

## 🔎 Fase: Set + Seek

### Actividad 01
---

#### ¿Cómo funciona este ejemplo?

Este programa hace que dos LEDs de la micro:bit parpadeen en diferentes posiciones y tiempos. Usa una clase llamada `Pixel` que guarda el estado del LED, el tiempo y cuándo debe cambiar.

Cada pixel empieza en el estado `"Init"`, donde se prende o apaga y guarda el tiempo actual. Luego pasa al estado `"WaitTimeout"`, donde espera hasta que pase un tiempo (el intervalo) para volver a cambiar el LED (encender o apagar).

#### Estados:

- `"Init"` → cuando se inicia el píxel y se muestra por primera vez.
- `"WaitTimeout"` → espera que pase el tiempo para cambiar el LED.

#### Eventos (lo que hace que algo cambie):

- Que pase el tiempo (`utime.ticks_diff(...) > intervalo`)

#### Acciones:

- Mostrar el LED con `display.set_pixel(...)`
- Cambiar el valor del LED (9 o 0)
- Guardar el nuevo tiempo con `utime.ticks_ms()`

### Actividad 02
---


```python
from microbit import *
import utime

class Semaforo:
    def __init__(self):
        self.estado = "Rojo"
        self.tiempoInicio = utime.ticks_ms()

    def update(self):
        tiempoActual = utime.ticks_ms()
        tiempoPasado = utime.ticks_diff(tiempoActual, self.tiempoInicio)

        if self.estado == "Rojo":
            display.clear()
            display.set_pixel(2, 0, 9)  # LED Rojo
            if tiempoPasado > 3000:
                self.estado = "Verde"
                self.tiempoInicio = tiempoActual

        elif self.estado == "Verde":
            display.clear()
            display.set_pixel(2, 4, 9)  # LED Verde
            if tiempoPasado > 3000:
                self.estado = "Amarillo"
                self.tiempoInicio = tiempoActual

        elif self.estado == "Amarillo":
            display.clear()
            display.set_pixel(2, 2, 9)  # LED Amarillo
            if tiempoPasado > 1000:
                self.estado = "Rojo"
                self.tiempoInicio = tiempoActual

semaforo = Semaforo()

while True:
    semaforo.update()
```

#### ¿Cómo funciona?
El semáforo empieza en rojo, luego pasa a verde, después a amarillo, y vuelve a rojo otra vez. Cada color dura cierto tiempo. Cambia automáticamente con un contador de milisegundos.

#### Estados
- Rojo
- Verde
- Amarillo

#### Eventos
- Pasa el tiempo (cuando el tiempo supera un valor se cambia de estado)

#### Acciones
- Encender un LED según el color
- Limpiar la pantalla
- Cambiar al siguiente estado
- Reiniciar el tiempo para el siguiente cambio

### Actividad 03:
---

<img width="838" height="1022" alt="image" src="https://github.com/user-attachments/assets/14d51a8a-7bf4-4143-9447-cfdb1bd57e35" />

```python
from microbit import *
import utime

STATE_INIT = 0
STATE_HAPPY = 1
STATE_SMILE = 2
STATE_SAD = 3

HAPPY_INTERVAL = 1500
SMILE_INTERVAL = 1000
SAD_INTERVAL = 2000


currentState= STATE_INIT
startTime=0
interval=0

def tarea1():
    global currentState
    global startTime
    global interval
    
    if currentState == STATE_INIT:
        display.show(Image.HAPPY)
        startTime = utime.ticks_ms()
        interval = HAPPY_INTERVAL
        currentState= STATE_HAPPY
    elif currentState == STATE_HAPPY:
       if utime.ticks_diff(utime.ticks_ms(), startTime) > interval:
            display.show(Image.SMILE)
            startTime= utime.ticks_ms()
            interval = SMILE_INTERVAL
            currentState= STATE_SMILE
       if button_a.was_pressed():
           display.show(Image.SAD)
           interval = SAD_INTERVAL
           startTime= utime.ticks_ms()
           currentState= STATE_SAD
    elif currentState == STATE_SMILE:
        if utime.ticks_diff(utime.ticks_ms(), startTime) > interval:
            display.show(Image.SAD)
            interval = SAD_INTERVAL
            startTime= utime.ticks_ms()
            currentState= STATE_SAD
        if button_a.was_pressed():
           display.show(Image.HAPPY)
           interval = HAPPY_INTERVAL
           startTime= utime.ticks_ms()
           currentState= STATE_HAPPY
    elif currentState == STATE_SAD:
        if utime.ticks_diff(utime.ticks_ms(), startTime) > interval:
            display.show(Image.HAPPY)
            interval = HAPPY_INTERVAL
            startTime= utime.ticks_ms()
            currentState= STATE_HAPPY
        if button_a.was_pressed():
            display.show(Image.SMILE)
            interval = SMILE_INTERVAL
            startTime= utime.ticks_ms()
            currentState= STATE_SMILE
    else:
        display.show(Image.RABBIT)

while True:
    tarea1()
```

<img width="1327" height="841" alt="image" src="https://github.com/user-attachments/assets/8fae25c0-f958-4319-9be7-8f88d403fd90" />


#### ¿Por qué este programa permite realizar varias tareas de forma concurrente?
Este programa usa una máquina de estados que revisa constantemente dos cosas: el paso del tiempo y si se ha presionado un botón. Gracias a esto, puede mostrar una secuencia de imágenes (feliz, sonriente y triste) con sus tiempos específicos, mientras también responde de inmediato si el botón A se presiona. Por eso se dice que atiende múltiples eventos a la vez, sin que uno bloquee al otro.


#### Estados, eventos y acciones
Estados:
- **STATE_INIT**  
  No pasa nada todavía. Acción: Se muestra la cara feliz y se guarda el tiempo actual.

- **STATE_HAPPY**  
  - Evento: Si presiono el botón A → Acción: muestra una cara triste y cambia a `STATE_SAD`.  
  - Evento: Si pasan más de 1.5 segundos → Acción: muestra una cara sonriente y cambia a `STATE_SMILE`.

- **STATE_SMILE**  
  - Evento: Si presiono el botón A → Acción: vuelve a mostrar cara feliz y cambia a `STATE_HAPPY`.  
  - Evento: Si pasa más de 1 segundo → Acción: muestra una cara triste y cambia a `STATE_SAD`.

- **STATE_SAD**  
  - Evento: Si presiono el botón A → Acción: muestra una cara sonriente y cambia a `STATE_SMILE`.  
  - Evento: Si pasan más de 2 segundos → Acción: muestra una cara feliz y vuelve a `STATE_HAPPY`.

#### Vectores de prueba
 
#### Vector de prueba 1

- **Condición inicial:** El micro:bit está mostrando la cara feliz (`STATE_HAPPY`).
- **Evento:** Se presiona el botón A.
- **Resultado esperado:** Cambia a cara triste (`STATE_SAD`).
- **Resultado real:**  Cambia a `STATE_SAD`.

---

#### Vector de prueba 2

- **Condición inicial:** El micro:bit está mostrando la cara feliz (`STATE_HAPPY`), y pasa el tiempo del intervalo sin que se presione ningún botón.
- **Evento:** Transcurre más de 1500 ms sin interacción.
- **Resultado esperado:** Cambia a cara sonriente (`STATE_SMILE`).
- **Resultado real:**  Cambia automáticamente a `STATE_SMILE`.

---

#### Vector de prueba 3

- **Condición inicial:** El micro:bit está mostrando la cara triste (`STATE_SAD`).
- **Evento:** Se presiona el botón A *antes de que se cumpla el tiempo del intervalo*.
- **Resultado esperado:** Cambia a cara sonriente (`STATE_SMILE`).
- **Resultado real:**  Cambia a `STATE_SMILE` inmediatamente.
