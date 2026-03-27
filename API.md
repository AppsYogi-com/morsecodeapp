# API Reference

Full documentation for `@morsecodeapp/morse` v0.1.0.

All exports are available from both `@morsecodeapp/morse` and `@morsecodeapp/morse/core`.

---

## Table of Contents

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

All types are exported and available for import:

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
