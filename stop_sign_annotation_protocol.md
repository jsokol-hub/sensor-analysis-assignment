# Stop Sign Behavior Annotation Protocol

## Overview
This document provides clear guidelines for annotating driver behavior near stop signs in dashcam video footage. The goal is to create consistent, high-quality labels that can be used to train machine learning models for automated stop sign compliance detection.

## Annotation Categories

### Primary Behavior Classification

**1. FULL_STOP**
- Vehicle comes to a complete stop before or at the stop line
- Duration: Vehicle remains stationary for at least 2 seconds
- Position: Stop occurs before crossing the stop line or intersection boundary

**2. ROLLING_STOP**
- Vehicle significantly reduces speed but does not come to a complete stop
- Movement: Vehicle maintains minimal forward motion (very slow, crawling speed)
- Duration: Brief pause or very slow movement through the intersection

**3. SLOW_APPROACH**
- Vehicle reduces speed when approaching the stop sign but maintains moderate speed
- Movement: Noticeable deceleration but maintains steady forward motion
- No clear intention to stop
- If vehicle speed is clearly above crawling (e.g. parking lot speed) but still shows visible deceleration, mark as SLOW_APPROACH.

**4. NO_COMPLIANCE**
- Vehicle maintains normal speed or shows minimal deceleration
- No apparent acknowledgment of the stop sign
- Drives through intersection without stopping or significant slowing

### Secondary Annotations (Optional)

**Stop Position Quality:**
- `BEFORE_LINE`: Stopped before the stop line/crosswalk
- `AT_LINE`: Stopped at or slightly past the stop line
- `PAST_LINE`: Stopped significantly past the stop line
- `N/A`: If stop line or crosswalk is not visible.

**Traffic Conditions:**
- `CLEAR`: No other vehicles or pedestrians visible
- `TRAFFIC_PRESENT`: Other vehicles or pedestrians in the area
- `UNCLEAR`: Cannot determine due to video quality/angle

## Annotation Guidelines

### What to Look For

1. **Speed Changes**: Look for brake lights, deceleration patterns
2. **Stop Duration**: Count seconds of complete stillness
3. **Stop Position**: Relative to stop line, crosswalk, or intersection boundary
4. **Vehicle Movement**: Any forward motion during the "stop"

### Visual Speed Assessment (No Speed Data Available)

Since only video is available, use these visual cues to estimate relative speed:

**Very Slow/Crawling Speed:**
- Objects in background move very slowly relative to vehicle
- Can clearly count individual frames of movement
- Pedestrians would easily outpace the vehicle
- Similar to parking lot speeds

**Moderate/Steady Speed:**
- Background objects move at noticeable pace
- Clear forward momentum maintained
- Faster than walking pace but slower than normal driving
- Similar to residential street speeds

**Normal/High Speed:**
- Background moves quickly past camera
- Minimal visible deceleration
- Maintains typical driving pace
- Similar to through-traffic speeds

### Decision Tree

```
1. Does the vehicle come to a complete stop (no forward motion)?
   YES → Go to step 2
   NO → Go to step 3

2. Does the stop last at least 2 seconds?
   YES → FULL_STOP
   NO → ROLLING_STOP

3. Does the vehicle significantly reduce speed (brake lights, obvious deceleration)?
   YES → Go to step 4
   NO → NO_COMPLIANCE

4. Does the vehicle slow to a crawling/very slow speed?
   YES → ROLLING_STOP
   NO → SLOW_APPROACH
```

### Edge Cases and Special Situations

**Multi-way Stops:**
- Focus only on the annotated vehicle's behavior
- Ignore other vehicles' actions unless they directly affect the target vehicle

**Unclear Stop Signs:**
- If stop sign is not clearly visible in video, mark as `UNCLEAR`
- Only annotate if stop sign presence is obvious from context

**Poor Video Quality:**
- If speed/stopping behavior cannot be determined, mark as `UNCLEAR`
- Do not guess - mark uncertain cases appropriately

**Traffic Interference:**
- If another vehicle blocks the view of stopping behavior, mark as `UNCLEAR`
- If target vehicle is following another vehicle closely, focus on independent behavior

**Camera Movement:**
- Sudden jolt or vibration is not necessarily vehicle motion — rely on background shift

**Hill Slope:**
- When on an incline, stopping behavior may be harder to judge — use background motion

**Non-Sign-Related Stops:**
-  If the vehicle stops for reasons unrelated to the stop sign (e.g. obstacle, animal), mark as UNCLEAR

## Quality Control

### Consistency Checks
- Review annotations for similar scenarios
- Ensure consistent application of speed thresholds
- Verify stop duration measurements

### Common Mistakes to Avoid
1. **Confusing rolling stops with full stops**: Ensure complete cessation of movement
2. **Ignoring stop duration**: A brief touch-and-go is not a full stop
3. **Misjudging speed**: Use visual cues like brake lights and relative motion
4. **Inconsistent position assessment**: Maintain consistent reference points

## Training Signal Optimization

### Why These Categories Matter

**FULL_STOP vs ROLLING_STOP:**
- Critical distinction for legal compliance
- Different risk profiles for safety models
- Clear behavioral difference for ML training

**SLOW_APPROACH vs NO_COMPLIANCE:**
- Indicates driver awareness vs. inattention
- Different intervention strategies needed
- Helps model understand intent vs. oversight

### Annotation Balance
- Aim for representative distribution across categories
- Don't force equal distribution - reflect real-world behavior
- Flag unusual or edge cases for review

## Technical Specifications

### Video Analysis
- **Frame Rate**: Analyze at normal playback speed initially
- **Slow Motion**: Use for precise stop duration measurement
- **Multiple Views**: If available, use all camera angles

### Timing Measurements
- **Time Start**: When vehicle begins decelerating for the stop sign (typically 3-5 seconds before intersection)
- **Time End**: When vehicle has completed the stop sign interaction and resumed normal speed (typically 2-3 seconds after leaving intersection)
- **Stop Duration**: Count from complete cessation to first movement (for FULL_STOP classification)

### Time Interval Guidelines
- **Start Point**: Begin timing when driver clearly reacts to stop sign (brake lights, visible deceleration)
- **End Point**: End timing when vehicle returns to normal driving behavior after the intersection
- **Total Duration**: Typically 5-10 seconds for complete stop sign interaction
- **Focus Period**: The critical behavior occurs in the middle of this interval

### Output Format
```
{
  "video_id": "string",
  "time_start": "HH:MM:SS",
  "time_end": "HH:MM:SS",
  "primary_behavior": "FULL_STOP"|"ROLLING_STOP"|"SLOW_APPROACH"|"NO_COMPLIANCE",
  "stop_position": "BEFORE_LINE"|"AT_LINE"|"PAST_LINE"|"N/A",
  "traffic_conditions": "CLEAR"|"TRAFFIC_PRESENT"|"UNCLEAR",
  "confidence": "HIGH"|"MEDIUM"|"LOW",
  "notes": "string (optional)"
}
```

### Confidence Levels
1. `HIGH`: Behavior is clearly visible and unambiguous (good lighting, no obstructions)
2. `MEDIUM`: Some uncertainty due to moderate occlusion, camera angle, or lighting
3. `LOW`: Behavior difficult to judge, likely edge case or video quality issue

## Assumptions and Limitations

### Assumptions Made
1. **Video Quality**: Sufficient resolution to determine vehicle movement
2. **Camera Position**: Dashcam provides adequate view of intersection
3. **Stop Sign Visibility**: Stop sign presence is clear from context
4. **Single Vehicle Focus**: Annotation focuses on the vehicle with the dashcam

### Limitations Acknowledged
1. **No Speed Data**: Relying on visual estimation of speed
2. **No Audio**: Cannot use engine/brake sounds as cues
3. **Limited Context**: No knowledge of driver intent or external factors
4. **Perspective Bias**: Single camera viewpoint may miss some behaviors

### Simplifications for Remote Annotators
1. **Binary Decisions**: Clear yes/no criteria where possible
2. **Visual Cues Only**: No complex inference required
3. **Standardized Timing**: Fixed thresholds for consistency
4. **Clear Examples**: Provide reference videos for each category

## Expected Training Impact

This annotation scheme is designed to:
1. **Maximize Signal**: Clear behavioral distinctions for ML training
2. **Minimize Noise**: Reduce annotator subjectivity through clear guidelines
3. **Enable Automation**: Categories map directly to automated detection features
4. **Support Safety**: Focus on legally and safety-relevant behaviors

The resulting labels will enable training of models that can:
- Detect stop sign compliance in real-time
- Assess driver behavior quality
- Provide feedback for driver assistance systems
- Support fleet safety monitoring applications 