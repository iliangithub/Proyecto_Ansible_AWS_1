# Introducción Ansible.

Ansible, no es un servicio, usa los módulos de Python, los scripts de Python para conectarse a la máquina y verter el código, y devolver un output.

Es necesario que la otra máquina tenga Python?

La Arquitectura de Ansible es simple. En la máquina que esté instalado Ansible, tenemos Playbooks, que es un archivo con las instrucciones y tareas que tiene que realizar.

Ese archivo, hará referencia a otro componente de la arquitectura Ansible, que es el Inventory, que es otro archivo pero en él aparecen los Hosts a los que se les va a ejecutar esos comandos, los Hosts, o el grupo de Hosts.

Y también en el Playbook hay otro componente de la arquitectura Ansible, que son los módulos.

Luego, Ansible utilizará la "Ansible config" y crear el script de Python, y luego pues se ejecutará.

# El proyecto.

Voy a desplegar una máquina con Ansible en AWS, que me permita controlar de forma remota.

# Instancia EC2 de Ansible.

- Nombre: `Ansible_control_EC2`
- AMI: `Ubuntu Server 22.04 LTS (HVM), SSD Volume Type`
- Tipo de instancia: `t2.micro`
- Crear un nuevo par de claves (RSA, .pem)

**Editamos la configuración de red:**
- ![image](https://github.com/user-attachments/assets/8884a75c-1307-4378-9c9c-802c6c5e192b)

y ya. La creamos.

# Instancias (3) EC2 de CentOs.
- AMI: `CentOS Stream 9 (x86_64)`
- Tipo de instancia: `t2.micro`
