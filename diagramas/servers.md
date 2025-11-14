# Arquitetura Corporativa — Guia Completo de Infraestrutura (AD, DNS, DHCP, PXE, Virtualização, SAN, VLANs, DMZ, Segurança)

Este documento reúne **boas práticas, padrões, modelos e arquitetura completa** para criação de uma infraestrutura corporativa moderna e escalável.

Serve como:
- 📘 Base para estudos  
- 🧩 Padrão para equipes de TI  
- 🏛️ Guia de implementação  
- 🛡️ Referência de segurança  
- 🗂️ Material de documentação corporativa  

---

## 🔵 1. Visão Arquitetural Geral

A arquitetura é composta por:

- **Serviços centrais**: AD, DNS, DHCP  
- **Infra de armazenamento**: SAN (iSCSI/FC), File Server  
- **Virtualização**: Cluster Hyper-V ou VMware  
- **Serviços complementares**: Backup, PXE, Intranet  
- **Segurança**: Firewall, DMZ, VLANs, segmentação, hardening  
- **Automação**: Scripts, IaC, padronização  
- **Monitoramento**: Logs, alertas, métricas  

---

## 🔵 2. Infraestrutura — Componentes principais

| Componente | Função | IP Exemplo | Observações |
|-----------|--------|------------|-------------|
| Firewall / Gateway | Perímetro | 192.168.1.1 | Regras LAN, DMZ, Wi‑Fi, VPN |
| DC01 | AD + DNS + DHCP | 192.168.1.10 | Domínio corporativo |
| DC02 | AD Secundário | 192.168.1.11 | Redundância |
| FS01 | File Server | 192.168.1.20 | SMB, cotas, DFS opcional |
| PXE01 | PXE/iPXE/WDS | 192.168.1.50 | Deploy de imagens |
| BKP01 | Backup | 192.168.1.40 | Veeam, Bacula, etc. |
| HV01 / HV02 | Hypervisors | VLAN MGMT | Alta disponibilidade |
| SAN | Armazenamento | rede storage | LUNs para cluster |
| WEB01 | Intranet | 192.168.1.30 | Aplicações internas |
| WEB-PUB | Servidor público | DMZ | Site externo |
| APs | Wi-Fi | VLAN 20 | Guest + Corp |
| Estações | Clientes | DHCP | Autenticação AD |

---

## 🔵 3. Segmentação de Rede (VLANs)

| VLAN | Nome | Sub-rede | Função |
|------|-------|-----------|--------|
| 10 | LAN | 192.168.1.0/24 | Servidores e desktops |
| 20 | Wi-Fi | 192.168.20.0/24 | Dispositivos móveis |
| 30 | Guest Wi-Fi | 172.16.30.0/24 | Internet somente |
| 50 | DMZ | 192.168.50.0/24 | Serviços públicos |
| 60 | VOIP | 192.168.60.0/24 | Telefones |
| 70 | Cameras | 192.168.70.0/24 | CCTV |
| 99 | MGMT | 192.168.99.0/24 | Switches / Hypervisors |

Boas práticas:
- Separar tráfego por tipo, risco e criticidade.  
- Criar ACLs explícitas entre VLANs.  
- Bloquear tráfego lateral desnecessário.  
- Gerenciamento sempre isolado.  

---

## 🔵 4. Active Directory — Boas Práticas

### OU Structure (exemplo)
- **Corp.local**
  - **Servidores**
    - DC
    - Arquivos
    - Aplicações
  - **Computadores**
    - Financeiro
    - RH
    - Operação
  - **Usuários**
    - Admins
    - Corporativo
  - **Grupos**
    - Segurança
    - Acesso a pastas
    - Funções específicas

### GPOs essenciais
- Hardening de estações  
- Disable USB (exceto permitidos)  
- Wallpaper corporativo  
- Auditoria avançada  
- Configuração de updates  
- Deploy de softwares via GPO  

### Segurança
- Desabilitar admin local default  
- MFA para administradores  
- Senhas complexas  
- LAPS aplicado  
- Auditoria de autenticações  

---

## 🔵 5. DHCP — Estrutura recomendada

### Escopo Exemplo:
- **192.168.1.100 a 192.168.1.200**
- Gateway: 192.168.1.1
- DNS: 192.168.1.10 e 192.168.1.11
- Lease: 8 horas
- Reservas: impressoras, APs, switches

### Opções adicionais
- PXE Boot → opção 66 e 67  
- Definir classes de equipamentos  
- Registrar eventos no Syslog  

---

## 🔵 6. DNS — Zona Interna

- Zona principal: **corp.local**
- Zonas reversas configuradas
- Aging & Scavenging habilitado
- Forwarders para: 1.1.1.1 / 8.8.8.8

Regra de ouro:
> DNS do cliente sempre deve ser o DC.  

---

## 🔵 7. Storage + Cluster

### Storage
- Multipath (MPIO) habilitado  
- LUNs separadas por função (VMs, ISOs, Backups)  
- Snapshots + retenção  

### Cluster de Hypervisors
- Alta disponibilidade  
- Live migration  
- vSwitch com VLAN trunk  
- Hosts em VLAN MGMT  

---

## 🔵 8. PXE/iPXE — Deploy Corporativo

### Fluxo
1. Criar imagem Windows com Sysprep  
2. Capturar com DISM  
3. Hospedar no PXE/WDS ou iPXE  
4. Automatizar com unattend.xml  
5. Registrar máquina no AD automaticamente  

### Scripts recomendados:
- Python: envio de log + inventário  
- PowerShell: pós-instalação  
- Bash: verificação de hosts  

---

## 🔵 9. DMZ — Padrões

Servidores isolados da LAN.  
Exemplo de serviços:
- Web externa  
- Reverse proxy  
- APIs públicas  

Regras mínimas:
- Entrada permitida: porta 80/443 somente  
- Saída restrita  
- Sem comunicação direta com AD  

---

## 🔵 10. Segurança — Checklist Completo

### Hardening
- CIS Benchmark  
- Firewall interno ativo  
- SSH com chave  
- Senhas expiram  
- LAPS ativo  
- TLS 1.0/1.1 desabilitado  

### Monitoramento
- Logs de auditoria  
- Zabbix / Prometheus / Grafana  
- Alertas para:
  - Falhas de login  
  - Perda de comunicação  
  - Falta de espaço  
  - Latência no storage  

### Backup
- 3–2–1 rule  
- Teste de restore mensal  
- Criptografia em repouso  

---

## 🔵 11. Arquitetura Física (conceitual)

- Firewall controla acesso à Internet  
- Switch Core distribui VLANs  
- Hypervisors conectados à SAN  
- Servidores essenciais na LAN  
- DMZ isolada  
- Wi-Fi separado  
- Management separado  

---

## 🔵 12. Estrutura recomendada de repositório

```
/docs/
  README.md
  topology.md
  naming_convention.md
  network_inventory.md
  vlan_plan.md
  pxe_guide.md
  ad_best_practices.md

/scripts/
  windows/
  linux/
  pxe/
  inventory/

/infra/
  ansible/
  terraform/

/diagrams/
  topology.svg
  topology.png

/config/
  dhcp_scopes.conf
  dns_zones.conf
```

---

## 🔵 13. Arquivos sugeridos para continuar o projeto

- **Inventário de Rede**  
- **Plano de VLANs completo**  
- **Naming Convention corporativa**  
- **Playbooks Ansible**  
- **Guia de PXE com imagens personalizadas**  
- **Checklist de Hardening**  
- **Runbooks de manutenção e incidentes**  

---

## Fim do documento
Versão: v2 — Infraestrutura Expandida
