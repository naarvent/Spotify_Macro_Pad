# Spotify Macro Pad
A simple Macro Pad for Spotify Desktop coded in Python

IMPORTANT: YOU HAVE TO START A TRACK REPRODUCTION IN YOUR SPOTIFY DESKTOP APP, IF YOU ONLY OPEN IT, IT ISN'T GONNA WORK, YOU HAVE TO START A TRACK AND LATER THE MACRO PAD WILL WORK.


If you want to change the keys, you have to change the name in "def on_press". 
If you didn't change anything it works with ↑, ↓, ←, →, PgDown, PgUp. 

    ↑ to increase the volume in 20% 
    ↓ to decrease the volume in 20% 
    ← to play the previous track 
    → to play a random track 
    PgDown to stop and play the track 
    PgUp to activate the keys that will affect the macro pad (↑, ↓, ←, →, PgDown)
    Del to randomize the track choice
    
If you didn't press the PgUp key, the Macro Pad isn't gonna work, you have to press it, to keep using the keys that affect the Macro Pad without making any action in him you have to press it again.

To work you have to install:

    pynput
    spotipy
    request (optional)
  
To install them you have to execute:

      pip install pynput
      pip install spotipy
      pip install request (optional)

  
To get the Device ID you have to make this:

    from spotipy.oauth2 import SpotifyOAuth
    import spotipy
    
    # Configura tus credenciales de Spotify (reemplaza estos valores por los tuyos)
    CLIENT_ID = 'CLIENT_ID'
    CLIENT_SECRET = 'CLIENT_SECRET'
    REDIRECT_URI = 'http://localhost:8889/callback/'
    
    # Autenticación y acceso a la API
    sp = spotipy.Spotify(auth_manager=SpotifyOAuth(client_id=CLIENT_ID,
                                                   client_secret=CLIENT_SECRET,
                                                   redirect_uri=REDIRECT_URI,
                                                   scope="user-library-read user-read-playback-state user-modify-playback-state"))
    
    
    devices = sp.devices()
    
    for device in devices['devices']:
        print(f"Device Name: {device['name']}, Device ID: {device['id']}")


Also (this is optional) if you want to execute this with Windows when it starts, you have to convert it in to an .exe, to make this you have to install pyinstaller, so put into the console:

    pip install pyinstaller

Later, you have to navigate to the file directory using:

    cd "File_Directory"

Finally, to convert the .py into an .exe, you have to put:

    pyinstaller --onefile "Name_Of_The_File.py"

And to execute him with Windows press Win+R and type:

    shell:startup

Make a direct access (more recomendated) or put the .exe into the folder.


This Macro Pad is free to use, it is this simple code:

    from pynput import keyboard
    from spotipy.oauth2 import SpotifyOAuth
    import spotipy
    from datetime import datetime
    import random
    
    # Configuración de las credenciales de Spotify
    SPOTIPY_CLIENT_ID = 'CLIENT_ID'
    SPOTIPY_CLIENT_SECRET = 'CLIENT_SECRET'
    SPOTIPY_REDIRECT_URI = 'http://localhost:8889/callback/'  # Debe coincidir con la URI registrada en Spotify
    PLAYLIST_ID = 'PLAYLIST_ID'
    
    # Archivo de registro
    log_file = "macro_pad_log.txt"
    
    # Configurar Spotify
    sp = spotipy.Spotify(auth_manager=SpotifyOAuth(
        client_id=SPOTIPY_CLIENT_ID,
        client_secret=SPOTIPY_CLIENT_SECRET,
        redirect_uri=SPOTIPY_REDIRECT_URI,
        scope="user-read-recently-played user-modify-playback-state user-read-playback-state playlist-read-private"
    ))
    
    # Estado del macro pad (activo/inactivo) y aleatoriedad
    is_active = False
    is_random = False  # Estado de aleatoriedad
    current_index = 0  # Índice de la canción actual dentro de la playlist
    
    # Función para escribir en el log
    def write_to_log(entry):
        """Escribe una entrada en el archivo de registro."""
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        with open(log_file, "a", encoding="utf-8") as f:
            f.write(f"{timestamp} - {entry}\n")
    
    # Obtener las canciones de la playlist
    def get_playlist_tracks():
        """Obtiene las canciones de una playlist específica."""
        try:
            results = sp.playlist_items(PLAYLIST_ID, fields='items.track.uri', additional_types=['track'])
            tracks = [item['track']['uri'] for item in results['items']]
            write_to_log(f"Playlist cargada con {len(tracks)} canciones.")
            return tracks
        except Exception as e:
            write_to_log(f"Error al obtener las canciones de la playlist: {e}")
            return []
    
    playlist_tracks = get_playlist_tracks()
    
    # Funciones de reproducción
    def toggle_playback():
        """Pausa o reanuda la reproducción sin cambiar la canción actual."""
        try:
            playback_info = sp.current_playback()
            if playback_info and playback_info.get('is_playing', False):
                sp.pause_playback()
                write_to_log("Reproducción pausada.")
            else:
                sp.transfer_playback(device_id='DEVICE_ID', force_play=True)
                write_to_log("Reproducción reanudada.")
        except Exception as e:
            write_to_log(f"Error al pausar/reanudar la reproducción: {e}")
    
    def play_next_track():
        """Avanza a la siguiente canción en la playlist o elige una aleatoria si is_random es True."""
        global current_index
        try:
            if playlist_tracks:
                if is_random:
                    current_index = random.randint(0, len(playlist_tracks) - 1)
                else:
                    current_index = (current_index + 1) % len(playlist_tracks)
                sp.start_playback(context_uri=f'spotify:playlist:{PLAYLIST_ID}', offset={"position": current_index})
                write_to_log(f"Cambiando a la canción: índice {current_index}")
            else:
                write_to_log("La playlist está vacía.")
        except Exception as e:
            write_to_log(f"Error al cambiar a la siguiente canción: {e}")
    
    def play_previous_track():
        """Retrocede a la canción anterior en la playlist."""
        global current_index
        try:
            if playlist_tracks:
                current_index = (current_index - 1) % len(playlist_tracks)
                sp.start_playback(context_uri=f'spotify:playlist:{PLAYLIST_ID}', offset={"position": current_index})
                write_to_log(f"Cambiando a la canción anterior: índice {current_index}")
            else:
                write_to_log("La playlist está vacía.")
        except Exception as e:
            write_to_log(f"Error al retroceder a la canción anterior: {e}")
    
    # Funciones de volumen
    def set_volume(change):
        """Ajusta el volumen en función del cambio especificado."""
        try:
            playback_info = sp.current_playback()
            if playback_info and playback_info.get('device', {}).get('volume_percent') is not None:
                current_volume = playback_info['device']['volume_percent']
                new_volume = max(0, min(100, current_volume + change))
                sp.volume(new_volume)
                write_to_log(f"Volumen ajustado a {new_volume}%")
            else:
                write_to_log("No se pudo obtener el volumen actual.")
        except Exception as e:
            write_to_log(f"Error al ajustar el volumen: {e}")
    
    # Manejadores de teclado
    def on_press(key):
        """Maneja las teclas presionadas."""
        global is_active, is_random
        try:
            if key == keyboard.Key.page_up:  # Activar/desactivar el Macro Pad
                is_active = not is_active
                status = "activado" if is_active else "desactivado"
                write_to_log(f"Macro Pad {status}")
            elif not is_active:
                return
            elif key == keyboard.Key.page_down:  # Pausar/reproducir
                toggle_playback()
            elif key == keyboard.Key.left:  # Canción anterior
                play_previous_track()
            elif key == keyboard.Key.right:  # Canción siguiente
                play_next_track()
            elif key == keyboard.Key.up:  # Subir volumen
                set_volume(20)
            elif key == keyboard.Key.down:  # Bajar volumen
                set_volume(-20)
            elif key == keyboard.Key.delete:  # Cambiar aleatoriedad
                is_random = not is_random
                status = "activada" if is_random else "desactivada"
                write_to_log(f"Aleatoriedad {status}")
        except Exception as e:
            write_to_log(f"Error en la tecla {key}: {e}")
    
    def start_macro_pad():
        """Inicia el Macro Pad."""
        print("Macro Pad iniciado. Usa 'PgUp' para activarlo/desactivarlo.")
        with keyboard.Listener(on_press=on_press) as listener:
            listener.join()
    
    if __name__ == "__main__":
        write_to_log("Macro Pad iniciado")
        start_macro_pad()

