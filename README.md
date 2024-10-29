III. PROPOSED METHOD
A. DATA
In this study, we used the Healthcare Provider Fraud Detec-
tion Analysis dataset published in Kaggle [31]. The dataset
consists of four sets: beneficiary, inpatient claims, outpatient
claims, and provider fraud labels. TABLE 2 presents the
number of records in the dataset, and TABLE 3 shows the
features available in the dataset. As shown in TABLE 3, there
is a set of inpatient and outpatient claims. It includes a list of
different healthcare providers and their services to patients by
physicians (attending, operating, and other physicians). Each
claim contains a set of diagnosis codes and a set of healthcare
service codes registered according to the International Clas-
VOLUME 11, 2023
Outpatient Dataset
Beneficiary Id
Claim Id
Claim Start Date
Claim End Date
Provider
Insurance Claim Amount Reimbursed
Attending Physician
Operating Physician
Other Physician
Claim Diagnosis Code 1
Claim Diagnosis Code 2
Claim Diagnosis Code 3
Claim Diagnosis Code 4
Claim Diagnosis Code 5
Claim Diagnosis Code 6
Claim Diagnosis Code 7
Claim Diagnosis Code 8
Claim Diagnosis Code 9
Claim Diagnosis Code 10
Claim Procedure Code 1
Claim Procedure Code 2
Claim Procedure Code 3
Claim Procedure Code 4
Claim Procedure Code 5
Claim Procedure Code 6
Deductible Amount Paid
Claim Admit Diagnosis Code
Label Dataset
Provider
Potential Fraud
sification of Diseases, Ninth Revision, Clinical Modification
(ICD-9-CM) [109] and shown in the Claim Diagnosis Codes
and the Claim Procedure Codes. In the dataset, the providers
have been labeled fraudulent or not, while the claims do not
have such a label.
B. FEATURE ENGINEERING
In our problem, there are three participants: healthcare
providers, physicians (attending, operating, or other), and
beneficiaries (patients). The dataset only has explicit features
(e.g., age) for beneficiaries. Other participants do not include
such features. However, we can find lots of information from
their claims. Due to the fragmented behavior in healthcare
fraud, it is essential to pay special attention to the history
of claims for each participant. Since, in this study, we fo-
cused on the fraud committed by providers, we limited our
feature selection to providers’ features. Also, for simplicity,
we limited our features to claim procedure codes and left
the use of other features for future work. Claim procedure
codes can present providers’ features such as their specialties
implicitly.For using claim procedure codes as features, we
used the HcpcsVec embedding technique [81] in three steps:
5
This work is licensed under a Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 License. For more information, see https://creativecommons.org/licenses/by-nc-nd/4This article has been accepted for publication in IEEE Access. This is the author's version which has not been fully edited and
content may change prior to final publication. Citation information: DOI 10.1109/ACCESS.2024.3425892
Sh. Mardani et al.: Preparation of Papers for IEEE TRANSACTIONS and JOURNALS
FIGURE 1. Embedding of procedure codes as provider features. Each claim can include multiple procedure codes listed as PRC1 to PRC4. The combined
dataset is created by combining Outpatient and inpatient datasets. Then, the number of occurrences of each claim procedure for each healthcare provider
is calculated (Provider-Service Occurrences dataset). The final dataset is obtained by normalizing the Provider-Service Occurrences dataset.
1) Combined Dataset: Combine inpatient and outpatient
datasets.
2) Provider-Service Occurrences: Group claims by each
provider and sum up the number of claims for each
claim procedure.
3) Normalized Provider-Service Occurrences: Use Min-
Max Scaling [110] for normalizing Provider-Service
Occurrences.
In other words, we consider the number of occurrences
of each claim procedure for a healthcare provider as its fea-
tures. FIGURE 1 shows the embedding of procedure codes
(PRCs) as provider features. PRCs are claim procedure code
features whose corresponding values show ICD-9-CM pro-
6
cedure codes. Each claim can include multiple PRCs that
are listed as PRC1 to PRC4. After combining outpatient
and inpatient datasets, we group all claims by each provider
and sum up the number of claims for each claim procedure
(provider-Service Occurrences). Finally, we normalize the
resulting dataset by Min-Max scaling. Since there are 1324
unique procedure codes in the dataset, in addition to a null
value, the number of features equals 1325.
C. GRAPH STRUCTURE
In healthcare, fraud is committed by one participant alone or
by more than one. In these situations, there are two types of
relationships between participants that form fraud scenarios:
VOLUME 11, 2023
This work is licensed under a Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 License. For more information, see https://creativecommons.org/licenses/by-nc-nd/4This article has been accepted for publication in IEEE Access. This is the author's version which has not been fully edited and
content may change prior to final publication. Citation information: DOI 10.1109/ACCESS.2024.3425892
Sh. Mardani et al.: Preparation of Papers for IEEE TRANSACTIONS and JOURNALS
FIGURE 2. A sample of the relationship between different participants and its transformation into a network of providers. Left) A heterogeneous graph of
physicians, providers, and beneficiaries. Right) the desired target graph.
1) When a fraudster commits fraud alone, he/she will re-
peat it in different situations with different participants.
2) When fraudsters work together, they trust each other
and try to cover each other to conceal their fraud.
For example, consider a fraudster who misuses the ID card
of a beneficiary and receives some services from a health-
care provider. He/she may repeat it with different healthcare
providers to prevent being detected. Another example can be
a healthcare provider and a beneficiary colluding with each
other and billing unrealistic claims for an insurance company.
These relationships may exist between every two or three
participants: Provider and beneficiary, beneficiary and physi-
cian, physician and physician, or between beneficiary, physi-
cian, and provider. Therefore, if a participant has committed
fraud, other participants with a relationship with it would
be suspicious, which is a case of interdependency. Since all
claims are billed by a provider to an insurance company,
whether the provider is aware of the fraud of other participants
or not, it is responsible for the fraud. Thus, for capturing
the interdependency, we consider two types of relationships
between providers: (i) providers who have provided services
to the same patient and (ii) providers who have employed
the same physician. We model their relationships by an un-
weighted graph G=(V, R) where V represents providers, and
R represents relations between them. Relations (edges) are
presented by (vi , rk , vj ) where rk is the relation type (edge
type): (i) provided services to the same patient, (ii) employed
the same physician.
We only keep the history up to thirty days. In other words,
we create the edges if the difference of the start date in the
claims that have the same participant (physician/ beneficiary)
is at most 30 days. FIGURE 2 shows a sample of the relation-
ship between different participants and its transformation into
VOLUME 11, 2023
a network of providers.
On the left side of FIGURE 2, a heterogeneous graph is
shown. The graph shows the relationship between parties
(physicians, providers, and beneficiaries). Some parties are
connected to more than one claim. For example, beneficiary 1
(Ben1) is connected to two claims (Clm1 and Clm2), and each
of the claims is connected to different healthcare providers
(Prv1 and Prv2). It means the beneficiary has received health-
care services from the providers (Prv1 and Prv2). Thus, in our
target graph (right side of FIGURE 2), two nodes, Prv1 and
Prv2, are connected because of the same beneficiary (edge
type r1). It also works for the same physicians. Prv2 and Prv3
are connected (edge type r2) in the target graph (right side)
due to two shared physicians (Phy5 and Phy6) in the primitive
graph (left side).
D. GRAPH EMBEDDING
In this section, we explain our supervised model based on
the Graph ATtention Network (GAT) [103] for healthcare
provider fraud detection. In our graph, there are different
edge types. At first, for each type of edge, we design the
transformation matrix Mk to project the features of nodes into
the feature space based on the edge type k. The projection
process can be shown as follows:
h′i = Mk .hi
(1)
in which hi and h′i are the original and projected features of
node i, respectively. To learn the weight of different nodes, we
perform self-attention mechanism [104]. Given a node pair (i,
j) connected via edge type k, the attention eij k calculates the
importance of node j’s features to node i. It can be formulated
as follows:
eij k = att(h′i , h′j , k)
(2)
7
This work is licensed under a Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 License. For more information, see https://creativecommons.org/licenses/by-nc-nd/4This article has been accepted for publication in IEEE Access. This is the author's version which has not been fully edited and
content may change prior to final publication. Citation information: DOI 10.1109/ACCESS.2024.3425892
Sh. Mardani et al.: Preparation of Papers for IEEE TRANSACTIONS and JOURNALS
Here, att is a deep neural network which performs the atten-
tion. We use masked attention [104] in the way that eij k is
only computed for thenodes j ∈ Nik , where Nik is the set of
neighbors of node i in the graph based on the edge type k.
We consider the first-order neighbors of i (including i) in
all our experiments. To make comparable coefficients across
different nodes, we normalize them using the softmax func-
tion:
exp(σ(aTk .[h′i ∥h′j ]))
(3)
αij k = softmax j (eij k ) = P
T
′
′
l∈N k exp(σ(ak .[hi ∥hl ]))
i
in which .T represents transposition, ∥ denotes the concate-
nate operation, σ denotes the activation function, and ak is the
attention vector for edge type k. As we can see from equation
(3), the (i,j)’s features influence on the weight coefficient.
Because of the concatenation order in the numerator and the
difference between their neighbors that makes the normalized
term (denominator), the weight coefficient αij k is asymmetric
and makes different contributions to each node. Then, we
aggregate the neighbor’s projected features of node i by the
corresponding coefficients to calculate the edge type-based
embedding as follows:
X
′
zi k = σ(
αij k .h′j )
(4)
j∈Nik
in which zki is the learned embedding of node i for the edge
type k. Multi-head attention [104] is used to stabilize the
learning process of self-attention. We execute M independent
attention mechanisms (equation (4)) and concatenate their
features as follows:
M
X
′
zi k = ∥ σ(
αij k .h′j )
(5)
m=1
j∈Nik
In our experiment, we applied two GAT layers. The first
layer aggregates the messages received from their immediate
neighbors and sends a graph representation to the next layer.
The second layer updates the current representation of the
graph by aggregating the messages received from their neigh-
bors. As a result, each layer increases the receptive field by
one hop:
X
′′
(l1 )
(l )
zi k = σ(
αij k .h′j 1 )
(6)
j∈Nik
(l1 )
(l )
in which aij k
and h′j 1 denote the weight coefficient and
the projected feature of node j in the first GAT layer, respec-
tively.
After GAT layers, we use a feed-forward layer:
X ′′
zi = σ(
zi k .wi k + bi k )
(7)
i∈V
in which wi k denotes weight and bi k is bias in edge-type k
embedding. Given the K edge types, we will have K groups
of node embeddings, denoted as zk , that are concatenated as
follows:
′
K
zi k = ∥ zi k
k=1
8
(8)
TABLE 4. The model hyperparameters.
Hyperparameter
the dimension of the attention vector
The number of attention heads
Learning rate
Regularization
the number of epochs
Dropout
Tested value(s)
[20, 28]
[1, 4, 8]
[0.001, 0.01]
[0.0005, 0.001, 0.005]
100
0.6
Finally, the resulting feature space is passed through a feed-
forward neural network and prediction y′i is calculated:
X ′′
y′i = σ(
zi k .wi k + bi k )
(9)
i∈V
FIGURE 3 presents the overall framework of the proposed
model. According to the previous section, we make a graph of
healthcare providers’ relationships, including two edge types:
the same beneficiary (r1) and the same physician (r2). As seen
in FIGURE 3, we have a two-layer GAT embedding and a
feed-forward layer for each edge type. Resulting embeddings
from two edge types are concatenated and used as an input
for a feed-forward layer. We Minimize the Cross-Entropy
equation represented in (10) between the ground truth and the
prediction:
1 X
yi log (y′i ) + (1 − yi ) log(1 − y′i )
(10)
CE =
V i∈V
where yi denotes the actual label for node i and y′i denotes the
predicted label for node i.
IV. EXPERIMENTAL RESULTS
A. IMPLEMENTATION DETAIL
We initialized the parameters randomly and optimized the
model using Adam [111]. Hyperparameters and their values
are shown in TABLE 4 . For some hyperparameters, a range
of values was considered and tested. We split the dataset into
a training set, a validation set, and a test set with a ratio of
60%, 20%, and 20%, respectively, to ensure fairness.
Our experiments were divided into two stages. At first, we
performed our model with different hyperparameters to find
the most appropriate ones. In this stage, we also implemented
a Graph transformer network (GTN) [112], a deep learning-
based graph embedding method for heterogeneous graphs
to evaluate the performance of GATs in our model. In the
second stage, we compared the effectiveness of our model
with some conventional algorithms including, Random Forst
[113], XGBoost [114], and Logistic Regression [115], to
check if the complexity in our model create an advantage
over these algorithms. We chose these algorithms because
Akbar et al. [32] and Herland et al. [19] have shown that these
algorithms have the best performance on imbalanced datasets.
It should be noted that we tested our approach on publicly
available datasets in which the inter-parties relations are avail-
able. Thus, we could not test our approach on datasets such
as Medicare Part B [116]. Therefore, we compared our model
with the algorithms on [32] (Random Forst and XGBoost),
VOLUME 11, 2023
This work is licensed under a Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 License. For more information, see https://creativecommons.org/licenses/by-nc-nd/4This article has been accepted for publication in IEEE Access. This is the author's version which has not been fully edited and
content may change prior to final publication. Citation information: DOI 10.1109/ACCESS.2024.3425892
Sh. Mardani et al.: Preparation of Papers for IEEE TRANSACTIONS and JOURNALS
FIGURE 3. The overall framework of the proposed model.A two-layer GAT embedding and a feed-forward layer are considered for each edge type. The
resulting embeddings are concatenated and used as an input for a feed-forward layer.
which has the best performance on the database. For more
certainty, we added the evaluation of the algorithm logistic
regression, which is shown as the best performance algorithm
on the Medicare Part B in [19].
B. RESULT ANALYSIS
For evaluation, we calculated the measure of precision, recall,
F1-score, and AUC. Since, the performance of our model and
GTN are depended on their hyperparameters, we tested these
models with different hyperparameters to find the best per-
formance for each one. In TABLE 5 we have summarized the
best results of our model and GTN model on the healthcare
provider fraud detection dataset. Also, in FIGURE 4 to 7 we
have presented precision, recall, F1-score, and AUC respec-
tively. Considering different hyperparameters and measures,
our GAT-based model performs better than the GTN. As
seen the GAT-based model (attention dimension=20, head=1,
LR=0.001, Regularization= 0.005), the GAT-based model
(attention dimension=28, head=4, LR=0.01, Regularization
= 0.0005), the GAT-based model (attention dimension=28,
head=8, LR=0.001, Regularization = 0.001), and the GAT-
based model (attention dimension=28, head=4, LR=0.01,
Regularization = 0.0005), show the best precision, recall, F1-
score and AUC respectively.
Although GTN can recognize more complex relationships
and dependencies due to the use of additional transformer-like
layers, it has not performed well here. By investigating the
precision measure on the train set, we found that the precision
of the GTN model was close to 0.98 in all configurations for
the train data set, while the precision was not more than 0.5
for validation and test data sets. It can indicate that the model
is overfitting because of its inappropriate architecture for this
specific task. The result shows that the GAT-based model,
VOLUME 11, 2023
with its more straightforward architecture, has generalized
and performed better.
The results presented in TABLE 6 show that these mod-
els also outperform Random Forest, XGBoost, and Logistic
Regression models. However, in our domain, detecting a non-
fraudulent provider as fraudulent (type 1 error) is better than
not detecting a fraudulent provider as fraudulent (type 2 error)
[32]. Therefore, we chose the GAT-based model (attention
dimension=28, head=4, LR=0.01, Reg= 0.0005) as our best
model because it has the highest recall.
V. CONCLUSION
Healthcare fraud leads to increased healthcare expenses for
insurers, increased premiums for policyholders, dissatisfac-
tion of legitimate patients, and severe damage to the health-
care system. Thus, healthcare fraud detection is critically im-
portant. In this paper, we have proposed a healthcare provider
fraud detection model that uses a GAT to capture the relation
and interdependency between participants in claims to find
fraudulent behavior. We use both intrinsic and relationship
features simultaneously in our model to increase the fraud
detection model performance. Our model outperforms other
fraud detection methods in this domain with higher recall.
For future work, we consider investigating the effect of us-
ing other features (e.g., diagnosis code) and applying feature
selection methods. Also, we will use an under/oversampling
method due to the fact that fraud detection datasets are nor-
mally biased toward majority classes (normal class) and lead
to poor performance on the minority class (fraud class). Thus,
by under/oversampling, the model can learn to better distin-
guish between classes and improve overall performance.
Since the model is applicable in every domain where fraud
schemes are formed based on the parties’ relationships, per-
9
This work is licensed under a Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 License. For more information, see https://creativecommons.org/licenses/by-nc-nd/4This article has been accepted for publication in IEEE Access. This is the author's version which has not been fully edited and
content may change prior to final publication. Citation information: DOI 10.1109/ACCESS.2024.3425892
Sh. Mardani et al.: Preparation of Papers for IEEE TRANSACTIONS and JOURNALS
FIGURE 4. Precision of our model and GTN model on the dataset for different hyperparameters.
FIGURE 5. Recall of our model and GTN model on the dataset for different hyperparameters.
10
VOLUME 11, 2023
This work is licensed under a Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 License. For more information, see https://creativecommons.org/licenses/by-nc-nd/4This article has been accepted for publication in IEEE Access. This is the author's version which has not been fully edited and
content may change prior to final publication. Citation information: DOI 10.1109/ACCESS.2024.3425892
Sh. Mardani et al.: Preparation of Papers for IEEE TRANSACTIONS and JOURNALS
FIGURE 6. F1-score of our model and GTN model on the dataset for different hyperparameters.
FIGURE 7. AUC of our model and GTN model on
