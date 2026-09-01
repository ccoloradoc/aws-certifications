# AWS AI/ML Services

No domain folder previously existed for this — the slide deck's own table of contents treats Machine Learning as a distinct top-level section, and none of these services fit the existing compute/storage/database/networking/security folders. Created while processing pages 541-600.

## From slides (pages 541-600)

### Amazon Rekognition

- Finds objects, people, text, and scenes in images/video using ML; facial analysis/search (build a "familiar faces" DB or compare against celebrities)
- Use cases: labeling, content moderation, text detection, face detection/analysis (age/gender/emotion), face search & verification, celebrity recognition, pathing (e.g. sports analytics)
- Content Moderation: flags inappropriate/offensive content in images/video with a configurable minimum confidence threshold; can route flagged content to Amazon Augmented AI (A2I) for human review

### Amazon Transcribe

- Speech-to-text via automatic speech recognition (ASR)
- Can redact PII automatically; supports automatic language identification for multi-lingual audio
- Use cases: transcribing support calls, closed captioning/subtitling, generating searchable metadata for media

### Amazon Polly

- Text-to-speech using deep learning
- Pronunciation Lexicons customize word pronunciation (stylized words, acronyms)
- SSML (Speech Synthesis Markup Language) adds emphasis, phonetic pronunciation, breathing/whispering effects, and a "Newscaster" speaking style

### Amazon Translate

- Natural, accurate language translation for localizing content (websites, apps) at scale

### Amazon Lex & Amazon Connect

- **Lex** — same tech as Alexa: ASR (speech→text) + NLU (intent recognition); builds chatbots/call-center bots
- **Connect** — cloud-based virtual contact center; receives calls, builds contact flows, integrates with CRMs; no upfront cost, ~80% cheaper than traditional contact center solutions

### Amazon Comprehend

- Serverless NLP: detects language, extracts key phrases/entities/places/people/brands/events, sentiment analysis, topic modeling/organizing a text collection
- Use cases: mining customer-interaction emails for what drives positive/negative sentiment, auto-grouping articles by topic
- **Comprehend Medical** — NLP over unstructured clinical text (physician notes, discharge summaries, test results) to detect Protected Health Information (PHI) via the `DetectPHI` API; commonly fed by S3 documents, Kinesis Data Firehose real-time data, or Transcribe output

### Amazon SageMaker AI

- Fully managed service to build/train/deploy ML models in one place, instead of stitching together infra yourself

### Amazon Kendra

- ML-powered document search: extracts answers from text/PDF/HTML/PowerPoint/Word/FAQs using natural-language queries
- Incremental Learning — improves ranking from user interaction feedback; results can also be manually fine-tuned (importance, freshness, custom relevance)

### Amazon Personalize

- Fully managed real-time recommendation engine (same tech Amazon.com uses) — product recommendations/re-ranking, targeted marketing
- Integrates into existing websites/apps/SMS/email; no need to build/train/deploy your own models — implement in days, not months
- Use cases: retail, media & entertainment

### Amazon Textract

- Extracts text, handwriting, and structured data (forms, tables) from scanned documents/images using AI/ML
- Use cases: financial services (invoices, reports), healthcare (medical records, insurance claims), public sector (tax forms, IDs, passports)

### One-line summary table (from slides)

- Rekognition — face detection, labeling, celebrity recognition
- Transcribe — audio → text
- Polly — text → audio
- Translate — language translation
- Lex — conversational chatbots
- Connect — cloud contact center
- Comprehend — NLP
- SageMaker — general-purpose ML platform
- Kendra — ML-powered search
- Personalize — real-time recommendations
- Textract — text/data extraction from documents

## Notes

<!-- Your own notes go here. -->
