# R3Dirt5Bridge v0.5 — handbrake and shifter buttons

One English Windows app for MOZA R3 native wheel input, Steam DiRT 5 FFB launch and USB handbrake/shifter inputs used as buttons. **Controller** remains the first tab; **Native wheel / FFB / Handbrake** is second; **Shifter buttons** is third. The analog handbrake adapter has been retired.

## Handbrake setup

1. Open the second tab. Select **Hand Brake**, physical axis **Slider 1**. Move the lever once to initialize its driver.
2. Ensure Input increases from 0% released to 100% fully pulled. Use **Invert handbrake input** if it runs backwards. The optional released/pulled captures recalibrate the range and detect its direction.
3. Choose **Button key** (default **Space**) and select **Use handbrake button**.
4. Assign that keyboard key to the **Handbrake** action in DiRT 5. The lever now presses the selected key at **50% or more** and releases it below **50%**. You can pull the lever past halfway while the game is waiting for a key binding, or assign the chosen key directly.
5. Keep this app open. The selected key is sent only while this installation of DiRT 5 is the active window. Switching away from the game, disabling the checkbox, disconnecting the lever or closing the app releases it. Changing the selected key also releases the old key first.

The meter still shows physical travel for setup; the game receives an on/off keyboard key, not an analog axis or a virtual Xbox controller. The output is held once while above threshold, without repeated key-down injection. Supported keys are Space, Left Shift, Left Ctrl, H, B, X, J, F9, E and Q.

Settings: `%LOCALAPPDATA%\R3Dirt5Bridge\handbrake.json`. Fields store device IDs, physical axis, released/pulled endpoints, calibration, input inversion, enabled state and `OutputKey`. Existing physical calibration and inversion are retained. The obsolete native-axis inversion is ignored.

## Shifter buttons

1. Open the third tab, **Shifter buttons**, and select **Generic USB Joystick**. Put the gearbox in sequential mode. This connected device reports 12 buttons.
2. Leave **Use shifter buttons** checked. Physical **button 4** sends **E**, and physical **button 3** sends **Q**, using the same keyboard output as the handbrake. The app does not select in-game actions for these keys.
3. In DiRT 5 **keyboard controls**, select the action you want to assign and move the lever in the required direction. You can also press E or Q on the keyboard to assign it. For sequential shifting, assign the button-4 direction to Gear Up and button-3 direction to Gear Down. Use manual transmission when testing gear changes.
4. Keep the app open and start your native FFB game session from the second tab as usual. Both accessory outputs keep working with their tabs hidden. No Xbox bridge needs to be started.

Change the two output keys if necessary. **Swap buttons** reverses the physical directions; **Learn button A/B** allows different physical buttons. Choose distinct keys for the two shifter buttons and the enabled handbrake. Closing/disabling, losing game focus, disconnecting, or pressing both mapped shifter buttons releases output. Center/release the lever before rearming. Holding a button injects one key-down, followed by a key-up on release, without repeated key-down injection.

Settings: `%LOCALAPPDATA%\R3Dirt5Bridge\shifter.json`. Displayed physical button numbers start at 1; stored `UpButton`/`DownButton` indices start at 0. Those legacy field names identify outputs A/B and do not choose the game's actions. Wheel paddles retain their existing native bindings.

## Native wheel and FFB

The R3 native profile is already installed on this computer and the user confirmed correct steering, pedals, wheel buttons and FFB before this update. Keep rotation **540°** in Pit House. Use the second tab for native play; selecting the second or third tab stops the Xbox fallback reader and output.

With DiRT 5 closed, open this app and leave **Automatically enable native FFB for Steam / VR launches** checked on the second tab. Wait for **FFB ready for Steam / VR**. Then start your usual VR launcher, Steam shortcut, or the app's **Start Steam game** button. Keep the app open. Automatic detection has no three-minute limit and rearms when the game closes. No separate preparation is needed for each restart.

A new game session displays **Native FFB applied** with its process ID after the runtime write is read back. The wheel/Xbox preview is disposed before patching the new session. **Arm / retry automatic FFB** repeats validation after an error; close the game first. If the app was opened after the game was already running, it asks you to close/restart the game so the patch can be applied during initialization. First force testing should use 20% in Pit House.

The checkbox stores `AutoFfb` in the existing `handbrake.json`. Disabling automatic detection affects future launches; it does not remove a patch from an already-running game. **Start Xbox fallback** disables the automatic native watcher to keep the two modes separate. Re-enable the checkbox for native R3 play.

The FFB launcher validates Steam `DIRT5.exe` **1.0.277037.506**, its full hash, native R3 profile, device capabilities and function context before patching nine bytes in running game memory. The exe on disk and VR files remain unchanged. The memory patch ends when the game closes; the automatic watcher applies it again for each new process. Game-file modification times and lengths are checked on each launch against the files validated when arming, and function context is checked before every write. Changed files stop automatic preparation and require revalidation.

Game path: `F:\SteamLibrary\steamapps\common\DIRT 5`.
FFB log: `%LOCALAPPDATA%\R3Dirt5Bridge\ffb-launcher.log`.

## Retired analog adapter

The earlier owned `dinput8.dll` is removed from the game folder, with a verified recovery copy. Version 0.5 also checks for the owned legacy adapter during game preparation and backs it up before removal. Unrelated DLLs are preserved. The new button mode does not install a game DLL or require a virtual device driver.

## Xbox fallback

**Start Xbox fallback** explicitly starts the optional controller emulation. The first **Controller** tab and Xbox steering/buttons/diagnostics retain the original fallback configuration in `%LOCALAPPDATA%\R3Dirt5Bridge\config.json`. Its axis wizard captures center, right endpoint, full throttle and full brake. Steering maps to Left Stick X, throttle to RT, brake to LT, and button learning maps paddles/buttons to Xbox actions. Steering is linear without an added deadzone or curve. The game can still apply its own stick processing.

The fallback requires ViGEmBus and supplies no wheel FFB. Use the native second tab for this installation. Switching between the two modes requires the game to be closed. Existing wheel calibration and button mappings are retained.

## Build and checks

The complete runtime is included in the binary ZIP. Extract all files and launch `R3Dirt5Bridge.exe`. Building from source requires **.NET 10 SDK** on Windows x64; no C++ tools are needed:

```powershell
.\build.ps1
```

Individual checks:

```powershell
.\R3Dirt5Bridge.exe --check
.\R3Dirt5Bridge.exe --self-test
.\R3Dirt5Bridge.exe --watch-self-test
.\R3Dirt5Bridge.exe --auto-ffb-ready-test
.\R3Dirt5Bridge.exe --handbrake-self-test
.\R3Dirt5Bridge.exe --button-self-test
.\R3Dirt5Bridge.exe --shifter-self-test
.\R3Dirt5Bridge.exe --gui-self-test
```

The watcher test patches two real child-process allocations and verifies both writes; it never patches a game or sends force. The automatic-ready check exercises form/worker startup with game patching prohibited. The button and shifter self-tests use recording senders and sends no real keys. The GUI mode check needs the existing wheel configuration and connected devices, with the game/app closed. The earlier 55-test core mapping suite is part of the build script. See `VERIFICATION.md` for current checks and in-game test status.

The FFB approach derives from [DiRT5-MOZA-R5-Native-FFB-Fix](https://github.com/mikispike/DiRT5-MOZA-R5-Native-FFB-Fix); its MIT reference license is included. The Steam address was independently verified for this installation.
