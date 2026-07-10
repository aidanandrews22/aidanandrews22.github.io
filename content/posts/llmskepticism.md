I do not think LLMs are useless. I am simply skeptical if they will surpass human intelligence.

The best way to sum up my take on LLMs is through analogy—consider fossil fuels, objectively, they have been the most revolutionary commodity for humanity in the energy space. Without fossil fuels, arguably at least, we would not have large-scale adoption of electricity. 

In 2005 (my year of birth), fossil fuels supplied roughly 86% of global primary energy (Energy Institute, Statistical Review). Today, we spend trillions on renewable investment and the entire Paris Agreement to replace fossil fuels. LLMs are in the same boat, wide-scale adoption -> billions in funding on different methods (Lecun, Sutskever, Fei-Fei Li).

I claim LLMs are instrumentally essential for progress toward a truly non-anthropic posthuman intelligence — but not necessarily **architecturally final**.

**The mimicry problem.** Humans playing against AlphaGo or Stockfish claim it is akin to playing an alien. Ke Jie, the world #1 Go player, said after losing 0–3 to AlphaGo in 2017 that it played like 
> *a god of Go” and that “not a single human has touched the edge of the truth of Go.*

AlphaGo Zero learned entirely through self-play with **zero human data** and beat the human-trained version 100–0 (Silver et al., Nature 2017). AlphaZero mastered chess in 4 hours of self-play. These systems discovered knowledge humans never had. LLMs, by contrast, are explicitly benchmarked and optimized to be indistinguishable from humans. Every major LLM benchmark—MMLU, HumanEval, GSM8K—measures fidelity to human answers. If we want intelligence that truly usurps humanity, should it not develop it in an alien way?

**Transformers cannot natively learn continually.** They model a probability distribution over training data. No backpropagation happens during inference; weights are frozen at deployment. Furthermore, Dohare et al. (Nature 2024) showed that deep networks in continual-learning settings
> gradually lose plasticity until they learn no better than a shallow network,
and that in practice 
> the most common strategy for incorporating new data has been simply to discard the old network and train a new one from scratch.

Biological neurons update synaptic weights continuously through LTP, LTD, and spike-timing-dependent plasticity. The brain has no “training phase” versus “deployment phase.” In-context learning is the closest thing we have to inference-time learning. But context windows are limited to 1M tokens, and it has been shown that introducing volume/variety degrades performance (context rot).

**Scaling.**
François Chollet: 
> LLMs do “program fetching,” not program synthesis. “As you’re scaling up, you are not increasing the intelligence of the system one bit.”
Kaplan et al. (2020) warned that power-law scaling means “diminishing returns with increasing scale.” Toby Ord’s analysis (Forethought, 2025) showed that halving test loss requires a **million-fold** increase in compute. An AAAI 2025 survey of 475 AI researchers found **76% said scaling current approaches is unlikely to achieve AGI.** Ilya Sutskever himself declared at NeurIPS 2024: “Pre-training as we know it will end.”

With this understanding, we can now break up progress into two possible branches:

(1) Overcome the limitations of LLMs, which will require circumventing the issues of training, evaluation, and architecture simultaneously. 

(2) LLMs are not the right solution, and we need a novel architecture. The post-transformer landscape already exists. The question is whether the field treats it as supplementary or as the main road.

I am inclined to believe (2).
