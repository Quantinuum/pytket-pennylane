# pytket-pennylane

> [!WARNING]
> This extension is not currently maintained.

[![Slack](https://img.shields.io/badge/Slack-4A154B?style=for-the-badge&logo=slack&logoColor=white)](https://tketusers.slack.com/join/shared_invite/zt-18qmsamj9-UqQFVdkRzxnXCcKtcarLRA#)
[![Stack Exchange](https://img.shields.io/badge/StackExchange-%23ffffff.svg?style=for-the-badge&logo=StackExchange)](https://quantumcomputing.stackexchange.com/tags/pytket)

[Pytket](https://tket.quantinuum.com/api-docs/index.html) is a python module for interfacing
with tket, a quantum computing toolkit and optimising compiler developed by Quantinuum.

`pytket-pennylane` is an extension to `pytket` that allows `pytket` circuits to converted to pennylane.

See the PennyLane [documentation](https://pennylane.readthedocs.io) to get an intro to PennyLane.

Some useful links:

- [API Documentation](https://tket.quantinuum.com/extensions/pytket-pennylane/)

## Getting started

`pytket-pennylane` is compatible with Python versions 3.11 to 3.13 on Linux, MacOS
and Windows. To install, run:

```shell
pip install pytket-pennylane
```

This will install `pytket` if it isn't already installed, and add new classes
and methods into the `pytket.extensions` namespace.

## How to use

To use the integration once installed, initialise your pytket backend (in this example, an `AerBackend` which uses Qiskit Aer), and construct a PennyLane `PytketDevice` using this backend:

```python
import pennylane as qml
from pytket.extensions.qiskit import AerBackend

# initialise pytket backend
pytket_backend = AerBackend()

# construct PennyLane device
dev = qml.device(
    "pytket.pytketdevice",
    wires=2,
    pytket_backend=pytket_backend,
    shots=1000
)

# define a PennyLane Qnode with this device
@qml.qnode(dev)
def my_quantum_function(x, y):
    qml.RZ(x, wires=0)
    qml.RX(y, wires=1)
    return qml.expval(qml.PauliZ(0) @ qml.PauliZ(1))

# call the node
print(my_quantum_function(0.1, 0.2))

```

The example above uses the Pytket default compilation pass for the backend, you can change the optimisation
level of the default backend pass (0, 1 or 2) by setting the `optimisation_level` parameter:

```python
dev = qml.device(
    "pytket.pytketdevice",
    wires=2,
    pytket_backend=pytket_backend,
    optimisation_level=2,
    shots=1000
)
```

You can also use any Pytket [compilation pass](https://tket.quantinuum.com/user-manual/manual_compiler.html) using the `compilation_pass` parameter, which is used instead of the default pass:

```python
from pytket.passes import PauliSimp, SequencePass

# use a Chemistry optimised pass before the backend's default pass

custom_pass = SequencePass([PauliSimp(), pytket_backend.default_compilation_pass()])

dev = qml.device(
    "pytket.pytketdevice",
    wires=2,
    pytket_backend=pytket_backend,
    compilation_pass=custom_pass,
    shots=1000
)

```

## Bugs, support and feature requests

Please file bugs and feature requests on the Github
[issue tracker](https://github.com/Quantinuum/pytket-pennylane/issues).

## Development

To install an extension in editable mode run:

```shell
pip install -e .
```

We have set up the repo to be used with uv. You can use also:

```shell
uv sync --python 3.12
```

to install the package. This will automatically pick up all requirements for the tests as well.

## Contributing

Pull requests are welcome. To make a PR, first fork the repo, make your proposed
changes on the `main` branch, and open a PR from your fork. If it passes
tests and is accepted after review, it will be merged in.

### Code style

#### Formatting

All code should be formatted using
[ruff](https://docs.astral.sh/ruff/formatter/), with default options. This is
checked on the CI.

#### Type annotation

On the CI, [mypy](https://mypy.readthedocs.io/en/stable/) is used as a static
type checker and all submissions must pass its checks. You should therefore run
`mypy` locally on any changed files before submitting a PR. You can run them with:

```shell
uv run mypy --config-file=mypy.ini --no-incremental --explicit-package-bases pytket tests
```

#### Linting

We use [ruff](https://github.com/astral-sh/ruff) on the CI to check compliance with a set of style requirements (listed in `ruff.toml`).
You should run `ruff` over any changed files before submitting a PR, to catch any issues.

An easy way to meet all formatting and linting requirements is to issue `pre-commit run --all-files`.

If you are using uv running `uv run pre-commit run --all-files --show-diff-on-failure` will install the package and run all the checks.

### Tests

To run the tests for this module:

1. ensure you have installed `pytest`, `hypothesis`, and any modules listed in
   the `dev-dependencies` section of the `pyproject.toml` file;
   (If you are using uv this will be picked up automatically.)
2. run `pytest`.

When adding a new feature, please add a test for it. When fixing a bug, please
add a test that demonstrates the fix.
