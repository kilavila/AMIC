# AMIC
Automated Magick Image Converter. Optimizing images for web development.

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
      resizeWidth=1080
      imageQuality=80

      if [[ $imageWidth -gt $imageHeight ]] && [[ $imageWidth -gt 2000 ]]; then
        resizeWidth=1920
      fi

      if [ $(wc -c <"$path$file") -lt 200000 ]; then
        imageQuality=100
      elif [ $(wc -c <"$path$file") -lt 350000 ]; then
        imageQuality=90
      fi

      resizeRatio=$(echo "scale=7; $imageHeight / $imageWidth" | bc)
      resizeHeight=$(echo "scale=0; $resizeRatio * $resizeWidth" | bc | cut -d'.' -f1)

      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality 80 "$webpDir/${file%.*}.desktop.webp"
      convert "$path$file" -resize "$resizeWidth"x"$resizeHeight" -quality 80 "$jpgDir/${file%.*}.desktop.jpg"
    fi
  done
```
