# Requisitos — checklist detalhado

Fonte: enunciado "Simulação packet tracer (em dupla ou individual)".

## 1. Infraestrutura física

- [ ] Prédio de 3 andares: térreo, 1º e 2º.
- [ ] **Térreo tombado**: nenhuma alteração estrutural (sem piso elevado, sem bandejamento). Pensar em: canaletas de superfície só onde já exista infra, ou priorizar **Wi-Fi/wireless** no térreo para minimizar cabeamento novo, ou usar os **shafts/dutos verticais já existentes** entre andares (sem abrir piso).
- [ ] 1º e 2º andar: sem essa restrição — cabeamento estruturado convencional é permitido (ver aula 07 do curso de Redes: TIA/EIA 568/569, limite de 90 m de permanent link / 100 m de canal).

## 2. Uso de cada andar

- [ ] **Térreo:** espaço de exposições aberto ao público + 60 posições de trabalho + copa com IoT.
- [ ] **1º andar:** datacenter + (possivelmente) posições de trabalho + copa com IoT.
- [ ] **2º andar:** só posições de trabalho + copa com IoT.

## 3. Serviços de rede exigidos

- [ ] **Ethernet cabeada** para todos os funcionários do 1º e 2º andar.
- [ ] **VoIP** para todos os funcionários do 1º e 2º andar (VLAN de voz dedicada é a prática padrão).
- [ ] **Wi-Fi para funcionários** em **todos os andares**, inclusive o térreo.
- [ ] **Wi-Fi separado para o público**, só no térreo (rede isolada da rede corporativa).
- [ ] **IoT** ao menos para as copas automatizadas, nos 3 andares.

## 4. Dimensionamento

- [ ] Rede dimensionada para **300 funcionários reais** (capacidade de endereçamento, portas, APs).
- [ ] Simulação reduzida a **~10% dos dispositivos de usuário** (≈ 30 estações finais) para o Packet Tracer não travar.
- [ ] Documentar a diferença entre "capacidade projetada" e "carga simulada" — é o ponto que a defesa presencial deve esclarecer.

## 5. Entrega

- [ ] Arquivo `.pkt`.
- [ ] Defesa presencial, em data a combinar — preparar para explicar: topologia, VLANs/endereçamento, onde ficou cada serviço (Ethernet/VoIP/Wi-Fi/IoT), e a decisão de redução para 10%.
