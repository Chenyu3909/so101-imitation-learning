# Experiment Log

## Task

Pick up a white bottle and place it in a white box.

## Sensors

- Base-mounted camera
- Wrist-mounted camera
- 640×480 at 30 FPS for both streams

## 25-demonstration baseline

### 10k training updates

Behavior:
- moved generally toward the bottle;
- consistently missed to the right;
- visibly shaky.

### Additional 10k fine-tuning from the 10k model

Behavior:
- rightward miss became worse;
- shakiness remained.

Interpretation:
- the model was not simply undertrained;
- additional fitting appeared to reinforce bias in the limited dataset.

## 50-demonstration model

Added 25 new demonstrations with more variation in object/target placement and careful, smooth grasp execution.

### 10k training updates

Behavior:
- successfully completed pick-and-place;
- less shaky;
- more accurate.

Evaluation:
- Center: 5/5
- Close Right: 0/5
- Slight Left: 5/5
- Big Swing: 4/5
- Overall: 14/20 = 70%

### Additional 10k fine-tuning from the 10k model

Evaluation:
- Center: 5/5
- Close Right: 3/5
- Slight Left: 4/5
- Big Swing: 4/5
- Overall: 16/20 = 80%

Note: this was an additional training run initialized from the 10k model weights with a fresh optimizer. It was not a true optimizer-state resume.

## Rollout observations

- Target control-loop rate: 30 Hz
- One observed run averaged about 27.9 Hz with occasional latency spikes.
- The policy sometimes continued to make grasping motions after successful placement because the rollout duration had not ended and no explicit task-success termination condition was implemented.

## Main engineering conclusion so far

Increasing dataset diversity from 25 to 50 demonstrations improved real-world behavior more than simply training the original 25-demo model longer.

## Next experiment

Reproduce the same pick-and-place task in Isaac Lab and test whether simulation pretraining can reduce the number of real demonstrations needed for comparable physical success.
