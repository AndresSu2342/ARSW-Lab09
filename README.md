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

   ![Image](https://github.com/user-attachments/assets/f71527b3-7a8c-4f89-a2ca-2ca920c45536)
   
   ![Image](https://github.com/user-attachments/assets/b42ee11a-ce20-45db-a3fa-bf2b9f0a9965)
   
   ![Image](https://github.com/user-attachments/assets/beb7ecc3-e5fc-455a-862c-c269f1bfcecd)
   
   ![Image](https://github.com/user-attachments/assets/087760dc-e999-449a-8870-67bb4e7bcc42)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

   ![Image](https://github.com/user-attachments/assets/75218613-6868-43b6-92cf-4f478d4e9cc3)
   
   ![Image](https://github.com/user-attachments/assets/fc178d50-9155-4daf-9a3a-f5e0e1d69142)

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).

   ![Image](https://github.com/user-attachments/assets/0d997464-ae02-4f66-a0ed-5b9e8fbf0dc5)
   
   ![Image](https://github.com/user-attachments/assets/74ce2278-4e7d-4108-b803-69959f8c01ea)
   
   ![Image](https://github.com/user-attachments/assets/56858e61-e431-4975-9cc1-1d56e947b305)

4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

   ![Image](https://github.com/user-attachments/assets/deb28da2-4f4f-47b6-8c3e-c58dafaf761a)

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

   ![Image](https://github.com/user-attachments/assets/523f5c94-a209-473d-8cb1-becfd760fc6e)

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

   ![Image](https://github.com/user-attachments/assets/033d9254-d816-4238-8196-733c91a06b17)
   
   ![Image](https://github.com/user-attachments/assets/845aea9f-d4ed-4c2b-8e58-e9005eaa8bac)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000   


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

   ![Image](https://github.com/user-attachments/assets/ab677ce3-e611-4ac8-b50d-529675d0678f)

   ![Image](https://github.com/user-attachments/assets/3db1a8cc-0194-4583-becf-7df4f9b0f3f9)

   ![Image](https://github.com/user-attachments/assets/0e3f4168-56dc-4311-a528-7890bb5a83cd)

   ![Image](https://github.com/user-attachments/assets/f0292cc9-0767-4c04-83c1-dc8480e4026f)


10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

   ![Image](https://github.com/user-attachments/assets/802730fc-9981-4b7a-819b-44c1aa6a6c74)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

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

   ![Image](https://github.com/user-attachments/assets/d5b65406-1052-4d6d-9466-adcf831a083e)

   ![Image](https://github.com/user-attachments/assets/ebf74418-9862-4f0c-a131-ad6d2f6afbd2)

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

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

2. ¿Brevemente describa para qué sirve cada recurso?

   Recurso | Tipo | ¿Para qué sirve?
   scalability_lab_KEY | SSH Key | Clave pública SSH que se usó para conectarse de forma segura a la VM.
   scalability_lab_KEY2 | SSH Key | Otra clave pública SSH (posiblemente creada por error o por probar otra conexión).
   VERTICAL-SCALABILITY | Virtual Machine | La máquina virtual real. Este es el "servidor" donde corre tu app Node.js.
   VERTICAL-SCALABILITY-ip | Public IP Address | La dirección IP pública que se usa para conectarte a la VM desde internet.
   VERTICAL-SCALABILITY-nsg | Network Security Group | Como un firewall: controla qué tráfico (puertos y protocolos) puede entrar o salir de la VM.
   VERTICAL-SCALABILITY-vnet | Virtual Network (VNet) | Red privada virtual donde está conectada tu VM. Permite que otros recursos se comuniquen dentro del mismo entorno sin exponer todo a internet.
   vertical-scalability851 | Network Interface (NIC) | Interfaz de red que conecta la VM a la red virtual. Es lo que permite que la VM tenga una IP y pueda "hablar" con el mundo.
   VERTICAL-SCALABILITY_OsDisk... | Disk | El disco duro virtual que contiene el sistema operativo de la VM (Ubuntu en este caso).

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

    Cuando ejecutámos npm FibonacciApp.js así nomás en una terminal SSH, el proceso se asocia a tu sesión. Entonces, si cerrás la consola o se cae la conexión, se muere el proceso junto con ella. 
    Esto pasa porque el proceso no está corriendo como servicio ni en segundo plano. Por eso usamos herramientas como forever, pm2 o incluso un nohup para dejar el proceso corriendo en segundo plano aunque cierres sesión.

    Por seguridad, las VMs en Azure no permiten tráfico externo por cualquier puerto. Solo puertos permitidos explícitamente en el Network Security Group (NSG) podrán ser accedidos desde Internet. 
    Nuestra aplicación corre en el puerto 3000, que por defecto está bloqueado. Entonces, necesitamos agregar una regla de entrada (inbound rule) que permita tráfico TCP al puerto 3000 para que podamos entrar desde el navegador o Postman

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.
5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

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

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.











