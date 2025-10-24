This guide will walk you through setting up the project environment using **`uv`** and running the Jupyter notebooks.

## Prerequisites

Before you begin, ensure you have \> **Python 3.9+** installed on your system. You can check this by running:

```bash
python3 --version
```

---

## 🚀 Setup Instructions

Follow these steps to create a dedicated environment and install all necessary dependencies.

### 1\. Get the Project

First, clone this repository (or download the source code) and navigate into the project directory.

```bash
git clone https://github.com/daniel-secgra/workshop-tutorial.git
cd workshop-tutorial
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

**On Mac/Linux:**

```bash
python3 -m ipykernel install --user --name=myproject_kernel
```

---

**On Windows (CMD):**

```bash
python -m ipykernel install --user --name=myproject_kernel
```

_(You can replace `myproject_kernel` with any name you prefer, just remember it for the next section.)_

---

## 🔒 AWS Credentials

Before running the notebooks, you must configure your AWS credentials so the code can access your AWS account. Choose **one** of the methods below.

### Method 1: Set in Terminal (Temporary)

This method sets your credentials as environment variables for your current terminal session. They will be lost when you close the terminal.

**On macOS / Linux:**

```bash
export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY"
export AWS_SECRET_ACCESS_KEY="YOUR_SECRET_KEY"
# Optional: If using temporary credentials, also set the session token
# export AWS_SESSION_TOKEN="YOUR_SESSION_TOKEN"
```

**On Windows (CMD):**

```bash
set AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY"
set AWS_SECRET_ACCESS_KEY="YOUR_SECRET_KEY"
# Optional: If using temporary credentials, also set the session token
# set AWS_SESSION_TOKEN="YOUR_SESSION_TOKEN"
```

### Method 2: Create AWS Credentials File (Recommended)

This method securely stores your credentials in a dedicated file, which is the standard practice.

1.  Find or create the AWS credentials file.

    - **On macOS / Linux:** `~/.aws/credentials`
    - **On Windows:** `C:\Users\YOUR_USERNAME\.aws\credentials`

2.  If the file (or the `.aws` directory) doesn't exist, create it.

3.  Add the following content to the file, replacing the placeholder values with your actual credentials.

    ```ini
    [default]
    aws_access_key_id = YOUR_ACCESS_KEY
    aws_secret_access_key = YOUR_SECRET_KEY
    # Optional: If using temporary credentials, also add the session token
    # aws_session_token = YOUR_SESSION_TOKEN
    ```

**Important:** After setting your credentials using either method, you must start Jupyter Notebook **from the same terminal** where you set them (if using Method 1) or from a new terminal (if using Method 2).

---

## 🔬 How to Run

Once everything is installed and your credentials are set, you're ready to run the notebooks.

1.  **Start Jupyter Notebook**
    Make sure your virtual environment is still active and run:

    ```bash
    jupyter notebook
    ```

2.  **Open Your Notebook**
    Your web browser will open. Navigate to and click on any `.ipynb` file in the project directory to open it.


You're all set\! You can now run the cells in the notebook using the isolated environment you just created.
