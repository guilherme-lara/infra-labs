graph TD

%% =======================
%% REDE CORPORATIVA
%% =======================

subgraph INTERNET["🌎 Internet"]
end

subgraph FIREWALL["🛡️ Firewall / Gateway (192.168.1.1)"]
end

INTERNET --> FIREWALL

%% -----------------------
%% VLANs INTERNAS
%% -----------------------

subgraph VLAN_LAN["🔷 VLAN LAN — 192.168.1.0/24"]
    
    subgraph SERVERS["🗄️ Servidores Internos"]

        DC01["🖥️ DC01
        - Active Directory
        - DNS
        - DHCP
        IP: 192.168.1.10"]

        FS01["📁 FS01
        - File Server
        - SMB Shares
        IP: 192.168.1.20"]

        WEB01["🌐 WEB01
        - Intranet / IIS
        IP: 192.168.1.30"]

        BKP01["💾 BKP01
        - Servidor de Backup
        - Veeam/Bacula
        IP: 192.168.1.40"]

    end

    subgraph CLIENTES["💻 Estações / Dispositivos"]
        PC01["💻 PC01"]
        PC02["💻 PC02"]
        NB01["💼 Notebook"]
    end

end

%% -----------------------
%% HYPERVISOR CLUSTER
%% -----------------------

subgraph HYPER["🖧 Cluster de Virtualização"]
    HV01["🧩 Hypervisor 01
    Hyper-V/VMware"]
    HV02["🧩 Hypervisor 02
    Hyper-V/VMware"]
    SAN["📦 SAN Storage
    iSCSI / Fibre Channel"]
end

HV01 --- SAN
HV02 --- SAN

%% -----------------------
%% WI-FI VLAN
%% -----------------------

subgraph VLAN_WIFI["📶 VLAN Wi-Fi — 192.168.20.0/24"]
    AP01["📡 Access Point (Wi-Fi CORP)"]
    WIFI_CLIENTS["📱 Dispositivos Wi-Fi"]
end

%% -----------------------
%% DMZ
%% -----------------------

subgraph DMZ["🛑 DMZ — 192.168.50.0/24"]
    WEBPUB["🌍 Servidor Web Público
    (Reverse Proxy / Nginx / IIS)"]
end


%% =======================
%% LIGAÇÕES
%% =======================

FIREWALL --> VLAN_LAN
FIREWALL --> VLAN_WIFI
FIREWALL --> DMZ

%% Hypervisors provide VMs to servers
HYPER --> DC01
HYPER --> FS01
HYPER --> WEB01
HYPER --> BKP01
HYPER --> WEBPUB

%% DHCP / DNS responses
DC01 -. DHCP/DNS .-> PC01
DC01 -. DHCP/DNS .-> PC02
DC01 -. DHCP/DNS .-> NB01
DC01 -. DHCP/DNS .-> WIFI_CLIENTS

%% Clients access servers
PC01 --> FS01
PC01 --> WEB01
PC01 --> DC01

NB01 --> FS01
NB01 --> WEB01

AP01 --> WIFI_CLIENTS
