# rankpruning

**rankpruning** is a python package for state-of-the-art binary classification with **partially mislabeled training examples**. This machine learning package implements the Rank Pruning algorithm and other methods for P̃Ñ learning (binary classification where some fraction of positive example labels are uniformly randomly flipped and some fraction of negative example labels are uniformly randomly flipped). Rank Pruning is theoretically grounded and trivial to use. The Rank Pruning algorithm ([Curtis G. Northcutt](http://www.curtisnorthcutt.com/), [Tailin Wu](http://cuaweb.mit.edu/Pages/Person/Page.aspx?PersonId=26273), & [Isaac L. Chuang](http://feynman.mit.edu/ike/homepage/index.html), 2017) is under review at UAI 2017 as a submitted conference publication. A version of the paper is available on arXiv at this link: (coming soon in the next 5 days!). The `RankPruning()` class:
- works with any probabilistic classifer (e.g. neural network, logistic regression)
- is fast (time-efficient), taking about 2-3 times the training time of the classifier)
- also computes the fraction of noise in the positive and negative sets
- provides state-of-the-art (as of 2017) F1 score, AUC-PR, accuracy, etc. for binary classification with mislabeled training data (P̃Ñ learning).
- also works well when noise examples drawn from a third distribution are mixed into the training data.

We provide both Jupyter Notebook and python implementations of most files for portability and ease of use. A tutorial is also provided in the tutorial/tutorial.ipynb file. 

### Classification with Rank Pruning is easy

```python
rp = RankPruning(clf=logreg()) # or a CNN(), or NaiveBayes(), etc.
rp.fit(X, s)
pred = rp.predict(X)
``` 

It is trained with:
1. a feature matrix **X**
2. a vector **s** of binary (0 or 1) labels where an unknown fraction of labels may be mislabeled (flipped)
3. ANY probabilistic classifier **clf** as long as it has `clf.predict_proba()`, `clf.predict()`, and `clf.fit()` defined. 

Ideally, given training feature matrix **X** and noisy labels **s** (instead of the hidden, true labels **y**), fit **clf** as if you had called `clf.fit(X, y)` not `clf.fit(X, s)`, even though **y** is not available.#

### How does Rank Pruning work?

**rankpruning** is based on a joint research effort between the Massachusetts Institute of Technology's Department of Electrical Engineering and Computer Science, Office of Digital Learning, and Department of Physics. The Rank Pruning algorithm is theoretically grounded and trivial to use. **rankpruning** embodies the "learning with confident examples" paradigm and works as follows:
1. estimate the fraction of mislabeling in both the positive and negative sets
2. use these estimates to rank examples by confidence of being correctly labeled
3. prune out likely mislabeled data
4. train on the pruned set (an intended subset of the correctly labeled training data)   

### Installation

To use the **rankpruning** package, simply:

```
$ python setup.py
```

or if you intend to modify the files

```
$ python setup.py develop
```

then you can

```
import rankpruning
from rankpruning import RankPruning
from rankpruning import other_pnlearning_methods
```

If you wish to use the tutorial_and_testing package, a few additional dependencies are needed. See below.

#### Dependencies

**rankpruning** requires sklearn and numpy. Installation with `python setup.py` installs them for you. 

Since Rank Pruning works for any probabilistic classifer, we provide a CNN (convolutional neural network). Using this classifier requires two additional dependencies. 

To use our CNN with conda:

```
# Linux/Mac OS X, Python 2.7/3.4/3.5, CPU only:
$ conda install -c conda-forge tensorflow
$ conda install keras
```

With pip, first follow the instructions for installing tensorflow [here](https://www.tensorflow.org/versions/r0.10/get_started/os_setup#pip_installation), then install keras using: 

```
sudo pip install keras
```

We also provided a basic tutorial to test out Rank Pruning. The tutorial and testing examples also depend on the following four packages:
- scipy
- pandas
- matplotlib
- jupyter

#### User Installation

In your command line interface (terminal on Mac), go to the directory of your
choosing and clone the repo.

```
$ cd directory_you_wish_to_install_rankpruning
$ git clone git@github.com:cgnorthcutt/rankpruning.git
```

To use the **rankpruning** package in this directory, import the package as:

```python
import rankpruning
```

To import the Rank Pruning algorithm for classification with mislabeled
training data:

```python
from rankpruning import RankPruning
```




### Simple Example: Comparing Rank Pruning with other models for P̃Ñ learning.

```python
from __future__ import print_function
import rankpruning
import pnlearning_methods
import numpy as np

# Libraries uses only for the purpose of this example
from scipy.stats import multivariate_normal
from sklearn.metrics import precision_recall_fscore_support as prfs
from sklearn.metrics import accuracy_score as acc
from sklearn.linear_model import LogisticRegression

# Create the training dataset with positive and negative examples
# drawn from two-dimensional Guassian distributions.
neg = multivariate_normal.rvs(mean=[2,2], cov=[[10,-1.5],[-1.5,5]], size=1000)
pos = multivariate_normal.rvs(mean=[8,8], cov=[[1.5,1.3],[1.3,4]], size=500)
X = np.concatenate((neg, pos))
y = np.concatenate((np.zeros(len(neg)), np.ones(len(pos))))

# For this example, choose the following mislaeling noise rates.
frac_pos2neg = 0.55 # rh1, P(s=0|y=1) in literature
frac_neg2pos = 0.15 # rh0, P(s=1|y=0) in literature

# Generate s, the observed noisy label vector (flipped uniformly randomly with noise rates).
s = y * (np.cumsum(y) <= (1 - frac_pos2neg) * sum(y))
s_only_neg_mislabeled = 1 - (1 - y) * (np.cumsum(1 - y) <= (1 - frac_neg2pos) * sum(1 - y))
s[y==0] = s_only_neg_mislabeled[y==0]

# Create testing dataset:
neg_test = multivariate_normal.rvs(mean=[2,2], cov=[[10,-1.5],[-1.5,5]], size=2000)
pos_test = multivariate_normal.rvs(mean=[8,8], cov=[[1.5,1.3],[1.3,4]], size=1000)
X_test = np.concatenate((neg_test, pos_test))
y_test = np.concatenate((np.zeros(len(neg_test)), np.ones(len(pos_test))))

# We choose logistic regression, but rank pruning can use 
# any probabilistic classifier such as CNN(), or NaiveBayes(), etc.
clf = LogisticRegression()

# Initilize models: 
models = {
  "Baseline" : pnlearning_methods.BaselineNoisyPN(clf = clf),
  "Rank Pruning" : rankpruning.RankPruning(clf = clf),
  "Rank Pruning (noise rates given)": rankpruning.RankPruning(frac_pos2neg, frac_neg2pos, clf = clf),
  "Elk08 (noise rates given)": pnlearning_methods.Elk08(e1 = 1 - frac_pos2neg, clf = clf),
  "Liu16 (noise rates given)": pnlearning_methods.Liu16(frac_pos2neg, frac_neg2pos, clf = clf),
  "Nat13 (noise rates given)": pnlearning_methods.Nat13(frac_pos2neg, frac_neg2pos, clf = clf),
}

# For the models, fit on (X, s) and predict on X_test:
for key in models.keys():
  model = models[key]
  model.fit(X, s)
  pred = model.predict(X_test)
  pred_proba = model.predict_proba(X_test) # Produces P(y=1|x)

  print("\n%s Model Performance:\n==============================\n" % key)
  print(
    "Accuracy:", acc(y_test, pred), "|", 
    "Precision:", prfs(y_test, pred)[0], "|", 
    "Recall:", prfs(y_test, pred)[1], "|",
    "F1:", prfs(y_test, pred)[2]
  )
```
