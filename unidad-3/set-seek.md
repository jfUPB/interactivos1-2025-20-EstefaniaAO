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
