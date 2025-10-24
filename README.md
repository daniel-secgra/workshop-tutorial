Here's a step-by-step tutorial based on your commands, formatted for a `README.md` file.

---

This guide will walk you through setting up the project environment using **`uv`** and running the Jupyter notebooks.

## Prerequisites

Before you begin, ensure you have > **Python 3.9+** installed on your system. You can check this by running:

```bash
python3 --version
```

---

## 🚀 Setup Instructions

Follow these steps to create a dedicated environment and install all necessary dependencies.

### 1\. Get the Project

First, clone this repository (or download the source code) and navigate into the project directory.

```bash
git clone [your-repository-url]
cd [your-project-directory]
```

### 2\. Install `uv`

This project uses **`uv`**, a very fast Python package installer. If you don't have it, install it using `pip`:

```bash
pip install uv
```

### 3\. Create the Virtual Environment

Use `uv` to create a new virtual environment. This will create a `.venv` directory to store your project's packages.

```bash
uv venv
```

### 4\. Activate the Environment

You must activate the environment to use it.

**On macOS / Linux:**

```bash
source .venv/bin/activate
```

**On Windows (CMD):**

```bash
.venv\Scripts\Activate
```

You'll know it's active when you see `(.venv)` or **tutorial** at the beginning of your terminal prompt.

### 5\. Install Dependencies

With the environment active, use `uv sync` to install all required packages from the `requirements.txt` (or `pyproject.toml`) file.

```bash
uv sync
```

### 6\. Link Environment to Jupyter

To make your new environment available as an option inside Jupyter Notebook, run the following command. This creates a new "kernel" named `myproject_kernel`.

```bash
python3 -m ipykernel install --user --name=myproject_kernel
```

_(You can replace `myproject_kernel` with any name you prefer, just remember it for the next section.)_

---

## 🔬 How to Run

Once everything is installed, you're ready to run the notebooks.

1.  **Start Jupyter Notebook**
    Make sure your virtual environment is still active and run:

    ```bash
    jupyter notebook
    ```

2.  **Open Your Notebook**
    Your web browser will open. Navigate to and click on any `.ipynb` file in the project directory to open it.

3.  **Select the Kernel**
    This is the most important step\! Once the notebook is open:

    - Go to the **Kernel** menu at the top.
    - Click on **Change kernel**.
    - Select **`myproject_kernel`** (or the name you chose in step 6).

You're all set\! You can now run the cells in the notebook using the isolated environment you just created.
