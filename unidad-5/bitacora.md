
# Evidencias de la unidad 5

## Actividad 01:
> *En esta actividad vas a poner a funcionar el caso de estudio de la unidad anterior y lo vas a repasar de nuevo. Mira, es muy importante que le dedique un tiempo generoso a revisar de nuevo el caso de estudio, ya que es un ejemplo muy completo y te va a ayudar a entender mejor el resto de la unidad.*

#### Describe cómo se están comunicando el micro:bit y el sketch de p5.js. ¿Qué datos envía el micro:bit?

El micro:bit y el sketch en p5.js se comunican mediante un puerto serial (UART). El micro:bit toma los valores del acelerómetro (`xValue`, `yValue`) y el estado de los botones (`aState`,` bState`), y los envía en una sola línea de texto en formato ASCII separados por comas:

``` py
data = "{},{},{},{}\n".format(xValue, yValue, aState,bState)
```

#### ¿Cómo es la estructura del protocolo ASCII usado?

Los cuatro valores se mandan como texto ASCII, separados por comas. Cada línea de datos termina con un salto de línea (\n).
Usando la App de conexión serial es algo como esto:

`-36,228,True,False\n`

#### Muestra y explica la parte del código de p5.js donde lee los datos del micro:bit y los transforma en coordenadas de la pantalla.

En p5.js se leen los datos del puerto serial con `port.readUntil("\n")`. Luego se dividen en una lista de valores y se transforman en enteros o booleanos, dependiendo de si es `xValue`/`yvalue` o `aState`/`bState` respectivamente. Los valores de X y Y se ajustan para ubicarlos en el centro de la pantalla `(+ windowWidth/2, + windowHeight/2)`.

``` js
    if (port.availableBytes() > 0) {
      let data = port.readUntil("\n");
      if (data) {
        data = data.trim();
        let values = data.split(","); // acá se separan los datos recibidos en cada coma
        if (values.length == 4) {
          microBitX = int(values[0]) + windowWidth / 2; // primer valor es xValue
          microBitY = int(values[1]) + windowHeight / 2; // segundo es yValue
          microBitAState = values[2].toLowerCase() === "true"; // acá se compara el aState a ver si es true
          microBitBState = values[3].toLowerCase() === "true"; // acá el bState
          updateButtonStates(microBitAState, microBitBState);
        } else {
          print("No se están recibiendo 4 datos del micro:bit");
        }
      }
```

#### ¿Cómo se generan los eventos A pressed y B released que se generan en p5.js a partir de los datos que envía el micro:bit?

Para A pressed empieza cuando el aState que envía el micro:bit sea true y el dato pasado sea false, A pressed seguirá siendo true hasta que devuelva false. 

Ejemplo en la app de conexión serial:
```
152,216,False,False
-32,212,True,False // Aquí empieza A pressed, ya que el anterior valor de aState fue false y ahora es true, la acción se ejecuta desde ahora.
-36,228,True,False
-16,296,True,False
28,212,False,False // Aquí termina A pressed, la acción deja de ejecutarse.
```

En p5.js:
``` js
  if (newAState === true && prevmicroBitAState === false) {
    // create a new random color and line length
    lineModuleSize = random(50, 160);
    // remember click position
    clickPosX = microBitX;
    clickPosY = microBitY;
    print("A pressed");
  }

(...)

  prevmicroBitAState = newAState;
```

Para B released va a suceder cuando `bState==false` Y el estado bState pasado sea true.
```
-144,276,False,False
-132,260,False,True // Llega el dato de que bState es true pero el evento no se ejecuta aún.
-144,264,False,True
-136,272,False,True
-132,240,False,True
-148,300,False,False // bState actualmente es false y en el dato anterior fue true, es decir que B fue liberado (B released), en este momento se ejecuta el evento.
```

En p5.js:
``` js
  if (newBState === false && prevmicroBitBState === true) {
    c = color(random(255), random(255), random(255), random(80, 100));
    print("B released");
  }

  prevmicroBitBState = newBState;
```

#### Capturas de pantalla de los algunos dibujos que hayas hecho con el sketch.

<img width="1080" height="400" alt="Diseño sin título (2)" src="https://github.com/user-attachments/assets/0dce8ffc-7f9e-41b2-a9f7-ce0d9d7cdc05" />

<img width="1080" height="400" alt="Diseño sin título (3)" src="https://github.com/user-attachments/assets/9bc4615c-93cc-40f8-b32b-088dcfb93e06" />

## Actividad 02:

>Abre la aplicación, configura el puerto, deja los valores por defecto y presiona Conectar. Selecciona el puerto del micro:bit (mbed Serial port) y presiona Conectar. Luego, en la sección de Recepción de Datos, en Mostrar datos como, selecciona Texto.

### Captura el resultado del experimento anterior. ¿Por qué se ve este resultado?

<img width="1005" height="285" alt="image" src="https://github.com/user-attachments/assets/68d55670-7a33-488b-b468-337b3637cf85" />


>Ahora cambia la opción de Mostrar datos como a Todo en Hex y vuelve a capturar el resultado.

### Captura el resultado del experimento anterior. Lo que ves ¿Cómo está relacionado con esta línea de código?
```py
data = struct.pack('>2h2B', xValue, yValue, int(aState), int(bState))
```

<img width="1025" height="306" alt="image" src="https://github.com/user-attachments/assets/24d53308-52bf-4ff4-9ff2-59c65b084f9e" />


### ¿Qué ventajas y desventajas ves en usar un formato binario en lugar de texto en ASCII?

#### Cambiamos el código del micro:bit a:
```py
# Imports go at the top
from microbit import *
import struct
uart.init(115200)
display.set_pixel(0,0,9)

while True:
    if accelerometer.was_gesture('shake'):
        xValue = accelerometer.get_x()
        yValue = accelerometer.get_y()
        aState = button_a.is_pressed()
        bState = button_b.is_pressed()
        data = struct.pack('>2h2B', xValue, yValue, int(aState), int(bState))
        uart.write(data)
```

### Captura el resultado del experimento. ¿Cuántos bytes se están enviando por mensaje? ¿Cómo se relaciona esto con el formato '>2h2B'? ¿Qué significa cada uno de los bytes que se envían?

<img width="1003" height="296" alt="image" src="https://github.com/user-attachments/assets/9e92b9f0-5033-4d09-b046-8d01e2102638" />

*Al agitarlo una sola vez.*

<img width="1002" height="307" alt="image" src="https://github.com/user-attachments/assets/79175b72-59e2-4395-bd71-0c285bc4f7e7" />

*Al agitarlo varias veces.*


### Recuerda de la unidad anterior que es posible enviar números positivos y negativos para los valores de xValue y yValue. ¿Cómo se verían esos números en el formato '>2h2B'?

>Ahora realiza el siguiente experimento para comparar el envío de datos en ASCII y en binario.

```py
# Imports go at the top
from microbit import *
import struct
uart.init(115200)
display.set_pixel(0,0,9)

while True:
    if accelerometer.was_gesture('shake'):
        xValue = accelerometer.get_x()
        yValue = accelerometer.get_y()
        aState = button_a.is_pressed()
        bState = button_b.is_pressed()
        data = struct.pack('>2h2B', xValue, yValue, int(aState), int(bState))
        uart.write(data)
        uart.write("ASCII:\n")
        data = "{},{},{},{}\n".format(xValue, yValue, aState,bState)
        uart.write(data)
```

### Captura el resultado del experimento. ¿Qué diferencias ves entre los datos en ASCII y en binario? ¿Qué ventajas y desventajas ves en usar un formato binario en lugar de texto en ASCII? ¿Qué ventajas y desventajas ves en usar un formato ASCII en lugar de binario?

<img width="997" height="285" alt="image" src="https://github.com/user-attachments/assets/7d05d185-4671-4a28-9b34-aaa06b28a5c4" />

*Texto*

<img width="1001" height="295" alt="image" src="https://github.com/user-attachments/assets/623e8e88-1e01-452b-aa48-ef28bf69a5ee" />

*Todo en HEX*

<img width="1002" height="291" alt="image" src="https://github.com/user-attachments/assets/1c558b7a-1a7a-4faa-9ef8-079d29bd1f75" />

*Texto+Especiales(HEX)*

## Actividad 03

```py
# Imports go at the top
from microbit import *
import struct
uart.init(115200)
display.set_pixel(0,0,9)

while True:
    """
    xValue = accelerometer.get_x()
    yValue = accelerometer.get_y()
    aState = button_a.is_pressed()
    bState = button_b.is_pressed()
    """
    xValue = 500
    yValue = 524
    aState = True
    bState = False
    data = struct.pack('>2h2B', xValue, yValue, int(aState), int(bState))
    uart.write(data)
    sleep(100) # Envia datos a 10 Hz
```

### Explica por qué en la unidad anterior teníamos que enviar la información delimitada y además marcada con un salto de línea y ahora no es necesario.

### Compara el código de la unidad anterior relacionado con la recepción de los datos seriales que ves ahora. ¿Qué cambios observas?

Unidad anterior:
```js
if (port.availableBytes() >= 6) {
    let data = port.readBytes(6);
    if (data) {
    const buffer = new Uint8Array(data).buffer;
    const view = new DataView(buffer);
    microBitX = view.getInt16(0);
    microBitY = view.getInt16(2);
    microBitAState = view.getUint8(4) === 1;
    microBitBState = view.getUint8(5) === 1;
    updateButtonStates(microBitAState, microBitBState);

    print(`microBitX: ${microBitX} microBitY: ${microBitY} microBitAState: ${microBitAState} microBitBState: ${microBitBState} \n` );

    /*
    microBitX = int(values[0]) + windowWidth / 2;
    microBitY = int(values[1]) + windowHeight / 2;
    microBitAState = values[2].toLowerCase() === "true";
    microBitBState = values[3].toLowerCase() === "true";
    updateButtonStates(microBitAState, microBitBState);
    */
    }
}
```

Actual:
```js
    if (port.availableBytes() > 0) {
      let data = port.readUntil("\n");
      if (data) {
        data = data.trim();
        let values = data.split(",");
        if (values.length == 4) {
          microBitX = int(values[0]) + windowWidth / 2;
          microBitY = int(values[1]) + windowHeight / 2;
          microBitAState = values[2].toLowerCase() === "true";
          microBitBState = values[3].toLowerCase() === "true";
          updateButtonStates(microBitAState, microBitBState);
        } else {
          print("No se están recibiendo 4 datos del micro:bit");
        }
```

> Ahora te voy a pedir que ejecutes el código de p5.js muchas veces y que estés muy atento a la consola. Lo que haremos es a tratar de reproducir un error que tiene este código. El error es de sincronización y se produce cuando los 6 bytes que lee el código de p5.js no corresponden a los mismos 6 bytes que envía el micro:bit.

### ¿Qué ves en la consola? ¿Por qué crees que se produce este error?

Primera ejecución

<img width="750" height="210" alt="image" src="https://github.com/user-attachments/assets/670b7506-94d8-4601-b7eb-03add63709f1" />

<img width="750" height="210" alt="image" src="https://github.com/user-attachments/assets/87d2fcf3-5ccc-4d4e-b69b-890f5ac8ab89" />

Segunda ejecución

<img width="750" height="210" alt="image" src="https://github.com/user-attachments/assets/9df7ef0a-eb0e-4e9a-85ed-78ecc4f5de62" />

Tercera

<img width="750" height="210" alt="image" src="https://github.com/user-attachments/assets/a8e18be9-553d-4048-9cad-f0ba4affb1f4" />

Cuarta

<img width="750" height="210" alt="image" src="https://github.com/user-attachments/assets/4bc2ca74-9a9b-4862-b21b-680668b56443" />

Quinta

<img width="750" height="210" alt="image" src="https://github.com/user-attachments/assets/cf748237-e18f-426c-9257-c387e54de47c" />


> En el caso del micro:bit se enviará un paquete de 8 bytes:
```
Byte 0: Header (0xAA)
Bytes 1-6: Datos (dos enteros de 16 bits y dos bytes para estados)
Byte 7: Checksum (suma de los 6 bytes de datos módulo 256)
```

> De nuevo, el código del micro:bit quedaría así:
```py
from microbit import *
import struct

uart.init(115200)
display.set_pixel(0, 0, 9)

while True:
    xValue = 500
    yValue = 524
    aState = True
    bState = False
    # Empaqueta los datos: 2 enteros (16 bits) y 2 bytes para estados
    data = struct.pack('>2h2B', xValue, yValue, int(aState), int(bState))
    # Calcula un checksum simple: suma de los bytes de data módulo 256
    checksum = sum(data) % 256
    # Crea el paquete con header, datos y checksum
    packet = b'\xAA' + data + bytes([checksum])
    uart.write(packet)
    sleep(100)  # Envía datos a 10 Hz
```

Ahora, el código de p5.js quedaría así:

```js
let serialBuffer = []; // Buffer para almacenar bytes recibidos

let c;
let lineModuleSize = 0;
let angle = 0;
let angleSpeed = 1;
const lineModule = [];
let lineModuleIndex = 0;
let clickPosX = 0;
let clickPosY = 0;

function preload() {
  lineModule[1] = loadImage("02.svg");
  lineModule[2] = loadImage("03.svg");
  lineModule[3] = loadImage("04.svg");
  lineModule[4] = loadImage("05.svg");
}

let port;
let connectBtn;
let microBitConnected = false;

const STATES = {
  WAIT_MICROBIT_CONNECTION: "WAITMICROBIT_CONNECTION",
  RUNNING: "RUNNING",
};
let appState = STATES.WAIT_MICROBIT_CONNECTION;
let microBitX = 0;
let microBitY = 0;
let microBitAState = false;
let microBitBState = false;
let prevmicroBitAState = false;
let prevmicroBitBState = false;

function setup() {
  createCanvas(windowWidth, windowHeight);
  background(255);

  port = createSerial();
  connectBtn = createButton("Connect to micro:bit");
  connectBtn.position(0, 0);
  connectBtn.mousePressed(connectBtnClick);
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open("MicroPython", 115200);
  } else {
    port.close();
  }
}

function updateButtonStates(newAState, newBState) {
  // Generar eventos de keypressed
  if (newAState === true && prevmicroBitAState === false) {
    // create a new random color and line length
    lineModuleSize = random(50, 160);
    // remember click position
    clickPosX = microBitX;
    clickPosY = microBitY;
    print("A pressed");
  }
  // Generar eventos de key released
  if (newBState === false && prevmicroBitBState === true) {
    c = color(random(255), random(255), random(255), random(80, 100));
    print("B released");
  }

  prevmicroBitAState = newAState;
  prevmicroBitBState = newBState;
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}

function readSerialData() {
  // Acumula los bytes recibidos en el buffer
  let available = port.availableBytes();
  if (available > 0) {
    let newData = port.readBytes(available);
    serialBuffer = serialBuffer.concat(newData);
  }

  // Procesa el buffer mientras tenga al menos 8 bytes (tamaño de un paquete)
  while (serialBuffer.length >= 8) {
    // Busca el header (0xAA)
    if (serialBuffer[0] !== 0xaa) {
      serialBuffer.shift(); // Descarta bytes hasta encontrar el header
      continue;
    }

    // Si hay menos de 8 bytes, espera a que llegue el paquete completo
    if (serialBuffer.length < 8) break;

    // Extrae los 8 bytes del paquete
    let packet = serialBuffer.slice(0, 8);
    serialBuffer.splice(0, 8); // Elimina el paquete procesado del buffer

    // Separa datos y checksum
    let dataBytes = packet.slice(1, 7);
    let receivedChecksum = packet[7];
    // Calcula el checksum sumando los datos y aplicando módulo 256
    let computedChecksum = dataBytes.reduce((acc, val) => acc + val, 0) % 256;

    if (computedChecksum !== receivedChecksum) {
      console.log("Checksum error in packet");
      continue; // Descarta el paquete si el checksum no es válido
    }

    // Si el paquete es válido, extrae los valores
    let buffer = new Uint8Array(dataBytes).buffer;
    let view = new DataView(buffer);
    microBitX = view.getInt16(0);
    microBitY = view.getInt16(2);
    microBitAState = view.getUint8(4) === 1;
    microBitBState = view.getUint8(5) === 1;
    updateButtonStates(microBitAState, microBitBState);

    console.log(
      `microBitX: ${microBitX} microBitY: ${microBitY} microBitAState: ${microBitAState} microBitBState: ${microBitBState}`
    );
  }
}

function draw() {
  //******************************************
  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
    microBitConnected = false;
  } else {
    microBitConnected = true;
    connectBtn.html("Disconnect");
  }

  //*******************************************

  switch (appState) {
    case STATES.WAIT_MICROBIT_CONNECTION:
      // No puede comenzar a dibujar hasta que no se conecte el microbit
      // evento 1:
      if (microBitConnected === true) {
        // Preparo todo para el estado en el próximo frame
        print("Microbit ready to draw");
        strokeWeight(0.75);
        c = color(181, 157, 0);
        noCursor();
        port.clear();
        prevmicroBitAState = false;
        prevmicroBitBState = false;
        appState = STATES.RUNNING;
      }

      break;

    case STATES.RUNNING:
      // EVENTO: estado de conexión del microbit
      if (microBitConnected === false) {
        print("Waiting microbit connection");
        cursor();
        appState = STATES.WAIT_MICROBIT_CONNECTION;
        break;
      }

      //EVENTO: recepción de datos seriales del micro:bit

      readSerialData();

      if (microBitAState === true) {
        let x = microBitX;
        let y = microBitY;

        if (keyIsPressed && keyCode === SHIFT) {
          if (abs(clickPosX - x) > abs(clickPosY - y)) {
            y = clickPosY;
          } else {
            x = clickPosX;
          }
        }

        push();
        translate(x, y);
        rotate(radians(angle));
        if (lineModuleIndex != 0) {
          tint(c);
          image(
            lineModule[lineModuleIndex],
            0,
            0,
            lineModuleSize,
            lineModuleSize
          );
        } else {
          stroke(c);
          line(0, 0, lineModuleSize, lineModuleSize);
        }
        angle += angleSpeed;
        pop();
      }

      break;
  }
}

function keyPressed() {
  if (keyCode === UP_ARROW) lineModuleSize += 5;
  if (keyCode === DOWN_ARROW) lineModuleSize -= 5;
  if (keyCode === LEFT_ARROW) angleSpeed -= 0.5;
  if (keyCode === RIGHT_ARROW) angleSpeed += 0.5;
}

function keyReleased() {
  if (key === "s" || key === "S") {
    let ts =
      year() +
      nf(month(), 2) +
      nf(day(), 2) +
      "_" +
      nf(hour(), 2) +
      nf(minute(), 2) +
      nf(second(), 2);
    saveCanvas(ts, "png");
  }
  if (keyCode === DELETE || keyCode === BACKSPACE) background(255);

  // reverse direction and mirror angle
  if (key === "d" || key === "D") {
    angle += 180;
    angleSpeed *= -1;
  }

  // default colors from 1 to 4
  if (key === "1") c = color(181, 157, 0);
  if (key === "2") c = color(0, 130, 164);
  if (key === "3") c = color(87, 35, 129);
  if (key === "4") c = color(197, 0, 123);

  // load svg for line module
  if (key === "5") lineModuleIndex = 0;
  if (key === "6") lineModuleIndex = 1;
  if (key === "7") lineModuleIndex = 2;
  if (key === "8") lineModuleIndex = 3;
  if (key === "9") lineModuleIndex = 4;
}
```

###  Analiza el código, observa los cambios. Ejecuta y luego observa la consola. ¿Qué ves?

Ahora esto se agrega al comienzo del código.
```js
let serialBuffer = []; // Buffer para almacenar bytes recibidos
```

Se crea una función `readSerialData()`  en vez del `if (port.availableBytes() >= 6)` dentro de `draw()`.

```js
function readSerialData() {
  // Acumula los bytes recibidos en el buffer
  let available = port.availableBytes();
  if (available > 0) {
    let newData = port.readBytes(available);
    serialBuffer = serialBuffer.concat(newData);
  }

  // Procesa el buffer mientras tenga al menos 8 bytes (tamaño de un paquete)
  while (serialBuffer.length >= 8) {
    // Busca el header (0xAA)
    if (serialBuffer[0] !== 0xaa) {
      serialBuffer.shift(); // Descarta bytes hasta encontrar el header
      continue;
    }

    // Si hay menos de 8 bytes, espera a que llegue el paquete completo
    if (serialBuffer.length < 8) break;

    // Extrae los 8 bytes del paquete
    let packet = serialBuffer.slice(0, 8);
    serialBuffer.splice(0, 8); // Elimina el paquete procesado del buffer

    // Separa datos y checksum
    let dataBytes = packet.slice(1, 7);
    let receivedChecksum = packet[7];
    // Calcula el checksum sumando los datos y aplicando módulo 256
    let computedChecksum = dataBytes.reduce((acc, val) => acc + val, 0) % 256;

    if (computedChecksum !== receivedChecksum) {
      console.log("Checksum error in packet");
      continue; // Descarta el paquete si el checksum no es válido
    }

    // Si el paquete es válido, extrae los valores
    let buffer = new Uint8Array(dataBytes).buffer;
    let view = new DataView(buffer);
    microBitX = view.getInt16(0);
    microBitY = view.getInt16(2);
    microBitAState = view.getUint8(4) === 1;
    microBitBState = view.getUint8(5) === 1;
    updateButtonStates(microBitAState, microBitBState);

    console.log(
      `microBitX: ${microBitX} microBitY: ${microBitY} microBitAState: ${microBitAState} microBitBState: ${microBitBState}`
    );
  }
}
```

Y ya en `draw()` solo queda lo que estaba al inicio:

```js
function draw() {
  //******************************************
  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
    microBitConnected = false;
  } else {
    microBitConnected = true;
    connectBtn.html("Disconnect");
  }

  //*******************************************
```

Támbien veo que en el switch appstate agregaron al final:
```JS
        port.clear();
        prevmicroBitAState = false;
        prevmicroBitBState = false;
```
Que no estaba antes.

Pequeño pero en case `STATES.RUNNING` cuando se verifica si hay conexión ya hay un `break`.
También ahora se llama a `readSerialData()`.


Primer intento:
<img width="871" height="189" alt="image" src="https://github.com/user-attachments/assets/c0af67bc-ef5d-4851-9238-c52aa4e6b720" />

Segundo intento:
<img width="870" height="148" alt="image" src="https://github.com/user-attachments/assets/4c4b3ae3-691b-418d-83d7-6a6f8123df2c" />

Tercer intento:
<img width="865" height="135" alt="image" src="https://github.com/user-attachments/assets/ef001ded-accb-487a-b8c2-2a3b57e5b2d1" />


> La versión final de los programas de micro:bit y p5.js son las siguientes:

```py
from microbit import *
import struct

uart.init(115200)
display.set_pixel(0, 0, 9)

while True:
    xValue = accelerometer.get_x()
    yValue = accelerometer.get_y()
    aState = button_a.is_pressed()
    bState = button_b.is_pressed()
    # Empaqueta los datos: 2 enteros (16 bits) y 2 bytes para estados
    data = struct.pack('>2h2B', xValue, yValue, int(aState), int(bState))
    # Calcula un checksum simple: suma de los bytes de data módulo 256
    checksum = sum(data) % 256
    # Crea el paquete con header, datos y checksum
    packet = b'\xAA' + data + bytes([checksum])
    uart.write(packet)
    sleep(100)  # Envía datos a 10 Hz
```

```js
let serialBuffer = []; // Buffer para almacenar bytes recibidos

let c;
let lineModuleSize = 0;
let angle = 0;
let angleSpeed = 1;
const lineModule = [];
let lineModuleIndex = 0;
let clickPosX = 0;
let clickPosY = 0;

function preload() {
  lineModule[1] = loadImage("02.svg");
  lineModule[2] = loadImage("03.svg");
  lineModule[3] = loadImage("04.svg");
  lineModule[4] = loadImage("05.svg");
}

let port;
let connectBtn;
let microBitConnected = false;

const STATES = {
  WAIT_MICROBIT_CONNECTION: "WAITMICROBIT_CONNECTION",
  RUNNING: "RUNNING",
};
let appState = STATES.WAIT_MICROBIT_CONNECTION;
let microBitX = 0;
let microBitY = 0;
let microBitAState = false;
let microBitBState = false;
let prevmicroBitAState = false;
let prevmicroBitBState = false;

function setup() {
  createCanvas(windowWidth, windowHeight);
  background(255);

  port = createSerial();
  connectBtn = createButton("Connect to micro:bit");
  connectBtn.position(0, 0);
  connectBtn.mousePressed(connectBtnClick);
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open("MicroPython", 115200);
  } else {
    port.close();
  }
}

function updateButtonStates(newAState, newBState) {
  // Generar eventos de keypressed
  if (newAState === true && prevmicroBitAState === false) {
    // create a new random color and line length
    lineModuleSize = random(50, 160);
    // remember click position
    clickPosX = microBitX;
    clickPosY = microBitY;
    print("A pressed");
  }
  // Generar eventos de key released
  if (newBState === false && prevmicroBitBState === true) {
    c = color(random(255), random(255), random(255), random(80, 100));
    print("B released");
  }

  prevmicroBitAState = newAState;
  prevmicroBitBState = newBState;
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}

function readSerialData() {
  // Acumula los bytes recibidos en el buffer
  let available = port.availableBytes();
  if (available > 0) {
    let newData = port.readBytes(available);
    serialBuffer = serialBuffer.concat(newData);
  }

  // Procesa el buffer mientras tenga al menos 8 bytes (tamaño de un paquete)
  while (serialBuffer.length >= 8) {
    // Busca el header (0xAA)
    if (serialBuffer[0] !== 0xaa) {
      serialBuffer.shift(); // Descarta bytes hasta encontrar el header
      continue;
    }

    // Si hay menos de 8 bytes, espera a que llegue el paquete completo
    if (serialBuffer.length < 8) break;

    // Extrae los 8 bytes del paquete
    let packet = serialBuffer.slice(0, 8);
    serialBuffer.splice(0, 8); // Elimina el paquete procesado del buffer

    // Separa datos y checksum
    let dataBytes = packet.slice(1, 7);
    let receivedChecksum = packet[7];
    // Calcula el checksum sumando los datos y aplicando módulo 256
    let computedChecksum = dataBytes.reduce((acc, val) => acc + val, 0) % 256;

    if (computedChecksum !== receivedChecksum) {
      console.log("Checksum error in packet");
      continue; // Descarta el paquete si el checksum no es válido
    }

    // Si el paquete es válido, extrae los valores
    let buffer = new Uint8Array(dataBytes).buffer;
    let view = new DataView(buffer);
    microBitX = view.getInt16(0) + windowWidth / 2;
    microBitY = view.getInt16(2) + windowHeight / 2;
    microBitAState = view.getUint8(4) === 1;
    microBitBState = view.getUint8(5) === 1;
    updateButtonStates(microBitAState, microBitBState);

  }
}

function draw() {
  //******************************************
  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
    microBitConnected = false;
  } else {
    microBitConnected = true;
    connectBtn.html("Disconnect");
  }

  //*******************************************

  switch (appState) {
    case STATES.WAIT_MICROBIT_CONNECTION:
      // No puede comenzar a dibujar hasta que no se conecte el microbit
      // evento 1:
      if (microBitConnected === true) {
        // Preparo todo para el estado en el próximo frame
        print("Microbit ready to draw");
        strokeWeight(0.75);
        c = color(181, 157, 0);
        noCursor();
        port.clear();
        prevmicroBitAState = false;
        prevmicroBitBState = false;
        appState = STATES.RUNNING;
      }

      break;

    case STATES.RUNNING:
      // EVENTO: estado de conexión del microbit
      if (microBitConnected === false) {
        print("Waiting microbit connection");
        cursor();
        appState = STATES.WAIT_MICROBIT_CONNECTION;
        break;
      }

      //EVENTO: recepción de datos seriales del micro:bit

      readSerialData();

      if (microBitAState === true) {
        let x = microBitX;
        let y = microBitY;

        if (keyIsPressed && keyCode === SHIFT) {
          if (abs(clickPosX - x) > abs(clickPosY - y)) {
            y = clickPosY;
          } else {
            x = clickPosX;
          }
        }

        push();
        translate(x, y);
        rotate(radians(angle));
        if (lineModuleIndex != 0) {
          tint(c);
          image(
            lineModule[lineModuleIndex],
            0,
            0,
            lineModuleSize,
            lineModuleSize
          );
        } else {
          stroke(c);
          line(0, 0, lineModuleSize, lineModuleSize);
        }
        angle += angleSpeed;
        pop();
      }

      break;
  }
}

function keyPressed() {
  if (keyCode === UP_ARROW) lineModuleSize += 5;
  if (keyCode === DOWN_ARROW) lineModuleSize -= 5;
  if (keyCode === LEFT_ARROW) angleSpeed -= 0.5;
  if (keyCode === RIGHT_ARROW) angleSpeed += 0.5;
}

function keyReleased() {
  if (key === "s" || key === "S") {
    let ts =
      year() +
      nf(month(), 2) +
      nf(day(), 2) +
      "_" +
      nf(hour(), 2) +
      nf(minute(), 2) +
      nf(second(), 2);
    saveCanvas(ts, "png");
  }
  if (keyCode === DELETE || keyCode === BACKSPACE) background(255);

  // reverse direction and mirror angle
  if (key === "d" || key === "D") {
    angle += 180;
    angleSpeed *= -1;
  }

  // default colors from 1 to 4
  if (key === "1") c = color(181, 157, 0);
  if (key === "2") c = color(0, 130, 164);
  if (key === "3") c = color(87, 35, 129);
  if (key === "4") c = color(197, 0, 123);

  // load svg for line module
  if (key === "5") lineModuleIndex = 0;
  if (key === "6") lineModuleIndex = 1;
  if (key === "7") lineModuleIndex = 2;
  if (key === "8") lineModuleIndex = 3;
  if (key === "9") lineModuleIndex = 4;
}
```

### ¿Qué cambios tienen los programas y ¿Qué puedes observar en la consola del editor de p5.js?

<img width="1734" height="836" alt="image" src="https://github.com/user-attachments/assets/e8c4ac5a-4450-490d-8319-9cc1b499455e" />

Apply
---

## Actividad 04

```py
from microbit import *
import struct

uart.init(115200)
display.set_pixel(0, 0, 9)

while True:
    xValue = accelerometer.get_x()
    yValue = accelerometer.get_y()
    aState = button_a.is_pressed()
    bState = button_b.is_pressed()
    data = struct.pack('>2h2B', xValue, yValue, int(aState), int(bState))
    checksum = sum(data) % 256
    packet = b'\xAA' + data + bytes([checksum])
    uart.write(packet)
    sleep(100)
```

Para aplicar las cosas me voy a guiar completamente en el ejercicio de la actividad 03.

Agrego al principio:

```js
let serialBuffer = [];
```
Para almacenar los bytes recibidos.

Voy a enfocarme en cambiar esta parte del código

```js
 if (port.available() > 0) {
    let data = port.readUntil("\n");
    if (data && data.length > 0) {
      let values = data.trim().split(",");
      if (values.length === 4) {
        // acelerómetro
        const xValue = parseInt(values[0], 10);
        const yValue = parseInt(values[1], 10);
        if (!Number.isNaN(xValue)) {
          targetX = map(xValue, -1024, 1024, 0, width);
        }
        if (!Number.isNaN(yValue)) {
          targetY = map(yValue, -1024, 1024, 0, height);
        }

        // botones
        if (values[2] === "True") {
          microbitClick();
        }
        if (values[3] === "True") {
          drawMode = (drawMode === 1) ? 2 : 1;
        }
      }
    }
  }
```

Por lo que veo en la actividad 03 toda esa parte del código básicamente se va y mejor para aplicar el framing.
