---
title: Performance e elegância! Escrevendo uma CLI CRUD utilizando ScyllaDB e Ruby
description: Como desenvolver uma CLI com ações CRUDs consumindo o banco de dados ScyllaDB com Ruby.
tags: 'ruby,scylladb,braziliandevs,beginners'
cover_image: ''
canonical_url: null
published: false
---

Boas pessoas desenvolvedoras precisam saber fazer CRUD não é mesmo? Então já pensou em ser capaz de produzir um CRUD com um banco de dados NoSQL montado para **alta** escalabilidade e ainda mais utilizando uma linguagem elegante e simples? Não? Pois muito que bem, nesse artigo você vai aprender como construir uma CLI utilizando a gem `dry-cli` consumindo o banco de dados ScyllaDB com a gem `cassandra-driver`

Para saber um pouco mais sobre **o que é ScyllaDB** e em quais contextos essa ferramenta é útil, recomendo ler a [documentação oficial](https://cloud.docs.scylladb.com/stable/scylladb-basics/)

> Disclaimer: Esse artigo assume conhecimento básico com bancos de dados visto que o objetivo vai ser focar na utilização em conjunto com ruby, para mais informações especificas sobre Scylla DB recomendo:
> 1. Os artigos criados pelo Developer Advocate da ScyllaDB DanielHe4rt no [Dev.To](https://dev.to/danielhe4rt)
> 2. Os cursos gratuitos promovidos pela própria ScyllaDB no [ScyllaDB University](https://university.scylladb.com/)

## Table of Contents

- [Iniciando o projeto](#1-iniciando-o-projeto)
- [Definindo a camada de injeção de dependências](#2-definindo-a-camada-de-injeção-de-dependências)
- [Definindo o boilerplate para nossa CLI](#3-definindo-o-boilerplate-para-nossa-cli)
- [Implementando nossos comandos](#4-implementando-nossos-comandos)
- [Conclusão](#conclusão)

## 1. Iniciando o projeto

### 1.1 Resolvendo bibliotecas de sistema para instalar o driver

A gem que vamos utilizar para comunicar com o ScyllaDB se chama [cassandra-driver](https://github.com/datastax/ruby-driver/), infelizmente ela exige bibliotecas de sistema relacionadas ao banco de dados cassandra para serem instaladas, a forma mais simples de ter essas bibliotecas instaladas é instalar o próprio cassandra na maquina:

> Disclaimer: Não vamos utilizar o banco cassandra para nada na prática desse artigo, apenas precisamos do pacote instalado para que a gem consiga buscar as bibliotecas necessárias ao seu funcionamento. Visto que o ScyllaDB é baseado originalmente no Cassandra podemos utilizar tranquilamente.

Para instalar o cassandra no MacOS:

```sh
brew install cassandra
```

Para instalar o cassandra no Linux(Debian):

```sh
sudo apt-get install -y cassandra
```

Caso sua Distro/Sistema operacional não tenha sido listada, recomendo sempre recorrer a [Documentação Oficial](https://cassandra.apache.org/doc/latest/cassandra/getting_started/installing.html)

### 1.2 Iniciando o projeto

Com as bibliotecas de sistema corretamente instaladas, podemos iniciar nosso projeto utilizando [bundler](https://bundler.io/):

```sh
mkdir project_scylla && cd project_scylla && bundle init
```

### 1.3 Instalando dependências de projeto

Agora podemos finalmente adicionar as gems necessárias para o projeto:

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

Como somos bons rubistas devemos fazer o setup do nosso IRB como a primeira ação no projeto correto?

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

Excelente! Agora temos o básico do nosso setup concluído! Vamos prosseguir definindo a camada de injeção de dependência e um provedor para incorporar a funcionalidade do ScyllaDB.

## 2. Definindo a camada de injeção de dependências

Nesse artigo vamos seguir um modelo igual mostrado no meu [artigo sobre sinatra com injeção de dependências](https://dev.to/cherryramatis/creating-a-sinatra-api-with-system-wide-dependency-injection-using-dry-system-10mp), para um detalhamento maior de como esse setup e feito e o porquê é feito, por favor referencie a esse link.

### 2.1 Criando o container principal

Como descrito no artigo referenciada acima, para nossa camada de injeção de dependências funcionar precisamos definir um container principal que vai servir de referencia para buscar dependências e registrar novos providers.

Agora vamos criar um arquivo em `config/application.rb` com o seguinte conteúdo:

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

Neste arquivo recém criado estamos definindo a localização dos componentes ao longo da nossa aplicação, a principio isso vai definir auto loading desses arquivos para que não precisemos usar `require` sempre que usamos alguma classe de algum arquivo.

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

Agora que temos um container principal para nossa aplicação, podemos definir o único provider desse projeto que vai ser a conexão com o ScyllaDB.

Para isso crie um arquivo em `config/provider/database.rb` com o seguinte conteúdo:

```ruby
# frozen_string_literal: true

Application.register_provider(:database) do
  prepare do
    require 'cassandra'
    require_relative '../../lib/migration_utils'
    require_relative '../constants'
  end

  start do
    cluster = Cassandra.cluster(
      username: ENV.fetch('DB_USER', nil),
      password: ENV.fetch('DB_PASSWORD', nil),
      hosts: ENV.fetch('DB_HOSTS', nil).split(',')
    )

    connection = cluster.connect

    MigrationUtils.create_keyspace(session: connection) if MigrationUtils.keyspace_exist?(session: connection)
    MigrationUtils.create_table(session: connection) if MigrationUtils.table_exist?(session: connection)

    connection = cluster.connect(KEYSPACE_NAME)

    register('database.connection', connection)
  end
end
```

Para as credenciais recomendo utilizar o serviço [cloud ScyllaDB](https://cloud.scylladb.com/), nele você consegue criar um cluster super rápido e ganhar acesso a todas as credenciais de maneira super simples.

> Um exemplo .env pode ser visto abaixo:

```
DB_USER=scylla
DB_PASSWORD=password
DB_HOSTS=node-0.amazonaws,node-1.amazonaws,node-2.amazonaws
```

Caso você tenha lido o artigo sobre injeção de dependências mencionado acima esse provider parece bem simples, mas também temos o uso de algumas classes novas que ainda não criamos e portanto vamos verificá-las passo a passo:

#### Definindo as constantes para nosso projeto

Nessa aplicação vamos manter nomes de keyspace e tabela definidos em constantes, você pode alterar para receber por parâmetro ou variável de ambiente, mas para simplicidade vamos deixar apenas com uma constante mesmo.

Crie um arquivo em `config/constants.rb` com o seguinte conteúdo:

```ruby
# frozen_string_literal: true

KEYSPACE_NAME = 'media_player'
TABLE_NAME = 'playlist'
```

#### Criando uma classe utilitária para criar nosso banco

Como podemos ver no exemplo do provider anterior, estamos usando a classe `MigrationUtils` para produzir atividades comuns para a inicialização do nosso keyspace e da nossa tabela.

Agora vamos seguir passo a passo pelos métodos necessários para a criação do nosso keyspace e tabelas.

##### Checando se um keyspace ou tabela existe

Antes de prosseguirmos com a criação dos nossos keyspaces e tabelas, é crucial verificar se eles já existem a fim de evitar a execução desnecessária da função. Para isso, vamos implementar métodos booleanos da seguinte maneira:

Primeiramente, criaremos um arquivo chamado `migration_utils.rb`, localizado em
`lib/migration_utils.rb`, e o preencheremos com o código descrito abaixo:

```ruby
class MigrationUtils
  # @param session [Cassandra#Cluster]
  # @return [Boolean]
  def self.keyspace_exist?(session:)
    query = <<~SQL
      select keyspace_name from system_schema.keyspaces WHERE keyspace_name=?
    SQL

    session.execute_async(query, arguments: [KEYSPACE_NAME]).join.rows.size.zero?
  end

  # @param session [Cassandra#Cluster]
  # @return [Boolean]
  def self.table_exist?(session:)
    query = <<~SQL
      select keyspace_name, table_name from system_schema.tables where keyspace_name = ? AND table_name = ?
    SQL

    session.execute_async(query, arguments: [KEYSPACE_NAME, TABLE_NAME]).join.rows.size.zero?
  end
end
```

Aqui, estamos implementando métodos estáticos que serão executados antes de configurarmos a camada de injeção de dependência. Por essa razão, aceitamos a sessão do banco de dados como parâmetro para essas funções. Com essa sessão, podemos usar o método `execute_async` para enviar uma consulta CQL. Esse método também nos permite utilizar placeholders `?` para os parâmetros e especificar os valores em um objeto `arguments: []`.

Como esse método funciona de forma assíncrona, precisamos usar o método `join` para esperar essa [Future](https://docs.datastax.com/en/developer/ruby-driver/3.2/api/cassandra/future/) finalizar e nos retornar um objeto. Com o objeto em mãos, podemos acessar a propriedade rows sendo esta uma lista contendo todas as linhas referentes a query mostrada acima.

Para finalizar a implementação do método retornando um booleano, vamos checar se a lista esta vazia ou não checando se o tamanho da lista é zero com `.size.zero?`

> Disclaimer: Precisamos usar o `.size.zero?` pois o retorno é um [Enumerator](https://ruby-doc.org/core-2.6/Enumerator.html), que não possui o método `.empty?`

##### Criando os keyspaces e tabelas

Agora que criamos métodos responsáveis por checar a existência de um keyspace e uma tabela, precisamos criar os métodos que vão criar eles caso já não existam correto?

Para isso, vamos continuar trabalhando na classe localizada em `lib/migration_utils.rb` com o seguinte conteúdo:

```ruby
class MigrationUtils
  # @param session [Cassandra#Cluster]
  # @return [void]
  def self.create_keyspace(session:)
    query = <<~SQL
      CREATE KEYSPACE #{KEYSPACE_NAME}
      WITH replication = {
        'class': 'NetworkTopologyStrategy',
        'replication_factor': '3'
      }
      AND durable_writes = true
    SQL

    session.execute_async(query).join
  end

  # @param session [Cassandra#Cluster]
  # @return [void]
  def self.create_table(session:)
    query = <<~SQL
      CREATE TABLE #{KEYSPACE_NAME}.#{TABLE_NAME} (
        id uuid,
        title text,
        album text,
        artist text,
        created_at timestamp,
        PRIMARY KEY (id, created_at)
      );
    SQL

    session.execute_async(query).join
  end
end
```

Aqui vamos seguir o mesmo padrão onde recebemos o `session` como parâmetro e o usamos para executar uma query assíncrona com `execute_async`, como não precisamos lidar com o retorno dessas queries podemos apenas usar o `join` para esperar ela finalizar.

> Note: Para qualquer duvida referente as queries em si recomendo fortemente os links mencionados da [ScyllaDB University](https://university.scylladb.com/).

### 2.3 Carregando o novo provider na nossa aplicação

Agora que definimos e entendemos o nosso provider de banco de dados, precisamos carregá-lo para que seja injetado nos nossos dois pontos principais da aplicação:

No `main.rb` vamos adicionar o require:

```ruby
require_relative 'config/provider/database'
```

E no `bin/console` a mesma coisa:

```ruby
require_relative '../config/provider/database'
```

## 3. Definindo o boilerplate para nossa CLI

Agora que temos uma camada de banco de dados e auto requiring pronta pra ser usada, vamos utilizar a outra grande gem desse projeto para definir os comandos da nossa CLI. Vem ai `dry-cli` senhoras e senhores!

Nesse primeiro momento vamos nos preocupar em apenas definir o boilerplate para nossa CLI, sem nos preocupar com a implementação real certo?

Para isso, vamos definir o modulo que vai registrar todos os comandos, localizado em `lib/cli.rb` com o seguinte conteúdo:

```ruby
# frozen_string_literal: true

require 'dry/cli'

module Cli
  extend Dry::CLI::Registry

  register 'add', Commands::Add
end
```

Nesse modulo inicial podemos usar uma [DSL](https://dev.to/vinistock/write-a-simple-dsl-in-ruby-1jgi) para registrar novos comandos com o `register`, essa DSL é fornecida ao extender o modulo `Dry::CLI::Registry`.

Agora que registramos um comando `Add`, vamos criar a classe referente a ele localizada em `lib/cli/commands/add.rb` com o seguinte conteúdo:

```ruby
# frozen_string_literal: true

require 'dry/cli'

module Cli
  module Commands
    class Add < Dry::CLI::Command
      desc 'This command add a new song to the playlist'

      argument :title, type: :string, required: true, desc: 'The title of the song'
      argument :album, type: :string, required: true, desc: 'The name of the album of that song'
      argument :artist, type: :string, required: true, desc: 'The name of the artist of band'

      def call(title:, album:, artist:)
        puts "Add command -> Title: #{title}, Album: #{album}, Artist: #{artist}"
      end
    end
  end
end
```

Nessa classe podemos ver outra DSL fornecida por herdar a classe `Dry::CLI::Command`, com ela podemos prover uma descrição para o comando usando `desc`, declarar quais argumentos vamos receber para esse comando junto com seu tipo, validação e descrição com `argument` e muito mais!

Logo após definir os metadados do nosso comando, definimos um método `call` que vai receber os argumentos definidos como parâmetros nomeados.

Em nosso arquivo `main.rb` podemos inicializar a nossa CLI com:

```ruby
require 'dry/cli'

Dry::CLI.new(Cli).call
```

Finalmente, executando nossa aplicação com `ruby main.rb` devemos ter o seguinte output:

```sh
$ ruby main.rb
Commands:
  main.rb add TITLE ALBUM ARTIST                 # This command add a new song to the playlist
```

## 4. Implementando nossos comandos

Agora que temos um boilerplate e um entendimento básico quanto ao funcionamento da gem `dry-cli` podemos nos preocupar em implementar alguns comandos simples para o nosso CRUD.

### 4.1 Implementando o primeiro comando Add

Como já temos a classe para esse comando criada, vamos apenas começar a trabalhar na implementação do método `call` da seguinte forma:

```ruby
# frozen_string_literal: true

require 'dry/cli'

module Cli
  module Commands
    class Add < Dry::CLI::Command
      desc 'This command add a new song to the playlist'

      argument :title, type: :string, required: true, desc: 'The title of the song'
      argument :album, type: :string, required: true, desc: 'The name of the album of that song'
      argument :artist, type: :string, required: true, desc: 'The name of the artist of band'

      def initialize
        super
        @repo = Application['database.connection']
      end

      def call(title:, album:, artist:)
        query = <<~SQL
          INSERT INTO #{KEYSPACE_NAME}.#{TABLE_NAME} (id,title,artist,album,created_at) VALUES (now(),?,?,?,?);
        SQL

        @repo.execute_async(query, arguments: [title, artist, album, Time.now]).join

        puts "Song '#{title}' from artist '#{artist}' Added!"
      end
    end
  end
end
```

Nesta implementação, primeiro vamos injetar nossa conexão com o banco de dados por meio da camada de injeção de dependência no construtor da classe e então iremos utilizar o método já conhecido `execute_async` para inserir um novo dado na tabela correta.

Um ponto importante a ressaltar é o uso da função `now()` na query, essa função é nativa do banco de dados e serve para inserir um novo UUID no padrão esperado pelo ScyllaDB, dessa forma não precisamos lidar com geração de UUID pelo lado da linguagem.

Bem simples certo? vamos continuar com os próximos comandos do nosso CRUD seguindo a mesma arquitetura já proposta.

### 4.2 Implementando o comando List

Após ter elaborado um comando para adicionar novas músicas, avançaremos para a criação de um comando destinado a listar todas as músicas já criadas. Para isso vamos criar um arquivo em `lib/cli/commands/list.rb` com o seguinte conteúdo:

```ruby
# frozen_string_literal: true

require 'dry/cli'

module Cli
  module Commands
    class List < Dry::CLI::Command
      desc 'This command shows all the created songs'

      def initialize
        super
        @repo = Application['database.connection']
      end

      def call
        query = <<~SQL
          SELECT * FROM #{KEYSPACE_NAME}.#{TABLE_NAME};
        SQL

        @repo.execute_async(query).join.rows.each do |song|
          puts <<~MSG
            ID: #{song['id']} | Song Name: #{song['title']} | Album: #{song['album']} | Created At: #{song['created_at']}
          MSG
        end
      end
    end
  end
end
```

Novamente, estamos aqui fazendo uma query `SELECT` simples e percorrendo pelos resultados no array `rows` para mostrar ao usuário final, é importante ressaltar que os fields acessados com `row['title']` correspondem aos fields que criamos quando fizemos o `CREATE TABLE` no provider.

Não podemos esquecer de registrar esse comando então vamos modificar o arquivo `lib/cli.rb`:

```ruby
# frozen_string_literal: true

require 'dry/cli'

module Cli
  extend Dry::CLI::Registry

  register 'add', Commands::Add
  register 'list', Commands::List # <= Novo comando
end
```

Perfeito! Agora podemos tanto adicionar quanto listar musicas :D

Uma simples demo do que temos agora tanto com add quanto list:

<a href="https://asciinema.org/a/604950" target="_blank"><img src="https://asciinema.org/a/604950.svg" /></a>

### 4.3 Implementando o comando Delete

Vamos explorar agora um comando bastante interessante. Neste caso, iremos exibir uma lista de músicas acompanhadas por seus indices, permitindo ao usuário selecionar uma música específica através da posição correspondente.

Para realizar isso, vamos desenvolver o comando e implementar um método que seja responsável por listar as músicas com suas respectivas posições numéricas. Além disso, iremos também aguardar o input do usuário.

A seguir, vamos criar um novo arquivo localizado em `lib/cli/commands/delete.rb` com o seguinte conteúdo:

```ruby
# frozen_string_literal: true

require 'dry/cli'

module Cli
  module Commands
    class Delete < Dry::CLI::Command
      desc 'This command will prompt for a song to be deleted and then delete it'

      def initialize
        super
        @repo = Application['database.connection']
      end

      def call
        songs = @repo.execute_async("SELECT * FROM #{KEYSPACE_NAME}.#{TABLE_NAME}").join.rows

        song_to_delete_index = select_song_to_delete(songs:)

        query = <<~SQL
          DELETE FROM #{KEYSPACE_NAME}.#{TABLE_NAME} WHERE id = ?
        SQL

        song_to_delete = songs.to_a[song_to_delete_index]

        @repo.execute_async(query, arguments: [song_to_delete['id']]).join
      end

      private

      # @param songs [Array<Hash>]
      def select_song_to_delete(songs:)
        songs.each_with_index do |song, index|
          puts <<~DESC
            #{index + 1})  Song: #{song['title']} | Album: #{song['album']} | Artist: #{song['artist']} | Created At: #{song['created_at']}
          DESC
        end

        print 'Select a song to be deleted: '
        $stdin.gets.chomp.to_i - 1
      end
    end
  end
end
```

No método `select_song_to_delete` nós recebemos uma lista de musicas e percorremos por ela com um índice usando o método `each_with_index`, dessa forma conseguimos mostrar uma mensagem no modelo `1) Song: | Album: | Artist: | Created At:`. Ainda nesse método esperamos o input do usuário com o `$stdin.gets.chomp` e repassamos no retorno como um integer convertendo com `.to_i`.

Já no método `call`, começamos fazendo uma consulta para obter todas as músicas registradas na tabela. Em seguida, passamos esse conjunto de dados para o método que permite ao usuário selecionar uma em especifico e retornar o índice da mesma. Usamos esse índice para escolher uma música específica no array e, em seguida, executamos uma consulta `DELETE` para removê-la.

> Disclaimer: Antes de selecionar um item especifico, precisamos transformar em um array com `.to_a` visto que essas linhas são um [Enumerator](https://ruby-doc.org/core-2.6/Enumerator.html).

Uma demo mostrando o uso prático do comando:

<a href="https://asciinema.org/a/604952" target="_blank"><img src="https://asciinema.org/a/604952.svg" /></a>

Não podemos esquecer de registrar esse comando então vamos modificar o arquivo `lib/cli.rb`:

```ruby
# frozen_string_literal: true

require 'dry/cli'

module Cli
  extend Dry::CLI::Registry

  register 'add', Commands::Add
  register 'list', Commands::List
  register 'delete', Commands::Delete # <= Novo comando
end
```

### 4.4 Implementando o comando Update

Agora vamos criar um comando que vai juntar todos os conceitos mostrados
anteriormente, este será um `update` e vai performar da seguinte forma:

1. Vamos aceitar parâmetros como `title`, `album`, `artist` para usar como parte do update (semelhante ao que fizemos no comando `add`)
2. Vamos mostrar para o usuário uma lista das musicas cadastradas e esperar o input do usuário com um índice (semelhante ao que fizemos no comando `delete`)

Perfeito! Vamos criar um comando localizado em `lib/cli/commands/update.rb` com o seguinte conteúdo:

```ruby
# frozen_string_literal: true

require 'dry/cli'

module Cli
  module Commands
    class Update < Dry::CLI::Command
      desc 'This command will prompt for a song to be updated and use the argument information to updated it'

      argument :title, type: :string, required: true, desc: 'The title of the song'
      argument :album, type: :string, required: true, desc: 'The name of the album of that song'
      argument :artist, type: :string, required: true, desc: 'The name of the artist of band'

      def initialize
        super
        @repo = Application['database.connection']
      end

      def call(title:, album:, artist:)
        songs = @repo.execute_async("SELECT * FROM #{KEYSPACE_NAME}.#{TABLE_NAME}").join.rows

        song_to_update_index = select_song_to_update(songs:)

        query = <<~SQL
          UPDATE #{KEYSPACE_NAME}.#{TABLE_NAME} SET title = ?, artist = ?, album = ? WHERE id = ? AND created_at = ?
        SQL

        song_to_update = songs.to_a[song_to_update_index]

        @repo.execute_async(query, arguments: [title, artist, album, song_to_update['id'] song_to_update['created_at']]).join
      end

      private

      # @param songs [Array<Hash>]
      def select_song_to_update(songs:)
        songs.each_with_index do |song, index|
          puts <<~DESC
            #{index + 1})  Song: #{song['title']} | Album: #{song['album']} | Artist: #{song['artist']} | Created At: #{song['created_at']}
          DESC
        end

        print 'Select a song to be updated: '
        $stdin.gets.chomp.to_i - 1
      end
    end
  end
end
```

Como pode ver, esse comando é realmente uma junção entre os conceitos do comando `add` (arguments) e conceitos do comando `delete` (método select pelo índice).

Sobre a query `update` é importante ressaltar que no ScyllaDB temos duas primary keys(nesse caso `id` e `created_at`), portanto precisamos utilizar ambas as informações para que o banco de dados ache corretamente nossa linha.

Uma demo mostrando o funcionamento do comando:

<a href="https://asciinema.org/a/604955" target="_blank"><img src="https://asciinema.org/a/604955.svg" /></a>

## Conclusão

Espero que esse artigo tenha sido útil! Tentei ao máximo focar na integração entre Ruby e ScyllaDB pois não encontrei nada com uma linguagem simples para iniciantes. Para duvidas direcionadas especificamente ao ScyllaDB recomendo fortemente os artigos do [DanielHe4rt](https://dev.to/danielhe4rt) e o [ScyllaDB University](https://university.scylladb.com/).

E não se esqueça de dar continuidade a este projeto que você acaba de criar!  Deixo como um desafio a busca por aprimoramentos na arquitetura que construímos. Abaixo, elenco alguns pontos de melhoria evidentes, porém, encorajo você a identificar outros que possam ter passado despercebidos por mim. A prática é o segredo para se tornar uma pessoa desenvolvedora habilidosa!
😄

1. As funções `select_song_to_delete` e `select_song_to_update` são iguais, talvez mover ela para algum local comum?
2. No comando `add` nós printamos uma mensagem de sucesso para o usuário, mas não seguimos isso nos outros comandos, podemos melhorar essa experiencia de usuário?

May the force be with you! 🍒
