# Knee rehabilitation exercise recognition

Design project at Chalmers by Linnea Person, Leo Barberan and Benjamin Houshmand.

We put a rotary encoder and two IMUs on a medical knee brace and trained
classifiers to recognise which of five rehabilitation exercises is being done.

## What is in the notebook

`exercise_classifier.ipynb`:

1. Preprocessing: zero the knee angle with the calibration phase at the start
   of each recording, keep the exercise phase, standardise each channel.
2. Windows of 256 samples with 50% overlap, light augmentation (noise,
   scaling, small shifts).
3. A 1D CNN and a bidirectional LSTM, each trained on five sensor
   combinations (all sensors, encoder only, encoder + IMU1, encoder + IMU2,
   encoder + IMU1 gyroscope).
4. A hybrid model with separate CNNs for the encoder and IMU1 and an LSTM on
   top.
5. The hybrid model on a three-exercise version of the data.

## Results (test recordings)

| Model                       | Sensors              | Test accuracy |
|-----------------------------|----------------------|---------------|
| CNN                         | all                  | 0.51          |
| CNN                         | encoder              | 0.53          |
| CNN                         | encoder + IMU1       | 0.74          |
| CNN                         | encoder + IMU1 gyro  | 0.73          |
| LSTM                        | all                  | 0.49          |
| LSTM                        | encoder              | 0.47          |
| LSTM                        | encoder + IMU1       | 0.76          |
| LSTM                        | encoder + IMU2       | 0.40          |
| LSTM                        | encoder + IMU1 gyro  | 0.68          |
| Hybrid CNN + LSTM           | encoder + IMU1       | **0.79**      |
| Hybrid CNN + LSTM, 3 classes| encoder + IMU1       | **0.86**      |

The numbers are from one training run each (no fixed seed), so expect some
variation when rerunning. The CNN with encoder + IMU2 still has to be rerun
(the old run had a data-loading mistake).

Validation accuracy was 0.95-0.99 for most models, much higher than on the
test set. The validation windows are a random split of the training windows,
and neighbouring windows overlap and come from the same recording, so the
validation set is too easy. The test set uses separate recordings and is the
number to trust.

## Data

The recordings are not public. Each recording is a CSV named
`Exercise<e>Setup<s>Set<n>.csv` with a `phase` column (`Calibrate`,
`Exercise`, ...), `EncoderAngle_deg`, `AngularVel_dps`, and
`IMU{1,2}_{AX,AY,AZ}_g`, `IMU{1,2}_{GX,GY,GZ}_dps`. The notebook expects:

```
data/train/      data/test/        five exercises
data/train_3ex/  data/test_3ex/    exercises 2, 3 and 4
```

## Running

```bash
pip install -r requirements.txt
jupyter notebook exercise_classifier.ipynb
```
