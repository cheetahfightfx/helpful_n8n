# Google Forms Lead to CRM Pipeline

## Overview
Automatically processes form submissions from Google Forms, validates and scores leads, assigns them to sales team members, and triggers appropriate follow-up actions based on lead quality.

## Business Value
- Reduces lead response time from hours to minutes
- Automatically scores and prioritizes leads for better conversion
- Eliminates manual data entry and routing
- Ensures no leads fall through the cracks
- Provides immediate acknowledgment to prospects
- Creates audit trail of all lead interactions

## Prerequisites

### Required n8n Nodes
- Google Forms Trigger (built-in)
- Google Sheets (built-in)
- Gmail (built-in)
- Slack (built-in)
- IF (built-in)
- Set (built-in)
- Sticky Note (built-in)

### Required Credentials
1. **Google OAuth2 (for Forms, Sheets, Gmail)**
   - Enable Google Forms API, Sheets API, Gmail API
   - Create OAuth2 credentials in Google Cloud Console
   - Configure in n8n with appropriate scopes

2. **Slack OAuth2**
   - Create Slack app in your workspace
   - Add bot token scopes: `chat:write`, `channels:read`
   - Install app to workspace and copy bot token

## Setup Instructions

### 1. Create Google Form
Create a form with these recommended fields:
- Email Address (required)
- Full Name (required)
- Company Name
- Phone Number
- Project Budget (multiple choice)
- Project Details/Message (paragraph)

### 2. Create Google Sheet Database
Create a spreadsheet with these columns:
```
Timestamp | Email | Full Name | Company | Phone | Budget | Message | Lead Score | Priority | Assigned To | Source | Status
```

### 3. Configure Slack Channel
1. Create a `#sales-leads` channel
2. Add the n8n bot to the channel
3. Copy the channel ID for the workflow

### 4. Import and Configure Flow
```bash
# Import the workflow
curl -o google-forms-lead-pipeline.json [your-github-raw-url]
```

Update these node configurations:
- **Google Forms Trigger**: Select your form
- **Google Sheets**: Select your leads database spreadsheet
- **Slack**: Configure channel ID and customize message format
- **Gmail nodes**: Update sender email addresses

### 5. Customize Lead Scoring
Edit the "Score and Assign Lead" node to match your criteria:

```javascript
// Current scoring factors:
// - Valid email: +10 points
// - Company provided: +20 points  
// - Phone provided: +15 points
// - High budget indicators: +30 points
// - Detailed message (>100 chars): +15 points
// - Business email domain: +20 points

// Lead priority thresholds:
// - Hot: 70+ points
// - Warm: 40-69 points
// - Cold: <40 points
```

### 6. Configure Sales Team Assignment
Update the `assignedSalesperson` logic in the scoring node:

```javascript
const salesTeam = [
  'alice@company.com',    // Senior rep (gets hot leads)
  'bob@company.com', 
  'charlie@company.com'
];
```

## Testing

### Test Form Submission
Submit a test form with this data:
```
Email: john.doe@testcompany.com
Full Name: John Doe
Company: Test Company Inc
Phone: (555) 123-4567
Budget: $5,000 - $10,000
Message: We need help with a complex integration project involving multiple systems and APIs.
```

Expected behavior:
1. Lead gets scored (should be 75+ points = Hot)
2. Added to Google Sheets
3. Slack notification sent
4. Urgent email sent to assigned salesperson
5. Auto-response sent to prospect

### Validation Testing
Test with invalid data:
```
Email: invalid-email
Full Name: (empty)
```

Expected: Admin notification sent, no further processing.

## Production Considerations

### Performance Optimization
- Google Forms Trigger polls every minute (API limits apply)
- For high-volume forms, consider Google Sheets trigger instead
- Monitor Google API quotas and usage

### Lead Scoring Refinement
- Track conversion rates by score ranges
- Adjust scoring weights based on actual data
- Consider additional factors:
  - Industry/vertical
  - Company size (employees, revenue)
  - Geographic location
  - Referral source

### Sales Team Management
- Implement lead cap per salesperson
- Add vacation/availability logic
- Track response times and conversion rates
- Consider territory-based assignment

### Data Quality
- Add phone number format validation
- Implement duplicate detection
- Sanitize and normalize company names
- Add GDPR compliance fields if needed

## Monitoring & Analytics

### Key Metrics to Track
- Form submission volume
- Lead score distribution
- Sales team response times
- Conversion rate by lead score
- Source attribution

### Recommended Dashboard Fields
Add these calculated fields to your Google Sheet:
- Response Time (time to first contact)
- Conversion Status (qualified/unqualified)
- Deal Value (if converted)
- Days to Close

## Advanced Features to Add

### 1. Follow-up Sequences
- Day 1: Immediate response (current)
- Day 3: Check-in if no response
- Day 7: Final follow-up with value content
- Day 30: Re-engagement campaign

### 2. Lead Enrichment
- Integrate with Clearbit/ZoomInfo for company data
- LinkedIn profile lookup
- Social media validation
- Company website analysis

### 3. CRM Integration
- Sync with Salesforce/HubSpot/Pipedrive
- Create deals automatically for hot leads
- Update lead status bidirectionally
- Track email engagement

### 4. Advanced Routing
- Round-robin with weighting
- Skill-based assignment
- Timezone-aware routing
- Workload balancing

## Common Issues

### Form Trigger Not Working
- Verify Google Forms API is enabled
- Check OAuth2 permissions include Forms scope
- Ensure form is published and accepting responses
- Test with manual form submission

### Lead Scoring Issues
- Verify field names match form exactly
- Check for case sensitivity in budget detection
- Test scoring logic with console.log in Function node
- Validate regex patterns for email domains

### Email Delivery Problems
- Check Gmail API quotas (250 quota units per user per 100 seconds)
- Verify sender email has necessary permissions
- Test email templates for spam triggers
- Consider using dedicated email service for production

### Slack Notifications Failed
- Verify bot has correct permissions
- Check channel ID format (starts with 'C')
- Ensure bot is added to the target channel
- Test with direct message first

## Customization Examples

### For Different Industries

**Software/SaaS:**
- Add "Current Solution" field
- Score technical complexity indicators
- Route by company size/ARR potential

**Professional Services:**
- Add "Project Timeline" field
- Score urgency indicators
- Route by service type expertise

**E-commerce:**
- Add "Product Category" field
- Score by order volume potential
- Route by platform expertise

## License
MIT License - Feel free to use and modify for your projects.

## Support
For issues or questions:
1. Check Google API quotas and permissions
2. Verify all credentials are properly configured
3. Test individual nodes in isolation
4. Check n8n execution logs for detailed error messages