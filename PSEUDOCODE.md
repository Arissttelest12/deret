# Pseudocode Game Deret Aritmatika

## 🎯 Main Game Class

```pseudocode
CLASS ArithmeticSequenceGame
    VARIABLES:
        level = 1
        nyawa = 5
        deret = empty array
        emptyPositions = empty array
        filledAnswers = empty object
        totalSlots = 10
        emptySlots = 3
        levelConfig = configuration object for each level
        dragged = null (for drag & drop)
    
    METHOD init():
        CALL setupLevel()
        CALL createParticles()
        CALL initializeAnimations()
        CALL render()
        SETUP event listeners
    
    METHOD setupLevel():
        SET deret = generateDeret()
        SET emptyPositions = generateEmptyPositions()
        SET filledAnswers = empty object
        LOG game state for debugging
    
    METHOD generateDeret():
        GET config = levelConfig[level] OR defaultConfig
        SET beda = config.beda
        SET [minStart, maxStart] = config.startRange
        SET startNumber = random number between minStart and maxStart
        
        INITIALIZE empty array deret
        SET current = startNumber
        
        FOR i from 0 to totalSlots - 1:
            ADD current to deret
            SET current = current + beda
        
        RETURN deret
    
    METHOD generateEmptyPositions():
        INITIALIZE array positions with values [0, 1, 2, ..., totalSlots-1]
        
        // Fisher-Yates shuffle
        FOR i from positions.length - 1 down to 1:
            SET j = random integer between 0 and i
            SWAP positions[i] with positions[j]
        
        RETURN first emptySlots elements of positions
    
    METHOD renderSlots():
        GET slotsDiv = document.getElementById("slots")
        CLEAR slotsDiv content
        
        FOR i from 0 to totalSlots - 1:
            CREATE new div element as slot
            
            IF i is in emptyPositions:
                SET slot class = "slot empty"
                SET slot id = "slot-" + i
                SET slot data-answer = deret[i]
                SET slot text = "?"
                ADD drag & drop event listeners to slot
            ELSE:
                SET slot class = "slot fixed"
                SET slot text = deret[i]
            
            SET initial animation state
            APPEND slot to slotsDiv
    
    METHOD renderChoices():
        GET choicesDiv = document.getElementById("choices")
        CLEAR choicesDiv content
        
        SET neededNumbers = emptyPositions.map(pos => deret[pos])
        SET distractors = generateDistractors(neededNumbers)
        SET allChoices = combine neededNumbers and distractors
        SHUFFLE allChoices randomly
        SET finalChoices = ensure all neededNumbers are included
        
        FOR each number in finalChoices:
            CREATE new div element as box
            SET box class = "box"
            SET box text = number
            SET box draggable = true
            SET box id = "box-" + number + "-" + index
            ADD dragstart event listener
            SET initial animation state
            APPEND box to choicesDiv
    
    METHOD generateDistractors(neededNumbers):
        INITIALIZE empty array distractors
        SET beda = levelConfig[level]?.beda OR (level + 1)
        
        FOR each number in neededNumbers:
            FOR each offset in [-beda, -beda+1, -1, 1, beda-1, beda]:
                SET distractor = number + offset
                IF distractor > 0 AND distractor not in neededNumbers AND distractor not in deret:
                    ADD distractor to distractors
        
        REMOVE duplicates from distractors
        RETURN first 6 elements of distractors
    
    METHOD ensureAllNeededNumbers(neededNumbers, allChoices):
        SET missing = neededNumbers.filter(number not in allChoices)
        RETURN combine allChoices with missing
    
    // Drag & Drop Methods
    METHOD dragStart(e):
        SET dragged = e.target
        SET e.target opacity = 0.5
        SET e.target transform = scale(0.9)
    
    METHOD dragOver(e):
        PREVENT default behavior
        ADD 'drag-over' class to e.target
    
    METHOD dragLeave(e):
        REMOVE 'drag-over' class from e.target
    
    METHOD drop(e):
        GET slot = e.target
        REMOVE 'drag-over' class from slot
        
        SET nilai = parseInt(dragged.text)
        SET kunci = parseInt(slot.data-answer)
        
        IF nilai != kunci:
            CALL handleWrongAnswer(slot)
        ELSE:
            CALL handleCorrectAnswer(slot, nilai)
    
    METHOD handleWrongAnswer(slot):
        DECREMENT nyawa by 1
        CALL updateUI()
        
        // Animations
        ANIMATE hearts with scale effect
        ANIMATE slot with shake effect
        
        SHOW error alert with remaining lives
        
        IF nyawa == 0:
            CALL gameOver()
    
    METHOD handleCorrectAnswer(slot, nilai):
        SET slot text = nilai
        REMOVE 'empty' class from slot
        ADD 'correct' class to slot
        
        ANIMATE dragged with disappear effect
        
        ADD entry to filledAnswers with slot id and nilai
        CALL updateProgress()
        
        IF filledAnswers length == emptySlots:
            CALL levelComplete()
    
    METHOD updateProgress():
        SET progress = (filledAnswers length / emptySlots) * 100
        UPDATE progress bar width
    
    METHOD levelComplete():
        SHOW success animation
        SHOW success alert
        IF player confirms:
            CALL nextLevel()
    
    METHOD nextLevel():
        INCREMENT level by 1
        SET nyawa = minimum(nyawa + 1, 5) // Bonus life
        CALL setupLevel()
        CALL render()
    
    METHOD gameOver():
        SHOW game over alert
        IF player confirms retry:
            RESET level to 1
            RESET nyawa to 5
            CALL setupLevel()
            CALL render()
    
    METHOD updateUI():
        UPDATE level display
        UPDATE hearts display
    
    METHOD createParticles():
        // Create floating background particles
        FOR i from 0 to 14:
            CREATE particle element
            SET random position and size
            ADD to particles container
    
    METHOD initializeAnimations():
        // Animate UI elements on load
        ANIMATE game title
        ANIMATE subtitle
        ANIMATE info bar
        ANIMATE hint section
    
    METHOD render():
        CALL renderSlots()
        CALL renderChoices()
        CALL updateUI()
        
        // Animate slots appearance
        ANIMATE all slots with stagger effect
        
        // Animate choices appearance
        ANIMATE all boxes with stagger effect
```

## 🎮 Game Flow

```pseudocode
START GAME
    CREATE new ArithmeticSequenceGame instance
    CALL game.init()
    
    WAIT for player interaction
    
    ON drag start:
        CALL game.dragStart(event)
    
    ON drag over slot:
        CALL game.dragOver(event)
    
    ON drag leave slot:
        CALL game.dragLeave(event)
    
    ON drop on slot:
        CALL game.drop(event)
    
    ON game completion:
        CALL game.levelComplete()
    
    ON game over:
        CALL game.gameOver()

END GAME
```

## 📊 Level Configuration

```pseudocode
levelConfig = {
    1: { beda: 2,  startRange: [1, 10]  },
    2: { beda: 3,  startRange: [1, 15]  },
    3: { beda: 4,  startRange: [1, 20]  },
    4: { beda: 5,  startRange: [5, 25]  },
    5: { beda: 6,  startRange: [5, 30]  },
    6: { beda: 7,  startRange: [10, 35] },
    7: { beda: 8,  startRange: [10, 40] },
    8: { beda: 9,  startRange: [15, 45] },
    9: { beda: 10, startRange: [15, 50] },
    10: { beda: 11, startRange: [20, 55] }
}
```

## 🔧 Key Features

1. **Random Position Generation**: Posisi kosong selalu acak
2. **Smart Distractors**: Pilihan salah yang mirip dengan jawaban
3. **Progress Tracking**: Progress bar menunjukkan selesai berapa
4. **Level Scaling**: Semakin tinggi level, semakin sulit
5. **Life System**: Nyawa terbatas, game over jika habis
6. **Smooth Animations**: Transitions dan effects yang smooth
7. **Responsive Design**: Bisa dimainkan di berbagai ukuran layar