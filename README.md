# Blender Rendering with Google Colab

Render Blender projects using Google Colab and save the output directly to Google Drive.

## Requirements

- Google Account
- Google Drive
- Blender project (`.blend` file)

---

## 1. Download Blender

```python
!wget https://download.blender.org/release/Blender4.3/blender-4.3.0-linux-x64.tar.xz
```

---

## 2. Mount Google Drive

```python
from google.colab import drive
drive.mount('/content/drive')
```

This allows Colab to access your `.blend` files and store rendered images directly in Google Drive.

---

## 3. Extract Blender

```python
!tar xf blender-4.3.0-linux-x64.tar.xz
```

---

## 4. Specify Your Blender Project

```python
filename = '/content/drive/MyDrive/YourFolder/YourFile.blend'
```

Example:

```python
filename = '/content/drive/MyDrive/Blender/MyScene.blend'
```

---

## 5. Start Rendering

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
| `-E 'CYCLES'` | Use Cycles render engine |
| `-o` | Output location |
| `-s` | Start frame |
| `-e` | End frame |
| `-a` | Render animation |
| `-F 'PNG'` | Output image format |
| `--cycles-device OPTIX` | Use GPU acceleration |

---

## Example

Input file:

```python
filename = '/content/drive/MyDrive/Blender/Animation.blend'
```

Output directory:

```python
'/content/drive/MyDrive/RenderedFrames/Animation_'
```

Frames will be saved as:

```
Animation_0001.png
Animation_0002.png
Animation_0003.png
...
```

---

## Notes

- Google Colab sessions are temporary.
- Rendering speed depends on the GPU assigned by Colab.
- Ensure enough Google Drive storage is available.
- For large projects, consider rendering in segments and combining them later.

---

## Tested With

- Blender 4.3.0
- Google Colab
- Cycles Renderer
- NVIDIA GPU (OPTIX)

---

## License

MIT License
