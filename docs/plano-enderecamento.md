# Plano de endereçamento

Toda a rede interna vive em **`10.10.0.0/16`** (faixa privada da RFC 1918). Concentrar tudo em um único `/16` permite que o roteador de borda tenha **uma única rota** para a rede interna (sumarização).

## 1. VLANs

| ID | Nome | Uso | Onde aparece |
|---|---|---|---|
| 10 | DADOS | PCs Ethernet | 1º e 2º andar |
| 20 | VOZ | Telefones IP | 1º e 2º andar |
| 30 | WIFI-FUNC | Wi-Fi corporativo | Todos os andares |
| 40 | WIFI-PUBLICO | Visitantes | Térreo |
| 50 | IOT | Dispositivos das copas | Todos os andares |
| 60 | DATACENTER | Servidores | 1º andar |
| 99 | GERENCIA | Gerência de switches, APs e roteador | Todos |
| 998 | NATIVA | VLAN nativa dos trunks, **sem hosts** | Trunks |
| 999 | PARKING | Portas não usadas, desligadas | Todos |

Por que **por função e não por andar**: se um funcionário muda de andar, não muda de rede; a política de segurança é única para o prédio inteiro; e o endereço indica o papel do dispositivo.

## 2. Fórmula de dimensionamento

Um prefixo `/n` reserva `32 − n` bits para hosts:

**hosts úteis = 2^(32−n) − 2** (menos o endereço de rede e o de broadcast)

| Prefixo | Hosts úteis |
|---|---|
| /22 | 1022 |
| /23 | 510 |
| /24 | 254 |
| /26 | 62 |
| /27 | 30 |
| /30 | 2 |

Critério: **hosts previstos × 1,3** (30% de margem) e o menor bloco que comporta esse valor.

## 3. Dimensionamento por VLAN (demanda total, 300)

| VLAN | Previsão | × 1,3 | Bloco | Úteis | Ocupação |
|---|---|---|---|---|---|
| 10 DADOS | 240 PCs | 312 | /23 | 510 | 47% |
| 20 VOZ | 240 telefones | 312 | /23 | 510 | 47% |
| 30 WIFI-FUNC | 300 × 2 = 600 | 780 | /22 | 1022 | 59% |
| 40 WIFI-PUBLICO | 300 simultâneos | 390 | /23 | 510 | 59% |
| 50 IOT | 3 × 10 = 30 | 39 | /26 | 62 | 48% |
| 60 DATACENTER | ~40 | 52 | /26 | 62 | 65% |
| 99 GERENCIA | ~20 | 26 | /27 | 30 | 67% |
| Link roteador–core | 2 | 2 | /30 | 2 | 100% |

O link ponto a ponto usa `/30` porque tem exatamente duas pontas.

## 4. Mapa de blocos

Sem sobreposição e alinhados à fronteira de cada tamanho (um `/23` só começa em terceiro octeto par; um `/22`, em múltiplo de 4).

| VLAN | Sub-rede | Faixa | Gateway (SVI no core) |
|---|---|---|---|
| 10 | 10.10.0.0/23 | 10.10.0.0 – 10.10.1.255 | 10.10.0.1 |
| 20 | 10.10.2.0/23 | 10.10.2.0 – 10.10.3.255 | 10.10.2.1 |
| 30 | 10.10.4.0/22 | 10.10.4.0 – 10.10.7.255 | 10.10.4.1 |
| 40 | 10.10.8.0/23 | 10.10.8.0 – 10.10.9.255 | 10.10.8.1 |
| 50 | 10.10.10.0/26 | 10.10.10.0 – 10.10.10.63 | 10.10.10.1 |
| 60 | 10.10.10.64/26 | 10.10.10.64 – 10.10.10.127 | 10.10.10.65 |
| 99 | 10.10.10.128/27 | 10.10.10.128 – 10.10.10.159 | 10.10.10.129 |
| Link roteador–core | 10.10.255.0/30 | .1 (roteador) e .2 (core) | — |

Espaço livre para crescer: de `10.10.10.160` até `10.10.254.255`.

## 5. Endereços fixos sugeridos

Proposta inicial. Confirmar e ajustar durante a construção.

| Equipamento | VLAN | Endereço |
|---|---|---|
| Roteador de borda (lado core) | link | 10.10.255.1 |
| Core L3 (lado roteador) | link | 10.10.255.2 |
| Core L3 (gerência) | 99 | 10.10.10.129 (é o gateway: SVI da VLAN 99) |
| SW-DC | 99 | 10.10.10.131 |
| SW-A1 | 99 | 10.10.10.132 |
| SW-A2 | 99 | 10.10.10.133 |
| SW-T | 99 | 10.10.10.134 |
| Servidor DHCP | 60 | 10.10.10.66 |
| Servidor DNS | 60 | 10.10.10.67 |
| Controlador IoT (a definir) | 60 | 10.10.10.68 |

## 6. DHCP

Um escopo por VLAN, dimensionado para a sub-rede inteira (os 300 reais), mesmo que só ~30 hosts peçam endereço na simulação. O servidor fica no datacenter e atende as demais VLANs por **relay** (`ip helper-address` na interface virtual de cada VLAN, no core), porque o broadcast do DHCP não atravessa VLANs.

| VLAN | Escopo | Excluídos (fixos) | Gateway | DNS |
|---|---|---|---|---|
| 10 | 10.10.0.0/23 | 10.10.0.1 – 10.10.0.10 | 10.10.0.1 | 10.10.10.67 |
| 20 | 10.10.2.0/23 | 10.10.2.1 – 10.10.2.10 | 10.10.2.1 | 10.10.10.67 |
| 30 | 10.10.4.0/22 | 10.10.4.1 – 10.10.4.10 | 10.10.4.1 | 10.10.10.67 |
| 40 | 10.10.8.0/23 | 10.10.8.1 – 10.10.8.10 | 10.10.8.1 | DNS externo (sem acesso ao datacenter) |
| 50 | 10.10.10.0/26 | 10.10.10.1 – 10.10.10.10 | 10.10.10.1 | 10.10.10.67 |

Voz (VLAN 20): além de IP, máscara e gateway, o telefone precisa receber **option 150** apontando para o equipamento que faz a telefonia (o roteador, no Packet Tracer). Endereço a confirmar na etapa 3.

## 7. Capacidade física para 300

Portas cabeadas = estações + dispositivos IoT + uplinks, com 20% de reserva. APs e eventuais portas extras se somam a isso (quantidade de APs: a definir na etapa 4).

| Andar | Conta | Total com reserva | Switches de 48 portas |
|---|---|---|---|
| 1º | 90 + 10 + 2 = 102 | ~123 | 3 |
| 2º | 150 + 10 + 2 = 162 | ~195 | 5 |
| Térreo | 10 + 2 = 12 (sem estações cabeadas) | ~15 | 1 (de 24 portas) |
| Datacenter | ~40 | ~48 | 1 |

Telefone e PC compartilham a mesma porta (voice VLAN), por isso a voz não consome portas adicionais. Estes números dependem das premissas de [`requisitos.md`](requisitos.md).

## 8. O que se simula (10%) e o que se projeta (100%)

| Item | Projetado (300) | Simulado |
|---|---|---|
| Estações cabeadas | 240 | 24 (9 no 1º, 15 no 2º) |
| Wi-Fi funcionários | 600 dispositivos | 6 (térreo) + alguns notebooks |
| Wi-Fi público | 300 dispositivos | 2 a 3 |
| IoT | 30 | 3 a 6 (1 a 2 por copa) |
| Escopos DHCP | Sub-rede inteira de cada VLAN | Idem (configurados completos) |
| Switches | Quantidade da seção 7 | 1 por andar + datacenter |

A contagem exata da simulação (30 dispositivos de usuário) será fechada na construção.
