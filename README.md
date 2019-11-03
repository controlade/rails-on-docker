# README

## Highlights
- Puedes unificar la experiencia de desarrollo en equipo teniendo ambientes replica de produccion que estan disponibles para trabajar en segundos.
- No necesitas tener ni ruby ni rvm ni rbenv instalado en local dev machine.
- Docker hace la magia para que todo se ejecute en el container, con las versiones correctas.
- Los cambios hechos en local dev machine se actualizan en el Docker container de manera transparente, sin necesidad de reiniciar ningun servicio.

## Reference
[Dockerizing a Ruby on Rails Application](https://semaphoreci.com/community/tutorials/dockerizing-a-ruby-on-rails-application)

## Requirements
El proyecto fue construido usando:

  * macOS 10.14.2
  * Docker Desktop Version 2.0.0.0-mac81 (29211)
  * git version 2.15.0 for macOS

## Steps

#### Descargar el proyecto

Clonar este repositorio en la maquina de desarrollo
```
$ git clone git@github.com:controlade/rails-on-docker.git
$ cd rails-on-docker
```

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

#### Edit project’s files

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

#### Dockerize Rails App

Add following files:

-  Dockerfile (change postgresql-client-9.4 to postgresql-9.5 to avoid an error)
- .dockerignore
- Docker-compose.yml

#### Create Volumes for Postgres and Redis
```
$ docker volume create --name drkiq-postgres
$ docker volume create --name drkiq-redis
```

#### Run everything
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

_ActiveRecord::NoDatabaseError_ is expected due to database has not been initialized yet.  Go to next step.

#### Initialize Database
```
$ docker-compose run drkiq rake db:reset
$ docker-compose run drkiq rake db:migrate
```

#### Run everything (again)
```
$ docker-compose up > docker-compose-run-2.txt
```

## Testing it out
```
# start services, and leave their terminals open to see logs
$ docker-compose up
```

Go to http://localhost:8000/ on web browser

You should be greeted with the typical Rails introduction page, if not initialize database again.

To stop the test, do ```control + C``` to stop containers

## Notes
- To run Rails commands we need to prepend the following:

```
$ docker-­compose run drkiq rails [command, arguments and options]
```
For example:
```
docker-compose run drkiq rails console
```

