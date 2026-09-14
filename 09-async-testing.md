# Async Testing Standards

This document extends `03-testing-standards.md` with patterns for testing async/await code, coroutines, and concurrent operations in Python 3.7+.

## Setup: pytest-asyncio configuration

- Install `pytest-asyncio` in `pyproject.toml` (if not already present).
- Configure in `pyproject.toml`:
  ```toml
  [tool.pytest.ini_options]
  asyncio_mode = "auto"
  ```
- This mode auto-detects async tests and fixtures without manual event loop setup.

## Marking async tests

- Use `@pytest.mark.asyncio` on every async test function:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  async def test_async_function_returns_result() -> None:
      result = await async_function()
      assert result == "expected"
  ```
- Combine with other markers (`@pytest.mark.unit`, `@pytest.mark.integration`) as normal.

## Async fixtures

- Async fixtures use the same syntax as sync fixtures, but declared `async def`:
  ```python
  @pytest.fixture
  async def async_database() -> AsyncIterator[Database]:
      db = Database()
      await db.connect()
      yield db
      await db.disconnect()
  ```
- Use `AsyncIterator` for cleanup; `async` generators automatically handle setup/teardown.
- Scope rules are identical to sync fixtures: `scope="function"` (default) for isolation, `scope="module"` only for expensive read-only resources.

## Testing async functions

- Call async functions with `await`:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  async def test_fetch_user_by_id(async_database: Database) -> None:
      user = await async_database.fetch_user(user_id=123)
      assert user.name == "Alice"
  ```

## Testing with mocked async calls

- Use `mocker.AsyncMock()` (from `pytest-mock` via `unittest.mock`) to mock async functions:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  async def test_service_calls_api(mocker: MockerFixture) -> None:
      mock_api = mocker.AsyncMock(return_value={"status": "ok"})
      service = Service(api=mock_api)
      
      result = await service.check_status()
      mock_api.assert_called_once()
      assert result == {"status": "ok"}
  ```
- For coroutines that raise exceptions:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  async def test_api_timeout_raises_error(mocker: MockerFixture) -> None:
      mock_api = mocker.AsyncMock(side_effect=asyncio.TimeoutError())
      service = Service(api=mock_api)
      
      with pytest.raises(asyncio.TimeoutError):
          await service.check_status()
  ```

## Testing concurrent operations

- Use `asyncio.gather()` to run multiple coroutines and verify they all complete:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  async def test_concurrent_requests() -> None:
      results = await asyncio.gather(
          async_function(1),
          async_function(2),
          async_function(3),


## Mocking time & delays

- Use `pytest-freezegun` or `mocker.patch` to mock `asyncio.sleep()` and timers:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  async def test_retry_with_backoff(mocker: MockerFixture) -> None:
      mock_sleep = mocker.patch("asyncio.sleep")
      mock_api = mocker.AsyncMock(side_effect=[Exception(), {"ok": True}])
      
      result = await retry_with_backoff(mock_api, retries=2)
      assert mock_sleep.called  # Verify delay occurred
      assert result == {"ok": True}
  ```

## Edge cases

- **Empty coroutine:** Test that a no-op async function completes:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  async def test_empty_async_function() -> None:
      result = await no_op_async()
      assert result is None
  ```
- **Exception in coroutine:** Test exception propagation:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  async def test_async_function_raises_error() -> None:
      with pytest.raises(ValueError, match="invalid input"):
          await async_function_that_raises()
  ```
- **Event loop cleanup:** Handled automatically by `pytest-asyncio` — no manual teardown needed.

## Coverage & pragma: no cover

- Mark platform-specific async code (e.g., Windows-only event loop setup) with `# pragma: no cover`:
  ```python
  if sys.platform == "win32":  # pragma: no cover
      asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
  ```
- Never mark core async logic as untestable — write a test instead.

## Anti-patterns

- **Don't use `asyncio.run()` in tests** — `pytest-asyncio` manages the event loop.
  ```python
  # ❌ Bad
  def test_async():
      result = asyncio.run(async_function())
      assert result == "expected"
  
  # ✅ Good
  @pytest.mark.asyncio
  async def test_async() -> None:
      result = await async_function()
      assert result == "expected"
  ```
- **Don't mix sync and async without `sync_to_async`** — keep boundaries clear.
- **Don't use `time.sleep()` in async tests** — use `asyncio.sleep()` or mocking instead.

      )
      assert len(results) == 3
  ```
- Use `asyncio.wait()` to test race conditions or timeouts:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  async def test_first_result_wins() -> None:
      done, pending = await asyncio.wait(
          [asyncio.sleep(0.1), asyncio.sleep(0.2)],
          return_when=asyncio.FIRST_COMPLETED,
      )
      assert len(done) == 1
      for task in pending:
          task.cancel()
  ```

## Testing task cancellation

- Use `asyncio.CancelledError` to test graceful shutdown:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  async def test_task_cancellation() -> None:
      task = asyncio.create_task(long_running_operation())
      await asyncio.sleep(0.01)
      task.cancel()
      
      with pytest.raises(asyncio.CancelledError):
          await task
  ```

## Testing context managers (async with)

- Test `__aenter__` and `__aexit__` by using `async with`:
  ```python
  @pytest.mark.unit
  @pytest.mark.asyncio
  async def test_async_context_manager() -> None:
      async with AsyncResource() as resource:
          assert resource.is_open
      assert resource.is_closed
  ```
