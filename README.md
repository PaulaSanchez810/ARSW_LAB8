### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW
> Integrantes:
> * 👩 Paula Andrea Guevara Sánchez.
> * 👨 Daniel Felipe Rincón Muñoz.

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

![Imágen 1](images/part1/part1-vm-basic-config.png)

> ![Imágen 1](https://github.com/PaulaSanchez810/ARSW_LAB8/blob/master/images/part1/crecionMaquina.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

> ![](https://github.com/PaulaSanchez810/ARSW_LAB8/blob/master/images/part1/1-6.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
> ![](https://github.com/PaulaSanchez810/ARSW_LAB8/blob/master/images/part1/1-6.2.png)
>    * 1000000 = 38s
>    * 1010000 = 31s
>    * 1020000 =31s
>    * 1030000 =31s
>    * 1040000 =32s
>    * 1050000 = 33s
>    * 1060000 = 34s
>    * 1070000 = 33s
>    * 1080000 = 33s
>    * 1090000 = 34s    

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

> ![Imágen 2](https://github.com/PaulaSanchez810/ARSW_LAB8/blob/master/images/part1/1-8.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
>    * 1000000 = 25s
>    * 1010000 = 27s
>    * 1020000 = 29s
>    * 1030000 = 29s
>    * 1040000 = 28s
>    * 1050000 = 28s
>    * 1060000 = 30s
>    * 1070000 = 30s
>    * 1080000 = 30s
>    * 1090000 = 32s   

13. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

> No se cumple con los requerimientos de escalabilidad, debido a que una petición se demora 30 segundos, debido a que la mejora que se hizo antes de escalar maquina y escalarla fue minima. 

15. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
> crea los siguientes recursos como muestra en la imagen. 
> ![Imágen 1](https://github.com/PaulaSanchez810/ARSW_LAB8/blob/master/images/part1/crecionMaquina.png)

3. ¿Brevemente describa para qué sirve cada recurso?
> * VERTICAL-SCALABILITY : Maquina Virtual.
> * Vertical-scalability566: Interfases de red.
> * VERTICAL-SCALABILITY-ip: dirección IP publica.
> * VERTICAL-SCALABILITY-nsg: para establecer reglas de seguridad sobre la red.
> * SCALABILITY_LAB-vnet: red interna.

5. 
¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? 
> Porque el proceso esta siendo utilizado por el usuario conectado y cuando se desconecta se detiene el proceso. 
¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
> Para que se pueda acceder al servicio que esta corriendo en ese puerto, desde un medio externo.

7. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.
> Porque son números muy grandes y tiene que hacer varias iteraciones.
9. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
> Porque es un algoritmo computacionalmente pesado.
11. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    > el tiempos de ejecución fue de 50.7, porque estaban corriendo dos peticiones en paralelo.
    * Si hubo fallos documentelos y explique.
    > ECONNRESET: significa el el recurso cerro la concxion, se creo que fue porque el servidor estaba muy cargado y no puso reponder a la solicitud.
12. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
pegar imagen 
13. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?
> No. 
    ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
> se procesa un poco mas reapido pero no es mucha la diferencia es muy minim.
14. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
> La infraestructura se vuelve muhco más costosa, en nuestro caso, el costo aumentó en casi 20 veces por mes.
15. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
> Si hubo mejora en el consumo de CPU debido a que la segunda máquina tenía dos CPUs, y la mejoa en los tiempos de respuesta fué mínima, de no mas de 5 segundos.
16. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?
> No, el comportamiento es lineal.

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

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?
   >Azure Load Balancer puede configurarse como un balanceador público o interno:
   > *Público: Tendrá una IP pública en el frontend en la que recibirá peticiones que repartirá entre las máquinas del backend.
   > *Interno: Tendrá una IP privada y no será accesible desde Internet. Igual que en el público, las peticiones que lleguen a la IP del frontend se distribuirán entre las             máquinas del backend.
*  ¿Qué es SKU, qué tipos hay y en qué se diferencian?
   > SKU es la abreviatura de 'Unidad de mantenimiento de existencias. En términos de la nube de Microsoft Azure, básicamente significan un SKU que se puede comprar bajo un          producto. Tiene un montón de formas diferentes del producto.
*  ¿Por qué el balanceador de carga necesita una IP pública?
   >  la dirección IP de front-end es pública, lo que permite que se agregue como punto de conexión al perfil de Traffic Manager más adelante.
* ¿Cuál es el propósito del *Backend Pool*?
   > El Backend Pool define el grupo de recursos que servirán tráfico para una regla de balanceo de carga específica.
* ¿Cuál es el propósito del *Health Probe*?
   > El Health Probe permite al balanceador detectar el estado de los endpoints, con esta información determina cuales instancias del Backend Pool recibirán nuevas peticiones.
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
   > La Load Balancing Rule define cómo el tráfico es distribuido a la máquinas virtuales, se pueden definir sin persistencia, o con persistencia de IP del cliente, cuando esta      opción está habilitada, las peticiones enviadas por un cliente siempre serán redirigidas hacia la misma máquina. La escalabilidad  puede verse afectada ya que una máquina        nueva no se utilizará a menos que haya clientes nuevos.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
   >  Una Virtual Network es una red interna en Azure que cumple la función de conectar internamente diferentes componentes en la nube. Una Subnet es una sub-red en una Virtual       Network. Los Address Space son las direcciones de red asignables dentro de una Virtual Network. Los Address Range son las direcciones de red asignables dentro de una             Subnet
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
   > Una Availability Zone es una ubicación física única en una región. Cada zona esta hecha de uno o mas centros de datos, equipados con energía, refrigeración y servicio de        red independientes. Los servicios Zone-Redundant replican las aplicaciones y datos a través de las Availability Zones para protegerlas de puntos únicos de fallo.
* ¿Cuál es el propósito del *Network Security Group*?
   > El Network Security Group nos ayuda a mapear reglas de red que pueden ser aplicadas a diferentes recursos a la vez, de esta forma, si queremos realizar una modificación de      red, lo haremos desde del grupo, y no desde cada una de las máquinas.
* Informe de newman 1 (Punto 2)
   IMAGEN NEWMAN
* Presente el Diagrama de Despliegue de la solución.
   



