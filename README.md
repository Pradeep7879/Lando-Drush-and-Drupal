# Lando-Drush-and-Drupal

**What is lando?**
Lando is for developers. using lando we quickly specify and painlessly spin up the services and tools needed to develop our projects. It's a free, open source, cross-platform, local development environment and DevOps tool built on Docker container technology.

**Configure Lando for project.**
----
run command
1.  lando init
2.  lando build/rebuid(you will need to lando rebuild your app for those changes to be applied.)
3.  lando info
4.  lando db-import file_name.sql
5.  lando db-export

It create the .lando.yml file, all configuration used in your project, find here.

example file below .lando.yml
----------------------------

    name: drupal-project-name
    recipe: drupal9
    config:
      via: nginx  
      webroot: docroot  
      database: mariadb:10.4    
      composer_version: '2.0.8'  
      php: '7.4'  
      drush: '^10'  
      xdebug: false


    services:
      appserver:
        config:
          php: lando/php.ini
        scanner: false
        overrides:
          environment:
            MYSQL_HOSTNAME: 'database'
            MYSQL_DATABASE: 'drupal9'
            MYSQL_PASSWORD: 'drupal9'
            MYSQL_PORT: '3306'
            MYSQL_USER: 'drupal9'
            DRUSH_OPTIONS_URI: 'http://drupal-project.lndo.site'
            DRUSH_OPTIONS_ROOT: '/app'
            PHP_IDE_CONFIG: 'serverName=drupal-project.lndo.site'
            COMPOSE_HTTP_TIMEOUT: 3600
            PWD: '/app'
            MINK_DRIVER_ARGS: '["chrome", null, "http://chromedriver:4444/wd/hub"]'
            MINK_DRIVER_ARGS_WEBDRIVER: '["chrome", null, "http://chromedriver:4444/wd/hub"]'
            MINK_DRIVER_CLASS: 'Drupal\FunctionalJavascriptTests\DrupalSelenium2Driver'
        build_as_root:
          - curl -sL https://deb.nodesource.com/setup_12.x | bash -
          - apt-get install -y nodejs
          - npm install -g grunt-cli
        build:
          #- /app/lando/composer/preinstall.sh
          - /app/lando/appserver.build.sh

      node:
        type: node:12
        ssl: false
        scanner: false
        port: 6006
        overrides:
          environment:
            NODE_ENV: "development"
        command: tail -f /dev/null

      mailhog:
        type: mailhog
        hogfrom:
          - appserver

      phpcs:
        type: compose
        services:
          image: pathtoproject/phpcs-drupal:latest
          command: tail -f /dev/null

    proxy:
      mailhog:
        - drupal-ugm.mh.lndo.site

    tooling:
      phpcs:
        service: phpcs
        description: 'Run PHPCS locally'
        dir: /app
        cmd:
          - phpcs

      blt:
        service: appserver
        cmd: /app/vendor/bin/blt

      phpcbf:
        service: phpcs
        description: 'Run PHPCBF locally'
        dir: /app
        cmd:
          - phpcbf

      npm:
        service: appserver
        description: 'Run npm locally'
        cmd:
          - npm

----------------------

   Some Useful link to setup the Lando
   
     http://www.learnwebtech.in/install-drupal-9-using-composer-with-lando/
     
     
----------------------

other way to create Lando.yml file with node, npm and gulp using lando
    
    name: d9-d
    recipe: drupal9
    config:
      webroot: web
      xdebug: true
    services:
      database:
        type: mysql
        portforward: true
        healthcheck: ""
      appserver:
        type: php:7.4
        build_as_root:
          - a2enmod headers proxy proxy_http proxy_fcgi proxy_ajp
      solr:
        type: solr:7
        portforward: true
        # config:
        #   dir: web/modules/contrib/search_api_solr/jump-start/solr7/config-set
      node:
        type: node:14
        # build:
        #   - npm install
        #   - gulp
        globals:
          gulp-cli: latest
    tooling:
      npm:
        service: node
      node:
        service: node
      gulp:
        service: node

