# OpenBatch

Uma interface web segura e acessível para gerenciamento de jobs em clusters de Computação de Alto Desempenho (HPC) que utilizam **SLURM** em ambientes de nuvem privada.

O OpenBatch visa abstrair a complexidade da linha de comando, buscando oferecer um portal visual para pesquisadores e administradores submeterem cargas de trabalho, monitorarem recursos e gerenciarem arquivos.

---

## Arquitetura

O sistema segue uma arquitetura de três camadas, projetada para segurança e escalabilidade:

1.  **Frontend:** Desenvolvido em **React**. Comunica-se com o backend via API REST e WebSockets. O código roda no navegador do cliente e ajusta dinamicamente as conexões para o host correto.
2.  **Backend:** Desenvolvido em **Node.js**.
      - Atua como um *Gateway* seguro entre a web e o cluster.
      - Gerencia autenticação via **PAM** (integrado aos usuários Linux do sistema).
      - Executa comandos privilegiados com transição de contexto (`su`/`runuser`) para garantir que cada ação ocorra sob o UID/GID do usuário logado.
      - Utiliza `node-pty` para emulação de terminal via WebSocket.
3.  **Infraestrutura:**
      * **Gerenciador:** SLURM Workload Manager.
      * **Autenticação:** Munge.
      * **Isolamento:** Cgroups (v2) e Limites de Processos do Kernel.

---

## Funcionalidades

  - **Autenticação Integrada:** Login utilizando as credenciais de sistema do cluster (Linux/PAM/LDAP).
  - **Terminal Web:** Acesso total ao shell do usuário (`bash`) diretamente no navegador via [xterm.js](https://github.com/xtermjs/xterm.js).
  - **Monitoramento em Tempo Real:** Visualização de jobs (squeue), nós e recursos do cluster.
  - **Gestão de Módulos:** Upload seguro de arquivos `.zip` com varredura automática de **antivírus (ClamAV)** e extração automática no diretório do usuário.
  - **Submissão de Jobs:** Interface gráfica para criação de scripts `sbatch`.

-----

## Deploy Automatizado

Para realizar o deploy completo deste projeto em um ambiente de produção ou cluster, utilizamos o **Ansible**.
Os scripts de infraestrutura como código foram separados para um repositório dedicado. 

Por favor, acesse o repositório [openbatch-setup](https://github.com/openbatch-project/openbatch-setup) para instruções completas de como provisionar o ambiente.

-----

## Desenvolvimento e Execução Manual

Para rodar o projeto localmente ou desenvolver novas features.

### 1\. Pré-requisitos Locais

  - Node.js (v18+)
  - Compiladores C++ (`build-essential` ou equivalente)
  - Headers PAM (`libpam0g-dev`)

### 2\. Instalação

O projeto está dividido em duas partes: `frontend` e `backend`. Você precisará instalar as dependências de ambos.

Clone o repositório:
```bash
git clone https://github.com/marcusmartinss/openbatch.git
cd openbatch
```

Instale as dependências do Frontend:
```bash
cd frontend
npm install
```

Instale as dependências do Backend:
```bash
cd ../backend
npm install
```

### 3\. Execução

#### Compilar o Frontend

Gera os arquivos estáticos na pasta `dist/` do frontend para que o backend possa servi-los.

```bash
cd ../frontend
npm run build
```

*(Durante o desenvolvimento contínuo, você também pode usar `npm run dev` na pasta frontend).*

#### Iniciar o Backend

Para que a autenticação PAM e a criação de terminais funcionem plenamente, o backend deve ter permissões elevadas (ou rodar como root).

```bash
cd ../backend
sudo npm start
```

O terminal exibirá:

> `Servidor OpenBatch rodando em http://localhost:8080`

Acesse em seu navegador: **http://localhost:8080**

-----

## Segurança

O projeto segue práticas de **DevSecOps** e _Hardening_:

  - **Anti-Malware:** Todos os uploads passam por scan do ClamAV antes de serem processados.
  - **Validação de Input:** Verificação contra ataques "Zip Slip" e bloqueio de extensões executáveis perigosas.
  - **Proteção de Recursos:** Configuração de `limits.conf` para evitar _Fork Bombs_.
  - **Isolamento:** Jobs do Slurm rodam dentro de Cgroups dedicados.
