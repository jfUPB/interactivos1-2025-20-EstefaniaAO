# Evidencias de la unidad 4

## Código

[Enlace a la aplicación a modificar](http://www.generative-gestaltung.de/2/sketches/?01_P/P_2_2_3_02)

Código a modificar:

``` js
// P_2_2_3_02
//
// Generative Gestaltung – Creative Coding im Web
// ISBN: 978-3-87439-902-9, First Edition, Hermann Schmidt, Mainz, 2018
// Benedikt Groß, Hartmut Bohnacker, Julia Laub, Claudius Lazzeroni
// with contributions by Joey Lee and Niels Poldervaart
// Copyright 2018
//
// http://www.generative-gestaltung.de
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

/**
 * form mophing process by connected random agents
 * two forms: circle and line
 *
 * MOUSE
 * click               : start a new circe
 * position x/y        : direction and speed of floating
 *
 * KEYS
 * 1-2                 : fill styles
 * 3-4                 : form styles circle/line
 * arrow up/down       : step size +/-
 * f                   : freeze. loop on/off
 * Delete/Backspace    : clear display
 * s                   : save png
 */
'use strict';

var formResolution = 15;
var stepSize = 2;
var distortionFactor = 1;
var initRadius = 150;
var centerX;
var centerY;
var x = [];
var y = [];

var filled = false;
var freeze = false;
var drawMode = 1;

function setup() {
  createCanvas(windowWidth, windowHeight);

  // init shape
  centerX = width / 2;
  centerY = height / 2;
  var angle = radians(360 / formResolution);
  for (var i = 0; i < formResolution; i++) {
    x.push(cos(angle * i) * initRadius);
    y.push(sin(angle * i) * initRadius);
  }

  stroke(0, 50);
  strokeWeight(0.75);
  background(255);
}

function draw() {
  // floating towards mouse position
  centerX += (mouseX - centerX) * 0.01;
  centerY += (mouseY - centerY) * 0.01;

  // calculate new points
  for (var i = 0; i < formResolution; i++) {
    x[i] += random(-stepSize, stepSize);
    y[i] += random(-stepSize, stepSize);
  }

  if (filled) {
    fill(random(255));
  } else {
    noFill();
  }

  switch (drawMode) {
  case 1: // circle
    beginShape();
    // start controlpoint
    curveVertex(x[formResolution - 1] + centerX, y[formResolution - 1] + centerY);

    // only these points are drawn
    for (var i = 0; i < formResolution; i++) {
      curveVertex(x[i] + centerX, y[i] + centerY);
    }
    curveVertex(x[0] + centerX, y[0] + centerY);

    // end controlpoint
    curveVertex(x[1] + centerX, y[1] + centerY);
    endShape();
    break;
  case 2: // line
    beginShape();
    // start controlpoint
    curveVertex(x[0] + centerX, y[0] + centerY);

    // only these points are drawn
    for (var i = 0; i < formResolution; i++) {
      curveVertex(x[i] + centerX, y[i] + centerY);
    }

    // end controlpoint
    curveVertex(x[formResolution - 1] + centerX, y[formResolution - 1] + centerY);
    endShape();
    break;
  }
}

function mousePressed() {
  // init shape on mouse position
  centerX = mouseX;
  centerY = mouseY;

  switch (drawMode) {
  case 1: // circle
    var angle = radians(360 / formResolution);
    var radius = initRadius * random(0.5, 1);
    for (var i = 0; i < formResolution; i++) {
      x[i] = cos(angle * i) * radius;
      y[i] = sin(angle * i) * radius;
    }
    break;
  case 2: // line
    var radius = initRadius * random(0.5, 5);
    var angle = random(PI);

    var x1 = cos(angle) * radius;
    var y1 = sin(angle) * radius;
    var x2 = cos(angle - PI) * radius;
    var y2 = sin(angle - PI) * radius;
    for (var i = 0; i < formResolution; i++) {
      x[i] = lerp(x1, x2, i / formResolution);
      y[i] = lerp(y1, y2, i / formResolution);
    }
    break;
  }
}

function keyReleased() {
  if (key == 's' || key == 'S') saveCanvas(gd.timestamp(), 'png');
  if (keyCode == DELETE || keyCode == BACKSPACE) background(255);
  if (key == '1') filled = false;
  if (key == '2') filled = true;
  if (key == '3') drawMode = 1;
  if (key == '4') drawMode = 2;

  if (keyCode == UP_ARROW) stepSize++;
  if (keyCode == DOWN_ARROW) stepSize--;
  stepSize = max(stepSize, 1);

  // pause/play draw loop
  if (key == 'f' || key == 'F') freeze = !freeze;
  if (freeze) {
    noLoop();
  } else {
    loop();
  }
}

```

[Enlace a la aplicación modificada](URL)

Código modificado:

``` js
'use strict';
let formResolution = 15;
let stepSize = 2;
let initRadius = 150;
let centerX, centerY;
let x = [];
let y = [];
let drawMode = 1;

let port, connectBtn, connectionInitialized = false;
let lastA = false, lastB = false; // <-- booleans

// posición suavizada
let targetX, targetY;

// helper: bytes disponibles (soporta availableBytes() o available())
function bytesAvailable() {
  if (!port) return 0;
  if (typeof port.availableBytes === 'function') return port.availableBytes();
  if (typeof port.available === 'function') return port.available();
  return 0;
}

// helper: convertir "True"/"true"/"1" a boolean
function toBool(s) {
  if (!s) return false;
  const t = String(s).trim().toLowerCase();
  return t === 'true' || t === '1';
}

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
  // no empezar hasta que se conecte el micro:bit
  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
    return; // <-- clave
  }
  connectBtn.html("Disconnect");

  if (port.opened() && !connectionInitialized) {
    port.clear();
    connectionInitialized = true;
  }

  // leer datos solo si hay bytes disponibles
  if (bytesAvailable() > 0) {
    let data = port.readUntil("\n");
    if (data && data.length > 0) {
      let values = data.trim().split(",");
      if (values.length === 4) {
        // acelerómetro
        const xValue = parseInt(values[0], 10);
        const yValue = parseInt(values[1], 10);
        if (!Number.isNaN(xValue)) {
          targetX = width / 2 + map(xValue, -1024, 1024, -200, 200);
        }
        if (!Number.isNaN(yValue)) {
          targetY = height / 2 + map(yValue, -1024, 1024, -200, 200);
        }

        // botones (robusto a "True"/"true"/"1")
        const aState = toBool(values[2]);
        const bState = toBool(values[3]);

        // flancos
        if (aState && !lastA) {
          microbitClick();               // A = click (nueva forma)
        }
        if (bState && !lastB) {
          drawMode = (drawMode === 1) ? 2 : 1;  // B = alternar modo
        }

        lastA = aState;
        lastB = bState;
      }
    }
  }

  // suavizar movimiento (interpolación)
  centerX = lerp(centerX, targetX, 0.2);
  centerY = lerp(centerY, targetY, 0.2);

  // mover puntos
  for (let i = 0; i < formResolution; i++) {
    x[i] += random(-stepSize, stepSize);
    y[i] += random(-stepSize, stepSize);
  }

  noFill();

  if (drawMode === 1) {
    // círculo
    beginShape();
    curveVertex(x[formResolution - 1] + centerX, y[formResolution - 1] + centerY);
    for (let i = 0; i < formResolution; i++) {
      curveVertex(x[i] + centerX, y[i] + centerY);
    }
    curveVertex(x[0] + centerX, y[0] + centerY);
    curveVertex(x[1] + centerX, y[1] + centerY);
    endShape();
  } else {
    // línea
    beginShape();
    curveVertex(x[0] + centerX, y[0] + centerY);
    for (let i = 0; i < formResolution; i++) {
      curveVertex(x[i] + centerX, y[i] + centerY);
    }
    curveVertex(x[formResolution - 1] + centerX, y[formResolution - 1] + centerY);
    endShape();
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
INDEX
``` js
<!DOCTYPE html>
<html>
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.7.2/p5.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.7.2/addons/p5.dom.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.7.2/addons/p5.sound.min.js"></script>

    <!-- Generative Design Dependencies here -->
    <!-- GG Bundled -->
    <script src="https://cdn.jsdelivr.net/gh/generative-design/Code-Package-p5.js@master/libraries/gg-dep-bundle/gg-dep-bundle.js"></script>
    <!-- Opentype -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/opentype.js/0.7.3/opentype.min.js"></script>
    <!-- Rita -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/rita/1.3.11/rita-small.min.js"></script>
    <!-- Chroma -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chroma-js/1.3.6/chroma.min.js"></script>
    <!-- Jquery -->
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"
      integrity="sha256-FgpCb/KJQlLNfOu91ta32o/NMZxltwRo8QtmkMRdAu8="
      crossorigin="anonymous"></script>
<script src="https://unpkg.com/@gohai/p5.webserial@^1/libraries/p5.webserial.js"></script>
    <!-- sketch additions -->

    <link rel="stylesheet" type="text/css" href="style.css">
  </head>
  <body>

    <!-- main -->
    <script src="sketch.js"></script>
  </body>
</html>
``` 

## Video

[Video demostratativo](URL)












