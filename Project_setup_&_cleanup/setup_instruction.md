
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

## **1. S3 Buckets**  
### **a. Create `text-to-audio-psk-file` Bucket**  
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

---

## **2. Lambda Functions**  
### Create **`PresignUrl` Function**  

- **Runtime**: Python 3.13 or higher.
- click **create function**.
![0006](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0006.png)
![0007](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0007.png)
![0008](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0008.png)
- In the **Code** tab: Paste the [code](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/Lambda%20-%20Python%20Code/PresignUrl-psk.py).
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

### Create **`ConvertTextToAudio` Function**  

- **Runtime**: Python 3.13 or higher.  
- In the **Code** tab: Paste the [code](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/Lambda%20-%20Python%20Code/AudioConvertion.py).
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

---

## **3. Set Up Amazon SES**

- Go to Amazon SES service.
- Choose **Email**, and enter your desired email address.
- The person will receive verification email.
![0022](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0022.png)
![0023](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0023.png)
- After confirm the action your identity status will be **verified**.
![0024](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0024.png)
![0025](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0025.png)

---

## **4. Configure S3 Event Triggers**

In `text-to-audio-psk-file` bucket → **Properties → Event Notifications**:  
- Name the event
- Assign **uploads** folder to access pdf and docx file from the user
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

## **5. Create Static Website Hosting Bucket**

- Create **new bucket** for static website hosting (do not start with capital letter of the bucket name)
- I've named `website-hosting-buck`.
- Disable **public access blocking**, because we are testing to access the website directly, u know.
- Check the *acknowledge of turning off the public access blocking*
- Create bucket.
![0030](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0030.png)

### Configure Static Website
- Click on **website-hosting-buck**
- Under **permissions** tab, edit **bucket policy**
- Generate the `getObject` action from policy generator for all user and paste it.
- ### Don't forget to add * behind the / .
- Save changes

![0031](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0031.png)
![0032](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0032.png)

- Under **properties** tab, edit **static website hosting** and **enable** it.
- Add "index.html" in index document
- Save changes
![0033](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0033.png)

## **6. Set Up API Gateway**

- Create **API gateway**  : **Build** HTTP API
- Assign API name "API-to-request-Url"
- Add integrations, choose PresignUrl lambda function
- Configure Post method with path and click next
- After that click create
![0034](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0034.png)
![0035](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0035.png)
![0036](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0036.png)

- Configuring CORS, write "*" and click add
- Follow the instruction as the image and click save
- Under Stages, click default stage and copy the invoke Url to give s3 bucket for upload document file
![0037](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0037.png)
![0038](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0038.png)
---

## **7. Static Website Deployment** 

1. Update `index.html` with your API Gateway endpoint:  
   ```javascript
   const API_URL = "https://your-api-id.execute-api.region.amazonaws.com/presignurl";
   ```
![0039](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0039.png)

2. Upload `index.html` to the static hosting bucket.  
3. Access the site via the bucket’s **Endpoint URL** (e.g., `http://static-website-host.s3-website.region.amazonaws.com`).  

![0040](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0040.png)
![0041](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0041.png)

4. Please make sure you deploy both functions, otherwise you will get the Error message.
![0042](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0042.png)

---

## **8. Configure IAM Permissions**

### For PresignUrl Role
- Click the **PresignUrl** IAM role

![0043](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0043.png)

- Create inline policy and attach the policy to iam role
- Choose the S3 service
- Filter for "putObject" and choose PutObject
- Choose All resource in S3 service
- Name the policy and create policy.

![0044](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0044.png)
![0045](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0045.png)
![0046](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0046.png)

### For ConvertTextToAudio Role

- Add inline policies for **convertTextToAudio** IAM role
- Add `putObject`, `getObject`, `listBucket` Actions and choose all resources for **S3** service
- Add **Polly** service and filter the `SynthesizeSpeech` action allowed for all resources
- Add **SES** service and filter the `SendEmail` action allowed
- Your policy should consist of 5 allow service actions
- Name the policy and create policy.

![0047](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0047.png)
![0048](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0048.png)
![0049](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0049.png)

- Your policy will be look like mine , hope you get it , mate.
![0050](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0050.png)

---

## **9. Final Configuration**

- Under permissions of the convertion bucket, edit **CORS** and paste the ![code](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/Lambda%20-%20Python%20Code/CORS%20for%20S3%20bucket.txt) provided in github repo.
![0051](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0051.png)
![0052](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0052.png)
![0053](https://github.com/pyaesoekyaw/text-to-audio-complete-AWS-PROJECT/blob/main/images/0053.png)

## Verification

- Access the website and add your document and enjoy the process.
- Audio files will arrive into bucket.
- Your email will also receive the audio file from AWS.


---


**License**: MIT  
**Repository**: [github.com/your-repo](https://github.com/pyaesoekyaw)  
