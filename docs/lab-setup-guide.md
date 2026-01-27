# 🛠️ Guia de Instalação e Configuração do Laboratório

Este documento descreve os passos técnicos realizados para provisionar o ambiente de **Threat Detection** utilizando VirtualBox, Wazuh e Ubuntu Server.

## 📋 Pré-requisitos
* **Virtualizador:** Oracle VirtualBox 7.2.4 + Extension Pack.
* **Imagens:**
    * Wazuh Manager 4.14.2 (OVA).
    * Ubuntu Server 22.04 LTS (ISO).
* **Hardware do Host:** Processador i5, 8GB RAM (mínimo).

---

## 1. Configuração de Rede (Hardening)
Para garantir o isolamento do laboratório da rede doméstica, mas permitindo acesso ao dashboard, foi utilizada uma **NAT Network** com **Port Forwarding**.

1.  **Criação da Rede NAT (VirtualBox):**
    * **Nome:** `Lab-SOC-Net`
    * **CIDR:** `10.0.2.0/24`
    * **DHCP:** Habilitado
2.  **Regra de Redirecionamento de Portas (Port Forwarding):**
    * Objetivo: Acessar o Dashboard HTTPS via Host.
    * **Regra:** Host Port `4443` -> Guest Port `443` (IP `10.0.2.6`).

---

## 2. Deploy do Wazuh Manager (SIEM)
1.  Importação da OVA oficial do Wazuh.
2.  **Alocação de Recursos:** 4GB RAM / 2 vCPUs.
3.  **Configuração de Rede:** Interface definida para `NAT Network: Lab-SOC-Net`.
4.  **IP Estático:** Configurado internamente para `10.0.2.6`.

---

## 3. Configuração do Endpoint (Vítima)
1.  Instalação limpa do **Ubuntu Server 22.04.3.
2.  **Alocação de Recursos:** 1GB RAM / 1 vCPU.
3.  **Instalação do Agente:** Deploy realizado via comando `wget` gerado pelo Wazuh Manager.
4.  **Tuning de Monitoramento (FIM Realtime):**
    * Edição do arquivo `/var/ossec/etc/ossec.conf`.
    * Alteração na tag `<syscheck>` para monitoramento em tempo real de diretórios críticos:
    ```xml
    <directories check_all="yes" realtime="yes" report_changes="yes">/etc</directories>
    ```

---

## 4. Setup de Emulação de Adversário (Atomic Red Team)
Instalação do framework para execução de TTPs baseados no MITRE ATT&CK.

**Passo A: Instalação do PowerShell Core no Linux**
```bash
# Instalação das dependências e repositório Microsoft
sudo apt-get install -y wget apt-transport-https software-properties-common
wget -q "[https://packages.microsoft.com/config/ubuntu/$(lsb_release](https://packages.microsoft.com/config/ubuntu/$(lsb_release) -rs)/packages-microsoft-prod.deb"
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get install -y powershell
```

**Passo B: Instalação do Módulo Atomic Red Team Executado dentro do shell `pwsh`**
```Powershell
IEX (IWR '[https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1](https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1)' -UseBasicParsing);
Install-AtomicRedTeam -getAtomics
```

## 5. Validação
- Conectividade: Dashboard acessível via `https://localhost:4443` (ou IP do Host).
- Agente: Status "Active" no painel de gerenciamento.
- Alertas: Teste de criação/deleção de arquivos em `/etc` gerando alertas de integridade em < 5 segundos.
