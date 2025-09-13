# Facebook Page API Setup Guide

## Quick Summary (One Line)

Create Page → Create App → generate user token with pages scopes → GET /me/accounts → copy access_token from JSON (that is the Page token) → paste Page token into blue Access Token box → POST /{page_id}/subscribed_apps → then post with POST /{page_id}/feed or /photos.

## Full Steps (Do These in Order)

### 1) Create the Facebook Page

Facebook → Create → Page → fill name/category.

Open Page → Settings → Page Access / Page Roles (or Page Access in New Pages Experience) → confirm your personal profile is listed as Admin (Full Control).

If your Page is owned by a Business, add your personal profile as a person with Facebook access (Full Control).

### 2) Create a Meta App

developers.facebook.com → My Apps → Create App → choose Business.

Note App ID and App Secret (don't paste the secret anywhere public).

In App → add products you need (e.g., Instagram, Messenger if required).

### 3) Use Graph Explorer to Generate a User Token (For Your App)

Open Graph API Explorer.

Top-right: select your app from the Meta App dropdown.

Set User or Page to User Token.

**Permissions (check these minimal ones):**
- pages_show_list
- pages_manage_posts
- pages_read_engagement
- (if IG: instagram_basic, instagram_content_publish)

Click Generate Access Token and complete the Facebook auth popup.

### 4) Get Your Page ID and the Page Token

With the user token active, run:

```
GET /me/accounts
```

Look inside the JSON data array. Copy these two values from the Page object:
- `id` → this is `{page_id}`
- `access_token` → this is the Page access token (IMPORTANT: this is the token you will use to post & subscribe the app)

**Important:** the token in the top-right blue box is the user token. The token inside the /me/accounts JSON is the Page token. Use the Page token for page management.

### 5) Install / Connect Your App to the Page (We Used the API)

**Crucial Graph Explorer trick:** paste the Page token into the blue Access Token box (top-right) so Graph Explorer uses it for the request. Don't rely on putting access_token in Params only.

Now run (Graph Explorer or curl):

#### Graph Explorer
- Method: `POST`
- Path: `/{page_id}/subscribed_apps`
- Params: `subscribed_fields = feed,messages,mention`
- (leave access_token out of Params if you pasted the Page token into the blue box)

#### curl (bash):
```bash
curl -X POST "https://graph.facebook.com/v23.0/{page_id}/subscribed_apps" \
  -d "subscribed_fields=feed,messages,mention" \
  -d "access_token=PASTE_PAGE_TOKEN_HERE"
```

**Expected response:**
```json
{"success": true}
```

**Alternate UI route:** Business Settings → Accounts → Pages → select Page → Connected Assets / Apps → add your app.

### 6) Post to the Page (Text)

Either in Graph Explorer (with Page token in blue box) or via curl:

**Path:**
```
POST /{page_id}/feed
```

**Params:**
- `message = Hello from API`
- `access_token = <page_token>` // or use blue-box page token

#### curl example:
```bash
curl -X POST "https://graph.facebook.com/v23.0/{page_id}/feed" \
  -d "message=Test post from API" \
  -d "access_token=PASTE_PAGE_TOKEN_HERE"
```

Response returns a post_id.

### 7) Post Image to the Page

**Path:**
```
POST /{page_id}/photos
```

**Params:**
- `url = https://picsum.photos/600`
- `caption = Test photo`
- `access_token = <page_token>`

Response returns a photo/post id.

### 8) Verify a Post

```
GET /{post_id}?fields=id,permalink_url&access_token={page_token}
```

Open permalink_url to view the post on the Page.

### 9) Make Tokens Long-Lived (For n8n / Automation)

Exchange short-lived user token → long-lived user token:

```
GET https://graph.facebook.com/v23.0/oauth/access_token?
  grant_type=fb_exchange_token&
  client_id={APP_ID}&
  client_secret={APP_SECRET}&
  fb_exchange_token={SHORT_LIVED_USER_TOKEN}
```

Use returned long-lived user token to call:

```
GET /me/accounts?access_token={LONG_LIVED_USER_TOKEN}
```

→ this returns long-lived Page tokens (≈60 days). Save the Page token for n8n.

### 10) Debug Tokens (If Anything Looks Wrong)

```
GET /debug_token?input_token={TOKEN_TO_CHECK}&access_token={APP_ID}|{APP_SECRET}
```

Check data.is_valid, data.type (should be PAGE for page token), and data.app_id (must equal your app id).

### 11) n8n: How to Use the Page Token

Put the long-lived Page token into your n8n credential for Facebook / HTTP request.

Use endpoints `/{page_id}/feed` or `/{page_id}/photos` with POST.

Keep token secure; refresh before expiry.

### 12) Common Errors & Fixes (Copy/Paste Checklist)

- **`data: []` from /me/accounts** → your user isn't an Admin for any Page, or you didn't grant pages_show_list during auth. Fix: make yourself Admin + regenerate user token with scopes.

- **(#210) A page access token is required** → Graph Explorer was using the blue-box token (user token). Fix: paste Page token into the blue Access Token field (not only Params).

- **403 Forbidden when calling subscribed_apps** → token not bound to your app or token lacks scopes. Fix: ensure token's app_id matches your app (use debug_token), regenerate token for your app.

- **publish_actions deprecated** → use Page tokens + Pages permissions (pages_manage_posts, pages_read_engagement).

### 13) Security & Production Notes

Never share tokens or App Secret publicly. If leaked, regenerate/revoke immediately.

For public/production usage you will need App Review to get these permissions approved for non-developers. While in development mode, only app admins/testers can use the app.

For Instagram posting, link the IG Business account to the Page (Page settings → connected Instagram account) and use IG endpoints (`/{ig_business_id}/media` + media_publish) and instagram_content_publish permission.

## Final Short Checklist to Reproduce From Scratch

1. Create FB Page & confirm your personal profile is Admin.
2. Create Meta App (note App ID/Secret).
3. Graph Explorer → select app → generate user token with pages scopes.
4. GET /me/accounts → copy id and access_token (Page token).
5. Paste Page token into blue Access Token box.
6. POST /{page_id}/subscribed_apps with subscribed_fields=feed,messages,mention → expect {"success":true}.
7. Post using POST /{page_id}/feed or /photos with the Page token.
8. Exchange for long-lived tokens for automation.
