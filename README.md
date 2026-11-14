# Pr-tica-de-Simula-o-de-Ambiente-Hier-rquico---Configura-es-de-VLANs-tipo-Trunk
Essa atividade Orientada pelo professor Jhordano, teve com intuito de fazer o treinamento para fazer o "- Configurações de VLANs tipo trunk" 

# Ambiente hierárquico: enlaces trunk (Atividade 07)

Tutorial de configuração dos enlaces trunk (IEEE 802.1Q) entre os switches de acesso, distribuição e núcleo, no Cisco Packet Tracer.

- **Arquivo da simulação:** `Aula_de_switches_16_10.pkt`
- **Curso:** Redes de Computadores, IFRO
- **Disciplina:** Comutação de Redes Locais

## Sumário

1. [Visão geral da topologia](#visão-geral-da-topologia)
2. [Fundamentação: trunk e 802.1Q](#fundamentação-trunk-e-8021q)
3. [Mapeamento das interfaces](#mapeamento-das-interfaces)
4. [Passo a passo](#passo-a-passo)
5. [Comandos usados](#comandos-usados)
6. [Verificação](#verificação)
7. [Teste de conectividade](#teste-de-conectividade)
8. [Evidências](#evidências)

## Visão geral da topologia

| Camada | Equipamentos |
|---|---|
| Borda | Roteador R-BORDA-WAN (Cisco 1841) |
| Núcleo | SW-NUCLEO-01, SW-NUCLEO-02 |
| Distribuição | SW-DIST-01, SW-DIST-02 |
| Acesso | SW-ACESSO-01 a SW-ACESSO-04 (Cisco 2960-24TT) |
| Dispositivos finais | PC1 a PC4, Laptop1 a Laptop4, Server1 |

![Topologia](diagrama.png)

## Fundamentação: trunk e 802.1Q

- **Enlace trunk:** um único link físico que transporta o tráfego de várias VLANs entre switches.
- **IEEE 802.1Q:** padrão que identifica a VLAN de cada quadro com uma etiqueta (tag) de 4 bytes inserida no cabeçalho Ethernet. O switch que recebe o quadro lê a etiqueta e o encaminha para a VLAN correta.
- **VLAN nativa:** a única VLAN cujos quadros trafegam sem etiqueta no trunk. Neste cenário permanece a VLAN 1 (padrão) nos dois lados de cada enlace.
- **VLANs permitidas:** lista das VLANs que podem passar pelo trunk. Neste cenário, apenas as VLANs 10 (ALUNOS) e 20 (VISITANTES).

## Objetivo

- Configurar como trunk as portas de interligação entre acesso, distribuição e núcleo.
- Permitir somente as VLANs 10 e 20 nesses enlaces.

## Mapeamento das interfaces

### VLANs

| ID | Nome |
|---|---|
| 10 | ALUNOS |
| 20 | VISITANTES |

### Enlaces trunk

| Enlace | Portas (lado A) | Portas (lado B) |
|---|---|---|
| SW-ACESSO-01 ↔ SW-DIST-01 | SW-ACESSO-01: Fa0/5 | SW-DIST-01: Fa0/1 |
| SW-ACESSO-02 ↔ SW-DIST-01 | SW-ACESSO-02: Fa0/3 | SW-DIST-01: Fa1/1 |
| SW-ACESSO-03 ↔ SW-DIST-02 | SW-ACESSO-03: Fa0/3 | SW-DIST-02: Fa0/1 |
| SW-ACESSO-04 ↔ SW-DIST-02 | SW-ACESSO-04: Fa0/3 | SW-DIST-02: Fa1/1 |
| SW-DIST-01 ↔ SW-NUCLEO-01 | SW-DIST-01: Fa4/1, Fa5/1 | SW-NUCLEO-01: Fa1/1, Fa2/1 |
| SW-DIST-02 ↔ SW-NUCLEO-02 | SW-DIST-02: Fa4/1, Fa5/1 | SW-NUCLEO-02: Fa1/1, Fa2/1 |
| SW-NUCLEO-01 ↔ SW-NUCLEO-02 | SW-NUCLEO-01: Fa6/1, Fa7/1, Fa8/1, Fa9/1 | SW-NUCLEO-02: Fa0/1, Fa6/1, Fa8/1, Fa9/1 |

Portas que **não** são trunk: Fa5/1 do SW-NUCLEO-01 (Server1), Fa0/1 do SW-NUCLEO-01 e Fa7/1 do SW-NUCLEO-02 (R-BORDA-WAN), e as portas de acesso Fa0/1 e Fa0/2 dos switches de acesso.

## Passo a passo

### Passo 1: confirmar as portas de interligação

Antes de configurar, confirme em qual porta está cada cabo, em cada switch:

```
show cdp neighbors
```

### Passo 2: configurar os switches de acesso

SW-ACESSO-01 (uplink na Fa0/5):

```
enable
configure terminal
interface fastEthernet 0/5
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 end
copy running-config startup-config
```

SW-ACESSO-02, SW-ACESSO-03 e SW-ACESSO-04 (uplink na Fa0/3):

```
enable
configure terminal
interface fastEthernet 0/3
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 end
copy running-config startup-config
```

### Passo 3: configurar os switches de distribuição

SW-DIST-01 e SW-DIST-02 (mesmo bloco):

```
enable
configure terminal
interface range fastEthernet 0/1 , fastEthernet 1/1 , fastEthernet 4/1 , fastEthernet 5/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 end
copy running-config startup-config
```

### Passo 4: configurar os switches de núcleo

SW-NUCLEO-01:

```
enable
configure terminal
interface range fastEthernet 1/1 , fastEthernet 2/1 , fastEthernet 6/1 , fastEthernet 7/1 , fastEthernet 8/1 , fastEthernet 9/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 end
copy running-config startup-config
```

SW-NUCLEO-02:

```
enable
configure terminal
interface range fastEthernet 0/1 , fastEthernet 1/1 , fastEthernet 2/1 , fastEthernet 6/1 , fastEthernet 8/1 , fastEthernet 9/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 end
copy running-config startup-config
```

> Na Atividade 08, os enlaces entre núcleos e entre núcleo e distribuição passam a formar port-channels (EtherChannel), e o modo trunk também é configurado nas interfaces `port-channel`.

## Comandos usados

| Comando | Função |
|---|---|
| `show cdp neighbors` | Mostra os vizinhos e as portas de cada enlace |
| `interface range <portas>` | Aplica a mesma configuração a várias portas |
| `switchport mode trunk` | Força a porta a operar como trunk |
| `switchport trunk allowed vlan 10,20` | Permite somente as VLANs 10 e 20 no trunk |
| `show interfaces trunk` | Lista as portas em trunk, VLANs permitidas e ativas |
| `show vlan brief` | Lista as VLANs e as portas de acesso |

## Verificação

Em cada switch:

```
show interfaces trunk
```

Resultado esperado:

| Camada | Esperado |
|---|---|
| Acesso | Uplink em `trunking`, VLANs 10 e 20 permitidas e ativas |
| Distribuição | Portas para os acessos e para o núcleo em `trunking` |
| Núcleo | Portas para a distribuição e para o outro núcleo em `trunking` |

Em todos os enlaces, a coluna `Encapsulation` mostra `802.1q` e a `Native vlan` mostra `1`.

## Teste de conectividade

Os trunks permitem que dispositivos da mesma VLAN em ramos opostos se comuniquem. Use **Desktop > Command Prompt**.

| Teste | Origem | Comando | Esperado |
|---|---|---|---|
| VLAN 10, ramos opostos | PC1 | `ping 192.168.10.4` | Responde |
| VLAN 20, ramos opostos | Laptop1 | `ping 192.168.20.4` | Responde |
| Caminho percorrido | PC1 | `tracert 192.168.10.4` | Um salto (camada 2) |
| VLANs diferentes | PC1 | `ping 192.168.20.1` | Falha (`Request timed out`) |

O primeiro ping pode dar timeout por causa do ARP; repita uma vez antes de concluir.

## Evidências

- [ ] `show interfaces trunk` dos oito switches
- [ ] `show cdp neighbors` dos switches de acesso e distribuição
- [ ] Ping entre dispositivos da mesma VLAN em ramos opostos
- [ ] `tracert` com um salto
- [ ] Ping entre VLANs diferentes falhando
- [ ] Topologia final, com os cabos verdes

## Autor

João Arthur Amorim Maciel
