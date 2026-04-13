# Model Card: Music Recommender Simulation

## 1. Model Name

**VibeFinder 1.0**

---

## 2. Intended Use

VibeFinder 1.0 suggests the top 5 songs from a small catalog based on a user's preferred genre, mood, energy level, and acoustic preference. It is designed for classroom exploration and demonstration — not for real users on a production platform. It assumes the user can express their taste as a single set of fixed preferences. It is not suitable for users whose taste shifts by context (e.g., workout vs. studying), for languages other than English, or for catalogs larger than a few hundred songs without a performance upgrade.

---

## 3. How the Model Works

VibeFinder reads a spreadsheet of songs, each described by genre, mood, energy, and acousticness. It then compares each song to the user's stated preferences using a point system.

- If a song's genre matches the user's favorite, it earns 2 points.
- If the song's mood matches, it earns 1 point.
- For energy, the system measures how close the song's energy is to the user's target and awards up to 1 point — a perfect match gives 1.0, a big mismatch gives close to 0.
- If the user likes acoustic music and the song is highly acoustic (or vice versa), it earns a bonus 0.5 points.

All songs are then ranked from highest to lowest score and the top 5 are returned along with the specific reasons for their score.

---

## 4. Data

The catalog contains **25 songs** stored in `data/songs.csv`. Each song has 10 attributes: id, title, artist, genre, mood, energy, tempo_bpm, valence, danceability, and acousticness.

Genres represented: pop, lofi, rock, ambient, jazz, synthwave, indie pop, electronic, country, metal, hip-hop, folk, r&b.

Moods represented: happy, chill, intense, moody, relaxed, focused, sad.

Limitations of the data:
- Classical, blues, soul, and reggae are entirely absent.
- Mood labels are subjective — one listener's "relaxed" is another's "boring."
- All songs are fictional; the data reflects the bias of the person who created it, not real listener demographics.
- The catalog is too small for diversity: 3 of 25 songs are lofi, which means lofi users get a perfect top 3 but very few alternatives.

---

## 5. Strengths

- The system works well for users with a clear, single-genre preference — a "pop/happy" profile reliably surfaces the two pop tracks above everything else.
- Explanations are transparent: the user sees exactly why each song ranked where it did.
- The energy similarity calculation rewards closeness rather than just "high" or "low," which means a chill user is not penalized by accidentally getting loud songs.
- Genre-first ordering is predictable, which makes the system easy to reason about during testing.

---

## 6. Limitations and Bias

**Genre dominance (filter bubble risk):** Genre is worth 2.0 points, double any other feature. In a real deployment this would create a filter bubble — a jazz fan would almost never see a rock song, even if that rock song perfectly matched their energy and mood. Over time, this reinforces narrow taste rather than broadening it.

**Dataset genre imbalance:** 3 of 25 songs are lofi and 3 are pop. Users who prefer lofi or pop receive deeper, more varied recommendations than users who prefer metal or country, which each have only one or two representatives.

**No temporal or contextual awareness:** The system does not know whether the user is at the gym, studying, or going to sleep. A single profile is applied universally.

**Ignored features:** Tempo (bpm), valence, and danceability are loaded but never used in scoring. A song with a perfect genre/mood/energy match but an unpleasant tempo still scores the same as one with a pleasant tempo.

**Acoustic preference is binary:** `likes_acoustic` is True or False. There is no gradient — a user who "mostly" likes acoustic music is treated the same as one who only ever listens to acoustic music.

---

## 7. Evaluation

Three primary profiles were tested:

| Profile | Genre | Mood | Energy | Top Result |
|---------|-------|------|--------|------------|
| High-Energy Pop | pop | happy | 0.85 | Sunrise City (4.47) |
| Chill Lofi Study | lofi | chill | 0.38 | Library Rain (4.47) |
| Deep Intense Rock | rock | intense | 0.92 | Storm Runner (4.49) |

**What matched intuition:** All three top results "felt right." A high-energy pop profile getting Sunrise City, and a rock/intense profile getting Storm Runner, are sensible outcomes.

**What surprised me:** The "Chill Lofi Study" profile returned "After Midnight" (jazz) and "Spacewalk Thoughts" (ambient) in positions 4–5. Those are reasonable alternatives, but a real lofi listener might find jazz intrusive. The system correctly identifies mood/acoustic similarity but cannot distinguish the sonic texture of genres.

**Experiment — weight shift:** Halving the genre weight from 2.0 to 1.0 caused the pop profile to surface Block Party (hip-hop, happy, 0.79 energy) above Gym Hero (pop, intense). This reveals that at default weights, genre dominates; at reduced weights, mood and energy alignment drives the result.

**Experiment — feature removal:** Commenting out the mood check caused the chill lofi profile's rank 4 and 5 slots to shift from jazz/ambient to synthwave. Without mood, acousticness and energy alone decide, and synthwave's moderate acousticness no longer penalizes it.

---

## 8. Future Work

1. **Add diversity penalty:** Prevent any single artist or genre from appearing more than twice in the top 5. This would break filter bubbles without requiring collaborative data.
2. **Use tempo and valence in scoring:** A tempo-range preference (e.g., "I like 80–100 bpm") and a valence target would let the system differentiate "energetic but sad" from "energetic and euphoric" — two musically very different states.
3. **Context-aware profiles:** Allow the user to define multiple profiles (workout, study, wind-down) and select one at runtime, rather than using a single static profile.

---

## 9. Personal Reflection

The biggest surprise was how "smart" the recommendations feel even though the algorithm is only four arithmetic rules. Seeing "Library Rain" appear at the top of a chill lofi profile felt correct — until I realized the system had no idea what music sounds like. It just compared numbers. That gap between the algorithm's blindness and the output's apparent intelligence is exactly what makes AI systems both impressive and risky: the rules work until they don't, and it is not always obvious when they stop working.

Using AI tools to scaffold the code was genuinely helpful for boilerplate — generating the CSV reader, suggesting sorted() vs .sort(), and drafting docstrings. But I had to override the AI's first instinct on scoring more than once; it suggested a pure cosine similarity approach that would have hidden the per-feature reasoning in a single opaque number. Keeping the scoring transparent (genre match = always +2.0, visible in the output) was a deliberate human decision the AI did not push for on its own.

This project changed how I think about Spotify's "Discover Weekly." What feels like deep personalization is likely a much larger version of this same loop — score every song, sort, return top-k — plus collaborative filtering on billions of users. The feeling of being "understood" by the app is real, but it comes from the density of data and the size of the catalog, not from the algorithm understanding music the way a human does. Human judgment still matters enormously in deciding *which features to weight* and *how to prevent the system from trapping users in their existing taste*.
