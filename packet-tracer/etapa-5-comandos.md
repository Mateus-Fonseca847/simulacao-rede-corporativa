# Etapa 5 · Segurança (NAT, ACLs, SSH e camada 2)

Arquivos gerados ao fim da etapa: `rede-v5-seguranca.pkt` e `rede-final.pkt`. Testes: seção "Etapa 5" de [`docs/testes.md`](../docs/testes.md). Política aplicada: [`docs/politica-de-acesso.md`](../docs/politica-de-acesso.md).

## Objetivo

Até a etapa 4 o core roteia tudo entre tudo. Esta etapa aplica a matriz de acesso:

- **NAT/PAT** no R-BORDA, para as redes internas saírem pela internet com o endereço público;
- **6 ACLs estendidas** no core, uma por papel, aplicadas na entrada (`in`) da SVI de cada VLAN de origem;
- **SSH v2** em todos os equipamentos, com a gerência limitada à VLAN 99;
- **port-security e BPDU guard** nas portas de acesso dos switches.

## O que se configura onde

| Equipamento | O que recebe nesta etapa |
|---|---|
| R-BORDA | `access-list 1` (redes que podem sair), NAT overload na Fa0/1, `ip nat inside/outside`, SSH |
| Core-L3 | ACL-FUNCIONARIOS (SVIs 10 e 30), ACL-VOZ (20), ACL-PUBLICO (40), ACL-IOT (50), ACL-DATACENTER (60), ACL-GERENCIA (99), SSH |
| SW-A1, SW-A2 | Port-security nas portas PC + telefone (máx. 3 MACs) e IoT (1 MAC); BPDU guard em todas as portas de acesso; SSH |
| SW-T | Port-security na porta IoT; BPDU guard nas portas de AP e IoT; SSH |
| SW-DC | Porta do PC-ADMIN (Fa0/24, VLAN 99); port-security nos servidores e no PC-ADMIN; BPDU guard; SSH |
| PC-ADMIN | IP fixo 10.10.10.130/27, gateway 10.10.10.129 (interface gráfica) |

## Comandos

### R-BORDA: NAT/PAT

```
enable
configure terminal
access-list 1 remark NAT-INTERNO
access-list 1 permit 10.10.0.0 0.0.1.255
access-list 1 permit 10.10.4.0 0.0.3.255
access-list 1 permit 10.10.8.0 0.0.1.255
access-list 1 permit 10.10.10.64 0.0.0.63
ip nat inside source list 1 interface FastEthernet0/1 overload
interface FastEthernet0/0
 ip nat inside
interface FastEthernet0/1
 ip nat outside
end
write memory
```

A `access-list 1` lista quem pode sair para a internet: DADOS (10), WIFI-FUNC (30), WIFI-PUBLICO (40) e DATACENTER (60). VOZ (20) e IOT (50) ficam de fora de propósito, porque a matriz de acesso não lhes dá internet. `overload` é o PAT: todas as máquinas internas compartilham o 203.0.113.1, separadas pelo número de porta.

### Core-L3: ACLs

```
enable
configure terminal
ip access-list extended ACL-FUNCIONARIOS
 permit udp any eq bootpc any eq bootps
 deny ip any 10.10.2.0 0.0.1.255
 deny ip any 10.10.8.0 0.0.1.255
 deny ip any 10.10.10.0 0.0.0.63
 permit icmp any host 10.10.10.130 echo-reply
 permit tcp any host 10.10.10.130 established
 deny ip any 10.10.10.128 0.0.0.31
 permit ip any any
ip access-list extended ACL-VOZ
 permit udp any eq bootpc any eq bootps
 permit udp 10.10.2.0 0.0.1.255 host 10.10.10.66 eq bootps
 permit ip 10.10.2.0 0.0.1.255 host 10.10.255.1
 permit ip 10.10.2.0 0.0.1.255 10.10.2.0 0.0.1.255
 deny ip any any
ip access-list extended ACL-PUBLICO
 permit udp any eq bootpc any eq bootps
 permit udp 10.10.8.0 0.0.1.255 host 10.10.10.66 eq bootps
 deny ip any 10.10.0.0 0.0.255.255
 permit ip any any
ip access-list extended ACL-IOT
 permit udp any eq bootpc any eq bootps
 permit udp 10.10.10.0 0.0.0.63 host 10.10.10.66 eq bootps
 permit ip 10.10.10.0 0.0.0.63 host 10.10.10.68
 deny ip any any
ip access-list extended ACL-DATACENTER
 permit udp host 10.10.10.66 eq bootps any
 permit udp host 10.10.10.67 eq domain any
 permit tcp 10.10.10.64 0.0.0.63 any established
 permit icmp 10.10.10.64 0.0.0.63 any echo-reply
 permit ip host 10.10.10.68 10.10.10.0 0.0.0.63
 deny ip 10.10.10.64 0.0.0.63 10.10.0.0 0.0.255.255
 permit ip 10.10.10.64 0.0.0.63 any
ip access-list extended ACL-GERENCIA
 permit ip host 10.10.10.130 10.10.0.0 0.0.255.255
 permit ip 10.10.10.128 0.0.0.31 10.10.10.128 0.0.0.31
 deny ip any any
exit
interface Vlan10
 ip access-group ACL-FUNCIONARIOS in
interface Vlan20
 ip access-group ACL-VOZ in
interface Vlan30
 ip access-group ACL-FUNCIONARIOS in
interface Vlan40
 ip access-group ACL-PUBLICO in
interface Vlan50
 ip access-group ACL-IOT in
interface Vlan60
 ip access-group ACL-DATACENTER in
interface Vlan99
 ip access-group ACL-GERENCIA in
end
write memory
```

O que cada ACL faz, linha a linha, na ordem em que o core testa:

| ACL | Libera | Bloqueia |
|---|---|---|
| ACL-FUNCIONARIOS (10 e 30) | DHCP; respostas ao PC-ADMIN (ping e TCP já estabelecido); todo o resto, incluindo datacenter e internet | VOZ, WIFI-PUBLICO, IOT e gerência |
| ACL-VOZ (20) | DHCP; tudo para o R-BORDA (registro e chamadas); voz com voz | Todo o resto, incluindo internet |
| ACL-PUBLICO (40) | DHCP; internet | Qualquer destino em 10.10.0.0/16 |
| ACL-IOT (50) | DHCP; tudo para o SRV-IOT (10.10.10.68) | Todo o resto |
| ACL-DATACENTER (60) | Respostas de DHCP e DNS; TCP já estabelecido; respostas de ping; SRV-IOT para a VLAN de IoT; internet | Iniciar conexão para as VLANs internas |
| ACL-GERENCIA (99) | PC-ADMIN para toda a rede interna; tráfego dentro da gerência | Todo o resto |

A primeira linha de quase todas (`permit udp any eq bootpc any eq bootps`) deixa passar o pedido DHCP do cliente, que ainda não tem IP (sai de 0.0.0.0) e é capturado pelo relay. As ACLs do Packet Tracer não guardam estado; por isso as respostas são liberadas com `established` (TCP), `echo-reply` (ping) e porta de origem (DHCP e DNS).

### SSH v2 com gerência limitada (todos os 6 equipamentos)

```
enable
configure terminal
service password-encryption
ip domain-name empresa.local
enable secret SENHA-DE-TESTE
username admin secret SENHA-DE-TESTE
crypto key generate rsa
1024
ip ssh version 2
access-list 10 remark GERENCIA-VTY
access-list 10 permit 10.10.10.128 0.0.0.31
line con 0
 logging synchronous
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
end
write memory
```

Trocar `SENHA-DE-TESTE` por uma senha de teste simples (nunca uma senha real). O `1024` é a resposta à pergunta do tamanho da chave; SSH v2 exige pelo menos 768 bits. `crypto key generate rsa` só funciona depois de `hostname` e `ip domain-name`, que formam o nome da chave.

`access-class 10 in` limita quem pode abrir sessão nas linhas vty: só a VLAN 99 (10.10.10.128/27). Mesmo que uma ACL do core falhasse, um PC comum não chegaria ao prompt.

### SW-A1: port-security e BPDU guard

```
enable
configure terminal
interface range FastEthernet0/1 - 9
 switchport port-security
 switchport port-security maximum 3
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree bpduguard enable
interface FastEthernet0/21
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree bpduguard enable
interface FastEthernet0/20
 spanning-tree bpduguard enable
end
write memory
```

No **SW-A2** o mesmo bloco, com `interface range FastEthernet0/1 - 15`.

- **Máximo 3** nas portas PC + telefone: o MAC do PC, o do telefone na VLAN de voz e o do telefone na VLAN de dados (o telefone também aparece nela).
- **Máximo 1** (o padrão, sem a linha `maximum`) nas portas de IoT e servidores: um equipamento só.
- **Porta de AP sem port-security:** o AP é uma ponte, e os MACs de todos os smartphones associados aparecem nessa porta. Com limite, cada celular a mais seria uma violação.
- **`sticky`** grava os MACs aprendidos na configuração; com `write memory` eles sobrevivem a um reinício.

### SW-T: port-security e BPDU guard

```
enable
configure terminal
interface FastEthernet0/3
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree bpduguard enable
interface range FastEthernet0/1 - 2
 spanning-tree bpduguard enable
end
write memory
```

### SW-DC: porta do PC-ADMIN, port-security e BPDU guard

```
enable
configure terminal
interface FastEthernet0/24
 description PC-ADMIN
 switchport mode access
 switchport access vlan 99
 spanning-tree portfast
 no shutdown
interface range FastEthernet0/1 - 3 , FastEthernet0/24
 switchport port-security
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree bpduguard enable
end
write memory
```

### PC-ADMIN (interface gráfica)

Desktop > IP Configuration > Static: IP `10.10.10.130`, máscara `255.255.255.224`, gateway `10.10.10.129`, DNS `10.10.10.67`.

## Verificação

| Comando / ação | Onde | O que conferir |
|---|---|---|
| `ping 203.0.113.10` | PC da VLAN 10 | Responde |
| `show ip nat translations` | R-BORDA | Linhas icmp com `inside local 10.10.0.11` → `inside global 203.0.113.1`, uma por porta |
| `ping 10.10.10.67` | Smartphone de visitante | 100% de perda (ACL-PUBLICO) |
| `ping 203.0.113.10` | Smartphone de visitante | Responde |
| `ping 10.10.10.129` e `ping 10.10.2.14` | PC da VLAN 10 | 100% de perda (ACL-FUNCIONARIOS) |
| `show access-lists` | Core | Contadores `(N match(es))` nas linhas exercitadas pelos testes |
| `ssh -l admin 10.10.10.133` | PC-ADMIN (Command Prompt) | Pede senha e entra no SW-A2 |
| `show ssh` | SW-A2, com a sessão aberta | A sessão do PC-ADMIN, versão 2 |
| `ssh -l admin 10.10.10.133` | PC da VLAN 10 | Connection timed out |
| `show port-security` | Switches de acesso | Portas protegidas, ação `Restrict`, contador de violações |
| `show port-security interface fa0/1` | SW-A1 | MACs sticky aprendidos, máximo 3 |

## Problemas comuns

- **Depois das ACLs, os clientes param de pegar IP:** faltou a linha `permit udp any eq bootpc any eq bootps` no topo da ACL da VLAN, ou a ACL-DATACENTER bloqueia a resposta do SRV-DHCP (ela precisa da linha `permit udp host 10.10.10.66 eq bootps any` antes do `deny`).
- **O ping sai, mas a resposta não volta:** a ACL não guarda estado. A resposta é um pacote novo, que entra pela SVI do outro lado e precisa ser liberada lá (`echo-reply`, `established`).
- **Uma regra não tem efeito:** a ordem importa e a primeira linha que casa decide. Um `permit ip any any` acima de um `deny` anula o `deny`. Para mudar a ordem no Packet Tracer, o mais seguro é `no ip access-list extended NOME` e colar a ACL de novo.
- **Tudo bloqueado numa VLAN:** toda ACL termina com um `deny ip any any` implícito. Uma ACL aplicada com nome errado (inexistente) não filtra nada; uma ACL criada pela metade bloqueia quase tudo.
- **NAT sem traduções:** faltou `ip nat inside` ou `ip nat outside` em uma das interfaces, ou a rede de origem não está na `access-list 1`.
- **`crypto key generate rsa` recusado:** falta `hostname` ou `ip domain-name`.
- **SSH recusado até do PC-ADMIN:** `login local` sem `username` criado, ou a `access-class 10` com a rede errada.
- **Linhas vty 5 a 15 nos 2960:** o SW-T e o SW-DC têm 16 linhas vty e o bloco acima só restringe as linhas 0 a 4. As outras estão com `login` sem senha, o que recusa a conexão, mas o mais limpo é repetir o bloco com `line vty 5 15` (ou usar `line vty 0 15`).
- **Porta parou de passar tráfego depois de trocar o PC:** com `sticky`, o MAC antigo continua gravado e o novo conta como violação. Remover o MAC antigo da porta (`no switchport port-security mac-address sticky <MAC>`) e salvar.
