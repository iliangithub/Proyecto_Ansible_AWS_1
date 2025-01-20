# Introducción Ansible.

Ansible, no es un servicio, usa los módulos de Python, los scripts de Python para conectarse a la máquina y verter el código, y devolver un output.

Es necesario que la otra máquina tenga Python?

La Arquitectura de Ansible es simple. En la máquina que esté instalado Ansible, tenemos Playbooks, que es un archivo con las instrucciones y tareas que tiene que realizar.

Ese archivo, hará referencia a otro componente de la arquitectura Ansible, que es el Inventory, que es otro archivo pero en él aparecen los Hosts a los que se les va a ejecutar esos comandos, los Hosts, o el grupo de Hosts.

Y también en el Playbook hay otro componente de la arquitectura Ansible, que son los módulos.

Luego, Ansible utilizará la "Ansible config" y crear el script de Python, y luego pues se ejecutará.

# El proyecto.

Voy a desplegar una máquina con Ansible en AWS, que me permita controlar de forma remota
