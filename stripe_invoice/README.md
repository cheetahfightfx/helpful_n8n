# Stripe Invoice Automation Flow

## Overview
Automatically generates and sends professional invoices when Stripe payments are received. Includes customer data enrichment, HTML invoice generation, email delivery with attachments, cloud storage backup, and team notifications.

## Business Value
- Eliminates manual invoice creation and sending
- Provides immediate payment confirmation to customers
- Creates organized audit trail of all transactions
- Improves cash flow visibility for accounting teams
- Reduces payment-to-invoice time from hours to seconds
- Ensures consistent invoice formatting and branding

## Prerequisites

### Required n8n Nodes
- Webhook (built-in)
- Stripe (built-in)
- Gmail (built-in)
- Google Drive (built-in)
- Slack (built-in)
- IF (built-in)
- Set (built-in)
- Respond to Webhook (built-in)
- Sticky Note (built-in)

### Required Credentials
1. **Stripe API**
   - Get API key from Stripe Dashboard → Developers → API keys
   - Use restricted key with read permissions for customers and payment intents

2. **Google OAuth2 (for Gmail and Drive)**
   - Enable Gmail API and Google Drive API
   - Create OAuth2 credentials with appropriate scopes
   - Configure in n8n with permissions for sending emails and creating files

3. **Slack OAuth2**
   - Create Slack app with bot token
   - Add scopes: `chat:write`, `channels:read`
   - Install to workspace and add to accounting channel

## Setup Instructions

### 1. Configure Stripe Webhook
In your Stripe Dashboard:
1. Go to Developers → Webhooks
2. Create endpoint with URL: `https://your-n8n-instance.com/webhook/stripe-payment`
3. Select events: `payment_intent.succeeded`
4. Copy webhook signing secret for production security

### 2. Create Google Drive Folder
1. Create "Invoices" folder in Google Drive
2. Copy the folder ID from URL (string after `/folders/`)
3. Update the folder ID in the "Save Invoice to Drive" node

### 3. Configure Slack Channel
1. Create `#accounting` channel (or use existing)
2. Add your n8n bot to the channel
3. Copy channel ID and update in "Notify Accounting Team" node

### 4. Import and Configure Flow
```bash
# Save and import the workflow
curl -o stripe-invoice-automation.json [your-github-raw-url]
```

Update these configurations:
- **Webhook path**: Customize if needed
- **Company details**: Update invoice template with your business info
- **Email sender**: Configure Gmail node with your business email
- **Drive folder**: Set correct folder ID for invoice storage
- **Slack channel**: Update channel ID for notifications

### 5. Customize Invoice Template
Edit the invoice HTML in "Generate Invoice HTML" node:

```html
<!-- Update company information -->
<div class="company-info">
    <h1>Your Company Name</h1>
    <p>123 Business Street<br>
    City, State 12345<br>
    Phone: (555) 123-4567<br>
    Email: billing@company.com</p>
</div>
```

Customize:
- Company name and logo
- Business address and contact info
- Brand colors and styling
- Tax calculations if applicable
- Terms and conditions

## Testing

### Test Payment Intent
Use Stripe test mode with this webhook payload:
```json
{
  "type": "payment_intent.succeeded",
  "data": {
    "object": {
      "id": "pi_test_12345",
      "amount": 9999,
      "currency": "usd",
      "receipt_email": "test@example.com",
      "description": "Website Development Service",
      "created": 1640995200,
      "customer": "cus_test_12345",
      "billing_details": {
        "name": "John Doe"
      }
    }
  }
}
```

Expected workflow execution:
1. Webhook receives payment event
2. Payment data extracted and formatted
3. Customer details fetched from Stripe
4. Professional invoice HTML generated
5. Email sent to customer with invoice attachment
6. Invoice saved to Google Drive
7. Accounting team notified via Slack
8. Success response sent to Stripe

### Manual Testing
1. Create test payment in Stripe Dashboard
2. Verify webhook triggers n8n workflow
3. Check email delivery to customer
4. Confirm invoice saved in Google Drive
5. Validate Slack notification sent
6. Review invoice formatting and content

## Production Considerations

### Security
- Implement webhook signature verification:
```javascript
// Add to webhook node for production
const crypto = require('crypto');
const signature = req.headers['stripe-signature'];
const endpointSecret = 'whsec_your_secret';
const payload = req.body;

const computedSignature = crypto
  .createHmac('sha256', endpointSecret)
  .update(payload, 'utf8')
  .digest('hex');

if (signature !== computedSignature) {
  throw new Error('Invalid signature');
}
```

### Performance
- Stripe webhook timeout is 30 seconds
- Optimize email sending for large attachments
- Consider async processing for high-volume merchants
- Monitor Google API quotas (Gmail: 1 billion quota units per day)

### Error Handling
Add error handling nodes for:
- Stripe API failures (customer not found)
- Email delivery failures
- Google Drive quota exceeded
- Slack notification failures

### Data Privacy
- Store customer data securely
- Implement data retention policies
- Add GDPR compliance features if needed
- Audit trail for invoice access

## Advanced Features

### 1. PDF Generation
Replace HTML with PDF generation:
```javascript
// Use PDF generation service
const pdfResponse = await fetch('https://api.html-to-pdf.com/convert', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    html: invoiceHtml,
    options: { format: 'A4', margin: '1cm' }
  })
});
```

### 2. Multi-Currency Support
```javascript
// Enhanced currency formatting
const formatCurrency = (amount, currency) => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: currency.toUpperCase()
  }).format(amount);
};
```

### 3. Tax Calculations
```javascript
// Add tax calculation logic
const calculateTax = (amount, customerCountry, productType) => {
  const taxRates = {
    'US': { 'digital': 0.08, 'physical': 0.065 },
    'GB': { 'digital': 0.20, 'physical': 0.20 },
    // Add more countries
  };
  
  const rate = taxRates[customerCountry]?.[productType] || 0;
  return amount * rate;
};
```

### 4. Accounting Integration
Add QuickBooks/Xero integration:
```javascript
// Sync to accounting software
const syncToAccounting = async (invoiceData) => {
  await fetch('https://api.quickbooks.com/v3/invoice', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${accessToken}` },
    body: JSON.stringify(invoiceData)
  });
};
```

## Monitoring & Analytics

### Key Metrics
- Invoice generation success rate
- Email delivery rate
- Average processing time
- Customer payment-to-invoice time
- Storage usage trends

### Recommended Dashboards
Create monitoring for:
- Daily payment volume
- Failed invoice generations
- Email bounce rates
- Drive storage utilization
- Webhook response times

## Troubleshooting

### Webhook Not Triggering
- Verify Stripe webhook URL is correct
- Check n8n webhook is active and accessible
- Validate SSL certificate
- Review Stripe webhook logs for delivery attempts

### Invoice Generation Failures
- Check customer data completeness
- Validate HTML template syntax
- Verify template variables are populated
- Test with minimal invoice template

### Email Delivery Issues
- Check Gmail API quotas and limits
- Verify sender email authentication
- Test email content for spam triggers
- Validate recipient email addresses

### Drive Upload Failures
- Check Google Drive API quotas
- Verify folder permissions
- Validate file size limits
- Test with smaller invoice files

### Slack Notification Problems
- Verify bot permissions in channel
- Check channel ID format
- Test with direct messages first
- Validate message formatting


## Customization Examples

### For SaaS Companies
- Add subscription details to invoices
- Include billing period information
- Add usage metrics and overage charges
- Implement proration calculations

### For E-commerce
- Include product line items
- Add shipping and handling charges
- Support multiple tax rates
- Include discount codes and promotions

### For Professional Services
- Add time tracking details
- Include project milestones
- Support hourly vs fixed pricing
- Add expense reimbursements

## License
MIT License - Feel free to use and modify for your projects.

## Support
For issues or questions:
1. Check Stripe webhook delivery logs
2. Verify all API credentials are current
3. Test individual nodes in isolation
4. Review n8n execution logs for detailed errors
5. Check third-party service status pages