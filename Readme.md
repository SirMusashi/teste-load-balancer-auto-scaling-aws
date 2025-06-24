# Projeto de Infraestrutura Web com Auto Scaling na AWS

![AWS](https://img.shields.io/badge/AWS-%23232F3E.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![EC2](https://img.shields.io/badge/EC2-orange?style=for-the-badge&logo=amazonaws&logoColor=white)
![Shell Script](https://img.shields.io/badge/shell_script-%23121011.svg?style=for-the-badge&logo=gnu-bash&logoColor=white)

Este projeto demonstra a criação de uma infraestrutura web resiliente e escalável na AWS. Utilizando um Classic Load Balancer para distribuir tráfego e um Auto Scaling Group para gerenciar instâncias EC2, a arquitetura se adapta automaticamente à demanda, garantindo alta disponibilidade e otimização de custos.

## Arquitetura Final
A arquitetura implementada direciona o tráfego do usuário através de um Load Balancer, que o distribui para um grupo de instâncias EC2. Este grupo é gerenciado por um Auto Scaling Group, que monitora a carga e ajusta o número de instâncias ativas.

**Fluxo de uma Requisição**:

``Usuário`` → ``Internet`` → ``DNS do Load Balancer`` → ``Classic Load Balancer (CLB)`` → ``Instância EC2 Saudável (EC2 Instance)``

## Componentes Principais:

* **VPC (Virtual Private Cloud)** : A rede isolada onde todos os recursos foram criados.

* **Classic Load Balancer (CLB)** : O ponto de entrada único que distribui o tráfego.

* **Auto Scaling Group (ASG)** : O cérebro que automatiza a criação e destruição de instâncias.

* **Instâncias EC2 (EC2 Instances)** : Os servidores que rodam a aplicação web.

* **CloudWatch Alarms** : Os gatilhos que monitoram as métricas e acionam o ASG.

### 1. Criando a VPC
**Caminho** : ``VPC`` > ``Suas VPCs `` > ``Criar VPC ``

**Nome** : projeto-autoscaling

**Configuração** :

    * Criada com 2 Zonas de Disponibilidade (Availability Zones) para alta disponibilidade.

    * 2 Sub-redes Públicas, uma em cada Zona de Disponibilidade.

    * Um Internet Gateway anexado à VPC para permitir comunicação com a internet.

    * Tabela de Rotas configurada para direcionar o tráfego das sub-redes para o Internet Gateway.



![alt text](/imagens/image-2.png)

![alt text](/imagens/image-3.png)

![alt text](/imagens/image-4.png)

### 2. Habilitando um IP público
**Caminho** : ``VPC`` > ``Sub-redes`` > ``Ações`` > ``Editar configurações da sub-rede`` > ``Habilitar a atribuição automática de IPs públicos IPv4`` para ambas.

**Configuração Crítica**: As sub-redes públicas foram configuradas para atribuir um endereço IP público automaticamente a qualquer instância lançada nelas.

![alt text](/imagens/image-5.png)

![alt text](/imagens/image-6.png)


### 3. Criando um grupo de segurança
**Caminho** : ``EC2`` > ``Grupos de segurança`` > ``Criar grupo de segurança``

**Security Group (sg-clb-http)** : Permite tráfego de entrada na porta 80 de qualquer lugar (``0.0.0.0/0``).

![alt text](/imagens/image-7.png)

### 4. Criando um Load Balancer
**Caminho** : ``EC2`` > ``Load Balancers`` > ``Criar load balancer`` > ``Classic Load Balancer``

Responsável por balancear a carga entre as instâncias.

**Nome** : meu-classic-lb

**Tipo** : Classic

**Listener** : Configurado para receber tráfego HTTP na porta 80.

**Sub-redes** : Associado às duas sub-redes públicas da nossa VPC.

![alt text](/imagens/image.png)

![alt text](/imagens/image-1.png)

![alt text](/imagens/image-8.png)

![alt text](/imagens/image-9.png)

![alt text](/imagens/image-10.png)


### 5. Criando um Launch Template

**Caminho** : ``EC2`` > ``Modelos de Execução`` > ``Criar Modelo de execução``

Configuradas através de um Launch Template para garantir a padronização.

* Launch Template: meu-template-app

* AMI (Amazon Machine Image): Amazon Linux 2

* Tipo de Instância: t2.micro (Free Tier)

* Security Group (sg-instancias-asg):

* Permite tráfego SSH (porta 22) a partir de um IP específico para gerenciamento.

* Permite tráfego HTTP (porta 80) exclusivamente a partir do Security Group do Load Balancer (sg-clb-http).

![alt text](/imagens/image-11.png)

![alt text](/imagens/image-12.png)

![alt text](/imagens/image-13.png)

![alt text](/imagens/image-14.png)

![alt text](/imagens/image-15.png)

![alt text](/imagens/image-16.png)

**User Data** : Script de inicialização que executa na primeira vez que a instância é ligada para:

* Instalar e iniciar o servidor web Apache (httpd).

* Criar uma página index.html.

* Criar um endpoint de teste em /teste que simula uma carga de 5 segundos.

```Bash
#!/bin/bash
yum update -y
yum install -y httpd
chkconfig httpd on
service httpd start
echo "Hello World from $(hostname -f)" > /var/www/html/index.html

cat <<EOL > /var/www/cgi-bin/teste
#!/bin/bash
echo "Content-Type: text/plain"
echo ""
echo "Requisição recebida com sucesso na instância: $(hostname)"
sleep 5
EOL

chmod +x /var/www/cgi-bin/teste

sed -i 's/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf
echo 'ScriptAlias /teste /var/www/cgi-bin/teste' > /etc/httpd/conf.d/cgi.conf
service httpd restart
```


### 6. Criando O Auto Scaling Group
**Caminho** : ``EC2`` > ``Grupos Auto Scaling `` > ``Criar grupo do Auto Scaling``

O componente central que automatiza a escalabilidade.

**Configuração de Grupo** :

* Capacidade Mínima: 1 instância.

* Capacidade Desejada: 1 instância.

* Capacidade Máxima: 3 instâncias.

**Rede** : Configurado para lançar instâncias nas duas sub-redes públicas.

**Associação** : Ligado diretamente ao Classic Load Balancer ``meu-classic-lb``.


![alt text](/imagens/image-17.png)

![alt text](/imagens/image-18.png)

![alt text](/imagens/image-19.png)

![alt text](/imagens/image-20.png)

![alt text](/imagens/image-21.png)

![alt text](/imagens/image-22.png)

![alt text](/imagens/image-23.png)

### 7. Conferindo

``EC2``

![alt text](/imagens/image-24.png)

``LOAD BALANCER``

![alt text](/imagens/image-25.png)

### 8. Testando o endpoint

``COPIANDO O DNS``

![alt text](/imagens/image-26.png)

``DNS NO NAVEGADOR``

![alt text](/imagens/image-27.png)

``IMPORTANTE`` : Delay de 5 segundos: A página não vai carregar imediatamente. Haverá uma espera de exatamente 5 segundos. Isso é proposital, causado pelo comando sleep 5 no nosso script para simular uma tarefa pesada. 

### 9. Configurando as Regras de Escalonamento Automático

CAMINHO: ``EC2`` > ``Auto Scaling `` > ``MEU GRUPO DE AUTO SCALING`` > ``Criar política de escalibilidade dinâmica``

![alt text](/imagens/image-28.png)

As regras que governam o comportamento do ASG.

* **Política de Aumento (Scale-Out)** :

    * **Tipo** : Escalabilidade Simples

    * **Gatilho** : O alarme do CloudWatch ``Alarme-Carga-Alta-ASG`` .

    * **Condição do Alarme** : Dispara quando a métrica customizada (``RequestCount / HealthyHostCount``) fica acima de 10 por 1 minuto.

    * **Ação** : ``Adicionar`` 1 nova instância.


![alt text](/imagens/image-30.png)

![alt text](/imagens/image-31.png)

* **Política de Redução (Scale-In)** :

    * **Tipo** : Escalabilidade Simples

    * **Gatilho** : O alarme do CloudWatch ``Alarme-Carga-Baixa-ASG``.

    * **Condição do Alarme** : Dispara quando a métrica customizada (``RequestCount / HealthyHostCount``) fica **abaixo** de 5 por 3 períodos de 1 minuto em uma janela de 5 minutos.

    * **Ação** : ``Remove`` 1 instância.

![alt text](/imagens/image-33.png)

![alt text](/imagens/image-32.png)

``CONFERENCIA NO AUTO SCALING``

![alt text](/imagens/image-34.png)



### 10. Testando o Auto Scaling Gerando Carga

**Ferramenta de Teste** : Usei o hey, como sugerido na atividade. Ele é um programa pequeno e fácil de instalar a partir de sua página no **GitHub** : https://github.com/rakyll/hey. 

```BASH
hey -z 10m http://<SEU_DNS_DO_LOAD_BALANCER>/teste
```

![alt text](/imagens/image-35.png)

![alt text](/imagens/image-36.png)

``AUTO SCALING HISTORICO``

![alt text](/imagens/image-37.png)



``TERMINAL PÓS TESTE``

![alt text](/imagens/image-38.png)

## Tudo certo!  (: