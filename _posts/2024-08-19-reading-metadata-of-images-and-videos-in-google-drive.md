---
layout: post
title:  "Reading image and video metadata in google drive folders recursively"
date:   2024-08-19 15:40:00
subtitle: 
img:
tags: [python,metadata,image,video] # add tag
published: true
---
## Reading image and video metadata in google drive folders recursively

### First create an OAuthClient using google cloud

#### Enable the Google Drive API

Navigate to "APIs & Services" > "Library".
Search for "Google Drive API" and enable it.

#### Create OAuth 2.0 Credentials

- Go to "APIs & Services" > "Credentials".
- Click "Create Credentials" and select "OAuth client ID".
- Configure the consent screen. For development, you can set it as an "Internal" app, which will allow only users in your organization (or the account you're using) to test it. If you choose "External," you'll need to verify the app for public use.
- In the test user add the google email ID which you will use for authenticating the consent screen.

### Python script to extract image and video metadata

We will utilize the pydrive2 library to access google drive contents. Here is ths script. When the script runs, first it will go to the app consent screen, you should log in to your gmail that you added in the previous configuration of the consent screen, then it should run. Make sure this same email access has at least read access to the google drive folder that you wish to crawl:

```python
# %%
from pydrive2.auth import GoogleAuth
from pydrive2.drive import GoogleDrive
from PIL import Image, ExifTags, TiffImagePlugin
import os
import subprocess
import json
import tempfile
from pprint import pprint
import csv
from fractions import Fraction
# %%
# Authenticate and create the PyDrive client.
gauth = GoogleAuth()
# Creates local webserver and automatically handles authentication.
gauth.LocalWebserverAuth()
drive = GoogleDrive(gauth)
# %%
def list_drive_files(folder_id):
    """List all files in the Google Drive root directory."""
    query = f"'{folder_id}' in parents and trashed=false"
    file_list = drive.ListFile({'q': query}).GetList()
    return file_list


def check_exif_metadata(file_id):
    """Check if the file has EXIF metadata."""
    file = drive.CreateFile({'id': file_id})
    file.FetchMetadata()
    mime_type = file['mimeType']
    
    # Create a temporary file
    with tempfile.NamedTemporaryFile(delete=False) as temp_file:
        try:
            # Download the file to a temporary location
            file.GetContentFile(temp_file.name, mimetype=mime_type)

            # Check if the file is an image
            if mime_type.startswith('image/'):
                with Image.open(temp_file.name) as img:
                    exif_data = img._getexif()
                    if exif_data:
                        print(f"EXIF metadata found for file: {file['title']}")
                        exif = {
                            ExifTags.TAGS[k]: v
                            for k, v in exif_data.items()
                            if k in ExifTags.TAGS and type(v) not in [bytes, TiffImagePlugin.IFDRational]
                        }
                        return exif
                    else:
                        #print(f"No EXIF metadata found for file: {file['title']}")
                        return None
            else:
                return None
        finally:
            # Ensure the temporary file is removed
            temp_file.close()
            os.unlink(temp_file.name)
def check_video_metadata(file_id):
    """Check for metadata in video files using ffprobe."""
    file = drive.CreateFile({'id': file_id})
    file.FetchMetadata()

    # Create a temporary file
    with tempfile.NamedTemporaryFile(delete=False) as temp_file:
        try:
            # Download the file to a temporary location
            file.GetContentFile(temp_file.name)

            # Run ffprobe command to extract metadata
            cmd = [
                'ffprobe', '-v', 'error', '-show_entries',
                'format_tags', '-print_format', 'json', temp_file.name
            ]
            result = subprocess.run(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
            metadata = json.loads(result.stdout)
            
            # Check if there are any tags/metadata
            return metadata.get('format', {}).get('tags')
        except Exception as e:
            print(f"Error processing video file {temp_file.name}: {e}")
            return None
        finally:
            # Ensure the temporary file is removed
            temp_file.close()
            os.unlink(temp_file.name)

def convert_value(value):
    """Convert individual EXIF value to a JSON-serializable format."""
    if isinstance(value, IFDRational):
        return float(value)  # Convert IFDRational to float
    elif isinstance(value, bytes):
        return value.decode(errors="ignore")  # Decode bytes to string
    elif isinstance(value, tuple):
        return tuple(convert_value(v) for v in value)  # Recursively convert tuple elements
    elif isinstance(value, list):
        return [convert_value(v) for v in value]  # Recursively convert list elements
    elif isinstance(value, dict):
        return {k: convert_value(v) for k, v in value.items()}  # Recursively convert dict elements
    else:
        return value

def convert_exif_to_serializable(exif):
    """Recursively convert EXIF metadata to a JSON-serializable format."""
    return convert_value(exif)
# %%
# %%
# List files and check for EXIF metadata
folders =  ["PUT_YOUR_FOLDER_ID_HERE"]
with open("metdata_attack_on_hindu.csv","w") as f:
    writer = csv.writer(f)
    writer.writerow(["name","parent_dir_name","mime_type","gdrive_id","parent_gdrive_id","metadata"])
    while folders:
        current_folder = folders.pop()
        fold = drive.CreateFile({'id': current_folder})
        fold.FetchMetadata()
        current_folder_name = fold['title']
        print(f"{current_folder_name=}")
        files = list_drive_files(current_folder)
        print(f"There are total {len(files)} file or folder in this directory")
        for file in files:
            print(file["title"])
            file.FetchMetadata()
            mime_type = file['mimeType']
            print(mime_type)
            if "folder" in mime_type:
                print(f"{file['title']} is a folder")
                folders.append(file["id"])
                exif=None
            elif mime_type.startswith("image/"):
                print(file["id"])
                print("this file is an image")
                # if "imageMediaMetadata" in file:
                    # print(file["imageMediaMetadata"])
                    # if "location" in file["imageMediaMetadata"]:
                    #     print("location found in image")
                exif = check_exif_metadata(file["id"])
                pprint(exif)
            elif mime_type.startswith("video/"):
                print(file["id"])
                print("this file is a video")
                exif = check_video_metadata(file["id"])
                pprint(exif)
            else:
                print(file["id"])
                print("This file is neither image nor a video")
                exif = None
            writer.writerow([file["title"],fold["title"],mime_type,file["id"],fold["id"],json.dumps(convert_value(exif))])
```

