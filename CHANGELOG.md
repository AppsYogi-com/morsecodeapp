# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2025-07-13

### Added

- **Audio playback** — `MorsePlayer` class powered by the Web Audio API with play/pause/resume/stop
- **WAV export** — `toWav()`, `toWavBlob()`, `toWavUrl()`, `downloadWav()` (44.1 kHz, 16-bit PCM)
- **6 sound presets** — `telegraph`, `radio`, `military`, `sonar`, `naval`, `beginner`
- **Gain envelope** — click-free audio with configurable attack/release ramps
- **Event callbacks** — `onPlay`, `onPause`, `onResume`, `onStop`, `onEnd`, `onSignal`, `onCharacter`, `onProgress`
- **Scheduler** — `buildSchedule()`, `scheduleDuration()` for timed tone/silence events
- **Sub-path export** — `@morsecodeapp/morse/audio` for tree-shakeable audio-only imports
- **191 tests** across 12 files; 99%+ statement coverage

## [0.1.0] - 2025-03-27

### Added

- **Core encode/decode** — `encode()`, `decode()`, `encodeDetailed()`, `decodeDetailed()`
- **11 character sets** — ITU, American, Latin Extended, Cyrillic, Greek, Hebrew, Arabic, Persian, Japanese (Wabun), Korean (SKATS), Thai
- **Charset registry** — `getCharset()`, `listCharsets()`, `listCharsetsDetailed()`, `detectCharset()`
- **10 prosigns** — SOS, AR, SK, BT, KN, AS, CL, CT, SN, HH with `encodeProsign()`, `decodeProsign()`
- **PARIS timing** — `timing()`, `farnsworthTiming()`, `duration()`, `formatDuration()`
- **Statistics** — `stats()` for dot/dash counts, duration, word/character counts
- **Validation** — `isValidMorse()`, `isEncodable()`, `isDecodable()`, `findInvalidChars()`, `findInvalidPatterns()`
- **Tree-shakeable** — sub-path export `@morsecodeapp/morse/core`
- **Dual format** — ESM + CJS + TypeScript declarations
- **Full test suite** — 79 tests, 99%+ coverage
