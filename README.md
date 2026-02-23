# 🖥️ Django All Projects - Full-Stack Web Applications

A **comprehensive collection of Django web applications** demonstrating complete full-stack development including Lost & Found platform, My Tennis, My Car, and My Store projects with authentication, databases, and user interfaces.

## 🎯 Overview

This collection showcases:
- ✅ Multiple Django projects
- ✅ User authentication & authorization
- ✅ Database design (Models & ORM)
- ✅ URL routing & views
- ✅ Django templates & forms
- ✅ Admin dashboard
- ✅ REST APIs
- ✅ Deployment patterns

## 📁 Projects Overview

### 1. Lost & Found Platform

**Purpose**: Community marketplace for lost & found items

```
Features:
├── User Registration & Authentication
├── Post Lost/Found Items
├── Image Upload & Gallery
├── Search & Filter
├── Contact Messages
├── Admin Dashboard
└── Email Notifications
```

### 2. My Tennis

**Purpose**: Tennis tournament management system

```
Features:
├── Player Profiles
├── Tournament Scheduling
├── Match Scoring
├── Rankings & Leaderboards
├── Team Management
└── Statistics & Records
```

### 3. My Car

**Purpose**: Vehicle management & analytics

```
Features:
├── Car Inventory
├── Maintenance Tracking
├── Fuel Consumption Logs
├── Expense Management
├── Service Reminders
└── Performance Analytics
```

### 4. My Store

**Purpose**: E-commerce platform

```
Features:
├── Product Catalog
├── Shopping Cart
├── Order Management
├── Payment Integration
├── Inventory Tracking
├── Customer Reviews
└── Admin Backend
```

## 🏗️ Django Architecture (Common Across All)

### Project Structure

```
project_name/
├── manage.py              # Django management script
├── requirements.txt       # Python dependencies
├── project_name/          # Project settings
│   ├── settings.py        # Configuration
│   ├── urls.py            # Root URL routing
│   ├── wsgi.py            # WSGI entry point
│   └── asgi.py            # ASGI for async
├── apps/                  # Django applications
│   ├── users/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── forms.py
│   │   └── templates/
│   ├── items/
│   │   └── (same structure)
│   └── core/
│       └── (shared functionality)
├── templates/             # HTML templates
│   ├── base.html
│   ├── home.html
│   └── app_templates/
├── static/                # CSS, JS, images
│   ├── css/
│   ├── js/
│   └── images/
├── media/                 # User uploaded files
├── migrations/            # Database schema versions
└── .env                   # Environment variables
```

## 🔧 Core Components

### User Model & Authentication

```python
# apps/users/models.py
from django.db import models
from django.contrib.auth.models import AbstractUser
from django.db.models.signals import post_save
from django.dispatch import receiver

class CustomUser(AbstractUser):
    """Extended user model with additional fields"""
    
    ROLE_CHOICES = [
        ('user', 'Regular User'),
        ('moderator', 'Moderator'),
        ('admin', 'Administrator'),
    ]
    
    profile_picture = models.ImageField(
        upload_to='profiles/',
        blank=True,
        null=True
    )
    bio = models.TextField(max_length=500, blank=True)
    phone_number = models.CharField(max_length=15, blank=True)
    location = models.CharField(max_length=100, blank=True)
    role = models.CharField(max_length=20, choices=ROLE_CHOICES, default='user')
    is_verified = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        db_table = 'users'
        verbose_name = 'User'
        verbose_name_plural = 'Users'
    
    def __str__(self):
        return f"{self.username} ({self.get_role_display()})"
    
    def is_moderator(self):
        return self.role in ['moderator', 'admin']
    
    def is_admin_user(self):
        return self.role == 'admin'

# Create user profile on user creation
@receiver(post_save, sender=CustomUser)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=CustomUser)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
```

### Lost & Found Item Model

```python
# apps/items/models.py
from django.db import models
from django.contrib.auth import get_user_model
from django.utils.text import slugify

User = get_user_model()

class Category(models.Model):
    """Item categories"""
    name = models.CharField(max_length=100, unique=True)
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True)
    
    class Meta:
        verbose_name_plural = 'Categories'
    
    def __str__(self):
        return self.name

class Item(models.Model):
    """Lost or Found item listing"""
    
    STATUS_CHOICES = [
        ('lost', 'Lost'),
        ('found', 'Found'),
        ('claimed', 'Claimed'),
        ('resolved', 'Resolved'),
    ]
    
    # Core fields
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    description = models.TextField()
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='lost')
    category = models.ForeignKey(Category, on_delete=models.PROTECT, related_name='items')
    
    # User & location
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='items')
    location = models.CharField(max_length=200)
    latitude = models.FloatField(null=True, blank=True)  # For mapping
    longitude = models.FloatField(null=True, blank=True)
    
    # Dates
    date_lost_found = models.DateTimeField()
    posted_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    # Additional
    reward = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    is_urgent = models.BooleanField(default=False)
    views_count = models.PositiveIntegerField(default=0)
    
    class Meta:
        ordering = ['-posted_at']
        indexes = [
            models.Index(fields=['status', '-posted_at']),
            models.Index(fields=['category', 'status']),
        ]
    
    def __str__(self):
        return self.title
    
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.title)
        super().save(*args, **kwargs)

class ItemImage(models.Model):
    """Multiple images per item"""
    item = models.ForeignKey(Item, on_delete=models.CASCADE, related_name='images')
    image = models.ImageField(upload_to='items/%Y/%m/%d/')
    uploaded_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        ordering = ['uploaded_at']

class Message(models.Model):
    """Direct messaging between users"""
    sender = models.ForeignKey(User, on_delete=models.CASCADE, related_name='sent_messages')
    recipient = models.ForeignKey(User, on_delete=models.CASCADE, related_name='received_messages')
    item = models.ForeignKey(Item, on_delete=models.CASCADE, related_name='messages')
    content = models.TextField()
    sent_at = models.DateTimeField(auto_now_add=True)
    read = models.BooleanField(default=False)
    
    class Meta:
        ordering = ['-sent_at']
    
    def __str__(self):
        return f"{self.sender.username} → {self.recipient.username}"
```

### Views & URL Routing

```python
# apps/items/views.py
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib.auth.decorators import login_required
from django.views.generic import ListView, DetailView, CreateView, UpdateView
from django.contrib.auth.mixins import LoginRequiredMixin
from django.db.models import Q
from .models import Item, Category, Message
from .forms import ItemForm, MessageForm

class ItemListView(ListView):
    """Display all items with filtering"""
    model = Item
    template_name = 'items/item_list.html'
    context_object_name = 'items'
    paginate_by = 12
    
    def get_queryset(self):
        queryset = Item.objects.select_related('user', 'category').prefetch_related('images')
        
        # Filtering
        status = self.request.GET.get('status')
        category = self.request.GET.get('category')
        search = self.request.GET.get('search')
        
        if status:
            queryset = queryset.filter(status=status)
        if category:
            queryset = queryset.filter(category__slug=category)
        if search:
            queryset = queryset.filter(
                Q(title__icontains=search) |
                Q(description__icontains=search) |
                Q(location__icontains=search)
            )
        
        return queryset
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['categories'] = Category.objects.all()
        context['item_count'] = self.get_queryset().count()
        return context

class ItemDetailView(DetailView):
    """Display item details"""
    model = Item
    template_name = 'items/item_detail.html'
    context_object_name = 'item'
    slug_field = 'slug'
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        item = self.get_object()
        
        # Increment views
        item.views_count += 1
        item.save(update_fields=['views_count'])
        
        # Messages
        if self.request.user.is_authenticated:
            context['messages'] = Message.objects.filter(
                Q(item=item) &
                (Q(sender=self.request.user) | Q(recipient=self.request.user))
            ).order_by('-sent_at')
            context['message_form'] = MessageForm()
        
        return context

@login_required
def create_item(request):
    """Create new lost/found item"""
    if request.method == 'POST':
        form = ItemForm(request.POST)
        if form.is_valid():
            item = form.save(commit=False)
            item.user = request.user
            item.save()
            
            # Handle image uploads
            images = request.FILES.getlist('images')
            for image in images:
                ItemImage.objects.create(item=item, image=image)
            
            return redirect('item-detail', slug=item.slug)
    else:
        form = ItemForm()
    
    return render(request, 'items/create_item.html', {'form': form})
```

### URL Configuration

```python
# apps/items/urls.py
from django.urls import path
from . import views

app_name = 'items'

urlpatterns = [
    path('', views.ItemListView.as_view(), name='item-list'),
    path('create/', views.create_item, name='create-item'),
    path('<slug:slug>/', views.ItemDetailView.as_view(), name='item-detail'),
    # ... more URLs
]

# project/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('apps.core.urls')),
    path('users/', include('apps.users.urls')),
    path('items/', include('apps.items.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

## 📝 Forms & Validation

```python
# apps/items/forms.py
from django import forms
from .models import Item, Message

class ItemForm(forms.ModelForm):
    """Form for creating/editing items"""
    
    class Meta:
        model = Item
        fields = ['title', 'description', 'status', 'category', 
                 'location', 'date_lost_found', 'reward', 'is_urgent']
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control'}),
            'description': forms.Textarea(attrs={'class': 'form-control', 'rows': 4}),
            'date_lost_found': forms.DateTimeInput(attrs={'type': 'datetime-local'}),
            'reward': forms.NumberInput(attrs={'class': 'form-control'}),
        }
    
    def clean_title(self):
        title = self.cleaned_data.get('title')
        if len(title) < 5:
            raise forms.ValidationError('Title must be at least 5 characters')
        return title

class MessageForm(forms.ModelForm):
    """Form for messaging other users"""
    
    class Meta:
        model = Message
        fields = ['content']
        widgets = {
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 3,
                'placeholder': 'Type your message...'
            })
        }
```

## 🚀 Admin Customization

```python
# apps/items/admin.py
from django.contrib import admin
from .models import Item, Category, ItemImage

class ItemImageInline(admin.StackedInline):
    """Inline image upload in item admin"""
    model = ItemImage
    extra = 1

@admin.register(Item)
class ItemAdmin(admin.ModelAdmin):
    list_display = ('title', 'status', 'user', 'category', 'posted_at', 'views_count')
    list_filter = ('status', 'category', 'is_urgent', 'posted_at')
    search_fields = ('title', 'description', 'location')
    readonly_fields = ('posted_at', 'updated_at', 'views_count')
    inlines = [ItemImageInline]
    
    fieldsets = (
        ('Item Information', {
            'fields': ('title', 'slug', 'description', 'category', 'status')
        }),
        ('Location & Dates', {
            'fields': ('location', 'latitude', 'longitude', 'date_lost_found')
        }),
        ('Additional', {
            'fields': ('reward', 'is_urgent', 'user')
        }),
        ('Statistics', {
            'fields': ('views_count', 'posted_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )
    
    def save_model(self, request, obj, form, change):
        if not change:  # Creating new item
            obj.user = request.user
        super().save_model(request, obj, form, change)

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ('name', 'slug')
    prepopulated_fields = {'slug': ('name',)}
```

## 💡 Interview Talking Points

### Q: How do you handle user uploads securely?

```
Answer:
1. File validation:
   - Check file types/MIME
   - Limit file size (settings.FILE_UPLOAD_MAX_MEMORY_SIZE)
   
2. Storage:
   - Store outside MEDIA_ROOT for private files
   - Use signed URLs for temporary access
   
3. Permissions:
   - Only owner can access own files
   - Validate permissions in views
```

## 🌟 Portfolio Value

✅ Full-stack Django expertise
✅ Database design & ORM
✅ User authentication patterns
✅ Multiple project experience
✅ Admin customization
✅ Form handling & validation
✅ CSS/template integration
✅ Production-ready structure

## 📄 License

MIT License - Educational Use

---

**Project Highlights**:
- 4 complete Django applications
- User authentication & authorization
- Complex relational databases
- Custom admin interfaces
- Image upload handling
- Search & filtering
- RESTful APIs
- Professional deployment-ready code

