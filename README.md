# Helper Scripts

## Setting up your environment
The following instructions recount how I've gone about setting up my
environment on the rice machines.

### Installing Python requirements
I created a Python3 virtual environment, though you could probably skip this
and move right to instaling requirements.
```
virtualenv -p python3 venv
```

Activate the virtual environment by running:
```
source venv/bin/activate
```

Install requirements by running:
```
pip install -r requirements.txt
```

You can deactivate the virtual environment with:
```
deactivate
```

### Installing AutoDock Vina

Navigate to http://vina.scripps.edu/download.html. Copy the link to the Linux
download. Run the following commands.
```
curl -O [link]
tar xzvf autodock_vina_1_1_2_linux_x86.tgz
```

I recommend adding this line to your `.bashrc` file, since the docking script
relies on this variable being set.
```
export VINA_ROOT="[wherever you installed AutoDock Vina]"
```
Example:
```
export VINA_ROOT="$HOME/autodock_vina_1_1_2_linux_x86"
```

### Installing MGL Tools

Navigate to http://mgltools.scripps.edu/downloads. Copy the link to the proper
tarball installer. Run the following commands.
```
curl -O [link]
tar -xzf [filename]
cd [folder name]
./install.sh
```

I recommend adding this line to your `.bashrc` file, since some of these
scripts rely on this variable being set.
```
export MGL_ROOT="[wherever you installed MGL Tools]"
```
Example:
```
export MGL_ROOT="$HOME/mgltools_x86_64Linux2_1.5.6"
```

## Converting ligand SMILES strings into PDB files

Use the `ligand_smiles_to_pdb.py` script for this. It reads from `smiles.csv`
and outputs to the `ligand_pdb` directory.

If you haven't already, activate your virtual environment:
```
source venv/bin/activate
```

Then run the script:
```
python ligand_smiles_to_pdb.py
```

## Getting relative side chain distances from amino acids

Use the `side_chain_dists.py` script for this. It reads from the `aminoacid_pdbs`
directory and outputs to the `aminoacid_dists` directory.

If you haven't already, activate your virtual environment:
```
source venv/bin/activate
```

Then run the script:
```
python side_chain_dists.py
```

## Getting the PDB files of translations from hNaV1.7

Use the `create_protein_translation_pdbs.py` script for this. It reads from the 
`aminoacid_dists` directory and the `base_protein.pdb` and `protein_translations.csv` 
files and outputs to the `protein_pdb` directory.

If you haven't already, get the relative side chain distances:
```
python side_chain_dists.py
```

Then run the script:
```
python create_protein_translation_pdbs.py
```

## Converting ligand PDB files into PDBQT files
Make sure you have your `MGL_ROOT` variable set. Then run:
```
./ligand_pdb_to_pdbqt.sh
```
This takes files from the `ligand_pdb` directory and puts the converted files
in the `ligand_pdbqt` directory.

## Converting protein PDB files into PDBQT files
Make sure you have your `MGL_ROOT` variable set. Then run:
```
./protein_pdb_to_pdbqt.sh
```
This takes files from the `protein_pdb` directory and puts the converted files
in the `protein_pdbqt` directory.

## Docking the ligands to the proteins
Make sure you have your `VINA_ROOT` variable set. Then run:
```
python3 dock.py
```
This takes files from the `protein_pdbqt` and `ligand_pdbqt` directories and
outputs files in the `docked` directory. It uses the configuration file
`conf.txt`. Logs are placed in the `vina_logs` directory.

By default, this command will not dock pairs that have already been docked. To
do so, add the `-o` command line option:
```
python3 dock.py -o
```

## Preparing HDF files for running through pafnucy
Make sure you have your `PAFNUCY_ROOT` environment variable set. Then run
```
./pafnucy_prepare.sh [options]
```
This reads from the `blocked` directory and the `affinities.csv` file and
outputs HDF files in the `datasets` directory for the training, validation, and
test datasets. By default, these files are `data_train.hdf`, `data_val.hdf`,
and `data_test.hdf` respectively, but these can be configured with the options
below.

Options (all are optional)
* `--shuffle`, `-s`: shuffle the dataset before partitioning. The default is to
  not shuffle if this option is not specified.
* `--train`, `-tr`: a number between 0 and 1 representing the fraction of the
  dataset that should be used for training. This is 1 by default.
* `--val`, `-v`: a number between 0 and 1 representing the fraction of the
  dataset that should be used for validation. This is 0 by default.
* `--prefix`, `-p`: the prefix to use for the data files. This is "data" by
  default.

The portion of the dataset that is not partitioned to the training or
validation set will be partitioned to the test set.

Example:
```
./pafnucy_prepare.sh --shuffle --train 0.8 --val 0.1 --prefix data2
```

## Run Pafnucy through datapoints
Add these two lines to your `.bashrc` file, since the
script relies on these variables being set.
```
export PAFNUCY_ROOT="[wherever you installed Pafnucy]"
export BLOCKED_ROOT="[pwd of smiles-convert/blocked]"
```
After having set these variables, run the following:
```
./run_pafnucy.sh
```
This script takes a datapoint, where each datapoint is represented by 
a directory in `smiles-convert/blocked/*` and it outputs two files:
`smiles-convert/blocked/*/complexes.hdf` and `smiles-convert/blocked/*/predictions.csv`
in each datapoint directory. 
