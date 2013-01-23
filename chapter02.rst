======================
Capítulo 2: Começando
======================

Instalar o Django é um processo de vários passos, em função das múltiplas partes que
envolve os cenários de desenvolvimento web hoje em dia. Neste capítulo, vamos leva-los através de 
como se instalar o framework e as suas dependencias.

Como o Django é "apenas" código Python, ele é executado em qualquer lugar que rode Python -- incluindo
alguns telefones celulares! Mas este capítulo abrange apenas os cenários comums para se 
instalar o Django. Vamos assumir que você esteje instalando em uma máquina desktop/laptop ou um servidor.

Depois, no Capítulo 12, nós iremos cobrir como fazer o deploy do Django para um site em produção.

Instalando o Django
====================

Django é puramente escrito em Python, então o primeiro passo para instalar o 
framework é ter certeza de que você tem o Python instalado.

Versões do Python
------------------

O núcleo do framework Django (versão 1.4) trabalha com qualquer versão do Python da 2.5
até a 2.7. O GIS(Geographic Information Systems) do Django exige o Python 2.5 até o 2.7.

Se você não tem certeza da versão do Python que que tem instalada e você tem
completa liberdade para tomar essa decisão, escolha a última versão da série 2.x: versão 2.7.
Embora Django funcione igualmente bem com qualquer versão 2.5 até 2.7, a ultima 
versão do Python tem boa performace de desempenho e idiomas adicionais
características que talvez você deseje ultilizar em suas aplicações. Além disso, 
add-ons de terceiros que você possa querer ultilizar talvez nescessite da mais recente que 
o Python 2.5, então ultilize a ultima versnao do Python mantendo o leque de opções em aberto.

.. admonition:: Django and Python 3.0

    No momento em que este livro estava sendo escrito, o Python 3.0 estava sendo lançado, mas Django
    foi testado apenas de experimentalmente. Python 3.0 intruduziu a 
    um número substancial de incompatibilidades com as versões anteriores, mundando 
    a si mesmo, e, como resultado, muita das principais bibliotecas Python e
    frameworks, incluindo Django, ainda não tinha pego.

    Django 1.5 dará suporte para o Python 2.6, 2.7, e 3.2. Entretando,
    suporte para o Python 3.2 é considerado um "preview", o que significa que
    os desenvolvedores Django não estão confiantes o suficiente para prometer
    estabilidade da produção. Para isso sugiro que você espere até a versão do Django 1.6 

Instalação
-----------

Se você estiver no Linux ou no Mac OS X, você provavelmente tem o Python instalado.
Escreva ``python`` no prompt de comando (ou em Applications/Utilities/Terminal, no Mac OS X).
Se você vê algo como isso, é porque o Python está instalado::

    Python 2.7.3rc2 (default, Apr 22 2012, 22:30:17)
    [GCC 4.6.3] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>>

Caso contrário, você prescisa fazer o download e instalar o Python. Isso é fácil e rápido
instruções detalhadas poderão ser encontradas em http://www.python.org/download/

Instalando Django
=================

Em um determinado momento, duas dinstintas versões dinstintas do Django estão disponíveis para você:
a última versão oficial e a versão de desenvolvimento. A versão que você decidir instalar 
depende das suas prioridades. Se você quer uma versão testada e estável, ou se você quer uma
versão contendo os recursos mais recentes. Talvez você possa contribuir para o Django, á custa de
estabilidade?

Nós recomendamos trabalhar com um lançamento oficial, mas é importante saber que a versão 
de desenvolvimento existe, porque você irá encontrar menções na documentação e pelos 
membros da comunidade.

Instalando um lançamento oficial
------------------------------

Um lançamento oficial tem um número de versão, como 1.4.2, 1.4.1 ou 1.4, 
um lançamento não oficial tem um número de versão, como 1.4.2, 1.4.1 ou 1.4, e a última 
estará sempre disponível em http://www.djangoproject.com/download/.

Se você está em uma distribuição Linux que inclui um pacote do Django, é uma boa ideia
usar a versão do distribuidor. Dessa forma você terá atualizações de segurança juntamente com
o resto dos pacotes do seu sistema.

Se você não tem acesso a uma versão pré-empacotada, você pode fazer o download e instalar
o framework manualmente. Para isso, primeiramente faça o do tarball, que será chamado
de algo como ``Django-1.4.2.tar.gz``. (Não importa em qual diretório local você tenha baixado
este arquivo inicial; O processo de instalação colocará os arquivos do Django no lugar correto.) 
Então, descompacte-o e e execute ``setup.py install``, como você faz com a maioria das bibliotecas
do Python.

Veja como esse processo se parece em sistemas Unix:

#. ``tar xzvf Django-1.4.2.tar.gz``
#. ``cd Django-*``
#. ``sudo python setup.py install``

No Windows, nós recomendamos o uso do 7-Zip (http://www.djangoproject.com/r/7zip/)
para descomprimir os arquivos ``.tar.gz``. Uma vez com os arquivos descompactados, inicie o DOS
shell (o "Prompt de Comando") com os privilégios de administrador e execute o comando seguinte 
dentro do diretório cujo nome começa com ``Django-``::

    python setup.py install

No caso de você estiver curioso: os arquivos do Django serão instalados no local da
instalação do seu Python no ``site-packages`` diretório -- um diretório aonde o o Python
procura por biblioteca de terceiros. Normalmente é um lugar como ``/usr/lib/python2.7/site-packages``.

Instalando a versão de "Desenvolvimento"
----------------------------------------

Django ultiliza o Git (http://git-scm.com) para o controle de versão. A mais
recente e maior versão de desenvolvimento do Django se encontra no seu oficial 
repositório do Git (https://github.com/django/django). Você deve considerar a 
instalação dessa versão se você quer trabalhar on the bleeding edge, ou se você quiser 
contribuir com o código do Django.

Git é gratuíto, um uma distribuição de código live de revisão de controle de sistema, e o
time do Django usa ele para controlar as mudanças no código fonte do Django. Você pode fazer
o download e instalar o Git no http://git-scm.com/download, mas é mas fácil instalar com
o controlador de pacots do seu sistema operacional. Você pode usar o Git para pegar a mais 
recente versão dos códigos do Django e, a qualquer momento, você pode atualizar os códigos 
da sua versão local do Django para obter as últimas alterações feitas pelos os desenvolvedores
Django.

Ao usar a versão de desenvolvimento, tenha em mente que não há coisas como garantias de que
não será quebrada a qualquer momento. Como dito, alguns membros do time do Django executa sites
de produção em versões de desenvolvimento, para que eles tenham um incentivo de mantê-los estáveis.

Para pegar a última versão do Django, siga esses passos:

#. Tenha certeza que você tenha o Git instalado. Você pode pegá-lo gratuitamente em http://git-scm.com/,
e você pode achar a excelente documentação em http://git-scm.com/documentation.

#. Clone o repositório usando o seguinte comando ``git clone https://github.com/django/django djmaster``

#. Localize na instalação do Python o diretório ``site-packages``. Normalmente
   fica localizada em um lugar como ``/usr/lib/python2.7/site-packages``. Se você não faz idéia,
   escreva isso no prompt de comando::

       python -c 'import sys, pprint; pprint.pprint(sys.path)'

    A saída resultante deverá incluir o diretório do seu ``site-packages``.

#. Dentro do diretório ``site-packages``, crie um arquivo chamado ``djmaster.pth``
  e edite ele para conter o caminho completo para o seu diretório ``djmaster``.
  Por exemplo, o arquivo poderia conter essa linha::

       /path/to/djmaster

#. Coloque ``djmaster/django/bin`` em seu PATH do sistema. Este diretório inclui
   inclui utilitários de administração, tais como o ``django-admin.py``.

.. admonition:: Dica:

    Se arquivos ``.pth`` são novos para você, você pode aprender mais sobre eles em
    http://www.djangoproject.com/r/python/site-module/.

Depois de baixar pelo Git e seguir os passos anteriores, não há nescessidade de 
executar ``python setup.py install``-- você acabou de fazer o trabalho manualmente!

Como o código do Django muda frequentemente com correção de bugs e adção de recurso,
você provavelmente queira atualiza-lo de vez em quando. Para atualizar o código, 
apenas execute o comando ``git pull origin master`` dentro do diretório ``djmaster``.
Quando você executar o comando, o Git entrará em contato com https://github.com/django/django,
determinando se algo código fonte foi mudado, e atualizando a sua versão local com as alterações
que tem sido feitas desde o último update. It's quite slick.

Finalmente, se você usa a versão de desenvolvimento do Django, você deve saber como 
qual a versão do Django está sendo executada. Conhecer o número da sua versão será importante
se você nescessitar correr até a comunidade para ajuda, ou se você apresentar melhorias 
para o framework. Nestes casos, você deverá dizer as pessoas que revisionam, também conhecido como "commit", 
o que você está usando. Para saber o seu atual "commit", digite "git log -1" de dentro de um diretório ``django``, e 
olhe para os identificadores depois de "commit". Este número muda toda vez que o Django é alterado,
seja por meio de uma correção de bug, adição de funcionalidade, documentação ou melhoria de qualquer coisa.

Testando a instalação do Django
================================

Para um feedback positivo de pós instalação, pare um instante para ver se a instalação
está funcionando. Em um shell de comando, mude para outro diretório (ex., *não* o diretório que que
contém o diretório ``django``) e come
For some post-installation positive feedback, take a moment to test whether the
installation worked. In a command shell, change into another directory (e.g.,
*not* the directory that contains the ``django`` directory) and start the
Python interactive interpreter by typing ``python``. If the installation was
successful, you should be able to import the module ``django``:

    >>> import django
    >>> django.VERSION
    (1, 4, 2, 'final', 0)

.. admonition:: Interactive Interpreter Examples

    The Python interactive interpreter is a command-line program that lets you
    write a Python program interactively. To start it, run the command
    ``python`` at the command line.

    Throughout this book, we feature example Python interactive interpreter
    sessions. You can recognize these examples by the triple
    greater-than signs (``>>>``), which designate the interpreter's prompt. If
    you're copying examples from this book, don't copy those greater-than signs.

    Multiline statements in the interactive interpreter are padded with three
    dots (``...``). For example::

        >>> print """This is a
        ... string that spans
        ... three lines."""
        This is a
        string that spans
        three lines.
        >>> def my_function(value):
        ...     print value
        >>> my_function('hello')
        hello

    Those three dots at the start of the additional lines are inserted by the
    Python shell -- they're not part of our input. We include them here to be
    faithful to the actual output of the interpreter. If you copy our examples
    to follow along, don't copy those dots.

Setting Up a Database
=====================

At this point, you could very well begin writing a Web application with Django,
because Django's only hard-and-fast prerequisite is a working Python
installation. However, odds are you'll be developing a *database-driven* Web
site, in which case you'll need to configure a database server.

If you just want to start playing with Django, skip ahead to the
"Starting a Project" section -- but keep in mind that all the examples in this
book assume you have a working database set up.

Django supports four database engines:

* PostgreSQL (http://www.postgresql.org/)
* SQLite 3 (http://www.sqlite.org/)
* MySQL (http://www.mysql.com/)
* Oracle (http://www.oracle.com/)

For the most part, all the engines here work equally well with the core Django
framework. (A notable exception is Django's optional GIS support, which is much
more powerful with PostgreSQL than with other databases.) If you're not tied to
any legacy system and have the freedom to choose a database backend, we
recommend PostgreSQL, which achieves a fine balance between cost, features,
speed and stability.

Setting up the database is a two-step process:

* First, you'll need to install and configure the database server itself.
  This process is beyond the scope of this book, but each of the four
  database backends has rich documentation on its Web site. (If you're on
  a shared hosting provider, odds are that they've set this up for you
  already.)

* Second, you'll need to install the Python library for your particular
  database backend. This is a third-party bit of code that allows Python to
  interface with the database. We outline the specific, per-database
  requirements in the following sections.

If you're just playing around with Django and don't want to install a database
server, consider using SQLite. SQLite is unique in the list of supported
databases in that it doesn't require either of the above steps. It merely reads
and writes its data to a single file on your filesystem, and Python versions 2.5
and higher include built-in support for it.

On Windows, obtaining database driver binaries can be frustrating. If you're
eager to jump in, we recommend using Python 2.7 and its built-in support for
SQLite.

Using Django with PostgreSQL
----------------------------

If you're using PostgreSQL, you'll need to install either the ``psycopg`` or
``psycopg2`` package from http://www.djangoproject.com/r/python-pgsql/. We
recommend ``psycopg2``, as it's newer, more actively developed and can be
easier to install. Either way, take note of whether you're using version 1 or
2; you'll need this information later.

If you're using PostgreSQL on Windows, you can find precompiled binaries of
``psycopg`` at http://www.djangoproject.com/r/python-pgsql/windows/.

If you're on Linux, check whether your distribution's package-management
system offers a package called "python-psycopg2", "psycopg2-python",
"python-postgresql" or something similar.

Using Django with SQLite 3
--------------------------

You're in luck: no database-specific installation is required, because Python
ships with SQLite support. Skip ahead to the next section.

Using Django with MySQL
-----------------------

Django requires MySQL 4.0 or above. The 3.x versions don't support nested
subqueries and some other fairly standard SQL statements.

You'll also need to install the ``MySQLdb`` package from
http://www.djangoproject.com/r/python-mysql/.

If you're on Linux, check whether your distribution's package-management system
offers a package called "python-mysql", "python-mysqldb", "mysql-python" or
something similar.

Using Django with Oracle
------------------------

Django works with Oracle Database Server versions 9i and higher.

If you're using Oracle, you'll need to install the ``cx_Oracle`` library,
available at http://cx-oracle.sourceforge.net/. Use version 4.3.1 or higher, but
avoid version 5.0 due to a bug in that version of the driver.  Version 5.0.1
resolved the bug, however, so you can choose a higher version as well.

Using Django Without a Database
-------------------------------

As mentioned earlier, Django doesn't actually require a database. If you just
want to use it to serve dynamic pages that don't hit a database, that's
perfectly fine.

With that said, bear in mind that some of the extra tools bundled with Django
*do* require a database, so if you choose not to use a database, you'll miss
out on those features. (We highlight these features throughout this book.)

Starting a Project
==================

Once you've installed Python, Django and (optionally) your database
server/library, you can take the first step in developing a Django application
by creating a *project*.

A project is a collection of settings for an instance of Django, including
database configuration, Django-specific options and application-specific
settings.

If this is your first time using Django, you'll have to take care of some
initial setup. Create a new directory to start working in, perhaps something
like ``/home/username/djcode/``.

.. admonition:: Where Should This Directory Live?

    If your background is in PHP, you're probably used to putting code under the
    Web server's document root (in a place such as ``/var/www``). With Django,
    you don't do that. It's not a good idea to put any of this Python code
    within your Web server's document root, because in doing so you risk the
    possibility that people will be able to view your raw source code over the
    Web. That's not good.

    Put your code in some directory **outside** of the document root.

Change into the directory you created, and run the command
``django-admin.py startproject mysite``. This will create a ``mysite``
directory in your current directory.

.. note::

    ``django-admin.py`` should be on your system path if you installed Django
    via its ``setup.py`` utility.

    If you're using the development version, you'll find ``django-admin.py`` in
    ``djmaster/django/bin``. Because you'll be using ``django-admin.py``
    often, consider adding it to your system path. On Unix, you can do so by
    symlinking from ``/usr/local/bin``, using a command such as ``sudo ln -s
    /path/to/django/bin/django-admin.py /usr/local/bin/django-admin.py``. On
    Windows, you'll need to update your ``PATH`` environment variable.

    If you installed Django from a packaged version for your Linux
    distribution, ``django-admin.py`` might be called ``django-admin`` instead.

If you see a "permission denied" message when running
``django-admin.py startproject``, you'll need to change the file's permissions.
To do this, navigate to the directory where ``django-admin.py`` is installed
(e.g., ``cd /usr/local/bin``) and run the command ``chmod +x django-admin.py``.

The ``startproject`` command creates a directory containing five files::

    mysite/
        manage.py
        mysite/
            __init__.py
            settings.py
            urls.py
            wsgi.py

.. note:: Doesn't match what you see?

    The default project layout recently changed. If you're seeing a
    "flat" layout (with no inner ``mysite/`` directory), you're probably using
    a version of Django that doesn't match this tutorial version. This book covers
    Django 1.4 and above, so if you're using an older version you probably want to
    consult Django's official documentation.

    The documentation for Django 1.X version is available at https://docs.djangoproject.com/en/1.X/.

These files are as follows:

* ``mysite/``: The outer ``mysite/`` directory is just a container for your project.
  Its name doesn't matter to Django; you can rename it to anything you like.

* ``manage.py``: A command-line utility that lets you interact with this
  Django project in various ways. Type ``python manage.py help`` to get a
  feel for what it can do. You should never have to edit this file; it's
  created in this directory purely for convenience.

* ``mysite/mysite/``: The inner ``mysite/`` directory is the actual Python package
  for your project. Its name is the Python package name you'll need to use to
  import anything inside it (e.g. ``import mysite.settings``).

* ``__init__.py``: A file required for Python to treat the ``mysite``
  directory as a package (i.e., a group of Python modules). It's an empty
  file, and generally you won't add anything to it.

* ``settings.py``: Settings/configuration for this Django project. Take a
  look at it to get an idea of the types of settings available, along with
  their default values.

* ``urls.py``: The URLs for this Django project. Think of this as the
  "table of contents" of your Django-powered site.

* ``wsgi.py``: An entry-point for WSGI-compatible webservers to serve your project.
  See How to deploy with WSGI (https://docs.djangoproject.com/en/1.4/howto/deployment/wsgi/) for more details.

Despite their small size, these files already constitute a working Django
application.

Running the Development Server
------------------------------

For some more post-installation positive feedback, let's run the Django
development server to see our barebones application in action.

The Django development server (also called the "runserver" after the command
that launches it) is a built-in, lightweight Web server you can use while
developing your site. It's included with Django so you can develop your site
rapidly, without having to deal with configuring your production server (e.g.,
Apache) until you're ready for production. The development server watches your
code and automatically reloads it, making it easy for you to change your code
without needing to restart anything.

To start the server, change into your project container directory (``cd mysite``),
if you haven't already, and run this command::

    python manage.py runserver

You'll see something like this::

    Validating models...
    0 errors found.

    Django version 1.4.2, using settings 'mysite.settings'
    Development server is running at http://127.0.0.1:8000/
    Quit the server with CONTROL-C.

This launches the server locally, on port 8000, accessible only to connections
from your own computer. Now that it's running, visit http://127.0.0.1:8000/
with your Web browser. You might see a different Django version depending on
which version of Django you have installed. You'll see a "Welcome to Django" page shaded in a
pleasant pastel blue. It worked!

One final, important note about the development server is worth mentioning
before proceeding. Although this server is convenient for development, resist
the temptation to use it in anything resembling a production environment. The
development server can handle only a single request at a time reliably, and it
has not gone through a security audit of any sort. When the time comes to
launch your site, see Chapter 12 for information on how to deploy Django.

.. admonition:: Changing the Development Server's Host or Port

    By default, the ``runserver`` command starts the development server on port
    8000, listening only for local connections. If you want to change the
    server's port, pass it as a command-line argument::

        python manage.py runserver 8080

    By specifying an IP address, you can tell the server to allow non-local
    connections. This is especially helpful if you'd like to share a
    development site with other members of your team. The IP address
    ``0.0.0.0`` tells the server to listen on any network interface::

        python manage.py runserver 0.0.0.0:8000

    When you've done this, other computers on your local network will be able
    to view your Django site by visiting your IP address in their Web browsers,
    e.g., http://192.168.1.103:8000/ . (Note that you'll have to consult your
    network settings to determine your IP address on the local network. Unix
    users, try running "ifconfig" in a command prompt to get this information.
    Windows users, try "ipconfig".)

What's Next?
============

Now that you have everything installed and the development server running,
you're ready to :doc:`learn the basics <chapter03>` of serving Web pages with Django.
