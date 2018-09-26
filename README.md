# Code and Literature Repository for Investigating Meta-Learning Algorithms in the Context of Recommender Systems

## Inspiration

The main inspiration for this research is based on the work performed by the ADAPT group at the Trinity college in Dublin. Of special note for this project is the research outlined in the paper [One-at-a-time: A Meta-Learning Recommender-System for Recommendation-Algorithm Selection on Micro Level](https://arxiv.org/abs/1805.12118).

## Abstract

The DonorsChoose.org dataset of past donations provides a feature-rich corpus of user and item matches which is yet unexplored in the scientific literature of recommender systems. The matching of donors to project in which they might be interested in is a classical recommendation task. Due to the availability of item-, user- and transaction-features, the corpus allows for different data exploration techniques to be applied and allows different meta-learning approaches to be tested. This study aims at providing baselines for recommender systems using testing methods from cross-validation and leave-one-out. Several filtering techniques are explored ranging from collaborative, content-based and hybrid approaches. The algorithm's performance is measured via the recall of recommended projects in a Top-N test.

## Code Snippets

### Non-interactive plotting

```python
import matplotlib as mpl

mpl.use('cairo')

import matplotlib.pyplot as plt
```

### Visualizations

* Plot the donated amount in bins on a logarithmic scale

```python
items_orig = donations[['ProjectID', 'DonorID', 'DonationAmount']]

plt.figure()
plt.hist(items_orig['DonationAmount'], bins=np.logspace(np.log10(items_orig['DonationAmount'].min()), np.log10(items_orig['DonationAmount'].max()), num=28 + 1), histtype='step')
plt.gca().set_xscale('log')
plt.xlabel('Donated Amount')
plt.ylabel('#Occurrence')
plt.tight_layout()
plt.savefig('DonationAmount - Distribution of the donated amount on a logarithmic scale.pdf')
plt.close()
```

* Plot the donated amount in bins on a logarithmic scale for clean subset

```python
items_orig = donations.groupby(['DonorID', 'ProjectID'])['DonationAmount'].sum().reset_index()
# Perform preliminary data cleaning
items_orig = items_orig.drop(items_orig.query('0. <= DonationAmount <= 2.').index)
value_counts = items_orig['DonorID'].value_counts()
items_orig = items_orig[items_orig['DonorID'].isin(value_counts.index[value_counts >= 2])]

plt.figure()
plt.hist(items_orig['DonationAmount'], bins=np.logspace(np.log10(items_orig['DonationAmount'].min()), np.log10(items_orig['DonationAmount'].max()), num=13 + 1), density=True, histtype='step')
plt.gca().set_xscale('log')
plt.xlabel('Donated Amount')
plt.ylabel('Frequency')
plt.tight_layout()
plt.savefig('DonationAmount - Distribution of the donated amount on a logarithmic scale (for donors with at least 2 donations, excluding duplicates and low donations).pdf')
plt.close()
```

* Plot the distribution of ratings

```python
plt.figure()
plt.hist(items['DonationAmount'], bins=5, density=True, histtype='step')
plt.xlabel('Rating')
plt.ylabel('Frequency')
plt.tight_layout()
plt.savefig('DonationAmount - Distribution of ratings for logarithmic bins and excluded outliers.pdf')
plt.close()
```

* RMSE for collaborative filtering techniques

```python
plt.figure()
plt.grid(b=False, axis='x')

algorithms_name = ['zero', 'mean', 'random', 'SKLearn-KNN', 'SKLearn-NMF', 'SKLearn-SVD', 'SciPy-SVD']
average_rmse = [np.sqrt(np.square(np.zeros(items.shape[0]) - items['DonationAmount']).mean()),
  np.sqrt(np.square(np.full(items.shape[0], items['DonationAmount'].mean()) - items['DonationAmount']).mean()),
  np.sqrt(np.square(np.random.uniform(low=min(items['DonationAmount']), high=max(items['DonationAmount']), size=items.shape[0]) - items['DonationAmount']).mean()),
  np.sqrt(items['SquareErrorSKLearn-KNN'].mean()),
  np.sqrt(items['SquareErrorSKLearn-NMF'].mean()),
  np.sqrt(items['SquareErrorSKLearn-SVD'].mean()),
  np.sqrt(items['SquareErrorSciPy-SVD'].mean())]

plt.errorbar(np.arange(len(average_rmse)), average_rmse, xerr=0.45, markersize=0., ls='none')

plt.xticks(np.arange(len(algorithms_name)), algorithms_name)

plt.xlabel('Algorithm')
plt.ylabel('Test RMSE')

plt.gcf().autofmt_xdate()
plt.tight_layout()

plt.savefig('Collaborative Filters - RMSE for DIY algorithms and some baselines.pdf')
plt.close()
```

* Recall@N for collaborative and content-based filters

```python
plt.figure()
plt.grid(b=False, axis='x')

algorithms_name = ['SKLearn-KNN', 'SKLearn-NMF', 'SKLearn-SVD', 'SciPy-SVD', 'Tfidf']
algorithms_pretty_name = ['SKLearn-KNN', 'SKLearn-NMF', 'SKLearn-SVD', 'SciPy-SVD', 'SKLearn-TF-IDF']
average_recall = [items['RecallAtPosition' + alg_name].mean() for alg_name in algorithms_name]

plt.errorbar(np.arange(len(average_recall)), average_recall, xerr=0.45, markersize=0., ls='none')

plt.xticks(np.arange(len(algorithms_pretty_name)), algorithms_pretty_name)
plt.ylim(ymin=-1)

plt.xlabel('Algorithm')
plt.ylabel('Average position in Top-N test set')

plt.gcf().autofmt_xdate()
plt.tight_layout()

plt.savefig('Collaborative and Content-based Filters - Average position in Top-N test set for various algorithms.pdf')
plt.close()
```

```python
plt.figure()
plt.grid(b=False, axis='x')

algorithms_name = ['SKLearn-KNN', 'SKLearn-SVD', 'Tfidf']
algorithms_pretty_name = ['SKLearn-KNN', 'SKLearn-SVD', 'SKLearn-TF-IDF']

plt.hist([items['RecallAtPosition' + alg_name] for alg_name in algorithms_name], bins=10, density=True, label=algorithms_pretty_name, histtype='step')

plt.legend(loc=9)
plt.xlabel('Position in Top-N test set')
plt.ylabel('Frequency')

plt.tight_layout()

plt.savefig('Collaborative and Content-based Filters - Distribution of position in Top-N test set for various algorithms.pdf')
plt.close()
```

* Learning subsystem Recall@N performance

```python
plt.figure()
plt.grid(b=False, axis='x')

algorithms_name = ['SKLearn-KNN', 'SKLearn-SVD', 'Tfidf', 'FastText']
recall_pos = [items['RecallAtPosition' + alg_name].values for alg_name in algorithms_name] + [items[['RecallAtPosition' + alg_name for alg_name in algorithms_name]].min(axis=1).values]
algorithms_pretty_name = ['KNN', 'SVD', 'TF-IDF', 'FastText', 'Combined']

plt.boxplot(recall_pos, positions=np.arange(len(algorithms_pretty_name)), meanline=True, showmeans=True, showfliers=False)

plt.xticks(np.arange(len(algorithms_pretty_name)), algorithms_pretty_name)
plt.ylim(ymin=-1)

plt.xlabel('Algorithm')
plt.ylabel('Position in Top-N test set')

plt.gcf().autofmt_xdate()
plt.tight_layout()

plt.savefig('Learning subsystem - Position in Top-N test set for various algorithms.pdf')
plt.close()
```

```python
plt.figure()
plt.grid(b=False, axis='x')

algorithms_name = ['SKLearn-KNN', 'SKLearn-SVD', 'Tfidf', 'FastText']
algorithms_pretty_name = {'SKLearn-KNN': 'KNN', 'SKLearn-SVD': 'SVD', 'Tfidf': 'TF-IDF', 'FastText': 'FastText'}
algorithms_value_counts = items[['RecallAtPosition' + alg_name for alg_name in algorithms_name]].idxmin(axis=1).value_counts().rename(dict(zip(['RecallAtPosition' + alg_name for alg_name in algorithms_name], algorithms_name))).to_dict()

plt.hist([items['RecallAtPosition' + alg_name] for alg_name in algorithms_name], bins=10, density=True, label=['{:<s} ({:<2.2%} overall best)'.format(algorithms_pretty_name[alg_name], algorithms_value_counts[alg_name] / items.shape[0]) for alg_name in algorithms_name], histtype='step')

plt.legend(loc=9)
plt.xlabel('Position in Top-N test set')
plt.ylabel('Frequency')

plt.tight_layout()

plt.savefig('Learning subsystem - Distribution of position in Top-N test set for various algorithms.pdf')
plt.close()
```

* Meta-learner performance for classification and error prediction

```python
meta_subset = meta_items.loc[test_idx]

plt.figure()
plt.grid(b=False, axis='x')

meta_algorithms_name = [('Bagging', 'Bagging'), ('DecisionTree', 'DecisionTree'), ('BalancedDecisionTree', 'BalancedDTree'), ('GradientBoosting', 'GradientBoosting'), ('NeuralNetwork', 'NeuralNetwork')]
algorithm_selection_columns = [('MetaSubalgorithmPrediction', 'CL'), ('MetaPrediction', 'EP')]
meta_algorithms_column = np.array([[pre[0] + meta_alg_name[0] for pre in algorithm_selection_columns] for meta_alg_name in meta_algorithms_name]).flatten()
meta_algorithms_pretty_name = np.array([[pre[1] + ' ' + meta_alg_name[1] for pre in algorithm_selection_columns] for meta_alg_name in meta_algorithms_name]).flatten()
average_recall = [meta_subset[c].mean() for c in meta_algorithms_column]

plt.errorbar(np.arange(len(average_recall)), average_recall, color=np.array([[c for _ in range(len(algorithm_selection_columns))] for c in plt.rcParams['axes.prop_cycle'].by_key()['color'][:len(meta_algorithms_name)]]).flatten(), xerr=0.45, markersize=0., ls='none')
plt.axhline(y=meta_subset.lookup(meta_subset.index, meta_subset['SubalgorithmCategory']).mean(), color='orange', linestyle='--')

plt.xticks(np.arange(len(meta_algorithms_pretty_name)), meta_algorithms_pretty_name)
plt.ylim(ymin=-1)

plt.xlabel('Algorithm')
plt.ylabel('Average position in Top-N test set')

plt.gcf().autofmt_xdate()
plt.tight_layout()

plt.savefig('Meta-learner as Classifier and Error Predictor - Average position in Top-N test set for various meta-learner algorithms.pdf')
plt.close()
```

* Learning subsystem Recall@N performance with augmented filtering techniques

```python
# Make LaTeX look like default matplotlib
plt.rc('text', usetex=True)
mpl.rcParams['mathtext.fontset'] = 'custom'
mpl.rcParams['mathtext.rm'] = 'Bitstream Vera Sans'
mpl.rcParams['mathtext.it'] = 'Bitstream Vera Sans:italic'
mpl.rcParams['mathtext.bf'] = 'Bitstream Vera Sans:bold'

plt.figure()
plt.grid(b=False, axis='x')

algorithms_name = ['SKLearn-KNN', 'SKLearn-SVD', 'GroupByDonorStateCityZip-SKLearn-SVD', 'GroupByDonorStateCity-SKLearn-SVD', 'Tfidf', 'FastText']
recall_pos = [items['RecallAtPosition' + alg_name].values for alg_name in algorithms_name] + [items[['RecallAtPosition' + alg_name for alg_name in algorithms_name]].min(axis=1).values]
algorithms_value_counts = items[['RecallAtPosition' + alg_name for alg_name in algorithms_name]].idxmin(axis=1).value_counts().rename(dict(zip(['RecallAtPosition' + alg_name for alg_name in algorithms_name], algorithms_name))).to_dict()

algorithms_name = algorithms_name + ['Combined']
algorithms_value_counts['Combined'] = items.shape[0]
algorithms_pretty_name = {'SKLearn-KNN': 'KNN', 'SKLearn-SVD': 'SVD', 'GroupByDonorStateCityZip-SKLearn-SVD': 'SVD (State, City, Zip)', 'GroupByDonorStateCity-SKLearn-SVD': 'SVD (State, City)', 'Tfidf': 'TF-IDF', 'FastText': 'FastText', 'Combined': 'Combined'}

plt.boxplot(recall_pos, positions=np.arange(len(algorithms_pretty_name)), meanline=True, showmeans=True, showfliers=False)

# This got a little bit out of hand...
# Actually just the percentage of each algorithm's contribution in the combined best is printed in a smaller font below the algorithm's name
plt.xticks(np.arange(len(algorithms_pretty_name)), [r'{{\fontsize{{1em}}{{3em}}\selectfont{{}}{0:<s}}}{1}{{\fontsize{{0.8em}}{{3em}}\selectfont{{}}{2:<2.2f}\%}}'.format(algorithms_pretty_name[alg_name], '\n', 100 * algorithms_value_counts[alg_name]  / items.shape[0]) for alg_name in algorithms_name])
plt.ylim(ymin=-1)

plt.xlabel('Algorithm')
plt.ylabel('Position in Top-N test set')

plt.gcf().autofmt_xdate()
plt.tight_layout()

plt.savefig('Learning subsystem - Position in Top-N test set for various algorithms with augmented filtering techniques.pdf')
plt.close()

mpl.rcParams.update(mpl.rcParamsDefault)
```

* Meta-learner performance for classification and error prediction with augmented learning subsystem filtering techniques

```python
meta_subset = meta_items.loc[test_idx]

plt.figure()
plt.grid(b=False, axis='x')

colors = plt.rcParams['axes.prop_cycle'].by_key()['color']

meta_algorithms_name = [('MetaSubalgorithmPredictionBaggingCl', 'CL Bagging', colors[0]), ('MetaPredictionBaggingRg', 'EP Bagging', colors[3]), ('MetaSubalgorithmPredictionDecisionTreeCl', 'CL Decision Tree', colors[0]), ('MetaPredictionDecisionTreeRg40', 'EP Decision Tree', colors[3]), ('MetaSubalgorithmPredictionUserClusterKMeans', 'User-Clustering', colors[0]), ('MetaPredictionGradientBoostingRg', 'EP Gradient Boosting', colors[3]), ('MetaSubalgorithmPredictionStackingDecisionTree', 'Stacking DTree', colors[4])]
average_recall = [meta_subset[c].mean() for c in list(zip(*meta_algorithms_name))[0]]

plt.errorbar(np.arange(len(average_recall)), average_recall, color=list(zip(*meta_algorithms_name))[2], xerr=0.45, markersize=0., ls='none')
plt.axhline(y=meta_subset[meta_subset['SubalgorithmCategory'].mode()[0]].mean(), color='orange', linestyle='--')
plt.axhline(y=meta_subset.lookup(meta_subset.index, meta_subset['SubalgorithmCategory']).mean(), color='orange', linestyle='-')

plt.xticks(np.arange(len(meta_algorithms_name)), list(zip(*meta_algorithms_name))[1])
plt.ylim(ymin=-1)

plt.xlabel('Algorithm')
plt.ylabel('Average position in Top-N test set')

plt.gcf().autofmt_xdate()
plt.tight_layout()

plt.savefig('Meta-learner as Classifier and Error Predictor - Average position in Top-N test set for various meta-learner algorithms with augmented learning subsystem filtering techniques.pdf')
plt.close()
```

## Past Roadmap

* Find a suitable dataset for meta-learning
  * Candidates should provide information about the user, the item and about the context of each transaction
  * Adequate sources might be [kaggle](https://www.kaggle.com), [Google public datasets](https://cloud.google.com/public-datasets/) and previous [RecSys challenges](https://recsys.acm.org/)
* Evaluate existing software frameworks for their applicability as meta-feature generators
  * Meta-feature algorithms should include collaborative, content based and possibly deep learning based approaches
  * Suitable frameworks might be Tensorflow, scikit-learn and higher level libraries like Keras and scikit-surprise
* Train and compare various meta-learning models
  * Predict either rating error or reformulate algorithm selection as classification problem
  * Evaluate model using appropriate variables, possible candidates might be the normalized discounted cumulative gain or the Kendall rank correlation coefficient

## Outlook

* Decaying rating based on the date of the donation
* Use average algorithm with lowest overall error for each cluster in the user-clustering approach
* Algorithm Selection as ranking task using Meta-Learning
