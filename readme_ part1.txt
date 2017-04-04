
1.安装Django:
https://docs.djangoproject.com/en/1.10/topics/install/
https://docs.djangoproject.com/en/1.10/howto/windows/

2.编写您的第一个Django应用程序
    1.第1部分 :https://docs.djangoproject.com/en/1.10/intro/tutorial01/
      创建项目:
         django-admin startproject mysite

         让我们看看startproject创建的内容：
         mysite/
            manage.py
            mysite/
            __init__.py
            settings.py
            urls.py
            wsgi.py

        这些文件是：

        外部mysite/根目录只是一个项目的容器。它的名字与Django无关; 您可以将其重命名为您喜欢的任何内容。
        manage.py：一个命令行实用程序，可以让您以各种方式与此Django项目进行交互。您可以阅读django-admin和manage.pymanage.py中的所有详细信息 。
        内部mysite/目录是项目的实际Python包。它的名字是您需要用来导入其中的任何内容的Python包名称（例如mysite.urls）。
        mysite/__init__.py：一个空的文件，告诉Python这个目录应该被认为是一个Python包。如果您是Python初学者，请阅读官方Python文档中有关软件包的更多信息。
        mysite/settings.py：此Django项目的设置/配置。 Django设置会告诉你所有关于设置的工作原理。
        mysite/urls.py：该Django项目的URL声明; 您的Django动力网站的“目录”。您可以在URL调度程序中阅读有关URL的更多信息。
        mysite/wsgi.py：WSGI兼容的Web服务器为您的项目提供服务的入口点。有关详细信息，请参阅如何使用WSGI进行部署。

    2.启动服务器:
        python manage.py runserver
    您将在命令行中看到以下输出：
        Performing system checks...

        System check identified no issues (0 silenced).

        You have unapplied migrations; your app may not work properly until they are applied.
        Run 'python manage.py migrate' to apply them.

        March 31, 2017 - 15:50:53
        Django version 1.10, using settings 'mysite.settings'
        Starting development server at http://127.0.0.1:8000/
        Quit the server with CONTROL-C.

        您已经开始使用Django开发服务器，这是一个纯粹以Python编写的轻量级Web服务器。我们将其与Django结合在一起，因此您可以快速开发，而无需处理配置生产服务器（如Apache），直到您准备好生产。
    现在是一个很好的时间来注意：不要在任何类似于生产环境的任何地方使用这个服务器。它只适用于开发。（我们正在开发Web框架，而不是Web服务器。）
    现在服务器正在运行，请使用Web浏览器访问http://127.0.0.1:8000/。您会看到一个“欢迎来到Django”页面，在愉快的，浅蓝色的粉彩中。有效！

    改变端口:
        默认情况下，该runserver命令在端口8000的内部IP上启动开发服务器。
        如果要更改服务器的端口，请将其作为命令行参数传递。例如，此命令启动端口8080上的服务器：
        python manage.py runserver 8080

        如果要更改服务器的IP，请将其与端口一起传递。所以要听所有的公共IP（有用的，如果你想炫耀你的工作在网络上的其他电脑上），使用：
        python manage.py runserver 0.0.0.0:8000
        开发服务器的完整文档可以在参考文献中找到 runserver。https://docs.djangoproject.com/en/1.10/ref/django-admin/#django-admin-runserver

    3.创建投票应用程序
        现在，您的环境 - 一个“项目” - 已经建立起来，您将开始做工作。

        您在Django中编写的每个应用程序都包含遵循一定约定的Python包。Django自带一个实用程序，可以自动生成应用程序的基本目录结构，因此您可以专注于编写代码而不是创建目录。

        您的应用程序可以生活在Python路径的任何位置。在本教程中，我们将在您的manage.py 文件旁边创建我们的投票应用程序，以便它可以作为自己的顶级模块导入，而不是子模块mysite。

        要创建您的应用程序，请确保您与目录位于同一目录，manage.py 并键入以下命令：

        python manage.py startapp polls

        将创建一个目录polls，其目录如下：
            polls/
                __init__.py
                admin.py
                apps.py
                migrations/
                    __init__.py
                models.py
                tests.py
                views.py

        写下您的第一个视图
        我们来写第一个View。打开文件polls/views.py 并放入以下Python代码：

        from django.http import HttpResponse

        def index(request):
            return HttpResponse("Hello, world. You're at the polls index.")

        这是Django中最简单的视图。要调用视图，我们需要将其映射到一个URL - 为此，我们需要一个URLconf。
        要在polls目录中创建一个URLconf，创建一个名为urls.py。您的应用目录应该如下所示：

        在polls/urls.py文件中包含以下代码：
        from django.conf.urls import url
        from . import views

        urlpatterns = [
            url(r'^$', views.index, name='index'),
        ]

      下一步是将根URLconf指向polls.urls模块。在 mysite/urls.py添加一条import用于django.conf.urls.include和插入include()的urlpatterns列表，所以你必须：

        from django.conf.urls import include, url
        from django.contrib import admin

        urlpatterns = [
            url(r'^polls/', include('polls.urls')),
            url(r'^admin/', admin.site.urls),
        ]

        该include()函数允许引用其他URLconfs。请注意，该include()函数的正则表达式 没有$（字符串匹配字符），而是尾部的斜杠。每当Django遇到 include()时，它会排除与该点匹配的任何部分，并将剩余的字符串发送到随附的URLconf进行进一步处理。

        背后的想法include()是使即插即用的URL变得容易。由于民意调查是在自己的URLconf（polls/urls.py）中，它们可以被放置在“/ polls /”下面，或者在“/ fun_polls /”下面，或者在“/ content / polls /”或其他路径根目录下，工作。

    您现在已将index视图连接到URLconf中。让它验证它的工作，运行以下命令：

    python manage.py runserver

    在浏览器中转到http：// localhost：8000 / polls /，您应该看到文本“ Hello，world”。你在投票指数。“，您在index视图中定义 。

    该url()函数传递四个参数，两个必需：regex和view，和两个可选：kwargs，和name。在这一点上，值得审查这些论据。

    url()参数：正则表达式
        术语“正则表达式”是一种常用的短格式，意思是“正则表达式”，它是用于匹配字符串中的模式的语法，或者在这种情况下是url模式。Django从第一个正则表达式开始，并将其放在列表中，将请求的URL与每个正则表达式进行比较，直到找到匹配的一个。

        请注意，这些正则表达式不搜索GET和POST参数或域名。例如，在请求中 https://www.example.com/myapp/，URLconf将寻找myapp/。在请求中https://www.example.com/myapp/?page=3，URLconf也会查找myapp/。

        如果您需要正则表达式的帮助，请参阅维基百科的条目和re模块的文档。此外，Jeffrey Friedl的O'Reilly书“掌握正则表达式”是非常棒的。然而，实际上，您不需要是正则表达式的专家，因为您只需要知道如何捕获简单的模式。实际上，复杂的正则表达式的查找性能会很差，所以你可能不应该依靠正则表达式的全部功能。

        最后，一个性能说明：这些正则表达式是第一次加载URLconf模块时被编译。它们超级快（只要查找不是太复杂，如上所述）。

    url()参数:view

        当Django找到一个正则表达式匹配时，Django会调用指定的视图函数，其中一个HttpRequest对象作为第一个参数，任何来自正则表达式的“捕获”值作为其他参数。如果正则表达式使用简单的捕获，则值作为位置参数传递; 如果它使用命名捕获，则值作为关键字参数传递。我们稍后会给出一个例子。

    url()参数：kwargs

        任意关键词参数可以在字典中传递到目标视图。我们不会在教程中使用Django的这个功能。

    url()参数：name

        命名您的URL可让您从Django其他地方明确地引用它，特别是在模板中。这个强大的功能可让您全面更改项目的URL模式，同时只触摸单个文件。

        当您对基本请求和响应流程感到满意时，请阅读 本教程的第2部分，开始使用数据库。

        https://docs.djangoproject.com/en/1.10/intro/tutorial02/