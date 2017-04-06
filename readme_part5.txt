
5.编写你的第一个Django应用，部分5
    第5部分 :https://docs.djangoproject.com/en/1.11/intro/tutorial05/
    本教程将在教程4停止之前开始。我们已经构建了一个Web-poll应用程序，现在我们将为它创建一些自动测试。

    1.引入自动测试
        什么是自动测试？
            测试是检查代码操作的简单例程。

            测试运行在不同的层次。一些测试可能适用于一个细微的细节（特定的模型方法是否按预期的方式返回值），而其他测试可以检查软件的整体操作（站点上的一系列用户输入是否产生所需的结果？）。这与以前在教程2中所做的那种测试没有什么不同，它使用 shell检查方法的行为或运行应用程序并输入数据来检查它的行为。

            自动化测试有什么不同之处在于测试工作是由系统完成的。您创建一组测试一次，然后在对应用进行更改时，您可以检查代码是否仍然按照原始的方式工作，而无需执行耗时的手动测试。

        为什么你需要创建测试

            那么为什么要创建测试，为什么现在呢？

            你可能会觉得你的盘子已经足够了，只是学习Python / Django，还有另外一件事要学习和做，可能看起来压倒一切，也许是不必要的。毕竟，我们的民意调查申请工作现在很开心，经历创建自动化测试的麻烦不会使其工作更好。如果创建投票应用程序是Django编程的最后一个位置，那么您将无法做到这一点，所以您不需要知道如何创建自动化测试。但是，如果不是这样，现在是学习的好时机。

        测试会节省你的时间

            直到某一点，“检查它似乎工作”将是一个令人满意的测试。在更复杂的应用程序中，组件之间可能会有数十种复杂的交互。

            任何这些组件的更改可能会对应用程序的行为产生意想不到的后果。检查它仍然“似乎工作”可能意味着运行您的代码的功能与二十个不同的变化的测试数据只是为了确保你没有破坏的东西 - 不是很好的使用你的时间。

            当自动化测试可以在几秒钟内为您做到这一点时尤其如此。如果出现问题，测试也将有助于识别导致意外行为的代码。

            有时看起来可能会让自己脱离生产性的创意编程工作，面对编写测试的无懈可击的业务，特别是当您知道代码正常工作时。

            然而，编写测试的任务比花费小时手动测试应用程序要更加充分，或者试图找出新引入的问题的原因。


        测试不只是发现问题，他们阻止他们

            将测试仅仅视为发展的消极方面是错误的。

            没有测试，应用程序的目的或预期行为可能是相当不透明的。即使这是你自己的代码，你也会发现自己正在试图找出它在做什么。

            测试改变了 他们从内部点亮你的代码，当出现问题时，他们将重点放在已经出错的部分 - 即使你甚至没有意识到错误。

        测试使您的代码更有吸引力

            您可能已经创建了一个辉煌的软件，但您会发现，许多其他开发人员将拒绝查看它，因为它缺乏测试; 没有测试，他们不会相信。Django的原始开发人员之一Jacob Kaplan-Moss说：“没有测试的代码被设计破坏了”。

            其他开发人员想要在您的软件中看到测试被认真对待之前，是您开始编写测试的另一个原因。

        测试帮助团队合作

            以前的观点是从维护应用程序的单个开发者的角度编写的。复杂的应用程序将由团队维护。测试保证同事不会无意中破解您的代码（并且您不会在不知情的情况下破坏它们）。如果你想作为Django程序员生活，你必须善于写作测试！

        基本测试策略

            写测试有很多方法。

            一些程序员遵循一个名为“ 测试驱动开发 ”的学科 ; 他们实际上在编写代码之前写下他们的测试。这可能看起来是反直觉的，但实际上它类似于大多数人经常会做的：他们描述一个问题，然后创建一些代码来解决它。测试驱动开发简单地在Python测试用例中形成问题。

            更多的时候，一个新的测试人员会创建一些代码，然后决定它应该有一些测试。也许早些时候写一些测试会更好一些，但是从来没有太迟。

            有时很难弄清楚在哪里开始编写测试。如果你已经写了几千行Python，选择要测试的东西可能并不容易。在这种情况下，无论是在添加新功能还是修复错误时，在下次进行更改时编写第一个测试都是非常有益的。

            所以我们这样做吧。

    2.写我们的第一个测试

        我们确定了一个错误

            幸运的是，在一个小错误polls的应用为我们立即进行修复：该Question.was_published_recently()方法返回True，如果Question是最后一天（这是正确的）内发布，而且如果Question的pub_date领域是未来（这当然不是） 。

            要检查该bug是否真的存在，使用Admin创建一个日期在将来的问题，并使用以下方法检查方法shell：

                >>> import datetime
                >>> from django.utils import timezone
                >>> from polls.models import Question
                >>> # create a Question instance with pub_date 30 days in the future
                >>> future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))
                >>> # was it published recently?
                >>> future_question.was_published_recently()
                True

            由于未来的事情不是“最近”，这显然是错误的。

        创建一个测试来暴露错误

            我们刚刚在shell测试中所做的工作正是我们在自动化测试中所能做的，所以让我们把它变成一个自动测试。

            应用程序测试的常规地方在应用程序的 tests.py文件中; 测试系统将自动在任何名称开头的文件中找到测试test。

            将以下内容放在应用程序的tests.py文件中polls：

            polls/tests.py
                import datetime

                from django.utils import timezone
                from django.test import TestCase

                from .models import Question


                class QuestionMethodTests(TestCase):

                    def test_was_published_recently_with_future_question(self):
                        """
                        was_published_recently() should return False for questions whose
                        pub_date is in the future.
                        """
                        time = timezone.now() + datetime.timedelta(days=30)
                        future_question = Question(pub_date=time)
                        self.assertIs(future_question.was_published_recently(), False)

            我们在这里做的是创建一个django.test.TestCase子类，其方法可以在将来创建一个Question实例pub_date。然后我们检查输出was_published_recently()- 哪个 应该是False。

         运行测试

            python manage.py test polls

            你会看到像：

                Creating test database for alias 'default'...
                System check identified no issues (0 silenced).
                F
                ======================================================================
                FAIL: test_was_published_recently_with_future_question (polls.tests.QuestionMethodTests)
                ----------------------------------------------------------------------
                Traceback (most recent call last):
                  File "/path/to/mysite/polls/tests.py", line 16, in test_was_published_recently_with_future_question
                    self.assertIs(future_question.was_published_recently(), False)
                AssertionError: True is not False

                ----------------------------------------------------------------------
                Ran 1 test in 0.001s

                FAILED (failures=1)
                Destroying test database for alias 'default'...

            这是怎么回事
                python manage.py test polls看着在测试中polls的应用
                它发现了一个类的子django.test.TestCase类
                它为了测试而创建了一个特殊的数据库
                它寻找测试方法 - 名字开头的方法 test
                在test_was_published_recently_with_future_question其中创建了一个现在30天的Question 实例pub_date
                ...并使用该assertIs()方法，它发现它的 was_published_recently()返回True，尽管我们希望它返回 False
                测试通知我们哪个测试失败，甚至发生故障的行。

            修正错误

                我们已经知道问题是什么：如果将来Question.was_published_recently()会返回。修改方法 ，只有当日期也在过去时才会返回：Falsepub_datemodels.pyTrue

            polls/models.py

                def was_published_recently(self):
                now = timezone.now()
                return now - datetime.timedelta(days=1) <= self.pub_date <= now

            并再次运行测试：

                Creating test database for alias 'default'...
                System check identified no issues (0 silenced).
                .
                ----------------------------------------------------------------------
                Ran 1 test in 0.001s

                OK
                Destroying test database for alias 'default'...

                在识别错误后，我们写了一个暴露的测试，并更正了代码中的错误，以便我们的测试通过。

                许多其他事情在将来可能会出现错误，但我们可以肯定，我们不会无意中重新引入这个错误，因为只需运行测试就会立即警告我们。我们可以把这个应用程序的这一小部分考虑在一个安全的地方永远固定下来。

            更全面的测试

                我们在这里，我们可以进一步确定was_published_recently() 方法; 事实上，如果在修复一个我们引入另一个的bug的话会是非常尴尬的。

                向同一个类添加两个测试方法，以更全面地测试该方法的行为：

                polls/tests.py
                    def test_was_published_recently_with_old_question(self):
                    """
                    was_published_recently() should return False for questions whose
                    pub_date is older than 1 day.
                    """
                    time = timezone.now() - datetime.timedelta(days=30)
                    old_question = Question(pub_date=time)
                    self.assertIs(old_question.was_published_recently(), False)

                def test_was_published_recently_with_recent_question(self):
                    """
                    was_published_recently() should return True for questions whose
                    pub_date is within the last day.
                    """
                    time = timezone.now() - datetime.timedelta(hours=1)
                    recent_question = Question(pub_date=time)
                    self.assertIs(recent_question.was_published_recently(), True)

                现在我们有三个测试，证实Question.was_published_recently() 了过去，最近和将来的问题返回合理的价值。

                再一次，polls这是一个简单的应用程序，但无论如何，它将在未来发展很复杂，并且与其他代码进行交互，我们现在有一些保证，我们编写测试的方法将以预期的方式运行。

            测试视图

                民意调查申请是相当不分青红皂白的：它会发布任何问题，包括pub_date未来领域的问题。我们应该改善这个。pub_date在未来设定应该意味着问题在那一刻被公布，但到那时才是看不见的。

            视图的测试

                当我们修复上面的错误时，我们首先编写测试，然后编写代码来修复它。实际上这是测试驱动开发的一个简单的例子，但是我们做的这个工作并不重要。

                在我们的第一个测试中，我们密切关注代码的内部行为。对于这个测试，我们想检查一下用户通过Web浏览器所体验的行为。

                在我们尝试修复任何东西之前，让我们来看看我们所掌握的工具。

            Django的测试客户端
                Django提供了一个测试Client来模拟用户在视图级别与代码进行交互。我们可以使用它tests.py 甚至在shell。

                我们将重新开始shell，在那里我们需要做一些不必要的事情tests.py。首先是设置测试环境shell：

                >>> from django.test.utils import setup_test_environment
                >>> setup_test_environment()

                setup_test_environment()安装模板渲染器，这将允许我们检查响应中的一些附加属性 response.context，否则将不可用。请注意，此方法不会设置测试数据库，因此将根据现有数据库运行以下操作，并且输出可能会略有不同，具体取决于您已创建的问题。如果您TIME_ZONE的settings.py内容不正确，您可能会收到意想不到的结果 。如果您不记得早点设置，请在继续之前查看。

                接下来，我们需要导入测试客户端类（稍后tests.py我们将使用django.test.TestCase它自己的客户端附带的类，所以这不是必需的）：

                >>> from django.test import Client
                >>> # create an instance of the client for our use
                >>> client = Client()

                有了这个准备，我们可以请客户为我们做一些工作：

                >>> # get a response from '/'
                >>> response = client.get('/')
                >>> # we should expect a 404 from that address
                >>> response.status_code
                404
                >>> # on the other hand we should expect to find something at '/polls/'
                >>> # we'll use 'reverse()' rather than a hardcoded URL
                >>> from django.urls import reverse
                >>> response = client.get(reverse('polls:index'))
                >>> response.status_code
                200
                >>> response.content
                b'\n    <ul>\n    \n        <li><a href="/polls/1/">What&#39;s up?</a></li>\n    \n    </ul>\n\n'
                >>> # If the following doesn't work, you probably omitted the call to
                >>> # setup_test_environment() described above
                >>> response.context['latest_question_list']
                <QuerySet [<Question: What's up?>]>


            提高我们的观点:
               民意测验清单显示尚未发布的民意调查（即将来有这种民意测验 pub_date）。我们来解决这个问题。

               在教程4中，我们介绍了基于类的视图，基于ListView：

               polls/views.py

               class IndexView(generic.ListView):
                    template_name = 'polls/index.html'
                    context_object_name = 'latest_question_list'

                    def get_queryset(self):
                        """Return the last five published questions."""
                        return Question.objects.order_by('-pub_date')[:5]

                我们需要修改get_queryset()方法并进行更改，以便通过与之进行比较来检查日期timezone.now()。首先我们需要添加一个导入：

                polls/views.py
                    from django.utils import timezone

                那么我们必须get_queryset像这样修改方法：

                polls/views.py

                def get_queryset(self):
                    """
                    Return the last five published questions (not including those set to be
                    published in the future).
                    """
                    return Question.objects.filter(
                        pub_date__lte=timezone.now()
                    ).order_by('-pub_date')[:5]

                Question.objects.filter(pub_date__lte=timezone.now())返回一个包含小于或等于 - 即早于或等于 - 的Questions的查询集。pub_datetimezone.now

            测试我们的新视图

                现在，您可以通过启动运行服务器，在浏览器中加载站点Questions，以过去和将来的日期进行创建，并检查仅列出已发布的站点，从而满足您的需求。你不希望每一次都做这些改变，这可能会影响到这一点 - 所以我们还要根据上面的shell会话创建一个测试 。

                将以下内容添加到polls/tests.py：

                polls/tests.py
                    from django.urls import reverse

                我们将创建一个快捷方式函数来创建问题以及一个新的测试类：

                polls/tests.py

                    def create_question(question_text, days):
                    """
                    Creates a question with the given `question_text` and published the
                    given number of `days` offset to now (negative for questions published
                    in the past, positive for questions that have yet to be published).
                    """
                    time = timezone.now() + datetime.timedelta(days=days)
                    return Question.objects.create(question_text=question_text, pub_date=time)


                class QuestionViewTests(TestCase):
                    def test_index_view_with_no_questions(self):
                        """
                        If no questions exist, an appropriate message should be displayed.
                        """
                        response = self.client.get(reverse('polls:index'))
                        self.assertEqual(response.status_code, 200)
                        self.assertContains(response, "No polls are available.")
                        self.assertQuerysetEqual(response.context['latest_question_list'], [])

                    def test_index_view_with_a_past_question(self):
                        """
                        Questions with a pub_date in the past should be displayed on the
                        index page.
                        """
                        create_question(question_text="Past question.", days=-30)
                        response = self.client.get(reverse('polls:index'))
                        self.assertQuerysetEqual(
                            response.context['latest_question_list'],
                            ['<Question: Past question.>']
                        )

                    def test_index_view_with_a_future_question(self):
                        """
                        Questions with a pub_date in the future should not be displayed on
                        the index page.
                        """
                        create_question(question_text="Future question.", days=30)
                        response = self.client.get(reverse('polls:index'))
                        self.assertContains(response, "No polls are available.")
                        self.assertQuerysetEqual(response.context['latest_question_list'], [])

                    def test_index_view_with_future_question_and_past_question(self):
                        """
                        Even if both past and future questions exist, only past questions
                        should be displayed.
                        """
                        create_question(question_text="Past question.", days=-30)
                        create_question(question_text="Future question.", days=30)
                        response = self.client.get(reverse('polls:index'))
                        self.assertQuerysetEqual(
                            response.context['latest_question_list'],
                            ['<Question: Past question.>']
                        )

                    def test_index_view_with_two_past_questions(self):
                        """
                        The questions index page may display multiple questions.
                        """
                        create_question(question_text="Past question 1.", days=-30)
                        create_question(question_text="Past question 2.", days=-5)
                        response = self.client.get(reverse('polls:index'))
                        self.assertQuerysetEqual(
                            response.context['latest_question_list'],
                            ['<Question: Past question 2.>', '<Question: Past question 1.>']
                        )


                我们再来看一下这些。

                首先是一个问题的快捷功能，create_question在创建问题的过程中要重复一些。

                test_index_view_with_no_questions不会创建任何问题，但检查消息：“没有轮询可用”，并验证latest_question_list 是空的。请注意，django.test.TestCase该类提供了一些额外的断言方法。在这些例子中，我们使用 assertContains()和 assertQuerysetEqual()。

                在test_index_view_with_a_past_question，我们创建一个问题并验证它是否出现在列表中。

                在test_index_view_with_a_future_question，我们pub_date在将来创造一个问题 。每个测试方法重置数据库，所以第一个问题不再存在，索引也不应该有任何问题。

                等等。实际上，我们正在使用测试来讲述网站上的管理员输入和用户体验的故事，并检查在每个国家和系统状态的每一个新的变化，预期的结果被公布。

            测试DetailView
                我们的工作很好 然而，即使未来的问题没有出现在索引中，如果用户知道或猜测到正确的URL，用户仍然可以访问它们。所以我们需要添加一个类似的约束DetailView：

                polls / views.py

                    class DetailView(generic.DetailView):
                    ...
                    def get_queryset(self):
                        """
                        Excludes any questions that aren't published yet.
                        """
                        return Question.objects.filter(pub_date__lte=timezone.now())


                当然，我们将增加一些测试，以检查一个Question，其 pub_date在过去可以显示，而一个具有pub_date 在未来是不是：


                polls/tests.py

                    class QuestionIndexDetailTests(TestCase):
                        def test_detail_view_with_a_future_question(self):
                            """
                            The detail view of a question with a pub_date in the future should
                            return a 404 not found.
                            """
                            future_question = create_question(question_text='Future question.', days=5)
                            url = reverse('polls:detail', args=(future_question.id,))
                            response = self.client.get(url)
                            self.assertEqual(response.status_code, 404)

                        def test_detail_view_with_a_past_question(self):
                            """
                            The detail view of a question with a pub_date in the past should
                            display the question's text.
                            """
                            past_question = create_question(question_text='Past Question.', days=-5)
                            url = reverse('polls:detail', args=(past_question.id,))
                            response = self.client.get(url)
                            self.assertContains(response, past_question.question_text)

            更多测试的想法

                我们应该添加一个类似的get_queryset方法ResultsView并为该视图创建一个新的测试类。这与我们刚刚创造的非常相似; 其实会有很多重复。

                我们也可以以其他方式改进我们的应用程序，一路上添加测试。例如，Questions可以在没有的网站上发布它是愚蠢的Choices。所以，我们的意见可以检查，排除这样的 Questions。我们的测试将创建一个Question没有Choices然后测试，它没有发布，以及创建一个类似Question 的 Choices，并测试它被发布。

                也许登录管理员用户应该被允许看到未发布的 Questions，但不是普通的访问者。再次：无论需要添加到软件中以完成这一点，都应该伴随着一个测试，无论你是先写测试，然后使代码通过测试，或者先编写代码中的逻辑，然后写一个测试证明给我看。

                在某一点上，你一定要看看你的测试，并想知道你的代码是否患有测试膨胀，这使我们感到：

            当测试时，更多更好

                似乎我们的测试正在失控。在这个速度下，我们的测试中将会比在我们的应用程序中更多的代码，并且与其他代码的优雅的简洁性相比，重复是不美观的。

                没关系。让他们成长。在大多数情况下，您可以编写一次测试，然后忘记一下。当您继续开发您的程序时，它将继续执行其有用的功能。

                有时测试需要更新。假设我们修正了我们的观点，以便只Questions与Choices发布。在这种情况下，我们许多现有的测试将会失败 - 告诉我们哪些测试需要修改以使它们更新，所以在这个程度上测试可以帮助照顾自己。

                最糟糕的是，随着您继续开发，您可能会发现您有一些现在已经是冗余的测试。即使这不是问题; 在测试中的冗余是一个很好的事情。

                只要您的测试合理安排，就不会变得难以控制。良好的经验规则包括：

                TestClass每个模型或视图分开
                一个单独的测试方法，用于您要测试的每组条件
                描述其功能的测试方法名称

            进一步测试

                本教程仅介绍一些测试的基础知识。您可以做更多的工作，还有一些非常有用的工具可用于实现一些非常聪明的事情。

                例如，虽然我们在这里的测试已经涵盖了模型的一些内部逻辑以及我们的视图发布信息的方式，但您可以使用“浏览器”框架（如Selenium）来测试HTML在浏览器中的实际呈现方式。这些工具不仅可以检查Django代码的行为，还可以检查您的JavaScript。看到测试启动浏览器，并开始与您的网站进行交互，就好像人们正在开车一样！Django包括LiveServerTestCase 促进与Selenium等工具集成。

                如果您有一个复杂的应用程序，您可能希望自动运行测试，每次提交都是为了持续集成的目的，因此质量控制本身至少是部分自动化的。

                发现应用程序的未测试部分的一个好方法是检查代码覆盖。这也有助于识别脆弱甚至死亡代码。如果您无法测试一段代码，通常意味着代码应该重构或删除。覆盖将有助于识别死亡代码。有关详细信息，请参阅 与coverage.py集成。

                Django中的测试有关于测试的全面信息。


            下一步是什么？¶

                有关测试的详细信息，请参阅Django中的测试。
                https://docs.djangoproject.com/en/1.11/topics/testing/

                当您对测试Django视图感到满意时，请阅读 本教程的第6部分，了解静态文件管理。
                https://docs.djangoproject.com/en/1.11/intro/tutorial06/






