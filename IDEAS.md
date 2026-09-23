# Keymap ideas

Capture only. Nothing here is applied until it gets its own commit to
`config/corne.keymap`. Newest at the top of each section.

Current thumb cluster (same on every layer):

```
left:   &kp LCTRL     &kp LGUI        &lt SYM SPACE
right:  &lt NUM RET   &lt CTRL BSPC   &cht RALT DEL
```

## Ideas

### Swap Option and Ctrl (2026-09-21) — BUILT, PR #1 (2026-09-23)

Option (RALT, right outer thumb, tap = DEL) and Ctrl (LCTRL, left outer
thumb) should switch places.

Rough shape:

```
left:   &kp LALT      &kp LGUI        &lt SYM SPACE
right:  &lt NUM RET   &lt CTRL BSPC   &cht LCTRL DEL
```

Built as `&kp LALT` left / `&hpt RCTRL DEL` right, where `hpt` is a new
hold-preferred hold-tap (see the Option+minus diagnosis below for why).
The same PR also fixes two build breaks unrelated to the keymap: the
Zephyr 4.1 board id (`nice_nano//zmk`) and the removed
`CONFIG_WS2812_STRIP` symbol.

Things that were thought about before doing it:

- Ctrl becomes a hold-tap (`cht`, tap-preferred, 200ms). Fast Ctrl chords
  may tap DEL instead of holding Ctrl until the timing settles.
- Ctrl chords on right-hand letters (N, E, I, O, H, ...) become same-hand
  holds with the thumb. Cross-hand chords (Ctrl+C, Ctrl+A, Ctrl+R) get
  easier.
- Removes the Ctrl+Space = NUL (`^@`) mistake seen 2026-09-01, since Ctrl
  and Space are no longer neighbours. Option+Space takes its place; in
  many macOS apps that inserts a non-breaking space, so check kitty.
- The layer named `CTRL` has nothing to do with the Ctrl modifier. Worth
  renaming (NAV?) at the same time so the keymap reads clearly.
- tmux prefix and Helix bindings that lean on Ctrl should be re-tested on
  the new thumb.

### Why Option+minus does not work (2026-09-21)

Not the terminal: kitty has `macos_option_as_alt yes`, tmux has extended
keys on, macOS layout is U.S., no Karabiner. It is firmware timing.

1. Option is `&cht RALT DEL`, a `tap-preferred` hold-tap with a 200ms
   term. It only becomes Option after 200ms of holding; released sooner
   it taps DEL. Pressing other keys does not speed the decision up.
2. Minus lives on SYM, reached by `&lt SYM SPACE`, which is also
   `tap-preferred` with a 200ms term.
3. ZMK captures every key pressed while a hold-tap is undecided and
   replays it after the decision. So Space is not even looked at until
   Option has resolved, and the minus key is not looked at until Space
   has resolved. Option+minus needs both thumbs held for roughly 400ms
   before the minus key counts. A normal chord is over in 150ms, so the
   real output is DEL, a space, or the letter under the key.

Ctrl and Cmd do not have this problem because they are plain `&kp` on
the thumbs: one 200ms wait for the layer, not two.

Test to confirm: in a text field, hold the Option thumb, count one, hold
Space, count one, tap the minus key. It should type an en dash.

Testing note: the Claude Code input is not a valid test. kitty has
`macos_option_as_alt yes`, so Option+minus arrives as Alt+minus, and Claude
Code binds nothing to Alt+minus, so nothing visible happens even when the
keyboard is right. Use `kitten show-key -m kitty` in a spare pane: it
prints `alt+minus` (keyboard fine, app ignores it) or `delete` (hold-tap
timing). If the goal is typing an en or em dash in the terminal, that is a
kitty setting (`macos_option_as_alt left` frees the right Option key for
characters), not a keymap change.

Target use (2026-09-21): Helix multi-cursor. Two neighbouring bindings:

- `Alt-,` = remove_primary_selection (drops one cursor). Comma is on the
  base layer, so this is Option thumb + comma: one 200ms hold, no layer.
  If this also fails at speed, the hold-tap alone is the problem.
- `Alt-minus` = merge_selections (collapse all cursors into one span).
  Minus is on SYM, so this is the stacked 400ms case above.

Cheapest fix if only Helix matters: remap in `~/.config/helix/config.toml`
so both commands sit on base-layer keys, e.g. `A-m` = merge_selections.
Fix that helps everywhere: make the Option thumb plain `&kp` or
`hold-preferred`.

Implication for the Option/Ctrl swap above: the swap as sketched moves
this exact problem onto Ctrl (Ctrl+C, Ctrl+A, and every Ctrl+layer key).
Whichever modifier sits on that thumb should be a plain `&kp`, or a
`hold-preferred` hold-tap, not `tap-preferred`. DEL can move to the CTRL
layer, which has room.

## Observations from reading the keymap (2026-09-21)

- `media_layer` binds `&lt 5 SEMI` on the SEMI key, but there is no layer
  5 (layers are 0..4). Probably meant `&trans` or `&kp SEMI`. Bug
  candidate.
- Cut/copy/paste tap dances were added (68e38ac, 00468c3) and then dropped
  in the "redesign layers" commit (a76ff19). If they were dropped for a
  reason, note it here so they do not get re-proposed.
- F13-F19 are unused, and NUM and CTRL have plenty of `&none` slots.
  Nothing in kitty, zsh, tmux or Helix claims F13+, so a layer key
  emitting one is a guaranteed conflict-free binding.
- Backspace is `&lt CTRL BSPC`. Held slightly too long it enters the
  layer and emits nothing, which reads as "backspace did nothing".
  Candidate for a `quick-tap` or `hold-trigger-on-release` tweak.
