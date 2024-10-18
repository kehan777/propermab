# propermab

`propermab` is a Python package for calculating and predicting molecular features and properties of monoclonal antibodies (mAbs).
https://www.biorxiv.org/content/10.1101/2024.10.10.616558v1.full


## Installation (Linux)

First set up a conda environment by running the following commands on the terminal
```bash
git clone https://github.com/regeneron-mpds/propermab.git
conda env create -f propermab/conda_env.yml
conda activate propermab
```
Now install the `propermab` package with
```bash
pip install -e propermab/
```

### APBS
The APBS tool v3.0.0 is used by `propermab` to calculate electrostatic potentials. Download the tool and unzip it to a directory of your choice.
```bash
wget https://github.com/Electrostatics/apbs/releases/download/v3.0.0/APBS-3.0.0_Linux.zip -O apbs.zip
unzip apbs.zip
```
mv libreadline.so.7 to /propermab/APBS-3.0.0.Linux/lib; 
mv libtinfo.so.5 to /propermab/APBS-3.0.0.Linux/lib

You can find the value of `hmmer_binary_path` by issuing the following command on your terminal
```bash
dirname $(which hmmscan)
```

The value of `atom_radii_file` should point to a file named `amber.siz`. This file is part of https://github.com/delphi001/DelphiPka

Installation NanoShaper obtained from NanoShaper repository (https://gitlab.iit.it/SDecherchi/nanoshaper).

NanoShaper can be compiled on Linux for x86-64 with gcc tested from 8 to 9.5. 
Pre-requisites libraries are: boost, gmp, mpfr.
To install please run the setup.py script and select if a standalone, delphi library, or .so (for API usage) is required.

```bash
python setup.py
```	


### Configuration
Edit the `default_config.json` file to specify the path for each of the entries in the file
```python
{
    "hmmer_binary_path" : "",
    "nanoshaper_binary_path" : "/ABPS_PATH/APBS-3.0.0.Linux/bin/NanoShaper",
    "apbs_binary_path" : "/ABPS_PATH/APBS-3.0.0.Linux/bin/apbs",
    "pdb2pqr_path" : "pdb2pqr",
    "immunebuilder_weights_dir" : "",
    "multivalue_binary_path" : "/ABPS_PATH/APBS-3.0.0.Linux/share/apbs/tools/bin/multivalue",
    "atom_radii_file" : "",
    "apbs_ld_library_paths" : ["LIB_PATH", "/ABPS_PATH/APBS-3.0.0.Linux/lib/"]
}
```
eg:
```python
{
    "hmmer_binary_path" : "/home/adsb/amber22_src/build/CMakeFiles/miniconda/install/envs/propermab/bin/",
    "nanoshaper_binary_path" : "/mnt/d/propermab/nanoshaper/build/NanoShaper" ,
    "apbs_binary_path" : "/mnt/d/propermab/APBS-3.0.0.Linux/bin/apbs",
    "pdb2pqr_path" : "/home/adsb/amber22_src/build/CMakeFiles/miniconda/install/envs/propermab/bin/pdb2pqr",
    "multivalue_binary_path" : "/mnt/d/propermab/APBS-3.0.0.Linux/share/apbs/tools/bin/multivalue",
    "immunebuilder_weights_dir" : "/home/adsb/amber22_src/build/CMakeFiles/miniconda/install/envs/propermab/lib/python3.8/site-packages/ImmuneBuilder/trained_model",
    "atom_radii_file" : "/mnt/d/propermab/DelphiPkaSalt/param/amber.siz",
    "apbs_ld_library_paths" : ["LIB_PATH", "/mnt/d/propermab/APBS-3.0.0.Linux/lib/"]
}
```

## Example
### Using `propermab` Python API
You can calculate the molecular features directly from a structure PDB file. Note that this assumes that the residues in PDB file are IMGT numbered and that the heavy chain is named H and the light chain is named L.
```python
from propermab import defaults
from propermab.features import feature_utils

defaults.system_config.update_from_json('./default_config.json')

mol_feature = feature_utils.calculate_features_from_pdb('./pembrolizumab_ib.pdb')
print(mol_feature)
```
Or you can provide a pair of heavy and light chain sequences, `propermab` then calls the `ABodyBuilder2` model to predict the structure, which will be used as the input for feature calculation.
```python
from propermab import defaults
from propermab.features import feature_utils

defaults.system_config.update_from_json('./default_config.json')

heavy_seq = 'QVQLVQSGVEVKKPGASVKVSCKASGYTFTNYYMYWVRQAPGQGLEWMGGINPSNGGTNFNEKFKNRVTLTTDSSTTTAYMELKSLQFDDTAVYYCARRDYRFDMGFDYWGQGTTVTVSS'
light_seq = 'EIVLTQSPATLSLSPGERATLSCRASKGVSTSGYSYLHWYQQKPGQAPRLLIYLASYLESGVPARFSGSGSGTDFTLTISSLEPEDFAVYYCQHSRDLPLTFGGGTKVEIK'
mol_features = feature_utils.get_all_mol_features(heavy_seq, light_seq, num_runs=1) #num_runs=5
print(mol_features)
```
Be sure to replace HEAVY_SEQ and LIGHT_SEQ with the actual sequences. Different runs of `ABodyBuilder2` can result in some difference in sidechain conformations due to the relaxation step in `ABodyBuilder2`. This in turn can affect values of some of the molecular features `propermab` calculates. If the average feature value across multiple runs is desired, one can increase `num_runs`. `get_all_mol_features()` returns a Python dictionary in which the keys are feature names and the values are the corresponding lists of feature values from multiple runs.

## Third-party software
`propermab` requires separate installation of third party software which may carry their own license requirements, and should be reviewed by the user prior to installation and use
