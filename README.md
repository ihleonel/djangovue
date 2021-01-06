# Django + Vue
## Proceso de creación de un proyecto básico

Iniciamos creando los siguientes archivos:

- requirements.txt
- Dockerfile
- docker-compose.yml
- .env

El archivo requirements.txt tendrá las dependencias python necesarias para nuestro proyecto

```txt
django==3.1
gunicorn==20.0.4
psycopg2==2.8.5
python-decouple==3.3
```
En el archivo Dockerfile definimos la imagen base que usaremos para levantar el servicio donde estará corriendo django
```dockerfile
FROM python:3.8.5
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
COPY . /code/
```
El archivo docker-compose.yml tendrá definido los servicios que se levantarán en este sistema.
```yml
version: '3'

services:
  db:
    image: postgres:9.6.18
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
      PGDATA: /var/lib/postgresql/data/pgdata
    ports:
      - ${DB_PORT}:${DB_PORT}
    networks:
      - djangonetwork

  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - 8000:8000
    depends_on:
      - db
    networks:
      - djangonetwork

networks:
  djangonetwork:
      driver: bridge

```
En el archivo .env definiremos variables de entorno que servirán para configurar nuestro sistema. Los valores que se presentan en el archivo en este caso solo son de ejemplo.
```txt
SECRET_KEY=secret
DEBUG=True
DB_NAME=homestead
DB_ENGINE=django.db.backends.postgresql_psycopg2
DB_USER=user
DB_PASSWORD=secret
DB_HOST=db
DB_PORT=5432
```

Una vez creados estos archivos pasamos a crear nuestro proyecto django. En la raíz de nuestro proyecto, donde tenemos nuestro archivo `docker-compose.yml` ejecutamos el siguiente comando:
```sh
docker-compose run web django-admin.py startproject djangovue .
```
El directorio debería quedar de la siguiente forma:
```sh
djangovue/
├── djangovue
│   ├── asgi.py
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── docker-compose.yml
├── Dockerfile
├── manage.py
└── requirements.txt
```

Como el proyecto fue creado usando el servicio "web" definido en `docker-compose.yml` los archivos tendrán como propietario al usuario `root`:
```sh
drwxr-xr-x  4 jarvis jarvis 4096 ene  5 19:39 .
drwxr-xr-x 46 jarvis jarvis 4096 ene  4 21:24 ..
drwxr-xr-x  2 root   root   4096 ene  5 19:39 djangovue
-rw-r--r--  1 jarvis jarvis  543 ene  5 19:26 docker-compose.yml
-rw-r--r--  1 jarvis jarvis  150 ene  5 19:26 Dockerfile
-rw-r--r--  1 jarvis jarvis  152 ene  5 19:27 .env
-rwxr-xr-x  1 root   root    665 ene  5 19:39 manage.py
-rw-r--r--  1 jarvis jarvis  325 ene  5 21:14 README.md
-rw-r--r--  1 jarvis jarvis   66 ene  4 20:37 requirements.txt
```
Para no tener problemas de permisos al momento de modificar estos archivos y carpetas cambiaremos sus propietario y grupo, en la raíz del proyecto ejecutamos el comando:
```sh
sudo chown -R $USER:$USER .
```
Ahora procederemos a editar el archivo `settings.py`. Cambiaremos las siguientes variables:
```python
from decouple import config

SECRET_KEY = config('SECRET_KEY')

DEBUG = config('DEBUG', cast=bool)

ALLOWED_HOSTS = ['*']

DATABASES = {
    'default': {
        'ENGINE': config('DB_ENGINE'),
        'NAME': config('DB_NAME'),
        'USER': config('DB_USER'),
        'PASSWORD': config('DB_PASSWORD'),
        'HOST': config('DB_HOST'),
        'PORT': config('DB_PORT')
    }
}
```
Las variables toman los valores definidos en el archivo `.env`.

> La variable `SECRET_KEY` también toma el valor configurado en el `.env` este es un valor muy importante y que no debería ser compartido.

Una vez configurado el archivo `settings.py` pasaremos a iniciar todos los servicios especificados en el `docker-compose.yml`. Corremos el comando:

```sh
docker-compose up
```

Si todo salio bien debería estar corriendo nuestro sistema en http://localhost:8000
