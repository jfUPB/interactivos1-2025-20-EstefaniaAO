
# Evidencias de la unidad 5

<a name="act1"></a>
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

<a name="act2"></a>
## Actividad 02:

>Abre la aplicación, configura el puerto, deja los valores por defecto y presiona Conectar. Selecciona el puerto del micro:bit (mbed Serial port) y presiona Conectar. Luego, en la sección de Recepción de Datos, en Mostrar datos como, selecciona Texto.

### Captura el resultado del experimento anterior. ¿Por qué se ve este resultado?

Se ve así porque los datos ya no se están enviando como texto (ASCII), sino en formato binario. Al mostrarlos como si fueran texto, la aplicación intenta interpretar cada byte como un carácter imprimible, pero muchos de esos bytes no corresponden a caracteres válidos. Por eso aparecen cuadros negros, símbolos raros o letras sin sentido.

<img width="1005" height="285" alt="image" src="https://github.com/user-attachments/assets/68d55670-7a33-488b-b468-337b3637cf85" />


>Ahora cambia la opción de Mostrar datos como a Todo en Hex y vuelve a capturar el resultado.

### Captura el resultado del experimento anterior. Lo que ves ¿Cómo está relacionado con esta línea de código?
```py
data = struct.pack('>2h2B', xValue, yValue, int(aState), int(bState))
```

Lo que se ve en HEX es exactamente la representación binaria de esos valores que se están empaquetando con struct.pack. El formato >2h2B significa: dos enteros cortos (xValue y yValue) y dos enteros sin signo (aState y bState).
Por eso en modo texto era ilegible, pero en modo HEX se puede ver byte por byte cómo se están transmitiendo esos datos de manera compacta.

<img width="1025" height="306" alt="image" src="https://github.com/user-attachments/assets/24d53308-52bf-4ff4-9ff2-59c65b084f9e" />


### ¿Qué ventajas y desventajas ves en usar un formato binario en lugar de texto en ASCII?

Investigando más:
Ventajas:

- Ocupa menos espacio y es más eficiente al transmitir.
- Permite enviar datos numéricos directamente sin convertirlos a cadenas.
- La comunicación es más rápida y precisa porque no hay caracteres adicionales como comas o saltos de línea.
- Es un mnúmero fijo de bytes entonces no tiene que leer hasta cierto punto si no que se sabe que datos son que bytes.

Desventajas:

- No es legible (menos que el ASCII al menos), se ve como símbolos raros. Por lo tanto es más dificil de analizar y entender a simple vista.
- Emisor y receptor tienen que estar como iguales en el formato exacto de empaquetado (orden de bytes, tamaño de cada dato, etc.), tener muy claro que dato lleva a que y de que tamaño es, cosas así.

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

Cada mensaje tiene 6 bytes en total. Esto se debe al formato '>2h2B':

2h: dos enteros cortos con signo (xValue y yValue), cada uno ocupa 2 bytes entonces 4 bytes en total.
2B: dos enteros sin signo de 1 byte cada uno (aState y bState) entonces 2 bytes en total.

En resumen: 2h2B = 2*2 + 2*1 = 6 bytes.

Significado de los bytes:

- Bytes 1 y 2:  xValue
- Bytes 3 y 4:  yValue.
- Byte 5:  estado de aState (0 si no está presionado, 1 si está presionado).
- Byte 6:  estado de bState (0 si no está presionado, 1 si está presionado).

<img width="1003" height="296" alt="image" src="https://github.com/user-attachments/assets/9e92b9f0-5033-4d09-b046-8d01e2102638" />

*Al agitarlo una sola vez.*

<img width="1002" height="307" alt="image" src="https://github.com/user-attachments/assets/79175b72-59e2-4395-bd71-0c285bc4f7e7" />

*Al agitarlo varias veces.*


### Recuerda de la unidad anterior que es posible enviar números positivos y negativos para los valores de xValue y yValue. ¿Cómo se verían esos números en el formato '>2h2B'?

Mirando los valores cuando lo agito a la derecha o a la izquierda:

Como los dos primeros campos son enteros con signo: Los primeros dos numeritos son el signo y los otros dos el valor como tal, viendo en hex:

xValue = 100 en hex sería 00 64. 

Un número negativo se representa con dos f. Ejemplo:

xValue = -100  en hex sería ff 9C. 

(Las traducciones de los números como tal las busqué en internet y con)

<img width="800" height="448" alt="image" src="https://github.com/user-attachments/assets/05554fe0-02b8-4fb8-ae9a-373648626047" />

Entonces si muevo el micro:bit hacia la derecha entonces es un valor positivo en xValue, los primeros dos bytes son algo como 00 xx. Si lo muevo hacia la izquierda, el valor es negativo, los dos bytes son algo como ff xx.

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

En ASCII los datos me aparecen escritos de una forma que entiendo más fácil, como con números separados por comas y hasta True/False para los botones. Es como más directo de leer.

En binario en cambio se ve en hex, lleno de números que no entiendo a simple vista. Ahí ya no sé qué significa cada cosa si no lo analizo bien, aunque sé que es más compacto.


---

Ventajas del binario:

- Los mensajes ocupan menos espacio.
- Se mandan más rápido.
- El código o como la computadora no tiene que convertir los datos, ya van directos.

Desventajas del binario:

- Es más difícil de leer, se ve como raro ya que el 100 positivo es algo y el negativo es otro, no es tan intuitivo como el ASCII para saberselo de memoria.
- Para entender en el lector de seriales que pasa y depurar no ayuda mucho porque no sé qué significa cada byte sin hacer cálculos o buscar.

---

Ventajas del ASCII:

- Es mucho más fácil de leer, aprender y entender.
- Apenas conecto el monitor serial ya veo los valores de una.
- No me tengo que preocupar por interpretar bytes.
- Quizá también más facil para entender en un principio, ya que es claro que se separan por comas y que valor es cada cosa

Desventajas del ASCII:

- Ocupa más espacio porque cada número se manda como varios caracteres.
- Es un poquito más lento.
- Si quiero procesar los datos, toca volver a convertirlos a número.


<img width="997" height="285" alt="image" src="https://github.com/user-attachments/assets/7d05d185-4671-4a28-9b34-aaa06b28a5c4" />

*Texto*

<img width="1001" height="295" alt="image" src="https://github.com/user-attachments/assets/623e8e88-1e01-452b-aa48-ef28bf69a5ee" />

*Todo en HEX*

<img width="1002" height="291" alt="image" src="https://github.com/user-attachments/assets/1c558b7a-1a7a-4faa-9ef8-079d29bd1f75" />

*Texto+Especiales(HEX)*

<a name="act3"></a>
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

Antes, como los datos se mandaban en texto (ASCII), tocaba poner comas para separar cada valor y al final un salto de línea para saber dónde terminaba un mensaje y empezaba el otro. Si no, todo quedaba pegado y no había manera de distinguir qué valor era cuál.

Ahora ya no es necesario porque los datos se mandan en binario, y yo ya sé de antemano que cada mensaje ocupa exactamente 6 bytes (2 para x, 2 para y y 1 para cada botón). Entonces no tengo que usar comas ni saltos de línea, porque simplemente sé que cada movimiento del microbit se lee de 6 en 6 y listo.

### Compara el código de la unidad anterior relacionado con la recepción de los datos seriales que ves ahora. ¿Qué cambios observas?

Unidad actual:
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

- Se revisa si hay al menos 6 bytes disponibles y se leen directo con readBytes(6).
- Después se interpretan con un DataView, sacando los enteros de 16 bits y los bytes según la posición.
- Ya no hay comas, ni saltos de línea, ni texto que convertir: los datos vienen listos en binario.

Anterior:
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

- Se usaba readUntil("\n") para leer hasta el salto de línea.
- Luego tocaba hacer split(",") para separar los datos por comas.
- También había que convertir los valores de texto ("true", "false", números en string) a valores lógicos o enteros.



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

Lo hice muchas veces porque no entendía que pasaba hasta que vi bien y pues tengo una idea general.

En la primera ejecución se ven los valores de x y y normales (500 y 524) y los estados de los botones correctos pero luego aparecen valores raros como -3070 o 3073, que no tienen sentido para el acelerómetro e igual se esta mandando un numero cosntante.

Yo creo que esto se debe a un error de sincronización, el programa de p5.js está leyendo los datos en bloques de 6 bytes, pero a veces no empieza justo en el primer byte del mensaje sino que se “corre” un poco y termina interpretando mal los valores. Es decir, en vez de leer xValue desde donde debería, arranca en medio del dato o se salta un byte, y eso hace que aparezcan números extraños.

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

En la consola ya no se ven esos errores de sincronización. Antes, cuando p5.js agarraba mal los 6 bytes, los valores se desfasaban y aparecían números absurdos.

- El programa del micro:bit ahora ya no manda los datos como texto separado por comas y saltos de línea, sino como un paquete binario estructurado.
- Incluye un header fijo (0xAA) que sirve de marca para saber dónde empieza el paquete.
- Empaqueta las lecturas de los ejes y botones en binario (struct.pack), ocupando menos espacio y siendo más rápido de transmitir.
- Agrega un checksum, que es como un número de control: suma los bytes y permite detectar si el paquete llegó mal o incompleto.

en p5.js

- Ya no lee líneas de texto, ahora junta los bytes que van llegando en un buffer.
- Busca el header 0xAA y sólo procesa un paquete cuando está completo.
- Revisa el checksum: si no coincide, descarta ese paquete.
- Al tener esta estructura, se evitan los desajustes raros que antes hacían que aparecieran números locos como -3070.

Apply
---

<a name="act4"></a>
## Actividad 04

https://editor.p5js.org/estefaao2006/full/nT4KmnlU1

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

Antes usaba esta lógica con readUntil("\n") para leer ASCII:

```
 if (port.available() > 0) {
    let data = port.readUntil("\n");
    if (data && data.length > 0) {
      let values = data.trim().split(",");
      ...
    }
 }
```

Eso ya no sirve porque ahora no hay comas ni saltos de línea, todo viene en binario.

Por lo que veo en la actividad 03 toda esa parte del código básicamente se va para aplicar el framing.
Me doy cuenta de que voy a tener que cambiar más cosas en el código que en la actividad porque yo llamo directamente a la función a la hora de leer los datos. Ahora me toca aplicarlo de forma distinta al borrar todo el `draw()`.

Después de borrar la parte que leía los datos con readUntil("\n"), paso a implementar el framing.

Primero cambio el draw() para que sea el encargado de ir leyendo byte por byte y guardarlos en el buffer que había declarado antes.

```js
function draw() {
while (port.available() > 0) {
const byte = port.read();
serialBuffer.push(byte);
```

Con esto los bytes se van acumulando. Pero como puede entrar cualquier cosa, necesito asegurarme de que el primer byte siempre sea el que marca el inicio del paquete. En este caso, el paquete empieza con 0xAA. Entonces agrego una verificación para limpiar los que no sirvan:

```js
if (serialBuffer[0] !== 0xAA) {
serialBuffer.shift();
continue;
}
```

Ya con eso me aseguro de que los datos que quedan en el buffer son parte de un paquete bueno.

Ahora, cada paquete tiene un tamaño fijo. Entonces, cuando el buffer ya tiene suficientes bytes (8 en este caso),  reviso de una

```js
if (serialBuffer.length >= 8) {
const payload = serialBuffer.slice(1, 7);
const checksum = serialBuffer[7];
const sumCheck = payload.reduce((a, b) => a + b, 0) % 256;
```
esto guiandome en el código de la act. 3
```js
if (checksum === sumCheck) {
```

Si todo coincide, entonces ahora sí puedo interpretar los datos. Para eso DataView, que lee directamente los valores que había empaquetado en el micro:bit.

```js
const dataView = new DataView(new Uint8Array(payload).buffer);
const xValue = dataView.getInt16(0);
const yValue = dataView.getInt16(2);
const aState = dataView.getUint8(4);
const bState = dataView.getUint8(5);
```

Un pequeño arreglo otra vez al código de coordenadas 

```js
targetX = map(xValue, -1024, 1024, 0, width);
targetY = map(yValue, -1024, 1024, 0, height);
```

Y manejo las acciones de los botones igual que antes, pero ahora usando los valores que recibo en binario:

```js
if (aState === 1) {
microbitClick();
}
if (bState === 1) {
drawMode = (drawMode === 1) ? 2 : 1;
}
}
```

```js
serialBuffer = [];
}
}
}
```

Por organizar lo pongo en una funcion aparte y no en el draw.
Antes, toda la lectura se hacía dentro de draw().
Ahora, como tengo una función que procesa los datos binarios, simplemente llamo a readSerialData()

Y inalmente, dentro de draw() en lugar de tener todo el bloque donde leía con readUntil("\n"), ahora simplemente llamo:
```

readSerialData();
```

Código final:

```js
'use strict';
let formResolution = 15;
let stepSize = 2;
let initRadius = 150;
let centerX, centerY;
let x = [];
let y = [];
let drawMode = 1;

let port, connectBtn, connectionInitialized = false;

// posición suavizada
let targetX, targetY;

// buffer para datos binarios
let serialBuffer = [];

function setup() {
  createCanvas(windowWidth, windowHeight);

  // centro inicial
  centerX = width / 2;
  centerY = height / 2;
  targetX = centerX;
  targetY = centerY;

  let angle = radians(360 / formResolution);
  for (let i = 0; i < formResolution; i++) {
    x.push(cos(angle * i) * initRadius);
    y.push(sin(angle * i) * initRadius);
  }

  stroke(0, 50);
  strokeWeight(0.75);
  background(255);

  // micro:bit
  port = createSerial();
  connectBtn = createButton("Connect to micro:bit");
  connectBtn.position(80, 80);
  connectBtn.mousePressed(connectBtnClick);
}

function draw() {
  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
    return;
  }
  connectBtn.html("Disconnect");

  if (port.opened() && !connectionInitialized) {
    port.clear();
    connectionInitialized = true;
  }

  // leer datos del puerto en binario
  readSerialData();

  // suavizar movimiento con interpolación
  centerX = lerp(centerX, targetX, 0.2);
  centerY = lerp(centerY, targetY, 0.2);

  // mover puntos
  for (let i = 0; i < formResolution; i++) {
    x[i] += random(-stepSize, stepSize);
    y[i] += random(-stepSize, stepSize);
  }

  noFill();

  if (drawMode === 1) {
    beginShape();
    curveVertex(x[formResolution - 1] + centerX, y[formResolution - 1] + centerY);
    for (let i = 0; i < formResolution; i++) {
      curveVertex(x[i] + centerX, y[i] + centerY);
    }
    curveVertex(x[0] + centerX, y[0] + centerY);
    curveVertex(x[1] + centerX, y[1] + centerY);
    endShape();
  } else {
    beginShape();
    curveVertex(x[0] + centerX, y[0] + centerY);
    for (let i = 0; i < formResolution; i++) {
      curveVertex(x[i] + centerX, y[i] + centerY);
    }
    curveVertex(x[formResolution - 1] + centerX, y[formResolution - 1] + centerY);
    endShape();
  }
}

function readSerialData() {
  let available = port.availableBytes();
  if (available > 0) {
    let newData = port.readBytes(available);
    serialBuffer = serialBuffer.concat(newData);
  }

  // paquete = 1 header + 2*2 bytes + 2 botones + 1 checksum = 8 bytes
  while (serialBuffer.length >= 8) {
    if (serialBuffer[0] !== 0xaa) {
      serialBuffer.shift();
      continue;
    }
    if (serialBuffer.length < 8) break;

    let packet = serialBuffer.slice(0, 8);
    serialBuffer.splice(0, 8);

    let dataBytes = packet.slice(1, 7);
    let receivedChecksum = packet[7];
    let computedChecksum = dataBytes.reduce((acc, val) => acc + val, 0) % 256;

    if (computedChecksum !== receivedChecksum) {
      console.log("Checksum error");
      continue;
    }

    let buffer = new Uint8Array(dataBytes).buffer;
    let view = new DataView(buffer);

    const xValue = view.getInt16(0);
    const yValue = view.getInt16(2);
    const aState = view.getUint8(4) === 1;
    const bState = view.getUint8(5) === 1;

    if (!Number.isNaN(xValue)) {
      targetX = map(xValue, -1024, 1024, 0, width);
    }
    if (!Number.isNaN(yValue)) {
      targetY = map(yValue, -1024, 1024, 0, height);
    }

    if (aState) {
      microbitClick();
    }
    if (bState) {
      drawMode = (drawMode === 1) ? 2 : 1;
    }
  }
}

function microbitClick() {
  if (drawMode === 1) {
    let angle = radians(360 / formResolution);
    let radius = initRadius * random(0.5, 1);
    for (let i = 0; i < formResolution; i++) {
      x[i] = cos(angle * i) * radius;
      y[i] = sin(angle * i) * radius;
    }
  } else {
    let radiusL = initRadius * random(0.5, 5);
    let angleL = random(PI);
    let x1 = cos(angleL) * radiusL;
    let y1 = sin(angleL) * radiusL;
    let x2 = cos(angleL - PI) * radiusL;
    let y2 = sin(angleL - PI) * radiusL;
    for (let i = 0; i < formResolution; i++) {
      x[i] = lerp(x1, x2, i / formResolution);
      y[i] = lerp(y1, y2, i / formResolution);
    }
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

Nota propuesta: 5

Porque respondí, hice todos los experimentos y las cosas conscientemente. Todo tiene evidencias, las definiciones preguntadas y aprendidas en la unidad 5. Lo único que considero es que no plantee como tal las preguntas porque estaba siguiendo la ruta de pregunta.

# Evidencias de la unidad 5

1. Profundidad de la Indagación: 5

Aunque las preguntas iniciales no fueron planteadas por mí, me dediqué a explorarlas con detalle y a responderlas de manera reflexiva. Por ejemplo, al analizar en qué escenarios un protocolo ASCII podría ser preferible a uno binario, no me limité a dar una respuesta rápida, sino que profundicé en las ventajas relacionadas con su legibilidad y facilidad de depuración. Como se muestra en la [actividad 1](#act1), esta exploración me permitió comprender cómo las decisiones de diseño en los protocolos de comunicación tienen implicaciones prácticas en el flujo de datos.  

También reflexioné sobre la importancia de las estrategias de framing y su relación con la organización de la información transmitida. Aunque no formulé las preguntas por cuenta propia, aproveché las que me fueron planteadas para profundizar en los temas y relacionarlos con contextos de uso reales. Como se ve en la [actividad 2](#act2), al responderlas logré fortalecer mi curiosidad sobre los principios de comunicación de datos, lo que justifica que me ubique en el nivel más alto de este criterio.

2. Calidad de la Experimentación: 5

En mis experimentos busqué no solo comprobar que el sistema funcionara, sino también aislar problemas específicos y comprender sus causas. Realicé pruebas para observar cómo se estructuraban los datos en ASCII y cómo se comportaba la comunicación cuando variaba la información enviada. Como se muestra en la [actividad 2](#act2), este proceso me permitió identificar la relevancia del formato y comprobar la necesidad de un framing que evitara errores de interpretación.  

Además, intenté diseñar escenarios de prueba más exigentes de lo que se pedía inicialmente, con el fin de evaluar la robustez del sistema. En la [actividad 3](#act3) y [actividad 4](#act4), por ejemplo, logré reproducir situaciones en las que aparecían inconsistencias, lo que me permitió comprobar de forma consistente cómo las soluciones propuestas resolvían los problemas. De esta manera, mi experimentación no se limitó a verificar resultados, sino entender como funcionan y verdaderamente entender los términos.  

3. Análisis y Reflexión: 5

En mi bitácora no me limité a verificar que las soluciones funcionaran, sino que reflexioné sobre sus implicaciones y limitaciones. Por ejemplo, al comparar el uso de ASCII con el binario, analicé el trade-off entre eficiencia de transmisión y facilidad de lectura. Como se muestra en la [actividad 3](#act3), este análisis me permitió entender que cada decisión técnica trae ventajas y desventajas que deben considerarse dependiendo del contexto de uso.  

 En la [actividad 3](#act3) y [actividad 4](#act4) (aplicación), por ejemplo, reflexioné sobre la importancia de mantener protocolos simples pero claros, lo que me permitió ver cómo cada paso en la transmisión tiene un efecto directo en la fiabilidad de la comunicación.  

4. Apropiación y Articulación de Conceptos: 5

Durante el desarrollo de las actividades logré apropiarme de los conceptos y explicarlos con mis propias palabras, relacionándolos como un sistema interdependiente. No me limité a describir qué es la comunicación serial, sino que expliqué cómo el flujo de bytes asíncronos requiere de protocolos y framing para convertirse en un intercambio confiable. Como se ve en la [actividad 1](#act1), [actividad 2](#act2) y [actividad 3](#act3), conseguí expresar con claridad esta relación y mostrar cómo los elementos técnicos se complementan.  

También utilicé ejemplos y analogías para explicar los conceptos de una manera más accesible, lo que demuestra que no me limité a memorizar definiciones, sino que los reelaboré en mi propio lenguaje. En la [actividad 4](#act4), esto se refleja en cómo logré articular la importancia de imponer reglas sobre el “caos” de los datos, mostrando así un dominio conceptual sólido y justificando mi ubicación en el nivel más alto de este criterio.  

---

1. En la unidad anterior abordaste la construcción de un protocolo ASCII. En esta unidad realizaste lo propio con un protocolo binario. Realiza una tabla donde compares, según la aplicación que modificaste en la fase de aplicación de ambas unidades, los siguientes aspectos: eficiencia, velocidad, facilidad, usos de recursos. Justifica con ejemplos concretos tomados de las aplicaciones modificadas.
2. ¿Por qué fue necesario introducir framing en el protocolo binario?
3. ¿Cómo funciona el framing?
4. ¿Qué es un carácter de sincronización?
5. ¿Qué es el checksum y para qué sirve?
6. En la función readSerialData() del programa en p5.js:
- ¿Qué hace la función concat? ¿Por qué?
~~~
function readSerialData() {
    let available = port.availableBytes();
    if (available > 0) {
        let newData = port.readBytes(available);
        serialBuffer = serialBuffer.concat(newData);
    }
~~~

- En la función readSerialData() tenemos un bucle que recorre el buffer solo si este tiene 8 o más bytes ¿Por qué?
~~~
  while (serialBuffer.length >= 8) {
    if (serialBuffer[0] !== 0xaa) {
      serialBuffer.shift();
      continue;
    }
~~~

- En el código anterior qué significa 0xaa?

- En el código anterior qué hace la función shift y la instrucción continue? ¿Por qué?

- Si hay menos de 8 bytes qué hace la instrucción break? ¿Por qué?
~~~
    if (serialBuffer.length < 8) break;
~~~

- ¿Cuál es la diferencia entre slice y splice? ¿Por qué se usa splice justo después de slice?
~~~
let packet = serialBuffer.slice(0, 8);
serialBuffer.splice(0, 8);
~~~
- A la siguiente parte del código se le conoce como programación funcional ¿Cómo opera la función reduce?
~~~
     let computedChecksum = dataBytes.reduce((acc, val) => acc + val, 0) % 256;
~~~
¿Por qué se compara el checksum enviado con el calculado? ¿Para qué sirve esto?
~~~
if (computedChecksum !== receivedChecksum) {
    console.log("Checksum error in packet");
    continue;
}
~~~

En el código anterior qué hace la instrucción continue? ¿Por qué?

¿Qué es un DataView? ¿Para qué se usa?

let buffer = new Uint8Array(dataBytes).buffer;
let view = new DataView(buffer);

¿Por qué es necesario hacer estas conversiones y no simplemente se toman tal cual los datos del buffer?
    microBitX = view.getInt16(0) + windowWidth / 2;
    microBitY = view.getInt16(2) + windowHeight / 2;
    microBitAState = view.getUint8(4) === 1;
    microBitBState = view.getUint8(5) === 1;

