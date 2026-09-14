# Django Testing Standards

This document extends `03-testing-standards.md` with Django-specific patterns for models, views, serializers, authentication, and database isolation.

## Database isolation & transactions

- **Use `@pytest.mark.django_db`** on any test that touches the database (models, queries, fixtures).
  ```python
  @pytest.mark.unit
  @pytest.mark.django_db
  def test_user_creation_saves_to_db() -> None:
      user = User.objects.create(username="alice", email="alice@example.com")
      assert User.objects.count() == 1
      assert user.id is not None
  ```
- **Transactional safety:** By default, `pytest-django` wraps each test in a transaction and rolls it back after. This ensures test isolation — no cross-test pollution.
- **Scope:** Use `@pytest.mark.django_db(transaction=False)` only if the test *requires* commits (e.g., testing signal handlers that run post-commit). Most tests should use default (transactional).
- **Fixtures with DB:** Fixtures that create model instances must also carry `@pytest.mark.django_db`:
  ```python
  @pytest.fixture
  @pytest.mark.django_db
  def sample_user() -> User:
      return User.objects.create(username="test_user")
  ```

## Model testing

- Test model methods, properties, validators, and custom managers in isolation.
- Use `model_bakery` (optional, but recommended) for rapid object creation instead of verbose `User.objects.create(...)`:
  ```python
  from model_bakery import baker

  @pytest.mark.unit
  @pytest.mark.django_db
  def test_user_full_name_property() -> None:
      user = baker.make(User, first_name="Alice", last_name="Smith")
      assert user.get_full_name() == "Alice Smith"
  ```
- Test `clean()` and field validators by calling them directly (no DB hit needed):
  ```python
  @pytest.mark.unit
  def test_email_field_rejects_invalid_format() -> None:
      user = User(email="not-an-email")
      with pytest.raises(ValidationError):
          user.full_clean()
  ```

## View testing (function & class-based views)


## Serializer testing (Django REST Framework)

- Serializers are pure logic (no DB required unless explicitly calling `.save()`). Test them in unit tests.
  ```python
  @pytest.mark.unit
  def test_user_serializer_validates_email() -> None:
      data = {"username": "alice", "email": "invalid"}
      serializer = UserSerializer(data=data)
      assert not serializer.is_valid()
      assert "email" in serializer.errors
  ```
- Test `create()` and `update()` separately:
  ```python
  @pytest.mark.unit
  @pytest.mark.django_db
  def test_user_serializer_create() -> None:
      data = {"username": "alice", "email": "alice@example.com"}
      serializer = UserSerializer(data=data)
      assert serializer.is_valid()
      user = serializer.save()
      assert user.username == "alice"
  ```
- Use `many=True` sparingly in tests; verify list behavior with a parametrized test or a single assertion on list length.

## Authentication & permissions

- Mock or use `client.force_login()` for token-based auth (JWT, DRF Token).
  ```python
  @pytest.mark.unit
  @pytest.mark.django_db
  def test_api_requires_token(client: Client) -> None:
      response = client.get("/api/protected/", HTTP_AUTHORIZATION="Bearer invalid_token")
      assert response.status_code == 401
  ```
- For permission tests, use `mocker` to spy on permission classes:
  ```python
  @pytest.mark.unit
  def test_permission_denied_for_non_owner(mocker: MockerFixture) -> None:
      user_a = baker.make(User)
      user_b = baker.make(User)
      obj = baker.make(Article, author=user_a)
      
      permission = IsAuthor()
      request = mocker.Mock(user=user_b)
      assert not permission.has_object_permission(request, None, obj)
  ```

## Signals & middleware

- Signals are called synchronously in tests. If a signal triggers a task queue (Celery), mock the queue.
  ```python
  @pytest.mark.unit
  @pytest.mark.django_db
  def test_post_save_signal_sends_email(mocker: MockerFixture) -> None:
      mock_send = mocker.patch("app.tasks.send_welcome_email.delay")
      user = baker.make(User)
      mock_send.assert_called_once_with(user.id)
  ```
- Middleware is unit-tested by mocking `HttpRequest` and `HttpResponse`:
  ```python
  @pytest.mark.unit
  def test_middleware_adds_security_header(mocker: MockerFixture) -> None:
      middleware = SecurityHeaderMiddleware(lambda r: HttpResponse())
      request = mocker.Mock(spec=HttpRequest)
      response = middleware(request)
      assert response.get("X-Custom-Header") == "value"
  ```

## Async views (Django 3.1+)

- Use `pytest-asyncio` + `pytest-django` together. Mark async tests with `@pytest.mark.asyncio`:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  @pytest.mark.django_db
  async def test_async_view_returns_200(async_client: AsyncClient) -> None:
      response = await async_client.get("/api/async-endpoint/")
      assert response.status_code == 200
  ```
- Use `async_client` fixture (provided by `pytest-django` if `pytest-asyncio` is installed).
- For async ORM queries, use `sync_to_async`:
  ```python
  from asgiref.sync import sync_to_async

  @pytest.mark.unit
  @pytest.mark.asyncio
  @pytest.mark.django_db
  async def test_async_model_query() -> None:
      user = await sync_to_async(baker.make)(User)
      assert await sync_to_async(User.objects.count)() == 1
  ```

## Settings & configuration

- Use `@override_settings` to temporarily change Django settings in a test:
  ```python
  from django.test import override_settings

  @pytest.mark.unit
  @override_settings(DEBUG=True, ALLOWED_HOSTS=["testserver"])
  def test_debug_mode_behavior() -> None:
      from django.conf import settings
      assert settings.DEBUG is True
  ```
- For environment-specific settings (test DB, cache, etc.), configure in `pytest.ini` or `conftest.py` — never hardcode in tests.

## Fixtures & conftest

- Place Django-specific fixtures in `tests/conftest.py`:
  ```python
  import pytest
  from model_bakery import baker
  from app.models import User

  @pytest.fixture
  @pytest.mark.django_db
  def admin_user() -> User:
      return baker.make(User, is_staff=True, is_superuser=True)
  ```
- **Fixture scope:** Use `scope="function"` (default) for most fixtures to ensure test isolation. Only use `scope="module"` for read-only, expensive resources that do not carry state.

## Coverage & pragma: no cover

- Django ORM querysets, migrations, and admin classes have paths hard to reach in unit tests. Document each `# pragma: no cover`:
  ```python
  class UserAdmin(admin.ModelAdmin):
      list_display = ["username", "email"]
      
      def save_model(self, request, obj, form, change):  # pragma: no cover
          # Called by admin UI only, tested via integration tests
          super().save_model(request, obj, form, change)
  ```
- Avoid `# pragma: no cover` for business logic — write a test instead.


- Use `pytest-django`'s `client` fixture (the Django test client) for integration-style view testing.
  ```python
  @pytest.mark.unit
  @pytest.mark.django_db
  def test_user_list_returns_200(client: Client) -> None:
      response = client.get("/api/users/")
      assert response.status_code == 200
  ```
- Test view status codes, response content, headers, and redirects.
- For authentication-required views, use `client.force_login(user)`:
  ```python
  @pytest.mark.unit
  @pytest.mark.django_db
  def test_admin_view_requires_login(client: Client) -> None:
      response = client.get("/admin/")
      assert response.status_code == 302  # Redirect to login
      
      admin_user = baker.make(User, is_staff=True)
      client.force_login(admin_user)
      response = client.get("/admin/")
      assert response.status_code == 200
  ```
- For POST/PUT/DELETE, include data:
  ```python
  @pytest.mark.unit
  @pytest.mark.django_db
  def test_create_user_via_post(client: Client) -> None:
      response = client.post("/api/users/", {"username": "alice", "email": "alice@example.com"})
      assert response.status_code == 201
      assert User.objects.count() == 1
  ```

  # Django Testing Standards & Safety

## Database & Migrations (CRITICAL)
- **Migration Rule:** Whenever you modify any `models.py` file, you MUST immediately run `python manage.py makemigrations`. Failing to generate migrations will break the test database setup.
- Use `@pytest.mark.django_db` on any test touching the database.

## Model Testing
- Test models in isolation using `model_bakery`.
- Test `clean()` and validators directly (no DB hit needed).

## View Testing (Function & Class-based views)
- Use `pytest-django`'s `client` fixture for integration view testing.
  ```python
  @pytest.mark.unit
  @pytest.mark.django_db
  def test_user_list_returns_200(client: Client) -> None:
      response = client.get("/api/users/")
      assert response.status_code == 200
