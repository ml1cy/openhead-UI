# Android implementation notes

Target: **plain Android tablet**, app running as launcher. Not Android Automotive OS — none of `CarPropertyManager`, `RadioManager` or the car app library is available.

## Shell

- Single activity, `WindowCompat.setDecorFitsSystemWindows(false)`, immersive sticky, `keepScreenOn`.
- Register as `HOME`/`LAUNCHER` and use **lock task mode** (device owner via ADB provisioning) so the tablet cannot be navigated out of.
- Landscape locked. The design is authored at 1920×1080; treat 132px dock, 20px gaps and the type scale as dp values against a 1920dp-wide design width and scale proportionally for other panels. Below ~1280dp wide, drop the right rail on Radio and the 3×2 tile grid to 2×3.
- Keep the boot path short — a head unit is judged on how fast it shows something after ignition. Splash straight into Home with cached state; hydrate sources asynchronously.

## The single now-playing model

Every source publishes into one `MediaSession`; the home hero, the radio hero and the dock read only from its `MediaController`. This is the spine of the design — do it first, before any individual source.

```
sealed interface Source { object Fm; object Am; object Dab; object Stream; object Bluetooth }
data class NowPlaying(
  val source: Source, val title: String, val subtitle: String,
  val artUri: Uri?, val position: Duration?, val duration: Duration?, val live: Boolean
)
```

## Per feature

### Android Auto projection — already in the repo
The Projecting frame is a `SurfaceView`. Keep the existing USB/Wi-Fi transport code; the redesign only changes what surrounds it. Audio focus: projection takes exclusive focus, so the radio and media sources must pause and the dock must reflect it.

### Internet radio — easy
Media3/ExoPlayer, HLS and Icecast/SHOUTcast. Metadata from ICY headers feeds the subtitle. Foreground service with a media notification.

### Navigation — hand off, do not rebuild
`Intent(ACTION_VIEW, "google.navigation:q=…")` for Maps, or OsmAnd's AIDL API if you want it in-process. The floating manoeuvre card can be populated from a `NotificationListenerService` reading the nav app's ongoing notification. The map pane in the design is a placeholder; if you want a real map in-app, MapLibre with offline MBTiles is the sane choice for a car.

### Vehicle data — ELM327
Bluetooth SPP or USB to an ELM327 clone; poll standard PIDs (`0C` RPM, `0D` speed, `05` coolant, `2F` fuel level, `42` module voltage). Poll at 2–5 Hz, not faster — cheap adapters choke. Tyre pressure is not a standard OBD PID; it needs either a manufacturer-specific PID or an aftermarket TPMS receiver, so treat that card as optional. Trip figures are yours to accumulate and persist.

### Camera — USB UVC
`libuvc`/UVCCamera (or `android.hardware.usb` + a UVC wrapper) rendered into a `SurfaceView`; the platform Camera2 API generally will not see a USB camera on a stock tablet. Reverse trigger needs a real signal — a USB GPIO board or a CAN interface on the reverse light circuit. Dashcam loop: `MediaRecorder` writing fixed-length segments into a ring buffer directory, "Save last 30 s" copies the tail out.

### FM / AM / DAB+ — hardware required
There is no Android radio API here. You need a tuner module on USB:
- FM/AM: a TEF6686 or Si4735 board behind a USB-serial bridge. You implement the command protocol, seek, and RDS group decoding (PS for station name, RT for the now-playing text) yourself.
- DAB+: a separate DAB module or an RTL-SDR with a DAB decoder. This is substantially more work than FM.

The tuning UI is designed around this: the ruler window, 0.1 MHz / 9 kHz steps, seek-to-next-station and RDS text all map onto what these modules expose. If radio hardware slips, the Radio screen still works as DAB/Internet/Bluetooth only — hide the FM and AM tabs.

### Bluetooth media and telephony — the privileged part
- **A2DP sink** (phone → head unit audio) is disabled on stock Android. It needs `bluetooth.profile.a2dp.sink.enabled=true` in the platform config, which means a custom ROM or a platform-signed build. Same story for **AVRCP** controller metadata.
- **HFP client** (hands-free calling) likewise: `BluetoothHeadsetClient` is a system API. Without it the Phone screen can dial via `ACTION_CALL` on a tablet with a SIM, but it cannot act as a car kit for the driver's phone.
- Contacts and call log come over **PBAP** (`BluetoothPbapClient`), also privileged.

If a custom ROM is out of scope, the honest fallbacks are: let Android Auto own calls and phone audio entirely, and cut the Phone dock item; or ship the Phone screen as dial-out only and label it as such. Decide this early — it changes the dock.

## Suggested order

1. Shell: launcher, dock, status bar, quick-settings drawer, theme tokens, day/night.
2. The `MediaSession` spine plus the Home screen reading from it.
3. Wrap the existing Android Auto projection in the new Auto screen.
4. Internet radio — proves the whole source abstraction end to end with no hardware.
5. Settings, profiles, navigation hand-off.
6. OBD and the Vehicle screen.
7. Camera.
8. Radio hardware.
9. Bluetooth sink / HFP, once the platform question is settled.

## Theming

Map the token tables in the README onto a Compose `ColorScheme` (or a plain `CompositionLocal` holding the token set — the palette is not Material-shaped, so a custom holder is less friction than bending M3 roles). Day/night should switch on the light sensor with a manual override, hysteresis on the threshold, and a crossfade rather than a hard cut. Brightness in the drawer drives `WindowManager.LayoutParams.screenBrightness`.
