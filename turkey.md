# Turkey Visa Application System

This document contains all Turkey-specific implementation details, API endpoints, and usage instructions for the Turkey visa application system.

## 📋 Application Workflow

### Step 1: Start Application

- Select passport country from supported countries
- Enter email address
- Generate unique application ID
- Send confirmation email with resume link

### Step 2: Applicant Details

- Personal information (name, DOB, place of birth, parents' names)
- Passport details (number, issue/expiry dates)
- Automatic validation rules (age ≥ 18, passport expiry ≥ 6 months beyond journey)

### Step 3: Document Upload

- Supporting documents (visa/residence permit from Schengen countries)
- Document validation (expiry dates, issuing countries)
- Passport upload
- Additional document support

### Step 4: Add Additional Applicants

- Add multiple applicants under the same application
- Same validation rules apply to all applicants
- Group application processing

## 🛠 API Endpoints

### Base URLs

- **Global Document Service**: `/api/v1/`
- **Turkey API**: `/api/v1/turkey/`

### Global Document Upload Endpoints

#### POST `/api/v1/single/document`

Upload a single document file.

**Request:** `multipart/form-data`

- `document`: File (PDF, JPG, JPEG, PNG)

**Response:**

```json
{
  "success": true,
  "message": "Document uploaded successfully",
  "data": {
    "name": "document.pdf",
    "url": "https://res.cloudinary.com/.../document.pdf",
    "publicId": "document_1234567890_abc123",
    "uploadedAt": "2024-01-15T10:30:00.000Z",
    "size": 245760,
    "format": "pdf"
  }
}
```

#### POST `/api/v1/multiple/document`

Upload multiple document files with optional folder organization.

**Request:** `multipart/form-data`

- `documents`: Files (PDF, JPG, JPEG, PNG) - max 10 files
- `folder`: Optional folder path for organization

**Response:**

```json
{
  "success": true,
  "message": "3 documents uploaded successfully",
  "data": [
    {
      "name": "passport.pdf",
      "url": "https://res.cloudinary.com/.../turkey_visa/TUR-A1B2C3D4/passport.pdf",
      "publicId": "turkey_visa/TUR-A1B2C3D4/passport_1234567890_abc123",
      "uploadedAt": "2024-01-15T10:30:00.000Z",
      "size": 245760,
      "format": "pdf"
    }
  ],
  "count": 3
}
```

#### DELETE `/api/v1/document/:publicId`

Delete a document from Cloudinary.

#### GET `/api/v1/document/:publicId`

Get document information from Cloudinary.

### Turkey-Specific Endpoints

#### GET `/api/v1/turkey/visa-fee`

Get visa fee information for supported countries.

**Query Parameters:**

- `country` (optional): Filter by specific country

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "country": "India",
      "visaFee": 49,
      "duration": "30 Days",
      "numberOfEntries": "Single-Entry",
      "serviceFee": 35,
      "currency": "USD"
    }
  ]
}
```

#### GET `/countries`

Get list of all supported countries.

**Response:**

```json
{
  "success": true,
  "data": ["India", "Australia", "China", ...],
  "count": 45
}
```

#### POST `/start`

Start a new visa application.

**Request Body:**

```json
{
  "passportCountry": "India",
  "visaType": "Electronic Visa",
  "destination": "Turkey",
  "email": "applicant@example.com"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Application started successfully",
  "data": {
    "applicationId": "TUR-A1B2C3D4",
    "email": "applicant@example.com",
    "status": "started",
    "currentStep": 1,
    "nextStep": "applicant-details",
    "estimatedTotalFee": 84
  }
}
```

#### POST `/applicant-details`

Save main applicant details.

**Request Body:**

```json
{
  "applicationId": "TUR-A1B2C3D4",
  "applicantDetails": {
    "givenNames": "John",
    "surname": "Doe",
    "dateOfBirth": "1990-01-15",
    "placeOfBirth": "Mumbai",
    "motherName": "Jane Doe",
    "fatherName": "Robert Doe",
    "passportNumber": "P1234567",
    "passportIssueDate": "2020-01-15",
    "passportExpiryDate": "2030-01-15"
  }
}
```

#### POST `/documents`

Upload and register applicant documents for the visa application.

**Request Body:**

```json
{
  "applicationId": "TUR-A1B2C3D4",
  "documents": {
    "supportingDocuments": [
      {
        "documentType": "Visa",
        "issuingCountry": "Germany",
        "documentNumber": "V123456789",
        "expiryDate": "2026-12-31",
        "isUnlimited": false
      }
    ],
    "additionalDocuments": [
      {
        "name": "Employment Letter",
        "url": "https://res.cloudinary.com/your-cloud/turkey_visa/TUR-A1B2C3D4/employment_letter.pdf",
        "publicId": "turkey_visa/TUR-A1B2C3D4/employment_letter_123456",
        "uploadedAt": "2024-01-15T10:30:00.000Z"
      }
    ]
  }
}
```

**Response (Success):**

```json
{
  "success": true,
  "message": "Documents uploaded successfully",
  "data": {
    "applicationId": "TUR-A1B2C3D4",
    "status": "documents_completed",
    "currentStep": 4,
    "nextStep": "add-applicant",
    "documentsCount": {
      "supportingDocuments": 1,
      "additionalDocuments": 1
    },
    "totalDocuments": 2
  }
}
```

#### 🆕 **New Document Upload Architecture**

The system now uses a two-step process for better scalability:

##### **Step 1: Upload Files to Global Service**

**Single File Upload:**

```bash
POST /api/v1/single/document
Content-Type: multipart/form-data

# Form data: 'document' (file)
```

**Response:**

```json
{
  "success": true,
  "message": "Document uploaded successfully",
  "data": {
    "name": "Employment Letter.pdf",
    "url": "https://res.cloudinary.com/.../employment_letter.pdf",
    "publicId": "employment_letter_1705312200000_12345",
    "uploadedAt": "2024-01-15T10:30:00.000Z",
    "size": 245760,
    "format": "pdf"
  }
}
```

**Multiple Files Upload:**

```bash
POST /api/v1/multiple/document
Content-Type: multipart/form-data

# Form data: 'documents' (multiple files)
# Optional: 'folder' (string) for organization
```

**Response:**

```json
{
  "success": true,
  "message": "2 documents uploaded successfully",
  "data": [
    {
      "name": "Employment Letter.pdf",
      "url": "https://res.cloudinary.com/.../employment_letter.pdf",
      "publicId": "turkey_visa/TUR-A1B2C3D4/employment_letter_123456",
      "uploadedAt": "2024-01-15T10:30:00.000Z",
      "size": 245760,
      "format": "pdf"
    },
    {
      "name": "Bank Statement.pdf",
      "url": "https://res.cloudinary.com/.../bank_statement.pdf",
      "publicId": "turkey_visa/TUR-A1B2C3D4/bank_statement_789012",
      "uploadedAt": "2024-01-15T10:35:00.000Z",
      "size": 189440,
      "format": "pdf"
    }
  ],
  "count": 2
}
```

##### **Step 2: Register Documents with Application**

After uploading files, use the returned metadata in the document registration:

```javascript
const completeDocumentUpload = async (applicationId, files) => {
  try {
    // Step 1: Upload files to global service
    const formData = new FormData();
    files.forEach((file) => formData.append('documents', file));
    formData.append('folder', `turkey_visa/${applicationId}`);

    const uploadResponse = await fetch('/api/v1/multiple/document', {
      method: 'POST',
      body: formData,
    });

    const uploadResult = await uploadResponse.json();

    // Step 2: Register with application
    const documents = {
      supportingDocuments: [
        {
          documentType: 'Visa',
          issuingCountry: 'Germany',
          documentNumber: 'V123456789',
          expiryDate: '2026-12-31',
          isUnlimited: false,
        },
      ],
      additionalDocuments: uploadResult.data,
    };

    const registerResponse = await fetch('/api/v1/turkey/documents', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        applicationId,
        documents,
      }),
    });

    return await registerResponse.json();
  } catch (error) {
    console.error('Document upload failed:', error);
    throw error;
  }
};
```

##### **Benefits of This Architecture:**

- **Separation of Concerns**: File upload logic is separate from document registration
- **Reusability**: Global upload service can be used by all countries
- **Better Error Handling**: Failed uploads don't affect document registration
- **Scalability**: Easy to add new upload features without modifying country-specific logic
- **Performance**: Reduced payload size in document registration requests

#### POST `/add-applicant`

Add additional applicants to the application.

**Request Body:**

```json
{
  "applicationId": "TUR-A1B2C3D4",
  "applicant": {
    "givenNames": "Jane",
    "surname": "Doe",
    "dateOfBirth": "1992-03-20",
    "placeOfBirth": "Mumbai",
    "motherName": "Mary Doe",
    "fatherName": "Robert Doe",
    "passportNumber": "P7654321",
    "passportIssueDate": "2021-03-20",
    "passportExpiryDate": "2031-03-20",
    "documents": {
      "supportingDocuments": [],
      "additionalDocuments": []
    }
  }
}
```

#### POST `/submit`

Submit the complete application for processing.

**Request Body:**

```json
{
  "applicationId": "TUR-A1B2C3D4"
}
```

#### GET `/application/:applicationId`

Get application details by ID.

**Query Parameters:**

- `email` (optional): Verify application ownership

## 📊 Database Schema

### TurkeyApplication Model

- **applicationId**: Unique 8-character ID (TUR-XXXXXXX)
- **passportCountry**: Applicant's passport country
- **email**: Applicant's email address
- **mainApplicant**: Main applicant details and documents
- **additionalApplicants**: Array of additional applicants
- **status**: Application status (draft, started, applicant_details_completed, documents_completed, submitted, processing, approved, rejected)
- **currentStep**: Current step in application process (1-4)
- **visaFee**: Visa fee for the passport country
- **serviceFee**: Fixed service fee ($35)
- **totalFee**: Calculated total fee

### TurkeyVisaFee Model

- **country**: Country name
- **visaFee**: Visa fee amount
- **duration**: Visa validity period (30 Days / 90 Days)
- **numberOfEntries**: Single-Entry / Multiple-Entry
- **serviceFee**: Fixed service fee ($35)
- **isActive**: Whether the fee is currently active

## 🔒 Validation Rules

### Passport Validation

- Must be at least 18 years old
- Passport expiry must be at least 6 months beyond intended journey date
- Passport number format validation

### Document Validation

- Supporting documents must be from Schengen countries (if applicable)
- Document expiry dates must be valid (unless marked as unlimited)
- File size limits (5MB default)
- Supported file types: PDF, JPG, JPEG, PNG

### Email Validation

- Standard email format validation
- Unique application per email (draft applications only)

## 💳 PayPal Payment Integration

The Turkey visa system includes integrated PayPal payment processing for secure online payments. This section covers the complete payment flow and API usage.

### Payment Flow Overview

1. **Application Ready**: User completes all application steps (applicant details, documents)
2. **Create Order**: Backend creates PayPal order with total amount
3. **PayPal Redirect**: User redirected to PayPal for payment
4. **Payment Approval**: User approves payment on PayPal
5. **Return to Application**: User redirected back to application
6. **Capture Payment**: Backend captures the approved payment
7. **Application Paid**: Application status updated to 'paid'

### PayPal Payment Endpoints

#### POST `/api/v1/payment/paypal/create`

Creates a new PayPal order for visa application payment.

**Request Body:**

```json
{
  "applicationId": "TUR-AD2U4MJ5",
  "amount": 84,
  "currency": "USD",
  "description": "Turkey Visa Application Payment"
}
```

**Response:**

```json
{
  "success": true,
  "message": "PayPal order created successfully",
  "data": {
    "paymentId": "PAY-A1B2C3D4E",
    "orderId": "5O190127TN364715T",
    "approvalUrl": "https://www.sandbox.paypal.com/checkoutnow?token=5O190127TN364715T",
    "status": "CREATED",
    "amount": 84,
    "currency": "USD"
  }
}
```

**Error Response:**

```json
{
  "success": false,
  "error": {
    "code": "PAYMENT_ERROR",
    "message": "Application not found"
  }
}
```

#### POST `/api/v1/payment/paypal/capture`

Captures an approved PayPal payment order.

**Request Body:**

```json
{
  "orderId": "5O190127TN364715T",
  "applicationId": "TUR-AD2U4MJ5"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Payment captured successfully",
  "data": {
    "paymentId": "PAY-A1B2C3D4E",
    "orderId": "5O190127TN364715T",
    "transactionId": "8AC96375WN7079234",
    "status": "COMPLETED",
    "amount": 84,
    "currency": "USD",
    "payerEmail": "buyer@example.com",
    "capturedAt": "2025-09-04T10:30:00.000Z"
  }
}
```

#### GET `/api/v1/payment/:paymentId`

Retrieves payment status and details.

**Response:**

```json
{
  "success": true,
  "data": {
    "paymentId": "PAY-A1B2C3D4E",
    "applicationId": "TUR-AD2U4MJ5",
    "orderId": "5O190127TN364715T",
    "transactionId": "8AC96375WN7079234",
    "status": "COMPLETED",
    "amount": 84,
    "currency": "USD",
    "payerEmail": "buyer@example.com",
    "paypalStatus": "APPROVED",
    "createdAt": "2025-09-04T10:25:00.000Z",
    "updatedAt": "2025-09-04T10:30:00.000Z"
  }
}
```

#### POST `/api/v1/payment/paypal/webhook`

Handles PayPal webhook events for payment confirmations. This endpoint is called automatically by PayPal.

**Headers:**

```
paypal-transmission-id: 69cd13f0-d67e-11e5-baa3-778b53f4ae55
paypal-transmission-time: 2016-06-02T22:33:33Z
paypal-transmission-sig: 6Gq3NpZfzHXr7vEo6CD6gHKFBH4R...
paypal-cert-url: https://api.paypal.com/v1/notifications/certs/CERT-360caa42-fca2a594-a5cafa
paypal-auth-algo: SHA256withRSA
```

**Response:**

```json
{
  "success": true,
  "message": "Webhook processed successfully"
}
```

#### POST `/api/v1/payment/refund`

Processes a refund for a completed payment.

**Request Body:**

```json
{
  "paymentId": "PAY-A1B2C3D4E",
  "amount": 84,
  "reason": "Customer requested refund"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Payment refunded successfully",
  "data": {
    "paymentId": "PAY-A1B2C3D4E",
    "refundAmount": 84,
    "refundId": "8AC96375WN7079245",
    "refundedAt": "2025-09-04T11:00:00.000Z"
  }
}
```

### Payment Status Values

- **CREATED**: PayPal order created, waiting for user approval
- **APPROVED**: User approved payment on PayPal
- **COMPLETED**: Payment successfully captured
- **FAILED**: Payment failed or was denied
- **REFUNDED**: Payment was refunded
- **CANCELLED**: Payment was cancelled

### Frontend Integration Examples

#### Web (Next.js/React)

```javascript
// 1. Create PayPal order
const createOrder = async (applicationId, amount) => {
  const response = await fetch('/api/v1/payment/paypal/create', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      applicationId,
      amount: 84,
      currency: 'USD',
    }),
  });

  const data = await response.json();

  if (data.success) {
    // Redirect to PayPal
    window.location.href = data.data.approvalUrl;
  }
};

// 2. Capture payment (after PayPal redirect)
const capturePayment = async (orderId, applicationId) => {
  const response = await fetch('/api/v1/payment/paypal/capture', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      orderId,
      applicationId,
    }),
  });

  const data = await response.json();
  return data;
};
```

#### Mobile (React Native Expo)

```javascript
// Using WebView for PayPal approval
const handlePayment = async () => {
  const createResponse = await fetch('/api/v1/payment/paypal/create', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      applicationId: 'TUR-AD2U4MJ5',
      amount: 84,
      currency: 'USD',
    }),
  });

  const createData = await createResponse.json();

  if (createData.success) {
    // Open PayPal in WebView
    setShowWebView(true);
    setPaypalUrl(createData.data.approvalUrl);
    setOrderId(createData.data.orderId);
  }
};

// Handle PayPal redirect back to app
const handleNavigationStateChange = (navState) => {
  const { url } = navState;

  if (url.includes('success')) {
    // Capture payment
    capturePayment(orderId, applicationId);
    setShowWebView(false);
  } else if (url.includes('cancel')) {
    setShowWebView(false);
  }
};
```

### PayPal Configuration

#### Environment Variables

```env
# PayPal Configuration
PAYPAL_MODE=sandbox                    # 'sandbox' or 'production'
PAYPAL_CLIENT_ID=your_paypal_client_id
PAYPAL_CLIENT_SECRET=your_paypal_client_secret
PAYPAL_WEBHOOK_ID=your_paypal_webhook_id
```

#### Webhook Setup

1. **Sandbox**: Configure webhook URL in PayPal Developer Dashboard
2. **Production**: Configure webhook URL in PayPal Business Account
3. **Webhook URL**: `https://yourdomain.com/api/v1/payment/paypal/webhook`
4. **Events to Subscribe**:
   - `PAYMENT.CAPTURE.COMPLETED`
   - `PAYMENT.CAPTURE.DENIED`
   - `PAYMENT.CAPTURE.REFUNDED`

### Error Handling

#### Common Error Scenarios

**Application Not Ready:**

```json
{
  "success": false,
  "error": {
    "code": "PAYMENT_ERROR",
    "message": "Application is not ready for payment"
  }
}
```

**Payment Already Exists:**

```json
{
  "success": false,
  "error": {
    "code": "PAYMENT_ERROR",
    "message": "Payment already exists for this application"
  }
}
```

**PayPal API Error:**

```json
{
  "success": false,
  "error": {
    "code": "PAYPAL_ERROR",
    "message": "Failed to create PayPal order"
  }
}
```

### Testing PayPal Integration

#### Sandbox Testing

1. **Set Environment**: `PAYPAL_MODE=sandbox`
2. **Test Accounts**: Use PayPal Developer sandbox accounts
3. **Test Cards**: Use sandbox test payment methods
4. **Webhook Testing**: Use ngrok for local webhook testing

#### Production Setup

1. **Set Environment**: `PAYPAL_MODE=production`
2. **Live Credentials**: Use production PayPal credentials
3. **SSL Required**: HTTPS required for production webhooks
4. **Domain Verification**: Verify domain ownership for webhooks

### Payment Security Features

- **Server-to-Server**: All PayPal API calls happen server-side
- **Webhook Verification**: Signature verification for webhook authenticity
- **Idempotent Operations**: Prevents duplicate orders and captures
- **Input Validation**: Comprehensive validation using Zod schemas
- **Error Logging**: All payment errors are logged for debugging

## 📧 Email Notifications

The system sends automated emails for:

- **Application Started**: Confirmation with application ID and resume link
- **Application Completed**: Submission confirmation with processing details

Emails are sent using configurable providers (Gmail, SMTP, or Ethereal for testing).

## 🧪 Testing the API

### Health Check

```bash
curl http://localhost:3000/health
```

### Get Supported Countries

```bash
curl http://localhost:3000/api/v1/turkey/countries
```

### Get Visa Fees

```bash
curl "http://localhost:3000/api/v1/turkey/visa-fee?country=India"
```

## 🚀 Turkey-Specific Features

### Supported Countries

The system supports 45 countries with their specific visa fees:

**Free Visas (Service Fee Only - $35):**

- Iraq, Mexico, South Africa, Taiwan

**Popular Destinations:**

- India: $49 (30 Days, Single-Entry)
- China: $66 (90 Days, Multiple-Entry)
- Australia: $66 (90 Days, Multiple-Entry)
- Pakistan: $66 (30 Days, Single-Entry)

### Application Flow

1. **Start**: User selects country and provides email
2. **Details**: Personal and passport information
3. **Documents**: Upload supporting documents and passport
4. **Additional**: Add family members or group applicants
5. **Submit**: Final submission for processing

### Fee Calculation

- **Visa Fee**: Country-specific (from seed data)
- **Service Fee**: Fixed $35 per applicant
- **Total Fee**: (Visa Fee + Service Fee) × Number of Applicants

### Status Tracking

- `draft`: Initial state
- `started`: Application created
- `applicant_details_completed`: Personal details entered
- `documents_completed`: Documents uploaded
- `submitted`: Ready for processing
- `processing`: Under review
- `approved`: Application approved
- `rejected`: Application rejected

## 🛠 Turkey-Specific Scripts

### Seed Database

```bash
npm run seed
```

Populates the database with visa fees for all 45 supported countries.

### Development

```bash
npm run dev
```

## 📋 File Structure (Turkey)

```
backend/
├── models/
│   ├── TurkeyApplication.js      # Main application model
│   └── TurkeyVisaFee.js         # Visa fee model
├── controllers/
│   └── turkeyVisaController.js  # Turkey-specific controllers
├── routes/
│   └── turkeyVisa.js           # Turkey API routes
├── utils/
│   ├── application.js          # Application utilities
│   └── validation.js           # Turkey validation schemas
└── scripts/
    └── seedTurkeyVisaFees.js   # Database seeding
```

## 🔧 Configuration

### Environment Variables (Turkey-specific)

```env
# Turkey Application Settings
TURKEY_VISA_TYPES=Electronic Visa
TURKEY_DESTINATION=Turkey
TURKEY_SERVICE_FEE=35
TURKEY_MAX_APPLICANTS=10
TURKEY_SUPPORTED_COUNTRIES=45
```

### Email Templates

- `application-started.html`: Welcome email with application ID
- `application-completed.html`: Submission confirmation

## 🚀 Future Enhancements

- **Document Verification**: AI-powered document validation
- **Payment Integration**: Stripe integration for fee collection
- **SMS Notifications**: Additional notification channels
- **Real-time Status**: WebSocket updates for application status
- **Bulk Applications**: Corporate/group application handling

## 📞 Support

For Turkey-specific issues or questions:

- Check the application logs in `logs/` directory
- Verify database connectivity
- Ensure all environment variables are set
- Test API endpoints with the provided curl examples

---

**Turkey Visa Application System - Ready for Production** 🚀
