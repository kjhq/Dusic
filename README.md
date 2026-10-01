<div align="center">

# dusic

**a discord music bot you can just talk to.**
play songs, albums and playlists from spotify links or plain names, with slash commands or by mentioning the bot in english.

[![python](https://img.shields.io/badge/python-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![discord.py](https://img.shields.io/badge/discord.py-5865F2?style=flat-square&logo=discord&logoColor=white)](https://github.com/Rapptz/discord.py)
[![spotify](https://img.shields.io/badge/spotify%20api-1DB954?style=flat-square&logo=spotify&logoColor=white)](https://developer.spotify.com/documentation/web-api)
[![youtube music](https://img.shields.io/badge/youtube%20music-FF0000?style=flat-square&logo=youtubemusic&logoColor=white)](https://github.com/sigma67/ytmusicapi)
[![openai](https://img.shields.io/badge/openai-412991?style=flat-square&logo=openai&logoColor=white)](https://platform.openai.com/)
[![ffmpeg](https://img.shields.io/badge/ffmpeg-007808?style=flat-square&logo=ffmpeg&logoColor=white)](https://ffmpeg.org/)

[![add to discord](https://img.shields.io/badge/add%20to%20discord-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.com/oauth2/authorize?client_id=1225070174453891102)

</div>

---

## features

- **spotify in, music out**: paste a spotify track, album or playlist link, or just type a song name
- **natural language control**: mention the bot (`@Dusic play some daft punk`) and an llm picks the right action through function calling
- **slash commands** with spotify-powered autocomplete on `/play`
- **per-server queues**: albums and playlists are queued track by track
- **parallel downloads**: a pool of worker processes fetches audio ahead of playback and caches it on disk
- **follows you around**: if the bot is moved to another voice channel, playback moves with it; if it gets disconnected, the queue is cleared
- **error reporting**: errors (with stack traces) are posted to a discord webhook

---

## how it works

```mermaid
flowchart LR
    U[user] -->|/play or @mention| B[discord bot]
    B -->|@mention| AI[openai chat<br/>tool calling]
    AI -->|play / pause / skip ...| B
    B --> S[spotify web api<br/>track · album · playlist · search]
    S --> Y[ytmusicapi<br/>best youtube match]
    Y --> D[download workers<br/>pytube · multiprocessing]
    D --> C[(music/ cache)]
    C --> Q[guild queue]
    Q --> F[ffmpeg + opus] --> V[voice channel]
```

1. `/play` (or the llm) resolves the request against the spotify web api: a direct link, or the top search result
2. each track is matched to a youtube music result with `ytmusicapi`
3. download workers grab the audio with `pytube` into `music/`, skipping files already cached
4. a queue loop per server streams the next ready track into the voice channel through ffmpeg

---

## commands

| command | description |
|---|---|
| `/play <name or url>` | play a song, album or playlist (spotify url or search text) |
| `/pause` | pause the current song |
| `/resume` | resume playback |
| `/skip` | skip the current song |
| `/clear` | clear the queue and stop playback |
| `/leave` | leave the voice channel |

every command also works in plain english by mentioning the bot, e.g. `@Dusic skip this one` or `@Dusic pause`.

---

## tech stack

| area | tech |
|---|---|
| bot | python 3.12, `discord.py` (slash commands, cogs, voice) |
| ai | openai `gpt-3.5-turbo` with function calling |
| metadata | spotify web api (client credentials) via `aiohttp` |
| matching | `ytmusicapi` |
| audio | `pytube` downloads, ffmpeg + libopus playback, `PyNaCl` |
| config | `python-dotenv` |

---

## getting started

### prerequisites

- python 3.12 and [uv](https://docs.astral.sh/uv/)
- `ffmpeg` on your `PATH`
- the libopus shared library (e.g. `libopus.so.0` on linux)
- a [discord application](https://discord.com/developers/applications) with a bot token
- spotify api credentials and an openai api key

### install and run

```bash
git clone https://github.com/kjhq/Dusic.git
cd Dusic
uv venv
uv pip install -r requirements.txt
mkdir -p music   # audio cache; must exist before first start
# create .env (see configuration), then:
uv run main.py
```

---

## configuration

set these in a `.env` file in the project root:

| variable | purpose |
|---|---|
| `DISCORD_TOKEN` | discord bot token |
| `SPOTIFY_CLIENT_ID` | spotify app client id |
| `SPOTIFY_CLIENT_SECRET` | spotify app client secret |
| `OPENAI_API_KEY` | openai key for natural language control |
| `OPUS_PATH` | path to the libopus shared library |

other knobs (download worker count, cache folder, error webhook) are constants in `modules/helper/config.py`.

---

## project structure

```
Dusic/
├── main.py                     # bot entrypoint: loads cogs, opus, download workers
├── requirements.txt
└── modules/
    ├── ai.py                   # @mention handler, openai tool calling
    ├── queue.py                # per-guild queue + playback loop
    ├── download.py             # multiprocessing download workers (pytube)
    ├── discord_voice.py        # voice connect / play / stop
    ├── discord_voice_afk.py    # handles disconnects and channel moves
    ├── error_handler.py        # logs errors to a discord webhook
    ├── commands/
    │   ├── music_commands.py   # slash commands
    │   └── play.py             # play flow + autocomplete
    ├── search/
    │   ├── spotify.py          # spotify web api client
    │   └── youtube.py          # youtube music matching
    └── helper/                 # config, embeds, logger, types
```

---

## contributing

bugs and feature requests: open an issue. code: fork and open a pr.

---

<div align="center">

built by [kjhq](https://kjhq.dev) · [@kjhqdev](https://x.com/kjhqdev)

</div>
