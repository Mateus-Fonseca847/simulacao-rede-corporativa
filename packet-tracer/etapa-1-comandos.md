# Etapa 1 · VLANs e trunks

Arquivo gerado ao fim da etapa: `rede-v1-vlans.pkt`. Testes: seção "Etapa 1" de [`docs/testes.md`](../docs/testes.md).

## Objetivo

Montar a camada 2: os cinco switches, as VLANs por função, os quatro trunks 802.1Q até o core com VLAN nativa dedicada (998), as portas de PC + telefone, as portas sem uso estacionadas na VLAN 999 e a gerência (VLAN 99) com IP em cada switch. Ainda não há roteamento entre VLANs.

## O que se configura onde

| Equipamento | Modelo no Packet Tracer | O que recebe nesta etapa |
|---|---|---|
| Core-L3 | 3650-24PS | VLANs, trunks Gi1/0/1 a Gi1/0/4, SVI da VLAN 99 (10.10.10.129), parking Gi1/0/5–23 |
| SW-T (térreo) | 2960-24TT | VLANs, trunk Gi0/1, gerência 10.10.10.134, parking |
| SW-DC (datacenter) | 2960-24TT | VLANs, trunk Gi0/1, gerência 10.10.10.131, parking |
| SW-A1 (1º andar) | 3560-24PS | VLANs, trunk Gi0/1, portas PC + telefone Fa0/1–9, gerência 10.10.10.132, parking |
| SW-A2 (2º andar) | 3560-24PS | VLANs, trunk Gi0/1, portas PC + telefone Fa0/1–15, gerência 10.10.10.133, parking |

SW-A1 e SW-A2 são **3560-24PS, e não 2960**, porque precisam de PoE para alimentar os telefones IP pelo cabo (o "P" do nome indica portas PoE). O 3560 aceita dois encapsulamentos de trunk (ISL e 802.1Q), então exige `switchport trunk encapsulation dot1q` **antes** de `switchport mode trunk`. O 2960 e o 3650 só fazem 802.1Q e não têm esse comando.

### Cabos

| De | Para | VLANs no trunk |
|---|---|---|
| Core-L3 Gi1/0/1 | SW-T Gi0/1 | 30, 40, 50, 99 |
| Core-L3 Gi1/0/2 | SW-DC Gi0/1 | 60, 99 |
| Core-L3 Gi1/0/3 | SW-A1 Gi0/1 | 10, 20, 30, 50, 99 |
| Core-L3 Gi1/0/4 | SW-A2 Gi0/1 | 10, 20, 30, 50, 99 |

Portas reservadas para as próximas etapas (não estacionar agora): SW-DC Fa0/1–3 (servidores, etapa 3) e Fa0/24 (PC-ADMIN, etapa 5); SW-A1 e SW-A2 Fa0/20–21 (AP e IoT, etapa 4); SW-T Fa0/1–3 (APs e IoT, etapa 4).

## Comandos

### Bloco comum: VLANs (colar nos cinco switches, trocando o hostname)

```
enable
configure terminal
hostname SW-A1
no ip domain-lookup
vtp mode transparent
vlan 10
 name DADOS
vlan 20
 name VOZ
vlan 30
 name WIFI-FUNC
vlan 40
 name WIFI-PUBLICO
vlan 50
 name IOT
vlan 60
 name DATACENTER
vlan 99
 name GERENCIA
vlan 998
 name NATIVA
vlan 999
 name PARKING
exit
```

`vtp mode transparent` impede que um switch apague ou sobrescreva as VLANs dos outros por VTP. Cada switch tem a sua lista, e a lista é a mesma em todos.

### Core-L3 (3650-24PS)

```
interface GigabitEthernet1/0/1
 description UPLINK-SW-T
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 30,40,50,99
 switchport nonegotiate
interface GigabitEthernet1/0/2
 description UPLINK-SW-DC
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 60,99
 switchport nonegotiate
interface GigabitEthernet1/0/3
 description UPLINK-SW-A1
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,50,99
 switchport nonegotiate
interface GigabitEthernet1/0/4
 description UPLINK-SW-A2
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,50,99
 switchport nonegotiate
interface range GigabitEthernet1/0/5 - 23
 description PARKING
 switchport mode access
 switchport access vlan 999
 shutdown
interface Vlan99
 description GERENCIA
 ip address 10.10.10.129 255.255.255.224
 no shutdown
interface Vlan1
 shutdown
end
write memory
```

A Gi1/0/24 fica livre: vira porta roteada na etapa 2.

### SW-A1 e SW-A2 (3560-24PS)

```
interface GigabitEthernet0/1
 description UPLINK-CORE
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 10,20,30,50,99
 switchport nonegotiate
interface range FastEthernet0/1 - 9
 description PC-FONE
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20
 spanning-tree portfast
interface range FastEthernet0/10 - 19 , FastEthernet0/22 - 24 , GigabitEthernet0/2
 description PARKING
 switchport mode access
 switchport access vlan 999
 shutdown
interface Vlan99
 description GERENCIA
 ip address 10.10.10.132 255.255.255.224
 no shutdown
interface Vlan1
 shutdown
exit
ip default-gateway 10.10.10.129
end
write memory
```

No **SW-A2** mudam três coisas: as portas PC-FONE vão de `Fa0/1 - 15`, o parking passa a ser `FastEthernet0/16 - 19 , FastEthernet0/22 - 24 , GigabitEthernet0/2` e o IP de gerência é `10.10.10.133`.

### SW-T (2960-24TT)

```
interface GigabitEthernet0/1
 description UPLINK-CORE
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 30,40,50,99
 switchport nonegotiate
interface range FastEthernet0/4 - 24 , GigabitEthernet0/2
 description PARKING
 switchport mode access
 switchport access vlan 999
 shutdown
interface Vlan99
 description GERENCIA
 ip address 10.10.10.134 255.255.255.224
 no shutdown
interface Vlan1
 shutdown
exit
ip default-gateway 10.10.10.129
end
write memory
```

### SW-DC (2960-24TT)

```
interface GigabitEthernet0/1
 description UPLINK-CORE
 switchport mode trunk
 switchport trunk native vlan 998
 switchport trunk allowed vlan 60,99
 switchport nonegotiate
interface range FastEthernet0/4 - 23 , GigabitEthernet0/2
 description PARKING
 switchport mode access
 switchport access vlan 999
 shutdown
interface Vlan99
 description GERENCIA
 ip address 10.10.10.131 255.255.255.224
 no shutdown
interface Vlan1
 shutdown
exit
ip default-gateway 10.10.10.129
end
write memory
```

### PCs de teste

Para o teste 1.3, dois PCs na VLAN 10 (um em cada andar) recebem IP fixo temporário da faixa 10.10.0.0/23, com máscara 255.255.254.0, pela aba Desktop > IP Configuration. Na etapa 3 eles passam para DHCP.

## Verificação

| Comando | Onde | O que conferir |
|---|---|---|
| `show vlan brief` | Todos os switches | VLANs 10 a 999 com os nomes certos; portas PC-FONE na VLAN 10; parking na 999 |
| `show interfaces trunk` | Core e switches de acesso | Modo `on`, encapsulamento `802.1q`, native vlan `998`, lista de VLANs permitidas igual nos dois lados |
| `show interfaces status` | Switches de acesso | Portas sem uso como `disabled` e VLAN 999 |
| `ping 10.10.10.131` a `.134` | Core | Os quatro switches respondem pela VLAN 99 |
| `ping` entre os dois PCs | PC do 1º andar | Responde (mesma VLAN, andares diferentes, passando pelo core em camada 2) |

## Problemas comuns

- **`switchport mode trunk` dá erro no 3560** ("trunk encapsulation is Auto"): falta `switchport trunk encapsulation dot1q` antes. No 2960 e no 3650 o comando não existe, e não é preciso.
- **Trunk não sobe ou aparece "native VLAN mismatch"** no log: a nativa precisa ser 998 nos dois lados do cabo.
- **Ping entre PCs da VLAN 10 falha:** a VLAN 10 não está na `allowed vlan` de um dos trunks, ou o PC está com máscara /24 em vez de /23.
- **Ping do core à gerência falha:** a SVI 99 do core ou do switch está `administratively down` (faltou `no shutdown`), ou a VLAN 99 não foi permitida no trunk.
- **Porta da etapa 4 entrou no parking por engano:** basta reconfigurar a porta na etapa 4 com `no shutdown`.
