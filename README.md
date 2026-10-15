# robot-dataset-audit

A small open-source script that runs automated quality checks on a **LeRobot v3** dataset hosted on Hugging Face, and prints a one-page PASS / REVIEW / FAIL summary.

Built by [SRA Robotics](https://github.com/sraentertainment17-code/sra-robotics-sample-audit), independent robot dataset QA. Contact: founder@sra-robotics.com

## What it checks (9 pass/fail checks + 1 hint)

1. **Episodes**: are all episodes present, numbered 0 to N-1, none empty, frame count matches `info.json`?
2. **Duplicates**: is any episode an exact copy of another?
3. **Lengths**: is any episode under half or over double the normal (median) length?
4. **Timing**: steady frame rate, no skipped frames?
5. **Sync**: are camera videos in sync with the robot data?
6. **Videos** *(only with `--videos`, slow)*: does every video file decode end to end?
7. **Action vs state**: does the robot actually follow its commands?
8. **Actions**: any dead joints (never move in an episode) or clipped joints (stuck at the +/-100 limit)?
9. **Empty episodes**: does every episode actually move?
10. **Endings (hint only)**: does any episode end in an unusual pose? Marked **REVIEW**: a human should watch it.

Thresholds (e.g. 2x length, 1% clipping) are listed at the top of `audit_checks_v1.py` and can be changed there.

## Run it

```bash
pip install numpy pandas huggingface_hub pyarrow av
python audit_checks_v1.py MaxT90/so101-pick-place --videos
```

Leave off the dataset name to use the default (`lerobot/svla_so101_pickplace`). Leave off `--videos` to skip the slow video decode.

## Example result (from my sample audit of `MaxT90/so101-pick-place`, Oct 11, 2026)

Condensed from the [sample report](https://github.com/sraentertainment17-code/sra-robotics-sample-audit), not a raw terminal paste:

```
FAIL   #3 Lengths     episode 0 is 948 frames vs normal 438 (14.6 s); limits 219-876
PASS   #1-2 Episodes  all 50 present (0-49), shortest 346 frames
PASS   #1c Duplicates 50 unique
PASS   #4 Timing      steady 30 fps, 0 skipped
PASS   #5 Sync        0.000 s max gap
PASS   #5b Videos     22,771/22,771 frames decoded on both cameras
PASS   #7 Actions     no dead joints, clipping under 1%
REVIEW #9 Endings     watch episode 45: gripper left open
```

A human then confirmed episode 0 has a dead start (robot still for ~16 s). Result: 49 of 50 usable, PASS after 1 fix.

## Honest limitations

- LeRobot **v3** datasets only. Other formats are not supported.
- It cannot judge wrong labels or whether a camera image is clear. Those need a human watching the videos.
- The duplicate check only catches **exact** copies (not copies with added noise).
- A PASS means "no problems found by these checks", not "your robot will train well."
- The video check downloads every video file, so it is slow and uses disk space.
- Thresholds are rules of thumb I chose, not industry standards. Tune them for your data.

## License

MIT. See [LICENSE](LICENSE).
