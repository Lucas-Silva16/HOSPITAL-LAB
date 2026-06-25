![Cisco Packet Tracer](https://img.shields.io/badge/Cisco_Packet_Tracer-Network_Simulation-1BA0D7?style=for-the-badge&logo=cisco)
![OSPF](https://img.shields.io/badge/Routing-OSPF-green?style=for-the-badge)
![HSRP](https://img.shields.io/badge/Redundancy-HSRP-orange?style=for-the-badge)
![VLANs](https://img.shields.io/badge/Segmentation-VLANs-blue?style=for-the-badge)
![ASA](https://img.shields.io/badge/Firewall-ASA_5505-red?style=for-the-badge)

<img width="1980" height="1080" alt="hospital-network-infrastructure" src="https://github.com/user-attachments/assets/11c74294-902c-4f14-8041-b5afa0ee91a6" />

# HOSPITAL-LAB Infraestrutura de Rede Hospitalar

Após a conclusão do curso de Network Technician da Cisco, desenvolvi este laboratório prático para consolidar os conhecimentos adquiridos. O projeto simula a infraestrutura de uma rede hospitalar, exigindo a integração de várias tecnologias essenciais: segurança no acesso (VLANs, Port Security, DHCP Snooping), alta disponibilidade no Core (HSRP, LACP, Rapid PVST+) e roteamento seguro (OSPF, Túnel GRE, Firewall ASA).

Mais do que a simples configuração dos equipamentos, o maior desafio e aprendizagem deste projeto esteve no troubleshooting ativo. Resolver problemas reais de encapsulamento, bloqueios de Spanning Tree e permissões de Firewall consolidou a minha capacidade de analisar, diagnosticar e gerir redes complexas.

Para manter a clareza e concisão deste relatório, os blocos de configuração e os comandos de verificação aqui documentados servem como exemplos representativos da lógica arquitetónica aplicada. Funcionalidades como o DHCP Snooping, o encaminhamento OSPF ou o Port Security foram aplicadas globalmente a todos os equipamentos das respetivas camadas (Distribuição, Core e Acesso), embora apenas um dispositivo exemplar esteja aqui detalhado para evitar redundância extrema. O ficheiro original do Packet Tracer contém a configuração integral de todos os nós da rede.

---

## Índice

1. [Tabela de VLANs](#1-tabela-de-vlans)
2. [Links L3 e Conexões](#2-links-l3-e-conexões)
3. [Configuração Base — Todos os Dispositivos](#3-configuração-base--todos-os-dispositivos)
4. [VLANs e Trunking](#4-vlans-e-trunking)
5. [EtherChannel (LACP)](#5-etherchannel-lacp)
6. [Spanning Tree — Rapid PVST+](#6-spanning-tree--rapid-pvst)
7. [SVIs e Roteamento L3 nos Core Switches](#7-svis-e-roteamento-l3-nos-core-switches)
8. [HSRP — Alta Disponibilidade de Gateway](#8-hsrp--alta-disponibilidade-de-gateway)
9. [OSPF](#9-ospf)
10. [WAN — Serial e Túnel GRE](#10-wan--serial-e-túnel-gre)
11. [Firewall ASA 5505](#11-firewall-asa-5505)
12. [DHCP — Servidor e Relay](#12-dhcp--servidor-e-relay)
13. [Portas Access nos Access Switches](#13-portas-access-nos-access-switches)
14. [Port Security](#14-port-security)
15. [DHCP Snooping](#15-dhcp-snooping)

---
<img width="1901" height="751" alt="image" src="https://github.com/user-attachments/assets/2de4a0b4-6fd7-4335-8c3f-855fe3652490" />


## 1. Tabela de VLANs

| VLAN | Nome | Hosts Necessários | Hosts Úteis | Prefixo | Rede | Máscara | Gateway (HSRP VIP) | Broadcast |
|------|------|:-----------------:|:-----------:|:-------:|------|---------|:------------------:|-----------|
| 10 | Dados | 60 | 62 | /26 | 192.168.10.0 | 255.255.255.192 | 192.168.10.1 | 192.168.10.63 |
| 50 | WiFi Doentes | 50 | 62 | /26 | 192.168.50.0 | 255.255.255.192 | 192.168.50.1 | 192.168.50.63 |
| 20 | VoIP | 40 | 62 | /26 | 192.168.20.0 | 255.255.255.192 | 192.168.20.1 | 192.168.20.63 |
| 30 | Clínica Crítica | 20 | 30 | /27 | 192.168.30.0 | 255.255.255.224 | 192.168.30.1 | 192.168.30.31 |
| 60 | Gestão | 15 | 30 | /27 | 192.168.60.0 | 255.255.255.224 | 192.168.60.1 | 192.168.60.31 |
| 40 | Servidores | 10 | 14 | /28 | 192.168.40.0 | 255.255.255.240 | 192.168.40.1 | 192.168.40.15 |
| 100 | DMZ | 5 | 6 | /29 | 192.168.100.0 | 255.255.255.248 | 192.168.100.1 | 192.168.100.7 |
| 99 | Nativa | 2 | 2 | /30 | 192.168.99.0 | 255.255.255.252 | — | 192.168.99.3 |

---

## 2. Links L3 e Conexões

| Link / Conexão | Dispositivo Local | Interface Local | Endereço IP | Máscara | Prefixo | Dispositivo Remoto | Interface Remota |
|:---|:---|:---|:---|:---|:---:|:---|:---|
| WAN Sede–Filial | R0-SEDE | Se0/2/0 | 10.0.0.1 | 255.255.255.252 | /30 | R1-FILIAL | Se0/1/0 |
| WAN Sede–Filial | R1-FILIAL | Se0/1/0 | 10.0.0.2 | 255.255.255.252 | /30 | R0-SEDE | Se0/2/0 |
| GRE Tunnel | R0-SEDE | Tunnel0 | 172.16.0.1 | 255.255.255.252 | /30 | R1-FILIAL | Tunnel0 |
| GRE Tunnel | R1-FILIAL | Tunnel0 | 172.16.0.2 | 255.255.255.252 | /30 | R0-SEDE | Tunnel0 |
| L3 Core A → Router | CORE-SW-A | Gi1/0/5 | 192.168.2.1 | 255.255.255.252 | /30 | R0-SEDE | Gi0/0/1 |
| L3 Core B → Router | CORE-SW-B | Gi1/0/5 | 192.168.2.5 | 255.255.255.252 | /30 | R0-SEDE | — |
| Rede Local Filial | R1-FILIAL | Gi0/0/0.10 | 192.168.200.1 | 255.255.255.192 | /26 | SW-FILIAL | — |
| Outside ASA | ASA | Vlan2 (E0/0) | 203.0.113.1 | 255.255.255.252 | /30 | R0-SEDE | Gi0/0/0 |

---

## 3. Configuração Base — Todos os Dispositivos

Aplicado em todos os switches e routers antes de qualquer configuração específica.

```
enable
configure terminal
hostname <NOME-DO-DISPOSITIVO>
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
```

### Desativar portas não utilizadas

**Core e Distribution Switches (portas GigabitEthernet):**
```
interface range gi1/0/X - X
 shutdown
 description PORTA_INATIVA
exit
interface range gi1/1/X - X
 shutdown
 description PORTA_INATIVA
exit
```

**Access Switches e SW-FILIAL (portas FastEthernet e Gigabit):**
```
interface range fa0/X - X
 shutdown
 description PORTA_INATIVA
exit
```

**R0-SEDE:**
```
interface range gi0/1/0-3
 shutdown
 description PORTA_INATIVA
exit
interface Serial0/2/1
 shutdown
 description PORTA_INATIVA
exit
```

**R1-FILIAL:**
```
interface range gi0/0/1-2
 shutdown
 description PORTA_INATIVA
exit
interface Serial0/1/1
 shutdown
 description PORTA_INATIVA
exit
```

```
end
write memory
```

---

## 4. VLANs e Trunking

### 4.1 Criação de VLANs — Todos os Switches

```
configure terminal

vlan 10
 name Dados
exit
vlan 20
 name VOIP
exit
vlan 30
 name Clinica
exit
vlan 40
 name Servidores
exit
vlan 50
 name WiFi Doentes
exit
vlan 60
 name Gestao
exit
vlan 99
 name Nativa
exit
vlan 100
 name DMZ
exit

end
write memory
```

### 4.2 Configuração de Trunks

```
interface <PORTA>
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,50,60,99,100
exit
```

###  Verificação — CORE-SW-A

```
CORE-SW-A#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gig1/0/6 ... Gig1/1/4
10   Dados                            active
20   VOIP                             active
30   Clinica                          active
40   Servidores                       active
50   VLAN0050                         active
60   Gestao                           active
99   Nativa                           active
100  DMZ                              active
```

>  Todas as VLANs do projeto presentes e ativas.

```
CORE-SW-A#show interfaces trunk

Port   Mode  Encapsulation  Status    Native vlan
Po1    on    802.1q         trunking  99
Gi1/0/3 on   802.1q         trunking  99
Gi1/0/4 on   802.1q         trunking  99

Port    Vlans in spanning tree forwarding state and not pruned
Po1     10,20,30,40,50,60,99,100
Gi1/0/3 10,20,30,40,50,60,99,100
Gi1/0/4 10,20,30,40,50,60,99,100
```

>  Trunks ativos com VLAN Nativa 99 e todas as VLANs permitidas em forwarding.

###  Verificação — AC-SW-URGENCIAS

```
AC-SW-URGENCIAS#show vlan brief

VLAN Name    Status  Ports
10   Dados   active  Fa0/4, Fa0/5
20   VOIP    active  Fa0/5
50   VLAN0050 active Fa0/3
99   Nativa  active
```

```
AC-SW-URGENCIAS#show interfaces trunk

Port   Mode  Status    Native vlan
Fa0/1  on    trunking  99
Fa0/2  on    trunking  99
Fa0/6  on    trunking  99
```

>  Portas de acesso nas VLANs corretas; uplinks em trunk com VLAN 99 nativa.

###  Verificação — DISTR-SW-1

```
DISTR-SW-1#show interfaces trunk

Port     Mode  Status    Native vlan
Gi1/0/1  on    trunking  99
Gi1/0/2  on    trunking  99
Gi1/0/4  on    trunking  99
Gi1/0/5  on    trunking  99
```

>  Todas as portas trunk no DISTR-SW-1 ativas. Gi1/0/2 aparece como `none` no forwarding — comportamento esperado do STP (porta bloqueada pelo Rapid-PVST+).

---

## 5. EtherChannel (LACP)

Configurado entre CORE-SW-A e CORE-SW-B para agregar duas ligações Gigabit em redundância e maior largura de banda.

```
configure terminal

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

###  Verificação — CORE-SW-A

```
CORE-SW-A#show etherchannel summary

Group  Port-channel  Protocol  Ports
------+-------------+---------+---------------------------
1      Po1(SU)       LACP      Gi1/0/1(P)  Gi1/0/2(P)
```

>  Port-Channel1 em uso (`U`), Layer 2 (`S`), ambas as portas ativas (`P`) via LACP.

---

## 6. Spanning Tree — Rapid PVST+

### Core Switches (Root Bridge)

```
configure terminal
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,50,60,99,100 root primary
end
write memory
```

### Restantes Switches

```
configure terminal
spanning-tree mode rapid-pvst
end
write memory
```

###  Verificação — DISTR-SW-1

```
DISTR-SW-1#show spanning-tree (excerto VLAN 10)

VLAN0010
  Root ID  Priority  24586
           Address   0001.4272.47C7     ← MAC do CORE-SW-A
           Cost      4
           Port      1 (GigabitEthernet1/0/1)

  Bridge ID Priority  32778

Interface   Role  Sts  Cost
Gi1/0/1     Root  FWD  4      ← uplink para CORE-SW-A
Gi1/0/2     Altn  BLK  4      ← uplink para CORE-SW-B bloqueado (redundância)
Gi1/0/4     Desg  FWD  19
Gi1/0/5     Desg  FWD  19
```

>  CORE-SW-A eleito Root Bridge. Porta alternativa para CORE-SW-B bloqueada pelo STP — redundância funcional.

---

## 7. SVIs e Roteamento L3 nos Core Switches

### CORE-SW-A (IPs .2 em cada VLAN)

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
interface vlan 99
 ip address 192.168.99.2 255.255.255.252
 no shutdown
exit
interface vlan 100
 ip address 192.168.100.2 255.255.255.248
 no shutdown
exit

end
write memory
```

### CORE-SW-B (IPs .3 em cada VLAN — VLAN 99 não configurada no SW-B)

> Mesma configuração, substituindo `.2` por `.3` em cada SVI. A VLAN 99 é omitida no CORE-SW-B.

### Ligação L3 ao Router (porta routed)

**CORE-SW-A:**
```
interface GigabitEthernet1/0/5
 no switchport
 ip address 192.168.2.1 255.255.255.252
 no shutdown
exit
```

**CORE-SW-B:**
```
interface GigabitEthernet1/0/5
 no switchport
 ip address 192.168.2.5 255.255.255.252
 no shutdown
exit
```

---

## 8. HSRP — Alta Disponibilidade de Gateway

O HSRP cria um IP virtual (VIP) partilhado entre CORE-SW-A (Active) e CORE-SW-B (Standby). Os end-devices usam sempre o VIP como gateway — a comutação é automática em caso de falha.

### CORE-SW-A — Active (priority 110)

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

### CORE-SW-B — Standby (priority 90)

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

###  Verificação — CORE-SW-A

```
CORE-SW-A#show standby brief

Interface  Grp  Pri  P  State   Active  Standby        Virtual IP
Vl10       1    110  P  Active  local   192.168.10.3   192.168.10.1
Vl20       1    110  P  Active  local   192.168.20.3   192.168.20.1
Vl30       1    110  P  Active  local   192.168.30.3   192.168.30.1
Vl40       1    110  P  Active  local   192.168.40.3   192.168.40.1
Vl50       1    110  P  Active  local   192.168.50.3   192.168.50.1
Vl60       1    110  P  Active  local   192.168.60.3   192.168.60.1
```

>  CORE-SW-A é Active em todas as VLANs. CORE-SW-B (.3) em Standby. VIPs (.1) operacionais.

---

## 9. OSPF

### CORE-SW-A

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

### CORE-SW-B

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

### DISTR-SW-1, DISTR-SW-2, DISTR-SW-3

```
configure terminal

ip routing

router ospf 1
 router-id 3.3.3.3   ! 4.4.4.4 para DISTR-SW-2 | 5.5.5.5 para DISTR-SW-3
 passive-interface default
 no passive-interface GigabitEthernet1/0/X   ! porta uplink para o Core
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

###  Verificação — CORE-SW-A (`show ip route`)

```
Gateway of last resort is 192.168.2.2 to network 0.0.0.0

O    10.0.0.0/30      [110/65]  via 192.168.2.2, GigabitEthernet1/0/5
C    192.168.2.0/30             GigabitEthernet1/0/5
C    192.168.10.0/26            Vlan10
C    192.168.20.0/26            Vlan20
C    192.168.30.0/27            Vlan30
C    192.168.40.0/28            Vlan40
C    192.168.50.0/26            Vlan50
C    192.168.60.0/27            Vlan60
C    192.168.99.0/30            Vlan99
O    192.168.200.0/26 [110/66]  via 192.168.2.2, GigabitEthernet1/0/5
S*   0.0.0.0/0        [1/0]     via 192.168.2.2
```

>  Rota OSPF para a WAN (10.0.0.0/30) e para a rede da Filial (192.168.200.0/26) aprendidas via R0-SEDE. Default route presente.

---

## 10. WAN — Serial e Túnel GRE

### R0-SEDE

```
configure terminal

interface Serial0/2/0
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shutdown
exit

interface Tunnel0
 ip address 172.16.0.1 255.255.255.252
 tunnel source Serial0/2/0
 tunnel destination 10.0.0.2
 no shutdown
exit
```

### R1-FILIAL

```
configure terminal

interface Serial0/1/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/0/0
 no ip address
 no shutdown
exit

interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.200.1 255.255.255.192
 ip helper-address 192.168.40.4
 no shutdown
exit

interface Tunnel0
 ip address 172.16.0.2 255.255.255.252
 tunnel source Serial0/1/0
 tunnel destination 10.0.0.1
 no shutdown
exit
```

> **Nota:** A interface `Gi0/0/0` do R1-FILIAL foi convertida para Router-on-a-Stick para resolver um problema de encapsulamento — o SW-FILIAL envia tráfego com etiqueta VLAN 10, e uma interface física plana rejeitava esse tráfego etiquetado.

###  Verificação — Túnel GRE (R0-SEDE)

```
R0-SEDE#show interfaces Tunnel0

Tunnel0 is up, line protocol is up (connected)
Internet address is 172.16.0.1/30
Tunnel source 10.0.0.1 (Serial0/2/0), destination 10.0.0.2
Tunnel protocol/transport GRE/IP
```

>  Túnel GRE ativo entre Sede (10.0.0.1) e Filial (10.0.0.2).

###  Verificação — Túnel GRE (R1-FILIAL)

```
R1-FILIAL#show interfaces Tunnel0

Tunnel0 is up, line protocol is up (connected)
Internet address is 172.16.0.2/30
Tunnel source 10.0.0.2 (Serial0/1/0), destination 10.0.0.1
Tunnel protocol/transport GRE/IP
```

>  Ambos os extremos do túnel UP/UP com encapsulamento GRE/IP.

###  Verificação — R0-SEDE (`show ip route`)

```
C    10.0.0.0/30        Serial0/2/0
C    172.16.0.0/30      Tunnel0
C    192.168.2.0/30     GigabitEthernet0/0/1
S    192.168.10.0/26    via 192.168.2.1
S    192.168.20.0/26    via 192.168.2.1
...
O    192.168.200.0/26   [110/65] via 10.0.0.2, Serial0/2/0
C    203.0.113.0/30     GigabitEthernet0/0/0
S*   0.0.0.0/0          via 203.0.113.1
```

>  Rede da filial (192.168.200.0/26) aprendida por OSPF. Default route aponta para o ASA (203.0.113.1).

---

## 11. Firewall ASA 5505

### Interfaces

```
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

### ACLs

```
! Permitir tráfego externo para servidores da DMZ
access-list OUTSIDE-TO-DMZ extended permit tcp any host 203.0.113.2 eq 80
access-list OUTSIDE-TO-DMZ extended permit tcp any host 203.0.113.2 eq 443
access-list OUTSIDE-TO-DMZ extended permit tcp any host 203.0.113.3 eq 25
access-list OUTSIDE-TO-DMZ extended permit tcp any host 203.0.113.3 eq 143
access-list OUTSIDE-TO-DMZ extended permit icmp any any
access-group OUTSIDE-TO-DMZ in interface outside

! Bloquear DMZ → rede interna; permitir o resto
access-list DMZ-TO-INSIDE extended deny ip 192.168.100.0 255.255.255.248 192.168.0.0 255.255.0.0
access-list DMZ-TO-INSIDE extended permit ip any any
access-group DMZ-TO-INSIDE in interface dmz
```

###  Verificação — ASA (`show route`)

```
S    192.168.0.0 255.255.0.0 [1/0] via 203.0.113.2, outside
C    192.168.100.0/29     dmz (Vlan3)
C    203.0.113.0/30       outside (Vlan2)
S*   0.0.0.0/0 [1/0]      via 203.0.113.2
```

>  ASA com rota estática para a rede interna via R0-SEDE e default route para Internet.

###  Verificação — ASA (`show access-list`)

```
access-list OUTSIDE-TO-DMZ; 5 elements
  line 1: permit tcp any host 203.0.113.2 eq www      (hitcnt=0)
  line 2: permit tcp any host 203.0.113.2 eq 443      (hitcnt=0)
  line 3: permit tcp any host 203.0.113.3 eq smtp     (hitcnt=0)
  line 4: permit tcp any host 203.0.113.3 eq 143      (hitcnt=0)
  line 5: permit icmp any any                         (hitcnt=0)

access-list DMZ-TO-INSIDE; 2 elements
  line 1: deny   ip 192.168.100.0/29 192.168.0.0/16  (hitcnt=0)
  line 2: permit ip any any                           (hitcnt=0)
```

>  ACLs aplicadas corretamente. ACLs de teste (`PERMITIR_TUDO`, `TESTE_TOTAL`, etc.) são resquícios de troubleshooting e podem ser removidas em produção com `no access-list <NOME>`.

---

## 12. DHCP — Servidor e Relay

### Problema identificado e resolução

Durante a implementação foram detetados dois problemas:

**Na Sede** — colisão de endereço L3: o servidor DHCP estava com o IP `192.168.40.3`, que colidia com a SVI do CORE-SW-B na VLAN 40. Solução: mover o servidor para `192.168.40.4` e atualizar os relays nos Core Switches.

**Na Filial** — quebra de encapsulamento: o SW-FILIAL enviava tráfego etiquetado com VLAN 10, mas a interface física do R1-FILIAL rejeitava frames etiquetados. Solução: converter a interface para Router-on-a-Stick com sub-interface `.10`.

### Configuração do Servidor DHCP Físico

```
IP Address:      192.168.40.4
Subnet Mask:     255.255.255.240
Default Gateway: 192.168.40.1
```

**Pools configurados** (DNS Server: `192.168.40.5` em todos):

| Pool | Gateway | Start IP | Máscara | Max Users |
|------|---------|----------|---------|-----------|
| POOL_VLAN10 | 192.168.10.1 | 192.168.10.10 | 255.255.255.192 | 52 |
| POOL_VLAN20 | 192.168.20.1 | 192.168.20.10 | 255.255.255.192 | 52 |
| POOL_VLAN30 | 192.168.30.1 | 192.168.30.10 | 255.255.255.224 | 20 |
| POOL_VLAN50 | 192.168.50.1 | 192.168.50.10 | 255.255.255.192 | 52 |
| POOL_VLAN60 | 192.168.60.1 | 192.168.60.10 | 255.255.255.224 | 20 |
| POOL_FILIAL | 192.168.200.1 | 192.168.200.10 | 255.255.255.192 | 53 |

### DHCP Relay — Core Switches

```
configure terminal

interface vlan 10
 no ip helper-address 192.168.40.3
 ip helper-address 192.168.40.4
exit
interface vlan 20
 no ip helper-address 192.168.40.3
 ip helper-address 192.168.40.4
exit
interface vlan 30
 no ip helper-address 192.168.40.3
 ip helper-address 192.168.40.4
exit
interface vlan 50
 no ip helper-address 192.168.40.3
 ip helper-address 192.168.40.4
exit
interface vlan 60
 no ip helper-address 192.168.40.3
 ip helper-address 192.168.40.4
exit

end
write memory
```

### SW-FILIAL — Trunk e Access

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

end
write memory
```

---

## 13. Portas Access nos Access Switches

Exemplo para **AC-SW-URGENCIAS** (adaptar para os restantes Access Switches):

```
! PC / dispositivo de dados
interface fa0/X
 switchport mode access
 switchport access vlan 10
exit

! PC com telefone IP (dados + VoIP)
interface fa0/X
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20
exit

! Access Point (WiFi Doentes) — apenas em AC-SW-URGENCIAS e AC-SW-INTERNAMENTO
interface fa0/X
 switchport mode access
 switchport access vlan 50
exit
```

---

## 14. Port Security

Configurado nos Access Switches para limitar o número de MACs por porta e proteger contra MAC flooding.

```
interface fa0/X
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
exit
```

###  Verificação — AC-SW-URGENCIAS

```
AC-SW-URGENCIAS#show port-security

Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
Fa0/4        2              1            0                  Shutdown
Fa0/5        2              0            0                  Shutdown
```

>  Port Security ativo. Fa0/4 com 1 MAC aprendido; Fa0/5 aguarda dispositivo. Zero violações.

---

## 15. DHCP Snooping

Configurado nos Distribution Switches para bloquear servidores DHCP não autorizados na rede.

```
configure terminal

ip dhcp snooping
ip dhcp snooping vlan 10,20,30,50

! Portas uplink para switches de confiança — trusted
interface GigabitEthernet1/0/1
 ip dhcp snooping trust
exit
interface GigabitEthernet1/0/2
 ip dhcp snooping trust
exit
interface GigabitEthernet1/0/4
 ip dhcp snooping trust
exit

! Portas para end-devices — untrusted (default)
interface GigabitEthernet1/0/5
 no ip dhcp snooping trust
exit

end
write memory
```

###  Verificação — DISTR-SW-1

```
DISTR-SW-1#show ip dhcp snooping

Switch DHCP snooping is enabled
DHCP snooping configured on VLANs: 10,20,30,50

Interface         Trusted  Rate limit (pps)
Gi1/0/1           yes      unlimited
Gi1/0/2           yes      unlimited
Gi1/0/4           yes      unlimited
Gi1/0/5           no       unlimited
```

>  DHCP Snooping ativo nas VLANs de utilizadores. Portas uplink trusted; porta para dispositivos finais untrusted — rogue DHCP servers bloqueados.

---

*Hospital São Lucas — Infraestrutura de Rede | Cisco Packet Tracer*
