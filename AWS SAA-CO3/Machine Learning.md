## Amazon Rekognition
- Find **objects, people, text, scenes** in images and videos using ML
- Facial analysis and facial search to do user verification, people counting
- Create a db of familar faces or compare against celebrities
- Use cases:
	- Labeling
	- Content moderation
	- Text detection
	- Face detection and analysis (gender, age range, emotions)
	- Face search and verification
	- Celebrity recognition
	- Pathing (sport game analysis)
### Content moderation
- Detect content that is inappropriate, unwanted, offensive (images and videos)
- used in social media, broadcast media, advertising, e commerce situations to create a safer user experience
- **Set a minimum Confidence Threshold for items that will be flagged**
- Flag sensitive content for manual review in Amazon Augmented AI (A2I)
- Help comply with regulations
- ![[Pasted image 20250803213312.png]]
## Amazon transcribe
- Automatically **convert speech to text**
- -Uses a **deep learning process called automatic speech recognition (ASR)** to convert speech to text quickly and accurately
- Automatically remove Personally Identifiable Information (PII) using Redaction
- Supports Automatic Language Identification for multi lingual audio
- Use case:
	- transcribe customer service calls
	- automate closed captioning and subtitling
	- generate metadata for media assets to create a fully searchable archive


## Amazon Polly
- Turn text into lifelike speech using deep learning
- Allowing you to create applications that talk

### Lexicon & SSML
- Customize the pronunciation of words with Pronunciation lexicons
	- Stylized words:  St3ph4ne => Stephane
	- Acronyms: AWS => Amazon Web Services
- Upload the lexicons and use them in the **SynthesizeSpeech** operation
- Generate speech from plain text or from documents marked up with Speech Synthesis Markup Language SSML - enables more customization
	- Emphasize specific words or phrases
	- using phonetic pronunciation
	- including breathing sounds, whispering
	- using newscaster speaking style


## Amazon Translate
- Natural and accurate language translation
- Amazon translate allows you to localize content - such as websites and applications - for internal users, to easily translate large volume of text

## Amazon Lex & connect
- Amazon Lex; same tech that powers Alexa
	- Automatic Speech Recognition (ASR) to convert speech to text
	- Natural language Understanding to recognize the intent of text, callers
	- helps build chatbots, call center bots
- Amazon connect: 
	- Receive calls, create contact flows, cloud based virtual contact cneter
	- Can integrate with other CRM system or AWS
	- No upfront payments, 80% cheaper than traditional contact center solutions
	- ![[Pasted image 20250804184041.png]]

## Amazon comprehend
- Natural Language Processing - NLP
- Fully managed and serverless service
- Uses ML to find insights and relationships in text
	- Language of the text
	- extracts key phrase, places, people, brands, events
	- understands how positive or negative the text is
	- Analyzes text using tokenization and parts of speech
	- automatically organizes a collection of text files by topic
- Sample use cases:
	- Analyze customer interactions (emails) to find what leads to a positive or negative experience
	- Create and groups article by topics that Comprehend will uncover
### Comprehend Medical
- detects and returns useful information in unstructured clinical test
	- test results
	- case notes
	- discharge summaries
	- physician's notes
- uses NLP to detect Protect Health Information (PHI) - DetectPHI API
- Store your documents in S3, analyze real time data with kinesis data firehose, or use Transcribe to transcribe patient narratives into text that can be analyzed by Amazon Comprehend Medical.
-

## Amazon SageMaker
- Fully managed service for developers/ data scientist to build ML models
- Typically difficult to do all the processes in one place + provision servers
- Machine learning process (simplified) predicting your exam score
- ![[Pasted image 20250804184646.png]]

## Amazon Kendra
- Fully managed Document search service powered by ML
- Extract answers from within a document (text,pdf,html,powerpoint,ms word)
- Natural language search capabilities
- Learn from user interactions/feedback to promote preferred results (incremental learning)
- Ability to manually fine tune search results (importance of data, freshness, custom)
- ![[Pasted image 20250804184839.png]]
## Amazon Personalize
- Fully managed ML service to build apps with real time personalized recommendations
- example: personalized product recommendations / re ranking,
	- User bought gardening tools, provide recommendations on the next one to buy
- Same tech used by amazon.com
- Integrates into existing websites, apps, SMS, emails
- Implement in days, not months (you dont need to build train and deploy ML solutions)

## Amazon Textract
- Automatically extract text, handwriting, data from any scanned docs using AI and  ML
- extract data from forms and tables
- read and process any type of docs
- use case:
	- financial services
	- healthcare
	- public sector


## Summary
- Rekognition: face detection,labeling,celeb recognition
- Transcribe: audio to text (subtitles)
- Polly: text to audio
- Translate: translations
- Lex: build conversational bots - chatbots
- Connect: cloud contact center
- Comprehend: natural language processing
- SageMaker: machine learning for every developer and data scientist
- Kendra: ML powered search engine
- Personalize: real time personalized recommendations
- textract: detect text and data in documents