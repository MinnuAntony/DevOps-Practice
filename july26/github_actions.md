## What are GitHub Actions?
- GitHub Actions is a CI/CD platform that allows you to automate software workflows directly in your GitHub repository.
- You can build, test, and deploy your code right from GitHub.


Got it ✅ — I’ll walk you through your **minimal YAML file** line by line,
explaining **what each line does**, **why it’s used**, and **why the syntax looks that way**.

---

## **Workflow Header**

```yaml
name: First Flask CI + Temporary Local Run
```

* **Purpose:** Gives your workflow a **readable name** that appears in the GitHub Actions UI.
* **Why used:** Helps identify this workflow in the Actions list.
* **Why syntax:** `name:` is a top‑level YAML key; value is a string.

---

```yaml
on:
  push:
    branches:
      - main
```

* **Purpose:** Defines **when** this workflow runs.
* **Why used:** We want it to trigger when new code is pushed to the `main` branch.
* **Why syntax:**

  * `on:` → event keyword (push, pull\_request, schedule, etc.).
  * `push:` → specific event type.
  * `branches:` → list of branches that will trigger it.
  * `- main` → YAML list syntax; dash means “item in a list”.

---

## **Jobs Section**

```yaml
jobs:
  build-and-run:
    runs-on: self-hosted
```

* **Purpose:** Declares a job called **`build-and-run`** that will run in your **self‑hosted EC2 runner**.
* **Why used:** Jobs group steps that must run together. `runs-on: self-hosted` tells Actions to pick your registered EC2 runner.
* **Why syntax:**

  * `jobs:` → YAML map; each key under it is a job name.
  * `runs-on:` → special key that sets the runner environment.
  * `self-hosted` is a reserved label meaning “use my registered self-hosted runner”.

---

## **Steps Section**

```yaml
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
```

* **Purpose:** Pulls your repo into the runner’s working directory.
* **Why used:** Without this, the runner has no copy of your source code.
* **Why syntax:**

  * `steps:` → list of actions or commands to run in order.
  * Each step starts with `-` (YAML list).
  * `name:` → human-friendly label for the step.
  * `uses:` → keyword for referencing a pre-built GitHub Action (here, `actions/checkout@v4`).

---

```yaml
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'
```

* **Purpose:** Installs and configures Python 3.10 for the job.
* **Why used:** Ensures your Python app runs in the intended Python version.
* **Why syntax:**

  * `uses:` → again, using a published GitHub Action.
  * `with:` → passes **input parameters** to that Action.
  * `python-version: '3.10'` → version is in quotes because YAML can misinterpret numbers with dots.

---

```yaml
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
```

* **Purpose:** Installs required Python packages for your app (Flask, pytest, etc.).
* **Why used:** Makes sure all dependencies from `requirements.txt` are available.
* **Why syntax:**

  * `run:` → tells GitHub Actions to execute shell commands directly.
  * `|` → YAML block scalar meaning “treat the following lines as one multi-line string”.
  * Each line inside runs in the same shell.

---

```yaml
      - name: Run tests
        run: pytest
```

* **Purpose:** Runs your `pytest` test suite.
* **Why used:** Catches bugs before starting the Flask server.
* **Why syntax:**

  * Simple `run:` with one command.

---

```yaml
      - name: Start Flask for testing (auto-stop)
        run: |
          flask run --host=127.0.0.1 --port=5000 &
          FLASK_PID=$!
          sleep 5
          echo "Testing Flask response..."
          curl --fail http://127.0.0.1:5000 || (echo "Flask did not respond" && kill $FLASK_PID && exit 1)
          sleep 20
          kill $FLASK_PID
          echo "Flask stopped."
```

### Line-by-line:

1. **`flask run --host=127.0.0.1 --port=5000 &`**

   * Runs Flask in background mode (`&`).
   * `--host=127.0.0.1` → bind to localhost only.
   * `--port=5000` → use port 5000.

2. **`FLASK_PID=$!`**

   * `$!` → shell variable for “PID of last background process”.
   * Saves Flask’s PID so we can stop it later.

3. **`sleep 5`**

   * Wait 5 seconds to give Flask time to start.

4. **`echo "Testing Flask response..."`**

   * Print message to log (optional but helps in Action logs).

5. **`curl --fail http://127.0.0.1:5000 || ( ... )`**

   * Tries to request the homepage.
   * `--fail` makes `curl` return a non‑zero exit code if HTTP status ≥ 400.
   * `|| (...)` → shell “OR” syntax: if curl fails, run the commands inside `( )`.

6. **`echo "Flask did not respond" && kill $FLASK_PID && exit 1`**

   * Inside the failure branch: log error, kill Flask process, and exit with failure status.

7. **`sleep 20`**

   * Keep Flask alive for 20 seconds for manual/extra testing (you used 2 minutes earlier).

8. **`kill $FLASK_PID`**

   * Stops Flask by killing the process ID we saved.

9. **`echo "Flask stopped."`**

   * Log confirmation.

---


