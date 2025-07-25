# Unidad 1

## 🔎 Fase: Set + Seek

### Actividad 01:
---

#### ¿Qué es un sistema físico interactivo?

Un sistema físico interactivo (SPI) es una combinación de hardware y software diseñada para reaccionar en tiempo real a estímulos del entorno o del usuario. Estos sistemas integran sensores, actuadores, microcontroladores y programación para crear experiencias donde la interacción física, como el movimiento, el tacto o la proximidad, forma parte esencial del funcionamiento. Son comunes en instalaciones artísticas, videojuegos físicos, dispositivos vestibles (wearables) y productos que buscan una relación más directa y sensorial con el usuario.

#### ¿Cómo podrías aplicar lo que has visto en tu perfil profesional?

Dado a que mi interés es un enfoque en diseño de productos y experiencias gamificadas, los sistemas físicos interactivos pueden ampliar la forma en la que el usuario es incluido en el proceso del juego, explorando distintos inputs para crear experiencias mucho más memorables y personalizadas.

Como estudiante de Ingeniería en Diseño de Entretenimiento Digital, me interesa generar productos que conecten lo tecnológico con lo emocional y lo cultural. Integrar SPI en mi perfil profesional me permite llevar las ideas más allá de la pantalla y diseñar experiencias físicas que respondan al cuerpo y a las acciones del usuario.

Puedo imaginar su aplicación en prototipos de videojuegos físicos o instalaciones interactivas donde se utilicen sensores de movimiento, botones físicos, superficies táctiles o incluso sensores de proximidad, con el fin de crear una retroalimentación más inmersiva. Este tipo de interacción hace que la experiencia sea mucho más significativa, adaptada y dinámica, lo cual es esencial en proyectos enfocados en el entretenimiento y la participación activa del usuario.


### Actividad 02:
---

#### ¿Qué es el diseño/arte generativo?

El *arte generativo* es una práctica artística donde se generan obras total o parcialmente a partir de un sistema autónomo, como algoritmos y procesos, guiados o intervenidos por un artista que parametriza y regula el procedimiento.

El *diseño generativo* es un proceso cíclico de diseño donde se utilizan sistemas autónomos para crear diferentes alternativas o soluciones, todas definidas por parámetros y restricciones del diseñador. Estos sistemas permiten generar una gran variedad de resultados, manteniendo un equilibrio entre control y aleatoriedad.

#### ¿Cómo podrías aplicar lo que has visto en tu perfil profesional?

El arte y diseño generativo me permite explorar nuevas formas de expresión visual y de creación de contenido. Este enfoque se alinea con mi interés por construir experiencias interactivas, gamificadas y centradas en el usuario.

Puedo aplicar el diseño generativo para desarrollar gráficos dinámicos, visualizaciones únicas o incluso niveles de juego que cambian con base en parámetros definidos por el jugador o el sistema. Además, me permite integrar procesos creativos que mezclan lo técnico con lo artístico, automatizando ciertas partes del flujo de trabajo sin perder el control creativo.

También puedo implementarlo en la generación procedural de contenido dentro de videojuegos, lo cual optimiza el desarrollo y permite ofrecer experiencias más variadas y personalizadas. Esta metodología es especialmente útil en la fase de prototipado, ya que facilita la creación rápida de múltiples alternativas visuales y estructurales, fortaleciendo la exploración creativa y técnica en mis proyectos como diseñadora de entretenimiento digital.


### Actividad 03:
---

#### En el sistemas físico interactivo identifica los inputs, outputs y el proceso.

**Inputs:**
- Los botones A y B del micro:bit.
- El gesto de sacudir el micro:bit (llevaba a C).
- El botón “Send Love” que está en la interfaz de p5.js.

**Outputs:**
- Cambios en el color del círculo en la interfaz hecha con p5.js (puede ponerse rojo, amarillo o verde según el input recibido).
- Imágenes en la pantalla LED del micro:bit, como una mariposa al inicio y luego un corazón seguido de una carita feliz cuando se recibe la letra 'h'.

**Proceso:**
Primero, el micro:bit detecta alguna interacción física, ya sea que se presione un botón o se sacuda el dispositivo. Cuando eso pasa, el micro:bit envía un mensaje por puerto serial (‘A’, ‘B’ o ‘C’) al computador. Luego, desde p5.js se recibe ese dato y se interpreta para cambiar el color del círculo en pantalla dependiendo de la letra recibida.

Además, si desde p5.js el usuario presiona el botón “Send Love”, se envía el carácter ‘h’ al micro:bit. Este lo recibe, interpreta ese mensaje y responde mostrando primero un corazón y luego una carita feliz en su pantalla LED.


### Actividad 04:
---

Quería hacer la actividad de forma mas creativa, en clase habia empezado a hacer un gato entonces quería completarlo y hacer que el mouse moviera las pupilas. Para que las pupilas se movieran con el mouse, usé map() para convertir la posición del mouse (que va de 0 a 400) a un rango más pequeño, de -12 a 12. Así controlo cuánto se mueve la pupila sin que se salga de cada ojo. Luego usé constrain() para asegurarme de que esos valores no se pasaran del límite. Todo eso se guarda en dx y dy, que básicamente son la distancia horizontal y vertical que debe moverse cada pupila desde su punto original.

#### Enlace:
[Pantalla completa](https://editor.p5js.org/estefaao2006/full/mpQ9UEhoW)

[Editable](https://editor.p5js.org/estefaao2006/sketches/mpQ9UEhoW)


#### Código:

  ``` js
    function setup() {
  createCanvas(400, 400); // el espacio donde va a aparecer el dibujo, tamaño 400x400
  noStroke(); // para que las figuras no tengan borde
}

function draw() {
  background(255); // fondo blanco

  // cabeza del gato
  fill(240, 190, 140); 
  ellipse(200, 200, 220, 200); // la cara

  // orejas
  fill(240, 190, 140); 
  triangle(130, 120, 160, 60, 170, 130);
  triangle(230, 130, 260, 60, 270, 120);

  // parte rosada de las orejas
  fill(255, 180, 200);
  triangle(140, 115, 160, 75, 165, 120);
  triangle(240, 115, 260, 75, 255, 120);

  // ojos
  fill(255);
  ellipse(160, 200, 50, 50); // ojo izquierdo
  ellipse(240, 200, 50, 50); // ojo derecho

  // las pupilas se mueven con el mouse

  let eyeOffset = 12; // este número define qué tanto pueden moverse las pupilas desde el centro del ojo. Teniendo en cuenta el tamaño del circulo, probando diferentes valores elegí finalmente 12.

  // acá saco cuánto se debe mover la pupila en X
  let dx = map(mouseX, 0, width, -eyeOffset, eyeOffset);
  // map() toma el valor de mouseX (que va de 0 a 400, porque el canvas mide eso) 
  // y lo convierte a un rango más pequeño, o sea, de -12 a 12, para que la pupila no se salga del ojo

  // constrain para asegurarme de que ese valor se quede dentro del rango
  dx = constrain(dx, -eyeOffset, eyeOffset);

  // lo mismo con Y, pero con el movimiento vertical
  let dy = map(mouseY, 0, height, -eyeOffset, eyeOffset);
  dy = constrain(dy, -eyeOffset, eyeOffset);

  // se crean las pupilas con los desplazamientos, que dependen de la posición del mouse
  fill(0); // color negro para las pupilas
  ellipse(160 + dx, 200 + dy, 20, 20); // pupila izquierda
  ellipse(240 + dx, 200 + dy, 20, 20); // pupila derecha

  // nariz
  fill(255, 100, 120);
  triangle(195, 215, 205, 215, 200, 225);

  // bigotes
  stroke(100);
  strokeWeight(2);
  line(130, 220, 90, 215);
  line(130, 230, 90, 230);
  line(130, 240, 90, 245);
  line(270, 220, 310, 215);
  line(270, 230, 310, 230);
  line(270, 240, 310, 245);
  noStroke();

  // boca (dos arquitos)
  stroke(100);
  noFill();
  arc(195, 235, 10, 10, 0, PI);
  arc(205, 235, 10, 10, 0, PI);
  noStroke();
}

  ```

#### Captura de pantalla:

<img width="402" height="425" alt="image" src="https://github.com/user-attachments/assets/eb66a43f-0321-4653-ac31-75dc9d35c937" />



### Actividad 05:
---

En esta actividad comenzamos a construir un sistema físico interactivo básico usando un micro:bit como dispositivo de entrada. La idea es que al presionar el botón A del micro:bit, se genere una señal que pueda ser leída por otro software, como p5.js, para producir una reacción visual en pantalla.

Primero, importamos todas las funciones necesarias con from microbit import *, lo que nos permite acceder a los botones, al sistema de comunicación serial y a los sensores del micro:bit.

Luego inicializamos la comunicación serial con uart.init(baudrate=115200). Esta línea es muy importante porque define la velocidad a la que se envían los datos desde el micro:bit. Elegimos 115200 baudios, que es una velocidad estándar para este tipo de comunicación y es compatible con muchas plataformas.

Después, creamos un bucle while True: que se repite constantemente (esto es típico en microcontroladores, donde no hay un "final" como tal, sino que siempre están escuchando o esperando una acción).

Dentro del bucle, usamos if button_a.was_pressed(): para verificar si el botón A fue presionado. Elegimos was_pressed() en vez de is_pressed() porque queremos detectar solo el momento en que se presiona, no si el botón sigue presionado (ya que eso podría enviar muchos mensajes seguidos y crear errores en la lectura del otro lado).

Cuando se detecta que el botón fue presionado, usamos uart.write('A') para enviar un carácter (en este caso la letra A) por el puerto serial. Este mensaje será recibido más adelante por el programa hecho en p5.js y servirá como señal para generar un cambio en la pantalla
