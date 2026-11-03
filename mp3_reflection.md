# MP3 Reflection

## Grounding & abstention

The assistant uses retrieved document chunks as the evidence for generating answers. The `should_abstain()` function uses this threshold to decide whether the retrieved results are relevant enough to proceed to answer generation.

During testing, the assistant initially abstained on some answerable questions even though relevant chunks had been retrieved. For example, the first evaluation showed a 100% retrieval hit rate but a grounded and cited rate of only 50%. I inspected the retrieved scores and excerpts and found that the information needed to answer some questions was spread across multiple chunks.

I improved the system prompt to instruct the LLM to read all retrieved excerpts and combine evidence across chunks where necessary. After this change, the assistant answered the previously refused answerable questions, and the final evaluation achieved a 100% grounded and cited rate.

The assistant correctly abstained on the out-of-domain learner question about Hamza Ali Mazari, as well as the evaluation questions about Professor Moriarty and the boiling point of water. The retrieval threshold is only the initial relevance check: the Moriarty question, for example, retrieved chunks with scores above 0.30 but was still correctly refused because the retrieved excerpts did not contain the answer.

## Citations

The assistant numbers retrieved chunks and uses the numbers as inline citations in its responses. The `validate_citations()` function checks the citation markers in an answer and flags markers that do not map to an actual retrieved source.

During offline self-testing, the function successfully identified a fabricated citation marker. In the final evaluation, all six answerable questions passed the grounded and cited check.

The evaluation confirms that the expected source was retrieved and cited, but this does not by itself prove that every individual claim is fully supported by its cited excerpt. This is an important distinction when assessing citation quality.

## Reading eval table

The final evaluation produced the following results:

- Retrieval hit rate: **100%** (6 answerable questions)
- Grounded & cited rate: **100%**
- Abstention accuracy: **100%** (3 should-abstain questions)
- Hallucination rate: **0%**

All six answerable questions passed, including my two learner questions: one about Colonel Stark and the hydraulic press in *The Engineer's Thumb*, and a multi-hop question tracing the Blue Carbuncle through the people involved in the goose. All three questions requiring abstention were correctly refused.

No single metric was weakest in the final run; all four reached their target values. The earlier grounded and cited rate of 50% showed the main weakness: retrieving the expected story did not always lead to a useful, cited answer. Improving the prompt to combine evidence across retrieved excerpts raised this metric to 100%.

The evaluation set contains only nine questions. Also, the hallucination metric is based on the assignment's evaluation logic and should not be taken as proof that every factual statement in every possible response is free from hallucination. A larger test set and closer review of whether each citation semantically supports its claim would provide stronger assurance.

## One surprise

One surprising observation was that a high retrieval score and a successful retrieval hit did not guarantee that the assistant would answer. In the initial run, the system retrieved relevant chunks for all six answerable questions, but it abstained on three of them.

The issue was not solved by lowering the retrieval threshold. Instead, explicitly instructing the LLM to consider all excerpts together improved its ability to combine information spread across chunks. This showed that retrieval quality and answer-generation quality are separate parts of a RAG system, and both need to be evaluated.
