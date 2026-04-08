# Django Patterns Skill

Idiomatic Django architecture, models, views, and service layer patterns.

## Project Structure

```
project/
├── config/                  # Project config (replaces default project dir)
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py
│   └── wsgi.py
├── apps/
│   ├── users/               # One directory per Django app
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── services.py      # Business logic lives here
│   │   ├── urls.py
│   │   └── tests/
│   └── billing/
├── common/                  # Shared utilities, base classes
├── manage.py
└── requirements/
    ├── base.txt
    ├── development.txt
    └── production.txt
```

## Models

```python
from django.db import models
from django.utils import timezone


class TimestampedModel(models.Model):
    """Abstract base model with created/updated timestamps."""
    created_at = models.DateTimeField(default=timezone.now)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True


class Organization(TimestampedModel):
    name = models.CharField(max_length=255)
    slug = models.SlugField(unique=True)
    is_active = models.BooleanField(default=True)

    class Meta:
        ordering = ['name']

    def __str__(self) -> str:
        return self.name

    def get_absolute_url(self) -> str:
        from django.urls import reverse
        return reverse('organizations:detail', kwargs={'slug': self.slug})
```

## Service Layer

Keep views thin. Put business logic in service functions:

```python
# apps/billing/services.py

from django.db import transaction
from .models import Subscription, Invoice
from .integrations import stripe_client


@transaction.atomic
def create_subscription(user, plan_id: str) -> Subscription:
    """Create a Stripe subscription and record it locally."""
    stripe_sub = stripe_client.create_subscription(
        customer_id=user.stripe_customer_id,
        price_id=plan_id,
    )
    return Subscription.objects.create(
        user=user,
        stripe_subscription_id=stripe_sub['id'],
        status=stripe_sub['status'],
        current_period_end=stripe_sub['current_period_end'],
    )
```

## Views (DRF)

```python
# apps/billing/views.py

from rest_framework import status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from .services import create_subscription
from .serializers import SubscriptionSerializer


@api_view(['POST'])
@permission_classes([IsAuthenticated])
def subscribe(request):
    plan_id = request.data.get('plan_id')
    if not plan_id:
        return Response({'error': 'plan_id is required'}, status=status.HTTP_400_BAD_REQUEST)

    subscription = create_subscription(user=request.user, plan_id=plan_id)
    return Response(SubscriptionSerializer(subscription).data, status=status.HTTP_201_CREATED)
```

## Serializers

```python
from rest_framework import serializers
from .models import Subscription


class SubscriptionSerializer(serializers.ModelSerializer):
    class Meta:
        model = Subscription
        fields = ['id', 'status', 'current_period_end', 'created_at']
        read_only_fields = fields
```

## QuerySet Patterns

```python
# Use select_related and prefetch_related aggressively
users = User.objects.select_related('profile').prefetch_related('subscriptions')

# Custom managers
class ActiveManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_active=True)

class Organization(TimestampedModel):
    objects = models.Manager()
    active = ActiveManager()
```

## Settings Split

```python
# config/settings/base.py
from pathlib import Path
import environ

env = environ.Env()
BASE_DIR = Path(__file__).resolve().parent.parent.parent
environ.Env.read_env(BASE_DIR / '.env')

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'rest_framework',
    'apps.users',
    'apps.billing',
]

# config/settings/production.py
from .base import *

DEBUG = False
ALLOWED_HOSTS = env.list('ALLOWED_HOSTS')
DATABASES = {'default': env.db('DATABASE_URL')}
```

## URL Organization

```python
# config/urls.py
from django.urls import path, include

urlpatterns = [
    path('api/v1/users/', include('apps.users.urls')),
    path('api/v1/billing/', include('apps.billing.urls')),
]

# apps/billing/urls.py
from django.urls import path
from . import views

app_name = 'billing'
urlpatterns = [
    path('subscribe/', views.subscribe, name='subscribe'),
]
```

## Common Pitfalls

- Never put business logic in models or views — use service functions
- Always use `@transaction.atomic` for multi-step writes
- Avoid N+1 queries — use `select_related`/`prefetch_related`
- Use `get_object_or_404` in views, not bare `.get()`
- Validate in serializers, not services
