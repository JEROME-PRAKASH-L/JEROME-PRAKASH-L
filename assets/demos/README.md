# DEMO CAPTURE // Recording guide

Drop finished demo clips in this folder, then uncomment the **DEMO MEDIA** block in the
root `README.md` (search for `DEMO MEDIA`). Nothing renders until you do, so the profile
never shows a broken image while you are still recording.

## Why this matters

The profile currently describes six projects but shows no footage of any of them running.
A five-second clip of hand tracking moving a real cursor proves more than a paragraph does,
and it is the first thing a recruiter or collaborator will remember.

## Priority order

| Priority | Clip | File name | Why it earns the space |
|:---:|:---|:---|:---|
| 1 | Hand-gesture mouse control | `gesture-mouse-control.gif` | Instantly legible, visually striking, hard to fake |
| 2 | SkillBridge Local walkthrough | `skillbridge-local.gif` | Shows a shipped, deployed product with real UI |
| 3 | Volume control by gesture | `gesture-volume-control.gif` | Pairs with clip 1 to show a body of vision work |
| 4 | Document analysis extraction | `doc-analysis.gif` | Show a messy input becoming structured output |

## Capture settings

- **Length:** 5–10 seconds. Loop cleanly; cut the moment the point lands.
- **Width:** 1000–1200 px, then scale down. The README renders it at 80% width.
- **Frame rate:** 15–20 fps is plenty for UI and keeps the file small.
- **File size:** stay under **8 MB** per GIF. GitHub serves them through its image proxy,
  and anything larger is slow on mobile.
- **Format:** GIF is the safe default — it autoplays inline everywhere on GitHub.
  MP4 does *not* autoplay in a README `<img>` tag.

## Recording the vision projects

1. Close notifications and unrelated windows first — the clip is a portfolio piece.
2. Record the **whole screen**, so the webcam preview *and* the cursor it is driving are
   both visible. Showing only the hand-tracking window proves nothing moved.
3. Do one clear, deliberate gesture. Fast, jittery movement reads as a glitch.
4. If the tracking overlay draws landmarks, leave them on — they show the system working.

## Tools

| Platform | Tool |
|:---|:---|
| Windows | ScreenToGif (free, trims and exports GIF directly) |
| macOS | Kap, or QuickTime screen recording plus Gifski |
| Linux | Peek, or `ffmpeg` + `gifski` |
| Any | OBS to record MP4, then convert |

Converting an MP4 to a well-sized GIF:

```bash
ffmpeg -i demo.mp4 -vf "fps=18,scale=1100:-1:flags=lanczos" -c:v pam \
  -f image2pipe - | gifski --fps 18 --quality 90 -o gesture-mouse-control.gif -
```

## Before you commit

- [ ] Clip is under 8 MB
- [ ] No personal data, private messages, or credentials visible on screen
- [ ] The action is obvious to someone who has never seen the project
- [ ] `alt` text in the README describes what happens, not just the project name
