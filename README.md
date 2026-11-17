# simulacao-rede-corporativa

> Simulação, no Cisco Packet Tracer, de uma rede corporativa de 300 funcionários em um prédio histórico tombado de 3 andares — VLANs, Ethernet, VoIP, Wi-Fi corporativo/público e IoT.

Trabalho da disciplina de **Redes de Computadores** (em dupla ou individual).

## Prazo

- **Entrega:** 17 de novembro de 2026, 23:59.
- **Defesa:** presencial, em data a combinar.
- Vários envios permitidos até o prazo — commitar e enviar cedo, refinar depois.

## O cenário (enunciado resumido)

Prédio histórico de 3 andares (térreo, 1º e 2º), **térreo tombado**: no térreo **não pode haver** alterações como piso elevado ou bandejamento para passagem de cabos.

| Andar | Uso |
|---|---|
| Térreo | Espaço de exposições, **aberto ao público**, 60 posições de trabalho. Copa automatizada com IoT. |
| 1º andar | **Datacenter**. Pode haver também posições de trabalho. Copa automatizada com IoT. |
| 2º andar | Apenas posições de trabalho. Copa automatizada com IoT. |

A rede deve incluir:
- [ ] Acesso **Ethernet e VoIP** para todos os funcionários do **1º e do 2º andar**.
- [ ] **Wi-Fi** para os funcionários em **todos os andares**.
- [ ] **Wi-Fi separado** para o público, **no térreo**.
- [ ] **IoT** ao menos para as copas (nos 3 andares).

## Restrição importante: o Packet Tracer não aguenta 300 estações

> Use **apenas 10% da carga de dispositivos de usuários**, mas **mantendo a rede com a capacidade para atender a demanda total**.

Ou seja: a **topologia, o endereçamento e o dimensionamento** (VLANs, sub-redes, portas de switch, capacidade de AP) devem ser pensados para **300 funcionários reais**, mas o **arquivo .pkt simulado** só precisa ter ~30 estações finais ativas (10%) para não travar o programa. Documente essa decisão no relatório — é esperado que o avaliador pergunte sobre isso na defesa presencial.

## Checklist de progresso

- [ ] Plano de endereçamento e VLANs (`docs/plano-enderecamento.md`)
- [ ] Topologia definida (switches por andar, uplinks, redundância se houver)
- [ ] Datacenter (1º andar) modelado
- [ ] VoIP configurado (VLAN de voz separada, telefones IP ou softphones)
- [ ] Wi-Fi dos funcionários (todos os andares)
- [ ] Wi-Fi público isolado (só térreo)
- [ ] IoT das copas (3 andares)
- [ ] Solução para o térreo tombado (sem piso elevado / sem bandejamento)
- [ ] Redução para 10% da carga, com nota explicando o dimensionamento real
- [ ] Arquivo `.pkt` final em `projeto-pkt/`
- [ ] Relatório/roteiro de defesa (`docs/`)

## Estrutura do repositório

```
docs/
  requisitos.md          checklist detalhado do enunciado
  plano-enderecamento.md VLANs, sub-redes, DHCP, capacidade por andar
  topologia.md           descrição da topologia e decisões de projeto
  defesa.md              roteiro para a apresentação presencial
projeto-pkt/
  rede-corporativa.pkt   arquivo do Packet Tracer (adicionar depois de exportado)
```

## Por que versionar isso no Git

O `.pkt` é um arquivo binário (não dá para ver diff linha a linha), mas vale versionar mesmo assim: guarda o histórico de versões entregáveis, e os documentos em Markdown (`docs/`) — que são o raciocínio por trás do projeto — ficam com histórico completo e comentável.

## Declaração de uso de IA

Parte da estrutura inicial deste repositório (README, checklists, esqueleto dos documentos de projeto) foi gerada com apoio de um modelo de linguagem (Claude, Anthropic), a partir do enunciado do trabalho. Confira a política da disciplina sobre uso de IA generativa antes da entrega; se exigida declaração formal, adapte o texto de `DECLARACAO_IAG.md` do repositório [`mensageria-identidade-unica`](https://github.com/Mateus-Fonseca847/mensageria-identidade-unica) (mesmo texto-modelo) para este trabalho.
