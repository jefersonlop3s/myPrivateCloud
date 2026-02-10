# ☁️ myPrivateCloud (Legacy v1.0.0)

![Status](https://img.shields.io/badge/Status-Archived-red)
![Version](https://img.shields.io/badge/Release-v1.0.0-blue)
![Tech](https://img.shields.io/badge/Infrastructure-Kubernetes%20%7C%20Vagrant%20%7C%20Docker-green)

Este repositório contém os manifestos de infraestrutura, configurações de orquestração e automação de uma Nuvem Privada (HomeLab). Este projeto serviu como laboratório para estudos avançados em **Kubernetes**, **Identity Management** e **Armazenamento Distribuído**.

> ⚠️ **Nota:** Este repositório é um snapshot de uma implementação estável e foi arquivado para fins de portfólio e consulta histórica.

---

## 🏛️ Arquitetura do Lab

A infraestrutura foi desenhada para modularidade, separando a camada de orquestração (Kubernetes) dos serviços essenciais e aplicações finais.

### 1. Provisionamento & Camada Base
* **Vagrant:** Ambiente de virtualização reproduzível (`/vagrant`) utilizado para subir o cluster e nós de teste.
* **Storage (NFS):** Implementação de `NFS-Server` via DaemonSet no Kubernetes para persistência de dados (PV/PVCs).

### 2. Kubernetes Core (Cluster System)
Componentes fundamentais para o funcionamento da rede e segurança do cluster:
* **Ingress Controller:** Nginx (v1.11.12 e v1.9.3) para roteamento de tráfego HTTP/HTTPS.
* **Load Balancer:** MetalLB (v0.13.11) para atribuição de IPs reais (Layer 2) aos serviços do cluster.
* **Cert-Manager:** Gestão automatizada de certificados (Self-Signed).

### 3. Essentials (Middleware & Backing Services)
Serviços de suporte que sustentam as aplicações:
* **Identity Management:** FreeIPA containerizado para gestão centralizada de usuários e políticas.
* **Databases:**
    * `MariaDB` & `PostgreSQL`: Bancos relacionais com persistência configurada via Secrets e PVCs.
    * `Redis`: Camada de cache.

### 4. Aplicações (End-User)
* **Code-Server:** Ambiente de desenvolvimento (IDE) remoto rodando sobre Kubernetes.
* **Nextcloud:** Nuvem de arquivos e colaboração (Implementação híbrida via Docker Compose + Nginx Proxy).
* **Nginx-Test:** Pods para validação de rotas de Ingress e Troubleshooting.

---

## 📂 Estrutura do Repositório

```text
.
├── apps/               # Aplicações finais (Manifestos K8s & Compose)
│   ├── code-server     # IDE Remota
│   ├── nextcloud       # File Storage
│   └── ...
├── essentials/         # Serviços de infraestrutura (DBs, Auth, Storage)
│   ├── freeipa
│   ├── mariadb / postgresql / redis
│   └── nfs-server
├── kubernetes/         # Componentes do Cluster (Ingress, MetalLB)
└── vagrant/            # IaaC para provisionamento de VMs
```
---

## 🚀 Destaques Técnicos

Segurança: Uso extensivo de Secret e ConfigMap para desacoplar configurações sensíveis.

Persistência: Definição clara de PersistentVolume (PV) e PersistentVolumeClaim (PVC) para todos os serviços stateful.

Networking: Configuração avançada de Ingress e MetalLB IPAddressPools.

Desenvolvido por Jeferson Lopes, Software Engineer | DevOps | Enthusiast

---
