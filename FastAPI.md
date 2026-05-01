

# Introducción

Antes de empezar con FastAPI, vamos a repasar algunos conceptos de Python, ya que dicho framework se construye a partir de Python 3.8+.

Antes de empezar, hay que tener en cuenta que Python tiene soporte adicional de "type hints" (también conocidos como "type annotations").

# Type hints

Los **type hints** en Python son anotaciones de tipo introducidas oficialmente en la version 3.5 que permiten especificar explicitamente los tipos de datos esperados para variables, argumentos de funciones y valores de retorno. Aunque Python es un lenguaje de tipado dinamico y estas anotaciones no afectan la ejecucion del codigo ni se imponen en tiempo de ejecucion, sirven como sugerencias para desarrolladores, IDEs y herramientas de analisis estatico.

Su uso principal es mejorar la legibilidad, facilitar el mantenimiento y permitir la deteccion temprano de errores mediante herramientas como **mypy, pyright** o los asistentes de codigo de los editores.

Ejemplo de codigo en Python sin **type hints**:
```python
def calcular_precio_total(cantidad, precio_unitario):
    return cantidad * precio_unitario

resultado = calcular_precio_total(3, 15.5)
print(resultado)  # Salida: 46.5   
```


Ejemplo de codigo en Python con **type hints**:
```python
def calcular_precio_total(cantidad: int, precio_unitario: float) -> float:
    return cantidad * precio_unitario

resultado = calcular_precio_total(3, 15.5)
print(resultado)  # Salida: 46.5   
```

Algo muy importante a tener en cuenta es que los type hints sirven solo como informacion, ya que Python no comprueba si los datos usados coinciden con los especificados.

Es muy recomendable usar esta estrategia en nuestro desarrollo en FastAPI ya que podemos declarar todos los tipos de datos estandar en Python. Para algunos casos de uso adicionales, quizas necesitemos importar algunos cosas de la libreria estandar **typing**, por ejemplo cuando queremos declarar que algo tiene "ningun tipo", se puede usar **Any** desde **typing**.

```python
from typing import Any

def some_function(data: Any) -> None:
	print(data)
```

## Tipos Genericos

Algunos tipos de datos, pueden tener tipos de parametros entre corchetes, para definir su tipo de dato interno, por ejemplo una "lista de strings" tendria el siguiente aspecto: **list[str]**.

Estos tipos de datos que pueden tomar tipos de parametros se denominan tipos genericos o genericos.

* list
* tuple
* set
* dict

### List

Como ejemplo, vamos a definir una lista de strings.
```python
def process_items(items: list[str])-> None:
    for item in items:
        print(item)
```

" Estos tipos internos en los corchetes se llaman type parameter. En este caso, ***str*** es el tipo de parametro pasado a ***list***".

Esto significa que la variable ***items*** es una ***lista***, y que cada elemento de esta lista es un ***str***.

### Tuple and Set 

Al igual que antes, a la hora de declarar *tuple* y *set* seguiremos la misma filosofia comentada previamente

```python
def process_items(items_t: tuple[int, int, str],
					 items_s: set[bytes]) 
					 -> tuple[tuple[int, int, str], set[bytes]]:
	return items_t, items_s
```

Esto significa:
	1. La variable *items_t* es una **tuple** con 3 elementos: *int*, *int* y un *str*.
	2. La variable *items_s* es un **set**, y cada uno de sus elementos es de tipo *bytes*.


### Dict

Para definir un *dict*, necesitamos pasar dos tipos de parametros separados por comas.

El primer tipo de parametro se utilizara como las keys del diccionario.
El segundo tipo de parametro es usara para el valor de los parametros del diccionario:

```python
def process_items(prices: dict[str, float])-> None:
	for item_name, item_price in price.items():
		print(item_name)
		print(item_price)
```

Esto significa:
1. La variable *prices* es un **dict**:
	1. Las *keys* de este **dict** son de tipo *str* (digamos que es el nombre de cada item).
	2. Los *values* de este **dict** son de tipo *float* (digamos que es el precio de cada item).


### Union

Se puede declarar que una variable pueda ser de **muchos tipos**, por ejemplo, *int* o *str*.

Para definirlo tienes que usar la barra vertical (|) para separar ambos tipos.

Esto es lo que se conoce como *union*, porque la variable puede ser cualquiera en la union de esos dos tipos de datos.

```python
def process_item(item: int | str)-> None:
	print(item)
```


### None

Igual que se puede declarar que un valor puede tener cualquier tipo, como *str*, tambien se puede declarar que no tiene ningun tipo.

```python
def say_hi(name: str | None = None) ->  None:
	if name is not None:
		print(f"Hey {name}!")
	else:
		print("Hello World")
```



## Clases como tipos

Tambien puedes declarar una clase como un tipo de variable:

Supongamos que tenemos una clase *Person*, con un nombre:

```python
class Person:
	def __init__(self, name: str):
		self.name = name
		
def get_person_name(one_person: Person)-> str:
	return one_person.name
```


## Pydantic models

**Pydantic** es una libreria de Python para ejecutar validacion de datos. Se tiene que declarar la "forma" de los datos como clases con atributos, y cada atributo tiene un tipo.

Entonces se crea una instancia de cada clase con algunos valores y los validara, convertira en los tipos apropiados (si fuera el caso) y nos proporcionara un objeto con todos los datos.
Con esto obtendremos toda la ayuda que nos pueda proporcionar **IDE** con ese objeto final.

```python
from datetime import datetime
from pydantic import BaseModel

class User(BaseModel):
	id: int
	name: str = "John Doe"
	signup_ts: datetime | None = None
	friend: list[int] = []
	
external_data = {
	"id": "123",
	"signup_ts": "2017-06-01 12:22",
	"friends": [1, "2", b"3"],
}

user = User(**external_data)
print(user)
# > User id=123 name='John Doe' signup_ts=datetime.datetime(2017, 6, 1, 12, 22) friends=[1, 2, 3]
print(user.id)
# > 123
```
 
Basicamente, para lo que nos sirve esta libreria es para **validar los datos en tiempo de ejecucion (runtime)** y **convertir tipos automaticamente**.


## Type Hints con Anotaciones en Metadatos

Python tambien tiene una caracteristica que permite poner metadatos adicionales en los *type hints* usando *Annotated*. Para ello, es necesario importar *Annotated* desde *typing*.

```python
from typing import Annotated

def say_hello(name: Annotated[str, "this is just metadata"]) -> str:
	return f"Hello {name}"
```

Python en si no hace nada con estas **Annotated** y para los **IDE** y otras herramientas, el tipo de dato es *str*.

Pero tu puedes usar este espacio de *Annotated* para otorgar a **FastAPI** informacion adicional en los metadatos sobre como quieres que se comporte la aplicacion. Lo mas importante es recordar que **el primer parametro** que pasas a *Annotated* is el tipo de dato actual. El resto son solo metadatos para otras herramientas.

Por ahora, solamente necesitas saber que *Annotated* existe, y que es un estandar de Pyhton. Mas tarde se vera lo poderoso que puede llegar a ser.

"El hecho de que este sea un estandar en Python significa que aun obtendras la mejor experiencia de desarrollador en tu editor, con las herramientas que utilizas para analizar y refactorizar tu codigo, etc.

Y tambien tu codigo sera muy compatible con muchas otras herramientas y bibliotecas de Python."


## Type hints en FastAPI

FastAPI se beneficia de estos "type hints" para hacer varias cosas:

1. Soporte para IDEs.
2. Comprobacion de Tipos.

Ademas FastAPI usa la mismas definiciones para:
	- Definir requisitos: Desde solicitar parametros de ruta, parametros de consulta, cuerpos, dependencias, etc.
	- Convertir datos: Desde el solicitado hasta el tipo requerido.
	- Validar datos: provenientes de cada solicitud:
		- Generar errores automaticos que se devuelven al cliente cuando el dato es invalido.
	-Documentar la API usando OpenAPI:
		-Que luego es utilizada por las interfaces de usuario de documentacion interactiva automatica.

Esto tal vez suene abstracto, no hay que preocuparse, se vera todo esto mas adelante. Lo importante es que al utilizar tipos estandar de Python, en un solo lugar (en lugar de añadir mas clases, decoradores, etc.), FastAPI hara gran parte del trabajo por nosotros.

Ante cualquier duda, podemos visitar el siguiente recurso: https://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html

# Concurrencia y async / await

Detalles sobre la sintaxis *async def* para funciones de operacion de ruta asi como informacion general sobre el codig asincrono, la concurrencia y el paralelismo.

Si se esta usando bibliotecas de terceros que indican que tienen que ser llamadas con *await*, como por ejemplo:
```python
results = await some_library()
```

A continuacion, declara tus funciones de operacion de ruta con *async def* de la siguiente manera.
```python
@app.get('/')
async def read_results():
	results = await some_library()
	return results
```

Solamente se puede usar *await* dentro de funciones creadas con *async def*.

Si se utiliza una biblioteca de terceros que se comunica con algun elementos (una base de datos, una API, el sistema de archivos, etc.) y no es compatible con el uso de *await* (como ocurre actualmente con la mayoria de bibliotecas de bases de datos), se tienen que declarar las funciones de operacion de rutas como de costumbre, simplemente con  *def*, de la siguiente manera:
```python
@app.get('/')
def results():
	results = some_library()
	return results
```

Si tu aplicacion (por alguna razon) no tiene que comunicarse con ningun otro elemento ni esperar a que este responda, utiliza *async def*, aunque no necesites usar *await* en su interior.

Si no lo sabes, usa el normal *def*.

Nota: "Se puede combinar el uso de *def* y *async def* en nuestras funciones de operacion de ruta tanto como se necesite y definir cada una de ellas utilizando la opcion que mejor se adapte a tus necesidades. FastAPI sabra gestionarlas adecuadamente".

En cualquier caso, en cualquiea de los casos anteriores, FastAPI seguira funcionando de forma asincrona y sera extremadamente rapido.

Sin embargo, si sigues los pasos anteriores, podras realizar algunas optimizaciones de rendimiento.

## Detalles tecnicos.

Versiones modernas de Python tienen soporte para "codigo asincrono" usando algo llamado "coroutines", con **async** y **await** sintaxis.

Si analizamos en detalle la frase anteriormente dicha, tenemos:
- **Asynchronous Code**
- **async and await**
- **Coroutines**

### Asynchronous Code

Es una forma de programar donde el programa:

- No se queda bloqueado esperando.
- Hace otras cosas mientras espera.

Normalmente se trata de operaciones lentas (I/O) como:
- red (peticiones HTTP, APIs)
- bases de datos
- lectura/escritura de archivos

La forma en la que funciona es la siguiente: el programa inicia una tarea lenta (ej: leer archivo),  en vez de quedarse parado hasta terminar, realiza otras tareas y va comprobando si la tarea a terminado, y si es asi, continua con ese resultado.

De esta manera el codigo es asincrono, porque el programa no espera de forma rigida (bloqueo), y puede continuar y volver despues.

La diferencia con sincrono es la siguiente:
- Sincrono: hace una cosa -> espera-> siguiente -> espera
- Asincrono: lanza tareas -> sigue trabajando -> vuelvo cuando acaban

**El tiempo no se pierde esperando: se reutiliza haciendo otras cosas**

Otro aspecto a tener en cuenta es no mezclar que es la concurrencia y el paralelimos, ya que la prmera es la habilidad de aprovechar el tiempo en el que te bloqueas a la espera del resultado de una tarea, haciendo otras tareas, y el paralelismo es ejecutar varias tareas al mismo tiempo real (en varios nucleos)


### Async and await

Las versiones modernas de Python permiten escribir codigo asincrono de forma muy similar al codigo normal (secuencial).

Cuando tienes una operacion que tarda (por ejemplo, obtener datos), puedes hacer:
```python
burgers = await get_burgers(2)
```

**await** le dice a Python que espere a que esto termine... pero mientras tanto puedes hacer otras cosas.

Para poder usar await, la funcion dede definirse asi:
```python
async def get_burguers(number: int) -> int:
	return burguers
```

Esto indica que la funcion puede pausarse, continuar despues o permitir que el programa haga otras tareas.

Algo muy importante a tener en cuenta es lo siguiente:
- No puedes usar **await** fuera de **async def**
- No puedes llamar una funcion **async** sin **await**

```python
burguers = get_burguers(2)
```

Esto es incorrecto, deberia ser de la siguiente manera:
```python
burgers = await get_burguers(2)
```

En FastAPI se usaría de la siguiente manera
```python
@app.get(/'burgers')
async def read_burguers():
	burgers = await get_burgers(2)
	return burgers
```

FastAPI se encarga de ejecutar la primera funcion **async**, asi no hay que preocuparse por ello.

Sin FastAPI se tendrian que usar librerias como:
- asyncio
- AnyIO
Para gestionar concurrencia avanzada, tambien existe Asyncer para facilitar el uso.

### Coroutines

Una **coroutine** es simplemente el termino tecnico para el objeto que devuelve una funciona definida con **async def**

Python entiende que:
- es como una funcion
- puede empezar y terminar
- pero tambien puede *pausarse* cuando hay un await

Muchas veces, todo el uso de *async* y *await* se resume diciendo que estas usando *coroutines*

**Una coroutine es una funciona que puede "pararse y continuar" sin bloquear el programa**


## Detalles muy tecnicos

En FastAPI, las funciones de los endpoints (path operations) se ejecutan de forma distinta según estén definidas con `async def` o `def`.

- Si defines una función con `async def`, se ejecuta directamente en el event loop.
- Si la defines con `def`, FastAPI la ejecuta en un threadpool (un hilo separado) y luego espera su resultado. Esto evita que el servidor se bloquee.

A diferencia de otros frameworks, usar `def` no es más eficiente en FastAPI. De hecho, puede ser menos eficiente debido al uso de hilos. Por eso, se recomienda usar `async def` siempre que sea posible, excepto cuando se trabaja con código bloqueante (por ejemplo, librerías síncronas).

### Dependencias

Las dependencias (`Depends`) siguen el mismo comportamiento:

- `async def` → se ejecuta en el event loop  
- `def` → se ejecuta en el threadpool  

FastAPI gestiona automáticamente la mezcla de ambas.

### Sub-dependencias

Puedes combinar dependencias asíncronas y síncronas sin problemas. FastAPI se encarga de ejecutar cada una en el contexto adecuado (event loop o threadpool).

### Funciones auxiliares (helpers)

Las funciones que llamas directamente en tu código no son gestionadas por FastAPI:

- Si defines una función con `def`, se ejecuta normalmente.
- Si la defines con `async def`, debes usar `await` al llamarla.

FastAPI solo aplica su lógica de ejecución automática a los endpoints y dependencias, no a las funciones internas que tú llamas manualmente.

### Resumen

- `async def` es la opción recomendada en general.
- `def` se usa cuando hay código bloqueante.
- FastAPI gestiona automáticamente cómo ejecutar endpoints y dependencias.
- Las funciones auxiliares dependen de cómo las llames tú.


# Environment Variables

Una variable de entorno (tambien conocida como "env var") es una variable que se encuentra fuera del codigo de Python, en el sistema operativo, y que puede ser leida por tu codigo de Python (o tambien por otros programas).

Las variables de entorno pueden resultar utiles para gestionar la configuracion de las aplicaciones, como parte de la instalacion de Phyton, etc.

## Creacion y uso de "Env Vars"

Se puede crear y usar variables de entorno in la shell (terminal), sin necesidad de Python.

```windows
$Env:MY_NAME = "Wade Wilson"
echo "Hello $Env:MY_NAME"
```

La respuesta de la terminal sera la siguiente: "Hello Wade Wilson".


## Leer *env vars* en Python

Al igual que se pueden crear variables de entorno fuera de Python, en un terminal de windows, tambien se pueden leer el Python.

Si tenemos un archivo Python llamado **main.py**:
```python
import os

name = os.getenv("MY_NAME", "World")
print(f"Hello {name} from Python")
```

El segundo argumento de la funcion "os.getenv()" es el valor por defecto para devolver. Si no se define, el valor por defecto sera *None*.

Entonces, si llamamos a nuestro programa Python:
```windows
python main.py

# Como la variable de entorno no se ha definido aun, veremos lo siguiente.
Hello World from Python

# Si creamos una variable de entorno antes
$Env:MY_NAME = "Wade Wilson"

# Llamamos al programa de nuevo
$ python main.py

# Ahora se puede leer la variable de entorno y veremos
Hello Wade Wilson from Python
```

Dado que las variables de entorno se pueden definir fuera del código, pero este puede leerlas, y no es necesario almacenarlas (incorporarlas a Git) junto con el resto de los archivos, es habitual utilizarlas para configuraciones o ajustes.

También se puede crear una variable de entorno solo para una ejecución concreta de un programa, de modo que solo esté disponible para ese programa y durante el tiempo que dure su ejecución.

Para ello, se puede crear justo antes del propio programa, en la misma linea:
```windows
# Crear una variable de entorno MY_NAME para el programa
$ MY_NAME = "Wade Wilson" python main.py

# La variable puede ser leida y obtendremos lo siguiente
Hello Wade Wilson from Python

# La variable no existe despues
$ python main.py

Hello World from Python
```

## Tipos y Validacion

Estas variables de entorno solo admiten cadenas de texto, ya que son externas a Python y deben ser compatibles con otros programas y con el resto del sistema (e incluso con diferentes sistemas operativos, como Linux, Windows o macOS).

Esto significa que cualquier valor que se lea en Python desde una variable de entorno será un tipo «str», y cualquier conversión a otro tipo o validación deberá realizarse en el código.

## **PATH** Environment Varible

Existe una variable de entorno especial llamada PATH que utilizan los sistemas operativos (Linux, macOS, Windows) para localizar los programas que se van a ejecutar.

El valor de la variable PATH es una cadena larga formada por directorios separados por dos puntos : en Linux y macOS, y por un punto y coma ; en Windows.
  
Por ejemplo, la variable de entorno PATH podría tener este aspecto:
```windows
C:\Program Files\Python312\Scripts;C:\Program Files\Python312;C:\Windows\System32
```
Esto significa que el sistema buscara programas en los siguientes directorios:

- *C:\Program Files\Python312\Scripts*
- *C:\Program Files\Python312*
- *C:\Windows\System32*

Cuando se escribe un comando en la terminal, el sistema operativo busca el programa en cada uno de los directorios listados en la variable de entorno PATH.

Por ejemplo, cuando escribes *python* en el terminal, el SO buscara por el programa llamado *python* en el primer directorio de la lista. 
Si lo encuentra, lo usara y sino esta se mantendra buscando en el resto de directorios de la variable *PATH*

Al instalar Python en tu ordenador, tendras que añadir el directorio correspondiente a la variable de entorno PATH.

# Entornos Virtuales

Cuando se trabaja con proyectos de Python, probablemente se deberia usar entornos virtuales (o mecanismo similar) para aislar los paquetes que se instalan en cada proyecto.

Un entorno virtual es diferente que una variable de entorno.
Una variable de entorno es una variable en el sistema que puede ser usada por programas
Un entorno virtual es un directorio que contiene algunos archivos
## Crear un proyecto

Primero creamos un directorio donde desarrollaremos el proyecto.

"C:\Users\USUARIO\Desktop\BackEnd\Code\1. EV"

## Crear un Entorno Virtual

Cuando empiezas a trabajar en un proyecto de Python por primera vez, crear un entorno virtual dentro del proyecto. Solo se necesita hacer esto una vez por proyecto, no cada vez que se trabaje en el.

Para crear el entorno virtual, se puede usar el modulo *venv* que viene con Python
```windows
python -m venv .venv
```

Que significa el comando?
- `python`: use the program called `python`
- `-m`: call a module as a script, we'll tell it which module next
- `venv`: use the module called `venv` that normally comes installed with Python
- `.venv`: create the virtual environment in the new directory `.venv`

Ese comando crea un entorno virtual en un directorio llamado **.venv**

Se puede crear el entorno virtual en un directorio diferente, pero hay una convencion de llamarlo **.venv**

## Activar el entorno virtual

Se tiene que activar el nuevo entorno virtual para que cualquier comando de Python que se ejecute o paquete que se instale lo utilice.

Se necesita hacer esto cada vez que se abra una nueva sesion de terminal para trabajar en el proyecto.

```Windows Power Shell
.venv\Scripts\Activate.ps1
```

``` Windows Bash
source .venv/Scripts/activate
```

Cada vez que instales un nuevo paquete en ese entorno , vuelve a activarlo. De este modo, te aseguras de que, si utilizas un programa de terminal (CLI) instalado por ese paquete, utilices el de tu entorno virtual y no cualquier otro que pueda estar instalado a nivel global, probablemente con una version diferente a la que necesites.

## Comprobar si el entorno virtual esta activo.

Para comprobar si el entorno virtual esta activo, se puede usar lo siguiente:
```Windows PowerShell
Get-Command python

C:\Users\user\code\awesome-project\.venv\Scripts\python
```

Si se muestra el binario *python* en *.venv\Scripts\python*, dentro de tu proyecto (en este caso *awesome-project*), entonces funciono.

## Actualizar *pip*

Si estas usando pip para instalar paquetes (viene con Python por defecto), deberias actualizarlo a la ultima version.
Muchos errores al instalar paquetes, se arreglan con actualizar pip.

Esto se deberia hacer normalmente una vez, justo despues de crear el entorno virtual.
```Windows
python -m pip install --upgrade pip
```
Asegurate de tener activo el entorno virtual antes de hacerlo.

Algunas veces, puedes enfrentar el error ' No module name pip' cuando intentas actualizar pip. Si esto pasa, instala y actualiza pip con el siguiente comando:
```Windows
python -m ensurepip --upgrade
```

## Añadiendo **.gitignore**

