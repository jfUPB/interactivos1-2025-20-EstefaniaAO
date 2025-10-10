
# Evidencias de la unidad 7


<img width="315" height="457" alt="image" src="https://github.com/user-attachments/assets/8acac7c2-acbb-4d1e-b6b9-878dc5bc65b8" />
<img width="383" height="134" alt="image" src="https://github.com/user-attachments/assets/a5043489-667b-4033-a3e7-a8187b8f5243" />

## ¿Qué URL de Dev Tunnels obtuviste? ¿Por qué crees que necesitamos usar esta URL en lugar de http://localhost:3000 o la IP local de tu computador para que el celular se conecte?

https://5t0fc03h-3000.use2.devtunnels.ms/
Ya que el computador y el celular pueden no estar conectados a la misma red local. Cuando estas hosteando localmente solo es para ese mismo computador, más no es un link que funcionaría en todos los demás.

## Describe brevemente qué hace npm install y npm start.



## ¿Qué mensajes observaste en la terminal del servidor al conectar el cliente de escritorio y el cliente móvil? ¿Eran diferentes los mensajes o identificadores?

Son iguales
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

## Solo se registra que un cliente se conectó sin importar cual, es decir, solo busca como tal el evento de conexión.

Describe el comportamiento observado: ¿Funcionó la interacción? ¿Hubo algún retraso (latencia)?

