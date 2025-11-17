# 9103major_project_mwei0358
# Time-Based Mushroom Scene

## Overview

This sketch animates our group’s mushroom landscape using **time** as the only driver.  
The static mushroom composition comes from our shared group code, and my individual contribution focuses on:

- A glowing, electric-style background made from a triangle network  
- A large central mushroom that gently **bobs** and **tilts**  
- Several smaller mushrooms that subtly **float up and down** with different timing  
- Soft **flickering dots and patterns** across caps, stems, and bases

No mouse, keyboard, or audio input is required – the whole scene is driven by the passage of time.


## How to Interact with the Work

- Open the sketch in a browser that supports p5.js (e.g. by running `index.html` with a local server or the p5.js editor).  
- Wait a moment after the page loads – the animation starts automatically and runs continuously.  
- Resize the browser window if you like:  
  - The background network and mushrooms will rescale to stay centred.  
- For the best experience, view the sketch in a larger window so you can see the entire mushroom scene.

There are **no controls** to press; the idea is to watch the scene slowly “breathe” and sparkle over time.



## Animation Driver: Time

For my individual code I chose **time** as the driver.

- The global variable `globalTime` is updated each frame using:

  ```js
  globalTime = millis() / 1000.0;


All key motions (bobbing, tilting, flickering, and background pulses) are based on `globalTime` together with `sin()`, `cos()`, and `noise()`.

There is no interaction, audio, or Perlin-noise–driven movement controlling the mushrooms directly – everything is synchronised to time and phase offsets.

### How My Version Differs from Other Group Members

Within our group, each member focuses on different animated properties:

- One group member lets the user **drag mushrooms around the screen**, and also trigger them to **fall to the bottom** or **float to the top**.
- Another member uses **audio frequency** to drive the **size of the mushroom**, the **scale of the patterns**, and the **colour of the background triangles**.
- A third member adds **strong pattern animation** on all mushrooms, and when a mushroom is **clicked**, it starts **pulsing bigger and smaller**; clicking again stops the pulse.

In contrast, **my version is fully time-based and non-interactive**:

- There is **no mouse or audio control**: the scene animates by itself using `globalTime`.
- I focus on **continuous motion** (bobbing and tilting), subtle **flickering dots**, and **electric pulses** along the background triangle network.
- The **overall composition and colours stay stable**, while the feeling of life comes from smooth, rhythmic movement and shared flicker rather than drastic changes in size, colour, or click events.

This makes my contribution distinct from the others while still using the same shared mushroom artwork.

## My Individual Animation Approach

### 1. Electric Triangle Background

* The background is built once in `buildBackground()` as a web of small triangles, and each triangle edge is stored in `bgLines`.
* In each frame, `drawBackgroundElectric(t)`:

  * Loops over all triangle edges
  * Uses `noise()` plus time to decide which edges are “active”
  * Lights up short segments that pulse along each edge, like energy travelling through a network
  * Varies stroke width and brightness so the lines feel like glowing sparks

Although the background geometry comes from the group code, the animated electric effect is my contribution and is completely time-based.

### 2. Main Mushroom Motion (Big Centre Mushroom)

In the `draw()` function, I animate the largest mushroom using simple trigonometric functions of time:

```js
const t = globalTime;
const mainBob  = 35 * sin(t * 0.55);                // Vertical bobbing
const mainTilt = radians(-7) + 0.04 * sin(t * 0.4); // Gentle rotation
```

Then I apply a sequence of transforms:

```js
translate(DESIGN_W * 0.35, DESIGN_H * 0.75 + mainBob);
rotate(mainTilt);
scale(0.7);
drawStemUniform();
drawCapReplica(0, -650, 880, 360);
```

* `drawStemUniform()` draws a tall stem filled with animated red dots.
* `drawCapReplica()` draws the large yellow-and-red cap with arc-shaped geometry and ring-like dot patterns.

The result is that the main mushroom breathes and sways, always driven by time.

### 3. Small Mushrooms Motion (Group Layout)

Each smaller mushroom is created from the shared group `SCENE_LAYOUT` using:

```js
const m = makeMushroomFromLayout(layout);
```

Inside `draw()`, I add an extra time-based vertical motion for each instance:

```js
const smallAmp  = 40;
const smallFreq = 2;

for (let i = 0; i < mushrooms.length; i++) {
  const m = mushrooms[i];
  const bob = smallAmp * sin(t * smallFreq + i * 0.7); // different phase per mushroom

  push();
  translate(0, bob);
  m.draw();
  pop();
}
```

* All mushrooms bob up and down, but each has a phase offset based on its index.
* This creates a more organic scene: the mushrooms never move in perfect sync, which avoids a mechanical feel.

Inside each `Mushroom` object, there is also an `anim` configuration, and `Mushroom.draw()` applies a smaller extra bob and sway using the mushroom’s `seed`. This compound motion (scene-level bob + per-mushroom wobble) makes each instance feel slightly different.

## Animated Properties and How They Differ

In my version, these properties of the image are animated:

* **Position**

  * The large mushroom moves vertically (bobbing) and is placed on a smooth time-based curve.
  * Smaller mushrooms float up and down with different phases.
* **Rotation**

  * The main mushroom has a slight rotational sway over time, giving it a “leaning in the wind” feeling.
* **Brightness and Flicker**

  * Many dots on the cap, stem, and base slightly flicker in brightness and hue over time.
  * The background edges light up as short pulses that travel along the triangle network.
* **Background Activity**

  * The background is not static: it continuously emits time-based sparks along edges.

Compared with other possible group approaches (such as changing colours more aggressively, resizing components, or revealing/hiding shapes), my focus is on continuous motion and subtle flicker:

* The composition itself stays recognisable.
* The feeling of life comes from movement over time rather than drastic geometric or colour changes.

## Key Helper Function: `pulsingColor`

One important helper function I use is:

```js
function pulsingColor(baseCol, x, y, speed = 0.6, intensity = 0.8) {
  const t = globalTime || 0;
  const n = noise(x * 0.03, y * 0.03, t * speed);

  const h0 = hue(baseCol);
  const s0 = saturation(baseCol);
  const b0 = brightness(baseCol);
  const a0 = alpha(baseCol);

  const hueRange = 120;
  const h = (h0 + (n - 0.5) * hueRange + 360) % 360;
  const b = constrain(lerp(b0 * 0.8, 100, n * intensity), 0, 100);
  const a = a0;

  return color(h, s0, b, a);
}
```

Technically, this function:

* Takes a base colour and a position `(x, y)`.
* Samples `noise()` using both position and `globalTime` to get a pseudo-random value `n` in `[0, 1]`.
* Slightly shifts the hue within a limited range (`hueRange`).
* Blends brightness between a darker version of the base and a brighter highlight, scaled by `intensity`.
* Returns a new colour that is similar to the original but gently flickers over time.

This helper is reused in several places:

* Cap dot patterns
* Stem dot patterns
* Base patterns
* Some of the large stem dots

Because the same function is used across different parts of the image, the whole scene shares a consistent, soft flickering style.

## Inspiration References

My animation approach is inspired by the Week 9 tutorial examples, especially:

### Part 3: “Chase the dot”

This example shows how to make one dot move, and then make a second dot orbit around it using `sin()` and `cos()`.

From this, I borrowed the idea of using time + trigonometry to drive motion and create orbits and offsets.

In my sketch, I use similar ideas to:

* Move the main mushroom along a smooth sine curve (bobbing).
* Offset each small mushroom’s motion by a phase (`+ i * 0.7`) so they are related, but not identical.

### Part 4 & 5: Bauhaus Head (custom shape with `push()`, `pop()`, `translate()`, `rotate()`, `scale()`)

These parts demonstrate how to transform a custom shape (built from multiple primitives) as a single unit, and how to manage multiple instances.

They influenced how I:

* Treat each mushroom (cap + stem + base) as one unit, transformed with `translate()`, `rotate()`, and `scale()`.
* Use a class (`Mushroom`) and a layout array (`SCENE_LAYOUT`) to manage many instances across the canvas.

Together, these tutorial examples encouraged me to think of the mushroom scene as a set of custom shapes that can be moved, rotated, and animated over time in a structured way.

## Technical Summary

* **Language / Library:** JavaScript with p5.js.
* **Core idea:** A static mushroom artwork from our group code is brought to life using time-based transformations and flickering colours.

**Key technical points:**

* `globalTime = millis() / 1000.0` is the main clock for animation.
* The background is a precomputed triangle mesh (`bgLines`) that is re-drawn each frame with short, noisy pulses (`drawBackgroundElectric(t)`).
* The main mushroom is animated with time-driven translation and rotation before `drawStemUniform()` and `drawCapReplica()` are called.
* Small mushrooms from `SCENE_LAYOUT` are moved with a vertical sine wave plus additional per-instance wobble inside the `Mushroom` class.
* The helper `pulsingColor()` adds subtle flicker to many dots, making the scene feel alive without breaking the original colour palette.

I did not use any external libraries beyond p5.js and the utilities provided in the course and group code. All additional animation logic is implemented directly in my own JavaScript.
