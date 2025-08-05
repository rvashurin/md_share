#### Awk2

> **Lack of verbalized confidence baselines**

While we don't have the time to compare CoCoA to verbalized in all settings considered in the paper, we included P(True) [https://arxiv.org/abs/2207.05221] in the baselines list for Gemma-12b experiments we conducted recently. Here are the results:

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

> real-world scenarios requiring an **explicit** confidence/uncertainty value within a [0,100%] range for decision-making

We kindly disagree with this statement, as decision-making requires selecting a threshold both for bounded and unbounded uncertainty values. While we agree that well-calibrated (and not just bound to a certain range) uncertainty lends itself easier for interpretation, we kindly point the reviewer to the table with results of isotonic calibration that we provided in the rebuttal. This approach directly ties the uncertainty to the expected quality of the output. We note that for some of the tasks (translation especially), there is no measurable difference in the calibration for **any** of the baselines or proposed methods. We believe that such low sensitivity makes for a poor evaluation approach.

At the same time, specifically ECE requires binary quality - correct or incorrect. As suggested by reviewer, we apply softmax to the uncertainty values, interpreting result as a probability of correct answer, and report the ECE on QA tasks where binary accuracy can be applicable:

**TABLE GOES HERE**

#### yPFe

> Regarding the CoCoA-MSP, the top-performing variant in the tables - $U_{MSP}$ does it use the logarithmic  or just the likelihood?

It uses logarithmic value for the confidence:  $U_{conf} = -\log(p(y^*|x))$.

> though there is a small factual error in line 961, as $-\log(p(y^*|x))$ inverts the order compared to $p(y^*|x)$ because of the negative sign

Indeed, but likelihood formulation uses $1 - p(y^*|x)$ as a $U_{conf}$ so the order is kept between logarithmic and likelihood formulations. We agree that this is confusing, as intuitively confidence implies that higher values represent higher confidence. Following [https://arxiv.org/abs/2305.19187] we use terms uncertainty/confidence to distinguish between types of uncertainty related to the generation process in general, and uncertainty of the particular output $y^*$. To reduce confusion in this matter, we should probably reverse the consistency term as well, to align the whole expression with the intuitive understanding of confidence. We will do this in the camera-ready version, thank you for highlighting this issue.

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

While additive formulation with normalized likelihood gets close to the the performance of multiplicative CoCoA MSP, it still more unstable, and falls short on NMT and SUM. At the same time it requires confidence to be a bounded likelihood, reducing the generality of the multiplicative formulation, where confidence term can be of any nature. For example, using token entropy in additive setting catastrophically reduces performance, while multiplicative formulation retains performance close to other choices of the confidence term.

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
