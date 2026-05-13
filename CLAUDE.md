# Vehicle Detection — Project Context

## What This Does
Processes a top-down traffic video with YOLO v8s + ByteTrack to track vehicles and
record each one's Closest Point of Approach (CPA) — the frame/timestamp where it was
nearest to the camera. Outputs annotated video + JSON/CSV CPA data.

## Key Files
| File | Role |
|---|---|
| `main.py` | Everything: `BBoxSmoother`, `ProximityTracker`, `VideoProcessor` |
| `bytetrack.yaml` | ByteTrack tracker thresholds |
| `requirements.txt` | `ultralytics>=8.0.0`, `opencv-python>=4.8.0` |

## Run
```bash
python main.py <input_video> --output result.mp4
# Model downloads automatically on first run (yolov8s.pt)
```

## Architecture Notes

**BBoxSmoother** — separates position (cx,cy) and size (w,h) smoothing:
- Position: adaptive EMA, alpha scales with displacement (`base=0.15`, `max=0.9`)
- Size: fixed EMA (`size_alpha=0.35`) — suppresses YOLO edge noise on large nearby objects
- Coasting: velocity extrapolation clamped to frame bounds to prevent boxes disappearing

**Epsilon threshold** (`EPSILON_THRESHOLD = 0.88`) — bottom 12% of frame. Boxes turn
green when a vehicle crosses this line. CPA is confirmed when the track expires.

**TRACK_PATIENCE = 30** — frames a vehicle may be absent before its CPA is confirmed.
During grace period the last box is held via velocity extrapolation (coasting).

**Detection** — runs on original resolution for stability, then scales for drawing.
All detected classes tagged as `"vehicle"` regardless of COCO class ID.

## Tuning Knobs
- `bytetrack.yaml` thresholds: raise `track_high_thresh` to reduce false positives
- `BBoxSmoother.size_alpha`: lower = smoother size growth, higher = more responsive
- `TRACK_PATIENCE`: higher = longer coasting before CPA confirmed
- `--scale`: output resolution multiplier (default 0.5)
