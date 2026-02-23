Original prompt: okay lets proceed with bounty circuit the aim is to use the assets as a concept and most remix games should optimized for mobile device with a 2:3 aspect ratio and need the game to be a single page html , css and javascript game cause thats what remix wants and can you tell me the assets you would want to make use of  if you're building the bounty circuit so i can send the links of each assets so you can integrate it into the game 1. Add this script tag to load the SDK in the head section:
<script src="https://cdn.jsdelivr.net/npm/@farcade/game-sdkindex.min.js"></script>

2. Add these SDK calls to the game:

// When the game is over (replace scoreValue with the actual score):
window.FarcadeSDK.singlePlayer.actions.gameOver({ score: scoreValue }); and need you integrate the game with the sdk also

// For haptic feedback on important interactions (like jumping or hitting obstacles):
window.FarcadeSDK.hapticFeedback();

// Add this to handle play again requests:
window.FarcadeSDK.onPlayAgain(() => {
  // Reset the game state here
  // For example: resetGame(), startNewGame(), etc.
});

// REQUIRED: Add this to handle mute/unmute - MUST be included even if the game doesn't currently have sound.
// This is non-negotiable and required for integration checks to pass.
// If the game has no audio, you can leave the callback empty or add a placeholder:
window.FarcadeSDK.onToggleMute((data) => {
  // Set game audio based on data.isMuted
  // For example: setMuted(data.isMuted)
  // If game has no sound, you can leave this empty but the handler MUST exist
});

3. Call gameOver when the player loses or completes the game.
4. Add haptic feedback for important game events.
5. IMPORTANT: The toggle_mute handler (window.FarcadeSDK.onToggleMute) MUST be included in your code, even if the game has no sound. This is required for integration checks and cannot be skipped.

Notes:
- Initialized project with Kenney Desert Shooter asset pack present locally.
- Need single-page mobile-first (2:3) game with integrated Farcade SDK hooks.

TODO:
- Build index.html with inline CSS/JS and single canvas gameplay.
- Implement Bounty Circuit loop: menu, 5 contracts, upgrades, win/lose states.
- Wire SDK gameOver/haptic/onPlayAgain/onToggleMute.
- Expose render_game_to_text and advanceTime hooks.
- Run Playwright-based validation loop if available.

Update:
- Created index.html single-page game scaffold with inline CSS/JS.
- Implemented Bounty Circuit core loop (menu, 5 contracts, enemies, scoring, upgrade selection, win/lose).
- Added Farcade SDK hooks: gameOver, hapticFeedback, onPlayAgain, onToggleMute.
- Added deterministic test hooks: window.render_game_to_text and window.advanceTime.
- Added local asset manifest at top of script for easy URL replacement.

Validation:
- Ran Playwright client against local server after implementing game.
- Initial run in sandbox failed due browser launch permissions; reran with elevated permissions successfully.
- Reviewed screenshots from output/web-game/shot-0.png..shot-2.png and state dumps state-0.json..state-2.json.
- No console/page error artifacts were produced in output/web-game.
- Applied visual tuning for terrain tile choices and sprite sizing; reran Playwright and rechecked screenshots/state.
- Attempted long timeout simulation; deterministic loop ended on menu state in that scenario, so run-end automation verification remains limited in this environment.

Open TODOs / suggestions:
- Re-validate gameOver callback firing in hosted Farcade environment where SDK is loaded.
- Replace local asset paths in ASSETS manifest with hosted URLs when provided.
- Optionally tune contract pacing (enemy threat / timer) after first external playtest.

Major refactor (campaign build):
- Replaced prior 5-contract loop with a 20-mission campaign architecture.
- Added data-driven level table (20 levels) with mission names, mission types, district themes, timer budgets, enemy budgets, and unique seeds.
- Implemented five objective types: eliminate, uplink activation, demolition, survive, and intel recovery + extraction.
- Added mission-specific entities and logic (terminals, destructible caches, intel pickups, extraction zone).
- Added persistent run build and weapon progression system with upgrades between missions.
- Added expanded map generation with deterministic district maps per level (roads, larger buildings, props, obstacles, per-level variation).
- Preserved single-page structure and Farcade SDK integration hooks.

Visual and gameplay tuning:
- Tuned map generation to reduce clutter (fewer/larger buildings, fewer cover blocks, cleaner road palette).
- Added spawn-point deduplication and anti-overlap enemy spawn checks.
- Kept menu layout/style and interaction model; updated copy to represent 20 missions.

Validation runs:
- Playwright run: output/web-game-v2 (3 iterations), reviewed screenshots + state JSON, no errors files generated.
- Menu-only run: output/web-game-v2-menu (1 iteration), verified preserved menu style and mode=menu state.
- Longer interaction run: output/web-game-v2-long (1 iteration), verified sustained gameplay stability and objective progress without console/page errors.

Open TODOs / suggestions:
- Playtest balance pass for early-mission survivability (Mission 1 can be punishing with no early upgrades).
- Optional: reduce enemy contact damage slightly for first 3 missions.
- Optional: add chapter intro cards or minimap for stronger mission readability.
- Re-verify Farcade gameOver callback in hosted environment where SDK is guaranteed loaded.

Map overhaul pass (post user map feedback):
- Reworked district rendering into authored layout templates (`TOWN_LAYOUTS`) with deterministic avenues/streets/connectors/plazas per level style.
- Added district-specific visual themes (`DISTRICT_THEMES`) controlling ground/path/patch palettes, road overlays, and map colors.
- Replaced noisy per-tile random map scatter with block-based lot construction and pattern-driven building placement.
- Added stronger road semantics via `roadMask` plus road tint/edge/lane overlays in `bakeMapCanvas`.
- Added landmark placement in plazas and rebuilt spawn-node selection from road intersections + edge injectors.
- Switched building rendering from full perimeter stripe tiling to cleaner roof-mass rendering with restrained facade accents.
- Fixed a major artifact: blocked thin `1-tile` building rectangles (`placeBuildingRect` now requires both width/height >= 3), which removed maze-like stripe clutter.

Latest validation runs:
- `output/web-game-v4`: initial post-refactor screenshots/state inspection showed continued visual striping.
- `output/web-game-v5`: first structured-layout pass, still visually busy.
- `output/web-game-v6`: simplified building facades + palette rebalance.
- `output/web-game-v7`: wall tile cleanup pass.
- `output/web-game-v8`: thin-building artifact fix verification.
- `output/web-game-v9`: final palette/map readability pass; roads now read as coherent lanes and menu is unchanged.
- `output/web-game-v9-menu`: menu-only check confirms preserved start screen style.

Current status:
- 20-mission campaign, objective types, upgrades, and Farcade SDK integration remain intact.
- Map visuals are significantly cleaner and more intentional than previous procedural-noise versions.

Remaining optional tuning:
- If more “hand-authored city” feel is desired, replace procedural lot patterns for first 3-5 missions with fully hand-crafted tile templates.
- Consider objective-aware micro landmarks per mission type (e.g., distinct uplink compounds vs cache depots) for stronger mission identity.

Handcrafted map implementation (all 20 missions):
- Added `HANDCRAFTED_LEVEL_LAYOUTS` with explicit per-mission map blueprints (avenues, streets, connectors, plazas, and district patch zones).
- Updated `generateTownMap(level)` to prefer handcrafted blueprint by `level.id` and only fall back to `TOWN_LAYOUTS` if missing.
- Removed layout jitter for handcrafted levels (no random shifts), keeping each mission map stable and authored.
- Added deterministic terrain patch painter and blueprint-driven patch placement.
- Made lot/building generation deterministic under handcrafted mode (no random block skipping).
- Added blueprint hooks for handcrafted `covers` and `props` plus deterministic clutter constraints.
- Kept mission logic, upgrade flow, and Farcade SDK integration unchanged.

Visual tuning after handcrafted pass:
- Switched road tile set in district themes to concrete-style tiles (`tile_0100`, `tile_0101`, `tile_0102`) for clearer street readability.

Validation runs for handcrafted pass:
- `output/web-game-v10`: verified gameplay + mission transition to Level 2 (`state-2.json` shows `level:2`, `mode:gameover`).
- `output/web-game-v11`: verified handcrafted map renders with concrete roads and stable collisions over 3 iterations.
- `output/web-game-v11b`: longer run sanity-check on Mission 1 state progression and sustained map stability.

Remaining optional follow-up:
- Add explicit handcrafted building rectangles per mission (currently road/plaza skeletons are handcrafted; building fill remains deterministic lot-based).
- Add mission-type-specific landmark compounds (uplink hubs, cache depots, extraction plazas) as bespoke tile compositions per mission.

Explicit building-footprint pass (requested follow-up):
- Added `HANDCRAFTED_BUILDING_FOOTPRINTS` (20 mission entries, one explicit footprint set per mission).
- Updated `generateTownMap(level)` to map mission-specific building footprints into `layout.buildings` for handcrafted levels.
- Building placement flow now prioritizes explicit handcrafted footprints first (`manual` style) and only uses procedural lot autofill as fallback when too few handcrafted footprints can be placed.
- Kept collision model unchanged (buildings still become `state.map.obstacles` via `placeBuildingRect`).

Validation for explicit-footprint pass:
- `output/web-game-v12` (3-iteration gameplay run):
  - Mission 1 completes and enters upgrade mode.
  - Mission 2 loads and is playable with handcrafted footprints visible.
  - State confirms transition to `level: 2` and stable objective tracking.
- Verified `HANDCRAFTED_BUILDING_FOOTPRINTS` contains 20 top-level mission arrays.

Notes:
- This pass delivers explicit per-mission building footprints while preserving handcrafted roads/plazas and existing mission systems.

Gameplay polish pass (objective clarity + enemy behavior + manual fire):
- Replaced player auto-fire with manual fire input:
  - Added `controls.shoot` + touch shoot button (upper-right) and keyboard fire keys (`J/K/X`, plus `B` for automation tests).
  - `updatePlayer` now fires only when shoot input is held and weapon cooldown allows it.
- Added mission objective clarity:
  - Added start-of-level mission briefing card (`state.levelBriefTimer`) showing mission name + objective description + goal progress.
  - HUD objective line now states objective description plus progress (`Eliminate all hostiles • Hostiles 3/5`).
  - Menu controls text updated to reflect manual fire + mission briefing.
- Reduced early-level aggressiveness:
  - Spawn pressure reduced for early levels in `setupMission` (lower initial + reserve counts).
  - `updateSpawns` now uses lower early max enemy cap and slower early spawn pacing.
  - Early enemy contact damage scaled down.
- Enemy aggression and pathing rework:
  - Enemies now have proximity-based aggro (`alertTimer`) and only become aggressive when player is near (or when hit).
  - Non-alert enemies patrol lightly instead of hard-chasing/shooting.
  - Added tile-grid pathfinding helpers (`buildWalkGrid`, `findTilePath`) and stepped collision movement (`resolveMoveStepped`) to stop wall clipping/tunneling and route enemies around obstacles.
  - Enemy movement now follows path waypoints to player instead of only direct-line pushes.

Validation artifacts:
- Manual fire + aggression/pathing run: `output/web-game-v13`.
  - State confirms manual firing and objective progression (`Hostiles 0/5` -> `Hostiles 4/5`) without auto-fire.
- Mission brief visibility run: `output/web-game-v13-brief`.
  - Screenshot shows briefing card at level start and objective text.
- No `errors-*.json` produced in these runs.

Support files added:
- `test-actions-manual-fire.json`
- `test-actions-brief.json`

Post-compaction validation pass:
- Re-ran Playwright manual-fire scenario using local HTTP serving and the develop-web-game client.
- Latest artifacts landed in `output/web-game/` (client ignored `--output-dir` in this invocation).
- Reviewed `shot-0.png`..`shot-2.png`: objective line is clear (`Eliminate all hostiles`), map collision visuals are coherent, and manual-fire gameplay progression is visible (`Hostiles 0/5` -> `4/5`).
- Reviewed `state-0.json`..`state-2.json`: mission description and progress fields are present and synced with HUD.
- No `errors-*.json` files generated in `output/web-game/`.

Open note:
- If needed, normalize all future test runs to a dedicated output folder by checking the exact supported CLI flag name for the Playwright client script.
