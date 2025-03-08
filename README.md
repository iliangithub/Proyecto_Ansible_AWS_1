# 1.0 El proyecto.

Voy a desplegar una máquina con Ansible en AWS, que me permita controlar de forma remota.

## 1.1 Instancia EC2 de Ansible.

- Nombre: `Ansible_control_EC2`.
- AMI: `Ubuntu Server 22.04 LTS (HVM), SSD Volume Type`.
- Tipo de instancia: `t2.micro`.
- Crear un nuevo par de claves (RSA, .pem).

**Editamos la configuración de red:**
  ![image](https://github.com/user-attachments/assets/8884a75c-1307-4378-9c9c-802c6c5e192b)

y ya. La creamos.

## 1.2 Instancias (3) EC2 de CentOs.
- Número de instancias: `3`.
- Nombre: `servidores_de_ansible`.
- AMI: `CentOS Stream 9 (x86_64)`.
- Tipo de instancia: `t2.micro`.
- Par de clave: `cliente`,`RSA`,`.pem`.

Editamos igualmente la configuración de red:

![image](https://github.com/user-attachments/assets/ffe8a2c7-1c4d-4255-9669-8ae45d41285b)

La primera regla es que desde mi IP pueda hacer ssh para conectarme a la instancia y la otra que el pueda conectarse desde la máquina de Ansible (he puesto Ansible_SG)

Una vez creadas, **les voy a cambiar el nombre manualmente a cada máquina para que se distingan**:

![image](https://github.com/user-attachments/assets/1b64d87e-58e8-4434-8323-d6f96e32bcff)

## 1.3 Instalar Ansible en la máquina.

Estas máquinas ya tienen SSH, todas. **Voy a acceder a Ansible_Control_EC2**

Es importante recalcar, que cada vez que hagamos SSH a nuestra instancia EC2, nos preguntará:

`Are you sure you want to continue conecting (yes/no[fingerprint])?`

Esto significa que básicamente nos muestra el ID de la máquina, SHA256: xxxxxxxxx.

Cuando yo vuelva a hacer SSH no nos lo volverá a pedir, eso, es porque hay un fichero que almacena esas fingerprints.

![image](https://github.com/user-attachments/assets/b59262e0-b28c-492e-80a1-84532f11fdd9)

Voy a mostrar el fichero para que lo pueda leer:

![image](https://github.com/user-attachments/assets/a07259e8-9360-4f45-9ae2-11ba3e144156)

Voy a eliminar el contenido del fichero.

```
cat /dev/null > ~/.ssh/known_hosts
```

Entonces, cuando yo vuelva a:

```
ssh -i "ansible.pem" ubuntu@ec2-34-228-37-255.compute-1.amazonaws.com
```

> [!TIP]
>Sin embargo, el fingerprint que almacene dependerá de la máquina de AWS y de su configuración en ese momento. Si la máquina no ha cambiado su configuración o clave SSH (por ejemplo, no se ha renovado la clave del servidor de la instancia de AWS), el fingerprint debería ser el mismo que el anterior. Si la instancia de AWS se ha reiniciado o ha sido reemplazada, o si AWS ha cambiado la clave SSH, el fingerprint podría ser distinto.
>
>En resumen:
>
>- Si la clave SSH de la máquina de AWS no ha cambiado, el fingerprint será el mismo.
>- Si la clave ha cambiado (por ejemplo, si la máquina de AWS fue reemplazada o se ha cambiado la clave SSH), el fingerprint será diferente.
>

**Ansible hace pues un proceso similar para conectarse a las máquinas.**

En cualquier caso, ya estaremos dentro de la máquina de Ansible, así que vamos a buscar Ansible download.

Tenemos la opción de hacerlo usando Python:
https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#pipx-install

En cualquier caso:

>[!IMPORTANT]
>Este es el importante:
>https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html
>
>Y los comandos para Ubuntu son estos:
>```
> sudo apt update && sudo apt install -y software-properties-common && sudo add-apt-repository --yes --update ppa:ansible/ansible && sudo apt install -y ansible
>```

Básicamente, es como una librería de Python, usa el 2 y el 3.

Voy a ver que se haya descargado:

```
ansible --version
```

## 1.4 Crear Inventario. (Y cosas relacionadas con el inventario).
>[!TIP]
>Recordemos, que el inventario, no es más que un fichero en el que se almacenan los Hosts a los que se les va a aplicar los comandos o lo que sea.
>
>Y también recordemos que el inventario puede estar en 3 formatos diferentes:
>  - Inventario Dinámico.
>  - **YAML o YML. (la que vamos a utilizar)**
>  - Texto plano, .ini.

Entonces, ahora voy a crear un directorio en el que voy a poner el inventario.
```
mkdir prueba_ansible
```

https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html

Es importante saber, que hay variables que componen el inventario:
- ansible_host.
- ansible_port.
- ansible_user.
- ansible_password. (ESTO NO SE UTILIZA NUNCA, JAMÁS).
- ansible_ssh_private_key_file. (Lo que SI, vamos a utilizar).

Voy a crear otra subcarpeta y el inventario en sí, porque vamos a hacer bastantes inventarios.
Y recordemos, que puedo ponerle el nombre que quiera al inventario:

```
ubuntu@ip-172-31-22-134:~$ cd prueba_ansible/
ubuntu@ip-172-31-22-134:~/prueba_ansible$ mkdir ejercicio1
ubuntu@ip-172-31-22-134:~/prueba_ansible$ cd ejercicio1
ubuntu@ip-172-31-22-134:~/prueba_ansible/ejercicio1$ nano inventario
```
************************

**Dentro del inventario:**

Si miramos UN POCO hacia arriba, veremos las variables. `ansible_host` y `ansible_user`.
Necesitamos saber la IP de la máquina y el nombre de usuario.

![image](https://github.com/user-attachments/assets/ea03a478-f715-4fa5-8cb1-382a76395137)

Así está quedando el inventario, vamos a buscar los datos.

La IP va a ser evidentemente la privada:

![image](https://github.com/user-attachments/assets/d023355e-0cf0-401b-b28e-973f8e1ccb98)

Y el usuario, pues tenemos que buscar la AMI oficial que nos hemos suscrito o hemos contratado.

![image](https://github.com/user-attachments/assets/e6eda012-3c7f-4f27-b32a-97a9442d6acc)

el usuario es `ec2-user`. Como es evidente, si hay usuario, nos hará falta también la contraseña.

Aunque todavía no la tengamos, recordemos que no vamos a utilizar "ansible_password".
Está totalmente prohibido.

La contraseña es el .pem que creamos anteriormente con la instancia, `cliente_2` o cliente. Entonces, voy a copiarla y pegarla dentro de la instancia.

```
cat cliente_2.pem
```

copiamos el resultado.

![image](https://github.com/user-attachments/assets/62f8eda5-df14-4f02-b69f-9a032bc76ee9)

Creamos el fichero que habíamos puesto en el inventario. Y como vemos, **estoy usando vim**
para que se inserte todo. Para entrar en modo insertar, lo primero que haremos es presionar `i`.

- Para salir, ESC 
- y luego :wq + ENTER

Listo, el cliente tiene que estar encendido.

- `ansible servidor_de_ansible01 -m ping`

**Tenemos el primer error:**

![image](https://github.com/user-attachments/assets/a4b67499-1804-4b1f-9ab0-6a0b14ecb3bd)

>[!TIP]
>https://kodekloud.com/community/t/warning-provided-hosts-list-is-empty-only-localhost-is-available-note-that-the-implicit-localhost-does-not-match-all/12046/2
>
>You need to mention inventory file. Otherwise it will find in the default location /etc/ansible/ which is empty.
>

```
ansible servidores_de_ansible01 -m ping -i inventario
```

Es importante que hagamos un `ls` para comprobar que el inventario se llame así, que  si no encontrará el default que está vacío. Y haciendo un `cat inventario` para ver que están escritos correctamente el nombre de la key.pem y su IP privada.

**Tenemos el segundo error:**

![image](https://github.com/user-attachments/assets/37a8d545-f446-467c-81c0-2681721850f7)

```
sudo chmod 400 clientekey.pem
```

![image](https://github.com/user-attachments/assets/d658b846-4b68-4f46-96ce-a2acc5b7aeef)

`-m ping` es el módulo ping, este viene por defecto instalado.

### 1.4.1 Opcional, que sea automático y no pida el fingerprint:

>Para empezar, nos gustaría que sea realmente 100% automático y que no tengamos que interactuar o hacer algo. No que me pregunte si quier guardar el fingerprint.
>
>Para ello, deberíamos de modificar el fichero `/etc/ansible/ansible.cfg`
>Si hacemos un cat:
>
>```
>ubuntu@ip-172-31-22-134:~$ sudo cat /etc/ansible/ansible.cfg
># Since Ansible 2.12 (core):
># To generate an example config file (a "disabled" one with all default settings, commented >out):
>#               $ ansible-config init --disabled > ansible.cfg
>#
># Also you can now have a more complete file by including existing plugins:
># ansible-config init --disabled -t all > ansible.cfg
>
># For previous versions of Ansible you can check for examples in the 'stable' branches of each version
># Note that this file was always incomplete  and lagging changes to configuration settings
>
># for example, for 2.9: https://github.com/ansible/ansible/blob/stable-2.9/examples/ansible.cfg
>```
>
>Nos va a enseñar un comando interesante:
>
>`ansible-config init --disabled -t all > ansible.cfg`
>
>Este comando, va a crear otro fichero de configuración con todo deshabilitado por defecto.
>
>Pero vamos a hacer una copia de seguridad del que está por defecto antes de hacer eso:
>```
>sudo -i
>```
>
>```
>mv /etc/ansible/ansible.cfg /etc/ansible/ansible.cfg.backup
>```
>
>```
>ansible-config init --disabled -t all > /etc/ansible/ansible.cfg
>```
>
>Una vez creado:
>
>Tenemos que buscar:
>
>![image](https://github.com/user-attachments/assets/edb4901f-9438-4176-94a2-a05b74aa26ef)
>
>le quitamos el `;` y ponemos false:
>
>![image](https://github.com/user-attachments/assets/4cffa81c-7b03-4d77-8781-f3ea01125c28)
>
>Guardamos y ya.

## 1.5 Inventario 2, con varios Hosts y tareas más complejas:

Voy a crear entonces, otra carpeta llamada ejercicio2:

`cd ..` Ya que estabamos en `/ejercicio1`

```
cp -r ejercicio1 ejercicio2
```

Nos vamos al directorio de ejercicio2 y editamos el inventario:

```
all:
  hosts:
    servidores_de_ansible01:
      ansible_host: 172.31.27.195
      ansible_user: ec2-user
      ansible_ssh_private_key_file: clientekey.pem
    servidores_de_ansible02:
      ansible_host: 172.31.26.9
      ansible_user: ec2-user
      ansible_ssh_private_key_file: clientekey.pem
    servidores_de_ansible03:
      ansible_host: 172.31.22.250
      ansible_user: ec2-user
      ansible_ssh_private_key_file: clientekey.pem
```

Los nombres de los Hosts, no significan nada, podemos ponerles lo que queramos, simplemente le he puesto "servidores_de_ansible0X" porque así estaban puestas las instancias, y para pues saber cual es cuál, pero que podemos usar el nombre que queramos.

Así nos quedaría el inventario. Podríamos seguir ejecutando el comando uno por uno, host por host: `ansible servidores_de_ansible01 -m ping -i inventario`, `ansible servidores_de_ansible02 -m ping -i inventario`, pero queremos pues hacer una especie de grupo, por así decirlo para ejecutar para todos esos Host a la vez:

```
all:
  hosts:
    servidores_de_ansible01:
      ansible_host: 172.31.27.195
      ansible_user: ec2-user
      ansible_ssh_private_key_file: clientekey.pem
    servidores_de_ansible02:
      ansible_host: 172.31.26.9
      ansible_user: ec2-user
      ansible_ssh_private_key_file: clientekey.pem
    servidores_de_ansible03:
      ansible_host: 172.31.22.250
      ansible_user: ec2-user
      ansible_ssh_private_key_file: clientekey.pem

  children:
    grupo_con_nombre_aleatorio:
      hosts:
        servidores_de_ansible01:
        servidores_de_ansible02:
    grupo_hamburguesa:
      hosts:
        servidores_de_ansible03:
```

Y ejecutamos el comando ping de antes, pero usando 1 de los grupos:

```
ansible grupo_con_nombre_aleatorio -m ping -i inventario
```

También podemos hacer un grupo de grupos.

```
all:
  hosts:
    servidores_de_ansible01:
      ansible_host: 172.31.27.195
      ansible_user: ec2-user
      ansible_ssh_private_key_file: clientekey.pem
    servidores_de_ansible02:
      ansible_host: 172.31.26.9
      ansible_user: ec2-user
      ansible_ssh_private_key_file: clientekey.pem
    servidores_de_ansible03:
      ansible_host: 172.31.22.250
      ansible_user: ec2-user
      ansible_ssh_private_key_file: clientekey.pem

  children:
    grupo_con_nombre_aleatorio:
      hosts:
        servidores_de_ansible01:
        servidores_de_ansible02:
    grupo_hamburguesa:
      hosts:
        servidores_de_ansible03:
    grupo_que_engloba_los_dos:
      children:
        grupo_con_nombre_aleatorio:
        grupo_hamburguesa:

```

```
ansible grupo_que_engloba_los_dos -m ping -i inventario
```

>[!IMPORTANT]
>También tenemos la posibilidad de hacer un **`all`**:
>
>All está declarada al principio del todo en el fichero de inventario:
>
>```
>ansible all -m ping -i inventario
>```
>
>```
>ansible '*' s -m ping -i inventario
>```
>
>O también podemos hacer que sean todos los hosts que empiecen por una palabra:
>
>```
>ansible 'web*' s -m ping -i inventario
>```

## 1.6 Inventario 3. Variables:

Evidentemente, se utiliza para cuando se repiten las cosas, y ahorrar así líneas de código.
Por ejemplo, antes se repite constantemente el `ansible_user: ec2-user` y `ansible_ssh_private_key_file: clientekey.pem `

El resultado final sería algo así:

```
all:
  vars:
    ansible_user: ec2-user
    ansible_ssh_private_key_file: clientekey.pem
  hosts:
    servidores_de_ansible01:
      ansible_host: 172.31.27.195
    servidores_de_ansible02:
      ansible_host: 172.31.26.9
    servidores_de_ansible03:
      ansible_host: 172.31.22.250

  children:
    grupo_con_nombre_aleatorio:
      hosts:
        servidores_de_ansible01:
        servidores_de_ansible02:
    grupo_hamburguesa:
      hosts:
        servidores_de_ansible03:
```

Sin embargo, en Udemy, él define las varaibles A NIVEL DE GRUPO:

```
all:
  hosts:
    servidores_de_ansible01:
      ansible_host: 172.31.27.195
    servidores_de_ansible02:
      ansible_host: 172.31.26.9
    servidores_de_ansible03:
      ansible_host: 172.31.22.250

  children:
    grupo_con_nombre_aleatorio:
      hosts:
        servidores_de_ansible01:
        servidores_de_ansible02:
    grupo_hamburguesa:
      hosts:
        servidores_de_ansible03:
    grupo_que_engloba_los_dos:
      children:
        grupo_con_nombre_aleatorio:
        grupo_hamburguesa:
      vars:
        ansible_user: ec2-user
        ansible_ssh_private_key_file: clientekey.pem
```

De esta forma, estás asignando pues a nivel de grupo las variables y NO A NIVEL DE TODOS.
**Lo cual es mejor y más seguro.**


## 1.7 Comandos Ad-Hoc

https://docs.ansible.com/ansible/latest/command_guide/intro_adhoc.html

Básicamente, los comandos Ad-Hoc son formas de interactuar con las máquinas a través de comandos, pero sin tener que recurrir al playbook. Se utiliza para hacer comandos simples y rápidos.

### 1.7.1 Explicación profunda de como funcionan los comandos en Ansible.

1. `ansible` (Para hacer entender al intérprete de comandos de que comandos estamos hablando, es obligatorio.
2. `[host]` (Donde se van a ejecutar o aplicar los comandos).
  - `all`
  - `*`
  - `grupo_de_hosts`
  - `el_nombre_de_un_host`
  - `la ip privada de un host`
  - `grupo_de_grupos`
### 1.7.2 Utilizar el "sudo" o escalar privilegios.

ansible all -m ansible.buildin.yum -a "name=httpd state=present" -i inventoy

## 1.8 Playbook & Módulos.

# 2.0 (OFF TOPIC) Problema con el que me he topado, pérdida de las claves .pem .

En resumen, formatee el ordenador y me he quedado sin las claves. No hay forma de cambiar el par-clave de la instancia, tampoco podemos contactar con Amazon en caso de que se nos pierda. Es en teoría imposible.

Yo aquí tengo varias opciones:
- Eliminar las instancias y crearlas de nuevo.
- Acceder AL VOLUMEN de la máquina y editar un fichero.

>[!IMPORTANT]
>Entonces, en este volumen, habrá un fichero: `.shh/authorized_keys`.
>Y para resumirlo mucho, en efecto, tenemos que acceder al volumen ¿Cómo?
>
>1. Tenemos que apagar la máquina afectada, la que no tenemos el par-clave.
>2. Hacer otra instancia, de rescate.
>3. Desacoplar el volumen de la máquina afectada, y conectarlo a la instancia de rescate.
>4. Editar el fichero en cuestión para añadirle el par de clave.
>5. Una vez editado, acoplamos de nuevo el volumen con el fichero editado con la instancia afectada.

**Para colmo, he terminado la instancia sin querer** Ya que me he puesto con este reto. Voy a hacerlo, terminarlo y demostrar que se puede.

**Es importante, que los dos estén en la misma zona de disponibilidad**.
Es decir, que si mi máquina está en `Zona de disponibilidad
us-east-1b` y mi otra máquina en otro, no funcionará.

Entonces, creo la máquina de rescate, sin más. Voy a también darle nombre a los volúmenes:

**PASO 1:**

![image](https://github.com/user-attachments/assets/856a1c80-3b74-4070-b454-1cfbe5be4e3f)

**********************************

**PASO 2:**

![image](https://github.com/user-attachments/assets/9206944c-5171-468d-86fd-556a8400878f)

Cómo hemos dicho la máquina afectada tiene que estar apagada, y desasociamos ese volumen.

Nos vamos a volúmenes:

**PASO 3**

![image](https://github.com/user-attachments/assets/090c5933-0b78-410b-8e48-2ac717362c3b)

![image](https://github.com/user-attachments/assets/a5ddaffe-2800-4bdd-85e3-2d70c59496fd)

Una vez lo desasocie, voy a asociarlo, pero con la máquina de rescate.

**PASO 4**

![image](https://github.com/user-attachments/assets/57dc26e7-052a-412a-a913-fb671276a2a7)

![image](https://github.com/user-attachments/assets/26fcc287-c5d3-4208-85fe-70295aadae43)

En nombre de dispositivo:

>[!TIP]
>Esto es un concepto interesante a explicar, básicamente en Windows las unidades de almacenamiento o particiones, usan nombres como `C: `, `E: `, `D: `.
>
>Es básicamente la nomenclatura de las particiones, volumenes, etc...
>En linux, no se usa esa "nomenclatura", usa otra:
>
> - `/dev/sda`, `/dev/sdb`, etc.: Representan discos físicos
>   - `/dev/sda1`, `/dev/sda2`, etc.: Son particiones dentro de un disco.
>
> Entonces, en nombre de dispositivo te dice que:
> - `/dev/sda1` ya está siendo utilizado, bueno, pues elijo el `/dev/sdd` y no pasa nada.
> - Es más, si tu pruebas con el /dev/sdb, no te va a funcionar aunque no aparezca como que no está disponible.

Ahora ya tenemos dos disco duros/volumenes a una misma instancia. Vamos a conectarnos...
Voy a abrir la terminal de Gitbash desde el escritorio, que es donde tengo la clave .pem de la máquina de rescate.

https://codigoencasa.com/aws-recuperar-llave-pem-curso-aws/

Ya estoy dentro de la máquina y ahora voy a hacer una carpeta de montaje de este nuevo volumen.
Hemos conectado a la máquina, pero no está montado.

`sudo su`

```
mkdir /mnt/tmp
```

y este comando lo he modificado A MIS preferencias, porque anteriormente en "nombre de dispostivo, pusimos `/dev/sdd`".

Si utilizo estos dos comandos, no va a funcionar: `mount /dev/sdd1 /mnt/tmp` o `mount /dev/sdd /mnt/tmp`

voy a poner este comando, para `lsblk`

Output:
```
NAME     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0      7:0    0 26.3M  1 loop /snap/amazon-ssm-agent/9881
loop1      7:1    0 73.9M  1 loop /snap/core22/1722
loop2      7:2    0 44.4M  1 loop /snap/snapd/23545
xvda     202:0    0    8G  0 disk
├─xvda1  202:1    0    7G  0 part /
├─xvda14 202:14   0    4M  0 part
├─xvda15 202:15   0  106M  0 part /boot/efi
└─xvda16 259:0    0  913M  0 part /boot
xvdd     202:48   0    8G  0 disk
├─xvdd1  202:49   0    7G  0 part
├─xvdd14 202:62   0    4M  0 part
├─xvdd15 202:63   0  106M  0 part
└─xvdd16 259:1    0  913M  0 part
```

Entonces, el volumen está identificado como xvdd y su partición principal es xvdd1.
Esto se debe a que, aunque en AWS haya puesto nombre de dispositivo `sdd`, como son versiones de Ubuntu más recientes pues se utiliza esta otra nomenclatura.

Literalmente lo que pone en el AWS:

![image](https://github.com/user-attachments/assets/4ee67dc4-da76-492b-8dd6-cc11d435f64a)

Finalmente se usa este comando:

```
mount /dev/xvdd1 /mnt/tmp
```

>[!TIP]
>¿Porqué se usa `tmp` en vez de `temp`?
>Ni idea. ChatGPT lo usa porque sí.
>

Una vez montado, voy a copiar el fichero `./.ssh/authorized_keys` DEL VOLUMEN DE RESCATE, hacia el afectado.

```
cp ./.ssh/authorized_keys /mnt/tmp/home/ubuntu/.ssh/authorized_keys
```

Una vez hecho eso, voy a cerrar la sesión, `exit` y `exit`.

![image](https://github.com/user-attachments/assets/5acf74d5-0590-4441-aba7-7c41719a9779)

Apago la máquina, me vuelvo a volúmenes y voy a desaociar el volumen ese y lo voy a volver a asociar a su instancia correspondiente.

**PASO 5**

![image](https://github.com/user-attachments/assets/8e9e7c91-205d-42f7-b52a-66d3a3b0d7c5)

**PASO 6**
![image](https://github.com/user-attachments/assets/3e95acb3-963e-439e-a635-43322a1b5f8b)

Pero esta vez, muy importante, recordemos que "nombre de dispositivo" antes tenía reservado el `/dev/sda1`, estaba ocupado porque allí estaba antes su volumen, ahora, evidentemente, no tiene nada, ningún volumen, pues lo pondremos allí.


![image](https://github.com/user-attachments/assets/a3b7bb55-0acc-4613-955d-c2d241c80c41)

eso es todo, ahora solo tenemos que iniciar la instancia y comprobar que efectivamente ha funcionado.

Si me voy a conectar...

El comando está compuesto por `ssh -i "el nombre anterior del archivo clave"`

Eso NO es lo que queremos, nosotros queremos usar el .pem nuevo, de la otra máquina de rescate, pues ya está, solo modificamos el comando anterior y ya:

`ssh -i "rescate.pem" ubuntu@ec2-44-223-xxx-xxx.compute-1.amazonaws.com`.

Como podemos comprobar hemos podido acceder efectivamente a la máquina:

![image](https://github.com/user-attachments/assets/69c54f3c-d37c-43f7-aea1-ae4deadc2740)
