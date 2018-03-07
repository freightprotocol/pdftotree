# pdftotree

[![GitHub issues](https://img.shields.io/github/issues/HazyResearch/pdftotree.svg)](https://github.com/HazyResearch/pdftotree/projects/2)
[![GitHub license](https://img.shields.io/github/license/HazyResearch/pdftotree.svg)](https://github.com/HazyResearch/pdftotree/blob/master/LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/HazyResearch/pdftotree.svg)](https://github.com/HazyResearch/pdftotree/stargazers)
[![Travis](https://img.shields.io/travis/HazyResearch/pdftotree.svg)](https://travis-ci.org/HazyResearch/pdftotree)
[![PyPI](https://img.shields.io/pypi/v/pdftotree.svg)](https://pypi.python.org/pypi/pdftotree)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/pdftotree.svg)](https://pypi.python.org/pypi/pdftotree)

[Fonduer](https://hazyresearch.github.io/snorkel/blog/fonduer.html) has been
successfully extended to perform information extraction from richly formatted
data such as tables. A crucial step in this process is the construction of the
hierarchical tree of context objects such as text blocks, figures, tables, etc.
The system currently uses PDF to HTML conversion provided by Adobe Acrobat.
However, Adobe Acrobat is not an open source tool, which may be inconvenient
for Fonduer users.

This package is the result of building our own module as replacement to Adobe
Acrobat. Several open source tools are available for pdf to html conversion but
these tools do not preserve the cell structure in a table. Our goal in this
project is to develop a tool that extracts text, figures and tables in a pdf
document and maintains the structure of the document using a tree data
structure.

## Dependencies

```
sudo apt-get install python3-tk
```

## Installation

To install this package from PyPi:

```
pip install pdftotree
```

Or, to install directly from this repository. Clone this repo and run:
```
python setup.py install
```

## Usage

### pdftotree as a Python package

```py
import pdftotree

pdftotree.parse(pdf_file, html_path=None, model_path=None, favor_figures=True, visualize=False):
```

### pdftotree

This is the primary command-line utility provided with this Python package.
This takes a PDF file as input, and produces an HTML-like representation of the
data.

```
usage: pdftotree [options] pdf_file

Script to extract tree structure from PDF files. Takes a PDF as input and
outputs an HTML-like representation of the document's structure. By default,
this conversion is done using heuristics. However, a model can be provided as
a parameter to use a machine-learning-based approach.

positional arguments:
  pdf_file              PDF file name for which tree structure needs to be
                        extracted

optional arguments:
  -h, --help            show this help message and exit
  -m MODEL_PATH, --model_path MODEL_PATH
                        Pretrained model, generated by extract_tables tool
  -o OUTPUT, --output OUTPUT
                        Path where tree structure should be saved. If none,
                        HTML is printed to stdout.
  -f FAVOR_FIGURES, --favor_figures FAVOR_FIGURES
                        Whether figures must be favored over other parts such
                        as tables and section headers
  -V, --visualize       Whether to output visualization images for the tree
  -v, --verbose         Output INFO level logging.
  -vv, --veryverbose    Output DEBUG level logging.
```

### extract_tables

```
usage: extract_tables [-h] [--mode MODE] --model-path MODEL_PATH
                      [--train-pdf TRAIN_PDF] --test-pdf TEST_PDF
                      [--gt-train GT_TRAIN] --gt-test GT_TEST --datapath
                      DATAPATH [--iou-thresh IOU_THRESH] [-v] [-vv]

Script to extract tables bounding boxes from PDF files using machine learning.
If `model.pkl` is saved in the model-path, the pickled model will be used for
prediction. Otherwise the model will be retrained. If --mode is test (by
default), the script will create a .bbox file containing the tables for the
pdf documents listed in the file --test-pdf. If --mode is dev, the script will
also extract ground truth labels for the test data and compute statistics.

optional arguments:
  -h, --help            show this help message and exit
  --mode MODE           Usage mode dev or test, default is test
  --model-path MODEL_PATH
                        Path to the model. If the file exists, it will be
                        used. Otherwise, a new model will be trained.
  --train-pdf TRAIN_PDF
                        List of pdf file names used for training. These files
                        must be saved in the --datapath directory. Required if
                        no pretrained model is provided.
  --test-pdf TEST_PDF   List of pdf file names used for testing. These files
                        must be saved in the --datapath directory.
  --gt-train GT_TRAIN   Ground truth train tables. Required if no pretrained
                        model is provided.
  --gt-test GT_TEST     Ground truth test tables.
  --datapath DATAPATH   Path to directory containing the input documents.
  --iou-thresh IOU_THRESH
                        Intersection over union threshold to remove duplicate
                        tables
  -v                    Output INFO level logging
  -vv                   Output DEBUG level logging
```

<details><summary>PDF List Format</summary><br>

The list of PDFs are simply a single filename on each line. For example:

```
1-s2.0-S000925411100369X-main.pdf
1-s2.0-S0009254115301030-main.pdf
1-s2.0-S0012821X12005717-main.pdf
1-s2.0-S0012821X15007487-main.pdf
1-s2.0-S0016699515000601-main.pdf
```

</details>

<details><summary>Ground Truth File Format</summary><br>

The ground truth is formatted to mirror the PDF List. That is, the first line
of the ground truth file provides the labels for the first document in
corresponding PDF list. Labels take the form of semicolon-separated tuples
containing the values `(page_num, page_width, page_height, top, left, bottom,
right)`. For example:

```
(10, 696, 951, 634, 366, 832, 653);(14, 696, 951, 720, 62, 819, 654);(4, 696, 951, 152, 66, 813, 654);(7, 696, 951, 415, 57, 833, 647);(8, 696, 951, 163, 370, 563, 652)
(11, 713, 951, 97, 47, 204, 676);(11, 713, 951, 261, 45, 357, 673);(3, 713, 951, 110, 44, 355, 676);(8, 713, 951, 763, 55, 903, 687)
(5, 672, 951, 88, 57, 203, 578);(5, 672, 951, 593, 60, 696, 579)
(5, 718, 951, 131, 382, 403, 677)
(13, 713, 951, 119, 56, 175, 364);(13, 713, 951, 844, 57, 902, 363);(14, 713, 951, 109, 365, 164, 671);(8, 713, 951, 663, 46, 890, 672)
```

One method to label these tables is to use
[DocumentAnnotation](https://github.com/payalbajaj/DocumentAnnotation), which
allows you to select table regions in your web browser and produces the
bounding box file.

</details>

#### Example Dataset: Paleontological Papers

A full set of documents and ground truth labels can be
[downloaded here](http://i.stanford.edu/hazy/share/fonduer/pdftotree_paleo.tar.gz).
You can train a machine-learning model to extract table regions by downloading
this dataset and extracting it into a directory named `data` and then running the
command below. Double check that the paths in the command match wherever you
have downloaded the data.

```
extract_tables --train-pdf data/paleo/ml/train.pdf.list.paleo.not.scanned --gt-train data/paleo/ml/gt.train --test-pdf data/paleo/ml/test.pdf.list.paleo.not.scanned --gt-test data/paleo/ml/gt.test --datapath data/paleo/documents/ --model-path data/model.pkl
```
The resulting model of this example command would be saved as `data/model.pkl`.

## For Developers

We are following [Semantic Versioning 2.0.0](https://semver.org/) conventions.
The maintainers will create a git tag for each release and increment the
version number found in
[`pdftotree/_version.py`](https://github.com/HazyResearch/pdftotree/blob/master/pdftotree/_version.py)
accordingly.

### Tests
To test changes in the package, you install it locally in your virtualenv by
running

```
python setup.py develop
```

Then you can run our tests

```
python setup.py test
```


