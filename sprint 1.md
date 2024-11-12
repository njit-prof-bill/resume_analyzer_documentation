
# Sprint 1: Core Setup and Input Handling

## 1.1 Project Repository Setup

### **Task 1**: Set up the GitHub Repository and Initialize the Project Structure
- **Description**:
  - Create a GitHub repository and set up the initial structure.
- **Instructions**:
  1. Create a new GitHub repository.
  2. Initialize with a `.gitignore`, `README.md`, and folders:
     ```
     ├── backend/
     ├── frontend/
     ├── docs/
     ├── tests/
     ├── .gitignore
     ├── README.md
     └── requirements.txt
     ```
  3. `README.md` should include:
     - Project name, description, team members, and contact info.
     - High-level project overview and goals.
     - Instructions on setting up the project locally.
  4. Invite the professor as a collaborator to your repository.
- **Unit Tests**: None required, but ensure all team members can access and clone the repository.

---

### **Task 2**: Create a Template for Documentation and Coding Standards
- **Description**:
  - Establish team documentation and coding standards.
- **Instructions**:
  1. In `docs/`, create:
     - `CONTRIBUTING.md`: Guidelines for branch naming (`feature/`, `fix/`), commit messages, and PR reviews.
     - `STYLE_GUIDE.md`: Define coding standards (e.g., camelCase, spacing, commenting).
     - `SETUP.md`: How to install dependencies and run the project.
  2. Review these with the team.
- **Unit Tests**: None required, but peer-review the documentation.

### **Task 3**: Set up Docker Configurations (Optional)
- **Description**:
  - Containerize the project.
- **Instructions**:
  1. Create `Dockerfile` for both backend and frontend.
  2. Set up a `docker-compose.yml`.
  3. Test with `docker-compose up`.
- **Unit Tests**: Ensure services start correctly on different machines.

---

## 1.2 User Registration, Authentication, and Session Management

### **Task 4**: Implement User Sign-Up Endpoint (API) Use FastAPI if Python
- **Description**:
  - Create a registration endpoint.
- **Instructions**:
  1. Endpoint: `POST /api/register`
  2. Request payload:
     ```json
     {
       "email": "user@example.com",
       "password": "securePassword",
       "username": "user123"
     }
     ```
  3. Validate that email is unique, hash password with `bcrypt`.
  4. Response:
     - Success (`201 Created`):
       ```json
       { "message": "User registered" }
       ```
     - Error (`400 Bad Request`).
- **Unit Tests**: Test input validation and email uniqueness.

### **Task 5**: Implement User Login Endpoint (API) and JWT-Based Session Management
- **Description**:
  - Create a login endpoint with JWT.
- **Instructions**:
  1. Endpoint: `POST /api/login`
  2. Request payload:
     ```json
     { "email": "user@example.com", "password": "securePassword" }
     ```
  3. Generate a JWT token with expiration.
  4. Response:
     ```json
     { "token": "jwt-token" }
     ```
- **Unit Tests**: Test successful/failed login attempts.

### **Task 6**: Develop Frontend Components for Registration and Login Forms
- **Description**:
  - Build forms in React.
- **Instructions**:
  1. Create `SignUp` and `Login` components.
  2. Fields:
     - SignUp: `Email`, `Username`, `Password`, `Confirm Password`.
     - Login: `Email`, `Password`.
  3. Use Axios to send data to backend.
- **Unit Tests**: Mock API responses.

### **Task 7**: Integrate Frontend with Authentication API
- **Description**:
  - Handle authentication on frontend.
- **Instructions**:
  1. Store JWT in `localStorage`.
  2. Protect routes (e.g., dashboard).
- **Unit Tests**: Test session handling.

---

Here are the in-depth technical specifications for **items 1.3 (Resume and Job Description Input Handling)** and **1.4 (Frontend Dashboard Layout)**. Each task in these sections will include detailed technical instructions, covering necessary form fields, data validation, backend logic, data structures, API endpoints, and unit tests.

---

# 1.3 Resume and Job Description Input Handling

---

### **Task 8**: Create API Endpoints to Accept Resume Upload (PDF) or Pasted Text with Validation

- **Objective**: Develop API endpoints for uploading resumes (PDFs) or pasting text job descriptions, with data validation for both inputs.

- **Technical Details**:
  1. **Endpoint for Resume Upload**:
     - URL: `POST /api/resume-upload`
     - Expected Content-Type: `multipart/form-data`
     - Request:
       - `resume_file`: File field for uploading a PDF or docx.
     - Backend Logic:
       - Check the file type to confirm it’s a PDF (e.g., MIME type `application/pdf`) or docx.
       - Validate the file size (should not exceed 2MB).
       - Temporarily store the file in memory or process it directly (no persistence required).
     - Response:
       - **Success** (`200 OK`):
         ```json
         {
           "message": "Resume uploaded successfully.",
           "status": "success"
         }
         ```
       - **Error** (`400 Bad Request`):
         ```json
         {
           "error": "Invalid file type. Only PDF files are allowed.",
           "status": "error"
         }
         ```
  
  2. **Endpoint for Job Description Text Input**:
     - URL: `POST /api/job-description`
     - Expected Content-Type: `application/json`
     - Request:
       - `job_description`: Text field for the job description (e.g., requirements, qualifications).
       - Validation: Maximum character count is 5,000 characters.
     - Backend Logic:
       - Ensure the text input does not exceed 5,000 characters.
       - Clean the text by removing extraneous whitespace.
     - Response:
       - **Success** (`200 OK`):
         ```json
         {
           "message": "Job description submitted successfully.",
           "status": "success"
         }
         ```
       - **Error** (`400 Bad Request`):
         ```json
         {
           "error": "Job description exceeds character limit.",
           "status": "error"
         }
         ```

- **Unit Tests**:
  - Test successful and failed file uploads (valid PDF, invalid file type, and oversized file).
  - Test successful and failed text input submissions (valid length, over 5,000 characters).

---

### **Task 9**: Implement Frontend Form for Resume Upload and Job Description Input

- **Objective**: Build a frontend form that allows users to upload a PDF resume or enter a job description as text.

- **Technical Details**:
  1. **Form Fields**:
     - **Resume Upload Field**:
       - Field Type: File input.
       - Accept only PDF files.
       - Display error message if an unsupported file type or oversized file is uploaded.
     - **Job Description Field**:
       - Field Type: Text area.
       - Character limit: 5,000 characters.
       - Display real-time character count.
       - Provide a warning if the character limit is approached or exceeded.

  2. **Form Submission**:
     - Use Axios or Fetch API to submit the form data:
       - Upload the resume to `POST /api/resume-upload`.
       - Submit the job description to `POST /api/job-description`.
     - Display submission success or error messages on the form.

- **Unit Tests**:
  - Test file input validation: supported file types, max size.
  - Test text area character count: under, at, and above the limit.
  - Mock API calls to validate successful and error responses.

---

### **Task 10**: Add Validation for Input Types (File Size, Format, Character Count)

- **Objective**: Add comprehensive validation for file uploads and text input fields.

- **Technical Details**:
  - **File Validation**:
    - Ensure that only PDF files are accepted.
    - Reject files larger than 2MB.
  - **Text Input Validation**:
    - Ensure the job description does not exceed 5,000 characters.
    - Provide client-side feedback on character count.

- **Unit Tests**:
  - Verify that non-PDF file uploads are rejected.
  - Check that files over 2MB are not accepted.
  - Confirm that job descriptions over 5,000 characters trigger an error.

---

### **Task 11**: Create Utility Functions to Parse and Extract Text from Uploaded Files

- **Objective**: Create utility functions for parsing and extracting text from uploaded PDFs.

- **Technical Details**:
  - **Backend Utility**:
    - Use libraries like PyPDF2 (Python) or pdf-lib (JavaScript) for text extraction.
    - The extracted text should be processed to remove unnecessary line breaks and whitespace.
  - **Example Function in Python**:
    ```python
    from PyPDF2 import PdfReader

    def extract_text_from_pdf(file):
        reader = PdfReader(file)
        text = ""
        for page in reader.pages:
            text += page.extract_text()
        return text.strip()
    ```
- **Unit Tests**:
  - Test text extraction on sample PDFs with varying structures.
  - Verify extracted text format (e.g., no extraneous whitespace).

---

### **Task 12**: Implement Temporary In-Memory Storage for Uploaded Data

- **Objective**: Store uploaded data temporarily in memory (no persistence to a database).

- **Technical Details**:
  - **Backend**:
    - Use an in-memory data store like Python’s dictionary or cache.
    - Store extracted resume text and job description in memory with session identifiers.
    - Example structure for temporary storage:
      ```python
      temp_storage = {
          "session_id_123": {
              "resume_text": "Extracted resume text here...",
              "job_description": "Submitted job description here..."
          }
      }
      ```
    - Ensure that data is cleared after processing to free memory.
- **Unit Tests**:
  - Test data insertion and retrieval in memory.
  - Ensure data is removed once processing is complete.

---

# 1.4 Frontend Dashboard Layout

---

### **Task 13**: Design the Dashboard UI for Viewing Analysis Results

- **Objective**: Create a user-friendly dashboard for displaying resume analysis results and job recommendations.

- **Technical Details**:
  - **Dashboard Layout**:
    - Display the following sections:
      - **Resume Fit Score**: Show a visual score indicating how well the resume matches the job description.
      - **Skills and Keywords Matched**: List matched keywords and skills.
      - **Improvement Suggestions**: Show specific areas for resume improvement.
  - **Visuals**:
    - Use progress bars or charts for the fit score.
    - Show feedback items in a list format.
- **Unit Tests**:
  - Verify that all dashboard elements render correctly.
  - Test with various fit score and feedback data to ensure adaptability.

---

### **Task 14**: Set up Basic Routing/Navigation for the Dashboard

- **Objective**: Implement navigation for the application.

- **Technical Details**:
  - **Routes**:
    - `/register`: Registration page.
    - `/login`: Login page.
    - `/dashboard`: Dashboard with analysis results (restricted to authenticated users).
  - **Frontend**:
    - Use React Router or Vue Router for navigation.
    - Redirect users to `/login` if they attempt to access `/dashboard` without authentication.
- **Unit Tests**:
  - Verify that restricted routes enforce login.
  - Confirm that navigation works across all routes.

---

### **Task 15**: Implement a Loading Spinner and Error Handling for API Requests

- **Objective**: Add loading indicators and error handling to improve user experience.

- **Technical Details**:
  - **Loading Indicator**:
    - Show a spinner when API requests are in progress.
    - Hide the spinner once data is loaded.
  - **Error Handling**:
    - Display error messages if API requests fail.
    - Use try-catch blocks and handle rejected promises in the frontend.
- **Unit Tests**:
  - Test that the loading spinner appears during API requests.
  - Test error handling by simulating failed API responses.

---

### **Task 16**: Add Unit Tests for Frontend Components (Registration, Input Forms, Dashboard)

- **Objective**: Ensure all frontend components have thorough unit tests.

- **Technical Details**:
  - **Components to Test**:
    - Registration and Login Forms: Validate input fields and form submission.
    - Input Forms for Resume and Job Description: Check file uploads, text area input, and character count.
    - Dashboard: Verify display of fit score, feedback, and suggestions.
  - **Testing Frameworks**:
    - Use Jest and React Testing Library for unit tests (if using React).
- **Unit Tests**:
  - Achieve at least 80% code coverage across all components.
  - Mock API calls and simulate various data states for the dashboard.
