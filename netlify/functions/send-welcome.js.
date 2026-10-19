'use strict';

const nodemailer = require('nodemailer');

// ---- Fixed, non-secret settings -------------------------------------------
const SITE_URL = 'https://desktwo.co.uk';
const REPLY_TO = 'team@desktwo.co.uk';
const LOGO_URL = SITE_URL + '/img/desk-two-logo-email.png';
const PRIVACY_URL = SITE_URL + '/privacy.html';

// ---- Helpers ----------------------------------------------------------------
const EMAIL_RE = /^[^\s@<>()[\]\\,;:"]+@[^\s@<>()[\]\\,;:"]+\.[A-Za-z]{2,}$/;

const json = (status, body, extra = {}) => ({
  statusCode: status,
  headers: { 'Content-Type': 'application/json', 'Cache-Control': 'no-store', ...extra },
  body: JSON.stringify(body),
});

const esc = (s) =>
  String(s).replace(/[&<>"']/g, (c) => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c]));

// Simple in-memory rate limiting. It resets when the function instance restarts,
// so treat it as a speed bump for casual abuse, not a hard guarantee.
const buckets = new Map();
function tooMany(key, limit, windowMs) {
  const now = Date.now();
  const fresh = (buckets.get(key) || []).filter((t) => now - t < windowMs);
  if (fresh.length >= limit) {
    buckets.set(key, fresh);
    return true;
  }
  fresh.push(now);
  buckets.set(key, fresh);
  if (buckets.size > 5000) {
    for (const [k, v] of buckets) if (!v.some((t) => now - t < windowMs)) buckets.delete(k);
  }
  return false;
}

// ---- Email 1: welcome email to the visitor -----------------------------------
function welcomeSubject() {
  return 'Your Desk Two booking link';
}

function welcomeText(bookingUrl) {
  return [
    'Hello,',
    '',
    "Thank you for getting in touch with Desk Two. We're Millie and Jack, and we'd love to hear about you, your week and where an extra pair of hands would help most.",
    '',
    'Choose a time that suits you for a short, no-pressure introductory call:',
    bookingUrl,
    '',
    "Prefer to write first? Just reply to this email and one of us will come straight back to you.",
    '',
    'Warm regards,',
    'Millie & Jack',
    'Desk Two - more time for what matters',
    '',
    '---',
    "You're receiving this one-off email because you asked for our booking link at desktwo.co.uk. We've used your email address only to send it and to follow up on your enquiry.",
    'Privacy policy: ' + PRIVACY_URL,
    'Questions or want your details removed? Email ' + REPLY_TO,
    'Desk Two, United Kingdom',
  ].join('\n');
}

function welcomeHtml(bookingUrl) {
  const u = esc(bookingUrl);
  return `<!DOCTYPE html>
<html lang="en-GB"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<meta name="color-scheme" content="light"><title>Your Desk Two booking link</title></head>
<body style="margin:0;padding:0;background:#fffeef;">
<div style="display:none;max-height:0;overflow:hidden;opacity:0;color:#fffeef;">Pick a time that suits you. We'd love to talk.</div>
<table role="presentation" width="100%" cellpadding="0" cellspacing="0" style="background:#fffeef;">
<tr><td align="center" style="padding:24px 12px;">
  <table role="presentation" width="560" cellpadding="0" cellspacing="0" style="width:100%;max-width:560px;background:#ffffff;border-radius:16px;overflow:hidden;border:1px solid #e3e8d8;">
    <tr><td align="center" style="background:#caf5f7;padding:26px 20px;">
      <img src="${LOGO_URL}" width="240" alt="Desk Two" style="display:block;width:240px;max-width:100%;height:auto;border:0;">
    </td></tr>
    <tr><td style="padding:36px 36px 8px 36px;font-family:Georgia,'Times New Roman',serif;color:#003d20;">
      <h1 style="margin:0;font-size:32px;line-height:1.15;font-weight:normal;">Lovely to meet you, <em>let&rsquo;s find a time.</em></h1>
    </td></tr>
    <tr><td style="padding:12px 36px 8px 36px;font-family:Helvetica,Arial,sans-serif;font-size:16px;line-height:1.65;color:#2b3a35;">
      <p style="margin:0 0 16px 0;">Hello,</p>
      <p style="margin:0 0 16px 0;">Thank you for getting in touch with Desk Two. We&rsquo;re Millie and Jack, and we&rsquo;d love to hear about you, your week and where an extra pair of hands would help most.</p>
      <p style="margin:0 0 24px 0;">Choose a time that suits you for a short, no-pressure introductory call:</p>
    </td></tr>
    <tr><td align="center" style="padding:0 36px 12px 36px;">
      <table role="presentation" cellpadding="0" cellspacing="0"><tr><td align="center" bgcolor="#003d20" style="border-radius:99px;">
        <a href="${u}" style="display:inline-block;padding:16px 32px;font-family:Helvetica,Arial,sans-serif;font-size:16px;font-weight:bold;color:#caf5f7;text-decoration:none;border-radius:99px;">Book your introductory call</a>
      </td></tr></table>
    </td></tr>
    <tr><td style="padding:12px 36px 8px 36px;font-family:Helvetica,Arial,sans-serif;font-size:13px;line-height:1.6;color:#5a6b64;text-align:center;">
      Button not working? Copy this link into your browser:<br><a href="${u}" style="color:#003d20;word-break:break-all;">${u}</a>
    </td></tr>
    <tr><td style="padding:20px 36px 32px 36px;font-family:Helvetica,Arial,sans-serif;font-size:16px;line-height:1.65;color:#2b3a35;">
      <p style="margin:0 0 16px 0;">Prefer to write first? Just reply to this email and one of us will come straight back to you.</p>
      <p style="margin:0;">Warm regards,<br><strong style="color:#003d20;">Millie &amp; Jack</strong><br><span style="color:#5a6b64;">Desk Two &middot; more time for what matters</span></p>
    </td></tr>
    <tr><td style="background:#f4f8ef;border-top:1px solid #e3e8d8;padding:20px 36px;font-family:Helvetica,Arial,sans-serif;font-size:12px;line-height:1.6;color:#5a6b64;">
      You&rsquo;re receiving this one-off email because you asked for our booking link at desktwo.co.uk. We&rsquo;ve used your email address only to send it and to follow up on your enquiry.<br>
      <a href="${PRIVACY_URL}" style="color:#003d20;">Privacy policy</a> &middot; Questions, or want your details removed? Email <a href="mailto:${REPLY_TO}" style="color:#003d20;">${REPLY_TO}</a><br>
      Desk Two, United Kingdom
    </td></tr>
  </table>
</td></tr></table>
</body></html>`;
}

// ---- Handler -----------------------------------------------------------------
exports.handler = async (event) => {
  if (event.httpMethod !== 'POST') {
    return json(405, { ok: false, error: 'Method not allowed.' }, { Allow: 'POST' });
  }

  const raw = event.body || '';
  if (raw.length > 2048) return json(413, { ok: false, error: 'Request too large.' });

  let data;
  try {
    data = JSON.parse(raw || '{}');
  } catch (e) {
    return json(400, { ok: false, error: 'Invalid request.' });
  }
  if (!data || typeof data !== 'object') return json(400, { ok: false, error: 'Invalid request.' });

  // Honeypot: real visitors never fill this in. Pretend success so bots move on.
  if (typeof data.website === 'string' && data.website.trim() !== '') {
    return json(200, { ok: true });
  }

  const email = typeof data.email === 'string' ? data.email.trim().toLowerCase() : '';
  if (!email || email.length > 254 || !EMAIL_RE.test(email)) {
    return json(400, { ok: false, error: 'Please enter a valid email address.' });
  }
  if (data.consent !== true) {
    return json(400, { ok: false, error: 'Please confirm you are happy for us to email you.' });
  }

  const h = event.headers || {};
  const ip = h['x-nf-client-connection-ip'] || (h['x-forwarded-for'] || '').split(',')[0].trim() || 'unknown';
  if (tooMany('ip:' + ip, 5, 10 * 60 * 1000) || tooMany('em:' + email, 2, 60 * 60 * 1000)) {
    return json(429, { ok: false, error: 'Too many requests. Please wait a few minutes and try again.' }, { 'Retry-After': '600' });
  }

  const required = ['GMAIL_USER', 'GMAIL_APP_PASSWORD', 'BOOKING_URL', 'NOTIFY_EMAIL'];
  const missing = required.filter((k) => !process.env[k]);
  if (missing.length) {
    console.error('Missing environment variables:', missing.join(', '));
    return json(500, { ok: false, error: 'The form is not set up yet. Please email ' + REPLY_TO + '.' });
  }
  const { GMAIL_USER, GMAIL_APP_PASSWORD, BOOKING_URL, NOTIFY_EMAIL } = process.env;
  if (!/^https:\/\//i.test(BOOKING_URL.trim())) {
    console.error('BOOKING_URL must start with https://');
    return json(500, { ok: false, error: 'The form is not set up yet. Please email ' + REPLY_TO + '.' });
  }
  const bookingUrl = BOOKING_URL.trim();

  const transporter = nodemailer.createTransport({
    host: 'smtp.gmail.com',
    port: 465,
    secure: true,
    auth: { user: GMAIL_USER.trim(), pass: GMAIL_APP_PASSWORD.replace(/\s+/g, '') },
    connectionTimeout: 6000,
    greetingTimeout: 6000,
    socketTimeout: 8000,
  });

  // Email 1: the visitor's welcome email. If this fails, tell the visitor.
  try {
    await transporter.sendMail({
      from: `"Desk Two" <${GMAIL_USER.trim()}>`,
      replyTo: REPLY_TO,
      to: email,
      subject: welcomeSubject(),
      text: welcomeText(bookingUrl),
      html: welcomeHtml(bookingUrl),
    });
  } catch (err) {
    console.error('Welcome email failed:', err && err.code, err && err.responseCode, err && err.message);
    return json(502, { ok: false, error: "We couldn't send your email just now. Please try again." });
  }

  // Email 2: internal heads-up. A failure here must not affect the visitor.
  try {
    const when = new Date().toLocaleString('en-GB', { timeZone: 'Europe/London', dateStyle: 'full', timeStyle: 'short' });
    await transporter.sendMail({
      from: `"Desk Two website" <${GMAIL_USER.trim()}>`,
      to: NOTIFY_EMAIL.trim(),
      subject: 'New booking-link request from the website',
      text: `Someone asked for the booking link.\n\nEmail: ${email}\nTime (UK): ${when}\n\nThe welcome email has been sent. Nothing is stored on the website; this message is the only record.`,
      html: `<p style="font-family:Helvetica,Arial,sans-serif;font-size:15px;color:#2b3a35;">Someone asked for the booking link.</p><p style="font-family:Helvetica,Arial,sans-serif;font-size:15px;color:#2b3a35;"><strong>Email:</strong> ${esc(email)}<br><strong>Time (UK):</strong> ${esc(when)}</p><p style="font-family:Helvetica,Arial,sans-serif;font-size:13px;color:#5a6b64;">The welcome email has been sent. Nothing is stored on the website; this message is the only record.</p>`,
    });
  } catch (err) {
    console.error('Notification email failed:', err && err.code, err && err.message);
  }

  return json(200, { ok: true });
};
