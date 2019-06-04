# Instalando um Aplicativo Ruby em um Servidor Linux

1. Clone o projeto:

  ```bash
  cd ~
  git clone https://github.com/celsoannes/railsgirls.git
  ```
1. Instale as dependencia do Aplicativo

  ```bash
  cd railsgirls
  bundle install --deployment --without development test
  ```

1. Configure ``database.yml`` e ``secrets.yml``

  ```bash
  vi config/database.yml
  ```
  
  Certifique-se de que a sessão ``production`` se pareça com isso:

  ```ruby
  production:
    adapter: sqlite3
    database: db/production.sqlite3
  ```
  
  O Rails também precisa de uma chave secreta exclusiva para criptografar suas sessões. A partir do Rails 4, esta chave secreta é armazenada em ``config/secrets.yml``. Mas primeiro, precisamos gerar uma chave secreta. Execute:
  
  ```bash
  bundle exec rake secret
  ```
  
  Este comando irá gerar uma chave secreta. Copie esse valor para sua área de transferência. Em seguida, abra ``config/secrets.yml``:
  
  ```bash
  vi config/secrets.yml
  ```
  
  Se o arquivo já existir, procure por isso:
  
  ```ruby
  production:
  secret_key_base: <%=ENV["SECRET_KEY_BASE"]%>
  ```
  
  Em seguida, substitua-o pelo seguinte. Se o arquivo ainda não existir, basta inserir o seguinte.
  
  ```ruby
  production:
    secret_key_base: the value that you copied from 'rake secret'
  ```
  
  Para evitar que outros usuários no sistema leiam informações confidenciais pertencentes ao seu aplicativo, vamos reforçar a segurança no diretório de configuração e no diretório do banco de dados:
  
  ```bash
  chmod 700 config db
  chmod 600 config/database.yml config/secrets.yml
  ```

1. Compilar os recursos do Rails e executar migrações de banco de dados

  Execute o seguinte comando para compilar ativos para o pipeline de ativos Rails e para executar migrações de banco de dados:
  
  ```bash
  bundle exec rake assets:precompile db:migrate RAILS_ENV=production
  ```

1. Determine o comando Ruby que o Passenger deve usar

  Precisamos informar ao Passenger qual comando Ruby ele deve usar para executar seu aplicativo, caso existam vários intérpretes de Ruby em seu sistema. Por favor, execute o comando ``assenger-config about ruby-command`` para descobrir qual o interpretador Ruby que você está usando. Por exemplo:
  
  ```bash
  $ passenger-config about ruby-command
passenger-config was invoked through the following Ruby interpreter:
    Command: /usr/local/rvm/gems/ruby-2.6.3/wrappers/ruby
    Version: ruby 2.6.3p62 (2019-04-16 revision 67580) [x86_64-linux]
  ```
  
  Por favor, tome nota do caminho depois de "Comando" (neste exemplo, ``/usr/local/rvm/gems/ruby-2.6.3/wrappers/ruby``). Você precisará dele em um dos próximos passos.

1. Editar arquivo de configuração do Nginx

  Precisamos criar um arquivo de configuração Nginx e configurar uma entrada de host virtual que aponte para seu aplicativo. Essa entrada de host virtual informa ao Nginx (e ao Passenger) onde seu aplicativo está localizado.

  ```bash
  sudo nano /etc/nginx/sites-enabled/myapp.conf
  ```
  
  Substitua ``myapp`` pelo nome do seu aplicativo.
  
  Coloque isso dentro do arquivo:
  
  ```bash
  server {
    listen 80;
    server_name yourserver.com;

    # Tell Nginx and Passenger where your app's 'public' directory is
    root /var/www/myapp/code/public;

    # Turn on Passenger
    passenger_enabled on;
    passenger_ruby /path-to-ruby;
  }
  ```
  
  Substitua ``yourserver.com`` pelo nome de host do seu servidor e substitua ``/var/www/myapp/code`` pelo caminho do diretório de código do seu aplicativo. No entanto, certifique-se de que o Nginx esteja configurado para apontar para o subdiretório público dentro dele!

  Substitua ``/path-to-ruby`` pelo comando Ruby obtido.

  Quando terminar, reinicie o Nginx:
  
  ```bash
  sudo service nginx restart
  ```  

Fontes: [Rails Girls Guide](https://guides.railsgirls.com/app), [Passenger Library](https://www.phusionpassenger.com/library/walkthroughs/deploy/ruby/ownserver/nginx/oss/stretch/deploy_app.html)
