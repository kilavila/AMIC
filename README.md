# AMIC
Automated Magick Image Converter. Optimizing images for web development.

> Always read through scripts before running them!
> 
> If you don't understand what it does, then ask someone..

```bash
#!/bin/bash
#
# ----------------------------------
#  Automated Magick Image Converter
# ----------------------------------
#
# Requirements:
#   - imagemagick
#   - inotify
#   - bc
#   - awk
#
# Author:  https://github.com/kilavila
#
# Copyright 2024 kilavila_
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# README
#   - This script converts images to webp and jpg
#   - Install required tools
#   - Run script
#   - Drop images in your pictures directory
#   - Converted files will be stored in optimized directory
#

user=$(whoami)
# CHANGE 'pictureDir' TO YOUR PICTURES DIRECTORY IF NECESSARY
pictureDir="/home/$user/Pictures"
webpDir="$pictureDir/optimized/webp"
jpgDir="$pictureDir/optimized/jpg"

mkdir -p "$webpDir"
mkdir -p "$jpgDir"

inotifywait -m -e create -e moved_to "$pictureDir" |
  while read path event file; do
    if [[ $file == *.jpg ]] || [[ $file == *.jpeg ]] || [[ $file == *.png ]] || [[ $file == *.gif ]] || [[ $file == *.webp ]]; then

      imageWidth=$(identify "$path$file" | awk '{print $3}' | cut -d'x' -f1)
      imageHeight=$(identify "$path$file" | awk '{print $3}' | cut -d'x' -f2)
      resizeWidth1440p=1440
      resizeWidth1080p=1080
      resizeWidth720p=720
      resizeWidth480p=480
      resizeWidth360p=360

      if [[ $imageWidth -gt $imageHeight ]]; then
        resizeWidth1440p=2560
        resizeWidth1080p=1920
        resizeWidth720p=1280
        resizeWidth480p=854
        resizeWidth360p=640
      fi

      # 2560x1440, 1440x2560
      imageQuality=75
      resizeRatio=$(echo "scale=7; $imageHeight / $imageWidth" | bc)
      resizeHeight=$(echo "scale=0; $resizeRatio * $resizeWidth1440p" | bc | cut -d'.' -f1)

      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality "$imageQuality" "$webpDir/${file%.*}.1440p.webp"
      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality "$imageQuality" "$jpgDir/${file%.*}.1440p.jpg"

      # 1920x1080, 1080x1920
      imageQuality=80
      resizeRatio=$(echo "scale=7; $imageHeight / $imageWidth" | bc)
      resizeHeight=$(echo "scale=0; $resizeRatio * $resizeWidth1080p" | bc | cut -d'.' -f1)

      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality "$imageQuality" "$webpDir/${file%.*}.1080p.webp"
      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality "$imageQuality" "$jpgDir/${file%.*}.1080p.jpg"

      # 1280x720, 720x1280
      imageQuality=85
      resizeHeight=$(echo "scale=0; $resizeRatio * $resizeWidth720p" | bc | cut -d'.' -f1)

      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality "$imageQuality" "$webpDir/${file%.*}.720p.webp"
      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality "$imageQuality" "$jpgDir/${file%.*}.720p.jpg"

      # 854x480, 480x854
      imageQuality=90
      resizeHeight=$(echo "scale=0; $resizeRatio * $resizeWidth480p" | bc | cut -d'.' -f1)

      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality "$imageQuality" "$webpDir/${file%.*}.480p.webp"
      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality "$imageQuality" "$jpgDir/${file%.*}.480p.jpg"

      # 640x360, 360x640
      imageQuality=95
      resizeHeight=$(echo "scale=0; $resizeRatio * $resizeWidth360p" | bc | cut -d'.' -f1)

      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality "$imageQuality" "$webpDir/${file%.*}.360p.webp"
      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality "$imageQuality" "$jpgDir/${file%.*}.360p.jpg"

    fi
  done
```
