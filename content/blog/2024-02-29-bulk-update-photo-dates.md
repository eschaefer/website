---
title: 'Bulk fixing photo and video dates'
tags: [blog, bash]
published: true
---

I have a lot of old digital photos with the correct date in the filename. But that date doesn't match anymore with the file creation date. This can pose a big problem for a some photo library software out there, which depends on file creation date to correctly organize the photos (looking at you Google Photos).

Most of my photos generated from a digital camera or phone have dates somewhere in the file names, formatted something like:

```
IMG_20161226_115358.jpg
20180101_141437.jpg
VID_20171202_124719.mp4
```

There _is_ a date in there. If only I could parse the date in each file, and manually set the creation date for each one accordingly...

## Regular expressions to the rescue

The date portion of the filename can be represented by the following regular expression:

```
[0-9]{8}_[0-9]{6}
```

In simple terms it means match any 8 digits (yyyymmdd) followed by `_`, followed by any 6 digits (hhmmss).

Here's a quick [regexr playground](https://regexr.com/7so6l) to look at it in action.

Using this in a bash script could be done like so:

```bash
#!/bin/bash

filename="IMG_20161226_115358.jpg"

if [[ "$filename" =~ ([0-9]{8}_[0-9]{6}) ]]; then
  date_from_filename="${BASH_REMATCH[0]}"
  echo "$date_from_filename" # prints 20161226_115358
fi
```

We have the date formatted like `yyyymmdd_hhmmss`. But we need to convert it into a generic timestamp that can be used. To do that we can use the `date` utility to accept a predefined date format, and parse it into a proper system date.

```bash
date -jf "%Y%m%d_%H%M%S" "20161226_115358"
# Mon Dec 26 11:53:58 CET 2016
```

We can also output the parsed date into a format that can be used by the `touch` utility, to set the file creation date...

```bash
date -jf "%Y%m%d_%H%M%S" "20161226_115358" +"%Y%m%d%H%M%S"
# 20161226115358
```

Jumping ahead a few steps, we can assemble these building blocks into a full bash script that can:

1. Accept a directory of files
2. For each file, extract a date from the file name
3. Parse the date and reformat it to be used to modify the file creation date
4. Gracefully fail in any of these steps with a helpful message

```bash
#!/bin/bash

# Usage example: bash date-fixer.sh ~/Pictures/backup/

# Check if the correct number of arguments is provided
if [ "$#" -ne 1 ]; then
  echo "Usage: $0 <directory_path>"
  exit 1
fi

directory_path="$1"

# Check if the directory exists
if [ ! -d "$directory_path" ]; then
  echo "Error: Directory not found: $directory_path"
  exit 1
fi

# Loop through all files in the directory
for filename in "$directory_path"/*; do
  # Extract just the filename without the full path
  basename=$(basename "$filename")

   # Skip hidden files
  if [[ "$basename" == .* ]]; then
    continue
  fi

  # Extract date from filename
  if [[ "$basename" =~ ([0-9]{8}_[0-9]{6}) ]]; then
    date_from_filename="${BASH_REMATCH[0]}"
    echo "Date from filename: $date_from_filename"

    # Convert parsed date and time to timestamp
    timestamp=$(date -jf "%Y%m%d_%H%M%S" "$date_from_filename" +"%Y%m%d%H%M%S")

    # Check for valid timestamp
    if [[ ! "$timestamp" =~ ^[0-9]+$ ]]; then
      echo "Invalid timestamp for $filename: $timestamp"
      continue
    fi

    # Modify file's creation time
    touch -t "$timestamp" "$filename"
    echo "Modified creation time of $filename"
  else
    echo "Skipping file without date in filename: $file"
  fi
done
```
