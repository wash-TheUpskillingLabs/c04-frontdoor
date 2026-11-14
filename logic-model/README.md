# Logic model page

The public logic model for The Upskilling Labs, built from [the PRD](../docs/PRD.md). Live at `/logic-model/` on this site.

All content lives in one file, [`content.json`](content.json). You don't need to touch the HTML to change what the page says.

## Updating the content

1. Open `content.json` on GitHub and click the pencil to edit.
2. Change the text you need. Keep the quotes, commas and brackets around it exactly as they are.
3. Set `lastUpdated` to today's date, written `YYYY-MM-DD`.
4. Commit to a new branch and open a pull request. The accuracy reviewer checks it against program records before it merges.

If the page shows "The model could not load" after a change, a comma or quote is probably missing. Paste the file into any JSON validator to find it.

## Rules for content (from the PRD)

- Each column holds 4 to 6 items, one plain sentence each.
- Each item has one or more `tags`: `tools`, `content`, `volunteering`, `learning`.
- Inputs, activities and outputs come from program records. Outcomes come from the working session.
- Outcomes are written as intended results, never as claims.
- Numbers only from real data or agreed targets. Anything still blank goes in square brackets, like `[#]` or `[X]%`, and shows on the page with a dotted underline.
- No personal details about members without their written consent.
- Review each quarter, alongside the Build Cycle and Summit calendar.

## Going live

While `status` is `"draft"`, the page shows a "Not final" notice. When the product owner signs off, change `status` to `"final"`. That removes the notice.

Before launch:

- Replace the `[bracketed]` placeholders, including `owner`.
- Replace the three links marked `confirm` with the real join, donate and events pages.
- Change `og:image` in `index.html` to the page's full URL, for example `https://<domain>/logic-model/og.png`.

## Sharing a filtered view

Pick a workstream and use **Copy link to this view**. The link opens with that workstream highlighted, for example `/logic-model/?workstream=volunteering`.

## Printing for grant packets

Print the page or save it as PDF from the browser. It prints in landscape on a white background, with all six columns side by side and link addresses written out. If a workstream is selected when you print, its items are outlined.
