# Setting Up Python Environment with Miniconda and JupyterLab

This guide will help you set up a Python environment using Miniconda and install JupyterLab. Follow the steps below:

## 1. Download Miniconda

- Visit the Miniconda download page: [Miniconda Downloads](https://docs.conda.io/en/latest/miniconda.html)
- Download the appropriate Miniconda installer for your operating system:
  - **Windows**: `Miniconda3 Windows 64-bit`
  - **macOS**: `Miniconda3 macOS 64-bit`
  - **Linux**: `Miniconda3 Linux 64-bit`

## 2. Installation

- Run the Miniconda installer that you downloaded.
- During the installation, **make sure to check the box to add Miniconda to your PATH** (this will allow you to use `conda` and `python` from the command line).
- For other options, you can leave the default settings as is.

## 3. Create Python Environment

Once Miniconda is installed, you can create a Python environment using the following commands:

1. Open a terminal (or Anaconda Prompt on Windows).
2. Run the following command to create a Python 3.11 environment:

   ```bash
   conda create -n p311 python=3.11
This will create a new environment named p311 with Python 3.11.

Activate the newly created environment:

conda activate p311
4. Install JupyterLab
Now that you're in the p311 environment, install JupyterLab:
```bash
pip install jupyterlab
```
5. Start JupyterLab
To start JupyterLab, simply run:
```bash
jupyter-lab
```
This will open JupyterLab in your web browser, where you can create and run notebooks.

6. Install required packages from requirement.txt

```bash
pip install -r requirements.txt
```

7. Create a Batch File to Automate Starting JupyterLab
To make it easier to start JupyterLab with your environment activated, you can create a .bat file that does this automatically.

Open a text editor (e.g., Notepad) and paste the following code:
```bash
@echo off
REM Activate the Miniconda environment 'p311'
call conda activate p311
REM Start JupyterLab
jupyter-lab
```
Save the file with a .bat extension, for example, s.bat.
