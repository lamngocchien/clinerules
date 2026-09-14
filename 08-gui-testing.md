# GUI Testing Standards

This document extends `03-testing-standards.md` with patterns for headless GUI testing, event injection, display mocking, and state isolation.

## Unit tests for GUI logic (no display)

- **Separate logic from UI rendering.** Test logic (button click handlers, state calculations, validation) as pure functions or mocked objects — never depend on actual widget rendering.
  ```python
  @pytest.mark.unit
  def test_calculator_button_click_updates_display(mocker: MockerFixture) -> None:
      mock_display = mocker.Mock()
      calc = Calculator(display=mock_display)
      calc.on_button_click("5")
      mock_display.update.assert_called_once_with("5")
  ```

## Event injection & user simulation

- **Mock user input** instead of relying on real keyboard/mouse events. Inject events directly into handler methods:
  ```python
  @pytest.mark.unit
  def test_keyboard_input_calls_handler(mocker: MockerFixture) -> None:
      handler = mocker.Mock()
      gui = GUI(on_key=handler)
      
      # Simulate pressing "5"
      gui._on_key_press(key_code=53, char="5")
      handler.assert_called_once_with("5")
  ```
- For GUI frameworks (Flet, Tkinter, PyQt), create minimal mock objects that replicate the event object signature:
  ```python
  @pytest.mark.unit
  def test_flet_button_click(mocker: MockerFixture) -> None:
      button = mocker.Mock()
      button.data = "="
      
      handler = mocker.Mock()
      on_button_click(button, handler)
      handler.assert_called_once_with("=")
  ```

## Display & widget mocking

- Mock widgets and display updates to verify the GUI logic calls the right methods without rendering:
  ```python
  @pytest.mark.unit
  def test_display_shows_result(mocker: MockerFixture) -> None:
      mock_display = mocker.Mock()
      gui = Calculator(display=mock_display)
      
      gui.calculate("2+3")
      mock_display.value = "5"
      
      assert mock_display.value == "5"
  ```
- For Flet (or similar), mock page/control methods:
  ```python
  @pytest.mark.unit
  def test_modal_dialog_shown(mocker: MockerFixture) -> None:
      mock_page = mocker.Mock()
      mock_dialog = mocker.Mock()
      
      show_history_modal(mock_page, mock_dialog)
      mock_page.dialog.assert_called_once()
      mock_page.update.assert_called_once()
  ```

## State isolation & fixtures

- Use fixtures to initialize GUI state before each test (e.g., a calculator in a known state):
  ```python
  @pytest.fixture
  def calculator_gui(mocker: MockerFixture) -> Calculator:
      mock_display = mocker.Mock()
      mock_history = mocker.Mock()
      return Calculator(display=mock_display, history=mock_history)
  
  @pytest.mark.unit
  def test_clear_button_resets_display(calculator_gui: Calculator) -> None:
      calculator_gui.display.value = "123"
      calculator_gui.on_clear()
      calculator_gui.display.value = ""
      assert calculator_gui.display.value == ""
  ```

## Integration tests for full GUI workflows

- Mark tests that exercise multiple GUI components together with `@pytest.mark.integration`:
  ```python
  @pytest.mark.integration
  def test_user_enters_expression_and_calculates(mocker: MockerFixture) -> None:
      mock_display = mocker.Mock()


## Coverage & untestable paths

- Mark rendering/platform-specific code with `# pragma: no cover`:
  ```python
  def render_window():  # pragma: no cover
      # Called by GUI framework event loop, tested via manual/E2E only
      page.update()
  ```
- Mark OS-specific display APIs:
  ```python
  if sys.platform == "darwin":  # pragma: no cover
      # macOS-specific window positioning
      window.move_to_center()
  ```
- **Never** mark business logic (calculations, state management, event handlers) as untestable — write a test instead.

## Testing with real widgets (E2E / slow)

- For end-to-end GUI tests, use framework-specific tools:
  - **Flet:** Host in test environment, use HTTP client to interact.
  - **Tkinter:** Use `tkinter.ttk.treeview` snapshot testing or `pytest-qt` for PyQt.
  - **PyQt:** Use `QTest.mouseClick()`, `QTest.keyClick()` to simulate real input.
- Mark these tests with `@pytest.mark.slow` and `@pytest.mark.integration` — they are not part of the fast TDD loop:
  ```python
  @pytest.mark.slow
  @pytest.mark.integration
  def test_full_calculator_workflow_e2e(flet_app) -> None:
      # Start app, click buttons, verify display
      # Requires actual Flet runtime
      pass
  ```

## Anti-patterns

- **Don't test widget construction directly** — test the logic that uses widgets.
  ```python
  # ❌ Bad
  def test_button_exists():
      button = Button(text="Click me")
      assert button.text == "Click me"
  
  # ✅ Good
  def test_button_click_calls_handler(mocker: MockerFixture) -> None:
      handler = mocker.Mock()
      on_button_click(button, handler)
      handler.assert_called_once()
  ```
- **Don't rely on display timing or animation** — mock timers and transitions.
- **Don't sleep in tests** — use mocking or synchronous event dispatch instead.

      mock_history = mocker.Mock()
      gui = Calculator(display=mock_display, history=mock_history)
      
      gui.on_button_click("2")
      gui.on_button_click("+")
      gui.on_button_click("3")
      gui.on_button_click("=")
      
      mock_display.update.assert_called_with("5")
      mock_history.add.assert_called_once()
  ```

## Headless testing (no window)

- Run tests without opening a window. Configure GUI framework to run headless:
  - **Flet:** Use `flet.app(target=...)` with `view=None` in test environment.
  - **Tkinter:** Use `Tk()` without `mainloop()`; call widget methods directly.
  - **PyQt:** Use `QTest` utilities for event simulation.
- Example (Tkinter):
  ```python
  @pytest.mark.unit
  def test_tkinter_button_command(mocker: MockerFixture) -> None:
      root = mocker.Mock()
      button = mocker.Mock()
      
      handler = mocker.Mock()
      button.config.assert_not_called()
      # Simulate button press
      handler()
      handler.assert_called_once()
  ```
