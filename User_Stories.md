# User Stories — World Action Models for Cross-Embodiment Robot Control

**Team members:** Krrish Thakku Suresh, Joshua Jerin
**Advisor:** Huu Tri Nguyen

---

## Stakeholder Map

| Category | Stakeholder | Why they matter |
|---|---|---|
| Primary | Robotics researcher adapting the model to a new robot | Directly trains, fine-tunes, and runs the model on a new embodiment |
| Secondary | Faculty advisor | Judges whether results actually show cross-embodiment generalization |
| Secondary | Lab manager responsible for robot hardware | Maintains the physical robot and pays the cost if it is damaged |
| Hidden | Lab members working near the robot during physical tests | Never touch the software, but are physically exposed to its mistakes |
| Hidden | People who pass through the camera's view during data collection | Can end up in training data without ever knowing the project exists |
| Hidden | Future students reusing our checkpoints and datasets | Only show up after the project ends, when they try to reproduce or extend it |

---

## User Stories

**US-01 (primary):** As a robotics researcher adding a new robot arm to the lab,
I want to adapt the pretrained world action model to that arm using only a small set of demonstrations,
so that I can get a working control policy without collecting thousands of new demonstrations for every robot.

**US-02 (secondary):** As the faculty advisor reviewing the team's results,
I want to compare the model's task success rate on robots it was trained on against robots it has never seen, alongside a single-embodiment baseline,
so that I can tell whether a performance gain reflects real cross-embodiment generalization rather than overfitting to one robot.

**US-03 (hidden):** As a lab member working near the robot during physical testing,
I want the robot's motion to stay inside a fixed workspace and speed limit no matter what action the model predicts,
so that a wrong prediction cannot cause the robot to strike me.

---

## INVEST Self-Check

| Story | Independent | Negotiable | Valuable | Estimable | Small | Testable |
|---|---|---|---|---|---|---|
| US-01 | Yes — depends only on having a pretrained model | Yes — number of demonstrations and adaptation method are open | Yes — core purpose of the project | Mostly — depends on which pretrained model we choose | Largest story; if it grows, split into "collect demonstrations" and "fine-tune and evaluate" | Yes — success rate on the new arm after N demonstrations |
| US-02 | Yes — evaluation can run on any checkpoint | Yes — which tasks and baseline are open | Yes — decides whether results are meaningful | Yes | Yes | Yes — two numeric success rates plus a baseline |
| US-03 | Yes — gate works regardless of model details | Yes — pass threshold is set with the lab manager | Yes — protects hardware | Yes | Yes | Yes — attempt to run an unevaluated checkpoint and observe that it is refused |
| US-04 | Yes — enforced below the model, independent of US-03 | Yes — exact limits chosen with the lab | Yes — personal safety | Yes | Yes | Yes — send out-of-range actions and measure that commanded motion stays within limits |
| US-05 | Yes — applies to the data pipeline only | Yes — method (drop, blur, restrict recording area) is open | Yes — privacy and compliance | Yes | Yes | Yes — inspect stored frames for identifiable faces |

No story names a UI element. Each story states a need and a benefit, and the specific mechanism is left for design in Weeks 5–7.

### Stories considered and cut

- *"As a researcher, I want a dashboard showing training curves"* — cut because it names a UI element rather than a need; the underlying need (judging results) is covered by US-02.
- *"As a researcher, I want the model to work on any robot"* — cut because it is not Small or Testable; narrowed into US-01.
- *"As the team, I want to use the GPU cluster"* — cut because it has no stakeholder benefit and is a project constraint, not a requirement.

---

## Use Cases

*Numeric thresholds below (demonstration count, success rate, speed and timing limits) are draft targets to be confirmed with our advisor and the lab, and revised in Week 9.*

### UC-01: Adapt model to a new robot embodiment

**Expands:** US-01
**Primary actor:** Robotics researcher
**Secondary actors:** Simulation environment, training compute (GPU node)

**Preconditions**
1. A pretrained world action model checkpoint exists and loads without error.
2. The new arm has an embodiment description (joint count, joint limits, action dimension) that loads in the simulator.
3. At least 50 demonstrations of the target task have been recorded on the new arm.
4. An evaluation task set for the new arm is defined in simulation.

**Main success flow**
1. Researcher selects the pretrained checkpoint and registers the new arm's embodiment description.
2. System validates the description in the simulator and reports the arm's joint count, joint limits, and action dimension.
3. Researcher provides the demonstration set for the target task.
4. System checks every demonstration against the registered action dimension and reports how many were accepted.
5. Researcher starts adaptation.
6. System fine-tunes the model and saves a new checkpoint linked to the demonstration set, training configuration, and embodiment description.
7. Researcher requests an evaluation of the new checkpoint in simulation.
8. System runs 50 evaluation episodes and reports the task success rate next to the single-embodiment baseline.
9. Researcher reviews the results and marks the checkpoint as a hardware candidate.
10. System records the checkpoint as simulation-evaluated, which is required before any physical run (US-03).

**Alternate flow A1 — success rate below target (from step 8)**
1. System reports a success rate below 60%.
2. System does not allow the checkpoint to be marked as a hardware candidate.
3. Researcher records additional demonstrations and returns to step 3.

**Exception flow E1 — demonstrations do not match the embodiment (from step 4)**
1. System finds one or more demonstrations whose action dimension differs from the registered arm.
2. System rejects the demonstration set before any training starts and lists each mismatched file with its expected and actual action dimension.
3. No checkpoint is created. Researcher corrects or removes the listed files and returns to step 3.

**Postcondition**
A new checkpoint exists that is linked to the exact demonstrations, configuration, and embodiment it was trained on, and has a recorded simulation success rate. It is marked as a hardware candidate only if that success rate is at least 60%.

---

### UC-02: Run a physical trial within motion limits

**Expands:** US-04
**Primary actor:** Researcher running the physical trial
**Secondary actors:** Robot controller, lab members present near the robot (protected stakeholder)

**Preconditions**
1. The checkpoint being tested is marked simulation-evaluated (UC-01, step 10).
2. A workspace boundary and a maximum end-effector speed of 0.25 m/s are configured in the motion-limit layer.
3. The emergency stop has been tested in the current session and stops the robot.

**Main success flow**
1. Researcher starts a physical trial with the selected checkpoint.
2. System loads the workspace boundary and speed limit and confirms both are active before enabling the robot.
3. Model predicts an action from the current camera observation and robot state.
4. System checks the action against the limits and sends it to the robot controller.
5. Steps 3–4 repeat until the task finishes or the researcher ends the trial.
6. System stops the robot and saves a log of every predicted action, every commanded action, and any limit events.

**Alternate flow A1 — predicted action exceeds a limit (from step 4)**
1. System finds that the predicted action would leave the workspace or exceed 0.25 m/s.
2. System clamps the action to the nearest value inside the limits, sends the clamped action, and logs the event.
3. Flow continues at step 3.

**Exception flow E1 — motion-limit layer stops responding (during steps 3–5)**
1. The motion-limit layer stops sending heartbeats to the robot controller.
2. Robot controller halts all motion and ignores further model commands.
3. System logs the failure time. The trial cannot resume until the researcher restarts the limit layer and repeats step 2.

**Postcondition**
The robot is stopped, all commanded motion during the trial stayed within the configured limits, and a complete trial log is saved.

---

## Acceptance Criteria

**UC-01**

AC-01.1 (main flow)
Given a loadable pretrained checkpoint and 50 valid demonstrations for the new arm,
When the researcher runs adaptation and then simulation evaluation,
Then a new checkpoint is saved with references to its demonstration set, configuration, and embodiment description, and a report shows the success rate over 50 episodes alongside the baseline success rate.

AC-01.2 (main flow)
Given an adapted checkpoint whose simulation success rate is at least 60% over 50 episodes,
When the researcher marks it as a hardware candidate,
Then the checkpoint's status changes to simulation-evaluated.

AC-01.3 (alternate flow A1)
Given an adapted checkpoint whose simulation success rate is below 60%,
When the researcher tries to mark it as a hardware candidate,
Then the request is refused and the checkpoint's status remains not evaluated.

AC-01.4 (exception flow E1)
Given a demonstration set in which at least one file's action dimension differs from the registered arm,
When the researcher starts adaptation,
Then zero training steps run, no new checkpoint is created, and every mismatched file is listed with its expected and actual action dimension.

**UC-02**

AC-02.1 (main flow)
Given a configured workspace boundary and a 0.25 m/s speed limit,
When a physical trial of at least 100 control steps is completed,
Then 100% of commanded positions in the trial log are inside the workspace boundary and the maximum logged end-effector speed is at most 0.25 m/s.

AC-02.2 (alternate flow A1)
Given a running trial,
When the model predicts an action outside the workspace boundary,
Then the commanded action sent to the robot is inside the boundary and a limit event is written to the log for that step.

AC-02.3 (exception flow E1)
Given a running trial,
When the motion-limit layer stops sending heartbeats,
Then robot motion stops within 100 ms and no further model command reaches the robot until the limit layer is restarted.
