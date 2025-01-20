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
