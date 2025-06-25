---
title: "GitHubFetch: A CLI Tool for Stylized GitHub Profiles"
date: 2025-06-15
draft: false
description:
  "A Python CLI tool inspired by neofetch that displays stylized GitHub profiles
  directly in your terminal with contribution heatmaps and repository stats."
tags:
  ["python", "cli", "github-api", "graphql", "open-source", "terminal", "pypi"]
cover:
  src: ./demo.gif
  alt:
    "GitHubFetch CLI tool in action showing GitHub profile with contribution
    heatmap"
toc: true
---

{{< figure src="demo.gif" alt="GitHubFetch CLI tool in action" class="media-container" >}}

Ever wanted to check out someone's GitHub profile without leaving your terminal?
I built **GitHubFetch** - a Python CLI tool inspired by system fetch tools like
`neofetch` that displays beautiful, stylized GitHub profiles directly in your
terminal.

## What is GitHubFetch?

GitHubFetch is a command-line tool that fetches and displays GitHub user
information in an aesthetically pleasing format. It shows user stats, pinned
repositories, and even renders a horizontal contribution heatmap - all from the
comfort of your terminal.

## Key Features

- 🎨 **Beautiful ASCII art and styling** inspired by neofetch
- 📊 **Contribution heatmap** displayed horizontally in the terminal
- 📌 **Pinned repositories** with star counts and descriptions
- 🔗 **GitHub GraphQL and REST API integration**
- 🚀 **Fast and lightweight** - written in pure Python
- 📦 **Easy installation** from PyPI

## Installation

You can install GitHubFetch directly from PyPI:

```bash
pip install githubfetch
```

Or clone and install from source:

```bash
git clone https://github.com/shubhpsd/githubfetch
cd githubfetch
pip install -e .
```

## Usage

Using GitHubFetch is simple:

```bash
# Display your own profile (requires GitHub token)
githubfetch

# Display any public GitHub user's profile
githubfetch username

# Display with additional options
githubfetch username --show-repos --ascii-art
```

## Technical Implementation

### GitHub API Integration

GitHubFetch leverages both GitHub's REST and GraphQL APIs to gather
comprehensive user data:

```python
import requests
from gql import gql, Client
from gql.transport.requests import RequestsHTTPTransport

class GitHubFetcher:
    def __init__(self, token=None):
        self.token = token
        self.rest_headers = {'Authorization': f'token {token}'} if token else {}

    def get_user_data(self, username):
        # REST API for basic user info
        response = requests.get(
            f'https://api.github.com/users/{username}',
            headers=self.rest_headers
        )
        return response.json()

    def get_contribution_data(self, username):
        # GraphQL for contribution calendar
        query = gql("""
            query($username: String!) {
                user(login: $username) {
                    contributionsCollection {
                        contributionCalendar {
                            weeks {
                                contributionDays {
                                    contributionCount
                                    date
                                }
                            }
                        }
                    }
                }
            }
        """)
        # Execute query and return data
```

### Contribution Heatmap Rendering

One of the coolest features is the horizontal contribution heatmap that mimics
GitHub's contribution graph:

```python
def render_contribution_heatmap(contribution_data):
    # Convert contribution data to visual blocks
    heatmap_chars = ['⬜', '🟩', '🟢', '🟡', '🔥']

    for week in contribution_data['weeks']:
        for day in week['contributionDays']:
            count = day['contributionCount']
            intensity = min(count // 5, 4)  # Scale to 0-4
            print(heatmap_chars[intensity], end='')
        print()  # New line for each week
```

### ASCII Art and Styling

The tool includes beautiful ASCII art and colored output using libraries like
`colorama` and `rich`:

```python
from rich.console import Console
from rich.panel import Panel
from rich.columns import Columns

console = Console()

def display_profile(user_data):
    # Create styled panels for different sections
    profile_panel = Panel(
        f"👤 {user_data['name']}\n"
        f"📧 {user_data['email']}\n"
        f"📍 {user_data['location']}",
        title="Profile",
        border_style="blue"
    )

    stats_panel = Panel(
        f"📁 Repositories: {user_data['public_repos']}\n"
        f"👥 Followers: {user_data['followers']}\n"
        f"👤 Following: {user_data['following']}",
        title="Statistics",
        border_style="green"
    )

    console.print(Columns([profile_panel, stats_panel]))
```

## Features Deep Dive

### 1. Profile Information Display

{{< figure src="main-demo.png" caption="Profile information display showing user stats, bio, and location" alt="GitHubFetch Profile Display" class="media-container" >}}

The tool displays comprehensive profile information including:

- User's real name and username
- Bio and location
- Public repository count
- Follower and following counts
- Account creation date

### 2. Contribution Heatmap

{{< figure src="heatmap-demo.png" caption="Horizontal contribution heatmap showing activity patterns over the year" alt="GitHubFetch Heatmap Display" class="media-container" >}}

The contribution heatmap shows:

- Daily contribution counts for the past year
- Color-coded intensity levels
- Horizontal layout optimized for terminal viewing
- Total contribution count

### 3. Pinned Repositories

The tool fetches and displays pinned repositories with:

- Repository names and descriptions
- Star counts and fork counts
- Primary programming languages
- Links to repositories

## Publishing to PyPI

One of the highlights of this project was publishing it to the Python Package
Index (PyPI). This involved:

1. **Package Structure Setup**:

```
githubfetch/
├── githubfetch/
│   ├── __init__.py
│   ├── main.py
│   └── utils.py
├── setup.py
├── README.md
└── requirements.txt
```

1. **Setup Configuration**:

```python
from setuptools import setup, find_packages

setup(
    name="githubfetch",
    version="1.0.0",
    author="Shubham Prasad",
    description="A CLI tool for viewing stylized GitHub profiles",
    long_description=open("README.md").read(),
    long_description_content_type="text/markdown",
    url="https://github.com/shubhpsd/githubfetch",
    packages=find_packages(),
    install_requires=[
        "requests",
        "gql",
        "colorama",
        "rich",
    ],
    entry_points={
        "console_scripts": [
            "githubfetch=githubfetch.main:main",
        ],
    },
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires=">=3.6",
)
```

3. **Publishing Process**:

```bash
# Build the package
python setup.py sdist bdist_wheel

# Upload to PyPI
twine upload dist/*
```

## Challenges and Solutions

### 1. API Rate Limiting

**Challenge**: GitHub API has rate limits for unauthenticated requests.
**Solution**: Implemented optional authentication token support and graceful
degradation.

### 2. Terminal Compatibility

**Challenge**: Different terminals handle colors and Unicode differently.
**Solution**: Used libraries like `colorama` for cross-platform color support
and fallback ASCII characters.

### 3. GraphQL Complexity

**Challenge**: GraphQL queries for contribution data were complex. **Solution**:
Used the `gql` library for structured query building and validation.

## What I Learned

Building GitHubFetch taught me valuable lessons about:

- **API Design**: Working with both REST and GraphQL APIs
- **CLI Development**: Creating user-friendly command-line interfaces
- **Package Distribution**: Publishing Python packages to PyPI
- **Cross-platform Compatibility**: Ensuring the tool works across different
  operating systems
- **Documentation**: Writing clear documentation for open-source projects

## Future Enhancements

I'm planning several improvements for future versions:

- 🔄 **Repository language statistics** pie chart in terminal
- 📈 **Commit frequency analysis** and trends
- 🏆 **Achievement badges** and GitHub trophies display
- ⚙️ **Configuration file** support for custom styling
- 🌟 **Organization profiles** support
- 📊 **Export options** (JSON, CSV) for profile data

## Installation and Usage

Ready to try GitHubFetch? Here's how to get started:

```bash
# Install from PyPI
pip install githubfetch

# Basic usage
githubfetch octocat

# With authentication (for private repos and higher rate limits)
export GITHUB_TOKEN="your_github_token"
githubfetch your-username
```

## Contributing

GitHubFetch is open source and welcomes contributions! Whether it's bug fixes,
new features, or documentation improvements, I'd love to have your help.

Check out the [GitHub repository](https://github.com/shubhpsd/githubfetch) to:

- Report bugs or request features
- Submit pull requests
- View the complete source code
- Read detailed documentation

---

**Links:**

- 📦 [PyPI Package](https://pypi.org/project/githubfetch/)
- 💻 [GitHub Repository](https://github.com/shubhpsd/githubfetch)
- 📚 [Documentation](https://github.com/shubhpsd/githubfetch#readme)

GitHubFetch represents my passion for creating developer tools that make
everyday tasks more enjoyable. Give it a try and let me know what you think!
