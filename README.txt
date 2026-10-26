================================================================================
PROYECTO: Tarea 3 y 4  Automatizado con Ansible + GitHub (LAMP, Samba, FTP)
SISTEMA OPERATIVO OBJETIVO: openSUSE 16.0
================================================================================

--------------------------------------------------------------------------------
a) ENLACE AL REPOSITORIO DE GITHUB
--------------------------------------------------------------------------------
URL del repositorio: https://github.com/LuzRodriguez2004/Tarea-Ansible

--------------------------------------------------------------------------------
b) PASOS PARA EJECUTAR EL PROYECTO DE ANSIBLE
--------------------------------------------------------------------------------

1. CLONAR EL REPOSITORIO
   Descargue el proyecto y acceda al directorio de trabajo:

   git clone https://github.com/LuzRodriguez2004/Tarea-Ansible.git
   cd Tarea-Ansible

2. INSTALAR COLECCIONES REQUERIDAS EN EL NODO CENTRAL
   Ejecute el siguiente comando para asegurar que los módulos de zypper y
   MariaDB estén disponibles en la máquina controladora:

   ansible-galaxy collection install community.general community.mysql ansible.posix

3. CONFIGURAR ACCESO SSH SIN CONTRASEÑA HACIA EL SERVIDOR DESTINO
   Genere la llave SSH dedicada para Ansible y cópiela al servidor openSUSE 16.0
   (reemplace 'usuario' y '192.168.1.x' por los datos del servidor destino):

   ssh-keygen -t ed25519 -f ~/.ssh/id_ansible -C "ansible-control" -N ""
   ssh-copy-id -i ~/.ssh/id_ansible.pub usuario@192.168.1.x

4. CONFIGURAR EL ARCHIVO DE INVENTARIO (inventory.ini)
   Edite el archivo 'inventory.ini' y coloque la dirección IP real del servidor
   destino openSUSE 16.0:

   [opensuse_servers]
   servidor_opensuse ansible_host=192.168.1.x

   * Nota de seguridad: Por buenas prácticas, 'inventory.ini' no contiene nombres
     de usuario ni contraseñas en texto plano.

5. CREDENCIALES
   Las credenciales de conexión SSH/sudo y las contraseñas de MariaDB y Samba se encuentran
en 'group_vars/all/credenciales.yml'.

6. VERIFICAR CONECTIVIDAD Y ESCALACIÓN DE PRIVILEGIOS
   Compruebe que el nodo de control se comunica con el servidor openSUSE 16.0:

   ansible opensuse_servers -m ping --ask-vault-pass
   ansible opensuse_servers -m command -a "whoami" --become --ask-vault-pass

7. EJECUTAR EL PLAYBOOK PRINCIPAL (DESPLIEGUE)
   Valide la sintaxis y ejecute el aprovisionamiento completo:

   ansible-playbook site.yml --syntax-check
   ansible-playbook site.yml --ask-vault-pass

8. COMPROBAR Y VALIDAR LOS SERVICIOS DESPLEGADOS

   Una vez finalizada la ejecución del playbook sin errores, ejecute las

   siguientes pruebas desde el nodo central:

   # 1. Confirmar que todos los demonios están activos (active/running)

   ansible opensuse_servers -m shell -a "systemctl is-active apache2 mariadb smb nmb vsftpd" --become --ask-vault-pass

   # 2. Verificar que Apache y PHP responden por HTTP (cambia por tu IP destino)

   curl -I http://192.168.1.x/info.php

   curl -I http://192.168.1.x/phpMyAdmin/

   # 3. Verificar que Samba lista el recurso compartido con el usuario creado

   smbclient -L //192.168.1.50 -U colab_samba

--------------------------------------------------------------------------------
c) ESTRUCTURA DE ROLES IMPLEMENTADOS Y OPTIMIZACIÓN
--------------------------------------------------------------------------------
- ansible.cfg : Optimizado con Pipelining SSH, ControlPersist (multiplexación)
                y caché local de facts en formato JSON para reducir tiempos de
                respuesta.
- roles/lamp  : Instala Apache2, PHP8, MariaDB y phpMyAdmin; asegura el motor
                de base de datos y despliega la plantilla phpmyadmin.conf.j2.
- roles/samba : Instala Samba, crea el usuario del sistema (nologin), registra
                credenciales en smbpasswd, crea el directorio compartido y
                aplica máscara de permisos 0770 mediante smb.conf.j2.
- roles/ftp   : Instala vsftpd y despliega la plantilla vsftpd.conf.j2 con
                jaula chroot local y puertos pasivos acotados.
================================================================================
