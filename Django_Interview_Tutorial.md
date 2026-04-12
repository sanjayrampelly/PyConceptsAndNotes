# Django Interview Tutorial - Quick Recap Guide

## Table of Contents
1. [Django Basics](#django-basics)
2. [Models & ORM](#models--orm)
3. [Views](#views)
4. [Templates](#templates)
5. [URLs & Routing](#urls--routing)
6. [Forms](#forms)
7. [Admin Interface](#admin-interface)
8. [Authentication & Authorization](#authentication--authorization)
9. [Middleware](#middleware)
10. [Signals](#signals)
11. [QuerySets & Database Queries](#querysets--database-queries)
12. [Class-Based Views](#class-based-views)
13. [REST Framework (DRF)](#rest-framework-drf)
14. [Testing](#testing)
15. [Common Interview Questions](#common-interview-questions)

---

## Django Basics

### What is Django?
Django is a high-level Python web framework that encourages rapid development and clean, pragmatic design. It follows the MTV (Model-Template-View) pattern.

### Key Features
- **Batteries included**: Admin panel, ORM, authentication, forms
- **Secure**: Protection against SQL injection, XSS, CSRF
- **Scalable**: Used by Instagram, Pinterest, Mozilla
- **ORM**: Database abstraction layer
- **DRY principle**: Don't Repeat Yourself
- **MTV pattern**: Model-Template-View architecture

### Installation & Project Setup
```bash
# Install Django
pip install django

# Create project
django-admin startproject myproject
cd myproject

# Create app
python manage.py startapp myapp

# Run development server
python manage.py runserver

# Create migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser
```

### Project Structure
```
myproject/
├── myproject/          # Project configuration
│   ├── __init__.py
│   ├── settings.py     # Settings
│   ├── urls.py         # Root URL configuration
│   ├── asgi.py         # ASGI config
│   └── wsgi.py         # WSGI config
├── myapp/              # Application
│   ├── migrations/     # Database migrations
│   ├── __init__.py
│   ├── admin.py        # Admin configuration
│   ├── apps.py         # App configuration
│   ├── models.py       # Database models
│   ├── tests.py        # Tests
│   └── views.py        # View functions
└── manage.py           # Management script
```


### Interview Question 1: Explain Django's MTV architecture.

**Answer:**
Django follows **MTV (Model-Template-View)** pattern, which is similar to MVC but with different naming:

**Model:**
- Represents data structure
- Handles database operations
- Defines business logic
- Maps to database tables

**Template:**
- Presentation layer
- HTML with Django template language
- Displays data to users
- Equivalent to "View" in MVC

**View:**
- Business logic layer
- Processes requests
- Interacts with models
- Returns responses
- Equivalent to "Controller" in MVC

```python
# Model (models.py)
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    published_date = models.DateTimeField(auto_now_add=True)

# View (views.py)
from django.shortcuts import render
from .models import Article

def article_list(request):
    articles = Article.objects.all()
    return render(request, 'articles/list.html', {'articles': articles})

# Template (templates/articles/list.html)
# <html>
# {% for article in articles %}
#     <h2>{{ article.title }}</h2>
#     <p>{{ article.content }}</p>
# {% endfor %}
# </html>

# URL (urls.py)
from django.urls import path
from . import views

urlpatterns = [
    path('articles/', views.article_list, name='article_list'),
]
```

---

## Models & ORM

### Basic Model
```python
from django.db import models
from django.contrib.auth.models import User

class Category(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name_plural = "Categories"
        ordering = ['name']
    
    def __str__(self):
        return self.name

class Post(models.Model):
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    ]
    
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='posts')
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True)
    content = models.TextField()
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='draft')
    published_date = models.DateTimeField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    tags = models.ManyToManyField('Tag', blank=True)
    views = models.IntegerField(default=0)
    
    class Meta:
        ordering = ['-published_date']
        indexes = [
            models.Index(fields=['slug']),
            models.Index(fields=['-published_date']),
        ]
    
    def __str__(self):
        return self.title
    
    def get_absolute_url(self):
        from django.urls import reverse
        return reverse('post_detail', args=[self.slug])

class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    
    def __str__(self):
        return self.name
```

### Field Types
```python
from django.db import models

class Product(models.Model):
    # Text fields
    name = models.CharField(max_length=200)  # Short text
    description = models.TextField()  # Long text
    slug = models.SlugField(unique=True)  # URL-friendly
    
    # Numeric fields
    price = models.DecimalField(max_digits=10, decimal_places=2)
    quantity = models.IntegerField(default=0)
    rating = models.FloatField(default=0.0)
    
    # Boolean
    is_active = models.BooleanField(default=True)
    
    # Date/Time
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    published_date = models.DateField()
    
    # File fields
    image = models.ImageField(upload_to='products/')
    document = models.FileField(upload_to='documents/')
    
    # Relationships
    category = models.ForeignKey('Category', on_delete=models.CASCADE)
    tags = models.ManyToManyField('Tag')
    
    # JSON field (Django 3.1+)
    metadata = models.JSONField(default=dict)
    
    # Email, URL
    email = models.EmailField()
    website = models.URLField()
```

### Relationships
```python
# One-to-Many (ForeignKey)
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE, related_name='books')

# Many-to-Many
class Student(models.Model):
    name = models.CharField(max_length=100)
    courses = models.ManyToManyField('Course', related_name='students')

class Course(models.Model):
    name = models.CharField(max_length=100)

# Many-to-Many with Through Model
class Enrollment(models.Model):
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    course = models.ForeignKey(Course, on_delete=models.CASCADE)
    enrolled_date = models.DateField(auto_now_add=True)
    grade = models.CharField(max_length=2)
    
    class Meta:
        unique_together = ['student', 'course']

# One-to-One
class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='profile')
    bio = models.TextField()
    avatar = models.ImageField(upload_to='avatars/')
```

### Interview Question 2: Explain on_delete options in Django ForeignKey.

**Answer:**
`on_delete` defines what happens when the referenced object is deleted:

**CASCADE**: Delete related objects
```python
author = models.ForeignKey(Author, on_delete=models.CASCADE)
# If author is deleted, all their books are deleted
```

**PROTECT**: Prevent deletion
```python
category = models.ForeignKey(Category, on_delete=models.PROTECT)
# Cannot delete category if it has products
```

**SET_NULL**: Set to NULL
```python
editor = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
# If user is deleted, editor field becomes NULL
```

**SET_DEFAULT**: Set to default value
```python
status = models.ForeignKey(Status, on_delete=models.SET_DEFAULT, default=1)
# If status is deleted, set to default status
```

**SET()**: Set to specific value
```python
def get_default_user():
    return User.objects.get(username='deleted')

author = models.ForeignKey(User, on_delete=models.SET(get_default_user))
```

**DO_NOTHING**: Do nothing (can cause database errors)
```python
reference = models.ForeignKey(Reference, on_delete=models.DO_NOTHING)
# No action taken, may violate database constraints
```

---

## Views

### Function-Based Views (FBV)
```python
from django.shortcuts import render, get_object_or_404, redirect
from django.http import HttpResponse, JsonResponse
from .models import Post
from .forms import PostForm

# Simple view
def home(request):
    return HttpResponse("Hello, World!")

# Render template
def post_list(request):
    posts = Post.objects.filter(status='published')
    return render(request, 'blog/post_list.html', {'posts': posts})

# Detail view
def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug)
    return render(request, 'blog/post_detail.html', {'post': post})

# Create view
def post_create(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            return redirect('post_detail', slug=post.slug)
    else:
        form = PostForm()
    return render(request, 'blog/post_form.html', {'form': form})

# Update view
def post_update(request, slug):
    post = get_object_or_404(Post, slug=slug)
    if request.method == 'POST':
        form = PostForm(request.POST, instance=post)
        if form.is_valid():
            form.save()
            return redirect('post_detail', slug=post.slug)
    else:
        form = PostForm(instance=post)
    return render(request, 'blog/post_form.html', {'form': form, 'post': post})

# Delete view
def post_delete(request, slug):
    post = get_object_or_404(Post, slug=slug)
    if request.method == 'POST':
        post.delete()
        return redirect('post_list')
    return render(request, 'blog/post_confirm_delete.html', {'post': post})

# JSON response
def api_posts(request):
    posts = Post.objects.values('title', 'slug', 'published_date')
    return JsonResponse(list(posts), safe=False)
```

### Request and Response
```python
from django.http import HttpResponse, JsonResponse, HttpResponseRedirect
from django.shortcuts import redirect

def my_view(request):
    # Request attributes
    method = request.method  # GET, POST, PUT, DELETE
    path = request.path  # /blog/posts/
    user = request.user  # Current user
    
    # GET parameters
    page = request.GET.get('page', 1)
    search = request.GET.get('q', '')
    
    # POST data
    if request.method == 'POST':
        title = request.POST.get('title')
        content = request.POST.get('content')
    
    # Files
    if request.FILES:
        uploaded_file = request.FILES['file']
    
    # Headers
    user_agent = request.META.get('HTTP_USER_AGENT')
    
    # Cookies
    session_id = request.COOKIES.get('sessionid')
    
    # Session
    request.session['cart'] = []
    cart = request.session.get('cart', [])
    
    # Response types
    return HttpResponse("Text response")
    return JsonResponse({'key': 'value'})
    return redirect('post_list')
    return HttpResponseRedirect('/posts/')
```

### Decorators
```python
from django.contrib.auth.decorators import login_required, permission_required
from django.views.decorators.http import require_http_methods, require_POST
from django.views.decorators.cache import cache_page

# Require login
@login_required
def profile(request):
    return render(request, 'profile.html')

# Require specific permission
@permission_required('blog.add_post')
def create_post(request):
    pass

# Require specific HTTP methods
@require_http_methods(["GET", "POST"])
def my_view(request):
    pass

@require_POST
def delete_post(request, pk):
    pass

# Cache view for 15 minutes
@cache_page(60 * 15)
def post_list(request):
    posts = Post.objects.all()
    return render(request, 'posts.html', {'posts': posts})

# Custom decorator
from functools import wraps

def ajax_required(f):
    @wraps(f)
    def wrapper(request, *args, **kwargs):
        if not request.is_ajax():
            return HttpResponseBadRequest()
        return f(request, *args, **kwargs)
    return wrapper

@ajax_required
def ajax_view(request):
    return JsonResponse({'status': 'ok'})
```


### Interview Question 3: What's the difference between render(), redirect(), and HttpResponse()?

**Answer:**

**render():**
- Renders a template with context
- Returns HttpResponse with rendered HTML
- Used for displaying pages

```python
def post_list(request):
    posts = Post.objects.all()
    return render(request, 'posts.html', {'posts': posts})
```

**redirect():**
- Returns HttpResponseRedirect
- Redirects to another URL
- Used after form submission (POST-Redirect-GET pattern)
- Can use URL name, path, or object

```python
def create_post(request):
    # ... save post ...
    return redirect('post_detail', slug=post.slug)  # URL name
    return redirect('/posts/')  # Path
    return redirect(post)  # Object with get_absolute_url()
```

**HttpResponse():**
- Returns raw HTTP response
- Used for plain text, JSON, files
- Most basic response type

```python
def simple_view(request):
    return HttpResponse("Plain text")
    return HttpResponse("<h1>HTML</h1>", content_type="text/html")
```

**Comparison:**
```python
# render() - Display page
return render(request, 'template.html', context)

# redirect() - Navigate to another page
return redirect('url_name')

# HttpResponse() - Return raw content
return HttpResponse("Text")

# JsonResponse() - Return JSON
return JsonResponse({'key': 'value'})
```

---

## Templates

### Template Syntax
```django
{# templates/blog/post_list.html #}
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Blog{% endblock %}</title>
</head>
<body>
    {# Variables #}
    <h1>{{ page_title }}</h1>
    <p>{{ post.title }}</p>
    <p>{{ post.content|truncatewords:30 }}</p>
    
    {# Filters #}
    {{ post.title|upper }}
    {{ post.published_date|date:"Y-m-d" }}
    {{ post.content|safe }}
    {{ value|default:"N/A" }}
    
    {# Tags #}
    {% if user.is_authenticated %}
        <p>Welcome, {{ user.username }}!</p>
    {% else %}
        <p>Please log in.</p>
    {% endif %}
    
    {# For loop #}
    {% for post in posts %}
        <article>
            <h2>{{ post.title }}</h2>
            <p>{{ post.content }}</p>
        </article>
    {% empty %}
        <p>No posts available.</p>
    {% endfor %}
    
    {# URL tag #}
    <a href="{% url 'post_detail' slug=post.slug %}">Read more</a>
    
    {# Static files #}
    {% load static %}
    <img src="{% static 'images/logo.png' %}" alt="Logo">
    
    {# Include #}
    {% include 'partials/header.html' %}
    
    {# CSRF token #}
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Submit</button>
    </form>
    
    {# With tag #}
    {% with total=posts.count %}
        <p>Total posts: {{ total }}</p>
    {% endwith %}
</body>
</html>
```

### Template Inheritance
```django
{# templates/base.html #}
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
    {% block extra_css %}{% endblock %}
</head>
<body>
    <header>
        {% block header %}
            <h1>My Website</h1>
        {% endblock %}
    </header>
    
    <nav>
        {% block nav %}
            <a href="{% url 'home' %}">Home</a>
            <a href="{% url 'about' %}">About</a>
        {% endblock %}
    </nav>
    
    <main>
        {% block content %}{% endblock %}
    </main>
    
    <footer>
        {% block footer %}
            <p>&copy; 2024 My Site</p>
        {% endblock %}
    </footer>
    
    {% block extra_js %}{% endblock %}
</body>
</html>

{# templates/blog/post_list.html #}
{% extends 'base.html' %}

{% block title %}Blog Posts{% endblock %}

{% block content %}
    <h2>All Posts</h2>
    {% for post in posts %}
        <article>
            <h3>{{ post.title }}</h3>
            <p>{{ post.content|truncatewords:50 }}</p>
        </article>
    {% endfor %}
{% endblock %}
```

### Custom Template Tags and Filters
```python
# myapp/templatetags/custom_tags.py
from django import template

register = template.Library()

# Simple filter
@register.filter
def multiply(value, arg):
    return value * arg

# Filter with name
@register.filter(name='cut')
def cut_filter(value, arg):
    return value.replace(arg, '')

# Simple tag
@register.simple_tag
def current_time(format_string):
    from datetime import datetime
    return datetime.now().strftime(format_string)

# Inclusion tag
@register.inclusion_tag('tags/show_results.html')
def show_results(poll):
    choices = poll.choice_set.all()
    return {'choices': choices}

# Usage in template:
# {% load custom_tags %}
# {{ value|multiply:5 }}
# {% current_time "%Y-%m-%d %H:%M" %}
```

### Interview Question 4: Explain Django template inheritance and its benefits.

**Answer:**
**Template inheritance** allows you to build a base template with common elements and extend it in child templates.

**Benefits:**
1. **DRY principle**: Avoid repeating HTML
2. **Consistency**: Maintain consistent layout
3. **Maintainability**: Update once, reflect everywhere
4. **Flexibility**: Override specific blocks

```django
{# base.html - Parent template #}
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Default Title{% endblock %}</title>
    {% block extra_css %}{% endblock %}
</head>
<body>
    <header>
        {% block header %}
            <nav>Site Navigation</nav>
        {% endblock %}
    </header>
    
    <main>
        {% block content %}
            {# Child templates must provide content #}
        {% endblock %}
    </main>
    
    <footer>
        {% block footer %}
            <p>Copyright 2024</p>
        {% endblock %}
    </footer>
</body>
</html>

{# child.html - Child template #}
{% extends 'base.html' %}

{% block title %}Custom Page Title{% endblock %}

{% block content %}
    <h1>Page Content</h1>
    <p>This content replaces the content block.</p>
{% endblock %}

{# Can also extend parent block #}
{% block footer %}
    {{ block.super }}  {# Include parent content #}
    <p>Additional footer content</p>
{% endblock %}
```

**Multiple levels of inheritance:**
```django
base.html → blog_base.html → post_detail.html
```

---

## URLs & Routing

### URL Configuration
```python
# myproject/urls.py (Root URLconf)
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),
    path('api/', include('api.urls')),
    path('', include('home.urls')),
]

# Serve media files in development
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

# myapp/urls.py (App URLconf)
from django.urls import path, re_path
from . import views

app_name = 'blog'  # Namespace

urlpatterns = [
    # Basic path
    path('', views.post_list, name='post_list'),
    
    # Path with parameter
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
    path('post/<slug:slug>/', views.post_by_slug, name='post_by_slug'),
    
    # Multiple parameters
    path('category/<str:category>/year/<int:year>/', views.posts_by_category_year),
    
    # Regex path
    re_path(r'^archive/(?P<year>[0-9]{4})/$', views.archive),
    
    # Path converters
    path('user/<uuid:user_id>/', views.user_profile),
    path('article/<path:article_path>/', views.article),
]

# Path converters:
# str - matches any non-empty string (default)
# int - matches zero or positive integers
# slug - matches slug strings (letters, numbers, hyphens, underscores)
# uuid - matches UUID
# path - matches any string including slashes
```

### URL Reversing
```python
# In views
from django.urls import reverse
from django.shortcuts import redirect

def my_view(request):
    # Reverse by name
    url = reverse('blog:post_detail', args=[1])
    url = reverse('blog:post_detail', kwargs={'pk': 1})
    
    # Redirect using reverse
    return redirect('blog:post_list')
    return redirect(reverse('blog:post_detail', args=[post.pk]))

# In templates
# {% url 'blog:post_detail' pk=post.pk %}
# {% url 'blog:post_detail' post.pk %}

# In models
class Post(models.Model):
    # ...
    def get_absolute_url(self):
        return reverse('blog:post_detail', kwargs={'slug': self.slug})
```

### Interview Question 5: What are path converters in Django URLs?

**Answer:**
**Path converters** capture parts of the URL and convert them to Python types.

**Built-in converters:**

```python
from django.urls import path

urlpatterns = [
    # str - Any non-empty string (excluding '/')
    path('posts/<str:category>/', views.by_category),
    # URL: /posts/technology/
    # category = "technology" (string)
    
    # int - Zero or positive integer
    path('posts/<int:pk>/', views.post_detail),
    # URL: /posts/123/
    # pk = 123 (integer)
    
    # slug - Slug string (letters, numbers, hyphens, underscores)
    path('posts/<slug:slug>/', views.post_by_slug),
    # URL: /posts/my-first-post/
    # slug = "my-first-post" (string)
    
    # uuid - UUID string
    path('user/<uuid:user_id>/', views.user_profile),
    # URL: /user/550e8400-e29b-41d4-a716-446655440000/
    # user_id = UUID object
    
    # path - Any string including slashes
    path('files/<path:file_path>/', views.download),
    # URL: /files/documents/2024/report.pdf
    # file_path = "documents/2024/report.pdf" (string)
]
```

**Custom path converter:**
```python
# converters.py
class FourDigitYearConverter:
    regex = '[0-9]{4}'
    
    def to_python(self, value):
        return int(value)
    
    def to_url(self, value):
        return '%04d' % value

# urls.py
from django.urls import path, register_converter
from . import converters, views

register_converter(converters.FourDigitYearConverter, 'yyyy')

urlpatterns = [
    path('articles/<yyyy:year>/', views.year_archive),
    # URL: /articles/2024/
    # year = 2024 (integer, must be 4 digits)
]
```

---

## Forms

### Django Forms
```python
# forms.py
from django import forms
from .models import Post

class ContactForm(forms.Form):
    name = forms.CharField(
        max_length=100,
        widget=forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Your name'})
    )
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={'class': 'form-control'})
    )
    subject = forms.CharField(max_length=200)
    message = forms.CharField(
        widget=forms.Textarea(attrs={'rows': 5})
    )
    
    def clean_email(self):
        email = self.cleaned_data.get('email')
        if not email.endswith('@example.com'):
            raise forms.ValidationError("Email must be from example.com domain")
        return email
    
    def clean(self):
        cleaned_data = super().clean()
        name = cleaned_data.get('name')
        message = cleaned_data.get('message')
        
        if name and message and name.lower() in message.lower():
            raise forms.ValidationError("Message cannot contain your name")
        
        return cleaned_data

# ModelForm
class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'category', 'status', 'tags']
        # Or exclude fields
        # exclude = ['author', 'created_at']
        
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control'}),
            'content': forms.Textarea(attrs={'class': 'form-control', 'rows': 10}),
            'status': forms.Select(attrs={'class': 'form-select'}),
        }
        
        labels = {
            'title': 'Post Title',
            'content': 'Post Content',
        }
        
        help_texts = {
            'title': 'Enter a catchy title',
        }
    
    def clean_title(self):
        title = self.cleaned_data.get('title')
        if Post.objects.filter(title=title).exists():
            raise forms.ValidationError("A post with this title already exists")
        return title
```

### Form Handling in Views
```python
from django.shortcuts import render, redirect
from .forms import ContactForm, PostForm

# Function-based view
def contact_view(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            # Access cleaned data
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            subject = form.cleaned_data['subject']
            message = form.cleaned_data['message']
            
            # Process form (send email, save to database, etc.)
            send_email(name, email, subject, message)
            
            return redirect('success')
    else:
        form = ContactForm()
    
    return render(request, 'contact.html', {'form': form})

# ModelForm view
def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            form.save_m2m()  # Save many-to-many relationships
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()
    
    return render(request, 'post_form.html', {'form': form})

# Update view
def update_post(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES, instance=post)
        if form.is_valid():
            form.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)
    
    return render(request, 'post_form.html', {'form': form, 'post': post})
```

### Form Rendering in Templates
```django
{# Manual rendering #}
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    
    {# Render all fields #}
    {{ form.as_p }}
    {# Or {{ form.as_table }} or {{ form.as_ul }} #}
    
    <button type="submit">Submit</button>
</form>

{# Custom rendering #}
<form method="post">
    {% csrf_token %}
    
    {% if form.errors %}
        <div class="alert alert-danger">
            {{ form.errors }}
        </div>
    {% endif %}
    
    {% for field in form %}
        <div class="form-group">
            {{ field.label_tag }}
            {{ field }}
            {% if field.help_text %}
                <small>{{ field.help_text }}</small>
            {% endif %}
            {% if field.errors %}
                <div class="error">{{ field.errors }}</div>
            {% endif %}
        </div>
    {% endfor %}
    
    <button type="submit">Submit</button>
</form>
```


---

## Admin Interface

### Basic Admin Registration
```python
# admin.py
from django.contrib import admin
from .models import Post, Category, Tag

# Simple registration
admin.site.register(Category)
admin.site.register(Tag)

# Custom admin class
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'status', 'published_date', 'views']
    list_filter = ['status', 'created_at', 'published_date', 'author']
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}
    raw_id_fields = ['author']
    date_hierarchy = 'published_date'
    ordering = ['-published_date']
    
    # Fieldsets for organizing form
    fieldsets = (
        ('Basic Information', {
            'fields': ('title', 'slug', 'author', 'category')
        }),
        ('Content', {
            'fields': ('content', 'tags')
        }),
        ('Publication', {
            'fields': ('status', 'published_date'),
            'classes': ('collapse',)
        }),
    )
    
    # Actions
    actions = ['make_published', 'make_draft']
    
    def make_published(self, request, queryset):
        queryset.update(status='published')
    make_published.short_description = "Mark selected posts as published"
    
    def make_draft(self, request, queryset):
        queryset.update(status='draft')
    make_draft.short_description = "Mark selected posts as draft"
```

### Interview Question 6: How do you customize the Django admin interface?

**Answer:**
Django admin can be customized using ModelAdmin class:

```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    # List page customization
    list_display = ['title', 'author', 'status']  # Columns to display
    list_filter = ['status', 'created_at']  # Filter sidebar
    search_fields = ['title', 'content']  # Search functionality
    list_editable = ['status']  # Edit directly in list
    list_per_page = 25  # Pagination
    
    # Form customization
    fields = ['title', 'content', 'author']  # Field order
    readonly_fields = ['created_at', 'updated_at']  # Read-only fields
    prepopulated_fields = {'slug': ('title',)}  # Auto-populate
    
    # Fieldsets for grouping
    fieldsets = (
        ('Basic', {'fields': ('title', 'content')}),
        ('Advanced', {'fields': ('status',), 'classes': ('collapse',)}),
    )
    
    # Custom actions
    def make_published(self, request, queryset):
        queryset.update(status='published')
    actions = [make_published]
```

---

## Authentication & Authorization

### User Authentication
```python
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.decorators import login_required

# Login view
def login_view(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(request, username=username, password=password)
        
        if user is not None:
            login(request, user)
            return redirect('home')
    
    return render(request, 'login.html')

# Logout view
def logout_view(request):
    logout(request)
    return redirect('home')

# Protected view
@login_required
def profile_view(request):
    return render(request, 'profile.html')
```

### Interview Question 7: Explain Django's authentication system.

**Answer:**
Django's authentication system handles user accounts, groups, permissions, and sessions.

**Components:**

1. **User Model:**
```python
from django.contrib.auth.models import User
user = User.objects.create_user('john', 'john@example.com', 'password123')
```

2. **Login/Logout:**
```python
from django.contrib.auth import login, logout

user = authenticate(username=username, password=password)
if user:
    login(request, user)  # Creates session
```

3. **Permissions:**
```python
@permission_required('blog.add_post')
def create_post(request):
    pass
```

---

## Middleware

### Custom Middleware
```python
# middleware.py
class CustomMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Code before view
        print(f"Request: {request.path}")
        
        response = self.get_response(request)
        
        # Code after view
        print(f"Response: {response.status_code}")
        
        return response

# Add to settings.py
MIDDLEWARE = [
    # ...
    'myapp.middleware.CustomMiddleware',
]
```

### Interview Question 8: What is middleware in Django?

**Answer:**
**Middleware** processes requests globally before they reach views and responses before they're sent to clients.

**Use cases:**
- Authentication
- Logging
- CORS handling
- Rate limiting
- Request/response modification

---

## Signals

### Django Signals
```python
from django.db.models.signals import post_save
from django.dispatch import receiver

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        UserProfile.objects.create(user=instance)
```

### Interview Question 9: Explain Django signals.

**Answer:**
**Signals** allow decoupled applications to get notified when actions occur.

**Common signals:**
- `pre_save`, `post_save` - Before/after model.save()
- `pre_delete`, `post_delete` - Before/after model.delete()
- `user_logged_in`, `user_logged_out` - Authentication events

---

## QuerySets & Database Queries

### Basic Queries
```python
# Get all
posts = Post.objects.all()

# Filter
published = Post.objects.filter(status='published')

# Exclude
drafts = Post.objects.exclude(status='published')

# Get single object
post = Post.objects.get(id=1)

# Count
count = Post.objects.count()

# Exists
has_posts = Post.objects.filter(author=user).exists()
```

### Field Lookups
```python
# Exact
Post.objects.filter(title__exact='Hello')

# Contains
Post.objects.filter(title__contains='Django')

# Greater than
Post.objects.filter(views__gt=100)

# Date lookups
Post.objects.filter(created_at__year=2024)
```

### Interview Question 10: What's the difference between select_related() and prefetch_related()?

**Answer:**

**select_related():**
- For ForeignKey and OneToOneField
- Uses SQL JOIN
- Single query

```python
posts = Post.objects.select_related('author').all()
```

**prefetch_related():**
- For ManyToManyField and reverse ForeignKey
- Separate queries
- Python joins results

```python
posts = Post.objects.prefetch_related('tags').all()
```

---

## Class-Based Views

### Generic Views
```python
from django.views.generic import ListView, DetailView, CreateView

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'

class PostCreateView(CreateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
```

### Interview Question 11: Advantages of Class-Based Views?

**Answer:**
- Code reuse through inheritance
- Mixins for composing functionality
- Less boilerplate code
- Built-in pagination, forms handling

---

## REST Framework (DRF)

### Serializers
```python
from rest_framework import serializers

class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'author', 'published_date']
```

### ViewSets
```python
from rest_framework import viewsets

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

---

## Testing

### Unit Tests
```python
from django.test import TestCase
from .models import Post

class PostModelTest(TestCase):
    def setUp(self):
        Post.objects.create(title="Test Post", content="Content")
    
    def test_post_creation(self):
        post = Post.objects.get(title="Test Post")
        self.assertEqual(post.content, "Content")
```

### Interview Question 12: How do you test Django applications?

**Answer:**
Django provides TestCase class for testing:

```python
from django.test import TestCase, Client

class ViewTest(TestCase):
    def test_post_list_view(self):
        response = self.client.get('/posts/')
        self.assertEqual(response.status_code, 200)
```

---

## Common Interview Questions

### Question 13: What is Django ORM?

**Answer:**
Django ORM (Object-Relational Mapping) allows you to interact with databases using Python code instead of SQL.

### Question 14: Explain CSRF protection in Django.

**Answer:**
CSRF (Cross-Site Request Forgery) protection prevents malicious websites from executing actions on behalf of authenticated users. Django uses CSRF tokens in forms.

### Question 15: What are Django migrations?

**Answer:**
Migrations are Django's way of propagating model changes to the database schema.

```bash
python manage.py makemigrations  # Create migrations
python manage.py migrate  # Apply migrations
```

---

## Quick Tips for Django Interviews

1. Understand MTV architecture
2. Master Django ORM and QuerySets
3. Know the difference between FBV and CBV
4. Understand Django's security features
5. Be familiar with Django REST Framework
6. Know how to optimize database queries
7. Understand middleware and signals
8. Practice with Django admin customization

---

## Conclusion

This tutorial covers essential Django concepts for interview preparation. Django is a powerful framework that follows best practices and provides everything needed for web development.

**Key Takeaways:**
- Django follows MTV pattern
- ORM provides database abstraction
- Built-in admin interface
- Strong security features
- Extensive ecosystem
- Scalable and maintainable

Good luck with your Django interview! 🚀
