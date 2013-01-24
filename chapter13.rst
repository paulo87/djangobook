=======================================
Capítulo 13: Gerando conteúdo Não-HTML
=======================================

Normalmente quando falamos sobre desenvolver Web sites, estamos falando sobre produzir
HTML. Naturalmente, existe muito mais na Web que HTML; usamos a Web
para distribuir dados em todo tipo de formato: RSS, PDFs, imagens, etc.

Até agora, nós concentramos no caso comum da produção de HTML, mas neste capítulo
vamos fazer um desvio e olhar para o uso de Django para produção de outros tipos de conteúdo.

Django tem convenientemente ferramentas embutidas que você pode usar para produzir algum conteúdo
não-HTML comum:

* RSS/Atom feeds

* Sitemaps (um formato XML originalmente desenvolvido pelo Google que dá sugestões
  para mecanismos de busca)

Vamos examinar cada uma dessas ferramentas um pouco mais tarde, mas primeiro abordaremos os
princípios básicos.

O básico: views e tipos MIME
============================

Lembre-se do capítulo 3 que uma função view é simplesmente uma função Python que
recebe um pedido Web e retorna uma resposta Web. Essa resposta pode ser o conteúdo HTML
de uma página Web, ou um redirecionamento, ou um erro 404, ou um documento XML,
ou uma imagem... ou qualquer coisa, na verdade.

Mais formalmente, uma função view de Django *precisa*

* Aceitar uma instância ``HttpRequest`` como seu primeiro argumento

* Retornar uma instância ``HttpResponse``

A chave para retornar conteúdo não-HTML de uma view reside na classe ``HttpResponse``
-- mais especificamente no argumento ``mimetype``. Ajustanto o tipo MIME,
podemos indicar ao navegador que retornamos uma resposta de um formato
diferente.

Por exemplo, vamos olhar uma view que retorna uma imagem PNG. Para
manter as coisas simples, vamos somente ler o arquivo do disco::

    from django.http import HttpResponse

    def my_image(request):
        image_data = open("/path/to/my/image.png", "rb").read()
        return HttpResponse(image_data, mimetype="image/png")

.. SL Tested ok

É isso! Se você substituir o caminho da imagem na chamada ``open()`` com o caminho
de uma imagem real, você pode usar essa view muito simples para servir imagens, e o
navegador vai exibir corretamente.

Outra coisa importante para manter em mente é que objetos ``HttpResponse``
implementam o padrão Python "file-like object" API. Isso significa que você pode usar
uma instância ``HttpResponse`` em qualquer lugar que Python (ou uma biblioteca de terceiros)
espera um arquivo.

Como exemplo de como isso funciona, vamos dar uma olhada na produção de CSV com
Django.

Produzindo CSV
==============

CSV é um simples formato de dados normalmente usado por um software de planilha. É basicamente
uma série de linhas de tabela, onde cada célula na linha é separada por uma vírgula (CSV
significa *comma-separated values*). Por exemplo, aqui estão alguns dados de passageiros
"incontroláveis" de uma companhia aérea em formato CSV::

    Ano,Passageiros incontroláveis da companhia aérea
    1995,146
    1996,184
    1997,235
    1998,200
    1999,226
    2000,251
    2001,299
    2002,273
    2003,281
    2004,304
    2005,203
    2006,134
    2007,147

.. note::

    A lista anterior contém números reais! Eles vieram da Administração
    de Aviação Federal dos Estados Unidos.

Embora CSV pareça simples, os seus detalhes de formatação não foram universalmente
acordados. Diferentes softwares produzem e consomem diferentes variações de
CSV, o tornando um pouco complicado de usar. Felizmente, Python vem com uma biblioteca
padrão CSV, ``csv``, que é praticamente "à prova de balas".

Pela razão do módulo ``csv`` operar em objetos arquivo ou similar, é muito fácil de usar
um ``HttpResponse`` em vez disso::

    import csv
    from django.http import HttpResponse

    # Número de passageiros incontroláveis em cada ano 1995 - 2005. Em uma aplicação real
    # isso poderia vir de uma base de dados ou outra forma de guardar dados.
    UNRULY_PASSENGERS = [146,184,235,200,226,251,299,273,281,304,203]

    def unruly_passengers_csv(request):
        # Cria o objeto HttpResponse com o cabeçalho CSV apropriado.
        response = HttpResponse(mimetype='text/csv')
        response['Content-Disposition'] = 'attachment; filename=unruly.csv'

        # Cria o escritor CSV usando o HttpResponse como o "arquivo."
        writer = csv.writer(response)
        writer.writerow(['Year', 'Unruly Airline Passengers'])
        for (year, num) in zip(range(1995, 2006), UNRULY_PASSENGERS):
            writer.writerow([year, num])

        return response

.. SL Tested ok

O código e os comentários devem ser bem claros, mas algumas coisas merecem especial
menção:

* A resposta é dada com o tipo MIME ``text/csv`` (ao invés do padrão
  ``text/html``). Isso diz aos navegadores que o documento é um arquivo CSV.

* A resposta recebe um cabeçalho adicional ``Content-Disposition``, que
  contém o nome do arquivo CSV. Esse cabeçalho (a parte do "anexo") vai instruir
  o navegador para solicitar por uma localização para salvar o arquivo ao invés
  de somente mostrá-lo. O nome desse arquivo é arbitrário; chame como quiser.
  Ele será usado pelos navegadores na caixa de diálogo "Salvar como".

  Para atribuir um cabeçalho em um ``HttpResponse``, apenas trate o
  ``HttpResponse`` como um dicionário e defina uma chave/valor.

* Conectar-se à API de geração de CSV é fácil: basta passar ``response`` como o
  primeiro argumento para ``csv.writer``. A função ``csv.writer`` recebe um
  objeto de tipo arquivo e objetos do tipo ``HttpResponse`` satisfazem essa
  exigência.

* Para cada linha no seu arquivo CSV, chame ``writer.writerow``, passando um objeto
  iterável como uma lista ou tupla.

* Para cada objeto recebido, o módulo CSV fica encarregado de transformá-lo em
  texto entre aspas -- portanto você não precisa se preocupar em escapar strings
  que contêm aspas ou vírgulas.

Esse é o modelo geral que você irá usar em qualquer momento que precise retornar conteúdo
não-HTML: crie um objeto de resposta ``HttpResponse`` (com um tipo MIME especial),
passe a ele algo como um arquivo, e depois retorne uma resposta.

Vamos dar uma olhada em mais alguns exemplos.

Gerando PDFs
============

Portable Document Format (PDF) é um formato desenvolvido pela Adobe usado para
representar documentos para impressão, completa com formatação de pixel perfeita,
fontes incoporadas e gráficos vetoriais em 2D. Você pode pensar em um documento PDF como um
equivalente digital de um documento impresso; de fato, PDFs são frequentemente usados na
distributing documents for the purpose of printing them.
distribuição de documentos com a finalidade de imprimi-los

Você pode facilmente gerar PDFs com Python e Django graças a excelente
biblioteca open source ReportLab (http://www.reportlab.org/rl_toolkit.html).
A vantagem de gerar arquivos PDF dinamicamente é que você pode criar
PDFs customizados para diferentes propósitos -- como para diferentes usuários ou
diferentes pedaços de conteúdo.

Por exemplo, seus humildes autores usaram Django e ReportLab no KUSports.com para
gerar tabelas do NCAA customizadas e prontas para impressão.

Instalando o ReportLab
----------------------

Antes de você fazer qualquer geração de PDF, no entanto, irá precisar instalar o ReportLab.
Normalmente é simples: somente fazer o download e instalar a biblioteca de
http://www.reportlab.org/downloads.html.

.. note::

    Se você estar usando uma distribuição Linux moderna, você pode querer seu
    utilitário de gerenciamento de pacotes antes de instalar o ReportLab. A maioria
    dos repositórios de pacotes tem o ReportLab adicionado.

    Por exemplo, se está usando o Ubuntu, um simples
    ``apt-get install python-reportlab`` vai fazer isso muito bem.

O guia de usuário (naturalmente disponível somente em um arquivo PDF) em
http://www.reportlab.org/rsrc/userguide.pdf possui instruções adicionais de
instalação.

Teste sua instalção importando ela no interpretador interativo de Python::

    >>> import reportlab

Se o comando não levantou nenhum erro, a instalação funcionou.

Escrevendo sua View
-------------------

Como CSV, gerar PDFs dinamicamente com Django é fácil porque a API do ReportLab
atua sobre objeto arquivo ou similar.

Aqui está um exemplo de "Hello World"::

    from reportlab.pdfgen import canvas
    from django.http import HttpResponse

    def hello_pdf(request):
        # Cria o objeto HttpResponse com os cabeçalho PDF apropriado.
        response = HttpResponse(mimetype='application/pdf')
        response['Content-Disposition'] = 'attachment; filename=hello.pdf'

        # Cria o objeto PDF, usando o objeto de resposta como seu "arquivo."
        p = canvas.Canvas(response)

        # Desenha coisas no PDF. É aqui onde a geração do PDF ocorre.
        # Veja a documentação do ReportLab para a lista completa de funcionalidades.
        p.drawString(100, 100, "Hello world.")

        # Fecha o objeto PDF, e está feito.
        p.showPage()
        p.save()
        return response

.. SL Tested ok

Algumas notas estão em ordem:

* Aqui usamos o tipo MIME ``application/pdf``. Isso diz aos navegadores que
  que o documento é um arquivo PDF, em vez de um arquivo HTML. Se você deixar de fora
  esta informação, navegadores vão provavelmente interpretar a resposta como HTML,
  o que vai resultar em algo estranho no janela do navegador.

* Conectar-se à API do ReportLab é fácil: basta passar ``response`` como primeiro
  argumento para ``canvas.Canvas``. A classe ``Canvas`` recebe um objeto de tipo
  arquivo e ``HttpResponse`` satisfaz essa condição.

* Todos os métodos subsequentes para geração do PDF são chamados no objeto PDF
  (nesse caso, ``p``), não em ``response``.

* Finalmente, é importante chamar ``showPage()`` e ``save()``no arquivo PDF
  -- ou então, você vai terminar com um arquivo PDF corrompido.

PDFs complexos
--------------

Se você está criando um documento PDF complexo (ou qualquer dado blob grande)
usando a biblioteca ``cStringIO`` como um local de armazenamento temporário para
o seu arquivo PDF. A biblioteca ``cStringIO`` disponibiliza uma interface para objetos
arquivo ou similar, que é escrito em C para eficiência máxima.

Aqui está o exemplo anterior de "Hello World" reescrito para usar ``cStringIO``::

    from cStringIO import StringIO
    from reportlab.pdfgen import canvas
    from django.http import HttpResponse

    def hello_pdf(request):
        # Cria o objeto HttpResponse com o cabeçalho PDF apropriado.
        response = HttpResponse(mimetype='application/pdf')
        response['Content-Disposition'] = 'attachment; filename=hello.pdf'

        temp = StringIO()

        # Cria o objeto PDF, usando o objeto StringIO como seu "arquivo."
        p = canvas.Canvas(temp)

        # Desenha coisas no PDF. Aqui está onde a geração do PDF ocorre.
        # Veja a documentação do ReportLab para a lista completa de funcionalides.
        p.drawString(100, 100, "Hello world.")

        # Fecha o objeto PDF.
        p.showPage()
        p.save()

        # Pega o valor do buffer StringIO e escreve na resposta.
        response.write(temp.getvalue())
        return response

.. SL Tested ok

Outras possibilidades
=====================

Há todo um conjunto de outros tipos de conteúdo que você pode gerar em Python.
Aqui estão mais algumas ideias e algumas indicações de bibliotecas que você
pode usar para implementá-las:

* *Arquivos ZIP*: A biblioteca padrão de Python vem com
  o módulo ``zipfile``, que pode tanto ler como escrever arquivos ZIP compactados.
  Pode usar isso para prover arquivos sob demanda de um conjunto de arquivos, ou
  talvez compactar documentos grandes quando solicitado. Poderia igualmente
  produzir arquivos TAR usando o módulo ``tarfile`` da biblioteca padrão.

* *Imagens dinâmicas*: A biblioteca de imagem de Python
  (http://www.pythonware.com/products/pil/) é um fantástico kit de ferramentas
  para produzir imagens (PNG, JPEG, GIF, e muito mais). Você pode usar
  ela para automaticamente reduzir as imagens em miniaturas, compor
  múltiplas imagens em um único frame, ou então fazer processamento de imagem
  baseado em Web.

* *Gráficos e tabelas*: existe uma numerosa quantidade de bibliotecas Python
  para a manipualção de gráficos e tabelas que você poderia usar para criar
  mapas, tabelas e gráficos sob demanda. É claro que não podemos listar todas
  elas, então aí vão apenas algumas das mais populares:

* ``matplotlib`` (http://matplotlib.sourceforge.net/) pode ser
  usada para produzir gráficos de alta qualidade usualmente gerados
  com MatLab ou Mathematica.

* ``pygraphviz`` (http://networkx.lanl.gov/pygraphviz/), uma
  interface para o kit de ferramentas de layout gráfico Graphviz
  (http://graphviz.org/), pode ser usada para gerar diagramas estruturados de
  grafos e conexões.

Em geral, qualquer biblioteca Python capaz de escrever em um arquivo pode ser conectada
em Django. As possibilidades são imensas.

Agora que olhamos o básico para gerar conteúdo não-HTML, vamos
aumentar o nível de abstração. Django vem com algumas ferramentas embutidas muito legais
para gerar tipos comuns de conteúdo não-HTML.

O Syndication Feed Framework
============================

Django vem com um framework de geração de feed de alto nível que
torna a criação de feeds RSS e Atom fácil.

.. admonition:: O que é RSS? O que é Atom?

    RSS e Attom são ambos formatos baseados em XML que podem ser usados para disponibilizar
    automaticamente atualizações "feeds" do conteúdo de seu site. Leia mais sobre RSS
    no http://www.whatisrss.com/, e tenha informações sobre Atom no
    http://www.atomenabled.org/.

Para criar um feed de distribuição, tudo que é necessário fazer é escrever uma pequena
classe de Python. Você pode criar tantos feeds quanto quiser.

O framework de geração de feeds de alto nível é uma view que está ligada ao ``/feeds/``
by convention. Django uses the remainder of the URL (everything after
por convenção. Django usa o restante da URL (tudo após
``/feeds/``) para determinar qual feed retornar.

Para criar um feed, você irá escrever uma classe ``Feed`` e apontar para ela no seu
URLconf.

Inicialização
--------------

Para ativar feeds no seu site Django, adicione essa URLconf::

    (r'^feeds/(?P<url>.*)/$', 'django.contrib.syndication.views.feed',
        {'feed_dict': feeds}
    ),

Essa linha diz ao Django para usar o framework RSS para lidar com todas URLs iniciadas com
``"feeds/"``. (Você pode modificar esse prefixo ``"feeds/"`` para atender às suas próprias necessidades.)

Essa linha do URLconf tem um argumento extra: ``{'feed_dict': feeds}``. Use esse
argumento extra para passar ao framework de distribuição os feeds que devem ser
publicados com essa URL.

Mais estritamente, ``feed_dict`` tem que ser um dicionário que mapeia um slug de
feed (isto é, a abreviação de sua URL) em sua classe ``Feed``. Você pode definir
``feed_dict`` no próprio URLconf. Segue um exemplo completo de URLconf::

    from django.conf.urls.defaults import *
    from mysite.feeds import LatestEntries, LatestEntriesByCategory

    feeds = {
        'latest': LatestEntries,
        'categories': LatestEntriesByCategory,
    }

    urlpatterns = patterns('',
        # ...
        (r'^feeds/(?P<url>.*)/$', 'django.contrib.syndication.views.feed',
            {'feed_dict': feeds}),
        # ...
    )

O exemplo anterior registra dois feeds:

* O feed representado por ``LatestEntries`` ficará em
  ``feeds/latest/``.

* O feed representado por ``LatestEntriesByCategory`` ficará em
  ``feeds/categories/``.

Uma vez que está criado, é preciso definir as próprias classes ``Feed``.

Uma classe ``Feed`` é uma simples classe de Python que representa um feed.
Um fid pode ser simples (e.g., um feed de site de notícias ou um feed básico mostrando
os últimos posts do blog) ou mais complexto (e.g., um feed mostrando todos
os posts de uma categoria particular, podendo a categoria variar).

Classes ``Feed`` devem ter subclasse ``django.contrib.syndication.feeds.Feed``. Elas
podem estar em qualquer lugar da árvore de seu código.

Um Feed simples
---------------

Esse exemplo simples descreve um feed das últimas cinco postagens de um
determinado blog::

    from django.contrib.syndication.feeds import Feed
    from mysite.blog.models import Entry

    class LatestEntries(Feed):
        title = "My Blog"
        link = "/archive/"
        description = "The latest news about stuff."

        def items(self):
            return Entry.objects.order_by('-pub_date')[:5]

As coisas importantes para se informar aqui são:

* A subclasse ``django.contrib.syndication.feeds.Feed`` da classe.

* ``title``, ``link`` e ``description`` correspondem aos elementos padrões RSS
  ``<title>``, ``<link>`` e ``<description>``, respectivamente.

* ``items()`` é simplesmente um método que retorna uma lista de objetos que
  pode ser incluída no feed como um elemento ``<item>``. Embora este exemplo
  retorne objetos ``Entry`` usando a API de base de dados de Django,
  ``items()`` não têm que retornar instâncias de modelo.

Há somente mais um passo. Em um feedd RSS, cada ``<item>`` tem um ``<title>``,
``<link>`` e ``<description>``. Precisamos dizer ao framework que dados por
nesses elementos.

* Para especificar os conteúdos de ``<title>`` e ``<description>``, crie
  templates Django chamados ``feeds/latest_title.html`` e
  ``feeds/latest_description.html``, onde ``latest`` é o ``slug``
  especificado na URLconf para o dado feed. Note que a extensão ``.html``
  é necessária.

  O sistema RSS renderiza esse template para cada item, passando para ele duas
  variáveis de contexto:

  * ``obj``: O objeto atual (um dos objetos
    retornados em ``items()``).

  * ``site``: Um objeto ``django.models.core.sites.Site`` representando
    o site. Isso é útil para ``{{ site.domain }}`` ou ``{{
    site.name }}``.

  Se não for criado um template para o título e nem para a descrição, o
  framework usará o template ``"{{ obj }}"`` por padrão -- essa é
  a representação normal de string do objeto. (Para objetos de modelo, esse será
  o método ``__unicode__()``.

  É possível também alterar os nomes desses dois templates especificando
  ``title_template`` e ``description_template`` como atributos de sua
  classe ``Feed``.

* Para especificar os conteúdos de ``<link>``, existem duas opções. Para cada
  item em ``items()``, Django primeiro tenta executar o método
  ``get_absolute_url()`` nesse objeto. Se esse método não existe,
  ele tenta chamar o método ``item_link()`` na classe ``Feed``,
  passando um único parâmetro, ``item``, que é o próprio objeto.

  Ambos ``get_absolute_url()`` e ``item_link()`` devem retornar as URLs dos itens
  como uma string normal de Python.

* Para o exemplo anterior ``LatestEntries``, podemos ter templates de feed muito
  simples. ``latest_title.html`` contém::

        {{ obj.title }}

  e ``latest_description.html`` contém::

        {{ obj.description }}

  É quase *muito* fácil...

.. SL Tested ok

Um Feed mais completo
---------------------

O framework também suporta feeds mais complexos, via parâmetros.

Por exemplo, digamos que seu blog ofereça um feed RSS para toda "tag" distinta
que você usou para categorizar seus posts. Seria tolo criar uma classe
``Feed`` separada para cada tag; isso violaria o princípio Don't Repeat Yourself
(DRY) e acoplaria os dados às regras de negócio da aplicação.

Ao invés disso, o framework deixa criar feeds genéricos
que retorna itens baseados na informação da URL do feed.

Seus feeds com tags específicas podem usar URLs como essas:

* ``http://example.com/feeds/tags/python/``:
  Retorna posts recentes com a tag "python"

* ``http://example.com/feeds/tags/cats/``:
  Retorna posts recentes com a tag "cats"

O slug, nesse caso, é ``"tags"``. O framework enxerga o caminho após o slug --
``'python'`` e ``'cats'`` -- e lhe permite criar um hook para informar o que
esse caminho extra significa e como ele deve influenciar a escolha de quais itens
são publicados no feed.

Um exemplo deixa isso claro. Aqui está o código para feeds de tag específica::

    from django.core.exceptions import ObjectDoesNotExist
    from mysite.blog.models import Entry, Tag

    class TagFeed(Feed):
        def get_object(self, bits):
            # No caso de "/feeds/tags/cats/dogs/mice/", ou outro
            # clutter, cheque se bits têm apenas um membro.
            if len(bits) != 1:
                raise ObjectDoesNotExist
            return Tag.objects.get(tag=bits[0])

        def title(self, obj):
            return "My Blog: Entries tagged with %s" % obj.tag

        def link(self, obj):
            return obj.get_absolute_url()

        def description(self, obj):
            return "Entries tagged with %s" % obj.tag

        def items(self, obj):
            entries = Entry.objects.filter(tags__id__exact=obj.id)
            return entries.order_by('-pub_date')[:30]

Aqui está o algoritmo básico do framework RSS, dada essa classe e uma
requisição à URL ``/feeds/tags/python/``:

#. O framework recebe a URL ``/feeds/tags/python/`` e percebe que existe uma parte
   a mais na URL depois do slug. Ele divide a string excedente em cada uma das
   barras (``"/"``) e chama o método ``get_object()`` da classe ``Feed``, passando
   como parâmetro essa parte excedente (já dividida).

   No caso do exemplo, a parte extra é ``['python']``. Em uma requisição para
   ``/feeds/tags/python/django/``, a divisão ficaria ``['python', 'django']``.

#. ``get_object()`` é responsável por recuperar determinado objeto ``Tag`` a
   partir dos ``bits`` dados (resultantes da divisão da parte extra da URL).

   Nesse caso, é usada a API de banco de dados do Django para recuperar o ``Tag``.
   Observe que ``get_object()`` deve lançar ``django.core.exceptions.ObjectDoesNotExist``
   se parâmetros inválidos forem fornecidos. Não há nenhum ``try`` ou ``except``
   em volta da chamada a ``Tag.objects.get()`` por que não é necessário. Aquela
   função lança ``Tag.DoesNotExist`` quando ocorre uma falha -- e ``Tag.DoesNotExist``
   é uma subclasse de ``ObjectDoesNotExist``. Lançar ``ObjectDoesNotExist`` em
   ``get_object()`` faz com que Django produza um erro 404 para essa requisição.

#. Para gerar os elementos de feed ``<title>``, ``<link>`` e ``<description>``,
   Django usa os métodos ``title()``, ``link()``, and ``description()``.
   No exemplo anterior, eles eram simples strings de atributos de classe, mas
   esse exemplo ilustra que eles podem ser strings *ou* métodos.
   Para cada ``title``, ``link``, and ``description``, Django segue
   esse algoritmo:

   #. Tenta chamar um método, passando o argumento ``obj``,
      onde ``obj`` é o objeto retornado por ``get_object()``.

   #. Falhando esse, tenta chamar um método sem argumentos.

   #. Falhando esse, usa o atributo da classe.

#. Finalmente, note que ``items()`` nesse exemplo também tem o argumento ``obj``.
   O algoritmo para ``items`` é o mesmo que o descrito no passo anterior
   -- primeiro, tenta ``items(obj)``, depois ``items()``, e depois
   finalmente atributo de classe ``items`` (que deve ser uma lista).

Documentação completa de todos os métodos e atributos da classe ``Feed`` está
sempre disponível na documentação oficial de Django
(http://docs.djangoproject.com/en/dev/ref/contrib/syndication/).

Especificando o tipo de Feed
----------------------------

Por padrão, o framework produz RSS 2.0. Para mudar isso,
adicione um atributo ``feed_type`` a sua classe ``Feed``::

    from django.utils.feedgenerator import Atom1Feed

    class MyFeed(Feed):
        feed_type = Atom1Feed

.. SL Tested ok

Note que você definiu ``feed_type`` para um objeto de classe, não uma instância.
Note that you set ``feed_type`` to a class object, not an instance. Tipos de feeds
disponíveis atualmente são mostrados na Tabela 11-1.

.. table:: Tabela 11-1. Tipos de Feed

    ===================================================  =====================
    Classe de Feed                                       Formato
    ===================================================  =====================
    ``django.utils.feedgenerator.Rss201rev2Feed``        RSS 2.01 (padrão)

    ``django.utils.feedgenerator.RssUserland091Feed``    RSS 0.91

    ``django.utils.feedgenerator.Atom1Feed``             Atom 1.0
    ===================================================  =====================

Enclosures
----------

Para especificar enclosures (i.e., recursos de mídia associados com um item de feed como
feeds de podcast), use ``item_enclosure_url``, ``item_enclosure_length``,
e ``item_enclosure_mime_type``, por exemplo::

    from myproject.models import Song

    class MyFeedWithEnclosures(Feed):
        title = "Example feed with enclosures"
        link = "/feeds/example-with-enclosures/"

        def items(self):
            return Song.objects.all()[:30]

        def item_enclosure_url(self, item):
            return item.song_url

        def item_enclosure_length(self, item):
            return item.song_length

        item_enclosure_mime_type = "audio/mpeg"

.. SL Tested ok

Isso pressupõe, é claro, que foi criado um objeto ``Song`` com os campos ``song_url``
e ``song_length`` (i.e., o tamanho em bytes).

Linguagem
---------

Feeds criados com o framework syndication automaticamente incluem uma
apropriada tag ``<language>`` (RSS 2.0) ou atributo ``xml:lang`` (Atom).
Isso vem diretamente da sua configuração de ``LANGUAGE_CODE``.

URLs
----

O método/atributo ``link``pode retornar uma URL absoluta (e.g.,
``"/blog/"``) ou uma URL com o domínio completo e o protocolo (e.g.,
``"http://www.example.com/blog/"``). Se ``link`` não retornar o domínio,
o framework syndication irá inserir o domínio do site,
de acordo com sua configuração ``SITE_ID``. (Veja o capítulo 16 para mais sobre ``SITE_ID``
e o framework de sites.)

Feeds Atom necessitam de ``<link rel="self">`` que define a localização atual
do feed. O framework preenche isso automaticamente.

Publicando Atom e Feeds RSS no Tandem
-------------------------------------

Alguns desenvolvedores gostam de deixar disponíveis ambas versões Atom e RSS
de seus feeds. Isso é fácil de fazer com Django: somente crie a subclasse de sua
classe ``feed`` e defina o ``feed_type`` para algo diferente. Depois atualize seu
URLconf para adicionar versões extras. Veja um exemplo completo:

    from django.contrib.syndication.feeds import Feed
    from django.utils.feedgenerator import Atom1Feed
    from mysite.blog.models import Entry

    class RssLatestEntries(Feed):
        title = "My Blog"
        link = "/archive/"
        description = "The latest news about stuff."

        def items(self):
            return Entry.objects.order_by('-pub_date')[:5]

    class AtomLatestEntries(RssLatestEntries):
        feed_type = Atom1Feed

E aqui está o que acompanha URLconf::

    from django.conf.urls.defaults import *
    from myproject.feeds import RssLatestEntries, AtomLatestEntries

    feeds = {
        'rss': RssLatestEntries,
        'atom': AtomLatestEntries,
    }

    urlpatterns = patterns('',
        # ...
        (r'^feeds/(?P<url>.*)/$', 'django.contrib.syndication.views.feed',
            {'feed_dict': feeds}),
        # ...
    )

.. SL Tested ok

O framework Sitemap
===================

O *sitemap* é um arquivo XML no seu site web que diz aos indexadores de busca
com que frequência suas páginas mudam e quão "importante" certas páginas são
em relação a outras páginas de seu site. Essa informação ajuda mecanismos de busca
a indexar seu site.

Por exemplo, aqui está um pedaço de um sitemap para um site web Django
(http://www.djangoproject.com/sitemap.xml)::

    <?xml version="1.0" encoding="UTF-8"?>
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      <url>
        <loc>http://www.djangoproject.com/documentation/</loc>
        <changefreq>weekly</changefreq>
        <priority>0.5</priority>
      </url>
      <url>
        <loc>http://www.djangoproject.com/documentation/0_90/</loc>
        <changefreq>never</changefreq>
        <priority>0.1</priority>
      </url>
      ...
    </urlset>

Para mais informações sobre sitemaps, veja http://www.sitemaps.org/.

O framework sitemap de Django automatiza a criação do arquivo XML
deixando que se expresse essa informação em código Python. Para criar um sitemap,
somente é preciso escrever uma classe ``Sitemap`` e apontar para ela em sua URLconf.

Instalação
----------

Para instalar a aplicação sitemap, siga essses passos:

#. Adicione ``'django.contrib.sitemaps'`` ao seu ``INSTALLED_APPS`` setting.

#. Tenha certeza que ``'django.template.loaders.app_directories.load_template_source'``
   está na configuração do seu ``TEMPLATE_LOADERS``. O comportamento padrão é que ele
   já esteja presente -- portanto você só precisará incluí-lo novamente se já
   tiver modificado essas configurações.

#. Tenha certeza que que você instalou o framework sites (veja o Capítulo 16).

.. note::
    A aplicação sitemap não instala qualquer tabela de banco de dados. A única
    razão da necessidade de ir para ``INSTALLED_APPS`` é que o carregador de template
    ``load_template_source`` pode achar os templates padrões.

Inicialização
--------------

Para ativar a geração de sitemap no seu site Django, adicione essa linha ao seu
URLconf::

    (r'^sitemap\.xml$', 'django.contrib.sitemaps.views.sitemap', {'sitemaps': sitemaps})

Essa linha comunica ao Django para construir um sitemap quando o cliente acessar
``/sitemap.xml``. Note que o caractere de ponto em ``sitemap.xml`` é escapado
com uma barra invertida, pois pontos tem um sentido especial em expressões regulares.

O nome do arquivo do sitemap não é importante, mas a localização sim. Motores
de busca vão somente indexar links no seu sitemap para o nível atual da URL e
abaixo. Por exemplo, se ``sitemap.xml`` está no seu diretório raiz, pode refenciar
qualquer URL no seu site. No entanto, se seu sitemap está em ``/content/sitemap.xml``,
pode somente referenciar URLs que comecem com ``/content/``.

A view de sitemap tem um adicional, argumento necessário: ``{'sitemaps':
sitemaps}``. ``sitemaps`` deve ser um dicionário que mapeia um rótulo de seção
curta (e.g., ``blog`` ou ``news``) para sua classe ``Sitemap` (e.g.,
``BlogSitemap`` ou ``NewsSitemap``). Pode também mapear uma *instância* de uma
classe ``Sitemap`` (e.g., ``BlogSitemap(some_var)``).

Classes Sitemap
---------------

Uma classe ``Sitemap`` é uma simples classe Python que representa uma "seção" de
entradas em seu sitemap. Por exemplo, uma classe ``Sitemap`` pode representar
todas as entradas de seu weblog, enquanto outro pode representar todos os
eventos em seu calendário de eventos.

No caso mais simples, todas essas seções fica agrupadas em
``sitemap.xml``, mas é também possível usar o framework para gerar um
index sitemap que refencia individualmente arquivos sitemap, um por seção
(como descrito em breve).

Classes ``Sitemap`` devem possuir subclasse ``django.contrib.sitemaps.Sitemap``. Podem
estar em qualquer lugar em sua árvore de código.

Por exemplo, vamos assumir que você tem um sistema de blog, com um modelo ``Entry`` e
deseja que seu sitemap inclua todos os links de seus posts no blog.
Aqui está como a classe ``Sitemap`` pode parecer::

    from django.contrib.sitemaps import Sitemap
    from mysite.blog.models import Entry

    class BlogSitemap(Sitemap):
        changefreq = "never"
        priority = 0.5

        def items(self):
            return Entry.objects.filter(is_draft=False)

        def lastmod(self, obj):
            return obj.pub_date

.. SL Tested ok

Declarar um ``Sitemap`` deve parecer similar a declarar um ``Feed``.
Isso é por design.

Assim como classes ``Feed``, membros ``Sitemap`` podem ser métodos ou
atributos. Veja as etapas da seção anterior "Um Exemplo Complexo" para saber
mais sobre como isso funciona.

Uma classe ``Sitemap`` pode definir os seguintes métodos/atributos:

* ``items`` (**obrigatório**): Fornece lista de objetos. O framework
  não se importa com o *tipo* dos objetos; o única coisa que importa é que
  esses objetos são passados aos métodos ``location()``, ``lastmod()``,
  ``changefreq()`` e ``priority()``.

* ``location`` (opcional): Fornece a URL absoluta para um determinado objeto.
  Aqui, "URL absoluta" significa uma URL que não inclui o protocolo ou domínio.
  Veja alguns exemplos:

  * Bom: ``'/foo/bar/'``
  * Ruim: ``'example.com/foo/bar/'``
  * Ruim: ``'http://example.com/foo/bar/'``

  Se ``location`` não for fornecido, o framework vai chamar o
  método ``get_absolute_url()`` em cada objeto retornado por
  ``items()``.

* ``lastmod`` (opcional): A data do objeto "last modification", como um
  objeto ``datetime`` de Python.

* ``changefreq`` (opcional): Quão frequentemente o objeto muda. Valores possíveis
  (tal como consta na especificação de Sitemaps) são os seguintes:

  * ``'always'``
  * ``'hourly'``
  * ``'daily'``
  * ``'weekly'``
  * ``'monthly'``
  * ``'yearly'``
  * ``'never'``

* ``priority`` (opcional): Uma prioridade de indexação sugerida entre ``0.0``
  e ``1.0``. A prioridade padrão de uma página é ``0.5``; veja a documentação de
  http://sitemaps.org/ para saber mais como o ``priority`` funciona.

Atalhos
-------

O framework sitemaps fornece algumas classes convenientes para casos comuns. Essas
são descritas nas seções a seguir.

FlatPageSitemap
```````````````

A classe ``django.contrib.sitemaps.FlatPageSitemap`` analisa todas as 'páginas planas'
definidas para o site e cria uma entrada para o sitemap. Essas entradas
incluem somente o atributo ``location`` -- não ``lastmod``,
``changefreq`` ou ``priority``.

Veja o Capítulo 16 para saber mais sobre 'páginas planas'.

GenericSitemap
``````````````
A classe ``GenericSitemap`` funciona como quaisquer views genéricas (veja o Capítulo 11) que
você já possui.

Para usá-la, crie uma instância, passando na mesma ``info_dict`` passadas as
views genéricas. O único requisito é que o dicionário tenha uma
entrada ``queryset``. Também pode ter uma entrada ``date_field`` que especifica
um campo de data para objetos  recuperados do ``queryset``. Isso vai ser usado para
o atributo ``lastmod`` no sitemap gerado. Pode também passar os argumentos chave
``priority`` e ``changefreq`` para o construtor de ``GenericSitemap``
para especificar os atributos para todas as URLs.

Veja um exemplo de uma URLconf usando ``FlatPageSitemap`` e
``GenericSiteMap`` (com o hipotético objeto ``Entry`` de antes)::

    from django.conf.urls.defaults import *
    from django.contrib.sitemaps import FlatPageSitemap, GenericSitemap
    from mysite.blog.models import Entry

    info_dict = {
        'queryset': Entry.objects.all(),
        'date_field': 'pub_date',
    }

    sitemaps = {
        'flatpages': FlatPageSitemap,
        'blog': GenericSitemap(info_dict, priority=0.6),
    }

    urlpatterns = patterns('',
        # some generic view using info_dict
        # ...

        # the sitemap
        (r'^sitemap\.xml$',
         'django.contrib.sitemaps.views.sitemap',
         {'sitemaps': sitemaps})
    )

Criando um Sitemap Index
------------------------

O framework sitemap também tem a habilidade de criar um index sitemap que
referencia individualmente arquivos sitemap, um por cada seção definido em seu
dicionário ``sitemaps``. As únicas diferenças no uso são as seguintes:

* Use duas views em sua URLconf:
  ``django.contrib.sitemaps.views.index`` e
  ``django.contrib.sitemaps.views.sitemap``.

* A view ``django.contrib.sitemaps.views.sitemap`` deve receber um argumento chave
  ``section``.

Aqui está como as linhas relevantes do URLconf devem parecer no exemplo anterior::

    (r'^sitemap.xml$',
     'django.contrib.sitemaps.views.index',
     {'sitemaps': sitemaps}),

    (r'^sitemap-(?P<section>.+).xml$',
     'django.contrib.sitemaps.views.sitemap',
     {'sitemaps': sitemaps})

Isso irá gerar automaticamente o arquivo ``sitemap.xml`` que referencia
``sitemap-flatpages.xml`` e ``sitemap-blog.xml``. As classes ``Sitemap``
e o dicionário ``sitemaps`` não muda em nada.

Pingando o Google
-----------------

Você pode querer dar um "ping" no Google quando seu sitemap mudar, para deixá-lo saber
para reindexar seu site. O framework fornece uma função para fazer isso:
``django.contrib.sitemaps.ping_google()``.

``ping_google()`` recebe um argumento opcional, ``sitemap_url``, que pode ser
a URL absoluta do sitemap de seu site (e.g., ``'/sitemap.xml'``). Se esse
argumento não for fornecido, ``ping_google()`` vai tentar descobrir o seu
sitemap realizando uma pesquisa inversa em seu URLconf.

``ping_google()`` levanta a exceção
``django.contrib.sitemaps.SitemapNotFound`` se não consegue determinar a
URL do sitemap.

Uma maneira útil de chamar ``ping_google()`` é do método ``save()`` de um modelo::

    from django.contrib.sitemaps import ping_google

    class Entry(models.Model):
        # ...
        def save(self, *args, **kwargs):
            super(Entry, self).save(*args, **kwargs)
            try:
                ping_google()
            except Exception:
                # Bare 'except' because we could get a variety
                # of HTTP-related exceptions.
                pass

Uma solução mais eficiente, no entanto, poderia ser chamar ``ping_google()`` de um
script ``cron`` ou alguma outra tarefa programada. Essa função faz uma requisição HTTP
ao servidores do Google, então talvez você não queira introduzir esse overhead
de rede cada vez que chama ``save()``.

Finalmente, se ``'django.contrib.sitemaps'`` está em seu ``INSTALLED_APPS``, depois
o ``manage.py`` vai incluir um novo comando, ``ping_google``. Isso é útil
para acessso a linha de comando para pingar. Por exemplo::

    python manage.py ping_google /sitemap.xml

O que vem a seguir?
===================

Em seguida, vamos continuar nos aprofundando nas ferramentas embutidas que Django lhe oferece.
`Capítulo 14`_ examina todas as ferramentas que você precisa para fornecer sites customizados para usuários:
sessões, usuários e autenticação.

.. _Chapter 14: ../chapter14/
