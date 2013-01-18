============================
Capítulo 3: Views e URLconfs
============================

No capítulo anterior foi explicado como montar um projeto Django e executar o
servidor de desenvolvimento do Django. Neste capítulo serão vistos os conceitos
básicos de como criar páginas dinâmicas com o Django.

Primeira página em Django: Hello World
======================================

Como primeiro objetivo iremos criar uma página que tem como saída a
famosa mensagem de exemplo: "Hello world."

Se estivesse publicando uma simples página Web sem um framework Web, bastaria digitar
"Hello World" em um arquivo texto, chamá-lo de ``hello.html`` e fazer upload do mesmo
para um diretório em um servidor Web em algum lugar. Note que nesse processo foram
especificadas duas peças chave de informação sobre a página: o conteúdo (a string 
``"Hello world"``) e sua URL (``http:://www.exemplo.com/hello.html``, ou talvez 
``http:://www.exemplo.com/files/hello.html`` caso tenha posto em um subdiretório)

Com Django, as mesmas coisas são especificadas, porém de modos diferentes. Os
conteúdos da página são produzidos por uma *função view (visão)*, e a URL é
especificada em um *URLconf*. Primeiro será escrita a função view "Hello World".

Primeira view
-------------

Dentro do diretório ``meusite`` que o ``django-admin.py startproject`` criou no
último capítulo, crie um arquivo vazio chamado ``views.py``. Esse módulo Python conterá
as views vistas neste capítulo. Note que não há nada especial sobre o nome ``views.py``
-- o Django não se preocupa como o arquivo é chamado, como será visto mais para
frente -- mas é uma boa ideia chamá-lo de ``views.py`` por ser uma convenção, para o
benefício de outros desenvolvedores que irão ler seu código.

A view "Hello world" é simples. Aqui está toda a função, incluindo os imports, que
deverá ser digitada no arquivo ``views.py``::

    from django.http import HttpResponse

    def hello(request):
        return HttpResponse("Hello world")

Vamos analisar esse código linha-por-linha:

* Primeiro, é importada a classe ``HttpResponse``, que está localizada no módulo
  ``django.http``. É necessário importá-la pois é usada no código.

* Depois é definida uma função chamada ``hello`` -- a função view.

  Cada função view tem pelo menos um parametro chamado ``request``, por
  convenção. Este é um objeto que contém informação sobre a requisição Web 
  atual que ativou a view, e é uma instância da classe 
  ``django.http.HttpRequest``. Nesse exemplo não é feito nada com ``request``,
  mas de qualquer modo deve ser o primeiro parametro da view.

  Note que o nome da função view não importa, não é necessário nomeá-la de
  uma maneira específica para que o django a reconheça. É chamanda de
  ``hello``, pois esse nome indica de forma clara a essência da view, mas nada
  impede de que fosse chamada de ``hello_wonderful_beautiful_world``, ou algo
  do tipo. Na próxima seção, "Primeiro URLconf", será explicado como o Django
  encontra essa função.

* A função só contém uma linha: simplesmente retorna um objeto ``HttpResponse``
  que foi instanciado com o texto ``Hello World``.

A lição principal aqui é a seguinte: a view é somente uma função Python que
recebe um ``HttpRequest`` como primeiro argumento e retorna uma instância de
``HttpRequest``. Para que uma função Python seja uma view Django é necessário
que faça essas duas coisas. (Existem exceções, mas chegaremos nelas depois.)

Seu Primeiro URLconf
--------------------

Se, neste ponto, for executado ``python manage.py runserver`` de novo, ainda será
exibida a mensagem "Welcome to Django", sem nenhum traço da view "Hello World".
Isso acontece, pois o projeto ``meusite`` ainda não sabe da view ``hello``; é
preciso mostrar explicitamente ao Django que essa view está sendo ativada em uma
URL particular. (Continuando a analogia anterior de publicar arquivos HTML estáticos,
neste ponto é criado o arquivo HTML, mas ainda não foi feito upload dele no servidor
web) Para ligar uma função view para uma URL com o Django, é utilizado um URLconf.

Um *URLconf* é como um índice remissivo para sua página Django. Basicamente é um
mapeamento entre URLs e funções view que devem ser chamadas para essas URLs. É como
dizer ao Django: "Para esta URL, chame este código, e para aquela URL, chame aquele
código". Por exemplo: "Quando alguém visita a URL ``/foo/``, a função view ``foo_view()``
é chamada, que está no arquivo Python ``views.py``."

Quando ``django-admin startproject`` foi executado no capítulo anterior, um URLconf 
foi criado automaticamente: o arquivo ``urls.py``. Por padrão ele parece com
algo assim::

    from django.conf.urls import patterns, include, url

    # Uncomment the next two lines to enable the admin:
    # from django.contrib import admin
    # admin.autodiscover()

    urlpatterns = patterns('',
        # Examples:
        # url(r'^$', 'mysite.views.home', name='home'),
        # url(r'^mysite/', include('mysite.foo.urls')),

        # Uncomment the admin/doc line below to enable admin documentation:
        # url(r'^admin/doc/', include('django.contrib.admindocs.urls')),

        # Uncomment the next line to enable the admin:
        # url(r'^admin/', include(admin.site.urls)),
    )

Este URLconf padrão inclui algumas funcionalidades comumente usadas no Django dentro
de comentários, para que ativá-las seja tão fácil quanto descomentar as linhas certas.
Se os códigos comentados forem ignorados, aqui está a essência de um URLconf::

    from django.conf.urls.defaults import patterns, include, url

    urlpatterns = patterns('',
    )

Vamos analisar linha-por-linha deste código:

* A primeira linha importa três funções do módulo ``django.conf.urls.defaults``,
  que são a infraestrutura do URLconf do Django: ``patterns``, ``include`` e
  ``urls``.

* A segunda linha chama a função ``patterns`` e salva o resultado em uma
  variável chamada ``urlpatterns``. A função ``patterns`` recebe um único
  argumento: uma string vazia. (A string pode ser usada para prover um
  prefixo comum para funções view, que será visto no :doc:`chapter08`.)

O que deve-se notar aqui é a váriavel ``urlpatterns``, que o Django espera encontrar
em seu módulo URLconf. Essa variável define o mapeamento entre URLs e o código que
manipula essas URLs. Por padrão, como pode-se ver, o URLconf está vazio -- sua
aplicação Django está vazia. (Como uma nota, é desse jeito que o Django mostra 
a página "Welcome to Django" vista no último capítulo. Se um URLconf está vazio,
o Django assume que um novo projeto foi iniciado e, portanto, mostra essa mensagem.)

Para adicionar uma URL e view ao URLconf, só é necessário adicionar um mapeamento
entre um URLpattern e a função view. Aqui mostra-se como fazer a ligação na nossa
view ``hello``::

    from django.conf.urls.defaults import patterns, include, url
    from mysite.views import hello

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
    )

(Note que os códigos comentados foram removidos para poupar espaço. É possível
deixar as linhas se quiser.)

Foram feitas duas mudanças aqui:

* Primeiro,  a view ``hello`` foi importada de seu módulo ``mysite/views.py``,
  que é acessado como ``mysite.views`` na sintaxe de importação do Python.
  (Isso assume que ``mysite/views.py`` está no seu diretório Python; veja a 
  barra lateral para detalhes.)

* Depois é adicionada a linha ``url(r'^ĥello/$'),`` à variável ``urlpatterns``. 
  Essa linha é denominada *URLpattern*. A função ``url()`` avisa ao Django como 
  lidar com a url que está sendo configurado. O primeiro argumento é uma string
  de correspondência de padrão (pattern-matching, uma expressão regular; será
  explicado melhor mais a frente) e o segundo argumento é a função view para 
  aquele padrão. ``url()`` pode levar outros argumentos, que serão vistos mais
  profundamente no :doc:`chapter08`.

.. nota::

  Um detalhe importante que foi introduzido é o caracter ``r`` no começo da string
  de expressão regular. Ele avisa o Python que a string é uma "raw string" -- seu
  conteúdo não deve interpretar contrabarras -- como na string ``'\n'``, que é uma
  string com um único caracter contendo uma nova linha. Quando ``r`` é adicionado,
  o Python não faz a substituição, então ``r'\n'`` é uma string composta de dois
  caracteres contendo uma contrabarra e um "n". Há uma colisão natural entre o uso
  de contrabarras pelo Python e as contrabarras utilizadas em expressões regulares,
  então é fortemente recomendado utilizar raw strings sempre que for definir
  expressões regulares em Python. Todos os URLpatterns neste livro serão raw strings.

Em poucas palavras, foi instruído ao Django que qualquer solicitação feita a URL
``/hello/`` deve ser manuseada pela função view ``hello``.

.. admonition:: Diretório Python

    O  *diretório Python* é a lista de diretório do sistema que o Python
    procura quando a instrução ``import`` é utilizada.

    Por exemplo, supondo que o diretório Python está em ``['', '
    '/usr/lib/python2.7/site-packages', '/home/username/djcode']``. Caso a instrução
    ``from foo import bar`` seja executada, o Python procurará um módulo chamado
    ``foo.py`` no diretório atual. (A primeira ocorrência no diretório Python, uma
    string vazia, significa "o diretório atual.") Se o arquivo não existir, o Python
    procurará pelo arquivo ``/usr/lib/python2.7/site-packages/foo.py``. Se esse
    arquivo também não existir, o Python tentará procurar pelo arquivo
    ``/home/username/djcode/foo.py``. Por fim, se *aquele* arquivo não existir, será
    levantado um ``ImportError``.

    Para descobrir seu diretório Python, inicie o interpretador interativo Python
    e digite isso::

        >>> import sys
        >>> print sys.path

    Geralmente não é necessário se preocupar com o diretório Python -- o Python e
    Django tomam conta disso automaticamente nos bastidores. (Configurar o diretório
    Python é uma das tarefas que o script ``manage.py`` faz.)

Vale a pena discutir a sintaxe desse URLpattern, pois pode não parecer óbvio a 
primeira vista. Apesar de querer encontrar a URL ``/hello/``, o padrão é um pouco
diferente disso. Aqui está o por quê:

* O Django remove a barra do começo de cada URL antes de checar os 
  URLpatterns. Isto significa que o URLpattern visto no capítulo não
  possui uma barra no começo em ``/hello/``. (A princípio isso pode
  parecer nada intuitivo, mas isso simplifica as coisas, como a inclusão
  de URLconfs dentro de outros URLconfs, que será visto no Capítulo 8.)

* O padrão inclui um acento circunflexo (``^``) e um sifrão (``$``). Esses
  são caracteres que possuem signficado especial em expressões regulares:
  o acento circunflexo signfica "o padrão deve casar com o começo da string"
  e o sifrão significa "o padrão deve casar com o fim da string."

  Esse conceito é melhor explicado através de um exemplo. Se ao invés de
  usar o padrão ``'^hello/'`` (sem o sifrão no final), então *qualquer* URL
  começada por ``/hello/`` seria casa, como ``/hello/foo`` e ``/hello/bar``,
  e não somente ``/hello/``. Semelhantemente, se o acento circunflexo fosse
  retirado (ou seja, ``'hello/$'``), o Django casaria *qualquer* URL
  terminada em ``hello/``, como ``/foo/bar/hello/``. Se fosse usado ``hello/``,
  sem acento circunflexo *ou* sifrão, então qualquer URL que contesse
  ``hello/`` seria casada, como ``/foo/hello/bar``. Desse modo usamos tanto
  o acento circunflexo quanto o sifão para garantir que somente a URL
  ``/hello/`` seja casada -- nada mais, nada menos.

  A maioria dos URLpatterns começarão com acentos circunflexos e terminarão
  com sifrão, mas é interessante ter a flexibilidade de poder casar de formas
  mais sofisticadas.

  Pode estar se perguntando o que acontece se a URL ``/hello`` (isto é, *sem*
  a barra no final) for requisitada. Essa URL *não* casaria, pois o URLpattern
  exige uma barra no fim. Entretanto, por padrão, qualquer requisição feita a
  uma URL que *não* casa com a URLpattern e *não* termina com uma barra é
  redirecionada para a mesma URL com uma barra no final. (Isso é controlado
  pela configuração ``APPEND_SLASH`` do Django, que é explicada no
  Apêndice D.)

  Se optar por utilizar URLs terminadas por barra, tudo o que precisa fazer
  é adicionar barras em cada URLpattern e atribuir ``True`` ao
  ``APPEND_SLASH``. Caso opte por *não* deixar barras no fim das URLs, ou 
  quiser escolher dependendo da URL, deixe o ``APEEND_SLASH`` como ``False``
  e colocar barras no fim de cada URLpattern que desejar.

Outro ponto importante sobre esse URLconf é que passamos a função view
``hello`` como um objeto sem chamar a função. Essa é uma das características
chaves do Python (e outras linguagens dinâmicas): funções são objetos de
primeira classe, o que quer dizer que é possível utilizá-las como quaisquer
outras variáveis. Maneiro, não?

Para testar as mudanças feitas no URLconf, inicie o servidor de desenvolvimento
do Django, como visto Capítulo 2, executando o comando ``python manage.py runserver``.
(Se já estiver rodando, não tem problema. O servidor detecta mudanças feitas no
código automaticamente e as recarrega, para não ser preciso reiniciar o servidor quando
houver mudança.) O servidor está rodando no endereço ``http://127.0.0.1:8000/``, então
abra o navegador e vá para ``http://127.0.0.1:8000/hello/``. Deve aparecer o texto
"Hello world" -- a saída da sua view Django.

Eba! Você fez sua primeira página Django.

.. admonition:: Expressões regulares

    *Expressões regulares* (ou *regexes*) são uma forma compacta de especificar
    padrões em um texto. Enquanto URLconfs do Django possibilitam o uso de
    qualquer regex para casas URLs, só serão utilizados alguns símbolos de
    regex na prática. Aqui estão alguns símbolos mais utilizados:

    ============  ==========================================================
    Símbolo       Casa com
    ============  ==========================================================
    ``.`` (dot)   Qualquer (um) caracter

    ``\d``        Qualquer (um) digito

    ``[A-Z]``     Qualquer caracter entre ``A`` e ``Z`` (caixa alta)

    ``[a-z]``     Qualquer caracter entre ``a`` e ``z`` (caixa baixa)

    ``[A-Za-z]``  Qualquer caracter entre ``a`` e ``z`` (caixa baixa e alta)

    ``+``         Um ou mais da expressão anterior (por ex., ``\d+`` casa um
                  ou mais digitos.

    ``[^/]+``     Um ou mais caracteres até (e não incluindo) uma barra

    ``?``         Zero ou um da expressão anterior (por ex., ``\d?`` casa
                  zero ou mais digitos

    ``*``         Zero or mais da expressão anterior (por ex., ``\d*`` casa
                  zero, um ou mais que um dígito

    ``{1,3}``     Entre um e três (incluso) da expressão anterior (por ex/,
                  ``\d{1,3}`` casa um, dois ou três digitos)
    ============  ==========================================================

    Para mais sobre expressões regulares, veja http://turing.com.br/material/regex/python_re.html

Uma nota rápida sobre erros 404
-------------------------------

Neste ponto, o URLconf define somente um URLpattern, o que lida com as
requisições da URL ``/hello/``. O que acontece quando é feito uma requisição
para outra URL?

Para descobrir, rode o servidor de desenvolvimento do Django e visite alguma
página como ``http://127.0.0.1:8000/goodbye/`` ou
``http://127.0.0.1:8000/hello/subdirectory/``, ou até ``http://127.0.0.1:8000/``
(a "raiz" do site). Deverá aparecer a mensagem "Page not found" (ver
Figura 3-1). O Django exibe essa mensagem, pois você requisitou uma URL que
não está especificada em seu URLconf.

.. figure:: graphics/chapter03/404.png
   :alt: Screenshot da página 404 do Django.

   Figura 3-1. Página 404 di Django

A função dessa página vai além de exibir a mensagem de erro 404. Ela também
mostra qual URLconf exatamente o Django usou para cada padrão nesse URLconf.
Com essa informação é possível saber por quê a URL requisitada lançou um
erro 404.

Naturalmente, essa informação sensível é só para você, o desenvolvedor Web.
Se esse fosse um site em produção disponível na Internet, não seria bom
exibir essa informação para o público. Por esse motivo, essa página "Page
not found" é exibida somente se seu projeto Django está no *modo debug*.
Mais para frente será explicado como desativar esse modo. Por enquanto, saiba
que cada projeto Django está no modo debug quando é criado pela primeira vez.
Se o projeto não esá no modo debug, o Django mostra outra mensagem 404.

Uma nota rápida sobre a raiz do site
------------------------------------

Como explicado na última seção, você verá uma mensagem de erro ao acessar a
raiz do site -- ``http://127.0.0.1:8000``. O Django não adiciona nada magicamente
à raiz do site; essa URL não é tratada de modo especial. O desenvolvedor deve
atribuir um URLpatternt a ela, como todo outro acesso em seu URLconf.

O URLpattern que casa com a raiz do site é um pouco não intuitivo, então será
explicado aqui. Quando for implementar a view da raiz do site, use a URLpattern
``'^$'``, que casa uma string vazia. Por exemplo::

    from mysite.views import hello, my_homepage_view

    urlpatterns = patterns('',
        url(r'^$', my_homepage_view),
        # ...
    )

Como o Django processa uma requisição
=====================================

Antes de continuar para nossa segunda função view, vamos parar e aprender um
pouco sobre como o Django funciona. Especificamente, quando é exibida a mensagem
"Hello world" ao visitar ``http://127.0.0.1:8000/hello/`` no seu navegador, o que
o Django faz por trás?

Tudo começa com o *arquivo de configurações*. Quando é executado ``python
manage.py runser``, o script procura por um arquivo chamado ``settings.py``,
dentro do site ``mysite``. Esse arquivo contem todo o tipo de configuração
para esse projeto Django, como (tudo em caixa alta): ``TEMPLATE_DIRS``,
``DATABASES`` e etc. A configuração mais importante é chamada de
``ROOT_URLCONF``. Essa configuração avisa ao Django qual módulo deve ser usado
como URLconf para esse site.

Lembra quando ``django-admin.py startproject`` criou os arquivos
``settings.py`` e ``urls.py``? O ``settings.py`` gerado automaticamente contém um
configuração ``ROOT_URLCONF`` que aponto para o ``urls.py`` gerado automaticamente.
Abra o arquivo ``settings.py`` e veja por si mesmo; deve parecer com algo assim::

    ROOT_URLCONF = 'mysite.urls'

Isso corresponde ao arquivo ``mysite/urls.py``.

Quando uma requisição é feita para uma URL em particular, por exemplo uma
requisição para ``/hello/``, o Django carrega o URLconf apontado pelo
``ROOT_URLCONF``. Então checa cada URLpattern naquele URLconf, a fim de comparar
a URL requisitada com os padrões, um por um, até encontrar um que case.
Quando um padrão é casado, é chamada a função view associada com esse padrão,
passando-a como um objeto ``HttpRequest`` como primeiro parametro. (O objeto
``HttpRequest`` será explicado melhor mais para a frente.)

Como visto no primeiro exemplo, uma função view deve retornar um
``HttpResponse``. Uma vez que isso é feito o Django lida com o resto, convertendo
o objeto Python para uma requisição web com os cabeçalhos HTTP e corpo (ou seja,
conteúdo da página) apropriados.

Resumindo:

1. Uma requisição chega a ``/hello/``.
2. Django determina o URLconf raiz analisando a configuração ``ROOT_URLCONF``.
3. Django examina todas as URLpatterns no URLconf até encontrar a primeira
   que case com ``/hello/``.
4. Se for casado, é chamado a função view associada.
5. A função view retorna um ``HttpResponse``.
6. Django converte o ``HttpResponse`` para uma resposta HTTP, que resulta em
   uma página Web.

Agora você sabe o básico de como fazer páginas em Django. É realmente simples,
só é necessário escrever funções view e mapeá-las a URLs através de URLconfs.

A segunda view: conteúdo dinânico
=================================

Nossa view "Hello world" serviu para demonstrar o básico de como o Django
funciona, mas não foi um exemplo de uma página *dinâmica*, pois o conteúdo
da página é sempre o mesmo. Cada vez que ``/hello`` é exibida, será mostrada
a mesma coisa; na verdade, poderia ser um arquivo HTML estático.

Para nossa segunda view, mas criar algo mais dinâmico -- uma página que exibe
a data e hora atuais. Esse exemplo é um bom e, simples, próximo passo, pois
não envolve banco de dados ou entrada de usuário, somente a saída do relógio
interno do servidor. Só é um pouco mais interessante que o "Hello world", mas
demonstrará alguns conceitos.

Esta view precisa fazer duas coisas: calcular a hora e data atuais, e retornar
um ``HttpResponse`` contendo esse valor. Se possui experiência com Python,
deve saber que há um módulo Python chamado ``datetime`` para calcular datas.
Aqui mostra como usá-lo::

    >>> import datetime
    >>> now = datetime.datetime.now()
    >>> now
    datetime.datetime(2008, 12, 13, 14, 9, 39, 2731)
    >>> print now
    2008-12-13 14:09:39.002731

Isso é simples o bastante e não tem nada a ver com o Django. É só código Python.
(Queremos enfatizar quais códigos são "só Python" vs código específico do Django.
Conforme for aprendendo Django, queremos ensiná-lo Python, para que possa usar
o mesmo para outros projetos que não envolvem Django).

Para o Django fazer com que a view mostre a data e hora atuais, só é necessário
colocar a instrução ``datetime.datetime.now()`` em uma view e retornar um
``HttpResponse``. Fica assim::

    from django.http import HttpResponse
    import datetime

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>É agora %s.</body></html>" % now
        return HttpResponse(html)

Como na função view ``hello``, esse código deve estar em ``views.py``. Note que
não mostramos a função ``hello`` neste exemplo por brevidade, mas por completude,
aqui está como todo o ``views.py`` fica::

    from django.http import HttpResponse
    import datetime

    def hello(request):
        return HttpResponse("Hello world")

    def current_datetime(request):
        now = datetime.datetime.now()
        html = "<html><body>É agora %s.</body></html>" % now
        return HttpResponse(html)

(De agora em diante não mostraremos código anterior em exemplos, exceto quando
necessário. Você deve saber diferenciar a partir do contexto quais partes do
exemplo são novas das antigas).

Vamos passar pelas mudanças que fizemos ao ``views.py`` para acomodar a view
``current_datetime``.

* Adicionamos um ``import datetime`` na parte superior do módulo, para
  calcular datas.

* O nova função ``current_datetime`` calcula a data e hora atuais, com um
  objeto ``datetime.datetime``, e a armazena como a variável local ``now``.

* A segunda linha de código dentro da view constrói uma resosta HTML usando
  funcionalidade do Python de formata strings. O ``%s`` na string é um
  placeholder, e o símbolo de porcentagem após a string significa
  "Substitua o ``%s`` da string com o valor da variável ``now``". A variável
  ``now`` é, tecnicamente, um objeto ``datetime.datetime``, não uma string,
  mas o caracter de formatação ``%s`` faz a conversão para sua representação
  em string, que é algo do tipo ``"2008-12-13 14:09:39.002731"``. Isso
  resultará em uma string HTML como
  ``"<html><body>It is now 2008-12-13 14:09:39.002731.</body></html>"``.

  (Sim, esse código HTML é inválido, pois tentamos manter os exemplos
  simples e curtos)

* Por fim, a view retorna um objeto ``HttpResponse`` que contém a resposta,
  assim como fizemos na view ``hello``.

After adding that to ``views.py``, add the URLpattern to ``urls.py`` to tell
Django which URL should handle this view. Something like ``/time/`` would make
sense::

    from django.conf.urls.defaults import patterns, include, url
    from mysite.views import hello, current_datetime

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
    )

We've made two changes here. First, we imported the ``current_datetime``
function at the top. Second, and more importantly, we added a URLpattern
mapping the URL ``/time/`` to that new view. Getting the hang of this?

With the view written and URLconf updated, fire up the ``runserver`` and visit
``http://127.0.0.1:8000/time/`` in your browser. You should see the current
date and time.

.. admonition:: Django's Time Zone

    Depending on your computer, the date and time may be a few hours off.
    That's because Django is time zone-aware and defaults to the
    ``America/Chicago`` time zone. (It has to default to *something*, and that's
    the time zone where the original developers live.) If you live elsewhere,
    you'll want to change it in ``settings.py``. See the comment in that file
    for a link to an up-to-date list of worldwide time zone options.

URLconfs and Loose Coupling
===========================

Now's a good time to highlight a key philosophy behind URLconfs and behind
Django in general: the principle of *loose coupling*. Simply put, loose coupling
is a software-development approach that values the importance of making pieces
interchangeable. If two pieces of code are loosely coupled, then changes made to
one of the pieces will have little or no effect on the other.

Django's URLconfs are a good example of this principle in practice. In a Django
web application, the URL definitions and the view functions they call are
loosely coupled; that is, the decision of what the URL should be for a given
function, and the implementation of the function itself, reside in two separate
places. This lets you switch out one piece without affecting the other.

For example, consider our ``current_datetime`` view. If we wanted to change the
URL for the application -- say, to move it from ``/time/`` to
``/current-time/`` -- we could make a quick change to the URLconf, without
having to worry about the view itself. Similarly, if we wanted to change the
view function -- altering its logic somehow -- we could do that without
affecting the URL to which the function is bound.

Furthermore, if we wanted to expose the current-date functionality at
*several* URLs, we could easily take care of that by editing the URLconf,
without having to touch the view code. In this example, our
``current_datetime`` is available at two URLs. It's a contrived example, but
this technique can come in handy::

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
        url(r'^another-time-page/$', current_datetime),
    )

URLconfs and views are loose coupling in action. We'll continue to point out
examples of this important philosophy throughout this book.

Your Third View: Dynamic URLs
=============================

In our ``current_datetime`` view, the contents of the page -- the current
date/time -- were dynamic, but the URL (``/time/``) was static. In most dynamic
Web applications, though, a URL contains parameters that influence the output
of the page. For example, an online bookstore might give each book its own URL,
like ``/books/243/`` and ``/books/81196/``.

Let's create a third view that displays the current date and time offset by a
certain number of hours. The goal is to craft a site in such a way that the page
``/time/plus/1/`` displays the date/time one hour into the future, the page
``/time/plus/2/`` displays the date/time two hours into the future, the page
``/time/plus/3/`` displays the date/time three hours into the future, and so
on.

A novice might think to code a separate view function for each hour offset,
which might result in a URLconf like this::

    urlpatterns = patterns('',
        url(r'^time/$', current_datetime),
        url(r'^time/plus/1/$', one_hour_ahead),
        url(r'^time/plus/2/$', two_hours_ahead),
        url(r'^time/plus/3/$', three_hours_ahead),
        url(r'^time/plus/4/$', four_hours_ahead),
    )

Clearly, this line of thought is flawed. Not only would this result in redundant
view functions, but also the application is fundamentally limited to supporting
only the predefined hour ranges -- one, two, three or four hours. If we decided
to create a page that displayed the time *five* hours into the future, we'd
have to create a separate view and URLconf line for that, furthering the
duplication. We need to do some abstraction here.

.. admonition:: A Word About Pretty URLs

    If you're experienced in another Web development platform, such as PHP or
    Java, you may be thinking, "Hey, let's use a query string parameter!" --
    something like ``/time/plus?hours=3``, in which the hours would be
    designated by the ``hours`` parameter in the URL's query string (the part
    after the ``?``).

    You *can* do that with Django (and we'll tell you how in Chapter 7), but
    one of Django's core philosophies is that URLs should be beautiful. The URL
    ``/time/plus/3/`` is far cleaner, simpler, more readable, easier to recite
    to somebody aloud and . . . just plain prettier than its query string
    counterpart. Pretty URLs are a characteristic of a quality Web application.

    Django's URLconf system encourages pretty URLs by making it easier to use
    pretty URLs than *not* to.

How, then do we design our application to handle arbitrary hour offsets? The
key is to use *wildcard URLpatterns*. As we mentioned previously, a URLpattern
is a regular expression; hence, we can use the regular expression pattern
``\d+`` to match one or more digits::

    urlpatterns = patterns('',
        # ...
        url(r'^time/plus/\d+/$', hours_ahead),
        # ...
    )

(We're using the ``# ...`` to imply there might be other URLpatterns that we
trimmed from this example.)

This new URLpattern will match any URL such as ``/time/plus/2/``,
``/time/plus/25/``, or even ``/time/plus/100000000000/``. Come to think of it,
let's limit it so that the maximum allowed offset is 99 hours. That means we
want to allow either one- or two-digit numbers -- and in regular expression
syntax, that translates into ``\d{1,2}``::

    url(r'^time/plus/\d{1,2}/$', hours_ahead),

.. note::

    When building Web applications, it's always important to consider the most
    outlandish data input possible, and decide whether or not the application
    should support that input. We've curtailed the outlandishness here by
    limiting the offset to 99 hours.

Now that we've designated a wildcard for the URL, we need a way of passing that
wildcard data to the view function, so that we can use a single view function
for any arbitrary hour offset. We do this by placing parentheses around the
data in the URLpattern that we want to save. In the case of our example, we
want to save whatever number was entered in the URL, so let's put parentheses
around the ``\d{1,2}``, like this::

    url(r'^time/plus/(\d{1,2})/$', hours_ahead),

If you're familiar with regular expressions, you'll be right at home here;
we're using parentheses to *capture* data from the matched text.

The final URLconf, including our previous two views, looks like this::

    from django.conf.urls.defaults import *
    from mysite.views import hello, current_datetime, hours_ahead

    urlpatterns = patterns('',
        url(r'^hello/$', hello),
        url(r'^time/$', current_datetime),
        url(r'^time/plus/(\d{1,2})/$', hours_ahead),
    )

With that taken care of, let's write the ``hours_ahead`` view.

``hours_ahead`` is very similar to the ``current_datetime`` view we wrote
earlier, with a key difference: it takes an extra argument, the number of hours
of offset. Here's the view code::

    from django.http import Http404, HttpResponse
    import datetime

    def hours_ahead(request, offset):
        try:
            offset = int(offset)
        except ValueError:
            raise Http404()
        dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
        html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
        return HttpResponse(html)

Let's step through this code one line at a time:

* The view function, ``hours_ahead``, takes *two* parameters: ``request``
  and ``offset``.

  * ``request`` is an ``HttpRequest`` object, just as in ``hello`` and
    ``current_datetime``. We'll say it again: each view *always* takes an
    ``HttpRequest`` object as its first parameter.

  * ``offset`` is the string captured by the parentheses in the
    URLpattern. For example, if the requested URL were ``/time/plus/3/``,
    then ``offset`` would be the string ``'3'``. If the requested URL were
    ``/time/plus/21/``, then ``offset`` would be the string ``'21'``. Note
    that captured values will always be *strings*, not integers, even if
    the string is composed of only digits, such as ``'21'``.

    (Technically, captured values will always be *Unicode objects*, not
    plain Python bytestrings, but don't worry about this distinction at
    the moment.)

    We decided to call the variable ``offset``, but you can call it
    whatever you'd like, as long as it's a valid Python identifier. The
    variable name doesn't matter; all that matters is that it's the second
    argument to the function, after ``request``. (It's also possible to
    use keyword, rather than positional, arguments in an URLconf. We cover
    that in Chapter 8.)

* The first thing we do within the function is call ``int()`` on ``offset``.
  This converts the string value to an integer.

  Note that Python will raise a ``ValueError`` exception if you call
  ``int()`` on a value that cannot be converted to an integer, such as the
  string ``'foo'``. In this example, if we encounter the ``ValueError``, we
  raise the exception ``django.http.Http404``, which, as you can imagine,
  results in a 404 "Page not found" error.

  Astute readers will wonder: how could we ever reach the ``ValueError``
  case, anyway, given that the regular expression in our URLpattern --
  ``(\d{1,2})`` -- captures only digits, and therefore ``offset`` will only
  ever be a string composed of digits? The answer is, we won't, because
  the URLpattern provides a modest but useful level of input validation,
  *but* we still check for the ``ValueError`` in case this view function
  ever gets called in some other way. It's good practice to implement view
  functions such that they don't make any assumptions about their
  parameters. Loose coupling, remember?

* In the next line of the function, we calculate the current date/time and
  add the appropriate number of hours. We've already seen
  ``datetime.datetime.now()`` from the ``current_datetime`` view; the new
  concept here is that you can perform date/time arithmetic by creating a
  ``datetime.timedelta`` object and adding to a ``datetime.datetime``
  object. Our result is stored in the variable ``dt``.

  This line also shows why we called ``int()`` on ``offset`` -- the
  ``datetime.timedelta`` function requires the ``hours`` parameter to be an
  integer.

* Next, we construct the HTML output of this view function, just as we did
  in ``current_datetime``. A small difference in this line from the previous
  line is that it uses Python's format-string capability with *two* values,
  not just one. Hence, there are two ``%s`` symbols in the string and a
  tuple of values to insert: ``(offset, dt)``.

* Finally, we return an ``HttpResponse`` of the HTML. By now, this is old
  hat.

With that view function and URLconf written, start the Django development server
(if it's not already running), and visit ``http://127.0.0.1:8000/time/plus/3/``
to verify it works. Then try ``http://127.0.0.1:8000/time/plus/5/``. Then
``http://127.0.0.1:8000/time/plus/24/``. Finally, visit
``http://127.0.0.1:8000/time/plus/100/`` to verify that the pattern in your
URLconf only accepts one- or two-digit numbers; Django should display a "Page
not found" error in this case, just as we saw in the section "A Quick Note
About 404 Errors" earlier. The URL ``http://127.0.0.1:8000/time/plus/`` (with
*no* hour designation) should also throw a 404.

.. admonition:: Coding Order

    In this example, we wrote the URLpattern first and the view second, but in
    the previous examples, we wrote the view first, then the URLpattern. Which
    technique is better?

    Well, every developer is different.

    If you're a big-picture type of person, it may make the most sense to you
    to write all of the URLpatterns for your application at the same time, at
    the start of your project, and then code up the views. This has the
    advantage of giving you a clear to-do list, and it essentially defines the
    parameter requirements for the view functions you'll need to write.

    If you're more of a bottom-up developer, you might prefer to write the
    views first, and then anchor them to URLs afterward. That's OK, too.

    In the end, it comes down to which technique fits your brain the best. Both
    approaches are valid.

Django's Pretty Error Pages
===========================

Take a moment to admire the fine Web application we've made so far . . . now
let's break it! Let's deliberately introduce a Python error into our
``views.py`` file by commenting out the ``offset = int(offset)`` lines in the
``hours_ahead`` view::

    def hours_ahead(request, offset):
        # try:
        #     offset = int(offset)
        # except ValueError:
        #     raise Http404()
        dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
        html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
        return HttpResponse(html)

Load up the development server and navigate to ``/time/plus/3/``. You'll see an
error page with a significant amount of information, including a ``TypeError``
message displayed at the very top: ``"unsupported type for timedelta hours
component: unicode"``.

What happened? Well, the ``datetime.timedelta`` function expects the ``hours``
parameter to be an integer, and we commented out the bit of code that converted
``offset`` to an integer. That caused ``datetime.timedelta`` to raise the
``TypeError``. It's the typical kind of small bug that every programmer runs
into at some point.

The point of this example was to demonstrate Django's error pages. Take some
time to explore the error page and get to know the various bits of information
it gives you.

Here are some things to notice:

* At the top of the page, you get the key information about the exception:
  the type of exception, any parameters to the exception (the ``"unsupported
  type"`` message in this case), the file in which the exception was raised,
  and the offending line number.

* Under the key exception information, the page displays the full Python
  traceback for this exception. This is similar to the standard traceback
  you get in Python's command-line interpreter, except it's more
  interactive. For each level ("frame") in the stack, Django displays the
  name of the file, the function/method name, the line number, and the
  source code of that line.

  Click the line of source code (in dark gray), and you'll see several
  lines from before and after the erroneous line, to give you context.

  Click "Local vars" under any frame in the stack to view a table of all
  local variables and their values, in that frame, at the exact point in the
  code at which the exception was raised. This debugging information can be
  a great help.

* Note the "Switch to copy-and-paste view" text under the "Traceback"
  header. Click those words, and the traceback will switch to a alternate
  version that can be easily copied and pasted. Use this when you want to
  share your exception traceback with others to get technical support --
  such as the kind folks in the Django IRC chat room or on the Django users
  mailing list.

  Underneath, the "Share this traceback on a public Web site" button will
  do this work for you in just one click. Click it to post the traceback to
  http://www.dpaste.com/, where you'll get a distinct URL that you can
  share with other people.

* Next, the "Request information" section includes a wealth of information
  about the incoming Web request that spawned the error: GET and POST
  information, cookie values, and meta information, such as CGI headers.
  Appendix G has a complete reference of all the information a request
  object contains.

  Below the "Request information" section, the "Settings" section lists all
  of the settings for this particular Django installation. (We've already
  mentioned ``ROOT_URLCONF``, and we'll show you various Django settings
  throughout the book. All the available settings are covered in detail in
  Appendix D.)

The Django error page is capable of displaying more information in certain
special cases, such as the case of template syntax errors. We'll get to those
later, when we discuss the Django template system. For now, uncomment the
``offset = int(offset)`` lines to get the view function working properly again.

Are you the type of programmer who likes to debug with the help of carefully
placed ``print`` statements? You can use the Django error page to do so -- just
without the ``print`` statements. At any point in your view, temporarily insert
an ``assert False`` to trigger the error page. Then, you can view the local
variables and state of the program. Here's an example, using the
``hours_ahead`` view::

    def hours_ahead(request, offset):
        try:
            offset = int(offset)
        except ValueError:
            raise Http404()
        dt = datetime.datetime.now() + datetime.timedelta(hours=offset)
        assert False
        html = "<html><body>In %s hour(s), it will be %s.</body></html>" % (offset, dt)
        return HttpResponse(html)

Finally, it's obvious that much of this information is sensitive -- it exposes
the innards of your Python code and Django configuration -- and it would be
foolish to show this information on the public Internet. A malicious person
could use it to attempt to reverse-engineer your Web application and do nasty
things. For that reason, the Django error page is only displayed when your
Django project is in debug mode. We'll explain how to deactivate debug mode
in Chapter 12. For now, just know that every Django project is in debug mode
automatically when you start it. (Sound familiar? The "Page not found" errors,
described earlier in this chapter, work the same way.)

What's next?
============

So far, we've been writing our view functions with HTML hard-coded directly
in the Python code. We've done that to keep things simple while we demonstrated
core concepts, but in the real world, this is nearly always a bad idea.

Django ships with a simple yet powerful template engine that allows you to
separate the design of the page from the underlying code. We'll dive into
Django's template engine in the :doc:`next chapter <chapter04>`.
