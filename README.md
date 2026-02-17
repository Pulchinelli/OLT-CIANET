# Template Zabbix 7 — Cianet GPON OLT G8PS X by SNMP

[![Zabbix 7.0](https://img.shields.io/badge/Zabbix-7.0-red?logo=zabbix)](https://www.zabbix.com/)
[![SNMP v2c](https://img.shields.io/badge/SNMP-v2c-blue)](https://en.wikipedia.org/wiki/Simple_Network_Management_Protocol)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Template para monitoramento de OLTs **Cianet G8PS X / G8PS X V2** via **SNMPv2c** no **Zabbix 7.x**.

Desenvolvido a partir de `snmpwalk` real em OLT de produção — Enterprise OID `1.3.6.1.4.1.17409` (Cianet Indústria e Comércio S/A).

> **Testado e validado em ambiente de produção** com 26 ONUs ativas, 5 portas PON em uso, 3 uplinks e firmware V1.1.0_201201 apenas no Zabbix 7.

---

## 📋 Funcionalidades

### Items Fixos (Sistema)

| Métrica | OID | Descrição |
|---|---|---|
| sysDescr | `1.3.6.1.2.1.1.1.0` | Descrição do sistema (modelo) |
| sysName | `1.3.6.1.2.1.1.5.0` | Hostname da OLT |
| sysLocation | `1.3.6.1.2.1.1.6.0` | Localização configurada |
| sysContact | `1.3.6.1.2.1.1.4.0` | Contato do responsável |
| sysUpTime | `1.3.6.1.2.1.1.3.0` | Uptime em centésimos de segundo |
| sysObjectID | `1.3.6.1.2.1.1.2.0` | Object ID do fabricante |
| Hostname Cianet | `...17409.2.3.1.2.1.1.2.1` | Hostname configurado na OLT |
| Modelo | `...17409.2.3.1.2.1.1.3.1` | Modelo do hardware |
| Serial Number | `...17409.2.3.1.1.13.0` | Número de série |
| Firmware | `...17409.2.3.1.3.1.1.8.1.0` | Versão do firmware |
| IP de Gerência | `...17409.2.3.1.1.2.0` | IP de gerência inband |
| MAC Address | `...17409.2.3.1.1.6.0` | MAC da interface inband |
| ICMP ping | Simple Check | Disponibilidade ICMP |
| ICMP loss | Simple Check | Perda de pacotes (%) |
| ICMP response time | Simple Check | Tempo de resposta (s) |
| SNMP availability | Internal | Status do agente SNMP |

### Discovery Rules (LLD)

| # | Discovery Rule | O que descobre | Items por entidade |
|---|---|---|---|
| 1 | **Uplink interfaces** | ge0/0/X, xge0/0/X | Status, tráfego in/out, velocidade |
| 2 | **PON ports** | pon0/0/1 a pon0/0/8 | ONUs autorizadas, max ONUs, BW total/alocado/livre, status, temperatura SFP, TX Power, Bias Current, tráfego in/out |
| 3 | **ONU interfaces** | pon0/0/X:Y | Status, tráfego in/out |
| 4 | **Power supplies** | power card 1, 2 | Status da fonte |
| 5 | **Fans** | fan card 1, 2, 3 | Status do ventilador |

### Triggers

| Trigger | Severidade | Condição |
|---|---|---|
| OLT reiniciada | ⚠️ Warning | Uptime < 10 minutos |
| OLT inacessível (ICMP) | 🔴 High | 3 pings consecutivos sem resposta |
| Alta perda ICMP | ⚠️ Warning | Perda > `{$ICMP_LOSS_WARN}`% |
| Alto RTT ICMP | ⚠️ Warning | RTT > `{$ICMP_RESPONSE_TIME_WARN}`s |
| SNMP indisponível | ⚠️ Warning | Sem coleta SNMP por `{$SNMP.TIMEOUT}` |
| Firmware alterado | ℹ️ Info | Mudança no valor do firmware |
| Uplink DOWN | 🟠 Average | Link uplink cai |
| PON DOWN (com ONUs) | 🔴 High | Porta PON cai com ONUs autorizadas |
| Temperatura SFP alta | ⚠️ Warning | Temp > `{$PON.SFP.TEMP.MAX}`°C |
| ONU offline | 🟠 Average | ONU muda para DOWN |
| Falha na fonte (PSU) | 🔴 High | Status ≠ Normal |
| Falha no ventilador | 🟠 Average | Status ≠ Normal |

### Value Maps

- `ifOperStatus` — up / down / testing / unknown / dormant / notPresent / lowerLayerDown
- `zabbix.host.available` — not available / available / unknown
- `Service state` — Down / Up
- `Cianet PSU Status` — Normal / Falha / Ausente
- `Cianet Fan Status` — Normal / Falha / Ausente

---

## 📁 Estrutura do Repositório

```
├── CIANET_GPON_OLT_G8PSX_SNMP.yaml   # Template para Zabbix 7
├── README.md                           # Este arquivo (PT-BR)
├── README.en.md                        # README em Inglês
└── LICENSE                             # Licença MIT
```

---

## ⚙️ Requisitos

- **Zabbix Server/Proxy** 7.x
- **OLT Cianet G8PS X** (ou G8PS X V2) com:
  - SNMP v2c habilitado
  - Comunidade de leitura configurada
- Conectividade **UDP/161** entre Zabbix e a OLT

---

## 🚀 Instalação

### 1. Importar o Template

1. No Zabbix frontend: **Data collection → Templates → Import**
2. Selecione o arquivo `CIANET_GPON_OLT_G8PSX_SNMP.yaml`
3. Em Rules, marque:
   - ✅ Templates: Create new + Update existing
   - ✅ Items, Triggers, Discovery rules, Value maps
4. Clique em **Import**

### 2. Criar o Host

1. **Data collection → Hosts → Create host**
2. Preencha:
   - **Host name:** `OLT-CIANET` (ou o nome desejado)
   - **Groups:** `OLTs Cianet` (ou o grupo do seu ambiente)
   - **Templates:** `Cianet OLT G8PS X by SNMP`
3. Na aba **Interfaces**, adicione interface SNMP:
   - **IP:** IP de gerência da OLT (ex.: `192.168.1.100`)
   - **Port:** `161`
   - **SNMP version:** `SNMPv2`
   - **Community:** `{$SNMP_COMMUNITY}`
4. (Opcional) Sobrescreva macros na aba **Macros** conforme seu ambiente

---

## 🔧 Macros

| Macro | Padrão | Descrição |
|---|---|---|
| `{$SNMP_COMMUNITY}` | `@read-write-community` | Comunidade SNMP da OLT |
| `{$SNMP.TIMEOUT}` | `5m` | Timeout de indisponibilidade SNMP |
| `{$ICMP_LOSS_WARN}` | `20` | Limite de perda ICMP (%) |
| `{$ICMP_RESPONSE_TIME_WARN}` | `0.15` | Limite de latência ICMP (s) |
| `{$PON.SFP.TEMP.MAX}` | `65` | Temperatura máxima SFP (°C) |

> 💡 **Dica:** Sobrescreva `{$SNMP_COMMUNITY}` no nível do host para cada OLT com comunidade diferente.

---

## 📊 Métricas em Produção

Em uma OLT G8PS X V2 típica com 5 PONs ativas e ~25 ONUs:

| Componente | Quantidade | Métricas geradas |
|---|---|---|
| Items fixos | 16 | 16 |
| Uplinks (6x) | 4 items/uplink | ~24 |
| PON ports (8x) | 11 items/PON | ~88 |
| ONUs (~25x) | 3 items/ONU | ~75 |
| PSUs (2x) | 1 item/PSU | 2 |
| Fans (3x) | 1 item/fan | 3 |
| **Total** | | **~208 métricas** |

---

## 📡 OIDs Enterprise Cianet (Referência)

```
1.3.6.1.4.1.17409                          # Enterprise OID Cianet
├── 2.3.1.1.*                               # System info (IP, MAC, serial, VLAN)
├── 2.3.1.2.*                               # Device info (hostname, model, uptime)
├── 2.3.1.3.*                               # Card info (firmware, serial)
├── 2.3.1.4.*                               # Power supply table
├── 2.3.1.5.*                               # Fan table
├── 2.3.2.1.*                               # Uplink port table (ge/xge)
├── 2.3.2.2.*                               # LAG table
├── 2.3.3.1.*                               # PON port table (ONUs, BW, status)
├── 2.3.3.5.*                               # PON optical (temp, TX, bias)
├── 2.3.7.*                                 # VLAN configuration
└── 2.3.9.*                                 # ONU detail/performance
```

---

## ⚠️ Limitações

- Template desenvolvido e testado para **Cianet G8PS X V2**. Outras OLTs Cianet (ex.: G16PS, G24PS) podem usar OIDs diferentes.
- Para ambientes com centenas de ONUs, considere aumentar os intervalos de coleta da LLD de ONUs ou desabilitá-la.
- O arquivo YAML é sensível à indentação — edite com cuidado.
- A MIB Host Resources (`1.3.6.1.2.1.25`) **não é suportada** por este equipamento.

---

## 🤝 Contribuições

Issues e Pull Requests são bem-vindos! Sugestões:

- Ajustes de OIDs para outros firmwares ou modelos Cianet
- Adição de dashboards ou gráficos prontos
- Otimização de triggers e thresholds
- Tradução para outros idiomas

---

## 📜 Licença

Este projeto está licenciado sob a [MIT License](LICENSE).

---

## 🙏 Créditos

- Desenvolvido por **Flyconecta** — Provedor de Internet Regional
- Baseado em dados reais de produção (snmpwalk completo na Enterprise MIB 17409)
- Template gerado com auxílio de análise automatizada de OIDs hexadecimais
