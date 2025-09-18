---
title: "GitHubFetch: neofetch but for GitHub Nerds"
date: 2025-06-15
draft: false
description:
  "Built a CLI tool to check GitHub profiles without leaving the terminal. As a
  Linux ricing enthusiast, this felt like a natural sidekick to neofetch."
tags: ["python", "cli", "github-api", "linux", "ricing", "terminal", "neofetch"]
cover:
  src: ./demo.gif
  alt: "GitHubFetch showing a GitHub profile in terminal"
toc: true
---

## What I Built

{{< video src="demo.mp4" >}}

As someone who spends way too much time ricing Linux setups and running
`neofetch` every five minutes, I thought - why not have something similar for
GitHub profiles? So I built **GitHubFetch** - for showing off developer stuff.

## Why This Exists

Look, I'm one of those people who rice their Linux setup way too much. You know
the type - constantly tweaking configs, trying new color schemes, and running
`fastfetch` or `neofetch` just to see those pretty system stats.

So naturally, I thought: "What if I could `neofetch` someone's GitHub profile?"
And that's how GitHubFetch was born.

## How It Works

Simple concept - throw it a username and get their GitHub profile displayed in
terminal:

```bash
# Check anyone's profile
githubfetch shubhpsd

# With heatmap for contribution nerds like me
githubfetch shubhpsd --heatmap

# First time? Set up your token
githubfetch --config
```

The best part? It renders the GitHub contribution heatmap horizontally in your
terminal. Took forever to get those Unicode blocks looking right, but totally
worth it.

## The Fun Challenges (And What I Learned)

**Getting those Unicode blocks right** - The contribution heatmap was trickier
than expected. Different terminals handle Unicode differently, so I had to
create fallback characters:

```python
# Different intensity levels for contributions
heatmap_chars = ['·', '▪', '▪', '■', '■']  # None, Few, Some, Many, Lots
# Fallback for terminals that don't support Unicode
fallback_chars = [' ', '.', 'o', 'O', '#']
```

**GitHub's dual API system** - First time seriously working with both REST and
GraphQL. Basic user info comes from REST, but contribution calendar data? That's
GraphQL only. Here's the query I spent way too long figuring out:

```python
graphql_query = {
    "query": """
        query($login: String!, $from: DateTime!, $to: DateTime!) {
            user(login: $login) {
                contributionsCollection(from: $from, to: $to) {
                    contributionCalendar {
                        totalContributions
                        weeks {
                            contributionDays {
                                date
                                contributionCount
                                color
                            }
                        }
                    }
                }
            }
        }
    """,
    "variables": {"login": username, "from": from_date, "to": to_date}
}
```

**Terminal colors and cross-platform compatibility** - Made a whole color
utility class because ANSI codes are a pain:

```python
class Color:
    def __init__(self):
        self.red = "\x1b[38;5;1m"
        self.green = "\x1b[38;5;149m"
        self.yellow = "\x1b[38;5;11m"
        self.reset = "\x1b[00m"

    def color(self, color_name, text):
        return f"{getattr(self, color_name, self.reset)}{text}{self.reset}"
```

**Rate limiting and token management** - GitHub gets cranky without
authentication. Built a whole token management system that stores tokens
securely and validates them:

```python
def validate_github_token(token):
    headers = {"Authorization": f"Bearer {token}"}
    response = requests.get("https://api.github.com/user", headers=headers)

    if response.status_code == 200:
        # Check for needed scopes
        scopes = response.headers.get('X-OAuth-Scopes', '')
        if 'read:user' not in scopes:
            print("Warning: Token may not have required 'read:user' scope")
        return True
    return False
```

## What I Used (And Why)

**Python** - The only language I know right now for this purpose.

**Rich library** - This was a game-changer for terminal UI. Makes creating
panels and colored output actually pleasant:

```python
from rich.console import Console
from rich.panel import Panel

console = Console()
profile_panel = Panel(
    f"👤 {user_data['name']}\n📧 {user_data['email']}",
    title="Profile",
    border_style="blue"
)
```

**PIL (Pillow)** - For processing GitHub avatar images and displaying them in
terminal with proper aspect ratios.

**Requests + GraphQL** - REST API for basic stuff, GraphQL for the complex
contribution data. GitHub's GraphQL API is actually really well designed once
you get the hang of it.

**Token storage with proper security** - Learned about file permissions the hard
way:

```python
def save_token(token):
    with open(token_path, "w") as f:
        json.dump({"github_token": token}, f)
    # Set file permissions to be readable only by user!
    os.chmod(token_path, 0o600)
```

## The Avatar Display Magic

Getting GitHub avatars to show in terminal was surprisingly complex. Had to
handle different `imgcat` versions and terminal compatibility:

```python
def display_side_by_side(user_data, starred_count, pinned_repos):
    # Download and process the avatar
    avatar_url = user_data.get('avatar_url')
    response = requests.get(avatar_url)
    img = Image.open(BytesIO(response.content))

    # Calculate aspect ratio for proper display
    width_px, height_px = img.size
    aspect_ratio = width_px / height_px
    image_width_cells = int(IMAGE_HEIGHT_CELLS * aspect_ratio * 2.0)

    # Use imgcat to display, then position text next to it
    subprocess.run([imgcat_cmd, "--height", str(IMAGE_HEIGHT_CELLS), temp_image.name])

    # Move cursor up and right to position text
    sys.stdout.write(f"\x1b[{IMAGE_HEIGHT_CELLS}A")
    sys.stdout.write(f"\x1b[{text_start_column}C")
```

## What I Learned

**CLI UX is harder than web UX** - Every terminal has quirks. Spent way more
time on error handling and fallbacks than expected.

**GraphQL is actually pretty neat** - More flexible than REST for complex nested
data. The type system and introspection features are really nice for
development.

**Python packaging ecosystem** - First time publishing to PyPI. There's a whole
world of distribution I never knew about.

**Security matters for CLI tools** - Token management, file permissions, secure
input handling - stuff you don't think about until you need it.

## Try It Out

```bash
pip install githubfetch
githubfetch shubhpsd
```

It's on [GitHub](https://github.com/shubhpsd/githubfetch) if you want to check
out the code or contribute.

---

Just a simple tool that scratches my own itch. Sometimes the best projects are
the ones that solve your daily annoyances while combining your weird hobbies
(Linux ricing + GitHub stalking).
