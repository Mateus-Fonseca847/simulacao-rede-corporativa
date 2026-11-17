# Etapa 2 · Roteamento

Arquivo gerado ao fim da etapa: `rede-v2-roteamento.pkt`. Testes: seção "Etapa 2" de [`docs/testes.md`](../docs/testes.md).

## Objetivo

Ligar a camada 3: o core passa a rotear entre as VLANs (uma SVI por VLAN, que é o gateway de cada sub-rede), ganha um link roteado até o roteador de borda e uma rota padrão. O R-BORDA recebe as duas interfaces e uma única rota de volta para `10.10.0.0/16`. Ao fim, qualquer VLAN alcança qualquer outra e a "internet" simulada. As restrições só chegam na etapa 5.

## O que se configura onde

| Equipamento | Modelo | O que recebe nesta etapa |
|---|---|---|
| Core-L3 | 3650-24PS | `ip routing`; SVIs das VLANs 10, 20, 30, 40, 50 e 60; relay DHCP nas SVIs de 10 a 50; Gi1/0/24 roteada (10.10.255.2/30); rota padrão |
| R-BORDA | 2811 | Hostname; Fa0/0 10.10.255.1/30 (lado core); Fa0/1 203.0.113.1/24 (lado internet); rota para 10.10.0.0/16 |
| INTERNET | Server-PT | IP fixo 203.0.113.10/24, gateway 203.0.113.1 (pela interface gráfica) |

### Cabos

| De | Para |
|---|---|
| Core-L3 Gi1/0/24 | R-BORDA Fa0/0 |
| R-BORDA Fa0/1 | INTERNET Fa0 |

## Comandos

### Core-L3

```
enable
configure terminal
ip routing
interface Vlan10
 description GW-DADOS
 ip address 10.10.0.1 255.255.254.0
 ip helper-address 10.10.10.66
 no shutdown
interface Vlan20
 description GW-VOZ
 ip address 10.10.2.1 255.255.254.0
 ip helper-address 10.10.10.66
 no shutdown
interface Vlan30
 description GW-WIFI-FUNC
 ip address 10.10.4.1 255.255.252.0
 ip helper-address 10.10.10.66
 no shutdown
interface Vlan40
 description GW-WIFI-PUBLICO
 ip address 10.10.8.1 255.255.254.0
 ip helper-address 10.10.10.66
 no shutdown
interface Vlan50
 description GW-IOT
 ip address 10.10.10.1 255.255.255.192
 ip helper-address 10.10.10.66
 no shutdown
interface Vlan60
 description GW-DATACENTER
 ip address 10.10.10.65 255.255.255.192
 no shutdown
interface GigabitEthernet1/0/24
 description LINK-ROTEADOR
 no switchport
 ip address 10.10.255.2 255.255.255.252
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 10.10.255.1
end
write memory
```

A SVI 60 não tem `ip helper-address`: o servidor DHCP está dentro da própria VLAN 60, e lá todos os hosts usam IP fixo. O `helper-address` já fica configurado nesta etapa, mesmo que o SRV-DHCP só apareça na etapa 3.

`no switchport` transforma a Gi1/0/24 em porta roteada: ela deixa de pertencer a uma VLAN e passa a ter IP próprio, como uma interface de roteador.

### R-BORDA (2811)

```
enable
configure terminal
hostname R-BORDA
no ip domain-lookup
interface FastEthernet0/0
 description LINK-CORE
 ip address 10.10.255.1 255.255.255.252
 no shutdown
interface FastEthernet0/1
 description INTERNET
 ip address 203.0.113.1 255.255.255.0
 no shutdown
exit
ip route 10.10.0.0 255.255.0.0 10.10.255.2
end
write memory
```

Uma rota só cobre a rede interna inteira porque todo o endereçamento está dentro de `10.10.0.0/16` (sumarização).

### INTERNET (interface gráfica)

Desktop > IP Configuration > Static: IP `203.0.113.10`, máscara `255.255.255.0`, gateway `203.0.113.1`.

## Verificação

| Comando | Onde | O que conferir |
|---|---|---|
| `show ip interface brief` | Core | SVIs 10 a 60 e 99 `up/up`; Gi1/0/24 com 10.10.255.2 |
| `show ip route` | Core | Uma rota `C` por VLAN, `C` para 10.10.255.0/30 e `S*` 0.0.0.0/0 via 10.10.255.1 |
| `show ip route` | R-BORDA | `S` 10.10.0.0/16 via 10.10.255.2 |
| `ping 10.10.255.1` | Core | Responde |
| `ping 10.10.0.1` | PC da VLAN 10 | O gateway responde |
| `ping 203.0.113.10` | PC da VLAN 10 | Responde (ainda sem NAT: o servidor responde direto para o 10.x, porque o R-BORDA tem rota de volta) |

## Problemas comuns

- **SVI fica `down/down`:** a SVI só sobe se a VLAN existir no core e houver ao menos uma porta ativa nela, o que inclui um trunk que a permita. A VLAN 40 sobe pelo trunk do SW-T, a 60 pelo do SW-DC.
- **`ip address` recusado na Gi1/0/24:** faltou `no switchport` antes.
- **PC alcança o gateway mas não a internet:** falta a rota padrão no core ou a rota de volta no R-BORDA. Sem a rota de volta, o ping sai mas a resposta se perde.
- **Ping entre VLANs falha:** faltou `ip routing`, ou o PC está sem gateway configurado.
- **Interfaces do 2811 `administratively down`:** em roteador as interfaces vêm desligadas, então o `no shutdown` é obrigatório.
