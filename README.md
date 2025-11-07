<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SNAP Study Progress Tracker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom CSS for the Circular Progress Bar */
        .progress-ring-container {
            width: 80px;
            height: 80px;
            position: relative;
        }

        .progress-ring {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            transform: rotate(-90deg);
        }

        .progress-ring__circle {
            transition: stroke-dashoffset 0.35s;
            transform: rotate(-90deg);
            transform-origin: 50% 50%;
        }
    </style>
</head>
<body class="bg-gray-50 font-sans p-4 sm:p-8">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                }
            }
        }

        const SUBJECTS = {
            'qa': {
                name: 'Quantitative Aptitude (QA) & DI',
                topics: [
                    "Arithmetic (Percentages, P&L, Ratio, Avg)", 
                    "Time & Work / TSD (Trains & Boats)", 
                    "SI & CI / Mixtures & Allegations",
                    "Number System (Divisibility, Remainders, Series)", 
                    "Surds and Indices / Logarithm",
                    "Simplification & Mathematical Reasoning",
                    "Linear & Quadratic Equations / Inequalities",
                    "Progressions (AP, GP, HP)",
                    "Functions & Set Theory",
                    "Mensuration (2D & 3D)", 
                    "Geometry (Triangles, Circles, Polygons, Coordinate)",
                    "Trigonometry / Binomial Theorem",
                    "Permutation, Combination & Probability", 
                    "Data Interpretation (Tables, Bar, Line, Pie Charts)",
                    "Algebra (Advanced Topics)"
                ]
            },
            'alr': {
                name: 'Analytical & Logical Reasoning (A-LR)',
                topics: [
                    "Series & Analogy (Number, Alphabet, Visual)", 
                    "Coding-Decoding & Meaningful Word/Matrix", 
                    "Blood Relations / Family Tree", 
                    "Direction Sense",
                    "Clocks & Calendars",
                    "Syllogisms (Venn Logic)", 
                    "Seating & Linear Arrangements (Puzzles)", 
                    "Input-Output Logic",
                    "Critical Reasoning (Assumption, Inference)",
                    "Verbal & Miscellaneous Reasoning"
                ]
            },
            'ge': {
                name: 'General English (GE)',
                topics: [
                    "Grammar: Parts of Speech & Subject-Verb Agreement", 
                    "Vocabulary: Synonyms/Antonyms", 
                    "Reading Comprehension: Short Passages & Strategy", 
                    "Para Jumbles (Easy to Moderate)", 
                    "Sentence Correction / Error Spotting", 
                    "Idioms & Phrases", 
                    "One-word Substitution", 
                    "Cloze Test", 
                    "RC Practice (Medium/Hard)", 
                    "Mixed Grammar (Tense + Preposition)"
                ]
            },
            'mocks': {
                name: 'Mock Tests & Analysis (20 Mocks)',
                topics: Array.from({ length: 20 }, (_, i) => `Full Mock Test #${i + 1} & Analysis`)
            }
        };

        let currentTab = 'qa';
        const STORAGE_KEY = 'snap_study_progress_v2'; // Updated key to reflect new structure

        // --- Core Logic Functions ---

        /**
         * Loads the completion status from localStorage.
         * @returns {Object} An object mapping subject key to an array of booleans (topic status).
         */
        function loadProgress() {
            const stored = localStorage.getItem(STORAGE_KEY);
            let progress = stored ? JSON.parse(stored) : {};
            
            // Initialize or clean up the progress object based on current SUBJECTS structure
            for (const key in SUBJECTS) {
                const totalTopics = SUBJECTS[key].topics.length;
                if (!progress[key] || progress[key].length !== totalTopics) {
                    // Reinitialize if structure changed or missing
                    progress[key] = new Array(totalTopics).fill(false);
                }
            }
            return progress;
        }

        /**
         * Saves the current progress object to localStorage.
         * @param {Object} progress - The progress object to save.
         */
        function saveProgress(progress) {
            localStorage.setItem(STORAGE_KEY, JSON.stringify(progress));
        }

        /**
         * Calculates the completion percentage for all subjects.
         * @param {Object} progress - The progress object.
         * @returns {Object} An object mapping subject key to its completion percentage (0-100).
         */
        function calculateProgress(progress) {
            const results = {};
            for (const key in SUBJECTS) {
                const total = SUBJECTS[key].topics.length;
                const completed = progress[key].filter(status => status).length;
                results[key] = total === 0 ? 0 : Math.round((completed / total) * 100);
            }
            return results;
        }

        // --- Rendering Functions ---

        function renderProgressRing(subjectKey, percentage) {
            const radius = 35;
            const circumference = 2 * Math.PI * radius;
            const offset = circumference - (percentage / 100) * circumference;

            return `
                <div class="progress-ring-container mx-auto">
                    <svg class="progress-ring" width="80" height="80">
                        <circle
                            stroke="#e5e7eb"
                            stroke-width="5"
                            fill="transparent"
                            r="${radius}"
                            cx="40"
                            cy="40"
                        />
                        <circle
                            class="progress-ring__circle"
                            stroke="#10b981"
                            stroke-width="5"
                            fill="transparent"
                            r="${radius}"
                            cx="40"
                            cy="40"
                            stroke-dasharray="${circumference} ${circumference}"
                            style="stroke-dashoffset: ${offset}"
                        />
                    </svg>
                    <div class="absolute inset-0 flex items-center justify-center text-lg font-bold text-gray-800">
                        ${percentage}%
                    </div>
                </div>
            `;
        }

        function renderDashboard() {
            const progress = loadProgress();
            const percentages = calculateProgress(progress);
            const dashboard = document.getElementById('dashboard');
            
            // Adjust grid for 4 subjects
            let html = '<div class="grid grid-cols-2 sm:grid-cols-4 gap-6">'; 

            for (const key in SUBJECTS) {
                const percentage = percentages[key];
                html += `
                    <div class="bg-white p-4 sm:p-6 rounded-xl shadow-lg border-t-4 ${percentage === 100 ? 'border-green-500' : 'border-blue-500'}">
                        <h3 class="text-lg font-semibold mb-4 text-center text-gray-700">${SUBJECTS[key].name}</h3>
                        ${renderProgressRing(key, percentage)}
                        <p class="text-xs text-center mt-3 text-gray-500">
                            ${progress[key].filter(status => status).length} of ${SUBJECTS[key].topics.length} completed
                        </p>
                    </div>
                `;
            }

            html += '</div>';
            dashboard.innerHTML = html;
        }

        function renderTabs() {
            const tabContainer = document.getElementById('tab-container');
            const contentContainer = document.getElementById('tab-content');
            const progress = loadProgress();

            let tabButtonsHtml = '';
            let tabContentHtml = '';

            for (const key in SUBJECTS) {
                const isActive = key === currentTab;
                const activeClass = isActive ? 'bg-blue-600 text-white shadow-md' : 'text-gray-700 hover:bg-gray-200';

                tabButtonsHtml += `
                    <button id="tab-${key}" onclick="switchTab('${key}')"
                        class="px-4 py-2 sm:px-6 sm:py-2 rounded-t-lg font-medium transition-colors ${activeClass} focus:outline-none text-sm sm:text-base">
                        ${SUBJECTS[key].name}
                    </button>
                `;
                
                tabContentHtml += `
                    <div id="content-${key}" class="p-4 sm:p-6 bg-white rounded-b-xl ${isActive ? '' : 'hidden'}">
                        ${renderTopicList(key, progress[key])}
                    </div>
                `;
            }

            tabContainer.innerHTML = tabButtonsHtml;
            contentContainer.innerHTML = tabContentHtml;
        }

        function renderTopicList(subjectKey, topicStatus) {
            const topics = SUBJECTS[subjectKey].topics;
            let html = '<ul class="space-y-3">';

            topics.forEach((topic, index) => {
                const isChecked = topicStatus[index];
                const bgClass = isChecked ? 'bg-green-50 border-green-300' : 'bg-white border-gray-200 hover:bg-gray-50';
                const textClass = isChecked ? 'line-through text-gray-500' : 'text-gray-800 font-medium';
                
                // Use a different style for mock test tracking to make it stand out
                const itemBg = subjectKey === 'mocks' ? 'bg-yellow-50 border-yellow-300' : bgClass;
                const itemText = subjectKey === 'mocks' && !isChecked ? 'font-bold text-gray-900' : textClass;


                html += `
                    <li class="flex items-center p-3 rounded-lg border shadow-sm ${itemBg}">
                        <label class="flex items-center w-full cursor-pointer">
                            <input type="checkbox" 
                                class="form-checkbox h-5 w-5 text-blue-600 rounded focus:ring-blue-500 border-gray-300" 
                                data-subject="${subjectKey}" 
                                data-index="${index}" 
                                ${isChecked ? 'checked' : ''} 
                                onchange="handleCheckboxChange(this)">
                            <span class="ml-4 text-sm sm:text-base ${itemText}">${topic}</span>
                        </label>
                    </li>
                `;
            });
            
            html += '</ul>';
            return html;
        }

        // --- Event Handlers ---

        function switchTab(key) {
            currentTab = key;
            renderTabs();
        }

        function handleCheckboxChange(checkbox) {
            const subjectKey = checkbox.getAttribute('data-subject');
            const index = parseInt(checkbox.getAttribute('data-index'), 10);
            const isChecked = checkbox.checked;

            const progress = loadProgress();
            
            // Update the specific topic status
            if (progress[subjectKey]) {
                progress[subjectKey][index] = isChecked;
                saveProgress(progress);
                
                // Re-render the affected parts
                renderDashboard();
                renderTabs(); 
            }
        }

        // --- Initialization ---

        window.onload = function() {
            renderDashboard();
            renderTabs();

            // Setup a simple reset button for testing purposes
            document.getElementById('reset-btn').addEventListener('click', () => {
                // Use a custom modal or message box instead of confirm()
                const resetConfirmed = window.prompt("Type 'RESET' to confirm deleting all your progress:");
                if (resetConfirmed && resetConfirmed.toUpperCase() === 'RESET') {
                    localStorage.removeItem(STORAGE_KEY);
                    // Force reload to clear all state correctly
                    window.location.reload(); 
                } else if (resetConfirmed !== null) {
                    console.log("Reset cancelled or incorrect confirmation.");
                }
            });
        };
    </script>

    <div class="max-w-6xl mx-auto">
        <header class="text-center mb-10">
            <h1 class="text-4xl font-extrabold text-gray-900 mb-2">SNAP High-Intensity Progress Tracker</h1>
            <p class="text-lg text-gray-600">Comprehensive study checklist and 20-mock tracker.</p>
        </header>

        <!-- Progress Dashboard (Circular Charts) -->
        <section class="mb-10 p-4 sm:p-6 bg-white rounded-xl shadow-2xl">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl sm:text-2xl font-bold text-gray-800">Overall Progress</h2>
                <button id="reset-btn" class="text-xs sm:text-sm text-red-600 hover:text-red-800 font-medium px-2 py-1 sm:px-3 sm:py-1 border border-red-300 rounded-full transition-colors">
                    Reset All Progress
                </button>
            </div>
            <div id="dashboard">
                <!-- Progress rings will be rendered here -->
            </div>
        </section>

        <!-- Topic Checklists -->
        <section class="shadow-2xl rounded-xl">
            <div class="flex flex-wrap border-b border-gray-200 bg-gray-100 rounded-t-xl" id="tab-container">
                <!-- Tab buttons will be rendered here -->
            </div>
            <div id="tab-content">
                <!-- Tab content with topic lists will be rendered here -->
            </div>
        </section>
    </div>
</body>
</html>
