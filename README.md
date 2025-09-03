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

## Baselines

Baseline 1: Code Name ``ISMIR2019``

Clone from https://github.com/music-x-lab/ISMIR2019-Large-Vocabulary-Chord-Recognition
```python
import numpy as np
np.int = int
from chord_recognition import chord_recognition
import os
from tqdm import tqdm
audio_path = ...
output_path = ...
if __name__ == '__main__':
    os.makedirs(output_path, exist_ok=True)
    for file in tqdm(os.listdir(audio_path)):
        if file.endswith('.wav'):
            chord_recognition(
                os.path.join(audio_path, file),
                os.path.join(output_path, file[:-4] + '.lab'),
                chord_dict_name='ismir2017'
            )

```

