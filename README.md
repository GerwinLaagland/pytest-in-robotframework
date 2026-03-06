# pytest-in-robotframework

`pytest-in-robotframework` lets you execute selected pytest tests from [Robot Framework](https://robotframework.org/), so you can keep Robot Framework as the top-level test runner while still using pytest features such as fixtures, parametrization, and Python-native test structure for specific keywords.

This project is part of the Robot Framework ecosystem and is intended for cases where Robot Framework remains the main orchestration layer, but some Python-based checks are better expressed and executed through pytest.

## Why use it

This library brings together two useful testing styles:

- **[Robot Framework](https://robotframework.org/)** for readable, keyword-driven test flows
- **pytest** for Python fixtures, parametrization, and compact test logic

With `@pytest_execute`, you can call a Python keyword from Robot Framework and have that keyword executed by pytest under the hood.

This is especially useful when you want to mix keyword-driven test design with Python-heavy test logic, or when you are gradually moving tests between Robot Framework and pytest.

It can also help when Robot Framework already fits the overall suite structure, but pytest is a more natural fit for smaller combinatoric cases, fixture-heavy logic, or Python-focused assertions. Robot Framework templates can cover some repeated patterns, while pytest parametrization is often more direct for these kinds of scenarios.

## Installation

```bash
pip install pytest-in-robotframework
```

## Quick start

Import the decorator in your Python library:

```python
from pytest_in_robotframework import pytest_execute
```

Use it on functions:

```python
from pytest_in_robotframework import pytest_execute

@pytest_execute
def test_my_function():
    assert True


def rf_keyword():
    print("Executed by Robot Framework")
```

Or on methods inside a pytest-style test class:

```python
from pytest_in_robotframework import pytest_execute

class TestMyWorld:

    @pytest_execute
    def test_my_method(self):
        assert True

    def rf_keyword(self):
        print("Executed by Robot Framework")
```

## How it works

- Functions or methods decorated with `@pytest_execute` run through pytest.
- Other Python keywords run normally through Robot Framework.
- Pytest naming conventions still apply:
  - Test functions should start or end with `test`
  - Test classes should start with `Test`

The final result from pytest is passed back to Robot Framework. If a decorated keyword fails, it is logged as a failure in Robot Framework results, and the pytest console output is included in Robot Framework output as additional information.

The goal is to let Robot Framework drive the overall flow while pytest handles selected Python tests behind the scenes.

## What you get

This approach makes it possible to:

- Keep Robot Framework suites readable
- Reuse Python code in a natural way
- Use pytest fixtures and parametrization where they fit best
- Combine both tools in a single Python-based workflow

It can also help during migration, when part of a test suite is moving from Robot Framework to pytest — or the other way around.

## Example

### Robot Framework suite

```robot
*** Settings ***
Documentation     Example integration of pytest under Robot Framework
Library           TestExperiment.py

*** Test Cases ***
Login User with Password
    Open Web Page    https://www.saucedemo.com/
    Test Login As    user    password
```

### Python library

```python
import time
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By

from pytest_in_robotframework import pytest_execute


class TestExperiment:
    options = webdriver.ChromeOptions()
    options.add_experimental_option("excludeSwitches", ["enable-logging"])
    driver = webdriver.Chrome(options=options)

    def open_web_page(self, page):
        self.driver.get(page)

    @pytest_execute
    @pytest.mark.parametrize(
        "user,password",
        [
            ("standard_user", "secret_sauce"),
            ("problem_user", "secret_sauce"),
        ],
    )
    def test_login_as(self, user, password):
        username = self.driver.find_element(By.ID, "user-name")
        username.clear()
        username.send_keys(user)

        my_password = self.driver.find_element(By.ID, "password")
        my_password.clear()
        my_password.send_keys(password)

        login_button = self.driver.find_element(By.ID, "login-button")
        login_button.click()

        time.sleep(1)
        button = self.driver.find_element(By.ID, "react-burger-menu-btn")
        button.click()

        time.sleep(1)
        button = self.driver.find_element(By.ID, "logout_sidebar_link")
        button.click()
```

In this example, Robot Framework controls the test flow, while `test_login_as` runs through pytest and uses pytest parametrization.

A practical benefit here is that both execution styles can work within a single Python instance, so shared state such as the Selenium driver can be reused across Robot Framework and pytest execution.

Sometimes pytest is the better fit, and sometimes Robot Framework is. This project aims to let you integrate both with minimal friction, while preserving compatibility and keeping Robot Framework in control of the overall execution flow.

## Notes and limitations

This project is intended to preserve normal pytest behavior for decorated tests. Existing pytest tests with `@pytest_execute` added should continue to work when run directly with pytest, including normal pytest functionality such as fixture resolution and parametrization.

The main use case is mixed execution: Robot Framework stays the primary test runner, while selected Python-based checks run through pytest under the hood.

The project started as a proof of concept, but after roughly two years in the open and relatively few reported issues, it is production-usable.

## Roadmap

Possible future improvements include:

- Better pytest logging experience inside Robot Framework output, with a structure closer to `pytest_robotframework` / `pytest-robotframework`
- Hypothesis compatibility
- Custom Robot Framework keyword names
- Decorator parameters for pytest execution
- Running pytest in a separate process
- Passing additional Robot Framework parameters into pytest execution, such as log levels

## Contributing

Bug reports, improvement ideas, and pull requests are welcome.

If you try this in a real Robot Framework project, feedback from practical use cases is especially valuable.

## Support

If this project has saved you time or made your day easier, consider supporting its development.

You can support the project in several ways:

- [Buy Me a Coffee](https://buymeacoffee.com/petrkus)
- [GitHub Sponsors](https://github.com/sponsors/petr-kus)
- ⭐ Star the repository
- Watch the project
- Open bug reports
- Suggest new features or improvements

Support helps keep the project active and encourages continued work on fixes, improvements, and new ideas.

Thank you for using the project and for supporting open source.