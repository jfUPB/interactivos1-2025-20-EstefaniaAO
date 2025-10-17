
# Evidencias de la unidad 7

# Índice de Actividades

1. [Actividad 01](#actividad-01)
2. [Actividad 02](#actividad-02)
3. [Actividad 03](#actividad-03)
4. [Actividad 04](#actividad-04)
5. [Actividad 05](#actividad-05)
6. [Autoevaluación](#autoevaluación)



## Actividad 01:

### ¿Qué URL de Dev Tunnels obtuviste? ¿Por qué crees que necesitamos usar esta URL en lugar de http://localhost:3000 o la IP local de tu computador para que el celular se conecte?

#### https://5t0fc03h-3000.use2.devtunnels.ms/

Ya que el computador y el celular pueden no estar conectados a la misma red local. Cuando estas hosteando localmente solo es para ese mismo computador, más no es un link que funcionaría en todos los demás.

### Describe brevemente qué hace npm install y npm start.

npm significa Node Package Manager, y sirve para gestionar las dependencias y scripts de un proyecto hecho con Node.js.

- npm install: descarga e instala todos los paquetes y dependencias listados en el archivo package.json. Esto incluye las librerías necesarias para que el proyecto funcione (por ejemplo, Express, Socket.io, etc.).
- npm start: ejecuta el script definido como “start” en package.json, que normalmente lanza el servidor del proyecto. En este caso, inicia la aplicación web para que empiece a escuchar conexiones desde el puerto 3000 (o el que esté configurado).

### ¿Qué mensajes observaste en la terminal del servidor al conectar el cliente de escritorio y el cliente móvil? ¿Eran diferentes los mensajes o identificadores?

Los mensajes fueron los mismos. Cada vez que se conectaba un cliente —ya fuera desde el navegador del computador o desde el celular— aparecía el mensaje "New client connected" en la terminal.
Esto ocurre porque el código del servidor solo imprime un texto genérico al detectar una conexión, sin mostrar el identificador del socket que se conectó.

<img width="150" height="34" alt="image" src="https://github.com/user-attachments/assets/440ff650-65ef-42b2-9fa8-f162ba13f95a" />
<img width="155" height="71" alt="image" src="https://github.com/user-attachments/assets/d728ccb9-2199-475a-9c0a-307fec4f550b" />

```js
io.on('connection', (socket) => {
    console.log('New client connected');
    socket.on('message', (message) => {
        console.log('Received message =>', message);
        socket.broadcast.emit('message', message);
    });

    socket.on('disconnect', () => {
        console.log('Client disconnected');
    });
});
```

Solo se registra que un cliente se conectó sin importar cual, es decir, solo busca como tal el evento de conexión.

### Describe el comportamiento observado: ¿Funcionó la interacción? ¿Hubo algún retraso (latencia)?

Sí, la interacción funcionó bien, los dos dispositivos se conectaron sin problema y se podían ver los mensajes entre ellos. Solo noté un pequeño retraso, pero nada grave. Creo que ese delay podía deberse al celular, porque al intentar arrastrar en la pantalla a veces se movía toda la ventana en lugar de registrar el drag, entonces tal vez fue más por eso que por la conexión. En general todo funcionó bien y se notaba que el envío de mensajes por Socket.io estaba activo.

## Actividad 02:

### Explica con tus propias palabras: ¿Por qué es necesario Dev Tunnels en este escenario y cómo funciona conceptualmente?

Es necesario Dev Tunnels porque estamos trabajando con un cliente móvil y un servidor que corre en mi computadora. Sin un túnel de desarrollo, el celular no podría conectarse directamente al servidor porque normalmente está detrás de un router o firewall que bloquea accesos externos. Conceptualmente, Dev Tunnels crea un “puente” seguro entre mi máquina local y la red pública, exponiendo temporalmente el servidor local para que cualquier dispositivo, en este caso el móvil, pueda comunicarse con él como si fuera parte de la misma red.

### Describe la función de touchMoved() y por qué se usa la variable threshold en el cliente móvil.

La función `touchMoved()` detecta cuando el usuario arrastra el dedo sobre la pantalla del móvil, permitiendo registrar el movimiento continuo en lugar de solo toques individuales. La variable `threshold` se usa para evitar que se registren movimientos demasiado pequeños o ruido del sensor táctil, asegurando que solo se envíen datos relevantes al servidor. Esto ayuda a que la interacción sea más estable y fluida, y evita saturar la comunicación con información innecesaria.

### Compara brevemente Dev Tunnels con simplemente usar la IP local. ¿Cuáles son las ventajas y desventajas de cada uno?

Usar la IP local es más sencillo y rápido si todos los dispositivos están en la misma red Wi-Fi, pero falla si quiero acceder desde fuera de la red local o si el router bloquea conexiones. Dev Tunnels funciona desde cualquier red y facilita pruebas remotas sin configurar routers ni firewalls, aunque puede ser un poco más lento y depende de un servicio externo para mantener el túnel activo. En resumen, la IP local es directo y rápido dentro de la misma red; Dev Tunnels es más flexible y seguro para pruebas externas.

### Coloca en tu bitácora capturas de pantalla del sistema completo funcionando. Esto lo puedes hacer abriendo tanto el mobile como el desktop en tu computador y tomando una captura de pantalla de todos los involucrados (celular, computador y terminal).

<img width="315" height="457" alt="image" src="https://github.com/user-attachments/assets/8acac7c2-acbb-4d1e-b6b9-878dc5bc65b8" />
<img width="383" height="134" alt="image" src="https://github.com/user-attachments/assets/a5043489-667b-4033-a3e7-a8187b8f5243" />
<img width="398" height="329" alt="image" src="https://github.com/user-attachments/assets/837916e2-eb4a-406d-8c80-980d782946bf" />
<img width="398" height="900" alt="image" src="https://github.com/user-attachments/assets/33416339-0f94-479f-83d0-ea8aac9ac591" />


## Actividad 03:

### ¿Cuál es la función principal de express.static('public') en este servidor? ¿Cómo se compara con el uso de app.get('/ruta', …) del servidor de la Unidad 6?

Básicamente `express.static('public')` sirve para que todos los archivos que estén en la carpeta `public` se puedan usar directamente desde el navegador, como HTML, CSS, JS, imágenes, todo eso. Entonces, no tienes que estar creando una ruta específica para cada archivo.  

En cambio, `app.get('/ruta', …)` es más manual, porque tienes que decir exactamente qué ruta sirve qué cosa. Es más flexible si quieres respuestas dinámicas, pero para archivos estáticos pues `express.static` es mucho más práctico y rápido.

### Explica detalladamente el flujo de un mensaje táctil:

Cuando el usuario mueve el dedo en el celular:  
1. **Evento que lo envía desde el celular:** Dentro del cliente, se hace un `socket.emit('message', mensaje)` normalmente en la función `touchMoved()`, entonces cada movimiento se envía al servidor.  
2. **Evento que lo recibe el servidor:** El servidor escucha con `socket.on('message', callback)` y ahí recibe el mensaje que envía el celular.  
3. **Qué hace el servidor con él:** El servidor primero lo imprime en consola con `console.log()` para ver qué está pasando y luego lo manda a los demás clientes conectados.  
4. **Evento que lo envía el servidor al escritorio:** Se usa `socket.broadcast.emit('message', mensaje)` para que todos los clientes excepto el que envió el mensaje lo reciban.  
5. **Por qué se usa socket.broadcast.emit en vez de io.emit o socket.emit:**  
   - `socket.emit` solo lo mandaría de vuelta al celular que lo envió, y pues no sirve para sincronizar con los otros.  
   - `io.emit` lo mandaría a todos, incluido el que lo envió, entonces se duplicaría.  
   - `socket.broadcast.emit` es como perfecto aquí porque manda el mensaje solo a los demás clientes, entonces los escritorios reciben el movimiento pero el celular no se confunde con su propio mensaje.

---

### Si conectaras dos computadores de escritorio y un celular a este servidor, y movieras el dedo en el celular, ¿Quién recibiría el mensaje retransmitido por el servidor? ¿Por qué?

Pues, los dos computadores de escritorio recibirían el mensaje, porque el servidor usa `socket.broadcast.emit`, que manda el mensaje a todos los clientes conectados excepto al que lo originó. Entonces, el celular que lo envió no recibe nada de vuelta, y los escritorios sí.

---

### ¿Qué información útil te proporcionan los mensajes console.log en el servidor durante la ejecución?

Los `console.log` son útiles porque te muestran, como, cuándo un cliente se conecta (`New client connected`), qué mensajes llegan (`Received message => ...`) y cuándo un cliente se desconecta (`Client disconnected`). Entonces, básicamente te ayudan a ver si todo está funcionando bien y a depurar problemas de conexión o de sincronización.

## Actividad 04:

### Realiza un diagrama donde muestres el flujo completo de datos y eventos entre los tres componentes: móvil, servidor y escritorio. Puedes ilustrar con un ejemplo de coordenadas táctiles (x, y) y cómo viajan a través del sistema.

<img width="789" height="888" alt="image" src="https://github.com/user-attachments/assets/19315fa8-610c-4752-bed2-deeee51e18e3" />

Primero el móvil y el escritorio se conectan al servidor con socket.connect(), y el servidor les devuelve la confirmación de que todo está bien, y entonces en el móvil cuando uno mueve el dedo o el mouse se ejecuta touchMoved(), que calcula la diferencia con la última posición (dx y dy) y si el movimiento es mayor que el threshold o si es el primer toque, entonces manda los datos al servidor con socket.emit('message', {x, y}). El servidor recibe ese mensaje y hace un broadcast, entonces el escritorio recibe ese mensaje con socket.on('message'), actualiza circleX y circleY y dibuja el círculo rojo en su canvas, y al mismo tiempo el móvil dibuja su canvas con las instrucciones. Entonces, todo esto muestra cómo viajan los datos de las coordenadas táctiles desde el móvil hasta el escritorio y cómo se actualiza la interfaz de cada uno, y también está reflejado en el diagrama cómo se manejan las desconexiones y los errores


## Actividad 05:

### Diseña una aplicación interactiva que use el touch del móvil para controlar una visuales de tema musical de tu elección. Las visuales correrán en una aplicación de escritorio (desktop). Recuerda que ambas aplicaciones las construirás usando p5.js y utilizando el servidor Node.js como puente.

Quería diseñar basado en la canción *“Watching Him Fade Away”* de **Mac DeMarco**, una canción lenta y medio triste.  

[watching_him_fade_away.mp3](https://github.com/user-attachments/files/22977796/watching_him_fade_away.mp3)

> “Mac juega al tira y afloja entre llamar a su padre, aparentemente para despedirse, o dejarlo ‘desvanecerse’ sin darle un cierre: la venganza máxima por un padre ausente.”

Mi idea fue convertir esa contradicción emocional en una experiencia interactiva, donde el usuario también tenga que hacer cosas para dejarlo ir y luego para mantenerlo ahí.

---

### Fase 1 – Verlo desaparecer

Desde la primera estrofa hasta el final del primer coro, quería reflejar la relación distante del artista con su padre.  
En la canción habla de como realmente nunca lo conoció, ya que lo abandonó al ser un niño, y que aún guarda el rencor. Aunque su padre esté enfermo una parte de el se preocupa y a la vez piensa que si no lo llama para despedirse va a quedar con el arrepentimiento de nunca decirle lo que pensaba de el a la cara.
En esta parte aparece una **ilustración del padre y el hijo**, y el usuario puede **arrastrar el dedo o el cursor para “tachar” al padre**, cubriéndolo con trazos oscuros.  

<img width="1920" height="1080" alt="padreHijo" src="https://github.com/user-attachments/assets/56604ce0-d750-471a-9aa4-8e26d05c3a4e" />


Los trazos crecen con cada golpe de la canción y se van haciendo más gruesos a medida que se acerca el cambio de fase, como si el rencor que le tiene se volviera más dificil de controlar. Hasta que llega el punto donde los trazos se mezclan con la ilustración (por eso es como un lineart) y el padre desaparece visualmente.

---

### Fase 2 – Intentar que no se borre

En la segunda parte, desde la siguiente estrofa hasta el final, todo se invierte.  
Ahora la ilustración se vuelve clara, y en lugar de trazar, el usuario *intenta borrar los puntos que aparecen* sobre la figura del padre, que representan como la enfermedad que empieza a taparlo en vez de los tachones de antes.  
Es un intento desesperado por mantenerlo visible, pero conforme la canción avanza, los puntos comienzan a multiplicarse hasta llenar la pantalla.

Al final, no importa cuánto se intente borrar, la figura termina "desapareciendo" parcialmente bajo los puntos. Representando como este termina desvaneciendose por su enfermedad sin que realmente se pueda hacer algo para evitarlo.

<img width="1920" height="1080" alt="whfa" src="https://github.com/user-attachments/assets/70570d68-d43b-4f29-a218-726d989ece92" />


### Incluye todos los códigos (servidor y clientes) en tu bitácora.

#### desktop

*sketch.js*
```js
let socket;
let shapes = [];
let currentShape = null;
let img, imgWhite;
let song;
let circleX = 0;
let circleY = 0;
let stage = 1;
let circleSize = 20;
let papaZone;
let papaZonePhase2;
let bpm = 139;
let beatInterval = 60 / bpm;
let lastBeatTime = 0;
let fft;
let blackFade = 0;
let spots = [];
let spotSpawnRate = 0;
let eraseRadius = 80;
let showCursor = true;
let canErase = true;

function preload() {
  img = loadImage('../assets/padreHijo.png');
  imgWhite = loadImage('../assets/whfa.png');
  song = loadSound('../assets/watching_him_fade_away.mp3');
}

function setup() {
  createCanvas(windowWidth, windowHeight);
  imageMode(CENTER);
  colorMode(RGB, 255);

  // Zonas
  papaZone = { x: width * 0.31, y: 0, width: width * 0.25, height: height };
  papaZonePhase2 = { x: width * 0.34, y: 0, width: width * 0.18, height: height };

  socket = io('http://localhost:3000');
  socket.on('message', data => {
    if (data && data.type === 'touch') {
      let x = data.x * (width / 300);
      let y = data.y * (height / 400);

      if (stage === 1) {
        if (
          x >= papaZone.x &&
          x <= papaZone.x + papaZone.width &&
          y >= papaZone.y &&
          y <= papaZone.y + papaZone.height
        ) {
          if (!currentShape) currentShape = new Shape();
          currentShape.addPoint(x, y);
          circleX = x;
          circleY = y;
        }
      } else if (stage === 2 && canErase) {
        eraseSpots(x, y);
        circleX = x;
        circleY = y;
      }
    }
  });

  select('canvas').mousePressed(() => {
    if (!song.isPlaying()) {
      song.play();
      song.setVolume(0.7);
      lastBeatTime = song.currentTime();
    }
  });

  fft = new p5.FFT();
  fft.setInput(song);
}

function draw() {
  if (stage === 1) drawPhase1();
  else if (stage === 2) drawPhase2();
}

// ---------------------------------------------------
// FASE 1
// ---------------------------------------------------
function drawPhase1() {
  background(220);
  image(img, width / 2, height / 2, width, height);
  drawDynamicBackground();

  if (song.isPlaying()) {
    let currentTime = song.currentTime();

    if (currentTime - lastBeatTime >= beatInterval) {
      lastBeatTime += beatInterval;
      if (currentShape) currentShape.onBeat();
      shapes.forEach(s => s.onBeat());
    }

    if (currentTime >= 60) shapes.forEach(s => s.expandRoots());

    if (currentTime >= 75 && stage === 1) {
      stage = 2;
      shapes = [];
      currentShape = null;
      blackFade = 0;
      circleSize = 60;
      spots = [];
    }
  }

  if (song.isPlaying()) {
    let currentTime = song.currentTime();
    let progress = constrain(map(currentTime, 0, 75, 0, 1), 0, 1); // progreso de la fase 1

    let waveform = fft.waveform();
    let avg = waveform.reduce((a, b) => a + abs(b), 0) / waveform.length;
    let pulse = map(avg, 0, 0.2, 1, 1.02);
    push();
    translate(width / 2, height / 2);
    scale(pulse);
    translate(-width / 2, -height / 2);

    // pasamos el progreso a cada shape
    shapes.forEach(s => {
      s.phaseProgress = progress;
      s.updateAndDraw();
    });
    if (currentShape) {
      currentShape.phaseProgress = progress;
      currentShape.updateAndDraw();
    }
    pop();
  } else {
    shapes.forEach(s => s.updateAndDraw());
    if (currentShape) currentShape.updateAndDraw();
  }

  drawCircleFollower();

  if (song.isPlaying() && song.currentTime() >= 70) {
    blackFade = min(blackFade + 3, 255);
    fill(0, 0, 0, blackFade);
    rect(0, 0, width, height);
  }
}

// ---------------------------------------------------
// FASE 2
// ---------------------------------------------------
function drawPhase2() {
  background(0);
  drawDynamicBackground();
  image(imgWhite, width / 2, height / 2, width, height);

  if (song.isPlaying()) {
    let t = song.currentTime();
    let total = song.duration();
    let remaining = total - t;
    let progress = constrain(map(t, 75, total, 0, 1), 0, 1);

    if (remaining > 15) {
      spotSpawnRate = map(progress, 0, 0.6, 0.1, 0.6);
    } else if (remaining > 8) {
      spotSpawnRate = map(remaining, 15, 8, 0.6, 1.5);
    } else if (remaining > 3) {
      spotSpawnRate = map(remaining, 8, 3, 1.5, 3.5);
    } else {
      spotSpawnRate = 6;
    }

    if (remaining <= 5) canErase = false;

    for (let i = 0; i < spotSpawnRate * 3; i++) {
      if (random() < 0.9) {
        let x, y;
        if (remaining > 10) {
          x = random(papaZonePhase2.x, papaZonePhase2.x + papaZonePhase2.width);
          y = random(height);
        } else {
          x = random(width);
          y = random(height);
        }

        let size = random(25, 90);
        let gray = random(200, 255);
        spots.push({
          x,
          y,
          size,
          c: color(gray, gray, gray),
          alpha: random(80, 100)
        });
      }
    }
  }

  noStroke();
  for (let s of spots) {
    fill(red(s.c), green(s.c), blue(s.c), s.alpha);
    ellipse(s.x, s.y, s.size);
  }

  if (song.isPlaying() && song.currentTime() >= song.duration() - 3) {
    fill(230, 230, 230, 10);
    rect(0, 0, width, height);
  }

  if (showCursor && canErase) {
    noFill();
    stroke(255, 255, 255, 70);
    strokeWeight(2);
    ellipse(circleX, circleY, eraseRadius * 0.7);
  }
}

function eraseSpots(x, y) {
  spots = spots.filter(s => dist(x, y, s.x, s.y) > s.size / 2 + eraseRadius / 2);
}

function drawCircleFollower() {
  circleX = lerp(circleX, currentShape?.points.slice(-1)[0]?.x || circleX, 0.2);
  circleY = lerp(circleY, currentShape?.points.slice(-1)[0]?.y || circleY, 0.2);
  fill(255, 0, 0, 180);
  noStroke();
  ellipse(circleX, circleY, circleSize, circleSize);
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
  papaZone = { x: width * 0.31, y: 0, width: width * 0.25, height: height };
  papaZonePhase2 = { x: width * 0.34, y: 0, width: width * 0.18, height: height };
}

// ---------------------------------------------------
// Shape con raíces y ecos
// ---------------------------------------------------
function Shape() {
  this.points = [];
  this.pendingPoints = [];
  this.strokeW = 1.6;
  this.jitter = 1;
  this.echoes = [];
  this.roots = [];
  this.phaseProgress = 0;

  this.addPoint = function (x, y) {
    this.pendingPoints.push(createVector(x, y));
  };

  this.update = function () {
    if (this.pendingPoints.length > 0) {
      let newPoint = this.pendingPoints.shift();
      this.points.push(newPoint);
      this.echoes.push({
        pos: newPoint.copy(),
        alpha: 80,
        size: random(4, 10)
      });
      if (newPoint.x >= papaZone.x && newPoint.x <= papaZone.x + papaZone.width) {
        this.roots.push({
          origin: newPoint.copy(),
          angle: random(TWO_PI),
          length: 0,
          maxLength: random(20, 80)
        });
      }
    }

    for (let e of this.echoes) {
      e.pos.add(p5.Vector.random2D().mult(0.3));
      e.alpha -= 1;
    }
    this.echoes = this.echoes.filter(e => e.alpha > 0);

    for (let r of this.roots) {
      if (r.length < r.maxLength) r.length += 0.4;
    }
  };

  this.onBeat = function () {
    this.jitter = min(this.jitter + 0.15, 3);
  };

  this.expandRoots = function () {
    for (let r of this.roots) {
      r.maxLength += 2;
      r.length = min(r.length + 1.5, r.maxLength);
    }
  };

  this.updateAndDraw = function () {
    this.update();
    this.draw();
  };

  this.draw = function () {
    noStroke();
    for (let e of this.echoes) {
      fill(0, 0, 30, e.alpha);
      ellipse(e.pos.x, e.pos.y, e.size);
    }

    if (this.points.length > 1) {
      noFill();
      beginShape();
      for (let p of this.points) {
        let inPapaZone = p.x >= papaZone.x && p.x <= papaZone.x + papaZone.width;

        // Grosor dinámico según el avance de la fase 1
        let phaseFactor = this.phaseProgress ? pow(this.phaseProgress, 2.5) : 0;
        let dynamicWeight = this.strokeW + map(phaseFactor, 0, 1, 0, 3);

        if (inPapaZone) {
          stroke(0, 0, 0);
          strokeWeight(dynamicWeight * 1.3);
        } else {
          stroke(120, 50, 40, 40);
          strokeWeight(dynamicWeight);
        }

        vertex(
          p.x + random(-this.jitter, this.jitter),
          p.y + random(-this.jitter, this.jitter)
        );
      }
      endShape();
    }

    stroke(0, 0, 0, 60);
    strokeWeight(1);
    for (let r of this.roots) {
      let endX = r.origin.x + cos(r.angle) * r.length;
      let endY = r.origin.y + sin(r.angle) * r.length;
      line(r.origin.x, r.origin.y, endX, endY);
      if (r.length > r.maxLength * 0.7 && random() < 0.1) {
        let branchAngle = r.angle + random(-PI / 6, PI / 6);
        let branchLen = random(10, 30);
        line(endX, endY, endX + cos(branchAngle) * branchLen, endY + sin(branchAngle) * branchLen);
      }
    }
  };
}

// ---------------------------------------------------
// Fondo FFT con verde pastel dinámico
// ---------------------------------------------------
function drawDynamicBackground() {
  if (!song.isPlaying()) return;
  let spectrum = fft.analyze();
  noStroke();

  for (let i = 0; i < spectrum.length; i += 10) {
    let amp = spectrum[i];
    let x = map(i, 0, spectrum.length, 0, width);
    let y = height / 2 + map(amp, 0, 255, -120, 120);
    let size = map(amp, 0, 255, 8, 70);

    let baseG = map(amp, 0, 255, 180, 240);
    let baseR = map(amp, 0, 255, 150, 210);
    let baseB = map(amp, 0, 255, 150, 190);
    fill(baseR, baseG, baseB, 35);

    ellipse(x, y, size, size);
  }
}

```

*index.html*
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/p5.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.6.0/addons/p5.sound.min.js"></script>
    <script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
    <!-- Si necesitas dependencias del original, descomenta: -->
    <script src="https://cdn.jsdelivr.net/gh/generative-design/Code-Package-p5.js@master/libraries/gg-dep-bundle/gg-dep-bundle.js"></script>
    <script src="sketch.js"></script>  <!-- Tu sketch.js actualizado -->
    <title>Desktop p5.js Application</title>
</head>
<body></body>
</html>
```

#### mobile

*sketch.js*
```js
let socket;
let lastTouchX = null; 
let lastTouchY = null; 
const threshold = 5;

function setup() {
    createCanvas(300, 400);
    background(220);
    socket = io();
}

function draw() {
    background(220);
    fill(0);
    textAlign(CENTER, CENTER);
    textSize(16);
    text('Arrastra sobre el papá', width / 2, height / 2);
}

function touchMoved() {
    if (socket && socket.connected) { 
        let dx = abs(mouseX - lastTouchX);
        let dy = abs(mouseY - lastTouchY);

        if (dx > threshold || dy > threshold || lastTouchX === null) {
            // Normalizamos coords entre 0 y 1 para seguridad
            let normX = constrain(mouseX, 0, width);
            let normY = constrain(mouseY, 0, height);

            let touchData = {
                type: 'touch',
                x: normX,
                y: normY
            };
            socket.emit('message', touchData);

            lastTouchX = normX;
            lastTouchY = normY;
        }
    }
    return false;
}

```

*index.html*
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/p5@1.11.0/lib/p5.min.js"></script>
    <script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
    <script src="sketch.js"></script>
    <title>Mobile p5.js Application</title>
</head>
<body></body>
</html>

```

#### server
```js
const express = require('express');
const http = require('http');
const socketIO = require('socket.io');

const app = express();
const server = http.createServer(app); 
const io = socketIO(server); 
const port = 3000;

app.use(express.static('public'));

io.on('connection', (socket) => {
    console.log('New client connected');
    socket.on('message', (message) => {
        console.log('Received message =>', message);
        socket.broadcast.emit('message', message);
    });

    socket.on('disconnect', () => {
        console.log('Client disconnected');
    });
});

server.listen(port, () => {
    console.log(`Server is listening on http://localhost:${port}`);
});
```

## Autoevaluación:

| Actividad | Estado    | Justificación |
|-----------|----------|---------------|
| Actividad 01 | Completo | Leí los recursos sobre los dev tunnels para entender su funcionamiento de forma más clara, aprendí varios detalles nuevos, realicé la práctica y logré acceder a la aplicación desde mi celular, revisando los mensajes de la consola y comprobando que todo funcionara. Además respondí todas las preguntas, reflexionando sobre cada paso. |
| Actividad 02 | Completo | Respondí todas las preguntas, reflexioné sobre la importancia de los dev tunnels, analizando sus ventajas y comparándolos con los IPS. También completé la actividad práctica y presenté las capturas de evidencia correspondientes. |
| Actividad 03 | Completo | Respondí todas las preguntas, analicé detalladamente el flujo del programa y las funciones del servidor, entendiendo cómo se conectan los distintos componentes y cómo se manejan los datos. |
| Actividad 04 | Completo | Realicé y revisé el diagrama siguiendo las indicaciones del código, pude ver claramente las interacciones entre los distintos actores en los diferentes momentos del programa y además expliqué brevemente el diagrama para reforzar mi comprensión. |
| Actividad 05 | Completo | Creé un visualizador interactivo usando touch y apoyándome en IA generativa para una canción, diseñando la idea, los dibujos y los momentos clave a implementar. Lo probé en p5.js y finalmente verifiqué que todo funcionara correctamente, asegurándome de que la experiencia fuera coherente y completa. |
