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

We also have tried an additive formulation of the form $r(y, y' \mid x) = u(y \mid x) + u(y \mid x)(1 - s(y,y'))$, where $u(y \mid x)$ in the second term acts like an adaptive scaling factor for the consistency:

| Method                        | llama8b/QA   | llama8b/NMT   | llama8b/SUM   |
|:------------------------------|:-------------|:--------------|:--------------|
| CoCoA MSP                     | 0.451        | **0.519**     | 0.378         |
| CoCoA PPL                     | **0.454**    | 0.481         | **0.387**     |
| CoCoA MTE                     | 0.447        | 0.478         | 0.380         |
| Adaptive CoCoA MSP            | 0.430        | 0.429         | 0.353         |
| Adaptive CoCoA PPL            | 0.418        | 0.424         | 0.384         |
| Adaptive CoCoA MTE            | 0.409        | 0.430         | 0.376         |
| Adaptive ProbCoCoA MSP        | 0.449        | 0.478         | 0.041         |
| Adaptive ProbCoCoA PPL        | 0.423        | 0.430         | **0.387**     |

Sadly, this form of additive risk falls short of the multiplicative CoCoA variations as well.

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
