# Unidad 1

## 🛠 Fase: Apply

--- 
### Actividad 05:
---
En esta actividad comenzamos a construir un sistema físico interactivo básico usando un micro:bit como dispositivo de entrada. La idea es que al presionar el botón A del micro:bit, se genere una señal que pueda ser leída por otro software, como p5.js, para producir una reacción visual en pantalla. Este ejercicio fue desarrollo cruzado, ya que programamos en el computador y luego enviamos la versión al dispositivo como tal.

Primero, importamos todas las funciones necesarias con from microbit import *, lo que nos permite acceder a los botones, al sistema de comunicación serial y a los sensores del micro:bit.

``` py
from microbit import *
```

Después, creamos un bucle while True: que se repite constantemente (esto es típico en microcontroladores, donde no hay un "final" como tal, sino que siempre están escuchando o esperando una acción).

Dentro del bucle, usamos if button_a.was_pressed(): para verificar si el botón A fue presionado. Elegimos was_pressed() en vez de is_pressed() porque queremos detectar solo el momento en que se presiona, no si el botón sigue presionado (ya que eso podría enviar muchos mensajes seguidos y crear errores en la lectura del otro lado). Es decir, si hubieramos usado is en vez de was se hubiera mandado el mensaje varias veces si lo mantenemos presionado, aquí nada mas se toma una vez cuando presionamos y soltamos.

Luego inicializamos la comunicación serial con uart.init(baudrate=115200). Esta línea es muy importante porque define la velocidad a la que se envían los datos desde el micro:bit. Elegimos 115200 baudios, que es una velocidad estándar para este tipo de comunicación y es compatible con muchas plataformas.

Cuando se detecta que el botón fue presionado, usamos uart.write('A') para enviar un carácter (en este caso la letra A) por el puerto serial. Este mensaje será recibido más adelante por el programa hecho en p5.js.


``` py
from microbit import *

uart.init(baudrate=115200)

while True:
    if button_a.was_pressed():
       uart.write('A')
```

Debo declarar ciertas variables como globales, las definimos así ya que necesitamos acceder a estas en diferentes momento, por lo tanto se deben declarar fuera de la función para que no sean locales. 

``` js
let port;
let connectBtn;

function setup() {
    createCanvas(400, 400); // crea un lienzo con 400x400 pixeles
    background(220); // se pinta el canvas de un color casi blanco.
    port = createSerial(); /* esta funcion crea una referencia al objeto que va a representar
    este puerto, la biblioteca nos permite acceder a esta funcion*/
    connectBtn = createButton('Connect to micro:bit');
    connectBtn.position(80, 300); // esta funcion mueve al boton a esa posicion.
    connectBtn.mousePressed(connectBtnClick); /*llama a la funcion connectBtnClick cuando se presiona, 
    se usa el "connectBtn." para que el programa sepa a que objeto aplicarle la funcion*/
}

function draw() {
}
```
Creamos una funcion donde si el puerto esta abierto lo cierra, pero si esta cerrado lo abre.

``` js
function connectBtnClick() {
    if (!port.opened()) {
        port.open('MicroPython', 115200);
    } else {
        port.close();
    }
}
```

``` js
function draw() {
    background(220);

    if (port.availableBytes() > 0) // revisamos si han llegado datos, todo esto se hace para darle retroalimentacion al usuario.
    {
        let dataRx = port.read(1);
        if (dataRx == "A") {
        fill("red");
    }
    } else {
        fill("green");
    }

    rectMode(CENTER);
    rect(width / 2, height / 2, 50, 50);

    if (!port.opened()) {
        connectBtn.html("Connect to micro:bit");
    } else {
        connectBtn.html("Disconnect");
    }
}
```

Sin embargo, no me interesa si fue presionado, me interesa si esta siendo presionado actualmente, por lo que modifico el codigo inicial en python.


``` py
from microbit import *

uart.init(baudrate=115200)

while True:

    if button_a.is_pressed():
        uart.write('A')
    else:
        uart.write('N')

    sleep(100)
```
``` js
let port;
let connectBtn;
let connectionInitialized = false;

function setup() {
  createCanvas(400, 400);
  background(220);
  port = createSerial();
  connectBtn = createButton("Connect to micro:bit");
  connectBtn.position(80, 300);
  connectBtn.mousePressed(connectBtnClick);
}

function draw() {
  background(220);

  if (port.opened() && !connectionInitialized) {
    port.clear();
    connectionInitialized = true;
  }

  if (port.availableBytes() > 0) {
    let dataRx = port.read(1);
    if (dataRx == "A") {
      fill("red");
    } else if (dataRx == "N") {
      fill("green");
    }
  }

  rectMode(CENTER);
  rect(width / 2, height / 2, 50, 50);

  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
  } else {
    connectBtn.html("Disconnect");
  }
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open("MicroPython", 115200);
    connectionInitialized = false;
  } else {
    port.close();
  }
}
```


### Actividad 06
---
``` py
from microbit import *

uart.init(baudrate=115200)
display.show(Image.BUTTERFLY)

while True:

    if button_a.is_pressed():
        uart.write('I')
    elif  button_b.is_pressed():
        uart.write('D')
    else:
        uart.write('N')
           
    sleep(20)

```
``` js
let port;
let connectBtn;
let x= 200
function setup() {
  createCanvas(400, 400);

  port = createSerial();
  connectBtn = createButton('Connect to micro:bit');
  connectBtn.position(80, 300);
  connectBtn.mousePressed(connectBtnClick);
    circle(x,200, 50)
}

function draw() {
  background(220);


      if(port.availableBytes() > 0){
        let dataRx = port.read(1);
        if(dataRx == 'I' && x>25) {
        fill("blue")
        x-=5
        }
        else if(dataRx == 'D' && x<375){
        fill("red")
          x+=5
        }}
      if (!port.opened()) {
        connectBtn.html('Connect to micro:bit');
    }
    else {
        connectBtn.html('Disconnect');
    }
    circle(x,200, 50)
}

function connectBtnClick() {
    if (!port.opened()) {
        port.open('MicroPython', 115200);
    } else {
        port.close();
    }
}
``` 


