# snackVision
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
import seaborn as sns
from sklearn import metrics

train = pd.read_csv('C:/Users/Paul Morris/Desktop/DeepLearn/train.csv')
test = pd.read_csv('C:/Users/Paul Morris/Desktop/DeepLearn/test.csv')
print(train.shape)
print(train.columns)
print(train.head())
X_train = train.drop(['image_id', 'folder'], axis=1)
print(X_train.columns)
null_entries = X_train.isnull().sum()
print(null_entries)
cols_w_nulls = null_entries[null_entries > 0].index

X = X_train.drop(['class_name'], axis=1)
y = X_train.pop('class_name')
print(X.columns)
print(y.head())
# Now that we have and X with all numeric no Na and a y all categorical we are ready to look at the data
plt.hist(y, bins=20, edgecolor='black', orientation="horizontal")
plt.show()
# Now lets see what columns are related to each other
from sklearn.feature_selection import SelectKBest
from sklearn.feature_selection import chi2

# extracting top 10 best features by applying SelectKBest class
bestfeatures = SelectKBest(score_func=chi2, k=4)
fit = bestfeatures.fit(X, y)
dfscores = pd.DataFrame(fit.scores_)
dfcolumns = pd.DataFrame(X.columns)
featureScores = pd.concat([dfcolumns, dfscores], axis=1)
featureScores.columns = ['Specs', 'Score']  # this names the df columns
print(featureScores.nlargest(4, 'Score'))  # prints best features
# Provides a visual
from sklearn.ensemble import ExtraTreesClassifier
import matplotlib.pyplot as plt

model = ExtraTreesClassifier()
model.fit(X, y)
print(model.feature_importances_)
feat_import = pd.Series(model.feature_importances_, index=X.columns)
feat_import.nlargest(4).plot(kind='barh')
plt.show()
# heatmap
import seaborn as sns

print(X.columns)
corrmat = X.corr()
top_corr_feat = corrmat.index
plt.figure(figsize=(20, 20))
g = sns.heatmap(X[top_corr_feat].corr(), annot=True, cmap='RdYlGn')
plt.show()
# tranform response to dummy variables
y_dummy = pd.get_dummies(y, columns=['class_name'], drop_first=True)
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
scaler.fit(X)
# Now lets do a Logistic Regression
from sklearn.linear_model import LogisticRegression
lreg = LogisticRegression()
lreg.fit(X, y)
pred = lreg.predict(X)
score = lreg.score(X, y)
print(score)
cm = metrics.confusion_matrix(y, pred)
plt.figure(figsize=(9, 9))
sns.heatmap(cm, annot=True, fmt=".3f", linewidths=.5, square=True, cmap='Blues_r')
plt.ylabel('Actual label')
plt.xlabel('Predicted label')
all_sample_title = 'Accuracy Score: {0}'.format(score)
plt.title(all_sample_title, size=15)
# Judging from that accuracy lets not use Logistic Regression
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, confusion_matrix, classification_report
from sklearn.model_selection import train_test_split
from sklearn.metrics import log_loss
rf_model = RandomForestClassifier()
rf_model.fit(X, y)
y_pred = rf_model.predict_proba(X)
print(y_dummy.shape)
print(y_pred.shape)
from sklearn.metrics import log_loss
#print(f"bce w/ sklearn: {log_loss(y, yhat, normalize=False):.4f}")
