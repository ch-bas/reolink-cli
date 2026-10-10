# AI-Driven Surveillance Tuning

This guide demonstrates how to use an AI agent (like Claude Code, Cursor, Codex, or Gemini) equipped with the `reolink-cli` skill to autonomously tune your camera's sensitivity settings. 

Tuning security cameras to balance true positive detections (people, vehicles) against false alarms (cobwebs, moving shadows, rain, insects) is notoriously difficult. By combining the `reolink-cli` plugin with an AI agent's goal-driven behavior, you can automate this calibration process.

## The Workflow

The ultimate geek-level workflow is to tell your AI agent: *"Watch the event log for this camera overnight, and iteratively tune the sensitivity until the false alarms from cobwebs stop, but make sure person detection stays reliable."*

The AI will then:
1. Connect to the camera and read its baseline settings (`info`, `detect ai get`, `detect motion get`).
2. Analyze recent history to quantify the noise (`events query`).
3. Make incremental, idempotent adjustments (`detect ai apply`, `detect motion apply`).
4. Re-evaluate the event log to confirm the noise has subsided.

## Prerequisites

- You must have `reolink-cli` and `reolink-gateway` installed (see the main repository [README](../../README.md)).
- The gateway must be running: `reolink-cli gateway start --addr 127.0.0.1:9000 &`
- You must have added at least one camera: `reolink-cli device add <camera-name> --host <ip> --user admin`
- Your preferred AI agent must be configured with the `reolink-cli` skill.

## How to Trigger the Agent

Depending on your agent platform, you can invoke the tuning workflow using natural language or slash commands.

### Example Prompts

**For Daylight Shadow / Foliage Noise:**
> "My 'front-door' camera is constantly triggering false alarms due to moving shadows and wind blowing the trees. Can you check the current detection settings, analyze today's event log, and tune the motion and AI sensitivity to fix this? Please preserve high thresholds for Person (>70) and Vehicle (>60) if possible."

**For Nighttime IR Flare / Bug Noise:**
> "The 'back-porch' camera is getting flooded with false motion events at night due to IR flare from cobwebs and bugs. Please tune the sensitivity settings to eliminate these false alarms without losing actual person detections."

**For Missed Detections:**
> "My 'driveway' camera missed a delivery truck today. Can you analyze the settings and increase the vehicle and person AI sensitivities, then monitor to make sure it doesn't cause a flood of false positives?"

### Using Goal-Driven Loops (Agent Dependent)

If your agent supports autonomous goal execution (like `/goal` in AGY/Gemini, or autonomous loop mode in Claude/Cursor), you can give it an open-ended calibration task:

> "/goal Monitor the 'yard' camera for the next 2 hours. If you see more than 5 false alarms, lower the motion sensitivity by 5. Repeat this until the false alarms stop."

## Firmware Considerations

The AI agent will attempt to use API-level commands to adjust settings. However, some battery-powered cameras may not support changing recording schedules via the API (returning `400` or `405` errors). 
If this occurs, the agent will gracefully detect the constraint and provide you with concise instructions on how to make the necessary schedule changes manually via the Reolink App, before continuing to monitor the results.
