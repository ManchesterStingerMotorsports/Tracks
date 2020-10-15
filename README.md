# Tracks
Resource of example race tracks for the FS-AI event that can be accessed remotly.

### Track Object
Each track object is stored as a JSON object which stores positions of the 4 cones used in the FS-AI event (blue, yellow, orange and big) aswell as the starting position and heading direction of the vehicle.
```
{
    "blue": [[x, y]],
    "yellow": [[x, y]],
    "orange": [[x, y]],
    "big": [[x, y]],
    "car": {
        "pos": [x, y],
        "heading": Θ
    }
}
```

### Remote Access
This method allows you to load the track over the internet rather than loading it from a file - which would mean having a copy of each track stored in the file system. 
```
import json
import random
import requests


def get_track(track_name=None):
    tracks = requests.get("https://raw.githubusercontent.com/ManchesterStingerMotorsports/Tracks/main/get-api.json")
    if tracks.status_code != 200:
        raise Exception("Network Error", f"Getting the track failed with code '{tracks.status_code}'.")
    
    tracks, track_url = tracks.json(), None
    
    if track_name == "skid-pad" or track_name == "Skid Pad":
        track_url = tracks["skid-pad"][0]["url"]
    elif track_name == "acceleration" or track_name == "Acceleration":
        track_url = tracks["acceleration"][0]["url"]
    else:
        if track_name in ["", None]:
            random_track = random.choice(tracks["endurance"])
            track_name = random_track["name"]
            track_url = random_track["url"]
        else:
            for track in tracks["endurance"]:
                if track["name"] == track_name:
                    track_url = track["url"]
    
    if track_url is None:
        valid_tracks = [track["name"] for track in tracks["endurance"]] + ["Skid Pad", "Acceleration"]
        raise KeyError(f"Track '{track_name}' was not found. Please choose one of {valid_tracks}.")

    track = requests.get(track_url)
    if track.status_code != 200:
        raise Exception("Network Error", f"Getting the track failed with code '{track.status_code}'.")
        
    return {
        "name": track_name,
        "track": track.json()
    }
```

### Rendering
Rendering can easily be cone using the python library 'matplotlib'.
```
import matplotlib.pyplot as plt


def render(track):
    plt.scatter([x[0] for x in track["blue"]], [x[1] for x in track["blue"]], color=(0, 0, 1))
    plt.scatter([x[0] for x in track["yellow"]], [x[1] for x in track["yellow"]], color=(0.9, 0.9, 0))
    plt.scatter([x[0] for x in track["big"]], [x[1] for x in track["big"]])
    plt.scatter([x[0] for x in track["orange"]], [x[1] for x in track["orange"]])

    plt.scatter([track["car"]["pos"][0]], [track["car"]["pos"][1]])
    
    plt.axis("equal")
    plt.show()
    
    
render(get_track("Skid Pad")["track"])
```
