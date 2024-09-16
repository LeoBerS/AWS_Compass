# AWS_Compass
Requisitos AWS e Linux
# Guia para Configuração de Instância EC2 na AWS

Este guia descreve como criar uma instância EC2 com a AMI Amazon Linux 2, gerar um Elastic IP, anexá-lo à instância e liberar portas de comunicação para acesso público. Além, de como gerar uma chave pública para acesso ao ambiente.

## Criando uma Instância EC2 com AMI Amazon Linux 2

### Pré-requisitos

- Uma conta AWS ativa.
- Acesso ao console do Amazon EC2.

### Passos para Criar uma Instância EC2

1. **Acesse o Console do Amazon EC2**:
   - Abra o console do Amazon EC2 em https://console.aws.amazon.com/ec2/.

2. **Inicie o Processo de Criação da Instância**:
   - No painel de navegação à esquerda, clique em "Instances" e depois em "Launch Instances".

3. **Adicionar Tags**:
   - Clique em "Add Tags" e adicione tags conforme necessário para organizar suas instâncias.

4. **Escolha a AMI**:
   - Em "Application and OS Images (Amazon Machine Image)", selecione "Amazon Linux 2 AMI (HVM), SSD Volume Type"

5. **Escolha o Tipo de Instância**:
   - Selecione "t3.small".

6. **Selecionar ou Criar um Par de Chaves**:
    - Selecione um par de chaves existente ou crie um novo.
    - Baixe o arquivo da chave privada (.pem) e salve-o em um local seguro.

7.  **Configurações de rede**:
   - VPC: use o sugerido ou escolha um da lista.
   - Sub-rede: selecione um da lista ou clique em Criar novasub-rede.
   - Atribuir IP público automaticamente: selecione da lista o Habilitar.
   - Em "Firewall (grupos de segurança):
   - - Crie um novo grupo de segurança ou selecione um existente.
   - Adicione regras para permitir o tráfego SSH (porta 22) de seu endereço IP:
   -- Tipo de origem: apenas para este exercício selecione "Qualquer lugar".

8. **Configurar Armazenamento**:
   - Ajuste o tamanho do armazenamento para 16 GB e certifique-se de que o tipo de volume seja "General Purpose SSD (gp2)".

9. **Revisar e Lançar**:
   - Revise todas as configurações na seção Resumo.
   - Se tudo estiver certo clique no botão "Launch Instances"

## Gerando e Anexando um Elastic IP

### Passos para Gerar e Anexar um Elastic IP

1. **Acesse o Console do Amazon EC2**:
   - Abra o console do Amazon EC2 em https://console.aws.amazon.com/ec2/.

2. **Navegue até Elastic IPs**:
   - No painel de navegação à esquerda, em "Network & Security", clique em "Elastic IPs".

3. **Alocar um Novo Elastic IP**:
   - Clique em "Allocate Elastic IP address".
   - Escolha a opção padrão, Insira Tags  e clique em "Allocate".

4. **Anotar o Elastic IP**:
   - Após a alocação, anote o Elastic IP gerado.

5. **Associar o Elastic IP à Instância EC2**:
   - Selecione o Elastic IP recém-criado na lista.
   - Clique em "Actions" e depois em "Associate Elastic IP address".
   - Na seção "Instance", selecione a instância EC2 que você criou anteriormente.
   - Escolha o endereço IP privado da instância (geralmente `eth0`).
   - Clique em "Associate".

## Liberando Portas de Comunicação para Acesso Público

### Passos para Liberar as Portas

1. **Acesse o Console do Amazon EC2**:
   - Abra o console do Amazon EC2 em https://console.aws.amazon.com/ec2/.

2. **Navegue até Security Groups**:
   - No painel de navegação à esquerda, em "Network & Security", clique em "Security Groups".

3. **Selecione o Grupo de Segurança**:
   - Selecione o grupo de segurança associado à sua instância EC2.

4. **Adicione Regras de Entrada**:
   - Clique na aba "Inbound rules" e depois em "Edit inbound rules".
   - Adicione as seguintes regras para liberar as portas necessárias:

     | Tipo       | Protocolo | Porta   | Origem       |
     |------------|-----------|---------|--------------|
     | SSH        | TCP       | 22      | 0.0.0.0/0    |
     | Custom TCP | TCP       | 111     | 0.0.0.0/0    |
     | Custom UDP | UDP       | 111     | 0.0.0.0/0    |
     | Custom TCP | TCP       | 2049    | 0.0.0.0/0    |
     | Custom UDP | UDP       | 2049    | 0.0.0.0/0    |
     | HTTP       | TCP       | 80      | 0.0.0.0/0    |
     | HTTPS      | TCP       | 443     | 0.0.0.0/0    |

   - Clique em "Save rules" para aplicar as mudanças.

## Conectando-se à Instância EC2

Após a instância ser lançada e as portas liberadas, você pode se conectar ao terminal Linux ao clicar no ID da instância e, na tela que se segue, no botão "Conectar":

1.  usando a aba "Conexão de instância do EC2", clicar no botão "Conectar" para se conectar ao terminal da sua instância EC2.


# Guia Completo para Configuração de Instância EC2 com NFS e Apache

Este guia descreve como configurar o NFS, criar um diretório, subir um servidor Apache, criar um script de validação e automatizar sua execução a cada cinco minutos em uma instância EC2 na AWS.

## Configurando o NFS

### Passos para Configurar o NFS

1. **Instale o NFS Server**:
   - No terminal da sua instância EC2, execute:
     ```
     sudo yum update -y
     ```
   - Caso tenha sido atualizado, para instalar execute:
     ```
     sudo yum install nfs-utils -y
     ```

2. **Crie o Diretório para o NFS**:
   - Crie um diretório para compartilhar via NFS:
     ```
     sudo mkdir -p /mnt/nfs_share
     ```

3. **Configure o NFS Exports**:
   - Edite o arquivo `/etc/exports` para adicionar o diretório que será compartilhado:
     ```
     sudo nano /etc/exports
     ```
   - Adicione a seguinte linha ao arquivo:
     ```
     /mnt/nfs_share *(rw,sync,no_subtree_check)
     ```

4. **Reinicie o Serviço NFS**:
   - Execute os seguintes comandos para aplicar as mudanças:
     ```
     sudo exportfs -a
     sudo systemctl restart nfs-server
     sudo systemctl enable nfs-server
     ```
5. **Verifique o status do NFS**:
   - Execute os seguintes comandos para aplicar as mudanças:
     ```
     sudo systemctl status nfs-server
     ```

## Criando um Diretório com seu nome no Filesystem do NFS

1. **Crie o Diretório "leonardo", por exemplo**:
   - No terminal da sua instância EC2, execute:
     ```
     sudo mkdir -p /mnt/nfs_share/leonardo
     ```
2. **Configure as permissões do diretório para acesso além do root:**:
   - No terminal da sua instância EC2, execute:
     ```
     sudo chown -R ec2-user:ec2-user /mnt/nfs_share/leonardo
     sudo chmod 755 /mnt/nfs_share/leonardo
     ```

## Subindo um Servidor Apache

### Passos para Instalar e Configurar o Apache

1. **Instale o Apache**:
   - No terminal da sua instância EC2, execute:
     ```
     sudo yum install httpd -y
     ```

2. **Inicie e Habilite o Apache**:
   - Execute os seguintes comandos para iniciar e habilitar o Apache:
     ```
     sudo systemctl start httpd
     sudo systemctl enable httpd
     ```

3. **Verifique o Status do Apache**:
   - Certifique-se de que o Apache está rodando:
     ```
     sudo systemctl status httpd
     ```

4. **Abra um navegador e digite o IP público da instância EC2 para visualizar a página do Apache**.

## Criando um Script de Validação do Serviço

### Passos para Criar o Script

1. **Crie o Script de Validação**:
   - No terminal da sua instância EC2, crie um arquivo de script:
     ```
     sudo nano /usr/local/bin/validaApache.sh
     ```
   - Adicione o seguinte conteúdo ao arquivo:
     ```
     #!/bin/bash

     TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
     SERVICE="httpd"
     STATUS=$(systemctl is-active $SERVICE)
     DIRECTORY="/mnt/nfs_share/leonardo"

     if [ "$STATUS" = "active" ]; then
         MESSAGE="ONLINE"
         echo "$TIMESTAMP - $SERVICE - $STATUS - $MESSAGE" > $DIRECTORY/apache_status_online.txt
     else
         MESSAGE="OFFLINE"
         echo "$TIMESTAMP - $SERVICE - $STATUS - $MESSAGE" > $DIRECTORY/apache_status_offline.txt
     fi
     ```

2. **Torne o Script Executável**:
   - Execute o seguinte comando para tornar o script executável:
     ```
     sudo chmod +x /usr/local/bin/validaApache.sh
     ```

## Automatizando a Execução do Script

### Passos para Automatizar a Execução a Cada Cinco Minutos

1. **Edite o Crontab**:
   - No terminal da sua instância EC2, execute:
     ```
     sudo nano /etc/crontab
     ```
   - Adicione a seguinte linha ao arquivo crontab para executar o script a cada cinco minutos:
     ```
     */5 * * * * root /usr/local/bin/validaApache.sh
     ```

## Verificando a Configuração

Após seguir todos os passos, o script será executado automaticamente a cada cinco minutos, verificando o status do serviço Apache e salvando os resultados no diretório `/mnt/nfs_share/leonardo`.

**Nota - execute o script manualmente se necessário**:
   - No terminal da sua instância EC2, execute:
     ```
     ./validaApache
     ```
