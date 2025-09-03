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

Baseline 1: Code Name ``Chordino``

Download sonic annotator and [NNLS Chroma 1.1](https://code.soundsoftware.ac.uk/projects/nnls-chroma/files]

```python
import os
import subprocess
from joblib import Parallel, delayed
from tqdm import tqdm

year = 2025

# Define the pure/major interval mapping
INTERVAL_MAPPING = [
    ['1', 0],  # Unison
    ['2', 2],  # Major Second
    ['3', 4],  # Major Third
    ['4', 5],  # Perfect Fourth
    ['5', 7],  # Perfect Fifth
    ['6', 9],  # Major Sixth
    ['7', 11]  # Major Seventh
]


def get_interval_robust(note1, note2):
    # note1, note2 are formatted as 'C', 'C#', 'D', etc.
    scale1 = 'C.D.EF.G.A.B'.index(note1[0]) + note1.count('#') - note1.count('b')
    scale2 = 'C.D.EF.G.A.B'.index(note2[0]) + note2.count('#') - note2.count('b')
    interval = (scale2 - scale1) % 12

    # Need to check real scale degree
    degree1 = 'CDEFGAB'.index(note1[0])
    degree2 = 'CDEFGAB'.index(note2[0])
    degree_interval = (degree2 - degree1) % 7

    interval_name, pure_interval = INTERVAL_MAPPING[degree_interval]
    if interval > pure_interval:
        interval_name = '#' * (interval - pure_interval) + interval_name
    elif interval < pure_interval:
        interval_name = 'b' * (pure_interval - interval) + interval_name
    return interval_name

CMD = [
    r'D:\Program Files (x86)\Sonic Visualiser\annotator\sonic-annotator.exe',
    '-d', 'vamp:nnls-chroma:chordino:simplechord',
    '-w', 'lab',
    '--lab-fill-ends',
    '--lab-stdout',
]

def process_file(input_path, output_path):
    os.makedirs(os.path.dirname(output_path), exist_ok=True)
    command = CMD + [input_path]
    # Store the results into a variable
    result = subprocess.run(command, capture_output=True, text=True)
    f = open(output_path, 'w')
    for line in result.stdout.strip().split('\n'):
        start_time, end_time, chord = line.split('\t')
        chord = chord.replace('"', '')
        if chord != 'N':
            if '/' in chord:
                root_quality, bass = chord.split('/')
            else:
                root_quality = chord
                bass = None
            if root_quality.endswith('dim7'):
                root, quality = root_quality[:-4], 'dim7'
            elif root_quality.endswith('m7b5'):
                root, quality = root_quality[:-4], 'hdim7'
            elif root_quality.endswith('maj7'):
                root, quality = root_quality[:-4], 'maj7'
            elif root_quality.endswith('dim'):
                root, quality = root_quality[:-3], 'dim'
            elif root_quality.endswith('aug'):
                root, quality = root_quality[:-3], 'aug'
            elif root_quality.endswith('m7'):
                root, quality = root_quality[:-2], 'min7'
            elif root_quality.endswith('m6'):
                root, quality = root_quality[:-2], 'min6'
            elif root_quality.endswith('m'):
                root, quality = root_quality[:-1], 'min'
            elif root_quality.endswith('7'):
                root, quality = root_quality[:-1], '7'
            elif root_quality.endswith('6'):
                root, quality = root_quality[:-1], 'maj6'
            else:
                root, quality = root_quality, 'maj'
            if bass is None:
                chord = f'{root}:{quality}'
            else:
                interval = get_interval_robust(root, bass)
                chord = f'{root}:{quality}/{interval}'
        f.write(f'{start_time} {end_time} {chord}\n')
    f.close()


def inference(input_folder, output_folder):
    tasks = []
    for file in os.listdir(input_folder):
        if file.endswith('.wav'):
            input_path = os.path.join(input_folder, file)
            output_path = os.path.join(output_folder, f'{file[:-4]}.lab')
            tasks.append([input_path, output_path])
    Parallel(n_jobs=-1)(delayed(process_file)(task[0], task[1]) for task in tqdm(tasks))

if __name__ == '__main__':
    for folder in os.listdir(os.path.join('..', 'ace-data')):
        inference(os.path.join('..', 'ace-data', folder), os.path.join('..', 'ace-output', str(year), folder, 'Chordino'))

```

Baseline 2: Code Name ``ISMIR2019``

Clone from https://github.com/music-x-lab/ISMIR2019-Large-Vocabulary-Chord-Recognition
```python
import numpy as np
np.int = int
from chord_recognition import chord_recognition
import os
from tqdm import tqdm

year = 2025

def inference(audio_path, output_path):
    os.makedirs(output_path, exist_ok=True)
    for file in tqdm(os.listdir(audio_path)):
        if file.endswith('.wav'):
            chord_recognition(
                os.path.join(audio_path, file),
                os.path.join(output_path, file[:-4] + '.lab'),
                chord_dict_name='ismir2017'
            )

if __name__ == '__main__':
    for folder in os.listdir(os.path.join('..', 'ace-data')):
        inference(os.path.join('..', 'ace-data', folder), os.path.join('..', 'ace-output', str(year), folder, 'Chordino'))

```

