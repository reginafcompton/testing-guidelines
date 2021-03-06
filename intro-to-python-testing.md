# `pytest` 101

* [Directory structure](#directory-structure)
* [Configure `pytest` and coordinate additional utilities](#configure-pytest-and-coordinate-additional-utilities)
* [Assert `True` == `True`](#assert-true--true)
* [Commands to run the tests](#commands-to-run-the-tests)
* [A note on filepaths](#a-note-on-filepaths)

## Directory structure

DataMade uses the `pytest` framework for testing Python. `pytest` auto-discovers all test modules by recursing into test directories and searching for `test_*.py` and `*_test.py` files. This mode of test discovery allows for some flexibility in directory-structure design: the tests can reside within nested app directories or in a stand-alone directory at the root of the main repo. At DataMade, we prefer the latter structure. Here’s how to get started.

To install pytest, run:

```bash
pip install pytest
```

Create a tests directory at root of your main directory:

```bash
├── a_unique_app/
├── another_unique_app/
├── tests/
└── ...
```

Within the tests directory, add the following files:

* **`__init__.py`**

  An empty init file to transform your tests directory into a tests module, so imports (i.e., `import from .test_config`) work correctly.

* **`test_config.py`**

  A configuration file that contains essential environment variables, i.e., INSTALLED_APPS, DATABASES, etc. (Careful! Do not put any secret data in here.) Read more about what goes in this file in the [Django set-up section](/framework-specific-patterns.md#setup). 

* **`conftest.py`**

  A file that defines fixtures, i.e., instances of objects or reusable behaviors (e.g., [the LargeLots conftest file](https://github.com/datamade/large-lots/blob/master/tests/conftest.py) has fixtures that create instances of the Lot, ApplicationStep, and Address classes)

* **`test_*.py`**

  A file with tests (as many as your application needs). Note: if you have multiple applications that require testing, these files can go inside an app-specific repo within the `tests` directory, as the below example shows.

```bash
└── tests/
    ├── conftest.py
    ├── test_config.py
    ├── a_unique_app/
    │   └── test_commands.py
    └── another_unique_app/
        └── test_form_submission.py
```

## Configure `pytest` and coordinate additional utilities

If you don't already have one, add a `setup.cfg` file to the root of your directory. This file provides a space for specifying configuration options and enabling integration of tools (testing and otherwise) – helpful, particularly, with flake8. (If you are new to testing and unfamiliar with flake8, then move ahead to the section on "Assert True is True." If not, then read on!)

Many DataMade projects use flake8 to enforce consistent and standard style patterns in Python. Yet, remembering to run both the pytest and flake8 test suites before pushing a branch to Github requires a certain degree of testing heedfulness. A `setup.cfg` file helps reduce the number of things to remember.

At the top of your `setup.cfg` file, add a section for pytest: this tells your application to assign testing options (as they would appear in `pytest.ini`, i.e., the initialization file for `pytest`).

Then, add parameters for testing setup. The below example, taken from [Dedupe service](https://github.com/datamade/dedupe-service/blob/master/setup.cfg), includes precise options for flake8 + pytest integration. 

```cfg
[tool:pytest]

# Run the flake8 option every time you execute `pytest`
addopts = --flake -v -p no:warnings

# Tell pytest to avoid specified patterns when recursing for test discovery
norecursedirs=tools .env alembic .git

# Provide parameters for flake8
flake8-max-line-length=160
flake8-select=E F W C90
flake8-ignore=E121 E123 E126 E226 E24 E704 W503 W504
```

## Assert `True == True`

Your application proudly contains the framework for productive, gainful testing practices. Well done! Now, it’s time to write a simple test.

[N.B. The above example directory assumes two generic apps: a_unique_app and another_unique_app. Likely, your site has apps of a different color – and, indeed, you may not need individual app folders in `tests` at all.]

Go to the tests directory (again, at the root of your main repository). Either within an app-test-repo of your choice or at the root of `tests`, create a test file, say, `test_my_app.py`. (Caution! The file should have `test_*.py` or `*_test.py` as its name.)

At the top, add `import pytest`. Then, create your first test. (Watch out! It should start or end with "test.")

```python
import pytest

def test_the_truth():
    assert True == True
```

The `assert` statement appears ubiquitously in python tests: it determines if both sides of a comparator are equivalent and, then, returns `True` or throws an `AssertionError`, causing a test to fail. Clearly, this simple test passes, but let’s run it to know for sure. The next section explains how to execute tests.

## Commands to run the tests

Executing the `pytest` framework is easy. It takes one command. Go to the root of your directory, and run:

```bash
pytest
```

Pytest discovers all test modules, executes them, and prints the results to terminal.

You can hone in on particular tests, too. As suggested above, you should have a `tests` directory with individual files (i.e., test modules) or app-specific repositories. Indicate the particular module or repo, and pytest will only execute those tests:

```bash
# Run all tests in a directory
pytest a_unique_app/
```

```bash
# Run all tests in a module
pytest test_my_app.py
```

```bash
# Run a particular test
pytest test_my_app.py::test_the_truth
```

You may have print statements in your tests: these can help you debug, better understand your codebase, and refactor. To turn off output capturing, add the `-s` flag, which allows printed output to appear in terminal:

```bash
pytest -s
```

Want to see more? less? You can increase verbosity with the `-v` flag: this tells pytest to include pretty-print statements, e.g. a brilliant green PASSED next to each successful test function. Alternatively, you can decrease verbosity with the `-q` or "quiet" flag.

```bash
# Increase verbosity
pytest -v

# Decrease verbosity
pytest -q
```

Tests fail. Running an entire test suite after a failure can be time-consuming and tedious. Tell pytest to stop after a failure with the `-x` flag:

```bash
pytest -x
```

You can debug a failing test with [Python debugger](https://docs.python.org/3.6/library/pdb.html), which transforms your terminal into a sandbox where you can play with code. Tell pytest to activate a pdb shell with the `--pdb` flag (note: two hyphens needed):

```bash
pytest --pdb
```

Did pytest not discover your tests? Check the names of your modules and test functions.

* Test modules should be labeled as `test_*.py` or `*_test.py`.
* Test functions should start with `test_`.

### A note on filepaths

Relative filepaths may work in your local development environment, but they aren't so friendly to Travis or our deployment environment. Save yourself the failed build: If your application includes filepaths, create an absolute filepath using `os`.

```python
import os


# Get the name of the directory your file lives in.
file_directory = os.path.dirname(__file__)

# Get the absolute path of the directory your file lives in.
absolute_file_directory = os.path.abspath(file_directory)
```

You can use this absolute path to build out other filepaths as needed using `os.path.join`. Note that your filepath **will be relative to the file you build it in**, i.e., if you are in `project_dir/some_module/script.py`, then `absolute_file_directory` will be `/path/to/project_dir/some_module`.

```python
# Build absolute path to data in the same directory as the script.
local_data = os.path.join(absolute_file_directory, 'local_data.csv')

# Build absolute path to data in a directory adjacent to (e.g., up one level) the script directory.
remote_data = os.path.join(absolute_file_directory, '..', 'another_directory', 'remote_data.json')
```

If you're building filepaths in a testing context, you can [include a fixture](intermediate-python-testing.md#fixtures) to return your project directory in `tests/conftest.py`, like this:

```python
@pytest.fixture
def project_directory():
    test_directory = os.path.abspath(os.path.dirname(__file__))
    return os.path.join(test_directory, '..')
```

Then, as before, use that path to build filepaths in your tests.

```python
def test_some_file(project_directory):
  some_file = os.path.join(project_directory, 'some_file.txt')
```
