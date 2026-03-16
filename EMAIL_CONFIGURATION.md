# Email Verification Configuration Guide

## Problem
Verification emails are not being sent during signup because SMTP is not configured in your Supabase project.

## Solution
You need to configure an SMTP provider in your Supabase project to enable email sending.

### Step 1: Go to Supabase Dashboard
1. Open your Supabase project
2. Navigate to **Authentication > Email Templates**
3. Look for the **SMTP Configuration** section

### Step 2: Configure SMTP
Choose one of these options:

#### Option A: Use a Third-Party SMTP Provider (Recommended)
Popular providers:
- **Resend** (resend.com) - Great for transactional emails
- **SendGrid** (sendgrid.com)
- **Mailgun** (mailgun.com)
- **AWS SES** (aws.amazon.com/ses/)

You'll need:
- SMTP Host
- SMTP Port
- SMTP Username
- SMTP Password
- From Email Address
- From Email Name

#### Option B: Enable Supabase Email Service (Limited Regions)
If available in your region, Supabase offers a built-in email service.

### Step 3: Test Email Configuration
1. After configuring SMTP, go to **Authentication > Email Templates**
2. Try sending a test email to verify the configuration works

### Step 4: Customize Email Templates (Optional)
In the Email Templates section, you can customize:
- Confirmation email subject and content
- Password recovery email
- Magic link email

## How It Works in This App

When a user signs up:
1. The signup API creates a user with `email_confirm: false`
2. Supabase automatically sends a confirmation email
3. The email contains a confirmation link
4. User clicks the link to verify their email
5. The user account is then fully activated

## Testing
To test in development:
1. Use a test email service like **Ethereal Email** (ethereal.email)
2. Or use your personal email if you've configured a real SMTP provider
3. Check your email inbox for the confirmation message

## Need Help?
- Supabase Email Docs: https://supabase.com/docs/guides/auth/auth-email
- Check your Supabase project activity logs for email sending errors
