<br>
<p align="center">
  <img width=384 src="https://download.nap.tech/identity/svg/logos/nap_logo_blue.svg">
</p>

Generates the NAP Framework system documentation, available at [docs.nap.tech](https://docs.nap.tech). Documentation includes the user manual and info on all available system resources. The user manual is generated from the `.md` files located in the `app/manual` directory. Framework documentation is extracted from [NAP source code](https://github.com/napframework/nap). Documentation is guaranteed to be in sync with the main branch.

| docs.nap.tech |
| ------------- |
[![Netlify Status](https://api.netlify.com/api/v1/badges/9eaa7116-9815-463b-8836-e1fc68b539a3/deploy-status)](https://app.netlify.com/sites/nap-docs/deploys)

## Dependencies
- [Python (3.8+)](https://www.python.org/downloads/) 
- [pip](https://pypi.org/project/pip/)
- [doxygen](https://doxygen.nl/)
- [graphviz](https://graphviz.org/)
- [git](https://git-scm.com/)
- [GitPython](https://github.com/gitpython-developers/GitPython/)

## Installation of the virtual python environment (pipenv)
Create a virtual python environment for the project and install all required python dependencies:

```shell
$ pip install --user pipenv
$ cd nap-docs
$ pipenv install
```

## Build using pipenv

Run the `build.py` script inside the `app` directory to auto-generate system documentation using `doxygen`. The script clones and pulls the NAP repo if required. The result will be copied into the `docs` directory.
```shell
$ pipenv shell
$ cd app
$ python build.py
$ exit
```

## Build without pipenv

Make sure all dependencies are installed first!
```shell
$ cd app
$ python build.py
```
