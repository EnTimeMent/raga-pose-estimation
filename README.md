# OpenPose for EnTimeMent

Library and CoLab script to facilitate running [OpenPose](https://github.com/CMU-Perceptual-Computing-Lab/openpose) and
to do some post-processing.

This library was created for the [EnTimeMent](https://entimement.dibris.unige.it) project.

<!-- TOC depthFrom:2 depthTo:2 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Running the CoLab script](#running-the-colab-script)
- [Installation of `entimement_openpose` python library](#installation-of-entimementopenpose-python-library)
- [Running the command line script](#running-the-command-line-script)
- [CSV format](#csv-format)

<!-- /TOC -->

## Running the CoLab script
Open [OpenPose_Colab.ipynb](OpenPose_Colab.ipynb) and click 'Run in CoLab'.

## Installation of `entimement_openpose` python library
Install dependencies of entimement_openpose using [conda](https://docs.conda.io/projects/conda/en/latest/index.html) or [miniconda](https://docs.conda.io/en/latest/miniconda.html):

```
conda env create -f environment.yml
```

Alternatively use `pip` to install the packages listed in [`environment.yml`](environment.yml).

## Running the command line script

```
$ python run_openpose.py --help
Usage: run_openpose.py [OPTIONS]

  Runs openpose on the video, does post-processing, and outputs CSV files.

Options:
  --input-video TEXT            Path to the video file on which to run
                                openpose

  --input-json TEXT             Path to a directory of previously generated
                                openpose json files

  --output-dir TEXT             Path to the directory in which to output CSV
                                files (and videos if required).

  --openpose-dir TEXT           Path to the directory in which openpose is
                                installed.

  --create-model-video          Whether to create a video showing the poses on
                                a blank background

  --create-overlay-video        Whether to create a video showing the poses as
                                an overlay

  --width INTEGER               Width of original video (mandatory for
                                creating video if  not providing input-video)

  --height INTEGER              Height of original video (mandatory for
                                creating video if  not providing input-video)

  --confidence-threshold FLOAT  Confidence threshold. Items with a confidence
                                lower than the threshold will be replaced by
                                values from a previous frame.

  --body-parts TEXT             Body parts to include in output. Should be a
                                comma-separated list of strings as in the list
                                at https://github.com/CMU-Perceptual-
                                Computing-Lab/openpose/blob/master/doc/output.
                                md#keypoint-ordering-in-cpython, e.g.
                                "LEye,RElbow". Overrides --upper-body-parts
                                and --lower-body-parts.

  --upper-body-parts            Output upper body parts only
  --lower-body-parts            Output lower body parts only
  --flatten TEXT                Export CSV in flattened format, i.e. with a
                                single header row (see README)

  --help                        Show this message and exit.
```

### Examples

Run OpenPose on a video to produce output CSVs (1 per person) in the `output` directory
and an overlay video:

```bash
python run_openpose.py --input-video=example_files/example_1person/short_video.mp4 --openpose-dir=../openpose --output-dir=output --create-overlay-video=y
```

Parse existing JSON files created by OpenPose to produce 1 CSV per person in the `outpu` folder, showing only upper body parts:

```bash
python run_openpose.py --input-json=example_files/example_1person/output_json --output-dir=output --upper-body-parts
```

## CSV format

The CSVs which are output give the positions for a single person. They contain one line per frame, and columns showing the x, y and confidence for each body part.

With `flatten=False` (default), the CSV output looks like this (note that this is a minimal example showing only the eyes):

```csv
Body Part,LEye,LEye,LEye,REye,REye,REye
Variable,x,y,c,x,y,c
0,977.627,285.955,0.907646,910.046,271.252,0.92357
1,974.83,286.009,0.910094,909.925,271.277,0.925763
...
```

which in table form looks like this:

| Body Part | LEye     | LEye     | LEye      | REye     | REye     | REye      |
|-----------|----------|----------|-----------|----------|----------|-----------|
| **Variable**  | **x**        | **y**        | **c**         | **x**        | **y**        | **c**         |
| 0         | 977\.627 | 285\.955 | 0\.907646 | 910\.046 | 271\.252 | 0\.92357  |
| 1         | 974\.83  | 286\.009 | 0\.910094 | 909\.925 | 271\.277 | 0\.925763 |


If you export CSVs with `flatten=False`, they can be read back into a `pandas.DataFrame` as follows:

```python
df = pd.read_csv(csv_path, f), header=[0,1], index_col=0)
```

The dataframe will have `MultiIndex` columns to allow easier access to the values.

With `flatten=True`, the CSV output looks like this:

```csv
,LEye_x,LEye_y,LEye_c,REye_x,REye_y,REye_c
0,977.627,285.955,0.907646,910.046,271.252,0.92357
1,974.83,286.009,0.910094,909.925,271.277,0.925763
...
```

which in table form looks like:

|   | LEye\_x  | LEye\_y  | LEye\_c   | REye\_x  | REye\_y  | REye\_c   |
|---|----------|----------|-----------|----------|----------|-----------|
| 0 | 977\.627 | 285\.955 | 0\.907646 | 910\.046 | 271\.252 | 0\.92357  |
| 1 | 974\.83  | 286\.009 | 0\.910094 | 909\.925 | 271\.277 | 0\.925763 |
