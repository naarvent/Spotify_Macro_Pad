# Spotify Macro Pad

A simple yet powerful Macro Pad for controlling Spotify Desktop using customizable keyboard shortcuts.  
Built in Python with `spotipy` and `pynput`, this script allows you to control playback, volume, track selection, and randomization—all with physical keys.


## 🎯 Features

- 🔊 Volume up/down in steps of 20%
- ⏯️ Play/pause toggle
- ⏮️ Previous and ⏭️ next track navigation
- 🔀 Enable or disable random track selection from a playlist
- 🖱️ Full keyboard interaction using the following keys by default:
  - `↑` Increase volume
  - `↓` Decrease volume
  - `←` Play previous track
  - `→` Play next (or random) track
  - `PgDown` Play/pause
  - `PgUp` Toggle Macro Pad on/off
  - `Del` Toggle shuffle mode

> 💡 Press `PgUp` to enable the Macro Pad. The keys will only respond when it is activated.

🧩 This Macro Pad is optimized for 60% keyboards, since it maps keys like arrows, Page Up, Page Down, and Delete—often easily accessible on that layout.
You can modify the key bindings by editing the on_press function and changing the key names. Each key is detected using keyboard.Key, and the logic is fully customizable.


## 📦 Requirements

Install the required Python libraries:

    pip install pynput spotipy requests


## 🚀 How to Use

1. Make sure the **Spotify Desktop App** is running and **a track is actively playing**.
2. Set your own Spotify credentials in the script:
   ```python
   SPOTIPY_CLIENT_ID = 'YOUR_CLIENT_ID'
   SPOTIPY_CLIENT_SECRET = 'YOUR_CLIENT_SECRET'
   SPOTIPY_REDIRECT_URI = 'http://localhost:8889/callback/'
   PLAYLIST_ID = 'YOUR_PLAYLIST_ID'
   
3. Replace DEVICE_ID with your own playback device (see below).
4. Run the script:
   
       python macro_pad.py


## 🔧 How to Get Your Spotify Device ID

from spotipy.oauth2 import SpotifyOAuth
import spotipy

sp = spotipy.Spotify(auth_manager=SpotifyOAuth(
    client_id='YOUR_CLIENT_ID',
    client_secret='YOUR_CLIENT_SECRET',
    redirect_uri='http://localhost:8889/callback/',
    scope="user-library-read user-read-playback-state user-modify-playback-state"
))

devices = sp.devices()
for device in devices['devices']:
    print(f"Device Name: {device['name']}, Device ID: {device['id']}")


## 📝 Logging
Every action (play, pause, shuffle toggle, etc.) is written to a log file with timestamps for debugging and tracking purposes.
The log is stored in a dedicated project folder for safety (see below).


## 📁 File Structure and Safety
All generated files such as .cache (for Spotify token storage) and the log file are stored inside a safe directory:
    Documents/
    └── naarvent's projects/
        └── SpotifyMacroPad/
            ├── .cache
            └── macro_pad_log.txt
This prevents interference with other Spotify-based tools or projects you may be using. The folder is created automatically on first run.


## 🖥️ Optional: Run on Windows Startup

1. Convert the script to .exe with PyInstaller:
   
    pip install pyinstaller
    pyinstaller --onefile macro_pad.py

2. Press Win + R, type:
   
   shell:startup

3. Place a shortcut to the .exe file inside the folder that opens.


## 🛠 Customization

You can change the key bindings in the on_press function. For example:

    if key == keyboard.Key.f1:
        toggle_playback()

You can also modify the volume step size, playlist logic, shuffle behavior, or integrate with your own playlist management system.


## 📄 License
This Macro Pad is open-source and free to use. Feel free to modify or improve it. If you share your version publicly, giving credit is appreciated!


## 🧠 Credits
Developed in Python using:
    spotipy for Spotify API
    pynput for keyboard input

## Code

    import os
    from pathlib import Path
    from pynput import keyboard
    from spotipy.oauth2 import SpotifyOAuth
    import spotipy
    from datetime import datetime
    import random
    
    # --- Secure project folder setup ---
    documents_path = Path.home() / "Documents"
    project_dir = documents_path / "naarvent's projects" / "SpotifyMacroPad"
    project_dir.mkdir(parents=True, exist_ok=True)
    
    log_file = project_dir / "macro_pad_log.txt"
    cache_path = project_dir / ".cache"
    
    # --- Spotify credentials (set your own) ---
    SPOTIPY_CLIENT_ID = 'YOUR_CLIENT_ID'
    SPOTIPY_CLIENT_SECRET = 'YOUR_CLIENT_SECRET'
    SPOTIPY_REDIRECT_URI = 'http://localhost:8889/callback/'
    PLAYLIST_ID = 'YOUR_PLAYLIST_ID'
    
    # --- Initialize Spotify API ---
    sp = spotipy.Spotify(auth_manager=SpotifyOAuth(
        client_id=SPOTIPY_CLIENT_ID,
        client_secret=SPOTIPY_CLIENT_SECRET,
        redirect_uri=SPOTIPY_REDIRECT_URI,
        scope="user-read-recently-played user-modify-playback-state user-read-playback-state playlist-read-private",
        cache_path=str(cache_path)
    ))
    
    # --- Playback state ---
    is_active = False
    is_random = False
    current_index = 0
    
    def write_to_log(entry):
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        with open(log_file, "a", encoding="utf-8") as f:
            f.write(f"{timestamp} - {entry}\n")
    
    def get_playlist_tracks():
        try:
            results = sp.playlist_items(PLAYLIST_ID, fields='items.track.uri', additional_types=['track'])
            tracks = [item['track']['uri'] for item in results['items']]
            write_to_log(f"Playlist loaded with {len(tracks)} tracks.")
            return tracks
        except Exception as e:
            write_to_log(f"Error fetching playlist: {e}")
            return []
    
    playlist_tracks = get_playlist_tracks()
    
    def toggle_playback():
        try:
            playback_info = sp.current_playback()
            if playback_info and playback_info.get('is_playing', False):
                sp.pause_playback()
                write_to_log("Playback paused.")
            else:
                sp.transfer_playback(device_id='YOUR_DEVICE_ID', force_play=True)
                write_to_log("Playback resumed.")
        except Exception as e:
            write_to_log(f"Playback toggle error: {e}")
    
    def play_next_track():
        global current_index
        try:
            if playlist_tracks:
                if is_random:
                    current_index = random.randint(0, len(playlist_tracks) - 1)
                else:
                    current_index = (current_index + 1) % len(playlist_tracks)
                sp.start_playback(context_uri=f'spotify:playlist:{PLAYLIST_ID}', offset={"position": current_index})
                write_to_log(f"Playing track index {current_index}.")
            else:
                write_to_log("Playlist is empty.")
        except Exception as e:
            write_to_log(f"Next track error: {e}")
    
    def play_previous_track():
        global current_index
        try:
            if playlist_tracks:
                current_index = (current_index - 1) % len(playlist_tracks)
                sp.start_playback(context_uri=f'spotify:playlist:{PLAYLIST_ID}', offset={"position": current_index})
                write_to_log(f"Playing previous track index {current_index}.")
            else:
                write_to_log("Playlist is empty.")
        except Exception as e:
            write_to_log(f"Previous track error: {e}")
    
    def set_volume(change):
        try:
            playback_info = sp.current_playback()
            if playback_info and playback_info.get('device', {}).get('volume_percent') is not None:
                current_volume = playback_info['device']['volume_percent']
                new_volume = max(0, min(100, current_volume + change))
                sp.volume(new_volume)
                write_to_log(f"Volume set to {new_volume}%")
            else:
                write_to_log("Volume info not available.")
        except Exception as e:
            write_to_log(f"Volume adjustment error: {e}")
    
    def on_press(key):
        global is_active, is_random
        try:
            if key == keyboard.Key.page_up:
                is_active = not is_active
                status = "activated" if is_active else "deactivated"
                write_to_log(f"Macro Pad {status}")
            elif not is_active:
                return
            elif key == keyboard.Key.page_down:
                toggle_playback()
            elif key == keyboard.Key.left:
                play_previous_track()
            elif key == keyboard.Key.right:
                play_next_track()
            elif key == keyboard.Key.up:
                set_volume(20)
            elif key == keyboard.Key.down:
                set_volume(-20)
            elif key == keyboard.Key.delete:
                is_random = not is_random
                status = "enabled" if is_random else "disabled"
                write_to_log(f"Shuffle mode {status}")
        except Exception as e:
            write_to_log(f"Key press error: {e}")
    
    def start_macro_pad():
        print("Macro Pad started. Press 'PgUp' to toggle activation.")
        with keyboard.Listener(on_press=on_press) as listener:
            listener.join()
    
    if __name__ == "__main__":
        write_to_log("Macro Pad initialized.")
        start_macro_pad()
