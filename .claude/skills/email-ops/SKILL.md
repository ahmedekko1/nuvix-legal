# Email Ops Skill

Send transactional emails, manage templates, handle bounces, and monitor deliverability.

## Provider Setup

### Resend (recommended for new projects)

```typescript
// src/lib/email/client.ts
import { Resend } from 'resend';

export const resend = new Resend(process.env.RESEND_API_KEY);
```

### SendGrid

```typescript
import sgMail from '@sendgrid/mail';
sgMail.setApiKey(process.env.SENDGRID_API_KEY!);
```

## Sending Email

```typescript
// src/lib/email/send.ts
import { resend } from './client';

interface EmailOptions {
  to: string | string[];
  subject: string;
  html: string;
  text?: string;
  from?: string;
  replyTo?: string;
}

export async function sendEmail(options: EmailOptions) {
  const { data, error } = await resend.emails.send({
    from: options.from ?? `Nuvix <noreply@nuvix.ai>`,
    to: options.to,
    subject: options.subject,
    html: options.html,
    text: options.text,
    reply_to: options.replyTo,
  });

  if (error) {
    throw new Error(`Email send failed: ${error.message}`);
  }

  return data;
}
```

## React Email Templates

```tsx
// src/emails/WelcomeEmail.tsx
import { Html, Head, Body, Container, Heading, Text, Button, Hr } from '@react-email/components';

interface WelcomeEmailProps {
  name: string;
  loginUrl: string;
}

export function WelcomeEmail({ name, loginUrl }: WelcomeEmailProps) {
  return (
    <Html>
      <Head />
      <Body style={{ fontFamily: 'Inter, sans-serif', backgroundColor: '#f9fafb' }}>
        <Container style={{ maxWidth: '600px', margin: '0 auto', padding: '40px 20px' }}>
          <Heading style={{ color: '#111827', fontSize: '24px' }}>
            Welcome to Nuvix, {name}!
          </Heading>
          <Text style={{ color: '#6b7280', fontSize: '16px', lineHeight: '24px' }}>
            Your account is ready. Click below to get started.
          </Text>
          <Button
            href={loginUrl}
            style={{ backgroundColor: '#0ea5e9', color: '#fff', padding: '12px 24px', borderRadius: '6px' }}
          >
            Get Started
          </Button>
          <Hr style={{ borderColor: '#e5e7eb', margin: '32px 0' }} />
          <Text style={{ color: '#9ca3af', fontSize: '12px' }}>
            You received this email because you signed up for Nuvix. 
            <a href="{{{unsubscribeUrl}}}">Unsubscribe</a>
          </Text>
        </Container>
      </Body>
    </Html>
  );
}
```

## Rendering and Sending Together

```typescript
import { render } from '@react-email/render';
import { WelcomeEmail } from '@/emails/WelcomeEmail';
import { sendEmail } from '@/lib/email/send';

export async function sendWelcomeEmail(user: { email: string; name: string }) {
  const html = render(WelcomeEmail({ name: user.name, loginUrl: `${process.env.APP_URL}/login` }));

  await sendEmail({
    to: user.email,
    subject: 'Welcome to Nuvix',
    html,
  });
}
```

## Webhook Handling (Bounces & Complaints)

```typescript
// src/app/api/webhooks/resend/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const payload = await req.json();

  switch (payload.type) {
    case 'email.bounced':
      await handleBounce(payload.data.email_id, payload.data.to[0]);
      break;
    case 'email.complained':
      await handleComplaint(payload.data.to[0]);
      break;
  }

  return NextResponse.json({ ok: true });
}

async function handleBounce(emailId: string, address: string) {
  // Mark email address as bounced in DB, stop sending to it
  await db.emailBounce.upsert({
    where: { address },
    create: { address, emailId, bouncedAt: new Date() },
    update: { emailId, bouncedAt: new Date() },
  });
}

async function handleComplaint(address: string) {
  // Unsubscribe immediately on complaint
  await db.user.updateMany({
    where: { email: address },
    data: { emailOptOut: true },
  });
}
```

## Deliverability Checklist

- [ ] SPF record configured for sending domain
- [ ] DKIM signing enabled (provider generates keys)
- [ ] DMARC policy set (`v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.com`)
- [ ] `From` address uses a real domain (not `gmail.com`)
- [ ] Unsubscribe link in every marketing email
- [ ] `List-Unsubscribe` header for bulk mail
- [ ] Plain text version alongside HTML
- [ ] Bounce/complaint webhooks handled and stored

## Environment Variables

```bash
RESEND_API_KEY=re_...
EMAIL_FROM=noreply@yourdomain.com
```
