# AGENTS.md

## Cursor Cloud specific instructions

This is the **tensorflow/models** repository — a collection of ML model examples targeting TensorFlow 1.x (1.15). It is NOT a web app or service; each model (official, research, tutorials, samples) is a standalone Python project.

### Environment

- **Python 3.7** is installed via deadsnakes PPA; a virtualenv lives at `/workspace/.venv`.
- **TensorFlow 1.15.5** with `protobuf<4` (required for compatibility).
- Activate the environment: `source /workspace/.venv/bin/activate`
- **PYTHONPATH** must include the repo root for imports to work: `export PYTHONPATH="/workspace:$PYTHONPATH"`

### Running Official Models

See `official/README.md` for the list. Example (MNIST, 1-epoch training):

```
source /workspace/.venv/bin/activate
export PYTHONPATH="/workspace:$PYTHONPATH"
python official/mnist/mnist.py --train_epochs=1
```

### Lint

```
source /workspace/.venv/bin/activate
export PYTHONPATH="/workspace:$PYTHONPATH"
pylint --disable=all --enable=E official/mnist/
```

### Tests

```
source /workspace/.venv/bin/activate
export PYTHONPATH="/workspace:$PYTHONPATH"
python -m pytest official/mnist/mnist_test.py -v
```

### Gotchas

- TF 1.15 requires `protobuf<4`. The update script pins this.
- Some `official/utils` tests have minor failures from TF 1.x deprecations — these are pre-existing, not caused by the environment.
- The repo has no `__init__.py` files; `PYTHONPATH` to the repo root is the supported import mechanism.
- Research models may have extra dependencies (Bazel, Cython, protoc, etc.) — install per-model as needed.
