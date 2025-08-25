# 📄 Resume ATS Workflow (Final Version)
![ATS Resume Screening](ats_resume_screening.png)

## Overview
This n8n workflow automates the process of **retrieving candidate resumes from Outlook**, **extracting data**, and **evaluating them** against stored job descriptions using an **AI-powered ATS scoring system**.  

It:
1. Connects to Microsoft Outlook to fetch new job application emails within the last 24 hours.
2. Downloads and processes attached resumes (PDF or DOCX → PDF).
3. Extracts text from resumes.
4. Matches the candidate's job title to stored job postings in Airtable.
5. Uses OpenAI to score the resume based on a strict **100-point rubric**.
6. Filters and stores shortlisted candidates in Google Drive and Airtable.

---

## ⚙️ Features

- **Automated Email Fetching**: Retrieves only recent emails with attachments from a specific Outlook folder.
- **File Type Handling**: Supports both PDF and DOCX (automatically converts DOCX to PDF).
- **Resume Text Extraction**: Extracts full resume text for AI processing.
- **Job Title Matching**: Compares email subject to a stored list of valid job titles.
- **AI Evaluation**: Uses OpenAI to score resumes based on:
  1. Education (20 pts)
  2. Tools, Certifications, Technologies (20 pts)
  3. Relevant Work Experience (20 pts)
  4. Skills Alignment with Job Duties (20 pts)
  5. Overall Resume Quality (20 pts)
- **Geographic Filtering**: Optionally filters candidates by country (`Netherlands` in this version).
- **Candidate Shortlisting**: Saves high-scoring candidates (≥ 50 pts) to:
  - **Google Drive** (resume file)
  - **Airtable** (candidate info & link to file)

---

## 🔗 Integrations Used

- **Microsoft Outlook (OAuth2)** – Fetch job application emails & attachments
- **ConvertAPI** – Convert DOCX resumes to PDF
- **Google Drive** – Store shortlisted candidate resumes
- **Airtable** – Store job descriptions and shortlisted candidate info
- **OpenAI API** – ATS scoring & job title matching

---

## 🛠 Setup Instructions

### 1. Required Credentials
Before importing this workflow, set up the following credentials in n8n:

- **Microsoft OAuth2 API** (Outlook access)
- **ConvertAPI** (DOCX to PDF conversion)
- **Google Drive OAuth2 API** (resume storage)
- **Airtable Personal Access Token**
- **OpenAI API Key**

### 2. Airtable Configuration
This workflow expects two tables in Airtable:

#### `Job Description`
- **Job Title** (Text) – List of valid job titles
- **Job Description** (Long Text) – Description text for AI scoring

#### `Resume info`
- **Name**
- **Email**
- **Phone**
- **Job Title**
- **Score**
- **Summary**
- **Resume Link**

### 3. Folder IDs
Update the Google Drive **folderId** parameter to point to your desired storage folder.

---

## 📅 Workflow Logic

### Trigger
- **Schedule Trigger**: Runs daily at **19:30**.

### Steps
1. **Fetch Outlook Folders**
   - Retrieve `Solicitates` folder ID.
2. **Filter Recent Emails**
   - Only last 24 hours.
3. **Check Attachments**
   - Continue only if attachments exist.
4. **Download Resume**
   - If PDF → proceed to extraction.
   - If DOCX → convert to PDF via ConvertAPI.
5. **Extract Resume Text**
6. **Match Job Title**
   - AI matches email subject to stored job titles.
7. **Fetch Job Description**
   - From Airtable based on matched job title.
8. **AI ATS Scoring**
   - Strict scoring rubric (max 80 unless exceptional).
   - Extracts: `score`, `summary`, `job_title`, `candidate_name`, `email`, `phone_number`, `country`.
9. **Filter Candidates**
   - Country = Netherlands (configurable).
   - Score ≥ 50.
10. **Save Data**
    - Upload resume to Google Drive.
    - Create record in Airtable with candidate details.

---

## 📌 Notes & Best Practices

- **Security**: Remove or rotate any hardcoded tokens before sharing.
- **Error Handling**: The workflow retries API calls on failure (`retryOnFail` enabled for critical steps).
- **Custom Country Filter**: Change the `If Country is Netherlands` node to filter by your preferred location.
- **Scoring Model**: Adjust AI prompt in `Message a model1` to change evaluation rules.

---

## 🚀 Example Output

```json
{
  "score": 76,
  "summary": "The candidate has solid experience and education but lacks familiarity with required technologies.",
  "job_title": "Reverse Supply Chain Engineer",
  "candidate_name": "Berna Ali",
  "email": "berna.ali@email.com",
  "phone_number": "+1-555-123-4567",
  "country": "Netherlands"
}
