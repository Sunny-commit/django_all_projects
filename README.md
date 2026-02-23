# Django All Projects

A comprehensive collection of Django web applications demonstrating various features, best practices, and real-world implementations.

## Overview

This repository showcases multiple Django projects covering different domains and use cases, including lost & found platforms, e-commerce, sports management, and car rental systems.

## Featured Projects

### 1. **Lost and Found Platform**
- **Purpose**: Help users find and post lost items
- **Features**:
  - User registration and authentication
  - Post creation and management
  - Search and filtering
  - Location-based matching
  - Notification system
  - Image uploads
  - User profiles

### 2. **Car Rental Management System (MyCarr)**
- **Purpose**: Manage car rental operations
- **Features**:
  - Car inventory management
  - Booking system
  - Payment processing
  - Driver verification
  - Rental agreements
  - Damage reporting
  - Invoice generation

### 3. **Sports Management Platform (My Tennis)**
- **Purpose**: Manage sports activities and tournaments
- **Features**:
  - Tournament creation
  - Player registration
  - Scheduling
  - Scoring system
  - Leaderboards
  - Match tracking
  - Statistics

### 4. **Store/Commerce System (MyStore)**
- **Purpose**: E-commerce and inventory management
- **Features**:
  - Product catalog
  - Shopping cart
  - Order management
  - Inventory tracking
  - Customer management
  - Payment integration
  - Analytics

## Technology Stack

### Backend
- **Django 3.0+**: Web framework
- **Django REST Framework**: API development
- **Django ORM**: Database abstraction
- **Celery**: Asynchronous tasks

### Database
- **PostgreSQL**: Relational database
- **SQLite**: Development database
- **Redis**: Caching and queuing

### Frontend
- **HTML5**: Markup
- **CSS3**: Styling
- **Bootstrap**: Responsive framework
- **JavaScript**: Client-side logic
- **jQuery**: DOM manipulation

### Additional Tools
- **Gunicorn**: WSGI server
- **Nginx**: Reverse proxy
- **Docker**: Containerization
- **AWS/Heroku**: Deployment

## Project Structure

```
django_all_projects/
├── lost_and_found/                    # Lost & Found app
│   ├── models.py                      # Data models
│   ├── views.py                       # View logic
│   ├── urls.py                        # URL routing
│   ├── forms.py                       # Form classes
│   ├── admin.py                       # Admin interface
│   ├── templates/                     # HTML templates
│   └── static/                        # CSS, JS, images
├── mycar/                             # Car rental app
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   └── ...
├── my_tennis/                         # Sports management
│   └── ...
├── mystore/                           # E-commerce store
│   └── ...
├── manage.py                          # Django management
├── requirements.txt                   # Dependencies
├── settings.py                        # Project settings
├── urls.py                            # Main URL config
└── README.md
```

## Installation & Setup

### Prerequisites
```
- Python 3.8+
- PostgreSQL 12+ (optional)
- pip and virtualenv
```

### Installation Steps

1. **Clone Repository**
```bash
git clone https://github.com/Sunny-commit/django_all_projects.git
cd django_all_projects
```

2. **Create Virtual Environment**
```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
```

3. **Install Dependencies**
```bash
pip install django djangorestframework pillow
pip install -r requirements.txt
```

4. **Configure Database**
```bash
# Update settings.py with database configuration
# Default: SQLite (no configuration needed for development)
```

5. **Run Migrations**
```bash
python manage.py migrate
```

6. **Create Superuser**
```bash
python manage.py createsuperuser
```

7. **Collect Static Files**
```bash
python manage.py collectstatic
```

8. **Run Development Server**
```bash
python manage.py runserver
```

The application will be available at `http://localhost:8000`

## Django Project Features

### Lost & Found Project

**Models:**
```python
- User (Django built-in)
- Post (Lost/Found items)
- Image (Item images)
- Comment (User comments)
- Location (Geographic data)
```

**Views:**
```python
- PostListView: Display all posts
- PostDetailView: Show single post
- PostCreateView: Create new post
- PostUpdateView: Edit post
- PostDeleteView: Remove post
- SearchView: Search posts
- LocationView: Map integration
```

**URL Patterns:**
```
/posts/                    # List all posts
/posts/create/            # Create post
/posts/<id>/              # Post detail
/posts/<id>/edit/         # Edit post
/posts/<id>/delete/       # Delete post
/search/                  # Search posts
/profile/<user_id>/       # User profile
```

### Car Rental System

**Models:**
```python
- Car: Vehicle details
- Booking: Rental bookings
- Payment: Transaction records
- Review: Customer reviews
- Document: Verification docs
```

**Features:**
- Availability calendar
- Real-time booking
- Payment gateway integration
- Insurance options
- Damage tracking
- Rating system

### Sports Management

**Models:**
```python
- Player: Player profiles
- Team: Team information
- Tournament: Event management
- Match: Match details
- Score: Match scoring
```

**Features:**
- Tournament brackets
- Live scoring
- Leaderboards
- Statistics tracking
- Schedule management

### E-Commerce Store

**Models:**
```python
- Product: Item catalog
- Category: Product categories
- Order: Customer orders
- OrderItem: Order line items
- Payment: Transaction processing
- Review: Product reviews
```

**Features:**
- Product filtering/sorting
- Shopping cart
- Checkout process
- Order tracking
- Inventory management

## Core Django Implementation

### Models (Database Layer)

```python
from django.db import models
from django.contrib.auth.models import User

class Post(models.Model):
    CATEGORIES = [
        ('LOST', 'Lost Item'),
        ('FOUND', 'Found Item'),
    ]
    
    title = models.CharField(max_length=200)
    description = models.TextField()
    category = models.CharField(max_length=10, choices=CATEGORIES)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    location = models.CharField(max_length=200)
    image = models.ImageField(upload_to='posts/')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    is_active = models.BooleanField(default=True)
    
    class Meta:
        ordering = ['-created_at']
    
    def __str__(self):
        return self.title
```

### Views (Business Logic)

```python
from django.views.generic import ListView, DetailView, CreateView
from django.contrib.auth.mixins import LoginRequiredMixin

class PostListView(ListView):
    model = Post
    template_name = 'posts/list.html'
    context_object_name = 'posts'
    paginate_by = 10
    
    def get_queryset(self):
        queryset = Post.objects.filter(is_active=True)
        category = self.request.GET.get('category')
        if category:
            queryset = queryset.filter(category=category)
        return queryset

class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    fields = ['title', 'description', 'category', 'location', 'image']
    template_name = 'posts/create.html'
    
    def form_valid(self, form):
        form.instance.user = self.request.user
        return super().form_valid(form)
```

### URLs (Routing)

```python
from django.urls import path
from . import views

urlpatterns = [
    path('posts/', views.PostListView.as_view(), name='post_list'),
    path('posts/<int:pk>/', views.PostDetailView.as_view(), name='post_detail'),
    path('posts/create/', views.PostCreateView.as_view(), name='post_create'),
    path('posts/<int:pk>/update/', views.PostUpdateView.as_view(), name='post_update'),
    path('posts/<int:pk>/delete/', views.PostDeleteView.as_view(), name='post_delete'),
]
```

### Templates (Presentation)

```html
<!-- posts/list.html -->
{% extends 'base.html' %}

{% block content %}
<div class="container">
    <h1>Lost & Found Items</h1>
    
    <div class="row">
        {% for post in posts %}
        <div class="col-md-4">
            <div class="card">
                <img src="{{ post.image.url }}" class="card-img-top">
                <div class="card-body">
                    <h5>{{ post.title }}</h5>
                    <p>{{ post.description|truncatewords:20 }}</p>
                    <a href="{% url 'post_detail' post.pk %}" class="btn btn-primary">
                        View Details
                    </a>
                </div>
            </div>
        </div>
        {% endfor %}
    </div>
</div>
{% endblock %}
```

## Authentication & Authorization

### User Registration
```python
from django.contrib.auth import authenticate, login
from django.contrib.auth.models import User

def register(request):
    if request.method == 'POST':
        username = request.POST['username']
        email = request.POST['email']
        password = request.POST['password']
        
        user = User.objects.create_user(
            username=username,
            email=email,
            password=password
        )
        login(request, user)
        return redirect('home')
    
    return render(request, 'register.html')
```

### Permission Classes
```python
from django.contrib.auth.decorators import login_required

@login_required
def edit_post(request, post_id):
    post = Post.objects.get(id=post_id)
    if post.user != request.user:
        return HttpResponseForbidden()
    # Edit logic
```

## Admin Interface

```python
from django.contrib import admin
from .models import Post, Comment

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'category', 'user', 'created_at']
    list_filter = ['category', 'created_at']
    search_fields = ['title', 'description']
    readonly_fields = ['created_at', 'updated_at']
    fieldsets = (
        ('General', {'fields': ('title', 'description', 'category')}),
        ('Media', {'fields': ('image',)}),
        ('Meta', {'fields': ('created_at', 'updated_at', 'is_active')}),
    )
```

## Form Handling

```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'description', 'category', 'location', 'image']
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control'}),
            'description': forms.Textarea(attrs={'class': 'form-control'}),
            'category': forms.Select(attrs={'class': 'form-control'}),
            'location': forms.TextInput(attrs={'class': 'form-control'}),
            'image': forms.FileInput(attrs={'class': 'form-control'}),
        }
```

## Middleware & Settings

### Security Settings
```python
# settings.py

ALLOWED_HOSTS = ['localhost', '127.0.0.1']
DEBUG = False  # Set to False in production
SECRET_KEY = 'your-secret-key'

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'rest_framework',
    'corsheaders',
    'lost_and_found',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
]

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'django_db',
        'USER': 'postgres',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

## Deployment

### Production Checklist
- [ ] `DEBUG = False`
- [ ] Configure allowed hosts
- [ ] Set secure cookies
- [ ] Enable HTTPS
- [ ] Configure static/media files
- [ ] Set up database backups
- [ ] Configure logging
- [ ] Set up CDN
- [ ] Configure email backend

### Docker Deployment
```dockerfile
FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "config.wsgi", "--bind", "0.0.0.0:8000"]
```

## Best Practices

✅ **Code Organization**
- Modular app structure
- Separation of concerns
- DRY principles

✅ **Database**
- Proper indexing
- Query optimization
- Connection pooling

✅ **Security**
- CSRF protection
- SQL injection prevention
- XSS protection
- Password hashing

✅ **Performance**
- Caching strategies
- Database optimization
- Async tasks with Celery
- CDN for static files

## Testing

```python
from django.test import TestCase
from .models import Post
from django.contrib.auth.models import User

class PostTests(TestCase):
    def setUp(self):
        self.user = User.objects.create_user('testuser', password='123')
        self.post = Post.objects.create(
            title='Test Post',
            user=self.user,
            category='LOST'
        )
    
    def test_post_creation(self):
        self.assertEqual(self.post.title, 'Test Post')
        self.assertEqual(self.post.user, self.user)
```

## API Development with Django REST Framework

```python
from rest_framework import serializers, viewsets
from .models import Post

class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = '__all__'

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

## Contributing

1. Fork repository
2. Create feature branch
3. Follow Django style guide
4. Add tests for new features
5. Submit pull request

## Troubleshooting

### Migration Issues
```bash
python manage.py makemigrations
python manage.py migrate --fake-initial
```

### Static Files Issues
```bash
python manage.py collectstatic --clear
```

### Database Issues
```bash
python manage.py flush  # Clear database
python manage.py migrate
```

## Resources

- [Django Official Documentation](https://docs.djangoproject.com)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Real Python Django Tutorials](https://realpython.com/tutorials/django/)

## Author

Pateti Chandu (Sunny-commit)

## License

MIT License - Free for educational and commercial use

## Support

For issues and questions, please open a GitHub issue.

## Roadmap

- [ ] API authentication (JWT)  
- [ ] Real-time notifications
- [ ] Advanced search/filtering
- [ ] Mobile app integration
- [ ] Performance optimization
- [ ] Comprehensive testing
