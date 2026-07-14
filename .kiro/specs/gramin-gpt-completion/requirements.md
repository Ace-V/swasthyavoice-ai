# Requirements Document

## Introduction

This feature completes the Swasthya-Voice backend API — a voice-based AI healthcare assistant for rural India. The existing backend handles text queries and location-aware health center lookups but is missing a speech-to-text (STT) pipeline, the voice endpoint that ties it together, proper dependency management, and developer onboarding tooling. This spec covers all six completion items needed to make the project production-ready.

## Glossary

- **STT_Module**: The `stt.py` Python module responsible for transcribing raw WAV audio bytes to Hindi text using the Vosk model.
- **Vosk_Model**: The pre-trained offline Hindi speech recognition model located at `models/vosk-model-small-hi-0.22/`.
- **Server**: The FastAPI application defined in `server.py`.
- **LLM_Module**: The `llm.py` module that queries the Sarvam AI language model and returns a Hindi response.
- **Voice_Endpoint**: The `/ask-voice/` POST endpoint that accepts an audio file upload and returns an AI-generated answer.
- **Requirements_File**: The `requirements.txt` file that lists all Python package dependencies for the project.
- **Env_Template**: The `.env.example` file that documents all required environment variables for local setup.
- **Developer**: A person setting up or contributing to the Swasthya-Voice project.

---

## Requirements

### Requirement 1: Speech-to-Text Transcription Module

**User Story:** As a developer, I want a self-contained STT module, so that raw Hindi audio can be transcribed to text and passed downstream to the LLM.

#### Acceptance Criteria

1. THE STT_Module SHALL expose a function `transcribe(audio_bytes: bytes) -> str` that accepts raw WAV audio bytes and returns a Hindi transcription string.
2. WHEN `transcribe` is called, THE STT_Module SHALL load the Vosk_Model from the path `models/vosk-model-small-hi-0.22/` relative to the project root.
3. WHEN `transcribe` is called with valid WAV audio bytes, THE STT_Module SHALL return the recognised Hindi text as a non-empty string.
4. IF the provided audio bytes cannot be decoded as valid WAV data, THEN THE STT_Module SHALL raise a `ValueError` with a descriptive message.
5. IF the Vosk_Model directory does not exist at the expected path, THEN THE STT_Module SHALL raise a `FileNotFoundError` with a descriptive message.
6. WHEN `transcribe` is called with audio that contains no recognisable speech, THE STT_Module SHALL return an empty string.
7. THE STT_Module SHALL suppress Vosk library log output so that it does not pollute application logs.
8. FOR ALL valid WAV inputs, transcribing the same audio bytes twice SHALL produce identical output strings (idempotence property).

---

### Requirement 2: Voice Query Endpoint

**User Story:** As a mobile client, I want to POST a raw audio file and receive an AI-generated Hindi health answer, so that voice-first users can interact with the assistant without typing.

#### Acceptance Criteria

1. THE Server SHALL expose a `POST /ask-voice/` endpoint that accepts a multipart file upload with field name `audio_file`.
2. WHEN a valid WAV audio file is uploaded to `/ask-voice/`, THE Server SHALL pass the file bytes to the STT_Module for transcription.
3. WHEN a transcript is produced by the STT_Module, THE Server SHALL pass that transcript to the LLM_Module's `query_llm` function.
4. WHEN the LLM_Module returns a response, THE Server SHALL return a JSON body `{"transcript": "<hindi text>", "answer": "<ai response>"}` with HTTP status 200.
5. IF the uploaded file cannot be parsed as valid WAV audio, THEN THE Server SHALL return HTTP status 422 with a JSON error detail.
6. IF the STT_Module returns an empty transcript, THEN THE Server SHALL return HTTP status 422 with a JSON error detail indicating that no speech was detected.
7. IF the LLM_Module fails to return a response, THEN THE Server SHALL return HTTP status 500 with a JSON error detail.
8. THE Server SHALL accept audio files up to 10 MB in size at the `/ask-voice/` endpoint.

---

### Requirement 3: Vosk Dependency Registration

**User Story:** As a developer, I want the `vosk` library listed in `requirements.txt`, so that `pip install -r requirements.txt` installs everything needed to run STT without manual steps.

#### Acceptance Criteria

1. THE Requirements_File SHALL include `vosk` as a listed dependency with a pinned version.
2. WHEN `pip install -r requirements.txt` is executed, THE Requirements_File SHALL result in the `vosk` package being installed in the environment.
3. THE Requirements_File SHALL not duplicate any existing package entry.

---

### Requirement 4: Dependency Version Pinning

**User Story:** As a developer deploying the application, I want all dependencies pinned to specific versions, so that builds are reproducible and do not break due to upstream changes.

#### Acceptance Criteria

1. THE Requirements_File SHALL specify an exact version for every listed package using the `==` operator.
2. THE Requirements_File SHALL include `vosk`, `fastapi`, `uvicorn[standard]`, `python-dotenv`, `openai`, and `requests` with pinned versions.
3. WHEN the pinned versions are installed, THE Requirements_File SHALL result in a working server that passes all existing endpoint tests.

---

### Requirement 5: Removal of Committed Test Artefact

**User Story:** As a maintainer, I want `response.mp3` removed from the repository, so that the repo does not contain leftover test files that could mislead contributors.

#### Acceptance Criteria

1. THE Repository SHALL not contain the file `response.mp3` after this change is applied.
2. WHERE a `.gitignore` file exists, THE Repository SHALL include `*.mp3` in `.gitignore` to prevent future accidental commits of audio test files.

---

### Requirement 6: Environment Variable Template

**User Story:** As a new developer, I want a `.env.example` file in the project root, so that I know exactly which environment variables to configure before running the server.

#### Acceptance Criteria

1. THE Env_Template SHALL exist as `.env.example` in the project root directory.
2. THE Env_Template SHALL document all environment variables required to run the Server, with placeholder values.
3. THE Env_Template SHALL include `SARVAM_API_KEY` with a placeholder value indicating where to obtain the key.
4. THE Env_Template SHALL not contain any real secret values or API keys.
5. WHEN a Developer copies `.env.example` to `.env` and fills in real values, THE Server SHALL start successfully and all endpoints SHALL be reachable.
