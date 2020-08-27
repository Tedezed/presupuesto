### Instalando en local

Para instalar la aplicación en local es necesario seguir los siguientes pasos:

* Instalar Python. La aplicación debería funcionar con Python 2.6, pero el desarrollo y los despliegues actuales usan 2.7.x, que es la versión recomendada.

* Instalar los componentes utilizados por la aplicación. Actualmente, la aplicación requiere coffin 0.4.0, así como django 1.4.2:
    
        $ pip install -r requirements/local.txt

    Incompatibilidad con otras versiones de coffin y django:
    * En coffin > 0.4.0 desaparece `coffin.common.env`.
    * En django > 1.6 desaparece el argumento [deprecado][4] `mimetype`.
    * En django > = 1.5 desaparece `django.views.generic.simple`.
    * En django < 1.4.2 no se ha incorporado aún la [compatibilidad][5] con `django.utils.six`.

* Borrar base de datos:

        $ dropdb -h localhost presupuestos

* Crear la base de datos:

        $ createdb -h localhost presupuestos

* Copiar `local_settings.py-example` a `local_settings.py` y modificar las credenciales de la base de datos. Elegir la carpeta donde se alojará el THEME o aspecto visual del proyecto.

* Crear el esquema de la base de datos y cargar los datos básicos:

        $ python manage.py syncdb
        $ python manage.py migrate

        $ python manage.py load_glossary
        $ python manage.py load_entities
        $ python manage.py load_stats
        $ python manage.py load_budget 2014

### Adaptando el aspecto visual

La aplicación soporta el concepto de 'themes' capaces de modificar el aspecto visual de la web: tanto recursos estáticos (imágenes, hojas de estilo...) como las plantillas que generan el contenido de la web. El repositorio [`presupuesto-dvmi`](https://github.com/civio/presupuesto-dvmi) de Civio -una adaptación del software de Aragón Open Data a los Presupuestos Generales del Estado- es un buen ejemplo de cómo puede organizarse el contenido de un theme. Si su proyecto tiene ámbito municipal puede basarse en el repositorio de [`presupuesto-torrelodones`](https://github.com/civio/presupuesto-torrelodones)

El theme a usar se configura mediante la variable `THEME` en local_settings.py. Es referenciada en diversos puntos de `settings.py` para instalar los directorios del theme (plantillas y recursos estáticos) justo antes de los de la aplicación principal.

Es necesario compilar todos los recursos estáticos. Para ello:

* Instalar herramientas de compresión y generación de bundles:

		$ npm install rollup
		$ ./node_modules/rollup/bin/rollup -c
		$ cd ./<directorio_del_theme>
		$ npm install
		$ npm install node-sass
		$ npm run css-build

### Configurando el buscador

Por defecto la aplicación usa el método estándar de búsqueda de texto de Postgres. Es posible crear métodos de búsqueda adaptados a un idioma concreto, de forma que -por ejemplo- Postgres ignore los acentos a la hora de buscar resultados. Si deseamos configurar la búsqueda para funcionar en español, creamos primero una nueva configuración de búsqueda, como se explica en la [documentación de Postgres](http://www.postgresql.org/docs/9.1/static/textsearch-configuration.html):

    $ psql presupuestos_dev

    > CREATE EXTENSION unaccent;

    > CREATE TEXT SEARCH CONFIGURATION unaccent_spa ( COPY = pg_catalog.spanish );

    > ALTER TEXT SEARCH CONFIGURATION unaccent_spa
        ALTER MAPPING FOR hword, hword_part, word
        WITH unaccent, spanish_stem;

Mientras hacemos pruebas en `psql` podemos configurar el método de búsqueda por defecto:

    > SET default_text_search_config = 'unaccent_spa';

Pero para usarlo de manera regular debemos configurar la aplicación, vía `local_settings.py`:

    'SEARCH_CONFIG': 'unaccent_spa'

### Arrancando el servidor

* Arrancar el servidor

        $ python manage.py runserver

* Arrancar el servidor en modo livereload:

        $ python manage.py livereload

[4]: https://docs.djangoproject.com/en/1.7/internals/deprecation/#deprecation-removed-in-1-7
[5]: https://docs.djangoproject.com/en/1.5/topics/python3/#philosophy

### Docker

Uso de la aplicación con docker:

* Compilar imagen docker:
```
docker-compose build
```

* Levantar entorno en local usando docker:
```
docker-compose up
```

* Ejecutar terminal sh en contenedor
```
docker exec -it presupuesto_presupuesto_1 sh
```

* Conexión local usando: `http://127.0.0.1:8000`
