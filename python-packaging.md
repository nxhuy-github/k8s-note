# Packaging Python Projects

## Tools
In Python packaging, there are `frontend` build tools and `backend` tools which can be used to describe different tools and librairies involved in the packaging process.

- `frontend` build tools, such as [pip](https://packaging.python.org/en/latest/key_projects/#pip) and [build](https://packaging.python.org/en/latest/key_projects/#build), are primarily responsible for managing the installation and distribution of Python packages
- `backend` tools like [setuptools](https://setuptools.pypa.io/en/latest/index.html), [Flit](https://flit.pypa.io/en/stable/) or [Poetry](https://python-poetry.org/) are responsible for configuring and defining the structure of Python packages, including metadata, dependencies, entry points, and more

## A bit of history
Python started out with [distutils](https://docs.python.org/3/library/distutils.html) as a build-in library, it was a collection of tools for packaging, etc. To improve on it, [setuptools](https://setuptools.pypa.io/en/latest/index.html) was created as a pip-installable package. Today, [distutils](https://docs.python.org/3/library/distutils.html) is officially deprecated starting with Python version 3.10, as decided in [PEP 632](https://peps.python.org/pep-0632/).

In the early stages, the `setup.py` script was specific to the [setuptools](https://setuptools.pypa.io/en/latest/index.html) build backend. Later on, the `setup.py` was used with the `setup.cfg` file to overcome some drawbacks (discuss later) in [setuptools](https://setuptools.pypa.io/en/latest/index.html).

Since then a lot has changed, mostly due to [PEP 517](https://peps.python.org/pep-0517/), [PEP 518](https://peps.python.org/pep-0518/) and the introduction of the `pyproject.toml` file. The goal of this file is to allow us to define what build tools are needed in order to build our package - no longer assuming it must be [setuptools](https://setuptools.pypa.io/en/latest/index.html). This makes it easier to use alternatives (like [Flit](https://flit.pypa.io/en/stable/) or [Poetry](https://python-poetry.org/)) to [setuptools](https://setuptools.pypa.io/en/latest/index.html). 

## setup.py - setup.cfg - pyproject.toml
### setup.py
The `setup.py` script is a Python file which plays as the centre of all activity in building, distributing, and installing modules using [setuptools](https://setuptools.pypa.io/en/latest/index.html) (or [distutils](https://docs.python.org/3/library/distutils.html)). And the  script consists mainly of a call to `setup()`.

Example
```
# Project structure
example_project/
├── setup.py
└── src
    └── exampleproject
        ├── __init__.py
        └── example.py
```
```python
# setup.py
from setuptools import setup

setup(
    name="exampleproject",
    install_requires=[
        "numpy",
        "pandas==1.5.3",
    ],
    package_dir={"": "src"},
    packages = setuptools.find_packages(
        where='src',
    ),
)
```

For more information about `setup()` function, please ref to [this](https://docs.python.org/3.10/distutils/setupscript.html#additional-meta-data).


### `setup.py` + `setup.cfg`
The problem with `setup.py` is that it is executable. To read the metadata, [setuptools](https://setuptools.pypa.io/en/latest/index.html) need to execute this script. For automation tools it its tool complicated and slow. To reflect that, the `setup.cfg` file (which is declarative by design) was introduced ans has become more popular for packaging. The advantage here is that the packaging software (e.g [setuptools](https://setuptools.pypa.io/en/latest/index.html)) doesn't need to _evaluate_ a Python file to get the metadata, but can simple _parse_ a readable configuration file.

Example
```python
# setup.py
from setuptools import setup

if __name__ == "__main__":
    setup()
```
```yaml
# setup.cfg
[metadata]
name = python_example

[options]
packages = find_namespace:
include_package_data = True
package_dir =
    =src

[options.packages.find]
where = src
```

### `pyproject.toml`
As we said above, the `pyproject.toml` is the specified file format of [PEP 517](https://peps.python.org/pep-0517/) which defines what build tools are needed to build our package. It is not strictly meant to replace `setup.py`, but rather to ensure its correct execution if it's still needed. 

- For `pyproject.toml` with [Flit](https://flit.pypa.io/en/stable/), refer [here](https://godatadriven.com/blog/minimal-pyproject-toml-example/)
- For `pyproject.toml` with [Poetry](https://python-poetry.org/), refer [here](https://python-poetry.org/docs/pyproject/)
- For `pyproject.toml` with [setuptools](https://setuptools.pypa.io/en/latest/index.html), refer [here](https://godatadriven.com/blog/a-practical-guide-to-setuptools-and-pyproject-toml/)

Example
```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
```

## Distributions
There are two types of distribution in Python
- [Source distribution](https://packaging.python.org/en/latest/glossary/#term-Source-Distribution-or-sdist) - `sdist`
    - A `sdist` contains source code that includes not only Python code but also the source code of any extension modules (usually in C or C++) bundled with the package.
    - The `sdist` will NOT INCLUDE **platform-specific binaries**. That means the end-users cannot consume it immediately - they have to build the packaged themselves once they download the `sdist`.  
    - Simple cmd to generate `sdist`
        - `python setup.py sdist`
        - `python setup.py sdist --formats=gztar,zip` if specify formats

- [Built distribution](https://packaging.python.org/en/latest/glossary/#term-Built-Distribution) - `bdist`
    - A `bdist` contains principally `.so`, `.dll`, `.dylib` for **binary modules**.
    - Installing a `bdist` in the end-users is immediate, as they don't need to build anything - from their perspective is that there's NO build stage.
    - The downside is that the package author have to build for multiple platforms and versions and upload all of the distributions for max compatibility - `bdist` is **platform-specific**.
    - `bdist` has taken **different formats** along the years. It was [eggs](https://packaging.python.org/en/latest/glossary/#term-Egg) (`.egg`) at a time but nowadays it is [wheel](https://packaging.python.org/en/latest/glossary/#term-Wheel) (`.whl`) format.
    - When a package provides both a [wheel](https://packaging.python.org/en/latest/glossary/#term-Wheel) and a [source distribution](https://packaging.python.org/en/latest/glossary/#term-Source-Distribution-or-sdist), `pip install` will prefer the [wheel](https://packaging.python.org/en/latest/glossary/#term-Wheel) if it’s compatible with the end-user's system. 
    - Simple cmd to generate `bdist`
        - `python setup.py bdist`
        - `python setup.py bdist --format=zip` 
        - `python setup.py bdist_wheel` - building a Pure-Python [wheel](https://packaging.python.org/en/latest/glossary/#term-Wheel)
     
[Different Types of Wheels](https://realpython.com/python-wheels/#different-types-of-wheels)
- A **universal** wheel contains py2.py3-none-any.whl. It **supports both Python 2 and Python 3 on any OS and platform**. The majority of wheels listed on the Python Wheels website are universal wheels.
- A **pure-Python** wheel contains either py3-none-any.whl or py2.none-any.whl. It supports **either Python 3 or Python 2, but not both**. It’s otherwise the same as a universal wheel, but it’ll be labeled with either py2 or py3 rather than the py2.py3 label.
- A **platform** wheel supports a specific Python version and platform. It contains segments indicating a specific Python version, ABI, operating system, or architecture.



## References
- [1] [A Practical Guide to Using Setup.py](https://godatadriven.com/blog/a-practical-guide-to-using-setup-py/)
- [2] [A Practical Guide to Setuptools and Pyproject.toml](https://godatadriven.com/blog/a-practical-guide-to-setuptools-and-pyproject-toml/)
- [3] [Python Packaging with Setuptools](https://itnext.io/python-packaging-12ef040c4ea0)
- [4] [What's the difference between setup.py and setup.cfg in python projects](https://stackoverflow.com/questions/39484863/whats-the-difference-between-setup-py-and-setup-cfg-in-python-projects)
