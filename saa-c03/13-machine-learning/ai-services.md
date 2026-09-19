# AWS AI/ML Services

| Service | Summary |
|---|---|
| [Amazon Q](#amazon-q) | AWS's generative-AI assistant; Amazon Q Developer is the in-IDE coding variant |
| [Amazon Rekognition](#amazon-rekognition) | Face/object/scene detection in images & video, labeling, celebrity recognition |
| [Amazon Transcribe](#amazon-transcribe) | Speech → text (ASR) |
| [Amazon Polly](#amazon-polly) | Text → speech |
| [Amazon Translate](#amazon-translate) | Language translation |
| [Amazon Lex & Amazon Connect](#amazon-lex--amazon-connect) | Conversational chatbots (Lex); cloud contact center (Connect) |
| [Amazon Comprehend](#amazon-comprehend) | Natural language processing (NLP); Comprehend Medical for clinical text |
| [Amazon SageMaker AI](#amazon-sagemaker-ai) | General-purpose, fully managed platform to build/train/deploy ML models |
| [Amazon Kendra](#amazon-kendra) | ML-powered natural-language document search |
| [Amazon Personalize](#amazon-personalize) | Fully managed, real-time recommendation engine |
| [Amazon Textract](#amazon-textract) | Text/handwriting/form/table extraction from scanned documents |

## Amazon Q

- AWS's generative-AI assistant, integrated as an assistant experience across the AWS ecosystem
- **Amazon Q Developer** — the developer-focused variant: an in-IDE chat assistant for code generation, code explanation, applying security best practices during implementation, and debugging assistance
- Framed as accelerating how fast new applications can be built while baking in security from earlier development phases rather than bolting it on later
- *(Not covered in the slide deck — sourced from Netec live training; see the full quote in `## Notes` below)*

## Amazon Rekognition

- Finds objects, people, text, and scenes in images/video using ML; facial analysis/search (build a "familiar faces" DB or compare against celebrities)
- Use cases: labeling, content moderation, text detection, face detection/analysis (age/gender/emotion), face search & verification, celebrity recognition, pathing (e.g. sports analytics)
- **Content Moderation** — flags inappropriate/offensive content in images/video with a configurable minimum confidence threshold; can route flagged content to Amazon Augmented AI (A2I) for human review

## Amazon Transcribe

- Speech-to-text via automatic speech recognition (ASR)
- Can redact PII automatically; supports automatic language identification for multi-lingual audio
- Use cases: transcribing support calls, closed captioning/subtitling, generating searchable metadata for media

## Amazon Polly

- Text-to-speech using deep learning
- Pronunciation Lexicons customize word pronunciation (stylized words, acronyms)
- SSML (Speech Synthesis Markup Language) adds emphasis, phonetic pronunciation, breathing/whispering effects, and a "Newscaster" speaking style

## Amazon Translate

- Natural, accurate language translation for localizing content (websites, apps) at scale

## Amazon Lex & Amazon Connect

- **Lex** — same tech as Alexa: ASR (speech→text) + NLU (intent recognition); builds chatbots/call-center bots
- **Connect** — cloud-based virtual contact center; receives calls, builds contact flows, integrates with CRMs; no upfront cost, ~80% cheaper than traditional contact center solutions

> Exam-wording cue: Lex is the conversational "brain" (understands what the caller/user wants); Connect is the contact-center infrastructure around it (routes calls, builds flows). They're commonly paired — Connect handles the call, Lex handles the bot logic inside it.

## Amazon Comprehend

- Serverless NLP: detects language, extracts key phrases/entities/places/people/brands/events, sentiment analysis, topic modeling/organizing a text collection
- Use cases: mining customer-interaction emails for what drives positive/negative sentiment, auto-grouping articles by topic
- **Comprehend Medical** — NLP over unstructured clinical text (physician notes, discharge summaries, test results) to detect Protected Health Information (PHI) via the `DetectPHI` API; commonly fed by S3 documents, Kinesis Data Firehose real-time data, or Transcribe output

> Exam-wording cue: "extract insights/sentiment/entities from free-form text" → Comprehend. "Extract text/data from a scanned document/form/table" → Textract (below) — Comprehend analyzes text that's already digitized, Textract is what digitizes it in the first place.

## Amazon SageMaker AI

- Fully managed service to build/train/deploy ML models in one place, instead of stitching together infra yourself

> Exam-wording cue: if a question needs a *custom* ML model (train your own on your own data) → SageMaker. If it just needs a pre-built capability (translate, transcribe, detect sentiment, recommend products) → the relevant purpose-built service above/below (Rekognition/Comprehend/Personalize/etc.) — no training required.

## Amazon Kendra

- ML-powered document search: extracts answers from text/PDF/HTML/PowerPoint/Word/FAQs using natural-language queries
- Incremental Learning — improves ranking from user interaction feedback; results can also be manually fine-tuned (importance, freshness, custom relevance)

## Amazon Personalize

- Fully managed real-time recommendation engine (same tech Amazon.com uses) — product recommendations/re-ranking, targeted marketing
- Integrates into existing websites/apps/SMS/email; no need to build/train/deploy your own models — implement in days, not months
- Use cases: retail, media & entertainment

## Amazon Textract

- Extracts text, handwriting, and structured data (forms, tables) from scanned documents/images using AI/ML
- Use cases: financial services (invoices, reports), healthcare (medical records, insurance claims), public sector (tax forms, IDs, passports)

## Notes

<!-- Your own notes go here. -->

Content sourced from slide deck, pages 541-600.

### From Netec live training (to review)

> Amazon Q named directly as AWS's generative-AI assistant, integrated as an assistant experience within the AWS ecosystem. Amazon Q Developer named as the developer-focused variant: an in-IDE chat assistant helping with code generation, code explanation, applying security best practices during implementation, and debugging assistance. Framed as accelerating how fast new applications can be built while helping bake in security from earlier development phases rather than bolting it on later.
>
> — *Netec S4, 1:21:19-1:21:38, 1:21:38-1:22:15, 1:22:15-1:22:48*
