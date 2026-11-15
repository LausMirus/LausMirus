
import json
import os
import urllib.request
from urllib.parse import urlencode
from datetime import datetime
from zoneinfo import ZoneInfo

# WakaTime configuration
TOKEN = os.environ["WAKATIME_API_KEY"]
TIMEZONE = "Asia/Kolkata"

# Your WakaTime account creation date (IST)
START_DATE = "2026-11-15"
END_DATE = datetime.now(ZoneInfo(TIMEZONE)).date().isoformat()

# README markers
START_MARKER = "<!--START_SECTION:waka-->"
END_MARKER = "<!--END_SECTION:waka-->"


# Fetch data from WakaTime API
def get_api_data(endpoint):
    url = f"https://wakatime.com/api/v1/{endpoint}"

    request = urllib.request.Request(
        url,
        headers={
            "Authorization": f"Bearer {TOKEN}",
            "Accept": "application/json",
        },
    )

    with urllib.request.urlopen(request, timeout=30) as response:
        return json.loads(response.read().decode("utf-8"))


# Format seconds into readable coding time
def format_time(seconds):
    seconds = int(seconds)

    hours, remainder = divmod(seconds, 3600)
    minutes = remainder // 60

    if hours > 0:
        return f"{hours} hrs {minutes} mins"

    if minutes > 0:
        return f"{minutes} mins"

    return f"{seconds} secs"


# Fetch all daily summaries
params = urlencode({
    "start": START_DATE,
    "end": END_DATE,
    "timezone": TIMEZONE,
})

print("Fetching WakaTime activity...")

response_data = get_api_data(
    f"users/current/summaries?{params}"
)

daily_summaries = response_data["data"]


# Calculate total coding time and language statistics
total_seconds = 0
languages = {}

for day in daily_summaries:

    total_seconds += day["grand_total"]["total_seconds"]

    for language in day.get("languages", []):

        name = language["name"]
        seconds = language["total_seconds"]

        languages[name] = languages.get(name, 0) + seconds


# Generate README content


# Generate WakaTime-style terminal output

BAR_LENGTH = 20

sorted_languages = sorted(
    languages.items(),
    key=lambda item: item[1],
    reverse=True
)

lines = [
    "```txt",
    f"From: {START_DATE} - To: {END_DATE}",
    "",
    f"Total Time: {format_time(total_seconds)}",
    "",
]

if total_seconds > 0:
    for name, seconds in sorted_languages:
        if seconds < 60:
            continue
        percentage = (seconds / total_seconds) * 100

        filled = round((percentage / 100) * BAR_LENGTH)
        bar = "█" * filled + "░" * (BAR_LENGTH - filled)

        lines.append(
            f"{name:<12} {bar} {percentage:5.1f}%  {format_time(seconds)}"
        )
else:
    lines.append("No activity tracked")

lines.append("```")

content = "\n".join(lines)


# Sort languages by coding time
sorted_languages = sorted(
    languages.items(),
    key=lambda item: item[1],
    reverse=True
)
#/print("\nLanguage statistics:")
#for name, seconds in sorted_languages:
#    lines.append(
#        f"| {name} | {format_time(seconds)} |"
#    )

#content = "\n".join(lines)


# Read existing README
with open("README.md", "r", encoding="utf-8") as file:
    readme = file.read()


# Verify README markers
if START_MARKER not in readme or END_MARKER not in readme:
    raise RuntimeError(
        "WakaTime README markers are missing."
    )

if readme.index(START_MARKER) >= readme.index(END_MARKER):
    raise RuntimeError(
        "WakaTime README markers are in the wrong order."
    )


# Replace only the WakaTime section
before, remainder = readme.split(START_MARKER, 1)
_, after = remainder.split(END_MARKER, 1)

updated_readme = (
    before
    + START_MARKER
    + "\n"
    + content
    + "\n"
    + END_MARKER
    + after
)


# Save updated README
with open("README.md", "w", encoding="utf-8") as file:
    file.write(updated_readme)


# Display results
print("\nWakaTime README updated successfully!")
print("-----------------------------------")
print("Period:", START_DATE, "to", END_DATE)
print("Total coding time:", format_time(total_seconds))

