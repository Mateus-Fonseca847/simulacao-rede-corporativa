# Topologia

## 1. Desenho

```
                     [INTERNET]  203.0.113.10 (servidor que simula a internet)
                          │ Fa0
                          │ 203.0.113.0/24
                     Fa0/1│ 203.0.113.1
                    [R-BORDA · 2811]          NAT/PAT, telefonia (CME), rota para 10.10.0.0/16
                     Fa0/0│ 10.10.255.1
                          │ 10.10.255.0/30
                  Gi1/0/24│ 10.10.255.2 (porta roteada)
                 [Core-L3 · 3650-24PS]        SVIs por VLAN, ACLs, relay DHCP, rota padrão
       Gi1/0/1 ┌──────────┬───────┴───┬──────────┐ Gi1/0/4
               │   Gi1/0/2│    Gi1/0/3│          │
          Gi0/1│     Gi0/1│      Gi0/1│     Gi0/1│
   [SW-T · 2960-24TT] [SW-DC · 2960-24TT] [SW-A1 · 3560-24PS] [SW-A2 · 3560-24PS]
    térreo             datacenter (1º)     1º andar            2º andar
    2 APs + copa       3 servidores        9 PCs + 9 fones     15 PCs + 15 fones
                       + PC-ADMIN          AP + copa           AP + copa
```

| Equipamento | Modelo no Packet Tracer | Andar | Função |
|---|---|---|---|
| R-BORDA | 2811 | 1º (junto ao datacenter) | Saída para a internet, NAT/PAT, telefonia (CME) |
| Core-L3 | 3650-24PS | 1º | Roteamento entre VLANs, ACLs, relay DHCP |
| SW-DC | 2960-24TT | 1º | Servidores (VLAN 60) e PC-ADMIN (VLAN 99) |
| SW-A1 | 3560-24PS | 1º | PCs + telefones, AP, copa |
| SW-A2 | 3560-24PS | 2º | PCs + telefones, AP, copa |
| SW-T | 2960-24TT | Térreo | APs e copa (sem estações cabeadas) |
| INTERNET | Server-PT | — | Simula um servidor na internet (203.0.113.10) |

## 2. Por que cada modelo

- **Core: 3650-24PS.** Switch de camada 3: roteia entre as VLANs com `ip routing` e uma SVI por VLAN, e aceita `no switchport` para transformar a Gi1/0/24 em porta roteada até o R-BORDA.
- **Andares 1 e 2: 3560-24PS, e não 2960.** O "P" do nome indica portas PoE. Os telefones IP 7960 do Packet Tracer precisam de energia: num 2960-24TT cada telefone exigiria um adaptador de tomada. O 3560-24PS entrega energia pelo próprio cabo de dados.
- **Térreo e datacenter: 2960-24TT.** Switches de acesso só de camada 2, sem telefones ligados. No Packet Tracer o AP não depende de PoE, então o 2960-24TT basta na simulação. **No projeto real** o switch do térreo precisa ter PoE, porque o argumento de não instalar tomadas no térreo tombado depende de alimentar os APs pelo cabo.
- **Borda: 2811.** Roteador com as duas interfaces Fast Ethernet necessárias e suporte a NAT e a `telephony-service` (CME), que faz a telefonia.

### Observação: `switchport trunk encapsulation dot1q`

O 3560 aceita dois encapsulamentos de trunk, ISL (proprietário da Cisco) e 802.1Q. Por isso ele começa com o encapsulamento em "auto" e recusa `switchport mode trunk` até que se escolha um. Nos uplinks do SW-A1 e do SW-A2 a ordem é:

```
switchport trunk encapsulation dot1q
switchport mode trunk
```

O 2960 e o 3650 só fazem 802.1Q, e nesses o comando de encapsulamento não existe.

## 3. Portas usadas

| Equipamento | Porta | Liga em | Modo / VLAN |
|---|---|---|---|
| Core-L3 | Gi1/0/1 | SW-T Gi0/1 | Trunk: 30, 40, 50, 99 |
| Core-L3 | Gi1/0/2 | SW-DC Gi0/1 | Trunk: 60, 99 |
| Core-L3 | Gi1/0/3 | SW-A1 Gi0/1 | Trunk: 10, 20, 30, 50, 99 |
| Core-L3 | Gi1/0/4 | SW-A2 Gi0/1 | Trunk: 10, 20, 30, 50, 99 |
| Core-L3 | Gi1/0/24 | R-BORDA Fa0/0 | Roteada, 10.10.255.2/30 |
| R-BORDA | Fa0/1 | INTERNET | 203.0.113.1/24 |
| SW-A1 | Fa0/1–9 | Telefone (PC atrás) | Access 10 + voice 20 |
| SW-A1 | Fa0/20 | AP-A1 | Access 30 |
| SW-A1 | Fa0/21 | IoT-Copa-A1 | Access 50 |
| SW-A2 | Fa0/1–15 | Telefone (PC atrás) | Access 10 + voice 20 |
| SW-A2 | Fa0/20 | AP-A2 | Access 30 |
| SW-A2 | Fa0/21 | IoT-Copa-A2 | Access 50 |
| SW-T | Fa0/1 | AP-T-FUNC | Access 30 |
| SW-T | Fa0/2 | AP-T-PUB | Access 40 |
| SW-T | Fa0/3 | IoT-Copa-T | Access 50 |
| SW-DC | Fa0/1 | SRV-DHCP (10.10.10.66) | Access 60 |
| SW-DC | Fa0/2 | SRV-DNS (10.10.10.67) | Access 60 |
| SW-DC | Fa0/3 | SRV-IOT (10.10.10.68) | Access 60 |
| SW-DC | Fa0/24 | PC-ADMIN (10.10.10.130) | Access 99 |

Todos os trunks: 802.1Q, `native vlan 998`, `switchport nonegotiate` e `allowed vlan` restrita às VLANs que existem naquele andar. Todas as outras portas estão na VLAN 999 (PARKING) e em `shutdown`. As portas Gi1/1/1–4 do módulo do core não têm configuração.

Gerência (SVI da VLAN 99): Core-L3 10.10.10.129 (gateway da VLAN), SW-DC .131, SW-A1 .132, SW-A2 .133, SW-T .134.

## 4. Modelo hierárquico com core colapsado

Redes corporativas costumam ter três camadas: **acesso** (onde os usuários se conectam), **distribuição** (agrega o acesso e aplica políticas) e **core** (transporte rápido). Para 300 usuários em 3 andares, três camadas separadas seriam exagero. Usamos o **core colapsado**: um switch L3 faz papel de distribuição e core, e cada andar tem um switch de acesso. Menos equipamentos, menos custo, mesma lógica hierárquica.

O core fica no 1º andar porque o datacenter está lá: é o ponto natural de concentração dos cabos (MDF), e cada andar tem seu armário de acesso (IDF). Termos da aula 07 do curso (cabeamento estruturado).

## 5. Cabeamento

- **Horizontal** (switch até a tomada): enlace permanente de até **90 m**, canal de até **100 m** com os patch cords (TIA/EIA-568). O switch por andar mantém todas as estações dentro do limite.
- **Backbone vertical** (entre andares): fibra óptica, por distância, imunidade a interferência e banda. No Packet Tracer os uplinks são Gigabit de cobre; a fibra é a escolha do projeto real.
- **PoE** nos switches de acesso: APs e telefones recebem energia pelo mesmo cabo de dados (802.3af até 15,4 W; 802.3at, ou PoE+, até 30 W). Evita instalar tomadas, o que importa no térreo tombado.
- **Telefone e PC na mesma porta:** o cabo do switch vai ao telefone, e o PC liga na porta "PC" do telefone. Um ponto de rede atende as duas coisas.

## 6. Térreo tombado

- Sem Ethernet para estações: as 60 posições de trabalho usam **Wi-Fi** (SSID EMPRESA).
- Cabeamento mínimo: uplink do SW-T, os dois APs (AP-T-FUNC e AP-T-PUB) e a copa (IoT-Copa-T).
- **PoE reduz o que precisa chegar ao térreo:** os APs ficam no teto e recebem energia pelo próprio cabo de rede, sem tomada nova em cada ponto. É mais um motivo para o térreo ser quase todo sem fio: cada estação cabeada exigiria um ponto de rede e, em geral, uma tomada.

### Decisão: o uplink do térreo sobe por prumada técnica existente

O SW-T fica no térreo e o core no 1º andar. Por isso esse uplink é a **única passagem vertical de cabo do projeto**, e ela precisa atravessar um pavimento tombado, que não aceita piso elevado nem bandejamento.

**Solução adotada:** aproveitar uma prumada técnica que o prédio já tem (shaft, caixa de escada ou prumada antiga). O cabo corre dentro de **eletroduto flexível**, e a passagem pela laje recebe **selagem corta-fogo**, para não criar um caminho de fogo e fumaça entre os andares.

- **Prumada compartilhada com a elétrica:** usar **fibra óptica** no lugar do par trançado. A fibra é imune à interferência eletromagnética dos cabos de energia. Isso exige porta óptica (ou conversor de mídia) nas duas pontas, no SW-T e no core.
- **Alternativa, se não houver prumada aproveitável:** canaleta aparente numa área de serviço (atrás da escada), **nunca atravessando o espaço de exposição** aberto ao público.

**Justificativa:** em bem tombado valem a **mínima intervenção** e a **reversibilidade**. Não se abre passagem nova na estrutura protegida; usa-se o que o prédio já oferece, e tudo o que é instalado (eletroduto, canaleta, cabo) pode ser removido sem deixar marca.

**Distância não é restrição.** O limite do enlace permanente é de 90 m (100 m de canal, com os patch cords), e subir dois pavimentos fica muito abaixo disso. Quem decide a solução é a passagem física, não o comprimento do cabo.

## 7. Redundância e STP

Dois uplinks por switch de acesso criam loops de camada 2 (tempestade de broadcast). O **STP** resolve elegendo um *root bridge* e bloqueando logicamente as portas redundantes; se um caminho cai, a porta bloqueada assume.

**Decisão adotada:** um uplink por andar e core único. Os switches rodam PVST (padrão), sem caminhos redundantes. Na defesa, reconhecer que o **core único é um ponto único de falha** e explicar como a redundância (segundo core, dois uplinks e Rapid PVST+) o resolveria.

Nas portas de acesso há `spanning-tree portfast` (a porta passa direto a encaminhar, sem esperar o STP) e **BPDU guard**: se chegar um BPDU por ali, sinal de que alguém ligou um switch, a porta é desligada.

Observação: o curso não tem uma aula só de STP; ele aparece nas aulas 13, 14 e 15 e na 27. Conferir com o caderno.

## 8. Wi-Fi no Packet Tracer

No mundo real, um AP anuncia vários SSIDs, cada um mapeado a uma VLAN pelo trunk. No Packet Tracer o AP anuncia um SSID só e não faz trunk. Abordagem adotada: **um AP por SSID**, cada um numa porta *access* da VLAN correspondente.

| AP | Onde | SSID | Segurança | VLAN |
|---|---|---|---|---|
| AP-A2 | SW-A2 Fa0/20 | EMPRESA | WPA2-PSK | 30 |
| AP-A1 | SW-A1 Fa0/20 | EMPRESA | WPA2-PSK | 30 |
| AP-T-FUNC | SW-T Fa0/1 | EMPRESA | WPA2-PSK | 30 |
| AP-T-PUB | SW-T Fa0/2 | VISITANTES | WPA2-PSK | 40 |

Termos da aula 20: os três APs do SSID EMPRESA, ligados ao mesmo sistema de distribuição, formam um **ESS**; cada área de cobertura de um AP é um **BSS**.

As cafeteiras IoT usam adaptador **Fast Ethernet cabeado**, e não Wi-Fi: num AP do Packet Tracer elas cairiam na VLAN 30 dos funcionários, e não na VLAN 50.

## 9. Decisões registradas

| Decisão | Alternativa descartada | Justificativa |
|---|---|---|
| VLAN por função | VLAN por andar | Política única, mobilidade sem reconfigurar, endereço indica o papel |
| Core L3 com SVIs | Router-on-a-stick | Roteamento em hardware, sem gargalo em um único link de trunk, escala melhor |
| Core colapsado | 3 camadas | Porte da empresa não justifica equipamento extra |
| 3560-24PS nos andares 1 e 2 | 2960-24TT | PoE para os telefones; exige `switchport trunk encapsulation dot1q` |
| Térreo só Wi-Fi | Cabear estações no térreo | Prédio tombado, o enunciado não exige Ethernet no térreo |
| Uplink do térreo por prumada existente | Abrir passagem nova; canaleta pelo espaço de exposição | Mínima intervenção e reversibilidade; canaleta em área de serviço só como alternativa |
| Um AP por SSID | Um AP com dois SSIDs | Limite do AP do Packet Tracer |
| IoT cabeada | IoT por Wi-Fi | Mantém a IoT na VLAN 50, isolada |
| `/16` único | Faixas dispersas | Uma rota de sumarização na borda, espaço para crescer |
| Margem de 30% | Dimensionar no limite | Sub-rede a 95% não aguenta contratação nem troca de equipamento |
| VLAN nativa 998 sem hosts | VLAN 1 padrão | Reduz exposição a VLAN hopping por double tagging |
| ACLs na entrada da SVI de origem | ACLs na saída da SVI de destino | ACL estendida perto da origem; descarta antes de rotear; também filtra o tráfego destinado ao próprio core |
| Telefonia no R-BORDA (CME) | Servidor de telefonia no datacenter | O 2811 do Packet Tracer faz `telephony-service`; a option 150 aponta para ele |
| Core único | Dois cores | Simplicidade; limitação assumida e explicada na defesa |

## 10. Pendências

- [x] Print final da topologia em `prints/topologia-final.png`
- [ ] Rótulos nos blocos do `.pkt` (notas do Packet Tracer)
- [x] Definir a passagem do uplink do térreo (prumada técnica existente, seção 6)
