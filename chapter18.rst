==============================================================
Capítulo 18: Integrando Banco de Dados Legados e Aplicações
==============================================================

Django é mais adequado para o chamado desenvolvimento "green-field" - isto é, começando
projetos a partir do zero, como se você estivesse construindo um edifício em um campo fresco
da grama verde. Mas, apesar do fato do Django favorecer projetos a partir do zero,
é possível integrar o framework à bancos de dados legados e
aplicações. Este capítulo explica a algumas estratégias de integração.


Integrando com banco de dados Legados
==================================

A camada de banco de dados do Django gera esquemas SQL a partir do código python -- mas com 
banco de dados legado, já exite os esquemas SQL. Nestes casos, você precisa criar modelos 
para as tabelas existes no banco de dados. Para este casos, Django vem com ferramentas que podem gerar o código 
do modelo lendo o layout das tabelas do banco de dados. Esta ferramenta é a "inspectdb", e pode ser utilizada executando
o comando "manage.py inspectdb".


Usando ``inspectdb``
-------------------

O utilitário "inspectdb"  examinará o banco de dados apontado pelo arquivo de configurações
,que determina uma representação do modelo de Django para cada uma das tabelas, e
imprime o modelo de código Python para a saída padrão.

Uma passagem através de um típico processo de integração com o de banco de dados legado.
As hipóteses são de que apenas o Django está instalado e que você tem um
banco de dados legado.

1. Criar um projeto Django executando ``django-admin.py startproject mysite`` (onde ``mysite`` é o nome do
    seu projeto). Utilizaremos ``mysite`` como o nome do projeto neste exemplo.


2. Editar o arquivo de configuração do projeto, ``mysite/settings.py``,
   para dizer ao Django quais os parâmetros de conexão e o nome do banco de dados.
   Especificamente, fornecer as seguintes configurações 
   ``DATABASE_NAME``, ``DATABASE_ENGINE``, ``DATABASE_USER``,
   ``DATABASE_PASSWORD``, ``DATABASE_HOST``, and ``DATABASE_PORT``.(Note que algumas configurações são opcionais. Leia o capítulo 5 para maiores informações)
   


3. Criar uma aplicação Django no seu projeto executando ``python mysite/manage.py startapp myapp``
   (onde ``myapp`` é o nome da aplicaçõa). Utilizaremos ``myapp`` como o nome da aplicação neste exemplo.
   

4. Executar o comando ``python mysite/manage.py inspectdb``. Ele examinará 
   as tabelas do banco de dados ``DATABASE_NAME`` e e imprimir a classe modelo gerado para cada tabela.
   Dê uma olhada na saída para ter uma idéia do que `` inspectdb `` pode fazer.
   

5. Salvar a saída no arquivo ``models.py`` da sua aplicação utilizando o redirecionamento de 
saída padrão do shell ::

       python mysite/manage.py inspectdb > mysite/myapp/models.py
       

6. Editar o arquivo ``mysite/myapp/models.py`` para limpar os modelos gerados e fazer
   as customizações necessárias. Daremos algumas dicas sobre isso na próxima seção.
   

Limpando os Modelos gerados
----------------------------

Como você deve esperar, a instrospecção no banoc de dados não é perfeita, e é necessário fazer alguma limpeza no código 
resultante do modelo. Aqui estão algumas dicas para lidar com os modelos gerados:

1. Cada tabela do bando de dados é convertida em uma classe do modelo. (isto é, existe um mapeamento um-para-um entre
   as tabelas do banco de dados e a classe do modelo. Isto quer dizer que será necessário refazer
   os modelos muitos-para-muitos das tabelas para objetos ``ManyToManyField`` 
   

2. Cada modelo gerado tem um atributo para cada campo, incluindo um campo de chave primária
   ``id``. No entanto, lembre-se que o Django adiciona automaticamente campo de chave primária
   ``id``  se um modelo não tiver uma chave primária.
    Assim, você vai querer remover todas as linhas que se parecem com isso ::
   
       id = models.IntegerField(primary_key=True)

   Não apenas estas linhas redundantes, mas também podem causar problemas se sua 
   aplicação adicionar *novos* registros a estas tabelas.
   

3. Cada tipo (por exemplo, ``CharField`` , ``DateField`` ) é determinado 
   analisando o tipo da coluna no banco de dados (por exemplo, ``VARCHAR`` , ``DATE`` ). Se o
   ``inspectdb`` não puder mapear o tipo da coluna para um tipo de campo no modelo, então será usado
   ``TextField`` e será inserido um comentário Python ``O tipo deste campo é uma sugestão.`` ,próximo ao campo do modelo gerado.
   . Fique atento a isso e mude o campo de acordo com suas necessidades.

   Se o campo no seu banco de dados não tiver nenhum equivalente no Django
   você pode deixa-lo desabilitado. Não é obrigatório incluir todos os campos da sua tabela
   na camada do modelo do Django.

4. Se o nome da coluna no banco de dados é uma palavra reservada em Python (como ``pass``,
   ``class``, ou ``for``), o ``inspectdb`` irá inserir ``'_field'`` ao nome do atributo e irá colocar
   ``db_coluna`` no nome do campo (por exemplo, ``pass``, ``class``, or ``for``).

   Por exemplo, se a tabela tem uma coluna chamada do tipo ``INT`` chamada ``for``,o modelo gerado terá um
   campo desta maneira::

       for_field = models.IntegerField(db_column='for')

   ``inspectdb`` irá inserir um comentário em Python
   ``'Campo renomeado pois essa é uma palavra reservada em Python.'`` próximo ao campo.

5. Se o banco de dados tiver tabelas que referenciam outras tabelas (como a maioria
   dos banco de dados fazem), talvez seja necessário reorganizar a ordem dos modelos gerados,
   de modo que os modelos que remetem a outros modelos sejam ordenados corretamente.
   Por exemplo, se o modelo ``Book`` tem uma ``ForeignKey`` para o modelo ``Author``, o
   modelo ``Author`` deve ser definido antes do modelo ``Book``. Se você precisa 
   criar um relacionamento em um modelo que ainda não foi definido, você pode usar um texto contendo 
   o nome do modelo ao invés do objeto modelo propriamente dito.

6. ``inspectdb`` detecta as chaves primárias do PostgreSQL, MySQL, and SQLite.
   Ou seja, ele insere ``primary_key=True`` onde é apropriado. Para os outros banco de dados
   ,você precisa inserir ``primary_key=True`` para pelo menos um campo em cada modelo
   , porque no modelo do Django são obrigatório ter campos ``primary_key=True``.

7. Detecção de chave estrangeira "Foreign-key" só funciona com o PostgreSQL e com certos tipos de tabelas do MySQL.
   Em outros casos, a chave estrangeira será gerada como ``IntegerField``s, assumindo a coluna da chave estrangeira como uma coluna ``INT``.

Integrating with an Authentication System
=========================================

It's possible to integrate Django with an existing authentication system --
another source of usernames and passwords or authentication methods.

For example, your company may already have an LDAP setup that stores a username
and password for every employee. It would be a hassle for both the network
administrator and the users themselves if users had separate accounts in LDAP
and the Django-based applications.

To handle situations like this, the Django authentication system lets you
plug in other authentication sources. You can override Django's default
database-based scheme, or you can use the default system in tandem with other
systems.

Specifying Authentication Backends
----------------------------------

Behind the scenes, Django maintains a list of "authentication backends" that it
checks for authentication. When somebody calls
``django.contrib.auth.authenticate()`` (as described in Chapter 14), Django
tries authenticating across all of its authentication backends. If the first
authentication method fails, Django tries the second one, and so on, until all
backends have been attempted.

The list of authentication backends to use is specified in the
``AUTHENTICATION_BACKENDS`` setting. This should be a tuple of Python path
names that point to Python classes that know how to authenticate. These classes
can be anywhere on your Python path.

By default, ``AUTHENTICATION_BACKENDS`` is set to the following::

    ('django.contrib.auth.backends.ModelBackend',)

That's the basic authentication scheme that checks the Django users database.

The order of ``AUTHENTICATION_BACKENDS`` matters, so if the same username and
password are valid in multiple backends, Django will stop processing at the
first positive match.

Writing an Authentication Backend
---------------------------------

An authentication backend is a class that implements two methods:
``get_user(id)`` and ``authenticate(**credentials)``.

The ``get_user`` method takes an ``id`` -- which could be a username, database
ID, or whatever -- and returns a ``User`` object.

The  ``authenticate`` method takes credentials as keyword arguments. Most of
the time it looks like this::

    class MyBackend(object):
        def authenticate(self, username=None, password=None):
            # Check the username/password and return a User.

But it could also authenticate a token, like so::

    class MyBackend(object):
        def authenticate(self, token=None):
            # Check the token and return a User.

Either way, ``authenticate`` should check the credentials it gets, and it
should return a ``User`` object that matches those credentials, if the
credentials are valid. If they're not valid, it should return ``None``.

The Django admin system is tightly coupled to Django's own database-backed
``User`` object described in Chapter 14. The best way to deal with this is to
create a Django ``User`` object for each user that exists for your backend
(e.g., in your LDAP directory, your external SQL database, etc.). Either you can
write a script to do this in advance or your ``authenticate`` method can do it
the first time a user logs in.

Here's an example backend that authenticates against a username and password
variable defined in your ``settings.py`` file and creates a Django ``User``
object the first time a user authenticates::

    from django.conf import settings
    from django.contrib.auth.models import User, check_password

    class SettingsBackend(object):
        """
        Authenticate against the settings ADMIN_LOGIN and ADMIN_PASSWORD.

        Use the login name, and a hash of the password. For example:

        ADMIN_LOGIN = 'admin'
        ADMIN_PASSWORD = 'sha1$4e987$afbcf42e21bd417fb71db8c66b321e9fc33051de'
        """
        def authenticate(self, username=None, password=None):
            login_valid = (settings.ADMIN_LOGIN == username)
            pwd_valid = check_password(password, settings.ADMIN_PASSWORD)
            if login_valid and pwd_valid:
                try:
                    user = User.objects.get(username=username)
                except User.DoesNotExist:
                    # Create a new user. Note that we can set password
                    # to anything, because it won't be checked; the password
                    # from settings.py will.
                    user = User(username=username, password='get from settings.py')
                    user.is_staff = True
                    user.is_superuser = True
                    user.save()
                return user
            return None

        def get_user(self, user_id):
            try:
                return User.objects.get(pk=user_id)
            except User.DoesNotExist:
                return None

For more on authentication backends, see the official Django documentation.

Integrating with Legacy Web Applications
========================================

It's possible to run a Django application on the same Web server as an
application powered by another technology. The most straightforward way of
doing this is to use Apache's configuration file, ``httpd.conf``, to delegate
different URL patterns to different technologies. (Note that Chapter 12 covers
Django deployment on Apache/mod_python, so it might be worth reading that
chapter first before attempting this integration.)

The key is that Django will be activated for a particular URL pattern only if
your ``httpd.conf`` file says so. The default deployment explained in Chapter
12 assumes you want Django to power every page on a particular domain::

    <Location "/">
        SetHandler python-program
        PythonHandler django.core.handlers.modpython
        SetEnv DJANGO_SETTINGS_MODULE mysite.settings
        PythonDebug On
    </Location>

Here, the ``<Location "/">`` line means "handle every URL, starting at the
root," with Django.

It's perfectly fine to limit this ``<Location>`` directive to a certain
directory tree. For example, say you have a legacy PHP application that powers
most pages on a domain and you want to install a Django admin site at
``/admin/`` without disrupting the PHP code. To do this, just set the
``<Location>`` directive to ``/admin/``::

    <Location "/admin/">
        SetHandler python-program
        PythonHandler django.core.handlers.modpython
        SetEnv DJANGO_SETTINGS_MODULE mysite.settings
        PythonDebug On
    </Location>

With this in place, only the URLs that start with ``/admin/`` will activate
Django. Any other page will use whatever infrastructure already existed.

Note that attaching Django to a qualified URL (such as ``/admin/`` in this
section's example) does not affect the Django URL parsing. Django works with the
absolute URL (e.g., ``/admin/people/person/add/``), not a "stripped" version of
the URL (e.g., ``/people/person/add/``). This means that your root URLconf
should include the leading ``/admin/``.

What's Next?
============

If you're a native English speaker, you might not have noticed one of the
coolest features of Django's admin site: it's available in more than 50
different languages! This is made possible by Django's internationalization
framework (and the hard work of Django's volunteer translators). The
`next chapter`_ explains how to use this framework to provide localized Django
sites.

.. _next chapter: ../chapter19/
