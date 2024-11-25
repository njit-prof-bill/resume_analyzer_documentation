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

### **Task 18**: Develop an API Endpoint to Send Resume and Job Description to the NLP API

- **Objective**:
  - Build a backend endpoint to connect with the NLP API and facilitate communication between the frontend and AI service. 
  Note: the examples here are illustrative and are not a copy and paste solution.

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


### **Task 21**: Develop Algorithms to Compare Resume Content with Job Description Requirements

- **Objective**: 
  - Create algorithms to evaluate how well the resume matches the job description based on keywords, skills, and experience.

- **Technical Details**:
  1. **Input Data**:
     - Resume text: String of extracted text from the uploaded PDF, docx, or pasted text.
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

## **2.3 Frontend Integration for Feedback Display**


### **Task 25**: Build Frontend Components to Display Fit Score and Feedback on the Dashboard

- **Objective**:
  - Create a responsive and user-friendly interface to display the fit score and actionable feedback.

- **Technical Details**:
  1. **Dashboard Layout**:
     - Use the framework from the sprint 1 work.
     - Divide the dashboard into three sections:
       - **Fit Score Visualization**: Display the fit score prominently using a progress bar, gauge chart, or numerical representation.
       - **Matched Skills**: List the keywords or skills from the resume that align with the job description.
       - **Improvement Suggestions**: Provide specific feedback on how to enhance the resume to better match the job description.
  2. **Integration with Backend**:
     - Fetch data from the `POST /api/fit-score` endpoint (defined in Task 24).
     - Example API response:
       ```json
       {
         "fit_score": 85,
         "feedback": [
           "Include experience with AWS services.",
           "Add projects demonstrating REST API development."
         ],
         "matched_keywords": ["Python", "REST APIs", "AWS"]
       }
       ```
     - Map this response to the dashboard sections.
  3. **Visual Design**:
     - Use libraries like Chart.js or D3.js for visualizations.
     - Example Fit Score Visualization:
       ```jsx
       <ProgressBar value={fitScore} max={100} />
       ```
     - Example Feedback List:
       ```jsx
       <ul>
         {feedback.map((item, index) => (
           <li key={index}>{item}</li>
         ))}
       </ul>
       ```
  4. **User Experience Enhancements**:
     - Include tooltips for feedback items with additional context.
     - Highlight matched keywords within the resume.

- **Unit Tests**:
  - Mock API responses to test the rendering of fit scores, feedback, and matched keywords.
  - Test for edge cases like missing or empty API data (e.g., no feedback).

---

### **Task 26**: Allow Users to Download a PDF Report of the Feedback and Recommendations

- **Objective**:
  - Provide users with the ability to download a detailed PDF report of their analysis results.

- **Technical Details**:
  1. **Backend Integration**:
     - Fetch the same data used for the dashboard from `POST /api/fit-score`.
  2. **Frontend Logic**:
     - Use a library like jsPDF or PDFKit to generate a PDF report.
     - Example Report Contents:
       - **Header**: Application title and analysis date.
       - **Fit Score Section**: Include a graphical representation (e.g., a gauge chart or numerical score).
       - **Matched Keywords**: List the keywords from the resume that align with the job description.
       - **Feedback Section**: Provide a detailed list of suggestions for improvement.
     - Example Code Snippet (using jsPDF):
       ```javascript
       import jsPDF from 'jspdf';

       function generatePDF(fitScore, matchedKeywords, feedback) {
           const doc = new jsPDF();
           doc.text("Resume Analysis Report", 10, 10);
           doc.text(`Fit Score: ${fitScore}%`, 10, 20);
           doc.text("Matched Keywords:", 10, 30);
           matchedKeywords.forEach((keyword, index) => {
               doc.text(`- ${keyword}`, 10, 40 + index * 10);
           });
           doc.text("Feedback:", 10, 60);
           feedback.forEach((item, index) => {
               doc.text(`- ${item}`, 10, 70 + index * 10);
           });
           doc.save("Resume_Analysis_Report.pdf");
       }
       ```
  3. **Frontend Button**:
     - Add a "Download Report" button to the dashboard that triggers the PDF generation.
     - Example Button:
       ```jsx
       <button onClick={() => generatePDF(fitScore, matchedKeywords, feedback)}>
         Download PDF Report
       </button>
       ```

- **Unit Tests**:
  - Validate the correct data is included in the generated PDF.
  - Test PDF creation and download functionality.

---

### **Task 27**: Add Filtering Options for Users to Focus on Specific Feedback Categories

- **Objective**:
  - Allow users to filter feedback by categories such as skills, experience, and formatting.

- **Technical Details**:
  1. **Feedback Categorization**:
     - Extend the backend API response to include categories for each feedback item:
       ```json
       {
         "fit_score": 85,
         "feedback": [
           { "category": "skills", "text": "Include experience with AWS services." },
           { "category": "experience", "text": "Add projects demonstrating REST API development." }
         ]
       }
       ```
  2. **Filter UI**:
     - Add a dropdown or checkbox group to filter feedback:
       ```jsx
       <select onChange={(e) => setFilter(e.target.value)}>
         <option value="all">All</option>
         <option value="skills">Skills</option>
         <option value="experience">Experience</option>
       </select>
       ```
  3. **Filtering Logic**:
     - Use a state variable to track the selected filter.
     - Filter the feedback list dynamically based on the selected category:
       ```jsx
       const filteredFeedback = feedback.filter((item) =>
         filter === "all" ? true : item.category === filter
       );
       ```

- **Unit Tests**:
  - Verify that selecting a filter updates the displayed feedback.
  - Test edge cases, such as selecting a filter with no matching feedback.

---

### **Task 28**: Add Unit Tests to Validate the Correct Display of Feedback and Fit Scores

- **Objective**:
  - Ensure all components for displaying feedback and fit scores are thoroughly tested.

- **Technical Details**:
  1. **Component Tests**:
     - Test the rendering of the fit score visualization:
       - Input: A range of scores (e.g., 0%, 50%, 100%).
       - Expected Output: Correct visualization for each score.
     - Test the feedback list:
       - Input: Various feedback arrays.
       - Expected Output: All feedback items are rendered correctly.
  2. **API Integration Tests**:
     - Mock API calls to ensure the dashboard components correctly handle and display data.
  3. **Filter Tests**:
     - Test filtering functionality:
       - Input: Feedback array with multiple categories.
       - Expected Output: Only feedback matching the selected filter is displayed.
  4. **Edge Cases**:
     - Handle scenarios with missing or incomplete data:
       - No fit score.
       - Empty feedback list.

- **Unit Testing Frameworks**:
  - Use Jest and React Testing Library (if React) or Vue Test Utils (if Vue.js).

---

## **2.4 Final Testing and Debugging**


### **Task 29**: Perform End-to-End Testing of the Application, Covering the Entire Workflow

- **Objective**:
  - Validate that the entire application workflow functions as intended, from user registration to feedback display.

- **Technical Details**:
  1. **Test Scenarios**:
     - Cover the following key user stories:
       - **User Registration**:
         - New users can register with a valid email, username, and password.
       - **User Login**:
         - Registered users can log in with correct credentials and are redirected to the dashboard.
       - **Resume Upload and Job Description Submission**:
         - Users can upload a valid PDF resume and paste a job description for analysis.
       - **Fit Score and Feedback Display**:
         - The dashboard displays the correct fit score and feedback after analysis.
       - **Report Download**:
         - Users can generate and download a PDF report of their analysis results.
  2. **Test Cases**:
     - **Positive Tests**:
       - Valid inputs for all endpoints.
       - Proper API responses for all user actions.
     - **Negative Tests**:
       - Invalid file types or oversized uploads.
       - Missing or invalid inputs for any endpoint.
       - Unauthorized access to protected routes (e.g., dashboard without login).
  3. **Automation Tools**:
     - Use a testing framework like **Playwright** or **Cypress** for frontend E2E tests.
     - Use **Postman** or **Pytest** for backend API testing.
  4. **Example Automated Test Workflow**:
     - **Registration Test**:
       ```javascript
       test('User can register successfully', async () => {
           await page.goto('http://localhost:3000/register');
           await page.fill('#email', 'test@example.com');
           await page.fill('#username', 'testuser');
           await page.fill('#password', 'TestPassword123');
           await page.fill('#confirm-password', 'TestPassword123');
           await page.click('button[type="submit"]');
           expect(await page.textContent('.success')).toBe('Registration successful');
       });
       ```
     - **Backend API Test (Pytest)**:
       ```python
       def test_login_endpoint(client):
           response = client.post("/api/login", json={
               "email": "test@example.com",
               "password": "TestPassword123"
           })
           assert response.status_code == 200
           assert "token" in response.json()
       ```

- **Unit Tests**:
  - Ensure 80–90% code coverage for all major components (frontend and backend).
  - Write additional tests to address uncovered lines or edge cases.

---

### **Task 30**: Write Project Documentation (Setup Guide, Usage Instructions, API Documentation)

- **Objective**:
  - Provide comprehensive documentation to guide users and developers in understanding, setting up, and using the application.

- **Technical Details**:
  1. **Setup Guide (`SETUP.md`)**:
     - Include:
       - **System Requirements**:
         - Node.js version, Python version, and any required dependencies.
       - **Installation Instructions**:
         ```bash
         git clone https://github.com/example/resume-analyzer.git
         cd resume-analyzer/backend
         pip install -r requirements.txt
         cd ../frontend
         npm install
         ```
       - **Running the Application**:
         ```bash
         # Backend
         uvicorn main:app --reload
         # Frontend
         npm run dev
         ```
  2. **Usage Instructions (`README.md`)**:
     - Include:
       - Application overview and key features.
       - Step-by-step guide for:
         - Registering and logging in.
         - Uploading a resume and submitting a job description.
         - Viewing analysis results and downloading a report.
  3. **API Documentation**:
     - Use a tool like **Swagger** (FastAPI integration) or **Postman Collections**.
     - Document all endpoints with:
       - **Method**: GET/POST
       - **URL**: Endpoint URL
       - **Request Body**: JSON format with required/optional fields.
       - **Response Body**: JSON format with sample success/error responses.
     - Example API Documentation:
       ```
       POST /api/fit-score
       Request:
       {
         "resume_text": "Resume content...",
         "job_description": "Job description..."
       }
       Response:
       {
         "fit_score": 85,
         "feedback": ["Include AWS experience", "Add REST API projects"]
       }
       ```

- **Unit Tests**:
  - Validate all setup instructions on a clean environment to ensure correctness.
  - Review API documentation for completeness and accuracy.

---

### **Task 31**: Conduct a Final Code Review and Refactoring Session to Improve Code Quality

- **Objective**:
  - Review the codebase to ensure it meets quality standards, and refactor as needed to improve maintainability and performance.

- **Technical Details**:
  1. **Code Review Checklist**:
     - **Readability**:
       - Code is well-commented and uses meaningful variable/function names.
     - **Consistency**:
       - Adheres to the defined coding standards in `STYLE_GUIDE.md`.
     - **Performance**:
       - Ensure efficient use of resources (e.g., avoid redundant API calls).
     - **Security**:
       - Verify that sensitive data (e.g., passwords, API keys) is handled securely.
     - **Error Handling**:
       - Ensure robust error handling for all API endpoints and frontend interactions.
  2. **Refactoring Opportunities**:
     - Identify and eliminate duplicate code.
     - Modularize large functions into smaller, reusable components.
     - Optimize queries or API calls for better performance.
  3. **Peer Review Process**:
     - Assign two team members to review each pull request.
     - Use comments to suggest improvements or approve changes.
  4. **Tooling**:
     - Use **ESLint/Prettier** (for JavaScript) or **Black/Flake8** (for Python) to enforce code style.
     - Integrate tools like **SonarQube** for static code analysis.

- **Unit Tests**:
  - Validate that all test cases pass after refactoring.
  - Ensure no new bugs are introduced during the refactoring process.

---
