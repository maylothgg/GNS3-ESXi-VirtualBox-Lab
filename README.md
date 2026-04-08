# GNS3-ESXi-VirtualBox-Lab
# Laboratorio de Redes con GNS3 y Virtualización en Windows 11

![Estado](https://img.shields.io/badge/Estado-Investigación_en_Curso-yellow) ![Plataforma](https://img.shields.io/badge/Plataforma-GNS3%20%2B%20Windows11-blue)

---

## 1. Arquitectura de Virtualización en Windows 11

### Aislamiento de Núcleo y VBS
Windows 11 utiliza **Aislamiento de Núcleo** y **Seguridad Basada en Virtualización (VBS)** para proteger el sistema operativo. Estas funciones crean entornos aislados en memoria para procesos críticos, previniendo que malware o vulnerabilidades afecten directamente al núcleo del sistema.  

**Impacto en la virtualización:**

- Limita el acceso directo a extensiones de virtualización por hardware (**VT-x/AMD-V**).  
- Puede provocar que **GNS3 VM** o **VirtualBox** no reconozcan soporte de virtualización anidada.  
- Reduce el rendimiento de laboratorios avanzados de red, especialmente al usar múltiples VMs.


**Recomendaciones:**

1. Desactivar la **Integridad de Memoria**:  
   `Configuración > Seguridad de Windows > Seguridad del dispositivo > Aislamiento de núcleo`  
2. Reiniciar el equipo.  
3. Verificar virtualización habilitada:

----
## 2. GNS3 VM: El Motor de Simulación

### KVM (Kernel-based Virtual Machine)

**¿Qué es KVM?**  
KVM es una tecnología de virtualización integrada en el kernel de Linux que permite convertir el sistema operativo en un hipervisor. Utiliza las extensiones de virtualización por hardware del procesador (VT-x de Intel o AMD-V de AMD) para ejecutar máquinas virtuales de manera eficiente y con un rendimiento casi nativo.

**Importancia en GNS3:**  
GNS3 utiliza una máquina virtual (GNS3 VM) que corre sobre un hipervisor (como VirtualBox o VMware). Cuando KVM está disponible y activo dentro de esa VM, GNS3 puede ejecutar routers y switches virtuales con aceleración por hardware, lo que significa:

- Menor consumo de CPU del host.  
- Mejor rendimiento y velocidad de simulación.  
- Mayor estabilidad en topologías grandes y complejas.

**Obligatoriedad de que aparezca "True":**  
Al iniciar GNS3, el servidor verifica si KVM está habilitado. Para un laboratorio profesional, **debe aparecer:**

Si aparece `False`, indica que no se está usando aceleración por hardware, lo que implica:

- Simulación lenta y con alta latencia.  
- Mayor uso de recursos del host.  
- Posibles errores en la conexión entre dispositivos.

**Causas comunes de que KVM aparezca False:**

- La virtualización por hardware (VT-x / AMD-V) está deshabilitada en la BIOS/UEFI.  
- Hyper-V o algún otro hipervisor está bloqueando el acceso a VT-x/AMD-V (como el Aislamiento de Núcleo en Windows 11).  
- La VM no está configurada para permitir virtualización anidada.

### Código para verificar KVM en GNS3 VM (Linux)

Conectado a la consola de la GNS3 VM (usualmente basada en Linux), ejecuta:

----

## 3. Integración con VirtualBox (Local)

### Configuración de Red (Host-Only)

El adaptador **Host-Only** en VirtualBox crea una red privada exclusiva entre el sistema anfitrión (host) y las máquinas virtuales (VMs). Esta configuración es fundamental para laboratorios de red, ya que permite que GNS3, a través de su interfaz gráfica (GUI) en el host, se comunique directamente con la GNS3 VM que actúa como servidor de simulación, sin interferencias ni dependencias de la red externa o internet.

Esta red aislada garantiza que el tráfico de simulación se mantenga dentro de un entorno controlado, mejorando la seguridad y el rendimiento. Además, facilita la configuración de direcciones IP estáticas, lo cual es crucial para que la GUI de GNS3 pueda acceder consistentemente a los servicios y dispositivos virtualizados.

**Pasos para crear y configurar el adaptador Host-Only:**

1. En VirtualBox, dirígete a `Archivo > Administrador de red Host-Only`.
2. Si no existe un adaptador Host-Only, crea uno nuevo.  
3. Configura la dirección IP del adaptador, por ejemplo, `192.168.56.1` con máscara de subred `255.255.255.0`.
4. En la configuración de red de la VM de GNS3, asigna este adaptador Host-Only para que la VM tenga conectividad en esa red.
5. Finalmente, desde el host, verifica la comunicación con la VM usando un ping a la IP configurada en la VM dentro de esa red.

-------

4. INTEGRACIÓN CON VMWARE ESXi (REMOTO)

• Arquitectura Cliente-Servidor:
  La integración de GNS3 con VMware ESXi se basa en un modelo cliente-servidor:
  
  - Cliente:
    Es la interfaz gráfica (GUI) de GNS3 instalada en la laptop del usuario.
    Desde aquí se diseñan las topologías y se gestionan los dispositivos.

  - Servidor:
    Es la GNS3 VM desplegada dentro de VMware ESXi.
    Esta VM se encarga de ejecutar los dispositivos (routers, switches, etc.)
    y procesar toda la simulación.

  - Comunicación:
    La GUI se conecta remotamente a la GNS3 VM mediante IP y puerto (por defecto 3080).
    Esto permite aprovechar los recursos del servidor ESXi (CPU/RAM) en lugar del equipo local.

------------------------------------------------------------

• Seguridad en vSwitch:
  Para que GNS3 funcione correctamente en ESXi, es necesario ajustar las políticas
  de seguridad del Port Group donde está conectada la GNS3 VM.

  Estas configuraciones son necesarias porque GNS3 genera tráfico con múltiples
  direcciones MAC y comportamientos de red avanzados.

  Configuraciones requeridas:

  - Promiscuous Mode: ACCEPT
    Permite que la interfaz reciba todo el tráfico de la red, no solo el dirigido a su MAC.
    Es necesario para capturar y reenviar paquetes en simulaciones.

  - MAC Address Changes: ACCEPT
    Permite que una máquina virtual cambie su dirección MAC sin ser bloqueada.
    GNS3 lo requiere para emular múltiples dispositivos.

  - Forged Transmits: ACCEPT
    Permite enviar paquetes con direcciones MAC distintas a la asignada.
    Es esencial para el correcto funcionamiento de routers y switches virtuales.

------------------------------------------------------------

• Configuración en ESXi (CLI opcional):

  esxcli network vswitch standard portgroup policy security set \
  -p "VM Network" \
  --allow-promiscuous=true \
  --allow-mac-change=true \
  --allow-forged-transmits=true

------------------------------------------------------------

• Configuración en GNS3 (Cliente):

  1. Ir a: Edit → Preferences

  2. En "Server":
     - Activar: Enable remote server
     - Host: IP de la GNS3 VM
     - Port: 3080

  3. En "GNS3 VM":
     - Activar: Enable the GNS3 VM
     - Provider: VMware ESXi

  4. Configurar conexión:
     - ESXi host: IP del servidor ESXi
     - Username: root
     - Password: contraseña
     - VM name: GNS3 VM

------------------------------------------------------------

• Resultado esperado:
  La GUI de GNS3 en la laptop controlará la GNS3 VM en ESXi,
  permitiendo ejecutar laboratorios de red de forma remota
  con mejor rendimiento y escalabilidad.
  ----

  5. MATRIZ DE SOLUCIÓN DE ERRORES (TROUBLESHOOTING)

• Objetivo:
  Identificar errores comunes al integrar GNS3 con VMware ESXi (remoto)
  y proporcionar soluciones técnicas claras.

------------------------------------------------------------

• Matriz de Errores:

  ERROR 1: KVM not available

  - Descripción:
    La GNS3 VM no puede usar virtualización por hardware.

  - Causas:
    * Virtualización (VT-x / AMD-V) deshabilitada en BIOS
    * Nested virtualization no habilitada en ESXi

  - Solución:
    1. Activar VT-x/AMD-V en BIOS del servidor físico
    2. En ESXi:
       VM → Edit Settings → CPU
       → Activar: "Expose hardware-assisted virtualization to the guest OS"

------------------------------------------------------------

  ERROR 2: uBridge permissions error

  - Descripción:
    Error al iniciar nodos indicando problemas con permisos de uBridge.

  - Causas:
    * Permisos incorrectos en la GNS3 VM
    * uBridge no ejecutándose correctamente

  - Solución:
    Ejecutar en la GNS3 VM:

    sudo chmod +x /usr/bin/ubridge
    sudo chown root:root /usr/bin/ubridge

    Reiniciar servicios:

    sudo systemctl restart gns3

------------------------------------------------------------

  ERROR 3: Firewall blocking port 3080

  - Descripción:
    La GUI de GNS3 no puede conectarse al servidor remoto.

  - Causas:
    * Firewall bloqueando el puerto 3080
    * Puerto incorrecto o IP incorrecta

  - Solución:

    En Linux (GNS3 VM):
    sudo ufw allow 3080/tcp

    O con iptables:
    sudo iptables -A INPUT -p tcp --dport 3080 -j ACCEPT

    Verificar conectividad:
    ping IP_GNS3_VM

------------------------------------------------------------

  ERROR 4: No hay conectividad entre nodos

  - Descripción:
    Los dispositivos en GNS3 no pueden comunicarse.

  - Causas:
    * Políticas de seguridad en ESXi mal configuradas

  - Solución:
    Activar en el Port Group:
    - Promiscuous mode: ACCEPT
    - MAC address changes: ACCEPT
    - Forged transmits: ACCEPT

------------------------------------------------------------

  ERROR 5: GNS3 VM no conecta con ESXi

  - Descripción:
    Error al enlazar la GNS3 VM desde la GUI.

  - Causas:
    * Credenciales incorrectas
    * Nombre de VM incorrecto
    * Problemas de red

  - Solución:
    Verificar:
    - IP del ESXi
    - Usuario: root
    - Nombre exacto de la VM
    - Conectividad (ping)

------------------------------------------------------------

• Resultado esperado:
  Identificar rápidamente problemas comunes y aplicar soluciones
  para garantizar el correcto funcionamiento de GNS3 con ESXi.
