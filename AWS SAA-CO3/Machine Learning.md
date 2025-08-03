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
- 