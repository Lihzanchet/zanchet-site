#!/usr/bin/env bash
set -euo pipefail

echo "== Travel Quickcheck =="
adb devices

DEVICE=$(adb devices | awk 'NR==2{print $1}')
[ -z "${DEVICE:-}" ] && { echo "No device found."; exit 1; }

echo -e "\n[device] $DEVICE"
MODEL=$(adb -s "$DEVICE" shell getprop ro.product.model | tr -d '\r')
ANDROID=$(adb -s "$DEVICE" shell getprop ro.build.version.release | tr -d '\r')
BATTERY=$(adb -s "$DEVICE" shell dumpsys battery | awk -F': ' '/level:/{print $2}' | tr -d '\r')
WIFI=$(adb -s "$DEVICE" shell dumpsys wifi | awk -F': ' '/Wi-Fi is /{print $2; exit}' | tr -d '\r')

echo "Model: $MODEL"
echo "Android: $ANDROID"
echo "Battery: ${BATTERY}%"
echo "Wi-Fi: $WIFI"

echo -e "\n[storage]"
adb -s "$DEVICE" shell df -h /sdcard | sed '1,1!b; s/^/mountpoint usage/'

echo -e "\n[timezone/locale]"
TZ=$(adb -s "$DEVICE" shell getprop persist.sys.timezone | tr -d '\r')
LOC=$(adb -s "$DEVICE" shell getprop persist.sys.locale | tr -d '\r')
echo "TZ: ${TZ:-unknown} | Locale: ${LOC:-unknown}"

echo -e "\nDone ✅"
