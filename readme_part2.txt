


2.编写您的第一个Django应用程序-2
    第2部分 :https://docs.djangoproject.com/en/1.10/intro/tutorial02/

    本教程从教程1中断开始。我们将设置数据库，创建您的第一个模型，并快速介绍Django的自动生成的管理站点。

   1.数据库设置

    现在，开放mysite/settings.py。这是一个普通的Python模块，模块级变量表示Django设置。

    任何其他东西来支持您的数据库。然而，在启动第一个真实项目时，您可能希望使用PostgreSQL等更为可扩展的数据库，以避免数据库切换头痛。

    如果您希望使用其他数据库，请安装相应的数据库绑定(https://docs.djangoproject.com/en/1.11/topics/install/#database-installation)，
    并更改项中的以下 项以匹配数据库连接设置：DATABASES 'default'

    ENGINE-要么 'django.db.backends.sqlite3'， 'django.db.backends.postgresql'， 'django.db.backends.mysql'，或 'django.db.backends.oracle'。其他后端也可用。
    NAME - 数据库的名称。如果您使用SQLite，数据库将是您计算机上的一个文件; 在这种情况下，NAME 应该是该文件的完全绝对路径，包括文件名。默认值，将文件存储在项目目录中。os.path.join(BASE_DIR, 'db.sqlite3')
    如果你不使用SQLite作为数据库，额外的设置，例如 USER，PASSWORD和HOST必须加入。有关详细信息，请参阅参考文档DATABASES。
    https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-DATABASES


    编辑mysite/settings.py时，设置TIME_ZONE为您的时区。

    另外，请注意INSTALLED_APPS文件顶部的设置。它包含在该Django实例中激活的所有Django应用程序的名称。应用程序可以在多个项目中使用，您可以打包和分发，以供他人在项目中使用。

    django.contrib.admin - 管理网站。你会很快使用它。
    django.contrib.auth - 认证系统。
    django.contrib.contenttypes - 内容类型的框架。
    django.contrib.sessions - 会话框架
    django.contrib.messages - 消息框架。
    django.contrib.staticfiles - 管理静态文件的框架。

    这些应用程序默认包含在一般情况下是方便的。

    其中一些应用程序使用至少一个数据库表，因此我们需要在数据库中创建表，然后才能使用它们。为此，请运行以下命令：

    python manage.py migrate

    该migrate命令查看该INSTALLED_APPS设置，并根据mysite/settings.py文件中的数据库设置和应用程序随附的数据库迁移创建任何必需的数据库表（稍后将介绍）。您会看到适用于每个迁移的消息。如果您有兴趣，请运行数据库的命令行客户端并输入\dt（PostgreSQL），（MySQL）， （SQLite）或（Oracle）以显示Django创建的表。SHOW TABLES;.schemaSELECT TABLE_NAME FROM USER_TABLES;


   2.创建模型
     现在我们将使用额外的元数据来定义你的模型 - 本质上是你的数据库布局。
     在我们的简单调查应用程序，我们将创建两个模式：Question和Choice。A Question有一个问题和一个出版日期。A Choice有两个字段：选择的文本和投票。每个Choice都与a相关联Question。

     这些概念由简单的Python类表示。编辑 polls/models.py文件，看起来像这样：

    代码很简单。每个模型都由一个类来表示django.db.models.Model。每个模型都有一些类变量，每个变量都表示模型中的数据库字段。

    每个字段由Field 类的实例表示- 例如，CharField用于字符字段和 DateTimeField数据时间。这告诉Django每个字段拥有什么类型的数据。

    每个Field实例的名称（例如 question_text或pub_date）是字段的名称，以机器友好的格式。您将在Python代码中使用此值，数据库将使用它作为列名。

    您可以使用可选的第一个位置参数来 Field指定人类可读的名称。这用于Django的几个内省部分，它可以作为文档。如果未提供此字段，Django将使用机器可读的名称。在这个例子中，我们只定义了一个可读的名字Question.pub_date。对于此模型中的所有其他字段，该字段的机器可读名称将足以作为其可读的名称。

    一些Field类有必要的参数。 CharField例如，要求你给它一个 max_length。这不仅在数据库模式中使用，而且用于验证，我们即将看到。

    A Field也可以有各种可选参数; 在这种情况下，我们将default值 设置votes为0。

    最后，注意一个关系的定义，使用 ForeignKey。这告诉Django每个Choice都与一个相关Question。Django支持所有常见的数据库关系：多对一，多对多和一一对应。


    3.激活模型
    那个小的模型代码给了Django很多信息。有了它，Django能够：

    为此应用程序创建数据库模式（语句）。CREATE TABLE
    创建用于访问Question和Choice对象的Python数据库访问API 。
    但首先我们需要告诉我们的项目该polls应用是安装的。

    要将该应用程序包括在我们的项目中，我们需要在设置中添加对其配置类的引用INSTALLED_APPS。该 PollsConfig班是在polls/apps.py文件中，所以它的虚线路径'polls.apps.PollsConfig'。编辑mysite/settings.py文件，并将该虚线路径添加到该INSTALLED_APPS设置。看起来像这样：

    mysite / settings.py

        INSTALLED_APPS  =  [
        'polls.apps.PollsConfig' ，
        'django.contrib.admin' ，
        'django.contrib.auth' ，
        'django.contrib.contenttypes' ，
        'django.contrib.sessions' ，
        'django.contrib.messages' ，
        'django.contrib.staticfiles' ，
    ]

    现在，Django知道包含该polls应用程序。我们来运行另一个命令：

    python manage.py makemigrations polls

    你应该看到类似于以下内容：

    Migrations for 'polls':
        polls/migrations/0001_initial.py:
        - Create model Choice
        - Create model Question
        - Add field question to choice

        通过运行makemigrations，您告诉Django您已经对模型进行了一些更改（在这种情况下，您已经创建了新的模型），并且希望将更改存储为迁移。

        迁移是Django如何存储对模型（以及数据库模式）的更改 - 它们只是磁盘上的文件。如果您喜欢，您可以阅读新模型的迁移; 这是文件 polls/migrations/0001_initial.py。别担心，Django每次都不会读取它们，但是如果您想手动调整Django如何更改内容，那么它们的设计是可编辑的。

        有一个命令可以为您运行迁移并自动管理数据库模式 - 这是一个调用migrate，我们稍后会介绍 - 首先，我们来看看迁移将运行什么SQL。该 sqlmigrate命令接受迁移名称并返回其SQL：

        python manage.py sqlmigrate polls 0001

        您应该看到类似于以下内容（我们已将其重新格式化为可读性）：

                BEGIN;
                --
                -- Create model Choice
                --
                CREATE TABLE "polls_choice" (
                    "id" serial NOT NULL PRIMARY KEY,
                    "choice_text" varchar(200) NOT NULL,
                    "votes" integer NOT NULL
                );
                --
                -- Create model Question
                --
                CREATE TABLE "polls_question" (
                    "id" serial NOT NULL PRIMARY KEY,
                    "question_text" varchar(200) NOT NULL,
                    "pub_date" timestamp with time zone NOT NULL
                );
                --
                -- Add field question to choice
                --
                ALTER TABLE "polls_choice" ADD COLUMN "question_id" integer NOT NULL;
                ALTER TABLE "polls_choice" ALTER COLUMN "question_id" DROP DEFAULT;
                CREATE INDEX "polls_choice_7aa0f6ee" ON "polls_choice" ("question_id");
                ALTER TABLE "polls_choice"
                  ADD CONSTRAINT "polls_choice_question_id_246c99a640fbbd72_fk_polls_question_id"
                    FOREIGN KEY ("question_id")
                    REFERENCES "polls_question" ("id")
                    DEFERRABLE INITIALLY DEFERRED;

                COMMIT;
         请注意以下事项：

                确切的输出将取决于您正在使用的数据库。以上示例为PostgreSQL生成。
                表名是通过组合应用程序的名称（自动生成polls）和模型的小写名字- question和 choice。（您可以覆盖此行为。）
                主键（ID）将自动添加。（你也可以重写这个。）
                按照惯例，Django附加"_id"到外键字段名称。（是的，你也可以覆盖这个。）
                外键关系是通过约束来明确的 。不要担心零件; 这只是告诉PostgreSQL不执行外键直到事务结束。FOREIGN KEYDEFERRABLE
                它适合您使用的数据库，因此数据库特定的字段类型（如auto_incrementMySQL），serial（PostgreSQL）或（SQLite））会自动为您处理。引用字段名称也是如此，例如使用双引号或单引号。integer primary key autoincrement
                该sqlmigrate命令实际上不会在数据库上运行迁移 - 它只是打印到屏幕上，以便您可以看到需要什么SQL Django认为。检查Django要执行的操作或者是否有需要SQL脚本进行更改的数据库管理员很有用。
                如果你有兴趣，你也可以跑 ; 这将检查项目中的任何问题，而不进行迁移或触摸数据库。python manage.py check

                现在，migrate再次运行以在数据库中创建这些模型表：

                 python manage.py migrate:

                              Operations to perform:
                              Apply all migrations: admin, auth, contenttypes, polls, sessions
                              Running migrations:
                              Rendering model states... DONE
                              Applying polls.0001_initial... OK

             该migrate命令采用所有尚未应用的迁移（Django跟踪使用您调用的数据库中的特殊表应用哪些django_migrations迁移），并根据数据库运行它们 - 基本上，将您对模型所做的更改与数据库。
         迁移非常强大，随着时间的推移，随着时间的推移，随着时间的推移，您可以随时更改模型，而无需删除数据库或表，并创建新的模型 - 它专门用于实时升级数据库，而不会丢失数据。我们将在本教程的后续部分中更深入地介绍它们，
    但是现在，请记住进行模型更改的三步指南：
             改变你的模型（in models.py）。
             运行以创建这些更改的迁移python manage.py makemigrations
             运行以将这些更改应用于数据库。python manage.py migrate
                有单独的命令来制作和应用迁移的原因是因为您将提交迁移到版本控制系统并将其发送到您的应用程序; 它们不仅可以使您的开发更容易，而且还可以被其他开发人员和生产中使用。
                请阅读django-admin文档，了解有关该manage.py实用程序可以执行的操作的完整信息。
     3.使用：
     现在，让我们进入交互式的Python shell，并使用Django提供的免费API。要调用Python shell，请使用以下命令：

     python manage.py shell

     我们使用这个，而不是简单地输入“python”，因为manage.py 设置了DJANGO_SETTINGS_MODULE环境变量，这给Django提供了你的mysite/settings.py文件的Python导入路径。

        绕过manage.py
         如果你不想使用manage.py，没问题。只是设置 DJANGO_SETTINGS_MODULE环境变量 mysite.settings，启动一个简单的Python shell，并设置Django：
            >>> import django
            >>> django.setup()
                    如果这引发了一个问题AttributeError，你可能会使用一个与本教程版本不符的Django版本。您将需要切换到较旧的教程或较新的Django版本。
                    您必须python从同一个目录运行manage.py，或者确保该目录位于Python路径上，这样才能 正常工作。import mysite
                    有关所有这些的更多信息，请参阅django-admin文档。https://docs.djangoproject.com/en/1.11/ref/django-admin/

    一旦你进入shell，请探索数据库API：

         >>> from polls.models import Question, Choice   # Import the model classes we just wrote.

         ＃系统中还没有问题。

         >>> Question.objects.all()
         <QuerySet []>
        ＃创建一个新的问题。
        ＃在默认设置文件中启用对时区的支持，所以
        ＃Django需要一个tzinfo的datetime为pub_date。使用timezone.now（）
        ＃而不是datetime.datetime.now（），它会做正确的事情。

         >>> from django.utils import timezone
            q = Question(question_text="What's new?", pub_date=timezone.now())

         ＃将对象保存到数据库中。你必须显式调用save（）。
         >>> q.save()

        ＃现在有一个ID。请注意，这可能会说“1L”而不是“1”，具体取决于
        您使用的数据库。这不是大笨蛋 它只是意味着你的
        ＃数据库后端更喜欢返回整数作为Python long integer
        ＃对象。

        >>> q.id

        ＃通过Python属性访问模型字段值。
        >>> q.question_text
        “新功能”
        >>> q.pub_date
        datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

        ＃通过更改属性来更改值，然后调用save（）。
        >>> q.question_text = "What's up?"
        >>> q.save()

        ＃objects.all（）显示数据库中的所有问题。
        >>> Question.objects.all()
        <QuerySet [<Question: Question object>]>



        等一下。完全是这个对象的无益表示。让我们来解决这个问题通过编辑模型（在 文件），并加入 到两个方法和 ：<Question: Question object>Questionpolls/models.py__str__()QuestionChoice
        polls/models.py
                   from django.db import models
                   from django.utils.encoding import python_2_unicode_compatible

                    @python_2_unicode_compatible  # only if you need to support Python 2
                    class Question(models.Model):
                        # ...
                        def __str__(self):
                            return self.question_text

                    @python_2_unicode_compatible  # only if you need to support Python 2
                    class Choice(models.Model):
                        # ...
                        def __str__(self):
                            return self.choice_text

        __str__()为模型添加方法非常重要，这不仅在处理交互式提示时方便您使用，还因为Django自动生成的管理员中使用了对象的表示。

        注意这些是普通的Python方法。我们添加一个自定义的方法，只是为了演示：

        polls / models.py

                import datetime

                from django.db import models
                from django.utils import timezone


                class Question(models.Model):
                    # ...
                    def was_published_recently(self):
                        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
        请注意添加和分别参考Python的标准模块和Django的时区相关的实用程序。如果您不熟悉Python中的时区处理，您可以在时区支持文档中了解更多信息。import datetimefrom django.utils import timezonedatetimedjango.utils.timezone

        保存这些更改并重新启动一个新的Python交互式shell ：python manage.py shell

        >>> from polls.models import Question, Choice

        ＃确保我们的__str __（）添加工作。
        >>> Question.objects.all()
        <QuerySet [<Question: What's up?>]>>

        ＃Django 提供了一个完全由＃个关键字参数驱动的丰富的数据库查找API

        >>> Question.objects.filter(id=1)
        <QuerySet [<Question: What's up?>]>
        >>> Question.objects.filter(question_text__startswith='What')
        <QuerySet [<Question: What's up?>]>

        ＃获取今年发布的问题。

        >>> from django.utils import timezone
        >>> current_year = timezone.now().year
        >>> Question.objects.get(pub_date__year=current_year)
        <Question: What's up?>

        ＃请求不存在的ID，这将引发异常。

        >>> Question.objects.get(id=2)
        Traceback (most recent call last):
            ...
        DoesNotExist: Question matching query does not exist.

        ＃通过主键查找是最常见的情况，所以Django提供了主键精确查找的快捷方式。
        ＃以下与Question.objects.get（id = 1）相同。

        >>> Question.objects.get(pk=1)
        <Question: What's up?>

        ＃确保我们的自定义方法有效。
        >>> q = Question.objects.get(pk=1)
        >>> q.was_published_recently()
        True

        ＃给问题几个选择。创建调用构造一个新的＃Choice
        对象，INSERT语句将选择添加到可用选项的集合
        中，并返回新的“Choice”对象。Django创建
        一个集合来保存ForeignKey关系的“另一面”
        （例如一个问题的选择），可以通过API访问它。

        >>> q = Question.objects.get(pk=1)

        ＃显示相关对象集中的任何选项 - 至今没有。

        ＃创建三个选择。
        >>> q.choice_set.create(choice_text='Not much', votes=0)
        <选择：不太多>
        >>> q.choice_set.create(choice_text='The sky', votes=0)
        <选择：天空>
        >>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

        ＃选择对象可以访问其相关的问题对象。
        >>> c.question
        <Question: What's up?>

        ＃反之亦然：问题对象可以访问Choice对象。
        >>> q.choice_set.all()
        <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
        >>> q.choice_set.count()
        3

        ＃API根据需要自动跟随关系。
        ＃使用双重下划线来分隔关系。
        ＃这样可以像你想要的那样深层次的工作; 没有限制
        ＃查找pub_date在今年的任何问题的所有选项
        ＃（重复使用上面创建的'current_year'变量）。

        >>> Choice.objects.filter(question__pub_date__year=current_year)
        <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

        ＃我们来删除一个选择。使用delete（）。

        >>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
        >>> c.delete()

        有关模型关系的更多信息，请参阅访问相关对象。https://docs.djangoproject.com/en/1.11/ref/models/relations/
        有关如何使用双下划线通过API执行字段查找的更多信息，请参阅字段查找。有关数据库API的完整详细信息，请参阅我们的数据库API参考。
        https://docs.djangoproject.com/en/1.11/topics/db/queries/

    3.介绍Django管理

        创建管理员用户

        首先，我们需要创建一个可以登录管理站点的用户。运行以下命令

        python manage.py createsuperuser

        输入所需的用户名，然后按Enter键。

        Username: admin

        然后将提示您输入所需的电子邮件地址：

        Email address: admin@example.com

        最后一步是输入你的密码。您将被要求输入密码两次，第二次作为第一次的确认。

        Password: **********
        Password (again): *********
        Superuser created successfully.

        启动开发服务器

        默认情况下，Django管理员站点被激活。让我们开始开发服务器并探索它。

        如果服务器没有运行启动它就像这样：

        python manage.py runserver

        现在，打开一个Web浏览器，然后转到本地域的“/ admin /” - 例如 http://127.0.0.1:8000/admin/。您应该看到管理员的登录屏幕：

        由于翻译是默认开启，登录屏幕可以显示在你自己的语言，这取决于你的浏览器的设置，如果Django的有翻译这门语言。
        https://docs.djangoproject.com/en/1.11/topics/i18n/translation/

        进入管理站点
            现在，尝试使用您在上一步中创建的超级用户帐户登录。你应该看到Django管理员索引页面：

            您应该看到几种类型的可编辑内容：groups and users。它们django.contrib.auth由Django 提供的认证框架提供。

    请投票应用程序的管理修改

    但是我们的投票应用程序在哪里？它不显示在管理员索引页面上。
    只需要做一件事：我们需要告诉管理员Question 对象有一个管理界面。要做到这一点，打开polls/admin.py 文件，并编辑它看起来像这样：

    polls / admin.py

    from django.contrib import admin
    from .models import Question
    admin.site.register(Question)

    探索免费的管理功能

    现在我们注册了Question，Django知道它应该显示在管理员索引页面上：

    点击“问题”。现在您正在“更改列表”页面查询问题。此页面显示数据库中的所有问题，并允许您选择一个来更改它。我们之前创建的“怎么了？”问题

    点击“What's up？”问题来编辑它：

    注意事项：

        表单是从Question模型自动生成的。
        不同的模型字段类型（DateTimeField， CharField）对应于适当的HTML输入小部件。每种类型的字段知道如何在Django管理员中显示自己。
        每个都DateTimeField获得免费的JavaScript快捷方式。日期获取“今日”快捷方式和日历弹出窗口，并且时间获取“Now”快捷方式和一个方便的弹出窗口，列出常用的时间。
        页面底部提供了几个选项：

        保存 - 保存更改并返回到此类型对象的更改列表页面。
        保存并继续编辑 - 保存更改并重新加载此对象的管理页面。
        保存并添加另一个 - 保存更改并加载此类型对象的新的空白表单。
        删除 - 显示删除确认页面。
        如果“发布日期”的值与教程1中创建问题的时间不匹配，则可能意味着您忘记了设置的正确值TIME_ZONE。更改它，重新加载页面，并检查是否显示正确的值。

        点击“今日”和“现在”快捷方式更改“发布日期”。然后点击“保存并继续编辑”，然后点击右上角的“历史记录”。您将看到一个页面，其中列出了通过Django管理员对此对象所做的所有更改，以及进行更改的人员的时间戳和用户名：

       当您对模型API感到满意并熟悉管理员网站时，请阅读本教程的第3部分，了解如何向我们的投票应用添加更多视图。
        https://docs.djangoproject.com/en/1.11/intro/tutorial03/
