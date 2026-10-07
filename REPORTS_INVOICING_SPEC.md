# MPS OS — Reports & Invoicing Module Spec

## Context
MPS OS — React + Vite + Tailwind + Firebase
Firebase project: mps-systems (africa-south1)
Live: https://mps-systems.web.app

Read ALL existing files in src/ and functions/index.js and firestore.rules before writing anything. Never break existing functionality.

Brand tokens:
  Primary:   #1A1A2E
  Secondary: #FFCD00
  Accent:    #2D9CDB
  Surface:   #F8F9FB
  Border:    #E4E4E4
  White:     #FFFFFF
  Success:   #27AE60
  Warning:   #F2994A
  Danger:    #EB5757

Rules: all prices in cents, all Firestore queries have limit() and orgId filter, every catch calls logError(), npm run build zero errors before any deploy.

---

## FIRESTORE COLLECTIONS

### /invoiceCounter/{orgId}
- lastNumber (int) — seed at 2432 if document does not exist

### /invoices/{id}
- orgId (string)
- invoiceNumber (string e.g. "INV-002433")
- clientId (string)
- clientName (string)
- linkedReportId (string optional)
- linkedReportType (string: monthly|handover optional)
- subject (string)
- lineItems (array of { id, description, qty, rate, amount } — all numbers in cents)
- subtotal (int cents)
- vatEnabled (bool default false)
- vatAmount (int cents)
- total (int cents)
- notes (string default "Thanks for your business.")
- terms (string default "Bank Details — FNB Account Number: 63 021 904 007 Branch Code: 210 835 Reference: Invoice # or Full Name")
- status (string: draft|sent|paid|overdue)
- invoiceDate (string YYYY-MM-DD default today)
- dueDate (string YYYY-MM-DD default end of current month)
- paidAt (timestamp optional)
- paidMethod (string: EFT|Cash|Card optional)
- pdfUrl (string optional)
- createdAt, createdBy, updatedAt, updatedBy, isDeleted

### /reports/{id}
- orgId (string)
- type (string: monthly|handover)
- clientId (string)
- clientName (string)
- status (string: draft|sent)
- accountManager (string default "Prayer")

FOR MONTHLY TYPE:
- reportMonth (string YYYY-MM)
- channels (object):
  - googleAds:
    - enabled (bool default false)
    - stats: { totalSpend, budget, totalLeads, costPerLead, avgCtr, avgCpc, avgQualityScore }
    - leadBreakdown: { phoneCalls: { thisMonth, lastMonth }, phoneCallsAd: { thisMonth, lastMonth }, whatsapp: { thisMonth, lastMonth }, formSubmissions: { thisMonth, lastMonth } }
    - aiNarrative: { whatsWorking, whatWeAreImproving, nextMonthFocus }
  - seo:
    - enabled (bool default false)
    - stats: { organicSessions, keywordsRanking, top3Keywords, avgPosition, impressions, clicks }
    - aiNarrative: { whatsWorking, whatWeAreImproving, nextMonthFocus }
  - localSeo:
    - enabled (bool default false)
    - stats: { profileViews, calls, directionRequests, websiteClicks, reviews }
    - aiNarrative: { whatsWorking, whatWeAreImproving, nextMonthFocus }
  - socialMedia:
    - enabled (bool default false)
    - stats: { followers, reach, impressions, engagement, postsPublished }
    - aiNarrative: { whatsWorking, whatWeAreImproving, nextMonthFocus }
  - tiktok:
    - enabled (bool default false)
    - stats: { followers, views, likes, shares, profileVisits }
    - aiNarrative: { whatsWorking, whatWeAreImproving, nextMonthFocus }
  - websitePerformance:
    - enabled (bool default false)
    - stats: { sessions, users, bounceRate, avgSessionDuration, topPage }
    - aiNarrative: { whatsWorking, whatWeAreImproving, nextMonthFocus }
- linkedInvoiceId (string optional)

FOR HANDOVER TYPE:
- siteUrl (string)
- launchDate (string YYYY-MM-DD)
- pagesBuilt (array of { title, url })
- screenshots (array of { label, storagePath, url })
- howItWorksContent (string)
- whatMpsManagesContent (string)
- howToGetSupportContent (string)
- aiExtras (string)
- linkedInvoiceId (string optional)

NOTE: wpAdminUrl, wpUsername, wpPassword, hostingProvider, hostingCpanelUrl, hostingUsername, hostingPassword are NEVER saved to Firestore. They live in React state only and are passed directly to the PDF generation function.

- createdAt, createdBy, updatedAt, updatedBy, isDeleted

### /reportTemplates/{id}
- orgId (string)
- type (string: handover-how-it-works|handover-mps-manages|handover-support)
- content (string)
- createdAt, createdBy, updatedAt, updatedBy

---

## FIRESTORE RULES — add to existing, never remove existing rules

```
match /invoiceCounter/{id} {
  allow read, write: if isAuthenticated();
}
match /invoices/{id} {
  allow read: if isAuthenticated() && resource.data.orgId == orgId();
  allow create: if isAuthenticated() && request.resource.data.orgId == orgId();
  allow update: if isAuthenticated() && resource.data.orgId == orgId()
    && resource.data.status != 'paid';
}
match /reports/{id} {
  allow read: if isAuthenticated() && resource.data.orgId == orgId();
  allow create: if isAuthenticated() && request.resource.data.orgId == orgId();
  allow update: if isAuthenticated() && resource.data.orgId == orgId();
}
match /reportTemplates/{id} {
  allow read: if isAuthenticated() && resource.data.orgId == orgId();
  allow write: if isOwner();
}
```

---

## CLOUD FUNCTIONS — add to existing functions/index.js
All functions: 30s timeout, validate orgId, logError() in every catch.
Secrets via Firebase secrets never hardcoded.

### 1. getNextInvoiceNumber (callable)
Input: { orgId }
- Read /invoiceCounter/orgId
- If does not exist: create with lastNumber: 2432
- Increment lastNumber by 1
- Write back to Firestore
- Pad to 6 digits: 2433 → "INV-002433"
- Return { invoiceNumber: "INV-002433", lastNumber: 2433 }

### 2. generateInvoicePdf (callable)
Input: { invoiceId }
- Read invoice from Firestore
- Build PDF with jsPDF matching INV-002432 layout:

  TOP LEFT:
    MPS logo placeholder box (40x20mm grey box with "MPS" text if no logo)
    "Marketing Premium Solutions" bold 11pt
    "307 Paul Kruger Street, Capital Park" 9pt
    "Pretoria, South Africa Gauteng 0084" 9pt
    "marketingpremiumsolutions@gmail.com" 9pt

  TOP RIGHT:
    "INVOICE" 32pt bold
    "# [invoiceNumber]" 11pt
    "Balance Due" 9pt label
    "R [total formatted x,xxx.00]" 14pt bold

  MIDDLE (two columns):
    RIGHT ALIGNED:
      "Invoice Date : [invoiceDate]"
      "Terms : Due end of the month"
      "Due Date : [dueDate]"
    LEFT:
      "Bill To" 8pt grey label
      "[clientName]" 11pt bold
      blank line
      "Subject :" 8pt grey label
      "[subject]" 10pt

  LINE ITEMS TABLE:
    Headers: # | Item & Description | Qty | Rate | Amount
    Header row: light grey background
    Each row: row number | description | qty | "R x,xxx.00" | "R x,xxx.00"
    Thin horizontal line between each row

  TOTALS (right aligned):
    "Sub Total" | "R x,xxx.00"
    If vatEnabled: "VAT (15%)" | "R x,xxx.00"
    "Total" bold | "R x,xxx.00" bold
    "Balance Due" bold | "R x,xxx.00" bold

  NOTES:
    "Notes" heading 9pt grey
    notes content 10pt

  TERMS & CONDITIONS:
    "Terms & Conditions" heading 9pt grey
    terms content 9pt (includes FNB bank details)

  FOOTER:
    "POWERED BY MPS" 7pt centered grey

- Save to Firebase Storage: orgId/invoices/invoiceId/invoice.pdf
- Update invoice doc: set pdfUrl to download URL
- Return { success: true, downloadUrl }

### 3. generateReportNarrative (callable)
Input: { reportId, channel }
- Read report from Firestore
- Get channel stats, clientName, reportMonth
- Map channel key to display name:
  googleAds → "Google Ads", seo → "SEO", localSeo → "Local SEO",
  socialMedia → "Social Media", tiktok → "TikTok",
  websitePerformance → "Website Performance"
- Call Claude API claude-sonnet-4-6 max_tokens 800
  API key from Firebase secret CLAUDE_API_KEY
- Prompt:
  "You are a digital marketing account manager at Marketing Premium Solutions
  in South Africa. Write a monthly report narrative for [clientName]
  for [channelDisplayName] performance in [reportMonth].
  Stats: [JSON.stringify(channelStats)]
  Return ONLY valid JSON with no markdown fences or other text:
  {
    whatsWorking: '2-3 sentences positive and specific to the numbers',
    whatWeAreImproving: '2-3 sentences honest and constructive',
    nextMonthFocus: '2-3 sentences forward-looking and actionable'
  }"
- Strip any ```json fences before parsing
- Update report.channels[channel].aiNarrative in Firestore using dot notation
- Return { success: true, narrative: { whatsWorking, whatWeAreImproving, nextMonthFocus } }

### 4. generateMonthlyReportPdf (callable)
Input: { reportId }
- Read full report from Firestore
- Get list of enabled channels in order:
  googleAds, seo, localSeo, socialMedia, tiktok, websitePerformance
- Build PDF with jsPDF:

  PAGE 1 — COVER:
    Full page rectangle navy #1A1A2E
    Top left: "MARKETING PREMIUM SOLUTIONS" white 9pt
    Top right: "MONTHLY REPORT" gold #FFCD00 9pt
    Center vertical middle: clientName white bold 36pt
    Below clientName: formatted reportMonth (e.g. "September 2026") white 18pt
    Bottom left: "Prepared by [accountManager]" white 9pt

  FOR EACH ENABLED CHANNEL:
    New page
    Section heading: channel display name, 18pt bold, color #1A1A2E
    Gold underline rule below heading

    STATS GRID (2 columns, 3 rows of stat boxes):
      Each box: label 8pt grey above, value bold 16pt below
      Google Ads stats: Total Spend | Budget Used | Total Leads |
        Cost per Lead | Avg CTR | Avg Quality Score
      SEO stats: Organic Sessions | Keywords Ranking | Avg Position |
        Impressions | Clicks | Top 3 Keywords
      Local SEO stats: Profile Views | Calls | Direction Requests |
        Website Clicks | Reviews | (blank)
      Social Media stats: Followers | Reach | Impressions |
        Engagement | Posts Published | (blank)
      TikTok stats: Followers | Views | Likes | Shares | Profile Visits | (blank)
      Website stats: Sessions | Users | Bounce Rate |
        Avg Session Duration | Top Page | (blank)

    IF GOOGLE ADS: LEAD BREAKDOWN TABLE below stats grid
      Heading: "Lead Breakdown — This Month vs Last Month" 11pt bold
      Headers: Source | This Month | Last Month | Change
      Rows:
        Phone calls (website) | thisMonth | lastMonth | calculated diff
        Phone calls (from ad) | thisMonth | lastMonth | diff
        WhatsApp messages | thisMonth | lastMonth | diff
        Form submissions | thisMonth | lastMonth | diff
        Total | sum | sum | diff
      Change column: green if positive, red if negative

    AI NARRATIVE (if aiNarrative exists for this channel):
      "What's Working" 11pt bold gold | whatsWorking text 10pt
      "What We're Improving" 11pt bold | whatWeAreImproving text 10pt
      "Next Month Focus" 11pt bold | nextMonthFocus text 10pt

  LAST PAGE:
    "Thank you for your business." centered 18pt bold
    blank line
    MPS contact block centered:
      "065 221 9944"
      "office@marketingpremiumsolutions.com"
      "wa.me/27652219944"
    If linkedInvoiceId exists:
      "Your invoice for this period has been sent separately."

- Save to Storage: orgId/reports/reportId/report.pdf
- Return { success: true, downloadUrl }

### 5. generateHandoverExtras (callable)
Input: { reportId, siteUrl, pageCount }
- Call Claude API claude-sonnet-4-6 max_tokens 300
- Prompt:
  "Write a warm professional welcome paragraph for a South African small
  business client whose new website just launched at [siteUrl]. The site
  has [pageCount] pages and was built by Marketing Premium Solutions.
  Keep it encouraging, practical, and under 120 words. Plain text only,
  no bullet points."
- Return { success: true, extras: string }
  Caller saves aiExtras to Firestore — this function does not write to Firestore

### 6. generateHandoverReportPdf (callable)
Input: {
  reportId,
  wpAdminUrl, wpUsername, wpPassword,
  hostingProvider, hostingCpanelUrl, hostingUsername, hostingPassword
}
IMPORTANT: passwords come from client state — never read from or write to Firestore
- Read report from Firestore (all fields except login credentials)
- Build PDF with jsPDF:

  PAGE 1 — COVER:
    Full navy background
    "WEBSITE HANDOVER REPORT" white 28pt bold centered vertically
    clientName white 22pt centered below title
    siteUrl white 14pt centered
    "Launch Date: [launchDate]" white 12pt centered
    "Prepared by Marketing Premium Solutions" white 9pt bottom left

  PAGE 2 — WHAT WE BUILT:
    "What We Built" heading 18pt bold
    Pages list: each page on its own row — title bold + url grey below
    If screenshots exist: embed each image with its label below
      Load each from storagePath using Firebase Admin Storage

  PAGE 3 — YOUR LOGIN DETAILS:
    Yellow warning box (#F2994A background):
      "Important: Save these details somewhere safe.
      MPS does not store your passwords after this report is delivered."
    
    "WordPress Admin" 14pt bold heading
    Table rows:
      "Admin URL" | wpAdminUrl
      "Username" | wpUsername
      "Password" | wpPassword
    
    blank space
    
    "[hostingProvider] Hosting" 14pt bold heading
    Table rows:
      "Control Panel URL" | hostingCpanelUrl
      "Username" | hostingUsername
      "Password" | hostingPassword

  PAGE 4 — HOW YOUR WEBSITE WORKS:
    "How Your Website Works" 18pt bold heading
    howItWorksContent text 10pt

  PAGE 5 — WHAT MPS MANAGES:
    "What MPS Manages" 18pt bold heading
    whatMpsManagesContent text 10pt

  PAGE 6 — HOW TO GET SUPPORT:
    "How To Get Support" 18pt bold heading
    howToGetSupportContent text 10pt
    blank line
    MPS contact block:
      "065 221 9944"
      "office@marketingpremiumsolutions.com"
      "WhatsApp: wa.me/27652219944"
      "Office hours: Monday to Thursday 08:00 to 17:00"

  PAGE 7 — WELCOME NOTE (only if aiExtras is not empty):
    "A Note From MPS" 18pt bold heading
    aiExtras text 10pt

- Save to Storage: orgId/reports/reportId/handover.pdf
- Return { success: true, downloadUrl }

---

## SEED DATA

On mount of HandoverReportPage, call seedTemplatesIfEmpty(db, orgId, userId):
- Check /reportTemplates where orgId == orgId — if any exist return early
- Batch write these three documents:

Document 1:
  type: 'handover-how-it-works'
  content: "Your website is built on WordPress, the world's most popular website platform. Here is what you need to know:\n\n• Your site is hosted on a secure server and backed up automatically every day\n• WordPress updates are managed by MPS every month\n• You can log in to your WordPress dashboard to view enquiries submitted through your contact forms\n• Your site is fully responsive — optimised for mobile phones, tablets, and desktop computers\n• SSL security is active — your site runs on https:// which builds trust with visitors and helps with Google rankings\n• Your domain name is registered and will be renewed annually — MPS will notify you 30 days before renewal"

Document 2:
  type: 'handover-mps-manages'
  content: "As part of your Hosting and Maintenance plan, MPS handles the following every month:\n\n• WordPress core, plugin, and theme updates\n• Daily automated backups stored securely off-site on Google Drive\n• Monthly security scans via Wordfence\n• SSL certificate monitoring and renewal\n• 1 hour of minor content edits per month (text changes, image swaps)\n• Uptime monitoring — if your site goes down, we are notified before you are\n• Monthly performance check to keep your site loading fast"

Document 3:
  type: 'handover-support'
  content: "Need help with your website? Here is how to reach us:\n\n• WhatsApp (fastest response): wa.me/27652219944\n• Email: office@marketingpremiumsolutions.com\n• Phone: 065 221 9944\n• Office hours: Monday to Thursday 08:00 to 17:00\n• Emergency (site down): WhatsApp us immediately — 2-hour response SLA\n• Content updates beyond your included monthly hour are billed at R750 per revision round\n• For new pages or major changes, we will quote you before starting work"

All seed docs: orgId, createdBy: 'system', createdAt: serverTimestamp(),
updatedAt: serverTimestamp(), updatedBy: 'system', isDeleted: false

---

## SCREENS TO BUILD

### src/divisions/finance/invoices/InvoicesListPage.jsx — route /invoices
- List invoices newest first limit(50) orgId filtered
- Outstanding total banner at top: "Outstanding: R xx,xxx.00"
  (sum of subtotal for all invoices where status != 'paid')
- Columns: Invoice # | Client | Subject | Total | Status badge | Date | Due Date
- Status badges: Draft grey | Sent blue | Paid green | Overdue red
- Filter tabs: All | Draft | Sent | Paid | Overdue
- New Invoice button top right → opens New Invoice modal
- Row click → /invoices/:id
- Empty state: "No invoices yet — create your first one"

New Invoice Modal (2 steps):
  Step 1:
    Client dropdown (searchable, from /clients collection)
    Optional: Link to report — dropdown of /reports for selected client
      (loads reports for that clientId when client is selected)
  Step 2:
    Subject text input (e.g. "Monthly Google Ads Services — October 2026")
    Invoice date (date input, default today formatted YYYY-MM-DD)
    Due date (date input, default last day of current month)
    VAT toggle (default off)
  On Create:
    Call getNextInvoiceNumber
    addDoc to /invoices with all fields + status: draft
    Navigate to /invoices/:id

### src/divisions/finance/invoices/InvoiceDetailPage.jsx — route /invoices/:id
Header row:
  Left: invoice number large | status badge | client name
  Right: Generate PDF button | Mark as Sent button | Mark as Paid button

Mark as Sent: sets status to 'sent' with confirmation
Mark as Paid: opens modal with payment date (default today) +
  payment method dropdown (EFT / Cash / Card) → sets status 'paid',
  saves paidAt and paidMethod
Once status is 'paid': entire form is read-only, buttons hidden except Generate PDF

Linked report chip below header if linkedReportId exists
  Click navigates to that report

FORM (editable until paid):
  Subject input
  Invoice date + due date inputs
  VAT toggle — when on, adds VAT 15% line to totals automatically

  LINE ITEMS TABLE:
    Column headers: # | Description | Qty | Unit Price (R) | Amount (R) | (remove)
    Each row:
      row number (auto)
      description text input
      qty number input
      unit price number input (display in rands, store in cents)
      amount (auto = qty × unitPrice, read only)
      remove button (trash icon)
    "Add line item" button below table — adds blank row

  Notes textarea (label "Notes", default "Thanks for your business.")
  Terms textarea (label "Terms & Conditions",
    default "Bank Details — FNB Account Number: 63 021 904 007 Branch Code: 210 835 Reference: Invoice # or Full Name")

SUMMARY PANEL (right side or below table):
  Sub Total: R xx,xxx.00
  VAT (15%): R xx,xxx.00 (only shows if vatEnabled)
  Total: R xx,xxx.00 bold
  Balance Due: R xx,xxx.00 bold larger
  All update live as line items change

PDF download link appears below Generate PDF button once generated

### src/divisions/reports/ReportsListPage.jsx — route /reports
- List reports newest first limit(50) orgId filtered
- Columns: Type badge | Client | Period or Title | Status badge | Created
- Type badges: Monthly blue (#2D9CDB) | Handover gold (#FFCD00 with dark text)
- Status badges: Draft grey | Sent green
- Filter tabs: All | Monthly | Handover
- New Report button top right → opens New Report modal
- Row click → /reports/:id for monthly, /reports/:id/handover for handover
- Empty state: "No reports yet"

New Report Modal:
  Type selector (two large cards to click): 
    "Monthly Marketing Report" | "Website Handover Report"
  Client dropdown (searchable)
  If Monthly selected: report month input (type="month" YYYY-MM)
  If Handover selected: site URL text input + launch date input
  Create button → addDoc /reports → navigate to correct detail page

### src/divisions/reports/MonthlyReportPage.jsx — route /reports/:id
SPLIT SCREEN layout:
  Left 50%: form (scrollable)
  Right 50%: live preview (scrollable, fixed position)

LEFT — FORM:
  Top bar:
    Client name (display only)
    Report month (display, e.g. "September 2026")
    Account manager text input (default "Prayer")
    Save Draft button | Link Invoice dropdown (invoices for this client)

  CHANNEL TOGGLES — horizontal row of 6 toggle switches:
    [ Google Ads ] [ SEO ] [ Local SEO ] [ Social Media ] [ TikTok ] [ Website ]
    Each toggle: label above, switch below
    Toggling on shows that channel section, off hides it completely

  GOOGLE ADS SECTION (shows when toggled on):
    Heading: "Google Ads" with blue dot indicator
    Stats grid (2 columns):
      Total Spend (R) | Budget (R)
      Total Leads | Cost per Lead (R)
      Avg CTR (%) | Avg CPC (R)
      Avg Quality Score (/10) | (blank)
    Lead Breakdown subsection:
      Subheading: "Lead Breakdown — This Month vs Last Month"
      4 rows each with This Month + Last Month inputs:
        Phone calls (website click)
        Phone calls (from ad)
        WhatsApp messages
        Form submissions
    "Generate AI Narrative" button (primary, calls generateReportNarrative channel:'googleAds')
    Loading state: "Generating narrative..."
    After generation — 3 editable textareas:
      "What's Working" | textarea with whatsWorking value
      "What We're Improving" | textarea with whatWeAreImproving value
      "Next Month Focus" | textarea with nextMonthFocus value

  SEO SECTION (shows when toggled on):
    Stats (2 columns):
      Organic Sessions | Keywords Ranking
      Avg Position | Impressions
      Clicks | Top 3 Keywords (text input)
    Generate AI Narrative button
    3 editable narrative textareas after generation

  LOCAL SEO SECTION (shows when toggled on):
    Stats (2 columns):
      Profile Views | Calls
      Direction Requests | Website Clicks
      Reviews | (blank)
    Generate AI Narrative button
    3 editable narrative textareas after generation

  SOCIAL MEDIA SECTION (shows when toggled on):
    Stats (2 columns):
      Followers | Reach
      Impressions | Engagement (%)
      Posts Published | (blank)
    Generate AI Narrative button
    3 editable narrative textareas after generation

  TIKTOK SECTION (shows when toggled on):
    Stats (2 columns):
      Followers | Views
      Likes | Shares
      Profile Visits | (blank)
    Generate AI Narrative button
    3 editable narrative textareas after generation

  WEBSITE PERFORMANCE SECTION (shows when toggled on):
    Stats (2 columns):
      Sessions | Users
      Bounce Rate (%) | Avg Session Duration
      Top Page (text) | (blank)
    Generate AI Narrative button
    3 editable narrative textareas after generation

  Bottom of form:
    Save Draft button
    Generate Report PDF button → calls generateMonthlyReportPdf → shows download link

RIGHT — LIVE PREVIEW:
  Scrollable panel, updates as form values change
  Cover section: navy background, client name large, report month below
  For each enabled channel:
    Section heading (channel name)
    Stats displayed in a 2x3 grid of stat boxes
    Lead breakdown table if Google Ads
    Narrative paragraphs if aiNarrative exists
  Thank you footer at bottom

### src/divisions/reports/HandoverReportPage.jsx — route /reports/:id/handover
Single column form (no split screen)

Header: "Website Handover Report" | client name | status badge
Save Draft button top right

SECTION 1 — Site Details:
  Client name (display)
  Site URL (text input)
  Launch date (date input)

SECTION 2 — Pages Built:
  Dynamic list — each row: Page Title input | URL input | remove button
  Add Page button (adds blank row)
  Import from Project button:
    Searches /websiteProjects where clientId matches
    If found: shows modal to select project, imports pagesBuilt array

SECTION 3 — Screenshots (up to 10):
  Grid of upload boxes (2 columns)
  Each box:
    Label text input (e.g. "Homepage", "About Us")
    Upload area (click or drag) → uploads to Firebase Storage
      Path: orgId/reports/reportId/screenshots/screenshot-N
    Thumbnail preview once uploaded
    Remove button
  Add Screenshot button (up to 10 max)

SECTION 4 — Login Details:
  Yellow warning banner (#F2994A light background):
    "These details will appear in the PDF only. They are never saved to the MPS system."
  WordPress subsection:
    Admin URL | Username | Password (all text inputs)
    Password field has show/hide toggle
  Hosting subsection:
    Provider name | Control Panel URL | Username | Password
    Password field has show/hide toggle
  These fields are React state only — NEVER included in any Firestore write

SECTION 5 — How Your Website Works:
  Label + large textarea
  Pre-filled on mount from /reportTemplates where type == 'handover-how-it-works'
  "Reset to template" button (restores original template content)
  "Enhance with AI" button → calls generateHandoverExtras →
    appends result to bottom of textarea

SECTION 6 — What MPS Manages:
  Label + large textarea
  Pre-filled from handover-mps-manages template
  Reset to template button

SECTION 7 — How To Get Support:
  Label + large textarea
  Pre-filled from handover-support template
  Reset to template button

SECTION 8 — Final Invoice:
  Toggle: "Generate final invoice with this report" (default on)
  If toggle on, show:
    Invoice subject input (default "Final Payment — [clientName] Website")
    Amount input (in rands e.g. 4500, converted to cents on save)
  On PDF generation:
    If toggle on: call getNextInvoiceNumber, create invoice doc,
      link invoiceId to report (update linkedInvoiceId)

Bottom action bar:
  Save Draft button
  Generate Handover Report PDF button:
    Saves current form state to Firestore (except login fields)
    If invoice toggle on: creates invoice
    Calls generateHandoverReportPdf with all fields including login state
    Shows PDF download link on success
    Shows invoice link if invoice was created

---

## NAVIGATION UPDATE
Add to src/components/Sidebar.jsx — new section "Finance & Reports":
  Invoices → /invoices  (icon: document/receipt)
  Reports → /reports    (icon: chart/clipboard)

---

## ROUTER UPDATE
Add to src/App.jsx (or wherever routes are defined):
  /invoices              → InvoicesListPage
  /invoices/:id          → InvoiceDetailPage
  /reports               → ReportsListPage
  /reports/:id           → MonthlyReportPage
  /reports/:id/handover  → HandoverReportPage

---

## ACCEPTANCE CRITERIA — report true/false for each

- /invoices renders and lists from Firestore
- Outstanding total calculates correctly at top
- New Invoice modal: client selector works, report link optional,
  invoice number auto-increments from INV-002432 (first = INV-002433)
- VAT toggle off by default, when on adds VAT 15% to totals
- /invoices/:id line items editable, totals update live
- Notes and terms editable
- Mark as Sent sets status correctly
- Mark as Paid opens modal, saves paidAt and paidMethod, form goes read-only
- Invoice PDF generates matching INV-002432 layout, download link works
- /reports renders and lists both report types with correct badges
- New Report modal: both types create and navigate correctly
- Monthly report: all 6 channel toggles show and hide sections correctly
- Monthly stats save to Firestore on Save Draft
- Generate AI Narrative calls function, result appears in editable textareas
- Live preview updates as stats are typed
- Monthly report PDF generates and downloads
- /reports/:id/handover all 8 sections render
- Screenshot upload saves to Firebase Storage, thumbnail shows
- Login details stay in React state only — NOT written to Firestore
- Handover templates pre-fill correctly from seeded data
- Reset to template button restores original content
- Final invoice creates automatically when toggle is on
- Handover PDF generates with login details from state, downloads
- Sidebar shows Finance and Reports section with both links
- All 5 routes work correctly
- All Firestore queries have limit() and orgId filter
- Every catch block calls logError()
- npm run build zero errors
- firebase deploy rules indexes functions hosting all clean

---

## DEPLOY ORDER
1. git add . && git commit -m "finance reports invoicing module"
2. npm run build — fix all errors, zero tolerance
3. firebase deploy --only firestore:rules
4. firebase deploy --only firestore:indexes
5. $env:FUNCTIONS_DISCOVERY_TIMEOUT=60; firebase deploy --only functions
6. firebase deploy --only hosting
7. Report true/false for every acceptance criteria item above
