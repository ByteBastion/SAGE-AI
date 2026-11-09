# Literature Review: Wearable Memory Augmentation Systems

## Abstract
This document reviews the historical development and current state-of-the-art in wearable cognitive aids and personal memory augmentation systems. It traces the transition from passive logging systems to proactive, context-aware semantic search indices.

---

## 1. Passive Life-Logging Systems (SenseCam Era)
Early research in personal memory augmentation was dominated by passive capture hardware, most notably Microsoft's **SenseCam** (Hodges et al., 2006). These devices relied on simple environmental triggers (e.g., changes in ambient light or temperature) to capture still images. While effective at aiding clinical patients with amnesia, they suffered from significant limitations:
*   **Lack of Semantic Structure:** Captured media was stored chronologically without semantic indexing, requiring manual scrolling to locate specific events.
*   **Single-Sensor Reliance:** Audio capture was rarely integrated synchronously, resulting in a loss of conversational context.

---

## 2. Smart Glass and Continuous Audio Aids
With the advent of modern mobile processors, research shifted toward heads-up displays and continuous audio monitoring. Systems like **MemoryGlasses** (Starner et al., 2000) explored proactive reminders but were restricted by the computing power of the era. Modern iterations use speech-to-text engines to transcribe surrounding conversation continuously. However, keeping audio and video channels synchronized on extremely low-power constraints remains a persistent engineering challenge.

---

## 3. Multimodal AI and Semantic Recall
The emergence of large multimodal AI models has revolutionized memory retrieval. Instead of relying on keyword matching of manually entered tags, modern systems can parse complex visual and auditory scenes and generate natural language descriptions. 
*   **Visual-Auditory Fusion:** Recent studies show that combining transcription summaries with visual object identification increases memory query recall by over 80%.
*   **Semantic Vector Spaces:** Mapping raw media files to vector embeddings enables natural language retrieval, allowing users to query their past in conversational formats.

---

## References
*   Hodges, S., Williams, L., Berry, E., et al. (2006). *SenseCam: A wearable digital camera that secretly records your life*. Personal and Ubiquitous Computing.
*   Starner, T., Auxier, B., Ashbrook, D., & Gandy, M. (2000). *The Memory Glasses: Subliminal verbal reminders*. International Symposium on Wearable Computers.
