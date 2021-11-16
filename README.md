### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contará con $200 USD para gastar durante 1 mes.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

### Parte 1.1 - Llave SSH publica

* Crearemos en Azure Cloud shell, en nuestro caso lo haremos con Azure Cloud Shell (bash)

<img width="850" height="720" src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/01-CreacionBash.png" alt="bash-console">
<br>


* Crearemos un llave SSH publica ejecutando el siguiente comando(el siguiente comando crea un par de claves SSH con ayuda del cifrado RSA y una longitud en bits de 4096).

    ```bash
    ssh-keygen -m PEM -t rsa -b 4096
    ```
<img width="850" height="720" src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/02-CreacionSSH.png" alt="creating-ssh">
<br>

* Consultaremos que llave SSH generamos, con el siguiente comando

    ```bash
    cat ~/.ssh/id_rsa.pub
    ```
<img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/03-PublicSSH.png" alt="public-ssh">
<br>

### Configuracion de maquina virtual

![Imágen 1](images/part1/part1-vm-basic-config.png)


### Maquina 

<img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/04-VirtualMachine.png" alt="virtual-machine">
<br>
<br>

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

<img  src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/05-conexionVM.png" alt="VM-connection">
<br>
<br>


3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).

<img  src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/06-nodeInstallation.png" alt="node-installation">
<br>


4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`


<img  src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/07-gitHub-npmInstall.png" alt="node-github">
<br>



5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`

<img  src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/08-npmForever.png" alt="npm-forever">
<br>

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000

    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/09-response-1000000.png" alt="response-1000000">
    <br>

    * 1010000

    <img  src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/09-response-1010000.png" alt="response-1010000">
    <br>

    * 1020000

    <img  src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/09-response-1020000.png" alt="response-1020000">
    <br>

    * 1030000

    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/09-response-1030000.png" alt="response-1030000">
    <br>

    * 1040000

    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/09-response-1040000.png" alt="response-1040000">
    <br>

    * 1090000  

    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/09-response-1090000.png" alt="response-1090000">
    <br>

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/09-change.png">

    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/09-commands01.png">
    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/09-commands02.png">

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

## Actualizando tamaño
<img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/12-resize.png" alt="resizing-vm">

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
    * 1000000

    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/10-response-1000000.png" alt="response-1000000">
    <br>

    * 1010000

    <img  src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/10-response-1010000.png" alt="response-1010000">
    <br>

    * 1020000

    <img  src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/10-response-1020000.png" alt="response-1020000">
    <br>

    * 1030000

    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/10-response-1030000.png" alt="response-1030000">
    <br>

    * 1040000

    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/10-response-1040000.png" alt="response-1040000">
    <br>

    * 1090000  

    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/10-response-1090000.png" alt="response-1090000">
    <br>

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

    Si se cumple con el escenario de calidad propuesto ya que al realizar la escalabilidad vertical pues al aumentar las especificaciones de la VM, se aprovechan de una mejor forma y los tiempos de respuesta de las solicitudes es menor a momentos previos.
Evidenciendolo mucho más en la siguiente imagen, que muestra la disminución de consumo de CPU de la VM.

<img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/12-cpu.png">
<br>

14. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
    Azure crea 5 recursos junto a la maquina virtual, los cuales son:

    * Virtual network/subnet
    * Public IP address
    * Network security group
    * Network interface
    * OS disk

2. ¿Brevemente describa para qué sirve cada recurso?

    * <b>Virtual network/subnet:</b><br> 
    
    Este tipo de red permite que diferentes recursos de Azure como lo son las máquinas virtuales, se puedan comunicarse de forma segura entre usuarios, con Internet y con las redes locales. VNet es similar a una red tradicional que funcionaría en su propio centro de datos, pero aporta las ventajas adicionales como lo son la escalabilidad, la disponibilidad y el aislamiento.
    
    * <b>Public IP address:</b><br>
    
    Dirección IP cuyo conjunto de números que identifica, de manera lógica y jerárquica, a una interfaz en la red de un dispositivo que utilice el protocolo o que corresponde al nivel de red del modelo TCP/IP, de tal manera que sea visible con internet y algunos servicios públicos de azure, un recurso cuya dirección ip sea publica, puede ser asociado a interfaces de red de máquinas virtuales, balanceadores de carga, firewalls, etc.

    
    * <b>Network security group:</b><br>
    
    Se utiliza "Azure network security group" para filtrar el trafico de red desde y para los recursos de Azure en una red virtual de Azure. Un grupo de seguridad se basa principalmente en un conjunto de reglas de seguridad que permiten o niegan el trafico de red entrante o el trafico de red saliente. Para cada regla, se debera especificar origen, destino, puerto y protocolo.

    * <b>Network interface:</b><br>

    Componente que permite que las VM de Azure se comunique con Internet y demas recursos locales, ya que es la capa IP la que selecciona una interfaz de red para el envío de paquetes.
    
    * <b>OS disk:</b><br>

    Almacenamiento del OS de la maquina virtual creada.
    
    
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos 
crear un *Inbound port rule* antes de acceder al servicio?

Cuando se inicia la conexion ssh con la VM, se inicia un proceso para este este servicio y este proceso quedara asociado al usuario el cual realizo la conexion, por lo que al cerrar la conexion este proceso se terminara.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

| N             | Standard_B1ls (s)    | Standars_B2ms (s) |
|---------------|----------------------|-------------------|
| 1000000       | 19.47 s              | 17.81 s           |
| 1010000       | 19.64 s              | 18.19 s           |
| 1020000       | 20.00 s              | 19.22 s           |
| 1030000       | 21.34 s              | 19.75 s           |
| 1040000       | 22.07 s              | 19.67 s           |
| 1050000       | 21.78 s              | 19.49 s           |
| 1060000       | 22.27 s              | 19.84 s           |
| 1070000       | 22.61 s              | 20.27 s           |
| 1080000       | 22.82 s              | 20.70 s           | 
| 1090000       | 24.11 s              | 21.99 s           |

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

<img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/13-cpu.png">
<br>

Cada petición realizada hace un gran uso de la CPU dado que se realizan cálculos con redundancia, esto porque no se esta haciendo uso de algun metodo o tecnica como la <b>memorización</b> la cual permita almacenar calculos que fueron realizados con anterioridad, además no se está implementando concurrencia lo que hace que se consuman más recursos y el tiempo de respuesta sea extenso.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:

    * Tiempos de ejecución de cada petición.

    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/11-Commands01.png">
    <br>

    El tiempo promedio de respuesta de cada petición fue de 17.4s con un total de data recivida de: 2.09MB (approx)
    
    <img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/11-Commands02.png">
    <br>

    El tiempo promedio de respuesta de cada petición fue de 17.5s con un total de data recivida de: 2.09MB (approx)

    * Si hubo fallos documentelos y explique.

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

| Size             | vCPUs  | RAM (GiB)  |  Data disks| Max IOPS |Temp storage (GiB) | Cost/month |
|------------------|--------|------------|------------|----------|-------------------|------------|
|B1ls              |  1     |0.5         |2           |160       |4                  |3,80 US$    |
|B2ms              |  2     |8           |4           |1920      |16                 |60,74 US$   |

En este caso ambos tamaños son utilizados para desarrollo y pruebas que por lo general su trafico de datos es bajo/medio.


8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

Aunque aumentar el tamaño de la máquina virtual significa una disminución de uso recursos (como se pudo observar en el consumo de CPU), aun hace falta replantear FibonacciApp de tal manera que se memoricen valores anteriormente calculados, esto para lograr disminuir aun mas los tiempos de respuesta de la aplicación.

Sin embargo no hay que olvidar que es un sólo servidor el que está atendiendo todas las peticiones web y por lo tanto no garantizaria siempre una alta disponibilidad ante un desbordamiento de peticiones.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

La maquina virtual se reinicia y a causa de todo esto se debe de realizar nuevamente una conexion segura usando el protocolo ssh, causando que en ese tiempo de reinicio la maquina no prestara su servicio web, causando que todas las peticiones que ocurran en este lapso de tiempo no sean atendidas. 

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

Si, dado que la máquina virtual dispone de más recursos para realizar sus cálculos y atender a las peticiones.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

Para realizar eso usaremos el siguiente comando, en diferentes consolas:
```bash
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 
```

* Informe petición 01

<img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/11-Postman01.png" alt="Postman01.png">
<br>
* Informe petición 02

<img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/11-Postman02.png" alt="Postman02.png">
<br>
* Informe petición 03

<img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/11-Postman03.png" alt="Postman03.png">
<br>
* Informe petición 04

<img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/11-Postman04.png" alt="Postman04.png">
<br>


El comportamiento del sistema no mejoró.

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

* Realizamos la actualización de la direccion ip en el archivo <i><b>[ARSW_LOAD-BALANCING_AZURE].postman_environment.json</b></i>

<img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/13-Ip-balanceador.png" alt="Ip-Balanceador-json">
<br>

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
    * Tipos de balanceadores:
        * **Balanceador de carga interno( Privado ):** Este balanceador de carga se encarga de equilibrar la carga de trafico de una red privada ( se utilizan unicamente direcciones ip privadas en la interfaz).

        * **Balanceador de carga publico:** Este balanceador de carga se encarga de equilibrar la carga de trafico de redes publicas, especificamente de la carga proveniente de internet, la dirección ip pública y el puerto son asigandos. 

        * **Balanceador de carga de puerta de enlace:** Es un balanceador que se adapta a escenarios de alto rendimiento y alta disponibilidad con dispositivos virtuales de red (NVA) de terceros. Con las capacidades de Gateway Load Balancer, puede implementar, escalar y administrar NVA fácilmente. Encadenar un balanceador de carga de puerta de enlace a su punto final público solo requiere un clic.
        
    * SKU( Stock Keeping Unit) Azure:
        * Representa una unidad de mantenimiento de existencias, lo cual significa que es un codigo unico asignado a un servicio o producto de Azure, el cual nos permite a nosotros como usuarios la posibilidad de comprar existencias de los mismos.
    * Tipos de SKU:
        *  **Estándar:** Productos estandar los cuales se pueden vender de manera individual o en paquetes, 
        *  **Esamblaje: Aquellos productos que se deben ensamblar antes de un envio, todos los SKU deberan encontrarse dentro de una misma instalacion.** 
        *  **Virtual: Aquellos productos que son virtuales, es decir que no necesitan de una instalacion fisica evitando asi un nivel de inventario** 
        *  **Componente: Productos incluidos en paquetes, esamblajes y colecciones, los cuales no se pueden vender de manera individual** 

* ¿Cuál es el propósito del *Backend Pool*?

 El proposito de backend pool es el almacenamiento las direcciones IP de las máquinas virtuales (NICs) conectadas al balanceador de carga, ademas de esto este componente define el grupo de recursos que brindarán tráfico para una Load Balancing Rule determinada.
 
 El backend pool es el conjunto de máquinas virtuales o instancias que atienden las solicitudes entrantes. Para escalar de manera rentable y satisfacer grandes volúmenes de tráfico entrante, generalmente se recomienda agregar más instancias a este grupo.

* ¿Cuál es el propósito del *Health Probe*?

El proposito del heralth probe es  la supervicion de el estado de la aplicación, este se utiliza para detectar fallos de una aplicación en un endpoint del backend; Las respuestas de Health Probe determinan qué instancias del backend pool recibirán nuevos flujos.

Se configura un Health Probe que el balanceador de carga puede utilizar para determinar si su instancia está en buen estado. Si su instancia falla en su Health Probe suficientes veces, dejará de recibir tráfico hasta que empiece a pasar Health Probe de nuevo.

* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.

El proposito del Load Balancing Rule es definir como se debera distribuir el trafico de las maquinas virtuales dentro del pool de backend.

Se define la configuración de IP del frontend para el tráfico entrante y el pool de IP del backend para recibir el tráfico. El puerto de origen y destino se definen en la regla. En azure exiten 3 tipos de sesiones de persistencia:

    * **Ninguno(hash-based):** Peticiones recurrentes de un mismo cliente podrían ser atendidas por máquinas diferentes del backend (No importa el estado de una peticion).
    * **IP del cliente:** : Todas las peticiones que procedan de una misma IP de origen serás atendidas por la misma máquina del backend.
    * **IP y protocolo del cliente:** Todas las peticiones que procedan de una misma IP y puerto de origen serán atendidas por la misma máquina del backend, pero si vienen de la misma IP pero con un puerto de origen diferente podrían ser atendidas por otra máquina del backend.

* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?

    * Red Virtual: Una red virtual es un una red que permite la interconexino de dispositivos y maquinas virtuales metiante software, independientemente de un ubicacion fisica.

    * Subnet: Es una segmentacion de una red fisica o red virtual, estas segmentaciones contaran con su propio rando de direcciones ip segun como se realice la particion de la direccion ip original.

    * Address space: Cuando se crea una red virtual, se debe de especificar un rango de direcciones ip las cuales no se superponen unas con otras( En otras palabras es la direccion ip que identifica la red, eg: 10.0.0.0/24) .


    * Address range: Determina el numero de direcciones que se tienen o se pueden tener en un address space y dependiendo de la cantidad de recursos que se necesiten en la red virtual, el rango aumentará o disminuirá.

* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea 
*zone-redundant*?

    * Availability Zone: Una availability zone, son zonas que buscan garantizar una alta disponibilidad replicando sus aplicaciones y datos con el fin de protegerlos de puntos de fallo, estas zonas se encuentran dentro de una region y cada una de ellas se compone de uno o mas centros de datos con refrigeracion y redes independientes.

    En este laboratorio seleccionamos 3 zonas de disponibilidad para garantizar una mejor disponibilidad y tolarancia a fallos dentro del sistema.
    
    * IP zone-redundant: Cuando utilizamos una ip zone-redundant azure separa física y lógicamente el gateway dentro de una region, lo cual permite mejorar la conectividad de la red privada y disminuye fallos a nivel de zona de disponibilidad.


* ¿Cuál es el propósito del *Network Security Group*?

El proposito de un Network security group es poder lograr filtrar el trafico desde y hacia los recursos de una red virtual de Azure, un grupo de seguridad nos permite definir diferentes reglas de entrada y salida para que permitan o nieguen el trafico de datos entre los recursos de Azure (Similar a las funcionalidades de un Firewall).

* Informe de newman 1 (Punto 2)



* Presente el Diagrama de Despliegue de la solución.

<img src="https://github.com/Ersocaut/ARSW-Lab08/blob/master/images/solution/13-Diagrama-Despliegue.png" alt="Diagrama-Despliegue">
<br>




## Referencias 

* [Pasos rápidos: Creación y uso de un par de claves pública-privada SSH para máquinas virtuales Linux en Azure](https://docs.microsoft.com/es-es/azure/virtual-machines/linux/mac-create-ssh-keys)

* [¿Qué es Azure Virtual Network?](https://docs.microsoft.com/es-es/azure/virtual-network/virtual-networks-overview#:~:text=Azure%20Virtual%20Network%20(VNet)%20es,y%20con%20las%20redes%20locales.)

* [Network security groups-Azure](https://docs.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)

* [Create an Inbound Port Rule](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule)

* [Azure Load Balancer new distribution mode](https://azure.microsoft.com/en-us/blog/azure-load-balancer-new-distribution-mode/)

* [Azure Load Balancer SKUs](https://docs.microsoft.com/en-us/azure/load-balancer/skus)

* [¿Qué significa SKU en Microsoft azure?](https://docs.microsoft.com/en-us/partner-center/develop/product-resources#sku)

* [¿Qué es Azure Load Balancer?](https://docs-microsoft-com.translate.goog/en-us/azure/load-balancer/load-balancer-overview?_x_tr_sl=en&_x_tr_tl=es&_x_tr_hl=es-419&_x_tr_pto=nui)

* [Gateway Load Balancer (Preview)](https://docs.microsoft.com/en-us/azure/load-balancer/gateway-overview)
 
* [¿Qué son las redes virtuales?](https://www.vmware.com/es/topics/glossary/content/virtual-networking.html)