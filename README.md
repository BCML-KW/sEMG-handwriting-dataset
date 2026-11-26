# sEMG Handwriting Dataset

Synchronized surface electromyography (sEMG) and pen-tablet recordings of 11 subjects
writing the 26 lowercase English letters. Each file holds one handwritten character:
8-channel forearm sEMG together with the pen trajectory and pen pressure recorded at
the same time.

The dataset accompanies the paper *Velocity-Constrained Representation Learning for
sEMG-Based Handwriting Recognition* (see [Citation](#citation)).

## At a glance

| | |
|---|---|
| Subjects | 11 (IDs `100`–`110`) |
| Classes | 26 lowercase letters, `a`–`z` |
| Samples per class per subject | 50 |
| Samples per subject | 1,300 |
| Total samples (files) | 14,300 |
| Total rows | 3,650,004 |
| Sample duration | 0.97–3.70 s (median 1.20 s) |
| Rows per sample | 200–742 (median 242) |
| sEMG | 8 channels, 200 Hz nominal |
| Size | about 340 MB |

## Folder layout

```
100/                  one folder per subject
    100_00_00.csv
    100_00_01.csv
    ...
    100_25_49.csv
101/
...
110/
splits.csv            train / validation / test split used in the paper
LICENSE
README.md
```

## File naming

`SSS_CC_NN.csv`

| Part | Meaning |
|---|---|
| `SSS` | Subject ID, `100`–`110` |
| `CC` | Letter class, `00`–`25`, in alphabetical order (`00` = a, `01` = b, ..., `25` = z) |
| `NN` | Repetition index of that letter for that subject, `00`–`49` |

Example: `103_07_12.csv` is subject 103 writing the letter `h`, repetition 12.

## File format

Each CSV has a header row and 12 columns. One row is one sEMG sample.

| Column | Unit | Description |
|---|---|---|
| `timestamp` | seconds | Arrival time of the sEMG sample on the recording PC (`time.perf_counter()`). The origin is arbitrary and differs between recording sessions; only differences within a file are meaningful. |
| `ch1`–`ch8` | raw device units | Raw sEMG from the eight Myo armband electrodes, signed 8-bit integers in [-128, 127]. No filtering was applied. |
| `tablet_x` | pixels | Horizontal pen position relative to the first contact point of the character. |
| `tablet_y` | pixels | Vertical pen position relative to the first contact point of the character. Positive values point downward (screen coordinates). |
| `pressure` | normalized, 0–1 | Pen pressure reported by the tablet driver. |

Every file starts at `tablet_x = 0`, `tablet_y = 0`, because the coordinates are stored
relative to the point where the pen first touched the tablet.

### Notes on timing

- A file covers one character, from pen-down to pen-up.
- The Myo armband transmits two sEMG samples per Bluetooth packet, and samples are
  timestamped when they arrive. Samples therefore appear in closely spaced pairs
  rather than at uniform 5 ms intervals. The mean rate is 200 Hz.
- The tablet reports pen data at about 133 Hz. Pen position and pressure were linearly
  interpolated onto the sEMG timestamps, so every row carries both modalities.

## Data acquisition

**sEMG**

- Device: Thalmic Labs Myo armband, 8 channels, 200 Hz, raw mode
- Placement: worn on the forearm of the writing arm, with the same orientation in all
  recording sessions

**Pen tablet**

- Device: Wacom CTL-472 with the manufacturer-provided stylus
- Recorded signals: 2D pen position and pen pressure, about 133 Hz

**Protocol**

- Subjects wore the armband on the writing arm and wrote on the tablet with the stylus.
- Letters were written in fixed alphabetical order, `a` to `z`.
- 25 repetitions of each letter were recorded in one cycle. The cycle was repeated
  after a 5-minute rest, giving 50 repetitions per letter.
- Subjects could take short breaks between repetitions.
- Every character was written as a single continuous stroke. Because character
  boundaries are defined by pen-tablet contact, subjects were shown single-stroke
  forms for `f`, `i`, `j`, `k`, `t` and `x`. No other constraint was placed on
  handwriting style.
- Writing speed was unconstrained, provided each sample lasted about 1 s or longer.

## Subjects

Eleven healthy volunteers took part (9 male, 2 female), aged 24 to 37 years. All were
right-handed. Subject IDs are arbitrary codes and carry no identifying information.

## Train / validation / test split

`splits.csv` lists the split used in the paper, one row per file:

| Column | Description |
|---|---|
| `subject` | Subject ID |
| `file` | File name |
| `split` | `train`, `val` or `test` |

The split is intra-subject and stratified by class: for each subject and each letter,
30 samples are for training, 10 for validation and 10 for testing (8,580 / 2,860 /
2,860 files in total). In the few-shot experiments of the paper, the K training
samples per class were drawn from the `train` portion, and the `val` and `test`
portions were kept fixed.

## Loading example

```python
import pandas as pd

df = pd.read_csv("100/100_00_00.csv")
emg = df[[f"ch{i}" for i in range(1, 9)]].to_numpy()   # (T, 8)
xy = df[["tablet_x", "tablet_y"]].to_numpy()           # (T, 2)

splits = pd.read_csv("splits.csv")
train_files = splits[(splits.subject == 100) & (splits.split == "train")].file
```

## Ethics

The study was approved by the Institutional Review Board of Kwangwoon University
(Approval No. 7001546-202401004-HR(SB)-009-03). All participants gave written informed
consent before taking part.

## Citation

If you use this dataset, please cite:

> J. Won, H. Kim, J. Lee, J. Lee, S. Yeon, C. Park, Velocity-Constrained Representation
> Learning for sEMG-Based Handwriting Recognition, manuscript under review, 2026.

The reference will be updated when the article is published.

## License

The dataset is released under the
[Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)
(CC BY 4.0). See `LICENSE`.

## Contact

Bio Computing & Machine Learning Lab (BCML), Department of Computer Engineering, Kwangwoon
University, Seoul, Republic of Korea. Corresponding author: Cheolsoo Park
(parkcheolsoo@kw.ac.kr).
