(https://github.com/user-attachments/files/34438231)
# Synthetic Test Corpus \& Evidence-Recovery Measurement

Analysis notebook for the mini-thesis **"Anti-Forensic Techniques in Encrypted Mobile Applications: Case Study of WhatsApp and Telegram"**.

* **Researcher:** Gerhard Uundjondjo (226111989)
* **Supervisor:** Mr Isaac Nhamu
* **Institution:** Department of Cyber Security, Faculty of Computing and Informatics, NUST
* **Notebook:** `Thesis\_Synthetic\_Corpus.ipynb` (final version)

\---

## What the notebook does

The notebook measures how much evidence WhatsApp's and Telegram's anti-forensic features remove from an Android phone. It does this in two halves.

**1. Before the experiment (Sections 0–9)**

* Builds a synthetic test corpus of 258 items (text, images, PDF documents and voice-note scripts). Each item carries a unique marker token, for example `AFXW0001QPP`.
* Writes the ground-truth manifest, data dictionary, provenance log and seeding run sheets.

**2. After the experiment (Sections 10–11)**

* Scans the logical ADB acquisitions taken before (T0) and after (T1) each anti-forensic action.
* Matches every recovered file back to its seeded item.
* Produces the recovery tables and figures reported in Chapter 4.

All corpus content is fictional. No personal data is used to build the corpus.

## Requirements

|Item|Version used|
|-|-|
|Python|3.14|
|numpy, pandas, matplotlib|any recent version|
|Pillow (`PIL`)|any recent version|
|Jupyter (VS Code Jupyter extension or JupyterLab)|any recent version|

Install the packages with:

```bash
pip install numpy pandas matplotlib pillow
```

Opening `.ogg` and `.opus` voice notes during manual verification needs an audio player that supports them, such as VLC. The VS Code audio preview cannot play them.

## Folder layout

Keep the notebook **next to** the corpus folder. The notebook creates the folders below if they do not exist.

```
The One To Use 2026/
├── Thesis\_Synthetic\_Corpus\_and\_Recovery\_v6.ipynb
├── README.md
└── UUNDJONDJO\_226111989\_AntiForensics\_Corpus/
    ├── 01\_design/            conditions, acquisition plan, acquisitions performed
    ├── 02\_corpus/
    │   ├── manifest/         ground\_truth\_manifest.csv  (the answer key)
    │   ├── media/            test images, PDF documents, voice-note scripts
    │   └── run\_sheets/       seeding\_run\_sheet.csv / .md
    ├── 03\_acquisitions/      raw ADB pulls + SHA-256 hash lists (never edited)
    ├── 04\_analysis/          item-level results, evidence hits, audits, voice-note verification
    ├── 05\_outputs/           final tables and figures used in the thesis
    └── 06\_documentation/     data dictionary, provenance log
```

## How to run

1. Open the notebook in VS Code or Jupyter, with the working directory set to the folder that contains it.
2. Close any CSV files that are open in Excel. Windows locks open files and the notebook will fail to write to them.
3. Click **Run All**. A full run takes about **15 minutes**, most of it spent hashing the acquisitions.
4. The results appear in `05\_outputs/` and `04\_analysis/`.

### The corpus is frozen

`FREEZE\_CORPUS` is set because the test items have already been sent to the phones. With this setting the notebook **reloads** the existing manifest and media and re-checks their hashes; it does **not** regenerate them. Do not turn this off. Regenerating the corpus would create files that no longer match what was actually seeded.

## Acquisitions scored

All acquisitions are logical ADB pulls from DEV01, a Samsung Galaxy A24 (SM-A245F) on Android 16, which is **not rooted**.

|ID|Run|Platform|Time point|Folder under `03\_acquisitions/`|Pulled|
|-|-|-|-|-|-|
|WA\_T0|WA\_run|WhatsApp 2.26.40.76|T0|`DEV01\_T0\_logical\_adb/wa\_media`|19 Oct 2026|
|WA\_T1|WA\_run|WhatsApp 2.26.40.76|T1|`DEV01\_T1\_logical\_adb/wa\_media`|20 Oct 2026|
|TG2\_T0|TG\_run2|Telegram 13.0.3|T0|`DEV01\_TG2\_T0\_logical\_adb`|21 Oct 2026|
|TG2\_T1|TG\_run2|Telegram 13.0.3|T1|`DEV01\_TG2\_T1\_logical\_adb`|22 Oct 2026|
|TG1\_T0|TG\_run1\_confounded|Telegram 12.10.3|T0|`DEV01\_T0\_logical\_adb/tg\_data`|9 Oct 2026|
|TG1\_T1|TG\_run1\_confounded|Telegram 13.0.3|T1|`DEV01\_T1\_logical\_adb/tg\_data`|20 Oct 2026|

* **Primary results:** `WA\_run` and `TG\_run2`.
* **Telegram run 1:** reported separately as confounded. The app updated itself and the phone's storage was 99% full between T0 and T1.
* **Text messages:** not scored. They exist only in the apps' private databases, which logical acquisition cannot reach without root.

### Long Windows paths

Some acquired paths are longer than Windows' 260-character limit. The notebook adds the `\\\\?\\` prefix automatically through `fs\_path()`. If you pull new data with `adb pull`, map a short drive letter first, for example `subst T: "<long path>"`.

## How items are matched

|Item type|Method|
|-|-|
|Text, PDF documents, document-mode images|Marker search (UTF-8 and UTF-16LE regex `AFX\[WT]\[0-9]{4}\[A-Z2-7]{3}`) and SHA-256 exact match|
|Photo-mode images (recompressed by the app)|64-bit difference hash (dHash), match at **≤ 10 bits**|
|Voice notes|Manual listening at T0. At T1, filled automatically if the same file is present with the same SHA-256|

### Safeguards against false matches

* **WhatsApp seeding-date rule.** A perceptual match on a WhatsApp file is accepted only if the file is named with the seeding date (`-20261019-`) or contains the item's marker in its name.
* **Visual checks.** Borderline and unexpected matches are confirmed by eye and recorded in `04\_analysis/visual\_checks.csv`.
* **`NOT\_SEEDED`.** Items never received in their designed form, `WA\_BASE\_021` and `WA\_BASE\_023`, are removed from the totals rather than counted as losses.
* **`EXCLUDE\_HITS`.** Hits caused by protocol deviations are logged with a reason in `evidence\_hits\_excluded\_deviations.csv`.

### Recovery rates and validation

Recovery rates are reported with **95% Wilson confidence intervals**.

Section 11 validates the pipeline against known answers before it is trusted:

* markers in both encodings and across chunk boundaries;
* zero false positives in random data;
* 40 of 40 recompressed images matched.

The validation results are written to `05\_outputs/pipeline\_selftest.csv`.

## Voice-note verification (manual step)

1. After a run, open `04\_analysis/voice\_note\_candidates.csv`. It lists every audio file found, with its full path.
2. Play each T0 candidate and match it to its read-aloud script in `02\_corpus/media/voice\_scripts/`.
3. In `04\_analysis/voice\_note\_verification.csv`, enter the file path and `1` (heard) or `0` (not found) for each T0 row.
4. Save, close Excel and **Run All** again. The T1 rows fill in automatically.

## Main outputs (`05\_outputs/`)

|File|Contents|Thesis|
|-|-|-|
|`t0\_vs\_t1\_by\_condition.csv`|Recovery per condition, T0 vs T1|Table 4.2|
|`recovery\_by\_condition\_timepoint.csv`|Same, with Wilson 95% CIs|Table 4.2|
|`recovery\_heatmap\_T0\_T1.png`|Heatmap of recovery by condition|Figure 4.1|
|`recovery\_by\_type\_sendmode.csv`|Recovery by item type and send mode|Table 4.3|
|`recovery\_by\_artefact\_source.csv`|Where evidence was found (media folders, cache)|Table 4.4|
|`exif\_retention.csv`|GPS and capture-time survival in recovered images|Table 4.5|
|`dv\_variable\_summary.csv`|Dependent-variable summary|Table 4.6|
|`corpus\_composition.csv` / `.png`|Corpus by condition and type|Table 3.2|
|`pipeline\_selftest.csv`|Instrument validation|Table 3.5|

Item-level detail is in `04\_analysis/`: `item\_level\_recovery.csv`, `evidence\_hits.csv` and `photo\_match\_audit.csv`.

## Privacy

DEV01 is the researcher's own phone, so the acquisitions also contain personal media.

* Personal files are only compared automatically against the test corpus, never analysed.
* The few inspected visually are described only in general terms.
* `03\_acquisitions/` must stay on the researcher's computer. Do not upload, share or commit it.
* No real phone numbers are recorded in the notebook. Accounts are referred to only by test labels.

## Version history

|Version|Change|
|-|-|
|v1–v3|Corpus generation, run sheets, self-test|
|v4|Scoring of the real acquisitions; frozen corpus; long-path support|
|v5|WhatsApp seeding-date rule; `NOT\_SEEDED`; voice-note verification workflow|
|v6|Look-alike rule no longer excludes document-mode images (fixes WhatsApp Exif results); cache-artefact measure counted over recovered items; voice notes found in `cache/` counted as cache artefacts|



