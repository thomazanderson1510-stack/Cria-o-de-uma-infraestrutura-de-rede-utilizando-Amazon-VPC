

## 1. Apresentação do projeto

Neste laboratório, assumi o papel de um engenheiro de suporte em Cloud da AWS para solucionar um problema de conectividade apresentado por um cliente de uma startup.

O cliente possuía uma infraestrutura de rede parcialmente configurada na AWS, porém não conseguia estabelecer comunicação com a internet, nem mesmo realizar um teste simples utilizando o comando `ping`.

Meu objetivo foi construir e configurar uma infraestrutura de rede completa, garantindo que uma instância EC2 pudesse estabelecer comunicação com destinos externos à VPC.

Para isso, realizei a criação e configuração dos seguintes recursos:

- Amazon VPC;
- Sub-rede pública (Public Subnet);
- Internet Gateway (IGW);
- Tabela de rotas (Route Table);
- Network ACL (NACL);
- Security Group;
- Instância Amazon EC2.

## 2. Cenário do cliente

O cliente precisava de auxílio para configurar sua rede virtual na AWS e permitir a comunicação com a internet.

Antes de iniciar a configuração, analisei os requisitos do laboratório e defini o endereçamento IP que seria utilizado.

| Recurso | Configuração |
|---|---|
| VPC | 192.168.0.0/18 |
| Sub-rede pública | 192.168.1.0/28 |
| Rota local | 192.168.0.0/18 |
| Rota para internet | 0.0.0.0/0 |

A VPC representa a rede principal, enquanto a sub-rede permite organizar os recursos dentro dessa infraestrutura.

## 3. Criação da VPC

Comecei criando uma VPC com o nome `Test VPC`.

A VPC funciona como uma rede virtual privada dentro da AWS, permitindo organizar os endereços IP, sub-redes e recursos computacionais.

Configurei o bloco CIDR IPv4 como `192.168.0.0/18`, definindo o espaço de endereçamento disponível para a infraestrutura.

**Configurações utilizadas:**
- Nome: Test VPC
- IPv4 CIDR: 192.168.0.0/18

### Evidência – VPC criada

<img width="1876" height="273" alt="image" src="https://github.com/user-attachments/assets/64b1317e-eb08-4bea-8278-fbed359bbaf0" />


## 4. Criação da sub-rede pública (Public Subnet)

Após criar a VPC, realizei a criação de uma sub-rede chamada `Sub-rede pública`.

A sub-rede é uma divisão da VPC, utilizada para organizar os recursos que serão executados dentro da infraestrutura.

Configurei o bloco CIDR `192.168.1.0/28`, mantendo a sub-rede dentro do intervalo de endereçamento da VPC.

**Configurações utilizadas:**
- Nome: Sub-rede pública
- VPC: Test VPC
- CIDR: 192.168.1.0/28
- Zona de disponibilidade: Sem preferência

É importante destacar que uma sub-rede não se torna pública apenas pelo nome. Para permitir conectividade com a internet, é necessário configurar uma tabela de rotas direcionando o tráfego para um Internet Gateway.

### Evidência – Sub-rede criada

<img width="1720" height="675" alt="image" src="https://github.com/user-attachments/assets/fa143480-6bf4-438a-acb1-a246e59d6eff" />
<img width="1362" height="264" alt="image" src="https://github.com/user-attachments/assets/74e1d30f-3f96-48e4-b047-48ba32450f9c" />


## 5. Criação do Internet Gateway (IGW)

Em seguida, criei um Internet Gateway com o nome `IGW test VPC`.

Esse componente permite a comunicação entre a VPC e a internet.

Após criar o gateway, realizei sua associação à VPC `Test VPC`.

Essa associação é necessária para que o Internet Gateway possa ser utilizado nas regras de roteamento da rede.

### Evidência – Internet Gateway criado e associado

<img width="1325" height="301" alt="image" src="https://github.com/user-attachments/assets/4c8f979a-6e70-4ca8-97ab-ce4a5a042e80" />


## 6. Criação e configuração da tabela de rotas (Route Table)

Depois de associar o Internet Gateway à VPC, criei uma tabela de rotas chamada `Tabela de rotas públicas`.

A tabela de rotas determina para onde o tráfego de rede deve ser encaminhado, de acordo com o endereço de destino.

Configurei as seguintes rotas:

| Destino | Alvo | Finalidade |
|---|---|---|
| 192.168.0.0/18 | Local | Comunicação interna da VPC |
| 0.0.0.0/0 | Internet Gateway | Comunicação com a internet |

A rota local permite a comunicação entre os recursos da VPC.

Já a rota `0.0.0.0/0` direciona o tráfego destinado a qualquer endereço IPv4 externo para o Internet Gateway.

Após configurar as rotas, associei a tabela de rotas à sub-rede pública.

### Evidência – Rotas configuradas

<img width="1332" height="440" alt="image" src="https://github.com/user-attachments/assets/13f1f7a5-2f4e-47fc-83ce-b6f69ef5b235" />


### Evidência – Associação da tabela de rotas
<img width="1307" height="467" alt="image" src="https://github.com/user-attachments/assets/2f9915cf-b8d3-4d96-bc54-f016c4e03e53" />
<img width="1707" height="495" alt="image" src="https://github.com/user-attachments/assets/f06c3dc2-5faf-4cf7-9425-9494aec1c0f3" />


## 7. Criação da Network ACL (NACL)

Depois de configurar o roteamento, criei uma Network ACL com o nome `Public Subnet NACL`.

A NACL funciona como um firewall no nível da sub-rede, controlando o tráfego de entrada e saída.

Configurei as seguintes regras:

**Regras de entrada (Inbound):**
- Número da regra: 100
- Tipo: All Traffic
- Origem: 0.0.0.0/0
- Ação: Allow

**Regras de saída (Outbound):**
- Número da regra: 100
- Tipo: All Traffic
- Destino: 0.0.0.0/0
- Ação: Allow

Essa configuração permite o tráfego de entrada e saída durante os testes de conectividade.

Um detalhe importante é que a NACL é stateless, ou seja, as regras de entrada e saída precisam ser configuradas independentemente.

Também é necessário associar a NACL à sub-rede para que suas regras sejam aplicadas.

### Evidência – Regras de entrada

<img width="1721" height="542" alt="image" src="https://github.com/user-attachments/assets/6d49c060-19d9-45db-b31b-bba47d1ec0ec" />



### Evidência – Regras de saída
<img width="1719" height="519" alt="image" src="https://github.com/user-attachments/assets/714b70d1-a289-4e8e-86d3-48b843454adb" />




## 8. Criação do Security Group

Na sequência, criei um Security Group chamado `public security group`.

Ele funciona como um firewall virtual associado à interface de rede da instância EC2.

Configurei as seguintes regras de entrada:

| Protocolo | Porta | Finalidade |
|---|---|---|
| SSH | 22 | Acesso administrativo |
| HTTP | 80 | Comunicação web |
| HTTPS | 443 | Comunicação web segura |

Para as regras de saída, permiti todo o tráfego.

Diferentemente da NACL, o Security Group é stateful, permitindo automaticamente o tráfego de resposta de conexões autorizadas.

### Evidência – Security Group configurado

<img width="1250" height="508" alt="image" src="https://github.com/user-attachments/assets/3a02ad4d-6ddb-4b86-b843-3473b971747c" />


<img width="1335" height="552" alt="image" src="https://github.com/user-attachments/assets/b0868486-d8f8-4e5f-a0e7-c7b0012e7186" />


## 9. Criação da instância Amazon EC2

Com a infraestrutura de rede configurada, iniciei a criação de uma instância EC2 para validar a conectividade.

O Amazon EC2 permite provisionar servidores virtuais na nuvem.

Utilizei as seguintes configurações:

| Configuração | Valor |
|---|---|
| Sistema operacional | Amazon Linux 2023 |
| Tipo de instância | t3.micro |
| Par de chaves | vockey |
| VPC | Test VPC |
| Sub-rede | Sub-rede pública |
| IP público automático | Habilitado |
| Security Group | public security group |

Um ponto importante foi selecionar a VPC e a sub-rede criadas anteriormente, garantindo que a instância fizesse parte da infraestrutura configurada.

Também habilitei a atribuição automática de um endereço IPv4 público, permitindo que a instância pudesse estabelecer comunicação IPv4 com a internet.

### Evidência – Instância EC2 criada

<img width="1703" height="820" alt="image" src="https://github.com/user-attachments/assets/f4f4e3b7-6dbc-45fa-9115-842503cf7514" />


### Evidência – Configurações de rede da EC2

<img width="1699" height="624" alt="image" src="https://github.com/user-attachments/assets/6c5d608b-b077-4928-afba-51f0639c668d" />


## 10. Acesso à instância utilizando SSH

Após iniciar a instância, aguardei até que seu estado mudasse para `Running`.

Em seguida, realizei o acesso administrativo utilizando SSH (Secure Shell).

Esse protocolo permite acessar e administrar servidores remotamente por meio de uma conexão segura.

Utilizei a configuração de chave disponibilizada pelo ambiente de treinamento, chamada `vockey`.

Após estabelecer a conexão, tive acesso ao terminal Linux da instância EC2.

### Evidência – Acesso SSH

<img width="1355" height="494" alt="image" src="https://github.com/user-attachments/assets/ee42c61e-443d-4adc-9715-0a77065beb42" />



## 11. Teste de conectividade utilizando PING

Essa foi uma das etapas mais importantes do laboratório.

O objetivo era validar se a instância EC2 conseguia estabelecer comunicação com um endereço externo à VPC.

Para isso, utilizei o comando:

```bash
ping 8.8.8.8
```

O endereço `8.8.8.8` é um endereço IPv4 público de um servidor DNS do Google, utilizado como destino para testar a conectividade IP.

O comando PING utiliza o protocolo ICMP para enviar solicitações de comunicação ao destino.

Durante esse processo, os componentes da infraestrutura trabalham em conjunto:

1. A instância EC2 gera o pacote ICMP.
2. O Security Group permite a saída do tráfego.
3. A NACL permite a saída do pacote pela sub-rede.
4. A tabela de rotas identifica que o destino é externo à VPC.
5. O tráfego é encaminhado para o Internet Gateway.
6. A resposta retorna pelo caminho da comunicação estabelecida.

### Evidência – Teste de conectividade

<img width="1868" height="972" alt="image" src="https://github.com/user-attachments/assets/f9386b1e-98d7-49ef-a696-685448a20d2e" />
<img width="1818" height="698" alt="image" src="https://github.com/user-attachments/assets/ad3485ba-dfa8-4b52-bcf3-087ea1be3408" />



## 12. Visão geral da infraestrutura criada

Após configurar todos os recursos, consegui compreender como cada componente participa da comunicação entre a instância EC2 e a internet.

A estrutura pode ser representada da seguinte maneira:

```text
                    INTERNET
                       |
                       |
              INTERNET GATEWAY
                       |
                       |
              +----------------+
              |    TEST VPC    |
              | 192.168.0.0/18 |
              +----------------+
                       |
                       |
              TABELA DE ROTAS
                 0.0.0.0/0
                       |
                       |
              SUB-REDE PÚBLICA
              192.168.1.0/28
                       |
              +----------------+
              |   NETWORK ACL  |
              +----------------+
                       |
              +----------------+
              | SECURITY GROUP |
              +----------------+
                       |
              +----------------+
              |   EC2 SERVER   |
              | Amazon Linux   |
              +----------------+
                       |
                       |
                 TESTE PING
```

## 13. Conhecimentos adquiridos

Durante esse laboratório, coloquei em prática conceitos importantes de redes aplicados à computação em nuvem.

Entre os principais conhecimentos adquiridos, destaco:

- Criação e configuração de redes virtuais utilizando Amazon VPC.
- Planejamento de endereçamento IPv4 e blocos CIDR.
- Criação e organização de sub-redes.
- Configuração de Internet Gateway.
- Configuração de tabelas de rotas.
- Associação de tabelas de rotas às sub-redes.
- Diferença entre Network ACL e Security Group.
- Provisionamento de servidores utilizando Amazon EC2.
- Acesso remoto a servidores Linux utilizando SSH.
- Testes de conectividade utilizando ICMP e PING.
- Entendimento do fluxo de comunicação entre uma infraestrutura AWS e a internet.

## 14. Conclusão

Esse laboratório foi importante para entender, na prática, como construir uma infraestrutura de rede dentro da AWS.

Inicialmente, o cenário apresentava um problema de conectividade, no qual o cliente não conseguia realizar comunicação com destinos externos à sua VPC.

Para solucionar esse problema, construí uma infraestrutura envolvendo endereçamento IP, sub-redes, roteamento, gateways e regras de segurança.

O principal aprendizado foi compreender que não basta criar uma instância EC2 ou uma VPC para ter conectividade com a internet. É necessário integrar corretamente todos os componentes da infraestrutura.

Essa atividade também me ajudou a relacionar conhecimentos de redes de computadores com os recursos de infraestrutura disponíveis na AWS, ampliando minha experiência prática em Cloud Computing.
