### Eval
**Answer:**
These are pretty well established. Easiest is exact match and token-level F1. We can use LLM as a judge ofc
**Reasoning:**
Much harder. There is chain of thought faithfulness metrics: early exit tests (truncate CoT to see if answer changes over time), per-hop supporting fact precision (ie . do retrieved paths include good reasoning paths?)
**Retrieval:**
RAGAS, entity coverage (ensure topics and answer entities are in the retrieved traversal), path recal (two fold: are retrieval paths consistent and do they include optimal reasoning paths)

### Ideas for novelty:
1. multi-modal (needs literature review)
2. geo-temporal dependency--make nodes location and time aware: this may be hard as data might not conform to this. A thought is something similar to [MIRAGE](https://mirage-benchmark.github.io/) where we ask follow ups to specify. However, we the utility of this is still constrained by presence in existing data sources.
