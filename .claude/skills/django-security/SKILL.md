# Django Security Skill

Harden Django applications against common vulnerabilities and configure production-safe security settings.

## Security Settings Checklist

```python
# config/settings/production.py

# HTTPS
SECURE_SSL_REDIRECT = True
SECURE_HSTS_SECONDS = 31536000          # 1 year
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# Clickjacking
X_FRAME_OPTIONS = 'DENY'

# Content type sniffing
SECURE_CONTENT_TYPE_NOSNIFF = True

# XSS filter (legacy browsers)
SECURE_BROWSER_XSS_FILTER = True

# Referrer policy
SECURE_REFERRER_POLICY = 'strict-origin-when-cross-origin'

# Permissions policy
PERMISSIONS_POLICY = {
    'geolocation': [],
    'microphone': [],
    'camera': [],
}
```

## Authentication & Authorization

```python
# Use Django's built-in password validators
AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator', 'OPTIONS': {'min_length': 12}},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]

# Argon2 password hashing (stronger than PBKDF2)
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
]
```

## CSRF Protection

Never exempt views that modify state. For APIs using DRF with token auth:

```python
# DRF settings — use token/JWT instead of session for APIs
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}
```

For cookie-based auth in SPAs, include CSRF token in headers:

```python
CSRF_TRUSTED_ORIGINS = ['https://app.yourdomain.com']
```

## SQL Injection Prevention

Always use the ORM. Never use raw SQL with user input:

```python
# NEVER do this
User.objects.raw(f"SELECT * FROM users WHERE email = '{email}'")

# Safe: ORM parameterizes automatically
User.objects.filter(email=email)

# Safe: parameterized raw query if absolutely necessary
User.objects.raw("SELECT * FROM users WHERE email = %s", [email])
```

## Input Validation

Validate all user input at the serializer layer:

```python
from rest_framework import serializers
import re


class UserRegistrationSerializer(serializers.Serializer):
    email = serializers.EmailField()
    password = serializers.CharField(min_length=12, write_only=True)
    name = serializers.CharField(max_length=100)

    def validate_name(self, value: str) -> str:
        if not re.match(r'^[a-zA-Z\s\-]+$', value):
            raise serializers.ValidationError("Name contains invalid characters.")
        return value.strip()
```

## Rate Limiting

```python
# Install: pip install django-ratelimit

from django_ratelimit.decorators import ratelimit

@ratelimit(key='ip', rate='5/m', block=True)
@api_view(['POST'])
def login(request):
    ...
```

## Secrets Management

```python
# Never hardcode secrets — always read from environment
import environ

env = environ.Env()

SECRET_KEY = env('DJANGO_SECRET_KEY')
DATABASE_URL = env('DATABASE_URL')
STRIPE_SECRET_KEY = env('STRIPE_SECRET_KEY')
```

Use `python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"` to generate `SECRET_KEY`.

## File Upload Security

```python
# Restrict allowed MIME types
ALLOWED_UPLOAD_TYPES = {'image/jpeg', 'image/png', 'image/webp', 'application/pdf'}
MAX_UPLOAD_SIZE = 10 * 1024 * 1024  # 10MB

def validate_upload(file):
    import magic
    mime = magic.from_buffer(file.read(2048), mime=True)
    file.seek(0)
    if mime not in ALLOWED_UPLOAD_TYPES:
        raise ValueError(f"Unsupported file type: {mime}")
    if file.size > MAX_UPLOAD_SIZE:
        raise ValueError("File too large")
```

Never serve uploaded files from `MEDIA_ROOT` without authentication. Use signed URLs (S3) or serve through a view with permission checks.

## Dependency Scanning

```bash
# Check for known vulnerabilities
pip install safety
safety check

# Or use pip-audit
pip install pip-audit
pip-audit
```

Run in CI on every push.

## Security Headers Audit

```bash
# Check deployed headers
curl -I https://yourapp.com | grep -i "strict-transport\|x-frame\|content-security\|x-content-type"
```

Or use https://securityheaders.com for a full report.
