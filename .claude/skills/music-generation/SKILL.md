---
name: music-generation
description: Use this skill when the user requests to generate, create, compose, or produce music or songs — background music, theme songs, jingles, or instrumental tracks. Generates a song from a style/mood prompt and optional lyrics via the MiniMax music API.
---

# Music Generation Skill

## Overview

This skill generates songs (vocal or instrumental) from a structured JSON spec using the
MiniMax music generation API (`/v1/music_generation`). You describe the style/mood/scene in
`prompt`, optionally provide `lyrics`, and the script returns an MP3.

## Workflow

### Step 1: Understand Requirements

Identify the desired style, mood, scene, language, and whether the user wants vocals or a
pure instrumental track. Decide whether to supply lyrics or let the model write them.

### Step 2: Create the Spec JSON

Write a JSON file in a scratch/working directory named `{descriptive-name}.json`:

```json
{
  "title": "Rainy Night Cafe",
  "prompt": "indie folk, melancholic, introspective, walking alone, cafe",
  "lyrics": "[verse]\nStreetlights glow the night wind sighs\n[chorus]\nPush the wooden door warm air inside"
}
```

Fields:
- `title` (optional): a human-readable name.
- `prompt` (required): style, mood, and scene. Drives the musical character.
- `lyrics` (optional): song lyrics. Use `\n` between lines and structure tags such as
  `[Intro]`, `[Verse]`, `[Pre Chorus]`, `[Chorus]`, `[Bridge]`, `[Outro]`.
- `is_instrumental` (optional, bool): set `true` for a pure instrumental track (no lyrics needed).

Behavior:
- `lyrics` provided → those lyrics are sung.
- `is_instrumental: true` → instrumental, no vocals.
- neither → the model auto-writes lyrics from `prompt` (`lyrics_optimizer`).

### Step 3: Execute Generation

```bash
python3 .claude/skills/music-generation/scripts/generate.py \
  --prompt-file <workdir>/rainy-night-cafe.json \
  --output-file <outputs>/rainy-night-cafe.mp3
```

Parameters:
- `--prompt-file`: Absolute path to the JSON spec (required).
- `--output-file`: Absolute path for the output MP3 (required).

## Environment

- `MINIMAX_API_KEY` (required): your MiniMax interface key.
- If the script prints `MINIMAX_API_KEY is not set`, ask the user to configure the key; never ask them to paste it into chat.
- Requires the `requests` Python package.
- `MINIMAX_API_HOST` (optional): default `https://api.minimaxi.com` (mainland China platform). Accounts from the international platform (minimax.io) must set `https://api.minimax.io`.
- `MINIMAX_MUSIC_MODEL` (optional): default `music-2.6-free` (works for all API-key users);
  paid/Token-Plan users can set `music-2.6` for higher limits.

## Output Handling

- Music is saved as MP3 (in the output path you pass).
- Returned audio must decode to nonempty bytes and have an ID3 tag header or MPEG frame signature. This is a header sanity check, not complete audio decoding.
- Malformed, empty or unrecognized audio is rejected before creating the output directory or opening the output file; an existing output is preserved.
- Missing audio reports "MiniMax returned no audio data"; whitespace-only hex reports "MiniMax audio decoded to empty bytes".
- Share the generated file with the user (e.g. with SendUserFile when available).
- Offer to iterate on style or lyrics if adjustments are needed.

## Notes

- Keep `prompt` focused on style/mood/scene; put the actual sung words in `lyrics`.
- For non-English songs, write `lyrics` in the target language.
