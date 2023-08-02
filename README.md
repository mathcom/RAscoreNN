# RAscore (XGB-excluding Version)
 * Some codes related to RAScoreXGB are removed to address the Scikit-learn version issue.
 * In this version, RAScoreNN that has no modification is available.

## Installation
```bash
pip install git+https://github.com/mathcom/RAscoreXGB
```

## Original Version
```
https://github.com/reymond-group/RAscore.git
```

----

# Retrosynthetic Accessibility (RA) score
 * RAscore is a score learned from the predictions of a computer aided synthesis planning tool (AiZynthfinder: https://github.com/MolecularAI/aizynthfinder). 
 * **RAscore is intended to be a binary score, indicating whether the underlying computer aided synthesis planning tool can find a route (1) or not (0) to a given compound.** 
 * The tool has been trained on 200,000 compounds from ChEMBL and so is limited to compounds within similar regions of chemical space. It is intended to predict the retrosynthetic accessibility of bioactive molecules. (Data can be found in `data.zip`)
 * Attempts to use the score on more exotic compounds such as those found in the GDB databases will not work: 
    * In this case the model will need to be switched to `GDBscore`, the corresponding models can be found in the models folder or downloaded from the pre-print server.

![alt text](RAscore/images/TOC.png)

## Known Issues
* The models may not generalise well in regions of chemical space that deviate from the training domains (ChEMBL and GDB subsets in this case), this is a problem inherent to ML based classifiers.
* The models have not been tested for the use case of molecular generation.
* **We highly recommend retraining the models on a sample of compounds representing your particular use case, the included data may be used to augment the data you generate**

## Installation 

Follow the steps in the defined order to avoid conflicts.

1. Create an environment (**note: the python version must be == 3.7, 3.8**):
```bash
conda create --name myenv python=3.7
conda activate myenv
```

or use an existing environment with python >= 3.7

2. Install rdkit 2020.03 (if already installed skip this step)
```bash
conda install -c rdkit rdkit -y
```

3. Install RAscore 
```bash
pip install git+https://github.com/mathcom/RAscore.git@master
```
The models.zip file will have to be downloaded and after unzipping the path to the relevant model will need to be specified on model instantiation.

or 

Clone and install the repository using (models should be included):
```
git clone https://github.com/mathcom/RAscore.git
pip install --editable .
```
If the models are not automatically included check that the models.zip file exists and unzip it into the desired location.

### Known Installation Issues 
**The following versions must be used in order to use the pretrained models:**
* python == 3.7
* scikit-learn == 0.22.1
* xgboost == 1.0.2
* tensorflow-cpu == 2.5.0

These requirements arise becuase of the pickling method used to save the model and compatibility issues arising between different versions.

## Usage
### Importing in Python
Depending on if you would like to use the XGB based or Tensorflow based models you can import different modules. 

To walk through the example in a jupyter notebook refer to `rascore_usage.ipynb`

```python
from RAscore import RAscore_NN #For tensorflow and keras based models
from RAscore import RAscore_XGB #For XGB based models

nn_scorer = RAscore_NN.RAScorerNN() 
xgb_scorer = RAscore_XGB.RAScorerXGB()

#Imatinib mesylate
imatinib_mesylate = 'CC1=C(C=C(C=C1)NC(=O)C2=CC=C(C=C2)CN3CCN(CC3)C)NC4=NC=CC(=N4)C5=CN=CC=C5.CS(=O)(=O)O'
nn_scorer.predict(imatinib_mesylate)
0.99522984

xgb_scorer.predict(imatinib_mesylate)
0.99259007

#Omeprazole
omeprazole = 'CC1=CN=C(C(=C1OC)C)CS(=O)C2=NC3=C(N2)C=C(C=C3)OC'
nn_scorer.predict(omeprazole)
0.99999106

xgb_scorer.predict(omeprazole)
0.9556329

#Morphine - Illustrates problem synthesis planning tools face with complex ring systems
morphine = 'CN1CC[C@]23c4c5ccc(O)c4O[C@H]2[C@@H](O)C=C[C@H]3[C@H]1C5'
nn_scorer.predict(morphine)
8.316945e-07

xgb_scorer.predict(morphine)
0.0028359715
```

### Command Line Interface
A command line interface is provided which allows batch processing and enables the flexibility of specifying models.\
```bash
Usage: RAscore [OPTIONS]

Example: A set of smiles `test.smi` are provided

`RAscore -f test.smi -c SMILES -o test.csv`

Default Model: XGBoost using ChEMBL and ECFP4 counts with features

Options:
  -f, --file_path TEXT      Absolute path to input file containing one SMILES
                            on each line. The column should be labelled
                            "SMILES" or if another header is used, specify it
                            as an option

  -c, --column_header TEXT  The name given to the singular column in the file
                            which contains the SMILES. The column must be
                            named.

  -o, --output_path TEXT    Output file path
  -m, --model_path TEXT     Absolute path to the model to use, if .h5 file
                            neural network in tensorflow/keras, if .pkl then
                            XGBoost

  --help                    Show this message and exit.
```
Further RAscore models are contained in the `RAscore/models/models.zip` folder if you wish to specify a different model than the default:
* RAscore
    * DNN_chembl_fcfp_counts
    * XGB_chembl_ecfp_counts
* GDBscore
    * DNN_gdbchembl_fcfp_counts
    * XGB_gdbchembl_ecfp_counts

## Retraining  
If you want to retrain models, or train your own models using the hyperparameter optimisation framework found in the 'model_building' folder, then the following should be installed in the environemnt aswell:\
`pip install -e .[retraining]`

The SYBA, SCscore and SAscore should also be downloaded for descriptor calculations and training scripts modified to reflect the locations of the models:
* https://github.com/lich-uct/syba
* https://github.com/connorcoley/scscore
* https://github.com/rdkit/rdkit/tree/master/Contrib/SA_Score

Please refer to the `model_building` folder for further information about retraining.

## Performance on Test Set
* Test set contains ca. 20,000 compounds from ChEMBL
* The model was able to separate clusters of solved/unsolved compounds as found by computing the average linkage
* RAscore can better differentiate between solved/unsolved compounds than existing methods.
![alt text](RAscore/images/RA_Score_histogram.png)

## Computation of Average Linkage 
![alt text](RAscore/images/average_linkage.png)

## Citation
The models have been published in Chemical Science
```
@Article{D0SC05401A,
	author ="Thakkar, Amol and Chadimová, Veronika and Bjerrum, Esben Jannik and Engkvist, Ola and Reymond, Jean-Louis",
	title  ="Retrosynthetic accessibility score (RAscore) – rapid machine learned synthesizability classification from AI driven retrosynthetic planning",
	journal  ="Chem. Sci.",
	year  ="2021",
	volume  ="12",
	issue  ="9",
	pages  ="3339-3349",
	publisher  ="The Royal Society of Chemistry",
	doi  ="10.1039/D0SC05401A",
	url  ="http://dx.doi.org/10.1039/D0SC05401A",
}
```

