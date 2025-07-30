# hosted-list

```mermaid
graph TB
%% Style definitions
classDef homeServer fill:#e2e8f0,stroke:#334155,stroke-width:2px
classDef service fill:#f8fafc,stroke:#64748b,stroke-width:1px
classDef monitoring fill:#d1fae5,stroke:#047857,stroke-width:1px
classDef security fill:#fee2e2,stroke:#991b1b,stroke-width:1px
classDef common fill:#fef3c7,stroke:#b45309,stroke-width:1px,font-style:italic
classDef cloudflare fill:#f0fdfa,stroke:#0f766e,stroke-width:1.5px
classDef rpi fill:#fde68a,stroke:#d97706,stroke-width:2px

%% Main Infrastructure
subgraph main_servers[Main Servers]
    direction LR
    
    %% Common services
    common["共通: Tailscale, Node Exporter, cAdvisor, Fail2ban, DNSCrypt-Proxy, Borg"]:::common
    
    subgraph balthasar[balthasar]
        direction TB
        minio[MinIO]
        cloudflared_b[Cloudflared]:::cloudflare
        
        subgraph social[Social]
            yamisskey[Misskey]
            neoquesdon[Neo-Quesdon]
        end
        
        subgraph matrix[Matrix]
            synapse[Synapse]
            element[Element]
        end
        
        subgraph apps[Apps]
            lemmy[Lemmy]
            outline[Outline]
            vikunja[Vikunja]
            cryptpad[CryptPad]
        end
    end
    
    subgraph caspar[caspar]
        direction TB
        cloudflared_c[Cloudflared]:::cloudflare
        
        subgraph security[Security]
            zitadel[Zitadel]
        end
        
        subgraph monitoring[Monitoring]
            grafana[Grafana]
            prometheus[Prometheus]
            uptime[Uptime Kuma]
        end
        
        subgraph CTF[CTF]
            ctfd[CTFd]
            vm[VM]
        end
        
        subgraph social_c[Social]
            nayamisskey[Misskey N/A]
            nostream[Nostr]
        end
    end
    
    subgraph raspberrypi[raspberrypi - Raspberry Pi 5<br/>NVMe SSD 2TB, 8GB RAM]
        direction TB
        playig[playit.gg]
        
        subgraph games[Games]
            minecraft[Minecraft Java]
        end
    end
    
    %% External entity
    internet((Internet)):::cloudflare
end

%% Core connections between main servers
zitadel --> outline
minio --> social & outline
element --> synapse
minecraft --> playig
prometheus --> grafana
uptime -.-> balthasar
uptime -.-> caspar
uptime -.-> raspberrypi

%% Cloudflared connections for external access
playig --> internet
cloudflared_b --> internet
cloudflared_c --> internet
yamisskey & neoquesdon & element & synapse & outline & vikunja & cryptpad & lemmy--> cloudflared_b
nayamisskey & nostream & grafana & ctfd & zitadel --> cloudflared_c

%% Apply styles
class balthasar,caspar homeServer
class raspberrypi rpi
class security,social,social_c,matrix,apps,games,CTF service
class monitoring monitoring
class security security
class cloudflared_b,cloudflared_c cloudflare
```
```mermaid
graph TB
%% Style definitions
classDef homeServer fill:#e2e8f0,stroke:#334155,stroke-width:2px
classDef service fill:#f8fafc,stroke:#64748b,stroke-width:1px
classDef monitoring fill:#d1fae5,stroke:#047857,stroke-width:1px
classDef security fill:#fee2e2,stroke:#991b1b,stroke-width:1px
classDef storage fill:#f3e8ff,stroke:#7e22ce,stroke-width:1.5px
classDef network fill:#fff3e0,stroke:#ef6c00,stroke-width:2px

%% Proxmox Host
subgraph proxmox["GMKtec NucBox K10 - Proxmox VE<br/>Core i9-13900HK, 64GB DDR5, 1TB NVMe"]
    %% Storage Configuration
    subgraph storage["Storage Pools"]
        local["local<br/>ISO, Templates, Backups"]:::storage
        local_lvm["local-lvm<br/>VM Disks"]:::storage
    end
    
    %% Network Bridges
    subgraph networks["Virtual Networks"]
        vmbr0["vmbr0 - External<br/>WAN Interface"]:::network
        vmbr1["vmbr1 - Internal LAN<br/>10.0.0.0/24"]:::network
        vmbr2["vmbr2 - DMZ<br/>192.168.100.0/24"]:::network
        vmbr3["vmbr3 - Management<br/>172.16.0.0/24"]:::network
    end
    
    %% Virtual Machines
    subgraph vms["Virtual Machines"]
        subgraph pfsense_vm["pfSense VM - 4c/8GB/32GB"]
            pfsense["pfSense 2.7+"]:::security
            suricata["Suricata IDS/IPS"]:::security
            haproxy["HAProxy"]:::service
            openvpn["OpenVPN"]:::security
        end
        
        subgraph omv_vm["OpenMediaVault VM - 4c/16GB/600GB"]
            omv["OpenMediaVault 7"]:::storage
            smb["SMB/CIFS Shares"]:::storage
            nfs["NFS Shares"]:::storage
            snapraid["SnapRAID"]:::storage
        end
        
        subgraph tpot_vm["T-Pot VM - 6c/24GB/200GB"]
            tpot["T-Pot 24.04+"]:::security
            cowrie["Cowrie SSH Honeypot"]:::security
            dionaea["Dionaea Multi-protocol"]:::security
            elasticpot["ElasticPot"]:::security
            kibana_tpot["Kibana Dashboard"]:::monitoring
        end
        
        subgraph malcolm_vm["Malcolm VM - 8c/32GB/500GB"]
            malcolm["Malcolm"]:::monitoring
            elasticsearch["Elasticsearch"]:::monitoring
            logstash["Logstash"]:::monitoring
            zeek["Zeek Network Analysis"]:::monitoring
            suricata_malcolm["Suricata IDS"]:::security
            kibana_malcolm["Kibana Analytics"]:::monitoring
        end
    end
end

%% Network connections
vmbr0 --> pfsense_vm
vmbr1 --> omv_vm
vmbr2 --> tpot_vm
vmbr2 --> malcolm_vm
vmbr3 --> pfsense_vm

%% Storage connections
local_lvm --> pfsense_vm
local_lvm --> omv_vm
local_lvm --> tpot_vm
local_lvm --> malcolm_vm

%% Service connections
tpot --> kibana_tpot
malcolm --> elasticsearch
suricata --> malcolm

%% Apply styles
class proxmox homeServer
class pfsense_vm,omv_vm,tpot_vm,malcolm_vm homeServer
```
```mermaid
graph TB
%% Style definitions
classDef homeServer fill:#e2e8f0,stroke:#334155,stroke-width:2px
classDef service fill:#f8fafc,stroke:#64748b,stroke-width:1px
classDef backup fill:#dbeafe,stroke:#2563eb,stroke-width:1.5px
classDef storage fill:#f3e8ff,stroke:#7e22ce,stroke-width:1.5px

%% External storage services
subgraph external[外部ストレージ]
    direction LR
    r2[Cloudflare R2]:::storage
    filen[Filen\nE2E暗号化]:::storage
end

%% Support Infrastructure 
subgraph internal[内部インフラ]
    direction TB
    
    %% Borg backup server
    subgraph raspi[raspi]
        borg_client[Borg Client]:::backup
        borg_data[(Backup Storage)]:::backup
    end
    
    %% Main servers with backup systems
    subgraph main_servers[Main Servers]
        direction LR
        
        subgraph balthasar[balthasar]
            direction TB
            yamisskey[Misskey]:::service
            yamisskey_db[(Misskey DB)]:::service
            minio[MinIO]:::storage
            borg_b[Borg Server]:::backup
        end
        
        subgraph caspar[caspar]
            nayamisskey[Misskey N/A]:::service
            borg_c[Borg Server]:::backup
        end
    end
end

%% Internal service connections
yamisskey --> yamisskey_db
yamisskey --> minio

%% DB backup connections to external storage
yamisskey_db -- "DB Backup" --> r2
yamisskey_db -- "DB Backup" --> filen

%% MinIO backup connections (Borg only)
minio -- "Borg Backup" --> borg_b

%% Borg backup connections
borg_b -- "Tailscale SSH" --> borg_client
borg_c -- "Tailscale SSH" --> borg_client
borg_client --> borg_data

%% Apply styles
class balthasar,caspar,raspi homeServer
class yamisskey,nayamisskey,yamisskey_db service
class borg_client,borg_b,borg_c,borg_data backup
class r2,filen,minio storage
```
```mermaid
graph TB
%% Style definitions
classDef homeServer fill:#e2e8f0,stroke:#334155,stroke-width:2px
classDef service fill:#f8fafc,stroke:#64748b,stroke-width:1px
classDef security fill:#fee2e2,stroke:#991b1b,stroke-width:1px
classDef cloudflare fill:#f0fdfa,stroke:#0f766e,stroke-width:1.5px
classDef internet fill:#e0f2fe,stroke:#0284c7,stroke-width:1.5px

%% Support Infrastructure with connections to main servers
subgraph support[Support Infrastructure]
    direction TB
    
    subgraph linode[Linode Servers]
        direction TB
        
        subgraph vpn[linode-vpn]
            algo[Algo VPN]:::security
        end
        
        subgraph proxy[linode-proxy]
            summaryproxy[Summary proxy]:::service
            mediaproxy[Media proxy]:::service
            squid[Squid]:::security
            cloudflared_proxy[Cloudflared]:::cloudflare
        end
    end
    
    %% Main server references with their social services
    subgraph main_servers[Main Servers]
        direction LR
        
        subgraph balthasar[balthasar]
            yamisskey[Misskey]:::service
            cloudflared_b[Cloudflared]:::cloudflare
        end
        
        subgraph caspar[caspar]
            nayamisskey[Misskey N/A]:::service
            cloudflared_c[Cloudflared]:::cloudflare
        end
    end
    
    %% External networks
    cloudflare_network[Cloudflare Network]:::cloudflare
    internet((Internet)):::internet
end

%% Connections to proxies
yamisskey --> summaryproxy & mediaproxy
nayamisskey --> summaryproxy & mediaproxy

%% Direct connections from Misskey to Cloudflared
yamisskey --> cloudflared_b
nayamisskey --> cloudflared_c

%% Misskey connections to VPN and proxy
yamisskey --> algo & squid
nayamisskey --> algo & squid

%% Direct VPN and proxy connections to internet
algo --> internet
squid --> internet

%% Cloudflare connections
cloudflared_b & cloudflared_c --> cloudflare_network
cloudflared_proxy --> cloudflare_network
cloudflare_network --> internet

%% Proxy service connections to Cloudflared
summaryproxy & mediaproxy --> cloudflared_proxy
```
