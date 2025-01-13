# Práctica 1: Simulador de Tráfico

## Detección de copias

Durante el curso se realizará control de copias de todas las prácticas,
comparando las entregas de todos los grupos de TP2. Se considera copia
la reproducción total o parcial del código de otros alumnos o cualquier
código extraı́do de Internet o de cualquier otra fuente, salvo aquellas
autorizadas explı́citamente por el profesor.

En caso de detección de copia se informará al *Comité de Actuación ante
Copias* que citará al alumno infractor y, si considera que es necesario
sancionar al alumno, propondrá una de las medidas siguientes:

-   Calificación de cero en la convocatoria de TP2 en la que se haya
    detectado la copia.

-   Calificación de cero en todas las convocatorias de TP2 del curso
    actual.

-   Apertura de expediente académico ante la *Inspección de Servicios de
    la Universidad*.

## Instrucciones generales

Las siguientes instrucciones **son estrictas**, es decir, debes
seguirlas obligatoriamente.

1.  Descárgate del Campus Virtual la plantilla del proyecto. Debes
    desarrollar la práctica usando esta plantilla.

2.  Pon los nombres de los componentes del grupo en el fichero
    "`NAMES.txt`". Cada miembro en una lı́nea separada.

3.  Debes seguir estrictamente la estructura de paquetes y clases
    sugerida por el profesor y/o descrita en el enunciado.

4.  Cuando entregues la práctica, sube un fichero **zip** del proyecto,
    incluyendo todos los subdirectorios excepto el subdirectorio
    **bin**. **Otros formatos (por ejemplo `7zip`, `rar`, etc.) no están
    permitidos**.

## Análisis y creación de datos `JSON` en Java {#sec:json}

JavaScript Object Notation[^1] (`JSON`) es un formato estándar de
fichero que utiliza texto y que permite almacenar propiedades de los
objetos utilizando pares clave-valor y arrays de tipos de datos.
Utilizaremos `JSON` para la entrada y salida del simulador. Una
estructura `JSON` es un texto estructurado de la siguiente forma:

::: center
`{ "key`$_1$`": value`$_1$`, ..., "key`$_n$`": value`$_n$` }`
:::

donde `key`$_i$ es una secuencia de caracteres (que representa una
clave) y `value`$_i$ es un valor `JSON` válido, es decir, un número, un
*string*, otra estructura `JSON`, o un array `[o`$_1$`,…,o`$_k$`]`,
donde `o`$_i$ es un valor `JSON` válido. Por ejemplo:

    {
      "type" : "new_vehicle",
      "data" : {
         "time"      : 1,
         "id"        : "v1",
         "maxspeed"  : 100,
         "class"     : 3,
         "itinerary" : ["j3", "j1", "j5", "j4"]
       }
    }

En el directorio [lib]{.sans-serif} se ha incluido una librerı́a para
analizar `JSON` y convertirlo en objetos Java fáciles de manipular (ya
se encuentra importada en el proyecto). También puedes usar esta
librerı́a para crear estructuras `JSON` y convertirlas en *strings*. Un
ejemplo de uso de esta librerı́a está disponible en el paquete
"[extra.json]{.sans-serif}".

Para comparar la salida de tu implementación (sobre los ejemplos que se
proporcionan) con la salida esperada, puedes usar el siguiente
comparador de ficheros en formato `JSON`, disponible en:
<http://www.jsondiff.com>. Además, el ejemplo que aparece en el paquete
"[extra.json]{.sans-serif}" incluye otra forma de comparar dos
estructuras `JSON`, usando directamente la librerı́a. Observa que dos
estructuras `JSON` se consideran sem'anticamente iguales si tienen el
mismo conjunto de pares clave-valor. No es necesario que las estructuras
sean sintácticamente idénticas. También suministramos un programa que
ejecuta tu práctica sobre un conjunto de ejemplos, y compara su salida
con la salida esperada --ver la
Sección [1.8](#sec:app){reference-type="ref" reference="sec:app"}.

## Introducción al simulador de tráfico

Una *simulación* permite ejecutar un modelo en un ordenador para poder
observar su comportamiento y aplicar este comportamiento a la vida real.
Las prácticas de TP2 consistirán en construir un simulador de tráfico,
que modelará *vehı́culos*, *carreteras* y *cruces*, teniendo en cuenta la
contaminación ambiental. Habrá diferentes polı́ticas en los cruces para
permitir el paso de los vehı́culos, diferentes tipos de carreteras en
función del grado de contaminación y vehı́culos con distintos
identificadores ambientales. De esta forma modelaremos una aplicación
(usando orientación a objetos) de un problema de la vida real.

Construiremos el simulador utilizando entrada/salida estándar (ficheros
y/o consola). El simulador contendrá una colección de *objetos
simulados* (vehı́culos y carreteras conectadas a través de cruces), otra
colección de *eventos* a ejecutar y un *contador de tiempo* que se
incrementará en cada paso de la simulación. Un paso de la simulación
consiste en realizar las siguientes operaciones:

1.  *Procesar los eventos*. En particular estos eventos pueden añadir
    y/o alterar el estado de los objetos simulados;

2.  *Avanzar* el estado actual de los objetos simulados atendiendo a su
    comportamiento.

3.  *Mostrar* el estado actual de los objetos simulados.

Los eventos se leen de un fichero de texto antes de que la simulación
comience. Una vez leı́dos, se inicia la simulación, que se ejecutará un
número determinado de unidades de tiempo (llamadas *ticks*) y, en cada
*tick*, se mostrará el estado de la simulación, bien en la consola o en
un fichero de texto.

## El modelo {#sec:model}

A lo largo de esta sección presentamos la lógica del simulador de
tráfico. En la Sección [1.5.1](#sec:simobj){reference-type="ref"
reference="sec:simobj"}, se muestran las diferentes clases necesarias
para modelar los cruces, carreteras y vehı́culos. En la Sección
[1.5.2](#sec:roadmap){reference-type="ref" reference="sec:roadmap"} se
describe la clase encargada de agrupar todos los objetos de la
simulación, es decir, la clase que implementa un *mapa de carreteras*.
El proceso de creación de eventos (vehı́culos, cruces, carreteras y
cambio de alguna de sus propiedades) aparece en la Sección
[1.5.3](#sec:events){reference-type="ref" reference="sec:events"}.
Finalmente, la Sección [1.5.4](#sec:sim){reference-type="ref"
reference="sec:sim"} contendrá la descripción de la clase que implementa
el simulador de tráfico, que es la clase responsable de controlar la
simulación.

### Objetos de la simulación {#sec:simobj}

Tendremos tres tipos de objetos simulados:

-   **Vehı́culos**, que viajan a través de carreteras y contaminan
    emitiendo $\mathtt{CO}_2$. Cada vehı́culo tendrá un itinerario, que
    serán todos los cruces por los que tiene que pasar.

-   **Carreteras de dirección única**, a través de las cuales viajan los
    vehı́culos, y que controlan la velocidad de los mismos para reducir
    la contaminación, etc.

-   **Cruces**, que conectan unas carreteras con otras, organizando el
    tráfico a través de semáforos. Los semáforos permiten decidir que
    vehı́culos pueden avanzar a su próxima carretera de su itinerario.
    Cada cruce tendrá asociada una colección de carreteras que llegan a
    él, a las que denominaremos *carreteras entrantes*. **Desde un cruce
    sólo se puede llegar directamente a otro cruce a través de una única
    carretera.**

Cada objeto simulado tendrá un *identificador* único y se podrá
actualizar a sı́ mismo siempre que se lo pida la simulación. Además
podrán devolver su estado en formato `JSON`. La simulación pedirá a cada
objeto que actualice su estado exactamente una vez por cada *tick* de la
simulación. Todos los objetos de la simulación extienden (directa o
indirectamente) a la siguiente clase:

    package simulator.model;

    public abstract class SimulatedObject {

      protected String _id;

      SimulatedObject(String id) {
        if ( id == null || id.length() == 9)
          throw new IllegalArgumentException("the 'id' must be a nonempty string.");
        else
          _id = id;
      }

      public String getId() {
        return _id;
      }

      @Override
      public String toString() {
        return _id;
      }

      abstract void advance(int time);
      abstract public JSONObject report();
    }

#### Vehı́culos

Habrá un único tipo de vehı́culo que implementaremos en la clase
[Vehicle]{.sans-serif}. Esta clase que extenderá a la clase
[SimulatedObject]{.sans-serif}, que se encuentra dentro del paquete
"[simulator.model]{.sans-serif}". La clase [Vehicle]{.sans-serif} debe
contener atributos (campos) para almacenar al menos la siguiente
información (recuerda que está prohibido declarar los atributos como
[public]{.sans-serif}):

-   *itinerario* (de tipo [List\<Junction\>]{.sans-serif}): una lista de
    cruces que representa el itinerario del vehı́culo. La clase
    [Junction]{.sans-serif} representa los cruces y se describe en la
    Sección [\[sec:Junction\]](#sec:Junction){reference-type="ref"
    reference="sec:Junction"}.

-   *velocidad máxima* (de tipo [int]{.sans-serif}): la velocidad máxima
    a la cual puede viajar el vehı́culo.

-   *velocidad actual* (de tipo [int]{.sans-serif}): la velocidad actual
    a la que está circulando el vehı́culo.

-   *estado* (de tipo enumerado [VehicleStatus]{.sans-serif} -- ver
    paquete "[simulator.model]{.sans-serif}"): el estado del vehı́culo,
    que puede ser *Pending* (todavı́a no ha entrado a la primera
    carretera de su itinerario), *Traveling* (circulando sobre una
    carretera), *Waiting* (esperando en un cruce), o *Arrived* (ha
    completado su itinerario).

-   *carretera* (de tipo [Road]{.sans-serif}): la carretera sobre la que
    el coche está circulando. Debe ser [null]{.sans-serif} en caso de
    que no esté en ninguna carretera. La clase [Road]{.sans-serif} se
    define en la Sección [1.5.1.2](#sec:claseRoad){reference-type="ref"
    reference="sec:claseRoad"}.

-   *localización* (de tipo [int]{.sans-serif}): la localización del
    vehı́culo en la carretera sobre la que está circulando, es decir, la
    distancia desde el comienzo de la carretera. El comienzo de la
    carretera es la localización $0$.

-   *grado de contaminación* (de tipo [int]{.sans-serif}): un número
    entre $0$ y $10$ (ambos inclusive) que se usa para calcular el
    $\mathtt{CO}_2$ emitido por el vehı́culo en cada paso de la
    simulación. Es el equivalente a los distintivos medioambientales que
    actualmente llevan los vehı́culos en el mundo real.

-   *contaminación total* (de tipo [int]{.sans-serif}): el total de
    $\mathtt{CO}_2$ emitido por el vehı́culo durante su trayectoria
    recorrida.

-   *distancia total recorrida* (de tipo [int]{.sans-serif}): la
    distancia total recorrida por el vehı́culo.

La clase [Vehicle]{.sans-serif} tiene una única constructora *package
protected*:

    Vehicle(String id, int maxSpeed, int contClass,
                 List<Junction> itinerary) {
      super(id);
      // TODO complete
    }

En esta constructora debes comprobar que los argumentos tienen valores
válidos y, en caso de que no los tengan, lanzar la excepción
correspondiente. Para hacer dicha comprobación debes tener en cuenta lo
siguiente:

::: inparaenum
[maxSpeed]{.sans-serif} tiene que ser positivo;

[contClass]{.sans-serif} debe ser un valor entre $0$ y $10$ (ambos
incluidos); y

la longitud de la lista [itinerary]{.sans-serif} es al menos $2$.
:::

Además, no se debe compartir el argumento [itinerary]{.sans-serif}, sino
que debes hacer una copia de dicho argumento en una lista de sólo
lectura, para evitar modificarlo desde fuera:

    Collections.unmodifiableList(new ArrayList<>(itinerary));

La clase [Vehicle]{.sans-serif} tiene los siguientes métodos, cuya
declaración debes respectar (recuerda que cuando no aparece modificador
de visibilidad, significa que el método es *package protected*):

-   [void setSpeed(int s)]{.sans-serif}: pone la velocidad actual al
    valor mı́nimo entre [s]{.sans-serif} y la velocidad máxima del
    vehı́culo. Lanza una excepción si [s]{.sans-serif} es negativo.

-   [void setContaminationClass(int c)]{.sans-serif}: pone el valor de
    contaminación del vehı́culo a [c]{.sans-serif}. Lanza una excepción
    si [c]{.sans-serif} no es un valor entre $0$ y $10$ (ambos
    incluidos).

-   [void advance(int time)]{.sans-serif}: si el estado del vehı́culo no
    es *Traveling*, no hace nada. En otro caso:

    (a) se actualiza su localización al valor mı́nimo entre (i) *la
        localización actual* más *la velocidad actual*; y (ii) la
        longitud de la carretera por la que está circulando.

    (b) calcula la contaminación $c$ producida utilizando la fórmula
        $c=l*f$, donde $f$ es el grado de contaminación y $l$ es la
        distancia recorrida en ese paso de la simulación, es decir, la
        nueva localización menos la localización previa. Después añade
        $c$ a la *contaminación total del vehı́culo* y también añade $c$
        al grado de contaminación de la carretera actual, invocando al
        método correspondiente de la clase [Road]{.sans-serif}.

    (c) si la localización actual (es decir la nueva) es mayor o igual a
        la longitud de la carretera, el vehı́culo entra en la cola del
        cruce correspondiente (llamando a un método de la clase
        [Junction]{.sans-serif}). Recuerda que debes modificar el estado
        del vehı́culo.

-   [void moveToNextRoad()]{.sans-serif}: mueve el vehı́culo a la
    siguiente carretera. Este proceso se hace *saliendo* de la carretera
    actual y *entrando* a la siguiente carretera de su itinerario, en la
    localización $0$. Para salir y entrar de las carreteras, debes
    utilizar el método correspondiente de la clase [Road]{.sans-serif}.
    Para encontrar la siguiente carretera, el vehı́culo debe preguntar al
    cruce en el cual está esperando (o al primero del itinerario en caso
    de que el estado del vehı́culo sea *Pending*) mediante una invocación
    al método correspondiente de la clase Junction. Observa que la
    primera vez que el vehı́culo llama a este método, el vehı́culo no sale
    de ninguna carretera ya que el vehı́culo todavı́a no ha empezado a
    circular y, que cuando el vehı́culo abandona el último cruce de su
    itinerario, entonces no puede entrar ya a ninguna carretera dado que
    ha finalizado su recorrido -- no olvides modificar el estado del
    vehı́culo.

    Este método debe lanzar una excepción si el estado de los vehı́culos
    no es *Pending* o *Waiting*. **Recuerda que el itinerario es una
    lista de cruces *de sólo lectura* y por tanto no puedes
    modificarla**. Es conveniente guardar un ı́ndice que indique el
    último cruce del itinerario que ha sido visitado por el vehı́culo.

-   [public JSONObject report()]{.sans-serif}: devuelve el estado del
    vehı́culo en el siguiente formato `JSON`:

         {
           "id" : "v1",
           "speed" : 20,
           "distance" : 60,
           "co2": 100,
           "class": 3,
           "status": "TRAVELING",
           "road" : "r4",
           "location" : 30
         }

    donde "`id`" es el identificador del vehı́culo; "`speed`" es la
    velocidad actual; "`distance`" es la distancia total recorrida por
    el vehı́culo; "`co2`" es el total de $\mathtt{CO}_2$ emitido por el
    vehı́culo; "`class`" es la etiqueta medioambiental del vehı́culo;
    "`status`" es el estado del vehı́culo que puede ser "`PENDING`",
    "`TRAVELING`", "`WAITING`" o "`ARRIVED`"; "`road`" es el
    identificador de la carretera sobre la que el vehı́culo está
    circulando; y "`location`" es su localización sobre la carretera. Si
    el estado del vehı́culo es *Pending* o *Arrived*, las claves "`road`"
    y "`location`" se deben omitir en el informe.

Además, define los siguientes *getters públicos* para consultar la
información correspondiente: [getLocation()]{.sans-serif},
[getSpeed()]{.sans-serif}, [getMaxSpeed()]{.sans-serif},
[getContClass()]{.sans-serif}, [getStatus()]{.sans-serif},
[getTotalCO2()]{.sans-serif}, [getItinerary()]{.sans-serif},
[getRoad()]{.sans-serif}.

También puedes definir *setters* siempre que sean privados. **Asegúrate
de que la velocidad del vehı́culo es $0$ cuando su estado no es
*Traveling***.

#### Carreteras {#sec:claseRoad}

En esta práctica tendremos dos tipos de carreteras. La diferencia entre
ambos tipos radica en cómo gestionan los nı́veles de contaminación.
Primero describimos la clase base [Road]{.sans-serif} (en el paquete
"[simulator.model]{.sans-serif}"), que extiende a
[SimulatedObject]{.sans-serif}. Después usaremos herencia para definir
los dos tipos de carreteras. La clase [Road]{.sans-serif} contendrá al
menos los siguientes campos o atributos:

-   *cruce origen* y *cruce destino* (ambos de tipo
    [Junction]{.sans-serif}): los cruces a los cuales la carretera está
    conectada. Circular por esa carretera es ir desde el cruce *origen*
    al cruce *destino*.

-   *longitud* (de tipo [int]{.sans-serif}): la longitud de la carretera
    (en alguna unidad para medir la distancia, por ejemplo metros).

-   *velocidad máxima* (de tipo [int]{.sans-serif}): la velocidad máxima
    permitida en esa carretera.

-   *lı́mite actual de velocidad* (de tipo [int]{.sans-serif}): un
    vehı́culo no puede circular por esa carretera a una velocidad
    superior al lı́mite establecido. Su valor inicial debe ser igual a la
    velocidad máxima.

-   *alarma por contaminación excesiva* (de tipo [int]{.sans-serif}):
    indica un lı́mite de contaminación que una vez superado impone
    restricciones al tráfico para reducir la contaminación.

-   *condiciones ambientales* (de tipo enumerado [Weather]{.sans-serif}
    -- ver el paquete "[simulator.model]{.sans-serif}"): las condiciones
    atmosféricas en la carretera. Este valor se usa para actualizar la
    velocidad de los vehı́culos, estado de contaminación, etc.

-   *contaminación total* (de tipo [int]{.sans-serif}): la contaminación
    total acumulada en la carretera, es decir, el total de
    $\mathtt{CO}_2$ emitido por los vehı́culos que circulan sobre la
    carretera.

-   *vehículos* (de tipo [List\<Vehicle\>]{.sans-serif}): una lista de
    vehı́culos que están circulando por la carretera -- **debe estar
    siempre ordenada por la localización de los vehı́culos (orden
    descendente)**. Observa que puede haber varios vehı́culos en la misma
    localización. Sin embargo, su orden de llegada a esa localización
    debe preservarse en la lista. Para eso el vehı́culo que llega el
    primero será el primero en circular -- recuerda que los algoritmos
    de ordenación de Java garantizan que elementos iguales no se
    reordenarán como resultado de la ordenación.

La clase [Road]{.sans-serif} tiene únicamente una constructora *package
protected*:

    Road(String id, Junction srcJunc, Junction destJunc, int maxSpeed, int contLimit, int length, Weather weather) {
      super(id);
      // ...
    }

La constructora debe añadir la carretera como carretera saliente a su
cruce origen, y como carretera entrante a su cruce destino. En esta
constructora debes comprobar que los argumentos tienen valores válidos
y, en otro caso, lanzar excepción. Los valores válidos son:
[maxSpeed]{.sans-serif} es positivo; [contLimit]{.sans-serif} es no
negativo; [length]{.sans-serif} es positivo; [srcJunc]{.sans-serif},
[destJunc]{.sans-serif} y [weather]{.sans-serif} son distintos de
[null]{.sans-serif}. Esta clase tiene los siguientes métodos, cuya
declaración debes respetar:

-   [void enter(Vehicle v)]{.sans-serif}: se utiliza por los vehı́culos
    para entrar a la carretera. Simplemente añade el vehı́culo a la lista
    de vehı́culos de la carretera (al final). Debe comprobar que la
    localización del vehı́culo es $0$ y que la velocidad del vehı́culo
    también es $0$. En otro caso lanzará una excepción.

-   [void exit(Vehicle v)]{.sans-serif}: lo utiliza un vehı́culo para
    abandonar la carretera. Simplemente elimina el vehı́culo de la lista
    de vehı́culos de la carretera.

-   [void setWeather(Weather w)]{.sans-serif}: pone las condiciones
    atmosféericas de la carretera al valor [w]{.sans-serif}. Debe
    comprobar que [w]{.sans-serif} no es [null]{.sans-serif} y lanzar
    una excepción en caso contrario.

-   [void addContamination(int c)]{.sans-serif}: añade [c]{.sans-serif}
    unidades de $\mathtt{CO}_2$ al total de la contaminación de la
    carretera. Tiene que comprobar que [c]{.sans-serif} no es negativo y
    lanzar una excepción en otro caso.

-   [abstract void reduceTotalContamination()]{.sans-serif}: método
    abstracto para reducir el total de la contaminación de la carretera.
    La implementación especı́fica la definirán las subclases de
    [Road]{.sans-serif}.

-   [abstract void updateSpeedLimit()]{.sans-serif}: método abstracto
    para actualizar la velocidad lı́mite de la carretera. La
    implementación especı́fica la definirán las subclases de
    [Road]{.sans-serif}.

-   [abstract int calculateVehicleSpeed(Vehicle v)]{.sans-serif}: método
    abstracto para calcular la velocidad de un vehı́culo
    [v]{.sans-serif}. La implementación especı́fica la definirán las
    subclases de [Road]{.sans-serif}.

-   [void advance(int time)]{.sans-serif}: avanza el estado de la
    carretera de la siguiente forma:

    (1) llama a [reduceTotalContamination]{.sans-serif} para reducir la
        contaminación total, es decir, la disminución de
        $\mathtt{CO}_2$.

    (2) llama a [updateSpeedLimit]{.sans-serif} para establecer el
        lı́mite de velocidad de la carretera en el paso de simulación
        actual.

    (3) recorre la lista de vehı́culos (desde el primero al último) y,
        para cada vehı́culo:

        1.  pone la velocidad del vehı́culo al valor devuelto por
            [calculateVehicleSpeed]{.sans-serif}.

        2.  llama al método [advance]{.sans-serif} del vehı́culo.

        Recuerda ordenar la lista de vehı́culos por su localización al
        final del método.

-   [public JSONObject report()]{.sans-serif}: devuelve el estado de la
    carretera en el siguiente formato `JSON`:

         {
          "id" : "r3",
          "speedlimit" : 100,
          "weather" : "SUNNY",
          "co2" : 100,
          "vehicles" : ["v1","v2",...],
         }

    donde "`id`" es el identificador de la carretera; "`speedlimit`" es
    la velocidad lı́mite actual; "`weather`" son las condiciones
    atmosféricas actuales; "`co2`" es la contaminación total; y
    "`vehicles`" es una lista de identificadores de vehı́culos que están
    circulando en la carretera (en el mismo orden en el que se han
    almacenado en la lista correspondiente).

Además, define los siguientes *getters públicos* para consultar la
información correspondiente: [getLength()]{.sans-serif},
[getDest()]{.sans-serif}, [getSrc()]{.sans-serif},
[getWeather()]{.sans-serif}, [getContLimit()]{.sans-serif},
[getMaxSpeed()]{.sans-serif}, [getTotalCO2()]{.sans-serif},
[getSpeedLimit()]{.sans-serif}, [getVehicles()]{.sans-serif}. El método
[getVehicles()]{.sans-serif} debe devolver una lista de *solo lectura*
(usando [Collections.unmodifiableList(vehicles)]{.sans-serif}). A
continuación describimos las dos clases de carreteras que debes
implementar, y que heredan de Road.

Esta clase de carreteras se utiliza para conectar ciudades y se
implementa a través de la clase [InterCityRoad]{.sans-serif}, que
extiende a [Road]{.sans-serif}. La clase debe colocarse dentro del
paquete "[simulator.model]{.sans-serif}" y ha de contener, al menos, los
siguientes métodos:

-   [reduceTotalContamination]{.sans-serif}: que reduce la contaminación
    total al valor:

    $\textsf{((100-x)*tc)/100}$

    donde $\textsf{tc}$ es la contaminación total actual y
    [x]{.sans-serif} depende de las condiciones atmosféricas: $2$ en
    caso de tiempo [SUNNY]{.sans-serif} (soleado), $3$ si el tiempo es
    [CLOUDY]{.sans-serif} (nublado), $10$ en caso de que el tiempo sea
    [RAINY]{.sans-serif} (lluvioso), $15$ si es [WINDY]{.sans-serif}
    (ventisca), y $20$ si el tiempo está [STORM]{.sans-serif}
    (tormentoso).

-   [updateSpeedLimit]{.sans-serif}: si la contaminación total excede el
    lı́mite de contaminación, entonces pone el lı́mite de la velocidad al
    $50\%$ de la velocidad máxima (es decir a
    "[maxSpeed/2]{.sans-serif}"). En otro caso pone el lı́mite de la
    velocidad a la velocidad máxima.

-   [calculateVehicleSpeed]{.sans-serif}: calcula la velocidad del
    vehı́culo como la velocidad lı́mite de la carretera. Si el tiempo es
    [STORM]{.sans-serif} lo reduce en un $20\%$ (es decir,
    "[(speedLimit\*8)/10]{.sans-serif}").

Esta clase de carreteras están dentro de las ciudades y se implementan a
través de la clase [CityRoad]{.sans-serif}, que extiende a
[Road]{.sans-serif} y debe estar dentro del paquete
"[simulator.model]{.sans-serif}". Estas carreteras son más restrictivas
cuando hay una contaminación excesiva. Nunca reducen la velocidad lı́mite
(que siempre es su máxima velocidad), pero calculan la velocidad de los
vehı́culos dependiendo del grado de contaminación. Es más, la reducción
de la contaminación no depende de las condiciones atmosféricas como
antes. El comportamiento de esta clase viene dado por los siguientes
métodos:

-   método [reduceTotalContamination]{.sans-serif}: que reduce el total
    de la contaminación en [x]{.sans-serif} unidades de $\mathtt{CO}_2$,
    donde [x]{.sans-serif} depende de las condiciones atmosféricas: $10$
    en caso de [WINDY]{.sans-serif}o [STORM]{.sans-serif}, y $2$ en otro
    caso. Asegúrate de que el total de contaminación no se vuelve
    negativo.

-   la velocidad lı́mite no cambia, siempre es la velocidad máxima.

-   método [calculateVehicleSpeed]{.sans-serif}: que calcula la
    velocidad de un vehı́culo usando la expresión
    "[((11-f)\*s)/11]{.sans-serif}", donde [s]{.sans-serif} es la
    velocidad lı́mite de la carretera y [f]{.sans-serif} es el grado de
    contaminación del vehı́culo.

#### Cruces

Los cruces en la simulación controlan las carreteras entrantes usando
semáforos, que pueden estar en *verde* o en *rojo*. Si el semáforo está
en verde, entoces se permite que los vehı́culos pasen, es decir que
abandonen el cruce y pasen a la siguiente carretera de su itinerario. Si
el semáforo está en *rojo*, entonces los vehı́culos se queden detenidos.
Cada carretera entrante tiene una cola en la cual los vehı́culos que
llegan se almacenan y, van abandonando la cola, cuando el semáforo se
pone en verde. En cada paso de la simulación puede haber únicamente una
carretera entrante al cruce que tenga un semáforo en *verde*. Tendremos
varias clases de cruces, que se diferenciarán en lo siguiente:

1.  la forma de elegir la carretera entrante que pondrá su semáforo en
    verde; y

2.  cómo eliminan los vehı́culos de la cola de la carretera entrante que
    tiene su semáforo en verde, es decir, qué vehı́culos de la cola
    pueden pasar a la siguiente carretera de su itinerario en cada paso
    de la simulación.

No vamos a utilizar herencia para definir los cruces, sino que los
cruces tendrán su propia estrategia. Las estrategias se encapsularán
usando una jerarquı́a de clases. A continuación vamos a describir como
encapsular las estrategias para cambiar las luces de los semáforos y
para eliminar los vehı́culos de las colas. Después presentaremos la clase
[Junction]{.sans-serif}, que utilizará esta jerarquı́a.

Estas estrategias nos servirán para decidir cuál de las carreteras
entrantes al cruce pondrá su semáforo en verde. Para implementar las
estrategias utilizaremos la siguiente interfaz, que debe colocarse en el
paquete "[simulator.model]{.sans-serif}":

    package simulator.model;

    public interface LightSwitchingStrategy {
      int chooseNextGreen(List<Road> roads, List<List<Vehicle>> qs, int currGreen, int lastSwitchingTime, int currTime)
    }

El método [chooseNextGreen]{.sans-serif} recibe como parámetros:

-   [roads]{.sans-serif}: la lista de carreteras entrantes al cruce.

-   [qs]{.sans-serif}: una lista de vehı́culos donde las listas internas
    de vehı́culos representan *colas*. La cola $i$-ésima corresponde a la
    cola de vehı́culos de la $i$-ésima carretera de la lista
    [roads]{.sans-serif}. Observa que usamos el tipo
    [List\<Vehicle\>]{.sans-serif} en lugar de
    [Queue\<Vehicle\>]{.sans-serif} para representar una cola, ya que la
    interfaz [Queue]{.sans-serif} en Java no garantiza ningún orden
    cuando se recorre la colección (y lo que queremos es recorrerla en
    el orden en el cual los elementos se añadieron).

-   [currGreen]{.sans-serif}: el ı́ndice (en la lista
    [roads]{.sans-serif}) de la carretera que tiene el semáforo en
    verde. El valor $-1$ se utiliza para indicar que todos los semáforos
    están en rojo.

-   [lastSwitchingTime]{.sans-serif}: el paso de la simulación en el
    cual el semáforo para la carretera [currGreen]{.sans-serif} se
    cambió de *rojo* a *verde*. Si [currGreen]{.sans-serif} es $-1$
    entonces es el último paso en el cual todos cambiaron a rojo.

-   [currTime]{.sans-serif}: el paso de simulación actual.

El método devuelve el ı́ndice de la carretera (en la lista
[roads]{.sans-serif}) que tiene que poner su semáforo a verde -- si es
el misma que [currGreen]{.sans-serif}, entonces, el cruce no considerará
el cambio. Si devuelve $-1$ significa que todos deberı́an estar en rojo.

Tenemos dos estrategias para cambiar los semáforos de color, que son
[RoundRobinStrategy]{.sans-serif} y [MostCrowdedStrategy]{.sans-serif}
(colocadas en el paquete "[simulator.model]{.sans-serif}"). Ambas
reciben en la constructora un parámetro [timeSlot]{.sans-serif} (de tipo
[int]{.sans-serif}), que representa el número de "ticks" consecutivos
durante los cuales la carretera puede tener el semáforo en verde. A
continuación definimos el comportamiento de ambas estrategias.

La estrategia [RoundRobinStrategy]{.sans-serif} se comporta como sigue:

1.  si la lista de carreteras entrantes es vacı́a, entonces devuelve
    $-1$.

2.  si los semáforos de todas las carreteras entrantes están rojos (es
    decir, [currGreen]{.sans-serif} es $-1$), entonces pone en verde el
    primer semáforo de la lista [roads]{.sans-serif} (es decir, devuelve
    $0$).

3.  si "[currTime-lastSwitchingTime \< timeSlot]{.sans-serif}", deja los
    semáforos tal cual están (es decir, devuelve
    [currGreen]{.sans-serif}).

4.  devuelve [currGreen+1]{.sans-serif} modulo la longitud de la lista
    [roads]{.sans-serif} (es decir, el ı́ndice de la siguiente carretera
    entrante, recorriendo la lista de forma circular).

La estrategia [MostCrowdedStrategy]{.sans-serif} se comporta como sigue:

1.  si la lista de carreteras entrantes es vacı́a, entonces devuelve
    $-1$.

2.  si todos los semáforos de las carreteras entrantes están en rojo,
    pone verde el semáforo de la carretera entrante con la cola más
    larga, empezando la búsqueda (en [qs]{.sans-serif}) desde la
    posición $0$. Si hay más de una carretera entrante con la misma
    longitud máxima de cola, entonces coge la primera que encuentra
    durante la búsqueda.

3.  si "[currTime-lastSwitchingTime \< timeSlot]{.sans-serif}", entonces
    deja los semáforos tal cual están (es decir devuelve
    [currGreen]{.sans-serif}).

4.  pone a verde el semáforo de la carretera entrante con la cola más
    larga, realizando una búsqueda circular (en [qs]{.sans-serif}) desde
    la posición [currGreen+1]{.sans-serif} modulo el número de
    carreteras entrantes al cruce. Si hay más de una carretera cuyas
    colas tengan el mismo tamaño maximal, entonces coge la primera que
    encuentra durante la búsqueda. Observa que podrı́a devolver
    [currGreen]{.sans-serif}.

Estas estrategias eliminan vehı́culos de las carreteras entrantes cuyo
semáforo esté a verde. Se modelan a través de la interfaz (que debes
colocar en el paquete "[simulator.model]{.sans-serif}"):

    package simulator.model;

    public interface DequeuingStrategy {
      List<Vehicle> dequeue(List<Vehicle> q);
    }

La interfaz tiene un único método que devuelve una lista de vehı́culos
(extraı́dos de [q]{.sans-serif}) a los que habrá que solicitar que se
muevan a sus siguientes carreteras. El método no debe modificar la cola
[q]{.sans-serif}, ni pedir a los vehı́culos que se muevan, ya que esta
tarea pertenece a la clase [Junction]{.sans-serif}. Implementaremos dos
estrategias para sacar elementos de la cola (que situaremos en el
paquete "[simulator.model]{.sans-serif}"):

-   [MoveFirstStrategy]{.sans-serif} devuelve una lista que incluye el
    primer vehı́culo de [q]{.sans-serif}.

-   [MoveAllStrategy]{.sans-serif} devuelve la lista de todos los
    vehı́culos que están en [q]{.sans-serif} (no devuelve
    [q]{.sans-serif}). El orden debe ser el mismo que cuando se itera
    [q]{.sans-serif}.

[]{#sec:Junction label="sec:Junction"} La funcionalidad de un cruce se
implementa a través de la clase [Junction]{.sans-serif}, que extiende a
[SimulatedObject]{.sans-serif}, y que debe estar colocada en el paquete
"[simulator.model]{.sans-serif}". La clase [Junction]{.sans-serif}
contiene al menos los siguientes atributos (que no pueden ser públicos):

-   *lista de carreteras entrantes* (de tipo
    [List\<Road\>]{.sans-serif}): una lista de todas las carreteras que
    entran al cruce, es decir, el cruce es su destino.

-   *mapa de carreteras salientes* (de tipo
    [Map\<Junction,Road\>]{.sans-serif}): un mapa de carreteras
    salientes, es decir, si [(j,r)]{.sans-serif} es un par clave-valor
    del mapa, entonces el cruce está conectado al cruce [j]{.sans-serif}
    a través de la carretera [r]{.sans-serif}. El mapa se usa para saber
    qué carretera seleccionar para llegar al cruce [j]{.sans-serif}.

-   *lista de colas* (de tipo
    [List\<List\<Vehicle$\sf >>$]{.sans-serif}): una lista de colas para
    las carreteras entrantes -- la cola $i$-ésima (representada como
    [List\<Vehicle\>]{.sans-serif}) corresponde a la $i$-ésima carretera
    en la *lista de carreteras entrantes*. Se recomienda guardar un mapa
    de "carretera-cola" (de tipo
    [Map\<Road,List\<Vehicles\>]{.sans-serif}) para hacer la búsqueda,
    en la cola de una carretera dada, de forma eficiente.

-   *ı́ndice del semáforo en verde* (de tipo [int]{.sans-serif}): el
    ı́ndice de la carretera entrante (en la lista de carreteras
    entrantes) que tiene el semáforo en verde. El valor $-1$ se usa para
    indicar que todas las carreteras entrantes tienen su semáforo en
    rojo.

-   *último paso de cambio de semáforo* (de tipo [int]{.sans-serif}): el
    paso en el cual el *ı́ndice del semáforo* en verde ha cambiado de
    valor. Su valor inicial es $0$.

-   *estragegia de cambio de semáforo* (de tipo
    [LightSwitchingStrategy]{.sans-serif}): una estrategia para cambiar
    de color los semáforos.

-   *estrategia para extraer elementos de la cola* (de tipo
    [DequeuingStrategy]{.sans-serif}): una estrategia para eliminar
    vehı́culos de las colas.

-   *coordenadas x e y* (de tipo [int]{.sans-serif}): coordenadas que se
    usarán para dibujar el cruce en la próxima práctica.

La clase [Junction]{.sans-serif} tiene únicamente la constructora
*package protected*:

    Junction(String id, LightSwitchStrategy lsStrategy, DequeingStrategy dqStrategy, int xCoor, int yCoor) {
      super(id);
      // ...
    }

Observa que la constructora recibe las estrategias como parámetros. Debe
comprobar que [lsStrategy]{.sans-serif} y [dqStrategy]{.sans-serif} no
son [null]{.sans-serif}, y que [xCoor]{.sans-serif} y
[yCoor]{.sans-serif} no son negativos, y lanzar una excepción en caso
contrario.

La clase [Junction]{.sans-serif} tiene los siguientes métodos (debes
respetar los modificadores de visibilidad tal cual se describen):

-   [void addIncommingRoad(Road r)]{.sans-serif}: añade [r]{.sans-serif}
    al final de la lista de carreteras entrantes, crea una cola (una
    instancia de [LinkedList]{.sans-serif} por ejemplo) para
    [r]{.sans-serif} y la añade al final de la lista de colas. Además,
    si has usado un mapa carretera-cola, entonces debes añadir el
    correspondiente par al mapa. Debes comprobar que la carretera
    [r]{.sans-serif} es realmente una carretera entrante, es decir, su
    cruce destino es igual al cruce actual y, lanzar una excepción en
    caso contrario.

-   [void addOutGoingRoad(Road r)]{.sans-serif}: añade el par
    [(j,r)]{.sans-serif} al mapa de carreteras salientes, donde
    [j]{.sans-serif} es el cruce destino de la carretera
    [r]{.sans-serif}. Tienes que comprobar que ninguna otra carretera va
    desde [this]{.sans-serif} al cruce [j]{.sans-serif} y, que la
    carretera [r]{.sans-serif}, es realmente una carretera saliente al
    cruce actual. En otro caso debes lanzar una excepción.

-   [void enter(Vehicle v)]{.sans-serif}: añade el vehı́culo
    [v]{.sans-serif} a la cola de la carretera [r]{.sans-serif}, donde
    [r]{.sans-serif} es la carretera actual del vehı́culo
    [v]{.sans-serif}.

-   [Road roadTo(Junction j)]{.sans-serif}: devuelve la carretera que va
    desde el cruce actual al cruce [j]{.sans-serif}. Para esto debes
    buscar en el mapa de carreteras salientes.

-   [void advance(time)]{.sans-serif}: avanza el estado del cruce como
    sigue:

    1.  utiliza la *estrategia de extracción de la cola* para calcular
        la lista de vehı́culos que deben avanzar y despúes les pide a los
        vehı́culos que se muevan a sus siguientes carreteras,
        eliminándolos de las colas correspondientes.

    2.  utiliza la *estrategia de cambio de semáforo* para calcular el
        ı́ndice de la siguiente carretera a la que hay que poner su
        semáforo en verde. Si es distinto del ı́ndice actual, entonces
        cambia el valor del ı́ndice al nuevo valor y pone el último paso
        de cambio de semáforo al paso actual (es decir, el valor del
        parámetro [time]{.sans-serif}).

-   [public JSONObject report()]{.sans-serif}: devuelve el estado del
    cruce en el siguiente formato `JSON`:

         {
          "id" : "j3",
          "green" : "r1",
          "queues" : [Q1,Q2,....]
         }

    donde "`id`" es el identificador del cruce; "`green`" es el
    identificador de la carretera con el semáforo en verde ("`none`" si
    todos están en rojo); y "`queues`" es la lista de colas de las
    carreteras entrantes, donde cada `Qi` tiene el siguiente formato
    `JSON`:

         {
          "road"  : "r3",
          "vehicles" : ["v1","v2",....]
         }

    donde "`road`" es el identificador de la carretera y "`vehicles`" es
    la lista de vehı́culos en el orden en que aparecen en la cola (el
    orden debe ser el mismo que el usado para recorrerla).

### Mapa de carreteras {#sec:roadmap}

El propósito de esta clase es agrupar todos los objetos de la
simulación. Esto facilita el trabajo del simulador. Se implementa a
través de la clase [RoadMap]{.sans-serif} que debe estar colocada dentro
del paquete "[simulator.model]{.sans-serif}". La clase
[RoadMap]{.sans-serif} tiene al menos los siguientes atributos, que no
pueden ser públicos:

-   *lista de cruces* de tipo [List\<Junction\>]{.sans-serif}.

-   *lista de carreteras* de tipo [List\<Road\>]{.sans-serif}.

-   *lista de vehı́culos* de tipo [List\<Vehicle\>]{.sans-serif}.

-   *mapa de cruces* de tipo [Map\<String,Junction\>]{.sans-serif}: un
    mapa de identificadores de cruces a los correspondientes cruces.

-   *mapa de carreteras* de tipo [Map\<String,Road\>]{.sans-serif}: un
    mapa de identificadores de carreteras a las correspondientes
    carreteras.

-   *mapa de vehı́culos* de tipo [Map\<String,Vehicle\>]{.sans-serif}: un
    mapa de identificadores de vehı́culos a los correspondientes
    vehı́culos.

Recuerda tener actualizadas las listas y los mapas para usar los mapas
en pro de la eficiencia de la búsqueda de un objeto; y usar las listas
para recorrer los objetos en el mismo orden en el cual han sido
añadidos.

La clase [RoadMap]{.sans-serif} tiene una única constructora *package
protected* por defecto, para inicializar los atributos mencionados a sus
valores por defecto. Además contiene los siguientes métodos (que deben
tener los modificadores de visibilidad descritos abajo):

-   [void addJunction(Junction j)]{.sans-serif}: añade el cruce
    [j]{.sans-serif} al final de la lista de cruces y modifica el mapa
    correspondiente. Debes comprobar que no existe ningún otro cruce con
    el mismo identificador.

-   [void addRoad(Road r)]{.sans-serif}: añade la carretera
    [r]{.sans-serif} al final de la lista de carreteras y modifica el
    mapa correspondiente. Debes comprobar que se cumplen lo siguiente:
    : (i) no existe ninguna otra carretera con el mismo identificador;
    y (ii) los cruces que conecta la carretera existen en el mapa de
    carreteras. En caso de que no se cumplan el método lanza una
    excepcion.

-   [void addVehicle(Vehicle v)]{.sans-serif}: añade el vehı́culo
    [v]{.sans-serif} al final de la lista de vehı́culos y modifica el
    mapa de vehı́culos en concordancia. Debes comprobar que los
    siguientes puntos se cumplen: (i) no existe ningún otro vehı́culo con
    el mismo identificador; y (ii) el itinerario es válido, es decir,
    existen carreteras que conecten los cruces consecutivos de su
    itinerario. En caso de que no se cumplan (i) y (ii), el método debe
    lanzar una excepción.

-   [public Junction getJunction(Srting id)]{.sans-serif}: devuelve el
    cruce con identificador [id]{.sans-serif}, y [null]{.sans-serif} si
    no existe dicho cruce.

-   [public Road getRoad(Srting id)]{.sans-serif}: devuelve la carretera
    con identificador [id]{.sans-serif}, y [null]{.sans-serif} si no
    existe dicha carretera.

-   [public Vehicle getVehicle(Srting id)]{.sans-serif}: devuelve el
    vehı́culo con identificador [id]{.sans-serif}, y [null]{.sans-serif}
    si no existe dicho vehı́culo.

-   [public List\<Junction\> getJunctions()]{.sans-serif}: devuelve una
    versión de *solo lectura* de la lista de cruces.

-   [public List\<Roads\> getRoads()]{.sans-serif}: devuelve una versión
    de *solo lectura* de la lista de carreteras.

-   [public List\<Vehicle\> getVehicles()]{.sans-serif}: devuelve una
    versión de *solo lectura* de la lista de vehı́culos.

-   [void reset()]{.sans-serif}: limpia todas las listas y mapas.

-   [public JSONObject report()]{.sans-serif}: devuelve el estado del
    mapa de carreteras en el siguiente formato `JSON`:

           {
             "junctions" : [J1Report,J2Report,...],
             "road" : [R1Report,R2Report,...],
             "vehicles" : [V1Report,V2Report,...],
           }

donde `JiReport`, `RiReport` y `ViReport`, son los informes de los
correspondientes objetos de la simulación. El orden en las listas `JSON`
debe ser el mismo que el orden de las listas correspondientes de
[RoadMap]{.sans-serif}.

### Eventos {#sec:events}

Los eventos nos permiten inicializar e interactuar con el simulador,
añadiendo vehı́culos, carreteras y cruces; cambiando las condiciones
atmosféricas de las carreteras; y cambiando el nivel de contaminación de
los vehı́culos. Cada evento tiene un tiempo en el cual debe ser
ejecutado. En cada tick $t$, el simulador ejecuta todos los eventos
correspondientes al paso $t$, en el orden en el cual fueron añadidos a
la cola de eventos. Primero vamos a definir una clase abstracta
[Event]{.sans-serif} (dentro del paquete
"[simulator.model]{.sans-serif}") para modelar un evento:

    package simulator.model;

    public abstract class Event implements Comparable<Event> {

      protected int _time;

      Event(int time) {
        if ( time < 1 )
          throw new IllegalArgumentException("Invalid time: "+time);
        else
          _time = time;
      }

      int getTime() {
        return _time;
      }

      @Override
      public int compareTo(Event o) {
        // TODO complete the method to compare events according to their _time
      }

      abstract void execute(RoadMap map);
    }

El campo "[\_time]{.sans-serif}" es el tiempo (o paso) en el cual este
evento tiene que ser ejecutado, y el método [execute]{.sans-serif} es el
método al que el simulador llama para ejecutar el evento. La
funcionalidad de este método se define en las subclases.

En lo que sigue vamos a describir los tipos de eventos que existen en el
simulador, todos ellos extenderán a la clase [Event]{.sans-serif} y
deben estar colocados dentro del paquete
"[simulator.model]{.sans-serif}". La constructora de cada evento recibe
algunos datos para ejecutar una operación cuando sea requerido, que se
almacena en los campos y se usa en el método [execute]{.sans-serif} para
ejecutar la funcionalidad correspondiente.

#### Evento "New Junction"

Este evento se implementa en la clase [NewJunctionEvent]{.sans-serif},
que extiende a [Event]{.sans-serif}. Tiene la siguiente constructora:

    public NewJunctionEvent(int time, String id, LightSwitchingStrategy lsStrategy, DequeuingStrategy dqStrategy, int xCoor, int yCoor) {
      super(time);
      // ...
    }

El método [execute]{.sans-serif} de este evento crea el cruce
correspondiente y lo añade al mapa de carreteras (el parámetro de
[execute]{.sans-serif}).

#### Evento "New Road"

Existen dos eventos para crear carreteras:
[NewCityRoadEvent]{.sans-serif} y [NewInterCityRoadEvent]{.sans-serif}.
Cada evento tiene una constructora de la forma:

    public NewCityRoadEvent(int time, String id, String srcJun, String destJunc, int length, int co2Limit, int maxSpeed, Weather weather) {
      super(time);
      // ...
    }

    public NewInterCityRoadEvent(int time, String id, String srcJun, String destJunc, int length, int co2Limit, int maxSpeed, Weather weather) {
      super(time);
      // ...
    }

El método [execute]{.sans-serif} de estos eventos crea una carretera y
la añade al mapa de carreteras. Estos dos eventos tienen mucho en común,
y por lo tanto tendrás que duplicar código. Después de implementarlos y
comprobar dichos eventos, refactorizalos incorporando una super clase
[NewRoadEvent]{.sans-serif} que incluye las partes comunes a ambas.

#### Evento "New Vehicle"

Este evento se implementa en la clase [NewVehicleEvent]{.sans-serif}.
Tiene la siguiente constructora:

    public NewVehicleEvent(int time, String id, int maxSpeed, int contClass, List<String> itinerary) {
      super(time);
      // ...
    }

El método [execute]{.sans-serif} de este evento crea un vehı́culo en
función de sus argumentos y lo añade al mapa de carreteras. Después
llama a su método [moveToNext]{.sans-serif} para comenzar su viaje.

#### Evento "Set Weather"

Este evento se implementa en la clase [SetWeatherEvent]{.sans-serif}.
Tiene como constructora a:

    public SetWeatherEvent(int time, List<Pair<String,Weather>> ws) {
      super(time);
      // ...
    }

Se debe comprobar que [ws]{.sans-serif} no es [null]{.sans-serif} y
lanzar una excepción en caso contrario. El método [execute]{.sans-serif}
recorre la lista [ws]{.sans-serif}, y para cada elemento
[w]{.sans-serif} pone las condiciones atmosféricas de la carretera con
identificador [w.getFirst()]{.sans-serif} a [w.getSecond()]{.sans-serif}
(ver el paquete "[simulator.misc]{.sans-serif}" para el código de la
clase [Pair]{.sans-serif}). Debe lanzar una excepción si la carretera no
existe en el mapa de carreteras.

#### Event "Set Contamination Class"

Este evento se implementa en la clase [SetContClassEvent]{.sans-serif}.
Tiene como constructora:

    public SetContClassEvent(int time, List<Pair<String,Integer> cs)  {
      super(time);
      // ...
    }

Debe comprobar que [cs]{.sans-serif} no es [null]{.sans-serif} y lanzar
una excepción en caso contrario. El método [execute]{.sans-serif}
recorre la lista [cs]{.sans-serif}, y para cada elemento de
[c]{.sans-serif}, pone el estado de contaminación del vehı́culo con
identificador [c.getFirst()]{.sans-serif} a
[c.getSecond()]{.sans-serif}. Debe lanzar una excepción si el vehı́culo
no existe en el mapa de carreteras.

### La clase "TrafficSimulator" {#sec:sim}

La clase del simulador es la única responsable de ejecutar la
simulación. Se implementa en la clase [TrafficSimulator]{.sans-serif}
dentro del paquete "[simulator.model]{.sans-serif}". Esta clase tiene al
menos los siguientes atributos que no pueden ser públicos:

-   *mapa de carreteras* (de tipo [RoadMap]{.sans-serif}): un mapa de
    carreteras en el cual se almacenan todos los objetos de la
    simulación.

-   *lista de eventos* (de tipo [List\<Event\>]{.sans-serif}): una lista
    de eventos a ejecutar. La lista está ordenada por el tiempo de los
    eventos. Si dos eventos tienen el mismo tiempo, el que fue añadido
    antes irá el primero en la lista -- para garantizar este uso, la
    clase [SortedArrayList]{.sans-serif} se explicará en clase, y se
    adjuntará su código en el paquete "[simulator.misc]{.sans-serif}".

-   *tiempo (paso) de la simulación* (de tipo [int]{.sans-serif}): el
    paso de la simulación, que inicialmente será $0$.

La clase [TrafficSimulator]{.sans-serif} tiene sólo una constructora
pública por defecto, que inicializa los campos a sus valores por
defecto. Además contendrá los siguientes métodos (debes mantener los
modificadores de visibilidad como se describen a continuación):

-   [public void addEvent(Event e)]{.sans-serif}: añade el evento
    [e]{.sans-serif} a la lista de eventos. Recuerda que la lista de
    eventos tiene que estar ordenada como se describió anteriormente.

-   [public void advance()]{.sans-serif}: avanza el estado de la
    simulación de la siguiente forma (**el orden de los puntos que
    aparecen a continuación es muy importante!**):

    1.  incrementa el tiempo de la simulación en $1$.

    2.  ejecuta todos los eventos cuyo tiempo sea el tiempo actual de la
        simulación y los elimina de la lista.

    3.  llama al método [advance]{.sans-serif} de todos los cruces.

    4.  llama al método [advance]{.sans-serif} de todas las carreteras.

-   [public void reset()]{.sans-serif}: limpia el *mapa de carreteras* y
    la *lista de eventos*, y pone el *tiempo de la simulación* a $0$.

-   [public JSONObject report()]{.sans-serif}: devuelve el estado del
    simulador en el siguiente formato `JSON`:

           {
             "time"  : 3,
             "state" : {
                         "junctions" : [...],
                         "road" : [...],
                         "vehicles" : [...]
                       }
           }

    donde "`time`" es el *tiempo de la simulación* actual, y "`state`"
    es lo que devuelve el método [report()]{.sans-serif} del *mapa de
    carreteras*.

## Controlador {#sec:controller}

Ahora que hemos definido las diferentes clases que forman parte de la
lógica del simulador, podemos empezar a testearlo escribiendo un método
que cree algunos eventos, los añada al simulador y después llame al
método [advance]{.sans-serif} del simulador varias veces para ejecutar
la simulación. Aunque esto es adecuado para testear la aplicación, no es
fácil para los usuarios utilizarla. La misión del controlador es ofrecer
una manera sencilla de utilizar la aplicación. En particular permite
cargar los eventos de ficheros de texto, y crear de forma automática los
eventos para añadirlos al simulador, ejecutando la simulación un número
determinado de pasos.

Vamos a usar estructuras `JSON` para describir eventos, y utilizaremos
factorı́as para parsear estas estructuras y transformarlas en objetos de
la simulación. En la
Sección [1.6.1](#sec:factories){reference-type="ref"
reference="sec:factories"} describimos las factorı́as necesarias para
facilitar la creación de eventos a partir de las estructuras `JSON`; y
en la Sección [1.6.2](#sec:controllerc){reference-type="ref"
reference="sec:controllerc"} describiremos el controlador, que es la
clase que permite cargar eventos desde un [InputStream]{.sans-serif} y
ejecutar el simulador un número concreto de pasos.

### Factorı́as {#sec:factories}

Como tenemos varias factorı́as, vamos a utilizar genéricos de Java para
evitar la duplicación de código. Pasamos ahora a mostrar cómo
implementar las factorı́as paso a paso. Todas las clases e interfaces
deben colocarse dentro del paquete "[simulator.factories]{.sans-serif}".
Modelamos una factorı́a a través de una interfaz genérica
[Factory\<T\>]{.sans-serif}:

    package simulator.factories;

    public interface Factory<T> {
      public T createInstance(JSONObject info);
    }

El método [createInstance]{.sans-serif} recibe como parámetro una
estructura `JSON` que describe el objeto a crear, y devuelve una
instancia de la clase correspondiente -- una instancia de un subtipo de
[T]{.sans-serif}. Si no reconoce la información contenida en
[info]{.sans-serif}, debe lanzar una excepción.

Para nuestros propósitos, necesitamos que la estructura `JSON` que se
pasa como parámetro a [createInstance]{.sans-serif}, contenga dos
claves:

-   *type*, que es un string que describe el tipo del objeto que se va a
    crear; y

-   *data*, que es una estructura `JSON` que incluye toda la información
    necesaria para crear la instancia. Por ejemplo lo que hay que pasar
    a la constructora correspondiente.

Existen muchas formas de definir una factorı́a. Para nuestra aplicación,
utilizaremos lo que se conoce como *builder based factory*, que permite
extender una factorı́a con más opciones sin necesidad de modificar su
código. El elemento básico en una *builder based factory* es el
*builder*, que es una clase capaz de crear una instancia de un tipo
especı́fico. Podemos modelarla como una clase genérica
[Builder\<T\>]{.sans-serif}:

    package simulator.factories;

    public abstract class Builder<T> {
      protected String _type;

      public Builder(String type) {
        if ( type == null )
          throw new IllegalArgumentException("Invalid type: "+type);
        else
          _type = type;
      }

      public T createInstance(JSONObject info) {

        T b = null;

        if (_type != null && _type.equals(info.getString("type"))) {
          b = createTheInstance(
                 info.has("data") ? info.getJSONObject("data") : new JSONObject());
        }

        return b;
      }

      protected abstract T createTheInstance(JSONObject data);
    }

Como se puede observar, su método [createInstance]{.sans-serif} recibe
un objeto `JSON`, y si tiene clave "`type`" cuyo valor es igual al campo
[\_type]{.sans-serif}, llama al método abstracto
[createTheInstance]{.sans-serif} con el valor de la clave "`data`" para
crear el objeto actual. En otro caso devuelve [null]{.sans-serif} para
indicar que es incapaz de reconocer la estructura `JSON`. Las clases que
extienden a [Builder\<T\>]{.sans-serif} son las responsables de asignar
un valor a [\_type]{.sans-serif} llamando a la constructora de la clase
[Builder]{.sans-serif}, y también de definir el método
[createTheInstance]{.sans-serif} para crear la instancia. Más tarde
veremos algunos "builders" que necesitamos, pero primero vamos a
describir cómo se pueden usar estos "builders" para crear una factorı́a.

Una *builder based factory* es una clase que tiene una lista de
"builders", y cuando se le pide que cree un objeto a partir de una
estructura `JSON`, recorre todos los builders hasta que encuentra uno
que sea capaz de crearlo.

    package simulator.factories;

    public class BuilderBasedFactory<T> implements Factory<T> {

      private List<Builder<T>> _builders;

      BuilderBasedFactory(List<Builder<T>> builders) {
        _builders = new ArrayList<>(builders);
      }

      @Override
      public T createInstance(JSONObject info) {
        if (info != null) {
          for (Builder<T> bb : _builders) {
            T o = bb.createInstance(info);
            if (o != null) return o;
          }
        }

        throw new IllegalArgumentException("Invalid value for createInstance: "+info);
      }
    }

Observa que la lista de "builders" se le pasa a la constructora por
parámetro, lo que significa que podemos extender la factorı́a añadiendo
más "builders" a la lista.

A continuación describimos las tres factorı́as que necesitamos en esta
práctica. Los "builders" deben devolver [null]{.sans-serif} (o lanzar la
correspondiente excepción) si algún dato no aparece en la estructura
`JSON` o bien hay alguna clave inválida.

#### Factorı́a para las estrategias de cambio de semáforo

Para esta factorı́a necesitamos dos "builders",
[RoundRobinStrategyBuilder]{.sans-serif} y
[MostCrowdedStrategyBuilder]{.sans-serif}, ambos extendiendo a
[Builder\<LightSwitchingStrategy\>]{.sans-serif}, ya que crean
instancias de las clases que implementan a
[LightSwitchingStrategy]{.sans-serif}.

La clase [RoundRobinStrategyBuilder]{.sans-serif} crea una instancia de
[RoundRobinStrategy]{.sans-serif} a partir de la siguiente estructura
`JSON`:

    {
       "type" : "round_robin_lss",
       "data" : {
                  "timeslot" : 5
                }
    },

La clave "`timeslot`" es opcional, y su valor por defecto es $1$.

La clase [MostCrowdedStrategyBuilder]{.sans-serif} crea una instancia de
[MostCrowdedStrategy]{.sans-serif} a partir de la siguiente estructura
`JSON`:

    {
       "type" : "most_crowded_lss",
       "data" : {
                  "timeslot" : 5
                }
    }

La clave "`timeslot`" es opcional y su valor por defecto es $1$.

En las dos estructuras anteriores, la clave "`timeslot`" es opcional,
con un valor por defecto de $1$. Una vez que se definan las clases
anteriores, se pueden usar para crear la factorı́a correspondiente de la
siguiente forma:

    List<Builder<LightSwitchingStrategy>> lsbs = new ArrayList<>();
    lsbs.add( new RoundRobinStrategyBuilder() );
    lsbs.add( new MostCrowdedStrategyBuilder() );
    Factory<LightSwitchingStrategy> lssFactory = new BuilderBasedFactory<>(lsbs);

Esta factorı́a se usará más tarde para parsear las estrategias asociadas
a los cruces (ver el "builder" para un cruce que se explica más abajo).

#### Factorı́a para definir las estrategias de extracción de la cola

Para esta factorı́a necesitamos dos "builders",
[MoveFirstStrategyBuilder]{.sans-serif} y
[MoveAllStrategyBuilder]{.sans-serif}, ambas extendiendo a
[Builder\<DequeuingStrategy\>]{.sans-serif}, ya que ambos "builders"
necesitan crear instancias de las clases que implementan a
[DequeuingStrategy]{.sans-serif}.

La clase [MoveFirstStrategyBuilder]{.sans-serif} crea una instancia de
[MoveFirstStrategy]{.sans-serif} a partir de la siguiente estructura
`JSON`:

    {
       "type" : "move_first_dqs",
       "data" : {}
    },

La clave "`data`" se puede omitir ya que no incluye ninguna información.

La clase [MoveAllStrategyBuilder]{.sans-serif} crea una instancia de
[MoveAllStrategy]{.sans-serif} a partir de la estructura `JSON`:

    {
       "type" : "move_all_dqs",
       "data" : {}
    },

La clave "`data`" se puede omitir ya que no incluye ninguna información.

Una vez definidas las clases anteriores, podemos utilizarlas para crear
la factorı́a correspondiente de la siguiente forma:

    List<Builder<DequeuingStrategy>> dqbs = new ArrayList<>();
    dqbs.add( new MoveFirstStrategyBuilder() );
    dqbs.add( new MoveAllStrategyBuilder() );
    Factory<DequeuingStrategy> dqsFactory = new BuilderBasedFactory<>(dqbs);

#### Eventos "Factory"

Para esta factorı́a necesitamos un "builder" para cada clase de evento
utilizado en la práctica, definidos en la Sección
 [1.5.3](#sec:events){reference-type="ref" reference="sec:events"}.
Todos los "builders" deben extender a [Builder\<Event\>]{.sans-serif},
ya que deben crear instancias de clases que extienden a
[Event]{.sans-serif}.

La clase [NewJunctionEventBuilder]{.sans-serif} crea una instancia de
[NewJunctionEvent]{.sans-serif} a partir de la siguiente estructura
`JSON`:

    {
      "type" : "new_junction",
      "data" : {
        "time" : 1,
        "id"   : "j1",
        "coor" : [100,200],
        "ls_strategy" : { "type" : "round_robin_lss", "data" : {"timeslot" : 5} },
        "dq_strategy" : { "type" : "move_first_dqs",  "data" : {} }
      }
    }

La clave "`coor`" es una lista que contiene las coordenadas
[x]{.sans-serif} e [y]{.sans-serif} (en este orden). Observa que su
sección "`data`" incluye estructuras `JSON` para describir las
estrategias que se deben usar. Asumimos que las factorı́as para estas
estrategias se suministran a la constructora de este "builder", es
decir, su constructora debe declararse como sigue (más tarde veremos
como pasar estas factorı́as a la constructora):

    public NewJunctionEventBuilder(Factory<LightSwitchingStrategy> lssFactory, Factory<DequeuingStrategy> dqsFactory)

La clase [NewCityRoadEventBuilder]{.sans-serif} crea una instancia de
[NewCityRoadEvent]{.sans-serif} a partir de la siguiente estructura
`JSON`:

    {
      "type" : "new_city_road",
      "data" : {
        "time"     : 1,
        "id"       : "r1",
        "src"      : "j1",
        "dest"     : "j2",
        "length"   : 10000,
        "co2limit" : 500,
        "maxspeed" : 120,
        "weather"  : "SUNNY"
      }
    }

La clase [NewInterCityRoadEventBuilder]{.sans-serif} crea una instancia
de [NewInterCityRoadEvent]{.sans-serif} a partir de:

    {
      "type" : new_inter_city_road
      "data" : {
        "time"     : 1,
        "id"       : "r1",
        "src"      : "j1",
        "dest"     : "j2",
        "length"   : 10000,
        "co2limit" : 500
        "maxspeed" : 120,
        "weather"  : "SUNNY"
      }
    }

Observa que las clases [NewCityRoadEventBuilder]{.sans-serif} y
[NewInterCityRoadEventBuilder]{.sans-serif} tienen parte de su código
igual, por ello podrı́as considerar hacer refactorización introduciendo
una superclase.

La clase [NewVehicleEventBuilder]{.sans-serif} crea una instancia de
[NewVehicleEvent]{.sans-serif} a partir de:

    {
      "type" : "new_vehicle",
      "data" : {
        "time"      : 1,
        "id"        : "v1",
        "maxspeed"  : 100,
        "class"     : 3,
        "itinerary" : ["j3", "j1", ...]
      }
    }

La clase [SetWeatherEventBuilder]{.sans-serif} crea una instancia de
[SetWeatherEvent]{.sans-serif} a partir de la estructura `JSON`:

    {
      "type" : "set_weather",
      "data" : {
        "time"     : 10,
        "info"     : [ { "road" : r1, "weather": "SUNNY" },
                       { "road" : r2, "weather": "STORM" },
                       ...
                     ]
      }
    }

La clase [SetContClassEventBuilder]{.sans-serif} crea una instancia de
[SetContClassEvent]{.sans-serif} a partir de:

    {
      "type" : "set_cont_class",
      "data" : {
        "time"     : 10,
        "info"     : [ { "vehicle" : v1, "class": 3 },
                       { "vehicle" : v4, "class": 2 },
                       ...
                     ]
      }
    }

Una vez implementadas las clases anteriores, podemos usarlas para crear
la factorı́a correspondiente, utilizando el siguiente código:

    List<Builder<Event>> ebs = new ArrayList<>();
    ebs.add( new NewJunctionEventBuilder(lssFactory,dqsFactory) );
    ebs.add( new NewCityRoadEventBuilder() );
    ebs.add( new NewInterCityRoadEventBuilder() );
    // ...
    Factory<Event> eventsFactory = new BuilderBasedFactory<>(ebs);

### El controlador {#sec:controllerc}

El controlador se implementa en la clase [Controller]{.sans-serif},
dentro del paquete "[simulator.control]{.sans-serif}". Esta clase es la
responsable de:

-   leer los eventos de un [InputStream]{.sans-serif} y añadirlos al
    simulador; y

-   ejecutar el simulador un número determinado de pasos, mostrando los
    diferentes estados en un [OutputStream]{.sans-serif}.

La clase [Controller]{.sans-serif} contiene al menos los siguientes
atributos (no públicos):

-   *traffic simulator* (de tipo [TrafficSimulator]{.sans-serif}):
    utilizado para ejecutar la simulación.

-   *events factory* (de tipo [Factory\<Event\>]{.sans-serif}): que se
    usa para parsear los eventos suministrados por el usuario.

Tiene sólo la constructora pública:

    public Controller(TrafficSimulator sim, Factory<Event> eventsFactory) {
      // TODO complete
    }

En la constructora debes comprobar que los valores de los parámetros no
son [null]{.sans-serif}, y lanzar una excepción en caso contrario.
Además esta clase tendrá los siguientes métodos:

-   [public void loadEvents(InputStream in)]{.sans-serif}: asumimos que
    [in]{.sans-serif} incluye (el texto de) una estructura `JSON` de la
    forma:

    ::: center
    `{ “events”: [`$e_1$`,…,`$e_n$`] }`
    :::

    donde cada $e_i$ es a su vez una estructura `JSON` asociada a un
    evento. Este método primero convierte la entrada `JSON` en un objeto
    [JSONObject]{.sans-serif} utilizando:

    ::: center
    [JSONObject jo = new JSONObject(new JSONTokener(in));]{.sans-serif}
    :::

    y después extrae cada $e_i$ de [jo]{.sans-serif}, crea el evento
    correspondiente [e]{.sans-serif} utilizando la *factorı́a de
    eventos*, y lo añade al simulador invocando al método
    [addEvent]{.sans-serif}. Este método debe lanzar una excepción si la
    entrada `JSON` no encaja con la de arriba.

-   [public void run(int n, OutputStream out)]{.sans-serif}: ejecuta el
    simulador [n]{.sans-serif} ticks, llamando al método
    [advance]{.sans-serif} exactamente [n]{.sans-serif} veces, y escribe
    los diferentes estados en [out]{.sans-serif} utilizando el siguiente
    formato `JSON`:

    ::: center
    `{ "states": [`$s_1$`,…,`$s_n$`] }`
    :::

    donde $s_i$ es el estado del simulador **después de** ejecutar el
    paso $i$. Observa que el estado $s_i$ se obtiene llamando al método
    [report()]{.sans-serif} del simulador de tráfico.

-   [public void reset()]{.sans-serif}: invoca al método
    [reset]{.sans-serif} del simulador de tráfico.

## Clase Main {#sec:main}

Te facilitaremos, junto con la práctica, un esqueleto de la clase
[Main]{.sans-serif}, que usa una librerı́a externa para simplificar el
parseo de las opciones de lı́nea de comandos. La clase
[Main]{.sans-serif} debe aceptar las siguientes opciones por lı́nea de
comandos:

    > java Main -h
    usage: Main [-h] -i <arg> [-o <arg>] [-t <arg>]
     -h,--help           Print this message
     -i,--input <arg>    Events input file
     -o,--output <arg>   Output file, where reports are written.
     -t,--ticks <arg>    Ticks to the simulator's main loop (default
                         value is 10).

3.25ex -1em Examples.

      java Main -i eventsfile.json -o output.json -t 100
      java Main -i eventsfile.json -t 100

El primer ejemplo escribe la salida en un fichero `output.json`,
mientras que el segundo escribe la salida en la consola. En ambos casos,
el fichero de entrada, conteniendo los eventos, es `eventsfile.json` y
el número de ticks es $100$. Simplemente tienes que completar los
métodos [initFactories]{.sans-serif} y [startBatchMode]{.sans-serif}, y
añadir código para procesar la opción [-t]{.sans-serif} (estudiando como
se ha hecho para otras opciones). Observa que es un argumento opcional,
que en caso de no ser proporcionado, tendrá como valor $10$.

## Otros comentarios {#sec:app}

-   El directorio [resources/examples]{.sans-serif} incluye un conjunto
    de ejemplos, junto con su salida esperada, que puedes usar para
    comprobar tu práctica (ver
    [resources/examples/README.md]{.sans-serif} para más detalles).
    Posiblemente añadiremos más ejemplos antes de la fecha de entrega.
    Tu práctica debe pasar todos los tests suministrados.

-   Para convertir un [String]{.sans-serif} [s]{.sans-serif} a su
    correspondiente valor [enum]{.sans-serif}, por ejemplo de tipo
    [Weather]{.sans-serif}, usa [Weather.valueOf(s)]{.sans-serif} -- o
    mejor usa [s.toUpperCase()]{.sans-serif} para evitar problemas de
    mayúsculas/minúsculas, asumiendo que todos los valores del tipo
    [enum]{.sans-serif} están definidos usando mayúsculas (esto es
    cierto para [Weather]{.sans-serif} y [VehicleStatus]{.sans-serif}).

-   Para escribir fácilmente en un [OutputStream]{.sans-serif}
    [out]{.sans-serif}, primero crea un [PrintStream]{.sans-serif}
    utilizando "[PrintStream p = new PrintStream(out);]{.sans-serif}" y
    después usa comandos como [p.println(\"\...\")]{.sans-serif},
    [p.print(\"\...")]{.sans-serif}, etc.

-   Para transformar un [JSONObject]{.sans-serif} [jo]{.sans-serif} a
    [String]{.sans-serif}, utiliza [jo.toString()]{.sans-serif} o
    [jo.toString(3)]{.sans-serif}. El primero es compacto, no escribe
    espacios en blanco. El segundo muestra la estructura `JSON`, donde
    el argumento es el número de espacios que añade a cada nivel de
    indentación.

[^1]: <https://en.wikipedia.org/wiki/JSON>
