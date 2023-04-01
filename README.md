```python
INSTALLED_APPS = [
    'app',
    'rest_framework'
]
```

```python
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=100)

    def __str__(self):
        return self.title
```


```python
from django.contrib import admin
from .models import Book

admin.site.register(Book)
```

## app/serializers.py
```python
from .models import Book
from rest_framework import serializers


class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model=Book
        fields='__all__'
```


```bash
python manage.py makemigrations
python manage.py migrate
```


```python

from django.contrib import admin
from django.urls import path
from app.views import BookAPIView
urlpatterns = [
    path('admin/', admin.site.urls),
    path('books/', BookAPIView.as_view(), name='book-list'),
    path('books/<int:pk>/', BookAPIView.as_view(), name='book-detail'),

    
]


```



```python


from rest_framework import generics
from rest_framework.response import Response
from .models import Book
from .serializers import BookSerializer
from django.http import HttpResponse





class BookAPIView(generics.GenericAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    
    
    def get(self, request, *args, **kwargs):
        if 'pk' in kwargs:
            try:
                book = self.get_object()
                serializer = self.get_serializer(book)
                return Response(serializer.data)
            except Book.DoesNotExist:
                return Response('Book does not exist')
        else:
            books = self.get_queryset()
            serializer = self.get_serializer(books, many=True)
            return Response(serializer.data)

   
    
        
    def post(self, request):
        try:

            serializer = self.get_serializer(data=request.data)
            serializer.is_valid(raise_exception=True)
            serializer.save()
            return Response(serializer.data)
        except:
            return Response('Either Id is wrong or Method is Wrong')
    
    def put(self, request, pk):
        try:     
            book = Book.objects.get(pk=pk)
            serializer = BookSerializer(book, data=request.data)
            serializer.is_valid(raise_exception=True)
            serializer.save()
            return Response(serializer.data)
        except Book.DoesNotExist:
            return Response('Book doenst exist')
    
    def delete(self, request, pk):
        try:
            book = Book.objects.get(pk=pk)
            book.delete()
            return Response('Succefully deleted')
        except:
            return Response("Book doesnt Exist")
    

```
