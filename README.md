# ace-task-captain-note

## Clone the evaluation tool

```bash
git clone https://github.com/jpauwels/MusOOEvaluator --recursive
```

If ``tinyxml2`` cannot be cloned:

```bash
# Update URL in repo config of the submodule
git config submodule.libMusOO/tinyxml2.url https://github.com/leethomason/tinyxml2.git

# Resync
git submodule sync --recursive

# Retry
git submodule update --init --recursive
```

## Build the evaluation tool

In Ubuntu:
```bash
conda create -n mirex-eval make
conda activate mirex-eval
conda install -n mirex-eval gxx_linux-64 boost make sysroot_linux-64
cd MusOOEvaluator/build/linux-gmake/
make LDFLAGS="-L$CONDA_PREFIX/lib -L$CONDA_PREFIX/x86_64-conda-linux-gnu/sysroot/usr/lib64"
```





