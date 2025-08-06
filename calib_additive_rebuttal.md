### Awk2

> **Lack of verbalized confidence baselines**

Several studies have examined how model size affects the ability to express verbalized uncertainty, with Kadavath et al. [1] and Xiong et al. [2] both finding that calibration and failure-prediction performance improve as model size increases and smaller models struggle with a task. Vashurin et al. [3] benchmark several verbalized-confidence methods and report that their performance for 7–8 B models is significantly worse than baselines like MSP or PPL.

To address concerns about model size in relation to verbalized confidence baselines, we included P(True) [1] among the baselines for the new experiments with larger Gemma-12B model.
Unfortunately, we did not have the capacity to compare CoCoA with verbalized methods across all settings explored in the paper, the results for this configuration are as follows:


| Method          | qa        | ats       | nmt       |
| :-------------- | :-------- | :-------- | :-------- |
| CocoaMSP        | 0.604     | _0.292_   | **0.632** |
| CocoaMTE        | _0.626_   | 0.288     | _0.628_   |
| CocoaPPL        | **0.636** | **0.296** | 0.627     |
| DegMat          | 0.503     | 0.095     | 0.312     |
| EigValLaplacian | 0.487     | 0.095     | 0.302     |
| MSP             | 0.523     | 0.194     | 0.472     |
| MTE             | 0.525     | 0.189     | 0.593     |
| MC-NSE          | 0.446     | 0.044     | 0.451     |
| MC-SE           | 0.480     | 0.012     | 0.340     |
| PTrue           | 0.133     | 0.056     | 0.032     |
| PPL             | 0.551     | 0.217     | 0.557     |
| SAR             | 0.500     | 0.089     | 0.483     |
| SemanticEntropy | 0.496     | 0.013     | 0.346     |

As can be seen, (1) Cocoa methods perform well, consistent with results from smaller models. (2) The evaluated verbalized uncertainty method demonstrates weaker performance.

> real-world scenarios requiring an **explicit** confidence/uncertainty value within a [0,100%] range for decision-making

We agree that in real-world decision-making scenarios, providing scores in an interpretable range (0–100%) is indeed important. We emphasize that the isotonic regression–calibrated scores presented in our rebuttal naturally fall within this range (they do transform the scores into the same range as the quality score, usually 0–100%) and can therefore be directly communicated to end users.

However, it is important to note that many existing SOTA-methods like Semantic Entropy or SAR, also provide scores that are not bounded to a specific range. In selective generation tasks, the primary concern is the method's performance on the selection objective itself; scores can subsequently be scaled or fitted to a desired range for presentation to the end user.

When it comes to ECE , while we agree that calibration is an important topic, our focus in this work is on error detection, and we therefore consider calibration to be somewhat out of scope for this submission-hence our original choice of evaluation metrics. It is worth noting that the quality of probabilistic forecasts can be decomposed into calibration and sharpness [4], making calibration only one component of prediction error. Even perfectly calibrated models can perform poorly at prediction, which is why we treat calibration as a secondary metric in our study.


Traditional calibration metrics like ECE are, since in all but two tasks, evaluation is based on continuous quality scores rather than binary outcomes.
Thus at the request of the reviewers, we used a known approach (see [5], [6]) to calibrate unbounded scores w.r.t. to expected output quality which we provided in our response. Specifically, we fit an isotonic regression model to map raw scores to observed output quality, and then report the mean squared error (MSE) between the calibrated scores (ranges from 0 to 1) and the actual quality of the text (ranges from 0 1). These allows us to evaluate how close are the UE scores to the actual quality of the text, which is a more relevant metric for our task than traditional ECE.

[1] https://arxiv.org/abs/2207.05221
[2] https://openreview.net/forum?id=gjeQKFxFpZ
[3] https://aclanthology.org/2025.tacl-1.11/
[4] https://sites.stat.washington.edu/raftery/Research/PDF/Gneiting2007jrssb.pdf
[5] https://direct.mit.edu/tacl/article/doi/10.1162/tacl_a_00737/128713/Benchmarking-Uncertainty-Quantification-Methods
[6] https://aclanthology.org/2024.emnlp-main.18.pdf


### yPFe

> Regarding the CoCoA-MSP, the top-performing variant in the tables - $U_{MSP}$ does it use the logarithmic  or just the likelihood?

It uses logarithmic value for the confidence:  $U_{conf} = -\log(p(y^*|x))$.

> though there is a small factual error in line 961, as $-\log(p(y^*|x))$ inverts the order compared to $p(y^*|x)$ because of the negative sign

Indeed, but likelihood formulation uses $1 - p(y^*|x)$ as a $U_{conf}$ so the order is kept between logarithmic and likelihood formulations. We agree that this is confusing, as intuitively confidence implies that higher values represent higher confidence. Following [https://arxiv.org/abs/2305.19187] we use terms uncertainty/confidence to distinguish between types of uncertainty related to the generation process in general, and uncertainty of the particular output $y^*$. To reduce confusion in this matter, we should probably reverse the consistency term as well, to align the whole expression with the intuitive understanding of confidence. We will do this in the camera-ready version (if accepted), thank you for highlighting this issue.

> Perhaps likelihood in range [0,1] and consistency in range [0,1] would lead to a better signal match in the additive setting (rather than log likelihood + log consistency )

Thank you for this important observation. Here we present the results of additive formulation with likelihood instead of logarithm for confidence term:

| Method                       | llama8b/QA | llama8b/NMT | llama8b/SUM |
| :--------------------------- | :--------- | :---------- | :---------- |
| CoCoA MSP                    | 0.451      | **0.519**   | 0.378       |
| CoCoA PPL                    | 0.454      | 0.481       | **0.387**   |
| CoCoA MTE                    | 0.447      | 0.478       | 0.380       |
| Additive CoCoA MSP (1.0)     | 0.424      | 0.403       | 0.330       |
| Additive CoCoA PPL (1.0)     | 0.455      | 0.467       | 0.368       |
| Additive CoCoA MTE (1.0)     | 0.101      | -0.082      | -0.331      |
| Additive ProbCoCoA MSP (1.0) | 0.449      | 0.475       | 0.035       |
| Additive ProbCoCoA PPL (1.0) | **0.459**  | 0.476       | 0.343       |

While additive formulation with normalized likelihood gets close to the the performance of multiplicative CoCoA MSP, its more unstable, and falls short on NMT and SUM. At the same time it requires confidence to be a bounded likelihood, reducing the generality of the multiplicative formulation, where confidence term can be of any nature. For example, using token entropy in additive setting catastrophically reduces performance, while multiplicative formulation retains performance close to other choices of the confidence term.

> At the same time, I do not fully understand why the difference in scales would be such a significant issue for the additive combination in the case of CoCoA-Light, since it already incorporates a small trained model whose output could be normalized to match the distribution of the confidence term.

Indeed, but the distribution of the confidence term can be task-specific. For example,  generations of different lengths have vastly different likelihoods. Learning this distribution would make the method less generally applicable. For example here we provide an additive formulation with a scaling factor for consistency term selected for best performance on QA tasks:

| Method                        | llama8b/QA   | llama8b/NMT   | llama8b/SUM   |
|:------------------------------|:-------------|:--------------|:--------------|
| CoCoA MSP                     | 0.451        | **0.519**         | 0.378         |
| CoCoA PPL                     | 0.454        | 0.481         | **0.387**     |
| CoCoA MTE                     | 0.447        | 0.478         | 0.380         |
| Additive CoCoA MSP (2.0)      | 0.429        | 0.408         | 0.332         |
| Additive CoCoA PPL (1.0)      | 0.455        | 0.467         | 0.368         |
| Additive CoCoA MTE (50.0)     | 0.435        | 0.414         | 0.007         |
| Additive ProbCoCoA MSP (0.55) | 0.447        | 0.487         | 0.044         |
| Additive ProbCoCoA PPL (0.9)  | **0.458**        | 0.474         | 0.352         |

While having strong performance in-domain (QA), it falls short on summarization.

At the same time, please recall that we define our multiplicative risk as $r(y, y' \mid x) = u(y \mid x) \dot (1 - s(y, y')$. This can be also viewed as an **additive** risk of the form $r(y, y' \mid x) = u(y \mid x) - u(y \mid x)s(y, y')$. In this formulation, $r_1 = u(y \mid x)$ represents a pure information-theoretic risk, while $r_2 = -u(y \mid x)s(y, y')$ is the risk from the sequence $y$ possibly being a semantic outlier, scaled by the adaptive factor $u(y \mid x)$ to match the signal level of the information-theoretic term. This can be viewed as applying an individual scaling factor in the additive formulation, instead of selecting a common scaling factor for all inputs.


### aWz6

>> I am not convinced but rather think MBR can be consider as a new angle of combing confidence and consistency score...
   I like this new story and think this is much better motivated that the current version, please consider revise the paper with this version.

We will surely revise the paper accordingly and thank the reviewer for bringing up this discussion.

>> Although I think theoretical formulation is missing.
> Agree on its empirical performance, but again I think this paper lack on the theoretical justification. A way to make this story better is to related to the motivation of how MBR is developed (see previous question).

STILL TODO

> I don't think the authors answered my question well, I am suggesting that in this [paper](https://arxiv.org/abs/2406.15627) regressing uq score with response quality does not seems to make sense other than binary categorical values. - which is related to but not directly refers to calibration.

Can you elaborate on why this approach is not suitable for continuous output scores? As we see it, it was proposed specifically to accommodate for both unbounded UE scores and continuous response correctness. The similar approach was concurrently proposed by another paper as well (https://aclanthology.org/2024.emnlp-main.18.pdf), thus we believe it is well-grounded in the recent literature on the subject.

> Thanks for the error bars, I am wondering how do you select the right scoring function if you do not have a hold out set? It seems like score influence the results quite a lot. (And are you sure these are significant?)

We selected the similarity score function based on its use in modern well-performing methods like https://arxiv.org/abs/2307.01379 as well as based on empirical performance. We stress that we use the same similarity function **across all experiments** (we do not change the function from task to task) and thus we did not need to use a held-out set specifically for this choice.

> Sounds fair, would have been better to explore more scoring functions though.

We have tested 4 scoring functions spanning simple lexical approach like Rouge-L, NLI, semantic similarity and factuality-based similarity functions. Can you elaborate on which kinds of similarity measurement that we missed, may best augment our existing results from your perspective?
