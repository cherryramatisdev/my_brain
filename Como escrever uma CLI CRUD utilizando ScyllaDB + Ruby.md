---
title: Performance e elegância! Escrevendo uma CLI CRUD utilizando ScyllaDB e Ruby
description: Como desenvolver uma CLI com ações CRUDs consumindo o banco de dados ScyllaDB com Ruby.
tags: 'ruby,scylladb,braziliandevs,beginners'
cover_image: ''
canonical_url: null
published: false
---

```
TODOS
- Definir o que eh o scyllaDB (linkar alguma doc do dani?)
- Linkar o projeto getting started do scylla ja que to baseando nele (https://github.com/cherryramatisdev/scylla-cloud-getting-started) (APOS MERGEAR)
```

Boas pessoas desenvolvedoras precisam saber fazer CRUD não é mesmo? Então já pensou em ser capaz de produzir um CRUD com um banco de dados noSQL montado para **alta** escalabilidade e ainda mais utilizando uma linguagem elegante e simples? Não? Pois muito que bem, nesse artigo você vai aprender como construir uma cli utilizando a gem `dry-cli` consumindo o banco de dados scyllaDB com a gem `cassandra-driver`

## 1. Iniciando o projeto

### 1.1 Resolvendo bibliotecas de sistema para instalar o driver

A gem que vamos utilizar para comunicar com o scyllaDB se chama [cassandra-driver](https://github.com/datastax/ruby-driver/), infelizmente ela exige bibliotecas de sistema para ser instalada então recomendo instalar o cassandra no seu sistema com:

> Mais detalhes podem ser achados em https://cassandra.apache.org/doc/latest/cassandra/getting_started/installing.html

Para macOS:

```sh
brew install cassandra
```

Para linux(debian):

```sh
sudo apt-get install -y cassandra
```

### 1.2 Iniciando o projeto

Com as bibliotecas de sistema corretamente instaladas, podemos iniciar nosso projeto utilizando [bundler](https://bundler.io/):

```sh
mkdir project_scylla && cd project_scylla && bundle init
```

### 1.3 Instalando dependências de projeto

Agora podemos finalmente adicionar nossas gems necessárias para o projeto:

```sh
bundle add cassandra-driver
bundle add dry-auto_inject
bundle add dry-system
bundle add zeitwerk
bundle add dry-cli
bundle add dotenv
```

- [cassandra-driver](https://github.com/datastax/ruby-driver/)
- [dry-auto_inject](https://github.com/dry-rb/dry-auto_inject)
- [dry-system](https://github.com/dry-rb/dry-system)
- [zeitwerk](https://github.com/fxn/zeitwerk)
- [dotenv](https://github.com/bkeepers/dotenv)
- [dry-cli](https://github.com/dry-rb/dry-cli)

### 1.4 Definindo um REPL

Como somos bons rubistas devemos fazer o setup do nosso IRB como a primeira coisa no projeto correto?

Para isso crie um script executável em `bin/console` com:

```sh
touch bin/console && chmod +x bin/console
```

Nesse arquivo vamos incluir um setup básico com IRB + Dotenv:

```ruby
#!/usr/bin/env ruby

require 'dotenv/load'

require 'irb'
IRB.start
```

Perfeito! Agora o básico do nosso setup está pronto! Vamos definir nossa layer de injeção de dependência e um provider para utilizar o scyllaDB

## 2. Definindo a layer de injeção de dependências

Vamos seguir um modelo igual ao mostrado no meu artigo sobre sinatra com injeção de dependências: https://dev.to/cherryramatis/creating-a-sinatra-api-with-system-wide-dependency-injection-using-dry-system-10mp

### 2.1 Criando o container principal

Para isso vamos criar um arquivo em `config/application.rb` com o seguinte conteúdo:

```ruby
# frozen_string_literal: true

require 'dry/system'

class Application < Dry::System::Container
  configure do |config|
    config.root = Pathname('.')

    config.component_dirs.loader = Dry::System::Loader::Autoloading

    config.component_dirs.add 'lib'
    config.component_dirs.add 'config'
  end
end

loader = Zeitwerk::Loader.new
loader.push_dir(Application.config.root.join('lib').realpath)
loader.push_dir(Application.config.root.join('config').realpath)
loader.setup
```

Aqui estamos definindo  a localização dos componentes ao longo da nossa aplicação, a principio isso vai definir autoloading desses arquivos para que não precisemos usar `require` em todos para usar.

Com o container definido já podemos modificar nosso script de REPL para que contenha o método `finalize!` da nossa aplicação recém definida.

```ruby
#!/usr/bin/env ruby

require 'dotenv/load'
require_relative '../config/application'

Application.finalize!

require 'irb'
IRB.start
```

Também vamos criar um arquivo `main.rb` na raiz do nosso projeto apenas com o `finalize!` para servir como um ponto de entrada na nossa CLI.

```ruby
require 'dotenv/load'
require_relative '../config/application'

Application.finalize!
```

### 2.2 Criando o provider de banco de dados

Agora que temos uma classe principal para nossa aplicação, podemos definir o único provider desse projeto que vai ser o database.

Para isso crie um arquivo em `config/provider/database.rb` com o seguinte conteúdo:

```ruby
# frozen_string_literal: true

Application.register_provider(:database) do
  prepare do
    require 'cassandra'
    require_relative '../../lib/migrate'
  end

  start do
    cluster = Cassandra.cluster(
      username: ENV.fetch('DB_USER', nil),
      password: ENV.fetch('DB_PASSWORD', nil),
      hosts: ENV.fetch('DB_HOSTS', nil).split(',')
    )

    connection = cluster.connect

    keyspace_name = KEYSPACE_NAME
    table_name = TABLE_NAME

    MigrationUtils.create_keyspace(session: connection, keyspace_name:) if MigrationUtils.keyspace_exist?(
      session: connection, keyspace_name:
    )

    MigrationUtils.create_table(session: connection, keyspace_name:, table_name:) if MigrationUtils.table_exist?(session: connection, keyspace_name:, table_name:)

    connection = cluster.connect(keyspace_name)

    register('database.connection', connection)
  end
end
```

Caso você tenha lido o artigo sobre injeção de dependências mencionado acima esse provider parece bem simples, mas também temos algumas coisas novas que ainda não criamos e portanto vamos passo a passo:

#### Definindo as constantes para nosso projeto

Nessa aplicação vamos manter nomes de keyspace e tabela definidos em constantes, você pode alterar para receber por paramêtro ou variável de ambiente, mas para simplicidade vamos deixar apenas com uma constante mesmo.

Crie um arquivo em `config/constants.rb` com o seguinte conteúdo:

```ruby
# frozen_string_literal: true

KEYSPACE_NAME = 'media_player'
TABLE_NAME = 'playlist'
```

#### Criando uma classe utilitária para iniciar nosso banco

Como podemos ver no exemplo do provider anterior, estamos usando a classe `MigrationUtils` para produzir atividades comuns para a inicialização do nosso keyspace e da nossa tabela.

Vamos agora passar metodo por metodo desta classe abaixo:

TODO

Crie um arquivo em ``