# 🧠 AI-Powered RSS → Twitter Automation Workflow

This n8n workflow automatically discovers trending articles, evaluates them with AI, and posts high-quality content directly to **Twitter (X)** — all without human intervention.

---

## 🚀 Overview

The workflow:
1. Pulls new articles from selected **RSS feeds**.  
2. Uses **Google Gemini AI** to score content relevance, credibility, and engagement potential.  
3. Filters and selects top-scoring articles.  
4. Generates concise, engaging Twitter posts using an **AI copywriter agent**.  
5. Automatically publishes tweets and logs each post to **Google Sheets** for tracking.

---

## ⚙️ Workflow Breakdown

### **1. RSS Feed Trigger**
Fetches fresh content from chosen sources (e.g., TechCrunch, Product Hunt, AI blogs).  
Example feed:  
https://rsshub.app/techcrunch/latest


### **2. Scoring AI**
**Model:** Google Gemini Chat (Simple Memory Model)  
**Purpose:** Analyze each article’s relevance and quality.  

**🧩 Prompt:**
```text
Evaluate the following article and provide a quality assessment.

Article title:
{{ $json.title }}

Article content:
{{ $json.content }}

Article link:
{{ $json.link }}

Now analyze the content and RETURN a JSON object with the following structure:

{
  "title": "...",
  "summary": "...",
  "relevance_score": 0-100,
  "credibility_score": 0-100,
  "engagement_score": 0-100,
  "overall_score": 0-100,
  "category": "AI | Automation | Tech | Data | Other",
  "verdict": "post" | "review" | "discard",
  "link": "..."
}

System Message:

You are an AI content analyst specializing in evaluating online articles for social-media sharing.
Score how interesting, relevant, and engaging each article would be for a tech and AI-focused audience.
Always respond in valid JSON format with concise, factual language.

3. Code in JavaScript

Parses the JSON output, extracts key fields, and prepares data for quality filtering.

4. Quality Filter

Passes only articles with:

overall_score >= 75


These move on to post generation.
Medium-quality items (50–74) are sent to “New for Review”; anything below 50 goes to Archive.

5. Post Generator AI Agent

Model: Google Gemini Chat
Purpose: Turn summaries into tweet-ready content.

🧩 Prompt:

You are given an article summary and metadata.

Create a post for Twitter following the rules below.

### Article Info
Title: {{$json.title}}
Summary: {{$json.summary}}
Link: {{$json.link}}

### Instructions
- Max 260 characters.
- Include 1 relevant emoji.
- Add 1–2 relevant hashtags (e.g., #AI #Automation).
- Include the article link at the end.

### Output Format
twitter_post: <tweet text here>


System Message:

You are an expert social media copywriter specialized in tech, AI, and automation topics.
Transform summarized articles into concise, engaging social media posts tailored for Twitter (X).
Maintain factual accuracy and a conversational tone suitable for a tech-savvy audience.

6. Code in JavaScript (Post Processor)

Cleans and formats the AI output:

const rawOutput = $json.output || "";
const twitterPart = rawOutput.split("twitter_post")[1]?.trim() || "";
const twitterPost = twitterPart.replace(/\s+/g, " ").trim();
return [{
  twitterPost,
  title: $json.title,
  link: $json.link,
  score: $json.overall_score,
  category: $json.category,
  summary: $json.summary,
  status: "Generated"
}];

7. Create Tweet

Automatically posts the final tweet via Twitter OAuth2.

8. Store Content (Google Sheets)

Appends metadata, tweet text, and posting status for recordkeeping.
