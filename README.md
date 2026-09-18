# Project Constraints Essay

## Project: World Action Models for Cross-Embodiment Robot Control

**Team members:** Krrish Thakku Suresh, Joshua Jerin<br>
**Advisor:** Huu Tri Nguyen, Department of Aerospace Engineering and Engineering Mechanics, nguye3hr@mail.uc.edu<br>
**Advisor fit:** Tri Nguyen's research areas are machine learning and explainable AI.

---

## Economic

Training large World Action Models from scratch would require more GPU compute than is practical for our senior design project, so we will build on existing pretrained models instead. We will prioritize open-source models, simulation environments, and hardware already available to the team or through UC rather than purchasing large amounts of dedicated compute. This makes fine-tuning and evaluating smaller experiments across robot embodiments more viable than attempting to train a foundation model from scratch.

## Professional

Our project requires knowledge across machine learning, robotics, simulation, and control because the World Action Model must connect visual observations and robot state to executable actions. We will divide the system into model training, simulation, robot integration, and evaluation components so team members can develop and test individual parts independently. Our advisor will help validate the model architecture and experimental methodology, especially when deciding whether performance improvements actually demonstrate useful cross-embodiment generalization.

## Ethical

Because the model can eventually produce actions for a physical robot, model outputs will first be evaluated in simulation before being allowed to control hardware. Physical testing will include limits on the robot's motion and action space so an incorrect prediction cannot directly cause an unrestricted movement. This creates a trade-off between autonomy and safety, since giving the model a larger action space may improve its ability to solve unfamiliar tasks while tighter limits make physical testing safer and more predictable.

## Security

The project will use camera observations, robot-state information, demonstrations, datasets, and trained model checkpoints during development. We will collect only the visual and robot data needed for training and evaluation and will avoid intentionally recording personally identifiable information in our datasets. Limiting data collection may reduce the diversity of real-world training data available to the model, so we will use simulation and controlled environments where possible rather than increasing dataset size at the expense of privacy.
