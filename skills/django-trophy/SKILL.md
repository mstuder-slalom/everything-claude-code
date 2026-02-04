---
name: django-trophy
description: Django-specific trophy testing with pytest-django, real database testing, integration-focused approach, and minimal mocking of external services only.
---

# Django Trophy Testing

Trophy testing methodology for Django applications using pytest-django, with focus on integration tests that use real databases and minimal mocking.

## When to Activate

- Testing Django views, models, and forms
- Verifying Django REST Framework APIs
- Testing database operations and queries
- Validating Django middleware and signals

## Test Organization

```
myproject/
├── myapp/
│   ├── models.py
│   ├── views.py
│   ├── serializers.py
│   └── tests/
│       ├── __init__.py
│       ├── conftest.py         # Shared fixtures
│       ├── test_views.py       # Integration tests (PRIMARY)
│       ├── test_api.py         # API integration tests
│       └── test_validators.py  # Unit tests (selective)
├── conftest.py                 # Project-wide fixtures
└── pytest.ini
```

## pytest.ini Configuration

```ini
[pytest]
DJANGO_SETTINGS_MODULE = myproject.settings.test
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = --reuse-db --tb=short
markers =
    slow: marks tests as slow
    integration: marks tests as integration tests
```

## Fixtures (conftest.py)

```python
import pytest
from django.contrib.auth import get_user_model
from rest_framework.test import APIClient

User = get_user_model()


@pytest.fixture
def api_client():
    """Real DRF API client"""
    return APIClient()


@pytest.fixture
def authenticated_client(api_client, user):
    """Authenticated API client"""
    api_client.force_authenticate(user=user)
    return api_client


@pytest.fixture
def user(db):
    """Create real user in test database"""
    return User.objects.create_user(
        username="testuser",
        email="test@example.com",
        password="testpass123"
    )


@pytest.fixture
def admin_user(db):
    """Create real admin user"""
    return User.objects.create_superuser(
        username="admin",
        email="admin@example.com",
        password="adminpass123"
    )
```

## Integration Test Patterns

### Testing Views with Real Database

```python
import pytest
from django.urls import reverse

@pytest.mark.django_db
class TestUserRegistration:
    """
    Scenarios from: openspec/changes/auth/specs/registration/spec.md
    """

    def test_successful_registration(self, client):
        """
        WHEN a user submits valid registration data
        THEN a new user account is created
        """
        url = reverse("register")
        data = {
            "username": "newuser",
            "email": "newuser@example.com",
            "password": "SecurePass123!",
            "password_confirm": "SecurePass123!"
        }

        response = client.post(url, data)

        assert response.status_code == 201
        assert User.objects.filter(email="newuser@example.com").exists()

    def test_duplicate_email_rejected(self, client, user):
        """
        WHEN a user submits registration with existing email
        THEN registration is rejected with error
        """
        url = reverse("register")
        data = {
            "username": "newuser",
            "email": user.email,  # Existing email
            "password": "SecurePass123!",
            "password_confirm": "SecurePass123!"
        }

        response = client.post(url, data)

        assert response.status_code == 400
        assert "email" in response.json()["errors"]
```

### Testing DRF APIs

```python
import pytest
from rest_framework import status

@pytest.mark.django_db
class TestArticleAPI:
    """
    Scenarios from: openspec/changes/articles/specs/api/spec.md
    """

    def test_list_articles(self, api_client, article_factory):
        """
        WHEN user requests article list
        THEN all published articles are returned
        """
        # Create test data in real database
        article_factory.create_batch(5, status="published")
        article_factory.create_batch(2, status="draft")

        response = api_client.get("/api/articles/")

        assert response.status_code == status.HTTP_200_OK
        assert len(response.data) == 5  # Only published

    def test_create_article_authenticated(self, authenticated_client):
        """
        WHEN authenticated user creates article
        THEN article is saved with user as author
        """
        data = {
            "title": "Test Article",
            "content": "Test content",
            "status": "draft"
        }

        response = authenticated_client.post("/api/articles/", data)

        assert response.status_code == status.HTTP_201_CREATED
        assert response.data["author"] == authenticated_client.handler._force_user.id

    def test_create_article_unauthenticated(self, api_client):
        """
        WHEN unauthenticated user tries to create article
        THEN request is rejected with 401
        """
        data = {"title": "Test", "content": "Test"}

        response = api_client.post("/api/articles/", data)

        assert response.status_code == status.HTTP_401_UNAUTHORIZED
```

### Testing Model Methods

```python
import pytest
from decimal import Decimal
from myapp.models import Order

@pytest.mark.django_db
class TestOrderModel:
    """
    Scenarios from: openspec/changes/orders/specs/model/spec.md
    """

    def test_calculate_total(self, order_with_items):
        """
        WHEN order has multiple items
        THEN total is calculated correctly with tax
        """
        # order_with_items is a fixture that creates real Order with OrderItems
        total = order_with_items.calculate_total()

        expected = sum(item.price * item.quantity for item in order_with_items.items.all())
        expected_with_tax = expected * Decimal("1.08")  # 8% tax

        assert total == expected_with_tax

    def test_apply_discount(self, order_with_items):
        """
        WHEN discount code is applied
        THEN total is reduced by discount percentage
        """
        original_total = order_with_items.calculate_total()

        order_with_items.apply_discount("SAVE20")  # 20% discount

        new_total = order_with_items.calculate_total()
        assert new_total == original_total * Decimal("0.80")
```

### Testing with Factories

```python
# factories.py
import factory
from myapp.models import User, Article

class UserFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = User

    username = factory.Sequence(lambda n: f"user{n}")
    email = factory.LazyAttribute(lambda obj: f"{obj.username}@example.com")
    password = factory.PostGenerationMethodCall("set_password", "testpass123")


class ArticleFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = Article

    title = factory.Faker("sentence")
    content = factory.Faker("paragraph")
    author = factory.SubFactory(UserFactory)
    status = "published"


# conftest.py
@pytest.fixture
def user_factory():
    return UserFactory

@pytest.fixture
def article_factory():
    return ArticleFactory
```

### Mocking External Services Only

```python
import pytest
from unittest.mock import patch

@pytest.mark.django_db
class TestPaymentProcessing:
    """Mock Stripe (external), use real database"""

    @patch("myapp.services.stripe.PaymentIntent.create")
    def test_successful_payment(self, mock_stripe, order, authenticated_client):
        """
        WHEN user submits valid payment
        THEN payment is processed and order is marked as paid
        """
        mock_stripe.return_value = {
            "id": "pi_test",
            "status": "succeeded"
        }

        response = authenticated_client.post(
            f"/api/orders/{order.id}/pay/",
            {"payment_method": "pm_test"}
        )

        assert response.status_code == 200
        order.refresh_from_db()
        assert order.status == "paid"
        assert order.payment_id == "pi_test"

    @patch("myapp.services.stripe.PaymentIntent.create")
    def test_failed_payment(self, mock_stripe, order, authenticated_client):
        """
        WHEN payment processing fails
        THEN order remains unpaid and error is returned
        """
        mock_stripe.side_effect = stripe.error.CardError(
            "Card declined", None, None
        )

        response = authenticated_client.post(
            f"/api/orders/{order.id}/pay/",
            {"payment_method": "pm_test"}
        )

        assert response.status_code == 400
        order.refresh_from_db()
        assert order.status == "pending"
```

## Database Transaction Handling

```python
import pytest
from django.db import transaction

@pytest.mark.django_db(transaction=True)
class TestAtomicOperations:
    """Tests that need real transaction behavior"""

    def test_order_creation_atomic(self, user):
        """
        WHEN order creation fails midway
        THEN no partial data is saved
        """
        with pytest.raises(ValueError):
            with transaction.atomic():
                order = Order.objects.create(user=user)
                OrderItem.objects.create(order=order, product_id=1, quantity=1)
                raise ValueError("Simulated failure")

        # Order should not exist due to rollback
        assert not Order.objects.filter(user=user).exists()
```

## Running Django Trophy Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=myapp --cov-report=html

# Run specific test class
pytest myapp/tests/test_views.py::TestUserRegistration

# Run integration tests only
pytest -m integration

# Reuse database for speed
pytest --reuse-db

# Create new database
pytest --create-db

# Run in parallel
pytest -n auto
```

## Trophy Test Report (Django)

```
========================= test session starts ==========================
myapp/tests/test_views.py::TestUserRegistration::test_successful_registration PASSED
myapp/tests/test_views.py::TestUserRegistration::test_duplicate_email_rejected PASSED
myapp/tests/test_api.py::TestArticleAPI::test_list_articles PASSED
myapp/tests/test_api.py::TestArticleAPI::test_create_article_authenticated PASSED
myapp/tests/test_api.py::TestArticleAPI::test_create_article_unauthenticated PASSED

========================= Trophy Test Results ==========================
Spec Coverage:
  auth/spec.md: 5/5 scenarios ✅
  articles/spec.md: 4/5 scenarios (1 failed)
  orders/spec.md: 3/3 scenarios ✅

Failed Scenarios:
  ❌ articles/spec.md: "Article search" - Search not implemented

Total: 12/13 scenarios verified (92%)
========================= 12 passed, 1 failed ==========================
```
