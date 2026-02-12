# GeoQuiz Race — Game Design Document

## Concept

GeoQuiz Race is a geography quiz mode for InnerEarth where players test their knowledge of geographic extremes and scientific facts by physically walking toward answer locations on the globe.

Three answer locations appear as glowing colored spheres on Earth's surface. The player walks toward the one they believe is correct at a locked speed of 10,000 km/h. After 10 questions, stats and a leaderboard are shown.

## Game Flow

1. Player clicks "Play GeoQuiz Race" or presses `P`
2. Speed locks to 10,000 km/h, UI controls are hidden
3. A question appears at the top of the screen
4. Three colored answer spheres (red/green/blue) appear on Earth with labels
5. Player walks (WASD) toward their chosen answer
6. When within ~556 km (5°) of a sphere, the answer is registered
7. Correct/wrong feedback displays for 2.5 seconds
8. Next question appears (repeat for 10 questions)
9. Stats screen shows results and score
10. Score is saved to the leaderboard (top 10, localStorage)

## Controls

- **WASD / Arrow keys** — Walk toward answer spheres
- **P** — Start quiz
- **Escape** — Cancel quiz and return to exploration mode

## Scoring System

**Score = correct_answers × 1000 ÷ total_time_seconds**

This rewards both accuracy and speed. A perfect game (10/10 correct) in 60 seconds scores ~167. A fast perfect game in 30 seconds scores ~333.

## Question Bank (20 questions, 10 per game)

Questions focus on surprising geographic/scientific knowledge, not just landmarks.

| # | Question | Correct Answer | Wrong Answer 1 | Wrong Answer 2 |
|---|----------|---------------|----------------|----------------|
| 1 | Farthest point from Earth's center? | Chimborazo, Ecuador | Mount Everest, Nepal | Kilimanjaro, Tanzania |
| 2 | Closest city to antipode of Madrid? | Auckland, NZ | Sydney, Australia | Buenos Aires, Argentina |
| 3 | Driest place on Earth? | Atacama Desert, Chile | Sahara Desert, Libya | Death Valley, USA |
| 4 | Highest elevation capital? | La Paz, Bolivia | Quito, Ecuador | Bogota, Colombia |
| 5 | Deepest ocean point? | Mariana Trench | Puerto Rico Trench | Java Trench |
| 6 | Largest coral reef system? | Great Barrier Reef | Red Sea Reef | Mesoamerican Reef |
| 7 | Most remote inhabited island? | Tristan da Cunha | Easter Island | Pitcairn Island |
| 8 | Strait separating Africa/Europe? | Strait of Gibraltar | Strait of Sicily | Bosphorus Strait |
| 9 | Coldest inhabited place? | Oymyakon, Russia | Yellowknife, Canada | Longyearbyen, Svalbard |
| 10 | Highest waterfall? | Angel Falls, Venezuela | Victoria Falls | Niagara Falls |
| 11 | Largest freshwater lake (area)? | Lake Superior | Lake Victoria | Lake Baikal |
| 12 | Magnetic North Pole location? | Canadian Arctic | Geographic North Pole | Northern Greenland |
| 13 | Most active volcano? | Kilauea, Hawaii | Mount Etna, Sicily | Mount Vesuvius, Italy |
| 14 | Lowest land point? | Dead Sea | Death Valley | Turpan Depression |
| 15 | Wettest place on Earth? | Mawsynram, India | Mount Waialeale, Hawaii | Buenaventura, Colombia |
| 16 | Longest river? | Nile River, Egypt | Amazon River, Brazil | Yangtze River, China |
| 17 | Largest desert? | Antarctic Ice Sheet | Sahara Desert | Arabian Desert |
| 18 | Tectonic boundary creating Himalayas? | Indo-Australian/Eurasian | Pacific/North American | Nazca/South American |
| 19 | Strongest ocean current? | Antarctic Circumpolar | Gulf Stream | Kuroshio Current |
| 20 | Most lightning-struck place? | Lake Maracaibo, Venezuela | Lake Victoria, Africa | Singapore |

## Leaderboard System

- Stored in `localStorage` under key `innerearth_geoquiz_leaderboard`
- Top 10 entries, sorted by score descending
- Each entry: `{ score, correct, total, timeSec, date }`
- Accessible from the stats screen or by playing the quiz

## Technical Details

- Answer spheres are Three.js sprites using canvas-based textures with radial gradients
- Collision detection uses `angularDistance()` (Haversine formula) with a 5° threshold
- Spheres pulse via `Math.sin()` animation in the render loop
- Speed is locked by overriding `walkSpeedKmh` in `updateWalking()` each frame
- Quiz state machine prevents interactions with non-quiz UI during gameplay

## Future Expansion Ideas

- **Difficulty levels** — Easy (major cities), Medium (current), Hard (obscure locations)
- **Multiplayer** — Race against friends via WebRTC
- **Custom question packs** — User-created question sets
- **Timed mode** — Fixed time limit instead of per-question timing
- **Streak bonuses** — Consecutive correct answers multiply score
- **Sound effects** — Audio feedback for correct/wrong answers
- **Question categories** — Filter by topic (geology, climate, oceans, etc.)
- **Hint system** — Reveal one wrong answer after a time threshold
