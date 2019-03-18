# Anaconda And Conda

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes

$ cat ~/.condarc
channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - defaults
ssl_verify: true
show_channel_urls: true

conda --version
conda -h
conda update conda

conda update --all

conda create -n <env_name> <package_names>
conda create -n study numpy matplotlib 

~/anaconda3/env

source activate <env_name>
source activate study

source deactivate

conda env list
conda info --envs
conda info -e

conda remove -n <env_name> --all

conda search --full-name <package_full_name>
conda search <text>
conda list
conda install -n <env_name> <package_name>
conda install <package_name>
conda remove -n <env_name> <package_name>
conda remove <package_name>

conda clean -a

rm -rf ~/anaconda3

conda install nb_conda
conda install -n study ipykernel
```

