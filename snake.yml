name: Generate DIWAKARA Snake

on:
  schedule:
    - cron: "0 */12 * * *"
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  generate:
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - uses: actions/checkout@v3

      - name: Generate DIWAKARA Snake SVG
        run: |
          cat > diwakara-snake.py << 'PYEOF'
          import math

          CELL = 18
          GAP = 2
          STEP = CELL + GAP

          # Pixel font for each letter (6 cols wide, 5 rows tall)
          LETTERS = {
              'D': [(0,0),(0,1),(0,2),(0,3),(0,4),(1,0),(2,0),(3,1),(3,2),(3,3),(1,4),(2,4)],
              'I': [(1,0),(2,0),(3,0),(2,1),(2,2),(2,3),(1,4),(2,4),(3,4)],
              'W': [(0,0),(4,0),(0,1),(4,1),(1,2),(3,2),(2,3),(0,3),(4,3),(0,4),(4,4)],
              'A': [(2,0),(1,1),(3,1),(0,2),(4,2),(0,3),(1,3),(2,3),(3,3),(4,3),(0,4),(4,4)],
              'K': [(0,0),(0,1),(0,2),(0,3),(0,4),(3,0),(2,1),(1,2),(2,3),(3,4)],
              'R': [(0,0),(0,1),(0,2),(0,3),(0,4),(1,0),(2,0),(3,1),(1,2),(2,2),(3,3),(3,4)],
          }

          NAME = ['D','I','W','A','K','A','R','A']
          SPACING = 6  # columns between letter starts

          def get_cells():
              cells = []
              col_offset = 1
              for letter in NAME:
                  for (c, r) in LETTERS[letter]:
                      cells.append((c + col_offset, r + 1))
                  col_offset += SPACING
              return cells

          cells = get_cells()
          cell_set = set(cells)

          max_c = max(c for c,r in cells)
          max_r = max(r for c,r in cells)

          W = (max_c + 3) * STEP
          H = (max_r + 3) * STEP

          # Snake path: row by row through letter cells
          def build_path():
              path = []
              for r in range(max_r + 2):
                  row = sorted([c for (c, rr) in cells if rr == r])
                  if not row:
                      continue
                  if r % 2 == 0:
                      path.extend([(c, r) for c in row])
                  else:
                      path.extend([(c, r) for c in reversed(row)])
              return path

          path = build_path()
          N = len(path)
          SNAKE_LEN = 7
          FRAME_COUNT = N + SNAKE_LEN + 5
          DURATION = FRAME_COUNT * 0.12  # seconds

          def make_keyframes(idx):
              # When is this cell lit as part of the snake?
              vals = ['0'] * FRAME_COUNT
              for f in range(FRAME_COUNT):
                  head = f % N
                  for i in range(SNAKE_LEN):
                      segment = (head - i) % N
                      if segment == idx:
                          brightness = (SNAKE_LEN - i) / SNAKE_LEN
                          vals[f] = f'{brightness:.2f}'
                          break
              return ';'.join(vals)

          lines = []
          lines.append(f'<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 {W} {H}" style="background:#0d1117">')
          lines.append('<defs><style>.c{rx:3;ry:3;}</style></defs>')

          for i, (c, r) in enumerate(path):
              x = c * STEP
              y = r * STEP
              kf = make_keyframes(i)
              lines.append(f'<rect class="c" x="{x}" y="{y}" width="{CELL}" height="{CELL}" fill="#1a3a1a"/>')
              lines.append(
                  f'<rect class="c" x="{x}" y="{y}" width="{CELL}" height="{CELL}" fill="#39d353" opacity="0">'
                  f'<animate attributeName="opacity" values="{kf}" dur="{DURATION:.1f}s" repeatCount="indefinite"/>'
                  f'</rect>'
              )

          lines.append('</svg>')

          svg = '\n'.join(lines)
          with open('diwakara-snake.svg', 'w') as f:
              f.write(svg)

          print(f"Generated diwakara-snake.svg ({len(path)} cells, {FRAME_COUNT} frames, {DURATION:.1f}s)")
          PYEOF
          python3 diwakara-snake.py

      - name: Push to output branch
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_branch: output
          publish_dir: .
          include_files: diwakara-snake.svg
          keep_files: true
