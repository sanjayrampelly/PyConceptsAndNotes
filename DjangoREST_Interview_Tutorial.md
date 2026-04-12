# Django REST Framework (DRF) Interview Tutorial - Quick Recap Guide

## Table of Contents
1. [DRF Basics](#drf-basics)
2. [Serializers](#serializers)
3. [Views & ViewSets](#views--viewsets)
4. [Routers](#routers)
5. [Authentication](#authentication)
6. [Permissions](#permissions)
7. [Pagination](#pagination)
8. [Filtering & Search](#filtering--search)
9. [Throttling](#throttling)
10. [Versioning](#versioning)
11. [Testing](#testing)
12. [Common Interview Questions](#common-interview-questions)

---

## DRF Basics

### What is Django REST Framework?
Django REST Framework (DRF) is a powerful toolkit for building Web APIs in Django. It provides features like serialization, authentication, permissions, and browsable API.

### Key Features
- **Serialization**: Convert complex data to JSON/XML
- **Authentication**: Token, Session, JWT, OAuth
- **Permissions**: Fine-grained access control
- **Browsable API**: Web interface for testing
- **ViewSets**: Combine logic for CRUD operations
- **Routers**: Automatic URL routing

### Installation & Setup
```bash
pip install djangorestframework
```

```python
# settings.py
INSTALLED_APPS = [
    ...
    'rest_framework',
]

REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
    ],
}
```

### Basic API Example
```python
# models.py
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=100)
    isbn = models.CharField(max_length=13, unique=True)
    published_date = models.DateField()
    price = models.DecimalField(max_digits=6, decimal_places=2)
    
    def __str__(self):
        return self.title

# serializers.py
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'isbn', 'published_date', 'price']

# views.py
from rest_framework import viewsets
from .models import Book
from .serializers import BookSerializer

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

# urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import BookViewSet

router = DefaultRouter()
router.register(r'books', BookViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]
```


### Interview Question 1: What is Django REST Framework and why use it?

**Answer:**
**Django REST Framework (DRF)** is a powerful toolkit for building Web APIs in Django.

**Why use DRF:**

1. **Serialization**: Automatic conversion between complex types and JSON
2. **Authentication**: Built-in support for multiple auth methods
3. **Permissions**: Granular access control
4. **Browsable API**: Interactive web interface for testing
5. **Validation**: Automatic request validation
6. **Documentation**: Auto-generated API documentation
7. **ViewSets & Routers**: Reduce boilerplate code
8. **Throttling**: Rate limiting built-in

**Comparison with plain Django:**

```python
# Plain Django - Manual JSON handling
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
import json

@csrf_exempt
def book_list(request):
    if request.method == 'GET':
        books = Book.objects.all().values()
        return JsonResponse(list(books), safe=False)
    elif request.method == 'POST':
        data = json.loads(request.body)
        book = Book.objects.create(**data)
        return JsonResponse({'id': book.id})

# DRF - Automatic handling
from rest_framework import viewsets

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    # Automatic GET, POST, PUT, PATCH, DELETE
```

---

## Serializers

### ModelSerializer
```python
from rest_framework import serializers
from .models import Book, Author, Review

# Basic ModelSerializer
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'isbn', 'price']
        # Or use '__all__' for all fields
        # fields = '__all__'
        # Or exclude specific fields
        # exclude = ['created_at']

# Read-only and write-only fields
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'isbn', 'price', 'created_at']
        read_only_fields = ['id', 'created_at']
        extra_kwargs = {
            'isbn': {'write_only': True}
        }

# Custom field
class BookSerializer(serializers.ModelSerializer):
    author_name = serializers.CharField(source='author.name', read_only=True)
    days_since_published = serializers.SerializerMethodField()
    
    class Meta:
        model = Book
        fields = ['id', 'title', 'author_name', 'days_since_published']
    
    def get_days_since_published(self, obj):
        from datetime import date
        return (date.today() - obj.published_date).days
```

### Nested Serializers
```python
class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Author
        fields = ['id', 'name', 'email']

class ReviewSerializer(serializers.ModelSerializer):
    class Meta:
        model = Review
        fields = ['id', 'rating', 'comment', 'created_at']

class BookDetailSerializer(serializers.ModelSerializer):
    author = AuthorSerializer(read_only=True)
    reviews = ReviewSerializer(many=True, read_only=True)
    review_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'reviews', 'review_count', 'price']
    
    def get_review_count(self, obj):
        return obj.reviews.count()
```

### Validation
```python
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'isbn', 'price']
    
    # Field-level validation
    def validate_price(self, value):
        if value < 0:
            raise serializers.ValidationError("Price cannot be negative")
        if value > 10000:
            raise serializers.ValidationError("Price too high")
        return value
    
    def validate_isbn(self, value):
        if len(value) != 13:
            raise serializers.ValidationError("ISBN must be 13 characters")
        return value
    
    # Object-level validation
    def validate(self, data):
        if data.get('price', 0) > 1000 and not data.get('author'):
            raise serializers.ValidationError(
                "Expensive books must have an author"
            )
        return data
    
    # Custom create
    def create(self, validated_data):
        # Custom logic before creating
        book = Book.objects.create(**validated_data)
        # Send notification, log, etc.
        return book
    
    # Custom update
    def update(self, instance, validated_data):
        instance.title = validated_data.get('title', instance.title)
        instance.price = validated_data.get('price', instance.price)
        instance.save()
        return instance
```

### Serializer vs ModelSerializer
```python
# Serializer - Manual field definition
class BookSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(max_length=200)
    author = serializers.CharField(max_length=100)
    price = serializers.DecimalField(max_digits=6, decimal_places=2)
    
    def create(self, validated_data):
        return Book.objects.create(**validated_data)
    
    def update(self, instance, validated_data):
        instance.title = validated_data.get('title', instance.title)
        instance.author = validated_data.get('author', instance.author)
        instance.price = validated_data.get('price', instance.price)
        instance.save()
        return instance

# ModelSerializer - Automatic from model
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'price']
    # create() and update() are automatic
```

### Interview Question 2: Explain serializers in DRF and their types.

**Answer:**
**Serializers** convert complex data types (QuerySets, model instances) to Python datatypes that can be rendered into JSON/XML.

**Types:**

**1. Serializer:**
- Manual field definition
- Full control over fields
- Must implement create() and update()

```python
class BookSerializer(serializers.Serializer):
    title = serializers.CharField(max_length=200)
    price = serializers.DecimalField(max_digits=6, decimal_places=2)
    
    def create(self, validated_data):
        return Book.objects.create(**validated_data)
```

**2. ModelSerializer:**
- Automatic field generation from model
- Automatic create() and update()
- Less code, more convenient

```python
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'price']
```

**3. HyperlinkedModelSerializer:**
- Uses hyperlinks instead of primary keys
- Better for RESTful APIs

```python
class BookSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Book
        fields = ['url', 'title', 'author']
```

**Key features:**
- **Validation**: Field and object-level validation
- **Nested serializers**: Represent relationships
- **Custom fields**: SerializerMethodField for computed values
- **Read-only/Write-only**: Control field access

---

## Views & ViewSets

### Function-Based Views (APIView)
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

@api_view(['GET', 'POST'])
def book_list(request):
    if request.method == 'GET':
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)
    
    elif request.method == 'POST':
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'DELETE'])
def book_detail(request, pk):
    try:
        book = Book.objects.get(pk=pk)
    except Book.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)
    
    if request.method == 'GET':
        serializer = BookSerializer(book)
        return Response(serializer.data)
    
    elif request.method == 'PUT':
        serializer = BookSerializer(book, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'DELETE':
        book.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### Class-Based Views (APIView)
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

class BookList(APIView):
    def get(self, request):
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)
    
    def post(self, request):
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class BookDetail(APIView):
    def get_object(self, pk):
        try:
            return Book.objects.get(pk=pk)
        except Book.DoesNotExist:
            return None
    
    def get(self, request, pk):
        book = self.get_object(pk)
        if not book:
            return Response(status=status.HTTP_404_NOT_FOUND)
        serializer = BookSerializer(book)
        return Response(serializer.data)
    
    def put(self, request, pk):
        book = self.get_object(pk)
        if not book:
            return Response(status=status.HTTP_404_NOT_FOUND)
        serializer = BookSerializer(book, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    def delete(self, request, pk):
        book = self.get_object(pk)
        if not book:
            return Response(status=status.HTTP_404_NOT_FOUND)
        book.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### Generic Views
```python
from rest_framework import generics

# List and Create
class BookList(generics.ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

# Retrieve, Update, Delete
class BookDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

# Other generic views:
# ListAPIView - Read-only list
# CreateAPIView - Create only
# RetrieveAPIView - Read-only single object
# UpdateAPIView - Update only
# DestroyAPIView - Delete only
```

### ViewSets
```python
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.response import Response

class BookViewSet(viewsets.ModelViewSet):
    """
    ModelViewSet provides:
    - list() - GET /books/
    - create() - POST /books/
    - retrieve() - GET /books/{id}/
    - update() - PUT /books/{id}/
    - partial_update() - PATCH /books/{id}/
    - destroy() - DELETE /books/{id}/
    """
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    
    # Custom action
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        book = self.get_object()
        book.status = 'published'
        book.save()
        return Response({'status': 'book published'})
    
    @action(detail=False, methods=['get'])
    def recent(self, request):
        recent_books = Book.objects.order_by('-published_date')[:10]
        serializer = self.get_serializer(recent_books, many=True)
        return Response(serializer.data)
    
    # Override methods
    def get_queryset(self):
        queryset = Book.objects.all()
        author = self.request.query_params.get('author')
        if author:
            queryset = queryset.filter(author__name=author)
        return queryset
    
    def perform_create(self, serializer):
        serializer.save(created_by=self.request.user)

# ReadOnlyModelViewSet - Only list and retrieve
class BookViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```


### Interview Question 3: What's the difference between APIView, Generic Views, and ViewSets?

**Answer:**

**APIView:**
- Most basic, full control
- Manual implementation of HTTP methods
- More code, more flexibility

```python
class BookList(APIView):
    def get(self, request):
        # Manual implementation
        pass
    
    def post(self, request):
        # Manual implementation
        pass
```

**Generic Views:**
- Pre-built common patterns
- Less code than APIView
- Mixins for combining functionality

```python
class BookList(generics.ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    # Automatic GET and POST
```

**ViewSets:**
- Combine related views
- Work with routers for automatic URL routing
- Least code, most abstraction

```python
class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    # Automatic CRUD operations
```

**When to use:**
- **APIView**: Custom logic, non-standard endpoints
- **Generic Views**: Standard CRUD with some customization
- **ViewSets**: Full CRUD with minimal code

**Comparison:**

| Feature | APIView | Generic Views | ViewSets |
|---------|---------|---------------|----------|
| Code | Most | Medium | Least |
| Flexibility | High | Medium | Low |
| URL routing | Manual | Manual | Automatic (routers) |
| Use case | Custom logic | Standard CRUD | Full REST API |

---

## Routers

### DefaultRouter
```python
from rest_framework.routers import DefaultRouter
from .views import BookViewSet, AuthorViewSet

router = DefaultRouter()
router.register(r'books', BookViewSet, basename='book')
router.register(r'authors', AuthorViewSet, basename='author')

urlpatterns = [
    path('api/', include(router.urls)),
]

# Generated URLs:
# GET    /api/books/          - list
# POST   /api/books/          - create
# GET    /api/books/{id}/     - retrieve
# PUT    /api/books/{id}/     - update
# PATCH  /api/books/{id}/     - partial_update
# DELETE /api/books/{id}/     - destroy
```

### SimpleRouter
```python
from rest_framework.routers import SimpleRouter

router = SimpleRouter()
router.register(r'books', BookViewSet)

# Similar to DefaultRouter but without API root view
```

### Custom Actions with Routers
```python
class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        # POST /api/books/{id}/publish/
        pass
    
    @action(detail=False, methods=['get'])
    def bestsellers(self, request):
        # GET /api/books/bestsellers/
        pass
```

### Nested Routers
```python
from rest_framework_nested import routers

router = routers.DefaultRouter()
router.register(r'authors', AuthorViewSet)

authors_router = routers.NestedDefaultRouter(router, r'authors', lookup='author')
authors_router.register(r'books', BookViewSet, basename='author-books')

# URLs:
# /api/authors/
# /api/authors/{author_id}/books/
# /api/authors/{author_id}/books/{id}/
```

---

## Authentication

### Token Authentication
```python
# settings.py
INSTALLED_APPS = [
    ...
    'rest_framework.authtoken',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
}

# Create tokens
python manage.py drf_create_token <username>

# Or programmatically
from rest_framework.authtoken.models import Token
token = Token.objects.create(user=user)

# Login view
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    path('api/login/', obtain_auth_token),
]

# Custom login
from rest_framework.authtoken.models import Token
from rest_framework.response import Response
from rest_framework.decorators import api_view

@api_view(['POST'])
def login(request):
    username = request.data.get('username')
    password = request.data.get('password')
    user = authenticate(username=username, password=password)
    
    if user:
        token, created = Token.objects.get_or_create(user=user)
        return Response({'token': token.key})
    return Response({'error': 'Invalid credentials'}, status=400)

# Use token in requests
# Header: Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b
```

### Session Authentication
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
    ],
}

# Uses Django's session framework
# Requires CSRF token for unsafe methods (POST, PUT, DELETE)
```

### JWT Authentication
```bash
pip install djangorestframework-simplejwt
```

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}

from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
}

# urls.py
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]

# Usage:
# POST /api/token/ with {username, password}
# Returns: {access, refresh}
# Header: Authorization: Bearer <access_token>
```

### Interview Question 4: Explain authentication methods in DRF.

**Answer:**

**1. Token Authentication:**
- Simple token-based auth
- One token per user
- Token stored in database
- Good for mobile apps

```python
# Header: Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b
```

**2. Session Authentication:**
- Uses Django sessions
- Cookie-based
- Requires CSRF protection
- Good for web applications

**3. JWT (JSON Web Token):**
- Stateless authentication
- Token contains user info
- Access and refresh tokens
- Good for microservices

```python
# Header: Authorization: Bearer <jwt_token>
```

**4. Basic Authentication:**
- Username:password in header
- Base64 encoded
- Not secure without HTTPS
- Good for testing only

**Comparison:**

| Method | Stateless | Mobile-friendly | Security | Use case |
|--------|-----------|-----------------|----------|----------|
| Token | No | Yes | Good | Mobile apps |
| Session | No | No | Good | Web apps |
| JWT | Yes | Yes | Very Good | Microservices |
| Basic | Yes | Yes | Poor | Testing only |

---

## Permissions

### Built-in Permissions
```python
from rest_framework import permissions

# AllowAny - No authentication required
class BookViewSet(viewsets.ModelViewSet):
    permission_classes = [permissions.AllowAny]

# IsAuthenticated - Must be logged in
class BookViewSet(viewsets.ModelViewSet):
    permission_classes = [permissions.IsAuthenticated]

# IsAuthenticatedOrReadOnly - Read for all, write for authenticated
class BookViewSet(viewsets.ModelViewSet):
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

# IsAdminUser - Only admin users
class BookViewSet(viewsets.ModelViewSet):
    permission_classes = [permissions.IsAdminUser]
```

### Custom Permissions
```python
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Custom permission to only allow owners to edit objects.
    """
    def has_object_permission(self, request, view, obj):
        # Read permissions for any request (GET, HEAD, OPTIONS)
        if request.method in permissions.SAFE_METHODS:
            return True
        
        # Write permissions only for owner
        return obj.owner == request.user

class IsAuthorOrReadOnly(permissions.BasePermission):
    def has_permission(self, request, view):
        # Check at view level
        if request.method in permissions.SAFE_METHODS:
            return True
        return request.user.is_authenticated
    
    def has_object_permission(self, request, view, obj):
        # Check at object level
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.author == request.user

# Usage
class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    permission_classes = [IsAuthorOrReadOnly]
```

### Method-specific Permissions
```python
class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    
    def get_permissions(self):
        if self.action in ['list', 'retrieve']:
            permission_classes = [permissions.AllowAny]
        elif self.action == 'create':
            permission_classes = [permissions.IsAuthenticated]
        else:
            permission_classes = [permissions.IsAdminUser]
        return [permission() for permission in permission_classes]
```

### Interview Question 5: How do you implement custom permissions in DRF?

**Answer:**
Custom permissions are created by subclassing `BasePermission` and implementing `has_permission()` and/or `has_object_permission()`.

```python
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Custom permission: owners can edit, others can only read
    """
    
    # View-level permission
    def has_permission(self, request, view):
        # Allow GET, HEAD, OPTIONS for everyone
        if request.method in permissions.SAFE_METHODS:
            return True
        # POST requires authentication
        return request.user.is_authenticated
    
    # Object-level permission
    def has_object_permission(self, request, view, obj):
        # Read for everyone
        if request.method in permissions.SAFE_METHODS:
            return True
        
        # Write only for owner
        return obj.owner == request.user

# Usage
class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    permission_classes = [IsOwnerOrReadOnly]
```

**Key methods:**
- `has_permission()`: View-level check (before retrieving object)
- `has_object_permission()`: Object-level check (after retrieving object)
- `SAFE_METHODS`: GET, HEAD, OPTIONS (read-only)

---

## Pagination

### PageNumberPagination
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}

# Custom pagination
from rest_framework.pagination import PageNumberPagination

class StandardResultsSetPagination(PageNumberPagination):
    page_size = 10
    page_size_query_param = 'page_size'
    max_page_size = 100

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    pagination_class = StandardResultsSetPagination

# Response:
# {
#     "count": 100,
#     "next": "http://api.example.com/books/?page=2",
#     "previous": null,
#     "results": [...]
# }
```

### LimitOffsetPagination
```python
from rest_framework.pagination import LimitOffsetPagination

class CustomLimitOffsetPagination(LimitOffsetPagination):
    default_limit = 10
    max_limit = 100

# URL: /api/books/?limit=10&offset=20
```

### CursorPagination
```python
from rest_framework.pagination import CursorPagination

class BookCursorPagination(CursorPagination):
    page_size = 10
    ordering = '-created_at'
    cursor_query_param = 'cursor'

# Best for large datasets
# Prevents users from jumping to arbitrary pages
```

---

## Filtering & Search

### Filtering
```bash
pip install django-filter
```

```python
# settings.py
INSTALLED_APPS = [
    ...
    'django_filters',
]

REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
    ],
}

# views.py
from django_filters.rest_framework import DjangoFilterBackend

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ['author', 'published_date', 'price']

# URL: /api/books/?author=John&price=29.99

# Custom filters
from django_filters import rest_framework as filters

class BookFilter(filters.FilterSet):
    min_price = filters.NumberFilter(field_name="price", lookup_expr='gte')
    max_price = filters.NumberFilter(field_name="price", lookup_expr='lte')
    title = filters.CharFilter(lookup_expr='icontains')
    
    class Meta:
        model = Book
        fields = ['author', 'published_date']

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    filterset_class = BookFilter

# URL: /api/books/?min_price=10&max_price=50&title=django
```

### Search
```python
from rest_framework import filters

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    filter_backends = [filters.SearchFilter]
    search_fields = ['title', 'author__name', 'isbn']

# URL: /api/books/?search=django

# Search with different lookups
search_fields = [
    'title',           # Exact match
    '^title',          # Starts with
    '=title',          # Exact match (case-sensitive)
    '@title',          # Full-text search (PostgreSQL)
]
```

### Ordering
```python
from rest_framework import filters

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    filter_backends = [filters.OrderingFilter]
    ordering_fields = ['price', 'published_date', 'title']
    ordering = ['-published_date']  # Default ordering

# URL: /api/books/?ordering=-price
# URL: /api/books/?ordering=price,title
```

---

## Throttling

### Built-in Throttling
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day'
    }
}

# Per-view throttling
from rest_framework.throttling import UserRateThrottle

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    throttle_classes = [UserRateThrottle]
```

### Custom Throttling
```python
from rest_framework.throttling import UserRateThrottle

class BurstRateThrottle(UserRateThrottle):
    scope = 'burst'

class SustainedRateThrottle(UserRateThrottle):
    scope = 'sustained'

# settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': {
        'burst': '60/min',
        'sustained': '1000/day'
    }
}

class BookViewSet(viewsets.ModelViewSet):
    throttle_classes = [BurstRateThrottle, SustainedRateThrottle]
```

---

## Versioning

### URL Path Versioning
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.URLPathVersioning',
    'DEFAULT_VERSION': 'v1',
    'ALLOWED_VERSIONS': ['v1', 'v2'],
}

# urls.py
urlpatterns = [
    path('api/<version>/books/', BookViewSet.as_view({'get': 'list'})),
]

# URL: /api/v1/books/
# URL: /api/v2/books/
```

### Query Parameter Versioning
```python
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.QueryParameterVersioning',
}

# URL: /api/books/?version=v1
```

### Header Versioning
```python
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.AcceptHeaderVersioning',
}

# Header: Accept: application/json; version=v1
```

### Version-specific Serializers
```python
class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    
    def get_serializer_class(self):
        if self.request.version == 'v1':
            return BookSerializerV1
        return BookSerializerV2
```

---

## Testing

### APITestCase
```python
from rest_framework.test import APITestCase
from rest_framework import status
from django.contrib.auth.models import User
from .models import Book

class BookAPITestCase(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.book = Book.objects.create(
            title='Test Book',
            author='Test Author',
            isbn='1234567890123',
            price=29.99
        )
    
    def test_get_book_list(self):
        response = self.client.get('/api/books/')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
    
    def test_get_book_detail(self):
        response = self.client.get(f'/api/books/{self.book.id}/')
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['title'], 'Test Book')
    
    def test_create_book_unauthenticated(self):
        data = {
            'title': 'New Book',
            'author': 'New Author',
            'isbn': '9876543210987',
            'price': 39.99
        }
        response = self.client.post('/api/books/', data)
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
    
    def test_create_book_authenticated(self):
        self.client.force_authenticate(user=self.user)
        data = {
            'title': 'New Book',
            'author': 'New Author',
            'isbn': '9876543210987',
            'price': 39.99
        }
        response = self.client.post('/api/books/', data)
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(Book.objects.count(), 2)
    
    def test_update_book(self):
        self.client.force_authenticate(user=self.user)
        data = {'title': 'Updated Title'}
        response = self.client.patch(f'/api/books/{self.book.id}/', data)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.book.refresh_from_db()
        self.assertEqual(self.book.title, 'Updated Title')
    
    def test_delete_book(self):
        self.client.force_authenticate(user=self.user)
        response = self.client.delete(f'/api/books/{self.book.id}/')
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        self.assertEqual(Book.objects.count(), 0)
```

### Testing with APIClient
```python
from rest_framework.test import APIClient

class BookAPITest(APITestCase):
    def setUp(self):
        self.client = APIClient()
        self.user = User.objects.create_user('test', 'test@test.com', 'test123')
    
    def test_with_token_auth(self):
        from rest_framework.authtoken.models import Token
        token = Token.objects.create(user=self.user)
        self.client.credentials(HTTP_AUTHORIZATION='Token ' + token.key)
        response = self.client.get('/api/books/')
        self.assertEqual(response.status_code, 200)
```

---

## Common Interview Questions

### Question 6: What is the difference between Serializer and ModelSerializer?

**Answer:**

**Serializer:**
- Manual field definition
- Must implement create() and update()
- More control, more code

```python
class BookSerializer(serializers.Serializer):
    title = serializers.CharField(max_length=200)
    price = serializers.DecimalField(max_digits=6, decimal_places=2)
    
    def create(self, validated_data):
        return Book.objects.create(**validated_data)
    
    def update(self, instance, validated_data):
        instance.title = validated_data.get('title', instance.title)
        instance.save()
        return instance
```

**ModelSerializer:**
- Automatic field generation from model
- Automatic create() and update()
- Less code, less control

```python
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'price']
```

### Question 7: How do you handle nested relationships in DRF?

**Answer:**
Use nested serializers:

```python
class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Author
        fields = ['id', 'name']

class BookSerializer(serializers.ModelSerializer):
    author = AuthorSerializer(read_only=True)
    author_id = serializers.IntegerField(write_only=True)
    
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'author_id']
```

### Question 8: What is the purpose of @action decorator in ViewSets?

**Answer:**
`@action` decorator creates custom endpoints in ViewSets:

```python
class BookViewSet(viewsets.ModelViewSet):
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        # POST /api/books/{id}/publish/
        book = self.get_object()
        book.status = 'published'
        book.save()
        return Response({'status': 'published'})
    
    @action(detail=False, methods=['get'])
    def recent(self, request):
        # GET /api/books/recent/
        recent_books = Book.objects.order_by('-created_at')[:10]
        serializer = self.get_serializer(recent_books, many=True)
        return Response(serializer.data)
```

- `detail=True`: Requires object ID (/books/{id}/action/)
- `detail=False`: No object ID (/books/action/)

### Question 9: How do you implement pagination in DRF?

**Answer:**
Three types of pagination:

**1. PageNumberPagination:**
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}
# URL: /api/books/?page=2
```

**2. LimitOffsetPagination:**
```python
# URL: /api/books/?limit=10&offset=20
```

**3. CursorPagination:**
```python
# URL: /api/books/?cursor=cD0yMDIw
# Best for large datasets
```

### Question 10: Explain throttling in DRF.

**Answer:**
Throttling controls request rate:

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day'
    }
}
```

**Types:**
- **AnonRateThrottle**: For anonymous users
- **UserRateThrottle**: For authenticated users
- **ScopedRateThrottle**: Different rates for different endpoints

### Question 11: How do you handle file uploads in DRF?

**Answer:**

```python
# serializer.py
class BookSerializer(serializers.ModelSerializer):
    cover_image = serializers.ImageField()
    
    class Meta:
        model = Book
        fields = ['id', 'title', 'cover_image']

# views.py
from rest_framework.parsers import MultiPartParser, FormParser

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    parser_classes = [MultiPartParser, FormParser]

# Client request:
# POST /api/books/
# Content-Type: multipart/form-data
# Body: {title: "Book", cover_image: <file>}
```

### Question 12: What is the difference between PUT and PATCH?

**Answer:**

**PUT:**
- Full update
- All fields required
- Replaces entire resource

```python
# PUT /api/books/1/
{
    "title": "New Title",
    "author": "New Author",
    "price": 29.99,
    "isbn": "1234567890123"
}
```

**PATCH:**
- Partial update
- Only changed fields required
- Updates specific fields

```python
# PATCH /api/books/1/
{
    "title": "New Title"
}
```

---

## Quick Tips for DRF Interviews

1. **Understand serializers**: Know when to use Serializer vs ModelSerializer
2. **Master ViewSets**: Understand ModelViewSet, ReadOnlyModelViewSet
3. **Authentication**: Know Token, Session, JWT differences
4. **Permissions**: Implement custom permissions
5. **Pagination**: Understand different pagination types
6. **Filtering**: Use django-filter for complex filtering
7. **Testing**: Write comprehensive API tests
8. **Performance**: Use select_related, prefetch_related
9. **Versioning**: Implement API versioning
10. **Documentation**: Use drf-yasg or drf-spectacular

---

## Conclusion

This tutorial covers essential Django REST Framework concepts for interview preparation. DRF is a powerful toolkit that makes building Web APIs fast and efficient.

**Key Takeaways:**
- Serializers handle data conversion and validation
- ViewSets reduce boilerplate code
- Multiple authentication methods available
- Fine-grained permission control
- Built-in pagination, filtering, throttling
- Automatic API documentation
- Comprehensive testing support

Good luck with your Django REST Framework interview! 🚀
