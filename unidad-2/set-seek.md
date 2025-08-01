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


#### ¿Por qué este programa permite realizar varias tareas de forma concurrente?
Este programa usa una máquina de estados que revisa constantemente dos cosas: el paso del tiempo y si se ha presionado un botón. Gracias a esto, puede mostrar una secuencia de imágenes (feliz, sonriente y triste) con sus tiempos específicos, mientras también responde de inmediato si el botón A se presiona. Por eso se dice que atiende múltiples eventos a la vez, sin que uno bloquee al otro.


#### Estados, eventos y acciones

- **STATE_INIT**  
  No pasa nada todavía. Se muestra la cara feliz y se guarda el tiempo actual.

- **STATE_HAPPY**  
  - Si presiono el botón A → muestra una cara triste y cambia a `STATE_SAD`.  
  - Si pasan más de 1.5 segundos → muestra una cara sonriente y cambia a `STATE_SMILE`.

- **STATE_SMILE**  
  - Si presiono el botón A → vuelve a mostrar cara feliz y cambia a `STATE_HAPPY`.  
  - Si pasa más de 1 segundo → muestra una cara triste y cambia a `STATE_SAD`.

- **STATE_SAD**  
  - Si presiono el botón A → muestra una cara sonriente y cambia a `STATE_SMILE`.  
  - Si pasan más de 2 segundos → muestra una cara feliz y vuelve a `STATE_HAPPY`.


 
#### Vector de prueba 1

- _Condición inicial:_ El micro:bit está mostrando la cara feliz (STATE_HAPPY).
- _Evento:_ Se presiona el botón A.
- _Resultado esperado:_ Cambia a cara triste (STATE_SAD).
- _Resultado real:_  Si asa. Cambia a STATE_SAD.
