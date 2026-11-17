# Defesa

Roteiro para a defesa presencial. A defesa testa se você entende o que fez, então escreva as respostas com suas palavras ao fim de cada etapa.

## 1. Perguntas prováveis

### Por que VLAN por função e não por andar?
Se as VLANs fossem por andar, quem muda de andar mudaria de rede, e as regras de segurança ficariam espalhadas. Por função, a política é uma só ("o Wi-Fi público não alcança o datacenter") e vale para o prédio inteiro. O endereço também indica o papel do dispositivo.

### Como a rede suporta 300 se você simulou 30?
O enunciado pede 10% da carga simulada, mantendo a capacidade total. Os blocos de endereço, os escopos DHCP e a quantidade de portas foram calculados para os 300 (com 30% de margem), e estão em `plano-enderecamento.md`. Na simulação, os escopos estão configurados completos; só poucos hosts pedem endereço.

### O que acontece se o core cair?
Com o core único, a comunicação entre VLANs e a saída para a internet param; cada andar continua comunicando dentro da própria VLAN. É um ponto único de falha, assumido por simplicidade. A solução é um segundo core com dois uplinks por switch de acesso e STP (ou Rapid PVST+) para evitar loops, o que ficaria como evolução.

### Como o visitante é impedido de chegar ao datacenter?
Ele está na VLAN 40, um domínio de broadcast separado. Para sair dela o tráfego passa pelo core, onde uma ACL estendida aplicada na entrada da interface da VLAN 40 só permite saída para a internet. Sem a ACL, o core roteia tudo entre tudo.

### Por que switch L3 e não router-on-a-stick?
No router-on-a-stick todo o tráfego entre VLANs passa por um único link de trunk até o roteador, que vira gargalo. O switch L3 roteia internamente, em hardware, com uma interface virtual por VLAN, e escala melhor. A aula 14 do curso cita as duas opções para conectar VLANs diferentes.

### Por que o térreo só tem Wi-Fi?
É um prédio tombado e o térreo não aceita piso elevado nem bandejamento. O enunciado só exige Ethernet e VoIP nos andares 1 e 2, então no térreo o cabeamento fica no mínimo (uplink, PoE dos APs e a copa), e os 60 funcionários usam Wi-Fi.

### Como o cabo chega ao switch do térreo se o prédio é tombado?
Pela prumada técnica que o prédio já tem (shaft, caixa de escada ou prumada antiga), com o cabo em eletroduto flexível e selagem corta-fogo na passagem pela laje. Se a prumada for compartilhada com a elétrica, o uplink vai em fibra óptica, que é imune à interferência eletromagnética. Se não houver prumada aproveitável, a alternativa é uma canaleta aparente numa área de serviço, atrás da escada, nunca pelo espaço de exposição. O princípio é o de bem tombado: mínima intervenção e reversibilidade. Não se abre passagem nova, e tudo o que é instalado sai sem deixar marca. A distância não pesa: dois pavimentos ficam muito abaixo dos 90 m do enlace permanente.

### O que é a VLAN nativa e por que a 998?
Em um trunk 802.1Q, os quadros da VLAN nativa vão sem tag; todos os outros levam tag. Usar a VLAN 1 padrão com hosts expõe a rede a VLAN hopping por double tagging (aula 15). Uma VLAN nativa dedicada e sem hosts (998) elimina esse caminho. A nativa deve ser a mesma nos dois lados do trunk.

### Como é o quadro 802.1Q?
Acrescenta 4 bytes entre o MAC de origem e o campo de tipo: TPID de 16 bits (0x8100), PCP de 3 bits (prioridade 802.1p), DEI de 1 bit e VID de 12 bits. Com 12 bits há 4096 valores, mas 0 e 4095 são reservados, restando 4094 VLANs. O quadro etiquetado chega a 1522 bytes.

### Por que telefone e PC dividem a mesma porta?
Na porta há uma VLAN de dados (sem tag, do PC) e uma VLAN de voz (com tag, do telefone), o que economiza portas e cabos. O telefone marca a voz com a VLAN 20 e prioridade 802.1p, e repassa o tráfego do PC.

### Para que serve o PoE?
Entrega dados e energia no mesmo cabo (802.3af até 15,4 W; 802.3at até 30 W). Alimenta APs e telefones sem tomada ao lado, o que importa no térreo tombado.

### Para que serve o relay DHCP?
O DHCP começa com broadcast, que não atravessa VLANs. O `ip helper-address` na interface virtual de cada VLAN converte o pedido em unicast para o servidor no datacenter.

### Por que NAT/PAT?
Os endereços `10.x` são privados (RFC 1918) e não são roteados na internet. O PAT permite que muitas máquinas internas compartilhem um único endereço público, distinguidas por número de porta.

### A bridge separa domínios de broadcast?
Não. Bridge e switch separam **domínios de colisão**; o domínio de broadcast só é segmentado por **roteador ou VLAN**. O slide da aula 13 diz o contrário, mas a nuance correta é essa; se cair na prova, responda com o que a professora ensinou e acrescente a diferença.

### Por que 3560 e não 2960 nos andares 1 e 2?
Por causa do PoE. Os telefones IP precisam de energia, e o 3560-24PS (o "P" do nome indica portas PoE) entrega essa energia pelo próprio cabo de dados. Com um 2960-24TT, cada um dos 24 telefones precisaria de um adaptador de tomada. No térreo e no datacenter não há telefones, então ficou o 2960-24TT. O efeito colateral: o 3560 suporta dois encapsulamentos de trunk (ISL e 802.1Q), então foi preciso `switchport trunk encapsulation dot1q` antes de `switchport mode trunk`; sem isso o switch recusa o modo trunk.

### Por que a option 150 do DHCP aponta para o roteador?
A option 150 diz ao telefone Cisco qual é o servidor TFTP de onde ele baixa a configuração. Quem faz a telefonia é o próprio R-BORDA (`telephony-service`, o CME), que gera e serve esses arquivos. Por isso o escopo VOZ entrega 10.10.255.1, o mesmo endereço do `ip source-address 10.10.255.1 port 2000`, onde o telefone depois se registra. É também o único destino fora da VLAN de voz que a ACL-VOZ libera. A option 150 é específica da Cisco; a padrão é a 66.

### Por que as ACLs ficam na entrada da SVI e não na saída?
Três motivos. Primeiro, a regra da Cisco para ACL estendida é aplicá-la perto da origem: o pacote proibido é descartado assim que entra no core, antes de ser roteado. Segundo, cada ACL `in` corresponde a uma linha da matriz de acesso (o que a VLAN X pode alcançar), o que deixa a política fácil de ler e de manter. Terceiro, uma ACL `out` não filtra o tráfego destinado ao próprio core, porque esse tráfego não sai por interface nenhuma; o bloqueio do PC da VLAN 10 para 10.10.10.129 (teste 5.9) só funciona porque a ACL está na entrada.

### Por que o Wi-Fi público usa DNS externo?
Se o visitante usasse o DNS interno (10.10.10.67), a ACL-PUBLICO teria de abrir a porta 53 para um servidor do datacenter, o que é um furo no isolamento e ainda expõe nomes internos, como `intranet.empresa.local`. Com o DNS externo (203.0.113.10) a ACL pode negar qualquer destino em 10.10.0.0/16. A única exceção é o DHCP, que chega ao SRV-DHCP pelo relay. O teste 5.1 mostra o visitante sem acesso ao 10.10.10.67, e o 5.2 mostra ele chegando à internet.

### O que faz o port-security com `violation restrict`, e por que não `shutdown`?
O port-security limita quantos MACs uma porta aceita: 3 nas portas de PC + telefone, 1 nas de IoT e servidores. Com `sticky`, os MACs aprendidos são gravados na configuração. Com `restrict`, quadros de um MAC além do limite são descartados, o contador de violações sobe e um log é gerado, mas a porta continua ativa e o PC e o telefone legítimos seguem funcionando. Com `shutdown`, a porta vai para *err-disabled* e cai inteira, telefone incluído, até um administrador intervir. Numa mesa com PC e telefone, alguém plugar um switch de mesa derrubaria o ramal daquela pessoa. O `restrict` contém o intruso sem tirar o serviço do ar e deixa registro. `shutdown` é mais rígido e faria sentido em portas de alto risco; `protect` descarta sem contar nem registrar.

## 2. Roteiro de demonstração (5 minutos)

1. **Topologia** (30 s): mostrar o desenho e apontar core, switches de acesso, borda e o térreo só com Wi-Fi.
2. **VLANs e trunks** (45 s): `show vlan brief` e `show interfaces trunk`, destacando a nativa 998.
3. **DHCP e roteamento** (60 s): PC recebe IP por relay e pinga outra VLAN permitida.
4. **VoIP** (45 s): chamada entre dois telefones.
5. **Wi-Fi** (45 s): notebook no SSID corporativo (VLAN 30) e outro no público (VLAN 40).
6. **Segurança** (60 s): o Wi-Fi público acessa a internet e é bloqueado no datacenter; `show access-lists` com os contadores.
7. **Capacidade** (30 s): abrir `plano-enderecamento.md` e mostrar a conta para 300.

## 3. Antes de entrar

- [ ] Abrir o `.pkt` final e conferir que roda sem erro
- [ ] Saber explicar cada comando de `configs/`
- [ ] Reler `plano-enderecamento.md` e `politica-de-acesso.md`
- [ ] Ter os prints dos testes à mão
- [ ] Treinar o roteiro de demonstração em voz alta, com cronômetro
