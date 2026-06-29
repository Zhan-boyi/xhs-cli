# Design: Creator-Center Posting Flow

## Fixed Flow

```text
CLI post command
  -> resolve image paths
  -> XhsClient.publish_note(...)
  -> open https://creator.xiaohongshu.com/publish/publish
  -> ensure creator login is not required
  -> switch to "上传图文"
  -> find image upload input
  -> upload images
  -> fill title
  -> fill body
  -> optionally declare AI-generated content
  -> click publish footer
  -> wait for success signal
```

## Auth Boundary

Publishing relies on browser session cookies. The CLI must not bypass login,
captcha, or security verification.

The user-visible behavior is:

- if normal saved cookies are missing, ask the user to run `xhs login`;
- if creator-center login is required, ask the user to open
  `https://creator.xiaohongshu.com`, finish login, then run `xhs login` again;
- if risk-control/captcha appears, stop and let the user complete it manually.

## Why Use Page Helpers

The creator page is not a stable API. The implementation keeps the public
surface small and isolates page-specific assumptions in helper methods:

- `_click_creator_image_tab`
- `_find_creator_image_file_input`
- `_is_image_file_input`
- `_maybe_select_ai_declaration`
- `_click_creator_publish_button`
- `_current_publish_result`

This makes future page changes easier to update without changing the CLI
contract.

## Publish Button Handling

The current creator page renders the publish footer as a closed-shadow
`xhs-publish-btn` custom element. Text selectors such as
`button:has-text("发布")` can fail because the actual button is not exposed as a
normal DOM button.

The implementation first targets `xhs-publish-btn` and clicks the known publish
region inside that footer. It then falls back to older normal-button selectors.

## AI Content Declaration

The command supports three modes:

- `--ai-generated`: always try to select "笔记含AI合成内容";
- `--no-ai-generated`: never select it;
- omitted: infer from common AI keywords in title/body.

If the creator page does not expose the declaration control, publishing
continues without failing.

## Result Semantics

`--json` returns:

```json
{
  "success": true,
  "note_id": ""
}
```

`note_id` is best-effort. A publish can be successful even when the creator page
does not expose a parseable note id.

