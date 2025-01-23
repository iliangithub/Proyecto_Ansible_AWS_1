# 1.0 Introducción Ansible.

Ansible, no es un servicio, usa los módulos de Python, los scripts de Python para conectarse a la máquina y verter el código, y devolver un output.

Es necesario que la otra máquina tenga Python?

La Arquitectura de Ansible es simple. En la máquina que esté instalado Ansible, tenemos Playbooks, que es un archivo con las instrucciones y tareas que tiene que realizar.

Ese archivo, hará referencia a otro componente de la arquitectura Ansible, que es el Inventory, que es otro archivo pero en él aparecen los Hosts a los que se les va a ejecutar esos comandos, los Hosts, o el grupo de Hosts.

Y también en el Playbook hay otro componente de la arquitectura Ansible, que son los módulos.

Luego, Ansible utilizará la "Ansible config" y crear el script de Python, y luego pues se ejecutará.

# 2.0 El proyecto.

Voy a desplegar una máquina con Ansible en AWS, que me permita controlar de forma remota.

## 2.1 Instancia EC2 de Ansible.

- Nombre: `Ansible_control_EC2`.
- AMI: `Ubuntu Server 22.04 LTS (HVM), SSD Volume Type`.
- Tipo de instancia: `t2.micro`.
- Crear un nuevo par de claves (RSA, .pem).

**Editamos la configuración de red:**
- ![image](https://github.com/user-attachments/assets/8884a75c-1307-4378-9c9c-802c6c5e192b)

y ya. La creamos.

## 2.2 Instancias (3) EC2 de CentOs.
- Número de instancias: `3`.
- Nombre: `servidores_de_ansible`.
- AMI: `CentOS Stream 9 (x86_64)`.
- Tipo de instancia: `t2.micro`.
- Par de clave: `cliente`,`RSA`,`.pem`.

Editamos igualmente la configuración de red:

![image](https://github.com/user-attachments/assets/ffe8a2c7-1c4d-4255-9669-8ae45d41285b)

La primera regla es que desde mi IP pueda hacer ssh para conectarme a la instancia y la otra que el pueda conectarse desde la máquina de Ansible (he puesto Ansible_SG)

Una vez creadas, les voy a cambiar el nombre:

![image](https://github.com/user-attachments/assets/1b64d87e-58e8-4434-8323-d6f96e32bcff)

manualmente.

## 2.3 Instalar Ansible en la máquina.

Estas máquinas ya tienen SSH, todas. Voy a acceder a Ansible_Control_EC2

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

Ansible hace pues un proceso similar para conectarse a las máquinas.

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

## 2.4 Crear Inventario. (Y cosas relacionadas con el inventario).
Recordemos, que el inventario, no es más que un fichero en el que se almacenan los Hosts a los que se les va a aplicar

Entonces, ahora voy a crear un directorio en el que voy a poner el inventario.
`mkdir prueba_ansible`.

https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html

### 2.4.2 Teoría sobre los inventarios (cortito).

1. **¿En qué ruta debe de estar el archivo de ruta?**
   ![image](https://github.com/user-attachments/assets/5fbb0644-33da-4f7b-a7c7-f1c1d68f6c96)

   Si usamos el `ansible --version` podemos ver en qué ruta se ha instalado... `/etc/ansible` entonces. La ruta del inventario tiene que estar allí, **según la documentación**.

   ![image](https://github.com/user-attachments/assets/94cbc56e-c39d-40a2-8159-1599e483ce95)

   Entonces, el directorio existe y el archivo también:

   ![image](https://github.com/user-attachments/assets/3553c7ab-28b0-48a9-9b4f-9880486c98be)


2. **Entonces, como he creado una carpeta para que contenga el inventario, pues voy a poner allí el inventario (Lo puedo llamar como yo quiera, no se tiene que llamar Hosts)
   ¿Cómo hago para usar ese inventario en la ruta alternativa?**
   
   Como por defecto Ansible mira el /etc/ansible/hosts. En el comando de ejecución, vamos a poner `-i` quedaría algo así:

   ```
   ansible-playbook example.yml -i /ruta_alternativa/inventory
   ```

4. Teoría: ¿Qué tipo de archivo puede ser el archivo inventario?
   - Inventario Dinámico.
   - YAML o YML.
   - Texto plano, .ini.

   Inventario Dinámico:
   ```
   ansible-inventory -i my_dinamic_inventory.py --list
   ```

   Formato YAML o YML:
   ```
   all:
      hosts:
         web01:
            ansible_host: 172.31.31.178
            ansible_user: ec2-user
            ansible_ssh_private_key_file:
         web02:
   
   ```

   Inventario texto plano .ini (Es la que está por defecto el el `/etc/ansible/hosts`):
   ```
   #comentario
   #los espacios en blanco los ignoran
   host1 ansible_host=192.168.1.11
   ansible_user=usuario1

   host2 ansible_host=192.168.1.12
   ansible_user=usuario1
    
   ansible_port=2222
   ansible_user=usuario2
   
   [webservers]
   host1
   host2

   [databases]
   db1 ansible_host=192.168.1.20
   ansible_user=dbadmin

   [databases:vars]
   ansible_python_interpreter=/usr/bin/python3
   env=production
   ```

