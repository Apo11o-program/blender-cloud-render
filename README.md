# Blender Rendering with Google Colab

Render Blender projects using Google Colab and save the output directly to Google Drive.

## Requirements

- Google Account
- Google Drive
- Blender project (`.blend` file)

---

## Usage

### 1. Download Blender

Download Blender (version 4.3.0) directly into the Colab runtime:

```python
!wget https://download.blender.org/release/Blender4.3/blender-4.3.0-linux-x64.tar.xz
```

### 2. Mount Google Drive

Connect Colab to Google Drive so the `.blend` file and render output can be accessed:

```python
from google.colab import drive
drive.mount('/content/drive')
```

### 3. Extract Blender

```python
!tar xf blender-4.3.0-linux-x64.tar.xz
```

### 4. Set the File Path

Replace the path below with the location of your `.blend` file in Google Drive:

```python
filename = '/content/drive/MyDrive/YourFolder/YourFile.blend'
```

### 5. Run the Render

```python
!./blender-4.3.0-linux-x64/blender -b $filename -noaudio -E 'CYCLES' -o '/content/drive/MyDrive/YourOutputFolder/YourProjectFileName_' -F 'PNG' -s 1 -e 720 -a -- --cycles-device OPTIX
```

**Argument reference:**

| Argument | Description |
|---|---|
| `-b $filename` | Runs Blender in background mode (no GUI) with the specified file |
| `-noaudio` | Disables audio (not needed for rendering) |
| `-E 'CYCLES'` | Uses the Cycles render engine |
| `-o '...'` | Output path and filename prefix for the rendered files |
| `-F 'PNG'` | Output image format |
| `-s 1` | Start frame |
| `-e 720` | End frame |
| `-a` | Renders the full animation range (from start to end frame) |
| `--cycles-device OPTIX` | Uses an NVIDIA GPU with OptiX for accelerated rendering (make sure the Colab runtime is set to GPU) |

Note: Before running the render, make sure the Colab runtime is set to GPU via `Runtime > Change runtime type > GPU`.

> **Important on argument order:** Blender executes command line arguments in the order they are written, and `-a` (or `-f`) must always be the last argument. If `-F` (output format) is placed *after* `-a`, the render will already have started using whatever format is saved inside the `.blend` file, and the `-F` flag will have no effect. Always set `-o`, `-F`, `-s`, and `-e` *before* `-a`, as shown above.

---

## Handling Folder or File Names with Spaces

If your folder or file name contains spaces (for example: `Output for your blender name`), the path must be wrapped in quotes to avoid errors. This applies to both Python variables and command line arguments.

**Example path with spaces:**

```python
filename = '/content/drive/MyDrive/Project Files/My Cool Scene.blend'
```

```python
!./blender-4.3.0-linux-x64/blender -b "$filename" -noaudio -E 'CYCLES' -o '/content/drive/MyDrive/Output for your blender name/Result_' -F 'PNG' -s 1 -e 720 -a -- --cycles-device OPTIX
```

Two important points here:
- The `$filename` variable is wrapped in double quotes (`"$filename"`) so the shell does not split the path into multiple separate arguments.
- The output path (`-o`) is still wrapped in single quotes as usual.

Without quotes, the shell treats each space as a new argument separator, which causes errors such as `No such file or directory`.

---

## Rendering to Video Formats (MP4 / MKV) Instead of PNG

The `-F` flag only accepts an image/container *category* (`PNG`, `JPEG`, `OPEN_EXR`, `FFMPEG`, etc.). It does **not** let you pick a specific video container (MP4, MKV) or codec (H.264, FFV1) directly as a command line value — those settings live inside the `.blend` file's Output Properties and must be set there, or overridden at render time with a small Python expression via `--python-expr`.

### Option A (Recommended): Render a PNG Sequence, Then Encode Separately

This is the safer approach for Colab specifically. If your Colab session disconnects or times out mid-render, a PNG sequence keeps every frame that finished rendering. If you render directly to a single MP4/MKV file and the session drops, the entire video file is often corrupted or incomplete, so you lose everything.

```python
!./blender-4.3.0-linux-x64/blender -b $filename -noaudio -E 'CYCLES' -o '/content/drive/MyDrive/YourOutputFolder/YourProjectFileName_' -F 'PNG' -s 1 -e 720 -a -- --cycles-device OPTIX
```

Once all frames are rendered, encode them into MP4 or MKV with `ffmpeg` (already available on Colab by default):

```python
# MP4 (H.264) - most widely compatible
!ffmpeg -framerate 24 -i '/content/drive/MyDrive/YourOutputFolder/YourProjectFileName_%04d.png' -c:v libx264 -pix_fmt yuv420p '/content/drive/MyDrive/YourOutputFolder/output.mp4'

# MKV (FFV1, lossless) - larger file, no quality loss
!ffmpeg -framerate 24 -i '/content/drive/MyDrive/YourOutputFolder/YourProjectFileName_%04d.png' -c:v ffv1 '/content/drive/MyDrive/YourOutputFolder/output.mkv'
```

Adjust `-framerate` to match your project's FPS, and `%04d` to match the frame number padding in your output filenames.

### Option B: Render Directly to Video

If you prefer Blender to output the video file directly, set the container and codec with `--python-expr` before triggering the render, then pass `-F FFMPEG`:

```python
# Direct MP4 output (H.264)
!./blender-4.3.0-linux-x64/blender -b $filename -noaudio -E 'CYCLES' --python-expr "import bpy; bpy.context.scene.render.ffmpeg.format = 'MPEG4'; bpy.context.scene.render.ffmpeg.codec = 'H264'" -o '/content/drive/MyDrive/YourOutputFolder/output' -F 'FFMPEG' -s 1 -e 720 -a -- --cycles-device OPTIX

# Direct MKV output (H.264 inside an MKV container)
!./blender-4.3.0-linux-x64/blender -b $filename -noaudio -E 'CYCLES' --python-expr "import bpy; bpy.context.scene.render.ffmpeg.format = 'MKV'; bpy.context.scene.render.ffmpeg.codec = 'H264'" -o '/content/drive/MyDrive/YourOutputFolder/output' -F 'FFMPEG' -s 1 -e 720 -a -- --cycles-device OPTIX
```

Notes:
- `--python-expr` must come before `-o`/`-F`/`-a` so the codec/container are set before rendering starts.
- Valid `ffmpeg.format` values include `MPEG4` (MP4 container), `MKV`, `WEBM`, `AVI`, `QUICKTIME`, `OGG`.
- Valid `ffmpeg.codec` values include `H264`, `H265`, `FFV1` (lossless), `HUFFYUV` (lossless), `MPEG4`, `WEBM`.
- If the render is interrupted partway through, the resulting video file will likely be unusable, since this approach writes one continuous file instead of separate frames. This is why Option A is generally recommended for long Colab renders.

---

- **Error `CUDA/OptiX device not found`**: Make sure the Colab runtime is set to use a GPU (`Runtime > Change runtime type > GPU`). Note that Blender 4.3 requires NVIDIA driver version 495.89 or newer for OptiX; this is rarely an issue on Colab since its runtimes ship with recent drivers, but it is worth knowing if you encounter device errors on other systems.
- **Error `No such file or directory`**: Double check the `.blend` file path and make sure it is wrapped in quotes if it contains spaces.
- **Render interrupted due to Colab session timeout**: Use a smaller frame range per session (adjust the `-s` and `-e` values), or consider Colab Pro for longer runtimes.
- **Output files not appearing in Drive**: Make sure the destination folder in the `-o` path already exists in Google Drive (Blender does not automatically create new folders).

---

## License

MIT License
