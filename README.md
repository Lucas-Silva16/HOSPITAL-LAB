# HOSPITAL-LAB  || Packet Tracer

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

### STP EM AMBOS OS CORE SW

```
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,50,60,99,100 root primary
```

### STP nos restantes SW

```
spanning-tree mode rapid-pvst
```

---