<!-- Added by HABS for the hackathon. Everything below the rule is the
     dataset's own documentation, unchanged. -->

## Where it is

The dataset is mounted read-only at `/data` inside your job, and `/data` **is**
the `DATASET/` root described below — so `labels.csv` is at `/data/labels.csv`
and the first participant is at `/data/CURATED/P001/`.

```python
def process(data_dir: Path, out_dir: Path) -> None:
    labels = pd.read_csv(data_dir / "labels.csv")
    bdf = data_dir / "CURATED" / "P001" / "P001_day-1_task-resting-state_multimodal.bdf"
```

The supplied loader comes with the data, at `/data/scripts/bdf_loader.py`:

```python
import sys; sys.path.insert(0, "/data/scripts")
from bdf_loader import load_bdf
raw, data_df, channels = load_bdf(bdf)
```

It needs `pyedflib` as well as `mne`, so both belong in your `requirements.txt`.

`labels.csv` covers 478 participants; 384 have recordings in `CURATED`. Join
on `participant_id` and check what you actually got, rather than assuming a
row has a recording behind it.

**It is 385 GB.** Read what you need and nothing else: a resting-state
recording is about 20 MB, an artefact block about 45 MB, and an
emotion-video recording 150–280 MB. Walking every BDF is 171 GB of reading
before your code does anything. `load_bdf` also returns the whole recording as
a DataFrame, which for a 22-minute emotion-video file is over a gigabyte of
memory — take the channels and the window you want.

---

# Dataset guide

This dataset contains synchronized brain and physiological recordings collected during video viewing, resting state and an artefact calibration task. It also includes participant information, event metadata and emotion annotations.

Participants viewed videos selected in part using an image-based questionnaire about their emotional associations. After viewing a video, they rated their response and, where available, selected up to three emotion categories. The video files themselves are not provided.

## Dataset organization

```text
DATASET/
├── CURATED/                 # Unprocessed recordings
│   ├── P001/
│   ├── P002/
│   └── ...
├── PREPROCESSED/            # Cleaned and preprocessed data
├── scripts/                 # Supplied scripts, including loading code
├── labels.csv               # Video-trial annotations
├── markers.csv              # Marker-code lookup
├── participants.csv         # Participant-level information
└── README.md
```

`CURATED` contains a separate folder for each participant. Each folder contains recordings and metadata for available recording days. The names use the participant ID, day and task:

```text
CURATED/P005/
├── P005_day-1_metadata.json
├── P005_day-1_task-emotion-video_multimodal.bdf
├── P005_day-1_task-emotion-video_multimodal.record_map
├── P005_day-1_task-resting-state_multimodal.bdf
├── P005_day-1_task-resting-state_multimodal.record_map
├── P005_day-1_task-artefact_multimodal.bdf
├── P005_day-1_task-artefact_multimodal.record_map
├── P005_day-2_metadata.json
└── ...
```

Each `.bdf` file contains a multimodal recording for one task segment. Its matching `.record_map` provides channel and modality information used by the supplied loading code. Each day's `.json` file contains session events and information about the exported segments. Keep each BDF file together with its matching record map.

The available tasks are:

| Task | Description |
|---|---|
| `emotion-video` | Video viewing and associated trial activity. |
| `resting-state` | Resting-state recording. |
| `artefact` | Artefact calibration block, including instructed actions. |

Available days and task segments differ between participants. Released segments cover selected parts of a recording session; some other tasks are not included.

## Signals

The recordings combine 64-channel electroencephalography (EEG) with electrocardiography (ECG), skin conductance, respiratory activity, gaze and pupil size. An event status channel is also described in the manuscript. Use the record map and loading code to identify the channels, units and sampling rates in each file.

EEG and the additional physiological signals were acquired at 1,000 Hz. Eye-tracking data were acquired at approximately 200 Hz and upsampled to 1,000 Hz for integration with the other modalities. Upsampling does not increase the original temporal resolution. Check the channel metadata and processing information for the specific release before interpreting the signals.

`PREPROCESSED` contains cleaned and preprocessed data. Consult its accompanying documentation or scripts for the processing steps and file organization.

## Session metadata and event timing

Each session metadata file contains an `events` list and a `dataset_export` section. The event field `onset_s` is measured in seconds relative to the canonical full session. To locate an event in a task segment, subtract that segment's `metadata_time_origin_s`:

```text
segment-relative onset = event onset_s − segment metadata_time_origin_s
```

For example, an event at 1,533.051 s in a segment whose `metadata_time_origin_s` is 369.510 s starts at 1,163.541 s within that segment. Check that the event falls within the relevant segment before extracting it.

| Metadata fields | Description |
|---|---|
| `marker_id`, `onset_s`, `duration_s` | Event marker, full-session onset and event duration. |
| `onset_source`, `duration_source`, `trial_type_source` | Sources used to determine the event timing or type. |
| `trial_type`, `media_file`, `instruction_text` | Event type, video filename where applicable, and instruction or event text. |
| `emotion_label`, `rating_tool`, `valence_score`, `arousal_score` | Event annotations where available; values may be null. |
| `dataset_export.participant_id`, `day`, `protocol`, `source` | Export identifiers and provenance. |
| `segments.<task>.file`, `record_map` | Filenames of the corresponding task recording and record map. |
| `metadata_time_origin_s`, `source_time_origin_s`, `source_to_metadata_offset_s` | Segment origins and alignment information. |
| `requested_start_s`, `requested_end_s`, `source_start_s`, `source_end_s` | Requested and source segment boundaries. |
| `duration_s`, `n_samples`, `alignment_max_residual_s` | Segment duration, sample count and alignment residual. |
| `event_indices`, `partial_event_indices` | Event-index lists; their availability and indexing should be checked in the release documentation. |

The metadata annotations and the fields in `labels.csv` may represent different sources. Check their provenance before treating them as interchangeable. `markers.csv` is a code lookup; it does not provide participant-specific event times.

## Trial annotations: `labels.csv`

Each row describes a video trial. Participant ID and day identify the session, and the event marker helps identify the trial. The same video can occur more than once for a participant, so the filename alone is not a unique trial key. Verify the identifiers and marker correspondence against the session metadata for the release being used.

Empty fields indicate unavailable values, not zero. Rating and self-report availability varies across participants, sessions and cohorts.

| Field | Description |
|---|---|
| `cohort` | Acquisition cohort. |
| `participant_id` | Participant identifier. |
| `gender` | Recorded participant gender, where available. |
| `day` | Recording-session number. |
| `video` | Filename or identifier of the presented video. |
| `start_marker`, `end_marker` | Event codes associated with video onset and offset. |
| `rating_valence` | Reported pleasantness, on a 1–9 scale. |
| `rating_arousal` | Reported activation, on a 1–9 scale. |
| `rating_dominance` | Reported perceived control, on a 1–9 scale where collected. |
| `self_reported_emotion_1`, `_2`, `_3` | Up to three participant-selected emotion categories, ranked by relevance, where available. |
| `default_emotion_label` | Predefined emotion associated with the stimulus. |
| `expected_emotion_label` | Emotion assigned for the participant and video using the individualized selection or fallback mapping. |
| `expected_emotion_source` | Source of the expected label: `emotinder` for personalized selection or `default` for fallback mapping. |
| `final_emotion_label` | Derived label combining several annotation sources. |
| `video_theme` | English-language theme associated with the video. |
| `video_presentation_order` | Presentation position where available; conventions may vary by cohort. |
| `flagged_outliers` | Combined screening indicator: `1` flags a trial for review; `0` does not guarantee that all inputs were available. |

The 12 emotion categories are anger, fear, disgust, sadness, joy, surprise, excitement, relaxation, guilt, sensuality, curiosity and neutral. The final label is produced using the default and expected labels, available self-reports, and information derived from valence–arousal ratings. It may differ from the participant's first-ranked self-report and should not be interpreted as a direct measurement of subjective experience. See the dataset paper for the label-refinement method.

A flagged trial is not necessarily incorrect or inattentive. Choose how to handle flagged and missing observations according to the research question and document that choice.

## Marker lookup: `markers.csv`

| Field | Description |
|---|---|
| `Emotion or task` | Emotion category or task instruction associated with the marker pair. |
| `Start Marker` | Start marker code. |
| `End Marker` | Corresponding end marker code. |
| `Repetition` | Repetition index in the marker mapping. |

Use the session metadata to find event times. Verify that marker codes and their corresponding events agree across the lookup, trial table and recording for the release being analyzed.

## Participant information: `participants.csv`

The participant table contains one row per participant ID, with cohort, demographic and questionnaire information where available. Questionnaire fields can include self-consciousness, empathy and personality scores, as well as session-specific anxiety and wellbeing measures. Coverage varies; consult the column names and release documentation for the available fields, questionnaire versions and scoring.

Join participant information to trials using `participant_id`, after checking that identifiers are consistent across the supplied files. Some people may have participated in more than one cohort under different IDs; consult the release documentation before treating IDs as guaranteed unique individuals.

## Using the data

Start with the supplied scripts to load a BDF and its matching record map. Use the session metadata to locate events within the relevant segment, then connect trials to `labels.csv` using the participant, day and verified marker. Inspect missing values, available modalities and quality flags before analysis.

When evaluating models, choose train/test splits that match the intended question. Recordings can share videos, and repeated sessions or participant IDs can introduce dependencies. A model evaluated on videos also present in training may recognize video-specific patterns, so its accuracy alone does not establish that it decodes emotion.
