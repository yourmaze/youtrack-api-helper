# YouTrack API Helper

Small shell helper for calling the YouTrack REST API with a permanent token.

The package intentionally does not contain real tokens, private environment files, or company-specific issue keys.

## Files

- `yt-api` - wrapper around `curl` for YouTrack REST API calls.
- `.env.example` - example credentials file.
- `SKILL.md` - optional Codex skill instructions for using this helper safely.

## Install

Clone the repository:

```bash
git clone https://github.com/yourmaze/youtrack-api-helper.git
cd youtrack-api-helper
chmod +x yt-api
```

Optionally add the folder to `PATH`, or call the script by path:

```bash
./yt-api GET '/api/users/me?fields=id,login,fullName'
```

## Create a YouTrack Token

1. Open YouTrack.
2. Open your user profile.
3. Go to account security, authentication, or permanent tokens.
4. Create a new permanent token.
5. Select the `YouTrack` scope.
6. Copy the token once and store it only in your local secret file.

Permanent tokens authorize REST API calls with the permissions of the user who created the token.

## Configure Credentials

Create a local credentials file:

```bash
mkdir -p ~/.config/youtrack-api-helper
cp .env.example ~/.config/youtrack-api-helper/youtrack.env
chmod 600 ~/.config/youtrack-api-helper/youtrack.env
```

Edit it:

```bash
nano ~/.config/youtrack-api-helper/youtrack.env
```

Example:

```bash
YOUTRACK_URL=https://youtrack.example.com
YOUTRACK_TOKEN=perm:your-token
```

You can also use a different credentials file:

```bash
YOUTRACK_ENV_FILE=/path/to/youtrack.env ./yt-api GET '/api/users/me?fields=id,login,fullName'
```

## Check Connection

```bash
./yt-api GET '/api/users/me?fields=id,login,fullName'
```

## Read an Issue

```bash
./yt-api GET '/api/issues/PROJECT-123?fields=idReadable,summary,description'
```

For richer context:

```bash
./yt-api GET '/api/issues/PROJECT-123?fields=id,idReadable,summary,description,project(shortName,name),customFields(name,value(name,localizedName,presentation,text,id)),comments(id,text,author(login,fullName),created,updated)'
```

## Add a Comment

```bash
./yt-api POST '/api/issues/PROJECT-123/comments?fields=id,text' '{"text":"Comment text"}'
```

## Create an Issue

Issue creation uses `POST /api/issues`. The exact JSON depends on the target project and its custom fields, so first inspect project and field metadata if you are not sure about ids and allowed values.

Minimal example:

```bash
./yt-api POST '/api/issues?fields=idReadable,summary' '{
  "project": {"shortName": "PROJECT"},
  "summary": "Short issue title",
  "description": "Issue description"
}'
```

## Safety Rules

- Do not commit real `.env` files.
- Do not paste tokens into chat, tickets, logs, or command examples.
- Use `fields=` in API requests to keep responses compact.
- Read operations are usually safe to run directly.
- For write operations, prepare a draft first and ask for explicit confirmation before sending the request.
- If YouTrack returns `401` or `403`, refresh permissions or create a new token. Do not retry with passwords.
