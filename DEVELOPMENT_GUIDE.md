# CursedAltar — Development Guide

**Goal:** Build a VR horror experience on Quest 3S with real ESP32 hardware (ultrasonic sensor + buzzer). The ultrasonic sensor on the floor detects the player's leg proximity to a cursed altar. Placing a cursed skull on the altar fires a real buzzer hidden under the table.

**Timeline:** 3 days
**Hardware:** Available now — building with real hardware from the start.

---

## GAME STORY

You are the last paranormal investigator sent to a cursed cabin in the woods. Every investigator before you has disappeared. Your job: find the source of the curse and contain it.

You enter the cabin. It is pitch black. You are already inside — the door is behind you. The only thing you can see is a faint warm glow — a lantern sitting on a stool beside you. You pick it up with your left hand. It's your only light source. A small radius around you is now visible.

You start to explore, holding the lantern out in front of you. It reveals the cabin piece by piece — a bed in the corner, a cabinet against the wall, a table in the center with a candle on it.

Suddenly, the door behind you slams and closes on its own. A loud slam echoes through the cabin. You spin around — somebody is closing the door from the outside. It stops. You are trapped.

As the silence returns, you notice movement — a stool (Where the lantern was sitting) near the door slowly slides across the floor on its own, scraping against the wood. No one is pushing it. Something is in here with you.

You turn back, looking for answers. You notice the cabinet. You point your free hand at it and pull the trigger. It creaks open. Nothing inside but darkness. But the candle on the table flares briefly — drawing your attention.

You start walking toward the table. The candle begins reacting to your presence. The closer you walk, the more violently it flickers. Your lantern starts flickering too. Whispers grow louder in your ears.

When you are very close, the candle suddenly dies. Total darkness for two seconds. Then the candle reignites — and a cursed skull has appeared on the table.

Silence. You grab the skull with your free hand. A faint heartbeat pulses in your ears. You place the skull back on the table. A harsh, eerie tone tears through the real world from beneath the table — the buzzer fires. The room calms. The lighting slowly warms. Text appears: "The room is sealed. For now."

---

## HARDWARE

### Components

| Component | Source |
|---|---|
| ESP32 Thing Plus (SparkFun, red board) | Borrowed from lab |
| HC-SR04 Ultrasonic Sensor | Borrowed from lab |
| Passive Piezo Buzzer (small, ×2 — one spare) | Borrowed from lab |
| Breadboard | Borrowed from lab |
| Jumper wires M-M (×10) | Borrowed from lab |
| Resistors: 1K0 ohm (×2) + 2K2 ohm (×2) | Borrowed from lab |
| USB-C cable | You already own |
| USB power bank | You already own |

### Hardware Concept

**Ultrasonic Sensor — Measuring Player Leg Distance**
The HC-SR04 is placed on the **floor, under or near the table/altar**, pointing outward toward the player's starting position. It measures how close the player's **legs** are to the altar as they walk toward it.

Why legs, not hands?
- Quest 3S already tracks hand position — using a sensor for hand distance adds nothing VR can't already do
- Quest 3S **cannot** track leg position — the ultrasonic sensor provides data that is impossible to get from the headset alone
- This justifies the physical hardware's existence

**Buzzer — Hidden Under the Table**
The passive buzzer is placed **under the table**, hidden from the player. When the curse is sealed (skull placed on altar), the buzzer fires from **below** — an unexpected physical location.

Why a physical buzzer?
- It produces a non-diegetic audio stimulus from an unexpected real-world location
- The player is wearing a VR headset — they cannot see where the sound comes from
- It conflicts with the VR soundscape, creating spatial audio disorientation
- A laptop speaker across the room cannot replicate this targeted, hidden placement
- Research on "break in presence" shows real-world sensory intrusion during VR immersion amplifies emotional response (see references.md for studies)

**Bidirectional Communication**
- **Physical → VR:** Ultrasonic sensor detects legs → ESP32 sends distance to Quest → candle reacts, whispers grow, skull materializes
- **VR → Physical:** Player places skull on altar in VR → Quest sends command to ESP32 → real buzzer fires

**Network Architecture**
```
Quest 3S ←——— TCP (port 7777) ———→ ESP32 Thing Plus
   │                                    │
   └──── Both on same WiFi network ─────┘
         (home WiFi or phone hotspot)
```

---

## GAMEPLAY FLOW

**Play area:** ~2m × 2m (VR cabin interior scaled to match real-world Guardian boundary)

**Duration:** approximately 2 minutes per playthrough.

**Controller usage:**
- **Left hand:** Holds the lantern (grabbed with left grip button). This is the player's light source for the entire experience.
- **Right hand:** Free for all other interactions — ray-cast (cabinet) and grab (skull).

### Scene Layout

The player starts **inside** the cabin. The door is a wall element behind them — it leads nowhere and never opens. All furniture and objects are within the ~2m × 2m space.

```
    ┌──────────────────────────┐
    │   [Bed]       [Cabinet]  │
    │                          │
    │        [Table/Altar]     │
    │        [Candle]          │
    │                          │
    │   ↑ ULTRASONIC SENSOR ↑  │
    │   (on floor under table, │
    │    pointing toward door) │
    │                          │
    │   [Stool+Lantern]        │
    │               [Door]     │
    │                          │
    │     ★ PLAYER STARTS ★    │
    │     (facing into cabin)  │
    └──────────────────────────┘
```

### Timeline Breakdown

**Arrival & Lantern Pickup (0:00 - 0:20)**
- Player spawns inside the cabin near the door, facing toward the table
- Pitch black. Only a faint warm glow from a lantern on a stool beside them
- Low ambient drone playing in headset
- **Interaction 1 — Grab the Lantern (Left Hand):** Player picks up the lantern with the left grip button. A point light activates, illuminating ~1m radius. The rest of the cabin remains dark. The player carries the lantern in their left hand for the rest of the experience.

**Door lock (0:20 - 0:30)**
- A few seconds after picking up the lantern, when the player moves around and looks at the door, the door slams from a slightly opened position to a completely closed position. **slams and closes on its own**
- Loud slam sound plays when it's locked
- Right after slam sound ends and door gets locked, the door lock sound plays automatically.
- **Scripted event** The player has to look at the door for it to get slammed after they pickup the lantern. The door lock sound triggers automatically after the door is slammed and locked.
- The player instinctively turns around to look after they grab the lantern. Looking at the door by holding the lantern triggers the door slam event. Which means the door should be slightly opened when the game starts.
- These 2 sounds will only be played once and not in a loop.
- Purpose: establishes something supernatural is present and the player is trapped.

**Poltergeist Stool (0:30 - 0:45)**
- After the door stops rattling, the stool (which the lantern was sitting on) **slowly slides across the floor when the player gazes at it after door is locked** over ~2-3 seconds
- Low scraping sound as it moves
- **Player interaction needed.** Triggered when player gazes at the stool by holding the lantern after the door slam ends.
- The stool moves slowly and deliberately — not flying, just sliding. Like an invisible force is pushing it.
- Purpose: escalates the supernatural threat. Something is in here.

**Rocking chair interaction**
-After stool slide completes → rocking chair is armed and starts rocking immediately (player is not looking at it). Sounds play in 5s on / 5s off cycles with fade out before each pause. When player looks at it (within 120°), it rocks for 1 more second then smoothly fades to a stop over 1 second. When player fully turns away (beyond 140°) and stays away for 2 seconds, it starts rocking again. The cycle repeats forever.

**Cabinet Interaction (0:45 - 1:00)**
- The player turns back toward the cabin interior, looking for answers
- They notice the cabinet
- **Interaction 2 — Ray-cast the Cabinet (Right/Left Hand):** Player points right/left controller at cabinet and pulls trigger. It slowly falls diagonally on the table. After the fall is done, it stays fallen on the table in that position. And the camera shakes up a little bit. The candle on the table flares briefly — drawing attention toward the altar. 

**The Approach (1:00 - 1:20)**
- Player walks toward the table/altar
- **Ultrasonic sensor detects their legs approaching**
- As distance decreases:
  - **>1.5m:** Candle flickers normally, faint whispers in VR audio
  - **1.0-1.5m:** Candle brightens, whispers get louder, lantern starts flickering erratically
  - **0.5-1.0m:** Candle pulses intensely, VR lighting shifts reddish, creepy sounds escalate
  - **<0.5m:** Candle suddenly dies. Total darkness for 2 seconds. Then candle reignites — **cursed skull has appeared on the table.** All sounds peak then go silent.

  **Before implementing the ultrasonic sensor**
  - The candle lights up when the cabinet has collided with table. The timing has to be perfect. 
  - If the player keep the A button of right controller pressed, the candle light will (Particle system) increase over time and reach to maximum. Maximum will be reached within 3 seconds from the moment the player keeps the button pressed. 
  - When the candle light reaches maximum, the skull will appear on the table with a fade in effect. The fade in effect should last for 2 seconds. After 2 seconds the skull will be completely visible. 

**The Ritual (1:20 - 1:50)**
- Dead silence. Just the skull on the table.
- **Interaction 3 — Grab the Skull (Right Hand):** Player grabs the skull with the right grip button. While holding it, a faint heartbeat sound pulses in VR audio. The candle pulses with it.
- Player places the skull back on the table surface.
- **Real buzzer fires from under the table** — harsh, unexpected, from below. Simultaneously: VR screen flashes, loud curse-sealing sound in headset.

**The Seal (1:50 - 2:00)**
- Buzzer stops after 1.5 seconds
- Room lighting slowly returns to warm normal over 2 seconds
- Candle calms to a gentle flicker
- Text appears: *"The room is sealed. For now."*
- Experience ends.

### Interaction Summary

| # | Interaction | Type | Hand | Object | Scare Purpose |
|---|------------|------|------|--------|---------------|
| 1 | Pick up lantern | Grab | Left | Lantern on stool | Player's only light source — creates vulnerability |
| 2 | Open the cabinet | Ray-cast | Right | Cabinet | Misdirection, draws attention to altar |
| 3 | Grab & place skull | Grab | Right | Cursed skull | Ritual climax — triggers real buzzer |
| — | Door slams on its own | Scripted auto | — | Door | You're trapped — something is here |
| — | Stool slides across floor | Scripted auto | — | Stool | Poltergeist presence — slow, creepy |
| — | Candle blowout + skull appear | Scripted auto | — | Candle + Skull | Peak terror — darkness then reveal |

---

## STEP-BY-STEP BUILD INSTRUCTIONS

### Step 1: Wire the Circuit
**Who:** You
**Action:**

**Wiring diagram:**
```
ESP32 Thing Plus         HC-SR04              Passive Buzzer
──────────────           ───────              ──────────────
5V (USB pin) ──────────→ VCC
GND ───────────────────→ GND ──────────────→ Negative (-)
GPIO 4 ────────────────→ TRIG
GPIO 5 ←── [1K0 res] ←── ECHO
              │
          [2K2 res]
              │
             GND

GPIO 6 ────────────────────────────────────→ Positive (+)
```

**Voltage divider detail (ECHO pin protection):**
```
HC-SR04 ECHO ────┬──── 1K0 ohm resistor ────→ ESP32 GPIO 5
                 │
                 └──── 2K2 ohm resistor ────→ GND

This drops the 5V ECHO signal to ~3.3V for the ESP32.
The 2K2 works as a safe substitute for 2K0 — the ratio is close enough.
```

**Important:**
- All GND connections must be common (ESP32 GND, HC-SR04 GND, buzzer GND — all connected on the breadboard ground rail)
- Double check wiring before powering on
- The HC-SR04 must receive 5V on VCC to function correctly
- The ESP32 Thing Plus has a 5V pin (USB passthrough) — use this for HC-SR04 VCC

**Expected result:** Circuit wired on breadboard. Not powered yet.

---

### Step 2: Install Arduino IDE & Upload ESP32 Code
**Who:** You + Me (Code)

**Action — You:**
1. Download and install Arduino IDE from https://www.arduino.cc/en/software
2. Open Arduino IDE
3. Go to File > Preferences
4. In "Additional Boards Manager URLs" add:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
5. Go to Tools > Board > Boards Manager
6. Search "esp32" and install "esp32 by Espressif Systems"
7. Connect ESP32 Thing Plus to PC via USB-C
8. Go to Tools > Board and select **"SparkFun ESP32 Thing Plus"** (or "ESP32 Dev Module" if not listed)
9. Go to Tools > Port and select the COM port that appeared when you plugged in the ESP32

**Action — Me:**
I will write `CursedAltar_ESP32.ino`.

**What the code does:**
- Connects to a WiFi network (you provide SSID and password)
- Prints assigned IP address to Serial Monitor (you note this down)
- Starts a TCP server on port 7777
- Reads ultrasonic sensor distance every 100ms
- Sends "DIST:<value>" to connected Unity client
- Listens for "BUZZER:ON" and "BUZZER:OFF" commands
- On "BUZZER:ON": plays an eerie tone on the passive buzzer using PWM
- On "BUZZER:OFF": stops the buzzer
- Handles client disconnection and reconnection

**Action — You (after I deliver the code):**
1. Open `CursedAltar_ESP32.ino` in Arduino IDE
2. Edit the WiFi credentials:
   ```
   const char* ssid = "YourWiFiName";
   const char* password = "YourPassword";
   ```
3. Click Upload
4. Open Serial Monitor (Tools > Serial Monitor, baud rate 115200)
5. You should see:
   ```
   Connecting to WiFi...
   Connected! IP: 192.168.x.x
   TCP Server started on port 7777
   ```
6. Note down the IP address

**Expected result:** ESP32 running, connected to WiFi, waiting for Unity to connect.

---

### Step 3: Test Hardware Standalone
**Who:** You
**Action:**
1. With Serial Monitor open, wave your hand above the ultrasonic sensor
2. You should see distance values:
   ```
   DIST:120
   DIST:85
   DIST:45
   DIST:15
   ```
3. Type "BUZZER:ON" in the Serial Monitor and press Enter — buzzer should sound
4. Type "BUZZER:OFF" — buzzer stops

**Expected result:** Sensor reads distance correctly. Buzzer responds to commands. Hardware verified independently before Unity integration.

---

### Step 4: Set Up the Cabin Scene in Unity
**Who:** You
**Status:** Unity project, Meta XR SDK, Cabin asset, and Skull asset already imported.

**Action:**
1. Open the `SampleScene` in Assets/Scenes/ (or create a new scene named "CursedAltar")
2. Delete the default Main Camera from the scene
3. Open **Meta > Tools > Building Blocks** panel
4. Add the **Camera Rig** Building Block into the scene
5. Add **Controller Tracking** Building Blocks (left and right)
6. Add the **Grab Interaction** Building Block — then reparent the interaction components off the default cube and delete the cube
7. Add the **Ray Interaction** Building Block — then reparent the interaction components off the default cube and delete the cube
8. Place cabin prefab from Assets/Cabin/Prefabs/
9. **Scale the cabin** in the Transform component until the interior is approximately 2m × 2m (must be tested on Quest 3S in the actual play space)
10. Place furniture prefabs inside the cabin:
    - **Table** — center of the room (this is the altar)
    - **Stool** — near the player start position (lantern sits on this)
    - **Bed** — one corner
    - **Cabinet** — against a wall, visible when player looks into the cabin
    - **Door** — behind the player start position (wall element, never opens)
    - **Candle** — on the table
    - **Lantern** — on the stool
11. Position the Camera Rig near the door, **facing into the cabin** toward the table (player starts looking at the stool/lantern, not at the door)
12. Darken the scene:
    - Select the Directional Light, reduce intensity to **0.0** (pitch black)
    - Or delete the Directional Light entirely
    - The only light comes from the lantern and the candle
13. Set up the lantern's Point Light:
    - The lantern prefab already has a Point Light child
    - Set range to ~2-3 (small radius)
    - Set intensity to ~1.5
    - **Disable the Point Light by default** — it activates when player picks up the lantern

**Expected result:** Dark cabin scene with furniture. Player spawns facing into the cabin, lantern on stool in front of them, door behind them.

---

### Step 5: Set Up the Lantern (Grab Object + Light Source)
**Who:** You + Me (Code)

**Action — You:**
1. Select the Lantern prefab in the scene
2. Add a **Rigidbody** component (Use Gravity: false, Is Kinematic: true)
3. Add a **Box Collider** or use existing collider
4. Add grab interaction:
   - Drag the **Grab Interaction** components onto the lantern (or right-click > Interaction SDK > Add Grab Interaction)
5. Make sure the Point Light child is **disabled by default**

**Action — Me:**
I will write `LanternController.cs`. You attach it to the lantern object.

**What the script does:**
- Detects when the lantern is grabbed
- Enables the Point Light and particle system (flame) on grab
- Light stays on as long as the player holds the lantern
- During the approach phase, lantern light flickers erratically based on distance data from ESP32Connection
- **Triggers the door slam event** ~5 seconds after being grabbed (starts the scare sequence)

**Expected result:** Player picks up lantern with left hand, light turns on. 5 seconds later, the door slams behind them.

---

### Step 6: Set Up the Door (Scripted Slam — NOT a player interaction)
**Who:** You + Me (Code)

**Action — You:**
1. Select the door object in the hierarchy
2. Add a **Rigidbody** component, set "Is Kinematic" to true
3. Add an **AudioSource** component (assign a loud door slam/creak sound)

No ray-cast components needed — the player does not interact with the door. It reacts on its own.

**Action — Me:**
I will write `DoorScare.cs`. You attach it to the door object.

**What the script does:**
- Has a public method `Trigger()` called by LanternController after lantern pickup
- When triggered:
  - Plays loud slam/creak sound
  - Rapidly shakes the door back and forth by 2-3° for ~0.5 seconds (code-driven rotation, no animator needed)
  - After shaking stops, **triggers the poltergeist stool event** after 3 seconds
  - One-shot — only triggers once

**Expected result:** ~5 seconds after lantern pickup, door slams and shakes on its own behind the player. Then stool starts sliding.

---

### Step 7: Set Up the Poltergeist Stool (Scripted Scare)
**Who:** You + Me (Code)

**Action — You:**
1. Select the stool in the scene (the same stool the lantern was on — player has already picked up the lantern, so the stool is now empty)
2. Add a **Rigidbody** component (Use Gravity: true, Is Kinematic: true)
3. Add a **Box Collider**
4. Add an **AudioSource** component (assign a slow scraping/dragging sound)

**Action — Me:**
I will write `PoltergeistObject.cs`. You attach it to the stool.

**What the script does:**
- Has a public method `Trigger()` called by DoorScare after the door slam
- Has a public `targetPosition` (Vector3) — set in the Inspector to where the stool should slide to
- When triggered:
  - Plays scraping/dragging sound
  - Slowly moves the stool from its current position to `targetPosition` over ~2-3 seconds using smooth interpolation (Lerp/MoveTowards)
  - The stool stays on the ground — it slides, not flies
  - One-shot — only triggers once

**Expected result:** After door slam, the stool slowly slides across the floor on its own. Creepy, deliberate, poltergeist movement.

---

### Step 8: Make the Cabinet Interactable (Ray-cast)
**Who:** You + Me (Code)

**Action — You:**
1. Select the cabinet object
2. Add a **Box Collider** component
3. Add a **Rigidbody** component, set "Is Kinematic" to true
4. Add a **ColliderSurface** — drag Box Collider into "Collider" field
5. Add a **RayInteractable** — drag ColliderSurface into "Surface" field
6. Add an **InteractableUnityEventWrapper** — drag RayInteractable into "Interactable View" field
7. Add an **AudioSource** (assign creak sound)

**Action — Me:**
I will write `CabinetController.cs`. You attach it to the cabinet.

**What the script does:**
- Listens for InteractableUnityEventWrapper's `WhenSelect` UnityEvent
- Plays cabinet creak sound
- Rotates the cabinet door open (code-driven rotation over ~0.5 seconds)
- Briefly flares the candle on the table (draws attention to altar)
- One-shot — only triggers once

**Setup — You (after attaching script):**
- In InteractableUnityEventWrapper, under `WhenSelect`, click "+" → drag cabinet object → select `CabinetController > OnCabinetSelected()`

**Expected result:** Point right controller at cabinet, pull trigger. It creaks open. Candle flares, attention drawn to altar.

---

### Step 9: Set Up the Candle (Proximity Reaction)
**Who:** You + Me (Code)

**Action — You:**
1. Locate the candle on the table
2. Ensure it has a **Point Light** component (Candle prefab already has one)
3. Set the Point Light:
   - Color: warm orange/yellow
   - Range: 3-5
   - Intensity: 0.5 (dim starting state)
4. Add an **AudioSource** component (assign creepy whisper audio, set to loop)

**Action — Me:**
I will write `CandleController.cs`. You attach it to the candle.

**What the script does:**
- Receives distance data from ESP32Connection (real ultrasonic sensor data via TCP)
- Maps distance to candle behavior:
  - **>1.5m:** Intensity 0.5 (dim, gentle flicker)
  - **1.0-1.5m:** Intensity 1.5 (brighter, pulsing). Whisper audio starts.
  - **0.5-1.0m:** Intensity 3.0 (bright, fast pulse). Whispers louder. Lantern flickers.
  - **<0.5m:** Candle dies (intensity 0). Total darkness 2 seconds. Reignites at 2.0. **Skull activated.**
- Also controls lantern flicker (communicates with LanternController)
- Has a simulation fallback: holding left controller X button decreases simulated distance (for testing without hardware)

**Expected result:** Walking toward table makes candle react via real sensor. At very close range, blackout then skull appears.

---

### Step 10: Set Up the Skull (Grab Object + Ritual Climax)
**Who:** You + Me (Code)

**Action — You:**
1. Place skull prefab on the table next to the candle (recommend SM_Skull01_Color01 or SM_Skull01_Color02)
2. **Set the skull as inactive** (uncheck checkbox in Inspector) — activated by CandleController
3. Add a **Rigidbody** (Use Gravity: true, Is Kinematic: false)
4. Add a **Box Collider** or **Mesh Collider**
5. Add grab interaction via Building Blocks
6. Add an **AudioSource** (assign heartbeat sound, set to loop)

**Action — Me:**
I will write `CursedSkull.cs`. You attach it to the skull.

**What the script does:**
- Detects grab (subscribes to GrabInteractable events)
- While held: plays heartbeat sound, candle pulses in sync
- Detects release near table (checks Y and XZ position relative to table)
- When placed on table:
  - Sends "BUZZER:ON" to ESP32 via TCP → **real buzzer fires**
  - VR screen flashes white
  - After 1.5 seconds: sends "BUZZER:OFF"
  - Room lighting gradually returns to warm normal over 2 seconds
  - Text appears: *"The room is sealed. For now."*

**Expected result:** Skull appears after blackout. Grab with right hand, hear heartbeat, place on table. Real buzzer fires. Room calms. Done.

---

### Step 11: ESP32 Connection Manager
**Who:** Me (Code)
**Action — You:** Create an empty GameObject named "NetworkManager" and attach the script.

**Action — Me:**
I will write `ESP32Connection.cs`.

**What the script does:**
- Connects to ESP32 via TCP at configurable IP and port (default: port 7777)
- Receives "DIST:<value>" messages and forwards to CandleController
- Sends "BUZZER:ON" / "BUZZER:OFF" when CursedSkull triggers it
- Reconnects automatically if connection drops
- Has a `simulationMode` boolean (default: false) as fallback for testing without hardware:
  - When true: X button simulates distance, buzzer commands log to console

**Inspector fields:**
- `esp32IP`: the IP address from Arduino Serial Monitor (e.g., "192.168.1.100")
- `esp32Port`: 7777
- `simulationMode`: false (set to true only if testing without hardware)

**Expected result:** Unity connects to ESP32 on Play. Real sensor data drives the experience.

---

### Step 12: Add Audio
**Who:** You
**Action:**
1. Create folder: Assets/Audio/
2. Import these audio clips (from freesound.org or any free source):

| # | Audio Clip | Used For | Type |
|---|-----------|----------|------|
| 1 | Ambient horror drone | Background atmosphere loop | Loop |
| 2 | Door slam/creak | Door scripted slam (loud, sudden) | One-shot |
| 3 | Slow scraping/dragging | Stool poltergeist slide | One-shot |
| 4 | Cabinet creak | Cabinet interaction | One-shot |
| 5 | Creepy whispers | Candle proximity (louder as player approaches) | Loop |
| 6 | Heartbeat | While holding the skull | Loop |
| 7 | Curse seal sound | Final moment — loud, dramatic | One-shot |

3. Assign clips to AudioSource components on each object
4. Create an empty GameObject "AmbientAudio" with AudioSource playing ambient drone on loop

Note: No separate buzzer simulation sound needed — the real buzzer handles it. The curse seal sound plays in VR simultaneously with the real buzzer for layered effect.

**Expected result:** Full audio atmosphere.

---

### Step 13: Test Full Loop (Editor + Hardware)
**Who:** You
**Action:**
1. ESP32 powered and connected to WiFi
2. Unity in Play mode, connected to ESP32 (check Console for "Connected to ESP32")
3. Test full sequence:
   - Pick up lantern (left hand) → light activates
   - ~5 seconds later → door slams on its own behind you
   - ~3 seconds after that → stool slowly slides across floor
   - Point right controller at cabinet → opens, candle flares
   - Walk toward ultrasonic sensor → candle reacts in VR
   - Get very close (<50cm) → blackout → skull appears
   - Grab skull (right hand) → heartbeat plays
   - Place on table → **real buzzer fires**, screen flashes, room calms

**Expected result:** Full bidirectional game loop working in editor with real hardware.

---

### Step 14: Build and Test on Quest 3S
**Who:** You
**Action:**
1. Make sure Quest 3S is on the **same WiFi network** as the ESP32
2. File > Build Settings > Switch Platform to Android > Build and Run
3. Put on Quest 3S
4. Set Guardian boundary to ~2m × 2m
5. Power ESP32 from USB power bank (disconnect from PC)
6. Test full experience in VR
7. **Adjust cabin scale** until VR walls match Guardian boundary
8. **Adjust furniture positions** for natural exploration flow

**Networking options:**
- **Home WiFi:** Use during development
- **Phone hotspot:** Use on demo day (portable, reliable)
- **University WiFi:** Likely won't work (device isolation)

**Expected result:** Fully wireless. ESP32 on power bank, Quest standalone. No PC needed.

---

### Step 15: Physical Demo Setup
**Who:** You
**Action:**

```
DEMO SETUP (top view):

    ┌────────────────────────────┐
    │                            │
    │  ┌──────────────────────┐  │
    │  │  Table               │  │
    │  │  (breadboard + ESP32 │  │
    │  │   + buzzer HIDDEN    │  │
    │  │   underneath)        │  │
    │  └──────────────────────┘  │
    │                            │
    │  [Ultrasonic on floor,     │
    │   pointing toward player]  │
    │                            │
    │         ~2m walk           │
    │                            │
    │       ┌──────────┐         │
    │       │  Player  │         │
    │       │  starts  │         │
    │       │  here    │         │
    │       └──────────┘         │
    │    (tape mark on floor)    │
    │                            │
    └────────────────────────────┘
         ~ 2m × 2m area
```

1. Place a table in the demo area
2. **Under the table:** breadboard + ESP32 + buzzer (hidden) + power bank
3. **On floor near table, pointing outward:** ultrasonic sensor, taped down
4. Mark play area (~2m × 2m) and player start position with tape
5. Calibrate: put on headset at start position, verify VR layout matches physical space
6. Phone hotspot in your pocket (if using hotspot for demo day)

**Expected result:** Clean setup. Hardware hidden. Player walks naturally.

---

### Step 16: Record Video
**Who:** You
**Action:**
1. Record two views simultaneously:
   - **VR view:** Quest's built-in screen recording
   - **Physical view:** Phone camera showing setup, player walking, buzzer firing
2. Show full game loop:
   - Lantern pickup, light activates
   - Door slams on its own, stool slides
   - Cabinet opens
   - Player walks toward table — show legs approaching sensor
   - Candle reacts, blackout, skull appears
   - Skull grab and placement
   - Buzzer fires — show it in physical view
   - Room calms, experience ends
3. Keep video 2-3 minutes maximum

**Expected result:** Clear demo video showing both VR and physical hardware interaction.

---

### Step 17: Write README
**Who:** Me (Code) + You (Review)

I will write README.md covering:
- Project overview and concept
- Scientific rationale (break in presence, cross-modal feedback — citing research)
- Hardware list and wiring diagram
- Software requirements
- Setup instructions
- How to run the project
- Demo video link

You review, add your video link, and submit.

---

## FILES I DELIVER

| File | Purpose |
|---|---|
| `CursedAltar_ESP32.ino` | ESP32 Arduino code (WiFi, TCP server, ultrasonic reading, buzzer control) |
| `ESP32Connection.cs` | Unity TCP client, receives distance, sends buzzer commands |
| `LanternController.cs` | Lantern grab detection, light activation, flickering, triggers door slam |
| `DoorScare.cs` | Door auto-slam and shake, triggers poltergeist stool |
| `PoltergeistObject.cs` | Stool slow slide to target position |
| `CabinetController.cs` | Cabinet open on ray-cast |
| `CandleController.cs` | Candle proximity reaction, blackout, skull activation |
| `CursedSkull.cs` | Skull grab, heartbeat, placement detection, buzzer trigger, end sequence |
| `README.md` | Project documentation for submission |

```
/Assets/Scripts/
  ├── ESP32Connection.cs
  ├── LanternController.cs
  ├── DoorScare.cs
  ├── PoltergeistObject.cs
  ├── CabinetController.cs
  ├── CandleController.cs
  └── CursedSkull.cs

/ESP32/
  └── CursedAltar_ESP32.ino

/README.md
```

---

## EVENT CHAIN

This shows how scripted events chain together automatically after the player picks up the lantern:

```
Player grabs lantern
    │
    ├── Light activates immediately
    │
    └── After ~5 seconds
            │
            ├── Door slams and shakes (DoorScare.cs)
            │
            └── After ~3 seconds
                    │
                    └── Stool slides across floor (PoltergeistObject.cs)

Player ray-casts cabinet (whenever they choose)
    │
    ├── Cabinet creaks open
    └── Candle flares (draws attention to altar)

Player walks toward altar (ultrasonic sensor active throughout)
    │
    ├── >1.5m: dim flicker, faint whispers
    ├── 1.0-1.5m: brighter, louder whispers, lantern flickers
    ├── 0.5-1.0m: intense pulse, red shift, sounds escalate
    └── <0.5m: blackout → skull appears

Player grabs skull and places on table
    │
    ├── Real buzzer fires (ESP32)
    ├── VR screen flash + seal sound
    └── After 1.5s: buzzer stops, room calms, end text
```

---

## FULL CHECKLIST

### Hardware
- [ ] Circuit wired on breadboard (ESP32 Thing Plus + HC-SR04 + buzzer + voltage divider)
- [ ] Arduino IDE installed with ESP32 board support
- [ ] ESP32 code uploaded, connects to WiFi, prints IP
- [ ] Hardware tested standalone (distance in Serial Monitor, buzzer responds)

### Unity Scene
- [ ] Camera Rig + Controller Tracking Building Blocks placed
- [ ] Grab + Ray Interaction Building Blocks added (cubes removed)
- [ ] Cabin placed and scaled to ~2m × 2m interior
- [ ] Furniture placed: Table, Stool, Bed, Cabinet, Door, Candle, Lantern
- [ ] Player starts inside cabin, facing toward table, door behind them
- [ ] Scene is pitch black (Directional Light off)
- [ ] Lantern: Rigidbody + Collider + Grab Interaction + LanternController.cs (light disabled by default)
- [ ] Door: Rigidbody (kinematic) + AudioSource + DoorScare.cs (no ray-cast — scripted only)
- [ ] Stool: Rigidbody (kinematic) + Collider + AudioSource + PoltergeistObject.cs (target position set)
- [ ] Cabinet: Box Collider + ColliderSurface + RayInteractable + InteractableUnityEventWrapper + CabinetController.cs
- [ ] Candle: Point Light + AudioSource + CandleController.cs
- [ ] Skull: Rigidbody + Collider + Grab Interaction + AudioSource + CursedSkull.cs (inactive by default)
- [ ] NetworkManager object with ESP32Connection.cs (esp32IP and port configured)
- [ ] 7 audio clips imported and assigned

### Integration & Testing
- [ ] Unity connects to ESP32 in editor — bidirectional communication works
- [ ] Full game loop works in editor with real hardware
- [ ] Event chain fires correctly: lantern → door slam → stool slide → (player explores) → candle reacts → skull → buzzer
- [ ] APK built and deployed to Quest 3S
- [ ] Quest and ESP32 on same WiFi — wireless loop works
- [ ] Cabin scale calibrated in VR to match physical play area

### Demo & Submission
- [ ] Physical demo setup arranged (sensor on floor, buzzer hidden, area marked)
- [ ] Video recorded (VR view + physical view)
- [ ] README written and reviewed
- [ ] Files submitted

**New additons**
-After the skull appeared on the table, a whisper sound played “takemewithyou.mp3”. This sound will be played every 5 seconds until the player grabs the skull with the controller trigger button. Once the player has grabbed the skulls, the whisper sound will not again be played. And after holding the skull, pressing the controller trigger button won’t release the skull. It will be stuck to the controller. 

-Right when the skull appears in the scene, the pentagram and the hanginggirl will also appear in the scene. Players can see those objects if they look around. The hanging girl from the ceiling will be slowly moving from a hinge position in rope within Y direction. As if it took its own life. 

-Player grabs the skull and take it closer to candles on the pentagram. Taking the skull closer to each candle will light them up (Just trigger the particle system on the candles, not the point light). 

-After lighting all 5 candles (The orders doesn’t matter), another whisper in the headset “returnmetowhereibelong.mp3” in every 5 seconds until the player grabs the skull and puts it back on the table. 

-Once the player puts the skull back on the table, the stare contest begins:

Stare contest: Right whenever the player puts the skull on the table, the skull will start glowing which proportionally progresses with the controller vibrations. The glowing effect’s max reach would be the controller’s maximum vibration limit. 

-Once the maximum limit reaches, the whole VR scene would slowly deem and black out within 5 seconds. After it’s completely blacked out, the  background music will still be played and the buzzer will trigger. 

-The buzzer sound will continue to loop for 10 seconds. And when in VR headset audio, a whisper will be played, “leavewhileyoustillcan.mp3”.

**New additional gameplay (part-1)**
Right whenever the player places the skull on the table, implement a 3 second delay before triggering the stare contest. After 3 seconds delay, the hanging girl will start moving at an increased speed and the door starts rattling back and forth and the “barndoor” sound plays in a loop. (For development of this mechanics- check how the door is behaving now at the very start of the game. It shuts smoothly and the door has a hinge position. You need to implement this door rattling carefully so it looks back and forth). Then after a 2 seconds delay trigger the stare contest so if the player looks at the skull without walking to the door, the stare game starts. Within the stare game and door rattle, if the player walks to the door without completing the stare contest, the slams shut just like the beginning of the game. If the door shuts and the player walks back to the table, the door starts rattling again. 

After completing the stare contest, the screen goes black like how it is now and the video will be played in VR. After the video ends, the screen remains black and the whisper sound plays, “leavewhileyoustillcan” and then the buzzer triggers. 


**New additional gameplay (part-2)**
After the cabinet hits the table, wait for 10 seconds and play the sound, “wallsremembercrosswaits”. Play this every 30 seconds and stop after the first interaction the player does with the cross. This is because once the player finds out how to interact with the cross, there is no need for playing the sound again in the game. 

After the player stands infront of the cross with a 20 degree gaze angle, system should take left/right controller thumbstick input. The cross should 5 correct thumbstick movement from the player. The correct order is below:


Up-Left-Down-Right-Up

Any wrong input in the cycle will restart the cycle from the beginning. Remember that. 

With every correct thumbstick input, the cross should glow red for 0.5 seconds. Take the same red glow effect from the skull. 

After 5 successful inputs, the cross should move upside down. A smooth move infront of player eyes. You have make the same hinge mechanism at the very bottom of the cross. From that hinge point the cross would move upside down. It will symbolise a demonic upside down cross. 

After the cross change its position, play the sound, “altarcall”. Play it every 10 seconds until the player reaches closer to the altar. You have to measure the distance between the table and the player. You have already built this mechanism for door shut at the very beginning of the game. Use the same mechanism here as well. Once the player reaches closer to the alter, stop the “altarcall” sound and display the “Text(TMP)” game object with a fade-in effect. 

Rest of the game remains the same as before. 

A very important point to remember:
-The controller thumbsticks have already built-in functionalities in the game. You have to disable that first and then write your thumbstick logic on that.

**New jumpscare additions**
Cross rotation jump scare:

When the cross fully rotates, play the sound, “churchbell” and decrease the lantern point light that the player is holding by 20%. So the event sequences are cross fully rotates ->churchbell sound plays -> lantern point light decrease. All these 3 events will happen at the same time. 

Hanging girl jump scare:
At ~80% of the skull glowing, everything in the room stops (All the sounds, background sound and whispering) and goes dark immediately (No fade out). This time don’t destroy the game objects in the background. The player can only see the Lantern in his hand in the pitch black room and nothing else (Remember to turn off the point light of the Lantern when room goes dark). But, the game should still detect the player's gaze in that darkness. 

Systematically, when the room goes dark at ~80% skull glow, the hanging girl along the rope should be positioned in the middle of the pentagram. That means the hanginggirl should be taken down to the floor level in the middle of the pentagram. So in the darkness, when player gazes at the hanginggirl object, below events should happen immediately: 

Light up the whole room again
Play the sound “pianostinger” for once
Play the sound “rockingchair” continuously in a loop
Play the sound “barndoor” continuously in a loop
Activate “RockingChair” game object
Activate door rattling along with the glow effect
Activate pentagram game object along with its candle children and the particle systems

All these events happen all at once. And last for 5 seconds. After 5 seconds, the game fades to black like before, goes completely dark and the video plays. After the video ends, “leavewhileyoustillcan” plays. This you already know.


**Final Gameplay**
The ESP32 development part:


Hardwares:
ESP32-S2 Thing plus (Utlizes WROOM chip)
Ultrasonic distance meter HC-S404
Passive piezo buzzer (4 pieces)
Cable, breadboard and jumper wires
No external power supply

The gameplay:
After the video finishes and the sound plays “leavewhileyoustillcan”, Player will be transmitted to to passthrough mixed reality mode from dark VR state. The transition should be like fade-in effect. From VR darkness -> player fading into the passthrough mixed reality mode where they can see everything in the real world. 

The ultrasonic distance meter will measure the distance of the player. If close enough, in the mixed reality environment, the object “vhs” will appear on the table. Currency the object is in the unity scene placed on the table. But deactivated. 

Player can now grab the vhs object with either of their controller trigger button as both hands are now free. 

Once they hold the vhs, the buzzers will fire for 5 seconds. After 5 seconds the buzzers will stop and  in the VR headset earphone they will hear the audio “sevendays”. The audio is in the Sounds folder. 

Audio ends and the vhs object disappears from player’s hands/controllers. 

The game ends here.

**additional instruction**
At the very start of the game, after 10 seconds play the sound "takethelight" and show the text hovering over the lantern "Right Trigger to grab". Once the player grabs the lantern, remove the text and no need to playing the sound again either. But if the player grabs the lantern before 10 seconds. Don't play the sound and do not show the text. 

Show the text on the cabinet, "Pull the left trigger, release what's inside." This text will be shown as a fade-in text when the player will approach the cabinet. So it should be gaze based interaction. Once the cabinet fall is triggered, remove the text from the cabinet. 

After the cabinet falls, if the player doesn't figure out how to do the cross ritual in 2 minutes, show the text on the cross, "Use left controller thumbstick." Once the user press a single left controller thumbstick, the text dissappears and will never be shown. 

After when the player puts the skull back on the table, after 1 minute play the sound, "comecloser"