# Chapter 9: Chunking, Indexing, and the Query You Did Not Anticipate

**Summary**: *Nearly every retrieval pipeline gets tuned in the wrong place. Teams adjust how much comes back, how often the index rebuilds, and how the answer is generated, when the defect is what a single retrieved piece contains. This chapter separates the decisions that change the content of a chunk from the ones that only change its quantity, sets out where stale material actually enters and leaves a pipeline, and covers the documented costs of feeding documents to a model.*

## Chunk Size Is Not the Chunking Decision

The characteristic failure is easy to describe and hard to see in an option list. A retrieved passage contains part of an answer, and the part that governs it lives somewhere else: the exception is in a different clause, the definition is in a different section, the qualifier is two paragraphs up.

Now watch the remedies people reach for. Increase the number of passages retrieved. Enlarge the chunks. Add overlap between adjacent chunks. Retrieve, then re-rank.

Every one of those changes how **much you get back**. None of them changes what a **retrieved piece contains**. More passages means more of the same kind of fragment. Bigger chunks means a bigger arbitrary window that still cuts wherever it happens to land. Overlap means the boundary moves slightly, which helps only if the governing text happened to sit just outside it.

The fix is to **cut each retrieval unit** so that it **stands on its own as a piece of meaning**, with **whatever it leans on attached** to it. A clause carrying its own definitions and exceptions can answer a question by itself. Four hundred tokens of whatever happened to fall inside a window cannot, and no human reading it would think otherwise.

Name it: **chunk by corpus, index by query**. Chunk boundaries follow the structure of the material you have. What you index and how you match follows the questions people ask. Those are different decisions, and they are usually made as one, badly.

## The Citable Unit Is the Block You Supplied

If a scenario cares about attribution granularity, one documented fact settles it.

A **text block** is the **minimal citable unit**, and Claude cites **whole blocks** rather than **substrings** inside them. Finer-grained citations are obtained by splitting the content into smaller blocks.<sup>[1]</sup>

That converts a presentation complaint into a chunking decision. When somebody says the citations are too coarse and point at a whole page, no prompt instruction improves it. Your **block boundaries are your citation boundaries**.

Two related details are worth having exactly.<sup>[1]</sup>

**Citations are disabled by default for search results**. A design that assumes attribution is automatic will ship without it.

When a search **fails or returns nothing**, return a **plain text block describing what happened** rather than **raising an error**. The model then explains the empty result and the conversation continues. Raising an error converts a normal outcome, no matching documents, into a broken turn.

The document types differ in how they are chunked and cited, and the third row is the one that matters for control:<sup>[2]</sup>

| Document type | How it is chunked | How it is cited |
|---|---|---|
| Plain text | Automatically chunked into sentences | By character indices |
| PDF with text content | Automatically chunked into sentences | By page number |
| **Custom content** | **No additional chunking** | By **block indices**, and recommended where you want more granular citations |

**Custom content is the option that gives you the boundaries**, because nothing re-chunks it after you decide. If a scenario wants control over what gets cited, that is the row.

## What a PDF Actually Costs

Feeding documents in has a documented price, and the range is wide enough to change a design.<sup>[3]</sup>

A **page of PDF typically consumes 1,500 to 3,000 tokens** depending on how dense it is. And the processing mode matters more than most people expect: text extraction alone uses roughly **1,000 tokens for a 3-page PDF**, while full visual understanding, which processes every page as both text and image, uses roughly **7,000 tokens for the same 3 pages**.

That is a sevenfold difference for the same document, so the **mode is a cost decision, not a quality preference**. Visual understanding earns its price on documents where layout carries meaning, tables, forms, diagrams, signatures. On a wall of paragraphs it is paying seven times over for information the text already contains.

One more consequence follows from the same page: PDF support relies on Claude's vision capabilities, so it **inherits the considerations that apply to any vision task**. A design treating PDF ingestion as a text problem has missed that.

## Stale Content Enters at Ingestion and Leaves at Filtering

When a pipeline keeps returning out-of-date material, there are exactly two stages where you can act on it, and the reason is structural rather than a matter of preference.

**Stale content enters at ingestion**. Something wrote it into the index.

**Stale content leaves at retrieval filtering**. Something declined to return it.

Every other stage sits downstream of the set of passages. Retrieving more of them, rebuilding more often, or changing how the answer is generated all operate on or after a passage set that already contains the stale material. You **cannot fix the contents of a set** by adjusting what **happens to it afterwards**.

That is why, when a scenario asks for two measures against staleness, the pair is ingestion and filtering. Cadence sounds like the obvious answer and is not: rebuilding nightly still leaves a full day of staleness, and if the problem is that superseded documents were never removed, no rebuild frequency touches it at all.

## Embeddings, and the Side the Text Sits On

Two facts that scenarios are built on.

**Anthropic does not offer its own embedding model** and points to a third-party provider.<sup>[4]</sup> An option that describes calling an Anthropic embedding endpoint is describing something that does not exist.

**Documents and queries are embedded with different input types**, marking which side of the retrieval each piece of text is on.<sup>[4]</sup> That asymmetry is easy to lose in an implementation and produces quietly degraded matching rather than an error.

There is a parallel worth drawing to the previous chapter's tool-catalogue problem, because the documented solution has the same shape. The tool search tool keeps a very large catalogue usable by searching it and loading only the definitions a task needs, rather than loading everything up front. Two variants exist, one where Claude builds regular expression patterns and one where it issues natural language queries against BM25, and a custom implementation can use embeddings or semantic search.<sup>[5]</sup> **Selecting from a large set at request time** is the **same problem** whether the set is **documents or tools**, and BM25 appearing beside embeddings here is the same lexical-versus-semantic split from chapter 6. Deferral is not a universal improvement, and the documentation says where it stops paying: **standard tool calling fits better under about ten tools**, when **every tool is used on every request**, or when the definitions are **small enough not to matter**. Deferred definitions still **cost input tokens once loaded**, so what changes is when they are **charged**, not whether.

## Reuse of One Document Is a Caching Problem, Coverage of Many Is a Retrieval One

Two scenarios read almost identically and take opposite answers, so it pays to have the discriminator ready rather than deriving it under time pressure.

**One large document, queried repeatedly inside one piece of work.** A filing, a contract, a specification. What repeats between requests is the **material itself**, unchanged, while the question moves. That is a **caching** shape: the document sits in the cacheable prefix and is paid for once across the run rather than on every question.

**A large collection where any one question needs a small slice.** A standards library, a knowledge base, an archive. What repeats is the **shape of the request** rather than its content, and no two questions want the same passage. That is a **retrieval** shape: index it, fetch what bears on the question, leave the rest unread.

The test is one question: **what stays identical between two consecutive requests?** Same material and a different question means cache it. Different material every time means retrieve it.

Then the constraint that overrides the performance argument entirely. When the content **may not leave the customer's boundary**, building an index is not on the table whatever its retrieval properties, because an index is a **durable second copy of the content** living in another system. Caching moves nothing outside the request path it was already on. Where a copy **comes to rest** is a **governance question**, and it gets answered **before the cost question does**.

Two summarising moves look like answers here and are not. **Summarising once at intake** and querying the summary afterwards is cheap and **discards the material the later questions are about**, so it trades the thing being asked for. **Moving to a smaller model** changes the rate on the meter and not the reading. The same oversized request is still assembled and still sent, so the volume that caused the problem is exactly where it was.

## How Wrong Answers Are Built on Retrieval Pipeline Questions
- **Quantity offered against a content defect**. The option retrieves more passages, enlarges chunks, adds overlap or re-ranks. The test: does this change what is **inside a retrieved piece**, or only how **many pieces** there are?
- A **downstream stage offered against an upstream problem**. The option tunes generation, breadth or cadence against material that is already in the passage set. The test: at which **stage does the bad content enter**, and at which **stage can it be excluded**?
- **Prompting offered against a structural boundary**. The option asks for finer citations by instruction. The test: what is the **smallest unit the system can cite**, and who **decided that boundary**?
- **Assuming a default that is off**. The test: does this **behaviour need enabling**? Citations for search results do.

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| Retrieved text misses the governing clause, definition or exception | Cut units that stand alone, and attach what they lean on | More passages, bigger chunks, overlap, or re-ranking |
| Citations are too coarse to be useful | Split the content into smaller blocks | Instructing the model to cite more precisely |
| You need control over citation boundaries | Custom content, which is not re-chunked and is cited by block index | Plain text or PDF, both auto-chunked into sentences |
| Attribution is expected on search results | Enable it; it is off by default | Assuming citations are automatic |
| A search returns nothing, or fails | Return a plain text block describing the outcome | Raising an error, which breaks the turn over a normal result |
| Stale material keeps being returned, two measures wanted | Act at ingestion and at retrieval filtering | Rebuild cadence, broader retrieval, or generation changes |
| PDF cost is a concern | Choose the processing mode deliberately; visual understanding costs about seven times text extraction | Treating mode as a quality setting with no price |
| Layout carries meaning: tables, forms, diagrams | Full visual understanding earns its cost | Text extraction, which discards the thing that mattered |
| An embedding model is needed | A third-party provider; Anthropic does not offer one | An option describing an Anthropic embeddings endpoint |
| A very large tool catalogue is hurting the context | Search the catalogue and load only what the task needs | Loading every definition up front |
| One large document is queried many times in one case | Cache it as the prefix; the material repeats and the question moves | Indexing it, which builds a second copy to solve a reuse problem |
| Content may not leave the customer boundary | An external index is out; a cache creates no durable copy elsewhere | Choosing on retrieval quality and treating the boundary as a later detail |

The habit worth carrying: for any proposed retrieval change, ask whether it alters the **contents of the passage set** or only its **size**. Most wrong answers here are quantity changes offered against a quality problem.

## What to Remember
- Retrieval failures where the **governing text is missing** are **content defects, not quantity defects**. More passages, larger chunks, overlap and re-ranking all change how **much comes back** and none of them changes what a **retrieved piece contains**.
- The fix is a retrieval unit that **stands on its own as a piece of meaning**, with **whatever it leans on attached**: a clause carrying its own definitions and exceptions, rather than a **fixed window that cuts wherever it lands**.
- Chunk by **corpus**, index by **query**. Boundaries follow the **structure of the material**; matching strategy follows the **questions people actually ask**. These are **two decisions** and they **get made as one**.
- A text block is the **minimal citable unit** and **whole blocks get cited, never substrings**, so finer citations come from **smaller blocks**, not from instructions. Your **block boundaries** are your **citation boundaries**.
- **Citations are off by default** for search results. When a search fails or returns nothing, **return a plain text block describing the outcome** so the model can explain it, rather than **raising an error and breaking the turn**.
- **Plain text** is auto-chunked into sentences and cited by **character index**; **PDFs with text** are auto-chunked into sentences and cited by **page**; **custom content** receives **no additional chunking** and is cited by **block index**, which makes custom content the choice when you want to **control granularity**.
- A PDF page typically costs **1,500 to 3,000 tokens**. Text extraction alone runs about **1,000 tokens for 3 pages** against roughly **7,000 tokens for the same 3 pages** with full visual understanding, so the **mode is a cost decision**, worth paying only where **layout carries meaning**. PDF support runs on **vision capabilities** and inherits their considerations.
- Stale content enters at **ingestion** and can be excluded at **retrieval filtering**. Those are the **only two stages** that act on it. Rebuild cadence, retrieval breadth and generation are all **downstream of the passage set** and cannot change what is in it.
- **Anthropic does not provide an embedding model**, and documents and queries are embedded with **different input types** marking which side of the retrieval each sits on.
- A very large tool catalogue is solved the **same way as a large corpus**: **search it at request time** and **load only what is needed**, by regular expression, by natural language against BM25, or by a custom embedding-based search.
- **Repeated material with a moving question is a caching shape**; **fresh material on every request is a retrieval shape**. The test is **what stays identical between two consecutive requests**. A **boundary constraint settles it before performance does**, because an **index is a durable second copy** of the content and a **cache is not**.

## Endnotes for this chapter

1. Search result content blocks let Claude attribute answers to supplied content; the page states that the text block is the minimal citable unit, that Claude cites whole blocks rather than substrings within a block, and that finer-grained citations are obtained by splitting search result content into smaller blocks. The same page states that citations are disabled by default for search results, and advises that when a search fails or returns nothing the tool should return a plain text block describing the outcome rather than raising an error, so that the model explains the empty result and the conversation continues. Anthropic, *Search results* (Claude Platform Docs, accessed 2026-08-25), [https://platform.claude.com/docs/en/build-with-claude/search-results](https://platform.claude.com/docs/en/build-with-claude/search-results)

2. Anthropic documents three document types for citations: plain text documents, which are automatically chunked into sentences and cited by character indices; PDF files with text content, also chunked into sentences and cited by page number; and custom content, which receives no additional chunking and is cited by block indices, and is recommended where more granular citations are wanted. Anthropic, *Citations* (Claude Platform Docs, accessed 2026-08-25), [https://platform.claude.com/docs/en/build-with-claude/citations](https://platform.claude.com/docs/en/build-with-claude/citations)

3. Each page of a PDF typically uses 1,500 to 3,000 tokens depending on content density; Anthropic contrasts a text-extraction-only mode using approximately 1,000 tokens for a 3-page PDF with a full visual understanding mode that processes each page as both text and image and uses approximately 7,000 tokens for the same 3 pages. The page also states that PDF support relies on Claude's vision capabilities and is therefore subject to the same considerations as other vision tasks. Anthropic, *PDF support* (Claude Platform Docs, accessed 2026-08-25), [https://platform.claude.com/docs/en/build-with-claude/pdf-support](https://platform.claude.com/docs/en/build-with-claude/pdf-support)

4. Anthropic states that it does not offer its own embedding model and directs readers to a third-party embeddings provider; the page's worked retrieval example embeds documents and queries with different input types, marking which side of the retrieval the text sits on. Anthropic, *Embeddings* (Claude Platform Docs, accessed 2026-08-25), [https://platform.claude.com/docs/en/build-with-claude/embeddings](https://platform.claude.com/docs/en/build-with-claude/embeddings)

5. The tool search tool is documented as the way to keep a very large tool catalogue usable: rather than loading every definition into the context window at the outset, Claude searches the catalogue and loads only the definitions a task needs. Two variants are documented, one in which Claude constructs regular expression patterns and one in which Claude issues natural language queries against BM25, and the page notes that a custom tool search can be implemented using embeddings or semantic search. Anthropic, *Tool search tool* (Claude Platform Docs, accessed 2026-08-25), [https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool)
