Sphinx
====== 

Esta es una herramienta excelente para documentar 
proyectos de Python, pero también para escribir libros 
como éste.  

Es recomendable crear un ambiente de Python para la instalación de los 
requerimientos.

Si usamos `miniconda` (con los ambientes en el directorio `/data/envs/`): 

:: 

    conda create --prefix /data/envs/sphinx-docs python=3.11

Activamos el ambiente

::

   conda activate /data/envs/sphinx-docs 

Agegamos los requerimentos en un archivo `pyproject.toml`:

.. code-block:: toml
   :caption: ``pyproject.toml``

    [build-system]
    requires = ["setuptools>=61.0"]
    build-backend = "setuptools.build_meta"

    [project]
    name = "progweb-docs"
    version = "0.1.0"
    requires-python = ">=3.10"
    dependencies = [
      "sphinx>=7.2",
      "furo>=2024.1.29",
      "myst-parser>=3.0.0",
      "sphinxcontrib-bibtex",
      "sphinx-book-theme>=1.1.4",
      "sphinx-copybutton",
      "sphinx-new-tab-link",
    ]

Instalamos dentro del ambiente:

:: 

    python -m pip install -e . 


En el directorio raíz de la documentación:

:: 

    sphinx-quickstart

Por ejemplo:

::

    Separate source and build directories (y/n) [n]: y
    Project name: Guías de Laboratorio
    Author name(s): Mario Garcia
    Project release []: 0.1
    Project language [en]: es

Para generar el html:

::
    
    make html

Para ver la primera versión de la documentación:

::

    xdg-open build/html/index.html

Me gusta más el template de libro, para esto editamos el archivo ``conf.py``
agregando:

.. code-block:: python

    extensions = ['sphinx_copybutton','sphinx_new_tab_link']

    copybutton_prompt_text = r">>>|\.\.\."
    copybutton_prompt_is_regexp = True

    templates_path = ['_templates']
    exclude_patterns = []

    html_theme = 'sphinx_book_theme'
    html_static_path = ['_static']

Para tener un servidor el html estático generado por Sphinx, podemos instalar 
``reloadserver``:

::
  
    pip install reloadserver

Y corremos ``reloadserver`` en el directorio raíz del html.


Una buena guía sobre `reStructuredText` es la propia documentación de `Sphinx`_. 


Publicar el libro en GitHub Pages
=================================

Una vez que nuestro libro de Sphinx se encuentra en un repositorio de GitHub,
podemos publicarlo automáticamente como un sitio web utilizando GitHub Actions y
GitHub Pages.

GitHub Pages permite publicar sitios estáticos directamente desde un repositorio.
En nuestro caso, el sitio estático será el HTML generado por Sphinx. 

La idea general del proceso es la siguiente:

#. Subimos el proyecto de Sphinx a GitHub.
#. GitHub Actions instala Python y las dependencias del proyecto.
#. Sphinx genera el sitio HTML.
#. GitHub Pages publica el contenido generado.

Estructura esperada del proyecto
--------------------------------

Para esta guía asumiremos una estructura similar a la siguiente:

.. code-block:: text

   lab-guide/
   ├── .github/
   │   └── workflows/
   │       └── pages.yml
   ├── source/
   │   ├── conf.py
   │   ├── index.rst
   │   └── ...
   ├── Makefile
   ├── make.bat
   └── pyproject.toml

El archivo ``pyproject.toml`` debe incluir las dependencias necesarias para
construir el libro. Por ejemplo:

.. code-block:: text

   sphinx
   sphinx-rtd-theme

Si el proyecto usa otras extensiones o temas, también deben agregarse a este
archivo.

.. note::

   El nombre exacto del directorio puede variar. Algunos proyectos usan
   ``source/`` para los archivos fuente de Sphinx, mientras que otros colocan
   ``conf.py`` e ``index.rst`` directamente en la raíz del repositorio. Lo
   importante es que el comando ``make html`` funcione correctamente antes de
   configurar GitHub Actions.

Probar la compilación local
---------------------------

Antes de publicar el libro, conviene verificar que Sphinx pueda generar el HTML
de manera local.

Desde la raíz del proyecto ejecutamos:

.. code-block:: console

   make html

Si todo funciona correctamente, Sphinx generará los archivos HTML dentro del
directorio:

.. code-block:: text

   build/html/

En Windows, si no se usa ``make``, puede ejecutarse:

.. code-block:: console

   .\make.bat html

También es posible ejecutar Sphinx directamente:

.. code-block:: console

   sphinx-build -b html source _build/html

En este último comando, ``source`` es el directorio donde se encuentra
``conf.py`` y ``_build/html`` es el directorio de salida.

Crear el archivo de GitHub Actions
----------------------------------

GitHub Actions utiliza archivos ``.yml`` para definir flujos de trabajo
automatizados. Para publicar el libro, crearemos el archivo:

.. code-block:: text

   .github/workflows/pages.yml

con el siguiente contenido:

.. code-block:: yaml

   name: Publicar libro Sphinx en GitHub Pages

   on:
     push:
       branches:
         - main
     workflow_dispatch:

   permissions:
     contents: read
     pages: write
     id-token: write

   concurrency:
     group: pages
     cancel-in-progress: false

   jobs:
     build:
       runs-on: ubuntu-latest

       steps:
         - name: Descargar repositorio
           uses: actions/checkout@v4

         - name: Configurar Python
           uses: actions/setup-python@v5
           with:
             python-version: '3.12'

         - name: Instalar dependencias
           run: |
             python -m pip install --upgrade pip
             python -m pip install -e .

         - name: Construir documentación
           run: |
             make html

         - name: Preparar GitHub Pages
           uses: actions/configure-pages@v5

         - name: Subir sitio generado
           uses: actions/upload-pages-artifact@v3
           with:
             path: build/html

     deploy:
       environment:
         name: github-pages
         url: ${{ steps.deployment.outputs.page_url }}

       runs-on: ubuntu-latest
       needs: build

       steps:
         - name: Publicar en GitHub Pages
           id: deployment
           uses: actions/deploy-pages@v4


.. _Sphinx: https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html


