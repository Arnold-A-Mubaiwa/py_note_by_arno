## Generic Views

### class based views

a class-based view allows you to respond to different HTTP 
request methods with different class instance methods, 
instead of with conditionally branching code inside a single 
view function.

``` 
from django.http import HttpResponse

def my_view(request):
    if request.method == 'GET':
        return HttpResponse('result')
```

is the same as

```
from django.http import HttpResponse
from django.views import View

class MyView(View):
    def get(self, request):
        return HttpResponse('result')
```
in the urls .py you then call the class base view as

```
from django.urls import path
from myapp.views import MyView

urlpatterns = [
    path('about/', MyView.as_view()),
]
```

### Handling Forms with class based views

A basic function-based view that handles forms may look like this

```
from django.http import HttpResponseRedirect
from django.shortcuts import render

from .forms import MyForm

def myview(request):
    if (request.method == 'POST'):
        form = MyForm(request.POST)
        if (form.is_valid()):
            return HttpResponseRedirect('/Success/')
    else:
        form = MyForm(initial={'key':'value'})

    return render(request, 'form_tamplate.html',{'form':form})
```

is the same as class-based view

```
from django.http import HttpResponseRedirect
from django.shortcut import render
from django.views import View
from .forms import MyForm

class MyFormView(View):
    form_class = MyForm
    initial = {'Key': 'value'}
    template_name = 'form_template.html'

    def get(self, request, *args, **kwargs):
        form = self.form_class(initial=self.initial)
        return render (request, self.template_name, {'form':form})

    def post(self, request, *args, **kwargs):
        form = self.form_class(request.POST)
        if form.is_valid():
            return HttpResponseRedirect('/Success/')
        return render(request, self.template_name, {'form':form})
```

### Decoratin in URLconf

You can adjust class-based views by decorating the result of the as_view() method. The easiest place to do this is in the URLconf where you deploy your view:

```
from django.contrib.auth.decorators import login_required, permission_required
from django.views.generic import TemplateView
from .views import VoteView

urlpatterns = [
    path('about/', login_required(TemplateView(template_name='secret.html'))),
    path('vote/', permission_required('polls.can_vote')(VoteView.as_view())),
]
```
This approach applies the decorator on a per-instance basis. If you want every instance of a view to be decorated, you need to take a different approach.

### Decorating the class

To decorate every instance of a class-based view, you need to decorate the class definition itself. To do this you apply the decorator to the dispatch() method of the class. You do that by transforming it using the @method_decorator() decorator

```
from django.contrib.auth.decorators import login_required
from django.utils.decorators import method_decorator
from django.views.generic import TemplateView

class ProtectedView(TemplateView):
    template_name = 'secret.html'

    @method_decorator(login_required)
    def dispatch(self, *args, **kwargs):
        return super().dispatch(*args, **kwargs)

```

Or, more succinctly, you can decorate the class instead and pass the name of the method to be decorated as the keyword argument name:

```
from django.contrib.auth.decorators import login_required
from django.utils.decorators import method_decorator
from django.views.generic import TamplateView

@method_decorator(login_required, name='dispatch')
class ProtectedView(TemplateView):
    template_name = 'secret.html'
```

If you have a set of common decorators used in several places, you can define a list or tuple of decorators and use this instead of invoking method_decorator() multiple times. These two classes are equivalent:

```
decorators = [never_cache, login_required]
@method_decorator(decorators, name='dispatch')
class ProtectedView(TemplateView):
    template_name = 'secret.html'
```

The decorators will process a request in the order they are passed to the decorator. In the example, never_cache() will process the request before login_required().

