# crace

crace is a browser-based multiplayer car racing game written in 2021 with p5.js and p5.play. Up to four players open the same page, enter a name, and hold the up arrow to drive their car along a vertical track. Player positions are shared through a Firebase Realtime Database so every browser sees the same race. The repository has no build step; it is a static HTML page plus a few JavaScript files.

## Requirements

- A modern desktop browser with JavaScript enabled.
- A local static file server (for example Python 3 or Node). Opening `index.html` directly with `file://` can block the image loads.
- Internet access at runtime: p5.js 0.9.0 and the Firebase 8.4.3 SDK are loaded from CDNs, and the game state is stored in a Firebase Realtime Database.
- A Firebase project of your own if the database referenced in `index.html` is no longer available. The config block in `index.html` points at a project named `database-car-d4c04`.

## Setup

```bash
git clone https://github.com/samarthpaliwal/crace.git
cd crace
```

If you want to use your own Firebase project, create a Realtime Database and replace the `firebaseConfig` object in `index.html` with your project's values. The database needs these keys at the root: `gameState` (number), `playerCount` (number), `players` (object), and `CarsAtEnd` (number). Setting `gameState` to `0` and `playerCount` to `0` resets the lobby.

Note on image paths: `sketch.js` loads images from `../images/...`, which is one directory above `index.html`. When the repository root is served directly those URLs do not resolve. Either change the paths in `sketch.js` to `images/...` or serve the parent directory and open `crace/index.html`.

## Usage

Serve the folder and open it in a browser:

```bash
python3 -m http.server 8000
# then open http://localhost:8000/
```

Controls:

- Type a name in the text box and click `Play` to join. The lobby waits until `playerCount` reaches 4.
- Hold the `Up Arrow` to move your car forward (10 units per frame).
- Your own car is marked with a red circle and the camera follows it.
- The race ends for a player when their distance passes 5200; the finishing order is written to `CarsAtEnd`.
- Click `Reset` (top right) to set `playerCount` and `gameState` back to 0 for everyone.

## How it works

`sketch.js` creates a full-window canvas, connects to Firebase, and runs a small state machine driven by the `gameState` value in the database: `0` shows the join form, `1` runs the race, `2` ends it. `js/Form.js` builds the name input and the `Play` and `Reset` buttons with p5.dom and increments `playerCount` when someone joins. `js/Player.js` wraps each player's record (`players/playerN` with `name` and `distance`) and listens for changes to all players. `js/Game.js` creates four car sprites with p5.play, positions each one from the shared `distance` values on every frame, scrolls the track image, and moves the local player's car while the up arrow is held. Once four players have joined the game writes `gameState = 1` and the race starts.

## Project layout

```
index.html        page, CDN scripts, Firebase config
sketch.js         preload, setup, draw, game state switch
js/Game.js        race loop, car sprites, camera
js/Player.js      per-player Firebase reads and writes
js/Form.js        join form and reset button
p5.play.js        vendored p5.play library
images/           track and four car sprites
style.css         canvas border
```
