
# Evidencias de la unidad 6

Caso de estudio:

page1.html
~~~html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Page 1</title>
    <script src="https://cdn.jsdelivr.net/npm/p5@1.11.0/lib/p5.min.js"></script>
    <script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
    <style>
        body {
            margin: 0;
            overflow: hidden;
        }
    </style>
</head>
<body>
    <!-- <h1>Welcome to Page 1</h1> -->
    <script defer src="page1.js"></script>
</body>
</html>
~~~

page1.js
~~~js
let currentPageData = {
    x: window.screenX,
    y: window.screenY,
    width: window.innerWidth,
    height: window.innerHeight
}

let previousPageData = {
    x: window.screenX,
    y: window.screenY,
    width: window.innerWidth,
    height: window.innerHeight
};

let remotePageData = { x: 0, y: 0, width: 100, height: 100 };
let point1 = [currentPageData.width / 2, currentPageData.height / 2];
let socket;
let isConnected = false;
let hasRemoteData = false;
let isFullySynced = false;
let connectionTimeout;

function setup() {
    createCanvas(windowWidth, windowHeight);
    frameRate(60);
    socket = io();

    socket.on('connect', () => {
        console.log('Connected with ID:', socket.id);
        isConnected = true;
        socket.emit('win1update', currentPageData, socket.id);
        
        // Solicitar sincronización después de un breve delay
        setTimeout(() => {
            socket.emit('requestSync');
        }, 500);
    });

    socket.on('getdata', (response) => {
        if (response && response.data && isValidRemoteData(response.data)) {
            remotePageData = response.data;
            hasRemoteData = true;
            console.log('Received valid remote data:', remotePageData);
            socket.emit('confirmSync');
        }
    });

    socket.on('fullySynced', (synced) => {
        isFullySynced = synced;
        console.log('Sync status:', synced ? 'SYNCED' : 'NOT SYNCED');
    });

    socket.on('peerDisconnected', () => {
        hasRemoteData = false;
        isFullySynced = false;
        console.log('Peer disconnected, waiting for reconnection...');
    });

    socket.on('disconnect', () => {
        isConnected = false;
        hasRemoteData = false;
        isFullySynced = false;
        console.log('Disconnected from server');
    });
}

function isValidRemoteData(data) {
    return data && 
           typeof data.x === 'number' && 
           typeof data.y === 'number' && 
           typeof data.width === 'number' && data.width > 0 &&
           typeof data.height === 'number' && data.height > 0;
}

function checkWindowPosition() {
    currentPageData = {
        x: window.screenX,
        y: window.screenY,
        width: window.innerWidth,
        height: window.innerHeight
    };

    if (currentPageData.x !== previousPageData.x || currentPageData.y !== previousPageData.y || 
        currentPageData.width !== previousPageData.width || currentPageData.height !== previousPageData.height) {

        point1 = [currentPageData.width / 2, currentPageData.height / 2]
        socket.emit('win1update', currentPageData, socket.id);
        previousPageData = currentPageData;
    }
}


function draw() {
    background(220);
    
    if (!isConnected) {
        showStatus('Conectando al servidor...', color(255, 165, 0));
        return;
    }
    
    if (!hasRemoteData) {
        showStatus('Esperando conexión de la otra ventana...', color(255, 165, 0));
        return;
    }
    
    if (!isFullySynced) {
        showStatus('Sincronizando datos...', color(255, 165, 0));
        return;
    }

    // Solo dibujar cuando esté completamente sincronizado
    drawCircle(point1[0], point1[1]);
    checkWindowPosition();
    
    let vector1 = createVector(currentPageData.x, currentPageData.y);
    let vector2 = createVector(remotePageData.x, remotePageData.y);
    let resultingVector = createVector(vector2.x - vector1.x, vector2.y - vector1.y);
    
    stroke(50);
    strokeWeight(20);
    drawCircle(resultingVector.x + remotePageData.width / 2, resultingVector.y + remotePageData.height / 2);
    line(point1[0], point1[1], resultingVector.x + remotePageData.width / 2, resultingVector.y + remotePageData.height / 2);
}

function showStatus(message, statusColor) {
    textSize(24);
    textAlign(CENTER, CENTER);
    noStroke();
    // Dibujar rectángulo de fondo para el texto
    fill(0, 0, 0, 150); // Negro semi-transparente
    rectMode(CENTER);
    let textW = textWidth(message) + 40;
    let textH = 40;
    rect(width / 2, 1*height / 6, textW, textH, 10);
    // Dibujar el texto
    fill(statusColor);
    text(message, width / 2, 1*height / 6);
}

function drawCircle(x, y) {
    fill(255, 0, 0);
    ellipse(x, y, 150, 150);
}

function windowResized() {
    resizeCanvas(windowWidth, windowHeight);
}

~~~

page2.html
~~~html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Page 2</title>
    <script src="https://cdn.jsdelivr.net/npm/p5@1.11.0/lib/p5.min.js"></script>
    <script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
    <style>
        body {
            margin: 0;
            overflow: hidden;
        }
    </style>
</head>

<body>
    <!-- <h1>Welcome to Page 2</h1> -->
    <script defer src="page2.js"></script>
</body>
</html>
~~~

page2.js
~~~js
let currentPageData = {
    x: window.screenX,
    y: window.screenY,
    width: window.innerWidth,
    height: window.innerHeight

}

let previousPageData = {
    x: window.screenX,
    y: window.screenY,
    width: window.innerWidth,
    height: window.innerHeight
};

let remotePageData = { x: 0, y: 0, width: 100, height: 100 };
let point2 = [currentPageData.width / 2, currentPageData.height / 2];
let socket;
let isConnected = false;
let hasRemoteData = false;
let isFullySynced = false;
let connectionTimeout;

function setup() {
    createCanvas(windowWidth, windowHeight);
    frameRate(60);
    socket = io();

    socket.on('connect', () => {
        console.log('Connected with ID:', socket.id);
        isConnected = true;
        socket.emit('win2update', currentPageData, socket.id);
        
        // Solicitar sincronización después de un breve delay
        setTimeout(() => {
            socket.emit('requestSync');
        }, 500);
    });

    socket.on('getdata', (response) => {
        if (response && response.data && isValidRemoteData(response.data)) {
            remotePageData = response.data;
            hasRemoteData = true;
            console.log('Received valid remote data:', remotePageData);
            socket.emit('confirmSync');
        }
    });

    socket.on('fullySynced', (synced) => {
        isFullySynced = synced;
        console.log('Sync status:', synced ? 'SYNCED' : 'NOT SYNCED');
    });

    socket.on('peerDisconnected', () => {
        hasRemoteData = false;
        isFullySynced = false;
        console.log('Peer disconnected, waiting for reconnection...');
    });

    socket.on('disconnect', () => {
        isConnected = false;
        hasRemoteData = false;
        isFullySynced = false;
        console.log('Disconnected from server');
    });
}

function isValidRemoteData(data) {
    return data && 
           typeof data.x === 'number' && 
           typeof data.y === 'number' && 
           typeof data.width === 'number' && data.width > 0 &&
           typeof data.height === 'number' && data.height > 0;
}

function checkWindowPosition() {
    currentPageData = {
        x: window.screenX,
        y: window.screenY,
        width: window.innerWidth,
        height: window.innerHeight
    };

    if (currentPageData.x !== previousPageData.x || currentPageData.y !== previousPageData.y || 
        currentPageData.width !== previousPageData.width || currentPageData.height !== previousPageData.height) {

        point2 = [currentPageData.width / 2, currentPageData.height / 2]
        socket.emit('win2update', currentPageData, socket.id);
        previousPageData = currentPageData; 
    }
}


function draw() {
    background(220);
    
    if (!isConnected) {
        showStatus('Conectando al servidor...', color(255, 165, 0));
        return;
    }
    
    if (!hasRemoteData) {
        showStatus('Esperando conexión de la otra ventana...', color(255, 165, 0));
        return;
    }
    
    if (!isFullySynced) {
        showStatus('Sincronizando datos...', color(255, 165, 0));
        return;
    }

    // Solo dibujar cuando esté completamente sincronizado
    drawCircle(point2[0], point2[1]);
    checkWindowPosition();
    
    let vector2 = createVector(remotePageData.x, remotePageData.y);
    let vector1 = createVector(currentPageData.x, currentPageData.y);
    let resultingVector = createVector(vector2.x - vector1.x, vector2.y - vector1.y);
    
    stroke(50);
    strokeWeight(20);
    drawCircle(resultingVector.x + remotePageData.width / 2, resultingVector.y + remotePageData.height / 2);
    line(point2[0], point2[1], resultingVector.x + remotePageData.width / 2, resultingVector.y + remotePageData.height / 2);
}

function showStatus(message, statusColor) {
    textSize(24);
    textAlign(CENTER, CENTER);
    noStroke();
    // Dibujar rectángulo de fondo para el texto
    fill(0, 0, 0, 150); // Negro semi-transparente
    rectMode(CENTER);
    let textW = textWidth(message) + 40;
    let textH = 40;
    rect(width / 2, 1*height / 6, textW, textH, 10);
    // Dibujar el texto
    fill(statusColor);
    text(message, width / 2, 1*height / 6);
}

function drawCircle(x, y) {
    fill(255, 0, 0);
    ellipse(x, y, 150, 150);
}

function windowResized() {
    resizeCanvas(windowWidth, windowHeight);
}
~~~

server.js
~~~js
const express = require('express');
const http = require('http');
const socketIO = require('socket.io');
const path = require('path');
const app = express();
const server = http.createServer(app); 
const io = socketIO(server); 
const port = 3000;

let page1 = { x: 0, y: 0, width: 100, height: 100 };
let page2 = { x: 0, y: 0, width: 100, height: 100 };
let connectedClients = new Map();
let syncedClients = new Set();

app.use(express.static(path.join(__dirname, 'views')));

app.get('/page1', (req, res) => {
    res.sendFile(path.join(__dirname, 'views', 'page1.html'));
});

app.get('/page2', (req, res) => {
    res.sendFile(path.join(__dirname, 'views', 'page2.html'));
});

io.on('connection', (socket) => {
    console.log('A user connected - ID:', socket.id);
    connectedClients.set(socket.id, { page: null, synced: false });
    
    socket.on('disconnect', () => {
        console.log('User disconnected - ID:', socket.id);
        connectedClients.delete(socket.id);
        syncedClients.delete(socket.id);
        // Notificar a otros clientes que se perdió la sincronización
        socket.broadcast.emit('peerDisconnected');
    });

    socket.on('win1update', (window1, sendid) => {
        console.log('Received win1update from ID:', socket.id, 'Data:', window1);
        if (isValidWindowData(window1)) {
            page1 = window1;
            connectedClients.set(socket.id, { page: 'page1', synced: false });
            socket.broadcast.emit('getdata', { data: page1, from: 'page1' });
            checkAndNotifySyncStatus();
        }
    });

    socket.on('win2update', (window2, sendid) => {
        console.log('Received win2update from ID:', socket.id, 'Data:', window2);
        if (isValidWindowData(window2)) {
            page2 = window2;
            connectedClients.set(socket.id, { page: 'page2', synced: false });
            socket.broadcast.emit('getdata', { data: page2, from: 'page2' });
            checkAndNotifySyncStatus();
        }
    });

    socket.on('requestSync', () => {
        const clientInfo = connectedClients.get(socket.id);
        if (clientInfo?.page === 'page1') {
            socket.emit('getdata', { data: page2, from: 'page2' });
        } else if (clientInfo?.page === 'page2') {
            socket.emit('getdata', { data: page1, from: 'page1' });
        }
    });

    socket.on('confirmSync', () => {
        syncedClients.add(socket.id);
        const clientInfo = connectedClients.get(socket.id);
        if (clientInfo) {
            connectedClients.set(socket.id, { ...clientInfo, synced: true });
        }
        checkAndNotifySyncStatus();
    });    
});

function isValidWindowData(data) {
    return data && 
           typeof data.x === 'number' && 
           typeof data.y === 'number' && 
           typeof data.width === 'number' && data.width > 0 &&
           typeof data.height === 'number' && data.height > 0;
}

function checkAndNotifySyncStatus() {
    const page1Clients = Array.from(connectedClients.entries()).filter(([id, info]) => info.page === 'page1');
    const page2Clients = Array.from(connectedClients.entries()).filter(([id, info]) => info.page === 'page2');
    
    const bothPagesConnected = page1Clients.length > 0 && page2Clients.length > 0;
    const allClientsSynced = Array.from(connectedClients.keys()).every(id => syncedClients.has(id));
    const hasMinimumClients = connectedClients.size >= 2;

    console.log(`Debug - Connected clients: ${connectedClients.size}, Page1: ${page1Clients.length}, Page2: ${page2Clients.length}, Synced: ${syncedClients.size}`);

    
    if (bothPagesConnected && allClientsSynced && hasMinimumClients) {
        io.emit('fullySynced', true);
        console.log('All clients are fully synced');
    } else {
        io.emit('fullySynced', false);
        console.log(`Sync status: pages=${bothPagesConnected}, synced=${allClientsSynced}, clients=${connectedClients.size}`);
    }
}

server.listen(port, () => {
    console.log(`Server is listening on http://localhost:${port}`);
});

~~~

## Actividad 01:

- ¿Qué ocurrió en la terminal cuando ejecutaste npm install? ¿Cuál crees que es su propósito?

~~~cmd
up to date, audited 121 packages in 753ms

17 packages are looking for funding
 run 'npm fund' for details

found 0 vulnerabilities
~~~
  
- ¿Qué mensaje específico apareció en la terminal después de ejecutar npm start? ¿Qué indica este mensaje?

~~~cmd
$ ../node server.js
Server is listening on http://localhost:3000
~~~

- Describe lo que ves inicialmente en page1 y page2 en tu navegador.

Inicialmente se ve lo siguiente al iniciar `page1` sin haber iniciado la otra.
<p align="center">
   <img width="589" height="575" alt="image" src="https://github.com/user-attachments/assets/b1f7e103-9346-4da2-bad9-cbe0f8d0b3c5" />
</p>

Cuando están las dos páginas abiertas y el servidor iniciado podemos ver en cada uno un canvas con un circulo rojo con contorno negro iniciado en la mitad, tambien tiene una linea recta que se conecta con la de la otra página.

- ¿Qué mensajes aparecieron en la terminal del servidor cuando abriste page1 y page2?

~~~
A user connected - ID: hKxHEpULorndgFr3AAAC
A user connected - ID: Plrrg6VfuV2qrAtRAAAD
~~~

- Describe qué sucede en ambas páginas del navegador cuando mueves una de las ventanas. ¿Cambia algo visualmente? ¿Qué mensajes aparecen (si los hay) en la consola del navegador (usualmente accesible con F12 -> Pestaña Consola) y en la terminal del servidor?

<img width="1920" height="1030" alt="image" src="https://github.com/user-attachments/assets/2c59a11c-404d-4b08-98e9-47cbd2cf5577" />

#### 

Al mover una de las ventanas, se manda un mensaje a la otra para que la linea siga unida sin importar a donde se mueva o si se cambia el tamaño del canvas, dando la idea de que están siemore conectadas.

#### 

<p align="center">
<img width="500" height="270" alt="image" src="https://github.com/user-attachments/assets/80fab357-0b03-4c4e-a81d-43539d8d77a9" />
<img width="500" height="270" alt="image" src="https://github.com/user-attachments/assets/a9440608-94f9-4f0c-95b1-3ca74c50c54d" />
</p>

~~~
Received win1update from ID: 47ET8aNdzdFhUxJxAAAB Data: { x: 375, y: 116, width: 598, height: 501 }
Debug - Connected clients: 1, Page1: 1, Page2: 0, Synced: 0
Sync status: pages=false, synced=false, clients=1
A user connected - ID: Ia2PK6d36OYdX-ZhAAAD
Received win2update from ID: Ia2PK6d36OYdX-ZhAAAD Data: { x: 975, y: 116, width: 666, height: 501 }
Debug - Connected clients: 2, Page1: 1, Page2: 1, Synced: 0
Sync status: pages=true, synced=false, clients=2
Debug - Connected clients: 2, Page1: 1, Page2: 1, Synced: 1
Sync status: pages=true, synced=false, clients=2
Debug - Connected clients: 2, Page1: 1, Page2: 1, Synced: 1
Sync status: pages=true, synced=false, clients=2
Debug - Connected clients: 2, Page1: 1, Page2: 1, Synced: 2
All clients are fully synced
~~~

## Actividad 02:

---

> **¿Qué es Internet?** Imagina Internet no como una “nube” etérea, sino como una gigantesca red de carreteras y cables conectando millones de lugares: bibliotecas, tiendas, oficinas, casas… y también unos lugares especiales llamados Servidores. Tu computador (o teléfono) es tu vehículo, conectado a estas carreteras.

### Piensa en cómo te conectas a Internet en casa o en la Universidad. ¿Usas Wi-Fi? ¿Un cable de red? Eso es simplemente tu “rampa de acceso” a la gran red de carreteras. ¿Qué pasaría si esa rampa se corta? Anota tus ideas.

Mi dispositivo móvil lo conecto a el internet directamente o con un amplificador de señal inalambrico en otro punto del apartamento. En el PC uso un cable de red. Asumo que si esa rampa de acceso se corta, es como si se dañara un puente entre los dos puntos. Puede que yo siga enviando señales sin embargo si la rampa de acceso no funciona entonces el receptor jamás va a poder leerlas o dar una retroalimentación.

---

> **Navegador y servidor**
> Tu navegador web (Chrome, Firefox, Safari, Edge…) es tu vehículo súper inteligente. No solo te lleva por las carreteras (Internet), sino que sabe cómo pedir cosas y, lo más importante, ¡Cómo mostrarte lo que recibe! Eres tú, el usuario, quien decide a dónde ir. Tú eres el Cliente.
> Un Servidor es como una biblioteca o un almacén gigante abierto 24/7, ubicado en algún punto de esa red de carreteras. Su trabajo principal es “servir” información o funcionalidad cuando un Cliente (como tu navegador) se la pide correctamente.
> La base: el modelo Cliente-Servidor = el Cliente pide, el Servidor responde.

### ¿Puedes identificar otros ejemplos de relaciones Cliente-Servidor en tu vida diaria (no necesariamente digitales)? Por ejemplo, al pedir comida en un restaurante. ¿Quién es el cliente y quién el servidor? ¿Qué se pide y qué se entrega?



---

> **¿Qué es una URL?**
> Para que tu Navegador sepa a qué Servidor específico ir dentro de esa inmensa red, necesita una dirección precisa. Esa dirección es la URL (Uniform Resource Locator).
> Desglosemos una URL típica: *http://www.ejemplo.com/pagina/index.html*
> - *http://*
> El protocolo. Son las reglas del idioma que usarán tu navegador y el servidor para hablar. ¡Volveremos a esto!
> - *www.ejemplo.com*
> El nombre de dominio. Es como el nombre del edificio o de la biblioteca. Detrás de escena, este nombre se traduce a una dirección numérica (la dirección IP) que sí indica la ubicación física en la red.
> - */pagina/index.html*
> La ruta específica dentro de ese servidor. Es como pedir ir a la “sección de arte, Estante 3, Libro 5” dentro de la biblioteca. Indica el recurso exacto que quieres.

### Toma la URL de tu sitio web favorito. Intenta identificar el protocolo, el nombre de dominio y la ruta (si la hay).

#### https://neal.fun/infinite-craft/

*Protocolo:* https://
*Nombre de dominio:* neal.fun
*Ruta:* /infinite-craft/

### ¿Qué crees que pasa si solo escribes el nombre de dominio (ej. www.google.com) sin una ruta específica?

Me manda al directorio de juegos de neal.fun, es la ruta la que me dirige al juego en particular.

<p align="center">
<img width="1919" height="1027" alt="image" src="https://github.com/user-attachments/assets/e1a73487-1166-4798-a133-32b8728df0c4" />
</p>

### ¿Qué “página por defecto” crees que te envía el servidor?

Cuando solo escribo neal.fun sin nada más, el servidor me manda a la página principal de la web, como al “inicio” de la biblioteca. O sea, no entro directo al juego, sino que me muestra el menú o la portada con las distintas opciones que ofrece el sitio. Eso pasa porque normalmente los servidores tienen configurado un archivo por defecto (tipo index.html) que se carga cuando no se especifica ninguna ruta.

---

> **Protocolo HTTP**
> Dijimos que http era el protocolo. ¿Pero qué significa eso? Recuerda las unidades anteriores donde usaste protocolos (ASCII, binario con framing) para que el micro:bit y p5.js se entendieran por el puerto serial. ¡Aquí es la misma idea, pero a gran escala!
> HTTP (HyperText Transfer Protocol) es el conjunto de reglas estándar que usan los Navegadores (Clientes) y los Servidores para comunicarse en la web.

### Compara HTTP con los protocolos seriales que usaste.
### ¿Qué similitudes encuentras?

En ambos casos hay un lenguaje común para que dos partes se entiendan. Con el micro:bit y p5.js usaba protocolos seriales que definían cómo iban a organizar los bytes y cuándo empezaba/terminaba un mensaje. Con HTTP pasa igual: el navegador y el servidor acuerdan reglas para pedir y entregar información.

### ¿Qué diferencias clave ves?

La comunicación serial con el micro:bit era más directa y sencilla, básicamente solo pasar bytes con cierto orden. En cambio, HTTP es mucho más estructurado: no solo manda datos, también manda cabeceras, métodos (GET, POST, etc.) y estados de respuesta (200, 404, 500…), lo cual hace que el navegador sepa exactamente qué hacer con lo que recibe.

### ¿Por qué crees que HTTP necesita ser más complejo que un simple envío de bytes como hacías con el micro:bit?

Porque en la web se transmite mucha más información y de diferentes tipos (texto, imágenes, videos, formularios, etc.), y además participan millones de dispositivos distintos. Se necesitan reglas claras para que todo sea compatible y seguro, no como con el micro:bit que era un circuito cerrado y simple.

> Cuando un servidor responde a una petición web, normalmente envía un paquete con tres tipos de archivos:
>
> - **HTML (HyperText Markup Language):** estructura de la página, define elementos como títulos, párrafos, imágenes y botones. (Analogía: el plano de una casa)
> - **CSS (Cascading Style Sheets):** estilo visual de los elementos, colores, tamaños, posiciones, fuentes. (Analogía: decoración, muebles, pintura)
> - **JavaScript (JS):** interactividad de la página, permite reaccionar a acciones del usuario, cambiar el HTML y CSS dinámicamente, enviar datos, etc. (Analogía: electricidad, plomería, electrodomésticos)
>
> Ejemplo: un formulario de login
> - HTML → los campos de texto y el botón
> - CSS → colores, fuentes, bordes redondeados, estilos visuales
> - JS → validaciones, mensajes de error instantáneos, mostrar/ocultar contraseña

### ¿Qué parte crees que es HTML (ej. los campos de texto, el botón)?

HTML: arma la base: el título “Inicia sesión”, las dos casillas donde escribo mi correo y contraseña, y el botón para entrar.

### ¿Qué parte es CSS (ej. el color del botón, el tipo de letra)?

CSS: le da estilo: que el fondo del formulario sea blanco con sombra, que el botón tenga esquinas redondeadas y que el texto del título esté más grande y en negrilla.

### ¿Qué parte es JavaScript (ej. la comprobación de si escribiste algo antes de enviar, el mensaje de “contraseña incorrecta” que aparece sin recargar la página)?

JavaScript: mete la lógica: que cuando presione “Entrar” verifique si el correo tiene el formato correcto, que muestre un contador de intentos fallidos, o que me deje ver/ocultar la contraseña con un ícono de ojito.

<img width="2240" height="2139" alt="image" src="https://github.com/user-attachments/assets/52f148be-47a2-4a50-868e-c1c51caf58e9" />

> - El navegador construye primero la estructura HTML (**DOM**) y aplica los estilos CSS.
> - El **JavaScript** se ejecuta cuando encuentra etiquetas `<script>` o mediante atributos como `defer` o `async`.
> - Existen dos modelos de ejecución:
>   - **Imperativo (como p5.js draw()):** se ejecuta paso a paso en un bucle constante.
>   - **Basado en eventos (JS moderno en la web):** las funciones se ejecutan solo cuando ocurre un evento (clic, mensaje del servidor, cambio de tamaño de ventana, etc.).
> - Ventajas del modelo basado en eventos:
>   - Más eficiente, solo responde cuando ocurre algo
>   - Interfaz más reactiva y dinámica
>   - Permite manejar múltiples interacciones sin sobrecargar la página

### Compara el bucle draw() de p5.js con este modelo de “esperar a que algo pase y reaccionar”. 

En draw() todo se ejecuta en un bucle constante, repitiendo las instrucciones ~60 veces por segundo aunque no pase nada nuevo. En cambio, el modelo basado en eventos espera que ocurra algo (como un clic o un cambio de tamaño) y solo entonces ejecuta la función correspondiente. Mientras no haya eventos, no consume recursos.

### ¿Qué ventajas crees que tiene el modelo basado en eventos para una interfaz de usuario web? 

- Es más eficiente, porque solo se ejecuta código cuando es necesario.
- La página se vuelve más reactiva: responde rápido a lo que hace el usuario.
- Permite manejar muchas interacciones diferentes sin bloquear el navegador.

### ¿Sería eficiente tener un bucle draw() redibujando toda la página 60 veces por segundo si nada ha cambiado?

No, sería un desperdicio de recursos. El navegador tendría que recalcular y pintar elementos que no cambiaron, lo que haría la página más lenta, especialmente si hay mucho contenido.

> Hasta ahora, JavaScript solo se ejecutaba dentro del navegador. Node.js permite usar el motor de JavaScript V8 de Google Chrome **fuera del navegador**, directamente en un servidor.  
> Esto significa que podemos escribir código del lado del servidor usando el **mismo lenguaje** que usamos en el cliente.  
> 
> En nuestro caso de estudio, `server.js` es un script de Node.js que:
> - Crea un servidor HTTP (como una biblioteca abierta 24/7).  
> - Escucha peticiones de los navegadores (por ejemplo, cuando pedimos `/page1`).  
> - Envía los archivos HTML, CSS y JS necesarios.  
> - Maneja comunicación en tiempo real entre cliente y servidor.

### ¿Por qué crees que podría ser útil usar JavaScript tanto en el cliente (navegador) como en el servidor?

Es útil porque no necesitamos aprender un lenguaje distinto para el servidor. Podemos manejar toda la lógica de la aplicación con JavaScript, lo que hace que el desarrollo sea más consistente y rápido, y facilita compartir código entre cliente y servidor, como funciones de validación o manejo de datos.

### ¿Se te ocurre alguna ventaja para los desarrolladores?
Algunas ventajas son:  
- Menos aprendizaje de lenguajes diferentes, todo en JS.  
- Posibilidad de compartir código entre cliente y servidor.  
- Facilita la comunicación en tiempo real y el manejo de datos entre cliente y servidor sin complicaciones.

> En HTTP tradicional, la comunicación es como enviar correos electrónicos: el cliente pide algo, espera, y el servidor responde. Funciona bien para páginas web, pero **no es eficiente para comunicación instantánea**.  
> 
> Los **WebSockets** permiten abrir una conexión directa y permanente entre el cliente y el servidor, de manera que ambos puedan enviarse mensajes al instante sin repetir peticiones HTTP.  
> 
> **Socket.IO** es una librería que facilita usar WebSockets y asegura que funcione incluso si no están disponibles, ofreciendo funciones como:
> - `socket.on('nombreDelMensaje', funcionReceptora)` → escuchar mensajes  
> - `socket.emit('nombreDelMensaje', datosAEnviar)` → enviar mensajes

### Resume con tus propias palabras la diferencia fundamental entre una comunicación HTTP tradicional y una comunicación usando WebSockets/Socket.IO
La diferencia principal es que HTTP es puntual, cada mensaje requiere una petición y respuesta separada, mientras que WebSockets/Socket.IO permiten una conexión permanente y bidireccional, donde cliente y servidor pueden enviarse mensajes instantáneamente sin esperar nuevas peticiones.

### ¿En qué tipo de aplicaciones has visto o podrías imaginar que se usa esta comunicación en tiempo real?
Se usa en aplicaciones como:  
- Chats y mensajería instantánea  
- Juegos en línea donde ves la posición de otros jugadores en tiempo real  
- Colaboración en documentos compartidos (como Google Docs)  

## Actividad 03:

---
### Experimento 01:

> Detén el servidor si está corriendo.  
> Cambia la primera ruta de /page1 a /pagina_uno.  
> Inicia el servidor.  
> Intenta acceder a http://localhost:3000/page1.  

<img width="535" height="118" alt="image" src="https://github.com/user-attachments/assets/c783461b-c4bd-449b-9086-523ef86bc71d" />

#### ¿Funciona?

<img width="283" height="108" alt="image" src="https://github.com/user-attachments/assets/009e6d37-292f-4525-8772-ee40c34b61ba" />
No, no funciona ya que la ruta no coincide con la nueva del server.

> Ahora intenta acceder a http://localhost:3000/pagina_uno.  

#### ¿Funciona?

Si.
<img width="1136" height="266" alt="image" src="https://github.com/user-attachments/assets/114e7066-ca66-4b9a-ac6e-417c0a79fbcc" />


> Restaura el código.  

#### ¿Qué te dice esto sobre cómo el servidor asocia URLs con respuestas?

El servidor directamente nombra la ruta a la cual debe acceder la persona para acceder al contenido.

---
### Experimento 02:

> Asegúrate de que el servidor esté corriendo (npm start).  
> Abre http://localhost:3000/page1 en una pestaña y observa la terminal del servidor.  

#### ¿Qué mensaje ves al abrir page1? Anota el ID.

<img width="1059" height="405" alt="image" src="https://github.com/user-attachments/assets/db231a4f-9490-42c3-9e5d-ea1ef51eac81" />

> Abre http://localhost:3000/page2 en otra pestaña y observa la terminal.  

#### ¿Qué mensaje ves al abrir page2? ¿El ID es diferente?

<img width="682" height="109" alt="image" src="https://github.com/user-attachments/assets/44deadaf-70eb-4c24-9785-fb23b97740b1" />

> Cierra la pestaña de page1 y observa la terminal.  

#### ¿Qué mensaje ves al cerrar la pestaña de page1? ¿Coincide el ID?

User disconnected - ID: XCCkvFac6tpd34SBAAAB

Si, el ID coincide con el de page1.

> Cierra la pestaña de page2 y observa la terminal.  

#### ¿Qué mensaje ves al cerrar la pestaña de page2?

User disconnected - ID: R_ZTJJldVSY2sBfgAAAD

---
### Experimento 03:

> Inicia el servidor y abre page1 y page2.  
> Mueve la ventana de page1 y observa la terminal del servidor.
<img width="1142" height="929" alt="image" src="https://github.com/user-attachments/assets/8aa52ea5-f77e-4614-8e67-642f84eb3b69" />


#### ¿Qué evento se registra al mover page1? ¿Qué datos (Data:) ves?

Por ejemplo:
~~~
Received win1update from ID: OxvM9vTXerJOe2NAAAAB Data: { x: 115, y: 103, width: 631, height: 562 }
Debug - Connected clients: 2, Page1: 1, Page2: 1, Synced: 2
All clients are fully synced
~~~

> Mueve la ventana de page2 y observa la terminal.  

#### ¿Qué evento se registra al mover page2? ¿Qué datos ves?

~~~
Received win2update from ID: lyuSC7ccUcriX0xIAAAD Data: { x: 1065, y: 209, width: 612, height: 579 }
Debug - Connected clients: 2, Page1: 1, Page2: 1, Synced: 2
All clients are fully synced
~~~

> Cambia `socket.broadcast.emit('getdata', page1);` por `socket.emit('getdata', page1);`  
> Reinicia el servidor y abre ambas páginas.  
> Mueve page1.  

#### ¿Se actualiza la visualización en page2? ¿Por qué sí o por qué no?

<img width="1919" height="999" alt="image" src="https://github.com/user-attachments/assets/b742903e-2365-4450-ad5a-5a338e47665b" />

Al reemplazar socket.broadcast.emit por socket.emit, la comunicación ya no se comparte con todos los clientes conectados, sino que se envía solo al mismo cliente que originó el mensaje. Por eso, cuando se mueve la ventana de page1, esa información no llega a page2, y parece como si estuvieran desconectadas entre sí. Lo que ocurre es que page2 nunca recibe los datos de page1, así que no puede actualizarse. En cambio, page1 sí sigue recibiendo lo que ella misma emite, pero eso no ayuda a la sincronización entre páginas.

---
### Experimento 04:

> Detén el servidor.  
> Cambia `const port = 3000;` a `const port = 3001;`.  
> Inicia el servidor.  

#### ¿Qué mensaje ves en la consola? ¿En qué puerto dice que está escuchando?

<img width="932" height="170" alt="image" src="https://github.com/user-attachments/assets/177e0ed8-9015-4215-a32c-5013e41ec586" />

> Intenta abrir http://localhost:3000/page1.  

#### ¿Funciona?

Nop
<img width="654" height="667" alt="image" src="https://github.com/user-attachments/assets/d7d8e2a6-85da-424c-9754-b71499f86338" />


> Intenta abrir http://localhost:3001/page1.  

#### ¿Funciona?

<img width="646" height="660" alt="image" src="https://github.com/user-attachments/assets/126348a1-c2fe-43f4-b71d-5233e67db081" />
<img width="1123" height="279" alt="image" src="https://github.com/user-attachments/assets/e57d0597-9829-4914-8353-b34bfdc5d235" />


#### ¿Qué aprendiste sobre la variable port y la función listen?

El valor de port funciona como la dirección concreta donde el servidor está disponible. Al estar declarado como constante, el programa siempre “atiende” en ese mismo lugar. Si desde el navegador intentas entrar usando otro puerto, no habrá respuesta porque ahí no hay ningún servicio escuchando. Es parecido a marcar un número equivocado: aunque el teléfono existe, no llegas a la persona correcta porque no es el que está asignado.

---


## Actividad 04:

--- 
### Experimento 01:

> Abre page2.html en tu navegador (con el servidor corriendo).
> Abre la consola de desarrollador (F12).
> Detén el servidor Node.js (Ctrl+C).

#### Refresca la página page2.html. Observa la consola del navegador. ¿Ves algún error relacionado con la conexión? ¿Qué indica?

w

#### Vuelve a iniciar el servidor y refresca la página. ¿Desaparecen los errores?

d

---
### Experimento 02:

> Comenta la línea socket.emit(‘win2update’, currentPageData, socket.id); dentro del listener connect.
> Reinicia el servidor y refresca page1.html y page2.html.
> Mueve la ventana de page2 un poco para que envíe una actualización.

#### ¿Qué pasó? ¿Por qué?

d

---
### Experimento 03: 

> Asegúrate de tener este console.log en page2.js.
> Abre ambas páginas.

#### Mueve la ventana de page1. Observa la consola del navegador de page2. ¿Qué datos muestra?

w

#### Mueve la ventana de page2. Observa la consola de page1. ¿Qué pasa? ¿Por qué?

d

---
### Experimento 04:

> Observa checkWindowPosition() en page2.js y modifica el código del if para comprobar si el código dentreo de este se ejecuta.
> Mueve cada ventana y observa las consolas.

#### ¿Qué puedes concluir y por qué?

d

---
### Experimento 05:

> Sé creativo.

#### Cambia el background(220) para que dependa de la distancia entre las ventanas. Puedes calcular la magnitud del resultingVector usando let distancia = resultingVector.mag(); y luego usa map() para convertir esa distancia a un valor de gris o color. background(map(distancia, 0, 1000, 255, 0)); (ajusta el rango 0-1000 según sea necesario).

wa

#### Inventa otra:

## Actividad 05:

### Explica tu idea y realiza algunos bocetos.

Mi idea es modificar la aplicación volviendo los circulos una princesa y un sapo. Básicamente mi idea es que al estar las páginas alineadas especificamente para que los labios de la princesa y el sapo se encuentren entonces el sapo se transforma en un príncipe.

#### La idea (profe perdón de antemano el final queda más bonito):

<p align="center">
<img width="643" height="365" alt="image" src="https://github.com/user-attachments/assets/c2ac1eb4-8311-4245-b027-63ef65060f65" />
</p>

En realidad la página de la princesa nunca cambiaría, solo la del sapo. Si veo que quizá puedo agregar muchas más cosas lo iré modificando 

- **La página del sapo:**

Planeo tener en cuenta la distancia que se mide constantemente entre los dos e implementar 2 casos según la distancia y un booleano quizá que tenga en cuenta que se hayan dado el beso sin alejarse mucho despues.

1. Si la posición no es la exacta del beso, sin importar la distancia a la que está, y además el bool es falso entonces será un sapo y nada cambia.
2. Si la posición es la del beso y el booleano es falso (entonces aún no se han dado el beso), el bool pasa a ser verdadero y el sapo se convierte en un principe.

Quizá agregué más cositas si puedo, como distintos dibujos en distintos momentos de la aplicación y también que pase algo más cuando se choquen las pantallas quizá que se vaya llenando un corazon y cuando se llene ahí si se convierta en principe. (El ultimo si pude)

#### Bocetos:



### Implementa tu idea.

Inicialmente me voy a dedicar a hacer los diseños simples de la princesa y el sapo e implementarlos en las páginas sin aún cambiar ninguna lógica.

Despúes de muchos intentos la verdad fue algo complejo dibujar con las figuras, voy a añadir fotos del proceso donde también intentando con una IA quedaban terribles

<img width="147" height="203" alt="image" src="https://github.com/user-attachments/assets/c05341c1-342e-4f56-8fe2-d275bf615447" />

<img width="154" height="132" alt="image" src="https://github.com/user-attachments/assets/82fab33f-9beb-40e2-b01a-af8547e05c2c" />

<img width="680" height="223" alt="image" src="https://github.com/user-attachments/assets/f31745ee-f031-4d33-9015-a7381148c228" />

<img width="200" height="272" alt="image" src="https://github.com/user-attachments/assets/808ea1bc-5795-4342-b7b2-7545d41656e2" />

<img width="253" height="262" alt="image" src="https://github.com/user-attachments/assets/c6a7b71e-ff71-4fee-86ed-c3a538417155" />

Ahí me rendí y decidí hacer batman principe y batman princesa. Usé chatGPT para que me hiciera la base que fue tambien un poco feo:
<img width="330" height="296" alt="image" src="https://github.com/user-attachments/assets/27cdbb7c-f498-4243-ac4f-953debab8a15" />
Y de ahí lo modifiqué yo hasta el resultado final.

```js
function drawPrincess(x, y) {
    push();
    translate(x, y);
    
    // corona
    fill(210, 160, 60);
    noStroke();
    rectMode(CENTER);
    rect(0, -102, 20, 5);

    // triángulos de la corona
    beginShape();
    vertex(-10, -104);
    vertex(-7, -118);
    vertex(-5, -104);
    endShape(CLOSE);

    beginShape();
    vertex(-2, -104);
    vertex(0, -118);
    vertex(2, -104);
    endShape(CLOSE);

    beginShape();
    vertex(5, -104);
    vertex(7, -118);
    vertex(10, -104);
    endShape(CLOSE);

    // joyas
    fill(255, 105, 180);
    ellipse(-7, -118, 4, 4);
    ellipse(0, -118, 4, 4);
    ellipse(7, -118, 4, 4);

    // cuerpo
    fill(0);
    beginShape();
    vertex(-60, 40);
    vertex(-40, -60);
    vertex(40, -60);
    vertex(60, 40);
    endShape(CLOSE);

    // capa
    beginShape();
    vertex(-60, 40);
    vertex(-80, 80);
    vertex(80, 80);
    vertex(60, 40);
    endShape(CLOSE);

    // cabeza con orejas
    beginShape();
    vertex(-20, -60);
    vertex(-20, -100);
    vertex(-16, -112);
    vertex(-12, -100);
    vertex(0, -100);
    vertex(12, -100);
    vertex(16, -112);
    vertex(20, -100);
    vertex(20, -60);
    endShape(CLOSE);

    // cara
    fill(255);
    rect(0, -75, 30, 20, 5);

    // ojos
    fill(255);
    quad(-14, -95, -6, -91, -2, -89, -11, -86);
    quad(14, -95, 6, -91, 2, -89, 11, -86);

    // boca
    stroke(0);
    strokeWeight(2);
    line(-5, -75, 5, -75);

    pop();
}

function drawFrog(x, y, scaleFactor = 0.5) {
    push();
    translate(x, y);
    scale(scaleFactor);

    // cuerpo
    fill(120, 220, 120);
    ellipse(0, 40, 140, 100);

    // barriga
    fill(255, 240, 180);
    ellipse(0, 50, 90, 70);

    // cabeza
    fill(120, 220, 120);
    ellipse(0, -20, 120, 90);

    // cachetes
    ellipse(-50, -5, 40, 40);
    ellipse(50, -5, 40, 40);

    // ojos
    fill(255);
    ellipse(-30, -45, 35, 35);
    ellipse(30, -45, 35, 35);
    fill(0);
    ellipse(-30, -45, 18, 18);
    ellipse(30, -45, 18, 18);

    // rubor
    fill(255, 150, 150, 180);
    ellipse(-30, -25, 18, 10);
    ellipse(30, -25, 18, 10);

    // sonrisa
    noFill();
    stroke(0);
    strokeWeight(4);
    arc(0, -20, 60, 40, 0, PI);

    // patas
    noStroke();
    fill(120, 220, 120);
    ellipse(-60, 80, 40, 30);
    ellipse(60, 80, 40, 30);

    // manchitas
    fill(70, 180, 90);
    ellipse(40, -10, 15, 15);
    ellipse(55, -20, 12, 12);
    ellipse(50, 0, 10, 10);

    pop();
}

function drawPrince(x, y) {
    push();
    translate(x, y);
    
    // corona
    fill(210, 160, 60);
    noStroke();
    rectMode(CENTER);
    rect(0, -102, 20, 5);

    // triángulos corona
    beginShape();
    vertex(-10, -104);
    vertex(-7, -118);
    vertex(-5, -104);
    endShape(CLOSE);

    beginShape();
    vertex(-2, -104);
    vertex(0, -118);
    vertex(2, -104);
    endShape(CLOSE);

    beginShape();
    vertex(5, -104);
    vertex(7, -118);
    vertex(10, -104);
    endShape(CLOSE);

    // joyas
    fill(70, 130, 180);
    ellipse(-7, -118, 4, 4);
    ellipse(0, -118, 4, 4);
    ellipse(7, -118, 4, 4);

    // cuerpo
    fill(0);
    beginShape();
    vertex(-60, 40);
    vertex(-40, -60);
    vertex(40, -60);
    vertex(60, 40);
    endShape(CLOSE);

    // capa
    beginShape();
    vertex(-60, 40);
    vertex(-80, 80);
    vertex(80, 80);
    vertex(60, 40);
    endShape(CLOSE);

    // cabeza
    beginShape();
    vertex(-20, -60);
    vertex(-20, -100);
    vertex(-16, -112);
    vertex(-12, -100);
    vertex(0, -100);
    vertex(12, -100);
    vertex(16, -112);
    vertex(20, -100);
    vertex(20, -60);
    endShape(CLOSE);

    // cara
    fill(255);
    rect(0, -75, 30, 20, 5);

    // ojos
    fill(255);
    quad(-14, -95, -6, -91, -2, -89, -11, -86);
    quad(14, -95, 6, -91, 2, -89, 11, -86);

    // boca
    stroke(0);
    strokeWeight(2);
    line(-5, -75, 5, -75);

    pop();
}
```
Agregué la lógica de `isPrince` para manejar la transformación de personaje cuando el corazón se llena:

```js
let isPrince = false; // princesa nunca empieza como príncipe

if (heartFill >= 100 && !isPrince) {
    isPrince = true;
    socket.emit('win1update', { ...currentPageData, isPrince }); // avisar cambio
}
```

Se diferenciaron los personajes propios y remotos con `remoteIsPrince` y control en el `draw()`:

```js
if (hasRemoteData) {
    if (remoteIsPrince) drawPrince(remoteX, remoteY);
    else drawFrog(remoteX, remoteY);
}
```

En la segunda página se invierte la lógica, manteniendo al remoto como princesa:

```js
if (isPrince) drawPrince(point2[0], point2[1]);
else drawFrog(point2[0], point2[1]);

drawPrincess(remoteX, remoteY);
```

Ajusté las coronas y joyas según el personaje para diferenciarlos visualmente:

```js
// Princesa
fill(255, 105, 180); // rosa

// Príncipe
fill(70, 130, 180); // azul zafiro
```

Se mantuvo la sincronización con sockets incluyendo `isPrince`:

```js
socket.on('getdata', (response) => {
    if (response && response.data && isValidRemoteData(response.data)) {
        remotePageData = response.data;
        hasRemoteData = true;
        if (response.data.isPrince !== undefined) {
            remoteIsPrince = response.data.isPrince;
        }
        socket.emit('confirmSync');
    }
});
```

Implementé el corazón como trigger de transformación al acercarse los personajes:

```js
if (dist(point1[0], point1[1], remoteX, remoteY) < 50) {
    heartFill = min(heartFill + 1, 100);
} else {
    heartFill = max(heartFill - 1, 0);
}
drawHeart((point1[0] + remoteX) / 2, (point1[1] + remoteY) / 2, heartFill);
```

Se actualizan los datos emitidos para sincronizar el estado de la transformación:

```js
socket.emit('win1update', { ...currentPageData, isPrince });
```


### Incluye todos los códigos (servidor y clientes) en tu bitácora.

#### Código del servidor.

```js
const express = require('express');
const http = require('http');
const socketIO = require('socket.io');
const path = require('path');
const app = express();
const server = http.createServer(app); 
const io = socketIO(server); 
const port = 3000;

let page1 = { x: 0, y: 0, width: 100, height: 100 };
let page2 = { x: 0, y: 0, width: 100, height: 100 };
let connectedClients = new Map();
let syncedClients = new Set();

app.use(express.static(path.join(__dirname, 'views')));

app.get('/page1', (req, res) => {
    res.sendFile(path.join(__dirname, 'views', 'page1.html'));
});

app.get('/page2', (req, res) => {
    res.sendFile(path.join(__dirname, 'views', 'page2.html'));
});

io.on('connection', (socket) => {
    console.log('A user connected - ID:', socket.id);
    connectedClients.set(socket.id, { page: null, synced: false });
    
    socket.on('disconnect', () => {
        console.log('User disconnected - ID:', socket.id);
        connectedClients.delete(socket.id);
        syncedClients.delete(socket.id);
        // Notificar a otros clientes que se perdió la sincronización
        socket.broadcast.emit('peerDisconnected');
    });

    socket.on('win1update', (window1, sendid) => {
        console.log('Received win1update from ID:', socket.id, 'Data:', window1);
        if (isValidWindowData(window1)) {
            page1 = window1;
            connectedClients.set(socket.id, { page: 'page1', synced: false });
            socket.broadcast.emit('getdata', { data: page1, from: 'page1' });
            checkAndNotifySyncStatus();
        }
    });

    socket.on('win2update', (window2, sendid) => {
        console.log('Received win2update from ID:', socket.id, 'Data:', window2);
        if (isValidWindowData(window2)) {
            page2 = window2;
            connectedClients.set(socket.id, { page: 'page2', synced: false });
            socket.broadcast.emit('getdata', { data: page2, from: 'page2' });
            checkAndNotifySyncStatus();
        }
    });

    socket.on('requestSync', () => {
        const clientInfo = connectedClients.get(socket.id);
        if (clientInfo?.page === 'page1') {
            socket.emit('getdata', { data: page2, from: 'page2' });
        } else if (clientInfo?.page === 'page2') {
            socket.emit('getdata', { data: page1, from: 'page1' });
        }
    });

    socket.on('confirmSync', () => {
        syncedClients.add(socket.id);
        const clientInfo = connectedClients.get(socket.id);
        if (clientInfo) {
            connectedClients.set(socket.id, { ...clientInfo, synced: true });
        }
        checkAndNotifySyncStatus();
    });    
});

function isValidWindowData(data) {
    return data && 
           typeof data.x === 'number' && 
           typeof data.y === 'number' && 
           typeof data.width === 'number' && data.width > 0 &&
           typeof data.height === 'number' && data.height > 0;
}

function checkAndNotifySyncStatus() {
    const page1Clients = Array.from(connectedClients.entries()).filter(([id, info]) => info.page === 'page1');
    const page2Clients = Array.from(connectedClients.entries()).filter(([id, info]) => info.page === 'page2');
    
    const bothPagesConnected = page1Clients.length > 0 && page2Clients.length > 0;
    const allClientsSynced = Array.from(connectedClients.keys()).every(id => syncedClients.has(id));
    const hasMinimumClients = connectedClients.size >= 2;

    console.log(`Debug - Connected clients: ${connectedClients.size}, Page1: ${page1Clients.length}, Page2: ${page2Clients.length}, Synced: ${syncedClients.size}`);

    
    if (bothPagesConnected && allClientsSynced && hasMinimumClients) {
        io.emit('fullySynced', true);
        console.log('All clients are fully synced');
    } else {
        io.emit('fullySynced', false);
        console.log(`Sync status: pages=${bothPagesConnected}, synced=${allClientsSynced}, clients=${connectedClients.size}`);
    }
}

server.listen(port, () => {
    console.log(`Server is listening on http://localhost:${port}`);
});

```
#### Page1

```js
let currentPageData = {
    x: window.screenX,
    y: window.screenY,
    width: window.innerWidth,
    height: window.innerHeight
}

let previousPageData = { ...currentPageData };
let remotePageData = { x: 0, y: 0, width: 100, height: 100 };

let point1 = [currentPageData.width / 2, currentPageData.height / 2];
let socket;
let isConnected = false;
let hasRemoteData = false;
let isFullySynced = false;
let remoteIsPrince = false;

let heartFill = 0;
let isPrince = false; // princesa nunca empieza como príncipe

function setup() {
    createCanvas(windowWidth, windowHeight);
    frameRate(60);
    socket = io();

    socket.on('connect', () => {
        isConnected = true;
        socket.emit('win1update', { ...currentPageData, isPrince }, socket.id);
        setTimeout(() => socket.emit('requestSync'), 500);
    });

    socket.on('getdata', (response) => {
        if (response && response.data && isValidRemoteData(response.data)) {
            remotePageData = response.data;
            hasRemoteData = true;
            if (response.data.isPrince !== undefined) {
                remoteIsPrince = response.data.isPrince;
            }
            socket.emit('confirmSync');
        }
    });

    socket.on('fullySynced', (synced) => {
        isFullySynced = synced;
    });

    socket.on('peerDisconnected', () => {
        hasRemoteData = false;
        isFullySynced = false;
        heartFill = 0;
        remoteIsPrince = false;
    });

    socket.on('disconnect', () => {
        isConnected = false;
        hasRemoteData = false;
        isFullySynced = false;
        heartFill = 0;
        remoteIsPrince = false;
    });
}

function isValidRemoteData(data) {
    return data &&
        typeof data.x === 'number' &&
        typeof data.y === 'number' &&
        typeof data.width === 'number' && data.width > 0 &&
        typeof data.height === 'number' && data.height > 0;
}

function checkWindowPosition() {
    currentPageData = {
        x: window.screenX,
        y: window.screenY,
        width: window.innerWidth,
        height: window.innerHeight
    };

    if (JSON.stringify(currentPageData) !== JSON.stringify(previousPageData)) {
        point1 = [currentPageData.width / 2, currentPageData.height / 2]
        socket.emit('win1update', { ...currentPageData, isPrince }); // enviar isPrince
        previousPageData = { ...currentPageData };
    }
}

function draw() {
    background(220);

    if (!isConnected) { showStatus('Conectando...', color(255, 165, 0)); return; }
    if (!hasRemoteData) { showStatus('Esperando otra ventana...', color(255, 165, 0)); return; }
    if (!isFullySynced) { showStatus('Sincronizando...', color(255, 165, 0)); return; }

    checkWindowPosition();

    let vector1 = createVector(currentPageData.x, currentPageData.y);
    let vector2 = createVector(remotePageData.x, remotePageData.y);
    let resultingVector = createVector(vector2.x - vector1.x, vector2.y - vector1.y);

    let remoteX = resultingVector.x + remotePageData.width / 2;
    let remoteY = resultingVector.y + remotePageData.height / 2;

    // Dibujar personaje propio
    drawPrincess(point1[0], point1[1]);

    // Dibujar personaje remoto
    if (hasRemoteData) {
        if (remoteIsPrince) drawPrince(remoteX, remoteY);
        else drawFrog(remoteX, remoteY);
    }

    // Heart
    if (dist(point1[0], point1[1], remoteX, remoteY) < 50) {
        heartFill = min(heartFill + 1, 100);
    } else {
        heartFill = max(heartFill - 1, 0);
    }

    if (heartFill >= 100 && !isPrince) {
        isPrince = true;
        socket.emit('win1update', { ...currentPageData, isPrince }); // avisar cambio
    }

    drawHeart((point1[0] + remoteX) / 2, (point1[1] + remoteY) / 2, heartFill);
}

function drawPrincess(x, y) {
    push();
    translate(x, y);
    
    // corona
    fill(210, 160, 60);
    noStroke();
    rectMode(CENTER);
    rect(0, -102, 20, 5);

    // triángulos de la corona
    beginShape();
    vertex(-10, -104);
    vertex(-7, -118);
    vertex(-5, -104);
    endShape(CLOSE);

    beginShape();
    vertex(-2, -104);
    vertex(0, -118);
    vertex(2, -104);
    endShape(CLOSE);

    beginShape();
    vertex(5, -104);
    vertex(7, -118);
    vertex(10, -104);
    endShape(CLOSE);

    // joyas
    fill(255, 105, 180);
    ellipse(-7, -118, 4, 4);
    ellipse(0, -118, 4, 4);
    ellipse(7, -118, 4, 4);

    // cuerpo
    fill(0);
    beginShape();
    vertex(-60, 40);
    vertex(-40, -60);
    vertex(40, -60);
    vertex(60, 40);
    endShape(CLOSE);

    // capa
    beginShape();
    vertex(-60, 40);
    vertex(-80, 80);
    vertex(80, 80);
    vertex(60, 40);
    endShape(CLOSE);

    // cabeza con orejas
    beginShape();
    vertex(-20, -60);
    vertex(-20, -100);
    vertex(-16, -112);
    vertex(-12, -100);
    vertex(0, -100);
    vertex(12, -100);
    vertex(16, -112);
    vertex(20, -100);
    vertex(20, -60);
    endShape(CLOSE);

    // cara
    fill(255);
    rect(0, -75, 30, 20, 5);

    // ojos
    fill(255);
    quad(-14, -95, -6, -91, -2, -89, -11, -86);
    quad(14, -95, 6, -91, 2, -89, 11, -86);

    // boca
    stroke(0);
    strokeWeight(2);
    line(-5, -75, 5, -75);

    pop();
}

function drawFrog(x, y, scaleFactor = 0.5) {
    push();
    translate(x, y);
    scale(scaleFactor);

    // cuerpo
    fill(120, 220, 120);
    ellipse(0, 40, 140, 100);

    // barriga
    fill(255, 240, 180);
    ellipse(0, 50, 90, 70);

    // cabeza
    fill(120, 220, 120);
    ellipse(0, -20, 120, 90);

    // cachetes
    ellipse(-50, -5, 40, 40);
    ellipse(50, -5, 40, 40);

    // ojos
    fill(255);
    ellipse(-30, -45, 35, 35);
    ellipse(30, -45, 35, 35);
    fill(0);
    ellipse(-30, -45, 18, 18);
    ellipse(30, -45, 18, 18);

    // rubor
    fill(255, 150, 150, 180);
    ellipse(-30, -25, 18, 10);
    ellipse(30, -25, 18, 10);

    // sonrisa
    noFill();
    stroke(0);
    strokeWeight(4);
    arc(0, -20, 60, 40, 0, PI);

    // patas
    noStroke();
    fill(120, 220, 120);
    ellipse(-60, 80, 40, 30);
    ellipse(60, 80, 40, 30);

    // manchitas
    fill(70, 180, 90);
    ellipse(40, -10, 15, 15);
    ellipse(55, -20, 12, 12);
    ellipse(50, 0, 10, 10);

    pop();
}

function drawPrince(x, y) {
    push();
    translate(x, y);
    
    // corona
    fill(210, 160, 60);
    noStroke();
    rectMode(CENTER);
    rect(0, -102, 20, 5);

    // triángulos corona
    beginShape();
    vertex(-10, -104);
    vertex(-7, -118);
    vertex(-5, -104);
    endShape(CLOSE);

    beginShape();
    vertex(-2, -104);
    vertex(0, -118);
    vertex(2, -104);
    endShape(CLOSE);

    beginShape();
    vertex(5, -104);
    vertex(7, -118);
    vertex(10, -104);
    endShape(CLOSE);

    // joyas
    fill(70, 130, 180);
    ellipse(-7, -118, 4, 4);
    ellipse(0, -118, 4, 4);
    ellipse(7, -118, 4, 4);

    // cuerpo
    fill(0);
    beginShape();
    vertex(-60, 40);
    vertex(-40, -60);
    vertex(40, -60);
    vertex(60, 40);
    endShape(CLOSE);

    // capa
    beginShape();
    vertex(-60, 40);
    vertex(-80, 80);
    vertex(80, 80);
    vertex(60, 40);
    endShape(CLOSE);

    // cabeza
    beginShape();
    vertex(-20, -60);
    vertex(-20, -100);
    vertex(-16, -112);
    vertex(-12, -100);
    vertex(0, -100);
    vertex(12, -100);
    vertex(16, -112);
    vertex(20, -100);
    vertex(20, -60);
    endShape(CLOSE);

    // cara
    fill(255);
    rect(0, -75, 30, 20, 5);

    // ojos
    fill(255);
    quad(-14, -95, -6, -91, -2, -89, -11, -86);
    quad(14, -95, 6, -91, 2, -89, 11, -86);

    // boca
    stroke(0);
    strokeWeight(2);
    line(-5, -75, 5, -75);

    pop();
}



function drawHeart(x, y, fillPercent) {
    push();
    translate(x, y);
    scale(0.5);

    let c = color(255, 0, 0);
    fill(red(c), green(c), blue(c), map(fillPercent, 0, 100, 50, 255));
    noStroke();

    beginShape();
    vertex(0, -30);
    bezierVertex(25, -60, 70, -20, 0, 50);
    bezierVertex(-70, -20, -25, -60, 0, -30);
    endShape(CLOSE);
    pop();
}

function showStatus(message, statusColor) {
    textSize(24);
    textAlign(CENTER, CENTER);
    noStroke();
    fill(0, 0, 0, 150);
    rectMode(CENTER);
    let textW = textWidth(message) + 40;
    rect(width / 2, height / 6, textW, 40, 10);
    fill(statusColor);
    text(message, width / 2, height / 6);
}

function windowResized() {
    resizeCanvas(windowWidth, windowHeight);
}

```
#### Page2

```js
let currentPageData = {
    x: window.screenX,
    y: window.screenY,
    width: window.innerWidth,
    height: window.innerHeight
}

let previousPageData = { ...currentPageData };
let remotePageData = { x: 0, y: 0, width: 100, height: 100 };

let point2 = [currentPageData.width / 2, currentPageData.height / 2];
let socket;
let remoteIsPrince = false;
let isConnected = false;
let hasRemoteData = false;
let isFullySynced = false;

let heartFill = 0;
let isPrince = false;

function setup() {
    createCanvas(windowWidth, windowHeight);
    frameRate(60);
    socket = io();

    socket.on('connect', () => {
        isConnected = true;
        socket.emit('win2update', { ...currentPageData, isPrince }, socket.id);
        setTimeout(() => socket.emit('requestSync'), 500);
    });

    socket.on('getdata', (response) => {
        if (response && response.data && isValidRemoteData(response.data)) {
            remotePageData = response.data;
            hasRemoteData = true;
            if (response.data.isPrince !== undefined) remoteIsPrince = response.data.isPrince;
            socket.emit('confirmSync');
        }
    });

    socket.on('fullySynced', (synced) => { isFullySynced = synced; });

    socket.on('peerDisconnected', () => {
        hasRemoteData = false;
        isFullySynced = false;
        heartFill = 0;
        isPrince = false;
        remoteIsPrince = false;
    });

    socket.on('disconnect', () => {
        isConnected = false;
        hasRemoteData = false;
        isFullySynced = false;
        heartFill = 0;
        isPrince = false;
        remoteIsPrince = false;
    });
}

function isValidRemoteData(data) {
    return data &&
        typeof data.x === 'number' &&
        typeof data.y === 'number' &&
        typeof data.width === 'number' && data.width > 0 &&
        typeof data.height === 'number' && data.height > 0;
}

function checkWindowPosition() {
    currentPageData = {
        x: window.screenX,
        y: window.screenY,
        width: window.innerWidth,
        height: window.innerHeight
    };

    if (JSON.stringify(currentPageData) !== JSON.stringify(previousPageData)) {
        point2 = [currentPageData.width / 2, currentPageData.height / 2];
        socket.emit('win2update', { ...currentPageData, isPrince });
        previousPageData = { ...currentPageData };
    }
}

function draw() {
    background(220);

    if (!isConnected) { showStatus('Conectando...', color(255, 165, 0)); return; }
    if (!hasRemoteData) { showStatus('Esperando otra ventana...', color(255, 165, 0)); return; }
    if (!isFullySynced) { showStatus('Sincronizando...', color(255, 165, 0)); return; }

    checkWindowPosition();

    let vector2 = createVector(remotePageData.x, remotePageData.y);
    let vector1 = createVector(currentPageData.x, currentPageData.y);
    let resultingVector = createVector(vector2.x - vector1.x, vector2.y - vector1.y);

    let remoteX = resultingVector.x + remotePageData.width / 2;
    let remoteY = resultingVector.y + remotePageData.height / 2;

    // Dibujar personaje propio
    if (isPrince) drawPrince(point2[0], point2[1]);
    else drawFrog(point2[0], point2[1]);

    // Dibujar personaje remoto
    if (hasRemoteData) {
        
        drawPrincess(remoteX, remoteY);
    }

    // Heart
    if (dist(point2[0], point2[1], remoteX, remoteY) < 50) {
        heartFill = min(heartFill + 1, 100);
    } else {
        heartFill = max(heartFill - 1, 0);
    }

    if (heartFill >= 100 && !isPrince) {
        isPrince = true;
        socket.emit('win2update', { ...currentPageData, isPrince });
    }

    drawHeart((point2[0] + remoteX) / 2, (point2[1] + remoteY) / 2, heartFill);
}

function drawPrincess(x, y) {
    push();
    translate(x, y);
    
    // corona
    fill(210, 160, 60);
    noStroke();
    rectMode(CENTER);
    rect(0, -102, 20, 5);

    // triángulos de la corona
    beginShape();
    vertex(-10, -104);
    vertex(-7, -118);
    vertex(-5, -104);
    endShape(CLOSE);

    beginShape();
    vertex(-2, -104);
    vertex(0, -118);
    vertex(2, -104);
    endShape(CLOSE);

    beginShape();
    vertex(5, -104);
    vertex(7, -118);
    vertex(10, -104);
    endShape(CLOSE);

    // joyas
    fill(255, 105, 180);
    ellipse(-7, -118, 4, 4);
    ellipse(0, -118, 4, 4);
    ellipse(7, -118, 4, 4);

    // cuerpo
    fill(0);
    beginShape();
    vertex(-60, 40);
    vertex(-40, -60);
    vertex(40, -60);
    vertex(60, 40);
    endShape(CLOSE);

    // capa
    beginShape();
    vertex(-60, 40);
    vertex(-80, 80);
    vertex(80, 80);
    vertex(60, 40);
    endShape(CLOSE);

    // cabeza con orejas
    beginShape();
    vertex(-20, -60);
    vertex(-20, -100);
    vertex(-16, -112);
    vertex(-12, -100);
    vertex(0, -100);
    vertex(12, -100);
    vertex(16, -112);
    vertex(20, -100);
    vertex(20, -60);
    endShape(CLOSE);

    // cara
    fill(255);
    rect(0, -75, 30, 20, 5);

    // ojos
    fill(255);
    quad(-14, -95, -6, -91, -2, -89, -11, -86);
    quad(14, -95, 6, -91, 2, -89, 11, -86);

    // boca
    stroke(0);
    strokeWeight(2);
    line(-5, -75, 5, -75);

    pop();
}

function drawFrog(x, y, scaleFactor = 0.5) {
    push();
    translate(x, y);
    scale(scaleFactor);

    // cuerpo
    fill(120, 220, 120);
    ellipse(0, 40, 140, 100);

    // barriga
    fill(255, 240, 180);
    ellipse(0, 50, 90, 70);

    // cabeza
    fill(120, 220, 120);
    ellipse(0, -20, 120, 90);

    // cachetes
    ellipse(-50, -5, 40, 40);
    ellipse(50, -5, 40, 40);

    // ojos
    fill(255);
    ellipse(-30, -45, 35, 35);
    ellipse(30, -45, 35, 35);
    fill(0);
    ellipse(-30, -45, 18, 18);
    ellipse(30, -45, 18, 18);

    // rubor
    fill(255, 150, 150, 180);
    ellipse(-30, -25, 18, 10);
    ellipse(30, -25, 18, 10);

    // sonrisa
    noFill();
    stroke(0);
    strokeWeight(4);
    arc(0, -20, 60, 40, 0, PI);

    // patas
    noStroke();
    fill(120, 220, 120);
    ellipse(-60, 80, 40, 30);
    ellipse(60, 80, 40, 30);

    // manchitas
    fill(70, 180, 90);
    ellipse(40, -10, 15, 15);
    ellipse(55, -20, 12, 12);
    ellipse(50, 0, 10, 10);

    pop();
}

function drawPrince(x, y) {
    push();
    translate(x, y);
    
    // corona
    fill(210, 160, 60);
    noStroke();
    rectMode(CENTER);
    rect(0, -102, 20, 5);

    // triángulos corona
    beginShape();
    vertex(-10, -104);
    vertex(-7, -118);
    vertex(-5, -104);
    endShape(CLOSE);

    beginShape();
    vertex(-2, -104);
    vertex(0, -118);
    vertex(2, -104);
    endShape(CLOSE);

    beginShape();
    vertex(5, -104);
    vertex(7, -118);
    vertex(10, -104);
    endShape(CLOSE);

    // joyas
    fill(70, 130, 180);
    ellipse(-7, -118, 4, 4);
    ellipse(0, -118, 4, 4);
    ellipse(7, -118, 4, 4);

    // cuerpo
    fill(0);
    beginShape();
    vertex(-60, 40);
    vertex(-40, -60);
    vertex(40, -60);
    vertex(60, 40);
    endShape(CLOSE);

    // capa
    beginShape();
    vertex(-60, 40);
    vertex(-80, 80);
    vertex(80, 80);
    vertex(60, 40);
    endShape(CLOSE);

    // cabeza
    beginShape();
    vertex(-20, -60);
    vertex(-20, -100);
    vertex(-16, -112);
    vertex(-12, -100);
    vertex(0, -100);
    vertex(12, -100);
    vertex(16, -112);
    vertex(20, -100);
    vertex(20, -60);
    endShape(CLOSE);

    // cara
    fill(255);
    rect(0, -75, 30, 20, 5);

    // ojos
    fill(255);
    quad(-14, -95, -6, -91, -2, -89, -11, -86);
    quad(14, -95, 6, -91, 2, -89, 11, -86);

    // boca
    stroke(0);
    strokeWeight(2);
    line(-5, -75, 5, -75);

    pop();
}






function drawHeart(x, y, fillPercent) {
    push();
    translate(x, y);
    scale(0.5);
    let c = color(255, 0, 0);
    fill(red(c), green(c), blue(c), map(fillPercent, 0, 100, 50, 255));
    noStroke();
    beginShape();
    vertex(0, -30);
    bezierVertex(25, -60, 70, -20, 0, 50);
    bezierVertex(-70, -20, -25, -60, 0, -30);
    endShape(CLOSE);
    pop();
}

function showStatus(message, statusColor) {
    textSize(24);
    textAlign(CENTER, CENTER);
    noStroke();
    fill(0, 0, 0, 150);
    rectMode(CENTER);
    let textW = textWidth(message) + 40;
    rect(width / 2, height / 6, textW, 40, 10);
    fill(statusColor);
    text(message, width / 2, height / 6);
}

function windowResized() {
    resizeCanvas(windowWidth, windowHeight);
}

```




