


3.编写您的第一个Django应用程序-3
    第3部分 :https://docs.djangoproject.com/en/1.11/intro/tutorial03/

    本教程从教程2中断开始。我们正在继续Web-poll应用程序，并将重点创建公共接口 - “视图”。

    1.概述
        视图是Django应用程序中的“类型”网页，通常用于特定功能并具有特定的模板。例如，在博客应用程序中，您可能有以下视图：

        博客首页 - 显示最新的几个条目。
        条目“详细”页面 - 单个条目的固定链接页面。
        基于年份的存档页面 - 显示给定年份的所有月份。
        基于月份的存档页面 - 显示所有日期与给定月份的条目。
        基于日的存档页面 - 显示给定日期中的所有条目。
        评论动作 - 处理向给定条目发布注释。
        在我们的投票申请中，我们将有以下四个视图：

        问题“索引”页面 - 显示最新的几个问题。
        问题“详细”页面 - 显示一个问题文本，没有结果，但表单投票。
        问题“结果”页面显示特定问题的结果。
        投票行动 - 处理特定问题中特定选择的投票。
        在Django中，网页和其他内容由视图提供。每个视图都由一个简单的Python函数（或者基于类视图的方法）来表示。Django将通过检查所请求的URL（确切地说，域名后的URL部分）来选择一个视图。

        现在在你的网络时代，你可能会碰到像“ME2 / Sites / dirmod.asp？sid =＆type = gen＆mod = Core + Pages＆gid = A6CD4967199A42D9B65B1B”这样的美女。您会很高兴知道，Django允许我们提供比这更优雅的 网址格式。

        网址格式只是URL的一般形式，例如： /newsarchive/<year>/<month>/。

        要从URL到视图，Django使用所谓的“URLconfs”。URLconf将URL模式（描述为正则表达式）映射到视图。

    2.写更多视图¶

        现在我们再添加一些视图polls/views.py。这些观点略有不同，因为他们提出了一个观点：

        polls / views.py

        def detail(request, question_id):
            return HttpResponse("You're looking at question %s." % question_id)

        def results(request, question_id):
            response = "You're looking at the results of question %s."
            return HttpResponse(response % question_id)

        def vote(request, question_id):
            return HttpResponse("You're voting on question %s." % question_id)

        polls.urls通过添加以下url()调用将这些新视图连接到模块中 ：

        polls/urls.py

        from django.conf.urls import url

        from . import views

        urlpatterns = [
            # ex: /polls/
            url(r'^$', views.index, name='index'),
            # ex: /polls/5/
            url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
            # ex: /polls/5/results/
            url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
            # ex: /polls/5/vote/
            url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
        ]


       在浏览器中查看“/ polls / 34 /”。它将运行该detail() 方法并显示您在URL中提供的任何ID。尝试“/ polls / 34 / results /”和“/ polls / 34 / vote /”，这些将显示占位符结果和投票页面。
        当有人从您的网站请求一个页面 - 例如“/ polls / 34 /”时，Django将加载mysite.urlsPython模块，因为它被ROOT_URLCONF设置指向 。它找到变量urlpatterns ，并按顺序遍历正则表达式。找到匹配后 '^polls/'，它将剥离匹配的文本（"polls/"），并将剩余的文本发送"34/"到“polls.urls”URLconf进行进一步处理。它匹配r'^(?P<question_id>[0-9]+)/$'，导致对detail()视图的调用如下所示：

        detail(request=<HttpRequest object>, question_id='34')

        该question_id='34'部分来自(?P<question_id>[0-9]+)。使用模式周围的括号“捕获”该模式匹配的文本，并将其作为参数发送给视图函数; ?P<question_id>定义将用于识别匹配模式的名称; 并且[0-9]+是用于匹配数字序列（即数字）的正则表达式。
        由于网址格式是正则表达式，因此您可以对它们进行任何限制。而且没有必要添加URL cruft，如 .html- 除非你想，在这种情况下你可以这样做：

        url(r'^polls/latest\.html$', views.index),

        但是，不要这样做。太傻了


    3.写意见，实际上做一些事情¶

        每个视图负责执行以下两项操作之一：返回一个HttpResponse包含所请求页面的内容的 对象，或者引发诸如此类的异常Http404。其余的取决于你。

        您的视图可以从数据库读取记录。它可以使用诸如Django的模板系统，也可以使用第三方Python模板系统。它可以生成PDF文件，输出XML，立即创建一个ZIP文件，任何你想要的，使用任何你想要的Python库。

        所有Django都是这样想的HttpResponse。还是一个例外。

        因为它很方便，我们使用Django自己的数据库API，我们在教程2中介绍。这是一个新的index() 视图，它显示系统中最新的5个投票问题，以逗号分隔，根据发布日期：

        polls/views.py

        from django.http import HttpResponse
        from .models import Question

        def index(request):
            latest_question_list = Question.objects.order_by('-pub_date')[:5]
            output = ', '.join([q.question_text for q in latest_question_list])
            return HttpResponse(output)

        # Leave the rest of the views (detail, results, vote) unchanged

        这里有一个问题：页面的设计在视图中是硬编码的。如果要更改页面的样式，则必须编辑此Python代码。所以让我们使用Django的模板系统通过创建视图可以使用的模板来将设计与Python分开。
        首先，在目录templates中创建一个polls目录。Django会在那里寻找模板。
        您的项目的TEMPLATES设置描述Django如何加载和呈现模板。默认设置文件配置DjangoTemplates 其APP_DIRS选项设置为 的后端True。按照惯例DjangoTemplates在每个的“template”子目录中查找INSTALLED_APPS。
        在templates刚创建的目录中，创建另一个目录polls，并创建一个名为的文件 index.html。换句话说，你的模板应该在 polls/templates/polls/index.html。由于app_directories 模板加载器的工作原理如上所述，您可以将Django中的这个模板简单地称为polls/index.html。

        模板命名空间

        现在我们可以直接将我们的模板放在 polls/templates（而不是创建另一个polls子目录）中，但实际上是一个坏主意。Django会选择它找到其名称匹配的第一个模板，如果您有一个同名的模板不同的应用程序，Django的将无法区分它们。我们需要能够将Django指向正确的方法，最简单的方法是通过命名来确定这一点。也就是说，将这些模板放在为应用程序本身命名的另一个目录中。

        将以下代码放在该模板中：

        polls/templates/polls/index.html

        {% if latest_question_list %}
            <ul>
            {% for question in latest_question_list %}
                <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
            {% endfor %}
            </ul>
        {% else %}
            <p>No polls are available.</p>
        {% endif %}

        现在让我们更新我们的index视图polls/views.py来使用模板：

        polls/views.py

        from django.http import HttpResponse
        from django.template import loader

        from .models import Question

        def index(request):
            latest_question_list = Question.objects.order_by('-pub_date')[:5]
            template = loader.get_template('polls/index.html')
            context = {
                'latest_question_list': latest_question_list,
            }
            return HttpResponse(template.render(context, request))

        该代码加载调用的模板， polls/index.html并传递一个上下文。上下文是将模板变量名称映射到Python对象的字典。
        通过将浏览器指向“/ polls /”来加载页面，您应该看到包含教程2中 “What's up”问题的项目符号列表。链接指向问题的详细页面。

      4.一个捷径:render()

        加载模板是一个非常常见的成语，填充上下文并返回一个HttpResponse具有渲染模板结果的 对象。Django提供了一个快捷方式。这是完整的index()视图，重写：
        polls/views.py

        from django.shortcuts import render

        from .models import Question
        def index(request):
            latest_question_list = Question.objects.order_by('-pub_date')[:5]
            context = {'latest_question_list': latest_question_list}
            return render(request, 'polls/index.html', context)

         请注意，一旦我们在所有这些观点做到了这一点，我们不再需要进口 loader和HttpResponse（你想保留HttpResponse，如果你仍然有存根方法detail， results和vote）。
         该render()函数将请求对象作为其第一个参数，模板名称作为其第二个参数，并将字典作为其可选的第三个参数。它返回HttpResponse 给定上下文渲染的给定模板的对象。

     5.提高404错误
     现在，我们来解决问题详细视图 - 显示给定投票的问题文本的页面。这里的看法：
     polls/views.py

        from django.http import Http404
        from django.shortcuts import render

        from .models import Question
        # ...
        def detail(request, question_id):
            try:
                question = Question.objects.get(pk=question_id)
            except Question.DoesNotExist:
                raise Http404("Question does not exist")
            return render(request, 'polls/detail.html', {'question': question})

        这里的新概念：Http404如果具有请求的ID的问题不存在，则该视图引发异常。
        我们稍后会讨论你可以放在哪个polls/detail.html模板中，但是如果你想快速得到上面的例子，
        一个文件只包含：
        polls / templates / polls / detail.html

        {{  question  }}


        会让你开始现在


       6.一个捷径：get_object_or_404()

        这是一个非常常见的成语使用get() 和提高Http404，如果对象不存在。Django提供了一个快捷方式。这是detail()视图，重写：

        polls / views.py

        from django.shortcuts import get_object_or_404, render

        from .models import Question
        # ...
        def detail(request, question_id):
            question = get_object_or_404(Question, pk=question_id)
            return render(request, 'polls/detail.html', {'question': question})

        该get_object_or_404()函数将Django模型作为其第一个参数和任意数量的关键字参数，它将传递给get()模型管理器的函数。Http404如果对象不存在，它会引发。

                哲学
                        为什么我们使用帮助函数，get_object_or_404() 而不是自动捕获ObjectDoesNotExist更高级别的 异常，或者使用模型API Http404而不是 加注ObjectDoesNotExist？

                        因为这会将模型层耦合到视图层。Django最重要的设计目标之一是保持松耦合。django.shortcuts模块中引入了一些受控耦合。

        还有一个get_list_or_404()功能，它的作用就是get_object_or_404()- 除了使用 filter()而不是 get()。Http404如果列表为空，它会引发 。

     7.使用模板系统
        回到detail()我们的投票应用程序的视图。给定上下文变量question，以下是polls/detail.html模板的外观：

        polls/templates/polls/detail.html

        <h1>{{ question.question_text }}</h1>
            <ul>
            {% for choice in question.choice_set.all %}
                <li>{{ choice.choice_text }}</li>
            {% endfor %}
            </ul>
        模板系统使用点查找语法访问变量属性。在例子中，首先Django对对象进行字典查找。否则，它会尝试一个属性查找 - 在这种情况下可以工作。如果属性查找失败，它将尝试列表索引查找。{{ question.question_text }}question

        在循环中发生方法调用： 被解释为Python代码 ，它返回一个可迭代的对象，适合在标签中使用。{% for %}question.choice_set.allquestion.choice_set.all()Choice{% for %}

        有关模板的更多信息，请参阅模板指南。https://docs.djangoproject.com/en/1.11/topics/templates/

     8.删除硬编码的网址模板
            请记住，当我们在polls/index.html 模板中写入一个问题的链接时，链接部分硬编码如下：

            <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>

            这种硬编码，紧密耦合的方法的问题是，在具有大量模板的项目上更改URL变得具有挑战性。但是，由于您url()在polls.urls模块中的函数中定义了name参数，因此可以通过使用模板标签来删除对url配置中定义的特定URL路径的依赖：{% url %}

            <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>

            这种方式是通过查看polls.urls模块中指定的URL定义 。您可以看到下面定义了“detail”的URL名称的位置：
            ...
            # the 'name' value as called by the {% url %} template tag
            url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
            ...

            如果你想改变投票细节视图别的东西的URL，也许是类似polls/specifics/12/的，而不是在模板中做（或模板），你会改变它polls/urls.py：

            ...
            # added the word 'specifics'
            url(r'^specifics/(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
            ...

     9.命名空间URL名称
        教程项目只有一个应用程序polls。在真正的Django项目中，可能有五，十，二十个应用程序或更多。Django如何区分它们之间的URL名称？例如，该polls应用程序具有一个detail 视图，因此也可能是与该博客相同的项目的应用程序。Django如何知道在使用模板标签时为url创建的应用视图 ？{% url %}
        答案是为您的URLconf添加命名空间。在polls/urls.py 文件中，继续添加一个app_name设置应用程序命名空间：

        polls/urls.py

        from django.conf.urls import url
        from . import views
        app_name = 'polls'
        urlpatterns = [
            url(r'^$', views.index, name='index'),
            url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
            url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
            url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
        ]

        现在更改您的polls/index.html模板：

        polls/templates/polls/index.html

        <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>

        指向命名空间细节视图：

        polls/templates/polls/index.html

        <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>

        阅读本教程的第4部分，了解简单的表单处理和通用视图。
        https://docs.djangoproject.com/en/1.11/intro/tutorial04/

