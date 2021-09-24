# Concurrencia en Smalltalk

Apuntes basados en http://books.pharo.org/booklet-ConcurrentProgramming/pdf/2020-02-18-Concur.pdf





> Smalltalk soporta la concurrencia de multiples programas , mediante procesos independientes llamados Threads o hilos, estos procesos son muy eficientes en el uso de memoria y uso del cpu como tal . los procesos en si son objetos de la clase Process en el Kernel de smalltalk. aclaramos que Process podria ser Llamada Thread (Puede que cambie en un futuro)

## Componentes de un Proceso

Los procesos tanto con los Threads se componen de metodos y atributos basicos para su uso en los programas. los mas significativos son la prioridad y su nombre 

- Prioridad: 
  La prioridad es un numero que utiliza el planificador de tareas para marcar una preferencia del proceso, este numero en smalltalk va entre 1 siendo el mas bajo hasta el 80 siendo el mas alto los procesos se agrupan en una lista de nombres  para establecer la prioridad entre 10 a 100 
- Nombre 
  Los procesos en si utilizan un nombre para identificar el objeto en ejecucion y para debug.

### Inicializar un proceso en smalltalk

La forma adecuada de iniciar un proceso es enviar un mensaje ```newProcess``` a un bloque, esto asignara el bloque a un Thread, se aclara que dicho Thread estara en estado de Listo (*suspended*) a la espera de poder ejecutarse, la forma de que el proceso entre en estado de ejecucion (*Runnable*) es mediante el mensaje ```resume``` lo cual hace consumir el tocken de ejecucion. 

Ejemplo:

```smalltalk
Transcript open.
[3 timesRepeat: [3 trace. ' ' trace ]] newProcess.
```

![1](https://github.com/danteGiuliano/Concurrencia-Smalltalk/blob/main/creacion%20de%20proceso.jpg)

al enviarle ``` self resume ``` al proceso que contiene el bloque, hara que ejecute las sentencias mostrandonos en el transcript su ejecucion, una terminado de ejecutar el proceso pasa a un estado de terminated.

### Tipos de prioridad en Smalltalk 

La prioridad en smalltalk puede utilizarse para mejorar la performance de ciertos procesos que el programador crea conveniente, dentro de la Clase ProcessorScheduler existen diferentes mensajes de clase para ajustar la prioridad de la  tarea a realizar se provee la siguiente tabla para dar un aproximado segun el tipo de tarea que se quiera hacer.

- [ ] 80 : timingPriority 
  El proceso se asemeja al tiempo real 
- [ ] 70: highIOPriority
  El proceso debe tener una alta prioridad, por ejemplo escuchar una conexion.
- [ ] 60: lowIOPriority
  El proceso requiere una prioridad del tipo input/output como teclados, puntos de acceso o servicios locales del ordenador
- [ ] 50: userInterruptPriority
  El proceso debe ejecutarse de inmediato, pero sin tener un uso excesivo del CPU.
- [ ] 40: userSchedulingPriority
  El proceso requiere una interaccion normal.
- [ ] 30: userBackgroundPriority
  El proceso corre en segundo plano.
- [ ] 20: systemBackgroundPriority
  El proceso requiere una prioridad de sistema, por ejemplo hacer chequeos de datos.
- [ ] 10: lowestPriority
  El proceso requiere la menor prioridad posible.

### Asignacion de prioridad 

La asignacion de prioridad se debe incluir en la inicializacion del proceso , el mensaje para esto es ```newProcessWith: unaPrioridad``` la prioridad puede asignarse mediante un tipo de prioridad que ofrece smalltalk o un numero entre 1-80 significando la prioridad baja-alta, si no especifica la prioridad, se le da por defecto 40 

```smalltalk
[3 timesRepeat: [3 trace. ' ' trace ]]newProcessWith:40 
es equivalente a 
[3 timesRepeat: [3 trace. ' ' trace ]]newProcessWith: (ProcessorScheduler classPool:#userBackgroundPriority )
```



### Mensaje Fork y ForkAt:

Como vimos antes la forma de crear un proceso es mediante el mensaje newProcess pero esto solo nos genera un Thread suspendido a la espera de poder ser activado mediante el mensaje resume, por eso mismo surge la implementacion del mensaje Fork, Fork es una abreviatura donde se le envia el mensaje newProcess  resume al bloque en cuestion. lo cual nos devuelve la ejecucion del proceso. 

```smalltalk
[3 timesRepeat: [3 trace. ' ' trace ]] newProcess resume 
es equivalente a 
[3 timesRepeat: [3 trace. ' ' trace ]] fork
```

lo mismo ocurre con ForkAt: siendo un equivalente de newProcessWith:

```smalltalk
[3 timesRepeat: [3 trace. ' ' trace ]] newProcessWith:30  
es equivalente a 
[3 timesRepeat: [3 trace. ' ' trace ]] forkAt:30
```

de esta forma podemos no solo, crear y asignar una prioridad. si no tambien delegar que inmediatamente se ejecute el codigo de blockCloasure.

### Prioridad del processor y yield.

prueba este algoritmo en el playground!

```smalltalk
|suba recurso |
Transcript clear open.
recurso :=0.
suba :=[recurso:=recurso+1.].
[5 timesRepeat:[suba value.recurso asString trace,' 'trace.]]fork.
[5 timesRepeat:[recurso trace asString,' 'trace.]]fork.
```

Salida @ 1 2 3 4 5 5 5 5 5 5 

que ocurre? Lo que ocurre con el planificador de procesos de  smalltalk es como gestiona los recursos de los procesos. si bien se ejecutan 2 procesos a la vez. el planificador va a dar una prioridad secuencial cuando se tenga la misma prioridad en este caso la prioridad por defecto. 

la forma de solucionar esto, es asignando una prioridad mayor al proceso que se quiera ejecutar primero, si bien esto no es lo adecuado puede funcionar, otra forma de asegurar que todos los hilos tengan la misma chance de ser ejecutados es utilizar el mensaje yield, esto lo que provoca es liberar el uso del cpu suspendiendo al proceso.

Solucion del ejercicio anterior.

```smalltalk
|suba recurso |
Transcript clear open.
recurso :=0.
suba :=[recurso:=recurso+1.].
[5 timesRepeat:[suba value.recurso asString trace,' 'trace. Processor yield]]fork.
[5 timesRepeat:[recurso trace asString,' 'trace.Processor yield]]fork.
```

Salida @ 1 1 2 2 3 3 4 4 5 5 

### Exclusion mutua 

Para los siguientes ejercicios, crearemos una clase dedntro de Pharo , que nos permita leer y escribir un archivo de texto de manera concurrente. crearemos la clase ```Buffer```

dentro tendra los siguientes metodos 
crear archivos txt 

```smalltalk
writeTxt:aStringFormat named:aNameArchive
"aStringFormat = La escritura dentro del txt (: "
|archive|
archive :=(File named:aNameArchive asFileReference fullName).
archive writeStreamDo: [ :stream | stream <<aStringFormat].
```

leer archivos txt

```smalltalk
readTxt:aFile
^((aFile readStream)next:aFile size) asString.

```





