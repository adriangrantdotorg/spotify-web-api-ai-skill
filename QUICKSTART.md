# 🚀 Quick Start Guide

Get the Spotify Web API Skill up and running in less than 5 minutes!

---

## ⚡ 60-Second Installation

### Step 1: Get Spotify Credentials (2 minutes)

1. Go to [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)
2. Click **"Create app"**
3. Fill in:
   - **App name**: "My Dashboard" (or any name)
   - **Redirect URI**: `http://localhost:8080/callback`
4. Copy your **Client ID** and **Client Secret**

### Step 2: Install the Skill (1 minute)

**For Claude Code:**
```bash
git clone https://github.com/adriangrantdotorg/spotify-web-api-ai-skill.git ~/.claude/skills/spotify-web-api
```

**For Cursor:**
```bash
git clone https://github.com/adriangrantdotorg/spotify-web-api-ai-skill.git .cursor/skills/spotify-web-api
```

**For Google Antigravity:**
```bash
git clone https://github.com/adriangrantdotorg/spotify-web-api-ai-skill.git .agent/skills/spotify-web-api
```

**For OpenCode:**
```bash
git clone https://github.com/adriangrantdotorg/spotify-web-api-ai-skill.git ~/.config/opencode/skills/spotify-web-api
```

**For Claude.ai:**
1. Download [this ZIP file](https://github.com/adriangrantdotorg/spotify-web-api-ai-skill/archive/refs/heads/main.zip)
2. Go to Settings → Capabilities → Upload skill
3. Upload the ZIP and toggle ON

### Step 3: Test It (1 minute)

Open your AI assistant and try this prompt:

```
Create a Python script that shows my currently playing track 
with proper rate limit handling
```

**Expected Result:** Your AI agent will generate code that:
- Uses `spotipy` with fail-fast configuration
- Handles 429 errors properly
- Extracts `Retry-After` headers
- Provides user feedback during rate limits

---

## 🎯 Your First Integration

### Option A: Simple Python Script

**Prompt:**
```
Create a Python script that displays my Spotify currently playing track.
Use the Spotify Web API skill best practices.
```

**What you'll get:**
```python
import spotipy
from spotipy.oauth2 import SpotifyOAuth

# Configured with fail-fast, no blocking retries
sp = spotipy.Spotify(
    auth_manager=SpotifyOAuth(scope="user-read-playback-state"),
    requests_timeout=10,
    status_retries=0,
    retries=0
)

try:
    track = sp.current_user_playing_track()
    if track and track['is_playing']:
        print(f"🎵 Now Playing: {track['item']['name']}")
        print(f"👤 Artist: {track['item']['artists'][0]['name']}")
except spotipy.exceptions.SpotifyException as e:
    if e.http_status == 429:
        retry_after = int(e.headers.get('Retry-After', 5))
        print(f"⏳ Rate limited. Retry in {retry_after} seconds.")
```

**Run it:**
```bash
pip install spotipy
python script.py
```

### Option B: React Dashboard

**Prompt:**
```
Create a React dashboard that polls my Spotify playback state 
every 10 seconds with the Page Visibility API pattern
```

**What you'll get:**
- Smart polling (10s active, 60s background)
- Rate limit handling with countdown UI
- Proper OAuth flow
- Clean component structure

---

## 🔍 Verify Installation

### Check 1: Skill is Loaded

**For Claude Code:**
```bash
# In Claude Code terminal
ls ~/.claude/skills/spotify-web-api
# Should show: skill/spotify-web-api/SKILL.md, README.md, etc.
```

**For Claude.ai:**
- Go to Settings → Skills
- Look for "spotify-web-api" in the list
- Ensure toggle is ON

### Check 2: Agent Recognizes It

Try this prompt:
```
What skills do you have for working with APIs?
```

Your agent should mention the Spotify Web API skill.

### Check 3: Instructions Work

Try this prompt:
```
What's the best way to handle Spotify rate limits?
```

Your agent should describe:
- 429 error handling
- `Retry-After` header extraction
- Fail-fast configuration
- Client-side backoff

---

## 🎨 Example Prompts to Try

### Rate Limiting
```
Show me how to handle Spotify rate limits in Python with proper retry logic
```

### Polling
```
Create a dashboard that polls playback state efficiently
```

### Authentication
```
Set up Spotify OAuth with the recommended scopes for a playlist manager
```

### Data Optimization
```
Fetch all tracks from a large playlist efficiently
```

### API Selection
```
Should I use the Web API or Web Playback SDK for my music player?
```

---

## 🐛 Troubleshooting

### "Skill not found"

**Solution:**
1. Check the skill is in the correct directory
2. Ensure skill/spotify-web-api/SKILL.md exists
3. Restart your AI assistant
4. Try mentioning the skill explicitly: "Use the Spotify Web API skill to..."

### "Invalid credentials"

**Solution:**
1. Verify your `.env` file has correct credentials
2. Check the redirect URI matches your Spotify app settings
3. Ensure you're using the right Client ID/Secret

### "Rate limited immediately"

**Solution:**
- This is expected during testing!
- The skill teaches proper handling of this
- Your generated code should show a countdown

### "Agent doesn't use the skill"

**Solution:**
1. Be explicit: "Use the Spotify Web API skill"
2. Mention Spotify in your prompt
3. Ask about rate limiting or polling specifically

---

## 📚 Next Steps

Now that you have the skill installed:

1. **Read the full docs**: See [README.md](README.md) for complete reference
2. **Build something**: Start your integration project
3. **Share feedback**: Open an issue or discussion
4. **Contribute**: See [CONTRIBUTING.md](CONTRIBUTING.md)

---

## 🆘 Get Help

- **Questions**: [GitHub Discussions](https://github.com/adriangrantdotorg/spotify-web-api-ai-skill/discussions)
- **Bugs**: [GitHub Issues](https://github.com/adriangrantdotorg/spotify-web-api-ai-skill/issues)
- **Documentation**: [README.md](README.md)

---

## 🎉 Success!

You're now ready to build Spotify integrations with AI assistance that follows production-ready best practices!

**Pro tip**: Star the repository to stay updated with new patterns and improvements!

---

<div align="center">
  <sub>Built with ❤️ for Music Lovers & the Open Source Community ✌🏾</sub>
</div>
