# README

## Highlights
- Puedes unificar la experiencia de desarrollo en equipo teniendo ambientes replica de produccion que estan disponibles para trabajar en segundos.
- No necesitas tener ni ruby ni rvm ni rbenv instalado en local dev machine.
- Docker hace la magia para que todo se ejecute en el container, con las versiones correctas.

## Reference
[Dockerizing a Ruby on Rails Application](https://semaphoreci.com/community/tutorials/dockerizing-a-ruby-on-rails-application)

## Steps

### Descargar el proyecto
Para que quedes con el proyecto en tu máquina, listo para iniciar edicion de codigo:

```$ docker run -it --rm -v "$PWD":/usr/src/app -w /usr/src/app rails:5 rails new --skip-bundle drkiq```

**Notas:**

- La primera vez (si no tienes los containers requeridos) toma más tiempo.
- La segunda vez que lo ejecutas, toma 3 segundos.
- Aplicar git a esto de una vez, si algo sale mal se puede backtrack...

```
$ cd drkiq
$ git init
$ git add .
$ git commit -m ‘initial commit’
```

### Edit project’s files

- Gemfile
- Use puma instead of unicorn
- Add ruby ‘2.5.1’
- Add pg, sidekiq, redis
- config/database.yml
- config/secrets.yml
- config/application.rb
- config/puma.rb
  - Ver: [Deploying Rails Applications with the Puma Web Server #on_worker_boot](https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#on-worker-boot)
- config/initializers/sidekiq.rb
- .drkiq.env
- .gitignore (to ignore .drkiq.env file)

### Dockerize Rails App

Add following files:

-  Dockerfile (change postgresql-client-9.4 to postgresql-9.5 to avoid an error)
- .dockerignore
- Docker-compose.yml

### Create Volumes for Postgres and Redis
```
$ docker volume create --name drkiq-postgres
$ docker volume create --name drkiq-redis
```

### Run everything
```
$ docker-compose up > docker-compose-run-1.txt
```

**Got error:**

```
drkiq_1     | bundler: failed to load command: puma (/usr/local/bundle/bin/puma)
drkiq_1     | URI::InvalidURIError: bad URI(is not URI?): tcp://0.0.0.0:0.0.0.0:8000
```

**Solution:**
Edit .drkiq.env file, remove 0.0.0.0 from the LISTEN_ON env var.

An error is expected due to database has not been initialized yet:

### Initialize Database
```
$ docker-compose run drkiq rake db:reset
$ docker-compose run drkiq rake db:migrate
```

### Run everything (again)
```
$ docker-compose up > docker-compose-run-2.txt
```
