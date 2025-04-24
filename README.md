### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

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

   Vamos al portal de azure, accedemos a la seccion de "virtual machines" y le damos a crear

   ![Image](https://github.com/user-attachments/assets/f71527b3-7a8c-4f89-a2ca-2ca920c45536)

   Ponemos las caracteristicas de la maquina dadas
   
   ![Image](https://github.com/user-attachments/assets/b42ee11a-ce20-45db-a3fa-bf2b9f0a9965)
   
   ![Image](https://github.com/user-attachments/assets/beb7ecc3-e5fc-455a-862c-c269f1bfcecd)

   Por ultimo revisamos, creamos y verificamos que se desarrollado correctamente
   
   ![Image](https://github.com/user-attachments/assets/087760dc-e999-449a-8870-67bb4e7bcc42)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

   Antes de conectarnos, consultamos la ip de la maquina en azure
    
   ![Image](https://github.com/user-attachments/assets/75218613-6868-43b6-92cf-4f478d4e9cc3)

   Usamos el comando en el cmd agregando el -i para que lea la llave ssh y nos pueda dar acceso
   
   ![Image](https://github.com/user-attachments/assets/fc178d50-9155-4daf-9a3a-f5e0e1d69142)

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).

   Habilitamos el repositorio NodeSource ejecutando el comando `curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`

   ![Image](https://github.com/user-attachments/assets/0d997464-ae02-4f66-a0ed-5b9e8fbf0dc5)

   Una vez habilitado el repositorio NodeSource, instale Node.js y npm escribiendo `sudo apt install nodejs`
   
   ![Image](https://github.com/user-attachments/assets/74ce2278-4e7d-4108-b803-69959f8c01ea)

   Verificamos que Node.js se haya instalado correctamente imprimiendo su version
   
   ![Image](https://github.com/user-attachments/assets/56858e61-e431-4975-9cc1-1d56e947b305)

4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

    Ejecutamos los 3 comandos para instalar la aplicacion

   ![Image](https://github.com/user-attachments/assets/deb28da2-4f4f-47b6-8c3e-c58dafaf761a)

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

   ![Image](https://github.com/user-attachments/assets/523f5c94-a209-473d-8cb1-becfd760fc6e)

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

   Para eso vamos a la seccion que nos mencionan y llenamos con los datos que se nos muestran

   ![Image](https://github.com/user-attachments/assets/033d9254-d816-4238-8196-733c91a06b17)

   Abrimos el navegador y consultamos con la url de la ip y el puerto 3000, ademas de llamar a la aplicacion y mandarle el numero a evaluar
   
   ![Image](https://github.com/user-attachments/assets/845aea9f-d4ed-4c2b-8e58-e9005eaa8bac)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:

    Acontinuacion, estos fueron los tiempos marcados desde la consola del navegador

   * Time 1000000 - 15.95 s
   * Time 1010000 - 16.28 s
   * Time 1020000 - 16.50 s
   * Time 1030000 - 16.86 s
   * Time 1040000 - 17.32 s
   * Time 1050000 - 19.25 s
   * Time 1060000 - 18.21 s
   * Time 1070000 - 18.58 s
   * Time 1080000 - 18.87 s
   * Time 1090000 - 19.21 s

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

   Podemos ver el monitoreo de la infraestructura y consumo de recursos, acontinuacion

   ![Image](https://github.com/user-attachments/assets/a0ae73ae-63d4-4550-817f-86f37875a939)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
   
    Instalamos newman con el comando dado

   ![Image](https://github.com/user-attachments/assets/ab677ce3-e611-4ac8-b50d-529675d0678f)

   Desde el explorador entramos al archivo y modificamos el archivo mencionado cambiando la ip al de nuestra maquina

   ![Image](https://github.com/user-attachments/assets/3db1a8cc-0194-4583-becf-7df4f9b0f3f9)

    Ejecutamos el comando para realizar la carga

   ![Image](https://github.com/user-attachments/assets/0e3f4168-56dc-4311-a528-7890bb5a83cd)

    Al final miramos los resultados dados por newman

   ![Image](https://github.com/user-attachments/assets/f0292cc9-0767-4c04-83c1-dc8480e4026f)


10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

   Seleccionamos el tamaño que se nos pide y reajustamos el tamaño de la maquina

   ![Image](https://github.com/user-attachments/assets/802730fc-9981-4b7a-819b-44c1aa6a6c74)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

    Vemos los nuevos tiempos de respuesta, acontinuacion

    * Time 1000000 - 12.73 s
    * Time 1010000 - 12.89 s
    * Time 1020000 - 12.97 s
    * Time 1030000 - 13.18 s
    * Time 1040000 - 13.80 s
    * Time 1050000 - 13.94 s
    * Time 1060000 - 14.07 s
    * Time 1070000 - 14.67 s
    * Time 1080000 - 14.70 s
    * Time 1090000 - 15.16 s

    Junto al monitoreo de los recursos y infraestructura

   ![Image](https://github.com/user-attachments/assets/d5b65406-1052-4d6d-9466-adcf831a083e)

   Y los nuevos resultados al ejecutar la carga nuevamente con newman

   ![Image](https://github.com/user-attachments/assets/ebf74418-9862-4f0c-a131-ad6d2f6afbd2)

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

   Reajustamos el tamaño de la maquina al inicial

   ![Image](https://github.com/user-attachments/assets/07cecee6-e253-406f-aa09-5fa1dddf289a)

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

   En total, Azure creó 7 recursos, que son:

   * scalability_lab_KEY – SSH Key
   * VERTICAL-SCALABILITY – Virtual machine
   * VERTICAL-SCALABILITY-ip – Public IP address
   * VERTICAL-SCALABILITY-nsg – Network security group
   * VERTICAL-SCALABILITY-vnet – Virtual network
   * vertical-scalability851 – Network Interface
   * VERTICAL-SCALABILITY_OsDisk... – Disk (Disco del SO)

    ![Image](https://github.com/user-attachments/assets/3357da95-aed6-495d-8ad2-74a7bde0d4cd)

2. ¿Brevemente describa para qué sirve cada recurso?

   | **Recurso**                          | **Tipo**                    | **¿Para qué sirve?**                                                                                           |
   |-------------------------------------|-----------------------------|----------------------------------------------------------------------------------------------------------------|
   | `scalability_lab_KEY`               | SSH Key                     | Clave pública SSH usada para conectarte de forma segura a la VM.                                               |
   | `VERTICAL-SCALABILITY`              | Virtual Machine             | La máquina virtual donde corre la app de Node.js.                                                              |
   | `VERTICAL-SCALABILITY-ip`           | Public IP Address           | Dirección IP pública de la VM para acceder desde Internet.                                                     |
   | `VERTICAL-SCALABILITY-nsg`          | Network Security Group      | Firewall que controla qué tráfico entra y sale de la VM (ej: permite puerto 3000 para la ejecucion de la app). |
   | `VERTICAL-SCALABILITY-vnet`         | Virtual Network (VNet)      | Red privada donde está conectada la VM. Facilita la comunicación entre recursos sin exponerlos.                |
   | `vertical-scalability851`           | Network Interface (NIC)     | Conecta la VM a la red virtual. Es como la tarjeta de red de la máquina.                                       |
   | `VERTICAL-SCALABILITY_OsDisk...`    | Disk                        | Disco virtual que contiene el sistema operativo (Ubuntu).                                                      |


3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

    Cuando ejecutámos npm FibonacciApp.js así nomás en una terminal SSH, el proceso se asocia a tu sesión. Entonces, si cerrás la consola o se cae la conexión, se muere el proceso junto con ella. 
    Esto pasa porque el proceso no está corriendo como servicio ni en segundo plano. Por eso usamos herramientas como forever, pm2 o incluso un nohup para dejar el proceso corriendo en segundo plano aunque cierres sesión.

    Por seguridad, las VMs en Azure no permiten tráfico externo por cualquier puerto. Solo puertos permitidos explícitamente en el Network Security Group (NSG) podrán ser accedidos desde Internet. 
    Nuestra aplicación corre en el puerto 3000, que por defecto está bloqueado. Entonces, necesitamos agregar una regla de entrada (inbound rule) que permita tráfico TCP al puerto 3000 para que podamos entrar desde el navegador o Postman

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

    ### Maquina tamaño B1ls (1 vCPU, 0.5 GiB RAM)

    | **Input (n)** | **Tiempo (s)** |
    |---------------|----------------|
    | 1,000,000     | 15.95          |
    | 1,010,000     | 16.28          |
    | 1,020,000     | 16.50          |
    | 1,030,000     | 16.86          |
    | 1,040,000     | 17.32          |
    | 1,050,000     | 19.25          |
    | 1,060,000     | 18.21          |
    | 1,070,000     | 18.58          |
    | 1,080,000     | 18.87          |
    | 1,090,000     | 19.21          |
    
    ---
    
    ### Maquina tamaño B2ms (2 vCPUs, 8 GiB RAM)
    
    | **Input (n)** | **Tiempo (s)** |
    |---------------|----------------|
    | 1,000,000     | 12.73          |
    | 1,010,000     | 12.89          |
    | 1,020,000     | 12.97          |
    | 1,030,000     | 13.18          |
    | 1,040,000     | 13.80          |
    | 1,050,000     | 13.94          |
    | 1,060,000     | 14.07          |
    | 1,070,000     | 14.67          |
    | 1,080,000     | 14.70          |
    | 1,090,000     | 15.16          |

    

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

    ### Maquina tamaño B1ls (1 vCPU, 0.5 GiB RAM)

   ![Image](https://github.com/user-attachments/assets/1c1f3d8d-5e51-4dba-bd59-0391a7c35598)

    ### Maquina tamaño B2ms (2 vCPUs, 8 GiB RAM)

   ![Image](https://github.com/user-attachments/assets/bc0377dc-17b1-46f3-ac7d-35c2aff3181d)



6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.

   ### Maquina tamaño B1ls (1 vCPU, 0.5 GiB RAM)

   ![Image](https://github.com/user-attachments/assets/f0292cc9-0767-4c04-83c1-dc8480e4026f)

   ### Maquina tamaño B2ms (2 vCPUs, 8 GiB RAM)

   ![Image](https://github.com/user-attachments/assets/ebf74418-9862-4f0c-a131-ad6d2f6afbd2)




7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

   Aunque ambos tamaños pertenecen a la serie B (Burstable) de máquinas virtuales en Azure, están diseñados para casos de uso muy distintos en cuanto a capacidad de procesamiento, escalabilidad y estabilidad operativa.

    1. Capacidad de cómputo y memoria
       * El tamaño B1ls es una de las instancias más pequeñas de Azure, con 1 vCPU y 0.5 GiB de RAM, pensada exclusivamente para cargas de trabajo extremadamente ligeras como pruebas simples, scripts de automatización o servidores mínimos de desarrollo. 
       * En cambio, B2ms cuenta con 2 vCPUs y 8 GiB de RAM, lo que le permite ejecutar aplicaciones de producción con un requerimiento moderado de recursos, como servidores web, APIs, y cargas interactivas.
    
    2. Manejo del rendimiento (créditos de CPU)
       * Ambas instancias funcionan bajo el modelo burstable, es decir, acumulan créditos de CPU cuando tienen bajo uso y los consumen al momento de necesitar más procesamiento. 
       Sin embargo, B2ms no solo tiene más vCPUs, sino que su capacidad para sostener picos de uso es mucho mayor, ya que acumula y mantiene más créditos, mientras que B1ls los agota muy rápidamente y luego queda limitado a un rendimiento mínimo sostenido.
    
    3. Estabilidad bajo carga
       * En aplicaciones que requieren tiempo de CPU constante o cálculos intensivos (como procesamiento de datos o funciones matemáticas complejas), B1ls se degrada rápidamente, presentando tiempos de respuesta inestables o lentitud significativa tras agotar sus créditos. 
       * B2ms, en cambio, ofrece un desempeño mucho más estable y sostenido, incluso si la carga se mantiene constante, lo que lo hace más adecuado para entornos donde la disponibilidad y el rendimiento no deben fluctuar.

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

    Parcialmente, al pasar de una máquina B1ls (1 vCPU, 0.5 GB RAM) a una B2ms (2 vCPU, 8 GB RAM), se observó una reducción clara en los tiempos de ejecución de la función Fibonacci y un uso más equilibrado del CPU.
    Por ejemplo, vemos que para el mismo rango de entradas (de 1,000,000 a 1,090,000), los tiempos pasaron de un promedio de 17.5s (en B1ls) a cerca de 13.5s (en B2ms). Esto indica que la función se beneficia de una mayor capacidad de cómputo, particularmente en términos de CPU y RAM disponibles.
    Aumentar la VM mejora el rendimiento, pero solo hasta cierto punto. Ya que si el código no escala bien con más recursos, seguir aumentandole hardware se vuelve una solución cara e ineficiente.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

   Cambiar el tamaño de una VM impacta directamente en la infraestructura en varios sentidos:

    1. **Costo:**
    El tamaño B2ms tiene un costo significativamente mayor que el B1ls. Esto puede ser prohibitivo si se escala horizontalmente o si se usa por largos períodos.
    
    2. **Consumo de recursos:**
    Una máquina más grande consume más recursos del proveedor (Azure en este caso), lo cual puede afectar la planificación presupuestaria o los límites de recursos de una suscripción.
    
    3. **Interrupciones:**
    Para cambiar el tamaño de una VM, normalmente se requiere reiniciar la máquina, lo que puede generar downtime si no se gestiona correctamente.
    
    4. **Sobreaprovisionamiento:**
    Si no se necesita toda la capacidad extra, se estaría pagando por recursos no utilizados, lo que no es óptimo.

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

    Sí, hubo mejora en ambos aspectos, y se puede justificar así:
    
    1. Consumo de CPU:
    
       * En B1ls, el CPU estaba al 97.32%, lo que indica saturación total. 
       * En B2ms, bajó a 50.52%, lo que significa que hay suficiente capacidad sobrante para mantener la app estable y posiblemente aceptar más peticiones simultáneas.
    
    2. Tiempos de respuesta:
    
       * Promedio en B1ls: 15.8s 
       * Promedio en B2ms: 12.8s
    
    Esto significa una mejora de aproximadamente 19% en el tiempo de respuesta, lo cual es notable para cargas de este tipo. Además, no hubo fallos en ninguna de las ejecuciones, lo que confirma la estabilidad del sistema incluso con alta demanda de procesamiento.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

    Al reducir la cantidad de ejecuciones paralelas vemos que el resultado del tiempo sigue siendo casi el mismo sin haber una notable mejora

    ![Image](https://github.com/user-attachments/assets/276c2bd6-43b8-4f8c-a0b9-7cb41a08e6e4)

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)
    
![Image](https://github.com/user-attachments/assets/c121d7ca-3af4-411b-a11d-af29da691cd6)
    
![Image](https://github.com/user-attachments/assets/6a7f374e-af6e-4b9c-b23c-9085541c34ee)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

![Image](https://github.com/user-attachments/assets/9a089ab1-8ab3-498b-ba85-819efdcd1208)

![Image](https://github.com/user-attachments/assets/fc85d94a-3d8a-4701-8775-c0de2bb3fcc1)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

![Image](https://github.com/user-attachments/assets/13a06564-ef6d-49db-b77f-9acaaa701b4c)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

![Image](https://github.com/user-attachments/assets/65e5739a-1d68-408e-90bb-7bbaf307fbde)

![Image](https://github.com/user-attachments/assets/3777a9d1-a0e2-4037-9831-a1ce70ec1c17)

![Image](https://github.com/user-attachments/assets/f631cc38-dc00-4797-abd5-0d73caa09851)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

![Image](https://github.com/user-attachments/assets/bce00ddf-53c7-48f3-9c06-ef427759ea6e)

![Image](https://github.com/user-attachments/assets/4749efbd-72da-476f-96e4-cd8cc333bbbc)

![Image](https://github.com/user-attachments/assets/a5aec92e-6e08-449d-984d-73f8646e89bf)

![Image](https://github.com/user-attachments/assets/7b194dbb-d9f7-40e5-a8c6-b428b8699111)

![Image](https://github.com/user-attachments/assets/b4f5ed68-5a17-483f-b72c-172f3257d358)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

    ![](images/part2/part2-vm-create1.png)

    Creamos las 3 maquinas virtuales diferenciandolas por el nombre y donde cada una va abarcar una zona de disponibilidad diferente

    ![Image](https://github.com/user-attachments/assets/ec8067b3-6fb8-4074-b0f4-4d01a21f263a)
    
    ![Image](https://github.com/user-attachments/assets/8d01b0c2-06ba-468f-b23e-d9fed6d2752c)
    
    ![Image](https://github.com/user-attachments/assets/25b8a5e6-a04d-4e24-9919-007d6b0df51f)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

    ![](images/part2/part2-vm-create2.png)

    Verificamos que se ha seleccionado la Virtual Network y la Subnet creadas anteriormente. Adicionalmente asignamos una IP pública y habilitamos la redundancia de zona.

    ![Image](https://github.com/user-attachments/assets/02064004-6333-4c54-9d21-2a69e0d9cdde)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

    ![](images/part2/part2-vm-create3.png)

    Damos a la opcion de avanzado, creamos nuestro network security group asignando las caracteristicas dadas, y consiguiente creamos la inbound rule para habilitar el trafico en el puerto 3000

    ![Image](https://github.com/user-attachments/assets/ce956e72-c969-4db8-939b-b95b5d907350)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

    ![](images/part2/part2-vm-create4.png)

    Asignamos nuestro balanceador de carga y el grupo de back-end creado anteriormente

    ![Image](https://github.com/user-attachments/assets/51e07607-bc5c-407b-867d-d23cc1bf8dfc)

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

    Creamos la maquina y verficamos que la implementacion se haya hecho correctamente

    ![Image](https://github.com/user-attachments/assets/658c7400-0dc9-4bc2-b84f-a69886e81c7a)

    Ejecutamos y accedemos a la maquina, luego clonamos el repositorio dado y ejecutamos el comando `curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash`
    
    ![Image](https://github.com/user-attachments/assets/073929b1-3f64-4c9c-bcc0-50ef3daa2080)
    
    Seguimos ejecutando `source /home/vm1/.bashrc`, luego instalamos node y las dependencias para ejecutar la aplicacion con el comando `npm install forever -g`

    ![Image](https://github.com/user-attachments/assets/aa5b0dd9-5975-4ce9-8a70-941cb2cc54fb)

    Por ultimo ejecutamos la aplicacion con el comando `forever start FibonacciApp.js`
    
    ![Image](https://github.com/user-attachments/assets/cc8124c2-bcf1-4a54-9d90-b7ddd2c50ec4)

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

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?

    Azure tiene dos tipos principales de balanceadores de carga:

    1. **Load Balancer Público**
       Se usa para distribuir tráfico desde internet hacia tus recursos internos, como máquinas virtuales.
       Ejemplo: Si estás montando una app web, este balanceador reparte el tráfico que entra desde fuera.

    2. **Load Balancer Interno**
       Se usa para distribuir tráfico dentro de una red virtual privada (VNet).
       Ideal para microservicios, bases de datos, o aplicaciones internas que no deben exponerse a internet.

    -------------------------

    **SKU = Stock Keeping Unit**, pero en Azure se usa para diferenciar la versión o tipo de recurso que estás usando. En el caso del Load Balancer, hay dos SKUs:

    **Basic**
    * Más limitado y gratuito. 
    * No soporta zonas de disponibilidad.
    * No se puede usar con redes virtuales globales (peering limitado).
    * No se puede actualizar a “Standard”.
    
    **Standard** 
    * Nivel empresarial.
    * Más confiable, escala automáticamente. 
    * Soporta zonas de disponibilidad (alta disponibilidad real). 
    * Incluye métricas, diagnósticos y control de acceso (NSGs). 
    * Requiere una IP pública “Standard”.
  
    ------------

    El balanceador de carga necesita una IP pública cuando se quiere exponer una aplicación o servicio a internet, ya que esta IP actúa como el punto de entrada visible desde fuera de la red virtual. Sin una IP pública, el balanceador no podría recibir solicitudes externas ni distribuirlas a los recursos internos, como máquinas virtuales o contenedores, que se encuentran dentro de la red privada de Azure. 
    
    Por eso, en escenarios donde se necesita acceso público (como páginas web o APIs), la IP pública es esencial para establecer la conexión entre los usuarios y los servicios alojados.

* ¿Cuál es el propósito del *Backend Pool*?

    El **Backend Pool** es el grupo de recursos que reciben el tráfico que llega al balanceador. Básicamente, ahí es donde metémos las máquinas virtuales, instancias de escalado automático (VMSS) u otros servicios que van a procesar las solicitudes del usuario.

* ¿Cuál es el propósito del *Health Probe*?

    El **Health Probe** se encarga de monitorear la salud de las instancias dentro del Backend Pool. Cada cierto tiempo, el Load Balancer hace chequeos de HTTP, TCP o HTTPS para asegurarse de que esas instancias estén vivas y funcionando correctamente.

* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.

    La **Load Balancing Rule** (regla de balanceo de carga) en Azure define cómo y cuándo se distribuye el tráfico desde una dirección IP frontal (frontend) hacia el grupo de instancias del Backend Pool.
    Su propósito es establecer el comportamiento de red deseado para garantizar una distribución eficiente del tráfico entre los recursos del backend.

    -----------

    Azure permite configurar una opción llamada "sesión persistente" (Session Persistence o Affinity) en la regla de balanceo, con los siguientes tipos:

    * **None:** No hay persistencia. Cualquier solicitud puede ir a cualquier instancia. 
    * **Client IP:** El tráfico del mismo cliente (por IP) siempre se dirige a la misma instancia. 
    * **Client IP and Protocol:** Se mantiene la afinidad por dirección IP y tipo de protocolo.
    
    La persistencia es útil cuando una aplicación mantiene estado, por ejemplo, sesiones de usuario en memoria, que deben atenderse siempre desde la misma instancia.

* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?

    Una **Virtual Network** (VNet) en Azure es una red privada definida por el usuario dentro de la nube, que permite que los recursos como máquinas virtuales, bases de datos o servicios de Azure se comuniquen entre sí de manera segura. Es el equivalente a una red física tradicional en un datacenter, pero implementada de forma virtual.

    Dentro de una VNet, se pueden crear **subredes** (Subnets), que dividen el espacio de direcciones IP de la VNet en segmentos más pequeños, permitiendo una organización lógica de recursos y la aplicación de reglas de seguridad más específicas.

    * El **Address Space** es el rango total de direcciones IP disponibles para la red virtual (por ejemplo, 10.0.0.0/16). 
    * El **Address Range** o prefijo de una subnet (por ejemplo, 10.0.1.0/24) define un subconjunto del address space asignado a una subred específica.

    Estas configuraciones permiten definir cómo se estructuran las direcciones IP dentro de la red, cómo se segmenta el tráfico, y facilitan el control del enrutamiento y la seguridad

* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?

    Las **Availability Zones** (Zonas de Disponibilidad) son ubicaciones físicas separadas dentro de una misma región de Azure, cada una con su propio suministro eléctrico, red y refrigeración. Su propósito principal es garantizar la alta disponibilidad y tolerancia a fallos: si una zona falla (por ejemplo, por un corte eléctrico), las otras siguen funcionando.

    Seleccionar **tres zonas diferentes** permite distribuir los recursos críticos (como máquinas virtuales, bases de datos o balanceadores de carga) de forma redundante entre ellas. Esto mejora significativamente la resiliencia del sistema frente a fallos zonales, asegurando la continuidad del servicio incluso si una zona completa se vuelve inaccesible.

    ------------

    Una IP pública zone-redundant es una dirección IP configurada para estar disponible en todas las zonas de disponibilidad de una región. Esto significa que puede enrutar tráfico hacia recursos ubicados en cualquiera de las zonas sin depender de una zona específica.

* ¿Cuál es el propósito del *Network Security Group*? 

    Un **Network Security Group** en Azure actúa como un firewall lógico a nivel de red, permitiendo o denegando el tráfico hacia y desde los recursos de una red virtual. El NSG contiene reglas que controlan el tráfico entrante y saliente basado en criterios como puertos, direcciones IP, protocolos y etiquetas de Azure.

    Su propósito principal es proteger los recursos (por ejemplo, máquinas virtuales o subredes completas) frente a accesos no autorizados o tráfico malicioso, aplicando políticas de seguridad que son fáciles de configurar, escalar y mantener.

* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.











