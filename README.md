# Laboratorio de Hacking Etico con Inteligencia Artificial



## Episodio 3 - Configuración de Wazuh y Nessus

Configuración de Wazuh y Nessus

En este episodio configuramos nuestra herramienta SIEM y XRD, Wazuh. Luego instalamos el agente de Wazuh en nuestra máquina virtual Kali, nuestro servidor Docker y nuestro firewall pfSense. Posteriormente, pasamos a nuestro escáner de vulnerabilidades, Nessus.

Por favor, sigue las instrucciones a continuación para instalar el agente de Wazuh en el firewall pfSense.


*******


### Resumen de la Instalación del Agente Wazuh en pfSense

1. Habilita SSH antes de conectarte al firewall.

2. Habilita FreeBSD para poder descargar el agente de Wazuh.

3. Habilita los registros del firewall (syslog ya está configurado).

4. En Wazuh: Crea un grupo y habilita una regla.



*******


### Habilitar FreeBSD para Descargar el Agente de Wazuh

Por defecto, los repositorios de paquetes de FreeBSD están deshabilitados en los firewalls pfSense. Sigue los pasos a continuación para habilitarlos en nuestro laboratorio.

Conéctate al firewall mediante SSH y navega al siguiente directorio:
```
cd /usr/local/etc/pkg/repos/
```

Edita el archivo pfsense.conf:
```
vi pfSense.conf
```

Configura lo siguiente:
```
FreeBSD: { enabled: yes }
```

Edita el archivo freebsd.conf:
```
vi FreeBSD.conf
```

Configura lo siguiente:
```
FreeBSD: { enabled: yes }
```


*******


### Instalar el Agente de Wazuh

Actualiza la caché de paquetes
```
pkg update
```

Busca el agente de Wazuh
```
pkg search wazuh-agent
```

Instala el agente de Wazuh para el firewall
```
pkg install wazuh-agent-4.12.0
```

*******


### Iniciar el Agente de Wazuh

Ingresa:
```
cd /var/ossec/etc
```


Luego edita el archivo ossec.conf
```
vi ossec.conf
```

Añade lo siguiente al archivo, esta es la dirección de tu servidor Wazuh:
```
<server>
  <address>10.10.1.51</address>
</server>
```

 
Habilita el agente ingresando lo siguiente:
```
sysrc wazuh_agent_enable="YES"
```

Crea un enlace simbólico
```
ln -s /usr/local/etc/rc.d/wazuh-agent /usr/local/etc/rc.d/wazuh-agent.sh
```

Inicia el agente
```
service wazuh-agent start
```




*******
### Habilitar los Registros del Firewall


¿Quieres monitorear los registros de tu firewall? Necesitamos crear un grupo pfSense y añadir lo siguiente.
En Wazuh, navega a Management > New Group (Gestión > Nuevo Grupo).

Edita el grupo y añade lo siguiente::
```
<localfile>
	<log_format>syslog</log_format>
	<location>/var/log/filter.log</location>
</localfile>
```

*******


### Crear una Regla Personalizada
Monitorea la salida en el archivo filter.log.
Navega a Management > Rules > Create a new rule (Gestión > Reglas > Crear una nueva regla) y añade lo siguiente:

```
<group name="pfsense,">
  <rule id="87701" level="5" overwrite="yes">
    <if_sid>87700</if_sid>
    <action>block</action>
    <description>pfSense firewall drop event.</description>
    <group>firewall_block,pci_dss_1.4,gpg13_4.12,hipaa_164.312.a.1,nist_800_53_SC.7,tsc_CC6.7,tsc_CC6.8,</group>
  </rule>
</group>
```
Añade la regla al grupo pfSense que creaste anteriormente.
