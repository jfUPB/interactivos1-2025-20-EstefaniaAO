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
      // Estado de configuración: ajustar tiempo
    } 
    else if (this.state === 'ARMED') {
      // Contador de tiempo
      if (millis() - this.startTime > 1000) {
        this.startTime = millis();
        this.count--;
        if (this.count <= 0) {
          this.state = 'EXPLODED';
        }
      }
    } 
    else if (this.state === 'EXPLODED') {
      // Espera reinicio
    }
  }

  display() {
    if (this.state === 'CONFIG') {
      text("Config: " + this.count, width/2, height/2);
      text("Aumenta: A | Disminuye: B | Armar: ESPACIO", width/2, height/2 + 40);
    } 
    else if (this.state === 'ARMED') {
      text("Tiempo: " + this.count, width/2, height/2);
      text("Introduce clave: A/B", width/2, height/2 + 40);
    } 
    else if (this.state === 'EXPLODED') {
      text("BOOM!", width/2, height/2);
      text("Reinicia tocando: R", width/2, height/2 + 40);
    }
  }

  keyInput(k) {
    if (this.state === 'CONFIG') {
      if (k === 'A') this.count = min(this.count + 1, 60);
      if (k === 'B') this.count = max(10, this.count - 1);
      if (k === ' ') {  // ESPACIO para armar
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
      if (k === 'R') { // Reinicio
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

#### Código fuente:

~~~ js

~~~

#### Código p5.js:

~~~ py
from microbit import * 
import utime 
import radio

display.clear ()

class RadioTask:
def __init__(self):
radio. config (group=16)
radio.on()
def update(self):
if button_a.was_pressed () :
radio. send ('A')
elif button_b. was_pressed () :
radio. send ('B')
elif accelerometer was_gesture ('shake'):
radio send ('S')
elif pin_logo. is_touched () :
radio. send ('T')
radioTask = RadioTask()
while True:
radioTask.update ()

~~~

#### Enlace al editor de p5.js con tu código:



#### Código del micro:bit:



