Descripción del proyecto

Este laboratorio simula una red híbrida que combina routing estático y routing dinámico (OSPF), incluyendo un router frontera y redistribución de rutas.

El objetivo es demostrar:

Conectividad entre redes con diferentes métodos de enrutamiento.

Redistribución de rutas estáticas hacia OSPF para integrar redes legadas y dinámicas.

Diseño modular y escalable de red que refleja escenarios reales de ingeniería de redes.

Topología de la red
PC0,PC1,PC2      PC3,PC4,PC5       PC6,PC7,PC8
    |                |                 |
   SW0              SW1               SW2
    |                |                 |
   R0 -------------- R1 -------------- R2
   (Estático)       (OSPF)           (OSPF)


R0: Router con rutas estáticas, conectividad hacia LANs 192.168.10.0/24 y 192.168.20.0/24.

R1 y R2: Routers OSPF, formando un dominio dinámico que conecta LAN 192.168.30.0/24 y el enlace al router R0.

SW0, SW1, SW2: Switches de acceso conectando PCs a los routers correspondientes.

PCs: Hosts finales de cada LAN, configurados con IP, máscara y gateway correspondiente.

Direccionamiento IP
Dispositivo	Interfaz	IP	            Máscara	        Gateway
PC0	        Fa0/1	    192.168.10.2	255.255.255.0	192.168.10.1
PC1	        Fa0/2	    192.168.10.3	255.255.255.0	192.168.10.1
PC2	        Fa0/3	    192.168.10.4	255.255.255.0	192.168.10.1
R0	        G0/0/1	    192.168.10.1	255.255.255.0	N/A
R0	        G0/0/0	    10.0.0.1	    255.255.255.252	N/A
R1	        G0/0	    10.0.0.2	    255.255.255.252	N/A
PC3	        Fa0/1	    192.168.20.2	255.255.255.0	192.168.20.1
PC4	        Fa0/2	    192.168.20.3	255.255.255.0	192.168.20.1
PC5	        Fa0/3	    192.168.20.4	255.255.255.0	192.168.20.1
R1	        G0/1	    192.168.20.1	255.255.255.0	N/A
R1	        S0/2/0	    10.0.0.5	    255.255.255.252	N/A
R2	        S0/1/0	    10.0.0.6	    255.255.255.252	N/A
PC6	        Fa0/1	    192.168.30.2	255.255.255.0	192.168.30.1
PC7	        Fa0/2	    192.168.30.3	255.255.255.0	192.168.30.1
PC8	        Fa0/3	    192.168.30.4	255.255.255.0	192.168.30.1
R2	        G0/1	    192.168.30.1	255.255.255.0	N/A
Configuración destacada
Router R0 (Estático)
enable
conf t
hostname R0

interface g0/0/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown

interface g0/0/1
 ip address 192.168.10.1 255.255.255.0
 no shutdown

ip route 192.168.20.0 255.255.255.0 10.0.0.2
ip route 192.168.30.0 255.255.255.0 10.0.0.2

Router R1 (OSPF + frontera con R0)
enable
conf t
hostname R1

interface g0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown

interface g0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown

interface s0/2/0
 ip address 10.0.0.5 255.255.255.252
 no shutdown

router ospf 1
 network 10.0.0.4 0.0.0.3 area 0
 network 192.168.20.0 0.0.0.255 area 0
 redistribute static subnets
 redistribute connected subnets

Router R2 (OSPF)
enable
conf t
hostname R2

interface s0/1/0
 ip address 10.0.0.6 255.255.255.252
 no shutdown

interface g0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown

router ospf 1
 network 10.0.0.4 0.0.0.3 area 0
 network 192.168.30.0 0.0.0.255 area 0

Conceptos implementados

Red híbrida: mezcla de rutas estáticas y dinámicas.

Router frontera (R1): inyecta rutas estáticas hacia OSPF mediante redistribute static subnets.

OSPF: permite que R2 y R3 aprendan automáticamente todas las rutas redistribuidas.

Conectividad final: todas las PCs pueden hacer ping entre sí, aunque algunas redes sean estáticas y otras dinámicas.

Escalabilidad: permite agregar más routers OSPF sin tocar la configuración estática de R0.

Pruebas de conectividad realizadas
# Desde PC0
ping 192.168.20.2
ping 192.168.30.2

# Desde PC6
ping 192.168.10.2
ping 192.168.20.3

# Desde R3
show ip route
# Debe mostrar rutas O E2 hacia la red 192.168.10.0/24

Observaciones

Las PCs dependen de su gateway por defecto para alcanzar redes remotas.

La redistribución de rutas es clave para que el dominio OSPF conozca redes externas al mismo.

Esta topología refleja un escenario real donde una empresa combina redes legadas estáticas con un backbone dinámico OSPF.