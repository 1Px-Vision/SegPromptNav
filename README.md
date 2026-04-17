
# SegPromptNav: Segmentation-Guided LLM Navigation for GPS-Denied UAVs

SegPromptNav is a research-oriented UAV navigation framework for GPS-denied environments that combines semantic image segmentation, prompt-based LLM reasoning, and lightweight control adaptation for autonomous aerial navigation. The system converts segmentation outputs into compact semantic prompt tokens, then uses a lightweight LLM-style prompt model to generate high-level navigation actions such as slowing down, biasing left or right, or entering safe mode. These actions are fused with the low-level navigation and control stack to improve decision-making in cluttered or uncertain environments.

---

## Overview

Traditional UAV navigation systems often rely on geometric planning and reactive control alone. In complex GPS-denied scenes such as urban canyons, road intersections, obstacle-dense corridors, or partially structured environments, semantic understanding can improve navigation decisions.
![](https://github.com/1Px-Vision/SegPromptNav/blob/main/Segmentation_LLM.jpg)

This project introduces a prompt-based navigation layer that:

- extracts semantic information from image segmentation,
- transforms scene structure into symbolic navigation tokens,
- builds a compact LLM prompt from navigation state and segmentation context,
- predicts a high-level action for safe motion adaptation,
- integrates that action into the navigation controller.

The framework is designed for simulation, experimentation, and research on agent-driven UAV autonomy.

---

## Key idea

The navigation pipeline follows this structure:

**Input image -> segmentation -> semantic prompt tokens -> LLM prompt -> navigation action -> controller adaptation**

Instead of sending raw images to the language model, the system generates a compact symbolic prompt such as:

```
CLEAR_MID GOAL_LEFT LLM_STRAIGHT SPD_MID RMSE_LOW MISSION_EARLY RISK_LOW GOAL_FAR FRONT_OPEN ROAD_PRESENT ROAD_LEFT VEH_NEAR BLDG_NEAR VEG_NEAR XWALK_ON
```

### Features
* Prompt-based UAV navigation in GPS-denied environments
* Semantic segmentation to prompt-token conversion
* Lightweight TinyGPT-style prompt model
* High-level action generation for navigation adaptation
* Segmentation-aware prompt engineering
* Integration with waypoint navigation and control loops
* Support for aerial urban imagery and obstacle-aware scene reasoning
* Research-friendly modular Python implementation

---

## NAVIGATION ACTIONS

![](https://github.com/1Px-Vision/SegPromptNav/blob/main/waypoint_Q_learning.jpg)

The LLM prompt model predicts one of the following actions:

- ACT_KEEP
- ACT_SLOW
- ACT_FAST
- ACT_LEFT_BIAS
- ACT_RIGHT_BIAS
- ACT_LLM_MORE
- ACT_LLM_LESS
- ACT_SAFE_MODE

These actions are then translated into changes in speed cap, steering bias, or semantic-control weighting.

---
## PROMPT STRUCTURE

The prompt combines two sources of information.

1. Base navigation tokens
These come from the current navigation state:

- clearance level
- heading error
- previous steering bias
- speed scale
- path tracking RMSE
- mission progress
- risk level
- distance to goal

2. Segmentation-derived tokens
These come from the segmented semantic map:

- front openness
- road presence
- road curvature
- nearby vehicles
- nearby buildings
- nearby vegetation
- crosswalk presence

Example combined prompt:

```
CLEAR_MID GOAL_LEFT LLM_STRAIGHT SPD_MID RMSE_LOW MISSION_MID RISK_LOW GOAL_MID FRONT_OPEN ROAD_PRESENT ROAD_LEFT VEH_NEAR BLDG_NEAR VEG_NEAR XWALK_ON

```

------------------------------------------------------------------
EXAMPLE APPLICATION WORKFLOW
------------------------------------------------------------------

A segmentation result is first generated from an aerial image:
```
image_path = "/content/test_aerial.jpg"

result = predict_and_visualize_image(
    image_path=image_path,
    model=teacher,
    id2name=id2name,
    id2code=id2code,
    target_size=input_size,
    min_region_pixels=80,
    ignore_background=True,
    normalize=True,
    save_dir=predictions_dir,
    show=True
)
```
Then the class-id segmentation map is extracted:
```
pred_seg_class_map = result["pred_class_map"]
```

This map is passed into the LLM navigation agent:
```
action, meta = agent.predict_from_state(
    dist_to_goal=dist_to_goal,
    cur_clear=cur_clear,
    heading_err_deg=heading_err_deg,
    last_llm={"steer_deg": -4.0, "speed_scale": 0.75},
    rmse_now=rmse_now,
    seg_i=seg_i,
    num_wps=num_wps,
    class_map=pred_seg_class_map,
    id2name=id2name,
)
```
The agent builds semantic prompt tokens, creates a prompt, and predicts the best high-level navigation action.

------------------------------------------------------------------
EXAMPLE GENERATED PROMPT
------------------------------------------------------------------

You are a UAV navigation agent in a GPS-denied environment.
Use the semantic navigation state below to choose one safe high-level action.
```
PROMPT_TOKENS:
CLEAR_MID GOAL_LEFT LLM_STRAIGHT SPD_MID RMSE_LOW MISSION_EARLY RISK_LOW GOAL_FAR FRONT_OPEN ROAD_PRESENT ROAD_LEFT VEH_NEAR BLDG_NEAR VEG_NEAR XWALK_ON

AVAILABLE_ACTIONS:
ACT_KEEP
ACT_SLOW
ACT_FAST
ACT_LEFT_BIAS
ACT_RIGHT_BIAS
ACT_LLM_MORE
ACT_LLM_LESS
ACT_SAFE_MODE
```
Return only the best action token.

------------------------------------------------------------------
PROJECT STRUCTURE
------------------------------------------------------------------

A typical structure for this repository can be:
```
SegPromptNav/
|
|-- README.md
|-- requirements.txt
|-- main.py
|-- navigation/
|   |-- llm_prompt_agent.py
|   |-- tiny_gpt_prompt_model.py
|   |-- prompt_builders.py
|   |-- control_adaptation.py
|   `-- waypoint_navigation.py
|
|-- segmentation/
|   |-- predict.py
|   |-- visualization.py
|   `-- utils.py
|
|-- configs/
|   `-- default_config.py
|
|-- examples/
|   |-- test_aerial.jpg
|   `-- Deploy_Sementation_drone_LLM.ipynb
|
`-- outputs/
    |-- overlays/
    `-- predictions/
```
Adjust this structure to match your repository.

------------------------------------------------------------------
INSTALLATION
------------------------------------------------------------------

Clone the repository:

git clone https://github.com/1Px-Vision/SegPromptNav.git
cd SegPromptNav

Install dependencies:
```
pip install -r requirements.txt
```
------------------------------------------------------------------
REQUIREMENTS
------------------------------------------------------------------

Typical Python dependencies may include:

- numpy
- matplotlib
- opencv-python
- Pillow
- tensorflow or torch depending on the segmentation backend

Add the exact dependencies used by your implementation in requirements.txt.

------------------------------------------------------------------
HOW IT WORKS
------------------------------------------------------------------

1. Segmentation stage
The input aerial or onboard image is processed by a segmentation model to produce a semantic class map.

2. Semantic token extraction
The class map is divided into task-relevant regions such as front-center, side bands, and crosswalk areas. These regions are analyzed to produce symbolic tokens.

3. Prompt construction
Navigation-state tokens and segmentation tokens are combined into a compact prompt.

4. Prompt-based decision
A lightweight TinyGPT-style model predicts a high-level action.

5. Navigation adaptation
The predicted action modifies the controller behavior, for example by reducing speed, biasing steering, or enabling safe mode.

------------------------------------------------------------------
RESEARCH MOTIVATION
------------------------------------------------------------------

This project explores how language-inspired symbolic reasoning can improve autonomous navigation when GPS is unavailable and perception is uncertain. Rather than relying only on geometry or only on deep control policies, the framework introduces a semantic decision layer between perception and control.

This makes the navigation logic:

- more interpretable,
- easier to debug,
- easier to extend,
- more suitable for agent-driven autonomy research.

------------------------------------------------------------------
CURRENT STATUS
------------------------------------------------------------------

This repository is intended for:

- simulation experiments,
- research prototyping,
- semantic navigation studies,
- prompt-based control adaptation,
- integration with UAV autonomy stacks.

It is not yet intended for safety-critical deployment without further validation.

------------------------------------------------------------------
FUTURE WORK
------------------------------------------------------------------

- online prompt adaptation from real flight data
- dynamic obstacle reasoning
- multi-agent swarm navigation
- SLAM-informed prompt generation
- reinforcement learning and LLM hybrid control
- deployment on edge hardware or FPGA-assisted platforms
- integration with real-time UAV simulators and dashboards

------------------------------------------------------------------
CITATION
------------------------------------------------------------------

If you use this work in academic research, please cite the related paper or repository once published.

@misc{segpromptnav,
  title={SegPromptNav: Segmentation-Guided LLM Navigation for GPS-Denied UAVs},
  author={1Px-Vision},
  year={2026},
  note={GitHub repository}
}


------------------------------------------------------------------
Note
------------------------------------------------------------------

This project builds on ideas from:

- semantic image segmentation,
- autonomous UAV navigation,
- prompt-based reasoning,
- GPS-denied navigation research,
- lightweight transformer-inspired decision models.
