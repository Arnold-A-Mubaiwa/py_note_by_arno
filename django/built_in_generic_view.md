# Built in generic views

### Generic views ass objects

Django comes with a handful of built-in generic views to help generate list and detail views of objects.

Using the following model:

```
from django.db import models

class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=50)
    city = models.CharField(max_length=60)
    state_province = models.CharField(max_length=30)
    country = models.CharField(max_length=50)
    website = models.URLField()

    class Meta:
        ordering = ["-name"]

    def __str__(self):
        return self.name

class Author(models.Model):
    salutation = models.CharField(max_length=10)
    name = models.CharField(max_length=200)
    email = models.EmailField()
    headshot = models.ImageField(upload_to='author_headshots')

    def __str__(self):
        return self.name

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField('Author')
    publisher = models.ForeignKey(Publisher, on_delete=models.CASCADE)
    publication_date = models.DateField()
```

To list publisher you define the view as follows

```
from django.views.generic import ListView
from books.models import Publisher

class PublisherListView(ListView):
    model = Publisher
```

Then you hook the view to the urls

```
from django.urls import path
from books.views import PublisherListView

urlpatterns = [
    path('publishers/', PublisherListView.as_view()),
]
```

Because we didn't specify the template_name in the view class Django will infer one which is publisher_list.html that should be in the books template dir

```
/path/to/project/books/templates/books/publisher_list.html
```

This template will be rendered against a context containing a variable called object_list that contains all the publisher objects. A template might look like this:

```
{% extend "base.html" %}

{% block content %}
    <h2>Publishers</h2>
    <ul>
        {% for publisher in object_list %}
            <li>{{ publisher }}</li>
        {% endfor %}
    </ul>
{% endblock %}
```

Instead of using object_list you can rename the object list to suit fit the Model

```
from django.views.generic import ListView
from books.models import Publishers

class PublisherListView(ListView):
    model = Publisher
    context_object_name = 'publisher_list'
```
Providing a useful context_object_name is always a good idea.

### Adding extra context

To add extra infomation to the DetailView than provided by generic view, you can provide ypur own implementations to the get_context_data() method.  The default implementation adds the object being displayed to the template, but you can override it to send more:

```
form django.views.generic import DetailView
from .models import Book, Publisher

class PublisherDetailView(DetailView):
    model = Publisher

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['book_list'] = Book.objects.all()
        return context
```

### Viewing subsets of objects
 The model argument is not the only way to specify the objects that the view will operate upon – you can also specify the list of objects using the queryset argument:

 using query set:

 ```
 from django.views.generic import DetailView
 from .models import Publisher

 class PublisherDetailView(DetailView):
    context_object_name = 'publisher'
    queryset = Publisher.objects.all()
```

to order your list you can use

```
queryset = Book.objects.order_by('-publication_date')
```

 If you want to present a list of books by a particular publisher, you can use the same technique: Note that along with a filtered queryset, we’re also using a custom template name.

```
 queryset = Book.objects.filter(publisher__name='ACME Publishing')
    template_name = 'books/acme_list.html'
```

### Dynamic Filtering
The common need is to filter down the objects given in a list page by some key in the URL
