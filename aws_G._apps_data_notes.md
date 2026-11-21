# G. APPS & DATA SERVICES (29 services, 5 categories)

Five categories, five questions:
- **App integration:** "How do the parts of my app talk to each other?"
- **Analytics:** "How do I turn raw data into answers?"
- **AI/ML:** "How do I add intelligence (text, speech, images, documents)?"
- **End-user computing:** "How do I give users apps, desktops, or browsers?"
- **Front-end, business & IoT:** "Niche families, each with one clear job."

---

## 18. APP INTEGRATION: "How do app parts talk?"

All six connect parts of an application. They differ by the **communication pattern**.

| Service            | Pattern                      | What it does                                                                                                                         | Why it exists separately                                                                    | Cost logic               |
| ------------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------- | ------------------------ |
| **SQS**            | **Queue**                    | Stores messages until a consumer **pulls** them. One message goes to **one consumer**. Standard and FIFO queues                      | **Decouples** parts so a slow or failed consumer doesn't break the sender                   | Per request (free tier)  |
| **SNS**            | **Pub/Sub broadcast**        | **Pushes** one message to **many subscribers** (SQS, Lambda, email, SMS, mobile push). Doesn't store messages                        | One event must notify many places at once (**fan-out**)                                     | Per publish and delivery |
| **EventBridge**    | **Event routing**            | Serverless **event bus**: events from AWS services, your apps and **SaaS partners**, rules route them to targets. Also **schedules** | Reacting to events with filtering rules and scheduled triggers, beyond simple notifications | Per event                |
| **Step Functions** | **Workflow**                 | Orchestrates **multi-step workflows** (visual state machine): sequence, branches, retries, error handling                            | Coordinating many Lambda functions or services in order is messy to code by hand            | Per state transition     |
| **API Gateway**    | **REST/HTTP API front door** | Create, publish, secure and monitor **REST, HTTP and WebSocket APIs**. Throttling, auth, caching. Often in front of Lambda           | Apps need a managed, secure public API                                                      | Per API call             |
| **AppSync**        | **GraphQL API**              | Managed **GraphQL** APIs with real-time data and offline sync for web/mobile apps                                                    | Front-ends that want exactly the data they ask for, in real time                            | Per query/operation      |

**Memory hook:** SQS = **hold** it. SNS = **shout** it. EventBridge = **route** it. Step Functions = **sequence** it. API Gateway = **door** (REST). AppSync = **door** (GraphQL).

**Common pattern:** SNS fan-out into several SQS queues. SNS broadcasts, each SQS queue buffers for its own consumer.

**Cue words**
- "decouple, queue, buffer, consumers pull, retain messages" → **SQS**
- "publish/subscribe, notify many, fan-out, email/SMS push" → **SNS**
- "event bus, react to events, SaaS events, schedule/cron" → **EventBridge**
- "workflow, orchestrate Lambda functions, retries, steps" → **Step Functions**
- "create REST API, throttling, front door for Lambda" → **API Gateway**
- "GraphQL, real-time sync" → **AppSync**

**Trap pairs**
- **SQS vs SNS:** pull and one consumer vs push and many subscribers.
- **SNS vs EventBridge:** simple fan-out notification vs rule-based event routing, SaaS sources and schedules.
- **Step Functions vs SQS:** an ordered *workflow* vs a *buffer*.
- **API Gateway vs AppSync:** REST/HTTP vs GraphQL.
- **Kinesis (Analytics) vs SQS:** real-time data *streams* for analysis vs a message queue.

**Check 1.** A web tier hands jobs to workers; messages must survive if workers are down. → **SQS**
**Check 2.** An order event must notify billing, email and inventory at once. → **SNS**
**Check 3.** Coordinate five Lambda functions in order with retries and error handling. → **Step Functions**

---

## 19. ANALYTICS: "Raw data to answers"

The six services sit at **different stages of a data pipeline**.

```
INGEST (real-time)  →  PREPARE  →  QUERY / SEARCH  →  VISUALIZE
Kinesis                Glue, EMR    Athena, OpenSearch   QuickSight
```
(Data usually lands in **S3** or **Redshift** in the middle. Redshift is the warehouse from the Data notes.)

| Service                | Stage               | What it does                                                                                                             | Cost logic                            |
| ---------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------- |
| **Kinesis**            | **Ingest**          | Collects and processes **real-time streaming data** (clickstreams, IoT, logs). Can deliver into S3, Redshift, OpenSearch | Per shard/throughput or data ingested |
| **Glue**               | **Prepare**         | **Serverless ETL** (extract, transform, load) plus a **Data Catalog** that describes your datasets                       | Per job run time                      |
| **EMR**                | **Prepare (heavy)** | Managed **big-data clusters** (Hadoop, Spark) for large custom processing                                                | Per cluster hour (or serverless)      |
| **Athena**             | **Query**           | **Serverless SQL directly on data in S3**. Nothing to load or manage                                                     | **Per data scanned**                  |
| **OpenSearch Service** | **Search**          | **Search and log analytics** with dashboards                                                                             | Per instance hour + storage           |
| **QuickSight**         | **Visualize**       | **BI dashboards** and charts, with ML insights                                                                           | Per user/session                      |

**Why similar ones exist**
- **Glue vs EMR:** both transform data. Glue is **serverless and simple** ETL. EMR is for **heavy, custom big-data frameworks** you control.
- **Athena vs Redshift:** Athena queries data **where it is (S3)**, pay per query. Redshift is a **warehouse you load** for heavy, repeated analytics.
- **Athena vs OpenSearch:** SQL over files vs **full-text search and log exploration**.

**Cue words**
- "real-time streaming, ingest clickstream data" → **Kinesis**
- "ETL, data catalog, prepare data" → **Glue**
- "Hadoop, Spark, big data processing" → **EMR**
- "SQL queries on S3, serverless, no loading" → **Athena**
- "search, log analytics, full-text search" → **OpenSearch**
- "dashboards, BI, visualize" → **QuickSight**

**Trap pairs**
- **Athena vs Redshift:** query in place vs warehouse.
- **Glue vs EMR:** serverless ETL vs customizable big data clusters.
- **Kinesis vs SQS:** streams of data for analytics vs messages between app parts.

**Check 1.** An analyst runs ad hoc SQL on CSV files in S3 without servers. → **Athena**
**Check 2.** Clickstream events must be captured in real time. → **Kinesis**
**Check 3.** Executives need interactive dashboards. → **QuickSight**

---

## 20. AI/ML: "Add intelligence"

Services differ by **the input they work on**. Two outliers: **SageMaker AI** (build your own) and **Q** (a generative AI assistant).

| Service | Input → Output | What it does | Cost logic |
|---|---|---|---|
| **Transcribe** | Speech → **text** | Speech-to-text | Per audio minute |
| **Polly** | Text → **speech** | Lifelike text-to-speech | Per characters |
| **Translate** | Text → **another language** | Neural machine translation | Per characters |
| **Comprehend** | Text → **insights** | NLP: sentiment, entities, key phrases, language | Per text unit |
| **Kendra** | Question → **answers from your documents** | Intelligent enterprise **search** with natural language | Per index/hour |
| **Lex** | Conversation → **chatbot** | Build **chatbots** and voice bots (same tech as Alexa) | Per request |
| **Rekognition** | Image/video → **objects, faces, text, moderation** | Image and video analysis | Per image/minute analyzed |
| **Textract** | Scanned documents → **text, forms, tables** | Goes beyond OCR to understand **structure** | Per page |
| **SageMaker AI** | Your data → **your own ML model** | **Build, train and deploy** custom ML models (for data scientists) | Per instance hours |
| **Q** | Natural language → **generative AI help** | Generative AI **assistant** (for business data and for developers) | Per user/month |

**Three levels (why they exist):**
1. **Ready-made AI services** (Transcribe, Polly, Translate, Comprehend, Kendra, Lex, Rekognition, Textract): no ML skills needed, just call them.
2. **Build your own** (SageMaker AI): when ready-made doesn't fit and you have data and data scientists.
3. **Generative AI assistant** (Q): ask in natural language, get answers or code help. (Bedrock, for foundation models via API, is not in your sheet but appears in the exam.)

**Opposites to keep straight:** Transcribe (speech to text) vs Polly (text to speech).

**AI concepts quick list**
- **AI** > **ML** > **Deep Learning** > **Generative AI**.
- **Training** = building the model. **Inference** = using it for predictions.
- **Foundation model / LLM** = large pre-trained model. **Prompt** = your input. **Hallucination** = confident but false output.
- **Supervised** (labeled data), **Unsupervised** (patterns), **Reinforcement** (rewards).
- **Responsible AI:** fairness, bias, explainability, privacy, transparency.

**Cue words**
- "speech to text, call transcripts" → **Transcribe**
- "text to speech, voice" → **Polly**
- "translate languages" → **Translate**
- "sentiment, entities, text analysis" → **Comprehend**
- "search company documents with natural language" → **Kendra**
- "chatbot, conversational interface" → **Lex**
- "images, video, faces, object detection, content moderation" → **Rekognition**
- "extract text, forms, tables from scanned documents" → **Textract**
- "build, train, deploy custom ML models" → **SageMaker AI**
- "generative AI assistant, answer questions about company data / help developers code" → **Q**

**Trap pairs**
- **Textract vs Rekognition:** documents and forms vs photos and video.
- **Comprehend vs Kendra:** *analyze* text vs *search* documents.
- **Lex vs Q:** build your own bot vs an assistant AWS already built.
- **SageMaker vs ready-made services:** custom model vs call an API.

**Check 1.** Extract fields and tables from scanned invoices. → **Textract**
**Check 2.** A call center wants a text version of recorded calls. → **Transcribe**
**Check 3.** A company wants to build and train its own custom ML model. → **SageMaker AI**

---

## 21. END-USER COMPUTING: "Stream to your users"

The three differ by **how much of the computer you stream**: a desktop, an app, or just a browser.

| Service | What is streamed | What it does | Cost logic |
|---|---|---|---|
| **WorkSpaces** | A **full virtual desktop** | **Persistent** Windows or Linux desktop for each user (DaaS) | Monthly or hourly per desktop |
| **AppStream 2.0** | A **single application** | Streams a desktop app to the user's browser **without a full desktop** | Per streaming instance hour |
| **WorkSpaces Secure Browser** | A **secure browser session** | Managed browser for reaching **internal websites and SaaS** securely; no data left on the device | Per user per month |

**Why three?** Different need levels. A full desktop is heavy. If users only need one app, stream the app. If they only need web access, stream a browser. Smaller scope, simpler, cheaper.

**Cue words**
- "virtual desktop, persistent desktop per employee" → **WorkSpaces**
- "stream a desktop application to a browser" → **AppStream 2.0**
- "secure browser access to internal web apps, no data on endpoint" → **WorkSpaces Secure Browser**

**Check 1.** Employees need a Windows desktop they can reach from any device. → **WorkSpaces**
**Check 2.** A company wants to deliver one legacy design application without giving users a full desktop. → **AppStream 2.0**

---

## 22. FRONT-END, BUSINESS & IoT: "Niche families"

Four unrelated services, each with **one job**. Learn each in one line.

| Service | Family | What it does | Cost logic |
|---|---|---|---|
| **Amplify** | **Front-end / mobile** | Build, deploy and **host full-stack web and mobile apps** (hosting, CI/CD, auth, API, storage) for front-end developers | Pay for the resources used |
| **Connect** | **Business apps** | **Cloud contact center** (phone and chat) with no upfront cost | Per minute of use |
| **SES** | **Business apps** | **Send emails at scale** (transactional, marketing) | Per email |
| **IoT Core** | **IoT** | **Connects IoT devices securely** to the cloud and routes their messages | Per message |

**Cue words**
- "host a web/mobile app, full-stack, front-end developers" → **Amplify**
- "contact center, customer calls" → **Connect**
- "send marketing or transactional emails to customers" → **SES**
- "connect devices, sensors, MQTT" → **IoT Core**

**Trap pairs**
- **SES vs SNS:** SES sends **emails to customers at scale**. SNS sends **notifications** (email/SMS) between systems.
- **Amplify vs Elastic Beanstalk:** front-end/full-stack web/mobile hosting vs deploying a backend application on managed EC2.

**Check 1.** A company wants to send newsletters to 1 million customers. → **SES**
**Check 2.** Thousands of sensors must send data securely to AWS. → **IoT Core**

---

## One-page recap

| Need | Pick |
|---|---|
| Buffer messages, decouple | SQS |
| Broadcast to many subscribers | SNS |
| Route events, SaaS events, schedules | EventBridge |
| Orchestrate a multi-step workflow | Step Functions |
| REST/HTTP/WebSocket API | API Gateway |
| GraphQL API | AppSync |
| Real-time streaming data | Kinesis |
| Serverless ETL + data catalog | Glue |
| Hadoop/Spark big data | EMR |
| SQL on S3, serverless | Athena |
| Search and log analytics | OpenSearch |
| BI dashboards | QuickSight |
| Speech to text / text to speech | Transcribe / Polly |
| Translation | Translate |
| Text analysis | Comprehend |
| Enterprise search | Kendra |
| Chatbot | Lex |
| Image/video analysis | Rekognition |
| Documents, forms, tables | Textract |
| Build custom ML | SageMaker AI |
| Generative AI assistant | Q |
| Virtual desktop | WorkSpaces |
| Stream an application | AppStream 2.0 |
| Secure browser access | WorkSpaces Secure Browser |
| Host web/mobile app | Amplify |
| Contact center | Connect |
| Send emails at scale | SES |
| Connect IoT devices | IoT Core |
