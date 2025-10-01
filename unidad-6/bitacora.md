
# Evidencias de la unidad 6

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



---

> **Protocolo HTTP**
> Dijimos que http era el protocolo. ¿Pero qué significa eso? Recuerda las unidades anteriores donde usaste protocolos (ASCII, binario con framing) para que el micro:bit y p5.js se entendieran por el puerto serial. ¡Aquí es la misma idea, pero a gran escala!
> HTTP (HyperText Transfer Protocol) es el conjunto de reglas estándar que usan los Navegadores (Clientes) y los Servidores para comunicarse en la web.

### Compara HTTP con los protocolos seriales que usaste.

l

### ¿Qué similitudes encuentras?

l

### ¿Qué diferencias clave ves?

l

### ¿Por qué crees que HTTP necesita ser más complejo que un simple envío de bytes como hacías con el micro:bit?

l

