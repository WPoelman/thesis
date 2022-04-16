# Thesis
This system is a syntax-semantics interface between Universal Dependencies (UD) and Discourse Representation Structures (DRS), as used in the Parallel Meaning Bank (PMB) project.
The primary DRS target format is the Simplified Box Notation (SBN).
The following images show the basic idea for the sentence **Tracy lost her glasses.** :

<table>
  <tr>
    <td><img src="img/../data/img/example_ud.png" alt="ud example" style="height: 300px;"/></td>
    <td><img src="img/../data/img/example_sbn.png" alt="sbn example" style="height: 300px;"/></td>
  </tr>
</table>

## Installation
1. Create and activate a virtual environment (Python 3.9.7 was used in development) (**Optional**)
2. Install `graphviz` to generate visualizations. (**Optional**)
   1. On Debian-based systems: `apt install graphviz libgraphviz-dev pkg-config`
3. Install GREW
   1. Follow the instructions on https://grew.fr/usage/install/
   2. Make sure `opam` is active by running `opam --version`
   3. If `opam` is not avaible, try activating it with `eval $(opam env)`
4. Install dependencies with `pip install -r requirements.txt`
   1. `requirements-ud.txt` contains alternative UD parsing systems (**Optional**)
   2. `requirements-dev.txt` contains development libraries for testing, formatting etc. (**Optional**)

Run `fix_all.sh` to format and test the project

## Data
The data comes from the Parallel Meaning Bank project (https://pmb.let.rug.nl/).
## Notes
- This project uses version 4.0.0 of the PMB dataset, which can be downloaded from here: https://pmb.let.rug.nl/data.php
- There are some minor issues with this version of the dataset that will be fixed in future versions:
  * There are several sense ids that contain whitespace
  * There are some empty `*.sbn` documents
  * There are several cyclic SBN graphs (they should all be Directed Acyclic Graphs (DAGs))
  * Some SBN files contain constants that cannot be distinguished from indices. For example: `en/silver/p15/d3131`
- Most of the docs with these issues are listed in the `misc` folder.
- The system warns or errors when it encounters these issues.
## Structure
The PMB has a specific file structure that is handy to understand when using the system.
The `data/test_cases` directory has a similar layout and can be used to see what is happening.
What the specific files mean will be explained in the usage section.

```
<language-1> /
  <p-collection-1> /
    <document-1> /
      --- The existing PMB files per document ---
      <lang>.drs.clf
      <lang>.drs.sbn
      <lang>.drs.xml
      <lang>.met
      <lang>.parse.tags
      <lang>.raw
      <lang>.status
      <lang>.tok.iob
      <lang>.tok.off

      --- These items get added when using all options provided by main.py ---
      <lang>.ud.<ud-system>.conll
      <lang>.drs.penman
      <lang>.drs.lenient.penman
      viz /
        <lang>.drs.png
        <lang>.ud.<ud-system>.png

      --- Predictions from pmb_inference.py when using all options get stored here ---
      predicted /
        output.penman
        output.lenient.penman
        output.sbn
        output.png
    <document-2> /
    ...
  <p-collection-2> /
  ...
<language-2> /
...
```

## Usage
### From scratch
To transform a single sentence to DRS in SBN format, run:

```
python inference.py --sentence "Tracy lost her glasses." --output_dir ./result 
```

This stores a `conll` file of the UD parse and the generated `sbn` file in `./result`.

---

If you already have a UD parse in `conll` format at hand, run:

```
python inference.py --ud <path-to-conll-file> --output_dir ./result
```
---

There are a number of additional tools and options included apart from the main graph transformations:

```
python inference.py \ 
  --sentence "Tracy lost her glasses." \
  --output_dir ./result \
  --store_visualizations \
  --store_penman
```

This store visualizations of the UD parse, the SBN graph as well as an AMR-like output of the SBN in Penman notation.
One of these is with sense ids fully intact, the other a more 'lenient' version with the senses cut into pieces and without the sense number.
The regular Penman output indirectly targets word sense disambiguation when scoring the output with SMATCH for instance, the lenient option does not do this and rewards correct lemmas and parts of speech.

For more details and additional options, run `inference.py --help`.

### With normal PMB dataset
The PMB does not come with UD parses or SBN graphs in Penman notation.
The script `main.py` can be used to interact with the PMB to gather information, store required files, train certain components used for inference and generate visualizations.

```
python main.py --starting_path <path-to-pmb-dataset> \
  --store_ud_parses \
  --search_dataset \
  --extract_mappings \
  --error_mine \
  --store_visualizations \
  --store_penman \
  --lenient_penman
```

This will recursively go through all PMB docs, do all possible operations on the data and generates all required files to run inference with.

For more details and additional options, run `main.py --help`.

### With enhanced PMB dataset
To store the results within the PMB dataset file structure and evaluate the generated output, you can use `pmb_inference.py`.

```
python pmb_inference.py -p data/test_cases -r results.csv
```

This will recursively go through the provided path, generate SBN with the conll files it finds and compares these with the `.penman` file in the same folder.
It will store all results per sentence and write the evaluation scores to `results.csv` for later analysis.

By default, the system will use 16 threads to go through the dataset and generate results.

---

Again, there are several additional options:

```
python pmb_inference.py -p <path-to-pmb-dataset> \
  --results_file results.csv \
  --max_workers 32 \
  --clear_previous \
  --store_visualizations \
  --store_sbn
```
This will go through the dataset with 32 workers, writing the results to `results.csv`, clearing previously predicted files if they exist, storing visualizations of the generated output as well the generated SBN itself.

For more details and additional options, run `pmb_inference.py --help`.
## Possible future improvements
- [ ] Support enhanced UD annotations (need CoreNLP binding: https://stanfordnlp.github.io/CoreNLP/depparse.html or keep an eye on this: https://github.com/stanfordnlp/stanza/issues/359) these are essential for certain case markings.


## Test cases
* **p00/d0004**: `entity` that combines multiple subtypes
* **p00/d0801**: multiple boxes
* **p00/d1593**: negation
* **p00/d2719**: case marking / pivot -> extended UD needed!
* **p03/d2003**: named entity 'components' combining in single item
* **p04/d0778**: double negation
* **p04/d1646**: connect owner

# TODO:
- [ ] check consistency quoted output + casing of output grew -> sbn and original sbn
- [ ] add rules to deal with numbers
- [ ] add proper edge mapping or triple classifier for edge labels
