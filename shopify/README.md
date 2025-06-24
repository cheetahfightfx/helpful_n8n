# Shopify Abandoned Cart Recovery Flow

## Overview
Automatically sends recovery emails to customers who abandon their shopping carts, with smart logic to avoid sending emails to customers who have already completed their purchase.

## Business Value
- Recovers 15-25% of abandoned carts on average
- Increases revenue without additional marketing spend
- Improves customer experience with timely, relevant emails
- Reduces manual follow-up work for sales teams

## Prerequisites

### Required n8n Nodes
- Webhook (built-in)
- Shopify (install via n8n community nodes)
- Gmail (built-in)
- IF (built-in) 
- Wait (built-in)
- Set (built-in)
- Respond to Webhook (built-in)
- Sticky Note (built-in)

### Required Credentials
1. **Shopify OAuth2 API**
   - Go to your Shopify Admin → Apps → Manage private apps
   - Create private app with permissions:
     - Read orders
     - Read and write customers
     - Read products
   - Copy API credentials to n8n

2. **Gmail OAuth2**
   - Enable Gmail API in Google Cloud Console
   - Create OAuth2 credentials
   - Configure in n8n Gmail node

## Setup Instructions

### 1. Import the Flow
```bash
# Save the JSON to a file
curl -o shopify-abandoned-cart.json [your-github-raw-url]

# Import in n8n UI or via CLI
n8n import:workflow --file=shopify-abandoned-cart.json
```

### 2. Configure Shopify Webhook
In your Shopify Admin:
1. Go to Settings → Notifications
2. Scroll to "Webhooks" section
3. Add webhook:
   - Event: `Checkouts abandoned`
   - Format: `JSON`
   - URL: `https://your-n8n-instance.com/webhook/abandoned-cart`

### 3. Update Node Configurations
1. **Webhook Node**: Update the webhook path if needed
2. **Gmail Node**: Configure your sending email address
3. **Shopify Nodes**: Test the connection with your credentials

### 4. Customize Email Template
Edit the HTML email template in the "Send Recovery Email" node:
- Update branding and styling
- Modify copy to match your voice
- Add discount codes if desired
- Include social media links

## Testing

### Test Data
Use this sample JSON in the webhook node for testing:
```json
{
  "email": "test@example.com",
  "billing_address": {
    "first_name": "John"
  },
  "total_price": "99.99",
  "abandoned_checkout_url": "https://your-store.com/checkout/abc123",
  "line_items": [
    {
      "title": "Test Product"
    }
  ],
  "customer": {
    "id": "123456",
    "tags": "existing-tag"
  }
}
```

### Manual Testing Steps
1. Activate the workflow
2. Send test webhook payload
3. Check email delivery
4. Verify customer tagging in Shopify
5. Test with customer who already purchased

## Production Considerations

### Performance
- Enable workflow execution order v1 for better performance
- Consider using Schedule Trigger instead of Wait for high-volume stores
- Add error handling for API rate limits

### Email Deliverability
- Use a dedicated sending domain
- Implement SPF, DKIM, and DMARC records
- Monitor bounce rates and spam complaints
- Consider using dedicated email service (SendGrid, Mailgun)

### Data Privacy
- Ensure GDPR compliance for EU customers
- Include unsubscribe links in emails
- Respect customer communication preferences
- Log webhook data appropriately

## Monitoring & Analytics

### Key Metrics to Track
- Email delivery rate
- Open rate (requires tracking pixels)
- Click-through rate to cart
- Recovery conversion rate
- Revenue recovered

### Recommended Improvements
1. Add email tracking with pixels
2. Implement A/B testing for subject lines
3. Create follow-up email sequence (Day 3, Day 7)
4. Add SMS recovery for high-value carts
5. Integrate with customer service tools

## Common Issues

### Webhooks Not Firing
- Check Shopify webhook configuration
- Verify n8n webhook URL is accessible
- Check firewall and SSL certificate
- Test with ngrok for local development

### Emails Not Sending
- Verify Gmail OAuth2 credentials
- Check sending limits (Gmail: 500/day)
- Review email content for spam triggers
- Test with different email providers

### Duplicate Emails
- Customer tagging prevents duplicates
- Check tag logic in Shopify node
- Implement additional deduplication if needed


## License
MIT License - Feel free to use and modify for your projects.

## Support
For issues or questions:
1. Check the troubleshooting section
2. Review n8n documentation
3. Test with provided sample data
4. Verify all credentials are properly configured