# JustRUA

JustRUA is a robot policy for the dual-arm ARX X5 in RoboDojo. The actor gets three RGB views, a depth estimate for each view, camera calibration, and measured finger and wrist readings. For example, a close command can coexist with fingers that stop partway. The next decision sees both the command and the measured positions.

One persistent GPT-6 Astra conversation controls each episode. The actor can use a local coding runtime to calculate a target from the observations, inspect its previous work, or issue an absolute end-effector target directly. The Cartesian planner converts that target into robot controls. Each motion executes at most its first 20 controls. The actor then sees the actual state and arrival error before deciding again. There is no policy limit on decision count; the simulator enforces its own episode budget.

## Use with XPolicyLab

This repository contains the complete JustRUA policy adapter. It runs inside the public RoboDojo and XPolicyLab projects. Install from the three public repositories below. A full evaluation also requires RoboDojo's simulator and assets, the external Depth Pro checkpoint, and an existing provider account with access to the selected actor.

From a **new workspace**:

```bash
git clone https://github.com/ZefanW/JustRUA.git
git clone --filter=blob:none https://github.com/RoboDojo-Benchmark/RoboDojo.git
export ROBODOJO_ROOT="$(cd RoboDojo && pwd)"

git -C "$ROBODOJO_ROOT" submodule update --init XPolicyLab
XPL="$ROBODOJO_ROOT/XPolicyLab"
XPL_REV=6baccd57ff958570d0db1588370289ee36c8bb70
git -C "$XPL" fetch origin "$XPL_REV"
git -C "$XPL" checkout --detach "$XPL_REV"

# Copy the adapter into an empty policy slot.
test ! -e "$XPL/policy/JustRUA" && cp -R JustRUA/policy/JustRUA "$XPL/policy/"
cd "$XPL"
```

With an existing RoboDojo installation, use its XPolicyLab checkout if it already includes the Direct EEF planner. The revision above is the tested base. Follow the [adapter setup and evaluation instructions](policy/JustRUA/README.md) from the XPolicyLab root. They include commands for installing the depth model and the coding runtime, running an offline WebSocket check, and evaluating a task.

## Source map

| Directory or file | What it does |
| --- | --- |
| [observations/](policy/JustRUA/observations) | Reads robot and camera measurements, and predicts colorized depth from RGB. |
| [actor/](policy/JustRUA/actor) | Keeps one native coding conversation per episode and isolates its writable scratch space. |
| [policy.py](policy/JustRUA/policy.py) | Applies the control limit to a planned motion and passes measured feedback to the next decision. |
| [deploy.py](policy/JustRUA/deploy.py) | Connects those components to RoboDojo's environment and XPolicyLab's planner. |

## License

The adapter is distributed under [Apache License 2.0](LICENSE). The robot measurement implementation in [observations/](policy/JustRUA/observations/LICENSE) includes code covered by its accompanying MIT license. Apple distributes the Depth Pro weights separately under the terms on the [pinned model card](https://huggingface.co/apple/DepthPro-hf/blob/de816c8ce7168afcb231f96d501d72b869d0beda/README.md).
