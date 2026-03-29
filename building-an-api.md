# Building an API
We have seen what a View is: a Python function or class that handles HTTPS requests and returns HTTP responses. It can return HTML, JSON; it can render templates, query databases...

One simple example:
```
# views.py
from django.shortcuts import render

def home(request):
    return render(request, 'home.html', {'title': 'Welcome'})
```
We can create a URL pattern which links the URL `/` to this view. The response from this view is a HTML page for a web browser.


An API (Application Programming Interface) is a way for programs to communicate. It is usually an endpoint in the application that returns structured data (like JSON or XML), instead of HTML. One simple example:

```
# views.py
from django.http import JsonResponse

def api_home(request):
    data = {'message': 'Hello API'}
    return JsonResponse(data)
```
We can create a URL pattern which links `/api/home/` to this view. The view's response is a JsonResponse:
```
{"message": "Hello API"}
```
which can be consumed by another program, not just a browser.

The way I see this, a View is a Python function which handles a request. When the response is structured data (such as JSON), which can be consumed by other applications and programs, then the view becomes an API endpoint. 

API endpoints are the specific locations within an API that accept and respond to request. Each endpoint thus, corresponds to a URL that may accept one or more HTTP methods (like `GET`, `POST`, `PUT`, or `DELETE`, which we will see later).

## The Django REST Framework (DRF)
There are different ways one can structure the endpoints of an API