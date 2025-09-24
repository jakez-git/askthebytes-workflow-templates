# Core Battle System Specification v0.1

## Document Status & Blockers

**Version**: 0.1  
**Status**: Initial Draft  
**Last Updated**: September 24, 2025  
**Document Type**: GEMINI-TARGETED Technical Specification  

### Current Blockers

- **BLOCKER-001**: Gem combo calculation complexity needs algorithmic validation
- **BLOCKER-002**: Mana generation rate balancing requires gameplay testing
- **BLOCKER-003**: Turn timer implementation pending UI/UX coordination
- **BLOCKER-004**: Damage calculation overflow protection needs implementation
- **BLOCKER-005**: Battle state synchronization across client/server requires networking architecture

### Related Sources

**Trello References**: 
- [Battle System Design Board](https://trello.com/b/battle-system) - Core mechanics planning
- [Game Balance Cards](https://trello.com/b/game-balance) - Damage/mana tuning parameters

**NotebookLM Sources**:
- Match-3 Genre Analysis Report - Comparative mechanics study
- RPG Battle Systems Documentation - Turn-based combat patterns
- Mobile Game Balancing Guidelines - F2P engagement metrics

---

## 1. Match-3 Gem Logic

### 1.1 Core Matching Rules

**Match Detection Algorithm**:
```
FOR each position (x,y) on 8x8 grid:
  CHECK horizontal matches: minimum 3 consecutive identical gems
  CHECK vertical matches: minimum 3 consecutive identical gems
  IDENTIFY all connected gem groups of same type
  MARK matches for simultaneous removal
```

**Gem Types & Properties**:
- **RED**: Fire damage gems, generate Fire mana
- **BLUE**: Water damage gems, generate Water mana  
- **GREEN**: Earth damage gems, generate Earth mana
- **YELLOW**: Air damage gems, generate Air mana
- **PURPLE**: Dark damage gems, generate Dark mana
- **WHITE**: Light damage gems, generate Light mana
- **SKULL**: Direct damage gems, no mana generation
- **SPECIAL**: Power-up gems (bombs, multipliers)

### 1.2 Move Resolution Sequence

**Phase 1**: Player Move Validation
```
1. VALIDATE move legality (adjacent swap, creates match)
2. EXECUTE gem swap
3. SCAN for immediate matches
4. IF no matches found: REVERT move, turn ends
5. IF matches found: PROCEED to Phase 2
```

**Phase 2**: Match Processing Loop
```
WHILE matches exist:
  1. CALCULATE damage/mana from current matches
  2. REMOVE matched gems
  3. APPLY gravity (gems fall down)
  4. GENERATE new gems at top
  5. SCAN for new matches formed by falling gems
  6. INCREMENT cascade counter
END WHILE
```

### 1.3 Cascade & Combo System

**Cascade Multipliers**:
- Cascade 1 (initial match): 1x multiplier
- Cascade 2: 1.25x multiplier
- Cascade 3: 1.5x multiplier
- Cascade 4+: 2.0x multiplier (max)

**Combo Scoring**:
```
Base Points = Match Size × Gem Value × Cascade Multiplier
Combo Bonus = (Total Gems Cleared - 3) × 10
Final Score = Base Points + Combo Bonus
```

**Special Match Patterns**:
- **4-Match Line**: Creates Bomb gem (destroys 3x3 area)
- **5-Match Line**: Creates Lightning gem (destroys entire row/column)
- **L/T Shape**: Creates Star gem (destroys all gems of target color)
- **5+ Match**: Creates Rainbow gem (destroys any adjacent color)

---

## 2. Combat Mechanics

### 2.1 Damage Calculation

**Base Damage Formula**:
```
Raw Damage = (Gems Matched × Base Gem Value × Cascade Multiplier)
Hero Bonus = Raw Damage × (Hero Attack Stat / 100)
Final Damage = (Raw Damage + Hero Bonus) × Element Bonus × Crit Multiplier
```

**Element Interactions**:
- Fire > Earth > Water > Fire (1.5x damage)
- Light <> Dark (mutual 1.5x damage)
- Air neutral to all elements (1.0x damage)

**Critical Hit System**:
- Base crit chance: 5%
- Hero crit stat increases chance
- Crit damage: 2.0x multiplier
- Special gems have increased crit chance

### 2.2 MOB (Monster) Mechanics

**MOB Types**:
- **Grunt**: Low HP, basic attacks, no special abilities
- **Elite**: Medium HP, one special ability, resistance to one element
- **Boss**: High HP, multiple abilities, complex AI patterns

**MOB AI Behavior**:
```
IF MOB health < 25%:
  PRIORITY: Defensive abilities or healing
ELSE IF player has type advantage:
  PRIORITY: Neutralize advantage or shield
ELSE:
  PRIORITY: Maximum damage dealing
```

**MOB Special Abilities**:
- **Armor**: Reduces incoming damage by fixed amount
- **Shield**: Absorbs next X points of damage
- **Poison**: Deals damage over time
- **Stun**: Skip player's next turn
- **Drain**: Steal mana from player

### 2.3 Hero System

**Hero Stats**:
- **Attack**: Increases damage multiplier
- **Defense**: Reduces incoming damage
- **Health**: Total HP pool
- **Mana**: Ability resource pool
- **Speed**: Turn order determination
- **Crit**: Critical hit chance bonus

**Hero Abilities**:
```
Ability Categories:
- DAMAGE: Direct attack spells (cost: 15-30 mana)
- UTILITY: Board manipulation (cost: 10-20 mana)
- HEALING: HP restoration (cost: 20-25 mana)
- BUFF/DEBUFF: Status effects (cost: 15-35 mana)
```

---

## 3. Turn & Mana System

### 3.1 Turn Structure

**Player Turn Sequence**:
```
1. REFRESH: Regenerate 2 mana points
2. INPUT: Accept player move (60-second timer)
3. RESOLVE: Process match-3 move and cascades
4. ABILITIES: Player can cast hero abilities
5. STATUS: Apply ongoing effects (poison, buffs)
6. END: Transfer turn to opponent/AI
```

**Turn Timer Logic**:
- Standard turn: 60 seconds
- Complex board state: 90 seconds
- Final turn warning: 10 seconds remaining
- Timeout consequence: Random valid move executed

### 3.2 Mana Generation

**Mana Generation Rules**:
```
Per Gem Matched:
- Basic 3-match: 1 mana of gem's element
- 4-gem match: 2 mana + create special gem
- 5+ gem match: 3 mana + create ultra gem
- Skull gems: 0 mana, direct damage only
```

**Mana Storage & Limits**:
- Maximum mana per element: 50 points
- Overflow mana: Converted to experience points
- Cross-element abilities: Cost mana from multiple pools

### 3.3 Victory/Loss Conditions

**Victory Conditions**:
1. **MOB Defeat**: Reduce enemy HP to 0
2. **Survival**: Last longer than time limit (survival mode)
3. **Objective**: Complete specific battle goals
4. **Forfeit**: Opponent surrenders

**Loss Conditions**:
1. **Hero Death**: Player HP reaches 0
2. **Timeout**: Fail to meet time objectives
3. **Surrender**: Player forfeits match
4. **Disconnect**: Network failure beyond timeout threshold

**End Game Processing**:
```
1. CALCULATE final scores and rewards
2. UPDATE player statistics and progression
3. SAVE battle log for analysis
4. RETURN to appropriate game mode menu
```

---

## 4. RNG & Tiebreaker Systems

### 4.1 Random Number Generation

**RNG Sources**:
- **Board Generation**: Deterministic seed + entropy
- **Critical Hits**: Mersenne Twister algorithm
- **Gem Drops**: Weighted probability tables
- **AI Decisions**: Controlled randomness with strategic bias

**RNG Validation**:
```
Each RNG call must:
1. LOG seed and generated value
2. VALIDATE range boundaries
3. RECORD for replay consistency
4. AUDIT for fair distribution
```

### 4.2 Tiebreaker Resolution

**Damage Tiebreakers**:
1. Player with higher combo count wins
2. If tied: Player with more recent move wins
3. If simultaneous: Higher hero level wins
4. Final fallback: Coin flip (seeded RNG)

**Turn Priority**:
- Higher Speed stat goes first
- Speed ties: Random determination
- Status effects can modify turn order

---

## 5. Battle Telemetry & Logging

### 5.1 Required Battle Data

**Move-Level Telemetry**:
```
CAPTURE per move:
- Timestamp (milliseconds precision)
- Board state before/after
- Player input coordinates
- Gems matched and cascade sequence
- Damage/mana calculations
- RNG seed values used
- Performance metrics (response time)
```

**Session-Level Telemetry**:
```
CAPTURE per battle:
- Total battle duration
- Victory/loss outcome
- Final scores and statistics
- Hero/MOB configurations used
- Network stability metrics
- Client device performance data
```

### 5.2 Log Call Specifications

**Critical Log Points**:
```
LOG_BATTLE_START(player_id, enemy_id, config)
LOG_MOVE_INPUT(coordinates, timestamp, board_hash)
LOG_MATCH_RESULT(gems_matched, damage_dealt, mana_gained)
LOG_CASCADE_EVENT(cascade_level, new_matches)
LOG_ABILITY_CAST(ability_id, mana_cost, target)
LOG_STATUS_EFFECT(effect_type, duration, target)
LOG_BATTLE_END(winner, final_stats, duration)
```

**Error Logging**:
```
ERROR_INVALID_MOVE(attempted_move, reason)
ERROR_DESYNC_DETECTED(client_state, server_state)
ERROR_TIMEOUT_OCCURRED(timeout_type, context)
ERROR_CALCULATION_OVERFLOW(calculation_type, values)
```

**Analytics Integration**:
- Battle outcomes feed into matchmaking algorithm
- Move patterns inform AI difficulty tuning
- Performance data guides optimization priorities
- Player behavior influences monetization features

---

## 6. Next Phase Action Items

### 6.1 Test Oracle Development

**SUBTASK-001**: Create unit test suite for damage calculation edge cases
- Priority: HIGH
- Estimated Effort: 5 days
- Dependencies: Math library validation
- Deliverable: Test suite covering overflow, underflow, precision loss

**SUBTASK-002**: Build cascade simulation framework
- Priority: HIGH  
- Estimated Effort: 8 days
- Dependencies: Board state modeling
- Deliverable: Automated cascade testing with known outcomes

**SUBTASK-003**: Implement RNG reproducibility testing
- Priority: MEDIUM
- Estimated Effort: 3 days
- Dependencies: Logging infrastructure
- Deliverable: Replay system for exact battle recreation

### 6.2 API Integration Hooks

**SUBTASK-004**: Design battle state API endpoints
- Priority: HIGH
- Estimated Effort: 6 days
- Dependencies: Backend architecture decisions
- Deliverable: RESTful API specification document

**SUBTASK-005**: Create real-time battle synchronization protocol
- Priority: HIGH
- Estimated Effort: 10 days
- Dependencies: WebSocket infrastructure
- Deliverable: Multi-client state consistency system

**SUBTASK-006**: Build telemetry data pipeline
- Priority: MEDIUM
- Estimated Effort: 7 days
- Dependencies: Analytics platform selection
- Deliverable: Automated data ingestion and processing

### 6.3 Edge Case Analysis

**SUBTASK-007**: Map all possible board deadlock scenarios
- Priority: MEDIUM
- Estimated Effort: 4 days
- Dependencies: Board generation algorithm
- Deliverable: Deadlock detection and resolution system

**SUBTASK-008**: Define network interruption recovery procedures
- Priority: HIGH
- Estimated Effort: 5 days
- Dependencies: Client-server architecture
- Deliverable: Graceful disconnection and reconnection handling

**SUBTASK-009**: Analyze memory usage patterns for long battles
- Priority: LOW
- Estimated Effort: 3 days
- Dependencies: Performance profiling tools
- Deliverable: Memory optimization recommendations

### 6.4 Integration Dependencies

- **UI/UX Team**: Turn timer visual indicators, damage number animations
- **Backend Team**: Database schema for battle logs, leaderboard systems
- **DevOps Team**: Telemetry data storage and retention policies
- **QA Team**: Battle simulation testing framework requirements
- **Analytics Team**: KPI definitions for battle engagement metrics

---

## Document Revision History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 0.1 | 2025-09-24 | Initial specification draft | System |

## Related Documentation

- [Game Design Document v2.1](../01-strategic/game-design.md) - High-level game vision
- [Technical Architecture Spec](../02-specifications/tech-architecture.md) - System integration requirements  
- [Player Progression System](../01-design/progression-spec.md) - Hero advancement mechanics
- [Monetization Strategy](../01-strategic/monetization-plan.md) - Revenue model integration points

**END SPECIFICATION**
