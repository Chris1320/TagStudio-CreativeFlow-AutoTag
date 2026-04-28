# TagStudio Creative Flow AutoTag

<img width="1920" height="1020" alt="TagStudio library tags" src="https://github.com/user-attachments/assets/5e9e0a49-e4ce-44b3-97d3-6b95551b8d9d" />
<img width="1920" height="1020" alt="AutoTag in action" src="https://github.com/user-attachments/assets/95ea5f68-fc5d-4bfb-8922-29552b709b13" />


This is a Python script that uses [Google Gemini](https://aistudio.google.com/) to automatically tag assets in a
[TagStudio](https://github.com/TagStudioDev/TagStudio) library. It analyzes media content and metadata and then
intelligently assign tags strictly from your existing TagStudio tags.

I specifically use this in my "**Creative Flow**" asset library, where I have accumulated more than 130,000
media assets that are in need of better organization. This includes image/video overlays, audio sound packs,
Photoshop templates, Premiere Pro presets, DaVinci Resolve transition packs, and more.

By default, this script looks for assets tagged with "Needs Metadata Update". It then gathers all available
information about the asset, including EXIF data, media properties (resolution, bitrate, etc.), and even the
filenames of surrounding sibling files in the same directory. This contextual information is sent to Google
Gemini, which suggests relevant tags based on the content and metadata of the asset.

If the AI's confidence score for the suggested tags meets or exceeds the specified threshold, those tags are
automatically applied to the asset in the TagStudio library. Mandatory tags are also added to indicate that
the asset has been processed by the AI, and previous tags can be removed if desired.

## Features

- Uses **Google Gemini** (default: `gemini-3.1-flash-lite-preview`) to understand images, videos, audio, and text files.
- Automatically downscales large images, videos, and audio locally (via `ffmpeg` and `Pillow`) to save upload bandwidth and API tokens.
- Extracts EXIF data, media properties (resolution, bitrate, etc.), and provides the AI with surrounding sibling filenames to help infer context.
- Applies new tags directly to the local TagStudio SQLite database based on AI suggestions, with configurable confidence thresholds.
- Configurable via CLI arguments, including which tags to process, which tags to add/remove, file type filters, and more.
- Supports retries and rate limit handling with configurable delay between requests.

## Requirements

- Python >= 3.14
- [uv](https://github.com/astral-sh/uv)
- FFmpeg & FFprobe
- A Google Gemini API Key
- A TagStudio library with assets to process

## Usage

You can run the script directly using `uv`:

```bash
uv run tscf_autotag.py /path/to/tagstudio/library
```

For more information, run the script with the `--help` flag to see all available options:

```bash
uv run tscf_autotag.py --help
```

If you are not using `uv`, you will have to install the required dependencies manually via `pip`:

```bash
pip install -r requirements.txt
python tscf_autotag.py /path/to/tagstudio/library
```

### Command-Line Options

| Argument                 | Description                                                             | Default                          |
| ------------------------ | ----------------------------------------------------------------------- | -------------------------------- |
| `library_path`           | **Required.** Path to the TagStudio library (directory).                |                                  |
| `--tags-to-process`      | Process only entries with any of these tag IDs.                         | `1090` ("Needs Metadata Update") |
| `--tags-to-add`          | Add these tag IDs to processed entries.                                 | `1130` ("Tagged by AI")          |
| `--tags-to-remove`       | Remove these tag IDs from processed entries.                            | `1090`                           |
| `--limit`                | Limit the number of entries to process (0 for no limit).                | `0`                              |
| `--model`                | The Google Gemini model to use for classification.                      | `gemini-3.1-flash-lite-preview`  |
| `--api-key`              | Google API key. Defaults to `GEMINI_API_KEY` env var.                   |                                  |
| `--min-confidence-score` | Minimum confidence score required to apply suggested tags (0.0 to 1.0). | `0.80`                           |
| `--filetype-include`     | Process only entries with these file suffixes (e.g., `mp4 jpg`).        | All files                        |
| `--filetype-exclude`     | Exclude entries with these file suffixes.                               | None                             |
| `--delay`                | Delay in seconds between requests to avoid rate limits.                 | `4.0`                            |

_Run `uv run tscf_autotag.py --help` for the full and updated list of available arguments._

## How It Works

1. **Discovery:** Identifies files in the TagStudio SQLite database (`.TagStudio/ts_library.sqlite`) containing the target tags.
2. **Context Gathering:** Determines MIME types and gathers metadata, EXIF, media properties (via `ffprobe`), and local surrounding sibling file names.
3. **Downscaling:** Temporarily downscales heavy media assets using `ffmpeg` or `Pillow` to reduce transfer times and API usage.
4. **AI Processing:** Uploads the media and sends a prompt containing your complete flat TagStudio taxonomy mapping to Google Gemini.
5. **Database Update:** Interprets the JSON response from Gemini, validates the confidence threshold, and applies the newly suggested tag IDs directly to the local TagStudio SQLite database.

## Compatibility

This script has been tested on **TagStudio Alpha v9.5.6**.