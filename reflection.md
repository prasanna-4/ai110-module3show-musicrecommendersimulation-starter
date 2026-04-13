# Reflection: Profile Comparison Notes

## High-Energy Pop vs. Chill Lofi Study

The pop profile and the lofi profile share almost no overlap in their top 5 results. The pop profile's top songs (Sunrise City, Gym Hero) have energy above 0.75 and acousticness below 0.2. The lofi profile's top songs (Library Rain, Midnight Coding) have energy below 0.45 and acousticness above 0.70. This makes sense: the two genres sit at opposite ends of both the energy and acousticness scales, so the scoring function correctly separates them. What it reveals is that if a user enjoys *both* high-energy pop and chill lofi depending on context, this system would only serve one of those moods at a time — it has no concept of situational listening.

## Chill Lofi Study vs. Deep Intense Rock

The chill lofi profile rewards low energy (0.38 target) and acoustic sound, while the rock profile rewards high energy (0.92 target) and non-acoustic production. Their recommendation lists are completely disjoint — not a single song appears in both top 5s. This is expected and correct. However, both profiles surface "honorable mention" songs from other genres at slots 4 and 5 (ambient and jazz for lofi; pop and electronic for rock). This shows the energy-similarity score is doing real work: when no more genre matches are available, songs with similar energy bubble up regardless of genre. A lofi listener might not appreciate ambient music, and a rock listener might not want pop, but the system cannot make that distinction without more genre-specific constraints.

## High-Energy Pop vs. Deep Intense Rock

Both profiles favor non-acoustic, high-energy tracks. Their top results share no titles, but several artists overlap (Max Pulse appears in both). The key difference is mood: the pop profile rewards "happy" and the rock profile rewards "intense." Without the mood check, these two profiles would converge on nearly the same list. This comparison shows that mood is the primary differentiator between two otherwise energy-matched profiles — and it only contributes 1.0 point versus genre's 2.0 points. In a larger catalog with more intense pop songs, the distinction would likely break down.

## Edge Case: Conflicting Preferences (energy: 0.9, mood: sad)

Testing a "high energy but sad" profile revealed a gap in the model. The system added up energy points and mood points independently, so it returned intense rock and metal tracks (which scored well on energy) over sad folk or ambient tracks (which scored well on mood). The resulting list felt emotionally "wrong" — the songs were loud and aggressive, not introspective. A real recommender might handle this with weighted mood-energy interaction terms or by using genre subgenres (e.g., "sad rock" vs. "angry rock"), but our simple additive model has no way to represent that nuance.
