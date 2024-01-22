---
layout: post
title:  "Creating a custom conda environment with specific python version"
date:   2024-01-22 16:37:00
subtitle: 
img:
tags: [python,conda,environment] # add tag
published: true
---
### Creating a new environment with a specific python version and then add it as a kernel in juypyterlab

```bash
conda create -n environment_name python=3.10.13
source activate environment_name
pip install ipykernel
python -m ipykernel install --user --name environment_name
```
Source: [https://stackoverflow.com/a/28840041](https://stackoverflow.com/a/28840041){:target="_blank"}

### Removing a kernel from the current jupyterlab environment list
```bash
jupyter kernelspec uninstall environment_name
```

### Uninstall an environemnt from conda
First list all the environments to see if the environment exist in conda
```bash
conda env list
```
If it exists then unistall the environment with all the installed packages using the following command
```
conda remove --name environment_name --all
```
Source: [https://www.freecodecamp.org/news/how-to-delete-an-environment-in-conda/](https://www.freecodecamp.org/news/how-to-delete-an-environment-in-conda/){:target="_blank"}
