# Facebook API Guide

## 🔑 Token & Setup

### Get all Pages you manage (with tokens)
```
GET /me/accounts
```

### Debug a token (check type, app_id, validity)
```
GET /debug_token?input_token={TOKEN_TO_CHECK}&access_token={APP_ID}|{APP_SECRET}
```

## 📄 Posting Content

### Post text to Page feed
```
POST /{page_id}/feed
message=Hello world from API
```

### Post image to Page
```
POST /{page_id}/photos
url=https://picsum.photos/600
caption=Test photo
```

### Post link to Page
```
POST /{page_id}/feed
message=Check this out
link=https://example.com
```

## 🔍 Reading Content

### Get latest posts from Page
```
GET /{page_id}/posts
```

### Get a specific post's details
```
GET /{post_id}?fields=id,message,permalink_url,created_time
```

### Get comments on a post
```
GET /{post_id}/comments
```

## 🔔 App & Subscription

### Connect app to Page (done once)
```
POST /{page_id}/subscribed_apps
subscribed_fields=feed,messages,mention
```

### Check which apps are connected to the Page
```
GET /{page_id}/subscribed_apps
```

## 👤 Page Info

### Get Page details
```
GET /{page_id}?fields=id,name,category,fan_count
```

### Get Page insights (likes, reach, etc.)
```
GET /{page_id}/insights
```

## 📌 Key habits to remember

- Always paste your **Page token** into the **blue Access Token box** (top-right).
- For posting → always use **Page token**, not user token.
- When token expires → regenerate user token → run `/me/accounts` again → get fresh Page token.
- Use `/debug_token` to confirm what token you're holding (type = PAGE, app_id = your app).
