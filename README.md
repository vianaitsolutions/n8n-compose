# ⚙️ Implantação Self-Hosted de n8n e Waha com Docker Compose na AWS

Este projeto, criado pela **Viana IT Solutions**, detalha a configuração e implantação de um ambiente de automação robusto e de baixo custo, utilizando as ferramentas open-source n8n e Waha (WhatsApp HTTP API).

Toda a infraestrutura é orquestrada com Docker Compose e hospedada em uma instância EC2 na AWS, demonstrando uma solução eficiente e escalável para automação de processos.

## 🎯 Motivação

O principal motivador para este projeto foi a busca por uma solução de automação poderosa que oferecesse controle total sobre os dados e, ao mesmo tempo, uma redução significativa de custos em comparação com alternativas como o n8n Cloud ou a hospedagem em VPS tradicionais.

Ao utilizar a camada gratuita (Free Tier) da AWS e o poder do Docker, conseguimos um ambiente de alta performance com um custo operacional mínimo.

## 🛠️ Stack de Tecnologias

* 🐋 **Orquestração:** Docker & Docker Compose
* ☁️ **Cloud:** AWS (EC2, VPC, Security Groups, Elastic IP)
* 🔁 **Proxy Reverso:** Caddy (com automação de SSL/TLS)
* 🤖 **Automação:** n8n
* 💬 **Gateway WhatsApp:** Waha
* 🗃️ **Banco de Dados:** PostgreSQL & Redis
* 🌐 **DNS:** HostGator

## 🚀 Guia de Implantação: Passo a Passo

A seguir, os passos realizados para a construção da infraestrutura e implantação dos serviços.

### Fase 1: Configuração da Infraestrutura na AWS

1.  **Provisionamento da Instância EC2**: Foi utilizada uma instância `t2.micro` (elegível para o Free Tier da AWS) com o sistema operacional Ubuntu.

2.  **Configuração de Rede e Segurança**:
    * Uma **VPC** customizada foi criada para isolar os recursos.
    * **Regras de Inbound** no Security Group foram configuradas para permitir tráfego:
        * **SSH (porta 22):** Restrito ao IP do administrador para garantir o acesso seguro.
        * **HTTPS (porta 443):** Aberto ao público (`0.0.0.0/0`) para acesso às aplicações.
        * **HTTP (porta 80):** Aberto ao público (`0.0.0.0/0`), um pré-requisito para que o Caddy possa gerar os certificados SSL/TLS via desafio HTTP-01.

3.  **Endereço de IP Estático (Elastic IP)**: Um Elastic IP foi alocado e associado à instância EC2. Isso garante que o servidor sempre terá o mesmo endereço IP público, o que é essencial para a configuração do DNS.

### Fase 2: Configuração do DNS

Com o IP público estático em mãos, foram criados os seguintes registros do tipo `A` no painel de DNS da HostGator:
* `n8n.vianait.com` apontando para o Elastic IP da EC2.
* `waha.vianait.com` apontando para o mesmo Elastic IP.

### Fase 3: Implantação da Aplicação

1.  **Versionamento e Configuração**: Inicialmente, um repositório privado no GitHub foi utilizado para armazenar e versionar os arquivos de configuração (`docker-compose.yml`, `Caddyfile`).

2.  **Setup no Servidor**: Os arquivos foram clonados para dentro da instância EC2. O arquivo `.env`, contendo as variáveis de ambiente e segredos, foi criado a partir do `.env.example`.

3.  **Execução**: Com todas as configurações prontas, os serviços foram iniciados em modo `detached` com o comando:
    ```bash
    docker compose up -d
    ```

## 🐳 Entendendo os Serviços (Containers)

> O Docker foi o grande facilitador deste projeto, permitindo empacotar cada serviço com suas dependências em containers isolados, garantindo um ambiente limpo, reprodutível e fácil de gerenciar.

* **Caddy**: Atua como nosso proxy reverso. Ele recebe todo o tráfego das portas 80 e 443, direciona para os serviços corretos (n8n ou Waha) e, o mais importante, gerencia automaticamente a emissão e renovação dos certificados SSL/TLS, garantindo que nossas aplicações sejam servidas via HTTPS.

* **n8n**: É o coração da nossa automação. Uma poderosa plataforma de "workflow automation" que permite conectar diferentes APIs e serviços para criar processos automatizados complexos, tudo através de uma interface visual baseada em nós.

* **Waha**: Uma implementação robusta da API não-oficial do WhatsApp. Ele atua como uma ponte (gateway), permitindo que o n8n (ou qualquer outra aplicação) envie e receba mensagens do WhatsApp via requisições HTTP.

* **Redis**: É um banco de dados em memória de alta performance, utilizado pelo n8n para gerenciamento de filas e cache. Ele garante a execução eficiente dos workflows, especialmente em cenários com alto volume de execuções.

* **PostgreSQL**: É o banco de dados relacional principal do n8n. Ele é responsável por armazenar de forma persistente todos os dados críticos, como os workflows criados, as credenciais salvas e os históricos de execução. Seu uso é fundamental para a operação e a integridade dos dados no n8n.

---
*Este projeto foi desenvolvido por [Viana IT Solutions](vianait.com).*
