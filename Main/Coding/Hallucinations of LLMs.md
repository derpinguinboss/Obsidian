[LLM Hallucinations in RAG QA - Thomas Stadelmann, deepset.ai - YouTube](https://www.youtube.com/watch?v=lsZCVmCBRlc)
Inherent Property



Internal Representation?
===
[Beyond Surface Statistics - AI SECRETLY builds visual models of the world - YouTube](https://www.youtube.com/watch?v=YytxbKigcXA)


Huh?
===
Noisy training data
Incomplete user prompts {Garfield example}
Too confident


Solution Attempts
===

Improved Prompts
---
More context in user prompts {Garfield example}
Multi shot prompting
etc.

Detecting Hallucinations
---
__Hallucination Detection Models e.g. BERTScore VS Human Assessment__
Human Assessment
- Very expensive
- Good annotation guidelines are required
- Does not scale well
[[BERTScore]]
- Only usable with source access
- Edge cases: Contradiction in the source documents 
- Used to indicate confidence to the user 

Hard, not impossible
[2305.06311.pdf](https://arxiv.org/pdf/2305.06311.pdf)

Retrieval Augmentation
---
[What is retrieval-augmented generation? | IBM Research Blog](https://research.ibm.com/blog/retrieval-augmented-generation-RAG)
Example BING, Generative Search Engines
[2304.09848.pdf](https://arxiv.org/pdf/2304.09848.pdf)


Conclusion
===
- Hallucinations are one of the biggest challenges for using LLMs
- Retrieval Augmentation and the right prompt can mitigate but not avoid this problem
- Automatic detection of hallucinations as a cornerstone for improving LLM-based systems is possible
- Detection is just a starting point
