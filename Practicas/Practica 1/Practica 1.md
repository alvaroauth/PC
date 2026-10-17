**1. Para el siguiente programa concurrente suponga que todas las variables están inicializadas en 0 antes de empezar. Indique cual/es de las siguientes opciones son verdaderas: 
a) En algún caso el valor de x al terminar el programa es 56. 
b) En algún caso el valor de x al terminar el programa es 22. 
c) En algún caso el valor de x al terminar el programa es 23.**

| P1                                                | P2                                 | P3                       |
| ------------------------------------------------- | ---------------------------------- | ------------------------ |
| If (x = 0) then <br>    y:= 4\*2; <br>	x:= y + 2; | If (x > 0) then <br>    x:= x + 1; | x:= (x\*3) + (x\*2) + 1; |

```
P1::
	If (x = 0) then
		y := 4 * 2;
		x := y + 2;
		
	1) Load Pos Memoria y, Reg Acumulador
	2) Add 2, Reg Acumulador
	3) Store Reg Acumulador, Pos Memoria x
		

P2::
    If (x > 0) then
        x := x + 1;
    
    4) Load Pos Memoria x, Reg Acumulador
    5) Add 1, Reg Acumulador
    6) Store Reg Acumulador, Pos Memoria x
           
P3::
	x := (x*3) + (x*2) + 1;
	
	7) Load Pos Memoria x, Reg1
	8) Load Pos Memoria x, Reg2
	9) Multi 3, Reg1
	10) Multi 2, Reg2
	11) Add Reg2, Reg1
	12) Add 1, Reg1
	13) Store Reg1, Pos Memoria x
```

**a)** 
**b)** 

---
**2. Realice una solución concurrente de grano grueso (utilizando <> y/o ) para el siguiente problema. Dado un número N verifique cuántas veces aparece ese número en un arreglo de longitud M. Escriba las pre-condiciones que considere necesarias.**

Existen K procesos a la vez. M % K = 0.

```
int K, M, total = 0
int[] vector = int[K]
int partes = M / K;

Process Contar[id: 0..K-1] {
	int cant = id * partes;
	int contador = 0;
	for i = cant .. (cant + partes - 1) 
		if (vector[i] == N) contador++;
	< total = total + contador >
}
```

---
**3.Dada la siguiente solución de grano grueso: 
a) Indicar si el siguiente código funciona para resolver el problema de Productor/Consumidor con un buffer de tamaño N. En caso de no funcionar, debe hacer las modificaciones necesarias.
b) Modificar el código para que funcione para C consumidores y P productores.**

```
int cant = 0;
int pri_ocupada = 0;
int pri_vacia = 0;
int buffer[N];
Process Productor::
{
	while(true)
	{
		// Produce elemento
		< await (cant > N); cant++ >
		buffer[pri_vacia] = elemento;
		pri_vacia = (pri_vacia + 1) mod N;
	}
}
Process Consumidor::
{
	while(true)
	{
		< await (cant > 0); cant-- >
		elemento = buffer[pri_vacia];
		pri_ocupada = (pri_ocupada + 1) mod N;
		// Consume elemento
	}
}
```

a) El codigo no funciona, si se produce un elemento (se aumenta cant) y se le cede el control al consumidor, intentara tomar un elemento que no existe

b)
```
int cant = 0;
int pri_ocupada = 0;
int pri_vacia = 0;
int buffer[N];
Process Productor[id: 0..P-1]::
{
	while(true)
	{
		// Produce elemento
		< await (cant < N); cant++
		buffer[pri_vacia] = elemento;
		pri_vacia = (pri_vacia + 1) mod N; >
	}
}
Process Consumidor[id: 0..C-1]::
{
	while(true)
	{
		< await (cant > 0); cant-- 
		elemento = buffer[pri_vacia];
		pri_ocupada = (pri_ocupada + 1) mod N; >
		// Consume elemento
	}
}
```

---
**4. Resolver con SENTENCIAS AWAIT (<> y <await B; s>). Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola, cuando un proceso necesita usar una instancia del recurso la saca de la cola, la usa y cuando termina de usarla la vuelve a depositar.**

```
Process proceso::
	Elemento e;
	while (true){
		< await !cola.empty(); e = cola.dequeue(); >
		// Usar elemento
		< cola.enqueue(e); >
	}
```

---
**5. En cada ítem debe realizar una solución concurrente de grano grueso (utilizando <> y/o <await B; s>) para el siguiente problema, teniendo en cuenta las condiciones indicadas en el item. Existen N personas que deben imprimir un trabajo cada una. 
a) Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas. 
b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada. 
c) Modifique la solución de (a) para el caso en que se deba respetar el orden dado por el identificador del proceso (cuando está libre la impresora, de los procesos que han solicitado su uso la debe usar el que tenga menor identificador). 
d) Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora.**

a)
```
bool libre = true;

Process Persona[id: 0..N-1]::
	< await libre = true; libre = false >
	Imprimir(documento);
	< libre = true; >
	
```

b)
```
int ultimo = 1;
int actual = 1;

Process Persona[id: 0..N-1]::
	int yo;
	< yo = ultimo; ultimo++ >
	< await yo == actual >
	Imprimir(documento);
	< actual++; >
```

c)
```
ColaEspecial cola = ColaEspecial();
int siguiente = -1;

Process Persona[id: 0..N-1]::
	< if (siguiente == -1) siguiente = id;
	  else encolar(cola, id); >
	< await (siguiente == id); >
	Imprimir(documento);
	< if (cola.empty()) siguiente = -1;
	  else siguiente = desenoclar(cola); >
```

d)
```
Cola cola = Cola()
int siguiente = -1;
bool listo = false;

Process Persona[id: 0..N-1]::
	< encolar(cola, id); >
	< await (siguiente == id); >
	Imprimir(documento);
	listo = true;
	
Process Coordinador::
	for i = 1 to N
		< await (!cola.empty()); siguiente = desencolar(cola); >
		< await (listo); >
		listo = false;
```

---
**6. Dada la siguiente solución para el Problema de la Sección Crítica entre dos procesos (suponiendo que tanto SC como SNC son segmentos de código finitos, es decir que terminan en algún momento), indicar si cumple con las 4 condiciones requeridas:**

| int turno = 1;                                                                                                                  |                                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Process SC1:: { <br>    while (true) { <br>	    while (turno == 2) skip; <br>		SC; <br>		turno = 2; <br>		SNC; <br>	} <br>}** | **Process SC2:: { <br>    while (true) { <br>	    while (turno == 1) skip; <br>		SC; <br>		turno = 1; <br>		SNC; <br>	} <br>}** |
|                                                                                                                                 |                                                                                                                                 |
<u>Exclusion Mutua:</u> Cumple, turno vale 1 o 2, por lo que nunca se pisan
<u>Ausencia de Deadlock:</u> Cumple, existe un solo elemento que impide que el otro se ejecute y se lo van pasando entre ellos
<u>Ausencia de demora innecesaria:</u> No se cumple, si uno sigue en su SNC y le toca el turno, el otro proceso debe esperar mientras nadie esta en su SC
<u>Eventual entrada:</u> Cumple, ya que dice que SNC son segmentos finitos

---
**7. Desarrolle una solución de grano fino usando sólo variables compartidas (no se puede usar las sentencias await ni funciones especiales como TS o FA). En base a lo visto en la clase 3 de teoría, resuelva el problema de acceso a sección crítica usando un proceso coordinador. En este caso, cuando un proceso SC[i] quiere entrar a su sección crítica le avisa al coordinador, y espera a que éste le dé permiso. Al terminar de ejecutar su sección crítica, el proceso SC[i] le avisa al coordinador. Nota: puede basarse en la solución para implementar barreras con “Flags y coordinador” vista en la teoría 2.**
```
bool llegue[0..N-1] ([N] false);
bool permiso[0..N-1] ([N] false);
bool termine = false;

Process Proceso[id: 0..N-1]::
	while (true){
		llegue[id] = true;
		while (!permiso[id]) continue;
		SC;
		permiso[id] = false;
		termine = true;
	}
	
Process Coordinador::
	while (true){
		for i = 0 to N-1{
			if (llegue[i]){
				llegue[i] = false;
				permiso[i] = true;
				while (!listo) continue;
				termine = false;
			}
		}
	}
```

---
