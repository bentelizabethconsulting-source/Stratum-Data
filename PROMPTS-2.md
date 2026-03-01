# Stratum-Data — Prompts, Session 2
*Continuation of the design conversation. For Session 1 prompts see PROMPTS.md.*

---

1. Ok, thank you. Now. I understand AIs are prone to hallucination, so please look for and provide a set of journal hyperlinks linking to the peer reviewed articles you looked at to provide answers to the different statements made in the readme.md and notes.md files. If there is no existing source, simply say so, do not make up a citation. I am more interested in factual answers than polished or impressive seeming ones. Can you do that?

2. Can you add that note and also record the prompt just used in new README and PROMPTS-2.md files

3. Can you copy your entire reply word for word, above both of the last two prompts, and add this to a Verification-Claude.md file, and add the preceding prompt and this one to the prompts-2.md file? I will be looking up the URLs to determine if they exist and say what you claim in a little while. I also should tell you that for any scholarly paper it is best to only use peer reviewed sources. Do you fail to because you do not have access to the articles? Or because they are not popular on the internet?

---

## Notes on Session 2

The first prompt in this session produced an important honesty audit of the content generated in Session 1. Key findings recorded here for future reference:

**Verified real sources found:**
- TF-IDF: Spärck Jones (1972), *Journal of Documentation*
- Louvain algorithm: Blondel et al. (2008), *Journal of Statistical Mechanics* — https://arxiv.org/abs/0803.0476
- Leiden algorithm: Traag et al. (2019), *Scientific Reports* — https://www.nature.com/articles/s41598-019-41695-z
- WGCNA: Langfelder & Horvath (2008), *BMC Bioinformatics* — https://link.springer.com/article/10.1186/1471-2105-9-559
- semopy: Igolkina & Meshcheryakov (2020) — https://arxiv.org/abs/1905.09376
- SEM fit indices (CFI/RMSEA thresholds): Hu & Bentler (1999), *Structural Equation Modeling* — http://expsylab.psych.uoa.gr/fileadmin/expsylab.psych.uoa.gr/uploads/papers/Hu_Bentler_1999.pdf

**Content confirmed as invented / unverified:**
- All specific numbers in the prototype (4,821 emails, 18,340 terms, module percentages, CFI = 0.94, RMSEA = 0.048, r = 0.74, lag = +4h, 38% of cases, <2 second detection time) — fictional illustrative examples, not real benchmarks
- The five cross-modal fusion modes — an original framework, no prior literature found
- Preference for lemmatisation over stemming — common practice claim, no specific paper found for this use case
- WGCNA applied to text — no paper found; WGCNA is from genomics and its applicability to document-term matrices is an open question

*Prompts recorded March 2026.*
