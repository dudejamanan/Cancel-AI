# CANCEL AI - System Design Document

## 1. Introduction

### 1.1 Purpose of This Document
This design document provides a comprehensive technical overview of the CANCEL AI system architecture, component interactions, data flow, and implementation strategies. It serves as a blueprint for developers, architects, and technical stakeholders to understand the internal workings of the platform.

### 1.2 High-Level System Overview
CANCEL AI is a multi-layered web application that leverages Large Language Models (LLMs) to simulate six distinct AI personas that evaluate user content from different perspectives. The system follows a client-server architecture with three primary layers:

- **Presentation Layer**: React-based frontend for user interaction
- **Application Layer**: FastAPI/Flask backend for business logic and orchestration
- **AI Processing Layer**: LLM-powered persona simulation and content analysis

The system processes user-submitted content through parallel AI evaluations, aggregates insights, generates risk assessments, and provides interactive feedback mechanisms.

## 2. System Architecture Overview

### 2.1 Layered Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                        │
│  React.js Frontend - User Interface & State Management      │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS/REST API
┌────────────────────────▼────────────────────────────────────┐
│                    APPLICATION LAYER                         │
│  FastAPI/Flask Backend - Business Logic & Orchestration     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Auth Service │  │Content Service│  │ Chat Service │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                   AI PROCESSING LAYER                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │Preprocessing │  │Persona Engine│  │Insight Agg.  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                         │                                    │
│              ┌──────────▼──────────┐                        │
│              │  LLM API / Ollama   │                        │
│              └─────────────────────┘                        │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                      DATA LAYER                              │
│  MongoDB / PostgreSQL - Persistent Storage                  │
│  Redis - Caching & Session Management                       │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Client-Server Model

The system follows a traditional client-server architecture:

- **Client (Frontend)**: Single Page Application (SPA) built with React.js
  - Handles user interactions and UI rendering
  - Manages local state and API communication
  - Provides real-time feedback and loading states

- **Server (Backend)**: RESTful API built with FastAPI or Flask
  - Processes business logic and orchestrates AI operations
  - Manages authentication and authorization
  - Interfaces with database and external AI services
  - Implements rate limiting and security measures

### 2.3 Data Flow Summary

1. **User Input**: User submits content via frontend interface
2. **Request Processing**: Backend validates and preprocesses content
3. **Parallel AI Evaluation**: Six personas analyze content simultaneously
4. **Insight Aggregation**: Results are combined and processed
5. **Response Generation**: Risk scores, emotion maps, and feedback are generated
6. **Client Update**: Frontend displays comprehensive analysis results
7. **Interactive Phase**: User can chat with personas or request rewrites
8. **Iteration**: User can edit content and trigger re-analysis


## 3. Component-Level Design

### 3.1 Frontend Design

#### 3.1.1 React Component Structure

```
src/
├── components/
│   ├── common/
│   │   ├── Button.jsx
│   │   ├── Input.jsx
│   │   ├── Modal.jsx
│   │   └── LoadingSpinner.jsx
│   ├── layout/
│   │   ├── Header.jsx
│   │   ├── Sidebar.jsx
│   │   └── Footer.jsx
│   ├── content/
│   │   ├── ContentInput.jsx
│   │   ├── PlatformSelector.jsx
│   │   └── ProfileSelector.jsx
│   ├── analysis/
│   │   ├── RiskScoreCard.jsx
│   │   ├── EmotionMap.jsx
│   │   ├── LegalStatusBadge.jsx
│   │   └── PersonaFeedback.jsx
│   ├── chat/
│   │   ├── PersonaChat.jsx
│   │   ├── ChatMessage.jsx
│   │   └── PersonaSelector.jsx
│   └── rewrite/
│       ├── VersionComparison.jsx
│       ├── SaferVersion.jsx
│       └── BolderVersion.jsx
├── pages/
│   ├── Dashboard.jsx
│   ├── AnalysisPage.jsx
│   ├── HistoryPage.jsx
│   └── ProfilePage.jsx
├── services/
│   ├── api.js
│   ├── auth.js
│   └── websocket.js
├── store/
│   ├── slices/
│   │   ├── authSlice.js
│   │   ├── contentSlice.js
│   │   └── analysisSlice.js
│   └── store.js
└── utils/
    ├── validators.js
    ├── formatters.js
    └── constants.js
```

#### 3.1.2 Dashboard Layout

The main dashboard consists of:

- **Header**: Navigation, user profile, quick actions
- **Content Input Panel**: Text area, media upload, platform/profile selectors
- **Analysis Results Panel**: Risk score, emotion map, legal status
- **Persona Feedback Section**: Tabbed interface showing each persona's feedback
- **Action Bar**: Buttons for chat, rewrite, export, re-analyze
- **History Sidebar**: Recent analyses with quick access

#### 3.1.3 Persona Interaction Panel

Interactive chat interface featuring:

- **Persona Avatar & Name**: Visual identification of selected persona
- **Chat History**: Scrollable message thread with user/AI distinction
- **Input Field**: Text input with send button
- **Context Indicator**: Shows which content is being discussed
- **Quick Questions**: Suggested follow-up questions
- **Exit/Switch**: Option to return to main view or switch personas

#### 3.1.4 State Management Approach

Using Redux Toolkit for centralized state management:

```javascript
// Store Structure
{
  auth: {
    user: {...},
    token: "...",
    isAuthenticated: boolean
  },
  content: {
    currentContent: "...",
    platform: "twitter",
    profile: {...},
    isDraft: boolean
  },
  analysis: {
    isLoading: boolean,
    results: {
      riskScore: 45,
      legalStatus: "review",
      emotionMap: {...},
      personaFeedback: [...]
    },
    saferVersion: "...",
    bolderVersion: "..."
  },
  chat: {
    activePersona: "legal",
    messages: [...],
    isTyping: boolean
  }
}
```

### 3.2 Backend Design

#### 3.2.1 FastAPI/Flask Structure

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                 # Application entry point
│   ├── config.py               # Configuration management
│   ├── dependencies.py         # Dependency injection
│   ├── api/
│   │   ├── __init__.py
│   │   ├── auth.py            # Authentication endpoints
│   │   ├── content.py         # Content submission endpoints
│   │   ├── analysis.py        # Analysis endpoints
│   │   ├── chat.py            # Persona chat endpoints
│   │   └── user.py            # User management endpoints
│   ├── services/
│   │   ├── __init__.py
│   │   ├── auth_service.py
│   │   ├── content_service.py
│   │   ├── analysis_service.py
│   │   ├── persona_service.py
│   │   └── rewrite_service.py
│   ├── ai/
│   │   ├── __init__.py
│   │   ├── preprocessor.py    # Content preprocessing
│   │   ├── persona_engine.py  # Persona simulation
│   │   ├── aggregator.py      # Insight aggregation
│   │   └── prompts.py         # Prompt templates
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── content.py
│   │   └── analysis.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── content_schema.py
│   │   └── analysis_schema.py
│   └── utils/
│       ├── __init__.py
│       ├── security.py
│       ├── validators.py
│       └── helpers.py
├── tests/
├── requirements.txt
└── .env
```

#### 3.2.2 Route Definitions

```python
# Authentication Routes
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/refresh

# Content Analysis Routes
POST   /api/analyze              # Submit content for analysis
GET    /api/analyze/{id}         # Get analysis results
POST   /api/analyze/{id}/reanalyze

# Persona Chat Routes
POST   /api/chat/{analysis_id}   # Send message to persona
GET    /api/chat/{analysis_id}   # Get chat history

# Rewrite Routes
POST   /api/rewrite/safer/{analysis_id}
POST   /api/rewrite/bolder/{analysis_id}

# User Routes
GET    /api/user/profile
PUT    /api/user/profile
GET    /api/user/history
DELETE /api/user/history/{id}
```

#### 3.2.3 Request Lifecycle

1. **Request Reception**: FastAPI receives HTTP request
2. **Authentication**: JWT token validation (if required)
3. **Validation**: Pydantic schema validation of request body
4. **Rate Limiting**: Check user's request quota
5. **Service Layer**: Business logic execution
6. **AI Processing**: Invoke AI layer for content analysis
7. **Database Operations**: Store/retrieve data as needed
8. **Response Formation**: Format response according to schema
9. **Response Delivery**: Return JSON response to client

#### 3.2.4 API Endpoints Design

**Example: Content Analysis Endpoint**

```python
@router.post("/analyze", response_model=AnalysisResponse)
async def analyze_content(
    content_request: ContentRequest,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    # Validate content
    validate_content(content_request.text)
    
    # Preprocess content
    preprocessed = await preprocessor.process(content_request.text)
    
    # Run parallel persona analysis
    persona_results = await persona_engine.analyze_parallel(
        content=preprocessed,
        platform=content_request.platform,
        profile=content_request.profile
    )
    
    # Aggregate insights
    insights = aggregator.aggregate(persona_results)
    
    # Calculate risk score
    risk_score = calculate_risk_score(persona_results)
    
    # Generate emotion map
    emotion_map = generate_emotion_map(persona_results)
    
    # Save to database
    analysis = save_analysis(db, current_user.id, insights)
    
    return AnalysisResponse(
        id=analysis.id,
        risk_score=risk_score,
        legal_status=insights.legal_status,
        emotion_map=emotion_map,
        persona_feedback=insights.feedback
    )
```


### 3.3 AI Processing Layer

#### 3.3.1 Preprocessing Module

The preprocessing module prepares content for AI analysis:

```python
class ContentPreprocessor:
    def __init__(self):
        self.sentiment_analyzer = SentimentAnalyzer()
        self.topic_detector = TopicDetector()
        
    async def process(self, content: str) -> PreprocessedContent:
        # Clean and normalize text
        cleaned_text = self.clean_text(content)
        
        # Sentiment analysis
        sentiment = self.sentiment_analyzer.analyze(cleaned_text)
        
        # Topic detection
        topics = self.topic_detector.detect(cleaned_text)
        
        # Keyword extraction
        keywords = self.extract_keywords(cleaned_text)
        
        # Trigger word detection
        triggers = self.detect_triggers(cleaned_text)
        
        return PreprocessedContent(
            original=content,
            cleaned=cleaned_text,
            sentiment=sentiment,
            topics=topics,
            keywords=keywords,
            triggers=triggers,
            metadata={
                "word_count": len(cleaned_text.split()),
                "char_count": len(cleaned_text),
                "readability_score": self.calculate_readability(cleaned_text)
            }
        )
    
    def clean_text(self, text: str) -> str:
        # Remove extra whitespace
        text = re.sub(r'\s+', ' ', text).strip()
        # Fix encoding issues
        text = text.encode('utf-8', 'ignore').decode('utf-8')
        # Normalize unicode characters
        text = unicodedata.normalize('NFKC', text)
        return text
```

#### 3.3.2 Persona Prompt Engineering Structure

Each persona has a carefully crafted system prompt:

```python
PERSONA_PROMPTS = {
    "legal": """
        You are a Legal Advisor AI persona evaluating content for legal risks.
        Focus on: copyright infringement, defamation, false claims, privacy violations,
        trademark issues, and regulatory compliance.
        
        Analyze the content and provide:
        1. Overall legal risk level (low/medium/high)
        2. Specific legal concerns with references to content
        3. Recommendations for mitigation
        4. Relevant legal considerations
        
        Be precise but not alarmist. Provide actionable guidance.
    """,
    
    "genz": """
        You are a Gen-Z Critic AI persona evaluating content for cultural relevance.
        Focus on: current trends, meme culture, generational appeal, authenticity,
        cringe factors, and viral potential.
        
        Analyze the content and provide:
        1. Cultural relevance score
        2. Trend alignment assessment
        3. Authenticity evaluation
        4. Suggestions for improvement
        
        Use Gen-Z language and perspective. Be honest but constructive.
    """,
    
    "ethics": """
        You are an Ethics Guardian AI persona evaluating moral implications.
        Focus on: social responsibility, inclusivity, potential harm, bias,
        stereotypes, and ethical considerations.
        
        Analyze the content and provide:
        1. Ethical risk assessment
        2. Potential harm identification
        3. Bias and stereotype detection
        4. Recommendations for ethical improvement
        
        Be thoughtful and consider diverse perspectives.
    """,
    
    # Similar prompts for ideological, brand, and creativity personas
}
```

#### 3.3.3 Parallel Execution of AI Personas

Using Python's asyncio for concurrent persona evaluation:

```python
class PersonaEngine:
    def __init__(self, llm_client):
        self.llm_client = llm_client
        self.personas = ["legal", "genz", "ethics", "ideological", "brand", "creativity"]
    
    async def analyze_parallel(
        self, 
        content: PreprocessedContent,
        platform: str,
        profile: dict
    ) -> List[PersonaResult]:
        # Create tasks for parallel execution
        tasks = [
            self.analyze_with_persona(persona, content, platform, profile)
            for persona in self.personas
        ]
        
        # Execute all personas concurrently
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        # Handle any failures gracefully
        valid_results = [r for r in results if not isinstance(r, Exception)]
        
        return valid_results
    
    async def analyze_with_persona(
        self,
        persona: str,
        content: PreprocessedContent,
        platform: str,
        profile: dict
    ) -> PersonaResult:
        # Build persona-specific prompt
        prompt = self.build_prompt(persona, content, platform, profile)
        
        # Call LLM API
        response = await self.llm_client.generate(
            system_prompt=PERSONA_PROMPTS[persona],
            user_prompt=prompt,
            temperature=0.7,
            max_tokens=1000
        )
        
        # Parse structured response
        parsed = self.parse_response(response, persona)
        
        return PersonaResult(
            persona=persona,
            assessment=parsed.assessment,
            concerns=parsed.concerns,
            recommendations=parsed.recommendations,
            severity=parsed.severity,
            raw_response=response
        )
```

#### 3.3.4 Insight Aggregation Logic

Combining persona outputs into unified insights:

```python
class InsightAggregator:
    def aggregate(self, persona_results: List[PersonaResult]) -> AggregatedInsights:
        # Extract all concerns
        all_concerns = []
        for result in persona_results:
            all_concerns.extend(result.concerns)
        
        # Determine legal status
        legal_result = next(r for r in persona_results if r.persona == "legal")
        legal_status = self.determine_legal_status(legal_result)
        
        # Aggregate recommendations
        recommendations = self.merge_recommendations(persona_results)
        
        # Identify critical issues
        critical_issues = [
            c for c in all_concerns 
            if c.severity in ["high", "critical"]
        ]
        
        return AggregatedInsights(
            legal_status=legal_status,
            persona_feedback=persona_results,
            all_concerns=all_concerns,
            critical_issues=critical_issues,
            recommendations=recommendations
        )
```

#### 3.3.5 Risk Score Calculation Logic

Weighted algorithm for overall risk assessment:

```python
def calculate_risk_score(persona_results: List[PersonaResult]) -> int:
    # Define persona weights
    weights = {
        "legal": 0.30,      # Highest weight - legal issues are critical
        "ethics": 0.20,     # High weight - ethical concerns matter
        "brand": 0.20,      # High weight - reputation protection
        "ideological": 0.15, # Medium weight - political sensitivity
        "genz": 0.10,       # Lower weight - cultural relevance
        "creativity": 0.05  # Lowest weight - creative impact
    }
    
    # Calculate weighted score
    total_score = 0
    for result in persona_results:
        persona_score = result.severity_to_score()  # Convert severity to 0-100
        weight = weights.get(result.persona, 0.1)
        total_score += persona_score * weight
    
    # Normalize to 0-100 range
    final_score = min(100, max(0, int(total_score)))
    
    return final_score
```

#### 3.3.6 Emotion Map Generation Logic

Predicting audience emotional responses:

```python
def generate_emotion_map(persona_results: List[PersonaResult]) -> EmotionMap:
    # Initialize emotion categories
    emotions = {
        "positive": {"joy": 0, "excitement": 0, "inspiration": 0, "trust": 0},
        "neutral": {"curiosity": 0, "indifference": 0, "confusion": 0},
        "negative": {"anger": 0, "offense": 0, "disappointment": 0, "fear": 0}
    }
    
    # Extract emotion predictions from each persona
    for result in persona_results:
        if hasattr(result, 'emotion_predictions'):
            for emotion, intensity in result.emotion_predictions.items():
                category = get_emotion_category(emotion)
                emotions[category][emotion] += intensity
    
    # Normalize to percentages
    total = sum(sum(cat.values()) for cat in emotions.values())
    if total > 0:
        for category in emotions:
            for emotion in emotions[category]:
                emotions[category][emotion] = round(
                    (emotions[category][emotion] / total) * 100, 1
                )
    
    # Calculate category totals
    category_totals = {
        cat: sum(emotions[cat].values()) 
        for cat in emotions
    }
    
    return EmotionMap(
        emotions=emotions,
        category_totals=category_totals,
        dominant_emotion=get_dominant_emotion(emotions)
    )
```

### 3.4 Interactive Persona Module

#### 3.4.1 Conversation Flow

```python
class PersonaChatService:
    def __init__(self, llm_client, db):
        self.llm_client = llm_client
        self.db = db
    
    async def send_message(
        self,
        analysis_id: str,
        persona: str,
        user_message: str,
        user_id: str
    ) -> ChatResponse:
        # Retrieve original analysis context
        analysis = self.db.get_analysis(analysis_id)
        
        # Get conversation history
        history = self.db.get_chat_history(analysis_id, persona)
        
        # Build context-aware prompt
        context_prompt = self.build_chat_context(
            analysis=analysis,
            persona=persona,
            history=history,
            user_message=user_message
        )
        
        # Generate persona response
        response = await self.llm_client.generate(
            system_prompt=PERSONA_PROMPTS[persona],
            user_prompt=context_prompt,
            temperature=0.8,
            max_tokens=500
        )
        
        # Save to chat history
        self.db.save_chat_message(
            analysis_id=analysis_id,
            persona=persona,
            user_message=user_message,
            ai_response=response
        )
        
        return ChatResponse(
            persona=persona,
            message=response,
            timestamp=datetime.now()
        )
```

#### 3.4.2 Context Handling

Maintaining conversation context across multiple turns:

```python
def build_chat_context(
    self,
    analysis: Analysis,
    persona: str,
    history: List[ChatMessage],
    user_message: str
) -> str:
    context = f"""
    Original Content: {analysis.content}
    Your Previous Assessment: {analysis.get_persona_feedback(persona)}
    
    Conversation History:
    """
    
    # Add recent conversation history (last 5 messages)
    for msg in history[-5:]:
        context += f"\nUser: {msg.user_message}"
        context += f"\nYou: {msg.ai_response}"
    
    context += f"\n\nUser's New Question: {user_message}"
    context += "\n\nProvide a helpful, persona-appropriate response:"
    
    return context
```

#### 3.4.3 Follow-up Question Routing

Intelligent routing based on question type:

```python
def route_followup_question(self, user_message: str, persona: str) -> str:
    # Detect question intent
    if self.is_clarification_request(user_message):
        return "clarification"
    elif self.is_example_request(user_message):
        return "example"
    elif self.is_alternative_request(user_message):
        return "alternative"
    else:
        return "general"
```

### 3.5 Improvement Engine

#### 3.5.1 Safer Rewrite Generation

```python
async def generate_safer_version(
    self,
    content: str,
    analysis: Analysis
) -> str:
    # Identify high-risk elements
    risks = [c for c in analysis.concerns if c.severity in ["high", "critical"]]
    
    prompt = f"""
    Original Content: {content}
    
    Identified Risks:
    {self.format_risks(risks)}
    
    Generate a safer version that:
    1. Maintains the core message and intent
    2. Removes or mitigates identified risks
    3. Keeps the tone and style similar
    4. Reduces overall risk score by at least 20 points
    
    Safer Version:
    """
    
    safer_version = await self.llm_client.generate(
        system_prompt="You are a content safety expert.",
        user_prompt=prompt,
        temperature=0.6
    )
    
    return safer_version
```

#### 3.5.2 Bolder Rewrite Generation

```python
async def generate_bolder_version(
    self,
    content: str,
    analysis: Analysis
) -> str:
    prompt = f"""
    Original Content: {content}
    Current Risk Score: {analysis.risk_score}
    
    Generate a bolder version that:
    1. Amplifies the core message
    2. Increases engagement potential
    3. Takes calculated creative risks
    4. Maintains brand appropriateness
    5. Stays within legal boundaries
    
    Bolder Version:
    """
    
    bolder_version = await self.llm_client.generate(
        system_prompt="You are a creative content strategist.",
        user_prompt=prompt,
        temperature=0.9
    )
    
    return bolder_version
```

#### 3.5.3 Version Comparison Logic

```python
def compare_versions(
    self,
    original: str,
    safer: str,
    bolder: str
) -> VersionComparison:
    return VersionComparison(
        original={
            "text": original,
            "changes": []
        },
        safer={
            "text": safer,
            "changes": self.highlight_differences(original, safer),
            "risk_reduction": "Estimated 20-30 point reduction"
        },
        bolder={
            "text": bolder,
            "changes": self.highlight_differences(original, bolder),
            "risk_increase": "Estimated 10-15 point increase"
        }
    )
```


## 4. Database Design

### 4.1 Schema Overview

The system uses a document-based approach (MongoDB) or relational approach (PostgreSQL) depending on deployment preference.

#### MongoDB Schema (Recommended for MVP)

**Advantages**: Flexible schema, easy to iterate, handles nested documents well

### 4.2 User Collection

```javascript
{
  _id: ObjectId,
  email: String (unique, indexed),
  password_hash: String,
  name: String,
  created_at: ISODate,
  updated_at: ISODate,
  email_verified: Boolean,
  subscription_tier: String, // "free", "pro", "enterprise"
  preferences: {
    default_platform: String,
    default_profile: Object,
    notification_settings: Object
  },
  usage_stats: {
    total_analyses: Number,
    analyses_this_month: Number,
    last_analysis_date: ISODate
  },
  api_quota: {
    daily_limit: Number,
    daily_used: Number,
    reset_date: ISODate
  }
}
```

**Indexes**:
- `email` (unique)
- `created_at` (for sorting)

### 4.3 Content History Collection

```javascript
{
  _id: ObjectId,
  user_id: ObjectId (foreign key, indexed),
  content: {
    original_text: String,
    media_url: String (optional),
    platform: String,
    creator_profile: Object
  },
  preprocessing: {
    cleaned_text: String,
    sentiment: Object,
    topics: Array,
    keywords: Array,
    triggers: Array,
    metadata: Object
  },
  analysis: {
    risk_score: Number,
    legal_status: String, // "clear", "review", "high_risk"
    emotion_map: {
      positive: Object,
      neutral: Object,
      negative: Object,
      category_totals: Object
    },
    persona_feedback: [
      {
        persona: String,
        assessment: String,
        concerns: Array,
        recommendations: Array,
        severity: String,
        raw_response: String
      }
    ]
  },
  rewrites: {
    safer_version: String,
    bolder_version: String,
    generated_at: ISODate
  },
  status: String, // "processing", "completed", "failed"
  created_at: ISODate,
  updated_at: ISODate,
  expires_at: ISODate // Auto-delete after 30 days
}
```

**Indexes**:
- `user_id` (for user queries)
- `created_at` (for sorting)
- `expires_at` (for TTL cleanup)

### 4.4 Session Storage

```javascript
{
  _id: ObjectId,
  user_id: ObjectId,
  token: String (indexed),
  refresh_token: String,
  created_at: ISODate,
  expires_at: ISODate,
  ip_address: String,
  user_agent: String
}
```

**Indexes**:
- `token` (unique, for fast lookup)
- `expires_at` (for TTL cleanup)

### 4.5 Persona Interaction Logs

```javascript
{
  _id: ObjectId,
  analysis_id: ObjectId (foreign key, indexed),
  user_id: ObjectId (indexed),
  persona: String,
  messages: [
    {
      role: String, // "user" or "assistant"
      content: String,
      timestamp: ISODate
    }
  ],
  created_at: ISODate,
  updated_at: ISODate
}
```

**Indexes**:
- `analysis_id` (for retrieving chat history)
- `user_id` (for user queries)

### 4.6 PostgreSQL Alternative Schema

For teams preferring relational databases:

```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(255),
    subscription_tier VARCHAR(50) DEFAULT 'free',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Content analyses table
CREATE TABLE content_analyses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    original_text TEXT NOT NULL,
    platform VARCHAR(50),
    risk_score INTEGER,
    legal_status VARCHAR(50),
    emotion_map JSONB,
    status VARCHAR(50) DEFAULT 'processing',
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP
);

-- Persona feedback table
CREATE TABLE persona_feedback (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    analysis_id UUID REFERENCES content_analyses(id) ON DELETE CASCADE,
    persona VARCHAR(50) NOT NULL,
    assessment TEXT,
    concerns JSONB,
    recommendations JSONB,
    severity VARCHAR(50),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Chat messages table
CREATE TABLE chat_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    analysis_id UUID REFERENCES content_analyses(id) ON DELETE CASCADE,
    persona VARCHAR(50) NOT NULL,
    role VARCHAR(20) NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

## 5. Deployment Architecture

### 5.1 Frontend Hosting

**Platform**: Vercel (Recommended)

**Configuration**:
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "react",
  "env": {
    "REACT_APP_API_URL": "@api_url"
  }
}
```

**Advantages**:
- Automatic deployments from Git
- Global CDN distribution
- Zero-config SSL
- Excellent React/Next.js support
- Free tier available

**Alternative**: Netlify, AWS Amplify, GitHub Pages

### 5.2 Backend Hosting

**Platform**: Render / Railway / AWS (Recommended: Render for MVP)

**Render Configuration** (`render.yaml`):
```yaml
services:
  - type: web
    name: cancel-ai-backend
    env: python
    buildCommand: pip install -r requirements.txt
    startCommand: uvicorn app.main:app --host 0.0.0.0 --port $PORT
    envVars:
      - key: DATABASE_URL
        sync: false
      - key: OPENAI_API_KEY
        sync: false
      - key: JWT_SECRET_KEY
        generateValue: true
      - key: ENVIRONMENT
        value: production
```

**Advantages**:
- Easy deployment from Git
- Automatic HTTPS
- Built-in monitoring
- Affordable pricing
- Good Python support

**Alternative**: AWS EC2, Google Cloud Run, Heroku, DigitalOcean

### 5.3 AI API Integration

**Primary Option**: OpenAI GPT-4 or Anthropic Claude

```python
# OpenAI Integration
from openai import AsyncOpenAI

client = AsyncOpenAI(api_key=os.getenv("OPENAI_API_KEY"))

async def generate_response(system_prompt: str, user_prompt: str):
    response = await client.chat.completions.create(
        model="gpt-4-turbo-preview",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt}
        ],
        temperature=0.7,
        max_tokens=1000
    )
    return response.choices[0].message.content
```

**Cost Optimization**:
- Use GPT-3.5-turbo for non-critical personas
- Implement aggressive caching
- Batch requests when possible
- Monitor token usage

### 5.4 Optional Open-Source LLM via Ollama

**For Cost Reduction**: Run local LLMs using Ollama

**Setup**:
```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull model
ollama pull llama2:13b

# Run server
ollama serve
```

**Integration**:
```python
import requests

def generate_with_ollama(prompt: str):
    response = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model": "llama2:13b",
            "prompt": prompt,
            "stream": False
        }
    )
    return response.json()["response"]
```

**Trade-offs**:
- Pros: No API costs, data privacy, unlimited usage
- Cons: Requires GPU server, lower quality, slower inference

### 5.5 Scaling Strategy

#### Horizontal Scaling

```
                    ┌─────────────┐
                    │Load Balancer│
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │Backend 1│       │Backend 2│       │Backend 3│
   └────┬────┘       └────┬────┘       └────┬────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                    ┌──────▼──────┐
                    │  Database   │
                    │   Cluster   │
                    └─────────────┘
```

**Implementation**:
- Use Docker containers for backend services
- Deploy multiple instances behind load balancer
- Use Redis for shared session storage
- Implement database read replicas for scaling reads

#### Auto-Scaling Configuration

```yaml
# Kubernetes example
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: cancel-ai-backend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cancel-ai-backend
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## 6. Security Design

### 6.1 API Key Protection

**Environment Variables**:
```python
# config.py
from pydantic import BaseSettings

class Settings(BaseSettings):
    OPENAI_API_KEY: str
    JWT_SECRET_KEY: str
    DATABASE_URL: str
    
    class Config:
        env_file = ".env"
        case_sensitive = True

settings = Settings()
```

**Never commit `.env` files**:
```gitignore
.env
.env.local
.env.production
```

### 6.2 Input Validation

**Pydantic Schemas**:
```python
from pydantic import BaseModel, validator, Field

class ContentRequest(BaseModel):
    text: str = Field(..., min_length=1, max_length=10000)
    platform: str = Field(..., regex="^(twitter|instagram|linkedin|tiktok|youtube|blog)$")
    
    @validator('text')
    def validate_text(cls, v):
        # Remove null bytes
        v = v.replace('\x00', '')
        # Check for suspicious patterns
        if re.search(r'<script|javascript:|onerror=', v, re.IGNORECASE):
            raise ValueError('Potentially malicious content detected')
        return v
```

**SQL Injection Prevention**:
```python
# Use parameterized queries
cursor.execute(
    "SELECT * FROM users WHERE email = %s",
    (email,)  # Parameters passed separately
)
```

### 6.3 Rate Limiting

**Implementation using slowapi**:
```python
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.post("/api/analyze")
@limiter.limit("10/minute")  # 10 requests per minute
async def analyze_content(request: Request):
    # Endpoint logic
    pass
```

**Tiered Rate Limits**:
```python
def get_rate_limit(user: User) -> str:
    limits = {
        "free": "10/day",
        "pro": "100/day",
        "enterprise": "1000/day"
    }
    return limits.get(user.subscription_tier, "10/day")
```

### 6.4 Secure Communication (HTTPS)

**Force HTTPS in Production**:
```python
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware

if settings.ENVIRONMENT == "production":
    app.add_middleware(HTTPSRedirectMiddleware)
```

**CORS Configuration**:
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://cancelai.com",
        "https://www.cancelai.com"
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
)
```

### 6.5 Authentication & Authorization

**JWT Token Generation**:
```python
from jose import jwt
from datetime import datetime, timedelta

def create_access_token(user_id: str) -> str:
    expire = datetime.utcnow() + timedelta(hours=24)
    payload = {
        "sub": user_id,
        "exp": expire,
        "iat": datetime.utcnow()
    }
    return jwt.encode(payload, settings.JWT_SECRET_KEY, algorithm="HS256")
```

**Token Verification**:
```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer

security = HTTPBearer()

async def get_current_user(token: str = Depends(security)):
    try:
        payload = jwt.decode(
            token.credentials,
            settings.JWT_SECRET_KEY,
            algorithms=["HS256"]
        )
        user_id = payload.get("sub")
        if user_id is None:
            raise HTTPException(status_code=401)
        return get_user_by_id(user_id)
    except jwt.JWTError:
        raise HTTPException(status_code=401)
```

