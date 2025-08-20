# Unidad 3

## 🔎 Fase: Set + Seek

### Actividad 01
---

*Esta vez construirás tres semáforos concurrente en el micro:bit.*

*Cada uno de los semáforos tendrá unos tiempos en rojo, amarillo y verde diferentes. Recuerda que el micro:bit tiene un solo display de 5x5 leds. Además, todos los leds son de color rojo. Así que tendrás que ser creativo para representar los colores amarillo y verde.*

*Los tiempos para los semáforos serán los siguientes:*

- *Semáforo 1: 5 segundos en rojo, 2 segundos en amarillo y 3 segundos en verde.*

- *Semáforo 2: 3 segundos en rojo, 1 segundo en amarillo y 2 segundos en verde.*

- *Semáforo 3: 4 segundos en rojo, 3 segundos en amarillo y 2 segundos en verde.*

---

#### Código:

~~~py
from microbit import *
import utime

x=0
tiempoVerde = 0
tiempoAmarillo= 0
tiempoRojo= 0

class Semaforo:
    
    def __init__(self, tiempoVerde, tiempoAmarillo, tiempoRojo, x):
        self.tiempoVerde= tiempoVerde
        self.tiempoAmarillo= tiempoAmarillo
        self.tiempoRojo= tiempoRojo
        self.x= x
        self.estado = "Rojo"
        self.tiempoInicio = utime.ticks_ms()

    def update(self):
        tiempoActual = utime.ticks_ms()
        tiempoPasado = utime.ticks_diff(tiempoActual, self.tiempoInicio)

        if self.estado == "Rojo":
            display.set_pixel(self.x, 2, 0) 
            display.set_pixel(self.x, 0, 9)  # LED Rojo
            if tiempoPasado > self.tiempoRojo:
                self.estado = "Verde"
                self.tiempoInicio = tiempoActual

        elif self.estado == "Verde":
            display.set_pixel(self.x, 0, 0) 
            display.set_pixel(self.x, 4, 9)  # LED Verde
            if tiempoPasado > self.tiempoVerde:
                self.estado = "Amarillo"
                self.tiempoInicio = tiempoActual

        elif self.estado == "Amarillo":
            display.set_pixel(self.x, 4, 0) 
            display.set_pixel(self.x, 2, 9)  # LED Amarillo
            if tiempoPasado > self.tiempoAmarillo:
                self.estado = "Rojo"
                self.tiempoInicio = tiempoActual

semaforo1 = Semaforo(3000, 2000, 5000, 0)
semaforo2 = Semaforo(2000, 1000, 3000, 2)
semaforo3 = Semaforo(2000, 3000, 4000, 4)

while True:
    semaforo1.update()
    semaforo2.update()
    semaforo3.update()
~~~

#### ¿Qué ventajas tiene usar una clase (class) en este caso para representar un semáforo?

Se puede usar para diferentes instancias. Así como usé el semáforo 1, 2 y 3, más no tuve que hacer una lógica para cada uno, solo se adaptan los parametros de las clases a cada uno. 

#### ¿Puedes ver cómo la técnica de programación con máquinas de estado y el uso de funciona no bloqueantes te permite que varios semáforos funcionen al mismo tiempo?

Si, ya que son fácilmente adaptables y escalables a otras instancias.



### Actividad 05
---

~~~ py

from microbit import *
import utime
import music

class Event:
    def __init__(self):
        self.value = 0
    def set(self, _val):
        self.value = _val
    def clear(self):
        self.value = 0
    def read(self):
        return self.value

class SerialTask:
    def __init__(self):
        uart.init(baudrate=115200)
    def update(self):
        if uart.any():
            data = uart.read(1)
            if data:
                if data[0] == ord('A'):
                    event.set('A')
                elif data[0] == ord('B'):
                    event.set('B')
                elif data[0] == ord('S'):
                    event.set('S')
                elif data[0] == ord('T'):
                    event.set('T')

class ButtonTask:
    def __init__(self):
        pass
    def update(self):
        if button_a.was_pressed():
            event.set('A')
        elif button_b.was_pressed():
            event.set('B')
        elif accelerometer.was_gesture('shake'):
            event.set('S')
        elif pin_logo.is_touched():
            event.set('T')

class BombTask:
    def __init__(self):
        self.PASSWORD = ['A','B','A']
        self.key = ['']*len(self.PASSWORD)
        self.keyindex = 0
        self.totalTime = 20000
        self.updateTime = utime.ticks_ms()
        self.state = 'CONFIG'
        display.clear()
        display.scroll(str(int(self.totalTime/1000)), delay=40, wait=False)
    def update(self):
        if self.state == 'CONFIG':
            if event.read() == 'A':
                event.clear()
                if self.totalTime < 60000:
                    self.totalTime += 1000
                    display.scroll(str(int(self.totalTime/1000)), delay=35)
            elif event.read() == 'B':
                event.clear()
                if self.totalTime > 10000:
                    self.totalTime -= 1000
                    display.scroll(str(int(self.totalTime/1000)), delay=35)
            elif event.read() == 'S':
                event.clear()
                self.updateTime = utime.ticks_ms()
                self.state = 'ARMED'
        elif self.state == 'ARMED':
            currentTime = utime.ticks_ms()
            if utime.ticks_diff(currentTime, self.updateTime) >= 1000 and self.totalTime > 0:
                self.updateTime = currentTime
                self.totalTime -= 1000
                display.scroll(str(int(self.totalTime/1000)), delay=40)
            if self.totalTime == 0:
                music.play(music.WAWAWAWAA)
                display.show(Image.SKULL)
                self.state = 'EXPLODED'
            elif event.read() == 'A':
                event.clear()
                self.key[self.keyindex] = 'A'
                self.keyindex += 1
            elif event.read() == 'B':
                event.clear()
                self.key[self.keyindex] = 'B'
                self.keyindex += 1
            if self.keyindex == len(self.key):
                passIsOK = True
                for i in range(len(self.key)):
                    if self.key[i] != self.PASSWORD[i]:
                        passIsOK = False
                        break
                if passIsOK:
                    self.totalTime = 20000
                    self.keyindex = 0
                    self.state = 'CONFIG'
                    display.scroll(str(int(self.totalTime/1000)), delay=40)
                else:
                    self.keyindex = 0
        elif self.state == 'EXPLODED':
            if event.read() == 'T':
                event.clear()
                self.totalTime = 20000
                self.updateTime = utime.ticks_ms()
                self.state = 'CONFIG'
                display.scroll(str(int(self.totalTime/1000)), delay=40)

event = Event()
serialTask = SerialTask()
buttonTask = ButtonTask()
bombTask = BombTask()

while True:
    serialTask.update()
    buttonTask.update()
    bombTask.update()


~~~

#### Diagrama

---

<img width="1009" height="1089" alt="Diagrama sin título drawio (1)" src="https://github.com/user-attachments/assets/af6de9c0-72c4-4a1a-8db8-dc60c2c71c04" />

---

| Estado inicial | Evento disparador | Acciones | Estado final |
|----------------|------------------|----------|--------------|
| CONFIG | A | ```if self.totalTime < 60000: self.totalTime += 1000 display.scroll(str(int(self.totalTime/1000)), delay=35)``` | CONFIG |
| CONFIG | B | ```if self.totalTime > 10000: self.totalTime -= 1000 display.scroll(str(int(self.totalTime/1000)), delay=35)``` | CONFIG |
| CONFIG | S | ```self.updateTime = utime.ticks_ms() self.state = 'ARMED'``` | ARMED |
| ARMED | Timer (cada 1000 ms) | ```self.totalTime -= 1000 display.scroll(str(int(self.totalTime/1000)), delay=40)``` | ARMED |
| ARMED | totalTime == 0 | ```music.play(music.WAWAWAWAA) display.show(Image.SKULL) self.state = 'EXPLODED'``` | EXPLODED |
| ARMED | A | ```self.key[self.keyindex] = 'A' self.keyindex += 1``` | ARMED (o CONFIG si password completa y correcta) |
| ARMED | B | ```self.key[self.keyindex] = 'B' self.keyindex += 1``` | ARMED (o CONFIG si password completa y correcta) |
| ARMED | Secuencia correcta (A-B-A) | ```self.totalTime = 20000 self.keyindex = 0 self.state = 'CONFIG' display.scroll(str(int(self.totalTime/1000)), delay=40)``` | CONFIG |
| ARMED | Secuencia incorrecta | ```self.keyindex = 0``` | ARMED |
| EXPLODED | T | ```self.totalTime = 20000 self.updateTime = utime.ticks_ms() self.state = 'CONFIG' display.scroll(str(int(self.totalTime/1000)), delay=40)``` | CONFIG |
