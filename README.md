# YouTube-Content-Automation
An n8n workflow that turns a single Telegram message into a finished, captioned vertical video and publishes it to YouTube Shorts. Send a topic or idea to your bot, and the workflow writes the script, fetches stock footage, renders a 1080x1920 video with text overlay, sends a preview back to Telegram, and uploads it to YouTube with an AI-generated title and description.

Workflow overview

Features
Telegram as the control panel: trigger a video by sending a message to your bot.
AI-written copy: Google Gemini generates the on-video text and, in a second pass, the YouTube title, description and tags.
Stock footage: pulls a relevant video from the Pixabay API.
Auto-formatting for Shorts: Cloudinary crops the clip to 1080x1920 (9:16) and burns the text onto it.
Preview before/while publishing: the finished video is sent back to you on Telegram.
One-step publishing: uploads directly to YouTube via the YouTube Data API v3.
How It Works
Step	Node	What it does
1	Telegram Trigger	Starts the workflow on every new message to the bot.
2	AI Agent (Google Gemini Chat Model)	Turns the message into the video text and a stock-footage search keyword.
3	Code in JavaScript	Parses the agent output into clean fields for the next nodes.
4	HTTP Request	GET to the Pixabay Videos API to fetch a matching clip.
5	HTTP Request1	POST to Cloudinary to upload the clip and apply the 1080x1920 crop and text overlay.
6	Wait	Gives Cloudinary time to finish processing the video.
7	HTTP Request2	Downloads the processed video as binary data.
8	Send a video	Sends the result back to the Telegram chat.
9	AI Agent1 (Google Gemini Chat Model)	Generates the YouTube title, description and tags.
10	Code in JavaScript1	Parses the metadata into JSON fields.
11	Merge (combine)	Combines the video binary with the generated metadata.
12	Upload a video	Publishes the video to YouTube.
Telegram Trigger -> AI Agent -> Code -> Pixabay -> Cloudinary -> Wait -> Download video
                                                                            |
                                          +---------------------------------+------------------+
                                          |                                                    |
                                   Send to Telegram                          AI Agent1 -> Code1 -+
                                                                                                 |
                                                                  Download video ----------------+-> Merge -> YouTube Upload
Prerequisites
An n8n instance (cloud or self-hosted)
A Telegram bot token from @BotFather
A Google AI Studio API key for Gemini
A Pixabay API key
A Cloudinary account (cloud name and an unsigned upload preset, or API key and secret)
A Google Cloud project with YouTube Data API v3 enabled and OAuth 2.0 credentials
Setup
Import the workflow In n8n, go to Workflows -> Import from file and select workflow.json.
Create credentials in n8n
Telegram API (bot token)
Google Gemini (PaLM) API key
YouTube OAuth2 API
Pixabay and Cloudinary keys (set inside the HTTP Request nodes, or as n8n credentials)
Configure the HTTP nodes
HTTP Request: add your Pixabay key and search query.
HTTP Request1: set your Cloudinary cloud name and upload preset.
Configure the YouTube node
Title and Description: mapped from the metadata JSON, e.g. {{ $json.video_title }}.
Category: pick a category (for example People & Blogs or Entertainment).
Input Binary Field: data.
Make sure #Shorts is in the title or description.
Activate the workflow and send a message to your Telegram bot.
Usage
Open your bot in Telegram.
Send a topic or idea, e.g. motivation for developers.
Wait a minute or two. The bot replies with the generated video.
The same video is uploaded to your YouTube channel.
Troubleshooting

Quota exceeded for quota metric 'Video Uploads' YouTube limits uploads per day for each Google Cloud project, and new or unverified projects have a very low limit. Failed attempts count too.

Wait for the daily reset (midnight Pacific Time).
Disable the YouTube node while testing earlier steps.
Request a quota increase under APIs & Services -> YouTube Data API v3 -> Quotas.

Category field shows "Error fetching options from YouTube" Usually caused by the same quota block, or an expired credential. Reconnect the YouTube credential and retry once the quota resets.

YouTube node reports missing binary data Check that the Merge node output still contains the Binary tab and that Input Binary Field matches the binary property name (default data).

Cloudinary video not ready Increase the Wait node duration so processing finishes before the download step.

Upload works but the video isn't a Short Shorts must be vertical, 3 minutes or less, and include #Shorts. The Cloudinary step already outputs 1080x1920.

Tech Stack

n8n, Telegram Bot API, Google Gemini, Pixabay API, Cloudinary, YouTube Data API v3, JavaScript

Roadmap
[ ] Add voice-over using text-to-speech
[ ] Add background music
[ ] Add approval step in Telegram before publishing
[ ] Error handling and retry for failed uploads
[ ] Schedule uploads to stay within quota
Contributing

Pull requests are welcome. For major changes, open an issue first to discuss what you'd like to change.
