
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
<img width="589" height="575" alt="image" src="https://github.com/user-attachments/assets/b1f7e103-9346-4da2-bad9-cbe0f8d0b3c5" />
Cuando están las dos páginas abiertas y el servidor iniciado podemos ver en cada uno un canvas con un circulo rojo con contorno negro iniciado en la mitad, tambien tiene una linea recta que se conecta con la de la otra página.

- ¿Qué mensajes aparecieron en la terminal del servidor cuando abriste page1 y page2?

~~~
A user connected - ID: hKxHEpULorndgFr3AAAC
A user connected - ID: Plrrg6VfuV2qrAtRAAAD
~~~

- Describe qué sucede en ambas páginas del navegador cuando mueves una de las ventanas. ¿Cambia algo visualmente? ¿Qué mensajes aparecen (si los hay) en la consola del navegador (usualmente accesible con F12 -> Pestaña Consola) y en la terminal del servidor?

<img width="1919" height="1027" alt="image" src="https://github.com/user-attachments/assets/2c59a11c-404d-4b08-98e9-47cbd2cf5577" />

Al mover una de las ventanas, se manda un mensaje a la otra para que la linea siga unida sin importar a donde se mueva o si se cambia el tamaño del canvas, dando la idea de que están siemore conectadas.

<img width="512" height="276" alt="image" src="https://github.com/user-attachments/assets/80fab357-0b03-4c4e-a81d-43539d8d77a9" />

Ampliando lo visto:
<img width="514" height="274" alt="image" src="https://github.com/user-attachments/assets/a9440608-94f9-4f0c-95b1-3ca74c50c54d" />


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
