# Etapa 4 · Wi-Fi e IoT

Arquivo gerado ao fim da etapa: `rede-v4-wifi-iot.pkt`. Testes: seção "Etapa 4" de [`docs/testes.md`](../docs/testes.md).

## Objetivo

Atender os dois requisitos sem fio e o de IoT: Wi-Fi de funcionários nos três andares (SSID EMPRESA, VLAN 30), Wi-Fi público separado no térreo (SSID VISITANTES, VLAN 40) e uma cafeteira IoT por copa (VLAN 50), registrada no controlador SRV-IOT. Os 60 funcionários do térreo só têm Wi-Fi, então é aqui que o térreo passa a ter usuários.

## O que se configura onde

| Equipamento | Como | Ligação | VLAN | O que recebe |
|---|---|---|---|---|
| AP-A2 | Interface gráfica | SW-A2 Fa0/20 | 30 | SSID EMPRESA, WPA2-PSK |
| AP-A1 | Interface gráfica | SW-A1 Fa0/20 | 30 | SSID EMPRESA, WPA2-PSK |
| AP-T-FUNC | Interface gráfica | SW-T Fa0/1 | 30 | SSID EMPRESA, WPA2-PSK |
| AP-T-PUB | Interface gráfica | SW-T Fa0/2 | 40 | SSID VISITANTES, WPA2-PSK |
| IoT-Copa-A2 | Interface gráfica | SW-A2 Fa0/21 | 50 | Adaptador Fast Ethernet, DHCP, Remote Server |
| IoT-Copa-A1 | Interface gráfica | SW-A1 Fa0/21 | 50 | Idem |
| IoT-Copa-T | Interface gráfica | SW-T Fa0/3 | 50 | Idem |
| SRV-IOT | Interface gráfica | SW-DC Fa0/3 | 60 | Serviço IoT ligado |
| 11 smartphones | Interface gráfica | Sem fio | 30 ou 40 | 8 de funcionários (EMPRESA), 3 de visitantes (VISITANTES) |
| SW-A1, SW-A2, SW-T | CLI | — | — | Portas dos APs e das copas |

As portas são as do `configs/`: nos andares, AP na Fa0/20 e IoT na Fa0/21; no térreo, AP-T-FUNC na Fa0/1, AP-T-PUB na Fa0/2 e IoT na Fa0/3.

**Um AP por SSID.** O AP do Packet Tracer anuncia um SSID só e não faz trunk. Por isso cada AP fica numa porta *access* da VLAN do seu SSID, e o térreo tem dois APs. Num projeto real, um AP anunciaria os dois SSIDs, cada um mapeado a uma VLAN por trunk.

## Comandos (CLI)

### SW-A1 e SW-A2

```
enable
configure terminal
interface FastEthernet0/20
 description AP-FUNC
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown
interface FastEthernet0/21
 description IOT-COPA
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
 no shutdown
end
write memory
```

### SW-T

```
enable
configure terminal
interface FastEthernet0/1
 description AP-FUNC
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown
interface FastEthernet0/2
 description AP-PUBLICO
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 no shutdown
interface FastEthernet0/3
 description IOT-COPA
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
 no shutdown
end
write memory
```

Os trunks da etapa 1 já levam as VLANs certas: 30 e 50 para os andares; 30, 40 e 50 para o térreo. A VLAN 40 só chega ao SW-T.

## Interface gráfica (não é CLI)

### Access points

Em cada AP, Config > **Port 1**:

| AP | SSID | Authentication | PSK Pass Phrase | Encryption |
|---|---|---|---|---|
| AP-A2, AP-A1, AP-T-FUNC | `EMPRESA` | WPA2-PSK | senha de teste dos funcionários | AES |
| AP-T-PUB | `VISITANTES` | WPA2-PSK | senha de teste dos visitantes | AES |

Os três APs do SSID EMPRESA usam o **mesmo SSID e a mesma senha**. Assim formam um único ESS, e o smartphone pode passar de um para outro sem reconfigurar. A senha dos visitantes é diferente: quem tem uma não entra na outra rede.

### Smartphones

Em cada smartphone, Config > **Wireless0**: o SSID (EMPRESA para os 8 de funcionários, VISITANTES para os 3 de visitantes), Authentication WPA2-PSK, a mesma pass phrase do AP e IP Configuration em **DHCP**. Posicionar cada um dentro da área de cobertura do AP correspondente.

### SRV-IOT

Services > **IoT** > **On**. Depois, pelo navegador de um PC que alcance o datacenter (Desktop > Web Browser > `http://10.10.10.68`), criar a conta de usuário em "Sign up now". Essa conta é a que as cafeteiras usam para se registrar.

### Cafeteiras IoT (IoT-Copa-A2, IoT-Copa-A1, IoT-Copa-T)

1. Advanced > I/O Config > Network Adapter: trocar pelo adaptador **Fast Ethernet cabeado** (`PT-IOT-NM-1CFE`).
2. Ligar o cabo na porta IOT-COPA do switch do andar.
3. Config > **FastEthernet0**: IP Configuration em **DHCP**. O IP vem do escopo IOT (10.10.10.0/26).
4. Config > Settings > IoT Server: **Remote Server**, Server Address `10.10.10.68`, o usuário e a senha criados no SRV-IOT, e **Connect**.

O adaptador cabeado foi escolhido em vez do Wi-Fi porque a IoT tem VLAN própria (50) e o AP do Packet Tracer só atende uma VLAN. Uma cafeteira no Wi-Fi cairia na VLAN 30, junto com os funcionários.

## Verificação

| Ação | Onde | O que conferir |
|---|---|---|
| `ipconfig` | Smartphone de funcionário | IP em 10.10.4.0/22 |
| `ping 10.10.10.67` | Smartphone de funcionário | Responde (Wi-Fi corporativo alcança o datacenter) |
| `ipconfig` | Smartphone de visitante | IP em 10.10.8.0/23, DNS 203.0.113.10 |
| `ping 203.0.113.10` | Smartphone de visitante | Responde |
| Página `http://10.10.10.68`, após login | Navegador de um PC | As 3 cafeteiras listadas como registradas |
| `show mac address-table interface fa0/20` | SW-A1 ou SW-A2 | MACs dos smartphones aprendidos na porta do AP, na VLAN 30 |

## Problemas comuns

- **Smartphone não associa:** SSID com maiúscula/minúscula diferente, ou pass phrase diferente da do AP. O Packet Tracer não avisa que a senha está errada; só não conecta.
- **Smartphone associa mas fica com 169.254.x.x:** a porta do AP está na VLAN errada ou em parking (desligada), ou a VLAN não está no trunk daquele switch.
- **Visitante pega IP da VLAN 30:** ele associou ao AP errado. Os SSIDs precisam ser diferentes e a senha de visitante não pode ser a mesma dos funcionários.
- **Cafeteira não registra no SRV-IOT:** sem IP (adaptador ainda é o sem fio, ou a porta não está na VLAN 50), serviço IoT desligado no servidor, ou usuário e senha diferentes da conta criada.
- **Depois da etapa 5, a cafeteira para de registrar:** a ACL-IOT só libera o tráfego para o host 10.10.10.68. Se o endereço do Remote Server estiver diferente, ele é bloqueado.
