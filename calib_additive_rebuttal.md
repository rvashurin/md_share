### yPFe

> The latest table represents, basically, an interpolation between the multiplicative CoCoA and a Confidence Score?

It can be viewed in that way, yes. We completely agree that an additive formulation of risk would be more grounded in theory, and keeping in mind the issue of different levels of signal in the confidence and consistency terms, we tried to used the confidence term itself to serve as an adaptive scaling factor on consistency. This allows to disentangle the signal from the confidence term from the signal from the consistency term. However, the empirical results still clearly show that multiplicative risk is the one to go for optimal performance across different tasks.

> also not sure what is ProbCoCoA PPL, since PPL is a log space quantity unlike MSP

ProbCoCoA PPL just uses the exponent of the log-perplexity. Indeed it's not really a probability, as it can easily be larger than 1. We added that to cover as much ground as possible with this ablation.

> These are all interesting details and observations, and I hope that the authors will discuss those in their paper.

We will surely expand our ablations with the results obtained during this discussions and augment our theoretical sections with some of the thoughts that were discussed here as well, all in the camera-ready version, if accepted.

We deeply thank the reviewer for the push towards additional empirical confirmation of the method and expanding the ablation substantially. We believe that we have exaustively answered reviewer's concerns during the discussion and kindly ask to revise the final score.


### aWz6

>> I am not convinced but rather think MBR can be consider as a new angle of combing confidence and consistency score...
   I like this new story and think this is much better motivated that the current version, please consider revise the paper with this version.

We will surely revise the paper accordingly and thank the reviewer for bringing up this discussion.

>> Although I think theoretical formulation is missing.
> Agree on its empirical performance, but again I think this paper lack on the theoretical justification. A way to make this story better is to related to the motivation of how MBR is developed (see previous question).

As discussed with the reviewer **yPFe**, we agree that from theoretical standpoint it would be easier to justify the additive combination of the risks. However, our **extensive** ablations of various forms of additive combinaions reported in the paper and performed during the rebuttal have conclusively shown that they fall short of multiplicative in our benchmark. Furthermore, we provided a motivation for the particular choice of risk combination (see previous responses) which the reviewer has described as "making a lot of sense". We will certainly add them to the theoretical discussion in the paper, if accepted.

> I don't think the authors answered my question well, I am suggesting that in this [paper](https://arxiv.org/abs/2406.15627) regressing uq score with response quality does not seems to make sense other than binary categorical values. - which is related to but not directly refers to calibration.

Can you elaborate on why this approach is not suitable for continuous output scores? As we see it, it was proposed specifically to accommodate for both unbounded UE scores and continuous response correctness. The similar approach was concurrently proposed by another paper as well (https://aclanthology.org/2024.emnlp-main.18.pdf), thus we believe it is well-grounded in the recent literature on the subject.

> Thanks for the error bars, I am wondering how do you select the right scoring function if you do not have a hold out set? It seems like score influence the results quite a lot. (And are you sure these are significant?)

We selected the similarity score function based on its use in modern well-performing methods like https://arxiv.org/abs/2307.01379 as well as based on empirical performance. We stress that we use the same similarity function **across all experiments** (we do not change the function from task to task) and thus we did not need to use a held-out set specifically for this choice.

> Sounds fair, would have been better to explore more scoring functions though.

We have tested 4 scoring functions spanning simple lexical approach like Rouge-L, NLI, semantic similarity and factuality-based similarity functions. Can you elaborate on which kinds of similarity measurement that we missed, may best augment our existing results from your perspective?
