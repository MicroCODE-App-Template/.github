# Metric Color Mapping Table

This document defines the unique colors assigned to each of the 10 dashboard metrics. These colors are used consistently in:

1. The count value displayed in the stat block
2. The line color in the growth chart

## Color Definitions Table

| #   | Metric     | Color Name | Hex Color | RGB                | Stat Block            | Chart Line          |
| --- | ---------- | ---------- | --------- | ------------------ | --------------------- | ------------------- |
| 0   | Accounts   | `blue`     | `#77AFE8` | rgb(119, 175, 232) | `valueColor='blue'`   | `color[0]='blue'`   |
| 1   | Users      | `red`      | `#F9627A` | rgb(249, 98, 122)  | `valueColor='red'`    | `color[1]='red'`    |
| 2   | Boats      | `green`    | `#75CD9F` | rgb(117, 205, 159) | `valueColor='green'`  | `color[2]='green'`  |
| 3   | Orgs       | `purple`   | `#8D8CC3` | rgb(141, 140, 195) | `valueColor='purple'` | `color[3]='purple'` |
| 4   | Clubs      | `orange`   | `#FF8C42` | rgb(255, 140, 66)  | `valueColor='orange'` | `color[4]='orange'` |
| 5   | Active     | `teal`     | `#4ECDC4` | rgb(78, 205, 196)  | `valueColor='teal'`   | `color[5]='teal'`   |
| 6   | Online     | `cyan`     | `#00CED1` | rgb(0, 206, 209)   | `valueColor='cyan'`   | `color[6]='cyan'`   |
| 7   | Offline    | `gray`     | `#95A5A6` | rgb(149, 165, 166) | `valueColor='gray'`   | `color[7]='gray'`   |
| 8   | Onboarding | `yellow`   | `#FFD93D` | rgb(255, 217, 61)  | `valueColor='yellow'` | `color[8]='yellow'` |
| 9   | Disabled   | `pink`     | `#FF6B9D` | rgb(255, 107, 157) | `valueColor='pink'`   | `color[9]='pink'`   |

## Implementation

### Colors JSON (`admin/console/src/components/chart/colors.json`)

All colors are defined in the colors.json file with the following structure:

```json
{
  "colorName": {
    "borderColor": "#HEXCOLOR",
    "backgroundColor": ["#HEXCOLOR", ...],
    "transparentColor": "rgba(...)",
    "pointRadius": 4,
    "pointHoverRadius": 5,
    "pointBorderWidth": 2,
    "pointBackgroundColor": "#FFFFFF",
    "pointHoverBackgroundColor": "#FFFFFF",
    "pointHoverBorderColor": "#HEXCOLOR"
  }
}
```

### Dashboard Component (`admin/console/src/views/dashboard.jsx`)

Each Stat component uses the `valueColor` prop to set the count color:

```jsx
<Stat
    value={stats?.data?.totalAccounts}
    valueColor='blue'  // Uses blue color for count
    ...
/>
```

### Chart Component (`admin/console/src/views/dashboard.jsx`)

The Chart component receives an array of colors matching the dataset order:

```jsx
<Chart
    color={['blue', 'red', 'green', 'purple', 'orange', 'teal', 'cyan', 'gray', 'yellow', 'pink']}
    showLegend={false}  // Legend removed as requested
    ...
/>
```

### Backend Growth Function (`admin/model/metrics.model.js`)

The datasets array order must match the color array order:

```javascript
const datasets = [
  { label: "Accounts", data: [] }, // Index 0 -> blue
  { label: "Users", data: [] }, // Index 1 -> red
  { label: "Boats", data: [] }, // Index 2 -> green
  { label: "Orgs", data: [] }, // Index 3 -> purple
  { label: "Clubs", data: [] }, // Index 4 -> orange
  { label: "Active", data: [] }, // Index 5 -> teal
  { label: "Online", data: [] }, // Index 6 -> cyan
  { label: "Offline", data: [] }, // Index 7 -> gray
  { label: "Onboarding", data: [] }, // Index 8 -> yellow
  { label: "Disabled", data: [] }, // Index 9 -> pink
];
```

## Color Order Consistency

The order must be consistent across:

1. Dashboard Stat components (top to bottom, left to right)
2. Chart color array
3. Backend datasets array

Current order: Accounts → Users → Boats → Orgs → Clubs → Active → Online → Offline → Onboarding → Disabled
