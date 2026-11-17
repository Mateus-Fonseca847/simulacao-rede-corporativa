# packet-tracer/

Os arquivos de simulação e os roteiros de comandos que os reconstroem.

## Arquivos `.pkt`

Um por etapa concluída, como ponto de volta. Se uma etapa quebrar a rede, abra a anterior em vez de desfazer na mão.

| Arquivo | Etapa | O que já funciona nele |
|---|---|---|
| `rede-v1-vlans.pkt` | 1 | Os 5 switches, as 9 VLANs, trunks com nativa 998, gerência pela VLAN 99. Ainda **sem** roteamento entre VLANs |
| `rede-v2-roteamento.pkt` | 2 | SVIs no core, roteador de borda, rota padrão. Já se fala entre VLANs e com a internet simulada |
| `rede-v3-servicos.pkt` | 3 | DHCP com relay, DNS interno, telefonia com os 24 ramais, as estações com PC e telefone |
| `rede-v4-wifi-iot.pkt` | 4 | Os 4 APs, os smartphones de funcionários e visitantes, as 3 copas IoT no controlador |
| `rede-v5-seguranca.pkt` | 5 | NAT/PAT, as 6 ACLs no core, SSH com acesso restrito à gerência, port-security |
| `rede-final.pkt` | — | **Versão entregue.** Igual à v5, com os rótulos de andar na área de trabalho |

## Roteiros de comandos

| Arquivo | Conteúdo |
|---|---|
| `etapa-1-comandos.md` | VLANs, trunks, portas de acesso, parking, IP de gerência |
| `etapa-2-comandos.md` | `ip routing`, SVIs com `ip helper-address`, porta roteada, rota padrão, roteador de borda |
| `etapa-3-comandos.md` | Escopos DHCP, registros DNS, `telephony-service` e os `ephone-dn` |
| `etapa-4-comandos.md` | SSID e WPA2 nos APs, associação dos clientes, adaptador e registro dos dispositivos IoT |
| `etapa-5-comandos.md` | NAT/PAT, as 6 ACLs estendidas, SSH com `access-class`, port-security e BPDU guard |

Cada roteiro traz o objetivo da etapa, os blocos prontos para colar na CLI, a verificação e os problemas comuns. As partes que não são CLI (escopos DHCP, SSID, IoT) estão descritas pela interface gráfica.

## Para abrir

Cisco Packet Tracer 8.2 ou superior. Senha dos equipamentos: `teste123` no modo enable, usuário `admin` com a mesma senha no SSH.

## Detalhes da montagem que valem lembrar

- O **core 3650-24PS vem sem fonte**: é preciso arrastar o módulo `AC-POWER-SUPPLY` para um slot vazio na aba *Physical*, senão o equipamento não liga.
- **SW-A1 e SW-A2 são 3560-24PS**, e não 2960, porque os telefones IP precisam de PoE. Como são switches L3, exigem `switchport trunk encapsulation dot1q` antes de `switchport mode trunk`.
- Os uplinks usam **cabo crossover** entre switches; entre PC e switch, cabo direto.
- `terminal length 0` não existe nessa versão do IOS simulado: a saída longa aparece paginada com `--More--`, avance com a barra de espaço.
