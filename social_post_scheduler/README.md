# Social Media Content Scheduler Flow

## Overview
Automatically publishes social media content across multiple platforms based on a Google Sheets content calendar. Includes platform-specific content formatting, scheduling with time windows, publishing tracking, and comprehensive analytics logging.

## Business Value
- Eliminates manual posting across multiple platforms
- Ensures consistent posting schedule and brand presence
- Provides detailed analytics and audit trails
- Reduces social media management workload by 80%
- Prevents missed posts and scheduling conflicts
- Enables bulk content planning and scheduling

## Prerequisites

### Required n8n Nodes
- Schedule Trigger (built-in)
- Google Sheets (built-in)
- Twitter (built-in)
- LinkedIn (built-in)
- Facebook Graph API (built-in)
- Slack (built-in)
- IF (built-in)
- Set (built-in)
- Sticky Note (built-in)

### Required Credentials
1. **Google OAuth2 (for Sheets)**
   - Enable Google Sheets API
   - Create OAuth2 credentials with sheets read/write permissions

2. **Twitter API v2**
   - Apply for Twitter Developer account
   - Create app and get API keys
   - Requires Twitter API v2 access

3. **LinkedIn API**
   - Create LinkedIn Developer application
   - Get OAuth2 credentials for posting
   - Requires LinkedIn Pages API access

4. **Facebook Graph API**
   - Create Facebook App in Meta Developers
   - Get Page Access Token
   - Add pages_manage_posts permission

5. **Slack OAuth2**
   - Create Slack app with bot token
   - Add chat:write scope for notifications

## Setup Instructions

### 1. Create Content Calendar Spreadsheet
Create Google Sheet with these columns:
```
Content | Platforms | Scheduled Time | Status | Image URL | Hashtags | Campaign | Priority | Row Number | Published Time | Platforms Published | Notes
```

Sample data format:
```csv
"Excited to share our latest blog post about AI trends!", "Twitter,LinkedIn", "2025-06-24 14:30:00", "Scheduled", "", "#AI #TechTrends #Innovation", "Q2Campaign", "high", "2", "", "", ""
"Check out our new product features in this video", "Facebook,LinkedIn", "2025-06-24 16:00:00", "Scheduled", "https://example.com/image.jpg", "#ProductUpdate #Innovation", "ProductLaunch", "normal", "3", "", "", ""
```

### 2. Create Analytics Tracking Sheet
Create separate sheet for analytics with columns:
```
Publish Date | Publish Time | Campaign | Platforms | Content Preview | Scheduled Time | Time Difference | Priority
```

### 3. Configure Social Media Apps

#### Twitter Setup
1. Apply for Twitter Developer account
2. Create new app in Twitter Developer Portal
3. Generate API keys and bearer token
4. Configure n8n Twitter node with credentials

#### LinkedIn Setup
1. Create LinkedIn Developer application
2. Request access to LinkedIn Pages API
3. Generate OAuth2 credentials
4. Configure n8n LinkedIn node

#### Facebook Setup
1. Create Facebook App in Meta for Developers
2. Add Facebook Login and Facebook Pages products
3. Generate Page Access Token
4. Configure n8n Facebook Graph API node

### 4. Import and Configure Flow
```bash
# Import the workflow
curl -o social-media-scheduler.json [your-github-raw-url]
```

Update these configurations:
- **Google Sheets nodes**: Set your content calendar spreadsheet ID
- **Analytics sheet**: Set your analytics spreadsheet ID
- **Facebook node**: Set your Facebook Page ID
- **Slack node**: Set your marketing team channel ID
- **Schedule**: Adjust frequency if needed (default: every 30 minutes)

### 5. Platform Configuration

#### Content Formatting Rules
The workflow automatically formats content for each platform:

**Twitter (280 characters)**:
- Truncates content if too long
- Adds hashtags if they fit within limit
- Adds ellipsis for truncated content

**LinkedIn (longer form)**:
- Full content with hashtags on new lines
- Adds campaign tag as additional hashtag
- Professional formatting

**Facebook (flexible length)**:
- Full content with hashtags
- Optimized for engagement

### 6. Time Zone Configuration
Update the time comparison logic for your timezone:
```javascript
// In "Filter Ready to Post" node, adjust for your timezone
const scheduledTime = new Date($json['Scheduled Time']);
const now = new Date();
// Add timezone offset if needed
const timezoneOffset = -5; // EST example
scheduledTime.setHours(scheduledTime.getHours() + timezoneOffset);
```

## Testing

### Test Content Calendar Entry
Add this test entry to your Google Sheet:
```
Content: "This is a test post from our automated system!"
Platforms: "Twitter,LinkedIn"
Scheduled Time: [current time + 5 minutes]
Status: "Scheduled"
Image URL: ""
Hashtags: "#Test #Automation"
Campaign: "Testing"
Priority: "normal"
```

Expected behavior:
1. Workflow reads content calendar every 30 minutes
2. Finds posts scheduled within 30-minute window
3. Formats content for each platform
4. Posts to specified platforms
5. Updates calendar with "Published" status
6. Logs analytics data
7. Notifies marketing team via Slack

### Manual Testing
1. Create test entries with different time ranges
2. Test with single and multiple platforms
3. Verify content formatting for each platform
4. Check status updates in calendar
5. Confirm analytics logging
6. Validate Slack notifications

## Production Considerations

### API Rate Limits
- **Twitter**: 300 tweets per 15-minute window
- **LinkedIn**: 100 posts per day per person/page
- **Facebook**: 600 calls per hour per app
- **Google Sheets**: 100 requests per 100 seconds per user

### Content Guidelines
- Keep original content under 260 characters for Twitter compatibility
- Use high-quality images (1080x1080 for best results)
- Include relevant hashtags (max 30 for Instagram, 2-3 for Twitter)
- Test content formatting before scheduling

### Error Handling
Add error handling for:
- API authentication failures
- Content formatting issues
- Image upload failures
- Spreadsheet access problems
- Network connectivity issues

### Security
- Use restricted API keys with minimal permissions
- Rotate credentials regularly
- Monitor for unusual posting activity
- Implement content approval workflows for sensitive accounts

## Advanced Features

### 1. Image Handling
Add image download and upload functionality:
```javascript
// Download image from URL
const imageResponse = await fetch($json.imageUrl);
const imageBuffer = await imageResponse.buffer();

// Upload to each platform with specific requirements
// Twitter: max 5MB, JPG/PNG/GIF
// LinkedIn: max 20MB, JPG/PNG
// Facebook: max 4MB, JPG/PNG/GIF
```

### 2. Content Personalization
```javascript
// Dynamic content based on platform
const platformContent = {
  'Twitter': $json.content + ' ' + $json.hashtags,
  'LinkedIn': $json.content + '\n\nRead more: ' + $json.url,
  'Facebook': $json.content + '\n\n' + $json.callToAction
};
```

### 3. Engagement Tracking
```javascript
// Add engagement metrics collection
const engagementData = {
  postId: response.id,
  platform: 'Twitter',
  likes: 0,
  shares: 0,
  comments: 0,
  reach: 0,
  impressions: 0
};
```

### 4. A/B Testing
```javascript
// Test different content variations
const variants = [
  { content: 'Version A content', hashtags: '#TestA' },
  { content: 'Version B content', hashtags: '#TestB' }
];

const selectedVariant = variants[Math.floor(Math.random() * variants.length)];
```

## Analytics and Reporting

### Key Metrics to Track
- Posts published per platform
- Posting accuracy (scheduled vs actual time)
- Content performance by campaign
- Platform engagement rates
- Peak posting times effectiveness

### Weekly Report Generation
Create automated weekly reports:
```javascript
// Generate weekly summary
const weeklyStats = {
  totalPosts: publishedPosts.length,
  platformBreakdown: {
    Twitter: twitterPosts.length,
    LinkedIn: linkedinPosts.length,
    Facebook: facebookPosts.length
  },
  campaignPerformance: groupBy(posts, 'campaign'),
  avgPostingDelay: calculateAverageDelay(posts)
};
```

### Dashboard Integration
- Connect to Google Data Studio for visual dashboards
- Export data to BI tools for advanced analytics
- Create real-time monitoring alerts
- Track ROI and conversion metrics

## Troubleshooting

### Common Issues

#### Posts Not Publishing
- Check platform API credentials
- Verify content meets platform guidelines
- Confirm scheduled time format is correct
- Check for content that violates platform policies

#### Time Zone Problems
- Verify scheduled times are in correct timezone
- Check server timezone vs local timezone
- Test with different time formats
- Add explicit timezone handling

#### Content Formatting Issues
- Test character limits for each platform
- Verify hashtag formatting
- Check for special characters that break formatting
- Test with various content lengths

#### Spreadsheet Access Errors
- Refresh Google OAuth2 credentials
- Check spreadsheet sharing permissions
- Verify column headers match exactly
- Test with simplified spreadsheet structure

### Error Recovery
- Add retry logic for failed API calls
- Log all errors with context for debugging
- Implement fallback posting options
- Create manual intervention alerts

## Best Practices

### Content Strategy
- Plan content 1-2 weeks in advance
- Maintain consistent posting schedule
- Vary content types (text, images, videos, links)
- Monitor competitor posting patterns
- Adjust timing based on audience analytics

### Platform Optimization
- Twitter: Focus on real-time engagement and trending topics
- LinkedIn: Professional content and thought leadership
- Facebook: Community building and longer-form content
- Instagram: Visual storytelling and behind-the-scenes content

### Campaign Management
- Use consistent campaign tags for tracking
- Coordinate campaigns across all platforms
- Plan seasonal and event-based content
- Monitor campaign performance and adjust strategy

## License
MIT License - Feel free to use and modify for your projects.

## Support
For issues or questions:
1. Check platform API status pages
2. Verify all credentials are current and have proper permissions
3. Test individual platform nodes in isolation
4. Review content for platform policy compliance
5. Check Google Sheets data format and permissions