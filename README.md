# hosted-list

## Service Architecture & Deployment Overview

```mermaid
graph TB
    %% Style definitions
    classDef homeServer fill:#e2e8f0,stroke:#334155,stroke-width:2px
    classDef service fill:#f8fafc,stroke:#64748b,stroke-width:1px
    classDef rpi fill:#fde68a,stroke:#d97706,stroke-width:2px
    classDef proxy fill:#e0e7ff,stroke:#3730a3,stroke-width:2px
    classDef cloudflare fill:#f0fdfa,stroke:#0f766e,stroke-width:1.5px
    classDef security fill:#fee2e2,stroke:#991b1b,stroke-width:1px
    classDef storage fill:#dcfce7,stroke:#16a34a,stroke-width:2px
    classDef local fill:#86efac,stroke:#16a34a,stroke-width:3px
    classDef activitypubStyle fill:#f3e8ff,stroke:#7c3aed,stroke-width:2px
    classDef nostrStyle fill:#fed7aa,stroke:#ea580c,stroke-width:2px
    classDef matrixStyle fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px
    classDef appsStyle fill:#fef3c7,stroke:#ca8a04,stroke-width:2px
    
    %% Main Infrastructure
    subgraph main_servers[Main Servers]
        direction LR
        
        subgraph balthasar[balthasar - 本番環境]
            direction TB
            cloudflared_b[Cloudflared]:::cloudflare
            nginx_b[Nginx + ModSecurity<br/>Reverse Proxy]:::proxy
            
            subgraph activitypub[ActivityPub]
                yamisskey[Misskey]:::service
                neoquesdon[Neo-Quesdon]:::service
            end
            
            subgraph nostr[Nostr]
               nostream[Nostream]:::service
            end
            
            subgraph matrix[Matrix]
                synapse[Synapse]:::service
                element[Element]:::service
            end
            
            subgraph apps[Apps]
                outline[Outline]:::service
                cryptpad[CryptPad]:::service
            end
            
            subgraph auth_services[認証・セキュリティ]
                zitadel[Zitadel]:::security
                mcaptcha[mCaptcha]:::security
            end
            
            subgraph storage_local[Storage]
                minio[MinIO]:::storage
            end
        end
        
        subgraph caspar[caspar - 実験・テスト環境]
            direction TB
            cloudflared_c[Cloudflared]:::cloudflare
            nginx_c[Nginx + ModSecurity<br/>Reverse Proxy]:::proxy
            
            subgraph CTF[CTF]
                ctfd[CTFd]:::service
                vm[VM]:::service
            end
            
            subgraph social_c[Social - テスト]
                nayamisskey[Misskey N/A]:::service
            end
        end
        
        subgraph raspberrypi[raspberrypi - Minecraft専用<br/>NVMe SSD 2TB, 8GB RAM]
            direction TB
            playig[playit.gg]:::service
            
            subgraph games[Games]
                minecraft[Minecraft Java<br/>6GB RAM]:::service
            end
        end
        
        internet((Internet)):::cloudflare
    end
    
    %% Local storage connections (thick lines)
    yamisskey --> minio
    outline --> minio
    
    %% Local authentication connections (within balthasar)
    outline --> zitadel
    yamisskey --> mcaptcha
    
    %% Other core connections
    element --> synapse
    minecraft --> playig
    
    %% Cloudflared to Nginx connections
    cloudflared_b --> nginx_b
    cloudflared_c --> nginx_c
    
    %% Nginx to services - balthasar
    nginx_b --> yamisskey
    nginx_b --> neoquesdon
    nginx_b --> nostream
    nginx_b --> element
    nginx_b --> synapse
    nginx_b --> outline
    nginx_b --> cryptpad
    nginx_b --> zitadel
    nginx_b --> mcaptcha
    
    %% Nginx to services - caspar
    nginx_c --> nayamisskey
    nginx_c --> ctfd
    
    %% External connections
    playig --> internet
    cloudflared_b --> internet
    cloudflared_c --> internet
    
    %% Apply styles to subgraphs
    class balthasar,caspar homeServer
    class raspberrypi rpi
    class activitypub activitypubStyle
    class nostr nostrStyle
    class matrix matrixStyle
    class apps appsStyle
    class games,CTF,social_c service
    class auth_services security
    class cloudflared_b,cloudflared_c cloudflare
    class nginx_b,nginx_c proxy
    class storage_local storage
```

## Proxmox Virtualization Platform & Security Environment

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
               haproxy["HAProxy"]:::service
               openvpn["OpenVPN"]:::security
           end
           
           subgraph tpot_vm["T-Pot VM - 8c/24GB/200GB"]
               tpot["T-Pot 24.04+"]:::security
               cowrie["Cowrie SSH Honeypot"]:::security
               dionaea["Dionaea Multi-protocol"]:::security
               elasticpot["ElasticPot"]:::security
               kibana_tpot["Kibana Dashboard"]:::monitoring
           end
           
           subgraph malcolm_vm["Malcolm VM - 12c/32GB/500GB"]
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
    vmbr2 --> tpot_vm
    vmbr2 --> malcolm_vm
    vmbr3 --> pfsense_vm
    
    %% Storage connections
    local_lvm --> pfsense_vm
    local_lvm --> tpot_vm
    local_lvm --> malcolm_vm
    
    %% Service connections
    tpot --> kibana_tpot
    malcolm --> elasticsearch
    suricata_malcolm --> malcolm
    
    %% Apply styles
    class proxmox homeServer
    class pfsense_vm,tpot_vm,malcolm_vm homeServer
```

## Infrastructure as Code & Automation Systems

```mermaid
graph TB
    %% Style definitions
    classDef iac fill:#f0f9ff,stroke:#0369a1,stroke-width:2px
    classDef automation fill:#cffafe,stroke:#06b6d4,stroke-width:2px
    classDef homeServer fill:#e2e8f0,stroke:#334155,stroke-width:2px
    classDef alert fill:#fef3c7,stroke:#d97706,stroke-width:2px

    %% Source Control
    subgraph source["ソースコード管理"]
        git["Git Repository<br/>Infrastructure as Code"]:::iac
        actions["GitHub Actions<br/>CI/CD Pipeline"]:::automation
    end

    %% Control Plane
    subgraph caspar["caspar - 制御ハブ"]
        terraform["Terraform<br/>インフラ定義・プロビジョニング"]:::iac
        ansible["Ansible<br/>設定管理・デプロイ"]:::automation
        cloud_init["Cloud-init<br/>VM初期化"]:::automation
    end

    %% Managed Infrastructure
    subgraph infra["管理対象インフラ"]
        subgraph proxmox_infra["Proxmox (Terraform管理)"]
            proxmox_vms["Virtual Machines<br/>• pfSense (4c/8GB/32GB)<br/>• T-Pot (8c/24GB/200GB)<br/>• Malcolm (12c/32GB/500GB)"]:::homeServer
            proxmox_storage["Storage: local-lvm"]:::homeServer
            proxmox_network["Networks: vmbr0-3"]:::homeServer
        end
        physical["物理サーバー (Ansible管理)<br/>• balthasar<br/>• caspar"]:::homeServer
        truenas["TrueNAS (Ansible管理)<br/>ストレージ・バックアップ"]:::homeServer
    end

    %% Notifications
    slack["Slack<br/>デプロイ通知"]:::alert
    discord["Discord<br/>変更通知"]:::alert

    %% Workflow
    git --> actions
    actions --> terraform
    actions --> ansible
    
    terraform -->|VM作成・更新<br/>ストレージ割り当て<br/>ネットワーク設定| proxmox_infra
    ansible -->|設定適用| physical
    ansible -->|設定適用| truenas
    cloud_init -->|初期設定| proxmox_vms

    %% Notifications
    actions --> slack
    terraform --> discord
    ansible --> discord

    %% Apply styles
    class caspar homeServer
    class proxmox_infra homeServer
```

## Monitoring ＆ Alert System

```mermaid
graph TB
    %% Style definitions
    classDef monitoring fill:#d1fae5,stroke:#047857,stroke-width:2px
    classDef homeServer fill:#e2e8f0,stroke:#334155,stroke-width:2px
    classDef alert fill:#fef3c7,stroke:#d97706,stroke-width:2px
    classDef app fill:#fce7f3,stroke:#be185d,stroke-width:1.5px
    
    %% Monitoring Hub (caspar) - セルフホスト化
    subgraph caspar["🏛️ caspar - 監視ハブ (セルフホスト)"]
        prometheus["Prometheus Server<br/>9090<br/>メトリクス収集・保存"]:::monitoring
        grafana["Grafana<br/>3000<br/>ダッシュボード"]:::monitoring
        uptime["Uptime Kuma<br/>3009<br/>死活監視"]:::monitoring
        alertmgr["AlertManager<br/>9093<br/>通知管理"]:::alert
    end
    
    %% All Monitored Systems (consolidated)
    subgraph systems["監視対象システム"]
        balthasar_node["balthasar<br/>Node/cAdvisor<br/>Misskey/Outline/MinIO"]:::homeServer
        joseph_node["joseph<br/>Node Exporter<br/>TrueNAS SCALE"]:::homeServer
        raspberry_node["raspberrypi<br/>Node Exporter<br/>Minecraft"]:::homeServer
        linode_node["linode_prox<br/>Media Proxy/Summaly"]:::homeServer
    end
    
    %% Application Notifications
    subgraph app_notify["アプリケーション通知"]
        misskey_webhook["Misskey<br/>Webhook"]:::app
        backup_notify["バックアップ<br/>結果通知"]:::app
    end
    
    %% External Notifications
    discord["Discord"]:::alert
    slack["Slack"]:::alert
    
    %% Monitoring Flow (Full Prometheus)
    systems --> prometheus
    prometheus --> grafana
    
    %% Alert Flow
    uptime --> alertmgr
    prometheus -->|アラートルール| alertmgr
    
    alertmgr --> discord
    alertmgr --> slack
    
    %% Direct App Notifications
    misskey_webhook --> discord
    backup_notify --> discord
    
    %% Apply styles
    class caspar monitoring
    class systems homeServer
```

## Storage & Backup Strategy

```mermaid
graph TB
    %% Style definitions
    classDef server fill:#e2e8f0,stroke:#334155,stroke-width:2px
    classDef service fill:#f8fafc,stroke:#64748b,stroke-width:1px
    classDef backup fill:#dbeafe,stroke:#2563eb,stroke-width:1.5px
    classDef storage fill:#f3e8ff,stroke:#7e22ce,stroke-width:1.5px
    classDef beelink fill:#ffb88c,stroke:#ffffff,stroke-width:2px,color:#ffffff
    classDef cloud fill:#f0fdfa,stroke:#0f766e,stroke-width:1.5px
    classDef zfs fill:#4c1d95,stroke:#c4b5fd,stroke-width:2px,color:#ffffff
    classDef encrypted fill:#fee2e2,stroke:#991b1b,stroke-width:2px
    classDef local fill:#dcfce7,stroke:#16a34a,stroke-width:2px

    %% External storage
    subgraph external["外部ストレージ"]
        r2["Cloudflare R2<br/>日次DBダンプ<br/>週次システム同期<br/>(メディア除外)"]:::cloud
        filen["Filen E2E<br/>日次メディア差分バックアップ<br/>rclone暗号化<br/>月次システムアーカイブ"]:::encrypted
    end

    %% Internal network
    subgraph internal["ローカルネットワーク"]
        
        %% Beelink TrueNAS
        subgraph beelink_nas["Beelink ME mini - TrueNAS SCALE 24.10"]
            subgraph m2_slots["M.2 スロット構成 (6個)"]
                emmc["eMMC 64GB<br/>TrueNAS OS"]:::storage
                slot23["Slot2-3: 2TB NVMe×2<br/>ZFS Mirror Pool"]:::zfs
                slot456["Slot4-6: 拡張用<br/>将来RAID-Z2対応"]:::storage
            end
            
            subgraph truenas_services["TrueNAS Services (Docker統一)"]
                zfs_pool["ZFS Pool (Mirror)<br/>スナップショット<br/>圧縮・重複排除"]:::zfs
                backup_svc["Backup Services<br/>pg_dump scheduler<br/>rsync server<br/>rclone Filen sync"]:::backup
                node_exporter["Node Exporter (Docker)<br/>監視エージェント"]:::service
            end
            
            dual_lan["デュアル2.5G LAN<br/>LAN1: メイン<br/>LAN2: 管理用"]:::beelink
        end
        
        %% Main servers
        subgraph servers["サーバー群"]
            subgraph balthasar["balthasar 本番"]
                misskey1["Misskey"]:::service
                db1["PostgreSQL DB"]:::service
                minio_local["MinIO<br/>オブジェクトストレージ<br/2TB"]:::local
                backup1["Backup Agent<br/>pg_dump + rsync"]:::backup
            end
        end
    end

    %% Service connections - ローカル接続
    misskey1 --> minio_local
    misskey1 --> db1

    %% MinIO → Filen 日次差分バックアップ (rsync経由)
    minio_local -.->|"rsync over SSH<br/>2.5G LAN"| backup_svc
    backup_svc ==>|"日次差分バックアップ<br/>rclone sync<br/>暗号化転送<br/>5-15分/日"| filen

    %% DB Backup flows - 既存維持
    db1 -.->|"日次DBダンプ<br/>2.5G LAN<br/>高速転送"| backup_svc
    db1 -.->|"日次DBダンプ<br/>直接R2"| r2
    
    %% TrueNAS internal flows
    backup_svc --> zfs_pool
    slot23 --> zfs_pool
    emmc --> truenas_services
    
    %% ZFS snapshots and replication
    zfs_pool -.->|"ZFS Send/Receive<br/>週次差分同期"| r2
    zfs_pool -.->|"ZFS スナップショット<br/>時系列バックアップ"| slot23
    
    %% System backup flows
    backup1 -.->|"システムバックアップ<br/>rsync over SSH"| backup_svc
    minio_local -.->|"メディアバックアップ<br/>rsync同期"| backup_svc
    
    %% Monitoring flows
    node_exporter -.->|"システム監視<br/>メトリクス送信"| zfs_pool
    backup_svc -.->|"バックアップ成功率<br/>転送統計<br/>暗号化検証"| node_exporter
    
    %% External sync flows  
    backup_svc -.->|"システム月次アーカイブ<br/>設定・ログ"| filen

    %% Apply styles
    class balthasar server
    class beelink_nas beelink
    class misskey1,db1,node_exporter service
    class backup_svc,backup1 backup
    class r2 cloud
    class filen encrypted
    class minio_local local
    class slot456,slot23,zfs_pool storage
    class emmc storage
```

## Network Traffic Flow & Proxy Configuration

```mermaid
graph TB
classDef homeServer fill:#e2e8f0,stroke:#334155,stroke-width:2px
classDef service fill:#f8fafc,stroke:#64748b,stroke-width:1px
classDef security fill:#fee2e2,stroke:#991b1b,stroke-width:1px
classDef cloudflare fill:#f0fdfa,stroke:#0f766e,stroke-width:1.5px
classDef user fill:#fef9c3,stroke:#ca8a04,stroke-width:1.5px
classDef federation fill:#f3e8ff,stroke:#7c3aed,stroke-width:1.5px
classDef direct fill:#dcfce7,stroke:#16a34a,stroke-width:2px
classDef tailscale fill:#fef3c7,stroke:#d97706,stroke-width:2px
classDef local fill:#dcfce7,stroke:#16a34a,stroke-width:3px

%% External actors
enduser([エンドユーザー<br/>Webブラウザ]):::user
external_servers([外部サーバー<br/>（他Misskey・画像・API等）]):::federation
bypass_services([DeepL API<br/>reCAPTCHA<br/>hCaptcha<br/>Cloudflare Challenges]):::direct

subgraph support[Support Infrastructure]
    direction TB
    
    subgraph linode[Linode Servers]
        direction TB
        subgraph proxy[linode-proxy]
            summaryproxy[Summary proxy<br/>独自IP]:::direct
            mediaproxy[Media proxy<br/>独自IP]:::direct
            squid[Squid プロキシ<br/>🔗 Tailscale ACL制限]:::tailscale
            warp[Cloudflare WARP<br/>drive.yami.ski除外]:::cloudflare
            cloudflared_p[Cloudflared]:::cloudflare
        end
    end
    
    subgraph homeservers[🏠 自宅サーバー群]
        direction TB
        subgraph balthasar_caspar[balthasar/caspar]
            nginx_misskey[Nginx + ModSecurity<br/>WAF・Reverse Proxy]:::security
            yamisskey[Misskey<br/>🔗 Tailscale接続]:::tailscale
            minio_local[MinIO<br/>オブジェクトストレージ]:::local
            cloudflared_bc[Cloudflared]:::cloudflare
        end
    end
end

%% エンドユーザーのアクセス経路（太線）
enduser ==>|"①Web UI アクセス"| cloudflared_bc
cloudflared_bc ==> nginx_misskey
nginx_misskey ==> yamisskey

%% 外部サーバーからの連合リクエスト（通常線）
external_servers -->|"②連合リクエスト"| cloudflared_bc

%% Misskeyからのローカル接続（太線・緑色）
yamisskey ==>|"③ローカル接続<br/>高速・低レイテンシ"| minio_local

%% Misskeyサーバーからの全外部通信はSquid経由
yamisskey ==>|"④🔗 Tailscale経由<br/>全外部通信"| squid
squid --> warp

%% WARPからの分岐
warp -->|"外部サーバーへ"| external_servers
warp ==>|"SummaryProxy<br/>アクセス"| cloudflared_p
warp -->|"外部URL情報取得"| summaryproxy
cloudflared_p ==> mediaproxy
cloudflared_p -.-> summaryproxy
squid ==>|"MediaProxy<br/>アクセス"| cloudflared_p

%% MediaProxyからMisskeyへ画像処理結果を返却
mediaproxy ==>|"⑤画像取得/変換結果<br/>返却"| cloudflared_bc

%% SummaryProxyからの返却
summaryproxy -.->|"⑥URL情報取得結果<br/>返却"| cloudflared_bc

%% プロキシバイパス対象（特定APIサービス）
yamisskey -.->|"プロキシバイパス<br/>API直接アクセス"| bypass_services
bypass_services -.->|"API結果返却<br/>（翻訳・CAPTCHA等）"| yamisskey
```
