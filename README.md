# E-commerce Automation System

> End-to-end n8n automation for e-commerce order processing with AI-powered email parsing, product lookup, and Monday.com integration

n8n:
<img width="1024" height="273" alt="image" src="https://github.com/user-attachments/assets/16c3cc31-5326-498e-924e-084d78226e14" />

<img width="1438" height="314" alt="image" src="https://github.com/user-attachments/assets/235c0287-542b-4968-b4c1-3ec5e75cecd7" />


Sheets:
<img width="484" height="887" alt="image" src="https://github.com/user-attachments/assets/0a07f119-3769-4121-97b0-5d2a50367420" />

Monday.com:
<img width="1900" height="558" alt="image" src="https://github.com/user-attachments/assets/0324a9c1-3a9c-418f-85ca-bc77692b1de5" />


## 📋 Overview

This project implements an automated system for processing customer orders from emails and managing product data. Built with n8n, it streamlines order management by automatically fetching product data, parsing customer emails with AI, and creating organized tasks in Monday.com.

### Key Features

- 🔄 **Automated Product Data Collection** - Fetches and stores product information from API
- 🤖 **AI-Powered Email Processing** - Uses OpenAI to extract order details from emails
- 🔍 **Smart Product Lookup** - Cross-references product IDs with stored data
- ✅ **Auto Task Creation** - Creates detailed Monday.com tasks with complete order info

## 🏗️ Architecture

The system consists of two interconnected n8n workflows:

1. **Product Data Pipeline** - Collects product data from API and stores in Google Sheets
2. **Email to Monday.com Task** - Processes customer emails and creates actionable tasks

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐
│   API       │────▶│ Google Sheets│◀────│  Email Inbox  │
│  (Products) │     │  (Database)  │     │  (Customer)   │
└─────────────┘     └──────────────┘     └───────────────┘
                           │                      │
                           │                      ▼
                           │              ┌──────────────┐
                           │              │  AI Extract  │
                           │              │   (OpenAI)   │
                           │              └──────────────┘
                           │                      │
                           └──────────┬───────────┘
                                      ▼
                              ┌──────────────┐
                              │  Monday.com  │
                              │    (Tasks)   │
                              └──────────────┘
```

## 🚀 Getting Started

### Prerequisites

- n8n instance (self-hosted or cloud)
- Google Cloud account (for Sheets API)
- OpenAI API key
- Monday.com account with API access
- Gmail account with IMAP enabled

### Installation

1. **Clone this repository**
   ```bash
   git clone (https://github.com/AION-2000/E-commerce-Automation-System)
   cd E-commerce-Automation-System
   ```

2. **Import workflows into n8n**
   - Open your n8n instance
   - Go to Workflows → Import from File
   - Import `Product Data Pipeline.json`
   - Import `Email to Monday.com Task.json`

3. **Configure credentials**
   - Google Sheets OAuth2
   - OpenAI API key
   - Monday.com API token
   - Gmail IMAP credentials

4. **Update configuration**
   - Set your Google Sheet ID in both workflows
   - Configure Monday.com board ID
   - Adjust column mappings as needed

## 📊 Workflow Details

### 1. Product Data Pipeline

**Purpose:** Fetches product data from API and stores it in Google Sheets

**Flow:**
1. Manual trigger initiates the workflow
2. HTTP Request fetches products from DummyJSON API
3. JavaScript code transforms data (ID, Name, Price, Category)
4. Google Sheets node appends all products to spreadsheet

**API Endpoint:** `https://dummyjson.com/products`

### 2. Email to Monday.com Task

**Purpose:** Processes customer order emails and creates Monday.com tasks

**Flow:**
1. IMAP trigger monitors Gmail inbox for new emails
2. JavaScript extracts email subject, body, sender, and date
3. OpenAI AI Agent parses email and structures order data
4. JavaScript converts AI output to JSON and adds ID field
5. Google Sheets lookup retrieves product details by ID
6. Merge node combines order data with product information
7. Edit Fields node prepares data for Monday.com
8. Monday.com node creates task with complete order details

**AI Extraction Fields:**
- Product ID
- Product Name
- Quantity
- Customer Name
- Delivery Address
- Phone Number
- Delivery Date

## 🧪 Testing

### Test Email Format

```
Subject: Urgent Order Request – Red T-Shirt (Product ID 42)

Body:
Hello Team,
I'd like to place an order for 3 units of the Red T-Shirt (Product ID: 42).
Please ensure delivery by October 10, 2025.

Customer: John Doe
Address: 45 Park Street, Dhaka
Phone: +880123456789

Can you confirm once this is scheduled?

Best regards,
John.
```

### Expected Results

✅ Email detected and parsed  
✅ Product ID 42 extracted by AI  
✅ Product details retrieved from Google Sheets  
✅ Monday.com task created with all information  

## 🐛 Known Issues

### 1. Pagination Not Fully Implemented
**Issue:** Product Data Pipeline only fetches the first page  
**Impact:** Limited to ~30 products  
**Fix:** Add loop node to fetch all pages before processing

### 2. Hardcoded Values
**Issue:** Some values in Edit Fields are hardcoded (quantity, date)  
**Impact:** Works for test email but not dynamic  
**Fix:** Replace with dynamic references from AI extraction

### 3. AI Model Version
**Issue:** Uses "gpt-4.1-mini" which may not be valid  
**Impact:** Potential API errors  
**Fix:** Update to correct model name (e.g., "gpt-4-mini")

### 4. Limited Error Handling
**Issue:** No retry logic or failure notifications  
**Impact:** Silent failures possible  
**Fix:** Add error handling nodes and alerts

### 5. Product Name Matching
**Issue:** Only searches by ID, not product name  
**Impact:** Fails if customer doesn't mention ID  
**Fix:** Add fuzzy matching for product names

## 🛡️ Error Handling Strategy

### Current Implementation
- Try-catch blocks in JavaScript nodes
- Console logging for debugging
- "Always Output Data" enabled on critical nodes

### Recommended Improvements
- **API Failures:** Add retry logic with exponential backoff
- **Missing Products:** Create fallback tasks for manual review
- **AI Errors:** Validate output and add confidence checks
- **Monday.com:** Verify board/column IDs before creating items

## 📝 Configuration

### Google Sheets Structure
| ID | Name | Price | Category |
|----|------|-------|----------|
| 1  | Product Name | 99.99 | electronics |

### Monday.com Columns
- `date4` - Delivery Date
- `text_mkwqw2rn` - Order ID
- `text_mkwqqf5h` - Product ID
- `text_mkwqd170` - Product Name
- `text_mkwqzkem` - Product Category
- `text_mkwqr970` - Product Price
- `text_mkwq3bc5` - Quantity
- `text_mkwqb5a9` - Customer Address
- `text_mkwq2ty0` - Phone Number

## 🔧 Technologies Used

- **n8n** - Workflow automation platform
- **OpenAI GPT-4** - AI for email parsing
- **Google Sheets** - Product data storage
- **Monday.com** - Task management
- **Gmail (IMAP)** - Email monitoring
- **DummyJSON API** - Product data source (for demo)

## 💡 Key Learnings

- **Data Flow Management:** Ensuring proper data structure between nodes
- **AI Integration:** Structuring prompts for reliable JSON output
- **API Pagination:** Understanding importance of complete data collection
- **Error Resilience:** Need for comprehensive error handling in production

## 🎯 Future Improvements

- [ ] Implement complete pagination handling
- [ ] Add dynamic value extraction for all fields
- [ ] Support multiple products per order
- [ ] Add webhook triggers for real-time processing
- [ ] Implement advanced error notifications (Slack/Email)
- [ ] Add product name fuzzy matching
- [ ] Create dashboard for order analytics
- [ ] Add order status tracking

## 👤 Author

**Shihab Shahriar Aion**

- GitHub: (https://github.com/AION-2000)
- LinkedIn: (https://www.linkedin.com/in/shihab-shahriar-aion-a1i2o3n4/)

## 🙏 Acknowledgments

- n8n community for excellent documentation
- OpenAI for powerful AI capabilities

---

⭐ If you found this project helpful, please give it a star!

📧 For questions or suggestions, feel free to open an issue.
