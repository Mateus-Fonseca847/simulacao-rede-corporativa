# configs/

A `running-config` final de cada equipamento, exportada do `rede-final.pkt`. Serve para três coisas: ler a configuração sem abrir o Packet Tracer, comparar com os roteiros de `packet-tracer/`, e refazer a rede se o arquivo de simulação se perder.

| Arquivo | Equipamento | O que concentra |
|---|---|---|
| `core-l3.txt` | `Core-L3` (3650-24PS) | `ip routing`, as 7 SVIs com os gateways e o `ip helper-address`, os 4 trunks, a porta roteada para a borda, as 6 ACLs estendidas e o SSH |
| `roteador-borda.txt` | `R-BORDA` (2811) | As duas interfaces, a rota de volta para 10.10.0.0/16, o NAT/PAT com a `access-list 1`, e a telefonia com os 24 `ephone-dn` |
| `sw-a1.txt` | `SW-A1`, 1º andar (3560-24PS) | Trunk, 9 portas de acesso com voice VLAN, AP, IoT, parking, port-security |
| `sw-a2.txt` | `SW-A2`, 2º andar (3560-24PS) | O mesmo, com 15 portas de acesso |
| `sw-dc.txt` | `SW-DC`, datacenter (2960-24TT) | Trunk restrito às VLANs 60 e 99, portas dos servidores e do PC de administração |
| `sw-t.txt` | `SW-T`, térreo (2960-24TT) | Trunk sem as VLANs de dados e voz, portas dos dois APs e da copa |

## Como foram gerados

Na janela do equipamento, aba **Config** → **Running Config** → **Export...**, salvando com o nome acima. Para atualizar depois de mudar algo, repita a exportação — não edite o `.txt` na mão, senão ele deixa de refletir a simulação.

## O que conferir nestes arquivos

- **Nos trunks:** `switchport trunk native vlan 998` e uma lista `allowed vlan` diferente em cada uplink.
- **Nas portas de acesso dos andares:** `switchport access vlan 10` com `switchport voice vlan 20` na mesma porta.
- **No core:** uma `interface Vlan` por VLAN, cada uma com seu gateway, e o `ip access-group ... in` aplicado na entrada.
- **Na borda:** `ip nat inside` e `ip nat outside` nas interfaces certas, e o `overload` que faz o PAT.

## Senhas

Tudo aqui usa `teste123` como senha de teste do laboratório, e o `service password-encryption` deixa os hashes visíveis no arquivo. **Nunca** coloque neste repositório senha real, sua ou de outro serviço.
