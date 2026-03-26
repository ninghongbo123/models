---
name: cloud-agents-starter-skill
description: Minimal Cloud-agent runbook for TensorFlow models repo with practical setup, auth, execution, feature-flag-style toggles, and area-specific test workflows.
---

# Cloud Agent Starter Skill (tensorflow/models)

Use this skill when you need to get productive quickly in this repo: bootstrap env, run a model entrypoint, and execute high-signal tests by codebase area.

## 1) Fast bootstrap (run first in a fresh terminal)

```bash
cd /workspace
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r official/requirements.txt
export PYTHONPATH="$PYTHONPATH:/workspace:/workspace/research:/workspace/research/slim"
```

### Login / auth (only when workflow needs remote services)

Most local tests in this repo do **not** require login.  
Use auth only for cloud/GCS workflows or private artifact access.

```bash
# Check current gcloud auth state
gcloud auth list

# If needed for cloud jobs / GCS reads
gcloud auth login --no-launch-browser
gcloud auth application-default login --no-launch-browser
```

## 2) Practical toggles ("feature flags" for local smoke runs)

This repo mostly uses CLI flags instead of a centralized feature-flag service.

- Force CPU-only path:
  ```bash
  export CUDA_VISIBLE_DEVICES=""
  ```
- Reduce noisy TF logs:
  ```bash
  export TF_CPP_MIN_LOG_LEVEL=2
  ```
- Make training smoke tests cheap:
  - `--train_epochs=1`
  - `--epochs_between_evals=1`
  - `--batch_size=<small>`
  - `--model_dir=/tmp/<run_name>`
  - For ResNet: `--use_synthetic_data=true`

## 3) Codebase-area workflows

### A. `official/` (recommended default starting point)

#### Start / run
```bash
cd /workspace
source .venv/bin/activate
export PYTHONPATH="$PYTHONPATH:/workspace"
python official/mnist/mnist.py \
  --train_epochs=1 \
  --epochs_between_evals=1 \
  --batch_size=64 \
  --model_dir=/tmp/official_mnist_smoke
```

#### Test workflow (high-signal + quick)
```bash
python official/mnist/mnist_test.py
python official/resnet/cifar10_test.py
```

When touching `official/resnet/*`, prefer one extra synthetic-data smoke run:
```bash
python official/resnet/cifar10_main.py \
  --use_synthetic_data=true \
  --train_epochs=1 \
  --epochs_between_evals=1 \
  --batch_size=128 \
  --model_dir=/tmp/official_resnet_smoke
```

---

### B. `research/object_detection/` (special setup required)

#### One-time setup before running tests
```bash
cd /workspace/research
source /workspace/.venv/bin/activate
protoc object_detection/protos/*.proto --python_out=.
export PYTHONPATH="$PYTHONPATH:/workspace:/workspace/research:/workspace/research/slim"
```

#### Test workflow (installation sanity + core builders)
```bash
python object_detection/builders/model_builder_test.py
python object_detection/utils/config_util_test.py
```

#### Optional local run skeleton
```bash
python object_detection/model_main.py \
  --pipeline_config_path=<pipeline_config.pbtxt> \
  --model_dir=/tmp/od_model_dir \
  --num_train_steps=100
```

If adding/changing protos, always re-run `protoc ...` before tests.

---

### C. `samples/languages/java/` (Java inference/training demos)

#### Start / run (`label_image`)
```bash
cd /workspace/samples/languages/java/label_image
mvn compile
# Then run with a real image path:
mvn -q exec:java -Dexec.args="<path-to-image>"
```

#### Test workflow
```bash
cd /workspace/samples/languages/java/label_image
mvn -q -DskipTests compile
cd /workspace/samples/languages/java/training
mvn -q -DskipTests compile
```

Use these as compile-time smoke checks when modifying Java sample code or scripts.

---

### D. `tutorials/` (lightweight tutorial examples)

#### Start / run
```bash
cd /workspace
source .venv/bin/activate
export PYTHONPATH="$PYTHONPATH:/workspace"
python tutorials/embedding/word2vec.py
```

#### Test workflow
```bash
python tutorials/embedding/word2vec_test.py
python tutorials/image/cifar10/cifar10_input_test.py
```

For RNN tutorial changes, also run:
```bash
python tutorials/rnn/ptb/reader_test.py
```

## 4) Common failure patterns (quick fixes)

- `ImportError: No module named official...`  
  Re-export `PYTHONPATH` to include `/workspace`.
- `object_detection` proto import errors  
  Re-run `protoc object_detection/protos/*.proto --python_out=.` from `research/`.
- Missing Java build tools  
  Install JDK + Maven in the environment image before retrying Java sample workflows.
- Dataset download is slow/flaky  
  Prefer unit tests first; use synthetic-data flags for training smoke checks.

## 5) How to update this skill when new runbook tricks are found

Keep updates short and operational. For each newly discovered trick:

1. Add it under the relevant codebase area (`official`, `research`, `samples`, `tutorials`).
2. Include:
   - exact command(s),
   - when to use it,
   - expected success signal (for example: specific log line or generated file).
3. If it replaces an older step, delete the old step in the same commit.
4. Prefer smallest reliable smoke test over long end-to-end pipelines.

Suggested mini-template for additions:

~~~markdown
### New trick: <short name>
- Use when: <symptom/change scope>
- Command: `<exact command>`
- Success signal: <1 line>
~~~
