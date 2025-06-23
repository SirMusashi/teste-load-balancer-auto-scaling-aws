### 1. Criando a VPC
Caminho: ``VPC`` > ``Suas VPCs `` > ``Criar VPC ``

![alt text](/imagens/image-2.png)

![alt text](/imagens/image-3.png)

![alt text](/imagens/image-4.png)

### 2. Habilitando um IP público
Caminho: ``VPC`` > ``Sub-redes`` > ``Ações`` > ``Editar configurações da sub-rede`` > ``Habilitar a atribuição automática de IPs públicos IPv4`` para ambas.

![alt text](/imagens/image-5.png)

![alt text](/imagens/image-6.png)


### 3. Criando um grupo de segurança
Caminho: ``EC2`` > ``Grupos de segurança`` > ``Criar grupo de segurança``

![alt text](/imagens/image-7.png)

### 4. Criando um Load Balancer
Caminho: ``EC2`` > ``Load Balancers`` > ``Criar load balancer`` > ``Classic Load Balancer``

![alt text](/imagens/image.png)

![alt text](/imagens/image-1.png)

![alt text](/imagens/image-8.png)

![alt text](/imagens/image-9.png)

![alt text](/imagens/image-10.png)


### 5. Criando um Launch Template

Caminho: ``EC2`` > ``Modelos de Execução`` > ``Criar Modelo de execução``

![alt text](/imagens/image-11.png)

![alt text](/imagens/image-12.png)

![alt text](/imagens/image-13.png)

![alt text](/imagens/image-14.png)

![alt text](/imagens/image-15.png)

![alt text](/imagens/image-16.png)

```Bash
#!/bin/bash
yum update -y
yum install -y httpd
chkconfig httpd on
service httpd start
echo "Hello World from $(hostname -f)" > /var/www/html/index.html

# Servidor básico com endpoint /teste
cat <<EOL > /var/www/cgi-bin/teste
#!/bin/bash
echo "Content-Type: text/plain"
echo ""
echo "Requisição recebida com sucesso na instância: $(hostname)"
sleep 5
EOL

chmod +x /var/www/cgi-bin/teste
# Habilitar CGI no Apache
sed -i 's/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf
echo 'ScriptAlias /teste /var/www/cgi-bin/teste' > /etc/httpd/conf.d/cgi.conf
service httpd restart
```


### 6. Criando O Auto Scaling Group
Caminho: ``EC2`` > ``Grupos Auto Scaling `` > ``Criar grupo do Auto Scaling``

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

``CRIANDO O ALARME 01``

![alt text](/imagens/image-30.png)

![alt text](/imagens/image-31.png)

``CRIANDO O ALARME 02``

![alt text](/imagens/image-33.png)

![alt text](/imagens/image-32.png)

``CONFERENCIA NO AUTO SCALING``

![alt text](/imagens/image-34.png)



### 10. Testando o Auto Scaling Gerando Carga

Ferramenta de Teste: Vamos usar o hey, como sugerido na atividade. Se você não o tiver, ele é um programa pequeno e fácil de instalar a partir de sua página no GitHub: https://github.com/rakyll/hey. (Se preferir, pode usar outra ferramenta como o ab do Apache).

```BASH
hey -z 10m http://<SEU_DNS_DO_LOAD_BALANCER>/teste
```

![alt text](/imagens/image-35.png)

![alt text](/imagens/image-36.png)

``AUTO SCALING HISTORICO``

![alt text](/imagens/image-37.png)



``TERMINAL PÓS TESTE``

![alt text](/imagens/image-38.png)