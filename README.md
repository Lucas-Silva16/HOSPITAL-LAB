![Cisco Packet Tracer](https://img.shields.io/badge/Cisco_Packet_Tracer-Network_Simulation-1BA0D7?style=for-the-badge&logo=cisco)

<img width="1980" height="1080" alt="hospital-network-infrastructure" src="https://github.com/user-attachments/assets/11c74294-902c-4f14-8041-b5afa0ee91a6" />


# HOSPITAL-LAB 
### Tabela de VLANs

| VLAN | NOME | HOST NECESSÁRIOS | HOST ÚTEIS | PREFIXO | REDE | MÁSCARA | GATEWAY | BROADCAST |
|------|------|-----------------|------------|---------|------|---------|---------|-----------|
| 10 | Dados | 60 | 62 | /26 | 192.168.10.0 | 255.255.255.192 | 192.168.10.1 | 192.168.10.63 |
| 50 | WiFi Doentes | 50 | 62 | /26 | 192.168.50.0 | 255.255.255.192 | 192.168.50.1 | 192.168.50.63 |
| 20 | VoIP | 40 | 62 | /26 | 192.168.20.0 | 255.255.255.192 | 192.168.20.1 | 192.168.20.63 |
| 30 | Clínica Crítica | 20 | 30 | /27 | 192.168.30.0 | 255.255.255.224 | 192.168.30.1 | 192.168.30.31 |
| 60 | Gestão | 15 | 30 | /27 | 192.168.60.0 | 255.255.255.224 | 192.168.60.1 | 192.168.60.31 |
| 40 | Servidores | 10 | 14 | /28 | 192.168.40.0 | 255.255.255.240 | 192.168.40.1 | 192.168.40.15 |
| 100 | DMZ | 5 | 6 | /29 | 192.168.100.0 | 255.255.255.248 | 192.168.100.1 | 192.168.100.7 |
| 99 | Nativa | 2 | 2 | /30 | 192.168.99.0 | 255.255.255.252 | — | 192.168.99.3 |

---

### Links L3 e Conexões 
| Link / Conexão | Dispositivo Local | Interface Local | Endereço IP | Máscara de Rede | Prefixo | Dispositivo Remoto | Interface Remota |
| :--- | :--- | :--- | :--- | :--- | :---: | :--- | :--- |
| **Link WAN (Sede-Filial)** | R0-SEDE | Se0/2/0 | 10.0.0.1 | 255.255.255.252 | /30 | R1-FILIAL | Se0/1/0 |
| **Link WAN (Sede-Filial)** | R1-FILIAL | Se0/1/0 | 10.0.0.2 | 255.255.255.252 | /30 | R0-SEDE | Se0/2/0 |
| **Ligação L3 Core A** | CORE-SW-A | Gi1/0/5 | 192.168.2.1 | 255.255.255.252 | /30 | R0-SEDE | - |
| **Ligação L3 Core B** | CORE-SW-B | Gi1/0/5 | 192.168.2.5 | 255.255.255.252 | /30 | R0-SEDE | - |
| **Rede Local Filial** | R1-FILIAL | Gi0/0/0 | 192.168.200.1 | 255.255.255.192 | /26 | SW-FILIAL | - |


## Dia 1


### CORE SWITCHES e DISTRIBUTION SWITCHES
> Aplicado nos CORE-SW-A, CORE-SW-B, DISTR-SW-1, DISTR-SW-2, DISTR-SW-3

```
enable
configure terminal
hostname X-SW-X
no ip domain-lookup
banner motd #Acesso Restrito a Staff do Hospital Sao Lucas#
enable secret SigmaAura16
username admin privilege 15 secret SigmaAura16
service password-encryption
login block-for 60 attempts 3 within 30
line console 0
 password SigmaAura16
 login local
 logging synchronous
 exec-timeout 5 0
exit
ip domain-name hospital.local
crypto key generate rsa modulus 2048
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
exit
interface range gi1/0/X - X
 shutdown
 description PORTA_INATIVA
exit
interface range gi1/1/X - X
 shutdown
 description PORTA_INATIVA
exit
end
write memory
```
---

### ACCESS SWITCHES e SWITCH FILIAL
> Aplicado nos AC-SW-URG, AC-SW-INT, AC-SW-ADM, AC-SW-FAR, SW-SERVERS, SW-FILIAL

```
enable
configure terminal
hostname X-SW-X
no ip domain-lookup
banner motd #Acesso Restrito a Staff do Hospital Sao Lucas#
enable secret SigmaAura16
username admin privilege 15 secret SigmaAura16
service password-encryption
login block-for 60 attempts 3 within 30
line console 0
 password SigmaAura16
 login local
 logging synchronous
 exec-timeout 5 0
exit
ip domain-name hospital.local
crypto key generate rsa modulus 2048
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
exit
interface range fa0/X - X
 shutdown
 description PORTA_INATIVA
exit
interface range gi0/X - X
 shutdown
 description PORTA_INATIVA
exit
end
write memory
```
---

### ROUTER SEDE (R0-SEDE)

```
enable
configure terminal
hostname R0-SEDE
no ip domain-lookup
banner motd #Acesso Restrito a Staff do Hospital Sao Lucas#
enable secret SigmaAura16
username admin privilege 15 secret SigmaAura16
service password-encryption
login block-for 60 attempts 3 within 30
line console 0
 password SigmaAura16
 login local
 logging synchronous
 exec-timeout 5 0
exit
ip domain-name hospital.local
crypto key generate rsa modulus 2048
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
exit
interface range gi0/1/0-3
 shutdown
 description PORTA_INATIVA
exit
interface Serial0/2/1
 shutdown
 description PORTA_INATIVA
exit
end
write memory
```

---

### ROUTER FILIAL (R1-FILIAL)

```
enable
configure terminal
hostname R1-FILIAL
no ip domain-lookup
banner motd #Acesso Restrito a Staff do Hospital Sao Lucas#
enable secret SigmaAura16
username admin privilege 15 secret SigmaAura16
service password-encryption
login block-for 60 attempts 3 within 30
line console 0
 password SigmaAura16
 login local
 logging synchronous
 exec-timeout 5 0
exit
ip domain-name hospital.local
crypto key generate rsa modulus 2048
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
line vty 0 4
 transport input ssh
 login local
 exec-timeout 5 0
exit
interface range gi0/0/1-2
 shutdown
 description PORTA_INATIVA
exit
interface Serial0/1/1
 shutdown
 description PORTA_INATIVA
exit
end
write memory
```

### Dia 2

### Configuracao das VLANS em todos os Switches

```
vlan 10
name Dados
exit
vlan 50
name WIFI Doentes
exit
vlan 20
name VOIP
exit
vlan 30
name Clinica
exit
vlan 60
name Gestao
exit
vlan 40 
name Servidores
exit
vlan 100
name DMZ
exit
vlan 99
name Nativa
exit
```

---

### Trunk (Todos os Switches)
```
interface (Porta)
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,50,60,99,100
exit
```
---

### L3 Conections

No CORE-SW-A – porta que liga ao Router0
```
interface GigabitEthernet1/0/5
 no switchport
 ip address 192.168.2.1 255.255.255.252
 no shutdown
exit
```
No CORE-SW-B – porta que liga ao Router0
```
interface GigabitEthernet1/0/5
 no switchport
 ip address 192.168.2.5 255.255.255.252
 no shutdown
exit

```
---

### Router 1
```
interface Serial0/1/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/0/0
 ip address 192.168.200.1 255.255.255.192
 no shutdown
exit
```
### Router 0
```
interface Serial0/2/0
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shutdown
exit
```

---

### Portas Access nos end devices de cada AC SW
Exemplo para o AC SW URGENCIAS

```
interface fa0/   
 switchport mode access
 switchport access vlan 10
exit

interface fa0/X   
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20
exit

(So o Urgencias e o Internamento têm oss AP's  os outros 2 AC SW nao tem estas linhas)
interface fa0/X   
 switchport mode access
 switchport access vlan 50
exit

```
---

### DIA 3

## EtherChannel

```
interface range GigabitEthernet1/0/1 - 2
 channel-group 1 mode active
 no shutdown
exit

interface Port-Channel1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,50,60,99,100
exit

end
write memory
```
---

### PVST EM AMBOS OS CORE SW

```
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,50,60,99,100 root primary
```

### PVST nos restantes SW

```
spanning-tree mode rapid-pvst
```

---

## DIA 4



### SVIS CORE-SW-A // CORE-SW-B
#### Para o CORE-SW-B o ip address muda de .2 para .3, segue a mesma configuracao
```
configure terminal

ip routing

interface vlan 10
 ip address 192.168.10.2 255.255.255.192
 no shutdown
exit

interface vlan 20
 ip address 192.168.20.2 255.255.255.192
 no shutdown
exit

interface vlan 30
 ip address 192.168.30.2 255.255.255.224
 no shutdown
exit

interface vlan 40
 ip address 192.168.40.2 255.255.255.240
 no shutdown
exit

interface vlan 50
 ip address 192.168.50.2 255.255.255.192
 no shutdown
exit

interface vlan 60
 ip address 192.168.60.2 255.255.255.224
 no shutdown
exit

```
 ### APENAS PARA O CORE-SW-A!
```
interface vlan 99                               
 ip address 192.168.99.2 255.255.255.252
 no shutdown
exit
```
```
interface vlan 100
 ip address 192.168.100.2 255.255.255.248
 no shutdown
exit

end
write memory
```
---

### CORE-SW-A (priority 110 — active):
```
configure terminal

ip routing

interface vlan 10
 ip address 192.168.10.2 255.255.255.192
 standby 1 ip 192.168.10.1
 standby 1 priority 110
 standby 1 preempt
 no shutdown
exit

interface vlan 20
 ip address 192.168.20.2 255.255.255.192
 standby 1 ip 192.168.20.1
 standby 1 priority 110
 standby 1 preempt
 no shutdown
exit

interface vlan 30
 ip address 192.168.30.2 255.255.255.224
 standby 1 ip 192.168.30.1
 standby 1 priority 110
 standby 1 preempt
 no shutdown
exit

interface vlan 40
 ip address 192.168.40.2 255.255.255.240
 standby 1 ip 192.168.40.1
 standby 1 priority 110
 standby 1 preempt
 no shutdown
exit

interface vlan 50
 ip address 192.168.50.2 255.255.255.192
 standby 1 ip 192.168.50.1
 standby 1 priority 110
 standby 1 preempt
 no shutdown
exit

interface vlan 60
 ip address 192.168.60.2 255.255.255.224
 standby 1 ip 192.168.60.1
 standby 1 priority 110
 standby 1 preempt
 no shutdown
exit

interface vlan 99
 ip address 192.168.99.2 255.255.255.252
 no shutdown
exit

interface vlan 100
 ip address 192.168.100.2 255.255.255.248
 standby 1 ip 192.168.100.1
 standby 1 priority 110
 standby 1 preempt
 no shutdown
exit

end
write memory
```

### CORE-SW-B (priority 90 — standby):
```
configure terminal

ip routing

interface vlan 10
 ip address 192.168.10.3 255.255.255.192
 standby 1 ip 192.168.10.1
 standby 1 priority 90
 standby 1 preempt
 no shutdown
exit

interface vlan 20
 ip address 192.168.20.3 255.255.255.192
 standby 1 ip 192.168.20.1
 standby 1 priority 90
 standby 1 preempt
 no shutdown
exit

interface vlan 30
 ip address 192.168.30.3 255.255.255.224
 standby 1 ip 192.168.30.1
 standby 1 priority 90
 standby 1 preempt
 no shutdown
exit

interface vlan 40
 ip address 192.168.40.3 255.255.255.240
 standby 1 ip 192.168.40.1
 standby 1 priority 90
 standby 1 preempt
 no shutdown
exit

interface vlan 50
 ip address 192.168.50.3 255.255.255.192
 standby 1 ip 192.168.50.1
 standby 1 priority 90
 standby 1 preempt
 no shutdown
exit

interface vlan 60
 ip address 192.168.60.3 255.255.255.224
 standby 1 ip 192.168.60.1
 standby 1 priority 90
 standby 1 preempt
 no shutdown
exit

interface vlan 100
 ip address 192.168.100.3 255.255.255.248
 standby 1 ip 192.168.100.1
 standby 1 priority 90
 standby 1 preempt
 no shutdown
exit

end
write memory
``` 

---

### OSPF 

## CORE-SW-A:
```
configure terminal

ip routing

router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.63 area 0
 network 192.168.20.0 0.0.0.63 area 0
 network 192.168.30.0 0.0.0.31 area 0
 network 192.168.40.0 0.0.0.15 area 0
 network 192.168.50.0 0.0.0.63 area 0
 network 192.168.60.0 0.0.0.31 area 0
 network 192.168.100.0 0.0.0.7 area 0
 network 192.168.2.0 0.0.0.3 area 0
exit

end
write memory
```
## CORE-SW-B:
```
configure terminal

router ospf 1
 router-id 2.2.2.2
 network 192.168.10.0 0.0.0.63 area 0
 network 192.168.20.0 0.0.0.63 area 0
 network 192.168.30.0 0.0.0.31 area 0
 network 192.168.40.0 0.0.0.15 area 0
 network 192.168.50.0 0.0.0.63 area 0
 network 192.168.60.0 0.0.0.31 area 0
 network 192.168.100.0 0.0.0.7 area 0
 network 192.168.2.4 0.0.0.3 area 0
exit

end
write memory
```

### DISTR-SW-1, DISTR-SW-2, DISTR-SW-3:
```
configure terminal

ip routing

router ospf 1
 router-id 3.3.3.3 (PARA O 2 passa para 4.4.4.4 & Para o 3 passa para 5.5.5.5)
 passive-interface default
 no passive-interface GigabitEthernet1/0/X
exit

end
write memory
```
### R0-SEDE
```
configure terminal

router ospf 1
 router-id 6.6.6.6
 network 192.168.2.0 0.0.0.3 area 0
 network 192.168.2.4 0.0.0.3 area 0
 network 192.168.2.8 0.0.0.3 area 0
 network 10.0.0.0 0.0.0.3 area 0
exit

end
write memory
```
### R1-FILIAL
```
configure terminal

router ospf 1
 router-id 7.7.7.7
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.200.0 0.0.0.63 area 0
 network 192.168.10.0 0.0.0.63 area 0
 network 192.168.20.0 0.0.0.63 area 0
 network 192.168.60.0 0.0.0.31 area 0
exit

end
write memory
```
### DIA 5

###  ASA 5505
---
```
Interfaces:
configure terminal

interface Vlan1
 nameif inside
 security-level 100
 ip address 192.168.2.10 255.255.255.252
 no shutdown
exit

interface Vlan2
 nameif outside
 security-level 0
 ip address 203.0.113.1 255.255.255.252
 no shutdown
exit

interface Vlan3
 no forward interface Vlan1
 nameif dmz
 security-level 50
 ip address 192.168.100.1 255.255.255.248
 no shutdown
exit

interface Ethernet0/0
 switchport access vlan 2
 no shutdown
exit

interface Ethernet0/1
 switchport access vlan 1
 no shutdown
exit

interface Ethernet0/2
 switchport access vlan 3
 no shutdown
exit

```
### NAT 
```
object network INSIDE-NET
 subnet 192.168.0.0 255.255.0.0
 nat (inside,outside) dynamic interface
exit

object network SERVER-HTTP
 host 192.168.100.2
 nat (dmz,outside) static 203.0.113.2
exit

object network SERVER-EMAIL
 host 192.168.100.3
 nat (dmz,outside) static 203.0.113.3
exit

```
### ACL:
```
access-list OUTSIDE-TO-DMZ extended permit tcp any host 203.0.113.2 eq 80
access-list OUTSIDE-TO-DMZ extended permit tcp any host 203.0.113.2 eq 443
access-list OUTSIDE-TO-DMZ extended permit tcp any host 203.0.113.3 eq 25
access-list OUTSIDE-TO-DMZ extended permit tcp any host 203.0.113.3 eq 143
access-group OUTSIDE-TO-DMZ in interface outside
access-list OUTSIDE-TO-DMZ extended permit icmp any any


access-list DMZ-TO-INSIDE extended deny ip 192.168.100.0 255.255.255.248 192.168.0.0 255.255.0.0
access-list DMZ-TO-INSIDE extended permit ip any any
access-group DMZ-TO-INSIDE in interface dmz

```

### DIA 6

### SERVERS
#### Todos os servers a configuracao correu bem, menos no DHCP
``` 
A falha global de DHCP na infraestrutura dividia-se em dois problemas arquiteturais distintos:

Na Sede (Colisão L3): O Servidor DHCP físico estava configurado com o IP .40.3. Bateu em colisão porque esse IP pertencia à interface virtual do CORE-SW-B. Moveu-se o servidor para o IP .40.4, e posteriormente o resto dos servidores tambem subiram 1 e reconfigurou-se o vetor de reencaminhamento (DHCP Relay) nos dois Core Switches para apontar para a morada nova.

Na Filial (Quebra de Encapsulamento): O Switch da Filial enviava os pedidos do PC com a etiqueta da VLAN 10, mas a porta do Router da Filial era uma interface física plana que rejeitava tráfego etiquetado. Converteu-se o Router da Filial num modelo Router-on-a-Stick (criando a sub-interface virtual .10)
```

```
Passo 1: No Servidor DHCP Físico
No separador Desktop -> IP Configuration

IP Address: 192.168.40.4
Subnet Mask: 255.255.255.240
Default Gateway: 192.168.40.1 

Em todos os Pools criados (VLAN 10, 20, 30, etc.), altererei o campo DNS Server para 192.168.40.5. (Tinha posto os IPS dos serves em conflito com os IPs das interfaces virtuais (SVIs) e o HSRP dos Core Switches (CORE-SW-A e CORE-SW-B)

Criar o Pool da Filial: Pool Name POOL_FILIAL | Gateway 192.168.200.1 | DNS 192.168.40.5 | Start IP 192.168.200.10 | Subnet Mask 255.255.255.192 | Max Users 53. 
```
--- 

### Na CLI dos Core Switches
```
configure terminal

interface vlan 10
 no ip helper-address 192.168.40.3
 ip helper-address 192.168.40.4

interface vlan 20
 no ip helper-address 192.168.40.3
 ip helper-address 192.168.40.4

interface vlan 30
 no ip helper-address 192.168.40.3
 ip helper-address 192.168.40.4

interface vlan 50
 no ip helper-address 192.168.40.3
 ip helper-address 192.168.40.4

interface vlan 60
 no ip helper-address 192.168.40.3
 ip helper-address 192.168.40.4

exit
write memory
```
---

### Na CLI do Router

```
configure terminal

interface GigabitEthernet0/0/0
 no ip helper-address 192.168.40.4
 no ip address
 no shutdown
exit

interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.200.1 255.255.255.192
 ip helper-address 192.168.40.4
 no shutdown
exit

write memory

```
---
### Na CLI do Switch (SW-FILIAL)
```
configure terminal

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10
exit

interface range fa0/2-3
 switchport mode access
 switchport access vlan 10
exit

write memory
```
