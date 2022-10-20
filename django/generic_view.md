## Generic Views

#### class based views

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

#### Handling Forms with class based views

