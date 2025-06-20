
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
- In the **Code** tab: Paste the python code from [PresignUrl.py](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/tree/main/Lambda%20-%20Python%20Code).
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
- In the **Code** tab: Paste the python code from [ConvertTextToAudio.py](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/tree/main/Lambda%20-%20Python%20Code).
- Limited Pre-Installed Libraries – Lambda’s default Python runtime does not include libraries like PyPDF2, pdf2text, or textract (required to extract text from PDFs/DOCs).
- **Layers** allow you to package dependencies separately and attach them to Lambda functions. 
- At the **Layers** tab : create Layer
- Attach the provided `.zip` layer (contains `PyPDF2`, `pdf2text`, etc.).
![0014](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0014.png)
![0015](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0015.png)
![0016](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0016.png)
- Under **Code** tab, scroll down until you see the **Layers** tab.
- Click on **Add Layer**,
  - choose Custom Layer : Pick precreated Layer and version 1.
  - Click **Add Layer**.
![0017](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0017.png)
![0018](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0018.png)
Under **Configuration** tab, Edit **General Configuration**. 
- **Timeout**: Set to **1m 30s** and Save changes.
![0019](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0019.png)
- **Environment Variables**:
- Let Edit the Environment variables for this lambda.
- Under **Configuration** → edit **Environment Variables**, add:
  - `bucket_name`: `text-to-audio-psk-file`.  
  - `sender_mail`: `your-verified-email@example.com`.
- This time you have to add an email which will also receive the Audio file as well.
![0020](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0020.png)
![0021](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0021.png)
### Set Up Amazon SES
- Go to Amazon SES service.
- Choose **Email**, and enter your desired email address.
- The person will receive verification email.
![0022](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0022.png)
![0023](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0023.png)
- After confirm the action your identity status will be **verified**.
![0024](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0024.png)
![0025](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0025.png)

---

### **3. Configure S3 Event Triggers**  
In `text-to-audio-psk-file` bucket → **Properties → Event Notifications**:  
- Name the event
- Assign **uploads** folder to access pdf or docx file from the user
- Check the `Put` and `Post` **Event types**
- Choose our `ConvertTextToAudio` lambda and **save changes**
- Do this step for two event notifications:
  1. For pdf file types
  2. For docx file types (with the same lambda)  
![0026](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0026.png)
![0027](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0027.png)
![0028](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0028.png)
![0029](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0029.png)

---

### **4. Create Static Website Hosting Bucket
- Create **new bucket** for static website hosting (do not start with capital letter of the bucket name)
- I've named `website-hosting-buck`.
- Disable **public access blocking**, because we are testing to access the website directly, u know.
- Check the *acknowledge of turning off the public access blocking*
- Create bucket.
![0030](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0030.png)
### Configure Static Website
- Click on **website-hosting-buck**
- Under **permissions** tab, edit **bucket policy**
- Generate the `getObject` action from policy generator for all user and paste it
- ### Don't forget to add * behind the / .
- Save changes

![0031](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0031.png)
![0032](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0032.png)

- Under **properties** tab, edit **static website hosting** and **enable** it.
- Add "index.html" in index document
- Save changes
![0033](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0033.png)

### **5. API Gateway**  
- **Create HTTP API** with a POST method (`/presignurl`) integrated with `PresignUrl` Lambda.  
- **Enable CORS** with `*` allowed origins.  
- **Deploy API** and note the `invoke-url` (e.g., `https://abc123.execute-api.region.amazonaws.com`).  

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
