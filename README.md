# EORE Dashboard

A public dashboard for EORE activities that **never touches the raw Survey123 layer**.

```
Survey123 EORE layer (private)
        │  ArcGIS Notebook, scheduled (EORE_Summary_Builder.ipynb)
        ▼
EORE_Public_Summary (private, editable)   ← 4 tables of totals only
        │  hosted view
        ▼
EORE_Public_Summary_View (public, query-only, export off)
        │
        ▼
index.html on GitHub Pages
```

Anyone can open the browser developer tools and see the view's URL. All they find
there are totals by month, district, organisation and donor. They won't find names,
photos, IDs, review comments, "Other" free text, community names or exact coordinates.

## Files

| File | What it is |
|---|---|
| `index.html` | The dashboard. Upload it to GitHub. |
| `demo-data.js` | Made-up data so you can preview the page. Delete it once you go live. |
| `notebook/EORE_Summary_Builder.ipynb` | Upload this to ArcGIS Online Notebooks. |
| `notebook/eore_summary_builder.py` | The same code as a plain script, if you want to read or edit it. |

## Setup

### 1. Run the notebook in ArcGIS Online
1. In ArcGIS Online go to **Notebook > New notebook > Upload**, choose `EORE_Summary_Builder.ipynb`, and use the **Standard** runtime.
2. In the settings cell, fill in `PORTAL_URL` and `PORTAL_USERNAME`. Leave `PORTAL_PASSWORD` empty and the notebook
   will ask for it when you run it by hand. Choose `PUBLISH_TO`: `"portal"` creates the summary in that portal,
   `"home"` creates it in the ArcGIS Online account the notebook runs in.
   Set `SOURCE_LAYER_URL` to your EORE layer, ending in `/FeatureServer/0`.
   Set `SHARE_VIEW_PUBLICLY = False` for the first run so you can check the figures before anything goes public.
3. Run all cells. The first run creates the private `EORE_Public_Summary` service and the `EORE_Public_Summary_View` view.
   At the end it prints the **view URL**.
4. Open the view item in ArcGIS Online and check it. **Settings** should show editing off and
   "Allow others to export" off. Look at the data tab: only totals.
5. Set `SHARE_VIEW_PUBLICLY = True` and run it again.
6. Schedule it: **Tasks > Create task**, for example daily at night. A scheduled run can't ask for the
   password, so for scheduled runs the password has to be typed into `PORTAL_PASSWORD`. Keep that notebook private
   in ArcGIS Online and never put it on GitHub. Only `index.html` (and the README) belong in the repository.

**Publishing to an ArcGIS Enterprise portal (`PUBLISH_TO = "portal"`)?** The dashboard only works if the portal can be
reached from the internet, sharing with Everyone is allowed, and the portal's allowed origins (CORS) include your
GitHub Pages address (`https://<user>.github.io`). If any of those isn't true, use `PUBLISH_TO = "home"`: the
notebook reads from the portal and publishes the summary to ArcGIS Online.

### 2. Put the dashboard on GitHub
1. In `index.html`, find `CONFIG` near the top and paste the view URL into `viewUrl`.
   You can also change `title`, `subtitle` and `country` there.
2. Delete `demo-data.js`, or leave it: it's ignored once `viewUrl` is set.
3. Push `index.html` to your repository and enable **Settings > Pages**.

If `viewUrl` is empty, the page shows the demo data with a yellow banner.

## What the dashboard shows

**Filters:** year, governorate, implementing organisation, donor.

**Headline figures:** people reached in sessions, with the girls/boys/women/men split; sessions; people per session;
persons with disabilities; communities reached; people trained (ToT); mass and digital media estimated reach.

**Charts:** people reached or sessions by month; map by district; people by governorate and sex/age; session venue;
target group; session type; organisations; disability by sex/age; media channels; ToT groups.

Every chart has a **Table** button, and the page has a light/dark toggle.

Counting rules, also printed on the page:
- **People reached** counts interpersonal EORE participants only (`benef_total`).
- **Mass media reach** is an estimate (`benef_mdm_total`). It's shown separately and never added to people reached.
- **ToT** participants are counted separately (`benef_tot_total`).
- Only reports with `sys_workflow_status = finalised` are counted (`FINALISED_ONLY`).
- Dates use `start_date` (falling back to `report_date`), converted to `Asia/Damascus` time so dates near midnight stay in the right month.

## Privacy settings in the notebook
- District map points are the average of activity locations **rounded to 1 decimal (~11 km)**. Change this with `DISTRICT_COORD_DECIMALS`,
  or supply official centroids in `DISTRICT_CENTROIDS`.
- Community names are only used to count communities. They are never published.

## Fix in the XLSForm before relying on the sex/age split
1. `choices` → `eore_tot_training_purpose`: the labels are shifted by one row
   (`Safety Training` shows "Teacher Training ToT", `Teacher Training` shows "Humanitarian/Development Actor ToT").
2. `survey` → Arabic labels for children are swapped: "Male - Child" shows طفلة and "Female - Child" shows طفل… ذكر,
   in the interpersonal, disability and media groups.

## If the summary layout changes
If you add new fields to the summary later, delete both `EORE_Public_Summary` and its view in ArcGIS Online and run the
notebook again. It recreates them, but the view URL changes, so update `CONFIG.viewUrl`.
