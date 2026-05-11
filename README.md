# Coastal Watch AI

An AI prototype that helps California park rangers spot illegal fishing in Marine Protected Areas by reading photos from shore cameras.

**UN SDG 14: Life Below Water**
**Team:** Mindy Baker · Anahy Diaz · Antonio Lopez · Francisco Rivas
**Course:** Fundamentals of MIS · Spring 2026

**🌐 Live demo:** https://antoniolopez02-oss.github.io/coastal-watch-ai/

---

## Problem

The Monterey Bay National Marine Sanctuary covers more than 6,000 square miles of protected ocean off the central California coast, including four no-take State Marine Reserves where commercial fishing is banned. A small ranger team patrols all of it.

The specific failure isn't a lack of rules. The bottleneck is detection. One ranger boat can only watch one stretch of coast at a time, and distance, weather, and glare cut visual range even there. Unauthorized vessels drop a line inside a no-take zone, fish for an hour, and leave before anyone arrives on the water. Fish populations decline because what no one sees, no one stops.

A park ranger like Officer Miller (a representative user we use across our materials) has to decide each shift which mile of coastline to watch — and which mile to leave unwatched. The system fails at the moment a violation is happening and no one is looking.

---

## AI Capability

We adapted **Lab 3: Image Recognition** — Google's Gemini 2.5 Flash, a multimodal large language model that takes a photo plus a written question and returns a written analysis.

It fits the failure point. Rangers already know what to do when they spot a violation; what they lack is eyes everywhere at once. A model that reads images turns every shore camera into a continuous observer that flags frames worth a human's attention. We send each photo to Gemini with a fixed instruction asking it to identify any vessel, describe any visible fishing gear, score the poaching probability LOW / MEDIUM / HIGH, and write a short alert message — exactly the four pieces of information a ranger needs before deciding to launch the patrol boat.

We chose Gemini over a narrower computer-vision classifier because Gemini writes its conclusion in plain language. A specialized model can output "boat: 0.78." Gemini outputs *"Small motorboat near restricted zone with visible fishing line"* — a sentence a ranger can act on without translation.

---

## Workflow

**What goes in.** A photo from a shore-mounted camera at the reserve. In the prototype, this is a still image uploaded manually; in production, it would be a frame pulled from the camera feed every few minutes.

**What the AI does.** The image is sent to Gemini 2.5 Flash with a fixed system prompt asking for a JSON response with seven fields: `vessel_present`, `vessel_type`, `fishing_gear_visible`, `gear_description`, `poaching_probability`, `reason`, and `ranger_alert`. The prompt explicitly frames the scene as a no-take MPA so Gemini scores against the right baseline. The model returns the structured assessment in seconds.

**What comes out, and who acts on it.** A ranger receives a push notification containing the photo and the AI's alert message. They review the photo, confirm or dismiss the call, and decide whether to drive out. The AI never issues citations, contacts vessels, or closes cases. It suggests; the human decides.

---

## Failure Case

**Input.** A photo of a single-person sea kayak in calm coastal water, with one person seated inside. No fishing gear in frame.

**What Gemini returned** when we ran this image through Step 4 of `CoastalWatchAI_Prototype.ipynb`:

```json
{
  "vessel_present": true,
  "vessel_type": "kayak",
  "fishing_gear_visible": false,
  "gear_description": "none",
  "poaching_probability": "LOW",
  "reason": "The vessel is a recreational kayak, and no fishing gear is visible.",
  "ranger_alert": "Recreational kayaker observed in MPA; no fishing activity detected."
}
```

**Why this is a failure, even though Gemini got the classification right.** Every signal in the JSON says this is a non-event — LOW probability, no gear, recreational use. And yet the system still produced a `ranger_alert` string. If the alert dispatcher forwards every non-empty `ranger_alert` to a ranger's phone (which is the simplest implementation), Officer Miller's phone buzzes for a kayaker. Once is fine. After fifty kayakers across a sunny weekend, he starts muting the channel — and the next alert, the one that actually shows a person setting a lobster trap, gets ignored along with the rest. The failure isn't a misclassification; it's that the system has no built-in gate between "I saw something" and "a human should look at this." This is the alert-fatigue problem, made visible in our own observed output.

---

## Oversight and Tradeoff

**Where human review sits.** A ranger reviews every alert before any patrol action is taken. The system never auto-cites, auto-dispatches, or auto-closes. The AI is an observation layer; the human is the decision layer.

**The one change, and what it costs.** Suppress the `ranger_alert` field whenever `poaching_probability` is `LOW`. In code, this is a one-line check in the alert dispatcher. The cost is real: we accept that a genuinely-marginal case Gemini happened to score LOW (a determined poacher in a kayak with a hidden line, for example) will not surface as an alert at all. We're trading some sensitivity at the bottom of the probability scale for protection of the signal-to-noise ratio at the top. We think this is the right tradeoff because an alert system that gets ignored is worse than one that misses an edge case — rangers losing trust in the channel breaks the whole pipeline, while a missed kayak-poacher is rare and recoverable through routine patrol.

---

## Repository contents

| File | What it is |
|------|------------|
| `CoastalWatchAI_Prototype.ipynb` | The working prototype, adapted from Lab 3. Outputs preserved. |
| `Coastal_Watch_AI_Deck.pptx` | 6-slide pitch deck for the UN council format. |
| `index.html` | Live web demo. Visitors paste their own Gemini API key and upload a photo. |
| `README.md` | This file. |

## Acknowledgments

Built for the Spring 2026 *Fundamentals of MIS* AI for Social Good project. Prototype code adapted from the course's Lab 3 (Image Recognition). Generative AI assistance was used during development to scaffold and debug code; design decisions and final outputs are the team's.
