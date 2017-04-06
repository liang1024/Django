
7.编写你的第一个Django应用，部分7
   第7部分 :https://docs.djangoproject.com/en/1.11/intro/tutorial07/

   本教程从教程6中断开始。我们正在继续Web-poll应用程序，并将专注于自定义Django自动生成的管理网站，我们首先在教程2中探讨。

   自定义管理窗体
        通过注册Question模型admin.site.register(Question)，Django能够构建一个默认的表单。通常，您需要自定义管理表单的外观和工作原理。你可以通过告诉Django注册对象时所需的选项来做到这一点。

        让我们看看如何通过重新排序编辑表单上的字段来实现。用以下替换admin.site.register(Question)行：

        polls/admin.py

            from django.contrib import admin

            from .models import Question


            class QuestionAdmin(admin.ModelAdmin):
                fields = ['pub_date', 'question_text']

            admin.site.register(Question, QuestionAdmin)

         您将遵循此模式 - 创建模型管理类，然后将其作为第二个参数传递给admin.site.register()任何时候您需要更改模型的管理选项。

        上述这个特别的变化使“出版日期”出现在“问题”字段之前：



        这不仅仅是两个字段令人印象深刻，而对于具有数十个字段的管理表单，选择直观的顺序是一个重要的可用性细节。

        说到几十个字段的表单，你可能想把这个表单分成几个字段：

        polls/admin.py

            from django.contrib import admin

            from .models import Question


            class QuestionAdmin(admin.ModelAdmin):
                fieldsets = [
                    (None,               {'fields': ['question_text']}),
                    ('Date information', {'fields': ['pub_date']}),
                ]

            admin.site.register(Question, QuestionAdmin)


         每个元组的第一个元素 fieldsets是字段集的标题。这是我们现在的形式：

        ...


   添加相关对象

           好的，我们有我们的问题管理页面，但一个Question有多个 Choices，管理页面不显示选择。

        然而。

        有两种方法来解决这个问题。第一个是Choice 像管理员一样向管理员注册Question。这很容易：

        polls/admin.py

            from django.contrib import admin

            from .models import Choice, Question
            # ...
            admin.site.register(Choice)

        现在，“选择”是Django管理员中可用的选项。“添加选择”表单如下所示：

        在该表单中，“问题”字段是包含数据库中每个问题的选择框。Django知道a ForeignKey应该在管理员中表示为一个<select>框。在这种情况下，现在只存在一个问题。

        另请注意“问题”旁边的“添加另一个”链接。与ForeignKey另一个关系的每个对象 都可以免费获取。当您点击“添加另一个”时，您将看到一个带有“添加问题”窗体的弹出窗口。如果您在该窗口中添加了一个问题并单击“保存”，Django会将问题保存到数据库中，并将其作为选定的选项动态添加到您正在查看的“添加选择”窗体中。

        但是，真的，这是Choice向系统添加对象的低效方法。如果您在创建Question对象时可以直接添加一堆选择，则会更好 。让我们这样做。

        删除模型的register()呼叫Choice。然后，将Question 注册码编辑为：

        polls/admin.py

            from django.contrib import admin

            from .models import Choice, Question


            class ChoiceInline(admin.StackedInline):
                model = Choice
                extra = 3


            class QuestionAdmin(admin.ModelAdmin):
                fieldsets = [
                    (None,               {'fields': ['question_text']}),
                    ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
                ]
                inlines = [ChoiceInline]

            admin.site.register(Question, QuestionAdmin)


        这告诉Django：“ Choice对象在Question管理页面上被编辑。默认情况下，为3个选项提供足够的字段。

        加载“添加问题”页面，看看它的外观：

        它的工作原理如下：有三个插槽用于相关的选择 - 由以下指定extra- 每当您返回到已经创建的对象的“更改”页面时，您将再次获得三个插槽。

        在三个当前插槽的最后，您将找到一个“添加另一个选择”链接。如果您点击它，将会添加一个新插槽。如果要删除添加的插槽，可以单击添加插槽右上方的X。请注意，您不能删除原来的三个插槽。此图片显示添加的插槽：


        一个小问题，但。显示用于输入相关Choice对象的所有字段需要大量的屏幕空间。为此，Django提供了一种显示内联相关对象的表格方式; 您只需要将ChoiceInline声明更改为：

        polls/admin.py

            class ChoiceInline(admin.TabularInline):
            #...

        使用TabularInline（而不是StackedInline），相关对象以更紧凑的基于表格的格式显示：

        请注意，有一个额外的“删除？”列允许删除使用“添加另一选择”按钮添加的行和已保存的行。

   自定义管理更改列表

        现在，问题管理页面看起来不错，让我们对“更改列表”页面进行一些调整 - 显示系统中所有问题的页面。

        这是现在的样子：


        默认情况下，Django显示str()每个对象。但有时如果我们可以显示个别的字段会更有帮助。要做到这一点，使用 list_displayadmin选项，这是一个字段名称的元组，作为列显示在对象的更改列表页面上：

        polls/admin.py
            class QuestionAdmin(admin.ModelAdmin):
            # ...
            list_display = ('question_text', 'pub_date')

         只是为了很好的措施，我们还包括教程2的was_published_recently() 方法：

         polls/admin.py
            class QuestionAdmin(admin.ModelAdmin):
            # ...
            list_display = ('question_text', 'pub_date', 'was_published_recently')

         现在问题更改列表页面如下所示：


         您可以单击列标题以was_published_recently排除这些值 - 除了在标题的情况下，因为不支持按任意方法的输出进行排序。另请注意，was_published_recently默认情况下，列标题 是方法的名称（下划线替换为空格），每行包含输出的字符串表示形式。

        您可以通过将该方法（in polls/models.py）的几个属性进行改进，如下所示：

        polls / models.py

            class Question(models.Model):
            # ...
            def was_published_recently(self):
                now = timezone.now()
                return now - datetime.timedelta(days=1) <= self.pub_date <= now
            was_published_recently.admin_order_field = 'pub_date'
            was_published_recently.boolean = True
            was_published_recently.short_description = 'Published recently?'


        有关这些方法属性的更多信息，请参见 list_display。

        polls/admin.py再次修改您的文件，并在Question更改列表页面中添加改进 ：使用过滤器 list_filter。将以下行添加到 QuestionAdmin：

        list_filter = ['pub_date']

        它增加了一个“过滤器”侧边栏，让人们通过pub_date字段过滤更改列表 ：


        显示的过滤器类型取决于您要过滤的字段类型。因为pub_date是DateTimeField，Django知道提供适当的过滤器选项：“任何日期”，“今天”，“过去7天”，“本月”，“今年”。

        这正在形成良好。我们添加一些搜索功能：


        search_fields = ['question_text']

        这会在更改列表的顶部添加一个搜索框。当有人输入搜索字词时，Django会搜索该question_text字段。您可以使用尽可能多的字段 - 尽管因为它LIKE在幕后使用查询，将搜索字段的数量限制为合理的数字将使您的数据库更容易进行搜索。

        现在也是一个很好的时间，注意到更改列表给你免费的分页。默认值是每页显示100个项目。，，，，和 所有的工作在一起，就像你认为他们应该。Change list paginationsearch boxesfiltersdate-hierarchiescolumn-header-ordering

    自定义管理的外观和感觉

        显然，每个管理页面顶部的“Django管理”都是荒谬的。这只是占位符文本。

        这很容易改变，但是，使用Django的模板系统。Django管理员由Django本身提供支持，其界面使用Django自己的模板系统。

    自定义项目的模板

        templates在项目目录中创建一个目录（包含该目录manage.py）。模板可以在Django可以访问的文件系统的任何地方。（Django作为您的服务器运行的任何用户运行。）但是，将模板保留在项目中是一个很好的惯例。

        打开设置文件（mysite/settings.py记住）并DIRS在TEMPLATES设置中添加一个 选项：

        mysite/settings.py

            TEMPLATES = [
                {
                    'BACKEND': 'django.template.backends.django.DjangoTemplates',
                    'DIRS': [os.path.join(BASE_DIR, 'templates')],
                    'APP_DIRS': True,
                    'OPTIONS': {
                        'context_processors': [
                            'django.template.context_processors.debug',
                            'django.template.context_processors.request',
                            'django.contrib.auth.context_processors.auth',
                            'django.contrib.messages.context_processors.messages',
                        ],
                    },
                },
            ]

        DIRS是加载Django模板时要检查的文件系统目录的列表; 这是一个搜索路径。

            组织模板
                就像静态文件一样，我们可以将所有的模板放在一个大的模板目录中，并且工作得很好。
                但是，属于特定应用程序的模板应放置在该应用程序的模板目录（例如polls/templates）而不是项目的（templates）中。
                我们将在可重用的应用程序教程 中更详细地讨论 我们为什么这样做。
                https://docs.djangoproject.com/en/1.11/intro/reusable-apps/

        现在创建一个名为admin里面的目录templates，并将admin/base_site.htmlDjango本身（django/contrib/admin/templates）的源代码中的默认Django管理模板目录中的模板复制到该目录中。

        Django源文件在哪里？

        如果您难以找到Django源文件位于系统上的位置，请运行以下命令：

            $ python -c "import django; print(django.__path__)"

         然后，只需编辑文件并用您自己的网站名称替换 （包括花括号）。你应该最终得到一段代码，如：{{ site_header|default:_('Django administration') }}

        {% block branding %}
        <h1 id="site-name"><a href="{% url 'admin:index' %}">Polls Administration</a></h1>
        {% endblock %}

        我们使用这种方法教你如何覆盖模板。在实际的项目中，您可能会使用该django.contrib.admin.AdminSite.site_header属性更容易地进行这种特定的定制。

        这个模板文件包含大量文字一样的 和。在和标签是Django的模板语言的一部分。Django渲染时，将会对该模板语言进行评估，以生成最终的HTML页面，就像我们在教程3中看到的那样。{% block branding %}{{ title }}{%{{admin/base_site.html

        请注意，任何Django的默认管理模板都可以被覆盖。要覆盖模板，只需执行与之相同的操作base_site.html- 将其从默认目录复制到自定义目录中，然后进行更改。

    自定义应用程序的模板

        精明的读者会问：如果DIRS默认为空，Django如何找到默认的管理模板？答案是，由于APP_DIRS设置为True，Django会templates/自动在每个应用程序包中查找一个子目录，以作为后备（不要忘记这 django.contrib.admin是一个应用程序）。

        我们的轮询应用程序不是很复杂，不需要自定义管理模板。但是如果Django的标准管理模板变得越来越复杂，需要修改其某些功能的标准管理模板，那么修改应用程序的模板而不是项目中的模板 会更加明智。这样，您可以将poll应用程序包含在任何新项目中，并放心，它会找到所需的自定义模板。

        有关Django如何查找其模板的更多信息，请参阅模板加载文档。
        https://docs.djangoproject.com/en/1.11/topics/templates/#template-loading

    定制管理索引页面
        在类似的注释中，您可能需要自定义Django管理员索引页面的外观。

        默认情况下，它会INSTALLED_APPS按照字母顺序显示已在管理应用程序中注册的所有应用程序。您可能需要对布局进行重大更改。毕竟，索引可能是管理员最重要的页面，它应该很容易使用。

        要定制的模板是admin/index.html。（与admin/base_site.html上一节中的操作相同 - 将其从默认目录复制到您的自定义模板目录）。编辑文件，你会看到它使用一个模板变量app_list。该变量包含每个已安装的Django应用程序。而不是使用它，您可以以任何您认为最佳的方式将链接强制代码到特定于对象的管理页面。


    下一步是什么？¶

    初学者教程结束于此。在此期间，您可能想从这里查看一些指向哪里的指针。
    https://docs.djangoproject.com/en/1.11/intro/whatsnext/

    如果您熟悉Python包装，并有兴趣了解如何将民意调查转化为“可重用的应用程序”，请参阅高级教程：如何编写可重用的应用程序。

    高级教程：如何编写可重用的应用程序¶
    https://docs.djangoproject.com/en/1.11/intro/reusable-apps/

    接下来读什么
    https://docs.djangoproject.com/en/1.11/intro/whatsnext/

    为Django编写你的第一个补丁
    https://docs.djangoproject.com/en/1.11/intro/contributing/


