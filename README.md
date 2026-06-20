# HOSPITAL-LAB  || Packet Tracer


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