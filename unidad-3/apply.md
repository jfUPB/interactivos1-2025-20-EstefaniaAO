# Unidad 3


## 🛠 Fase: Apply

### Actividad 06

---

*En esta actividad vas a transferir la técnica de programación con máquinas de estado a p5.js.*

*Crea la bomba versión 2.0 en p5.js. No olvides que al aplicar la técnica de máquinas de estado en micro:bit se evitaba colocar acciones por fuera de los eventos. En el caso de p5.js será necesario que tengas acciones por fuera de eventos porque es necesario dibujar el canvas en cada frame.*

---

#### Código fuente:

~~~ js

let bombTask;

function setup() {
  createCanvas(400, 400);
  textAlign(CENTER, CENTER);
  textSize(32);
  bombTask = new BombTask();
}

function draw() {
  background(0);
  fill(255);
  bombTask.update();
  bombTask.display();
}

class BombTask {
  constructor() {
    this.PASSWORD = ['A', 'B', 'A'];
    this.key = new Array(this.PASSWORD.length).fill('');
    this.keyindex = 0;
    this.count = 20;
    this.startTime = millis();
    this.state = 'CONFIG';
  }

  update() {
    if (this.state === 'CONFIG') {
    } 
    else if (this.state === 'ARMED') {
      if (millis() - this.startTime > 1000) {
        this.startTime = millis();
        this.count--;
        if (this.count <= 0) {
          this.state = 'EXPLODED';
        }
      }
    } 
  }

  display() {
    if (this.state === 'CONFIG') {
      text("Config: " + this.count, width/2, height/2);
      text("Aumenta: A | Disminuye: B | Armar: S", width/2, height/2 + 40);
    } 
    else if (this.state === 'ARMED') {
      text("Tiempo: " + this.count, width/2, height/2);
      text("Introduce clave: A/B", width/2, height/2 + 40);
    } 
    else if (this.state === 'EXPLODED') {
      text("BOOM!", width/2, height/2);
      text("Reinicia tocando: T", width/2, height/2 + 40);
    }
  }

  keyInput(k) {
    if (this.state === 'CONFIG') {
      if (k === 'A') this.count = min(this.count + 1, 60);
      if (k === 'B') this.count = max(10, this.count - 1);
      if (k === 'S') {
        this.startTime = millis();
        this.state = 'ARMED';
      }
    } 
    else if (this.state === 'ARMED') {
      if (k === 'A' || k === 'B') {
        this.key[this.keyindex] = k;
        this.keyindex++;

        if (this.keyindex === this.PASSWORD.length) {
          let passIsOK = true;
          for (let i = 0; i < this.key.length; i++) {
            if (this.key[i] !== this.PASSWORD[i]) {
              passIsOK = false;
              break;
            }
          }
          if (passIsOK) {
            this.count = 20;
            this.keyindex = 0;
            this.state = 'CONFIG';
          } else {
            this.keyindex = 0;
          }
        }
      }
    } 
    else if (this.state === 'EXPLODED') {
      if (k === 'T') {
        this.count = 20;
        this.startTime = millis();
        this.state = 'CONFIG';
      }
    }
  }
}

function keyPressed() {
  bombTask.keyInput(key.toUpperCase());
}

~~~

### Actividad 07

---

*Vas a practicar de nuevo la técnica de máquina de estados y eventos genéricos, pero esta vez vas a controlar la bomba desde el micro:bit y desde p5.js. TEN PRESENTE que la bomba estará corriendo en p5.js, pero deberás controlarla también desde los botones del micro:bit mediante el puerto serial.*

---

#### Código p5.js:

~~~ js
let bombTask;
let port;
let connectBtn;
let connectionInitialized = false;

function setup() {
  createCanvas(1000, 1000);
  port = createSerial();
  connectBtn = createButton("Connect to micro:bit");
  connectBtn.position(80, 300);
  connectBtn.mousePressed(connectBtnClick);
  textAlign(CENTER, CENTER);
  textSize(32);
  bombTask = new BombTask();
}

function draw() {
  if (port.opened() && !connectionInitialized) {
    port.clear();
    connectionInitialized = true;
  }
  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
  } else {
    connectBtn.html("Disconnect");
  }

  let data = port.read();
  if (data.length > 0) {
    bombTask.keyInput(data.trim());
  }

  background(0);
  fill(255);
  bombTask.update();
  bombTask.display();
}

class BombTask {
  constructor() {
    this.PASSWORD = ['A', 'B', 'A'];
    this.key = new Array(this.PASSWORD.length).fill('');
    this.keyindex = 0;
    this.count = 20;
    this.startTime = millis();
    this.state = 'CONFIG';
  }

  update() {
    if (this.state === 'CONFIG') {
    } else if (this.state === 'ARMED') {
      if (millis() - this.startTime > 1000) {
        this.startTime = millis();
        this.count--;
        if (this.count <= 0) {
          this.state = 'EXPLODED';
        }
      }
    }
  }

  display() {
    if (this.state === 'CONFIG') {
      text("Config: " + this.count, width/2, height/2);
      text("Aumenta: A | Disminuye: B | Armar: S", width/2, height/2 + 40);
    } else if (this.state === 'ARMED') {
      text("Tiempo: " + this.count, width/2, height/2);
      text("Introduce clave: A/B", width/2, height/2 + 40);
    } else if (this.state === 'EXPLODED') {
      text("BOOM!", width/2, height/2);
      text("Reinicia tocando: T", width/2, height/2 + 40);
    }
  }

  keyInput(k) {
    if (this.state === 'CONFIG') {
      if (k === 'A') this.count = min(this.count + 1, 60);
      if (k === 'B') this.count = max(10, this.count - 1);
      if (k === 'S') {
        this.startTime = millis();
        this.state = 'ARMED';
      }
    } else if (this.state === 'ARMED') {
      if (k === 'A' || k === 'B') {
        this.key[this.keyindex] = k;
        this.keyindex++;
        if (this.keyindex === this.PASSWORD.length) {
          let passIsOK = true;
          for (let i = 0; i < this.key.length; i++) {
            if (this.key[i] !== this.PASSWORD[i]) {
              passIsOK = false;
              break;
            }
          }
          if (passIsOK) {
            this.count = 20;
            this.keyindex = 0;
            this.state = 'CONFIG';
          } else {
            this.keyindex = 0;
          }
        }
      }
    } else if (this.state === 'EXPLODED') {
      if (k === 'T') {
        this.count = 20;
        this.startTime = millis();
        this.state = 'CONFIG';
      }
    }
  }
}

function keyPressed() {
  bombTask.keyInput(key.toUpperCase());
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open("MicroPython", 115200);
    connectionInitialized = false;
  } else {
    port.close();
  }
}

~~~

Error de referencia. Ayuda :(

#### Código del micro:bit:

~~~ py
from microbit import * 
uart.init(baudrate=115200)

display.clear ()

while True:
    if button_a.was_pressed ():
        uart.write('A')
    elif button_b.was_pressed ():
        uart.write('B')
    elif accelerometer.was_gesture ('shake'):
        uart.write('S')
    elif pin_logo.is_touched ():
        uart.write('T')
~~~

#### Enlace al editor de p5.js con tu código:

https://editor.p5js.org/estefaao2006/sketches/j6Me_XGWV

