
## Implementation Details

### Architecture

- **No frameworks or build tools** — pure HTML, CSS, and vanilla JavaScript
- **Module pattern**: Each algorithm is a self-contained module that registers with the core `AlgorithmLab` controller (name kept for internal consistency)
- **Animation engine**: A step-based queue system with play/pause/step-forward/back/reset functionality
- **Shared rendering**: Common visualization helpers (bar charts, array displays, matrix views, 2D canvas) used across algorithms

### Visualization Types

- **Bar Charts**: Used by Merge Sort, Quick Sort, Min/Max, and Largest Subarray Sum
- **Matrix Grids**: Used by Matrix Multiplication and Strassen's Algorithm
- **2D Canvas**: Used by Closest Pair of Points and Convex Hull for point/polygon rendering

### Design

- Soft light‑mode theme with off‑white backgrounds and subtle shadows
- Color-coded phases: Cyan (Divide), Purple (Conquer), Emerald (Combine)
- Category colors: Sorting (Cyan), Matrix (Purple), Search (Amber), Geometry (Emerald)
- Smooth CSS transitions and micro-animations
- Google Fonts: Inter (UI) + JetBrains Mono (code/values)

## Browser Compatibility

Tested on modern browsers supporting ES6+, CSS Grid, Flexbox, and Canvas API:
- Google Chrome 90+
- Mozilla Firefox 88+
- Microsoft Edge 90+
- Safari 14+

## Author

Created for Design and Analysis of Algorithms (DAA) Lab.