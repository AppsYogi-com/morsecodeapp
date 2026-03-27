# API Reference

Full documentation for `@morsecodeapp/morse` — core (v0.1.0) + audio (v0.2.0).

All core exports are available from `@morsecodeapp/morse` and `@morsecodeapp/morse/core`.  
All audio exports are available from `@morsecodeapp/morse` and `@morsecodeapp/morse/audio`.

---

## Table of Contents

### Core

- [encode()](#encode)
- [encodeDetailed()](#encodedetailed)
- [decode()](#decode)
- [decodeDetailed()](#decodedetailed)
- [Charsets](#charsets)
- [Prosigns](#prosigns)
- [Timing](#timing)
- [Stats](#stats)
- [Validation](#validation)
- [Types](#types)

### Audio

- [MorsePlayer](#morseplayer)
- [Sound Presets](#sound-presets)
- [WAV Export](#wav-export)
- [Scheduler](#scheduler)
- [Audio Types](#audio-types)

---

## encode()

Converts text to Morse code.

```ts
function encode(text: string, options?: EncodeOptions): string
```

**Examples:**

```ts
import { encode } from '@morsecodeapp/morse';

encode('SOS');                                     // '... --- ...'
encode('Hello World');                             // '.... . .-.. .-.. --- / .-- --- .-. .-.. -..'
encode('ПРИВЕТ', { charset: 'cyrillic' });         // '.--. .-. .. .-- . -'
encode('SOS', { dot: '•', dash: '—' });            // '••• ——— •••'
encode('SOS', { separator: '|', wordSeparator: ' // ' });
```

### EncodeOptions

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `charset` | `CharsetId` | `'itu'` | Character set to use |
| `fallbackCharsets` | `CharsetId[]` | `[]` | Additional charsets to try for unmatched characters |
| `dot` | `string` | `'.'` | Dot symbol in output |
| `dash` | `string` | `'-'` | Dash symbol in output |
| `separator` | `string` | `' '` | Separator between letters |
| `wordSeparator` | `string` | `' / '` | Separator between words |
| `invalid` | `string` | `'?'` | Replacement for characters with no mapping |

---

## encodeDetailed()

Like `encode()` but returns an `EncodeResult` with validity info.

```ts
function encodeDetailed(text: string, options?: EncodeOptions): EncodeResult
```

**Example:**

```ts
import { encodeDetailed } from '@morsecodeapp/morse';

encodeDetailed('A§B');
// { morse: '.- ? -...', valid: false, errors: ['§'] }

encodeDetailed('SOS');
// { morse: '... --- ...', valid: true, errors: [] }
```

### EncodeResult

| Field | Type | Description |
|-------|------|-------------|
| `morse` | `string` | The Morse code output |
| `valid` | `boolean` | `true` if all characters were mapped |
| `errors` | `string[]` | Characters that could not be encoded |

---

## decode()

Converts Morse code back to text.

```ts
function decode(morse: string, options?: DecodeOptions): string
```

**Examples:**

```ts
import { decode } from '@morsecodeapp/morse';

decode('... --- ...');                          // 'SOS'
decode('.... . .-.. .-.. --- / .-- --- .-. .-.. -..');  // 'HELLO WORLD'
decode('.-', { charset: 'cyrillic' });          // 'А'
decode('••• ——— •••', { dot: '•', dash: '—' }); // 'SOS'
```

### DecodeOptions

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `charset` | `CharsetId` | `'itu'` | Character set to use |
| `fallbackCharsets` | `CharsetId[]` | `[]` | Additional charsets to try for unmatched patterns |
| `dot` | `string` | `'.'` | Dot symbol in input |
| `dash` | `string` | `'-'` | Dash symbol in input |
| `separator` | `string` | `' '` | Separator between letters |
| `wordSeparator` | `string` | `' / '` | Separator between words |
| `invalid` | `string` | `'?'` | Replacement for unrecognized patterns |

---

## decodeDetailed()

Like `decode()` but returns a `DecodeResult` with validity info.

```ts
function decodeDetailed(morse: string, options?: DecodeOptions): DecodeResult
```

**Example:**

```ts
import { decodeDetailed } from '@morsecodeapp/morse';

decodeDetailed('... --- ... .-.-.-');
// { text: 'SOS.', valid: true, errors: [] }

decodeDetailed('... @@@ ...');
// { text: 'S?S', valid: false, errors: ['@@@'] }
```

### DecodeResult

| Field | Type | Description |
|-------|------|-------------|
| `text` | `string` | The decoded text |
| `valid` | `boolean` | `true` if all patterns were mapped |
| `errors` | `string[]` | Morse patterns that could not be decoded |

---

## Charsets

### getCharset()

Returns a `Charset` object by ID.

```ts
function getCharset(id?: CharsetId): Charset
```

```ts
import { getCharset } from '@morsecodeapp/morse';

const itu = getCharset('itu');
itu.charToMorse['A'];  // '.-'
itu.morseToChar['.-'];  // 'A'
itu.name;               // 'International (ITU)'
```

Throws an `Error` if the charset ID is unknown.

### listCharsets()

Returns an array of all available charset IDs.

```ts
function listCharsets(): CharsetId[]
```

```ts
listCharsets();
// ['itu', 'american', 'latin-ext', 'cyrillic', 'greek', 'hebrew',
//  'arabic', 'persian', 'japanese', 'korean', 'thai']
```

### listCharsetsDetailed()

Returns charset metadata including character count.

```ts
function listCharsetsDetailed(): Array<{ id: CharsetId; name: string; size: number }>
```

```ts
listCharsetsDetailed();
// [{ id: 'itu', name: 'International (ITU)', size: 54 }, ...]
```

### detectCharset()

Auto-detects the best charset for the given text by coverage score.

```ts
function detectCharset(text: string): CharsetId
```

```ts
detectCharset('ПРИВЕТ'); // 'cyrillic'
detectCharset('HELLO');  // 'itu'
detectCharset('こんにちは'); // 'japanese'
```

### Direct charset imports

Individual charsets are also exported directly:

```ts
import { itu, cyrillic, greek, arabic } from '@morsecodeapp/morse';

itu.charToMorse['S']; // '...'
cyrillic.charToMorse['Ж']; // '...-'
```

Available: `itu`, `american`, `latinExt`, `cyrillic`, `greek`, `hebrew`, `arabic`, `persian`, `japanese`, `korean`, `thai`

### Charset interface

```ts
interface Charset {
  readonly id: CharsetId;
  readonly name: string;
  readonly charToMorse: Readonly<Record<string, string>>;
  readonly morseToChar: Readonly<Record<string, string>>;
}
```

### CharsetId

```ts
type CharsetId =
  | 'itu' | 'american' | 'latin-ext' | 'cyrillic' | 'greek'
  | 'hebrew' | 'arabic' | 'persian' | 'japanese' | 'korean' | 'thai';
```

---

## Prosigns

Procedural signals — sent as single unbroken characters with no inter-character gaps.

### PROSIGNS

A `readonly Prosign[]` array of all 10 standard prosigns.

```ts
import { PROSIGNS } from '@morsecodeapp/morse';

PROSIGNS[0]; // { label: 'SOS', morse: '...---...', meaning: 'International distress signal' }
```

| Label | Morse | Meaning |
|-------|-------|---------|
| SOS | `...---...` | International distress signal |
| AR | `.-.-.` | End of message |
| SK | `...-.-` | End of contact / Silent Key |
| BT | `-...-` | Break / New paragraph |
| KN | `-.--. ` | Go ahead, named station only |
| AS | `.-...` | Wait / Stand by |
| CL | `-.-..-.." ` | Closing station |
| CT | `-.-.-` | Commence transmission |
| SN | `...-.` | Understood / Verified |
| HH | `........` | Error / Correction |

### encodeProsign()

Returns the morse pattern for a prosign label, or `undefined`.

```ts
function encodeProsign(label: string): string | undefined
```

```ts
encodeProsign('SOS'); // '...---...'
encodeProsign('AR');  // '.-.-.'
encodeProsign('XYZ'); // undefined
```

### decodeProsign()

Returns the prosign label for a morse pattern, or `undefined`.

```ts
function decodeProsign(morse: string): string | undefined
```

```ts
decodeProsign('...---...'); // 'SOS'
decodeProsign('.-.-.');     // 'AR'
```

### getProsign()

Returns the full `Prosign` object by label, or `undefined`.

```ts
function getProsign(label: string): Prosign | undefined
```

```ts
getProsign('SOS');
// { label: 'SOS', morse: '...---...', meaning: 'International distress signal' }
```

### listProsigns()

Returns an array of all prosign labels.

```ts
function listProsigns(): string[]
```

```ts
listProsigns(); // ['SOS', 'AR', 'SK', 'BT', 'KN', 'AS', 'CL', 'CT', 'SN', 'HH']
```

---

## Timing

Based on the PARIS standard — the word PARIS is 50 dot-units, so at W WPM: `unit = 1200 / W` ms.

### timing()

Calculate standard PARIS timing values.

```ts
function timing(wpm?: number): TimingValues
```

Default WPM is 20. Clamped to 1–60.

```ts
import { timing } from '@morsecodeapp/morse';

timing(20);
// { unit: 60, dot: 60, dash: 180, intraChar: 60, interChar: 180, interWord: 420 }

timing(5);
// { unit: 240, dot: 240, dash: 720, intraChar: 240, interChar: 720, interWord: 1680 }
```

### farnsworthTiming()

Characters sent at `charWpm`, with extra gaps to achieve `overallWpm`.

```ts
function farnsworthTiming(overallWpm?: number, charWpm?: number): TimingValues
```

Defaults: `overallWpm = 15`, `charWpm = 20`.

```ts
import { farnsworthTiming } from '@morsecodeapp/morse';

farnsworthTiming(10, 20);
// Characters at 20 WPM speed, but overall pace is 10 WPM
// Extra delay added to interChar and interWord gaps
```

### duration()

Calculate total playback duration of a morse string in milliseconds.

```ts
function duration(morse: string, wpm?: number): number
```

```ts
import { duration } from '@morsecodeapp/morse';

duration('... --- ...');     // 1620 (ms at 20 WPM)
duration('... --- ...', 10); // 3240 (ms at 10 WPM)
```

### formatDuration()

Format milliseconds to a human-readable string.

```ts
function formatDuration(ms: number): string
```

```ts
import { formatDuration } from '@morsecodeapp/morse';

formatDuration(500);   // '500ms'
formatDuration(3500);  // '3.5s'
formatDuration(65000); // '1m 5.0s'
```

### Constants

```ts
import { DEFAULT_WPM, MIN_WPM, MAX_WPM } from '@morsecodeapp/morse';

DEFAULT_WPM; // 20
MIN_WPM;     // 1
MAX_WPM;     // 60
```

### TimingValues

```ts
interface TimingValues {
  unit: number;      // Base unit in ms
  dot: number;       // 1 unit
  dash: number;      // 3 units
  intraChar: number; // Gap between signals within a character (1 unit)
  interChar: number; // Gap between characters (3 units, extended for Farnsworth)
  interWord: number; // Gap between words (7 units, extended for Farnsworth)
}
```

---

## Stats

### stats()

Compute statistics for a morse code string.

```ts
function stats(morse: string, wpm?: number): MorseStats
```

```ts
import { stats } from '@morsecodeapp/morse';

stats('... --- ...');
// {
//   dots: 6,
//   dashes: 3,
//   signals: 9,
//   characters: 3,
//   words: 1,
//   durationMs: 1620,
//   durationSec: '1.6',
//   durationFormatted: '1.6s'
// }

stats('.... . .-.. .-.. --- / .-- --- .-. .-.. -..', 15);
// Statistics at 15 WPM
```

### MorseStats

```ts
interface MorseStats {
  dots: number;             // Count of dots
  dashes: number;           // Count of dashes
  signals: number;          // dots + dashes
  characters: number;       // Number of letters/numbers
  words: number;            // Number of words
  durationMs: number;       // Estimated duration in milliseconds
  durationSec: string;      // Duration in seconds (1 decimal, e.g., '1.6')
  durationFormatted: string; // Human-readable (e.g., '1.6s', '1m 5.0s')
}
```

---

## Validation

### isValidMorse()

Check if a string contains only valid Morse characters (dots, dashes, spaces, slashes).

```ts
function isValidMorse(morse: string): boolean
```

```ts
isValidMorse('... --- ...');  // true
isValidMorse('HELLO');        // false
isValidMorse('... @@@ ...');  // false
```

### isEncodable()

Check if all characters in text have a mapping in the given charset.

```ts
function isEncodable(text: string, charset?: CharsetId): boolean
```

```ts
isEncodable('HELLO');              // true (defaults to 'itu')
isEncodable('HELLO', 'itu');       // true
isEncodable('Ж', 'itu');           // false
isEncodable('Ж', 'cyrillic');      // true
```

### isDecodable()

Check if all morse patterns can be decoded with the given charset.

```ts
function isDecodable(morse: string, charset?: CharsetId): boolean
```

```ts
isDecodable('... --- ...');       // true
isDecodable('... --- ...', 'itu'); // true
isDecodable('HELLO');              // false (not valid morse)
```

### findInvalidChars()

Find characters in text that cannot be encoded with the given charset.

```ts
function findInvalidChars(text: string, charset?: CharsetId): string[]
```

```ts
findInvalidChars('Hello!§');  // ['§']
findInvalidChars('ABC');      // []
```

### findInvalidPatterns()

Find morse patterns that cannot be decoded with the given charset.

```ts
function findInvalidPatterns(morse: string, charset?: CharsetId): string[]
```

```ts
findInvalidPatterns('... --- .---.--.');  // ['.---.--.' ] (not a real pattern)
findInvalidPatterns('... --- ...');       // []
```

---

## Types

All core types are exported and available for import:

```ts
import type {
  CharsetId,
  Charset,
  EncodeOptions,
  DecodeOptions,
  EncodeResult,
  DecodeResult,
  TimingValues,
  Prosign,
  MorseStats,
} from '@morsecodeapp/morse';
```

See also [Audio Types](#audio-types) for audio-specific types.

---

# Audio

> All audio exports are available from `@morsecodeapp/morse` and `@morsecodeapp/morse/audio`.
>
> `MorsePlayer` requires a browser with the Web Audio API.
> WAV export and the scheduler work in any JavaScript runtime.

---

## MorsePlayer

Web Audio API morse code player with play/pause/stop, gain envelope, and event callbacks.

```ts
import { MorsePlayer } from '@morsecodeapp/morse/audio';
```

### Constructor

```ts
new MorsePlayer(options?: MorsePlayerOptions)
```

**Example:**

```ts
const player = new MorsePlayer({
  wpm: 20,
  frequency: 600,
  waveform: 'sine',
  volume: 80,
  onEnd: () => console.log('Done!'),
});
```

### MorsePlayerOptions

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `wpm` | `number` | `20` | Words per minute (1–60) |
| `frequency` | `number` | `600` | Tone frequency in Hz (200–2000) |
| `waveform` | `WaveformType` | `'sine'` | Oscillator waveform |
| `volume` | `number` | `80` | Volume (0–100) |
| `farnsworth` | `boolean` | `false` | Enable Farnsworth spacing |
| `farnsworthWpm` | `number` | `15` | Farnsworth overall WPM |
| `gainEnvelope` | `GainEnvelopeOptions` | `{ attack: 0.01, release: 0.01 }` | Gain envelope for click-free audio |
| `audioContext` | `AudioContext` | auto-created | Existing AudioContext to reuse |
| `onPlay` | `() => void` | — | Fired when playback starts |
| `onPause` | `() => void` | — | Fired when playback is paused |
| `onResume` | `() => void` | — | Fired when playback resumes |
| `onStop` | `() => void` | — | Fired when playback is stopped |
| `onEnd` | `() => void` | — | Fired when playback ends naturally |
| `onSignal` | `(signal: 'dot' \| 'dash', charIndex: number) => void` | — | Fired for each dot or dash |
| `onCharacter` | `(char: string, morse: string, charIndex: number) => void` | — | Fired when a character finishes |
| `onProgress` | `(currentMs: number, totalMs: number) => void` | — | Fired periodically during playback |

### play()

Play morse code audio. Accepts text (auto-encodes) or raw morse.

```ts
async play(input: string, options?: PlayOptions): Promise<void>
```

Returns a Promise that resolves when playback ends or is stopped.

```ts
await player.play('Hello World');
await player.play('... --- ...', { morse: true });
await player.play('ПРИВЕТ', { charset: 'cyrillic' });
```

#### PlayOptions

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `morse` | `boolean` | `false` | If `true`, input is treated as raw morse code |
| `charset` | `CharsetId` | `'itu'` | Character set for encoding text input |

### pause()

Pause playback. Suspends the AudioContext.

```ts
player.pause();
```

### resume()

Resume playback from paused state.

```ts
await player.resume();
```

### stop()

Stop playback and reset to idle.

```ts
player.stop();
```

### dispose()

Dispose of all resources. Call when done with the player.

```ts
player.dispose();
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `state` | `PlayerState` | Current state: `'idle'`, `'playing'`, or `'paused'` |
| `totalTime` | `number` | Total duration of current playback in ms |
| `currentTime` | `number` | Elapsed time in ms |
| `progress` | `number` | Playback progress (0–1) |
| `wpm` | `number` | Get/set words per minute |
| `frequency` | `number` | Get/set tone frequency in Hz |
| `volume` | `number` | Get/set volume (0–100, live update during playback) |

---

## Sound Presets

Pre-configured audio settings tuned for different use cases.

```ts
import { presets, telegraph, radio } from '@morsecodeapp/morse/audio';
```

### Using a preset

Pass a preset directly to the MorsePlayer constructor:

```ts
import { MorsePlayer, presets } from '@morsecodeapp/morse/audio';

const player = new MorsePlayer(presets.telegraph);
await player.play('CQ CQ CQ');
```

Or spread a preset with overrides:

```ts
const player = new MorsePlayer({ ...presets.military, volume: 60 });
```

### Available Presets

| Name | Freq | WPM | Waveform | Character |
|------|------|-----|----------|-----------|
| `telegraph` | 550 Hz | 15 | square | Classic telegraph sounder — warm, clicky tone |
| `radio` | 600 Hz | 20 | sine | Clean amateur radio CW tone |
| `military` | 700 Hz | 25 | sine | Crisp military communication tone |
| `sonar` | 400 Hz | 12 | sine | Deep submarine sonar ping |
| `naval` | 650 Hz | 18 | triangle | Naval fleet communication tone |
| `beginner` | 600 Hz | 18 (Farnsworth 5) | sine | Slow Farnsworth spacing for learning |

### SoundPreset

```ts
interface SoundPreset {
  readonly name: string;
  readonly description: string;
  readonly wpm: number;
  readonly frequency: number;
  readonly waveform: WaveformType;
  readonly volume: number;
  readonly farnsworth?: boolean;
  readonly farnsworthWpm?: number;
  readonly gainEnvelope?: GainEnvelopeOptions;
}
```

### PresetName

```ts
type PresetName = 'telegraph' | 'radio' | 'military' | 'sonar' | 'naval' | 'beginner';
```

---

## WAV Export

Generate WAV audio files from text or morse code. Pure computation — works in any JavaScript runtime (Node.js, Bun, Deno, browsers).

```ts
import { toWav, toWavBlob, toWavUrl, downloadWav } from '@morsecodeapp/morse/audio';
```

### toWav()

Generate WAV audio data as raw bytes.

```ts
function toWav(input: string, options?: WavOptions): Uint8Array
```

```ts
import { toWav } from '@morsecodeapp/morse/audio';

const wav = toWav('SOS');
const wav = toWav('... --- ...', { morse: true, frequency: 800 });

// Write to file in Node.js
import { writeFileSync } from 'fs';
writeFileSync('sos.wav', wav);
```

### toWavBlob()

Generate a WAV Blob. Useful for creating audio elements in the browser.

```ts
function toWavBlob(input: string, options?: WavOptions): Blob
```

```ts
const blob = toWavBlob('SOS');
const audio = new Audio(URL.createObjectURL(blob));
audio.play();
```

### toWavUrl()

Generate a base64 data URL of the WAV file.

```ts
function toWavUrl(input: string, options?: WavOptions): string
```

```ts
const url = toWavUrl('SOS');
// 'data:audio/wav;base64,...'
```

### downloadWav()

Download a WAV file in the browser. No-op in non-browser environments.

```ts
function downloadWav(input: string, options?: WavOptions): void
```

```ts
downloadWav('SOS', { filename: 'sos.wav' });
```

### WavOptions

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `wpm` | `number` | `20` | Words per minute (1–60) |
| `frequency` | `number` | `600` | Tone frequency in Hz (200–2000) |
| `waveform` | `WaveformType` | `'sine'` | Oscillator waveform |
| `volume` | `number` | `80` | Volume (0–100) |
| `sampleRate` | `number` | `44100` | Sample rate in Hz |
| `gainEnvelope` | `GainEnvelopeOptions` | `{ attack: 0.01, release: 0.01 }` | Gain envelope |
| `farnsworth` | `boolean` | `false` | Enable Farnsworth spacing |
| `farnsworthWpm` | `number` | — | Farnsworth overall WPM |
| `charset` | `CharsetId` | `'itu'` | Character set for text input |
| `morse` | `boolean` | `false` | If `true`, input is raw morse code |
| `filename` | `string` | `'morse.wav'` | Filename for `downloadWav()` |

---

## Scheduler

Low-level module that converts a morse string and timing values into a timeline of timed events. Used internally by `MorsePlayer` and WAV export, but exposed for custom audio rendering.

```ts
import { buildSchedule, scheduleDuration, type ScheduleEvent } from '@morsecodeapp/morse/audio';
```

### buildSchedule()

Build a schedule of tones and silences from a morse string.

```ts
function buildSchedule(morse: string, timings: TimingValues): ScheduleEvent[]
```

```ts
import { buildSchedule } from '@morsecodeapp/morse/audio';
import { timing } from '@morsecodeapp/morse/core';

const t = timing(20);
const schedule = buildSchedule('... ---', t);
// [
//   { type: 'tone', start: 0, duration: 60, signal: 'dot', morseChar: '...', charIndex: 0 },
//   { type: 'silence', start: 60, duration: 60 },
//   { type: 'tone', start: 120, duration: 60, signal: 'dot', morseChar: '...', charIndex: 0 },
//   ...
// ]
```

### scheduleDuration()

Get total duration of a schedule in milliseconds.

```ts
function scheduleDuration(events: ScheduleEvent[]): number
```

```ts
const total = scheduleDuration(schedule); // e.g. 1620
```

### ScheduleEvent

```ts
interface ScheduleEvent {
  type: 'tone' | 'silence';
  start: number;       // Start time in ms from beginning
  duration: number;    // Duration in ms
  signal?: 'dot' | 'dash';     // Tone events only
  morseChar?: string;           // Morse pattern (e.g., '.-')
  charIndex?: number;           // Sequential character index
}
```

---

## Audio Types

All audio types are exported from `@morsecodeapp/morse/audio`:

```ts
import type {
  WaveformType,
  PlayerState,
  GainEnvelopeOptions,
  MorsePlayerOptions,
  PlayOptions,
  SoundPreset,
  WavOptions,
  PresetName,
  ScheduleEvent,
} from '@morsecodeapp/morse/audio';
```

### WaveformType

```ts
type WaveformType = 'sine' | 'square' | 'sawtooth' | 'triangle';
```

### PlayerState

```ts
type PlayerState = 'idle' | 'playing' | 'paused';
```

### GainEnvelopeOptions

```ts
interface GainEnvelopeOptions {
  attack: number;  // Attack time in seconds (ramp up). Default: 0.01
  release: number; // Release time in seconds (ramp down). Default: 0.01
}
```
