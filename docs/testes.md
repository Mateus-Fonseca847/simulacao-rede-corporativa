# Testes

Cada etapa só é dada como pronta depois que seus testes passam. Marque o resultado, anexe o print em `prints/` e salve um `.pkt` versionado em `packet-tracer/`.

Legenda: ⬜ a fazer · ✅ passou · ❌ falhou

A coluna "Print" traz o arquivo de `prints/` que comprova o teste (a lista completa, com o que cada um mostra, está em [`prints/README.md`](../prints/README.md)). Um print pode servir a mais de um teste. Os poucos ✅ sem print foram verificados no Packet Tracer, mas a tela não foi capturada. Os comandos de cada teste estão na seção "Verificação" de `packet-tracer/etapa-N-comandos.md`.

**Situação:** 24 testes passaram, 15 a fazer, nenhum falhou.

## Etapa 1: switches, VLANs e trunks

| # | Teste | Esperado | Resultado | Print |
|---|---|---|---|---|
| 1.1 | `show vlan brief` no SW-A2 | VLANs 10, 20, 30, 40, 50, 60, 99, 998 e 999 existem, com as portas nas VLANs certas | ✅ as 9 VLANs; Fa0/1-15 em DADOS e VOZ (voice VLAN), Fa0/20 em WIFI-FUNC, Fa0/21 em IOT | `etapa-1-vlans.png` |
| 1.2 | `show interfaces trunk` no core | 4 trunks ativos, 802.1q, nativa 998, só as VLANs necessárias em cada um | ✅ Gi1/0/1 a Gi1/0/4 em 802.1q, trunking, nativa 998; allowed 30,40,50,99 / 60,99 / 10,20,30,50,99 / 10,20,30,50,99 | `etapa-1-trunks.png` |
| 1.3 | Ping entre dois PCs da VLAN 10 (andares diferentes) | Responde | ✅ 4/4 | |
| 1.4 | Ping entre PCs de VLANs diferentes (ainda sem roteamento) | Falha | ⬜ | |
| 1.5 | Portas não usadas (SW-A2 e core) | `disabled`/`administratively down`, na VLAN 999 | ✅ SW-A2: Fa0/16-19, Fa0/22-24 e Gi0/2 em PARKING, `disabled` na VLAN 999; core: Gi1/0/5 a Gi1/0/23 em administratively down | `etapa-1-portas-status.png`, `etapa-1-vlans.png`, `etapa-2-svis.png` |
| 1.6 | Ping do core à gerência dos switches (10.10.10.131 a .134) | Os quatro respondem | ✅ os quatro respondem | |
| 1.7 | `show interfaces trunk` nos switches de acesso | Mesma nativa (998) e mesma lista de VLANs do lado do core | ⬜ | |
| 1.8 | `show vlan brief` no Core-L3, SW-A1, SW-T e SW-DC | As mesmas 9 VLANs em cada um | ⬜ | |

## Etapa 2: roteamento

| # | Teste | Esperado | Resultado | Print |
|---|---|---|---|---|
| 2.1 | Ping do PC ao seu gateway (SVI) | Responde | ⬜ | |
| 2.2 | Ping entre VLANs ainda sem ACL | Responde | ⬜ | |
| 2.3 | `show ip route` no core | Rotas conectadas e rota padrão para o roteador | ✅ "Gateway of last resort is 10.10.255.1"; conectadas das VLANs 10, 20, 30, 40, 50 e 60 e de 10.10.255.0/30 (Gi1/0/24); `S* 0.0.0.0/0 [1/0] via 10.10.255.1` | `etapa-2-rotas.png`, `etapa-2-rota-padrao.png` |
| 2.4 | Ping do core ao roteador (10.10.255.1) | Responde | ⬜ | |
| 2.5 | Ping de um PC da VLAN 10 a 203.0.113.10 (internet simulada) | Responde | ✅ PC1 responde (rede final, já com NAT) | `etapa-5-acl-dados-gerencia.png` |
| 2.6 | `show ip interface brief` no core | SVIs `up/up` com os gateways; Gi1/0/24 com 10.10.255.2 | ✅ SVIs 10, 20, 30, 40, 50, 60 e 99 up/up com seus IPs; Gi1/0/24 com 10.10.255.2 | `etapa-2-svis.png` |

## Etapa 3: serviços

| # | Teste | Esperado | Resultado | Print |
|---|---|---|---|---|
| 3.1 | PC da VLAN 10 pede IP por DHCP (via relay) | Recebe IP da faixa, gateway e DNS corretos | ✅ PC1: 10.10.0.11/255.255.254.0, gw 10.10.0.1, servidor DHCP 10.10.10.66, DNS 10.10.10.67 | `etapa-3-dhcp-pc.png` |
| 3.2 | Idem para VLANs 20, 30, 40 e 50 | Cada uma recebe da sua faixa | ⬜ (20 aparece em 3.4, 30 em 4.1 e 40 em 4.2; falta a 50) | |
| 3.3 | `nslookup intranet.empresa.local` no PC | Responde 10.10.10.67 | ✅ responde 10.10.10.67 | `etapa-3-dns.png` |
| 3.4 | Telefones se registram na telefonia | Ramais atribuídos | ✅ os 24 telefones registrados (`%IPPHONE-6-REGISTER` no R-BORDA), com endereços 10.10.2.x | `etapa-3-telefones.png` |
| 3.5 | Chamada entre dois telefones | Completa | ✅ | (`etapa-3-chamada-voip.png`) a capturar |
| 3.6 | PC atrás do telefone | Recebe IP da VLAN 10, telefone da VLAN 20 | ✅ PC1 em 10.10.0.11; telefones em 10.10.2.x | `etapa-3-dhcp-pc.png`, `etapa-3-telefones.png` |

## Etapa 4: Wi-Fi e IoT

| # | Teste | Esperado | Resultado | Print |
|---|---|---|---|---|
| 4.1 | Smartphone de funcionário associa ao SSID EMPRESA | IP da VLAN 30, alcança o datacenter | ✅ Smartphone2: 10.10.4.17/255.255.252.0, gw 10.10.4.1; ping a 10.10.10.67 4/4 | `etapa-4-wifi-func.png`, `etapa-4-wifi-func-acesso.png` |
| 4.2 | Smartphone de visitante associa ao SSID VISITANTES | IP da VLAN 40 | ✅ Smartphone8: 10.10.8.12/255.255.254.0, gw 10.10.8.1 | `etapa-4-wifi-publico.png` |
| 4.3 | Dispositivos IoT das copas registrados no controlador | IP da VLAN 50, registrados no SRV-IOT (10.10.10.68) | ✅ IoT-Copa-A2, IoT-Copa-A1 e IoT-Copa-T registrados, os três com indicador verde | `etapa-4-iot-servidor.png` |
| 4.4 | Wi-Fi corporativo em cada andar (AP-T-FUNC, AP-A1, AP-A2) | Um smartphone associa em cada AP | ⬜ | |
| 4.5 | Roaming: smartphone se move entre a área de dois APs | Reassocia e mantém IP da VLAN 30 | ⬜ | |

## Etapa 5: segurança

| # | Teste | Esperado | Resultado | Print |
|---|---|---|---|---|
| 5.1 | Wi-Fi público (Smartphone8) → 10.10.10.67 (datacenter) | Bloqueado | ✅ 100% de perda | `etapa-5-acl-bloqueio-publico.png` |
| 5.2 | Wi-Fi público (Smartphone8) → 203.0.113.10 (internet) | Permitido | ✅ 4/4 | `etapa-5-acl-bloqueio-publico.png` |
| 5.3 | IoT → PC da VLAN 10 | Bloqueado | ⬜ | |
| 5.4 | IoT → controlador (10.10.10.68), na rede final | Permitido | ⬜ | |
| 5.5 | PC da VLAN 10 → servidor do datacenter | Permitido | ⬜ | |
| 5.6 | PC da VLAN 10 → Wi-Fi público | Bloqueado | ⬜ | |
| 5.7 | `show ip nat translations` no R-BORDA | Mostra tradução de endereços internos | ✅ traduções icmp de 10.10.0.11 para 203.0.113.1, uma por porta | `etapa-5-nat.png` |
| 5.8 | `show access-lists` no core | Contadores de match nas regras testadas | ✅ as seis ACLs estendidas e a access-list 10, todas com contadores | `etapa-5-acl-contadores.png`, `etapa-5-acl-todas.png` |
| 5.9 | PC da VLAN 10 → 10.10.10.129 (gerência) | Bloqueado | ✅ 100% de perda, "Destination host unreachable" vindo do gateway | `etapa-5-acl-dados-gerencia.png` |
| 5.10 | PC da VLAN 10 → 10.10.2.14 (telefone, VLAN de voz) | Bloqueado | ✅ 100% de perda | |
| 5.11 | PC-ADMIN (10.10.10.130) → `ssh -l admin 10.10.10.133` | Entra | ✅ entra no SW-A2; `show ssh` mostra a sessão ativa (aes128-cbc) | `etapa-5-ssh-admin.png` |
| 5.12 | PC da VLAN 10 → `ssh -l admin 10.10.10.133` | Bloqueado | ✅ "Connection timed out" | `etapa-5-ssh-bloqueado.png` |
| 5.13 | Violação proposital de port-security (MAC a mais numa porta) | Quadros descartados, contador de violação sobe, porta continua `up` | ⬜ | |
| 5.14 | `show port-security` no SW-A2 | Limites configurados, modo Restrict | ✅ Fa0/1 a Fa0/15 com máximo 3 e 2 MACs aprendidos (PC + telefone); Fa0/21 com máximo 1; todas em Restrict, 0 violações | `etapa-5-port-security.png` |

Sobre o 5.12: o pedido de SSH do PC1 para em duas barreiras. A primeira é a ACL-FUNCIONARIOS, na entrada da SVI 10 do core, que nega qualquer destino em 10.10.10.128/27; por isso o pacote nem chega ao SW-A2. A segunda é o `access-class 10 in` nas linhas vty do SW-A2, que recusaria a sessão se a primeira falhasse.

## Teste geral (antes da entrega)

- [ ] Todos os testes acima passaram, ou as falhas estão registradas e explicadas
- [x] O `.pkt` final abre limpo e a simulação está em modo de tempo real
- [x] `configs/` tem o `show running-config` de cada equipamento
- [x] `prints/` tem a topologia e as evidências principais
- [x] A conta de capacidade para 300 está em `plano-enderecamento.md`
- [x] README atualizado
