# --Laboratorios-del-modulo-V
Practica 3: Cluster de Alta Disponibilidad HTTP 
==============================================
 Práctica de Alta Disponibilidad con Apache y Keepalived
==============================================

🖥️ APACHE HTTPD
----------------
sudo systemctl start httpd
    → Inicia el servicio de Apache

sudo systemctl status httpd
    → Muestra si Apache está activo, detenido o con errores

sudo nano /etc/httpd/conf/httpd.conf
    → Edita el archivo principal de configuración de Apache

grep ^Listen /etc/httpd/conf/httpd.conf
    → Muestra el puerto que Apache está escuchando

curl http://localhost:8080
    → Prueba localmente si Apache responde correctamente

sudo firewall-cmd --add-port=8080/tcp --permanent
    → Añade el puerto 8080 al firewall para acceso externo

sudo firewall-cmd --reload
    → Recarga la configuración del firewall para aplicar cambios

------------------------------------------------------------

🔧 DIAGNÓSTICO DE RED Y PUERTOS
-------------------------------
sudo netstat -tulpn | grep :8080
    → Identifica qué servicio está usando el puerto 8080

sudo ss -tulpn | grep :8080
    → Alternativa moderna a netstat para listar servicios por puerto

sudo kill -9 <PID>
    → Mata un proceso específico (reemplaza <PID> por el número real)

sudo rm -f /var/run/httpd/httpd.pid
    → Elimina el archivo PID residual que impide reiniciar Apache

------------------------------------------------------------

📦 INSTALACIÓN Y CONFIGURACIÓN DE KEEPALIVED
--------------------------------------------
sudo yum install keepalived -y
    → Instala Keepalived en el sistema

sudo mkdir -p /etc/keepalived
    → Crea el directorio de configuración si no existe

sudo nano /etc/keepalived/keepalived.conf
    → Abre o edita el archivo de configuración principal

sudo systemctl start keepalived
    → Inicia el servicio Keepalived

sudo systemctl restart keepalived
    → Reinicia Keepalived después de cambios

sudo systemctl status keepalived
    → Verifica si el servicio está activo y sin errores

sudo journalctl -xeu keepalived.service
    → Muestra el registro extendido del servicio para ver errores o problemas

------------------------------------------------------------

🌐 VALIDACIÓN DE IP FLOTANTE Y FAILOVER
---------------------------------------
ip a
    → Muestra todas las interfaces de red y sus direcciones IP

ip a | grep 192.168.0.250
    → Verifica si la IP flotante está activa en el servidor

ping 192.168.0.250
    → Prueba conectividad con la IP virtual

sudo poweroff
    → Apaga el servidor para probar el failover automático

------------------------------------------------------------

🧾 CONTENIDO DE KEEPALIVED.CONF

[SERVER1 - MASTER]
------------------
vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234
    }
    virtual_ipaddress {
        192.168.0.250
    }
}

[SERVER2 - BACKUP]
------------------
vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234
    }
    virtual_ipaddress {
        192.168.0.250
    }
}
