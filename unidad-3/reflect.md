# Unidad 3


## 🤔 Fase: Reflect

### Parte 1: recuperación de conocimiento (Retrieval Practice)

#### Describe con tus palabras qué es una máquina de estados. ¿Cuáles son sus cuatro componentes fundamentales que has utilizado en esta unidad?



#### Explica por qué la técnica de máquina de estados es tan útil para gestionar la “concurrencia” (atender varios eventos y tareas “al mismo tiempo”) en un dispositivo con un solo hilo de ejecución como el micro:bit o en p5.js. ¿Qué problema soluciona en comparación con usar funciones como sleep()?



#### Imagina que tienes que añadir una nueva funcionalidad a la bomba: si se recibe un evento especial (por ejemplo, una combinación de botones o un comando serial) mientras la cuenta regresiva está activa, el tiempo se reduce a la mitad. ¿Cómo modificarías tu diagrama de máquina de estados para incluir este nuevo evento y acción?

Sería una flechita que va a si misma en el estado armed donde si el arreglo de los inputs es igual al de la combinación de botones entonces el count se divide por 2.

#### Explica qué es un “vector de prueba” y por qué es una herramienta crucial para verificar que una máquina de estados funciona como se espera.

Un vector de prueba es una forma de verificar el correcto funcionamiento del programa al mirar todas las interacciones o como flujos del programa, de esta forma se busca entender desde que se parte y a que se quiere llegar y mirar si se cumple a la hora de ejecutar el código

### Parte 2: reflexión sobre tu proceso (Metacognición)

#### ¿Qué parte del diseño de la bomba te resultó más desafiante: crear el diagrama de estados o traducir ese diagrama a código? ¿Por qué?

Los diagramas sin tener el código, siempre se me ha resultado más sencillo empezar a implementar gradualmente las funciones que hacer un diagrama primero y de ahí traducirlo. Casi siempre termino haciendo el código primero y el diagrama despues. En tanto a la implementación de la bomba la parte mas desafiante para mi ha sido entender como funciona el sistema de python y js para el tiempo- (utime y millis)

#### Describe un error o “bug” que encontraste al implementar tu programa. ¿Cómo te ayudó pensar en términos de estados, eventos y transiciones a identificar y solucionar el problema?

Errores pequeños como: no poner toUpperCase entonces no reconocía las inputs, tuve una confusión porque en la unidad 2 no hice estado de exploded porque no había entendido en enunciado por lo que tuve que actualizar como había planteado el programa y entenderlo mejor

#### El problema de la bomba era complejo. ¿Qué estrategia usaste para abordarlo? ¿Comenzaste con una versión simple y añadiste funcionalidades poco a poco?

Primero intenté traducir la lógica y la estructura que tenía en python a js. Luego ese mismo código lo use en la última actividad pero implementando el sistema de conectar el micro:bit, la lectura de los inputs y usar uart.write en python

#### Ahora que entiendes el patrón de máquina de estados, ¿En qué otro tipo de proyecto o sistema de entretenimiento digital crees que podrías aplicarlo?

Creo que es algo que se puede aplicar perfectamente a cualquier proyecto que funcione por medio de eventos, que son la mayoría, para entender más el funcionamiento y tener vectores de prueba definidos para poder hacer buen control de errores e ir arreglando oportunamente los problemas del programa.
