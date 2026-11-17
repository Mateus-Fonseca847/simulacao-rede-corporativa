# Plano de endereçamento e VLANs (modelo a preencher)

> Modelo de tabela para organizar antes de abrir o Packet Tracer. Preencha com os valores que você decidir (os deste arquivo são só exemplo de formato).

## VLANs propostas

| VLAN ID | Nome | Andar(es) | Função | Observação |
|---|---|---|---|---|
| 10 | DADOS-1AND | 1º | Ethernet dos funcionários | |
| 11 | VOZ-1AND | 1º | VoIP (QoS/prioridade, 802.1p) | |
| 20 | DADOS-2AND | 2º | Ethernet dos funcionários | |
| 21 | VOZ-2AND | 2º | VoIP | |
| 30 | WIFI-FUNC | todos | Wi-Fi dos funcionários | um SSID, VLAN comum ou por andar |
| 40 | WIFI-PUBLICO | térreo | Wi-Fi do público, isolado | sem acesso à rede interna |
| 50 | IOT-COPAS | todos | IoT das copas | isolar de dados/voz por segurança |
| 60 | DATACENTER | 1º | Servidores | acesso restrito |
| 99 | GERENCIA | todos | Gerência dos switches/AP | management VLAN |

## Sub-redes (exemplo de esquema — ajuste ao que você decidir)

| VLAN | Sub-rede | Gateway | Faixa DHCP | Capacidade (hosts úteis) |
|---|---|---|---|---|
| 10 | 10.10.0.0/24 | 10.10.0.1 | .10–.250 | 254 — cobre os funcionários reais do 1º andar |
| 20 | 10.20.0.0/24 | 10.20.0.1 | .10–.250 | 254 |
| 30 | 10.30.0.0/23 | 10.30.0.1 | .10–.500 | 510 — cobre Wi-Fi de todos os andares |
| 40 | 10.40.0.0/25 | 10.40.0.1 | .10–.120 | 126 — só o público do térreo |
| 50 | 10.50.0.0/26 | 10.50.0.1 | .10–.50 | 62 — poucas copas |
| 60 | 10.60.0.0/27 | — (estático) | — | servidores fixos |

> Dica: dimensione as sub-redes pela **capacidade real de 300 funcionários**, não pelos ~30 que vão aparecer na simulação. Isso é justamente o que a Obs. do enunciado pede para não esquecer.

## Checklist de configuração no Packet Tracer

- [ ] Criar as VLANs nos switches de cada andar (`vlan <id>` / `name <nome>`).
- [ ] Configurar portas de acesso (funcionários, telefones IP, IoT) e portas trunk (uplinks entre switches e para o roteador/L3).
- [ ] Inter-VLAN routing: roteador-on-a-stick ou switch L3 (camada 3) — decidir e documentar o motivo da escolha.
- [ ] DHCP por VLAN (pool por sub-rede, ou servidor DHCP central com `ip helper-address`).
- [ ] Wi-Fi: Access Points com SSIDs separados para funcionários × público, mapeados nas VLANs corretas.
- [ ] ACLs ou isolamento para impedir que o Wi-Fi público e a VLAN de IoT alcancem dados/voz/datacenter.
- [ ] 10% da carga: reduzir o número de PCs/telefones/APs simulados, mantendo pelo menos 1 de cada tipo de serviço por andar para demonstrar que funciona.
