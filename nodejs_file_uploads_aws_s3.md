# Node.js File Uploads & Cloud Storage (AWS S3) 📁☁️
## Comprehensive Beginner's Guide

---

## Table of Contents
1. [Introduction & Concepts](#introduction--concepts)
2. [Environment Setup](#environment-setup)
3. [Basic File Upload with Multer](#basic-file-upload-with-multer)
4. [AWS S3 Integration](#aws-s3-integration)
5. [Complete Working Example](#complete-working-example)
6. [Best Practices & Security](#best-practices--security)
7. [Troubleshooting](#troubleshooting)
8. [Practice Exercises](#practice-exercises)

---

## 1. Introduction & Concepts

### What is File Upload? 🤔
File upload allows users to transfer files from their device to a server. It's like sending an email attachment.

### File Upload Flow
```mermaid
graph TD
    A[User Selects File] -->|HTTP POST| B[Server Receives File]
    B -->|Validate & Process| C{Storage Choice}
    C -->|Local| D[Server Disk]
    C -->|Cloud| E[AWS S3]
    E -->|Return URL| F[File Accessible Globally]
    D -->|Limited Access| G[Server Only]
```

### Why Use Cloud Storage?
- **Unlimited Storage**: No server disk limits
- **Global Access**: Files available worldwide
- **Reliability**: Automatic backups
- **Scalability**: Handles traffic spikes

### Key Technologies
- **Multer**: Handles file uploads in Express
- **AWS S3**: Cloud storage service
- **Express**: Web framework for Node.js

---

## 2. Environment Setup

### Prerequisites Checklist ✅
- Node.js (v14+) installed
- Text editor (VS Code recommended)
- AWS account (free tier available)
- Basic JavaScript knowledge

### Project Initialization
```bash
# Create project
mkdir file-upload-demo && cd file-upload-demo
npm init -y

# Install dependencies
npm install express multer aws-sdk dotenv
npm install --save-dev nodemon

# Create structure
mkdir uploads public
touch app.js .env .gitignore
```

### Package.json Scripts
```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  }
}
```

### Environment Variables (.env)
```env
# AWS Configuration
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=us-east-1
AWS_BUCKET_NAME=your-bucket-name

# Server Configuration
PORT=3000
```

### .gitignore
```
node_modules/
.env
uploads/
*.log
```

---

## 3. Basic File Upload with Multer

### Understanding Multer 📦
Multer is middleware that processes `multipart/form-data` (file uploads). It acts as a file handler for Express.

### Simple File Upload Setup
```javascript
// app.js - Basic Setup
const express = require('express');
const multer = require('multer');
const path = require('path');

const app = express();

// Configure storage
const storage = multer.diskStorage({
    destination: 'uploads/',
    filename: (req, file, cb) => {
        const uniqueName = Date.now() + '-' + file.originalname;
        cb(null, uniqueName);
    }
});

// Create upload middleware
const upload = multer({ 
    storage,
    limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
    fileFilter: (req, file, cb) => {
        // Accept only images
        if (file.mimetype.startsWith('image/')) {
            cb(null, true);
        } else {
            cb(new Error('Only images allowed!'));
        }
    }
});

// Routes
app.get('/', (req, res) => {
    res.send(`
        <form action="/upload" method="post" enctype="multipart/form-data">
            <input type="file" name="file" accept="image/*" required>
            <button type="submit">Upload</button>
        </form>
    `);
});

app.post('/upload', upload.single('file'), (req, res) => {
    if (!req.file) {
        return res.status(400).json({ error: 'No file uploaded' });
    }
    
    res.json({
        message: 'Upload successful!',
        filename: req.file.filename,
        size: req.file.size
    });
});

app.listen(3000, () => console.log('Server running on http://localhost:3000'));
```

### Test Local Upload
```bash
npm run dev
# Visit http://localhost:3000 and upload an image
```

---

## 4. AWS S3 Integration

### S3 Concepts Explained 📚

```mermaid
graph TD
    A[AWS Account] --> B[S3 Service]
    B --> C[Bucket Container]
    C --> D[Objects Files]
    C --> E[Folders Prefixes]
    D --> F[Public URLs]
    D --> G[Metadata]
```

### AWS Setup Steps

#### 1. Create S3 Bucket
1. Login to AWS Console → Search "S3"
2. Click "Create bucket"
3. Bucket name: `my-app-uploads-2024` (must be globally unique)
4. Region: Choose closest to users
5. Uncheck "Block all public access"
6. Create bucket

#### 2. Create IAM User (Security)
1. AWS Console → IAM → Users → Add user
2. Username: `file-upload-user`
3. Access type: Programmatic access
4. Attach policy: `AmazonS3FullAccess`
5. **Save credentials immediately!**

#### 3. Configure Bucket Policy
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::your-bucket-name/*"
        }
    ]
}
```

### S3 Upload Implementation

#### AWS Configuration
```javascript
// config/aws.js
const AWS = require('aws-sdk');
require('dotenv').config();

AWS.config.update({
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
    region: process.env.AWS_REGION
});

const s3 = new AWS.S3();
module.exports = s3;
```

#### Upload Function
```javascript
// utils/s3Upload.js
const s3 = require('../config/aws');

const uploadToS3 = async (fileBuffer, fileName, mimetype) => {
    const params = {
        Bucket: process.env.AWS_BUCKET_NAME,
        Key: `uploads/${Date.now()}-${fileName}`,
        Body: fileBuffer,
        ContentType: mimetype,
        ACL: 'public-read'
    };

    return await s3.upload(params).promise();
};

module.exports = { uploadToS3 };
```

---

## 5. Complete Working Example

### Full Application Code
```javascript
// app.js - Complete S3 Integration
const express = require('express');
const multer = require('multer');
const { uploadToS3 } = require('./utils/s3Upload');
require('dotenv').config();

const app = express();

// Use memory storage for S3 uploads
const upload = multer({ 
    storage: multer.memoryStorage(),
    limits: { fileSize: 10 * 1024 * 1024 },
    fileFilter: (req, file, cb) => {
        const allowed = ['image/jpeg', 'image/png', 'image/gif'];
        if (allowed.includes(file.mimetype)) {
            cb(null, true);
        } else {
            cb(new Error('Invalid file type'));
        }
    }
});

// Routes
app.get('/', (req, res) => {
    res.send(`
        <!DOCTYPE html>
        <html>
        <head><title>File Upload to S3</title></head>
        <body>
            <h1>Upload to AWS S3</h1>
            <form action="/upload-s3" method="post" enctype="multipart/form-data">
                <input type="file" name="file" accept="image/*" required>
                <button type="submit">Upload to S3</button>
            </form>
            <div id="result"></div>
        </body>
        </html>
    `);
});

app.post('/upload-s3', upload.single('file'), async (req, res) => {
    try {
        if (!req.file) {
            return res.status(400).json({ error: 'No file provided' });
        }

        const result = await uploadToS3(
            req.file.buffer,
            req.file.originalname,
            req.file.mimetype
        );

        res.json({
            success: true,
            message: 'File uploaded to S3!',
            fileUrl: result.Location,
            fileName: result.Key
        });

    } catch (error) {
        res.status(500).json({ 
            success: false, 
            error: error.message 
        });
    }
});

// Error handling
app.use((err, req, res, next) => {
    if (err instanceof multer.MulterError) {
        return res.status(400).json({ error: err.message });
    }
    res.status(500).json({ error: err.message });
});

app.listen(process.env.PORT || 3000, () => {
    console.log('🚀 Server running on http://localhost:3000');
});
```

### Testing Your Upload
```bash
# Start server
npm run dev

# Test with curl
curl -X POST -F "file=@image.jpg" http://localhost:3000/upload-s3
```

---

## 6. Best Practices & Security

### Security Essentials 🔒

#### File Validation
```javascript
const validateFile = (file) => {
    // Size check
    if (file.size > 10 * 1024 * 1024) {
        throw new Error('File too large');
    }
    
    // Type check
    const allowedTypes = ['image/jpeg', 'image/png'];
    if (!allowedTypes.includes(file.mimetype)) {
        throw new Error('Invalid file type');
    }
    
    // Extension check
    const allowedExts = ['.jpg', '.jpeg', '.png'];
    const ext = path.extname(file.originalname).toLowerCase();
    if (!allowedExts.includes(ext)) {
        throw new Error('Invalid file extension');
    }
};
```

#### Secure File Names
```javascript
const crypto = require('crypto');

const generateSecureFileName = (originalName) => {
    const ext = path.extname(originalName);
    const randomName = crypto.randomBytes(16).toString('hex');
    return `${Date.now()}-${randomName}${ext}`;
};
```

#### Environment Security
```javascript
// Never hardcode credentials
❌ const accessKey = "AKIAIOSFODNN7EXAMPLE";

// Always use environment variables
✅ const accessKey = process.env.AWS_ACCESS_KEY_ID;
```

### Performance Tips ⚡

1. **Use Memory Storage**: For S3 uploads, don't save to disk first
2. **Set Reasonable Limits**: Prevent abuse with file size limits
3. **Implement Compression**: Resize images before upload
4. **Use CDN**: CloudFront for faster global access

---

## 7. Troubleshooting

### Common Issues & Solutions

| Problem | Cause | Solution |
|---------|-------|----------|
| "Access Denied" | Wrong AWS credentials | Check `.env` file credentials |
| "Bucket not found" | Incorrect bucket name | Verify bucket name in AWS Console |
| "File too large" | Exceeds size limit | Adjust multer limits or compress file |
| "CORS error" | Browser security | Configure bucket CORS policy |

### Debug Tips 🔍
```javascript
// Debug AWS config
console.log('AWS Region:', process.env.AWS_REGION);
console.log('Bucket:', process.env.AWS_BUCKET_NAME);

// Debug file info
console.log('File info:', {
    name: req.file.originalname,
    size: req.file.size,
    type: req.file.mimetype
});
```

---

## 8. Practice Exercises

### Exercise 1: Basic Upload (Beginner)
- Create file upload form
- Validate file types (images only)
- Display success/error messages

### Exercise 2: Multiple Files (Intermediate)
- Allow multiple file selection
- Upload all files to S3
- Show progress for each file

### Exercise 3: Advanced Features (Advanced)
- Image resizing before upload
- File organization by date/user
- Delete files from S3

### Exercise 4: Production Ready (Expert)
- User authentication
- File access permissions
- Database integration
- Error logging

---

## Quick Reference 📋

### Essential Commands
```bash
# Setup
npm install express multer aws-sdk dotenv

# Test upload
curl -X POST -F "file=@test.jpg" localhost:3000/upload
```

### Key Code Patterns
```javascript
// Multer setup
const upload = multer({ storage: multer.memoryStorage() });

// S3 upload
const result = await s3.upload(params).promise();

// Error handling
app.use((err, req, res, next) => { /* handle errors */ });
```

### Environment Template
```env
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=secret...
AWS_REGION=us-east-1
AWS_BUCKET_NAME=my-bucket
PORT=3000
```

---

## Summary 🎯

**What You've Learned:**
- ✅ File upload fundamentals
- ✅ Multer configuration and usage
- ✅ AWS S3 setup and integration
- ✅ Security best practices
- ✅ Error handling and debugging

**Next Steps:**
1. Practice with exercises
2. Learn API testing (next topic)
3. Explore advanced S3 features
4. Build a complete file management system

**Key Takeaways:**
- Always validate files before upload
- Use environment variables for credentials
- Memory storage is better for cloud uploads
- Implement proper error handling

Happy coding! 🚀