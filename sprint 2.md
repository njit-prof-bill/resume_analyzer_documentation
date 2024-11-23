# Sprint 2: AI Integration, Feedback Generation, and Final Testing

## **2.1 AI Integration for NLP Analysis**

### **Task 17**: Research and Select a Free AI NLP API (e.g., Hugging Face, OpenAI)

- **Objective**:
  - Identify and integrate a suitable NLP API to analyze resume and job description text for generating fit scores and feedback.

- **Technical Details**:
  1. **Research Criteria**:
     - Ensure the API supports text-based NLP tasks such as summarization, keyword extraction, or text comparison.
     - Examples of free APIs:
       - Hugging Face Transformers API (e.g., BERT, DistilBERT models).
       - OpenAI GPT-4 API (free tiers or sandbox environments).
  2. **Integration Preparation**:
     - Obtain an API key and set up authentication:
       - Hugging Face: Requires a personal access token from their developer portal.
       - OpenAI: Requires an API key from their dashboard.
     - Store API credentials securely in environment variables:
       ```bash
       export HUGGINGFACE_API_KEY="your-huggingface-key"
       export OPENAI_API_KEY="your-openai-key"
       ```
  3. **Output Format**:
     - Ensure the API supports JSON-based responses that include keyword matching or text similarity scores.

- **Unit Tests**:
  - Mock API responses during testing to avoid exceeding free-tier usage limits.
  - Validate authentication by simulating requests with invalid or missing API keys.

---

### **Task 18**: Develop a FastAPI Endpoint to Send Resume and Job Description to the NLP API

- **Objective**:
  - Build a backend endpoint to connect with the NLP API and facilitate communication between the frontend and AI service.

- **Technical Details**:
  1. **Endpoint Definition**:
     - **URL**: `POST /api/analyze`
     - **Request Payload**:
       ```json
       {
         "resume_text": "Detailed resume content goes here...",
         "job_description": "Job description content goes here..."
       }
       ```
     - **Validation**:
       - Ensure both `resume_text` and `job_description` fields are present.
       - Enforce a character limit (e.g., max 10,000 characters per field).
  2. **Backend Logic**:
     - Parse the incoming request and validate the payload.
     - Construct a request to the chosen NLP API:
       - For Hugging Face:
         ```python
         import requests

         def analyze_text(resume_text, job_description):
             headers = {"Authorization": f"Bearer {HUGGINGFACE_API_KEY}"}
             data = {"inputs": {"resume": resume_text, "job_description": job_description}}
             response = requests.post("https://api-inference.huggingface.co/models/your-model", headers=headers, json=data)
             return response.json()
         ```
  3. **Response Format**:
     - **Success** (`200 OK`):
       ```json
       {
         "fit_score": 85,
         "feedback": [
           "Add skills related to project management.",
           "Improve your summary section to include specific achievements."
         ]
       }
       ```
     - **Error** (`400 Bad Request` or `500 Internal Server Error`):
       ```json
       {
         "error": "Unable to process the request. Please try again later."
       }
       ```

- **Unit Tests**:
  - Test with valid and invalid payloads (e.g., missing fields, oversized text).
  - Mock API responses to simulate success and failure scenarios.

---

### **Task 19**: Create Standardized Input/Output Data Structures to Communicate with the AI API

- **Objective**:
  - Define and enforce a standardized structure for inputs and outputs to ensure consistency across the system.

- **Technical Details**:
  1. **Input Data Structure**:
     - JSON format:
       ```json
       {
         "resume_text": "The content of the user's resume...",
         "job_description": "The content of the job description..."
       }
       ```
     - Validation Rules:
       - Both `resume_text` and `job_description` must be strings.
       - Character limit: 10,000 characters per field.
  2. **Output Data Structure**:
     - JSON format:
       ```json
       {
         "fit_score": 0-100,
         "feedback": [
           "String feedback item 1",
           "String feedback item 2"
         ]
       }
       ```
     - `fit_score`: Numeric value representing resume-job fit percentage.
     - `feedback`: Array of strings with suggestions for improvement.
  3. **Error Handling**:
     - Standardize error responses for invalid inputs or failed NLP API calls:
       ```json
       {
         "error": "Invalid input format or data."
       }
       ```

- **Unit Tests**:
  - Validate that requests and responses conform to the defined structure.
  - Simulate various edge cases (e.g., empty fields, unexpected data types).

---

### **Task 20**: Implement Logic to Parse the AI API Response and Generate Fit Scores and Feedback

- **Objective**:
  - Process the raw API response and extract actionable insights to be sent to the frontend.

- **Technical Details**:
  1. **Raw Response from NLP API**:
     - Example:
       ```json
       {
         "results": [
           {
             "fit_score": 0.85,
             "keywords": ["project management", "team leadership"],
             "feedback": [
               "Add skills related to project management.",
               "Improve your summary section to include specific achievements."
             ]
           }
         ]
       }
       ```
  2. **Processing Logic**:
     - Extract the `fit_score` and convert it to a percentage (e.g., `0.85 -> 85%`).
     - Format the `feedback` array for clear display on the frontend.
     - Example Output:
       ```json
       {
         "fit_score": 85,
         "feedback": [
           "Add skills related to project management.",
           "Improve your summary section to include specific achievements."
         ]
       }
       ```
  3. **Error Handling**:
     - Check for missing or malformed fields in the response.
     - Default to an error message if parsing fails:
       ```json
       {
         "error": "Failed to generate analysis results."
       }
       ```

- **Unit Tests**:
  - Test parsing logic with a variety of API responses (valid, incomplete, and malformed).
  - Validate that fit scores and feedback are correctly extracted and formatted.

---

## **2.2 Implementing Fit Score Calculation and Feedback**

---

### **Task 21**: Develop Algorithms to Compare Resume Content with Job Description Requirements

- **Objective**: 
  - Create algorithms to evaluate how well the resume matches the job description based on keywords, skills, and experience.

- **Technical Details**:
  1. **Input Data**:
     - Resume text: String of extracted text from the uploaded PDF or pasted text.
     - Job description: String of job requirements, qualifications, and responsibilities.
     - Example Input:
       ```json
       {
         "resume_text": "Experienced software engineer with Python and Java skills...",
         "job_description": "Looking for a software engineer with experience in Python, AWS, and REST APIs."
       }
       ```
  2. **Algorithm Design**:
     - **Keyword Matching**:
       - Tokenize both `resume_text` and `job_description` into individual words.
       - Normalize tokens by converting to lowercase and removing punctuation.
       - Compare tokens to count matched keywords.
     - **Weighting**:
       - Assign higher weights to critical sections of the job description (e.g., “Required Skills” vs. “Preferred Skills”).
       - Example: Match keywords in the “Required Skills” section contribute 70% of the score, while “Preferred Skills” contribute 30%.
     - **Score Calculation**:
       - Fit Score = \( \frac{\text{Number of Matched Keywords}}{\text{Total Keywords in Job Description}} \times 100 \)
  3. **Output**:
     - Fit score as a percentage (0–100).
  4. **Edge Cases**:
     - Empty inputs: Handle cases where either `resume_text` or `job_description` is missing or blank.

- **Unit Tests**:
  - Test the algorithm with inputs of varying lengths and complexity.
  - Validate that the fit score decreases with fewer matching keywords.
  - Verify handling of edge cases like empty or non-string inputs.

---

### **Task 22**: Create Functions to Calculate a Fit Score Based on Keywords, Skills, and Experience

- **Objective**: 
  - Build modular functions that encapsulate the fit score calculation logic.

- **Technical Details**:
  1. **Function Definition**:
     - Create a `calculate_fit_score` function:
       - Input: `resume_text` (string), `job_description` (string).
       - Output: Fit score (integer, 0–100).
     - Python Example:
       ```python
       import re
       from collections import Counter

       def calculate_fit_score(resume_text, job_description):
           # Tokenize and normalize text
           def tokenize(text):
               return re.findall(r'\b\w+\b', text.lower())

           resume_tokens = tokenize(resume_text)
           job_tokens = tokenize(job_description)

           # Count matching keywords
           resume_counter = Counter(resume_tokens)
           job_counter = Counter(job_tokens)

           matches = sum((resume_counter & job_counter).values())
           total_keywords = len(job_tokens)

           return int((matches / total_keywords) * 100) if total_keywords > 0 else 0
       ```
  2. **Integration**:
     - Call this function within the API logic (see Task 24).

- **Unit Tests**:
  - Test the function with mock resume and job description inputs.
  - Verify results for partial and full matches.
  - Test scenarios with edge cases (e.g., empty strings, non-alphanumeric characters).

---

### **Task 23**: Generate Feedback on Missing Skills and Sections in the Resume

- **Objective**:
  - Provide actionable feedback by identifying skills or keywords missing from the resume but present in the job description.

- **Technical Details**:
  1. **Input Data**:
     - Same input as Task 21.
  2. **Feedback Generation Logic**:
     - Extract missing keywords:
       - Compare the tokenized job description to the tokenized resume.
       - Identify words that are present in the job description but not in the resume.
     - Categorize feedback:
       - Highlight missing technical skills (e.g., “Python”, “AWS”).
       - Suggest improvements to sections like “Experience” or “Education”.
     - Example Feedback:
       ```json
       {
         "missing_keywords": ["AWS", "REST APIs"],
         "suggestions": [
           "Include experience with AWS services.",
           "Add projects demonstrating REST API development."
         ]
       }
       ```
  3. **Edge Cases**:
     - Handle scenarios where all job description keywords are present in the resume (output: no feedback needed).

- **Unit Tests**:
  - Verify that missing keywords are correctly identified.
  - Ensure feedback suggestions are relevant and actionable.
  - Test with varying job description lengths and complexities.

---

### **Task 24**: Implement an API Endpoint to Return Fit Score and Feedback to the Frontend

- **Objective**:
  - Create a backend API endpoint that returns the fit score and feedback to the frontend in a structured format.

- **Technical Details**:
  1. **Endpoint Definition**:
     - **URL**: `POST /api/fit-score`
     - **Request Payload**:
       ```json
       {
         "resume_text": "Your resume text here...",
         "job_description": "Your job description here..."
       }
       ```
     - **Validation**:
       - Ensure both `resume_text` and `job_description` are provided and are strings.
       - Enforce a character limit (e.g., max 10,000 characters per field).
  2. **Backend Logic**:
     - Validate inputs.
     - Call the `calculate_fit_score` function (Task 22).
     - Generate feedback using logic from Task 23.
     - Return a combined response:
       ```json
       {
         "fit_score": 85,
         "feedback": [
           "Include experience with AWS services.",
           "Add projects demonstrating REST API development."
         ]
       }
       ```
  3. **Error Handling**:
     - Return `400 Bad Request` if inputs are invalid.
     - Return `500 Internal Server Error` for processing failures.
  4. **Response Examples**:
     - **Success** (`200 OK`):
       ```json
       {
         "fit_score": 85,
         "feedback": [
           "Include experience with AWS services.",
           "Add projects demonstrating REST API development."
         ]
       }
       ```
     - **Error** (`400 Bad Request`):
       ```json
       {
         "error": "Invalid input data. Both resume_text and job_description are required."
       }
       ```

- **Unit Tests**:
  - Test API responses with valid and invalid payloads.
  - Mock algorithm functions to isolate API logic during testing.
  - Validate edge cases like empty or oversized inputs.

---

