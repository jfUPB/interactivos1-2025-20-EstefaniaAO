
# Evidencias de la unidad 5

## Actividad 01:
> *En esta actividad vas a poner a funcionar el caso de estudio de la unidad anterior y lo vas a repasar de nuevo. Mira, es muy importante que le dedique un tiempo generoso a revisar de nuevo el caso de estudio, ya que es un ejemplo muy completo y te va a ayudar a entender mejor el resto de la unidad.*

#### Describe cómo se están comunicando el micro:bit y el sketch de p5.js. ¿Qué datos envía el micro:bit?

``` py
data = "{},{},{},{}\n".format(xValue, yValue, aState,bState)
```

#### ¿Cómo es la estructura del protocolo ASCII usado?



#### Muestra y explica la parte del código de p5.js donde lee los datos del micro:bit y los transforma en coordenadas de la pantalla.

``` js
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
