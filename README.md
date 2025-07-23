<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Prompt Element Sorter</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8;
            color: #2d3748;
            line-height: 1.6;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }
        .container {
            max-width: 1400px;
            flex-grow: 1;
        }
        .card {
            background-color: #ffffff;
            border-radius: 0.75rem; /* rounded-xl */
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            border: 1px solid #e2e8f0;
        }
        .jumbled-statement {
            cursor: grab;
            background-color: #e0f2fe; /* blue-100 */
            color: #0b5e8c; /* blue-800 */
            border: 1px solid #90cdf4; /* blue-300 */
            padding: 0.75rem 1rem;
            border-radius: 0.5rem;
            font-weight: 500;
            margin-bottom: 0.75rem;
            flex-shrink: 0;
            transition: transform 0.1s ease-out, opacity 0.1s ease-out, background-color 0.2s;
            touch-action: none; /* For touch devices */
        }
        .jumbled-statement:active {
            cursor: grabbing;
        }
        .jumbled-statement.dragged {
            opacity: 0.5;
        }
        .jumbled-statement.placed {
            opacity: 0.7; /* Slightly faded when placed */
            /* Removed: cursor: default; and pointer-events: none; */
        }
        .jumbled-statement.correct {
            background-color: #d1fae5; /* green-100 */
            border-color: #38a169; /* green-600 */
            color: #10b981; /* green-500 */
        }
        .jumbled-statement.incorrect {
            background-color: #fee2e2; /* red-100 */
            border-color: #ef4444; /* red-500 */
            color: #ef4444; /* red-500 */
        }

        .drop-zone-element {
            min-height: 120px; /* Min height for each drop zone */
            border: 2px dashed #cbd5e0; /* gray-300 */
            background-color: #f8fafc; /* gray-50 */
            border-radius: 0.5rem;
            padding: 1rem;
            margin-bottom: 1rem;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start; /* Align dropped items to top */
            text-align: center;
            transition: border-color 0.2s, background-color 0.2s;
            position: relative; /* For placeholder */
        }
        .drop-zone-element.active {
            border-color: #4c51bf; /* indigo-700 */
            background-color: #e0e7ff; /* indigo-100 */
        }
        .drop-zone-placeholder {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #94a3b8; /* gray-400 */
            font-style: italic;
        }

        .btn {
            padding: 0.75rem 1.5rem;
            border-radius: 0.5rem;
            font-weight: 600;
            transition: all 0.2s ease-in-out;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }
        .btn:hover {
            opacity: 0.9;
            transform: translateY(-1px);
        }
        .btn:active {
            transform: translateY(0);
        }
        .btn-primary {
            background-color: #4c51bf; /* indigo-700 */
            color: white;
        }
        .btn-secondary {
            background-color: #a0aec0; /* gray-400 */
            color: #2d3748;
        }
        .btn-success {
            background-color: #38a169; /* green-600 */
            color: white;
        }
        .feedback-message {
            padding: 1rem;
            border-radius: 0.5rem;
            margin-top: 1rem;
            font-weight: 600;
            border: 1px solid;
        }
        .feedback-success {
            background-color: #d1fae5;
            color: #10b981;
            border-color: #38a169;
        }
        .feedback-warning {
            background-color: #ffedd5;
            color: #ea580c;
            border-color: #f97316;
        }
        .feedback-error {
            background-color: #fee2e2;
            color: #ef4444;
            border-color: #dc2626;
        }
        .modal {
            display: none; /* Hidden by default */
            position: fixed; /* Stay in place */
            z-index: 1000; /* Sit on top */
            left: 0;
            top: 0;
            width: 100%; /* Full width */
            height: 100%; /* Full height */
            overflow: auto; /* Enable scroll if needed */
            background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
            justify-content: center;
            align-items: center;
        }
        .modal-content {
            background-color: #fefefe;
            margin: auto;
            padding: 2.5rem;
            border-radius: 0.75rem;
            box-shadow: 0 8px 16px rgba(0,0,0,0.2);
            width: 90%;
            max-width: 700px;
            color: #2d3748;
        }
        .close-button {
            color: #aaa;
            float: right;
            font-size: 28px;
            font-weight: bold;
        }
        .close-button:hover,
        .close-button:focus {
            color: black;
            text-decoration: none;
            cursor: pointer;
        }
    </style>
</head>
<body class="p-6">
    <div class="container mx-auto py-8 flex flex-col items-center">
        <h1 class="text-4xl font-bold text-center text-indigo-800 mb-6">Prompt Element Sorter</h1>
        <p class="text-center text-gray-700 mb-8 max-w-3xl">Drag the prompt statements from the right into their correct "Prompt Element" categories on the left. Build a complete and effective prompt!</p>

        <!-- Game Info / Status Bar -->
        <div class="w-full card p-4 mb-8 flex justify-between items-center text-lg font-semibold">
            <span id="round-display" class="text-indigo-700">Scenario 1</span>
            <div class="flex items-center gap-2">
                <span class="text-gray-600">Time:</span>
                <span id="timer-display" class="text-red-600">00:00</span>
            </div>
            <div class="flex items-center gap-2">
                <span class="text-gray-600">Score:</span>
                <span id="score-display" class="text-green-600">0</span>
            </div>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 w-full flex-grow">
            <!-- Left Column: Prompt Element Drop Zones -->
            <div id="prompt-element-dropzones" class="lg:col-span-1 grid grid-cols-1 md:grid-cols-2 gap-4 auto-rows-min">
                <!-- Drop zones will be dynamically rendered here -->
            </div>

            <!-- Right Column: Scenario & Jumbled Statements -->
            <div class="lg:col-span-1 flex flex-col gap-6">
                <div class="card p-6">
                    <h2 class="text-2xl font-semibold mb-3 text-indigo-700">Current Scenario</h2>
                    <p id="scenario-text" class="text-gray-800 mb-2"></p>
                    <p id="task-prompt-text" class="text-gray-600 text-sm italic"></p>
                </div>

                <div class="card p-6 flex flex-col items-center flex-grow">
                    <h2 class="text-2xl font-semibold mb-4 text-indigo-700">Jumbled Prompt Statements</h2>
                    <div id="jumbled-statements-container" class="flex flex-col items-center w-full max-h-96 overflow-y-auto pr-2">
                        <!-- Jumbled statements will be injected here by JS -->
                    </div>
                </div>

                <div class="flex justify-center gap-4 mt-4">
                    <button id="check-prompt-btn" class="btn btn-primary">Check Prompt!</button>
                    <button id="next-scenario-btn" class="btn btn-secondary hidden">Next Scenario</button>
                </div>
            </div>
        </div>
    </div>

    <!-- Debrief Modal -->
    <div id="debrief-modal" class="modal">
        <div class="modal-content">
            <span class="close-button" id="close-debrief-modal">&times;</span>
            <h2 class="text-3xl font-bold text-indigo-800 mb-4">Round Results!</h2>
            <h3 class="text-xl font-semibold mb-3 text-gray-700">Score: <span id="modal-round-score" class="text-green-600"></span> points</h3>
            <div id="debrief-feedback-content" class="mb-6 text-gray-700 space-y-3">
                <!-- Feedback on correct/incorrect/missed will go here -->
            </div>
            <div class="flex justify-end mt-4">
                <button id="modal-next-scenario-btn" class="btn-primary px-6 py-2 rounded-lg font-bold">Continue</button>
            </div>
        </div>
    </div>

    <script>
        const promptElementsData = [
            { id: 'context', name: 'Contextual Clarity', placeholder: 'Background & Scenario' },
            { id: 'audience', name: 'Audience & Persona', placeholder: 'Who is the output for? AI\'s role?' },
            { id: 'purpose', name: 'Purpose & Goal', placeholder: 'Why are we doing this? Main objective?' },
            { id: 'constraints', name: 'Constraints & Limitations', placeholder: 'What to avoid/include (e.g., bias, length)?' },
            { id: 'format', name: 'Format & Structure', placeholder: 'How should the output look?' },
            { id: 'tone', name: 'Tone & Style', placeholder: 'How should the output sound?' },
            { id: 'evaluation', name: 'Evaluation Criteria', placeholder: 'How will success be judged?' }
        ];

        const scenarios = [
            {
                id: 1,
                scenario: "You're building an AI assistant to summarize user feedback for a new mobile app feature. The app allows users to easily track their daily water intake.",
                taskPrompt: "Your goal is to get a summary focusing on user experience, not just bug reports. The summary needs to be concise for busy product managers.",
                timerSeconds: 90,
                jumbledStatements: [
                    { id: 's1-1', text: 'The AI should act as a neutral data analyst.', correctElementId: 'audience' },
                    { id: 's1-2', text: 'Analyze reviews from users aged 25-45 who use fitness trackers.', correctElementId: 'audience' },
                    { id: 's1-3', text: 'The summary should be presented in 3-5 bullet points.', correctElementId: 'format' },
                    { id: 's1-4', text: 'Identify positive and negative sentiment regarding usability.', correctElementId: 'evaluation' },
                    { id: 's1-5', text: 'Do not include any suggestions for new features.', correctElementId: 'constraints' },
                    { id: 's1-6', text: 'Focus on how the feature integrates into daily routines.', correctElementId: 'context' },
                    { id: 's1-7', text: 'The main goal is to identify core UX pain points and delights.', correctElementId: 'purpose' },
                ],
            },
            {
                id: 2,
                scenario: "Your team is drafting user persona archetypes for a new mental wellness app. The app aims to support diverse needs, and it's crucial to avoid any pre-existing stereotypes.",
                taskPrompt: "Create 3 distinct user persona archetypes for a mental wellness app.",
                timerSeconds: 120,
                jumbledStatements: [
                    { id: 's2-1', text: 'Ensure personas are gender-neutral and avoid ageism.', correctElementId: 'constraints' },
                    { id: 's2-2', text: 'Focus on motivations, pain points, and goals related to mental health.', correctElementId: 'purpose' },
                    { id: 's2-3', text: 'The output is for UX designers and content creators.', correctElementId: 'audience' },
                    { id: 's2-4', text: 'Each persona should include a short narrative and 3 key characteristics.', correctElementId: 'format' },
                    { id: 's2-5', text: 'Consider users seeking support for stress, anxiety, or general well-being.', correctElementId: 'context' },
                    { id: 's2-6', text: 'The tone should be empathetic and non-judgmental.', correctElementId: 'tone' },
                    { id: 's2-7', text: 'Success means personas accurately reflect diverse user needs.', correctElementId: 'evaluation' },
                ],
            },
            {
                id: 3,
                scenario: "You need to generate ideas for **UI element variations** for a 'confirm purchase' button. The key challenge is to ensure these variations consider **accessibility for users with cognitive impairments**.",
                taskPrompt: "Suggest 4 distinct UI element variations for a 'confirm purchase' button.",
                timerSeconds: 100,
                jumbledStatements: [
                    { id: 's3-1', text: 'The suggestions are for a web development team.', correctElementId: 'audience' },
                    { id: 's3-2', text: 'Focus on clear, unambiguous language and reduced cognitive load.', correctElementId: 'constraints' },
                    { id: 's3-3', text: 'The goal is to prevent accidental purchases by easily confused users.', correctElementId: 'purpose' },
                    { id: 's3-4', text: 'Provide a brief explanation for each variation.', correctElementId: 'format' },
                    { id: 's3-5', text: 'Ensure options for varied visual contrast and simplified interactions.', correctElementId: 'evaluation' },
                    { id: 's3-6', text: 'Consider users who might struggle with complex interfaces or multiple options.', correctElementId: 'context' },
                    { id: 's3-7', text: 'The tone should be helpful and solution-oriented.', correctElementId: 'tone' },
                ]
            }
        ];

        // Game state variables
        let currentScenarioIndex = -1;
        let totalScore = 0;
        let timerId;
        let timeRemaining;
        // Map<elementId (string), Set<statementId (string)>>
        // This maps the dropzone element ID to a Set of statement IDs currently in it.
        let droppedStatementsMap = new Map();
        let currentJumbledStatements = []; // Array of {id, text, correctElementId} for current scenario
        let draggableStatementsInPlay = new Set(); // Stores IDs of statements NOT currently in a dropzone (i.e., still in the jumbled container)

        // HTML Element references
        const promptElementDropzones = document.getElementById('prompt-element-dropzones');
        const scenarioText = document.getElementById('scenario-text');
        const taskPromptText = document.getElementById('task-prompt-text');
        const jumbledStatementsContainer = document.getElementById('jumbled-statements-container');
        const timerDisplay = document.getElementById('timer-display');
        const scoreDisplay = document.getElementById('score-display');
        const roundDisplay = document.getElementById('round-display');
        const checkPromptBtn = document.getElementById('check-prompt-btn');
        const nextScenarioBtn = document.getElementById('next-scenario-btn');

        // Debrief Modal elements
        const debriefModal = document.getElementById('debrief-modal');
        const closeDebriefModalBtn = document.getElementById('close-debrief-modal');
        const modalRoundScore = document.getElementById('modal-round-score');
        const debriefFeedbackContent = document.getElementById('debrief-feedback-content');
        const modalNextScenarioBtn = document.getElementById('modal-next-scenario-btn');

        // --- Game Initialization ---
        function initGame() {
            renderPromptElementDropzones(); // Render static drop zones once
            addGlobalDragListeners(); // Attach drag-and-drop event listeners
            checkPromptBtn.addEventListener('click', endScenario);
            nextScenarioBtn.addEventListener('click', loadNextScenario);
            closeDebriefModalBtn.addEventListener('click', () => debriefModal.style.display = 'none');
            modalNextScenarioBtn.addEventListener('click', () => {
                debriefModal.style.display = 'none';
                loadNextScenario();
            });

            loadNextScenario(); // Start the first scenario
        }

        // --- Render UI Elements ---

        function renderPromptElementDropzones() {
            promptElementDropzones.innerHTML = '';
            promptElementsData.forEach(element => {
                const div = document.createElement('div');
                div.id = `drop-zone-${element.id}`;
                div.className = 'drop-zone-element card p-4';
                div.setAttribute('data-element-id', element.id); // Custom attribute for element ID
                div.innerHTML = `
                    <h4 class="font-bold text-indigo-800 text-lg mb-2">${element.name}</h4>
                    <p class="drop-zone-placeholder text-sm">${element.placeholder}</p>
                `;
                promptElementDropzones.appendChild(div);
            });
        }

        function renderJumbledStatements(statements) {
            jumbledStatementsContainer.innerHTML = '';
            draggableStatementsInPlay.clear(); // Clear tracking set for new round

            statements.forEach(statement => {
                const div = document.createElement('div');
                div.id = `statement-${statement.id}`;
                div.className = 'jumbled-statement w-11/12';
                div.draggable = true; // Statements are always draggable
                div.textContent = statement.text;
                div.setAttribute('data-statement-id', statement.id);
                div.setAttribute('data-correct-element-id', statement.correctElementId);
                jumbledStatementsContainer.appendChild(div);
                draggableStatementsInPlay.add(statement.id); // Initially all are in play
            });
        }

        function updateScoreDisplay() {
            scoreDisplay.textContent = totalScore;
        }

        function updateTimerDisplay() {
            const minutes = Math.floor(timeRemaining / 60);
            const seconds = timeRemaining % 60;
            timerDisplay.textContent = `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
        }

        function resetDropzones() {
            droppedStatementsMap.clear(); // Clear all tracked dropped statements
            promptElementDropzones.querySelectorAll('.drop-zone-element').forEach(dropZone => {
                // Remove all child elements that are jumbled statements
                Array.from(dropZone.children).forEach(child => {
                    if (child.classList.contains('jumbled-statement')) {
                        child.remove();
                    }
                });
                dropZone.classList.remove('active');
                // Ensure placeholder is visible for empty dropzones
                dropZone.querySelector('.drop-zone-placeholder').classList.remove('hidden');
            });
        }

        function resetJumbledStatementsContainer() {
            jumbledStatementsContainer.innerHTML = ''; // Clear all statements from the jumbled container
            draggableStatementsInPlay.clear(); // Clear tracking set
             jumbledStatementsContainer.style.outline = ''; // Remove any active outline
        }

        // --- Game Flow Control ---

        function loadNextScenario() {
            stopTimer();
            currentScenarioIndex++;
            if (currentScenarioIndex >= scenarios.length) {
                endGame();
                return;
            }

            const scenario = scenarios[currentScenarioIndex];
            // Clone statements to ensure clean state for each round
            currentJumbledStatements = scenario.jumbledStatements.map(stmt => ({ ...stmt }));

            resetDropzones();
            resetJumbledStatementsContainer(); // Clear old statements from the jumbled container

            scenarioText.textContent = scenario.scenario;
            taskPromptText.textContent = scenario.taskPrompt;
            roundDisplay.textContent = `Scenario ${currentScenarioIndex + 1}/${scenarios.length}`;

            renderJumbledStatements(currentJumbledStatements); // Populate with new statements

            checkPromptBtn.classList.remove('hidden');
            nextScenarioBtn.classList.add('hidden');

            timeRemaining = scenario.timerSeconds;
            updateTimerDisplay();
            startTimer();
        }

        function startTimer() {
            timerId = setInterval(() => {
                timeRemaining--;
                updateTimerDisplay();
                if (timeRemaining <= 0) {
                    clearInterval(timerId);
                    endScenario();
                }
            }, 1000);
        }

        function stopTimer() {
            if (timerId) {
                clearInterval(timerId);
                timerId = null;
            }
        }

        function endScenario() {
            stopTimer();
            checkPromptBtn.classList.add('hidden');
            nextScenarioBtn.classList.remove('hidden');

            // Evaluate placements
            evaluateScenario();
            showDebriefModal();
        }

        function endGame() {
            alert(`Game Over! You completed all scenarios. Your final score is: ${totalScore} points!`);
            // Reset game for replay, or direct to a final score screen
            totalScore = 0;
            currentScenarioIndex = -1;
            loadNextScenario(); // Start from the beginning
        }

        // --- Drag & Drop Logic ---

        function addGlobalDragListeners() {
            let draggedStatementId = null;
            let draggedFromParentId = null; // To track where the element came from (dropzone ID or 'jumbled-statements-container')

            document.addEventListener('dragstart', (e) => {
                if (e.target.classList.contains('jumbled-statement')) {
                    draggedStatementId = e.target.dataset.statementId;
                    e.dataTransfer.setData('text/plain', draggedStatementId);

                    // Determine if dragging from an existing drop zone or the initial container
                    const parentElement = e.target.parentElement;
                    if (parentElement.classList.contains('drop-zone-element')) {
                        draggedFromParentId = parentElement.dataset.elementId;
                    } else if (parentElement.id === 'jumbled-statements-container') {
                        draggedFromParentId = 'jumbled-statements-container';
                    }
                    e.dataTransfer.setData('text/parent-id', draggedFromParentId); // Store source parent ID

                    e.target.classList.add('dragged');
                }
            });

            document.addEventListener('dragend', (e) => {
                if (e.target.classList.contains('jumbled-statement')) {
                    e.target.classList.remove('dragged');
                }
                draggedStatementId = null;
                draggedFromParentId = null;
            });

            // Make all drop zones, including the initial jumbled statements container, valid drop targets
            document.querySelectorAll('.drop-zone-element, #jumbled-statements-container').forEach(dropTarget => {
                dropTarget.addEventListener('dragover', (e) => {
                    e.preventDefault(); // Allow drop
                    if (dropTarget.classList.contains('drop-zone-element')) {
                        dropTarget.classList.add('active'); // Visual cue for drop zone
                    } else if (dropTarget.id === 'jumbled-statements-container') {
                         // Optional: Add visual cue for jumbled container
                         dropTarget.style.outline = '2px dashed #4c51bf';
                    }
                });

                dropTarget.addEventListener('dragleave', () => {
                    if (dropTarget.classList.contains('drop-zone-element')) {
                        dropTarget.classList.remove('active');
                    } else if (dropTarget.id === 'jumbled-statements-container') {
                        dropTarget.style.outline = '';
                    }
                });

                dropTarget.addEventListener('drop', (e) => {
                    e.preventDefault();
                    if (dropTarget.classList.contains('drop-zone-element')) {
                        dropTarget.classList.remove('active');
                    } else if (dropTarget.id === 'jumbled-statements-container') {
                        dropTarget.style.outline = '';
                    }

                    const droppedId = e.dataTransfer.getData('text/plain');
                    const sourceParentId = e.dataTransfer.getData('text/parent-id');
                    const droppedElement = document.getElementById(`statement-${droppedId}`);

                    if (droppedElement) {
                        // Clean up from previous location if it was in a drop zone
                        if (sourceParentId && sourceParentId !== 'jumbled-statements-container') { // If it came from a categorized dropzone
                            if (droppedStatementsMap.has(sourceParentId)) {
                                droppedStatementsMap.get(sourceParentId).delete(droppedId);
                                // If the source dropzone becomes empty, show its placeholder
                                const sourceDropZone = document.getElementById(`drop-zone-${sourceParentId}`);
                                if (sourceDropZone && droppedStatementsMap.get(sourceParentId).size === 0) {
                                    sourceDropZone.querySelector('.drop-zone-placeholder').classList.remove('hidden');
                                }
                            }
                        }

                        // Remove correct/incorrect styling when moved (so they are fresh for re-evaluation)
                        droppedElement.classList.remove('correct', 'incorrect');

                        // Append the element to the new target
                        dropTarget.appendChild(droppedElement);

                        if (dropTarget.classList.contains('drop-zone-element')) {
                            // Dropped into a prompt element drop zone (categorized)
                            droppedElement.classList.add('placed'); // Mark as placed in a category
                            // It's draggable by default, no need to set draggable=true here
                            dropTarget.querySelector('.drop-zone-placeholder').classList.add('hidden'); // Hide placeholder

                            const targetElementId = dropTarget.dataset.elementId;
                            if (!droppedStatementsMap.has(targetElementId)) {
                                droppedStatementsMap.set(targetElementId, new Set());
                            }
                            droppedStatementsMap.get(targetElementId).add(droppedId);

                            draggableStatementsInPlay.delete(droppedId); // No longer in the initial pool
                        } else if (dropTarget.id === 'jumbled-statements-container') {
                            // Dropped back into the initial jumbled statements container (uncategorized)
                            droppedElement.classList.remove('placed'); // Not categorized
                            // It's draggable by default, no need to set draggable=true here
                            draggableStatementsInPlay.add(droppedId); // Back in the pool
                        }
                    }
                });
            });
        }

        // --- Evaluation Logic ---

        function evaluateScenario() {
            let roundScore = 0;
            debriefFeedbackContent.innerHTML = '';

            const correctStatementsPlaced = new Set();
            const incorrectStatementsPlaced = new Set();
            const missedStatements = new Set();

            // First, apply visual feedback based on current placement
            currentJumbledStatements.forEach(stmt => {
                const statementElement = document.getElementById(`statement-${stmt.id}`);
                statementElement.classList.remove('correct', 'incorrect'); // Reset visual state

                // Check if the statement is in *any* dropzone
                const currentParent = statementElement.parentElement;
                if (currentParent && currentParent.classList.contains('drop-zone-element')) {
                    const droppedIntoElementId = currentParent.dataset.elementId;
                    if (droppedIntoElementId === stmt.correctElementId) {
                        correctStatementsPlaced.add(stmt.id);
                        statementElement.classList.add('correct');
                    } else {
                        incorrectStatementsPlaced.add(stmt.id);
                        statementElement.classList.add('incorrect');
                    }
                } else {
                    // Statement is still in jumbled-statements-container or otherwise unplaced
                    missedStatements.add(stmt.id);
                    statementElement.classList.add('incorrect'); // Mark as incorrect if not placed correctly
                }
            });


            // Score calculation based on the sets
            currentJumbledStatements.forEach(stmt => {
                if (correctStatementsPlaced.has(stmt.id)) {
                    roundScore += 20; // Points for correct placement
                } else if (incorrectStatementsPlaced.has(stmt.id)) {
                    roundScore -= 10; // Penalty for incorrect placement
                } else if (missedStatements.has(stmt.id)) {
                    roundScore -= 5; // Penalty for missed
                }
            });

            // Debrief Modal Feedback
            // Correctly Placed
            if (correctStatementsPlaced.size > 0) {
                const div = document.createElement('div');
                div.className = 'bg-green-50 p-3 rounded-md text-green-800 border border-green-200';
                div.innerHTML = `<p class="font-bold">✅ Correctly Placed (${correctStatementsPlaced.size}/${currentJumbledStatements.length}):</p>`;
                correctStatementsPlaced.forEach(id => {
                    const stmt = currentJumbledStatements.find(s => s.id === id);
                    const element = promptElementsData.find(e => e.id === stmt.correctElementId);
                    div.innerHTML += `<p class="text-sm ml-2">- "${stmt.text}" was correctly placed in "${element.name}".</p>`;
                });
                debriefFeedbackContent.appendChild(div);
            }

            // Incorrectly Placed
            if (incorrectStatementsPlaced.size > 0) {
                const div = document.createElement('div');
                div.className = 'bg-red-50 p-3 rounded-md text-red-800 border border-red-200';
                div.innerHTML = `<p class="font-bold">❌ Incorrect Placements (${incorrectStatementsPlaced.size}):</p>`;
                incorrectStatementsPlaced.forEach(id => {
                    const stmt = currentJumbledStatements.find(s => s.id === id);
                    const actualDropZoneElement = document.getElementById(`statement-${id}`).parentElement;
                    let droppedIntoElementName = "Unknown Location"; // Default if something goes wrong
                    if (actualDropZoneElement.classList.contains('drop-zone-element')) {
                        const droppedIntoElement = promptElementsData.find(e => e.id === actualDropZoneElement.dataset.elementId);
                        droppedIntoElementName = droppedIntoElement ? droppedIntoElement.name : "Unknown Category";
                    } else if (actualDropZoneElement.id === 'jumbled-statements-container') {
                        droppedIntoElementName = "Jumbled Statements (Not Categorized)";
                    }

                    const correctElement = promptElementsData.find(e => e.id === stmt.correctElementId);
                    div.innerHTML += `<p class="text-sm ml-2">- "${stmt.text}" was placed in "${droppedIntoElementName}", but it belongs in "${correctElement.name}".</p>`;
                });
                debriefFeedbackContent.appendChild(div);
            }

            // Missed Statements
            if (missedStatements.size > 0) {
                const div = document.createElement('div');
                div.className = 'bg-yellow-50 p-3 rounded-md text-yellow-800 border border-yellow-200';
                div.innerHTML = `<p class="font-bold">⚠️ Missed Statements (${missedStatements.size}):</p>`;
                missedStatements.forEach(id => {
                    const stmt = currentJumbledStatements.find(s => s.id === id);
                    const correctElement = promptElementsData.find(e => e.id === stmt.correctElementId);
                    div.innerHTML += `<p class="text-sm ml-2">- "${stmt.text}" was not placed. It belongs in "${correctElement.name}".</p>`;
                });
                debriefFeedbackContent.appendChild(div);
            }

            if (correctStatementsPlaced.size === currentJumbledStatements.length) {
                const div = document.createElement('div');
                div.className = 'bg-indigo-50 p-3 rounded-md text-indigo-800 border border-indigo-200 mt-4';
                div.innerHTML = `<p class="font-bold">✨ Perfect! All statements correctly categorized.</p>`;
                debriefFeedbackContent.appendChild(div);
            }


            // Bonus for time remaining
            if (timeRemaining > 0) {
                const timeBonus = Math.floor(timeRemaining / 5); // 1 point for every 5 seconds remaining
                roundScore += timeBonus;
                const div = document.createElement('div');
                div.className = 'bg-blue-50 p-3 rounded-md text-blue-800 border border-blue-200 mt-4';
                div.innerHTML = `<p class="font-bold">⏱️ Time Bonus: +${timeBonus} points for ${timeRemaining} seconds remaining!</p>`;
                debriefFeedbackContent.appendChild(div);
            }

            roundScore = Math.max(0, roundScore); // Ensure score doesn't go negative for the round
            totalScore += roundScore;
            updateScoreDisplay();
            modalRoundScore.textContent = roundScore;
        }

        function showDebriefModal() {
            debriefModal.style.display = 'flex'; // Use flex to center
        }

        // Initialize the game when the DOM is fully loaded
        document.addEventListener('DOMContentLoaded', initGame);
    </script>
</body>
</html>
```
