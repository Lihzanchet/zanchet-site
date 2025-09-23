#!/usr/bin/env bash
set -euo pipefail

N="${1:-20}" # how many recent photos
DEST="./trip-sync-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$DEST"

echo "Listing recent $N photos from DCIM/Camera..."
# Index on device, most-recent first
LIST=$(adb shell "ls -1t /sdcard/DCIM/Camera/*.jpg 2>/dev/null | head -n $N" | tr -d '\r')

if [ -z "$LIST" ]; then
  echo "No photos found." && exit 0
fi

echo "$LIST" | nl
echo "Pulling..."
# Pull one by one so we get progress
while IFS= read -r P; do
  [ -z "$P" ] && continue
  adb pull "$P" "$DEST/" || true
done <<< "$LIST"

COUNT=$(ls -1 "$DEST" | wc -l | tr -d ' ')
ZIP="$DEST.zip"
echo "Zipping $COUNT files → $ZIP"
zip -qr "$ZIP" "$DEST"

echo "SHA256:"
sha256sum "$ZIP" || shasum -a 256 "$ZIP"

echo "Done ✅  See: $ZIP"
