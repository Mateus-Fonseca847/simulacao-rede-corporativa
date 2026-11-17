# Etapa 3 · Serviços (DHCP, DNS e VoIP)

Arquivo gerado ao fim da etapa: `rede-v3-servicos.pkt`. Testes: seção "Etapa 3" de [`docs/testes.md`](../docs/testes.md).

## Objetivo

Colocar os serviços no ar: servidores no datacenter com IP fixo, um escopo DHCP por VLAN (atendido pelo relay configurado na etapa 2), DNS interno e a telefonia IP no R-BORDA. Ao fim, os 24 PCs pegam IP sozinhos e os 24 telefones registram ramal.

## O que se configura onde

| Equipamento | Como | O que recebe nesta etapa |
|---|---|---|
| SW-DC | CLI | Portas Fa0/1–3 na VLAN 60, para os servidores |
| SRV-DHCP (10.10.10.66) | Interface gráfica | IP fixo; 5 escopos: DADOS, VOZ, WIFI-FUNC, WIFI-PUBLICO, IOT |
| SRV-DNS (10.10.10.67) | Interface gráfica | IP fixo; registros `intranet.empresa.local` e `www.exemplo.com` |
| SRV-IOT (10.10.10.68) | Interface gráfica | IP fixo (o serviço IoT é ligado na etapa 4) |
| R-BORDA | CLI | `telephony-service` e ramais 1001 a 1024 |
| 24 telefones IP 7960 | Cabos | 9 no SW-A1 (Fa0/1–9), 15 no SW-A2 (Fa0/1–15) |
| 24 PCs | Interface gráfica | DHCP; cada PC ligado na porta "PC" do seu telefone |

## Comandos (CLI)

### SW-DC: portas dos servidores

```
enable
configure terminal
interface FastEthernet0/1
 description SRV-DHCP
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast
 no shutdown
interface FastEthernet0/2
 description SRV-DNS
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast
 no shutdown
interface FastEthernet0/3
 description SRV-IOT
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast
 no shutdown
end
write memory
```

### R-BORDA: telefonia (CME)

```
enable
configure terminal
telephony-service
 max-ephones 30
 max-dn 30
 ip source-address 10.10.255.1 port 2000
 auto assign 1 to 30
exit
ephone-dn 1
 number 1001
ephone-dn 2
 number 1002
ephone-dn 3
 number 1003
ephone-dn 4
 number 1004
ephone-dn 5
 number 1005
ephone-dn 6
 number 1006
ephone-dn 7
 number 1007
ephone-dn 8
 number 1008
ephone-dn 9
 number 1009
ephone-dn 10
 number 1010
ephone-dn 11
 number 1011
ephone-dn 12
 number 1012
ephone-dn 13
 number 1013
ephone-dn 14
 number 1014
ephone-dn 15
 number 1015
ephone-dn 16
 number 1016
ephone-dn 17
 number 1017
ephone-dn 18
 number 1018
ephone-dn 19
 number 1019
ephone-dn 20
 number 1020
ephone-dn 21
 number 1021
ephone-dn 22
 number 1022
ephone-dn 23
 number 1023
ephone-dn 24
 number 1024
end
write memory
```

- `ip source-address 10.10.255.1 port 2000`: o endereço e a porta (SCCP) onde os telefones se registram. É a interface do R-BORDA voltada para dentro.
- `auto assign 1 to 30`: cada telefone que se registra recebe automaticamente o próximo ramal livre. Os blocos `ephone N` com o MAC de cada aparelho aparecem sozinhos na configuração. Não é preciso digitá-los.
- `max-ephones` e `max-dn` limitam quantos telefones e ramais o CME aceita; 30 deixa folga sobre os 24 simulados.

## Interface gráfica (não é CLI)

### Servidores: IP fixo

Em cada servidor, Desktop > IP Configuration > Static:

| Servidor | IP | Máscara | Gateway | DNS |
|---|---|---|---|---|
| SRV-DHCP | 10.10.10.66 | 255.255.255.192 | 10.10.10.65 | 10.10.10.67 |
| SRV-DNS | 10.10.10.67 | 255.255.255.192 | 10.10.10.65 | 10.10.10.67 |
| SRV-IOT | 10.10.10.68 | 255.255.255.192 | 10.10.10.65 | 10.10.10.67 |

### SRV-DHCP: escopos

Services > DHCP > **On**. Para cada linha, preencher os campos e clicar em **Add**:

| Pool Name | Default Gateway | DNS Server | Start IP Address | Subnet Mask | TFTP Server |
|---|---|---|---|---|---|
| DADOS | 10.10.0.1 | 10.10.10.67 | 10.10.0.11 | 255.255.254.0 | 0.0.0.0 |
| VOZ | 10.10.2.1 | 10.10.10.67 | 10.10.2.11 | 255.255.254.0 | **10.10.255.1** |
| WIFI-FUNC | 10.10.4.1 | 10.10.10.67 | 10.10.4.11 | 255.255.252.0 | 0.0.0.0 |
| WIFI-PUBLICO | 10.10.8.1 | **203.0.113.10** | 10.10.8.11 | 255.255.254.0 | 0.0.0.0 |
| IOT | 10.10.10.1 | 10.10.10.67 | 10.10.10.11 | 255.255.255.192 | 0.0.0.0 |

- O campo **TFTP Server** do Packet Tracer é a **option 150** do DHCP. No escopo VOZ ele aponta para o R-BORDA, de onde o telefone baixa a configuração antes de se registrar.
- O escopo **WIFI-PUBLICO** entrega o DNS externo (203.0.113.10), para que o visitante não precise de nenhum acesso ao datacenter.
- O "Maximum Number of Users" deve cobrir a sub-rede inteira, porque o escopo é dimensionado para os 300 funcionários, não para os 10% simulados (ver `docs/plano-enderecamento.md`).
- **Start IP:** confira no `.pkt`. O print `etapa-3-dhcp-pc.png` mostra o PC1 com 10.10.0.11, o que bate com a seção 6 do plano de endereçamento (`.1` a `.10` reservados para fixos, pool a partir de `.11`). Um teste anterior registrou um PC com 10.10.0.10, o que indicaria pool a partir de `.10`. A tabela segue o print; se o `.pkt` estiver em `.10`, ajuste o Start IP para `.11` nos cinco escopos.
- O `serverPool` padrão não pode ser apagado. Ele não atrapalha: na VLAN 60 todos os hosts têm IP fixo.

### SRV-DNS: registros

Services > DNS > **On**. Para cada linha, Type **A Record**, e **Add**:

| Name | Address |
|---|---|
| intranet.empresa.local | 10.10.10.67 |
| www.exemplo.com | 203.0.113.10 |

### Telefones e PCs

1. Ligar o cabo da porta do switch (Fa0/x) na porta **Switch** do telefone 7960, e a porta **PC** do telefone no PC.
2. O telefone recebe energia pelo cabo (PoE do 3560-24PS). Não é preciso adaptador de energia.
3. O telefone aprende a VLAN de voz (20) pelo CDP, pede IP no escopo VOZ, recebe a option 150 e se registra no R-BORDA.
4. No PC: Desktop > IP Configuration > **DHCP**. O PC sai sem tag pela VLAN de dados (10) e recebe IP do escopo DADOS.

## Verificação

| Comando / ação | Onde | O que conferir |
|---|---|---|
| `ipconfig` | PC da VLAN 10 | IP em 10.10.0.0/23, máscara 255.255.254.0, gateway 10.10.0.1, DNS 10.10.10.67 |
| `nslookup intranet.empresa.local` | PC da VLAN 10 | Resposta 10.10.10.67 |
| Log do console | R-BORDA | Uma mensagem `%IPPHONE-6-REGISTER` por telefone |
| `show ephone` | R-BORDA | 24 telefones registrados, cada um com seu ramal |
| `show running-config \| section ephone` | R-BORDA | Blocos `ephone 1` a `ephone 24` criados pelo auto-assign |
| Discar um ramal | Telefone (aba GUI) | A chamada completa |

## Problemas comuns

- **PC fica com 169.254.x.x:** o relay não chegou ao servidor. Conferir o `ip helper-address 10.10.10.66` na SVI e se o SRV-DHCP tem gateway (10.10.10.65): sem gateway, ele recebe o pedido e não consegue responder.
- **Telefone apagado:** a porta não está entregando PoE (switch errado, como um 2960-24TT) ou o cabo está na porta PC do telefone em vez da porta Switch.
- **Telefone pega IP mas não registra:** escopo VOZ sem TFTP Server 10.10.255.1, ou `ip source-address` com endereço diferente do da option 150.
- **Telefone pega IP da VLAN 10:** falta `switchport voice vlan 20` na porta do switch.
- **Telefone mostra "Configuring IP" para sempre:** o escopo VOZ está com gateway errado, ou o R-BORDA não tem rota de volta para 10.10.2.0/23 (a rota da etapa 2 cobre).
- **Visitante não resolve nomes:** o DNS dele é o servidor INTERNET (203.0.113.10), que precisa ter o serviço DNS ligado com seus próprios registros. A resolução de nomes pelo visitante não está entre os testes feitos.
