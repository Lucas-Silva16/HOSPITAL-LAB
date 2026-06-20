# HOSPITAL-LAB  || Packet Tracer


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


## 1ºa Configuracao

---

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
