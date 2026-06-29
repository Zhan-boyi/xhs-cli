# Spec: Creator Posting

## Requirement: Stable Image Note Command

The CLI SHALL publish image notes through:

```bash
xhs post "Title" --image image.png --content "Body" --json
```

### Scenario: Multiple Images

- GIVEN the user passes one or more `--image` paths
- WHEN the command runs
- THEN the CLI SHALL upload all provided images in one creator-center publish
  session.

### Scenario: Missing Image

- GIVEN an image path does not exist
- WHEN the command runs
- THEN the CLI SHALL fail before opening the creator page.

## Requirement: Creator-Center Page Adaptation

The CLI SHALL adapt to the current creator-center image-note page structure.

### Scenario: Page Defaults To Video Upload

- GIVEN the creator publish page opens in video mode
- WHEN `xhs post` runs
- THEN the CLI SHALL switch to "上传图文" before looking for a file input.

### Scenario: Multiple File Inputs

- GIVEN the page contains both image and video file inputs
- WHEN selecting the upload input
- THEN the CLI SHALL select an input whose `accept` supports image formats and
  SHALL reject video-only inputs.

### Scenario: Closed Publish Footer

- GIVEN the page exposes `xhs-publish-btn`
- WHEN publishing
- THEN the CLI SHALL click the publish area in that creator footer before trying
  legacy text-button selectors.

## Requirement: AI Declaration

The CLI SHALL allow explicit and automatic AI-generated content declaration.

### Scenario: Explicit AI Declaration

- GIVEN `--ai-generated`
- WHEN the creator page exposes "笔记含AI合成内容"
- THEN the CLI SHALL attempt to select that declaration.

### Scenario: Explicit No AI Declaration

- GIVEN `--no-ai-generated`
- WHEN publishing
- THEN the CLI SHALL NOT select the AI content declaration.

### Scenario: Auto Detection

- GIVEN neither flag is passed
- WHEN the title or content contains common AI terms
- THEN the CLI SHALL attempt to select the AI content declaration.

## Requirement: Auth And Verification Boundary

The CLI SHALL not bypass platform login, captcha, or security verification.

### Scenario: Creator Login Missing

- GIVEN the creator publish page redirects to creator login
- WHEN publishing
- THEN the CLI SHALL fail with a creator-login-required error.

### Scenario: Note ID Missing

- GIVEN the creator page confirms publish but does not expose a note id
- WHEN `--json` is used
- THEN the CLI MAY return an empty `note_id` with `success=true`.

