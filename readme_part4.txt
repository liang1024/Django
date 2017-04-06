
4.编写您的第一个Django应用程序-4
    第4部分 :https://docs.djangoproject.com/en/1.11/intro/tutorial04/

    本教程从教程3中断开始。我们正在继续Web-poll应用程序，并将重点介绍简单的表单处理和减少代码。

   1.写一个简单的形式

    我们从最后一个教程中更新我们的投票详细信息模板（“polls / detail.html”），以便模板包含一个HTML <form>元素

    polls/templates/polls/detail.html

        <h1>{{ question.question_text }}</h1>

        {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

        <form action="{% url 'polls:vote' question.id %}" method="post">
        {% csrf_token %}
        {% for choice in question.choice_set.all %}
            <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
            <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
        {% endfor %}
        <input type="submit" value="Vote" />
        </form>

        快速破解：

            上述模板显示每个问题选择的单选按钮。的 value每个单选按钮的是相关联的问题的选择的ID。的 name每个单选按钮的是"choice"。这意味着，当有人选择其中一个单选按钮并提交表单时，它将发送POST数据choice=#，其中＃是所选择的选项的ID。这是HTML表单的基本概念。
            我们设置窗体的action到，我们设置。使用（而不是 ）是非常重要的，因为提交此表单的行为将改变数据服务器端。无论何时创建一个改变数据服务器端的表单，请使用。这个提示不是Django的特有的; 这只是很好的Web开发实践。{% url 'polls:vote' question.id %}method="post"method="post"method="get"method="post"
            forloop.counter表示for标签经过其循环的次数
            由于我们正在创建POST表单（可能会影响修改数据），所以我们需要担心Cross Site Request Forgeries。幸运的是，你不用担心太多，因为Django带有一个非常易于使用的系统来防范它。简而言之，针对内部网址的所有POST表单都应使用 模板标签。{% csrf_token %}
            现在，我们来创建一个处理提交的数据的Django视图，并用它来处理。请记住，在教程3中，我们为包含此行的投票应用程序创建了一个URLconf：

        polls/urls.py
            url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),

        我们还创建了一个虚vote()函数的实现。让我们创建一个真正的版本。将以下内容添加到polls/views.py：

        polls/views.py
            from django.shortcuts import get_object_or_404, render
            from django.http import HttpResponseRedirect, HttpResponse
            from django.urls import reverse

            from .models import Choice, Question
            # ...
            def vote(request, question_id):
                question = get_object_or_404(Question, pk=question_id)
                try:
                    selected_choice = question.choice_set.get(pk=request.POST['choice'])
                except (KeyError, Choice.DoesNotExist):
                    # Redisplay the question voting form.
                    return render(request, 'polls/detail.html', {
                        'question': question,
                        'error_message': "You didn't select a choice.",
                    })
                else:
                    selected_choice.votes += 1
                    selected_choice.save()
                    # Always return an HttpResponseRedirect after successfully dealing
                    # with POST data. This prevents data from being posted twice if a
                    # user hits the Back button.
                    return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))

            这段代码包括我们在本教程中还没有介绍的几件事情：

            request.POST是一个类似字典的对象，可让您通过键名访问提交的数据。在这种情况下， request.POST['choice']返回所选择的选项的ID作为字符串。request.POST值总是字符串。

            请注意，Django还提供request.GET以相同的方式访问GET数据 - 但是我们request.POST在代码中明确使用，以确保仅通过POST调用更改数据。

            request.POST['choice']KeyError如果 choicePOST数据中没有提供，将会提高。上述代码检查 KeyError并重新显示带有错误消息的问题表单，如果choice没有给出。

            递增选择计数后，代码返回 HttpResponseRedirect而不是正常 HttpResponse。 HttpResponseRedirect采用一个参数：用户将被重定向到的URL（在这种情况下，我们如何构造URL，请参阅以下内容）。

            正如上面的Python注释所指出的那样，您应该始终HttpResponseRedirect在成功处理POST数据后返回 。这个提示不是Django的特有的; 这只是很好的Web开发实践。

            我们在这个例子reverse()中使用HttpResponseRedirect构造函数中的函数 。此功能有助于避免在视图功能中对URL进行硬编码。给定我们要将控件传递给的视图的名称以及指向该视图的URL模式的变量部分。
            在这种情况下，使用我们在教程3中设置的URLconf ，此reverse()调用将返回一个字符串

            '/polls/3/results/'

            那里的3价值在哪里question.id。然后，此重定向网址将调用'results'视图显示最终页面。

            如教程3request所述，是一个 HttpRequest对象。有关HttpRequest对象的更多信息 ，请参阅请求和响应文档。

            在有人投票的问题后，该vote()视图重定向到结果页面的问题。我们来写这个观点：

            polls/views.py

            from django.shortcuts import get_object_or_404, render

            def results(request, question_id):
                question = get_object_or_404(Question, pk=question_id)
                return render(request, 'polls/results.html', {'question': question})

            这detail()与教程3的视图几乎完全相同。唯一的区别就是模板名称。我们稍后会修复这个冗余。
            现在，创建一个polls/results.html模板：

            polls/templates/polls/results.html
            <h1>{{ question.question_text }}</h1>

                <ul>
                {% for choice in question.choice_set.all %}
                    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
                {% endfor %}
                </ul>

                <a href="{% url 'polls:detail' question.id %}">Vote again?</a>

                现在，请/polls/1/在浏览器中进行投票。您应该会看到一个结果页面，每次投票时都会更新。如果您提交表单没有选择，您应该看到错误消息。

                注意
                我们vote()认为的代码确实有一个小问题。它首先selected_choice从数据库获取对象，然后计算新值votes，然后将其保存回数据库。如果您的网站的两个用户尝试在完全相同的时间投票，这可能会出错：相同的值，就是42，将被检索votes。然后，对于两个用户，计算并保存新值43，但是44将是预期值。
                这被称为比赛条件。如果您有兴趣，可以阅读 使用F（）避免竞争条件，以了解如何解决此问题。

         使用通用视图：较少的代码更好
                在detail()（从教程3）和results() 意见是非常简单的-并且如上面提到的，冗余的。该index() 视图，显示民意调查的名单，是相似的。

                这些视图代表基本Web开发的常见情况：根据URL中传递的参数从数据库中获取数据，加载模板并返回渲染的模板。因为这很常见，Django提供了一个称为“通用视图”系统的快捷方式。

                通用视图将常见的模式抽象出来，甚至不需要编写Python代码来编写应用程序。

                我们将我们的投票应用转换为使用通用视图系统，因此我们可以删除一堆我们自己的代码。我们只需要采取几个步骤进行转换。我们会：

                转换URLconf。
                删除一些旧的，不需要的视图。
                介绍基于Django通用视图的新视图。
                请继续阅读。

                为什么代码洗牌？
                通常，在编写Django应用程序时，您将评估通用视图是否适合您的问题，您将从头开始使用它们，而不是重写代码。但是，本教程有意将焦点放在核心概念上，重点将写作的观点写到“硬”之中。
                在开始使用计算器之前，您应该了解基础数学。

                修改URL配置
                首先，打开polls/urls.py  URLconf并改变它：

                polls/urls.py

                from django.conf.urls import url

                    from . import views

                    app_name = 'polls'
                    urlpatterns = [
                        url(r'^$', views.IndexView.as_view(), name='index'),
                        url(r'^(?P<pk>[0-9]+)/$', views.DetailView.as_view(), name='detail'),
                        url(r'^(?P<pk>[0-9]+)/results/$', views.ResultsView.as_view(), name='results'),
                        url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
                    ]

                请注意，第二和第三模式的正则表达式中匹配模式的名称已从更改<question_id>为<pk>。

                修改视图

                接下来，我们将删除我们的老index，detail和results 视图，并使用Django的通用视图代替。要这样做，打开 polls/views.py文件并更改它：

                polls/views.py

                from django.shortcuts import get_object_or_404, render
                from django.http import HttpResponseRedirect
                from django.urls import reverse
                from django.views import generic

                from .models import Choice, Question


                class IndexView(generic.ListView):
                    template_name = 'polls/index.html'
                    context_object_name = 'latest_question_list'

                    def get_queryset(self):
                        """Return the last five published questions."""
                        return Question.objects.order_by('-pub_date')[:5]


                class DetailView(generic.DetailView):
                    model = Question
                    template_name = 'polls/detail.html'


                class ResultsView(generic.DetailView):
                    model = Question
                    template_name = 'polls/results.html'


                def vote(request, question_id):
                    ... # same as above, no changes needed.

                我们在这里使用两个通用视图： ListView和 DetailView。这两个视图分别提取了“显示对象列表”和“显示特定类型对象的详细信息页面”的概念。

                每个通用视图都需要知道它将采取什么样的模式。这是使用model属性提供的。
                该DetailView通用视图预计从URL中捕获的主键值被调用 "pk"，所以我们已经改变question_id，以pk用于通用视图。
                默认情况下，DetailView通用视图使用一个名为。在我们的例子中，它将使用模板。该 属性用于告诉Django使用特定的模板名称，而不是自动生成的默认模板名称。我们还为列表视图指定- 这确保结果视图和细节视图在呈现时具有不同的外观，即使它们都是 幕后的。<app name>/<model name>_detail.html"polls/question_detail.html"template_nametemplate_nameresultsDetailView

                同样，ListView通用视图使用一个叫做默认模板; 我们用来告诉 我们使用我们现有的 模板。<app name>/<model name>_list.htmltemplate_nameListView"polls/index.html"

                在本教程的前面部分，模板已提供了包含一个上下文question和latest_question_list 上下文变量。对于DetailView该question自动提供的变量-因为我们使用的Django模型（Question），Django的是能够确定一个适当的名称为上下文变量。但是，对于ListView，自动生成的上下文变量是 question_list。要覆盖这个，我们提供context_object_name 属性，指定我们要latest_question_list改用。作为一种替代方法，您可以更改模板以匹配新的默认上下文变量 - 但是让Django使用所需的变量要容易得多。

                运行服务器，并根据通用视图使用新的投票应用程序。

                有关通用视图的完整详细信息，请参阅通用视图文档。

                当您对表单和通用视图感到满意时，请阅读本教程的第5部分，了解如何测试我们的投票应用程序。

                https://docs.djangoproject.com/en/1.11/intro/tutorial05/








