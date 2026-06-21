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
!./blender-4.3.0-linux-x64/blender -b $filename -noaudio -E 'CYCLES' -o '/content/drive/MyDrive/YourOutputFolder/YourProjectFileName_'  -s 1 -e 720 -a -F 'PNG' -- --cycles-device OPTIX
```

**Argument reference:**

| Argument | Description |
|---|---|
| `-b $filename` | Runs Blender in background mode (no GUI) with the specified file |
| `-noaudio` | Disables audio (not needed for rendering) |
| `-E 'CYCLES'` | Uses the Cycles render engine |
| `-o '...'` | Output path and filename prefix for the rendered files |
| `-s 1` | Start frame |
| `-e 720` | End frame |
| `-a` | Renders the full animation range (from start to end frame) |
| `-F 'PNG'` | Output image format |
| `--cycles-device OPTIX` | Uses an NVIDIA GPU with OptiX for accelerated rendering (make sure the Colab runtime is set to GPU) |

Note: Before running the render, make sure the Colab runtime is set to GPU via `Runtime > Change runtime type > GPU`.

---

## Handling Folder or File Names with Spaces

If your folder or file name contains spaces (for example: `Output for your blender name`), the path must be wrapped in quotes to avoid errors. This applies to both Python variables and command line arguments.

**Example path with spaces:**

```python
filename = '/content/drive/MyDrive/Project Files/My Cool Scene.blend'
```

```python
!./blender-4.3.0-linux-x64/blender -b "$filename" -noaudio -E 'CYCLES' -o '/content/drive/MyDrive/Output for your blender name/Result_'  -s 1 -e 720 -a -F 'PNG' -- --cycles-device OPTIX
```

Two important points here:
- The `$filename` variable is wrapped in double quotes (`"$filename"`) so the shell does not split the path into multiple separate arguments.
- The output path (`-o`) is still wrapped in single quotes as usual.

Without quotes, the shell treats each space as a new argument separator, which causes errors such as `No such file or directory`.

---

## Troubleshooting

- **Error `CUDA/OptiX device not found`**: Make sure the Colab runtime is set to use a GPU (`Runtime > Change runtime type > GPU`).
- **Error `No such file or directory`**: Double check the `.blend` file path and make sure it is wrapped in quotes if it contains spaces.
- **Render interrupted due to Colab session timeout**: Use a smaller frame range per session (adjust the `-s` and `-e` values), or consider Colab Pro for longer runtimes.
- **Output files not appearing in Drive**: Make sure the destination folder in the `-o` path already exists in Google Drive (Blender does not automatically create new folders).

---

## License

MIT License
