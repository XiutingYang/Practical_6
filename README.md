# Practical_6

## Introduction

Today we will look at how to featurise drugs and train learners to predict blood brain barrier permeability. Fortunately, we already have a library that can featurise drugs for us. It's called RDKIT and works with both small molecules and macromolecules. In the first half, we will get acquinted with RDKIT and explore the different features it has, such as drawing molecules and quickly calculating features for us. In the second half, we will featurise hundreds of drugs and input the data into our ML models.


## Lab Task 1

1. First, you will pip install the rdkit library, which can be performed in Colab by:

``` python

!pip install rdkit-pypi

```

2. Once installed, you will need to select a drug to explore RDKIT's features. For this you will need its SMILES representation. Head over to PubChem and search for a drug. For example, "Paracetamol". You'll be presented with 'molecule cards'. Copy and paste the SMILES notation from, as illustrated below and highlighted by the orange arrow:

<img width="858" alt="image" src="https://github.com/Dr-M-ELBA/Practical_6/assets/158515515/c641ca03-b2e4-41c0-9afe-36396ef61c9b">

Then assign the SMILE to a simple variable, as such:

``` python

from rdkit import Chem
m = Chem.MolFromSmiles('CC(=O)NC1=CC=C(C=C1)O')

```

Draw the molecule so make sure it is the right one:

``` python

from rdkit.Chem import Draw
img = Draw.MolToImage(m)
img

```

Is it the right molecule?

3. The beauty of RDKIT is it can quickly perform calculations about your molecule(s). Calculations include count the number of atoms in a molecule, count the molecular weight, etc. You could of course do this manually, but as you will see later, this can be exhausting for large datasets. Don't worry if you are not an expert of drugs. RDKIT is reliable enough.

Let's have a look at some of its features. The code below quickly calculates the number of atoms in your molecule. Execute the code and check if the output is correct:

``` python

m.GetNumAtoms()

```

You would have noticed it only calculated non-hydrogen atoms. If you wanted to calculate hydrogen atoms, then:

``` python

m2 = Chem.AddHs(m)
m2.GetNumAtoms()

```

If you wanted to calculate the entire molecular weight:

``` python

from rdkit.Chem import Descriptors
Descriptors.MolWt(m)

```

Note descriptors are what the cheminformatics community calls features. As the name suggest, descriptors are descriptions of molecules. There are hundreds of these available. We will be working with 2D descriptors but there are also 3D descriptors - descriptors that provide information about a molecule's 3D structure. 


4. Another useful feature from RDKIT is its ability to compare the similarities between molecules.

``` python

from rdkit import DataStructs

mol1 = 'CC(=O)NC1=CC=C(C=C1)O' # Paracetamol
mol2 = 'CN1C=NC2=C1C(=O)N(C(=O)N2C)C' # Caffeine
mol3 = 'CN1C2=C(C(=O)N(C1=O)C)NC=N2' # Theophylline

fp1 = AllChem.GetMorganFingerprint(ms[0], 4)
fp2 = AllChem.GetMorganFingerprint(ms[1], 4)
fp3 = AllChem.GetMorganFingerprint(ms[2], 4)

DataStructs.TanimotoSimilarity(fp1, fp2) # replace the arguments to determine which two molecules are similar

```
According to the code above, which drug is the closest match to caffeine? Can you think of a good application for comparing the similarities between drugs?

Want to visualise the electronegativity of your molecule?

``` python
from rdkit.Chem.Draw import SimilarityMaps
mol = Chem.MolFromSmiles(mol2)
AllChem.ComputeGasteigerCharges(mol)
contribs = [mol.GetAtomWithIdx(i).GetDoubleProp('_GasteigerCharge') for i in range(mol.GetNumAtoms())]
fig = SimilarityMaps.GetSimilarityMapFromWeights(mol, contribs, colorMap='jet', contourLines=10)
```

We are nearing the end of the first task. To get more comfortable with featurising drugs, feel free to repeat the above code a few times with other molecules. PubChem has thousands of them. 

## Lab Task 2

For the second task, we will leverage descriptors to help us build effective ML models to predict drug biological activity. One such activity is whether a drug can cross the blood brain barrier (BBB). The BBB is a barrier that aims to prevent toxic molecules from reaching and consequently affecting our brain. It's a robust barrier and ensures our safety. However, it also prevents drugs from reaching the brain, which is a burden if you want to treat a brain-related disorder. So let's see if ML can be used to help scientists know if their drug can cross the BBB.

Your task will be to convert the SMILES notation from the dataset into 200 molecular descriptors for each drug in the dataset. Once featurised, use the molecular descriptors as inputs to predict whether a drug will cross the BBB.
