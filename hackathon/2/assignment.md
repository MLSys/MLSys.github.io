# Decentralized Reinforcement Learning over the Internet

## Changelog {#changelog}

### November 10, 2026 · Install on your own cluster {#install-on-your-cluster}

[Download the Wordle RL starter ZIP](https://linux-slai.tail6d76d1.ts.net/labs/weak-network-rl/mf2502-wordle-rl-starter-source.zip) and upload it to the cluster where you will run the lab. The ZIP contains this handout, the setup scripts, a two-GPU training check, and the evaluator for 32 public Wordle games. Download the model and Python dependencies separately using the commands below. This setup uses your own cluster storage.

Use a Linux x86-64 NVIDIA GPU container with glibc 2.35 or newer, a driver that supports CUDA 13.0, and `git`, `patch`, `unzip`, and [uv](https://docs.astral.sh/uv/getting-started/installation/) 0.11.1 or newer. The commands install Python 3.12, PyTorch 2.13 and vLLM 0.31 in the project's `.venv`. The [model weights](https://huggingface.co/PrimeIntellect/Qwen3-1.7B-Wordle-SFT) are about 4 GB. Installation needs access to GitHub, Hugging Face, PyPI, the PyTorch package index, and NLTK data. If your GPU nodes have limited internet access, use your cluster's login node or approved transfer route to stage the downloads.

Run these commands in a fresh work directory containing the downloaded ZIP.

```bash
unzip mf2502-wordle-rl-starter-source.zip
STARTER=$(realpath MF2502-Wordle-RL-starter-source/starter)

mkdir prime-rl
cd prime-rl
git init
git fetch --depth 1 https://github.com/PrimeIntellect-ai/prime-rl.git \
  9e15490dcab1d506ce3462e69b8aa5dde45c6b3a
git checkout --detach FETCH_HEAD
git -c url.https://github.com/.insteadOf=git@github.com: \
  submodule update --init --depth 1 -- \
  deps/verifiers deps/renderers deps/prime-envs deps/pydantic-config

uv sync --frozen --python 3.12 \
  --package prime-rl --package wordle --extra gpu --extra flash-attn

uvx --from huggingface_hub hf download PrimeIntellect/Qwen3-1.7B-Wordle-SFT \
  --revision b4052d5a2a14f6ecc9f00a85698084df436f626e \
  --exclude '*.bin' --local-dir model

.venv/bin/python -m nltk.downloader -d "$PWD/nltk_data" \
  words averaged_perceptron_tagger_eng

patch -p1 < "$STARTER/offline-runtime.patch"
cp "$STARTER/wordle_taskset.py" deps/verifiers/environments/wordle/wordle/taskset.py
cp "$STARTER"/{run.sh,smoke.toml,reference.json,reference_eval.py} .
```

Then open a shell in the same `prime-rl/` directory on a node with two visible GPUs.

```bash
bash run.sh check
bash run.sh smoke --run.name setup-check
```

The check runs one trainer and one worker for three updates. It verifies that the components work together before the lab. The teaching team will provide the four-GPU launcher and announce a submission path for each cluster. See [section 8](#public-evaluation) to test the starting model or a saved adapter.

## 1. Background {#why}

Large AI models are typically trained in purpose-built data centers, where GPUs exchange data over dedicated high-speed links. GPUs are also spread across personal workstations, university labs, and small organizations. Pooling that hardware over the internet could give independent researchers and small teams a way to train models of their own with less reliance on large data centers.

Reinforcement learning (RL) offers one way to divide the work. Some machines run the model and generate attempts at a task. Another machine uses those attempts and their rewards to update the model. INTELLECT-2 used this approach to train a 32-billion-parameter reasoning model: a dedicated training cluster updated the model, while GPUs contributed by the community generated training examples. [INTELLECT-2 technical report](https://arxiv.org/html/2505.07291v1#S2)

This lab brings that idea down to four GPUs: one trainer and three workers. The GPUs do their work locally and do not exchange weights over a high-speed GPU interconnect. Instead, the trainer writes each update once to a shared network filesystem, and each worker reads it independently. The course adds a fixed delivery delay to model a slow network link, so the experiment works even if the platform puts the GPUs in the same machine. A worker can wait for new weights or keep generating games with the version it already has. Waiting costs time; games produced with older weights can become too old for the trainer to use. You will use Wordle to test that trade-off.

## 2. The training task {#task}

Train a **Qwen3-1.7B language model to play Wordle** using **Group Relative Policy Optimization (GRPO)**. Start from the supplied Wordle checkpoint. In each game, the model has up to six guesses to find a hidden five-letter word. After each guess, the game tells the model which letters are correct and which are in the wrong positions. A complete game is one training attempt, also called a *rollout*. For each hidden word, GRPO compares the rewards from four independent games and uses the differences to update the model.

The experiment tests **whether workers can contribute more useful games under the same weak-link conditions by changing when they play and when they switch weights**. Use the following two ways of running the same training job.

| Run | How the workers operate |
| --- | --- |
| **Synchronous reference** | After every training update, the trainer publishes the new weights. All workers wait until each of them has loaded that version. The next batch of games starts only then. |
| **Your scheduling policy** | Change the scheduling code to decide which worker gets the next group of games, how many groups can be in progress on each worker, and when each worker loads a newer version. A group is four games for the same hidden word. Keep all four games on the same model version through the end of the group. For example, one worker can start another group with its current version while another worker loads the new version. |

For the comparison, start both runs from the same checkpoint and keep the training rules, GPU allocation, and link settings the same. Check how long each run waits, how many games the trainer actually uses, and how the trained models perform on the same public words. Choose **one trained checkpoint** (a saved version of your model) for grading. Its win rate on a separate set of Wordle games is your grade. Section 9 lists the files to submit.

## 3. GPU allocation and deadline {#setup-and-deadline}

Each student completes the lab individually with **four GPUs**. Run one trainer and three inference workers, one process per GPU. The course supplies the starting model, training configuration, and server setup.

You may start training during or after class and continue until the evening deadline on the day of the lab. Choose when to run short comparisons and when to let a promising run train longer. Save and check a trained checkpoint before submitting. The teaching team will announce the exact deadline. Submit from your own course storage as described in section 9.

## 4. One round of training {#training-update}

1. **The workers play.** A worker plays four games for each assigned word, starting each game from scratch. It returns the model's guesses, the clues the game gave it, and the final reward. The record includes the version of the model and the probabilities it assigned to the text it generated.
2. **The trainer collects complete groups.** The default batch is six groups of four games, for 24 games in one update. A win earns a reward of 1. A loss can earn partial credit. At the guess limit, the last guess earns one point per correctly placed letter and half a point per correct letter in the wrong place, divided by five. Four equal rewards give GRPO no relative signal about which attempt to favor.
3. **The trainer updates the model and publishes new weights.** The trainer changes a small set of model weights called a *LoRA adapter*. It writes the adapter once to the shared network filesystem. Each worker waits for the course's delivery delay, then reads and loads the adapter from the filesystem while keeping the starting model in memory.

The trainer checks how old a group is when it takes it for an update. With the default two-version limit, a trainer on version 8 can use complete groups played with versions 8, 7, and 6. When the trainer moves to version 9, a version-6 group still waiting in the queue expires. A GPU can finish every game it was given and still contribute little to training if too many groups expire.

## 5. Default configuration {#defaults}

The table lists the current defaults. The course will publish the final training and link settings with the launch instructions.

| Setting | Default |
| --- | --- |
| Starting model | `PrimeIntellect/Qwen3-1.7B-Wordle-SFT`, already fine-tuned to follow the Wordle game format. |
| Training algorithm | GRPO with LoRA rank 16, alpha 32, and learning rate `1e-5`. |
| Batch and task stream | 6 groups of 4 games per update. Use the course's sequence of target words and its reward rules. The groups that reach training may differ between schedules. |
| Game limits | Up to 6 guesses and 1,024 generated tokens per model response. The pilot starts with a 120-second timeout per game. |
| Weight updates | The trainer writes each completed update to the shared network filesystem. A worker can start loading it after the course's delivery delay and can use it once loading finishes. |
| Maximum age | At most 2 versions behind the trainer. A training batch may contain complete groups from different eligible versions. |

Use the course model, task stream, rewards, batch size, optimizer, game limits, link settings, and GPU allocation for both runs. Change the scheduling decisions described in section 2. Use games generated within the same run for its training updates.

## 6. Server setup {#start}

Follow the [installation instructions above](#install-on-your-cluster) from your own cluster. The optional two-GPU check writes its logs to `runs/setup-check/logs/latest/` and LoRA weights to `runs/setup-check/broadcasts/`. The four-GPU launch instructions will identify the command for the synchronous reference, the scheduling code you can edit, and where the full runs write their logs and adapters.

## 7. Comparing the two runs {#worker-scheduling}

First run the synchronous reference long enough to see a complete update. Check when the workers started and finished their games, when the trainer had enough games, and when each worker could use the new weights. Then make one change to the schedule. For the short comparison, start both runs from the same checkpoint and measure an equal interval beginning when the first games are dispatched. Count training updates that finish within that interval. You can let your preferred configuration train for longer before submitting.

| Measure | How to use it |
| --- | --- |
| Trainer wait and worker wait | Check whether the trainer waits for games or the workers wait for weights. |
| Groups accepted and groups expired | Check how much of the completed work the trainer could use. Accepted groups can still have equal rewards and give no relative learning signal. |
| Completed updates in the same interval | Check whether the schedule lets the trainer update the model more often. |
| Wins on the public games | Check whether the resulting model also plays better. |

If the trainer finishes few or no updates, first check whether complete groups are reaching it and whether they are still recent enough to use. If many groups expire, try starting fewer groups at once or loading newer weights sooner.

## 8. Public evaluation {#public-evaluation}

The starter includes **32 public Wordle games**. Their target words are excluded from its training stream. Use the same games to check the starting model and compare your checkpoints. Serve the starting model in one terminal and run the evaluator from another terminal in the same work directory.

```bash
# Terminal 1
bash run.sh serve --host 127.0.0.1 --port 8000

# Terminal 2
bash run.sh reference --output start-score.json
```

To check a trained model, stop the starting-model server and restart it with your exported adapter directory.

```bash
# Terminal 1
bash run.sh serve --host 127.0.0.1 --port 8000 \
  --lora-modules candidate=/absolute/path/to/adapter

# Terminal 2
bash run.sh reference --model candidate --output candidate-score.json
```

The report lists wins and per-game results. In the package check, the starting model won 1 of 32 games. One additional win changes the public score by about three percentage points.

## 9. Submission and grading {#server-submission}

Keep your submission in **one folder on your own persistent course storage**. The teaching team reads it in place. The course registry stores only the absolute path to that folder. Use storage that the grading machine can mount, and keep the folder available until grading is complete.

### Prepare your submission

Use the following names and layout. Submit exactly one adapter, exported from the checkpoint you want graded.

```
wordle-submission/
├── adapter/
│   ├── adapter_config.json
│   └── adapter_model.safetensors
├── report.md
├── code/
├── config/
└── logs/
    ├── reference.log
    ├── candidate.log
    └── final.log        # Required if the submitted adapter came from a separate run
```

Put your scheduling code and any changed launch scripts in `code/`, the settings used for your runs in `config/`, and the corresponding training logs in `logs/`. Include the files themselves so the grading machine can open them. To check your export, use the command in section 8 with your submission's `adapter/` directory as the adapter path.

Write a short Markdown report in `report.md`. One page is enough. Include:

- What you changed in the scheduler and what effect you expected.
- A table comparing the synchronous reference and your schedule over the same time window. Give the window length, trainer and worker wait time, groups accepted and expired, completed updates, and public Wordle wins out of 32 for each run. Say what changed and use the logs to explain why.
- Which run and update produced the submitted adapter, its public Wordle score, and the commands and configurations you used.

### Register the path

Run the following once the teaching team announces the registry for your cluster. Replace the example student ID, submission directory, and registry directory with your own values. Your submission and registry must both be reachable from your cluster.

```bash
STUDENT_ID=1234567890
SUBMISSION=/data/1234567890/wordle-submission
REGISTRY=/path/announced/for/your/cluster

(
  set -e
  SUBMISSION=$(realpath -e -- "$SUBMISSION")
  for FILE in report.md adapter/adapter_config.json adapter/adapter_model.safetensors; do
    if [ ! -s "$SUBMISSION/$FILE" ]; then
      printf 'Missing or empty file: %s\n' "$FILE" >&2
      exit 1
    fi
  done
  chmod -R a+rX,go-w -- "$SUBMISSION"
  RECEIPT=$(mktemp "$REGISTRY/.${STUDENT_ID}.XXXXXX")
  printf '%s\n' "$SUBMISSION" > "$RECEIPT"
  chmod 644 "$RECEIPT"
  mv -fT -- "$RECEIPT" "$REGISTRY/$STUDENT_ID.txt"
  cat "$REGISTRY/$STUDENT_ID.txt"
)
```

The command makes the submission files readable and keeps them writable only by their owner. The printed path is your receipt. To replace your submission before the deadline, run the command again with the same student ID. If the teaching team cannot reach the folder because a parent directory is private, ask them to check access with you.

The teaching team loads `adapter/` with the supplied base model and evaluates it on a separate set of Wordle games. **Your score is the fraction of those games the model wins.** The report and logs document how you trained it.
