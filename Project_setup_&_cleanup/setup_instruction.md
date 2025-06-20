
# **Serverless Text-to-Audio Converter on AWS**  
**Convert uploaded PDF/DOCX files to audio (MP3) using AWS Lambda, S3, Polly, and SES.**  

## **Architecture Overview**  
![AWS Architecture Diagram](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/The%20Diagram.png)

1. Users upload documents (PDF/DOCX) to S3 via a web interface.  
2. S3 triggers a Lambda function (`ConvertTextToAudio`) to extract text and synthesize speech via Amazon Polly.  
3. Processed audio files are saved to S3 and emailed to users via Amazon SES.  

---

## **Prerequisites**  
- AWS Account with permissions for:  
  - Lambda, S3, IAM, API Gateway, Polly, SES.  
- Verified SES email identity (for sending audio files).  
- Python 3.13+ (for testing locally).  

---

## **Setup Instructions**  

### **1. S3 Buckets**  
#### **a. Create `text-to-audio-psk-file` Bucket**  
- **Name**: `text-to-audio-psk-file` (replace `psk` with your initials).  
- **Block all public access** (enabled by default).
- **create bucket**.
![0000](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0000.png)
![0001](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0001.png)
![0002](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0002.png)
- **Create two folders**:
- Click on you bucket.Add file folder.
  - `uploads/` (for user inputted pdf or documents).  
  - `audio/` (for output MP3 files).
  - I have showed you how to add uploads/ folder. You have to do the same step for audio folder.
![0003](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0003.png)
![0004](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0004.png)
![0005](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0005.png)

#### **b. Create Static Website Hosting Bucket**  
- **Name**: `static-website-host` (must be lowercase).  
- **Disable public access blocking** (required for hosting).  
- **Enable static website hosting**:  
  - **Index document**: `index.html`.  
- **Bucket Policy** (replace `BUCKET_ARN`):  
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "BUCKET_ARN/*"
    }]
  }
  ```

---

### **2. Lambda Functions**  
#### **a. `PresignUrl` Function**  
- **Runtime**: Python 3.13 or higher.
- click **create function**.
![0006](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0006.png)
![0007](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0007.png)
![0008](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0008.png)
- **Code**: Paste [PresignUrl.py](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/tree/main/Lambda%20-%20Python%20Code).
- If you receive the warning box, you can just *accept* it.
![0009](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0009.png)
![0010](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0010.png)
- **Environment Variables**:
- Under **Configuration** → edit **Environment Variables**, add:
  - `BUCKET_NAME`: `text-to-audio-psk-file`.  
  -  This is my bucket name, don't forget to paste your bucket.
  -  We are going to send S3 bucket Url to receive the pdf or document file.
![0011](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0011.png)
![0012](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0012.png)
![0013](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0013.png)
#### **b. `ConvertTextToAudio` Function**  
- **Runtime**: Python 3.13 or higher.  
- **Code**: Paste [ConvertTextToAudio.py](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/tree/main/Lambda%20-%20Python%20Code).
- Limited Pre-Installed Libraries – Lambda’s default Python runtime does not include libraries like PyPDF2, pdf2text, or textract (required to extract text from PDFs/DOCs).
- **Layers** allow you to package dependencies separately and attach them to Lambda functions. 
- At the **Layers** tab : create Layer
- Attach the provided `.zip` layer (contains `PyPDF2`, `pdf2text`, etc.).
![0014](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0014.png)
![0015](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0015.png)
![0016](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0016.png)
- **Timeout**: Set to **1m 30s**.  
- **Environment Variables**:  
  - `bucket_name`: `text-to-audio-psk-file`.  
  - `sender_mail`: `your-verified-email@example.com`.  
- **Permissions**: Attach inline IAM policy allowing:  
  - S3 (`PutObject`, `GetObject`, `ListBucket`).  
  - Polly (`synthesizeSpeech`).  
  - SES (`sendEmail`).  

---

### **3. API Gateway**  
- **Create HTTP API** with a POST method (`/presignurl`) integrated with `PresignUrl` Lambda.  
- **Enable CORS** with `*` allowed origins.  
- **Deploy API** and note the `invoke-url` (e.g., `https://abc123.execute-api.region.amazonaws.com`).  

---

### **4. Configure S3 Event Triggers**  
In `text-to-audio-psk-file` bucket → **Properties → Event Notifications**:  
- **Create two rules** (for `uploads/` folder):  
  1. **Trigger on `.pdf`** → Invoke `ConvertTextToAudio`.  
  2. **Trigger on `.docx`** → Invoke `ConvertTextToAudio`.  
- **Event types**: `Put`, `Post`.  

---

### **5. Static Website Deployment**  
1. Update `index.html` with your API Gateway endpoint:  
   ```javascript
   const API_URL = "https://your-api-id.execute-api.region.amazonaws.com/presignurl";
   ```  
2. Upload `index.html` to the static hosting bucket.  
3. Access the site via the bucket’s **Endpoint URL** (e.g., `http://static-website-host.s3-website.region.amazonaws.com`).  

---

## **Testing the Workflow**  
1. **Upload a PDF/DOCX** via the web interface.  
2. Check:  
   - `audio/` folder in S3 for the output MP3.  
   - Your SES email for the audio attachment.  

---

## **Troubleshooting**  
| Issue | Solution |  
|-------|----------|  
| `ModuleNotFoundError` | Ensure the Lambda layer is attached. |  
| Lambda timeout | Increase timeout to >1m or optimize code. |  
| SES email not received | Check verified identities in SES. |  

---


**License**: MIT  
**Repository**: [github.com/your-repo](https://github.com/pyaesoekyaw)  
