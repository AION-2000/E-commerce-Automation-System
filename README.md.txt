E-commerce Automation System - README
Project Name: Ecommerce_automation
Submitted by: Shihab Shahriar Aion
Date: October 15, 2025

📋 Project Overview
This project implements an end-to-end automation system for processing customer orders from emails and managing product data. The system is built using n8n and consists of two main workflows that work together to streamline order management.
What This Automation Does:

Automatically fetches product data from an API and stores it in Google Sheets
Monitors incoming emails for customer orders
Uses AI to extract and structure order information
Looks up product details from the stored data
Creates organized tasks in Monday.com with complete order information


🏗️ Architecture
The project is divided into two interconnected workflows:
1. Product Data Pipeline
This workflow handles the collection and storage of product information.
2. Email to Monday.com Task
This workflow processes incoming customer emails and creates actionable tasks.

📊 Workflow 1: Product Data Pipeline
Purpose
This workflow fetches product data from a paginated API and stores it in Google Sheets for later reference.
How It Works
Step 1: Manual Trigger

The workflow starts when I manually execute it
This gives me control over when to refresh the product database

Step 2: HTTP Request

Connects to the DummyJSON products API (https://dummyjson.com/products)
Fetches the initial product data
The API returns products in a paginated format

Step 3: Data Transformation (JavaScript Code)

Extracts the products array from the API response
Transforms each product into a clean format with four fields:

ID: Product identifier (number)
Name: Product title
Price: Product price
Category: Product category


Returns an array of formatted product objects

Step 4: Google Sheets Storage

Appends all products to a Google Sheet titled "Product Data"
Maps the data to columns: ID, Name, Price, Category
Creates a persistent database that can be queried later

Pagination Implementation
Current Limitation:
The current implementation only fetches the first page of products from the API. I discovered during testing that the DummyJSON API returns all products in a single response with pagination metadata, but my workflow doesn't loop through multiple pages yet.
Why This Matters:
The assignment specifically mentioned handling pagination completely before processing. In a real-world scenario with hundreds or thousands of products across multiple pages, I would need to:

Check the total and limit fields in the API response
Calculate how many pages exist
Loop through all pages using the skip parameter
Collect all products before storing them

What I Learned:
For future improvements, I would add a loop node that continues fetching until all pages are retrieved, then batch process all products at once instead of page-by-page.

📧 Workflow 2: Email to Monday.com Task
Purpose
This workflow monitors incoming emails, extracts order information using AI, looks up product details, and creates tasks in Monday.com.
How It Works
Step 1: Email Trigger (IMAP)

Continuously monitors my Gmail inbox for new emails
Triggers automatically when a new email arrives
Configured with IMAP credentials to access Gmail securely

Step 2: Email Data Extraction (JavaScript Code)

Extracts key email components:

Subject line
Email body (prioritizes plain text over HTML)
Sender information
Date received


Prepares clean data for AI processing

Step 3: AI Processing (OpenAI Chat Model + AI Agent)

Uses GPT-4.1-mini to intelligently parse the email
Structured with a detailed prompt that instructs the AI to extract:

Product ID
Product name
Quantity ordered
Customer name
Delivery address
Phone number
Requested delivery date


The AI returns a JSON object with all extracted information
Returns null for any fields not found in the email

Step 4: JSON Parsing (JavaScript Code)

Takes the AI's text output and converts it to a proper JSON object
Removes markdown code blocks that the AI might include
Converts product_id to a number for matching with Google Sheets
Adds an "ID" field to enable merging with product data
Includes error handling for cases where AI output isn't valid JSON

Step 5: Product Lookup (Google Sheets)

Searches the Google Sheet for the product using the extracted product_id
Filters the sheet where ID column matches the order's product_id
Retrieves complete product information (Name, Price, Category)
This is the critical integration point mentioned in the requirements

Step 6: Data Merging

Combines the AI-extracted order data with the product information from Google Sheets
Uses a Merge node to join both data streams by position
Ensures all information is available for the final task creation

Step 7: Field Preparation (Edit Fields)

Prepares all data in the exact format Monday.com expects
Maps information to variables:

Board ID (hardcoded to my specific board)
Item name (Product Name + ID)
Order details (quantity, delivery date)
Customer information (name, address, phone)
Product details (ID, name, category, price)



Step 8: Monday.com Task Creation

Creates a new item in the specified Monday.com board
Populates all custom columns with the prepared data
Uses column IDs to match the board's structure
The task includes both order information and product details

Product Integration
How Product Lookup Works:
When an email mentions "Product ID 42" or similar, the workflow:

Extracts "42" from the email using AI
Converts it to a number
Searches the Google Sheet where ID = 42
Retrieves that product's name, price, and category
Includes all this information in the Monday.com task

This ensures that even if the email only mentions the product ID, the task will show the complete product details from our database.

🐛 Bugs and Issues Found
Issue #1: Pagination Not Fully Implemented
Problem: The Product Data Pipeline only fetches the first page of products.
Impact: If the API has more than 30 products (the default limit), we won't capture everything.
Solution Needed: Add a loop to fetch all pages before storing data.
Issue #2: Hardcoded Values in Email Workflow
Problem: Some values in the "Edit Fields" node are hardcoded (like quantity "3", date "October 10, 2025", etc.) instead of being dynamically pulled from the AI extraction.
Impact: The workflow works for the test email but won't adapt to different customer requests.
Solution Needed: Replace hardcoded values with dynamic references like {{ $json.quantity }}, {{ $json.delivery_date }}, etc.
Issue #3: AI Model Version
Problem: The OpenAI Chat Model uses "gpt-4.1-mini" which might not be the correct model identifier.
Impact: Could cause API errors if OpenAI doesn't recognize this model name.
Solution Needed: Verify the correct model name (likely "gpt-4-mini" or "gpt-3.5-turbo").
Issue #4: Error Handling
Problem: Limited error handling throughout the workflows.
What Could Go Wrong:

API might be down or return errors
AI might fail to extract information correctly
Product ID might not exist in Google Sheets
Monday.com API might fail
Solution Needed: Add error handling nodes to catch failures and send notifications or retry operations.

Issue #5: Product Name Matching
Problem: The workflow only looks up products by ID, not by name.
Impact: If a customer says "I want the Red T-Shirt" without mentioning the ID, the lookup would fail.
Solution Needed: Add fuzzy matching logic to search by product name as a fallback.

🛡️ Error Handling Strategy
Current Approach
My error handling is basic but functional:

JavaScript Code Nodes: Include try-catch blocks to handle JSON parsing errors
Console Logging: Log errors to help with debugging
Always Output Data: Some nodes are set to always output data even on failure

Recommended Improvements
For API Failures:

Add a "Check Response Status" condition
Retry failed requests with exponential backoff
Send alert notifications if API is consistently down

For Missing Product References:

Add a condition to check if Google Sheets returned any results
Create a fallback task that flags "Product Not Found" for manual review
Log missing product IDs for database updates

For AI Errors:

Validate that AI output contains required fields
Add a fallback to manual processing if AI confidence is low
Include example emails in the AI prompt for better accuracy

For Monday.com Integration:

Verify board and column IDs exist before creating items
Handle duplicate item prevention
Retry failed API calls


🎯 Assumptions Made

Single API Endpoint: I assumed all products would be available from one API endpoint without authentication requirements.
Email Format: I assumed customer emails would follow a reasonably consistent format with identifiable fields.
Product IDs: I assumed product IDs in emails would be numeric and match exactly with Google Sheets data.
Google Sheets Structure: I assumed a simple four-column structure would be sufficient for product data storage.
Monday.com Board: I assumed a specific board structure with predefined custom columns for order information.
One Product Per Order: The current implementation handles one product per email. Multiple products in a single order would require additional logic.
AI Accuracy: I assumed OpenAI would reliably extract information from well-structured emails.
Network Reliability: I assumed stable internet connectivity for all API calls.


🚀 Testing Results
Test Email Used
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
Workflow Execution
✅ Product Data Pipeline: Successfully fetched and stored products
✅ Email Trigger: Detected the test email
✅ AI Extraction: Correctly identified product ID, customer info, and requirements
✅ Product Lookup: Found Product ID 42 in Google Sheets
✅ Task Creation: Created Monday.com item with all details
What Worked Well

AI extraction was surprisingly accurate
Google Sheets integration was smooth
Monday.com task included all relevant information
The workflow executed end-to-end without manual intervention

What Needs Improvement

Dynamic value extraction instead of hardcoded fields
Better error messages
More robust pagination handling
Support for multiple products per order


📁 Project Files
All project files are organized in the Google Drive folder:

Product Data Pipeline.json - n8n workflow export for product fetching
Email to Monday.com Task.json - n8n workflow export for email processing
README.md - This documentation file
Video Walkthrough.mp4 - Demonstration video (link provided separately)


🔗 Resources Used

API: DummyJSON Products API
Storage: Google Sheets
AI: OpenAI GPT-4.1-mini
Task Management: Monday.com
Email: Gmail (IMAP)
Automation Platform: n8n


💭 Final Thoughts
This project was a great learning experience in building practical automation workflows. While there are areas for improvement (especially around pagination and error handling), the core functionality works as intended. The integration between AI extraction, database lookup, and task creation demonstrates how modern automation can significantly reduce manual work in order processing.
The most challenging part was ensuring data flowed correctly between nodes, especially when merging product information with order details. The most rewarding part was seeing the AI accurately extract structured data from unstructured email text.
Thank you for reviewing my submission. I'm excited about the possibility of working with Get Levrg Bangladesh Ltd and would welcome any feedback on how to improve this automation further!

Contact Information:
Shihab Shahriar Aion
aionshihabshahriar@gmail.com
+8801959040057