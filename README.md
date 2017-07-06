# Documentação para instalação do Jerakia

- O que é Jerakia?

  - Jerakia é uma ferramenta hierárquica de pesquisa de dados inspirada pela Hiera, com o objetivo de oferecer flexibilidade e fácil integração. O gerenciamento de pesquisa de dados tornou-se uma parte crucial de qualquer linha de ferramentas de automação e onde você armazena e como você consulta os dados que impulsionam sua infra-estrutura tornou-se um conceito cada vez mais desafiador.(site projeto)

- Neste procedimento foi utilizado o uma VM;

  - CentOS 7 (Jerakia 2.0), jerakia.example.com;
  - Caso esteja com o firewall habilitado não esquecer de realizar a liberação;
  - Não é preciso desabilitar o SELINUX;

#### 1. Primeiro precisamos configurar o repositório do jerakia, importando a gpg e configurando o repositório.

```shell
rpm --import https://rpm.packager.io/key

echo "[jerakia]
name=Repository for crayfishx/jerakia application.
baseurl=https://rpm.packager.io/gh/crayfishx/jerakia/centos7/stable
enabled=1" | tee /etc/yum.repos.d/jerakia.repo
```

#### 2. Após configurar os repositórios, podemos realizar a instalação do jerakia.

```shell
yum install jerakia -y
```

#### 3. Após a instalação do jerakia, precisamos criar a estrutura de diretório default para o armazenamento das chaves que serão feitas lookup.

  - Por padrão o jerakia utiliza o diretório /var/lib/jerakia/data, porém esse diretório não existe, primeiro precisamos criar esse diretório para depois iniciar a criação dos arquivos yaml que armazenam nossas chaves, para alterar a ordem de lookup, alteramos o arquivo /etc/jerakia/policy.d/default.rb, abaixo um exemplo do arquivo.

```shell
# Jerakia policy file.
#
# The default policy is called :default, you can add further policies by adding
# them into the policy.d folder as name.rb
#
policy :default do

  # Lookups are initiated in order, each lookup must define at least a datasource
  # to tell Jerakia where to source the data from 
  #
  lookup :main do
    datasource :file, {
      :docroot => '/var/lib/jerakia/data',
      :searchpath => [
        "hostname/#{scope[:certname]}",
        "environment/#{scope[:environment]}",
        "common",
      ],
      :format => :yaml
    }
  end
end
```

  - No arquivo padrão, ele irá buscar em /var/lib/jerakia/data/hostname/(nomedoservidor), caso não encontre a key ele irá buscar em /var/lib/jerakia/data/environment/(ambiente), caso não encontre nesse caminho ele irá buscar em /var/lib/jerakia/data/common. Caso não encontre não irá retornar nenhum valor.
  
#### 4. Abaixo iremos criar a estrutura como exemplo e realizar algumas pesquisas com o jerakia.

```shell
mkdir -p /var/lib/jerakia/data/{hostname,environment,common}/
```

####  - Após criar a estrutura de pesquisa, precisamos criar os arquivos que serão feitos os lookups das chaves.

#### 4.1 - Primeiro iremos criar os namespaces keys em common, contento a chave foo com o valor bar.

  - O arquivo para o namespace keys é /var/lib/jerakia/data/common/keys.yaml, caso precisamos criar um outro namespace alteramos o nome do arquivo pelo namespace desejado.
  
```shell
---
foo: bar
```

#### 4.2 - Para testar executamos o comando abaixo, retornando o valor cadastrado anteriormente.

```shell
jerakia lookup foo --namespace=keys
```
  - Devemos ter o retorno da chave conforme abaixo.

```shell
bar
```

#### 4.3 - Agora iremos criar um namespace para ambiente, será criado um ambiente dev para fazer o lookup.

  - Primeiro precisamos criar a estrutura
  
```shell
mkdir -p /var/lib/jerakia/data/environment/dev
```

#### 4.4 - Agora iremos criar o namespace contendo a chave de pesquisa.

  - O arquivo é /var/lib/jerakia/data/environment/dev/keys.yaml com o conteúdo abaixo.

```shell
---
foo: dev_bar
```

#### 4.5 - Para testar utilizamos o comando abaixo.

```shell
jerakia lookup foo --namespace=keys --metadata environment:dev
```

  - Devemos ter o retorno abaixo

```shell
dev_bar
```

#### 4.6 - Agora iremos criar a estrutura para o lookup em hostname, muito similar ao criado para o environment.

```shell
mkdir /var/lib/jerakia/data/hostname/jerakia.example.local
```

#### 4.7 - Iremos criar o namespace de lookup keys.

  - O arquivo é /var/lib/jerakia/data/hostname/jerakia.example.local/keys.yaml com o conteúdo abaixo.

```shell
---
foo: jerakia.example.local_bar
```

#### 4.8 - Para testar o lookup executamos o comando abaixo.

```shell
jerakia lookup foo --namespace=keys --metadata environment:dev certname:jerakia.example.local
```

  - O retorno esperado deve ser o abaixo.

```shell
jerakia.example.local_bar
``` 

#### Apartir de agora o data lookup do Jerakia está pronto, sendo preciso fomentar as keys.

#### Fontes.

http://jerakia.io/

http://jerakia.io/basics/install/

http://jerakia.io/basics/lookups/

http://jerakia.io/basics/cli/
