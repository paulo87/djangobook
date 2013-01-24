============================
Capítulo 11: views genéricas
============================

Novamente nos deparamos com um tema recorrente neste livro: em sua pior faceta,
desenvolvimento Web é chato e monótono. Até agora vimos como Django tenta
suplantar parte dessa monotonia nas camadas de modelo e template. No entanto,
desenvolvedores Web também enfrentam essa chatice na camada de View.

As *Views Genéricas* de Django foram desenvolvidas para amenizar esse sofrimento.
Elas abstraem alguns padrões e expressões comumente encontrados no
desenvolvimento da view de maneira que você pode escrever views comuns para dados
sem ter que escrever muito código. De fato, quase todos os exemplos de views nos
capítulos anteriores poderiam ser reescritos com a ajuda de views genéricas.

O Capítulo 8 discorreu brevemente sobre como você faria para criar uma view
"genérica". Para relembrar, podemos reconhecer algumas tarefas comuns -- como
exibir uma lista de objetos ou escrever código que exibe uma lista de *quaisquer*
objetos. Dessa maneira, o modelo em questão pode ser passado como um argumento
extra para o URLconf.

Django lança mão de views genéricas para os seguintes propósitos:

* Realizar tarefas "simples" que são bastante comuns: redirecionar para uma
  página diferente ou renderizar um dado template.

* Exibir páginas de "listagem" e "detalhamento" de um único objeto. As views
  ``event_list`` e ``entry_list`` do Capítulo 8 são exemplos de views de
  listagem. Uma página de evento único é um exemplo do que chamamos de view
  "detalhada".

* Apresentar objetos datados em páginas de arquivo por ano/mês/dia, detalhes
 associados e páginas "mais recentes". Os arquivos de ano, mês e dia do Blog
 do Django (http://www.djangoproject.com/weblog/) são implementados com esse
 tipo de view -- da mesma maneira como seriam os arquivos de um jornal web.

Vistas em conjunto, essas views fornecem interfaces fáceis para realizar as
tarefas mais comuns com as quais os desenvolvedores se deparam.

Usando views genéricas
======================

Todas essas views são usadas através da criação de dicionários de configuração
em seus arquivos URLconf e da passagem desses dicionários como o terceiro
parâmetro para a tupla da URLconf para um determinado padrão. (Veja "Passando
Opções Extras para as Funções da View" no Capítulo 8 para uma visão geral dessa
técnica).

Por exemplo, aqui está um URLconf simples que você poderia usar para apresentar
uma página "about" estática::

    from django.conf.urls.defaults import *
    from django.views.generic.simple import direct_to_template

    urlpatterns = patterns('',
        (r'^about/$', direct_to_template, {
            'template': 'about.html'
        })
    )

Apesar de parecer "mágica" à primeira vista -- veja só, uma view sem nenhum
código! --, é exatamente a mesma coisa que acontece nos exemplos do Capítulo 8:
a view ``direct_to_template`` simplesmente pega as informações do dicionário
dos parâmetros extras e as usa para a renderização da view.

Pelo fato de que essa view genérica -- e todas as outras -- é uma função de view
tão válida quanto qualquer outra, podemos reusá-la dentro de nossas próprias
views. Como exemplo, vamos estender nosso exemplo do "about" para mapear URLs
no formato ``/about/<qualquer_coisa>/`` para páginas
``about/<qualquer_coisa>.html`` estaticamente renderizadas. Faremos isso através
da modificação inicial do URLconf para apontar para uma função de view:

.. parsed-literal::

    from django.conf.urls.defaults import *
    from django.views.generic.simple import direct_to_template
    **from mysite.books.views import about_pages**

    urlpatterns = patterns('',
        (r'^about/$', direct_to_template, {
            'template': 'about.html'
        }),
        **(r'^about/(\\w+)/$', about_pages),**
    )

Em seguida, escreveremos a view ``about_pages``::

    from django.http import Http404
    from django.template import TemplateDoesNotExist
    from django.views.generic.simple import direct_to_template

    def about_pages(request, page):
        try:
            return direct_to_template(request, template="about/%s.html" % page)
        except TemplateDoesNotExist:
            raise Http404()

Estamos tratando ``direct_to_template`` como uma função qualquer. Como ela
retorna um ``HttpResponse``, podemos simplesmente retorná-la da maneira como
está. O único negócio ligeiramente complicado nessa parte é lidar com templates
inexistentes. Não queremos que nenhum desses templates cause um erro de servidor,
por isso capturamos a exceção ``TemplateDoesNotExist`` e retornamos um erro 404.

.. admonition:: Existe uma vulnerabilidade de segurança nesse código?

    Leitores mais atentos devem ter percebido a possibilidade de uma falha de
    segurança: estamos construindo o nome do template usando conteúdo
    interpolado do browser (``template="about/%s.html" % page``). À primeira
    vista, isso parece uma vulnerabilidade de *directory traversal* (discutida
    em detalhes no Capítulo 20). Mas será que realmente se trata de uma
    vulnerabilidade?

    Not exactly. Yes, a maliciously crafted value of ``page`` could cause
    directory traversal, but although ``page`` *is* taken from the request URL,
    not every value will be accepted. The key is in the URLconf: we're using
    the regular expression ``\w+`` to match the ``page`` part of the URL, and
    ``\w`` only accepts letters and numbers. Thus, any malicious characters
    (such as dots and slashes) will be rejected by the URL resolver before they
    reach the view itself.

    Não exatamente. Sim, um valor malicioso de ``page`` poderia causar directory
    traversal mas, mesmo que ``page`` *seja* recebido da URL de requisição, nem
    todo valor será aceito. O segredo está no URLconf: estamos usando a expressão
    regular ``\w+`` para pegar o parâmetro ``page`` da URL e ``\w`` só aceita
    letras e números. Assim sendo, qualquer caractere malicioso (como pontos e
    barras) será rejeitado pelo matcher de expressão regular associado à URL
    antes que eles cheguem à view.

Views genéricas de objetos
==========================

A view ``direct_to_template`` certamente é bastante útil, mas as views genéricas
de Django brilham de verdade quando são usadas para apresentar views do conteúdo
do nosso banco de dados. Por essa ser uma tarefa corriqueira, Django lança mão
de um punhado de views genéricas nativas que fazem a geração de views de listagem
e detalhamento de objetos incrivelmente fácil.

Vamos dar uma olhada em uma dessas views genéricas: a view de "lista de objeto".
Usaremos este objeto ``Publisher`` do Capítulo 5::

    class Publisher(models.Model):
        name = models.CharField(max_length=30)
        address = models.CharField(max_length=50)
        city = models.CharField(max_length=60)
        state_province = models.CharField(max_length=30)
        country = models.CharField(max_length=50)
        website = models.URLField()

        def __unicode__(self):
            return self.name

        class Meta:
            ordering = ['name']

Para construir uma página de lista de todos os publicadores, nós usaríamos um
URLconf com as seguintes linhas::

    from django.conf.urls.defaults import *
    from django.views.generic import list_detail
    from mysite.books.models import Publisher

    publisher_info = {
        'queryset': Publisher.objects.all(),
    }

    urlpatterns = patterns('',
        (r'^publishers/$', list_detail.object_list, publisher_info)
    )

Esse é todo o código Python que precisamos escrever. Contudo, ainda temos que
criar um template. Podemos dizer explicitamente à view ``object_list`` qual
template usar incluindo uma chave ``template_name`` no dicionário de argumentos
extra:

.. parsed-literal::

    from django.conf.urls.defaults import *
    from django.views.generic import list_detail
    from mysite.books.models import Publisher

    publisher_info = {
        'queryset': Publisher.objects.all(),
        **'template_name': 'publisher_list_page.html',**
    }

    urlpatterns = patterns('',
        (r'^publishers/$', list_detail.object_list, publisher_info)
    )

Quando o ``template_name`` estiver faltando, a view genérica ``object_list`` irá
inferir um template a partir do nome do objeto. Nesse caso, o template inferido
será ``"books/publisher_list.html"`` -- a parte "books" vem do nome do app que
define o modelo e a parte do "publisher" é simplesmente o nome do modelo em
letras minúsculas.

Este template será renderizado num contexto que contém a variável chamada
``object_list`` que guarda todos os objetos do tipo Publisher. Um template
simples pode parecer com o apresentado a seguir::

    {% extends "base.html" %}

    {% block content %}
        <h2>Publishers</h2>
        <ul>
            {% for publisher in object_list %}
                <li>{{ publisher.name }}</li>
            {% endfor %}
        </ul>
    {% endblock %}

.. SL Tested ok

(Observe que esse código pressupõe a existência de um template ``base.html``
tal qual foi fornecido em um exemplo do Capítulo 4).

E isso é tudo que temos para essa view genérica. Todas as funcionalidades legais
de views genéricas advêm da modificação do dicionário de "informações" passado
para elas. O Apêndice D documenta todas as views genéricas e suas opções em
detalhes; o restante deste capítulo irá considerar algumas das abordagens
comuns que você pode utilizar para customizar e estender views genéricas.

Estendendo views genéricas
==========================

Não há dúvida de que a utilização de views genéricas pode acelerar
consideravelmente o desenvolvimento. Para a maioria dos projetos, no entanto,
é chegado um momento em que as views genéricas não são mais suficientes. De fato,
uma das questões mais comuns entre desenvolvedores que estão ingressando em
Django é como fazer para que as views genéricas abranjam uma maior variedade
de situações.

Felizmente, para quase todos os casos há maneiras simples de estender views
genéricas para que elas abarquem uma maior quantidade de casos de uso. Essas
situações normalmente se equiparam a um punhado de padrões que são abordados
nas seções seguintes.

Criando contextos de templates "amigáveis"
------------------------------------------

Você deve ter percebido que o exemplo do template de listagem de publicadores
armazena todos os livros em uma variável chamada ``object_list``. Mesmo que isso
funcione bem, não é "amigável" para o template de autores: eles têm "simplesmente"
que "saber" que estão lidando com livros. Um nome melhor para essa variável seria
``publisher_list``; o conteúdo dessa variável fica, então, bastante óbvio.

Podemos mudar o nome dessa variável facilmente através do  argumento
``template_object_name``:

.. parsed-literal::

    from django.conf.urls.defaults import *
    from django.views.generic import list_detail
    from mysite.books.models import Publisher

    publisher_info = {
        'queryset': Publisher.objects.all(),
        'template_name': 'publisher_list_page.html',
        'template_object_name': 'publisher',
    }

    urlpatterns = patterns('',
        (r'^publishers/$', list_detail.object_list, publisher_info)
    )

Nesse template, a view genérica irá acrescentar ``_list`` a ``template_object_name``
para criar o nome de variável que representa a lista de itens.

Providing a useful ``template_object_name`` is always a good idea. Your coworkers
who design templates will thank you.

Fornecer um ``template_object_name`` útil é sempre uma boa ideia. Seus colegas de
desenvolvimento que implementam templates irão lhe agradecer.

Adicionando contexto extra
--------------------------

Algumas vezes você pode precisar apresentar alguma informação extra -- além
daquela fornecida pela view genérica. Pense, por exemplo, em exibir uma lista
de todos os outros publicadores na página de detalhamento de cada publicador.
A view genérica ``object_detail`` fornece o publicador ao contexto mas não
parece haver um jeito de fornecer uma lista de *todos* publicadores para esse
template.

Mas existe: todas as views genéricas recebem um parâmetro extra opcional,
``extra_context``. Trata-se de um dicionário de objetos extras que serão
adicionados ao contexto do template. Dessa maneira, para fornecer uma lista de
todos os publicadores na view de detalhamento, teríamos que usar um dicionário
de informações como este:

.. parsed-literal::

    publisher_info = {
        'queryset': Publisher.objects.all(),
        'template_object_name': 'publisher',
        **'extra_context': {'book_list': Book.objects.all()}**
    }

.. SL Tested ok

Esse código fornece uma variável ``{{ book_list }}`` ao contexto do template.
Tal padrão pode ser usado para passar qualquer informação para o template da
view genérica. É muito útil.

No entanto, existe, aqui, um bug sutil. Você pode detetá-lo?

The problem has to do with when the queries in ``extra_context`` are evaluated.
Because this example puts ``Book.objects.all()`` in the URLconf, it will
be evaluated only once (when the URLconf is first loaded). Once you add or
remove publishers, you'll notice that the generic view doesn't reflect those
changes until you reload the Web server (see "Caching and QuerySets" in
Appendix C for more information about when ``QuerySet`` objects are cached and
evaluated).

O problema está relacionado com o momento em que as queries em ``extra_context``
são executadas. Pelo fato de esse exemplo colocar ``Book.objects.all()`` no
URLconf, ele será executado somente uma vez (quando o URLconf for carregado).
Assim que você adicionar ou remover publicadores, perceberá que a view genérica
não condiz com as mudanças até que você recarrege o servidor Web (veja "Caching
e QuerySets" no Apêndice C para mais informações sobre quando ocorre cach e
execução de objetos ``QuerySet``).

.. note::

    Esse problema não se aplica ao argumento ``queryset`` da view genérica. Por
    Django estar ciente de que *nunca* deve ser feito cach de QuerySet, a view
    genérica irá cuidar de limpar o cache após a renderização de cada view.

A solução é utilizar um *callback* em ``extra_context`` em vez de um valor.
Qualquer invocável (isto é, uma função) que é passado ao ``extra_context`` será
executada quando a view for renderizada (ao invés de apenas uma única vez). Você
pode fazer isso com uma função definida explicitamente:

.. parsed-literal::

    **def get_books():**
        **return Book.objects.all()**

    publisher_info = {
        'queryset': Publisher.objects.all(),
        'template_object_name': 'publisher',
        'extra_context': **{'book_list': get_books}**
    }

Ou poderia usar uma versão menos óbvia mas mais curta que se vale do fato de que
``Book.objects.all`` é por si só uma função invocável:

.. parsed-literal::

    publisher_info = {
        'queryset': Publisher.objects.all(),
        'template_object_name': 'publisher',
        'extra_context': **{'book_list': Book.objects.all}**
    }

Atente para a falta de parênteses depois de ``Book.objects.all``. Isso faz com
que a função seja referenciada sem que seja executada (visto que ela será
executada pela view depois).

Visualizando subconjuntos de objetos
------------------------------------

Agora vamos olhar mais de perto a chave ``queryset`` que estivemos utilizando
por todo o caminho até aqui. A maioria das views genéricas recebem um desses
argumentos ``queryset`` -- é como a view sabe qual conjunto de objetos exibir
(veja "Selecionando objetos" no Capítulo 5 para uma introdução a objetos ``QuerySet``
e veja o Apêndice B para explicações detalhadas).

Para dar um exemplo simples, suponhamos que queremos ordenar uma lista de livros
de acordo com sua data de publicação -- com o mais recente primeiro:

.. parsed-literal::

    book_info = {
        'queryset': Book.objects.order_by('-publication_date'),
    }

    urlpatterns = patterns('',
        (r'^publishers/$', list_detail.object_list, publisher_info),
        **(r'^books/$', list_detail.object_list, book_info),**
    )

.. SL Tested ok

É um exemplo bem simples mas capaz de ilustrar a ideia de maneira clara. É bem
provável que você normalmente irá querer mais do que simplesmente reordenar
objetos. Se quiser exibir uma lista de livros de um determinado publicador, você
pode usar a mesma abordagem:

.. parsed-literal::

    **apress_books = {**
        **'queryset': Book.objects.filter(publisher__name='Apress Publishing'),**
        **'template_name': 'books/apress_list.html'**
    **}**

    urlpatterns = patterns('',
        (r'^publishers/$', list_detail.object_list, publisher_info),
        **(r'^books/apress/$', list_detail.object_list, apress_books),**
    )

.. SL Tested ok

Observe que, em conjunto com um ``queryset`` filtrado, estamos utilizando também
um nome de template diferente. Se não fizéssemos isso, a view genérica usaria o
mesmo template usado para a lista de objetos "baunilha" -- que certamente não é
o que queremos.

Observe também que essa não é uma maneira muito elegante de listar livros de
publicadores específicos. Se quiséssemos adicionar uma outra página de publicador,
iríamos precisar de mais um punhado de linhas no URLconf e tal abordagem seria
impraticável para mais que alguns poucos publicadores. Lidaremos com esse
problema na próxima seção.

Filtragem complexa com funções invólucros
-----------------------------------------

Outra necessidade bastante comum é filtrar os objetos dados em uma página de lista
utilizando algum parâmetro presenta na URL. Na seção anterior nós deixamos o nome
do publicador hard-coded no URLconf mas e se nós quiséssemos implementar uma view
que exibisse todos os livros de um publicador arbitrário? A solução é "encapsular"
a view genérica ``object_list`` para evitar escrever um monte de código à mão.
Como sempre, começamos por escrever um URLconf:

.. parsed-literal::

    urlpatterns = patterns('',
        (r'^publishers/$', list_detail.object_list, publisher_info),
        **(r'^books/(\\w+)/$', books_by_publisher),**
    )

Depois, escrevemos a view ``books_by_publisher`` propriamente dita::

    from django.shortcuts import get_object_or_404
    from django.views.generic import list_detail
    from mysite.books.models import Book, Publisher

    def books_by_publisher(request, name):

        # Look up the publisher (and raise a 404 if it can't be found).
        publisher = get_object_or_404(Publisher, name__iexact=name)

        # Use the object_list view for the heavy lifting.
        return list_detail.object_list(
            request,
            queryset = Book.objects.filter(publisher=publisher),
            template_name = 'books/books_by_publisher.html',
            template_object_name = 'book',
            extra_context = {'publisher': publisher}
        )

.. SL Tested ok

Isso funciona por que não há nada de especial nas views genéricas -- elas são
apenas funções Python. Como qualquer outra função de view, views genéricas
esperam um certo conjunto de argumentos e retornam objetos ``HttpResponse``.
Assim sendo, fica muito fácil encapsular uma view genérica por uma pequena função
que realiza trabalho adicional antes (ou depois; veja a próxima seção) manipulando
coisas que não estão na view genérica.

.. note::

    Observe que no exemplo anterior nós passamos o publicador sendo exibido
    atualmente na ``extra_context``. Isso é geralmente uma boa ideia em invólucros
    dessa natureza pois permite que o template saiba qual objeto "pai" está sendo
    processado.

Realizando trabalho extra
-------------------------

O último padrão comum que iremos averiguar envolve a realização de trabalho extra
antes ou depois de chamar uma view genérica.

Imagine que tivéssemos um campo ``last_accessed`` no objeto ``Author`` que estamos
usando para manter a informação da última vez em que a página do autor foi acessada.
A view genérica ``object_detail`` certamente não saberia nada acerca desse campo
mas, mais uma vez, nós podemos facilmente escrever uma view customizada para
manter esse campo atualizado.

A priori, iríamos precisar de adicionar o parâmetro de detalhe do autor no URLconf
para apontar para uma view customizada:

.. parsed-literal::

    from mysite.books.views import author_detail

    urlpatterns = patterns('',
        # ...
        **(r'^authors/(?P<author_id>\\d+)/$', author_detail),**
        # ...
    )

Só então escreveríamos nossa função invólucro::

    import datetime
    from django.shortcuts import get_object_or_404
    from django.views.generic import list_detail
    from mysite.books.models import Author

    def author_detail(request, author_id):
        # Delegate to the generic view and get an HttpResponse.
        response = list_detail.object_detail(
            request,
            queryset = Author.objects.all(),
            object_id = author_id,
        )

        # Record the last accessed date. We do this *after* the call
        # to object_detail(), not before it, so that this won't be called
        # unless the Author actually exists. (If the author doesn't exist,
        # object_detail() will raise Http404, and we won't reach this point.)
        now = datetime.datetime.now()
        Author.objects.filter(id=author_id).update(last_accessed=now)

        return response

.. note::

    Esse código não irá funcionar a menos que você adicione o campo
    ``last_accessed`` ao modelo ``Author`` e crie um template
    ``books/author_detail.html``.

.. SL Tested ok

Podemos utilizar uma abordagem semelhante para alterar a resposta retornada pela
view genérica. Se quiséssemos fornecer uma versão em texto simples da lista de
autores que pode ser baixada, poderíamos utilizar uma view como a seguinte::

    def author_list_plaintext(request):
        response = list_detail.object_list(
            request,
            queryset = Author.objects.all(),
            mimetype = 'text/plain',
            template_name = 'books/author_list.txt'
        )
        response["Content-Disposition"] = "attachment; filename=authors.txt"
        return response

.. SL Tested ok

Isso funciona por que a view genérica retorna objetos ``HttpResponse`` simples
que podem ser tratados como dicionários para setar campos HTTP. Esse
"Content-Disposition", por sinal, informa ao browser que faça o download e salve
a página ao invés de renderizá-la no browser.

O que vem a seguir?
===================

Neste capítulo abordamos somente algumas das views genéricas que Django provê
mas as ideias gerais apresentadas aqui devem ser aplicáveis -- guardadas as
devidas diferenças entre os seus tipos -- a qualquer view genérica. O Apêndice C
cobre todas as views disponíveis em detalhes e sua leitura é recomendável se você
quer aproveitar ao máximo essa poderosa funcionalidade.

Isso conclui a seção deste livro dedicada à "utilização avançada". No `next chapter`_,
cobriremos o deployment de aplicações Django.

.. _next chapter: ../chapter12/
