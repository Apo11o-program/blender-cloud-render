# Blender Rendering with Google Colab

Render Blender projects using Google Colab and save the output directly to Google Drive. This method allows you to render Blender scenes without using your local machine.

## Requirements

- Google Account
- Google Drive
- Blender project (`.blend` file)
- Google Colab
- Stable internet connection

---

## Download Blender

Download Blender 4.3.0 for Linux:

```python
!wget https://download.blender.org/release/Blender4.3/blender-4.3.0-linux-x64.tar.xz
```

---

## Mount Google Drive

Mount your Google Drive to access `.blend` files and save rendered images.

```python
from google.colab import drive
drive.mount('/content/drive')
```

---

## Extract Blender

Extract the downloaded archive:

```python
!tar xf blender-4.3.0-linux-x64.tar.xz
```

---

## Specify Your Project File

Replace the path below with the location of your `.blend` file in Google Drive.

```python
filename = '/content/drive/MyDrive/YourFolder/YourFile.blend'
```

Example:

```python
filename = '/content/drive/MyDrive/BlenderProjects/House.blend'
```

---

## Start Rendering

Run Blender in background mode and render the animation.

```python
!./blender-4.3.0-linux-x64/blender \
-b $filename \
-noaudio \
-E 'CYCLES' \
-o '/content/drive/MyDrive/YourOutputFolder/YourProjectFileName_' \
-s 1 \
-e 720 \
-a \
-F 'PNG' \
-- --cycles-device OPTIX
```

### Parameters

| Parameter | Description |
|------------|-------------|
| `-b` | Run Blender in background mode |
| `-noaudio` | Disable audio |
| `-E 'CYCLES'` | Use Cycles render engine |
| `-o` | Output file path |
| `-s 1` | Start frame |
| `-e 720` | End frame |
| `-a` | Render animation |
| `-F 'PNG'` | Output image format |
| `--cycles-device OPTIX` | Use NVIDIA OptiX acceleration |

---

## Example

```python
filename = '/content/drive/MyDrive/Projects/Animation.blend'

!./blender-4.3.0-linux-x64/blender \
-b $filename \
-noaudio \
-E 'CYCLES' \
-o '/content/drive/MyDrive/RenderOutput/Animation_' \
-s 1 \
-e 300 \
-a \
-F 'PNG' \
-- --cycles-device OPTIX
```

Rendered frames will be saved to:

```
MyDrive/RenderOutput/
```

---

## Notes

- Google Colab GPU availability depends on your account type and current usage limits.
- If OptiX is unavailable, you may use:

```python
--cycles-device CUDA
```

or

```python
--cycles-device CPU
```

- Ensure that your `.blend` file includes all required textures and assets.
- Rendering large projects may take several hours.

---

## License

MIT License
