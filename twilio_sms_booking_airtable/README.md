# Twilio SMS Booking Confirmation System

## Overview
Complete booking management system that accepts bookings via webhook, stores them in Airtable, sends SMS confirmations, automatically sends reminder messages 2 days before appointments, and processes customer responses to update booking status accordingly.

## Business Value
- Reduces no-shows by 40-60% through automated confirmation reminders
- Eliminates manual booking confirmation calls
- Provides real-time booking status updates
- Improves customer experience with instant confirmations
- Creates comprehensive audit trail of all booking communications
- Scales to handle hundreds of bookings without additional staff

## Prerequisites

### Required n8n Nodes
- Webhook (built-in)
- Schedule Trigger (built-in)
- Airtable (built-in)
- Twilio (built-in)
- IF (built-in)
- Set (built-in)
- Respond to Webhook (built-in)
- Sticky Note (built-in)

### Required Credentials
1. **Twilio Account**
   - Sign up at twilio.com
   - Get Account SID and Auth Token
   - Purchase a phone number for SMS sending
   - Configure webhook endpoint for incoming SMS

2. **Airtable API**
   - Create Airtable account and base
   - Generate API key from account settings
   - Create bookings table with required fields

## Setup Instructions

### 1. Create Airtable Base and Table
Create a base called "Booking Management" with a table called "Bookings":

**Required Fields:**
```
Booking ID (Single line text, Primary field)
Customer Name (Single line text)
Phone Number (Phone number)
Booking Date (Date)
Booking Time (Single line text)
Service (Single line text)
Created At (Date and time)
Confirmation Status (Single select: Pending, Confirmed, Cancelled)
Reminder Sent (Single select: No, Yes)
Confirmation Deadline (Date)
Reminder Sent At (Date and time)
Confirmed At (Date and time)
Cancelled At (Date and time)
Response Message (Long text)
Unclear Response At (Date and time)
```

### 2. Configure Twilio
1. Purchase a Twilio phone number
2. Configure webhook for incoming SMS:
   - URL: `https://your-n8n-instance.com/webhook/sms-response`
   - HTTP Method: POST
3. Copy Account SID and Auth Token for n8n configuration

### 3. Import and Configure Flow
```bash
# Import the workflow
curl -o twilio-booking-confirmation.json [your-github-raw-url]
```

Update these configurations:
- **Airtable Base ID**: Replace `YOUR_AIRTABLE_BASE_ID`
- **Airtable Table ID**: Replace `YOUR_BOOKINGS_TABLE_ID`
- **Twilio Phone Number**: Replace `YOUR_TWILIO_PHONE_NUMBER`
- **Webhook paths**: Customize if needed

### 4. Test Booking Creation
Send POST request to booking webhook:
```json
{
  "customerName": "John Doe",
  "phoneNumber": "+1234567890",
  "bookingDate": "2025-06-30",
  "bookingTime": "14:00",
  "service": "Dental Cleaning",
  "bookingId": "BK-12345"
}
```

Expected flow:
1. Booking data validated
2. Record created in Airtable
3. Confirmation SMS sent to customer
4. Success response returned

### 5. Test Reminder System
1. Create a test booking with confirmation deadline = today
2. Run the daily reminder check manually
3. Verify reminder SMS is sent
4. Check Airtable record is updated

### 6. Test Response Processing
Reply to reminder SMS with:
- "YES" - Should mark as confirmed
- "NO" - Should mark as cancelled
- "Maybe" - Should send clarification message

## Message Templates

### Initial Booking Confirmation
```
Hi [Customer Name]! Your booking for [Service] on [Date] at [Time] has been confirmed.

Booking ID: [Booking ID]

You'll receive a confirmation reminder 2 days before your appointment. Thank you!
```

### Reminder Message (2 days before)
```
Hi [Customer Name]! This is a reminder for your [Service] appointment on [Date] at [Time].

Please reply:
✅ YES to confirm
❌ NO to cancel

Booking ID: [Booking ID]

Thank you!
```

### Confirmation Response
```
Thank you [Customer Name]! Your booking for [Service] on [Date] at [Time] is now CONFIRMED.

Booking ID: [Booking ID]

See you soon!
```

### Cancellation Response
```
Hi [Customer Name], we've cancelled your booking for [Service] on [Date] as requested.

Booking ID: [Booking ID]

Feel free to book again anytime. Thank you!
```

### Clarification Request
```
Hi [Customer Name], we didn't understand your response. Please reply:

✅ YES to confirm your booking
❌ NO to cancel

For [Service] on [Date]
Booking ID: [Booking ID]
```

## Advanced Configuration

### 1. Custom Reminder Timing
Modify the confirmation deadline calculation:
```javascript
// Change from 2 days to 3 days before
const confirmationDeadline = new Date(new Date(bookingDate).getTime() - (3 * 24 * 60 * 60 * 1000));

// Or set specific time (e.g., 48 hours before at 10 AM)
const confirmationDeadline = new Date(bookingDate);
confirmationDeadline.setDate(confirmationDeadline.getDate() - 2);
confirmationDeadline.setHours(10, 0, 0, 0);
```

### 2. Multiple Reminder Attempts
Add escalation logic for non-responses:
```javascript
// In Airtable, add fields:
// - First Reminder Sent (Date)
// - Second Reminder Sent (Date)
// - Final Reminder Sent (Date)

// Modify filter to send multiple reminders
const filterFormula = `
AND(
  {Booking Date} >= TODAY(),
  {Confirmation Status} = 'Pending',
  OR(
    AND({First Reminder Sent} = BLANK(), {Confirmation Deadline} = TODAY()),
    AND({Second Reminder Sent} = BLANK(), {Confirmation Deadline} = DATEADD(TODAY(), -1, 'days')),
    AND({Final Reminder Sent} = BLANK(), {Confirmation Deadline} = DATEADD(TODAY(), -2, 'days'))
  )
)`;
```

### 3. Business Hours Restrictions
Add time-based sending logic:
```javascript
// Only send SMS during business hours
const now = new Date();
const hour = now.getHours();
const day = now.getDay(); // 0 = Sunday, 6 = Saturday

const isBusinessHours = (hour >= 9 && hour <= 17) && (day >= 1 && day <= 5);

if (!isBusinessHours) {
  // Schedule for next business day at 9 AM
  return; // Skip sending now
}
```

### 4. Service-Specific Templates
Customize messages based on service type:
```javascript
const serviceTemplates = {
  'Dental Cleaning': {
    reminder: 'Please remember to avoid eating 2 hours before your dental cleaning.',
    confirmation: 'Your dental cleaning is confirmed. Please bring your insurance card.'
  },
  'Hair Appointment': {
    reminder: 'Please arrive with clean, dry hair for your appointment.',
    confirmation: 'Your hair appointment is confirmed. We look forward to seeing you!'
  },
  'Consultation': {
    reminder: 'Please prepare any questions you have for your consultation.',
    confirmation: 'Your consultation is confirmed. We\'ll have 60 minutes together.'
  }
};

const template = serviceTemplates[service] || serviceTemplates['default'];
```

## Integration Examples

### 1. Website Booking Form Integration
```html
<!-- Simple booking form -->
<form id="bookingForm">
  <input type="text" name="customerName" placeholder="Full Name" required>
  <input type="tel" name="phoneNumber" placeholder="Phone Number" required>
  <input type="date" name="bookingDate" required>
  <input type="time" name="bookingTime" required>
  <select name="service" required>
    <option value="Consultation">Consultation</option>
    <option value="Treatment">Treatment</option>
    <option value="Follow-up">Follow-up</option>
  </select>
  <button type="submit">Book Appointment</button>
</form>

<script>
document.getElementById('bookingForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  const formData = new FormData(e.target);
  const booking = Object.fromEntries(formData);
  
  try {
    const response = await fetch('https://your-n8n-instance.com/webhook/booking-webhook', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(booking)
    });
    
    const result = await response.json();
    if (result.status === 'success') {
      alert('Booking confirmed! You will receive an SMS confirmation.');
    }
  } catch (error) {
    alert('Booking failed. Please try again.');
  }
});
</script>
```

### 2. Calendar Integration
```javascript
// Google Calendar integration
const createCalendarEvent = async (booking) => {
  const event = {
    summary: `${booking.service} - ${booking.customerName}`,
    start: {
      dateTime: `${booking.bookingDate}T${booking.bookingTime}:00`,
      timeZone: 'America/New_York'
    },
    end: {
      dateTime: `${booking.bookingDate}T${addHour(booking.bookingTime)}:00`,
      timeZone: 'America/New_York'
    },
    description: `Customer: ${booking.customerName}\nPhone: ${booking.phoneNumber}\nBooking ID: ${booking.bookingId}`
  };
  
  return await calendar.events.insert({
    calendarId: 'primary',
    resource: event
  });
};
```

### 3. CRM Integration
```javascript
// Sync confirmed bookings to CRM
const syncToCRM = async (booking) => {
  if (booking.confirmationStatus === 'Confirmed') {
    await fetch('https://api.yourcrm.com/contacts', {
      method: 'POST',
      headers: {
        'Authorization': 'Bearer YOUR_CRM_TOKEN',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        name: booking.customerName,
        phone: booking.phoneNumber,
        last_appointment: booking.bookingDate,
        service_type: booking.service,
        booking_id: booking.bookingId
      })
    });
  }
};
```

## Monitoring and Analytics

### Key Metrics to Track
- Booking confirmation rate
- No-show rate before/after implementation
- Average response time to confirmations
- Cancellation rate and timing
- Customer satisfaction with SMS system

### Airtable Views for Reporting
Create these views in Airtable:

**Today's Confirmations**
```
Filter: Confirmation Deadline = Today
Sort: Booking Time (ascending)
```

**Pending Confirmations**
```
Filter: Confirmation Status = Pending, Booking Date >= Today
Sort: Booking Date (ascending)
```

**Weekly Booking Summary**
```
Filter: Created At >= 7 days ago
Group by: Confirmation Status
```

### Automated Reporting
```javascript
// Weekly summary email
const weeklyStats = {
  totalBookings: bookings.length,
  confirmed: bookings.filter(b => b.status === 'Confirmed').length,
  cancelled: bookings.filter(b => b.status === 'Cancelled').length,
  pending: bookings.filter(b => b.status === 'Pending').length,
  noShowRate: calculateNoShowRate(bookings),
  avgResponseTime: calculateAvgResponseTime(bookings)
};
```

## Troubleshooting

### Common Issues

#### SMS Not Sending
- Check Twilio account balance
- Verify phone number format (must include country code)
- Confirm Twilio phone number is SMS-enabled
- Check for carrier-specific blocking

#### Reminders Not Triggering
- Verify schedule trigger is active
- Check Airtable filter formula syntax
- Confirm date calculations are correct
- Test with manual trigger execution

#### Responses Not Processing
- Verify Twilio webhook URL is correct
- Check webhook is accessible from internet
- Confirm response parsing logic
- Test with manual webhook calls

#### Airtable Updates Failing
- Check API key permissions
- Verify record ID is correct
- Confirm field names match exactly
- Test with simplified update operations

### Error Recovery
- Add retry logic for failed SMS sends
- Implement fallback phone calls for critical bookings
- Create manual intervention alerts
- Maintain audit logs for all operations

## Customization for Different Industries

### Healthcare/Dental
```javascript
// Add appointment preparation instructions
const healthcareReminder = `
Reminder: Your ${service} appointment on ${date} at ${time}.

Please arrive 15 minutes early and bring:
- Insurance card
- Valid ID
- List of current medications

Reply YES to confirm or NO to cancel.
`;
```

### Beauty/Salon
```javascript
// Include service-specific instructions
const beautyReminder = `
Hi ${name}! Reminder for your ${service} on ${date} at ${time}.

Preparation tips:
- Come with clean hair (for hair services)
- Remove nail polish (for manicures)
- Arrive 10 minutes early

Reply YES to confirm or NO to cancel.
`;
```

### Professional Services
```javascript
// Include meeting preparation
const consultationReminder = `
Hi ${name}! Reminder for your ${service} on ${date} at ${time}.

Please prepare:
- List of questions/concerns
- Relevant documents
- Budget information if applicable

Reply YES to confirm or NO to reschedule.
`;
```

## License
MIT License - Feel free to use and modify for your projects.

## Support
For issues or questions:
1. Check Twilio and Airtable API status
2. Verify all credentials are current
3. Test individual components in isolation
4. Review message logs for delivery status
5. Check webhook accessibility from external services