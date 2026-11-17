# prints/

Evidências do projeto: a topologia final, os testes de [`docs/testes.md`](../docs/testes.md) e as duas visualizações interativas. Cada linha de `testes.md` aponta para o print que a comprova.

## Topologia e visualizações

| Arquivo | O que mostra |
|---|---|
| `topologia-final.png` | A topologia inteira no Packet Tracer |
| `visualizacao-3d.png` | O mapa 3D de [`visualizacao-3d/`](../visualizacao-3d/index.html): o prédio com os 3 andares, o painel do equipamento selecionado e os filtros por VLAN |
| `visualizacao-fluxo.png` | O fluxograma de [`visualizacao-fluxo/`](../visualizacao-fluxo/index.html): o pacote parado na decisão da ACL, com o balão explicativo e o quadro naquele ponto |

## Testes

| Arquivo | O que mostra | Testes |
|---|---|---|
| `etapa-1-trunks.png` | Core-L3, `show interfaces trunk`: Gi1/0/1 a Gi1/0/4 em 802.1q, trunking, nativa 998, com a lista de VLANs de cada uplink (30,40,50,99 / 60,99 / 10,20,30,50,99 / 10,20,30,50,99) | 1.2 |
| `etapa-1-vlans.png` | SW-A2, `show vlan brief`: as 9 VLANs; Fa0/1-15 em DADOS e VOZ ao mesmo tempo (voice VLAN), Fa0/20 em WIFI-FUNC, Fa0/21 em IOT, Fa0/16-19, Fa0/22-24 e Gi0/2 em PARKING | 1.1, 1.5 |
| `etapa-1-portas-status.png` | SW-A2, `show interfaces status`: descrições PC-FONE, PARKING, AP-FUNC e IOT-COPA; portas de parking `disabled` na VLAN 999 | 1.5 |
| `etapa-2-rotas.png` | Core-L3, `show ip route`: "Gateway of last resort is 10.10.255.1" e as rotas conectadas das VLANs 10 a 60 | 2.3 |
| `etapa-2-rota-padrao.png` | Core-L3, continuação do `show ip route`: `S* 0.0.0.0/0 [1/0] via 10.10.255.1` e a rota conectada 10.10.255.0/30 (Gi1/0/24) | 2.3 |
| `etapa-2-svis.png` | Core-L3, `show ip interface brief`: SVIs 10, 20, 30, 40, 50, 60 e 99 up/up com seus IPs, Gi1/0/24 com 10.10.255.2, Gi1/0/5 a Gi1/0/23 em administratively down | 2.6, 1.5 |
| `etapa-3-dhcp-pc.png` | PC1, `ipconfig /all`: 10.10.0.11, 255.255.254.0, gateway 10.10.0.1, servidor DHCP 10.10.10.66, DNS 10.10.10.67, recebidos por DHCP com relay | 3.1, 3.6 |
| `etapa-3-dns.png` | PC1, `nslookup intranet.empresa.local`: resposta 10.10.10.67 | 3.3 |
| `etapa-3-telefones.png` | R-BORDA, mensagens `%IPPHONE-6-REGISTER` dos telefones registrados com endereços 10.10.2.x | 3.4, 3.6 |
| `etapa-3-chamada-voip.png` | Chamada entre dois ramais da telefonia (1001 e 1004) | 3.5 |
| `etapa-4-wifi-func.png` | Smartphone2 (funcionário): fim do ping a 10.10.10.67 (4/4) e `ipconfig` com 10.10.4.17, 255.255.252.0, gateway 10.10.4.1 | 4.1 |
| `etapa-4-wifi-func-acesso.png` | Smartphone2: `ping 10.10.10.67` completo, 4 de 4 respostas | 4.1 |
| `etapa-4-wifi-publico.png` | Smartphone8 (visitante): `ipconfig` com 10.10.8.12, 255.255.254.0, gateway 10.10.8.1 | 4.2 |
| `etapa-4-iot-servidor.png` | Navegador no SRV-IOT (10.10.10.68): IoT-Copa-A2, IoT-Copa-A1 e IoT-Copa-T registrados, com indicador verde | 4.3 |
| `etapa-5-acl-contadores.png` | Core-L3, `show access-lists`, primeira página: ACL-FUNCIONARIOS, ACL-VOZ, ACL-PUBLICO e o começo da ACL-IOT, com contadores | 5.8 |
| `etapa-5-acl-todas.png` | Core-L3, `show access-lists`: as seis ACLs estendidas e a access-list 10 da gerência, com contadores | 5.8 |
| `etapa-5-acl-dados-gerencia.png` | PC1 (VLAN 10): `ping 203.0.113.10` responde; `ping 10.10.10.129` 100% de perda, "Destination host unreachable" vindo do gateway | 2.5, 5.9 |
| `etapa-5-acl-bloqueio-publico.png` | Smartphone8 (visitante): `ping 203.0.113.10` 4/4; `ping 10.10.10.67` 100% de perda | 5.1, 5.2 |
| `etapa-5-nat.png` | R-BORDA, `show ip nat translations`: traduções icmp de 10.10.0.11 para 203.0.113.1, uma por porta | 5.7 |
| `etapa-5-ssh-admin.png` | PC-ADMIN (10.10.10.130): `ssh -l admin 10.10.10.133` entra no SW-A2; `show ssh` mostra a sessão ativa com aes128-cbc | 5.11 |
| `etapa-5-ssh-bloqueado.png` | PC1 (VLAN 10): o mesmo `ssh -l admin 10.10.10.133` termina em "Connection timed out" | 5.12 |
| `etapa-5-port-security.png` | SW-A2, `show port-security`: Fa0/1 a Fa0/15 com máximo 3 e 2 MACs aprendidos, Fa0/21 com máximo 1, todas em Restrict, 0 violações | 5.14 |

O par mais forte para a defesa é `etapa-5-ssh-admin.png` com `etapa-5-ssh-bloqueado.png`: o mesmo comando, duas origens, resultados opostos.

## Dicas

- Nome no padrão `etapa-N-descricao.png`.
- Mostre no print o comando e a saída completa, não só o resultado.
- Ao acrescentar um print, registre o nome dele na coluna "Print" de `docs/testes.md`.
- Prefira PNG e evite imagens muito pesadas.
