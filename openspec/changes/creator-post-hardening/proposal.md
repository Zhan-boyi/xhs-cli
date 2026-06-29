# Change: Harden Creator-Center Image Posting

## Why

`xhs post` is the stable command surface users and agents expect:

```bash
xhs post "Title" --image cover.png --content "Body" --json
```

The current Xiaohongshu creator-center page no longer matches the original
implementation in a few important places:

- the default creator page can mount the video uploader first;
- the image uploader must be selected explicitly;
- upload inputs can include video and image inputs, so the CLI must choose the
  image input deterministically;
- the body editor is often a contenteditable rich-text editor;
- the publish footer is rendered as `xhs-publish-btn`, not always as a normal
  text button;
- AI-generated content should be declared when the creator page exposes that
  control.

Without this change, the CLI command exists but publishing can stop at the
wrong uploader or fail to click the final publish control.

## What Changes

- Keep `xhs post` as the public publishing command.
- Add creator-center-specific helpers for:
  - switching to image-note mode;
  - selecting only image upload inputs;
  - filling rich-text body editors;
  - optionally declaring AI-generated content;
  - clicking the current `xhs-publish-btn` publish footer.
- Add `--ai-generated/--no-ai-generated`; when omitted, auto-detect common AI
  terms in title/body.
- Update README, README_EN, and skill docs with creator-login and AI-declaration
  behavior.
- Add unit tests for the command wiring and pure decision helpers.

## Non-Goals

- Do not add an unofficial publishing API.
- Do not bypass captcha, risk control, or manual security verification.
- Do not change search/read/interaction command behavior.
- Do not guarantee note_id extraction when the creator page does not expose it.

## Acceptance

- `xhs post "Title" --image image.png --content "Body" --json` remains the
  primary command.
- The command can publish image notes from the current creator page when the
  saved cookies include a valid creator-center session.
- `--ai-generated` and `--no-ai-generated` are documented and passed through to
  the publish client.
- Unit tests pass locally.

