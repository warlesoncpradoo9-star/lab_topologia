# 🐝 Laboratório com Containerlab

> Laboratório prático de **Observação de Protocolo** usando um ambiente de rede virtualizado com **Containerlab**.

[![Containerlab](https://img.shields.io/badge/Containerlab-v0.50+-blue?logo=linux)](https://containerlab.dev)
[![Docker](https://img.shields.io/badge/Docker-required-blue?logo=docker)](https://www.docker.com)
[![Licença](https://img.shields.io/badge/licença-GPL--2.0-green)](LICENSE)

---

## 📖 Visão Geral

Este laboratório utiliza o Containerlab como orquestrador de container para observação de protocolos e testes de segurança de redes.

**O que este laboratório demonstra:**
- Deploy de uma rede virtual com 2 nós usando Containerlab.
- Ataque DDos com hping3.
- Leitura de desempenho com iperf.
---

## Topologia

```
┌─────────────────────────────────────────┐
│               Máquina Host              │
│                                         │
│  ┌──────────┐ eth1   eth1 ┌──────────┐  │
│  │  node-a  ├─────────────┤  node-b  │  │
│  │10.0.0.1  │             │10.0.0.2  │  │
│  └──────────┘             └──────────┘  │
│    (emissor)                            │
└─────────────────────────────────────────┘
```
- node-a: Máquina Linux usando a imagem nicolaka/netshoot (distro focada em ferramentas de rede).
- node-b: Máquina Linux usando a imagem nicolaka/netshoot (distro focada em ferramentas de rede).


| Nó     | Endereço IP  | Função                                      |
|--------|-------------|---------------------------------------------|
| node-a | `10.0.0.1`  | Emissor de pacotes (origem do ping)         |
| node-b | `10.0.0.2`  | Emissor de pacotes (origem do ping)         |

---

## 🔧 Pré-requisitos


### 0. Requisitos do Sistema

Os seguintes requisitos devem ser atendidos para que a ferramenta containerlab seja executada com sucesso (https://containerlab.dev/install/):

- Um usuário com privilégios de sudo para executar o containerlab.

- Um servidor Linux, pode ser WSL2 (https://learn.microsoft.com/pt-br/windows/wsl/install).

### 1. Instalar o Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

> Saia e entre novamente na sessão após adicionar seu usuário ao grupo `docker`.

### 2. Instalar o Containerlab

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

Verifique a instalação:

```bash
containerlab version
```

---

## ⏬ Obtendo o Laboratório

Clone o repositório e acesse o diretório do laboratório:

```bash
git clone https://github.com/seunomegit/lab.git
cd lab
```

## 🐝 Passo 2 — Deploy da Topologia

```bash
sudo containerlab deploy -t lab.clab.yml --reconfigure
```

Isso irá:
- Criar dois containers Linux (`node-a` e `node-b`) com a imagem `nicolaka/netshoot`.
- Configurar os IPs nas interfaces `eth1` de cada nó.
- Criar um link virtual direto entre as interfaces `eth1` dos dois nós.

Verifique se o lab está rodando:

```bash
docker ps --filter "label=containerlab=lab"
```

---

## 🐝 Passo 3 — Verificar Conectividade Inicial


```bash
docker exec clab-lab-node-a ping -c 3 10.0.0.2
```

**Resultado esperado:** `0% packet loss`  

---




## 📚 Referências

- [Documentação Oficial do eBPF](https://ebpf.io/what-is-ebpf/)
- [Documentação do Containerlab](https://containerlab.dev/quickstart/)
- [Tutorial XDP (kernel.org)](https://github.com/xdp-project/xdp-tutorial)
- [libbpf GitHub](https://github.com/libbpf/libbpf)
- [nicolaka/netshoot — Container de diagnóstico de rede](https://github.com/nicolaka/netshoot)
