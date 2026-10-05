# WebAR using MindAR

Browser-based augmented reality built with [MindAR](https://github.com/hiukim/mind-ar-js) and [A-Frame](https://aframe.io). Point your camera at the target image and a 3D model appears on top of it. Rotate and zoom the model with gestures, and tap the info points to read about each part.

No app to install and no build step: the whole project is a single HTML file.

## Features

- **Image tracking**: detects a printed or on-screen image and anchors a 3D model to it.
- **Clickable hotspots**: tap the red points on the model to open a pop-up with a title and description.
- **Gestures**
  - Drag with one finger (or the mouse) to rotate. Left/right spins the model and up/down tilts it (up to 80°). After a quick swipe the model keeps spinning briefly, then slows down.
  - Pinch with two fingers (or scroll the mouse wheel) to zoom between 0.5× and 3×.
  - Rotation and zoom reset each time the target is detected again.
- **Camera error messages**: if the camera can't start, the page explains why (permission denied, camera in use, insecure connection) instead of staying on the loading screen.

## Try it

1. Open the page (see [Running locally](#running-locally) or [Deploying to GitHub Pages](#deploying-to-github-pages)).
2. Allow camera access.
3. Point the camera at the [target image](https://cdn.jsdelivr.net/gh/hiukim/mind-ar-js@1.2.5/examples/image-tracking/assets/card-example/card.png). You can print it or open it on another screen.

## Requirements

- A modern browser with WebGL and camera support.
- The page must be served over **HTTPS** or from **localhost**. Browsers block the camera on `file://` pages and on plain `http://` addresses such as `http://192.168.x.x`.
- An internet connection, because A-Frame, MindAR and the example assets load from CDNs.

## Running locally

Serve the folder with any static file server, for example:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000/main.html>.

In VS Code you can also right-click `main.html` and choose **Open with Live Server** (Live Server extension). Open the page in a real browser, because the preview panels inside VS Code can't access the camera.

To test on a phone you need HTTPS: deploy to GitHub Pages or use an HTTPS tunnel such as ngrok or Cloudflare Tunnel.

## Deploying to GitHub Pages

1. In the repository, go to **Settings → Pages**.
2. Under **Source**, choose **Deploy from a branch**, select `main` and `/ (root)`, then click **Save**.
3. After a minute or two the app is available at
   <https://ahmadhabb.github.io/WebAR-using-MindAR/main.html>

To serve it from the root URL instead, rename `main.html` to `index.html`.

## Customizing

All changes are made in `main.html`.

### Your own target image

1. Compile your image into a `.mind` file with the [MindAR Image Targets Compiler](https://hiukim.github.io/mind-ar-js-doc/tools/compile).
2. Add the `.mind` file to the repository and point `imageTargetSrc` on `<a-scene>` to it:
   ```html
   <a-scene mindar-image="imageTargetSrc: ./targets.mind;" ...>
   ```
3. Update the `<img id="card">` asset to your image and set the `<a-plane>` height to *image height ÷ image width* (its width is always 1).

### Your own 3D model

- Change the `src` of `<a-asset-item id="avatarModel">` to your `.gltf` or `.glb` file, then adjust the `scale` and `position` of `<a-gltf-model>`. In MindAR, 1 unit equals the width of the target image.
- Rotation and zoom happen around the entity with the `model-gesture` component. Set its `position` to the center of your model, and give the entity directly inside it the opposite values so the model stays in place.

### Hotspots

Each hotspot is one line inside the model wrapper, for example:

```html
<a-entity class="clickable" position="-0.16 0.5 0.38"
  hotspot="title: Hat; description: Write something about the hat here."></a-entity>
```

- `position` is in target units: x to the right, y up, z toward the camera. Keep z slightly in front of the model's surface so the point isn't hidden inside it.
- Don't use `;` in the title or description, because A-Frame uses it as a separator. Colons are fine.
- Add or remove lines to change the number of hotspots. Points that end up behind the model when it rotates are hidden and can't be tapped.

### Gestures

```html
<a-entity model-gesture="rotateSpeed: 0.4; maxTilt: 80; minScale: 0.5; maxScale: 3">
```

| Option | Default | Description |
|---|---|---|
| `rotateSpeed` | `0.4` | Degrees of rotation per pixel dragged |
| `maxTilt` | `80` | Maximum up/down tilt, in degrees |
| `minScale` | `0.5` | Smallest zoom level |
| `maxScale` | `3` | Largest zoom level |

A movement shorter than 10 px counts as a tap, so hotspots still open when the finger moves slightly.

### Interface language

The pop-up button and the camera error messages are in Indonesian. To translate them, edit the texts in `main.html`: the `Tutup` button and the strings in `cameraErrorMessage()`.

## Troubleshooting

**The page stays on the loading screen or shows a camera error**

- Open the page over HTTPS or localhost, in a regular browser tab.
- Allow camera access: click the icon on the left of the address bar, enable Camera, then reload.
- Close other apps or tabs that are using the camera.

**Webcam on Linux shows no video**

Some USB webcams send YUYV frames that the Linux `uvcvideo` driver flags as corrupted. Chrome drops these frames, so the video never starts. To avoid this, the page asks for 1280×720, which makes Chrome use the camera's MJPEG mode. If a camera still shows no video, start Chrome from a terminal with `--enable-logging=stderr` and look for `Dequeued v4l2 buffer contains corrupted data`.

## Credits

- [MindAR](https://github.com/hiukim/mind-ar-js) by HiuKim Yuen, for image tracking.
- [A-Frame](https://aframe.io), for the 3D scene.
- The target image and 3D model come from the card example in the MindAR repository.
