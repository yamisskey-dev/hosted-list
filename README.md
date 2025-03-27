# hosted-list

```mermaid
graph TB
%% Style definitions
classDef homeServer fill:#e2e8f0,stroke:#334155,stroke-width:2px
classDef service fill:#f8fafc,stroke:#64748b,stroke-width:1px
classDef monitoring fill:#d1fae5,stroke:#047857,stroke-width:1px
classDef security fill:#fee2e2,stroke:#991b1b,stroke-width:1px
classDef common fill:#fef3c7,stroke:#b45309,stroke-width:1px,font-style:italic

%% Main Infrastructure
subgraph main_servers[Main Servers]
    direction LR
    
    %% Common services updated to include DNSCrypt
    common["共通: Tailscale, Node Exporter, cAdvisor, Fail2ban, DNSCrypt-Proxy"]:::common
    
    subgraph balthasar[balthasar]
        direction TB
        minio[MinIO]
        suricata[Suricata]
        
        subgraph social[Social]
            yamisskey[Misskey]
            neoquesdon[Neo-Quesdon]
        end
        
        subgraph matrix[Matrix]
            synapse[Synapse]
            element[Element]
            jitsi[Jitsi Meet]
        end
        
        subgraph apps[Apps]
            outline[Outline]
            vikunja[Vikunja]
            cryptpad[CryptPad]
            searxng[SearXNG]
        end
        
        subgraph games[Games]
            minecraft[Minecraft]
            playig[playit.gg]
        end
    end
    
    subgraph caspar[caspar]
        direction TB
        
        subgraph security[Security]
            zitadel[Zitadel]
            stalwart[Stalwart]
            zeek[Zeek]
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
        end
    end
    
    %% Added melchior to main servers
    subgraph melchior[melchior]
        tpot[T-Pot]
    end
end

%% Core connections between main servers
suricata --> zeek
zitadel --> outline
minio --> social & outline
element --> synapse --> jitsi
minecraft --> playig
prometheus --> grafana
tpot --> prometheus
uptime -.-> balthasar
uptime -.-> caspar
uptime -.-> melchior

%% Apply styles
class balthasar,caspar,melchior homeServer
class security,social,social_c,matrix,apps,games,CTF service
class monitoring monitoring
class security security
```
```mermaid
graph TB
%% Style definitions
classDef homeServer fill:#f1f5f9,stroke:#475569,stroke-width:2px
classDef service fill:#f8fafc,stroke:#64748b,stroke-width:1px
classDef monitoring fill:#ecfdf5,stroke:#047857,stroke-width:1px
classDef security fill:#fef2f2,stroke:#b91c1c,stroke-width:1px
classDef network fill:#f5f3ff,stroke:#6d28d9,stroke-width:2px
classDef common fill:#fffbeb,stroke:#d97706,stroke-width:1px,font-style:italic

%% Support Infrastructure with connections to main servers
subgraph support[Support Infrastructure]
    direction TB
    
    subgraph linode[Linode Servers]
        direction TB
        
        subgraph vpn[linode-vpn]
            algo[Algo VPN]:::security
            xray[Xray-core]:::security
            warp[WARP]:::security
        end
        
        subgraph proxy[linode-proxy]
            summaryproxy[Summary proxy]:::service
            mediaproxy[Media proxy]:::service
            squid[Squid]:::security
        end
        
        subgraph app_server[linode-app]
            impostor[Impostor]:::service
        end
    end
    
    %% Borg backup server
    subgraph raspi[raspi]
        borgbackup[Borg]:::service
    end
    
    %% Main server references with their social services
    subgraph main_servers[Main Servers References]
        direction LR
        
        subgraph balthasar[balthasar]
            yamisskey[Misskey]:::service
        end
        
        subgraph caspar[caspar]
            nayamisskey[Misskey N/A]:::service
        end
    end
end

%% Connections to proxies
yamisskey --> summaryproxy & mediaproxy
nayamisskey --> summaryproxy & mediaproxy

%% VPN traffic flow
balthasar & caspar --> algo --> xray --> warp

%% Backup connections
borgbackup -.-> balthasar:::connection
borgbackup -.-> caspar:::connection

%% Apply styles
class balthasar,caspar,vpn,proxy,app_server,raspi homeServer
class linode,main_servers network
```
