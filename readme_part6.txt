

6.编写你的第一个Django应用，部分6
   第6部分 :https://docs.djangoproject.com/en/1.11/intro/tutorial06/

        本教程从教程5中断开始。我们构建了一个经过测试的Web-poll应用程序，现在我们将添加一个样式表和一个图像。

        除了服务器生成的HTML之外，Web应用程序通常需要提供渲染完整网页所需的附加文件（如图像，JavaScript或CSS）。在Django中，我们将这些文件称为“静态文件”。

        对于小型项目来说，这并不是一件大事，因为您可以将静态文件保存在Web服务器可以找到的地方。然而，在更大的项目中，特别是由多个应用程序组成的项目，处理每个应用程序提供的多组静态文件开始变得棘手。

        这django.contrib.staticfiles就是：它将每个应用程序（以及您指定的任何其他地方）的静态文件收集到单个位置，以便在生产中轻松投放。


   自定义您的应用程序的外观和感觉
        首先，在目录static中创建一个polls目录。Django会在那里寻找静态文件，类似于Django在里面找到模板polls/templates/。

        Django的STATICFILES_FINDERS设置包含一组查找器，它们可以从各种来源中发现静态文件。
        其中一个默认值是AppDirectoriesFinder在每个中寻找一个“static”子目录 INSTALLED_APPS，就像polls我们刚刚创建的一样。
        管理站点的静态文件使用相同的目录结构。

        在static刚创建的目录中，创建另一个目录，polls并创建一个名为的文件style.css。换句话说，您的样式表应该在polls/static/polls/style.css。
        由于AppDirectoriesFinderstaticfile finder的工作原理，您可以将Django中的这个静态文件简单地引用为polls/style.css类似于引用模板路径的方式。

              静态文件命名空间
                就像模板一样，我们可以直接将静态文件放在polls/static（而不是创建另一个polls 子目录）中，
                但实际上是一个坏主意。Django会选择它找到其名称匹配的第一个静态文件，
                如果您有相关的同名静态文件不同的应用程序，Django的将无法区分它们。我们需要能够将Django指向正确的方法
                ，最简单的方法是通过命名来确定这一点。也就是说，将这些静态文件放在为应用程序本身命名的另一个目录中。

        将以下代码放在该样式表（polls/static/polls/style.css）中：

        polls/static/polls/style.css

            li a {
                color: green;
            }

        接下来，添加以下内容polls/templates/polls/index.html：

        polls/templates/polls/index.html

            {% load static %}

            <link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}" />

        该模板标签生成静态文件的绝对路径。{% static %}

        这就是您需要做的开发工作。重新加载 http://localhost:8000/polls/，你应该看到问题链接是绿色（Django样式！），这意味着你的样式表被正确加载。

   添加背景图像

        接下来，我们将创建一个图像的子目录。在images目录中创建一个polls/static/polls/子目录。在这个目录下面，放一个图片叫做background.gif。换句话说，把你的形象 polls/static/polls/images/background.gif。

        然后，添加到您的stylesheet（polls/static/polls/style.css）：

        polls/static/polls/style.css
            body {
                background: white url("images/background.gif") no-repeat right bottom;
            }

         重新加载http://localhost:8000/polls/，你应该看到背景加载在屏幕的右下方。

            警告
                当然，模板标签不可用于静态文件，如Django不能生成的样式表。您应该始终使用相对路径将静态文件相互链接，因为您可以更改（由 模板标签用于生成其URL），而无需修改静态文件中的一组路径。{% static %}STATIC_URLstatic



          这些是基础知识。有关框架中包含的设置和其他位的更多详细信息，请参阅 静态文件howto和 staticfiles参考。
          https://docs.djangoproject.com/en/1.11/howto/static-files/
          https://docs.djangoproject.com/en/1.11/ref/contrib/staticfiles/
          部署静态文件讨论如何在真实服务器上使用静态文件。
          https://docs.djangoproject.com/en/1.11/howto/static-files/deployment/

          当您对静态文件感到满意时，请阅读本教程的第7部分，了解如何自定义Django自动生成的管理站点。
          https://docs.djangoproject.com/en/1.11/intro/tutorial07/



