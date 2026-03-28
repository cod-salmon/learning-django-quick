
# Retrieving objects from models
Once we have our model properly linked to a database table, we can start thinking about how to access its database table to either modify it or retrieve data from it. We can do this programmatically using Django's **object-relational mapper (ORM)**.

The Django ORM is a powerful database abstraction API that lets you create, retrieve, update and delete objects from the database "easily" (or rather, using Python classes and methods instead of SQL queries). It basically works as a Pythonic interface to interact with your database. It is compatible with MySQL, PostgreSQL, SQLite, Oracle and MariaDB.

To understand the Django ORM is to udnerstand two things: **managers** and **QuerySets**. 

As expected from its name, a QuerySet represents a collection of database queries used to retrieve objects from your database. QuerySets are returned by model managers, which act as the interface between your models and the database.

Every Django model comes with a default manager called `objects` (e.g., `User.objects`, `Post.objects`). This manager provides access to all records in the model’s database table.

In practice, however, you’ll often want to retrieve only a subset of those records, extract specific data, or control the order in which results are returned. You can do this by applying various methods to QuerySets. Below are some common examples.

## Playing with QuerySets
### Filtering QuerySets - examples
- `Post.objects.filter(id__exact=1)`: Returns a QuerySet with the objects which `id` field exactly matches the number 1.
- `Post.objects.filter(title__iexact='who was django reinhardt?'))`: Returns a QuerySet with the objects which `title` field exactly match the string "who was django reinhardt?" (ignoring cases, *case-insensitive*)
- `Post.objects.filter(title__contains='Django')`: Returns a QuerySet with the objects which `title` contains the exact word "Django".
- `Post.objects.filter(title__icontains='django')`: Returns a QuerySet with the objects which `title` contains the word "Django" (ignoring case, *case-insensitive*).
- `Post.objects.filter(id__in=[1, 3])`: Returns a QuerySet with the objects which `id` value is either 1 or 3.
- `Post.objects.filter(id__gt=3)`: Returns a QuerySet with the objects which `id` value is greater than 3.
- `Post.objects.filter(id__gte=3)`: Returns a QuerySet with all the objects which `id` field is greater or equal to 3.
- `Post.objects.filter(id__lt=3)`: Returns a QuerySet with all the objects which `id` field is less than 3.
- ` Post.objects.filter(id__lte=3`: Returns a QuerySet with all the objects which `id` field is less than or equal to 3.
- `Post.objects.filter(title__istartswith='who'`: Returns a QuerySet with all the objects which `title` field starts with the string "word" (ignoring cases).
- `Post.objects.filter(title__iendswith='reinhardt?')`: Returns a QuerySet with all the objects which `title` field ends with the word "reinhardt" (ignoring cases).
- `Post.objects.filter(publish__year=2024)`: Returns a QuerySet with all the objects which `publish` field's `year` field exactly matches 2024.
- `Post.objects.filter(publish__month=1`: Returns a QuerySet with all the objects which `publish` field's `month` field exactly matches 1 (January).
- `Post.objects.filter(author__username='admin'`: Returns a QuerySet with all the objects which `author`'s `username` exactly equals `admin`.
- `Post.objects.filter(author__username__starstwith='ad'`: Returns a QuerySet with all the objects which `author`'s `username` start with the exact word "ad".
- ` Post.objects.filter(publish__year=2024, author__username='admin')`: Returns a QuerySet with all the objects which `publish`'s `year` field exactly matches 2024 and which `author`'s `username` field matches "admin". 
- `Post.objects.filter(publish__year=2024).filter(author__username='admin')`: the same thing as before but chaining filter queries.

### Excluding QuerySets - examples
- `Post.objects.filter(publish__year=2024).exclude(title__startswith='Why')`: Returns a QuerySet with all objects which `publish`'s `year` matches 2024, excluding the objects which `title` field starts with the word "Why".

### Ordering QuerySets - examples
- `Post.objects.order_by('title')`: Returns a QuerySet with all the objects ordered by `title` field, in ascending order.
- `Post.objects.order_by('-title')`: Returns a QuerySet with all the objects ordered by `title` field, in descending order.
- `Post.objects.order_by('author', 'title')`: Returns a QuerySet with all the objects ordered by `author` , then by `title` field, in ascending order.

### Limiting QuerySets - examples
- `Post.objects.all()[:5]`: Limit the return QuerySet to the first 5 objects.
- `Post.objects.all()[3:6]`: Limit the return QuerySet to the objects between the 3rd position and the 6th position.

### Counting QuerySets - examples
- `Post.objects.filter(id_lt=3).count()`: Returns the number of objects in the returning QuerySet (the number of objects which `id` field is less than 3).

### Deleting QuerySets - examples
- `Post.objects.get(id=1).delete()`: Deletes all objects from the datatable which `id` field matches the number 1.

### Complex lookups with Q objects
When you do 
```
Post.objects.filter(publish__year=2024).filter(author__username='admin')
```
Django combines these conditions using an AND operator in the underlying SQL query.

To build more complex queries -such as those involving OR conditions- you can use `Q` objects. For example:
```
from django.db.models import Q

starts_who = Q(title__istartswith='who')
starts_why = Q(title__istartswith='why')

Post.objects.filter(starts_who | starts_why)
```
## Creating model managers
As mentioned before, the default manager for every model is the `objects` manager. This manager retrieves all objects in the database. However, we can define custom managers for models. For example, a custom manager to retrieve all posts that have a `PUBLISHED` status. To create a new manager, you add a new class to your `blog/models.py` file:
```
class PublishedManager(models.Manager):
    def get_queryset(self):
        return (
            super().get_queryset().filter(status=Post.Status.PUBLISHED)
        )
```
Note in the above the `PublishedManager` is a subclass of `models.Manager` (i.e., not `models.Model`). When `Post.published.all()` is called, we basically call `Post.objects.filter(status=Post.Status.PUBLISHED)`. Note we also need to include the new manager inside the `Post` class:

```
class Post(models.Model):
    # model fields
    # ...
    objects = models.Manager() # The default manager.
    published = PublishedManager() # Our custom manager.

    class Meta:
        ordering = ['-publish']
        indexes = [
            models.Index(fields=['-publish']),
        ]
    
    def __str__(self):
        return self.title

```

The first manager declared in a model becomes the default manager. You can use the Meta attribute default_manager_name to specify a different default manager. If no manager is defined in the model, Django automatically creates the objects default manager for it. If you declare any managers for your model but you want to keep the objects manager as well, you have to add it explicitly to your model.