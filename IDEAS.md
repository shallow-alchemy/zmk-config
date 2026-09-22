# Keymap ideas

Capture only. Nothing here is applied until it gets its own commit to
`config/corne.keymap`. Newest at the top of each section.

Current thumb cluster (same on every layer):

```
left:   &kp LCTRL     &kp LGUI        &lt SYM SPACE
right:  &lt NUM RET   &lt CTRL BSPC   &cht RALT DEL
```

## Ideas

### Swap Option and Ctrl (2026-09-21)

Option (RALT, right outer thumb, tap = DEL) and Ctrl (LCTRL, left outer
thumb) should switch places.

Rough shape:

```
left:   &kp LALT      &kp LGUI        &lt SYM SPACE
right:  &lt NUM RET   &lt CTRL BSPC   &cht LCTRL DEL
```

Things to think about before doing it:

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
