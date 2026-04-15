
# SegPromptNav: Segmentation-Guided LLM Navigation for GPS-Denied UAVs

SegPromptNav is a research-oriented UAV navigation framework for GPS-denied environments that combines semantic image segmentation, prompt-based LLM reasoning, and lightweight control adaptation for autonomous aerial navigation.

The system converts segmentation outputs into compact semantic prompt tokens, then uses a lightweight LLM-style prompt model to generate high-level navigation actions such as slowing down, biasing left or right, or entering safe mode. These actions are fused with the low-level navigation and control stack to improve decision making in cluttered or uncertain environments.

---

## Overview

Traditional UAV navigation systems often rely on geometric planning and reactive control alone. In complex GPS-denied scenes such as urban canyons, road intersections, obstacle-dense corridors, or partially structured environments, semantic understanding can improve navigation decisions.

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
