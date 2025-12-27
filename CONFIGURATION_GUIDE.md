# Configuration Guide for Different Regions

This guide shows you exactly where to edit the source code to configure this dashboard for your own region and Google Sheets data.

## Quick Checklist

- [ ] Update Google Sheets ID
- [ ] Update Province Sheet GIDs
- [ ] Update Staff Assignment Sheet GID
- [ ] Update Province Names and Mappings
- [ ] Update Region Name in UI
- [ ] Update Meta Tags and Descriptions

---

## Google Sheets Configuration

The main config file is `src/hooks/useGoogleSheetData.ts`. This is where most of your changes will be.

### Update Google Sheets ID

**Line 4** - Replace with your Google Sheets ID:

```typescript
const SHEET_ID = 'YOUR_GOOGLE_SHEET_ID_HERE';
```

To find your Sheet ID, look at your Google Sheets URL - it's the long string between `/d/` and `/edit`.

### Update Staff Assignment Sheet GID

**Line 6** - Replace with your staff assignment sheet's GID:

```typescript
const STAFF_ASSIGNMENT_GID = YOUR_STAFF_SHEET_GID;
```

To find the GID, click on the sheet tab in Google Sheets and look at the URL for `gid=XXXX`. It's usually a number.

### Update Province Sheets Configuration

**Lines 10-16** - Update the `PROVINCE_SHEETS` object with your provinces:

```typescript
export const PROVINCE_SHEETS = {
  // Replace these with your province keys and data
  province1: { name: 'Province Name 1', gid: 123456789 },
  province2: { name: 'Province Name 2', gid: 987654321 },
  province3: { name: 'Province Name 3', gid: 456789123 },
  // Add or remove provinces as needed
} as const;
```

A few things to keep in mind:
- The `gid` is the sheet tab ID (found in the URL when you click on each tab)
- The `name` is what will be displayed in the UI
- The key (e.g., `province1`) is used internally - use lowercase, no spaces

### Update Province Name Mapping

**Lines 19-25** - Update the `PROVINCE_NAME_MAP` to match your staff assignment sheet:

```typescript
const PROVINCE_NAME_MAP: Record<string, ProvinceKey> = {
  'PROVINCE NAME IN SHEET': 'provinceKey',
  'ANOTHER PROVINCE': 'anotherKey',
  // Map the exact province names from your staff assignment sheet
  // (in UPPERCASE) to the keys you defined in PROVINCE_SHEETS
};
```

For example, if your staff sheet has "NCR" but your key is `ncr`, you would add:
```typescript
'NCR': 'ncr',
```

This mapping is case-insensitive, but I've been using uppercase in the map just to be safe.

---

## Region Name Updates

You'll need to update the region name in a few places. It's a bit scattered, but here's where to find them:

### Header Component

**File:** `src/components/dashboard/Header.tsx`  
**Line 17** - Update the region name:

```typescript
Western Visayas Regional Dashboard
```

Change it to your region, like `"NCR Regional Dashboard"` or `"Central Luzon Regional Dashboard"`.

### Footer

**File:** `src/pages/Index.tsx`  
**Line 245** - Update the footer text:

```typescript
<p>Scale-Up Results Dashboard • Western Visayas Region</p>
```

Just change "Western Visayas Region" to your region name.

### HTML Meta Tags

**File:** `index.html`  
**Lines 8-13** - Update all the meta tags:

```html
<title>Scale-Up Results Dashboard | Your Region Name</title>
<meta name="description" content="DSWD Scale-Up Results Dashboard for Your Region. Track validation progress for your provinces." />
<meta property="og:title" content="Scale-Up Results Dashboard | Your Region Name" />
<meta property="og:description" content="Track validation progress for your provinces in Your Region." />
```

### README

**File:** `README.md`  
**Line 5** - Update the description:

```markdown
DSWD Scale-Up Results Dashboard for Your Region. Track validation progress for your provinces.
```

---

## Google Sheets Structure

Your Google Sheets need to follow a specific structure for the code to work properly.

### Province Data Sheets

Each province sheet should have columns in this order:
- **Column 0 (A):** LGU/Municipality Name
- **Column 1 (B):** TARGET (100K) - or just TARGET if you're using the simple format
- **Column 2 (C):** SYSTEM VALIDATED (100K) - or SYSTEM VALIDATED if simple format
- **Column 3 (D):** VARIANCE (100K) - or VARIANCE if simple format
- **Column 4 (E):** TARGET (200K) - optional, for extended format
- **Column 7 (H):** TOTAL TARGET - optional, for extended format
- **Column 8 (I):** TOTAL VALIDATED - optional, for extended format
- **Column 9 (J):** TOTAL VARIANCE - optional, for extended format

The code automatically detects whether you're using the extended format (with TOTAL columns) or the simple format, so you don't need to configure that.

### Staff Assignment Sheet

This one's pretty straightforward. It should have:
- **Column 0 (A):** PROVINCES (header)
- **Column 1 (B):** STAFF ASSIGNED (header)
- **Row 2+:** Province names and their staff counts

Example:
```
| PROVINCES        | STAFF ASSIGNED |
|------------------|----------------|
| NCR              | 15             |
| Central Luzon   | 12             |
```

---

## Special Cases

### Provinces with Only 200K Targets

If you have any province that only has 200K targets (like NIR in the original), you'll need to add special handling in the `fetchSheetData` function around **line 138**:

```typescript
if (provinceKey === 'yourProvinceKey') {
  // Special handling for provinces with only 200K targets
  target100k = 0;
  target200k = parseNumber(columns[4] || '0');
  target = target200k;
  systemResult = parseNumber(columns[5] || '0');
  systemVariance = parseNumber(columns[6] || '0');
}
```

---

## Testing

After you make your changes, here's what to do:

1. Start the dev server:
   ```bash
   npm run dev
   ```

2. Check the browser console for any errors. If something's wrong, the console will usually tell you.

3. Verify everything works:
   - All provinces load correctly
   - Staff assignments show up
   - Data refreshes automatically
   - Sorting works on all columns

---

## Troubleshooting

### "Failed to fetch province data"

This usually means:
- Your Sheet ID or GIDs are wrong - double check them
- Your Google Sheet isn't set to "Anyone with the link can view" - make sure it's publicly accessible

### Staff assignments not showing

Check these:
- Verify the `STAFF_ASSIGNMENT_GID` is correct
- Make sure province names in the staff sheet match exactly (case-insensitive) with what's in `PROVINCE_NAME_MAP`
- Ensure the staff count column has actual numeric values, not text

### Wrong data columns

If the data is showing up in the wrong columns, your sheet structure probably doesn't match the expected format. You might need to adjust the column indices in the `fetchSheetData` function if your structure is different.

---

## Files You Need to Edit

Here's a quick summary of what needs to be changed:

1. `src/hooks/useGoogleSheetData.ts` - Main configuration (Sheet IDs, GIDs, province names)
2. `src/components/dashboard/Header.tsx` - Region name in header
3. `src/pages/Index.tsx` - Region name in footer
4. `index.html` - Meta tags and page title
5. `README.md` - Project description

That's it! Just these 5 files and you're done.

---

## Tips

- Keep a backup of the original configuration values before you start changing things
- Test with one province first before adding all of them - it's easier to debug
- Use descriptive province keys (like `ncr`, `centralLuzon`) instead of generic ones (`p1`, `p2`) - you'll thank yourself later
- Make sure all Google Sheets are publicly viewable, or set up proper authentication if you need to keep them private
