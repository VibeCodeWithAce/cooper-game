# Cooper - The Dog Runner: Workflow Diagram

## Game Flow Architecture

```mermaid
graph TD
    Start([Game Loaded]) --> Init[Initialize Game]
    Init --> Title[TITLE SCREEN State]

    Title --> |User Press Space/Tap| Zoom[ZOOM_TRANSITION State]

    Zoom --> BallThrow[Ball Throw Animation<br/>90 frames]
    BallThrow --> ZoomOut[Camera Zoom Out<br/>3.0x → 1.0x scale]
    ZoomOut --> Playing[PLAYING State]

    Playing --> GameLoop{Game Loop<br/>60 FPS}

    GameLoop --> UpdateSystems[Update Systems]
    UpdateSystems --> UpdateCooper[Update Cooper]
    UpdateCooper --> ApplyGravity[Apply Gravity]
    ApplyGravity --> CheckGround{On Ground?}
    CheckGround --> |Yes| ResetJump[Reset Jump State]
    CheckGround --> |No| ContinueFall[Continue Falling]
    ResetJump --> UpdateAnim
    ContinueFall --> UpdateAnim[Update Animation<br/>8-frame cycle]

    GameLoop --> UpdateBackground[Update Background Layers]
    UpdateBackground --> CloudScroll[Clouds: 0.2x speed]
    CloudScroll --> BuildingScroll[Buildings/Trees: 0.5x speed<br/>1680px pattern loop]
    BuildingScroll --> AirplaneScroll[Airplanes: 0.3x speed]
    AirplaneScroll --> GroundScroll[Ground: 1.0x speed]

    GameLoop --> UpdateObstacles[Update Obstacles]
    UpdateObstacles --> MoveObstacles[Move at gameSpeed]
    MoveObstacles --> RemoveOffscreen[Remove Offscreen]
    RemoveOffscreen --> CheckSpawn{Time to Spawn?}
    CheckSpawn --> |Yes| SpawnObstacle[Spawn New Obstacle<br/>Based on Level]
    CheckSpawn --> |No| SkipSpawn
    SpawnObstacle --> ObstacleTypes{Obstacle Type}
    ObstacleTypes --> |Level 1-2| BasicObs[Hydrant, Cone]
    ObstacleTypes --> |Level 3-4| MailboxObs[+ Mailboxes]
    ObstacleTypes --> |Level 5-6| BarrierObs[+ Barrier, Double Cone]
    ObstacleTypes --> |Level 7+| AllObs[+ Trash Bin]

    GameLoop --> UpdateScore[Update Score]
    UpdateScore --> AddPoints[+1 point per frame]
    AddPoints --> CheckLevel{Score % 500 == 0?}
    CheckLevel --> |Yes| LevelUp[Level Up!]
    LevelUp --> IncreaseSpeed[Increase gameSpeed]
    IncreaseSpeed --> CheckDayNight{Level % 3 == 0?}
    CheckDayNight --> |Yes| ToggleDayNight[Toggle Day/Night]
    CheckDayNight --> |No| ContinuePlay
    CheckLevel --> |No| ContinuePlay[Continue Playing]

    GameLoop --> CollisionCheck[Collision Detection]
    CollisionCheck --> CheckHitbox{Cooper hits<br/>Obstacle?}
    CheckHitbox --> |Yes| GameOver[GAME_OVER State]
    CheckHitbox --> |No| SafePlay

    GameOver --> SaveHighScore{Score > High Score?}
    SaveHighScore --> |Yes| UpdateHighScore[Save to localStorage]
    SaveHighScore --> |No| ShowGameOver
    UpdateHighScore --> ShowGameOver[Display Game Over Screen]
    ShowGameOver --> WaitInput[Wait for User Input]
    WaitInput --> |Press Space/Tap| Title

    GameLoop --> Render[Render Frame]
    Render --> DrawBackground[Draw Background<br/>Sky, Clouds, Buildings, Ground]
    DrawBackground --> DrawObstacles[Draw Obstacles]
    DrawObstacles --> DrawCooper[Draw Cooper<br/>with Motion Blur]
    DrawCooper --> DrawUI[Draw UI<br/>Level, Score, High Score]
    DrawUI --> NextFrame[Request Next Frame]
    NextFrame --> GameLoop

    Playing --> |Space/Tap| Jump{Cooper Can Jump?}
    Jump --> |Not Jumping| ApplyJumpForce[Apply Jump Force<br/>velocityY = -15]
    Jump --> |Already Jumping| IgnoreInput[Ignore Input]
    ApplyJumpForce --> ContinuePlay
    IgnoreInput --> ContinuePlay

    SafePlay --> ContinuePlay
    SkipSpawn --> ContinuePlay
    ToggleDayNight --> ContinuePlay
    BasicObs --> ContinuePlay
    MailboxObs --> ContinuePlay
    BarrierObs --> ContinuePlay
    AllObs --> ContinuePlay

    style Start fill:#87CEEB
    style Title fill:#FFD700
    style Zoom fill:#FFA500
    style Playing fill:#00FF00
    style GameOver fill:#FF4444
    style GameLoop fill:#4A90E2
    style CollisionCheck fill:#DC143C
    style LevelUp fill:#FFD700
```

## System Interaction Diagram

```mermaid
graph LR
    User[User Input<br/>Space/Tap] --> InputHandler[Input Handler]

    InputHandler --> StateManager{State Manager}

    StateManager --> |TITLE| StartGame[Start Zoom Transition]
    StateManager --> |PLAYING| JumpSystem[Jump System]
    StateManager --> |GAME_OVER| ResetGame[Reset to Title]

    JumpSystem --> CooperPhysics[Cooper Physics Engine]
    CooperPhysics --> Gravity[Gravity System<br/>0.8 per frame]
    Gravity --> GroundCollision[Ground Collision<br/>y = 302]

    GameClock[Game Clock<br/>requestAnimationFrame] --> UpdateCycle[Update Cycle]
    UpdateCycle --> ScoreSystem[Score System<br/>+1 per frame]
    UpdateCycle --> ObstacleSystem[Obstacle System]
    UpdateCycle --> BackgroundSystem[Background System<br/>Parallax Scrolling]
    UpdateCycle --> PhysicsSystem[Physics System]

    ScoreSystem --> LevelSystem[Level System<br/>500 pts = Level Up]
    LevelSystem --> SpeedSystem[Speed System<br/>+0.6 speed per level]
    LevelSystem --> DayNightCycle[Day/Night Cycle<br/>Every 3 levels]

    ObstacleSystem --> SpawnManager[Spawn Manager<br/>Gap: 400-700px]
    SpawnManager --> ObstaclePool[Obstacle Pool<br/>Level-based types]

    PhysicsSystem --> CooperPhysics
    PhysicsSystem --> ObstacleMovement[Obstacle Movement<br/>Move at gameSpeed]

    CooperPhysics --> CollisionDetector[Collision Detector]
    ObstacleMovement --> CollisionDetector
    CollisionDetector --> |Hit Detected| GameOverTrigger[Trigger Game Over]
    CollisionDetector --> |Safe| ContinueGame[Continue Game]

    UpdateCycle --> RenderPipeline[Render Pipeline]
    RenderPipeline --> Layer1[Layer 1: Sky Gradient]
    Layer1 --> Layer2[Layer 2: Clouds 0.2x]
    Layer2 --> Layer3[Layer 3: Airplanes 0.3x]
    Layer3 --> Layer4[Layer 4: Buildings/Trees 0.5x<br/>14-element pattern]
    Layer4 --> Layer5[Layer 5: Ground 1.0x]
    Layer5 --> Layer6[Layer 6: Obstacles]
    Layer6 --> Layer7[Layer 7: Cooper + Motion Blur]
    Layer7 --> Layer8[Layer 8: UI Text]
    Layer8 --> Display[Display to Canvas]

    GameOverTrigger --> HighScoreCheck[High Score Manager<br/>localStorage]
    HighScoreCheck --> StateManager

    style User fill:#FFD700
    style GameClock fill:#4A90E2
    style CollisionDetector fill:#DC143C
    style Display fill:#00FF00
```

## Data Flow Diagram

```mermaid
flowchart TD
    Config[CONFIG Constants<br/>Sizes, Speeds, Physics] --> GameState[Game State Variables<br/>score, level, gameSpeed, isDayTime]

    GameState --> Cooper[Cooper Object<br/>x, y, velocityY, isJumping, animFrame]
    GameState --> Obstacles[Obstacles Array<br/>x, y, width, height, type]
    GameState --> Background[Background Layers<br/>clouds, buildings, airplanes offsets]

    UserInput[User Input Events] --> ActionHandler{Action Handler}
    ActionHandler --> |TITLE| TransitionToZoom[Set State: ZOOM_TRANSITION<br/>Init ball animation]
    ActionHandler --> |PLAYING| JumpAction[If !isJumping: velocityY = -15]
    ActionHandler --> |GAME_OVER| ResetAction[Reset game variables<br/>Set State: TITLE]

    GameLoopTick[Game Loop Tick] --> UpdatePhase[Update Phase]
    UpdatePhase --> CooperUpdate[Cooper Update<br/>velocityY += gravity<br/>y += velocityY]
    CooperUpdate --> CooperGround{y >= groundY?}
    CooperGround --> |Yes| CooperLand[y = groundY<br/>velocityY = 0<br/>isJumping = false]
    CooperGround --> |No| CooperAir[Continue in air]

    UpdatePhase --> ObstacleUpdate[Obstacle Update<br/>x -= gameSpeed<br/>Remove if x < -width]
    ObstacleUpdate --> SpawnCheck{nextObstacleDistance <= 0?}
    SpawnCheck --> |Yes| CreateObstacle[Create new obstacle<br/>Type based on level<br/>Reset distance: 400-700]
    SpawnCheck --> |No| NoSpawn[Continue]

    UpdatePhase --> BackgroundUpdate[Background Update<br/>Offset += speed * parallaxFactor<br/>Offset %= patternLength]

    UpdatePhase --> ScoreUpdate[Score Update<br/>score += 1<br/>Check level threshold]
    ScoreUpdate --> LevelCheck{score % 500 == 0?}
    LevelCheck --> |Yes| LevelUpAction[level += 1<br/>gameSpeed += 0.6<br/>Show level up timer<br/>Toggle day/night if level % 3]
    LevelCheck --> |No| ScoreOnly[Continue scoring]

    UpdatePhase --> CollisionPhase[Collision Phase]
    CollisionPhase --> HitboxCalc[Calculate Hitboxes<br/>Cooper: +4px margin<br/>Obstacles: +2px margin]
    HitboxCalc --> BoxCollisionCheck{Box Collision?}
    BoxCollisionCheck --> |Yes| TriggerGameOver[Set State: GAME_OVER<br/>Update high score if needed]
    BoxCollisionCheck --> |No| SafeContinue[Continue game]

    GameLoopTick --> RenderPhase[Render Phase]
    RenderPhase --> ClearCanvas[Clear Canvas]
    ClearCanvas --> DrawSky[Draw Sky Gradient<br/>Day vs Night colors]
    DrawSky --> DrawClouds[Draw Clouds<br/>Based on offset]
    DrawClouds --> DrawAirplanes[Draw Airplanes + Banners<br/>"BUILT WITH CLAUDE"]
    DrawAirplanes --> DrawBuildings[Draw Buildings/Trees/Signs<br/>14-element repeating pattern<br/>Deterministic windows]
    DrawBuildings --> DrawGround[Draw Ground + Tile Lines]
    DrawGround --> DrawAllObstacles[Draw All Active Obstacles<br/>Fire hydrant, mailbox, cone, etc]
    DrawAllObstacles --> DrawCooperSprite[Draw Cooper Sprite<br/>Motion blur layers<br/>8-frame leg animation]
    DrawCooperSprite --> DrawHUD[Draw HUD<br/>Level, Score, High Score]
    DrawHUD --> CheckLevelUpTimer{levelUpTimer > 0?}
    CheckLevelUpTimer --> |Yes| DrawLevelUp[Draw "LEVEL UP!" text<br/>With fade animation]
    CheckLevelUpTimer --> |No| FrameComplete[Frame Complete]
    DrawLevelUp --> FrameComplete

    FrameComplete --> NextTick[Request Next Animation Frame]
    NextTick --> GameLoopTick

    TriggerGameOver --> HighScoreStorage[localStorage.setItem<br/>'cooperHighScore']

    style Config fill:#87CEEB
    style GameLoopTick fill:#4A90E2
    style CollisionPhase fill:#DC143C
    style LevelUpAction fill:#FFD700
    style TriggerGameOver fill:#FF4444
```

## Game State Machine

```mermaid
stateDiagram-v2
    [*] --> TITLE: Game Init

    TITLE --> ZOOM_TRANSITION: User Press Space/Tap

    ZOOM_TRANSITION --> PLAYING: Zoom Complete<br/>(90 frames)

    PLAYING --> PLAYING: Game Loop<br/>Update & Render<br/>User Jump Input

    PLAYING --> GAME_OVER: Collision Detected

    GAME_OVER --> TITLE: User Press Space/Tap

    note right of TITLE
        - Display Cooper icon (2.5x scale)
        - Show title "COOPER"
        - Show "THE DOG RUNNER"
        - Show "How to Play" card
        - Show high score
        - Show "Built with Claude"
        - Wait for user input
    end note

    note right of ZOOM_TRANSITION
        - Ball throw animation (flat trajectory)
        - Camera zoom out (3.0x → 1.0x)
        - Background parallax active
        - Duration: 90 frames (~1.5s)
    end note

    note right of PLAYING
        - Update Cooper physics
        - Spawn & move obstacles
        - Update parallax background
        - Check collisions
        - Update score (1 pt/frame)
        - Level up every 500 pts
        - Day/night toggle every 3 levels
        - User can jump (if grounded)
    end note

    note right of GAME_OVER
        - Display final score
        - Display high score
        - Show "NEW HIGH SCORE!" if beaten
        - Darken background overlay
        - Wait for user input to restart
    end note
```

## Key Systems Overview

### 1. **Physics System**
- Gravity: 0.8 pixels/frame²
- Jump Force: -15 pixels/frame
- Ground Y: 302 pixels
- Cooper moves vertically only (horizontal is simulated by scrolling background)

### 2. **Parallax Scrolling System**
- **Clouds**: 0.2x speed, 400px loop
- **Airplanes**: 0.3x speed, random respawn
- **Buildings/Trees**: 0.5x speed, 1680px pattern (14 elements × 120px)
- **Ground**: 1.0x speed, 40px tile loop

### 3. **Obstacle System**
- Spawn gap: 400-700px (randomized)
- Level progression unlocks new obstacle types
- Types: Fire hydrant, mailbox (blue/green), cone, barrier, trash bin, double cone
- All move at `gameSpeed` (base 6 + level × 0.6)

### 4. **Scoring & Progression**
- Score: +1 point per frame (~60 pts/second)
- Level up: Every 500 points
- Speed increase: +0.6 per level
- Day/Night cycle: Every 3 levels

### 5. **Collision Detection**
- Hitbox margins: Cooper +4px, Obstacles +2px
- Simple AABB (Axis-Aligned Bounding Box) collision
- Single collision = Game Over

### 6. **Rendering Pipeline**
8 layers rendered per frame:
1. Sky gradient (day/night)
2. Clouds (parallax 0.2x)
3. Airplanes with banners (parallax 0.3x)
4. Buildings/Trees/Stop Signs/Traffic Lights (parallax 0.5x)
5. Ground with tile lines (1.0x)
6. Obstacles (foreground)
7. Cooper sprite with motion blur (foreground)
8. UI text (overlay)

---

**Game Performance**: ~200-400 draw calls per frame, targeting 60 FPS
