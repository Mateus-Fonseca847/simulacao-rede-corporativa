# Requisitos

Fonte: enunciado "Simulação packet tracer (em dupla ou individual)", entrega em 17/11/2026, 23:59, com defesa presencial a combinar.

## 1. O que o enunciado pede

Usar o Packet Tracer para **projetar e simular** a rede corporativa de uma empresa com **300 funcionários**, em um **prédio histórico de 3 andares (térreo, 1º e 2º)**. Por ser tombado, **o térreo não pode receber alterações como piso elevado ou bandejamento** para passagem de cabos.

Cenário por andar:

| Andar | O que existe |
|---|---|
| Térreo | Espaço de exposições aberto ao público e 60 posições de trabalho |
| 1º andar | Datacenter, e pode haver também posições de trabalho |
| 2º andar | Apenas posições de trabalho |
| Todos | Copas automatizadas com IoT |

A rede deve incluir:

1. Acesso Ethernet e VoIP para todos os funcionários do 1º e do 2º andar
2. Wi-Fi para os funcionários em todos os andares
3. Wi-Fi separado para o público no térreo
4. IoT, ao menos para as copas

Observação de desempenho: simular apenas **10% da carga de dispositivos de usuários**, **mantendo a rede com capacidade para a demanda total**.

## 2. Três tipos de coisa no enunciado

| Tipo | O que é | Exemplos aqui |
|---|---|---|
| Requisito funcional | O que a rede precisa fazer | Ethernet + VoIP nos andares 1 e 2, Wi-Fi, IoT |
| Restrição | O que limita como fazer | Prédio tombado, sem piso elevado nem bandejamento no térreo |
| Dimensionamento | Quanto a rede precisa aguentar | 300 funcionários (simulando 10%) |

## 3. Requisito → solução

| Requisito do enunciado | Solução de rede | VLAN |
|---|---|---|
| Ethernet nos andares 1 e 2 | Switches de acesso por andar | 10 DADOS |
| VoIP nos andares 1 e 2 | Telefone IP com voice VLAN (telefone e PC na mesma porta) | 20 VOZ |
| Wi-Fi para funcionários, todos os andares | AP com SSID corporativo | 30 WIFI-FUNC |
| Wi-Fi do público, só no térreo | AP separado, SSID próprio, isolado | 40 WIFI-PUBLICO |
| IoT nas copas (3 andares) | Dispositivos IoT em VLAN própria | 50 IOT |
| Datacenter no 1º andar | Servidores em sub-rede própria, ligados ao core | 60 DATACENTER |
| Gerência dos equipamentos | Acesso de administração separado do tráfego de usuários | 99 GERENCIA |

### Situação de cada requisito

Os números de teste e os prints são os de [`testes.md`](testes.md); os prints ficam em `prints/`.

| Requisito do enunciado | Situação | Como foi construído | Onde é demonstrado |
|---|---|---|---|
| Ethernet nos andares 1 e 2 | ✅ Atendido | SW-A1 (9 portas PC + telefone) e SW-A2 (15), VLAN 10; PCs em DHCP | Testes 1.1 (`etapa-1-vlans.png`), 1.3 e 3.1 (`etapa-3-dhcp-pc.png`) |
| VoIP nos andares 1 e 2 | ✅ Atendido (registro); chamada a testar | 24 telefones 7960 com voice VLAN 20, PoE do 3560-24PS, telefonia no R-BORDA, ramais 1001 a 1024 | Testes 3.4 (`etapa-3-telefones.png`) e 5.14 (`etapa-5-port-security.png`, PC + telefone na mesma porta); a chamada entre ramais (3.5) ainda não foi testada |
| Wi-Fi para funcionários, todos os andares | ✅ Atendido | AP-T-FUNC, AP-A1 e AP-A2 com SSID EMPRESA (WPA2-PSK), VLAN 30 | Teste 4.1 (`etapa-4-wifi-func.png`, `etapa-4-wifi-func-acesso.png`); o teste andar por andar (4.4) ainda não foi feito |
| Wi-Fi do público, só no térreo, separado | ✅ Atendido | AP-T-PUB com SSID VISITANTES (WPA2-PSK), VLAN 40, só no SW-T; ACL-PUBLICO libera só a internet | Testes 4.2 (`etapa-4-wifi-publico.png`), 5.1 e 5.2 (`etapa-5-acl-bloqueio-publico.png`) |
| IoT nas copas (3 andares) | ✅ Atendido | IoT-Copa-T, IoT-Copa-A1 e IoT-Copa-A2, cabeadas na VLAN 50, registradas no SRV-IOT | Teste 4.3 (`etapa-4-iot-servidor.png`) |
| Datacenter no 1º andar | ✅ Atendido | SW-DC com SRV-DHCP, SRV-DNS e SRV-IOT, VLAN 60 | Testes 3.1 (`etapa-3-dhcp-pc.png`) e 3.3 (`etapa-3-dns.png`) |
| Térreo sem piso elevado nem bandejamento | ✅ Atendido | Nenhuma estação cabeada no térreo; só uplink (por prumada técnica existente), 2 APs e a copa no SW-T | Desenho em [`topologia.md`](topologia.md) e `topologia-final.png` |
| Simular 10% mantendo capacidade para 300 | ✅ Atendido | Escopos DHCP e sub-redes para a sub-rede inteira; ver seção 5 | [`plano-enderecamento.md`](plano-enderecamento.md) |
| Gerência dos equipamentos (decisão do projeto) | ✅ Atendido | VLAN 99, SSH v2 com `access-class` limitado à VLAN 99 | Testes 1.6, 5.11 (`etapa-5-ssh-admin.png`) e 5.12 (`etapa-5-ssh-bloqueado.png`) |

O enunciado não exige Ethernet nem VoIP no térreo. Por isso os 60 funcionários dali usam **Wi-Fi**, e o cabeamento do térreo se limita ao mínimo (uplink do switch, alimentação PoE dos APs e dispositivos da copa).

## 4. Restrição do prédio tombado

- Nenhuma intervenção estrutural no térreo.
- Por onde passa o uplink do térreo sem obra estrutural: **resolvido**. O cabo sobe por prumada técnica já existente no prédio, em eletroduto flexível e com selagem corta-fogo na laje; a canaleta aparente em área de serviço fica como alternativa. Detalhes na seção 6 de [`topologia.md`](topologia.md).
- PoE nos APs evita instalar tomadas de energia ao lado de cada ponto.

## 5. Dimensionar para 300, simular 10%

O Packet Tracer perde desempenho com muitos dispositivos. Por isso:

- **Projetado (papel e configuração):** VLANs, sub-redes, escopos DHCP, quantidade de portas e uplinks, tudo para os 300.
- **Simulado (Packet Tracer):** 24 PCs (cada um atrás de um telefone IP), 11 smartphones e 3 dispositivos IoT, que provam o funcionamento.

Na prática, o escopo DHCP é configurado para centenas de hosts, mas só 30 se conectam. A conta de capacidade está em [`plano-enderecamento.md`](plano-enderecamento.md).

## 6. Premissas (confirmar ou ajustar)

O enunciado não traz estes números. São suposições de trabalho:

| Premissa | Valor assumido |
|---|---|
| Funcionários no térreo | 60 (dado do enunciado) |
| Funcionários no 1º andar | 90 |
| Funcionários no 2º andar | 150 |
| Estações cabeadas (1º + 2º) | 240 |
| Dispositivos Wi-Fi por funcionário | 2 (notebook + celular) |
| Público simultâneo no Wi-Fi do térreo | 300 dispositivos |
| Copas | 3 (uma por andar), ~10 dispositivos IoT cada |
| Servidores e equipamentos no datacenter | ~40 |
| Margem de crescimento | 30% |
| Simulação (10%) | Plano: 6 no térreo, 9 no 1º andar, 15 no 2º andar (total 30). Construído: 24 PCs com 24 telefones (9 no 1º, 15 no 2º), 11 smartphones (8 de funcionários, 3 de visitantes) e 3 dispositivos IoT |

Se o enunciado ou o professor fixar outros números, só os blocos de endereço e a quantidade de portas mudam. O método é o mesmo.

## 7. Pendências de confirmação

- [ ] Formato da entrega: só o `.pkt` ou também relatório/documentação?
- [ ] Distribuição real de funcionários entre os andares
- [ ] Se há exigência de equipamentos ou modelos específicos do Packet Tracer
- [ ] Data da defesa
