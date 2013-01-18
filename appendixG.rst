======================================
Apêndice G: Objetos Request e Response
======================================

O Django usa os objetos request e response para passar o estado do sistema. 

Quando uma página é requisitada, o Django cria um objeto ``HttpRequest``, que
contêm um metadado sobre a requisição. Então o Django executa a view apropriada, 
passado o ``HttpRequest`` como primeiro argumento para a função view. Cada
view é responsável por retornar um objeto ``HttpResponse``.

Nós usaremos esses objetos com maior freqüência durante o livro; esse apêndice explica 
todos as APIs dos objetos ``HttpRequest`` e ``HttpResponse``.

HttpRequest
===========

``HttpRequest`` representa uma simples requisição HTTP de algum agente de usuário.

Muitas informações importantes sobre o request está disponível como atributo na
instância ``HttpRequest`` (olhe a Tabela G-1). Todos os atributos exceto o 
``session`` devem ser considerados somente leitura.

.. table:: Tabela G-1. Atributos dos Objetos HttpRequest

    ==================  =======================================================
    Atributo            Descrição
    ==================  =======================================================
    ``path``            Uma string representando o caminho completo da página 
                        requisitada, sem incluir o domínio -- por exemplo,
                        ``"/musica/bandas/the_beatles/"``.

    ``method``          Uma string representando o método HTTP usado pelo request.
                        Precisa ser garantido que esteja em letras maiúsculas. Por
                        exemplo::

                            if request.method == 'GET':
                                faz_alguma_coisa()
                            elif request.method == 'POST':
                                faz_outra_coisa()

    ``encoding``        Uma string representando a codificação atual usada para
                        decodificar uma submissão de dados do formulário. (ou ``None``, 
                        significando que o setting ``DEFAULT_CHARSET`` está
                        sendo usado).

                        Você pode escrever esse atributo para mudar a codificação
                        usada durante o acesso aos dados do formulário. Qualquer acesso   
                        subseqüente ao atributo (assim como a leitura do ``GET`` ou
                        ``POST``) serão usados no novo valor do ``encoding``. Útil
                        se você souber se os dados do formulário não estão na codificação
                        ``DEFAULT_CHARSET``.

    ``GET``             Um objeto do tipo dicionário que contêm todos os parâmetros
                        HTTP GET dados. Veja o ``QueryDict`` na documentação posterior.

    ``POST``            Um objeto do tipo dicionário que contêm todos os parâmetros 
                        HTTP POST. Veja o ``QueryDict`` na documentação posterior.

                        É possível que uma requisição possa vir pelo POST com
                        um dicionário vazio ``POST`` -- se, digamos, o formulário foi
                        foi requisitado pelo método POST HTTP mas não inclue os 
                        dados do formulário. Portanto, não é possível usar o ``if
                        request.POST`` para checar o método POST;
                        no lugar, use ``if request.method == "POST"`` (veja
                        o item ``method`` dessa tabela).

                        Nota: ``POST`` *não* inclui informações de envio de 
                        arquivos. Veja em ``FILES``.

    ``REQUEST``         Por conveniência, um objeto do tipo dicionário busca
                        primeiro pelo ``POST``, e então pelo ``GET``. Inspirado
                        no ``$_REQUEST`` do PHP.

                        Por exemplo, se ``GET = {"nome": "john"}`` e ``POST
                        = {"idade": '34'}``, ``REQUEST["nome"]`` seria
                        ``"john"``, e ``REQUEST["idade"]`` seria ``"34"``.

                        É fortemente recomendado que você use ``GET`` e
                        ``POST`` em vez de ``REQUEST``, porque o padrão
                        é mais explícito.

    ``COOKIES``         Um dicionário padrão Python contendo todos os cookies.
                        Chaves e valores são strings. Veja o Capítulo 14 para
                        maiores informações sobre o uso dos cookies.

    ``FILES``           Um objeto do tipo dicionário que mapeia nomes de 
                        arquivos para os objetos ``UploadedFile``. Veja a
                        documentação do Django para maiores informações.

    ``META``            Um dicionário padrão Python contendo todos os cabeçalhos
                        HTTP disponíveis. Cabeçalhos disponíveis dependem do cliente
                        e dos servidor, mas aqui estão alguns exemplos:

                        * ``CONTENT_LENGTH``
                        * ``CONTENT_TYPE``
                        * ``QUERY_STRING``: A query string não analisada
                        * ``REMOTE_ADDR``: O endereço IP do cliente
                        * ``REMOTE_HOST``: O hostname do cliente
                        * ``SERVER_NAME``: O hostname do servidor
                        * ``SERVER_PORT``: A porta do servidor

                        Os cabeçalhos HTTP estão disponíveis no ``META`` como
                        chaves prefixadas com o ``HTTP_``, convertidos em
                        letras maiúsculas substituindo sublinhados por hífens.
                        Por exemplo:

                        * ``HTTP_ACCEPT_ENCODING``
                        * ``HTTP_ACCEPT_LANGUAGE``
                        * ``HTTP_HOST``: O cabeçalho HTTP ``Host`` enviado pelo cliente
                        * ``HTTP_REFERER``: A página referida, se existente
                        * ``HTTP_USER_AGENT``: A string do user-agente do cliente
                        * ``HTTP_X_BENDER``: O valor do cabeçalho ``X-Bender``, se definido

    ``user``            O objeto ``django.contrib.auth.models.User`` representando
                        o atual usuário logado. Se o usuário não estiver atualmente
                        logado, o ``user`` vai ser definido como instância do
                        ``django.contrib.auth.models.AnonymousUser``. Você pode
                        chama-los à parte com o ``is_authenticated()``, assim::

                            if request.user.is_authenticated():
                                # Faça alguma coisa com os usuários logados.
                            else:
                                # Faça alguma coisa com os usuários anônimos.

                        ``user`` está disponível apenas se sua instalação do
                        Django tiver o ``AuthenticationMiddleware`` ativado.

                        Para maiores detalhes sobre autenticação de usuários,
                        veja o Capítulo 14.

    ``session``         Um objeto do tipo dicionário de leitura e escrita,
                        representa a sessão atual. Está disponível apenas
                        se a sua instalação do Django ter a sessão support ativada.
                        Veja no Capítulo 14.

    ``raw_post_data``   O dado bruto do HTTP POST. É útil para processamento avançado.

    ==================  =======================================================

Objetos Request também possuem alguns métodos úteis, como mostra a Tabela G-2.

.. table:: Tabela G-2. Métodos HttpRequest

    ======================  ===================================================
    Method                  Description
    ======================  ===================================================
    ``__getitem__(key)``    Returns the GET/POST value for the given key,
                            checking POST first, and then GET. Raises
                            ``KeyError`` if the key doesn't exist.

                            This lets you use dictionary-accessing syntax on
                            an ``HttpRequest`` instance.

                            For example, ``request["foo"]`` is the same as
                            checking ``request.POST["foo"]`` and then
                            ``request.GET["foo"]``.

    ``has_key()``           Returns ``True`` or ``False``, designating whether
                            ``request.GET`` or ``request.POST`` has the given
                            key.

    ``get_host()``          Returns the originating host of the request using
                            information from the ``HTTP_X_FORWARDED_HOST`` and
                            ``HTTP_HOST`` headers (in that order). If they
                            don't provide a value, the method uses a
                            combination of ``SERVER_NAME`` and
                            ``SERVER_PORT``.

    ``get_full_path()``     Returns the ``path``, plus an appended query
                            string, if applicable. For example,
                            ``"/music/bands/the_beatles/?print=true"``

    ``is_secure()``         Returns ``True`` if the request is secure; that
                            is, if it was made with HTTPS.
    ======================  ===================================================

QueryDict Objects
-----------------

In an ``HttpRequest`` object, the ``GET`` and ``POST`` attributes are
instances of ``django.http.QueryDict``. ``QueryDict`` is a dictionary-like
class customized to deal with multiple values for the same key. This is
necessary because some HTML form elements, notably ``<select
multiple="multiple">``, pass multiple values for the same key.

``QueryDict`` instances are immutable, unless you create a ``copy()`` of them.
That means you can't change attributes of ``request.POST`` and ``request.GET``
directly.

``QueryDict`` implements the all standard dictionary methods, because it's a
subclass of dictionary. Exceptions are outlined in Table G-3.

.. table:: Table G-3. How QueryDicts Differ from Standard Dictionaries.

    ==================  =======================================================
    Method              Differences from Standard dict Implementation
    ==================  =======================================================
    ``__getitem__``     Works just like a dictionary. However, if the key
                        has more than one value, ``__getitem__()`` returns the
                        last value.

    ``__setitem__``     Sets the given key to ``[value]`` (a Python list whose
                        single element is ``value``). Note that this, as other
                        dictionary functions that have side effects, can
                        be called only on a mutable ``QueryDict`` (one that was
                        created via ``copy()``).

    ``get()``           If the key has more than one value, ``get()`` returns
                        the last value just like ``__getitem__``.

    ``update()``        Takes either a ``QueryDict`` or standard dictionary.
                        Unlike the standard dictionary's ``update`` method,
                        this method *appends* to the current dictionary items
                        rather than replacing them::

                            >>> q = QueryDict('a=1')
                            >>> q = q.copy() # to make it mutable
                            >>> q.update({'a': '2'})
                            >>> q.getlist('a')
                            ['1', '2']
                            >>> q['a'] # returns the last
                            ['2']

    ``items()``         Just like the standard dictionary ``items()`` method,
                        except this uses the same last-value logic as
                        ``__getitem()__``::

                             >>> q = QueryDict('a=1&a=2&a=3')
                             >>> q.items()
                             [('a', '3')]

    ``values()``        Just like the standard dictionary ``values()`` method,
                        except this uses the same last-value logic as
                        ``__getitem()__``.
    ==================  =======================================================

In addition, ``QueryDict`` has the methods shown in Table G-4.

.. table:: G-4. Extra (Nondictionary) QueryDict Methods

    ==========================  ===============================================
    Method                      Description
    ==========================  ===============================================
    ``copy()``                  Returns a copy of the object, using
                                ``copy.deepcopy()`` from the Python standard
                                library. The copy will be mutable -- that is,
                                you can change its values.

    ``getlist(key)``            Returns the data with the requested key, as a
                                Python list. Returns an empty list if the key
                                doesn't exist. It's guaranteed to return a
                                list of some sort.

    ``setlist(key, list_)``     Sets the given key to ``list_`` (unlike
                                ``__setitem__()``).

    ``appendlist(key, item)``   Appends an item to the internal list associated
                                with ``key``.

    ``setlistdefault(key, a)``  Just like ``setdefault``, except it takes a
                                list of values instead of a single value.

    ``lists()``                 Like ``items()``, except it includes all
                                values, as a list, for each member of the
                                dictionary. For example::

                                    >>> q = QueryDict('a=1&a=2&a=3')
                                    >>> q.lists()
                                    [('a', ['1', '2', '3'])]


    ``urlencode()``             Returns a string of the data in query-string
                                format (e.g., ``"a=2&b=3&b=5"``).
    ==========================  ===============================================

A Complete Example
------------------

For example, given this HTML form::

    <form action="/foo/bar/" method="post">
    <input type="text" name="your_name" />
    <select multiple="multiple" name="bands">
        <option value="beatles">The Beatles</option>
        <option value="who">The Who</option>
        <option value="zombies">The Zombies</option>
    </select>
    <input type="submit" />
    </form>

if the user enters ``"John Smith"`` in the ``your_name`` field and selects
both "The Beatles" and "The Zombies" in the multiple select box, here's what
Django's request object would have::

    >>> request.GET
    {}
    >>> request.POST
    {'your_name': ['John Smith'], 'bands': ['beatles', 'zombies']}
    >>> request.POST['your_name']
    'John Smith'
    >>> request.POST['bands']
    'zombies'
    >>> request.POST.getlist('bands')
    ['beatles', 'zombies']
    >>> request.POST.get('your_name', 'Adrian')
    'John Smith'
    >>> request.POST.get('nonexistent_field', 'Nowhere Man')
    'Nowhere Man'

.. admonition:: Implementation Note:

    The ``GET``, ``POST``, ``COOKIES``, ``FILES``, ``META``, ``REQUEST``,
    ``raw_post_data``, and ``user`` attributes are all lazily loaded. That means
    Django doesn't spend resources calculating the values of those attributes until
    your code requests them.

HttpResponse
============

In contrast to ``HttpRequest`` objects, which are created automatically by
Django, ``HttpResponse`` objects are your responsibility. Each view you write
is responsible for instantiating, populating, and returning an
``HttpResponse``.

The ``HttpResponse`` class lives at ``django.http.HttpResponse``.

Construction HttpResponses
--------------------------

Typically, you'll construct an ``HttpResponse`` to pass the contents of the
page, as a string, to the ``HttpResponse`` constructor::

    >>> response = HttpResponse("Here's the text of the Web page.")
    >>> response = HttpResponse("Text only, please.", mimetype="text/plain")

But if you want to add content incrementally, you can use ``response`` as a
filelike object::

    >>> response = HttpResponse()
    >>> response.write("<p>Here's the text of the Web page.</p>")
    >>> response.write("<p>Here's another paragraph.</p>")

You can pass ``HttpResponse`` an iterator rather than passing it
hard-coded strings. If you use this technique, follow these guidelines:

* The iterator should return strings.

* If an ``HttpResponse`` has been initialized with an iterator as its
  content, you can't use the ``HttpResponse`` instance as a filelike
  object. Doing so will raise ``Exception``.

Finally, note that ``HttpResponse`` implements a ``write()`` method, which
makes is suitable for use anywhere that Python expects a filelike object. See
Chapter 8 for some examples of using this technique.

Setting Headers
---------------

You can add and delete headers using dictionary syntax::

    >>> response = HttpResponse()
    >>> response['X-DJANGO'] = "It's the best."
    >>> del response['X-PHP']
    >>> response['X-DJANGO']
    "It's the best."

You can also use ``has_header(header)`` to check for the existence of a header.

Avoid setting ``Cookie`` headers by hand; instead, see Chapter 14 for
instructions on how cookies work in Django.

HttpResponse Subclasses
-----------------------

Django includes a number of ``HttpResponse`` subclasses that handle different
types of HTTP responses (see Table G-5). Like ``HttpResponse``, these subclasses live in
``django.http``.

.. table:: Table G-5. HttpResponse Subclasses

    ==================================  =======================================
    Class                               Description
    ==================================  =======================================
    ``HttpResponseRedirect``            The constructor takes a single argument:
                                        the path to redirect to. This can
                                        be a fully qualified URL (e.g.,
                                        ``'http://search.yahoo.com/'``) or
                                        an absolute URL with no domain (e.g.,
                                        ``'/search/'``). Note that this
                                        returns an HTTP status code 302.

    ``HttpResponsePermanentRedirect``   Like ``HttpResponseRedirect``, but it
                                        returns a permanent redirect (HTTP
                                        status code 301) instead of a "found"
                                        redirect (status code 302).

    ``HttpResponseNotModified``         The constructor doesn't take any
                                        arguments. Use this to designate that
                                        a page hasn't been modified since the
                                        user's last request.

    ``HttpResponseBadRequest``          Acts just like ``HttpResponse`` but
                                        uses a 400 status code.

    ``HttpResponseNotFound``            Acts just like ``HttpResponse`` but
                                        uses a 404 status code.

    ``HttpResponseForbidden``           Acts just like ``HttpResponse`` but
                                        uses a 403 status code.

    ``HttpResponseNotAllowed``          Like ``HttpResponse``, but uses a 405
                                        status code. It takes a single, required
                                        argument: a list of permitted methods
                                        (e.g., ``['GET', 'POST']``).

    ``HttpResponseGone``                Acts just like ``HttpResponse`` but
                                        uses a 410 status code.

    ``HttpResponseServerError``         Acts just like ``HttpResponse`` but
                                        uses a 500 status code.
    ==================================  =======================================

You can, of course, define your own ``HttpResponse`` subclass to support
different types of responses not supported out of the box.

Returning Errors
----------------

Returning HTTP error codes in Django is easy. We've already mentioned the
``HttpResponseNotFound``, ``HttpResponseForbidden``,
``HttpResponseServerError``, and other subclasses. Just return an instance of one
of those subclasses instead of a normal ``HttpResponse`` in order to signify
an error, for example::

    def my_view(request):
        # ...
        if foo:
            return HttpResponseNotFound('<h1>Page not found</h1>')
        else:
            return HttpResponse('<h1>Page was found</h1>')

Because a 404 error is by far the most common HTTP error, there's an easier
way to handle it.

When you return an error such as ``HttpResponseNotFound``, you're responsible
for defining the HTML of the resulting error page::

    return HttpResponseNotFound('<h1>Page not found</h1>')

For convenience, and because it's a good idea to have a consistent 404 error page
across your site, Django provides an ``Http404`` exception. If you raise
``Http404`` at any point in a view function, Django will catch it and return the
standard error page for your application, along with an HTTP error code 404.

Here's an example::

    from django.http import Http404

    def detail(request, poll_id):
        try:
            p = Poll.objects.get(pk=poll_id)
        except Poll.DoesNotExist:
            raise Http404
        return render(request, 'polls/detail.html', {'poll': p})

In order to use the ``Http404`` exception to its fullest, you should create a
template that is displayed when a 404 error is raised. This template should be
called ``404.html``, and it should be located in the top level of your template tree.

Customizing the 404 (Not Found) View
------------------------------------

When you raise an ``Http404`` exception, Django loads a special view devoted
to handling 404 errors. By default, it's the view
``django.views.defaults.page_not_found``, which loads and renders the template
``404.html``.

This means you need to define a ``404.html`` template in your root template
directory. This template will be used for all 404 errors.

This ``page_not_found`` view should suffice for 99% of Web applications, but
if you want to override the 404 view, you can specify ``handler404`` in your
URLconf, like so::

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        ...
    )

    handler404 = 'mysite.views.my_custom_404_view'

Behind the scenes, Django determines the 404 view by looking for
``handler404``. By default, URLconfs contain the following line::

    from django.conf.urls.defaults import *

That takes care of setting ``handler404`` in the current module. As you can
see in ``django/conf/urls/defaults.py``, ``handler404`` is set to
``'django.views.defaults.page_not_found'`` by default.

There are three things to note about 404 views:

* The 404 view is also called if Django doesn't find a match after checking
  every regular expression in the URLconf.

* If you don't define your own 404 view -- and simply use the default,
  which is recommended -- you still have one obligation: to create a
  ``404.html`` template in the root of your template directory. The default
  404 view will use that template for all 404 errors.

* If ``DEBUG`` is set to ``True`` (in your settings module), then your 404
  view will never be used, and the traceback will be displayed instead.

Customizing the 500 (Server Error) View
---------------------------------------

Similarly, Django executes special-case behavior in the case of runtime errors
in view code. If a view results in an exception, Django will, by default, call
the view ``django.views.defaults.server_error``, which loads and renders the
template ``500.html``.

This means you need to define a ``500.html`` template in your root template
directory. This template will be used for all server errors.

This ``server_error`` view should suffice for 99% of Web applications, but if
you want to override the view, you can specify ``handler500`` in your
URLconf, like so::

    from django.conf.urls.defaults import *

    urlpatterns = patterns('',
        ...
    )

    handler500 = 'mysite.views.my_custom_error_view'
