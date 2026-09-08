# train an agent to play Ms. Pac-Man

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/pepealonso95/pacman-dqn/blob/main/pacman_dqn.ipynb)

**[Open the notebook in Google Colab](https://colab.research.google.com/github/pepealonso95/pacman-dqn/blob/main/pacman_dqn.ipynb)**

Open **[pacman_dqn.ipynb](pacman_dqn.ipynb)**, choose three numbers, and run all cells.
The complete DQN is in the notebook. No coding or separate source files are needed.

| Your choice | What it controls | Starting point |
|---|---|---|
| Exploration | Fraction of random training moves after warm-up | `0.20` |
| Episodes | Number of training games | `100` |
| Learning rate | Size of each learning update | `0.0001` |

Try five episodes to check your setup. Useful Atari learning may require much longer runs.
Exploration stays constant after 1,000 random warm-up decisions. The code includes an instruction for Claude Code or Codex to ask the student for these three choices before training.

## Open and run

**Google Colab:** use the Colab button above, select Runtime → Change runtime type → T4 GPU if available, edit the three values in section 1, and choose Runtime → Run all. The setup cell installs packages automatically.

**Local Jupyter or VS Code:** clone or download this repository, open the notebook, select a Python 3.11–3.13 kernel, edit the three choices, and choose Run All. CUDA, Apple Silicon MPS, and CPU are detected automatically; a real training batch checks the selected device before the experiment starts.

If you need to install Jupyter first:

```sh
python -m venv .venv
# macOS / Linux:
source .venv/bin/activate
# Windows PowerShell: .venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m jupyter lab pacman_dqn.ipynb
```

## What you get

Each experiment saves a separate folder under `pacman_runs/`:

- Settings, hardware, and actual package versions in `config.json`.
- Untrained gameplay and new GIFs every 25 episodes.
- A training dashboard with raw score, loss, and exploration.
- All five untrained and trained evaluation scores in `comparison.json`.
- Untrained, periodic, and final model checkpoints for playback.
- CSV training history, elapsed time, decision count, and number of learning updates.
- A ZIP download with all results.

Use Interrupt / Stop once to end training early and save progress, then run the remaining cells for evaluation and download. The saved model includes any updates from the interrupted episode; the episode log contains completed episodes only. Checkpoints support playback, not exact training resumption. Rerunning training starts a new experiment from scratch.

In Colab, download the ZIP before the session ends. Commit the notebook, selected GIFs, plots, comparison, and your written reflection to your own repository. Generated run folders and checkpoints are ignored by default; copy the selected evidence into a `results/` folder to publish it. Keep large checkpoints locally or attach them to a release.

## How it works

The environment is `ALE/MsPacman-v5`. Four grayscale 84 × 84 frames feed a convolutional Q-network, which predicts action values. The agent uses experience replay, a target network, Adam, Huber loss, and a discounted reward target. Game over removes the future-value term; time limits do not.

Frame skipping happens only in `AtariPreprocessing` (four frames per decision); the base environment uses `frameskip=1`. Reset fills the stack with the new game's initial screen. Sticky-action probability is 0.25, no-op resets use up to 30 actions, and losing one life does not end the episode. Each game is capped at 3,000 decisions, about 200 seconds of game time.

Replay stores each current stack plus one new frame as `uint8`: about 168 MiB at 5,000 transitions, plus Python, network, and batch overhead. Training samples 32 transitions every four decisions after warm-up. The target network syncs every 1,000 decisions; gamma is 0.99. Training rewards are clipped to [-1, 1], while all reported game scores are raw.

Before/after evaluation uses the same five seeds, 5% exploration, and step cap. The baseline is an untrained network. Evaluation uses a separate environment and never updates replay or weights. The best GIF is selected by full-game score, but only its first 20 seconds are recorded. Inspect all five scores before claiming improvement. The small replay memory and fixed exploration simplify the classroom exercise; this is not a benchmark-scale DQN reproduction.

## Make your own submission repository

Fork this repository or create your own repository containing the notebook and selected results. The notebook works by itself; README and requirements support setup and explanation.

## Verification

Executed every notebook cell in order through a real Jupyter kernel on macOS Apple Silicon with MPS, Python 3.13.9, PyTorch 2.10.0, Gymnasium 1.3.0, ALE 0.11.2, OpenCV headless 4.14.0.94, NumPy 2.3.4, Matplotlib 3.10.6, and Pillow 11.3.0.

The classroom check used 20% exploration, **five episodes**, and learning rate 0.0001 with all other notebook settings unchanged: 3,306 decisions and 577 learning updates. All five before/after games completed. Verified changed model weights, finite learning losses after warm-up, checkpoint loading, replay reconstruction, frame-stack reset, different targets for termination and truncation, GIF frames, plot creation, and ZIP output. A separate shortened two-episode test exercised the periodic demonstration/checkpoint path.

This verifies execution, not strong Pac-Man performance. A full 100-episode run, CUDA, Windows, and hosted Colab have not been tested here. The distributed notebook has no saved outputs and retains its 100-episode starting value.

## Sources

- [ALE installation](https://ale.farama.org/getting-started/)
- [Gymnasium Atari preprocessing](https://gymnasium.farama.org/api/wrappers/misc_wrappers/#gymnasium.wrappers.AtariPreprocessing)
- [Gymnasium frame stacking](https://gymnasium.farama.org/api/wrappers/observation_wrappers/#gymnasium.wrappers.FrameStackObservation)
- [DQN paper](https://storage.googleapis.com/deepmind-media/dqn/DQNNaturePaper.pdf)
