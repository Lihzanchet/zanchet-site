#!/usr/bin/env bash
set -euo pipefail

TS=$(date +%Y%m%d-%H%M%S)
REMOTE="/sdcard/Pictures/travel-postcard-$TS.png"
LOCAL="./travel-postcard-$TS.png"

echo "Capturing screenshot on device..."
adb shell screencap -p "$REMOTE"

echo "Pulling to host..."
adb pull "$REMOTE" "$LOCAL"

echo "Verifying file..."
file "$LOCAL" || true
echo "Saved: $LOCAL"
