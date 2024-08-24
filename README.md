# Voice-Activated Personal Assistant with Google Calendar Integration

This repository contains a Python script for a voice-activated personal assistant that interacts with Google Calendar to retrieve events, make notes, and respond to user commands. The assistant listens for a specific wake word ("hey tim") and responds to various calendar and note-taking commands using Google's Text-to-Speech and Speech Recognition services.

## Table of Contents

1. [Features](#features)
2. [Installation](#installation)
3. [Setup](#setup)
4. [Usage](#usage)
5. [How It Works](#how-it-works)
6. [Contributing](#contributing)
7. [License](#license)

## Features

- **Google Calendar Integration**: Checks for events and appointments on specific dates.
- **Voice Commands**: Recognizes and processes voice commands to interact with Google Calendar or take notes.
- **Text-to-Speech**: Uses text-to-speech to read out calendar events and confirmations.
- **Note Taking**: Takes notes based on voice input and saves them as text files.

## Installation

To run this script, you need to have Python installed along with several Python libraries. You can install the required libraries using `pip`:

```bash
pip install google-api-python-client google-auth-httplib2 google-auth-oauthlib pyttsx3 SpeechRecognition pyaudio pytz
```

**Note**: For `pyaudio` installation on Windows, you might need additional steps such as installing a `.whl` file specific to your Python version.

## Setup

1. **Google Calendar API Credentials**:

   - Go to the [Google Developers Console](https://console.developers.google.com/).
   - Create a new project.
   - Enable the Google Calendar API for this project.
   - Create credentials for an "OAuth 2.0 Client ID" and download the `credentials.json` file.
   - Place `credentials.json` in the same directory as the script.

2. **Run the Script**:
   - The first time you run the script, it will open a browser window for you to authorize access to your Google Calendar.
   - After authorization, a `token.pickle` file will be created to store your credentials.

## Usage

1. **Start the Assistant**:

   - Run the script in your terminal or command prompt.
   - The assistant will start listening for the wake word: "hey tim".

2. **Voice Commands**:

   - After saying the wake word, you can ask about your schedule:
     - "What do I have on [date]?"
     - "Do I have plans on [date]?"
     - "Am I busy on [date]?"
   - To make a note, say:
     - "Make a note"
     - "Write this down"
     - "Remember this"

3. **Example Interaction**:
   - User: "Hey tim"
   - Assistant: "I am ready."
   - User: "What do I have on next Tuesday?"
   - Assistant: "You have 2 events on next Tuesday."

## How It Works

### Authentication

The script authenticates the user using the Google Calendar API with OAuth 2.0. It checks for a `token.pickle` file, and if it doesn't exist, it uses `credentials.json` to authenticate.

```python
def authenticate_google():
    creds = None
    if os.path.exists("token.pickle"):
        with open("token.pickle", "rb") as token:
            creds = pickle.load(token)

    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file("credentials.json", SCOPES)
            creds = flow.run_local_server(port=0)

        with open("token.pickle", "wb") as token:
            pickle.dump(creds, token)

    service = build("calendar", "v3", credentials=creds)
    return service
```

### Voice Commands and Calendar Events

The assistant continuously listens for the wake word ("hey tim"). Upon detecting the wake word, it listens for further commands and processes them.

- **Get Events**: The assistant fetches events for a specified day from Google Calendar.
- **Make Notes**: The assistant takes a voice note and saves it as a text file.

```python
def get_events(day, service):
    # Call the Calendar API to fetch events
    date = datetime.datetime.combine(day, datetime.datetime.min.time())
    end_date = datetime.datetime.combine(day, datetime.datetime.max.time())
    utc = pytz.UTC
    date = date.astimezone(utc)
    end_date = end_date.astimezone(utc)

    events_result = (
        service.events()
        .list(
            calendarId="primary",
            timeMin=date.isoformat(),
            timeMax=end_date.isoformat(),
            singleEvents=True,
            orderBy="startTime",
        )
        .execute()
    )
    events = events_result.get("items", [])

    if not events:
        speak("No upcoming events found.")
    else:
        speak(f"You have {len(events)} events on this day.")

        for event in events:
            start = event["start"].get("dateTime", event["start"].get("date"))
            start_time = str(start.split("T")[1].split("-")[0])
            if int(start_time.split(":")[0]) < 12:
                start_time = start_time + "am"
            else:
                start_time = (
                    str(int(start_time.split(":")[0]) - 12) + start_time.split(":")[1]
                )
                start_time = start_time + "pm"
            speak(event["summary"] + " at " + start_time)
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
