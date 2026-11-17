# Política de acesso

Sem ACLs o core roteia tudo entre tudo, e as VLANs separariam apenas o broadcast, sem dar segurança. A política abaixo é o que faz o isolamento acontecer de fato. Princípio: **menor privilégio**.

## 1. Matriz de acesso

Origem (linha) → destino (coluna). **Rascunho de política: validar nos testes de [`testes.md`](testes.md).**

| | Dados | Voz | Wi-Fi func. | Wi-Fi púb. | IoT | Datacenter | Gerência | Internet |
|---|---|---|---|---|---|---|---|---|
| **Dados (10)** | ✓ | ✗ | ✓ | ✗ | ✗ | ✓ | ✗ | ✓ |
| **Voz (20)** | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ (1) | ✗ | ✗ |
| **Wi-Fi func. (30)** | ✓ | ✗ | ✓ | ✗ | ✗ | ✓ | ✗ | ✓ |
| **Wi-Fi púb. (40)** | ✗ | ✗ | ✗ | ✓ | ✗ | ✗ | ✗ | ✓ |
| **IoT (50)** | ✗ | ✗ | ✗ | ✗ | ✓ | só controlador | ✗ | ✗ |
| **Datacenter (60)** | resposta | ✗ | resposta | ✗ | resposta | ✓ | ✗ | ✓ |
| **Gerência (99)** | só admin | só admin | só admin | ✗ | só admin | só admin | ✓ | ✗ |

(1) A telefonia roda no roteador, não no datacenter. O tráfego de registro e chamada dos telefones vai ao roteador de borda; liberar apenas as portas necessárias. O DHCP chega ao servidor pelo relay do core e não depende desta regra.

"resposta" significa que o datacenter responde a conexões iniciadas pela outra VLAN (usar `established` para TCP).

## 2. Decisões de política

- **Wi-Fi público:** só internet. DNS externo no escopo DHCP da VLAN 40, para não abrir acesso ao servidor DNS do datacenter.
- **IoT:** dispositivos de copa costumam ter software desatualizado e pouca segurança. Ficam isolados, falando só com o controlador. Se um for comprometido, o invasor está numa VLAN sem acesso ao resto.
- **Datacenter:** acessível por Dados e Wi-Fi funcionários. Não inicia conexão para as VLANs de usuário.
- **Gerência:** acessível só a partir de um host administrador.

## 3. Como aplicar no Packet Tracer

- ACLs **estendidas**, aplicadas **na entrada (`in`) da interface virtual da VLAN de origem**, no core.
- ACLs são **stateless** no Packet Tracer: para permitir respostas, usar `established` em TCP e liberar o necessário para UDP (por exemplo DNS).
- Todo fim de ACL tem um `deny ip any any` implícito. Conferir a ordem das linhas: a primeira que casa vale.
- Testar cada regra com ping e acesso a serviços, anotando o resultado em `testes.md`.

## 4. Reforço de camada 2 (aula 15 do curso)

| Ameaça | Defesa | No projeto |
|---|---|---|
| VLAN hopping (double tagging) | VLAN nativa dedicada e sem hosts; trunks fixos, sem negociação automática | Nativa 998 em todos os trunks; `switchport nonegotiate`; só as VLANs necessárias no trunk |
| MAC flooding / MAC spoofing | Port security (limitar MACs por porta) | Nas portas de acesso, onde o Packet Tracer permitir |
| ARP spoofing | DAI e IP Source Guard | Citar na defesa; depende do suporte no Packet Tracer |
| Broadcast storm | VLANs, STP, storm control | VLANs já reduzem o domínio de broadcast |
| Ataque ao STP | BPDU Guard nas portas de acesso | Se houver STP/portas de acesso configuradas |
| Portas esquecidas | Desligar e mover para VLAN sem uso | VLAN 999 (PARKING), portas em `shutdown` |

Nem todo recurso existe em todas as versões do Packet Tracer. Registrar em `testes.md` o que foi aplicado e o que ficou só como justificativa teórica.
