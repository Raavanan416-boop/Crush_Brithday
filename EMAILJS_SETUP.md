# Email setup (EmailJS + Gmail) — required, otherwise nothing is emailed

The site never sends anything unless you configure this. Until then, "Send Wish" saves the wish on the device
and shows "Email isn't set up yet" (it does NOT pretend it was sent, and the relationship popup does not appear).

## Values to replace in `index.html` (search for `EMAIL_CFG`)
| Constant | Where to find it | Safe in browser? |
|---|---|---|
| `publicKey` | EmailJS → Account → General → **Public Key** | Yes (public by design) |
| `serviceId` | EmailJS → Email Services → your Gmail service | Yes |
| `wishTemplateId` | EmailJS → Email Templates → Template 1 | Yes |
| `relationTemplateId` | EmailJS → Email Templates → Template 2 | Yes |
| `recipientEmail` | The Gmail address that should receive the emails | Visible in page source |

Never put an EmailJS **Private Key**, Gmail password, or app password in `index.html`.
Gmail is connected through EmailJS's Google sign-in (OAuth), so no password is stored in your code.

## Steps
1. Create an account at https://www.emailjs.com.
2. **Email Services → Add New Service → Gmail** → connect your Gmail. Copy the *Service ID*.
3. **Email Templates → Create New Template** (make two). In each, set **To Email** to your recipient address
   (recommended: hard-code it here so nobody can redirect emails; `{{to_email}}` also works and is sent by the app).
4. Template 1 (wish) — Subject: `{{subject}}`  (value sent: "💌 A Birthday Wish Has Been Submitted!")
   ```
   <h2>💌 A Birthday Wish Has Been Submitted!</h2>
   <p>Someone just finished your Birthday Surprise website and left this wish:</p>
   <blockquote>{{wish_text}}</blockquote>
   <p>Submitted: {{submitted_at}}</p>
   ```
5. Template 2 (relationship) — Subject: `{{subject}}`  (value sent: "💗 Birthday Surprise — Relationship Answer")
   ```
   <h2>💗 Relationship Answer</h2>
   <p><b>Selected:</b> {{relationship}}</p>
   <p><b>Message they saw:</b> {{relationship_message}}</p>
   <p><b>Their birthday wish:</b> {{wish_text}}</p>
   <p>Submitted: {{submitted_at}}</p>
   ```
6. Copy the two Template IDs and your Public Key into `EMAIL_CFG`.
7. Abuse protection: because the public key is visible, in EmailJS → Account → Security restrict allowed domains to your site,
   and keep "To Email" fixed in the templates. Free plan = 200 emails/month.
8. Host over HTTPS (or `http://localhost` with Live Server) — see below.

## Testing
1. Open the site (Live Server on PC, or your deployed HTTPS link on phone — `127.0.0.1` on a phone points to the phone itself).
2. Go to Page 5, leave the box empty → tap Send → you should see "Please type your wish first".
3. Type a wish → Send. Wait for "Wish sent with love!" (only appears after EmailJS returns success). Check the inbox for Email 1.
   Browser DevTools → Application → Local Storage should show `crush_birthday_wish` and `crush_birthday_wish_at`.
4. Tap "Yay!" → Home → popup appears after 2 seconds. Submit stays disabled until an option is chosen.
5. Choose an option → Submit → message card + "✅ delivered". Check the inbox for Email 2. localStorage has `crush_birthday_relationship`.
6. Failure tests: turn on airplane mode and Send → friendly error, wish kept, button changes to "Retry Sending".
   Put a wrong template ID → error shown, nothing claimed as sent. Double-tap Send quickly → only one email.
7. The popup does not reappear in the same browser session (sessionStorage flag `bs_rel_done`); close the tab to reset.

## Using EmailJS's default "Contact Us" template
The app now also sends the variables that template uses: `{{title}}` (email subject text), `{{name}}`, `{{time}}`, and `{{subject}}`
(which carries the full wish / relationship text), so the default template shows the real content with no dashboard changes.
For a cleaner email, edit the template: Subject = `{{title}}`, and in the content use `{{wish_text}}`, `{{relationship}}`,
`{{relationship_message}}`, `{{submitted_at}}` as shown above.
