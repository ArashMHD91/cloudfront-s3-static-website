# Hosting a Static Website Using Route 53 and Enhancing It with SSL & CloudFront

![Image](https://github.com/user-attachments/assets/f62b9ef9-208a-4433-ba25-338747cb83c4)

## Architecture Diagram

```
AWS Cloud
├── Amazon S3
│   ├── Public Bucket (Static Website Hosting) - arashcloud.engineer
│   ├── Private Bucket (For CloudFront) - random-s3-bucket
│   ├── Static Website Files (HTML, CSS, JS)
│
├── Amazon Route 53
│   ├── Hosted Zone - arashcloud.engineer
│   ├── A Record (Alias to S3 Website Endpoint)
│   ├── A Record (Alias to CloudFront Distribution)
│
├── AWS Certificate Manager (ACM)
│   ├── SSL Certificate for arashcloud.engineer and www.arashcloud.engineer
│
├── Amazon CloudFront
│   ├── Distribution (Pointing to Private S3 Bucket)
│   ├── Custom Domain with SSL Certificate
│   ├── Redirect HTTP to HTTPS
```

## Part A: Hosting a Static Website Using Only Route 53

### 1. Create an S3 Bucket
- Navigate to **Amazon S3** in the AWS console.
- Click **Create Bucket** and set the name as your domain (e.g., `arashcloud.engineer`).
- Enable **ACLs** and uncheck **Block all public access**.
- Enable **Bucket Versioning**.
- Enable **Static Website Hosting**:
  - Set **Index Document** to `index.html`.
- Save changes.

### 2. Update Bucket Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
```

### 3. Upload Static Website Files
- Go to **Objects** → **Upload**.
- Add your `index.html`, `CSS`, and `JS` files.
- Set **Predefined ACLs** to **Grant public-read access**.
- Click **Upload**.

### 4. Configure Route 53
- Navigate to **Route 53** → **Hosted Zones**.
- Create an **A Record**:
  - **Alias**: Enabled.
  - **Route traffic to**: **Alias to S3 website endpoint**.
  - **Region**: Select the same as your S3 bucket.
  - **S3 Endpoint**: Choose your bucket’s website endpoint.
  - Click **Create Record**.

Your website should now be accessible via your damain name, in my case `http://arashcloud.engineer`.

![Image](https://github.com/user-attachments/assets/f97ba9b5-51ae-4c0b-b40f-a22d0872a17b)

**Limitations:**
- No **HTTPS support**.
- No **DDoS protection**.
- Not ideal for production.

---

## Part B: Securing the Website with SSL and CloudFront

### 1. Create a New S3 Bucket (For CloudFront)
- Follow **Step 1** from Part A but use a **random bucket name** (not your domain) and upload static website content

### 2. Request an SSL Certificate in ACM
- Navigate to **AWS Certificate Manager (ACM)**.
- Click **Request a Certificate** → **Request a public certificate**.
- Add:
  - `arashcloud.engineer`
  - `www.arashcloud.engineer`
- Ensure the **region is `us-east-1` (N. Virginia)`** (Required for CloudFront).
- Click **Create records in Route 53** and wait for validation.

### 3. Create a CloudFront Distribution
- Navigate to **CloudFront** → **Create Distribution**.
- Set **Origin Domain Name** to your **private S3 bucket endpoint**.
- **Viewer Protocol Policy**: Redirect HTTP to HTTPS.
- **Alternate Domain Names (CNAMEs)**:
  - `arashcloud.engineer`
  - `www.arashcloud.engineer`
- **Custom SSL Certificate**: Select the one from ACM.
- **Default Root Object**: `index.html`.
- Click **Create Distribution**.

Once the status is **Enabled**, copy the **CloudFront Domain Name** (e.g., `d5a6l9vfkf5h0.cloudfront.net`).

### 4. Update Route 53 to Point to CloudFront
- Navigate to **Route 53** → **Hosted Zones**.
- Create a new **A Record**:
  - **Alias**: Enabled.
  - **Route traffic to**: **Alias to CloudFront distribution**.
  - **CloudFront Domain Name**: Paste from previous step.
  - Click **Create Record**.
![Image](https://github.com/user-attachments/assets/40baae7a-3692-4ad6-a628-da6659a87851)

### 5. Verify Setup
- Run:
  ```sh
  nslookup arashcloud.engineer
  ```
- Visit `https://arashcloud.engineer` and `https://www.arashcloud.engineer` to confirm HTTPS is working.

![Image](https://github.com/user-attachments/assets/f62b9ef9-208a-4433-ba25-338747cb83c4)

---

## Summary
- **Initial Setup**: Hosted the website using **S3 and Route 53**.
- **Security & Performance Upgrade**: Implemented **CloudFront and SSL**.
- **Final Result**: The website is now accessible via **HTTPS with improved security and performance**.

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

This project is part of my **AWS and DevOps Learning Journey**. Feel free to contribute or suggest improvements!

