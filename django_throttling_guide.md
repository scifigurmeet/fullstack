# Django Throttling - Beginner's Guide

## What is Throttling?

Throttling is a technique used to control the rate of requests that clients can make to your API. It helps protect your server from being overwhelmed by too many requests and ensures fair usage among all users.

**Key Benefits:**
- Prevents API abuse and DoS attacks
- Ensures fair resource allocation
- Improves server stability and performance
- Controls costs for external API usage
- Provides better user experience for all clients

## Types of Throttling

1. **Rate Limiting**: Limit number of requests per time period
2. **User-based Throttling**: Different limits for different users
3. **IP-based Throttling**: Limit requests from specific IP addresses
4. **Endpoint-specific Throttling**: Different limits for different API endpoints

## Setting Up Django REST Framework Throttling

### 1. Install Django REST Framework

```bash
pip install djangorestframework
pip install django
```

### 2. Create a New Django Project

```bash
django-admin startproject throttle_example
cd throttle_example
python manage.py startapp api
```

### 3. Configure Settings

```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'api',
]

# Basic DRF configuration
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '10/hour',    # Anonymous users: 10 requests per hour
        'user': '100/hour',   # Authenticated users: 100 requests per hour
    }
}

# Cache configuration for throttling (using default cache)
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',
    }
}
```

## Basic Throttling Examples

### Example 1: Simple API View with Throttling

```python
# api/views.py
from rest_framework.decorators import api_view, throttle_classes
from rest_framework.throttling import UserRateThrottle, AnonRateThrottle
from rest_framework.response import Response
from rest_framework import status

# Using function-based view with throttling
@api_view(['GET'])
@throttle_classes([AnonRateThrottle, UserRateThrottle])
def simple_throttled_view(request):
    """
    A simple view that demonstrates basic throttling
    """
    return Response({
        'message': 'This endpoint is throttled!',
        'user': str(request.user) if request.user.is_authenticated else 'Anonymous',
        'status': 'success'
    })

# Without throttling decorator (uses global settings)
@api_view(['GET'])
def global_throttled_view(request):
    """
    This view uses the global throttling settings from settings.py
    """
    return Response({
        'message': 'This uses global throttling settings',
        'status': 'success'
    })
```

### Example 2: Class-Based Views with Throttling

```python
# api/views.py
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.throttling import UserRateThrottle, AnonRateThrottle

class ThrottledAPIView(APIView):
    """
    Class-based view with throttling
    """
    throttle_classes = [AnonRateThrottle, UserRateThrottle]
    
    def get(self, request):
        return Response({
            'message': 'GET request with throttling',
            'user': str(request.user) if request.user.is_authenticated else 'Anonymous'
        })
    
    def post(self, request):
        return Response({
            'message': 'POST request with throttling',
            'data_received': request.data
        })
```

### Example 3: ViewSet with Throttling

```python
# api/views.py
from rest_framework import viewsets
from rest_framework.response import Response
from rest_framework.decorators import action

class ThrottledViewSet(viewsets.ViewSet):
    """
    ViewSet with throttling applied to all actions
    """
    throttle_classes = [AnonRateThrottle, UserRateThrottle]
    
    def list(self, request):
        return Response([
            {'id': 1, 'name': 'Item 1'},
            {'id': 2, 'name': 'Item 2'}
        ])
    
    @action(detail=False, methods=['get'])
    def custom_action(self, request):
        return Response({
            'message': 'This is a custom throttled action'
        })
```

## Custom Throttle Classes

### Example 1: Custom Rate Throttle

```python
# api/throttles.py
from rest_framework.throttling import UserRateThrottle

class CustomUserThrottle(UserRateThrottle):
    """
    Custom throttle that allows 5 requests per minute for authenticated users
    """
    scope = 'custom_user'

class BurstRateThrottle(UserRateThrottle):
    """
    Allow burst of requests but limit sustained usage
    """
    scope = 'burst'

class SustainedRateThrottle(UserRateThrottle):
    """
    Lower rate for sustained usage
    """
    scope = 'sustained'
```

Update settings.py:
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '10/hour',
        'user': '100/hour',
        'custom_user': '5/min',  # Custom rate
        'burst': '60/min',       # Allow bursts
        'sustained': '1000/day', # But limit daily usage
    }
}
```

### Example 2: IP-Based Custom Throttle

```python
# api/throttles.py
from rest_framework.throttling import BaseThrottle
from django.core.cache import cache
import time

class CustomIPThrottle(BaseThrottle):
    """
    Custom IP-based throttle with more control
    """
    
    def allow_request(self, request, view):
        # Get client IP
        ip = self.get_client_ip(request)
        
        # Create cache key
        cache_key = f'throttle_ip_{ip}'
        
        # Get current time
        now = time.time()
        
        # Get request history from cache
        history = cache.get(cache_key, [])
        
        # Remove old requests (older than 1 hour)
        history = [req_time for req_time in history if now - req_time < 3600]
        
        # Check if limit exceeded (10 requests per hour)
        if len(history) >= 10:
            return False
        
        # Add current request to history
        history.append(now)
        
        # Save back to cache
        cache.set(cache_key, history, 3600)  # Cache for 1 hour
        
        return True
    
    def get_client_ip(self, request):
        """Get client IP address"""
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip
    
    def wait(self):
        """Return time to wait before next request is allowed"""
        return 3600  # 1 hour
```

### Example 3: User Type Based Throttling

```python
# api/throttles.py
from rest_framework.throttling import UserRateThrottle

class PremiumUserThrottle(UserRateThrottle):
    """
    Higher limits for premium users
    """
    scope = 'premium'
    
    def get_cache_key(self, request, view):
        if request.user.is_authenticated:
            # Check if user is premium (you'll need to implement this logic)
            if hasattr(request.user, 'profile') and request.user.profile.is_premium:
                return self.cache_format % {
                    'scope': self.scope,
                    'ident': request.user.pk
                }
        
        # Fall back to regular user throttling
        return None

class RegularUserThrottle(UserRateThrottle):
    """
    Standard limits for regular users
    """
    scope = 'regular'
```

Update settings:
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': {
        'anon': '10/hour',
        'user': '100/hour',
        'premium': '1000/hour',  # Premium users get higher limits
        'regular': '100/hour',   # Regular users
    }
}
```

## URL Configuration

```python
# api/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import views

# Router for ViewSet
router = DefaultRouter()
router.register(r'throttled-viewset', views.ThrottledViewSet, basename='throttled')

urlpatterns = [
    path('simple/', views.simple_throttled_view, name='simple-throttled'),
    path('global/', views.global_throttled_view, name='global-throttled'),
    path('class/', views.ThrottledAPIView.as_view(), name='class-throttled'),
    path('', include(router.urls)),
]
```

```python
# throttle_example/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('api.urls')),
    path('api-auth/', include('rest_framework.urls')),  # For login/logout
]
```

## Advanced Throttling Patterns

### Example 1: Multiple Throttles on Same View

```python
# api/views.py
from rest_framework.decorators import api_view, throttle_classes
from rest_framework.response import Response
from .throttles import CustomIPThrottle, CustomUserThrottle

@api_view(['GET'])
@throttle_classes([CustomIPThrottle, CustomUserThrottle])
def multi_throttled_view(request):
    """
    This view is limited by both IP and user-based throttling
    The most restrictive limit will apply
    """
    return Response({
        'message': 'This endpoint has multiple throttling layers',
        'ip': request.META.get('REMOTE_ADDR'),
        'user': str(request.user)
    })
```

### Example 2: Conditional Throttling

```python
# api/views.py
from rest_framework.views import APIView
from rest_framework.response import Response

class ConditionalThrottleView(APIView):
    """
    Apply different throttling based on conditions
    """
    
    def get_throttles(self):
        """
        Override to apply different throttles based on conditions
        """
        if self.request.user.is_authenticated:
            # Authenticated users get user throttling
            if hasattr(self.request.user, 'profile') and self.request.user.profile.is_premium:
                # Premium users get higher limits
                return [PremiumUserThrottle()]
            else:
                # Regular authenticated users
                return [UserRateThrottle()]
        else:
            # Anonymous users get IP throttling
            return [AnonRateThrottle()]
    
    def get(self, request):
        return Response({
            'message': 'Throttling applied based on user type',
            'user_type': 'premium' if hasattr(request.user, 'profile') and request.user.profile.is_premium else 'regular'
        })
```

### Example 3: Endpoint-Specific Throttling

```python
# api/views.py
from rest_framework.viewsets import ViewSet
from rest_framework.decorators import action
from rest_framework.response import Response

class EndpointSpecificThrottling(ViewSet):
    """
    Different throttling for different endpoints
    """
    
    # Default throttling for all actions
    throttle_classes = [UserRateThrottle]
    
    def list(self, request):
        # Uses default throttling
        return Response({'message': 'List with default throttling'})
    
    @action(detail=False, methods=['post'], 
            throttle_classes=[AnonRateThrottle])  # Override throttling
    def upload(self, request):
        # This action has different throttling
        return Response({'message': 'Upload with stricter throttling'})
    
    @action(detail=False, methods=['get'], 
            throttle_classes=[])  # No throttling
    def public_info(self, request):
        # This action has no throttling
        return Response({'message': 'Public info with no throttling'})
```

## Testing Throttling

### Example 1: Manual Testing

Create a simple test script:

```python
# test_throttling.py
import requests
import time

BASE_URL = 'http://localhost:8000/api'

def test_throttling():
    """Test throttling by making multiple requests"""
    
    for i in range(15):  # Make 15 requests (more than the limit)
        try:
            response = requests.get(f'{BASE_URL}/simple/')
            print(f"Request {i+1}: Status {response.status_code}")
            
            if response.status_code == 429:  # Too Many Requests
                print("Throttled! Response:", response.json())
                break
            else:
                print("Response:", response.json())
                
        except requests.exceptions.RequestException as e:
            print(f"Error: {e}")
        
        time.sleep(1)  # Wait 1 second between requests

if __name__ == '__main__':
    test_throttling()
```

Run the test:
```bash
python test_throttling.py
```

### Example 2: Unit Testing Throttling

```python
# api/tests.py
from django.test import TestCase
from django.contrib.auth.models import User
from rest_framework.test import APIClient
from rest_framework import status
from django.core.cache import cache

class ThrottlingTestCase(TestCase):
    
    def setUp(self):
        self.client = APIClient()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        # Clear cache before each test
        cache.clear()
    
    def test_anonymous_throttling(self):
        """Test that anonymous users are throttled"""
        url = '/api/simple/'
        
        # Make requests up to the limit
        for i in range(10):  # Assuming 10/hour limit for anon users
            response = self.client.get(url)
            self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        # The 11th request should be throttled
        response = self.client.get(url)
        self.assertEqual(response.status_code, status.HTTP_429_TOO_MANY_REQUESTS)
    
    def test_authenticated_user_throttling(self):
        """Test that authenticated users have higher limits"""
        self.client.force_authenticate(user=self.user)
        url = '/api/simple/'
        
        # Authenticated users should have higher limits
        for i in range(20):  # Make more requests than anon limit
            response = self.client.get(url)
            if response.status_code == status.HTTP_429_TOO_MANY_REQUESTS:
                break
            self.assertEqual(response.status_code, status.HTTP_200_OK)
        else:
            # If we get here, we didn't hit the throttle limit
            self.fail("Expected to hit throttle limit")
    
    def test_throttle_headers(self):
        """Test that throttle headers are present"""
        response = self.client.get('/api/simple/')
        
        # Check for throttle headers (these may vary by implementation)
        self.assertIn('X-RateLimit-Limit', response.headers)
        self.assertIn('X-RateLimit-Remaining', response.headers)
```

Run tests:
```bash
python manage.py test
```

## Monitoring and Logging Throttling

### Example 1: Custom Throttle with Logging

```python
# api/throttles.py
import logging
from rest_framework.throttling import UserRateThrottle

logger = logging.getLogger(__name__)

class LoggingUserRateThrottle(UserRateThrottle):
    """
    User rate throttle that logs throttling events
    """
    
    def allow_request(self, request, view):
        allowed = super().allow_request(request, view)
        
        if not allowed:
            # Log throttling event
            user = request.user if request.user.is_authenticated else 'Anonymous'
            ip = request.META.get('REMOTE_ADDR', 'Unknown')
            
            logger.warning(
                f"Request throttled - User: {user}, IP: {ip}, "
                f"View: {view.__class__.__name__}, Path: {request.path}"
            )
        
        return allowed
```

### Example 2: Throttle Statistics View

```python
# api/views.py
from django.core.cache import cache
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAdminUser
from rest_framework.response import Response

@api_view(['GET'])
@permission_classes([IsAdminUser])
def throttle_stats(request):
    """
    Admin-only view to see throttling statistics
    """
    # This is a simple example - in production, you'd want more sophisticated tracking
    
    stats = {
        'cache_keys': [],
        'active_throttles': 0
    }
    
    # Get all cache keys (this is backend-specific)
    try:
        if hasattr(cache, '_cache'):  # For LocMemCache
            keys = list(cache._cache.keys())
            throttle_keys = [key for key in keys if 'throttle' in str(key)]
            stats['cache_keys'] = throttle_keys
            stats['active_throttles'] = len(throttle_keys)
    except Exception as e:
        stats['error'] = str(e)
    
    return Response(stats)
```

## Configuration Examples

### Example 1: Redis Cache for Production

```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

# Install redis
# pip install django-redis
```

### Example 2: Different Rates for Different Environments

```python
# settings.py
import os

# Different throttle rates based on environment
if os.environ.get('ENVIRONMENT') == 'production':
    THROTTLE_RATES = {
        'anon': '100/hour',
        'user': '1000/hour',
        'premium': '10000/hour',
    }
elif os.environ.get('ENVIRONMENT') == 'staging':
    THROTTLE_RATES = {
        'anon': '50/hour',
        'user': '500/hour',
        'premium': '5000/hour',
    }
else:  # Development
    THROTTLE_RATES = {
        'anon': '1000/hour',  # Higher limits for development
        'user': '10000/hour',
        'premium': '100000/hour',
    }

REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': THROTTLE_RATES
}
```

## Common Throttling Patterns

### 1. Graceful Degradation

```python
# api/views.py
from rest_framework.views import APIView
from rest_framework.response import Response

class GracefulDegradationView(APIView):
    """
    Provide different responses based on throttling status
    """
    
    def get(self, request):
        # Check if we're close to throttle limit
        throttle = UserRateThrottle()
        
        if not throttle.allow_request(request, self):
            # Instead of returning 429, return cached/simplified data
            return Response({
                'message': 'Serving cached data due to rate limiting',
                'data': self.get_cached_data(),
                'throttled': True
            })
        
        # Return full data
        return Response({
            'message': 'Full data response',
            'data': self.get_full_data(),
            'throttled': False
        })
    
    def get_cached_data(self):
        return {'simple': 'cached response'}
    
    def get_full_data(self):
        return {'complex': 'full response with lots of data'}
```

### 2. Throttle Bypass for Specific Users

```python
# api/throttles.py
from rest_framework.throttling import UserRateThrottle

class BypassableUserRateThrottle(UserRateThrottle):
    """
    Allow certain users to bypass throttling
    """
    
    def allow_request(self, request, view):
        # Check if user should bypass throttling
        if (request.user.is_authenticated and 
            (request.user.is_superuser or 
             hasattr(request.user, 'profile') and 
             request.user.profile.bypass_throttling)):
            return True
        
        # Apply normal throttling
        return super().allow_request(request, view)
```

## Troubleshooting Common Issues

### 1. Throttling Not Working

**Check these common issues:**

```python
# Make sure cache is configured
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
    }
}

# Make sure throttle classes are properly configured
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '10/hour',
        'user': '100/hour',
    }
}
```

### 2. Debug Throttling Issues

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from django.core.cache import cache

@api_view(['GET'])
def debug_throttle(request):
    """
    Debug view to check throttling status
    """
    from rest_framework.throttling import UserRateThrottle
    
    throttle = UserRateThrottle()
    
    # Get cache key that would be used
    cache_key = throttle.get_cache_key(request, None)
    
    # Get current throttle data
    throttle_data = cache.get(cache_key, [])
    
    return Response({
        'cache_key': cache_key,
        'current_requests': len(throttle_data),
        'throttle_data': throttle_data,
        'user': str(request.user),
        'is_authenticated': request.user.is_authenticated
    })
```

## Production Considerations

### 1. Use Redis for Distributed Systems

```bash
# Install Redis and django-redis
pip install redis django-redis
```

```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://redis-server:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
            'CONNECTION_POOL_KWARGS': {
                'max_connections': 50,
                'retry_on_timeout': True,
            }
        }
    }
}
```

### 2. Monitor Throttling Metrics

```python
# api/middleware.py
import time
from django.utils.deprecation import MiddlewareMixin

class ThrottleMetricsMiddleware(MiddlewareMixin):
    """
    Middleware to collect throttling metrics
    """
    
    def process_response(self, request, response):
        if response.status_code == 429:
            # Log throttling event for monitoring
            # In production, send to metrics service like Prometheus
            print(f"Throttled request: {request.path} from {request.META.get('REMOTE_ADDR')}")
        
        return response
```

## Summary

Django throttling provides powerful tools to protect your API from abuse and ensure fair usage. Key takeaways:

- **Use built-in throttle classes** for common scenarios
- **Create custom throttles** for specific business logic
- **Test thoroughly** in development and staging
- **Monitor throttling** in production
- **Use Redis** for distributed deployments
- **Configure appropriate rates** for different user types
- **Implement graceful degradation** when possible

Remember to balance security with user experience, and always test your throttling configuration thoroughly before deploying to production.