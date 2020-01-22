# Recovering from Biased Data: Can Fairness Constraints Improve Accuracy?

[Link](https://arxiv.org/pdf/1912.01094.pdf)

Rather than proposing a new fairness constraint/measure, they consider how different existing constraints can "help a learning algorithm to recover from biased training data and to produce a more accurate classifier."

Let's say there are data points corresponding to individuals, some of whom are part of advantaged group $A$ and some of whom are part of disadvantaged group $B$. Concern they are attempting to address: training data is potentially biased against group $B$ in that it systematically misrepresents the true distribution over features and labels in group $B$, while the training data for group $A$ is drawn from the true distribution for group $A$.

* I feel like this is only a small slice of what people mean when we talk about "biased datasets." Biased data IMO refers to data that may accurately capture the state of the world, including its biases. e.g., if members of Group $B$ have experienced systemic oppression, we should expect that their real-world risk distributions should be different than that of Group $A$.

Second form of bias: Human labelers might have inherent biases, causng some positive members of Group $B$ to be mislabeled sa negative.

Upshot: ERM subject to equal opportunity recovers the true Bayes Optimal hypothesis under a wide range of "bias models." 🚨 **Major assumption**: we assume that under the true data distribution, the Bayes-Optimal classifiers $h^*_A$ and $h^*_B$ classify the same fraction $p$ of their respective populations as positive, $h^*_A$ and $h^*_B$ have the same error rate $\eta$ on their respective populations, and that these errors are uniformly distributed.
* I feel like this assumption is super flawed! What about the application to criminal justice where we know that women have a lower risk of recidivism than men? In this case, enforcing equality actually hurts women.

## Related work
Results "avoid triggering" known impossibility results by assuming equal base rates across groups. Authors acknowledge that this is not necessarily realistic across all scenarios.

## True label generation
* Assume there exist a pair of Bayes optimal classifiers $h^**=(h^*_A,h^*_B)$ with $h^*_A,h^*_B\in\mathcal{H}:\mathcal{X}\rightarrow\{0,1\}$
* First we draw a data point $x$. With probability $1-r$, $x\sim\mathcal{D}_A$ and with probability $r$, $x\sim\mathcal{D}_B$
* Then we model the labels being produced by evaluating $h^*(x)$, using the classifier corresponding to the demographic group of $x$. If $x\in A$, then $h^*(x)=h^*_A(x)$. If $x\in B$, then $h^*(x)=h^*_B(x)$.
    * We assume that $h^*$ is imperfect and independently with probability $\eta$, the true label of $x$ gets does not correspond to the prediction $h^*(x)$.
    * In this setup, the classifier outputs a label, not a risk score.
* **Assumption**: $p=P(h_A^*(x)=1|x\in A)=P(h_B^*(x)=1|x\in B)$. 
    * Assumes the true probability of positive sample is same across both groups.
    * Also assumes that $\eta$ is the same for both groups.
* The two groups will have equal base rates (fraction of positive labels): i.e. $p(1-\eta)+(1-p)\eta$.

## Biased Training Data
1. **Under-representation bias**: positive examples from group $B$ are under-represented in the training data. It's the same label generation process as before, except if $x\in B$ and $y=1$, we discard the data point with probability $1-\beta$.
    * If $\eta=0$, then positive and negative regions are disjoint, so if we draw enough examples, we will see enough positive examples to find a low empirical error classifier, even if $\beta$ is high.
    * If $\beta$ is small (meaning that most positive examples get discarded) and $\eta$ is high enough, there could be more negative examples of Group B than positive examples in the positive region of $h^*_B$, in which case the model will classify *all* individuals from Group B as negative!
2. **Labeling bias**: same process as before, but instead of randomly discarding positive examples from $B$, we independently with probability $v$ flip the label to negative.

### General model of biased training data
Three parameters: $\beta_{POS}, \beta_{NEG}, v$

Discard each positive example of Group $B$ with probabiity $1-\beta_{POS}$ and discard each negative example of Group $B$ with probability $1-\beta_{NEG}$. Then, each positive example of Group $B$ is mislabeled as negative with probability $v$ to model Labeling Bias. 
* ❗ important: under-representation comes first

### Fairness constraints to consider
Equal opportunity: enforces that true positive rate in Group $B$ is the same as in Group $A$

Equalized odds: requires that false positive rates are equal across both groups (in addition to requiring the same true positive rate)

Demographic parity: $P(h(x)=1|x\in A)=P(h(x)=1|x\in B)$

Strategy also considered: **data Re-Weighting** where we change the training data distribution to up-weight the observed fraction of positives in the training data from Group $B$ to match the fraction of positives from the Group $A$ training data

Importantly, during the training process we *only have access to samples from the training distribution*, so we must check the fairness criterion on the biased training data.

## Recovery behavior across bias models
Two failure modes that we should be concerned with:
1. Bayes Optimal hypothesis may not satisfy fairness constraint when *evaluated on the biased data*
2. Within set of hypotheses satisfying fairness constraint, another hypothesis may have lower error than Bayes Optimal classifier $h^*$ on the biased data

### Under-representation bias
Equal Oppportunity and Equalized Odds avoid both failure modes. Demographic Parity fails to recover $h*$ in most cases.

* Equal Opportunity strongly recovers from under-representation bias so long as $(1-r)(1-2\eta)+r((1-\eta)\Beta-\eta)>0$.
    * Equalized Odds strongly recovers under the same conditions
* Demographic parity does **not** recover $h^*$! Counterexample: $\eta=0, p=1/2, \beta=1/2$. (So, zero labels get flipped from the $h^*$, but $1/2$ of the positive examples in Group $B$ will be dropped). The Bayes Optimal Classifier will classify $1/2$ of the examples from group $A$ as positive. The Bayes Optimal Classifier will classify $\frac{1/4}{1/4+1/2}=\frac{1}{3}$ as positive in group $B$; therefore, it will fail demographic parity.

### Labeling bias
Labeling bias cannot be corrected by equalized odds due to failure mode #1. In contrast, $h^*$ does satisfy equal opportunity on the biased data. Also, the re-weighting intervention will recover $h^*$ in this bias model.
* Consider $\eta=0$ but $\mathcal{v}\neq 0$ (so all of $h^*$'s labels are the true labels, but some of the positive examples in group $B$ get flipped). The Bayes Optimal Classifier will have a FPR of 0 on Group A. However, $h^*$ cannot produce a classifier that has a false positive rate of 0 for Group $B$. Classifying an individual whose label was "flipped" as part of the labeling bias as negative (in order to achieve 0 FPR on group $B$) would require decreasing the true positive rate from 1.

### General bias model
~~I don't fully understand this yet~~, but the up-shot is that re-weighting the data is **no longer sufficient** to recover the true classifer. They prove that equal-opportunity-constrained ERM recovers the Bayes Optimal Classifier $h^*$, so long as Group $A$ has sufficient mass and the signal is not too noisy.
* Consider: $\eta=0$ ($h^*$ labels are true labels), $p=1/4$, $\beta_{NEG}=1/3$, $\beta_{POS}=1$ (no positives in $B$ get filtered out, but $2/3$ of negatives in $B$ get filtered out), and $v=1/2$ (half of positives in $B$ get flipped to negative). In this case, $1/4*(1/2)=1/8$ of $B$ are positive, and $(3/4)(1/3) + 1/8=3/8$ are negative. Note that the overall fractions are still correct between $A$ and $B$; 1/4 of group $A$ has a positive label in the dataset and 1/4 of group$B$ has a positive label in the dataset as well.

## Conclusion
Even when the fair solution is the **right hypothesis in terms of both true accuracy and fairness**, the fairness interventions can be **tricked by the bias in the data**.

> The high-level message of this paper is that fairness interventions need not be in competition withaccuracy and may improve classification accuracy if training data is unrepresentative or biased; however these results will be connected to the true data distributions and features of the biased data-generation process.

## Lingering questions
* This is stupid, but is empirical risk minimization basically the same thing as minimizing a loss function on a training set?