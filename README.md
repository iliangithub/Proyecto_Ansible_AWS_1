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

Básicamente, los comandos Ad-Hoc son formas de interactuar con las máquinas a través de comandos, **pero sin tener que recurrir al playbook**. Se utiliza para hacer comandos simples y rápidos.

### 1.7.1 Explicación profunda de como funcionan los comandos en Ansible.

1. `ansible` (Para hacer entender al intérprete de comandos de que comandos estamos hablando, es obligatorio.
2. `[host]` (Donde se van a ejecutar o aplicar los comandos).
  - `all`
  - `*`
  - `grupo_de_hosts`.
  - `el_nombre_de_un_host`.
  - `la ip privada de un host`.
  - `grupo_de_grupos`.

3. `-m` (módulo).
4. `-i` (inventario).

5. `-a`
*A veces, podemos encontrar también el "-a" le indica a Ansible que le pases argumentos adicionales al módulo que estás utilizando. Esos argumentos pueden ser opciones o parámetros específicos que el módulo necesita para funcionar.*

por ejemplo. `ansible all -m shell -a "ping -c 4 google.com -i inventario"`

### 1.7.2 ¿Qué es un módulo?

Si ejecutamos este comando, `ansible-doc -l` nos listará todos los módulos disponibles...
Son muchísimos.

En cualquier caso, un módulo es una pieza de código, con una instrucción que ejecuta directamente en el terminal del sistema.

Eso quiere decir, que si yo hago un `ansible -m ping -i inventario`.
Va a ejecutar a los Hosts del inventario, el comando o lo que tenga que hacer del módulo ping, lo que esté escrito en ese módulo y las instrucciones que sean dentro.

Es decir, si yo decido consultar **EL CONTENIDO DEL MÓDULO PING, me encuentro con esto:**

```
#!/usr/bin/python
from ansible.module_utils.basic import AnsibleModule

def main():
    module = AnsibleModule(argument_spec={}, supports_check_mode=True)
    result = {"ping": "pong"}
    module.exit_json(**result)

if __name__ == '__main__':
    main()
```

Esto es lo que realmente está ejecutando por dentro.
Tenemos 2 opciones para ver el contenido de dentro:

- `ansible-doc -t module ping`

Y te imprime esto:

```
> MODULE ansible.builtin.ping (/usr/lib/python3/dist-packages/ansible/modules/p>

  A trivial test module, this module always returns `pong' on
  successful contact. It does not make sense in playbooks, but it is
  useful from `/usr/bin/ansible' to verify the ability to
  login and that a usable Python is configured.
  This is NOT ICMP ping, this is just a trivial test module that
  requires Python on the remote-node.
  For Windows targets, use the ansible.windows.win_ping
  module instead.
  For Network targets, use the ansible.netcommon.net_ping
  module instead.

OPTIONS (red indicates it is required):

   data    Data to return for the `ping' return value.
           If this parameter is set to `crash', the module will cause
           an exception.
        default: pong
        type: str

ATTRIBUTES:

        check_mode:
        description: Can run in check_mode and return changed status prediction>
          target, if not supported the action will be skipped.
        support: full

        diff_mode:
        description: Will return details on what has changed (or possibly needs>
          check_mode), when in diff mode
        support: none

        platform:
        description: Target OS/families that can be operated against
        platforms: posix
        support: N/A
```

- `nano /usr/lib/python3.*/site-packages/ansible/modules/system/ping.py`


Estos por ejemplo, son del **AWS Collection**. Si no los tuviera, y necesitara instalarlos:
`ansible-galaxy collection install amazon.aws` y si por algún casual los usara en mi playbook, necesito importar la colección.

```
collections:
  - amazon.aws
```

```
amazon.aws.autoscaling_group                                                   >
amazon.aws.autoscaling_group_info                                              >
amazon.aws.aws_az_info                                                         >
amazon.aws.aws_caller_info                                                     >
amazon.aws.aws_region_info                                                     >
amazon.aws.backup_plan                                                         >
amazon.aws.backup_plan_info                                                    >
amazon.aws.backup_restore_job_info                                             >
amazon.aws.backup_selection                                                    >
amazon.aws.backup_selection_info                                               >
amazon.aws.backup_tag                                                          >
amazon.aws.backup_tag_info                                                     >
amazon.aws.backup_vault                                                        >
amazon.aws.backup_vault_info                                                   >
amazon.aws.cloudformation                                                      >
amazon.aws.cloudformation_info                                                 >
amazon.aws.cloudtrail                                                          >
amazon.aws.cloudtrail_info                                                     >
amazon.aws.cloudwatch_metric_alarm                                             >
amazon.aws.cloudwatch_metric_alarm_info                                        >
amazon.aws.cloudwatchevent_rule                                                >
amazon.aws.cloudwatchlogs_log_group                                            >
amazon.aws.cloudwatchlogs_log_group_info                                       >
amazon.aws.cloudwatchlogs_log_group_metric_filter                              >
amazon.aws.ec2_ami                                                             >
amazon.aws.ec2_ami_info                                                        >
amazon.aws.ec2_eip                                                             >
amazon.aws.ec2_eip_info                                                        >
amazon.aws.ec2_eni                                                             >
amazon.aws.ec2_eni_info                                                        >
amazon.aws.ec2_import_image                                                    >
amazon.aws.ec2_import_image_info                                               >
amazon.aws.ec2_instance                                                        >
amazon.aws.ec2_instance_info                                                   >
amazon.aws.ec2_key                                                             >
amazon.aws.ec2_key_info                                                        >
amazon.aws.ec2_metadata_facts                                                  >
amazon.aws.ec2_security_group                                                  >
amazon.aws.ec2_security_group_info                                             >
amazon.aws.ec2_snapshot                                                        >
amazon.aws.ec2_snapshot_info                                                   >
amazon.aws.ec2_spot_instance                                                   >
amazon.aws.ec2_spot_instance_info                                              >
amazon.aws.ec2_tag                                                             >
amazon.aws.ec2_tag_info                                                        >
amazon.aws.ec2_vol                                                             >
amazon.aws.ec2_vol_info                                                        >
amazon.aws.ec2_vpc_dhcp_option                                                 >
amazon.aws.ec2_vpc_dhcp_option_info                                            >
amazon.aws.ec2_vpc_endpoint                                                    >
amazon.aws.ec2_vpc_endpoint_info                                               >
amazon.aws.ec2_vpc_endpoint_service_info                                       >
amazon.aws.ec2_vpc_igw                                                         >
amazon.aws.ec2_vpc_igw_info                                                    >
amazon.aws.ec2_vpc_nat_gateway                                                 >
amazon.aws.ec2_vpc_nat_gateway_info                                            >
amazon.aws.ec2_vpc_net
```

Y también tenemos la colección de ansible que siempre te viene por defecto sin importar que versión instales.

```
ansible.builtin.add_host                                                       >
ansible.builtin.apt                                                            >
ansible.builtin.apt_key                                                        >
ansible.builtin.apt_repository                                                 >
ansible.builtin.assemble                                                       >
ansible.builtin.assert                                                         >
ansible.builtin.async_status                                                   >
ansible.builtin.blockinfile                                                    >
ansible.builtin.command                                                        >
ansible.builtin.copy                                                           >
ansible.builtin.cron                                                           >
ansible.builtin.deb822_repository                                              >
ansible.builtin.debconf                                                        >
ansible.builtin.debug                                                          >
ansible.builtin.dnf                                                            >
ansible.builtin.dnf5                                                           >
ansible.builtin.dpkg_selections                                                >
ansible.builtin.expect                                                         >
ansible.builtin.fail                                                           >
ansible.builtin.fetch                                                          >
ansible.builtin.file                                                           >
ansible.builtin.find                                                           >
ansible.builtin.gather_facts                                                   >
ansible.builtin.get_url                                                        >
ansible.builtin.getent                                                         >
ansible.builtin.git                                                            >
ansible.builtin.group                                                          >
ansible.builtin.group_by                                                       >
ansible.builtin.hostname                                                       >
ansible.builtin.import_playbook                                                >
ansible.builtin.import_role                                                    >
ansible.builtin.import_tasks
```

### 1.7.2 ¿Porqué se utiliza "-m ping" en vez de: "ansible.builtin.ping"?

Entonces, puedes usar tanto:

```
ansible all -m ping
```

```
ansible all -m ansible.builtin.ping
```

**Se usa el primero porque es más corto** y literalmente se utiliza en comandos.
**SIN EMBARGO** en playbooks, se utiliza `ansible.builtin.ping`.

### 1.7.3 ¿Y si quiero realmente ejecutar comandos?

Espero que haya quedado claro que anteriormente, el módulo ping, precisamente muy ping no era, ese módulo, ejecutama un ping, hacia los servidores, no que los servidores ejecuten pings hacia el exterior.

```
ansible all -m command -a "ping -c 4 google.com" -i inventario
```

### 1.7.4 El ejercicio en sí.

Voy a instalar httpd, A.K.A apache2 a las máquinas:

```
ansible grupo_con_nombre_aleatorio -m ansible.builtin.yum -a "name=httpd state=present" -i inventario
```

Y nos da un error de que no somos usuario root:

```
servidores_de_ansible02 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.9"
    },
    "changed": false,
    "msg": "This command has to be run under the root user.",
    "results": []
}
```

recordemos, si miramos el inventario que el usuario en efecto, no es root, `es ec2-user`.
No vamos a modificar eso, pero ese es el motivo, para solucionarlo tenemos que usar --become.

```
ansible grupo_con_nombre_aleatorio -m ansible.builtin.yum -a "name=httpd state=present" -i inventario --become
```

Una vez lo ejecute, el output:

```
servidores_de_ansible01 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.9"
    },
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
        "Installed: httpd-tools-2.4.62-4.el9.x86_64",
        "Installed: apr-1.7.0-12.el9.x86_64",
        "Installed: centos-logos-httpd-90.8-2.el9.noarch",
        "Installed: mailcap-2.1.49-5.el9.noarch",
        "Installed: httpd-2.4.62-4.el9.x86_64",
        "Installed: mod_http2-2.0.26-4.el9.x86_64",
        "Installed: httpd-core-2.4.62-4.el9.x86_64",
        "Installed: apr-util-1.6.1-23.el9.x86_64",
        "Installed: apr-util-bdb-1.6.1-23.el9.x86_64",
        "Installed: httpd-filesystem-2.4.62-4.el9.noarch",
        "Installed: mod_lua-2.4.62-4.el9.x86_64",
        "Installed: apr-util-openssl-1.6.1-23.el9.x86_64"
    ]
}
```

Ha funcionado.

Si vuelvo a repetir el comando, me lo devuelve en verde:

```
servidores_de_ansible01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.9"
    },
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
[WARNING]: Platform linux on host servidores_de_ansible02 is using the
discovered Python interpreter at /usr/bin/python3.9, but future installation of
another Python interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
servidores_de_ansible02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.9"
    },
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
```

Por último, lo que yo podría hacer es hacer un "copy" de forma local, y pegarlo en el /var/www/html/index.html"

```
ansible grupo_con_nombre_aleatorio -m ansible.builtin.copy -a "src=index.html dest=/var/www/html/index.html" -i inventario --become
```

## 1.8 Playbook & Módulos. (YAML o YML).

Para crear el Playbook:

En Ansible se puede crear un playbook y llamarlo como queramos, siempre y cuando usemos la extensión .yml o .yaml

Importante, a cada acción se le llama play, y pues playbook a la lista de plays.

```
- hosts: grupo_con_nombre_aleatorio
  tasks:
    - yum:
        name: httpd
        state: present
```

Si quisiera hacer más tareas:

```
---
- name: Configurar servidor web
  hosts: grupo_con_nombre_aleatorio
  become: yes  # Elevar privilegios, root
  tasks:
    - name: Instalar Apache (httpd)
      yum:
        name: httpd
        state: latest

    - name: Desplegar configuración de Apache
      copy:
        src: file/httpd.conf  # Asegúrate de que este archivo exista
        dest: /etc/httpd/httpd.conf
        owner: root
        group: root
        mode: '0644'

- name: Configurar base de datos
  hosts: base_de_datos
  become: yes  # Necesario para instalar paquetes
  tasks:
    - name: Instalar PostgreSQL
      yum:
        name: postgresql
        state: latest
```

Para ejecutarlo:

```
ansible-playbook -i inventario nombre_playbook.yaml
```

>[!WARNING]
>Pero, recordemos, que yo dije de NO usar, como en los Ad Hocs los módulos de forma directa, como `-m yum`, si no, utilizar más bien: `-m ansible.builtin.yum`.
> ```
> ---
>- name: Configurar servidor web
>  hosts: grupo_con_nombre_aleatorio
>  become: yes  # Elevar privilegios
>  tasks:
>    - name: Instalar Apache (httpd)
>      ansible.builtin.yum:
>        name: httpd
>        state: latest
>
>    - name: Desplegar configuración de Apache
>      ansible.builtin.copy:
>        src: file/httpd.conf  # Asegúrate de que este archivo exista
>        dest: /etc/httpd/httpd.conf
>        owner: root
>        group: root
>        mode: '0644'
>
>- name: Configurar base de datos
>  hosts: base_de_datos
>  become: yes  # Necesario para instalar paquetes
>  tasks:
>    - name: Instalar PostgreSQL
>      ansible.builtin.yum:
>        name: postgresql
>        state: latest
>
> ```

## 1.9 Módulos, encontrarlos, usarlos y solucionar problemas.

https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html

Primero vamos a ver el Module Index, y aquí están todos, son clasificados por secciones, cloud, copy, file, archive, etc...

Por ej, me busco el del módulo copiar y lo pego:

![image](https://github.com/user-attachments/assets/4b29b297-479f-477d-a5c3-a49a5c6cd263)

Para modificarlo o añadir cosas, nos vamos a parámetros en la página web:

![image](https://github.com/user-attachments/assets/3155fda3-983e-4e98-afe3-d2f31fc38688)

Y como podemos ver en parámetros, hay algunos que son obligatorios:

![image](https://github.com/user-attachments/assets/3a830901-d275-452c-b5e2-c3aaca5538cd)

Este es el único obligatorio, por raro que parezca "src" no es obligatorio, por el simple hecho de que hay otras formas de copiar, podríamos buscar por el contenido del archivo "content" o que ponga el contenido directamente en el desitno.

### 1.9.1 Dependencias módulos.

Algunos módulos tienen dependencias, sobre todo los que son de Python. Por ejemplo:

![image](https://github.com/user-attachments/assets/ff8b24b5-6115-4c14-b681-edcce2c56333)

La de MySQL, https://docs.ansible.com/ansible/2.9/modules/mysql_db_module.html#mysql-db-module :

Si ejecuto el playbook:

*MySQL module is required: for Python 2.7 either PyMySQL, or MySQL-python*

Si hubieramos leido la documentación bien, aparecen requisitos:

![image](https://github.com/user-attachments/assets/ac89fa76-9e59-426a-b0c5-d673d91b478f)

Entonces, tenemos que ir a la máquina CentOS, e buscar los paquetes:

```
yum search python | grep -i mysql
```

```
 python3-PyMySQL
```

Entonces, una vez sepamos la dependencia, pues lo vamos a poner en el playbook para que se instale.

Luego nos va a dar otro error y es que no sabrá a donde conectarse.

y esto lo ponemos en el playbook:

```
login_unix_socket: /var/lib/mysql/mysql.sock
```
## 1.10 Ansible Configuration.

## 1.11 Handler.

## 1.12 Roles.

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
