<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Complete Personal Planner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f3f4f6; color: #1f2937; }
        .hidden { display: none; }
        .completed { color: #6b7280; }
        .task-title-completed { text-decoration: line-through; }
        .task-item { background-color: white; border-radius: 0.75rem; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1); transition: transform 0.2s ease-out, box-shadow 0.2s ease-out; }
        .task-item:hover { transform: translateY(-4px); box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1); }
        .task-item.completed { background-color: #f9fafb; opacity: 0.8; }
        .priority-flag { width: 10px; height: 10px; border-radius: 50%; display: inline-block; flex-shrink: 0; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(-10px); } to { opacity: 1; transform: translateY(0); } }
        .animate-fade-in { animation: fadeIn 0.4s ease-out; }
        .loader { border: 4px solid #e5e7eb; border-top: 4px solid #4f46e5; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        .modal { animation: fadeIn 0.2s ease-out; }
        #calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 4px; }
        .calendar-day { 
            background-color: white; 
            border-radius: 0.5rem; 
            padding: 0.5rem; 
            min-height: 110px; 
            font-size: 0.875rem; 
            color: #4b5563; 
            transition: background-color 0.2s; 
            cursor: pointer; 
            display: flex; 
            flex-direction: column; 
            position: relative; 
            box-sizing: border-box; /* Ensure padding/border are included in the element's total width and height */
        }
        .calendar-day:not(.other-month):hover { background-color: #eff6ff; }
        .calendar-day.other-month { background-color: #f9fafb; color: #d1d5db; cursor: default; }
        .calendar-day.is-today .calendar-day-header { color: white; background-color: #4f46e5; border-radius: 9999px; width: 28px; height: 28px; display: flex; align-items: center; justify-content: center; }
        .calendar-day-header { font-weight: 600; text-align: center; margin-bottom: 0.25rem; }
        .details-container { max-height: 0; overflow: hidden; transition: max-height 0.3s ease-out, padding 0.3s ease-out, margin 0.3s ease-out; }
        .details-container.open { max-height: 40vh; overflow-y: auto; padding-top: 1rem; margin-top: 1rem; }
        .category-filter-btn { transition: background-color 0.2s, color 0.2s; }
        .calendar-tasks-container { flex-grow: 1; overflow: hidden; }
        #calendar-tooltip { position: absolute; z-index: 50; width: max-content; max-width: 250px; background-color: #1f2937; color: white; padding: 0.75rem; border-radius: 0.5rem; box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1); pointer-events: none; opacity: 0; transition: opacity 0.2s; }
        .completion-timestamp { text-decoration: none; }
        .task-dot { width: 8px; height: 8px; border-radius: 50%; margin: 2px; }
        #kanban-board { display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 1rem; }
        .kanban-column { background-color: #f3f4f6; border-radius: 0.75rem; padding: 1rem; }
        .kanban-column-title { font-weight: 600; color: #4b5563; margin-bottom: 1rem; display: flex; align-items: center; gap: 0.5rem; }
        .kanban-tasks { min-height: 100px; space-y: 0.75rem; }
        .kanban-card { background-color: white; border-radius: 0.5rem; padding: 0.75rem 1rem; box-shadow: 0 1px 3px rgba(0,0,0,0.1); cursor: grab; }
        .kanban-card.dragging { opacity: 0.5; }
        .kanban-column.drag-over { background-color: #e0e7ff; }
        @keyframes flash-red { 50% { box-shadow: 0 0 0 2px rgba(239, 68, 68, 0.4); } }
        .flash-error { animation: flash-red 0.7s ease-out; }
        
        /* PULSING FIX: Use box-shadow that doesn't change dimensions */
        @keyframes pulse-red { 0% { box-shadow: 0 0 0 0 rgba(239, 68, 68, 0.7); } 70% { box-shadow: 0 0 0 8px rgba(239, 68, 68, 0); } 100% { box-shadow: 0 0 0 0 rgba(239, 68, 68, 0); } }
        @keyframes pulse-orange { 0% { box-shadow: 0 0 0 0 rgba(245, 158, 11, 0.7); } 70% { box-shadow: 0 0 0 8px rgba(245, 158, 11, 0); } 100% { box-shadow: 0 0 0 0 rgba(245, 158, 11, 0); } }

        .pulse-overdue { animation: pulse-red 2s infinite; }
        .pulse-due-soon { animation: pulse-orange 2s infinite; }

        .skeleton { background-color: #e5e7eb; border-radius: 0.5rem; }
        .skeleton-title { width: 60%; height: 20px; margin-bottom: 0.5rem; }
        .skeleton-meta { width: 40%; height: 16px; }
        @keyframes shimmer { 100% { transform: translateX(100%); } }
        .skeleton { position: relative; overflow: hidden; }
        .skeleton::after { content: ''; position: absolute; top: 0; right: 0; bottom: 0; left: 0; transform: translateX(-100%); background-image: linear-gradient(90deg, rgba(255,255,255,0) 0, rgba(255,255,255,0.3) 50%, rgba(255,255,255,0) 100%); animation: shimmer 2s infinite; }
        .task-notes-textarea { width: 100%; border: 1px solid #e5e7eb; border-radius: 0.5rem; padding: 0.5rem 0.75rem; font-size: 0.875rem; resize: none; transition: background-color 0.2s, border-color 0.2s; }
        .task-notes-textarea:focus { outline: none; border-color: #4f46e5; background-color: #fafafa; }
        #app-indicator.success { background-color: #10B981; } /* Green */
        #app-indicator.info { background-color: #3B82F6; } /* Blue */
        #app-indicator.warning { background-color: #EF4444; } /* Red */

        /* Added for the settings toggle styling */
        .toggle-checkbox:checked { right: 0; border-color: #4f46e5; }
        .toggle-checkbox:checked + .toggle-label { background-color: #4f46e5; }
        .toggle-label { cursor: pointer; text-indent: -9999px; width: 40px; height: 24px; background: #9ca3af; display: block; border-radius: 100px; position: relative; }
        .toggle-checkbox { opacity: 0; width: 0; height: 0; }
        .toggle-checkbox:checked ~ .toggle-label:before { transform: translateX(16px); }
        .toggle-label:before { content: ''; position: absolute; top: 2px; left: 2px; width: 20px; height: 20px; background: #fff; border-radius: 90px; transition: 0.3s; box-shadow: 0 0 1px 0 rgba(10, 10, 10, 0.29); }
        
        /* Styles for Calendar Day Details (now in a modal) */
        .calendar-task-item { background-color: white; border-radius: 0.5rem; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
        .calendar-task-item.completed { opacity: 0.7; }
        .calendar-task-item:not(:last-child) { margin-bottom: 0.75rem; }
    </style>
</head>
<body class="antialiased">

    <div id="loader" class="loader"></div>
    <div id="calendar-tooltip" class="hidden"></div>
    <div id="app-indicator" class="hidden fixed bottom-5 left-1/2 -translate-x-1/2 bg-gray-800 text-white text-sm py-2 px-4 rounded-full shadow-lg flex items-center gap-2 z-50">
        <svg id="indicator-spinner" class="animate-spin h-4 w-4 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
        <span id="indicator-text">Saving...</span>
    </div>

    <div id="add-task-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal"><div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl max-w-lg w-full"><h3 class="text-2xl font-bold mb-6 text-gray-800">Add New Task</h3><form id="add-task-form" class="space-y-4"><div class="relative"><input type="text" id="add-task-text" placeholder="What do you need to do?" class="w-full bg-gray-100 rounded-lg pl-4 pr-12 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required><button type="button" id="generate-subtasks-btn" title="✨ Generate Subtasks with AI" class="absolute right-2 top-1/2 -translate-y-1/2 p-2 rounded-full text-gray-500 hover:bg-indigo-100 hover:text-indigo-600 transition-colors"><svg class="w-5 h-5" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 3v4M3 5h4M6 17v4m-2-2h4m1-9l2.293-2.293a1 1 0 011.414 0l.707.707a1 1 0 010 1.414L12.707 10l-1.414 1.414a1 1 0 01-1.414 0L7.586 9.121a1 1 0 010-1.414L8.293 7a1 1 0 011.414 0L12 9.293l2.293-2.293a1 1 0 011.414 0l.707.707a1 1 0 010 1.414L13.414 12l1.414 1.414a1 1 0 010 1.414l-.707.707a1 1 0 01-1.414 0L12 13.414l-2.293 2.293a1 1 0 01-1.414 0l-.707-.707a1 1 0 010-1.414L10.586 12 9.172 10.586a1 1 0 010-1.414l.707-.707a1 1 0 011.414 0L12 8.586z" /></svg></button></div><div class="grid grid-cols-1 sm:grid-cols-2 gap-4"><input type="date" id="add-task-date" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required><input type="time" id="add-task-time" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"></div><div class="grid grid-cols-1 sm:grid-cols-2 gap-4"><select id="add-task-priority" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"><option value="low">Low Priority</option><option value="medium" selected>Medium Priority</option><option value="high">High Priority</option></select><select id="add-task-category" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"><option value="Personal">Personal</option><option value="Work">Work</option><option value="School">School</option><option value="Other">Other</option></select></div><div><label for="add-task-tags" class="block mb-2 text-sm font-medium text-gray-600">Tags (comma separated)</label><input type="text" id="add-task-tags" placeholder="#urgent, #project-x" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"></div><div class="grid grid-cols-1 sm:grid-cols-2 gap-4"><select id="add-task-repeat" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"><option value="none">Does not repeat</option><option value="daily">Daily</option><option value="weekly">Weekly</option><option value="monthly">Monthly</option></select><div><label for="add-task-reminder" class="block mb-1 text-sm font-medium text-gray-600">Reminder</label><select id="add-task-reminder" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"><option value="none">No reminder</option><option value="5">5 minutes before</option><option value="15">15 minutes before</option><option value="60">1 hour before</option></select><div id="add-email-toggle-container" class="hidden mt-2 pl-2"><label class="flex items-center space-x-2"><input type="checkbox" id="add-task-email-toggle" class="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500"><span class="text-sm text-gray-600">Also send an email reminder?</span></label></div></div></div><div id="generated-subtasks-container" class="hidden space-y-2 pt-2 max-h-32 overflow-y-auto"><h4 class="text-sm font-bold text-gray-600">✨ AI Generated Subtasks:</h4><ul id="generated-subtasks-list" class="space-y-1 text-sm text-gray-700"></ul></div><div class="flex justify-end gap-4 pt-4"><button type="button" id="cancel-add-task-btn" class="bg-gray-200 hover:bg-gray-300 font-bold py-2 px-6 rounded-lg">Cancel</button><button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-6 rounded-lg">Add Task</button></div></form></div></div>
    <div id="edit-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal"><div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl max-w-lg w-full"><h3 class="text-2xl font-bold mb-6 text-gray-800">Edit Task</h3><form id="edit-task-form" class="space-y-4"><input type="text" id="edit-task-text" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required><div class="grid grid-cols-1 sm:grid-cols-2 gap-4"><select id="edit-task-priority" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"><option value="low">Low</option> <option value="medium">Medium</option> <option value="high">High</option></select><select id="edit-task-category" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"><option value="Personal">Personal</option> <option value="Work">Work</option> <option value="School">School</option> <option value="Other">Other</option></select></div><div class="grid grid-cols-1 sm:grid-cols-2 gap-4"><input type="date" id="edit-task-date" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" required><input type="time" id="edit-task-time" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"></div><div><label for="edit-task-tags" class="block mb-2 text-sm font-medium text-gray-600">Tags (comma separated)</label><input type="text" id="edit-task-tags" placeholder="#urgent, #project-x" class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500"></div><textarea id="edit-task-notes" placeholder="Add notes..." class="w-full bg-gray-100 rounded-lg px-4 py-3 border focus:outline-none focus:ring-2 focus:ring-indigo-500" rows="3"></textarea><div class="flex justify-between gap-4 pt-4"><button type="button" id="edit-history-btn" class="hidden bg-gray-100 hover:bg-gray-200 text-sm font-bold py-2 px-4 rounded-lg">Edit History</button><div class="flex gap-4"><button type="button" id="cancel-edit-btn" class="bg-gray-200 hover:bg-gray-300 font-bold py-2 px-6 rounded-lg">Cancel</button><button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-6 rounded-lg">Save Changes</button></div></div></form></div></div>
    <div id="confirm-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal"><div class="bg-white p-8 rounded-2xl shadow-xl max-w-sm w-full text-center"><h3 class="text-xl font-bold mb-4">Confirm Deletion</h3><p class="text-gray-600 mb-6">This action cannot be undone.</p><div class="flex justify-center gap-4"><button id="cancel-delete-btn" class="bg-gray-200 hover:bg-gray-300 font-bold py-2 px-6 rounded-lg">Cancel</button><button id="confirm-delete-btn" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-6 rounded-lg">Delete</button></div></div></div>
    <div id="calendar-day-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal"><div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl max-w-md w-full"><h3 id="calendar-modal-title" class="text-xl font-bold mb-4 text-gray-800"></h3><div id="calendar-modal-tasks" class="space-y-2 max-h-80 overflow-y-auto"></div><div class="text-right mt-6"><button id="calendar-modal-close-btn" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-6 rounded-lg">Close</button></div></div></div>
    <div id="history-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal">
        <div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl max-w-lg w-full">
            <h3 class="text-2xl font-bold mb-6 text-gray-800">Task Edit History</h3>
            <div id="history-modal-content" class="space-y-4 max-h-96 overflow-y-auto"></div>
            <div class="text-right mt-6">
                <button id="history-modal-close-btn" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-6 rounded-lg">Close</button>
            </div>
        </div>
    </div>
    <div id="stats-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal"><div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl max-w-md w-full relative"><button id="stats-modal-close-btn" class="absolute top-4 right-4 text-gray-400 hover:text-gray-600"><svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg></button><h3 class="text-2xl font-bold mb-6 text-gray-800">Productivity Stats</h3><div class="space-y-4 max-h-[70vh] overflow-y-auto"><div class="flex justify-between items-center bg-gray-50 p-3 rounded-lg"><span class="font-medium text-gray-600">Total Tasks Completed:</span><span id="stats-total-completed" class="font-bold text-lg text-indigo-600">0</span></div><div class="flex justify-between items-center bg-gray-50 p-3 rounded-lg"><span class="font-medium text-gray-600">Overall Completion Rate:</span><span id="stats-completion-rate" class="font-bold text-lg text-indigo-600">0%</span></div><div class="flex justify-between items-center bg-gray-50 p-3 rounded-lg"><span class="font-medium text-gray-600">Tasks Finished This Week:</span><span id="stats-completed-this-week" class="font-bold text-lg text-indigo-600">0</span></div><div><h4 class="font-medium text-gray-600 mb-2">Completed Task Log:</h4><div id="stats-category-breakdown" class="space-y-3"></div></div></div></div></div>
    <div id="settings-modal" class="hidden fixed inset-0 bg-gray-900 bg-opacity-60 flex items-center justify-center z-50 p-4 modal"><div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl max-w-md w-full relative"><button id="settings-modal-close-btn" class="absolute top-4 right-4 text-gray-400 hover:text-gray-600"><svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg></button><h3 class="text-2xl font-bold mb-6 text-gray-800">Settings</h3><div class="space-y-4"><h4 class="font-semibold text-gray-700">Task View</h4><div class="flex items-center justify-between"><label for="toggle-show-completed" class="text-sm font-medium text-gray-600">Show Completed Tasks</label><div class="relative inline-block w-10 mr-2 align-middle select-none transition duration-200 ease-in"><input type="checkbox" id="toggle-show-completed" class="toggle-checkbox absolute block w-6 h-6 rounded-full bg-white border-4 appearance-none cursor-pointer"/><label for="toggle-show-completed" class="toggle-label block overflow-hidden h-6 rounded-full bg-gray-300 cursor-pointer"></label></div></div><div class="mt-6 pt-4 border-t border-gray-200"><h4 class="font-semibold text-red-600">Danger Zone</h4><p class="text-xs text-gray-500 mb-2">This action cannot be undone.</p><button id="delete-all-data-btn" class="w-full bg-red-100 text-red-700 hover:bg-red-200 font-bold py-2 px-4 rounded-lg">Delete All Task Data</button></div></div></div></div>

    <div id="auth-container" class="hidden container mx-auto max-w-md p-4 sm:p-8 mt-10 animate-fade-in"><div id="logout-success-message" class="hidden bg-green-100 border-green-400 text-green-700 px-4 py-3 rounded-lg mb-6 text-center"></div><div class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl"><h2 class="text-3xl font-bold text-center text-indigo-600 mb-8">Login</h2><form id="login-form"><input type="email" id="login-email" autocomplete="username" class="w-full bg-gray-100 rounded-lg p-3 border mb-4 focus:outline-none focus:ring-2 focus:ring-indigo-500" required placeholder="Email"><input type="password" id="login-password" autocomplete="current-password" class="w-full bg-gray-100 rounded-lg p-3 border mb-6 focus:outline-none focus:ring-2 focus:ring-indigo-500" required placeholder="Password"><button type="submit" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-6 rounded-lg shadow-md">Sign In</button></form><p id="auth-error" class="text-red-500 text-center mt-4"></p><p class="text-center text-sm text-gray-500 mt-6">To register an account, please contact the administrator.</p></div></div>

    <div id="planner-container" class="hidden container mx-auto p-4 md:p-8 max-w-7xl">
        <header class="flex flex-wrap justify-between items-center mb-8 gap-4">
            <div><h1 class="text-3xl md:text-4xl font-bold text-indigo-600">My Planner</h1><p id="current-date" class="text-gray-500 text-sm md:text-base"></p></div>
            <div class="flex items-center gap-2 sm:gap-3 relative">
                <div id="user-profile-area" class="flex items-center gap-2">
                    <svg class="w-10 h-10 text-gray-500" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z" clip-rule="evenodd"></path></svg>
                    <div class="hidden sm:block">
                        <p id="user-profile-name" class="font-semibold text-sm text-gray-800"></p>
                        <p class="text-xs text-gray-500">Welcome back!</p>
                    </div>
                </div>
                <button id="header-today-btn" title="Today's Tasks" class="relative bg-white hover:bg-gray-100 text-gray-600 p-2 rounded-full transition-colors duration-200 shadow-sm border"><svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"></path></svg></button>
                <button id="header-stats-btn" title="Productivity Stats" class="bg-white hover:bg-gray-100 text-gray-600 p-2 rounded-full transition-colors duration-200 shadow-sm border"><svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 3.055A9.001 9.001 0 1020.945 13H11V3.055z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20.488 9H15V3.512A9.025 9.025 0 0120.488 9z"></path></svg></button>
                <button id="header-settings-btn" title="Settings" class="bg-white hover:bg-gray-100 text-gray-600 p-2 rounded-full transition-colors duration-200 shadow-sm border"><svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path></svg></button>
                <button id="header-add-task-btn" title="Add New Task" class="bg-indigo-600 hover:bg-indigo-700 text-white p-2 rounded-full transition-colors duration-200 shadow-md"><svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg></button>
                <button id="header-logout-btn" title="Log Out" class="bg-white hover:bg-gray-100 text-gray-600 p-2 rounded-full transition-colors duration-200 shadow-sm border"><svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1"></path></svg></button>

                <div id="logout-panel" class="hidden absolute top-14 right-0 w-64 bg-white rounded-lg shadow-xl p-4 text-sm z-50 animate-fade-in border border-gray-100">
                    <p class="font-semibold text-gray-800 mb-2">Are you sure?</p>
                    <p class="text-gray-500 mb-4">You are currently signed in as <span id="logout-user-name" class="font-medium text-indigo-600"></span>.</p>
                    <div class="flex justify-end gap-2">
                        <button id="cancel-logout-btn" class="bg-gray-200 hover:bg-gray-300 py-1 px-3 rounded-lg">Cancel</button>
                        <button id="confirm-logout-btn" class="bg-red-500 hover:bg-red-600 text-white py-1 px-3 rounded-lg">Yes, Log Out</button>
                    </div>
                </div>
            </div>

            <div id="today-panel" class="hidden absolute top-20 right-8 md:right-32 mt-2 w-80 bg-white rounded-lg shadow-xl z-50 animate-fade-in">
                <div class="p-4 border-b flex justify-between items-center font-semibold">Today's Agenda <button id="today-panel-close-btn" class="text-gray-400 hover:text-gray-600">&times;</button></div>
                <div id="today-panel-content" class="max-h-96 overflow-y-auto"></div>
            </div>
        </header>

        <div id="login-success-message" class="hidden bg-green-100 border-green-400 text-green-700 px-4 py-3 rounded-lg mb-6 text-center"></div>
        <div id="task-success-message" class="hidden bg-green-100 border-green-400 text-green-700 px-4 py-3 rounded-lg mb-6 text-center"></div>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
            <div class="bg-white p-6 rounded-2xl shadow-xl"><h3 class="text-lg font-semibold text-gray-500 text-center">Upcoming Task</h3><div id="dashboard-next-task" class="mt-2 text-left"></div></div>
            <div class="bg-white p-6 rounded-2xl shadow-xl"><h3 class="text-lg font-semibold text-gray-500 text-center mb-2">Pending Tasks</h3><div id="dashboard-pending-breakdown" class="space-y-2 text-sm"></div></div>
            <div class="bg-white p-6 rounded-2xl shadow-xl"><h3 class="text-lg font-semibold text-gray-500 text-center">Overdue (<span id="dashboard-overdue-count">0</span>)</h3><div id="dashboard-overdue" class="mt-2 text-left"></div></div>
        </div>

        <div class="bg-white p-4 rounded-2xl shadow-xl mb-8 space-y-4">
            <div class="flex items-center bg-gray-100 rounded-lg p-1">
                <button id="view-list-btn" class="flex-1 px-4 py-2 text-sm font-semibold rounded-md bg-indigo-600 text-white">List</button>
                <button id="view-board-btn" class="flex-1 px-4 py-2 text-sm font-semibold rounded-md text-gray-600 hover:bg-gray-200">Board</button>
                <button id="view-calendar-btn" class="flex-1 px-4 py-2 text-sm font-semibold rounded-md text-gray-600 hover:bg-gray-200">Calendar</button>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div class="relative"><input type="text" id="searchInput" placeholder="Search tasks or #tags..." class="w-full bg-gray-100 rounded-lg px-4 py-2 border border-transparent focus:outline-none focus:ring-2 focus:ring-indigo-500"><span id="search-results-counter" class="hidden absolute right-3 top-1/2 -translate-y-1/2 text-sm text-gray-500"></span></div>
                <div id="category-filters" class="flex flex-wrap items-center gap-2"><span class="text-sm font-semibold text-gray-600 mr-2">Categories:</span><button data-category="All" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full flex items-center gap-2">All</button><button data-category="Personal" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full flex items-center gap-2">Personal</button><button data-category="Work" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full flex items-center gap-2">Work</button><button data-category="School" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full flex items-center gap-2">School</button><button data-category="Other" class="category-filter-btn text-sm font-medium px-3 py-1 rounded-full flex items-center gap-2">Other</button></div>
            </div>
        </div>
        
        <div id="list-view-container"><div id="task-list-container" class="space-y-4"></div></div>
        <div id="kanban-view-container" class="hidden"><div id="kanban-board"></div></div>

        <div id="calendar-view-container" class="hidden">
            <div class="bg-white p-6 rounded-2xl shadow-xl">
                <div class="flex justify-between items-center mb-4">
                    <button id="prev-month-btn" class="p-2 rounded-full hover:bg-gray-100">&lt;</button>
                    <h2 id="calendar-month-year" class="text-xl font-bold"></h2>
                    <button id="next-month-btn" class="p-2 rounded-full hover:bg-gray-100">&gt;</button>
                </div>
                <div class="grid grid-cols-7 gap-4 text-center font-semibold text-gray-500 text-sm mb-2"><div>Sun</div><div>Mon</div><div>Tue</div><div>Wed</div><div>Thu</div><div>Fri</div><div>Sat</div></div>
                <div id="calendar-grid"></div>
            </div>
        </div>
        
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-app.js";
        import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-auth.js";
        import { getFirestore, collection, addDoc, query, onSnapshot, doc, updateDoc, deleteDoc, serverTimestamp, orderBy } from "https://www.gstatic.com/firebasejs/9.15.0/firebase-firestore.js";

        const firebaseConfig = { apiKey: "AIzaSyA_LRcHCkClvlHeqDPTSKfGa5gY2uiuZ5E", authDomain: "mi-planificador-privado.firebaseapp.com", projectId: "mi-planificador-privado", storageBucket: "mi-planificador-privado.firebasestorage.app", messagingSenderId: "792700686473", appId: "1:792700686473:web:8d1f41076f12fa0103f658" };
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        let allTasks = [], isInitialLoad = true, unsubscribeTasks, taskToDeleteId = null, taskToEditId = null, timerInterval = null, currentView = 'list', currentSearchTerm = '', currentCategoryFilter = 'All', currentCalendarDate = new Date();
        const categoryMap = { 'Personal': {color: 'bg-blue-100 text-blue-800', icon: `<svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path></svg>`}, 'Work': {color: 'bg-purple-100 text-purple-800', icon: `<svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 13.255A23.931 23.931 0 0112 15c-3.183 0-6.22-.62-9-1.745M16 6V4a2 2 0 00-2-2h-4a2 2 0 00-2 2v2m4 6h.01M5 20h14a2 2 0 002-2V8a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"></path></svg>`}, 'School': {color: 'bg-green-100 text-green-800', icon: `<svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.246 18 16.5 18c-1.746 0-3.332.477-4.5 1.253"></path></svg>`}, 'Other': {color: 'bg-gray-200 text-gray-800', icon: `<svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 7h.01M7 3h5c.512 0 1.024.195 1.414.586l7 7a2 2 0 010 2.828l-5 5a2 2 0 01-2.828 0l-7-7A2 2 0 013 8V5a2 2 0 012-2h2z"></path></svg>`} };
        const categoryActiveMap = { 'Personal': 'bg-blue-500 text-white', 'Work': 'bg-purple-500 text-white', 'School': 'bg-green-500 text-white', 'Other': 'bg-gray-500 text-white' };
        const categoryDotMap = { 'Personal': 'bg-blue-500', 'Work': 'bg-purple-500', 'School': 'bg-green-500', 'Other': 'bg-gray-400' };
        const priorityFlagMap = { high: 'bg-red-500', medium: 'bg-yellow-500', low: 'bg-green-500' };

        const DOMElements = {
            loader: document.getElementById('loader'), appIndicator: document.getElementById('app-indicator'), indicatorText: document.getElementById('indicator-text'), authContainer: document.getElementById('auth-container'), plannerContainer: document.getElementById('planner-container'), calendarTooltip: document.getElementById('calendar-tooltip'),
            userProfileName: document.getElementById('user-profile-name'), currentDate: document.getElementById('current-date'),
            authError: document.getElementById('auth-error'), loginForm: document.getElementById('login-form'),
            loginSuccessMessage: document.getElementById('login-success-message'), logoutSuccessMessage: document.getElementById('logout-success-message'), taskSuccessMessage: document.getElementById('task-success-message'),
            confirmModal: document.getElementById('confirm-modal'), cancelDeleteBtn: document.getElementById('cancel-delete-btn'), confirmDeleteBtn: document.getElementById('confirm-delete-btn'),
            addTaskModal: document.getElementById('add-task-modal'), addTaskForm: document.getElementById('add-task-form'),
            cancelAddTaskBtn: document.getElementById('cancel-add-task-btn'), addTaskText: document.getElementById('add-task-text'), addTaskDate: document.getElementById('add-task-date'),
            addTaskTime: document.getElementById('add-task-time'), addTaskCategory: document.getElementById('add-task-category'), addTaskPriority: document.getElementById('add-task-priority'), addTaskTags: document.getElementById('add-task-tags'), addTaskRepeat: document.getElementById('add-task-repeat'), addTaskReminder: document.getElementById('add-task-reminder'), addTaskEmailToggleContainer: document.getElementById('add-email-toggle-container'), addTaskEmailToggle: document.getElementById('add-task-email-toggle'),
            editModal: document.getElementById('edit-modal'), editTaskForm: document.getElementById('edit-task-form'), cancelEditBtn: document.getElementById('cancel-edit-btn'),
            editTaskText: document.getElementById('edit-task-text'), editTaskDate: document.getElementById('edit-task-date'), editTaskTime: document.getElementById('edit-task-time'),
            editTaskCategory: document.getElementById('edit-task-category'), editTaskPriority: document.getElementById('edit-task-priority'), editTaskNotes: document.getElementById('edit-task-notes'), editTaskTags: document.getElementById('edit-task-tags'),
            taskListContainer: document.getElementById('task-list-container'), viewListBtn: document.getElementById('view-list-btn'), viewBoardBtn: document.getElementById('view-board-btn'), viewCalendarBtn: document.getElementById('view-calendar-btn'),
            listViewContainer: document.getElementById('list-view-container'), kanbanViewContainer: document.getElementById('kanban-view-container'), kanbanBoard: document.getElementById('kanban-board'), calendarViewContainer: document.getElementById('calendar-view-container'),
            searchInput: document.getElementById('searchInput'), searchResultsCounter: document.getElementById('search-results-counter'), categoryFilters: document.getElementById('category-filters'),
            calendarGrid: document.getElementById('calendar-grid'), calendarMonthYear: document.getElementById('calendar-month-year'), prevMonthBtn: document.getElementById('prev-month-btn'), nextMonthBtn: document.getElementById('next-month-btn'),
            dashboardNextTask: document.getElementById('dashboard-next-task'), dashboardPendingBreakdown: document.getElementById('dashboard-pending-breakdown'), dashboardOverdue: document.getElementById('dashboard-overdue'), dashboardOverdueCount: document.getElementById('dashboard-overdue-count'),
            statsModal: document.getElementById('stats-modal'), statsModalCloseBtn: document.getElementById('stats-modal-close-btn'),
            statsTotalCompleted: document.getElementById('stats-total-completed'), statsCompletionRate: document.getElementById('stats-completion-rate'),
            statsCompletedThisWeek: document.getElementById('stats-completed-this-week'), statsCategoryBreakdown: document.getElementById('stats-category-breakdown'), deleteAllDataBtn: document.getElementById('delete-all-data-btn'),
            headerStatsBtn: document.getElementById('header-stats-btn'), headerAddTaskBtn: document.getElementById('header-add-task-btn'), headerLogoutBtn: document.getElementById('header-logout-btn'),
            headerTodayBtn: document.getElementById('header-today-btn'), todayPanel: document.getElementById('today-panel'), todayPanelContent: document.getElementById('today-panel-content'),
            todayPanelCloseBtn: document.getElementById('today-panel-close-btn'),
            headerSettingsBtn: document.getElementById('header-settings-btn'), 
            settingsModal: document.getElementById('settings-modal'),
            settingsModalCloseBtn: document.getElementById('settings-modal-close-btn'),
            toggleShowCompleted: document.getElementById('toggle-show-completed'),
            
            // Re-added Calendar Modal Elements
            calendarDayModal: document.getElementById('calendar-day-modal'),
            calendarModalTitle: document.getElementById('calendar-modal-title'),
            calendarModalTasks: document.getElementById('calendar-modal-tasks'),
            calendarModalCloseBtn: document.getElementById('calendar-modal-close-btn'),

            // New History Elements
            historyModal: document.getElementById('history-modal'),
            historyModalContent: document.getElementById('history-modal-content'),
            historyModalCloseBtn: document.getElementById('history-modal-close-btn'),
            editHistoryBtn: document.getElementById('edit-history-btn'),

            // Logout Panel Elements
            logoutPanel: document.getElementById('logout-panel'),
            logoutUserName: document.getElementById('logout-user-name'),
            confirmLogoutBtn: document.getElementById('confirm-logout-btn'),
            cancelLogoutBtn: document.getElementById('cancel-logout-btn'),
        };

        onAuthStateChanged(auth, user => {
            if (user) {
                DOMElements.authContainer.classList.add('hidden'); DOMElements.plannerContainer.classList.remove('hidden');
                if (sessionStorage.getItem('loginJustOccurred')) { showFlashMessage(DOMElements.loginSuccessMessage, "Successfully logged in!"); sessionStorage.removeItem('loginJustOccurred'); }
                const emailName = user.email.split('@')[0];
                DOMElements.userProfileName.textContent = emailName;
                DOMElements.logoutUserName.textContent = user.email; // For logout confirmation
                loadTasks(user.uid); 
                if (timerInterval) clearInterval(timerInterval);
                timerInterval = setInterval(updateDynamicElements, 15000);
            } else {
                DOMElements.plannerContainer.classList.add('hidden'); DOMElements.authContainer.classList.remove('hidden');
                if(unsubscribeTasks) unsubscribeTasks();
                if (timerInterval) clearInterval(timerInterval); clearUI();
            }
            DOMElements.loader.classList.add('hidden');
        });

        async function loadTasks(userId) { isInitialLoad = true; refreshDynamicContent(); const tasksCollection = collection(db, "users", userId, "tasks"); if (unsubscribeTasks) unsubscribeTasks(); unsubscribeTasks = onSnapshot(query(tasksCollection, orderBy("date", "desc")), snapshot => { 
            // Ensure 'history' property exists on all tasks for safety
            allTasks = snapshot.docs.map(doc => ({ 
                id: doc.id, 
                ...doc.data(), 
                subtasks: doc.data().subtasks || [], 
                notes: doc.data().notes || '',
                history: doc.data().history || [] // Initialize history array
            })); 
            if (isInitialLoad) { isInitialLoad = false; } refreshDynamicContent(); 
        }); }
        
        // Helper to find differences for history logging
        function getChanges(oldTask, newTask) {
            const changes = [];
            const fields = ['text', 'date', 'time', 'priority', 'category'];
            
            fields.forEach(field => {
                const oldValue = oldTask[field] || 'N/A';
                const newValue = newTask[field] || 'N/A';
                
                if (oldValue !== newValue) {
                    changes.push({ field, oldValue, newValue });
                }
            });
            return changes;
        }

        async function crudOperation(action, data) { 
            const user = auth.currentUser; 
            if (!user) return; 
            const { id, ...payload } = data; 
            
            // Handle Edit History Logging
            if (action === 'update') {
                const oldTask = allTasks.find(t => t.id === id);
                const changes = getChanges(oldTask, payload);

                if (changes.length > 0) {
                    const newHistoryEntry = {
                        timestamp: new Date().toISOString(),
                        changes: changes,
                        action: 'Edited'
                    };
                    
                    // Add new entry to existing history or create a new one
                    payload.history = [...(oldTask.history || []), newHistoryEntry];
                }
            }

            showIndicator("Saving...", "info", true); 
            
            try { 
                if (action === 'add') await addDoc(collection(db, "users", user.uid, "tasks"), payload); 
                else if (action === 'update') await updateDoc(doc(db, "users", user.uid, "tasks", id), payload); 
                else if (action === 'delete') await deleteDoc(doc(db, "users", user.uid, "tasks", id)); 
                
                const successMessage = action === 'add' ? 'Task Added' : action === 'update' ? 'Changes Saved' : 'Task Deleted';
                showIndicator(successMessage, "success", false, 3000); 

            } catch (error) { 
                console.error("Firestore Error:", error); 
                showIndicator("Error! Failed to save.", "warning", false, 4000);
            }
        }
        
        function refreshDynamicContent() { 
            const filteredTasks = getFilteredTasks(); 
            if(currentSearchTerm || currentCategoryFilter !== 'All') { 
                const count = filteredTasks.length; 
                DOMElements.searchResultsCounter.textContent = `${count} result${count !== 1 ? 's' : ''}`; 
                DOMElements.searchResultsCounter.classList.remove('hidden'); 
            } else { 
                DOMElements.searchResultsCounter.classList.add('hidden'); 
            } 
            if (currentView === 'list') renderListView(filteredTasks); 
            else if (currentView === 'board') renderKanbanView(filteredTasks); 
            else renderCalendarView(filteredTasks); 
            updateDashboard(filteredTasks); 
            updateDynamicElements(); 
        }
        
        function getFilteredTasks() { 
            const now = new Date();
            const showCompleted = DOMElements.toggleShowCompleted ? DOMElements.toggleShowCompleted.checked : true;

            return allTasks
                .filter(task => {
                    if (!showCompleted && task.completed && task.completedAt) {
                        const completedDate = new Date(task.completedAt);
                        completedDate.setHours(23, 59, 59, 999);
                        return now <= completedDate; 
                    }
                    return true;
                })
                .filter(task => (task.text.toLowerCase().includes(currentSearchTerm.toLowerCase()) || (task.tags && task.tags.some(tag => tag.toLowerCase().includes(currentSearchTerm.toLowerCase())))) && (currentCategoryFilter === 'All' || task.category === currentCategoryFilter)); 
        }

        function renderListView(tasks) {
            DOMElements.taskListContainer.innerHTML = '';
            if (isInitialLoad) { DOMElements.taskListContainer.innerHTML = `<div class="space-y-4"><div class="task-item p-4"><div class="skeleton skeleton-title"></div><div class="skeleton skeleton-meta"></div></div><div class="task-item p-4"><div class="skeleton skeleton-title"></div><div class="skeleton skeleton-meta"></div></div><div class="task-item p-4"><div class="skeleton skeleton-title"></div><div class="skeleton skeleton-meta"></div></div></div>`; return; }
            if (tasks.length === 0) { DOMElements.taskListContainer.innerHTML = `<div class="text-center text-gray-500 py-10"><svg class="mx-auto h-12 w-12 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" aria-hidden="true"><path vector-effect="non-scaling-stroke" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" /></svg><h3 class="mt-2 text-sm font-medium text-gray-900">All caught up!</h3><p class="mt-1 text-sm text-gray-500">You have no pending tasks.</p></div>`; return; }
            const groupedTasks = tasks.reduce((acc, task) => { const group = getRelativeDateGroup(task.date); if (!acc[group]) acc[group] = []; acc[group].push(task); return acc; }, {});
            const groupOrder = ["Overdue", "Today", "Tomorrow"];
            const sortedGroupKeys = Object.keys(groupedTasks).sort((a, b) => { const aIndex = groupOrder.indexOf(a); const bIndex = groupOrder.indexOf(b); if (aIndex !== -1 && bIndex !== -1) return aIndex - bIndex; if (aIndex !== -1) return -1; if (bIndex !== -1) return 1; return new Date(a.split(', ')[1]) - new Date(b.split(', ')[1]); });
            sortedGroupKeys.forEach(groupName => {
                const tasksInGroup = groupedTasks[groupName];
                tasksInGroup.sort((a, b) => (a.completed - b.completed) || (new Date(`${a.date}T${a.time || '00:00'}`) - new Date(`${b.date}T${b.time || '00:00'}`)));
                const completedCount = tasksInGroup.filter(t => t.completed).length;
                const totalCount = tasksInGroup.length;
                const groupSection = document.createElement('div'); groupSection.className = 'animate-fade-in';
                groupSection.innerHTML = `<div><h3 class="text-xl font-bold text-gray-700">${groupName}</h3><span class="font-semibold text-gray-400 text-xs mt-1 block">${completedCount} of ${totalCount} completed</span></div><ul class="space-y-4 mt-2">${tasksInGroup.map(createTaskElement).join('')}</ul>`;
                DOMElements.taskListContainer.appendChild(groupSection);
            });
            attachDynamicListeners(DOMElements.taskListContainer);
        }
        
        function updateDashboard(tasks) { if(isInitialLoad) return; const pendingTasks = tasks.filter(t => !t.completed); const now = new Date(); const todayString = now.toISOString().split('T')[0]; const todayPendingCount = pendingTasks.filter(t => t.date === todayString).length; DOMElements.currentDate.innerHTML = `${now.toLocaleDateString('en-US', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric'})} <span id="today-task-counter" class="text-gray-400 font-normal ml-2"></span>`; document.getElementById('today-task-counter').textContent = todayPendingCount > 0 ? `(${todayPendingCount} tasks left)`: '(No tasks for today)'; const upcomingTasks = pendingTasks.map(t => ({ ...t, dateTime: new Date(`${t.date}T${t.time || '23:59:59'}`) })).filter(t => t.dateTime >= now).sort((a, b) => a.dateTime - b.dateTime); if (upcomingTasks.length > 0) { const firstTask = upcomingTasks[0]; const firstDueTime = firstTask.dateTime.getTime(); const allNextTasks = upcomingTasks.filter(t => t.dateTime.getTime() === firstDueTime); DOMElements.dashboardNextTask.innerHTML = `<div class="text-left">${allNextTasks.map(task => `<p class="font-bold text-lg text-indigo-600 break-words">${task.text}</p>`).join('')}<p class="text-sm text-gray-500 mt-1">${getRelativeDateGroup(firstTask.date)} ${firstTask.time ? `at ${formatTo12Hour(firstTask.time)}` : ''}</p></div>`; } else { DOMElements.dashboardNextTask.innerHTML = `<p class="text-gray-400 text-center">No upcoming tasks!</p>`; } DOMElements.dashboardPendingBreakdown.innerHTML = `<div class="flex justify-between"><span>Today:</span> <span class="font-bold">${todayPendingCount}</span></div> <div class="flex justify-between"><span>This Week:</span> <span class="font-bold">${pendingTasks.filter(t => { const d = new Date(t.date); const endOfWeek = new Date(now); endOfWeek.setDate(now.getDate() + 6 - now.getDay()); return d > now && d <= endOfWeek; }).length}</span></div> <div class="flex justify-between"><span>This Month:</span> <span class="font-bold">${pendingTasks.filter(t => { const d = new Date(t.date); const endOfWeek = new Date(now); endOfWeek.setDate(now.getDate() + 6 - now.getDay()); const endOfMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0); return d > endOfWeek && d <= endOfMonth; }).length}</span></div>`; const overdueTasks = pendingTasks.map(t => ({ ...t, dateTime: new Date(`${t.date}T${t.time || '23:59:59'}`) })).filter(t => t.dateTime < now).sort((a, b) => a.dateTime - b.dateTime); DOMElements.dashboardOverdueCount.textContent = overdueTasks.length; if (overdueTasks.length > 0) { DOMElements.dashboardOverdue.innerHTML = `<ul class="space-y-1 text-left">${overdueTasks.slice(0, 3).map(task => `<li class="font-semibold text-red-500 truncate">${task.text}</li>`).join('')}</ul>`; } else { DOMElements.dashboardOverdue.innerHTML = `<p class="text-gray-400 text-center">No overdue tasks!</p>`; } }
        function createTaskElement(task) { 
            const taskDateTime = new Date(`${task.date}T${task.time || '23:59:59'}`); 
            let timeDiffHtml = !task.completed ? `<span class="relative-time" data-datetime="${taskDateTime.toISOString()}"></span>` : ''; 
            const completedAtHtml = task.completed && task.completedAt ? `<div class="completion-timestamp text-green-600 font-semibold mt-1 text-xs flex items-center gap-1 pl-8"><svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg><span>Completed: ${new Date(task.completedAt).toLocaleDateString('en-US', { month: 'short', day: 'numeric' })} at ${formatTo12Hour(new Date(task.completedAt).toTimeString())}</span></div>` : ''; 
            
            const subtasksHtml = (task.subtasks || []).map((sub, index) => `<div class="flex items-center justify-between gap-2 ml-4"><div class="flex items-start gap-2 flex-1"><input type="checkbox" id="subtask-${task.id}-${index}" data-subtask-index="${index}" class="subtask-checkbox h-4 w-4 mt-1 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500" ${sub.completed ? 'checked' : ''}><div class="flex-1"><label for="subtask-${task.id}-${index}" class="text-sm ${sub.completed ? 'completed task-title-completed' : ''}">${sub.text}</label>${sub.completed && sub.completedAt ? `<div class="text-xs text-gray-400 completion-timestamp">Completed: ${new Date(sub.completedAt).toLocaleDateString('en-US', {month:'short',day:'numeric'})} at ${formatTo12Hour(new Date(sub.completedAt).toTimeString())}</div>` : ''}</div></div><div class="flex-shrink-0"><button class="edit-subtask-btn p-1 rounded-full hover:bg-gray-200" data-task-id="${task.id}" data-subtask-index="${index}"><svg class="w-4 h-4 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"></path></svg></button><button class="delete-subtask-btn p-1 rounded-full hover:bg-gray-200" data-task-id="${task.id}" data-subtask-index="${index}"><svg class="w-4 h-4 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg></button></div></div>`).join(''); 
            
            const subtaskProgress = task.subtasks && task.subtasks.length > 0 ? `(${task.subtasks.filter(s=>s.completed).length}/${task.subtasks.length})` : ''; 
            const categoryClass = categoryMap[task.category].color || categoryMap['Other'].color; 
            const hasNotes = task.notes && task.notes.trim() !== ''; 
            const hasSubtasks = task.subtasks && task.subtasks.length > 0; 
            const noteSnippet = hasNotes ? task.notes.substring(0, 40) + (task.notes.length > 40 ? '...' : '') : ''; 
            const subtaskIndicatorHtml = hasSubtasks ? `<svg class="w-4 h-4 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01"></path></svg>` : ''; 
            
            // Conditional Edit History Button in List View
            const editHistoryButton = (task.history && task.history.length > 0) 
                ? `<button class="show-history-btn text-gray-500 hover:text-indigo-600 p-1.5 rounded-full hover:bg-gray-100" title="Edit History" data-task-id="${task.id}"><svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 12m15.356 2H18.5V10m-1.42 6.5l-3.21 3.21a2 2 0 01-2.83 0l-3.2-3.21m8.88-8.88l-3.21-3.21a2 2 0 00-2.83 0l-3.2 3.21"></path></svg></button>`
                : '';

            return `<li class="task-item ${task.completed ? 'completed' : ''}" data-id="${task.id}" data-datetime="${taskDateTime.toISOString()}"><div class="flex items-start justify-between p-4"><div class="flex items-start gap-3 flex-grow min-w-0"><input type="checkbox" class="task-checkbox h-5 w-5 rounded border-gray-300 text-indigo-600 flex-shrink-0 mt-1 focus:ring-indigo-500" ${task.completed ? 'checked' : ''}><div class="min-w-0"><div class="flex items-center gap-2 flex-wrap"><span class="priority-flag ${priorityFlagMap[task.priority] || 'bg-gray-400'}"></span><span class="flex items-center gap-2 text-xs font-semibold px-2 py-1 rounded-full ${categoryClass}">${categoryMap[task.category].icon} ${task.category}</span><span class="font-medium break-words ${task.completed ? 'task-title-completed' : ''}">${task.text}</span></div><div class="text-xs text-gray-500 mt-1 flex items-center gap-3 flex-wrap pl-8 ${task.completed ? 'completed' : ''}"><span>${taskDateTime.toLocaleDateString('en-US', { day: 'numeric', month: 'short' })} ${formatTo12Hour(task.time)}</span>${!task.completed ? `<span class="mx-1">•</span> ${timeDiffHtml}` : ''}${hasNotes ? `<div class="flex items-center gap-1 text-gray-400" title="${task.notes}"><svg class="w-4 h-4 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 10h.01M12 10h.01M16 10h.01M9 16H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-5l-5 5v-5z"></path></svg><span class="truncate italic">${noteSnippet}</span></div>`: ''}${hasSubtasks ? subtaskIndicatorHtml : ''}</div> ${completedAtHtml} </div></div><div class="flex items-center flex-shrink-0 ml-2">${editHistoryButton}<button class="details-btn text-gray-500 hover:text-indigo-600 p-1.5 rounded-full hover:bg-gray-100" title="Details"><svg class="w-5 h-5 transition-transform" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg></button><button class="edit-btn text-gray-500 hover:text-indigo-600 p-1.5 rounded-full hover:bg-gray-100" title="Edit"><svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"></path></svg></button><button class="delete-btn text-gray-500 hover:text-red-500 p-1.5 rounded-full hover:bg-gray-100" title="Delete"><svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg></button></div></div><div class="details-container px-4"><div class="notes-container border-t pt-4"><h4 class="text-xs font-bold text-gray-500 uppercase mb-2">Notes</h4><textarea class="task-notes-textarea" placeholder="Click to add notes...">${task.notes || ''}</textarea></div><div class="subtask-container border-t pt-4 mt-4"><h4 class="text-xs font-bold text-gray-500 uppercase mb-2">Subtasks ${subtaskProgress}</h4><div class="space-y-2">${subtasksHtml}<div class="grid grid-cols-3 gap-2 pt-2"><input type="text" class="new-subtask-input col-span-2 bg-gray-100 rounded px-2 py-1 text-sm border focus:outline-none focus:ring-1 focus:ring-indigo-500" placeholder="New subtask..."><input type="date" class="new-subtask-date-input bg-gray-100 rounded px-2 py-1 text-sm border focus:outline-none focus:ring-1 focus:ring-indigo-500"><button class="add-subtask-btn col-span-3 mt-1 bg-indigo-500 hover:bg-indigo-600 text-white text-xs py-1 rounded">Add Subtask</button></div></div></div></div><div class="subtask-alert text-red-500 text-xs font-bold mt-2 px-4 pb-2 hidden"></div></li>`; }
        function renderKanbanView(tasks) { DOMElements.kanbanBoard.innerHTML = '<p class="text-center text-sm text-gray-500 mb-4 col-span-full">Drag and drop tasks to change their category.</p>'; const columns = { 'Personal': [], 'Work': [], 'School': [], 'Other': [] }; tasks.forEach(task => { if (columns[task.category]) { columns[task.category].push(task); }}); for (const category in columns) { const columnEl = document.createElement('div'); columnEl.className = 'kanban-column'; columnEl.dataset.category = category; columnEl.innerHTML = `<h3 class="kanban-column-title">${categoryMap[category].icon} ${category}</h3><div class="kanban-tasks"></div>`; const tasksContainer = columnEl.querySelector('.kanban-tasks'); columns[category].forEach(task => { const cardEl = document.createElement('div'); cardEl.className = 'kanban-card'; cardEl.dataset.id = task.id; cardEl.draggable = true; cardEl.innerHTML = `<div class="flex items-center gap-2"><svg class="w-5 h-5 text-gray-400 cursor-grab flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path></svg><span class="flex-grow">${task.text}</span></div>`; tasksContainer.appendChild(cardEl); }); DOMElements.kanbanBoard.appendChild(columnEl); } }

        // --- CALENDAR VIEW LOGIC ---

        function getTaskSummaryBadges(task) {
            const priorityFlag = `<span class="priority-flag ${priorityFlagMap[task.priority] || 'bg-gray-400'}"></span>`;
            const categoryBadge = `<span class="flex items-center gap-1 text-xs font-semibold px-2 py-0.5 rounded-full ${categoryMap[task.category]?.color || categoryMap['Other'].color}">${categoryMap[task.category]?.icon || categoryMap['Other'].icon} ${task.category}</span>`;
            const timeDisplay = task.time ? `<span class="text-gray-500 text-xs">${formatTo12Hour(task.time)}</span>` : '';

            return { priorityFlag, categoryBadge, timeDisplay };
        }

        // Updated renderCalendarView to apply urgency classes to the day box
        function renderCalendarView() { 
            const m = currentCalendarDate.getMonth(), y = currentCalendarDate.getFullYear(); 
            DOMElements.calendarGrid.innerHTML = ''; 
            DOMElements.calendarMonthYear.textContent = new Date(y, m).toLocaleDateString('en-US', { month: 'long', year: 'numeric' }); 
            const firstDay = new Date(y, m, 1).getDay(), daysInMonth = new Date(y, m + 1, 0).getDate(); 
            const filteredTasks = getFilteredTasks();
            
            for (let i = 0; i < firstDay; i++) DOMElements.calendarGrid.insertAdjacentHTML('beforeend', `<div class="calendar-day other-month"></div>`); 
            
            for (let d = 1; d <= daysInMonth; d++) { 
                const dateString = new Date(y, m, d).toISOString().split('T')[0];
                const dayEl = document.createElement('div'); 
                dayEl.className = 'calendar-day'; 
                dayEl.dataset.date = dateString; 

                const today = new Date(); 
                if (d === today.getDate() && m === today.getMonth() && y === today.getFullYear()) dayEl.classList.add('is-today'); 
                
                // Get tasks for the current day to place dots and initialize urgency classes
                const tasksOnDay = filteredTasks.filter(t => t.date === dateString);

                // This initial class setup is redundant but safe since updateDynamicElements runs right after refreshDynamicContent
                // We rely on updateDynamicElements to apply/remove the pulsing effects every 15s.

                const headerEl = document.createElement('div'); 
                headerEl.className = 'calendar-day-header'; 
                headerEl.textContent = d; 
                dayEl.appendChild(headerEl); 
                
                const tasksContainer = document.createElement('div'); 
                tasksContainer.className = 'flex flex-wrap pointer-events-none justify-center'; 
                tasksOnDay.slice(0, 9).forEach(task => { 
                    tasksContainer.innerHTML += `<span class="task-dot ${categoryDotMap[task.category] || 'bg-gray-400'}" title="${task.text}"></span>`; 
                }); 
                dayEl.appendChild(tasksContainer); 
                DOMElements.calendarGrid.appendChild(dayEl); 
            }
        }

        // Updated to use the Modal Pop-up
        function handleCalendarDayClick(e) { 
            const dayEl = e.target.closest('.calendar-day:not(.other-month)'); 
            if (!dayEl) return; 

            const dateString = dayEl.dataset.date;
            const date = new Date(dateString + 'T00:00:00'); 
            
            const tasks = allTasks
                .filter(t => t.date === dateString)
                .sort((a, b) => (new Date(`${a.date}T${a.time || '23:59:59'}`) - new Date(`${b.date}T${b.time || '23:59:59'}`)));

            DOMElements.calendarModalTitle.textContent = `Tasks for ${date.toLocaleDateString('en-US', { dateStyle: 'full' })}`; 
            DOMElements.calendarModalTasks.innerHTML = tasks.length 
                ? tasks.map(t => {
                    const { priorityFlag, categoryBadge, timeDisplay } = getTaskSummaryBadges(t);
                    const taskDateTime = new Date(`${t.date}T${t.time || '23:59:59'}`);
                    
                    // Determine task urgency classes for pulsing inside the modal
                    let urgencyClasses = '';
                    if (!t.completed) {
                        const now = new Date();
                        const diffHours = (taskDateTime.getTime() - now.getTime()) / 3600000;
                        if (diffHours < 0) {
                            urgencyClasses = 'pulse-overdue';
                        } else if (diffHours <= 1) {
                            urgencyClasses = 'pulse-due-soon';
                        }
                    }

                    // Reusing logic from the task list item for details dropdown
                    const notesHtml = t.notes ? `<div class="notes-container pt-4 border-t"><h4 class="text-xs font-bold text-gray-500 uppercase mb-2">Notes</h4><p class="text-sm text-gray-700 whitespace-pre-wrap">${t.notes}</p></div>` : '';
                    
                    return `
                        <div class="calendar-task-item p-3 ${t.completed ? 'completed opacity-90' : 'bg-white'} flex flex-col gap-2 shadow-sm border border-gray-100 ${urgencyClasses}" data-id="${t.id}" data-datetime="${taskDateTime.toISOString()}">
                            <div class="flex items-start justify-between">
                                <div class="flex items-start gap-3 flex-grow min-w-0">
                                    <input type="checkbox" class="task-checkbox h-4 w-4 rounded border-gray-300 text-indigo-600 flex-shrink-0 mt-1 focus:ring-indigo-500" ${t.completed ? 'checked' : ''} data-id="${t.id}">
                                    <div class="min-w-0">
                                        <div class="flex items-center gap-2 flex-wrap">
                                            ${priorityFlag}
                                            ${categoryBadge}
                                            <span class="font-medium break-words ${t.completed ? 'task-title-completed text-gray-500' : 'text-gray-800'}">${t.text}</span>
                                        </div>
                                        <div class="text-xs text-gray-500 mt-1 flex items-center gap-3 flex-wrap pl-8 ${t.completed ? 'completed' : ''}">
                                            <span>${timeDisplay}</span>
                                            ${!t.completed ? `<span class="mx-1">•</span> <span class="relative-time" data-datetime="${taskDateTime.toISOString()}"></span>` : ''}
                                        </div>
                                    </div>
                                </div>
                                <button class="details-btn text-gray-500 hover:text-indigo-600 p-1.5 rounded-full hover:bg-gray-100" title="Details"><svg class="w-5 h-5 transition-transform" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg></button>
                            </div>
                            <div class="details-container px-4">
                                ${notesHtml}
                            </div>
                        </div>
                    `;
                }).join('') 
                : '<p class="text-gray-500">No tasks for this day.</p>'; 

            // Attach listeners for interaction within the modal
            attachDynamicListeners(DOMElements.calendarModalTasks);

            DOMElements.calendarDayModal.classList.remove('hidden');
            updateDynamicElements(); // Apply pulsing urgency colors for tasks inside the modal
        }
        // --- END CALENDAR VIEW LOGIC ---

        function showHistoryModal(taskId) {
            const task = allTasks.find(t => t.id === taskId);
            if (!task || !task.history || task.history.length === 0) return;

            DOMElements.historyModalContent.innerHTML = task.history.map(entry => {
                const timestamp = new Date(entry.timestamp).toLocaleString();
                const changeDetails = entry.changes.map(change => 
                    `<li class="text-sm text-gray-700 ml-4">• <span class="font-semibold">${change.field}:</span> from <span class="text-red-500">${change.oldValue}</span> to <span class="text-green-600">${change.newValue}</span></li>`
                ).join('');

                return `
                    <div class="border p-3 rounded-lg bg-gray-50">
                        <p class="font-bold text-gray-800">${entry.action || 'Edited'}</p>
                        <p class="text-xs text-gray-500 mb-2">${timestamp}</p>
                        <ul class="space-y-1">${changeDetails}</ul>
                    </div>
                `;
            }).join('');

            DOMElements.historyModal.classList.remove('hidden');
        }


        function setupEventListeners() { 
            DOMElements.headerStatsBtn.addEventListener('click', calculateAndShowStats); 
            DOMElements.headerAddTaskBtn.addEventListener('click', showAddTaskModal); 
            
            // Logout confirmation panel toggle
            DOMElements.headerLogoutBtn.addEventListener('click', () => DOMElements.logoutPanel.classList.toggle('hidden'));
            DOMElements.confirmLogoutBtn.addEventListener('click', handleLogout); 
            DOMElements.cancelLogoutBtn.addEventListener('click', () => DOMElements.logoutPanel.classList.add('hidden'));

            DOMElements.headerTodayBtn.addEventListener('click', handleTodayClick); 
            
            DOMElements.headerSettingsBtn.addEventListener('click', () => { 
                DOMElements.settingsModal.classList.remove('hidden'); 
            });
            DOMElements.settingsModalCloseBtn.addEventListener('click', () => { 
                DOMElements.settingsModal.classList.add('hidden'); 
            });
            DOMElements.toggleShowCompleted.addEventListener('change', refreshDynamicContent);

            DOMElements.todayPanelCloseBtn.addEventListener('click', () => { 
                DOMElements.todayPanel.classList.add('hidden'); 
            }); 
            
            DOMElements.cancelAddTaskBtn.addEventListener('click', hideAddTaskModal); 
            DOMElements.addTaskForm.addEventListener('submit', handleAddTask); 
            DOMElements.searchInput.addEventListener('input', e => { 
                currentSearchTerm = e.target.value; 
                refreshDynamicContent(); 
            }); 
            DOMElements.categoryFilters.addEventListener('click', handleCategoryFilter); 
            DOMElements.viewListBtn.addEventListener('click', () => switchView('list')); 
            DOMElements.viewBoardBtn.addEventListener('click', () => switchView('board')); 
            DOMElements.viewCalendarBtn.addEventListener('click', () => switchView('calendar')); 
            DOMElements.cancelDeleteBtn.addEventListener('click', hideConfirmModal); 
            DOMElements.confirmDeleteBtn.addEventListener('click', () => { 
                if (taskToDeleteId) { 
                    crudOperation('delete', { id: taskToDeleteId }); 
                    hideConfirmModal(); 
                } 
            }); 
            DOMElements.cancelEditBtn.addEventListener('click', hideEditModal); 
            DOMElements.editTaskForm.addEventListener('submit', handleEditTask); 

            // History Button Listeners
            DOMElements.editHistoryBtn.addEventListener('click', () => showHistoryModal(taskToEditId));
            DOMElements.historyModalCloseBtn.addEventListener('click', () => DOMElements.historyModal.classList.add('hidden'));

            DOMElements.prevMonthBtn.addEventListener('click', () => { 
                currentCalendarDate.setMonth(currentCalendarDate.getMonth() - 1); 
                renderCalendarView(); 
            }); 
            DOMElements.nextMonthBtn.addEventListener('click', () => { 
                currentCalendarDate.setMonth(currentCalendarDate.getMonth() + 1); 
                renderCalendarView(); 
            }); 
            DOMElements.calendarGrid.addEventListener('mouseover', showCalendarTooltip); 
            DOMElements.calendarGrid.addEventListener('mouseout', hideCalendarTooltip); 
            DOMElements.calendarGrid.addEventListener('click', handleCalendarDayClick); 
            
            // Close the calendar task modal
            DOMElements.calendarModalCloseBtn.addEventListener('click', () => DOMElements.calendarDayModal.classList.add('hidden'));

            DOMElements.statsModalCloseBtn.addEventListener('click', () => DOMElements.statsModal.classList.add('hidden')); 
            DOMElements.deleteAllDataBtn.addEventListener('click', handleDeleteAllData); 
            DOMElements.kanbanBoard.addEventListener('dragstart', e => { 
                if(e.target.classList.contains('kanban-card')) { 
                    e.dataTransfer.setData('text/plain', e.target.dataset.id); 
                    e.target.classList.add('dragging'); 
                } 
            }); 
            DOMElements.kanbanBoard.addEventListener('dragend', e => { 
                if(e.target.classList.contains('kanban-card')) e.target.classList.remove('dragging'); 
            }); 
            DOMElements.kanbanBoard.addEventListener('dragover', e => { 
                e.preventDefault(); 
                const column = e.target.closest('.kanban-column'); 
                if(column) column.classList.add('drag-over'); 
            }); 
            DOMElements.kanbanBoard.addEventListener('dragleave', e => { 
                const column = e.target.closest('.kanban-column'); 
                if(column) column.classList.remove('drag-over'); 
            }); 
            DOMElements.kanbanBoard.addEventListener('drop', e => { 
                e.preventDefault(); 
                const column = e.target.closest('.kanban-column'); 
                if(column) { 
                    column.classList.remove('drag-over'); 
                    const taskId = e.dataTransfer.getData('text/plain'); 
                    const newCategory = column.dataset.category; 
                    crudOperation('update', {id: taskId, category: newCategory}); 
                } 
            }); 
            document.addEventListener('click', (e) => { 
                // Close panels on outside click
                if (!DOMElements.headerTodayBtn.contains(e.target) && !DOMElements.todayPanel.contains(e.target)) { 
                    DOMElements.todayPanel.classList.add('hidden'); 
                } 
                if (!DOMElements.headerLogoutBtn.contains(e.target) && !DOMElements.logoutPanel.contains(e.target)) {
                    DOMElements.logoutPanel.classList.add('hidden');
                }
            }); 
        }

        function attachDynamicListeners(container) { 
            container.querySelectorAll('li.task-item, .calendar-task-item').forEach(taskItem => {
                const taskId = taskItem.dataset.id;
                
                // Toggle Checkbox
                taskItem.querySelector('.task-checkbox')?.addEventListener('change', e => {
                    const isCompleted = e.target.checked; 
                    const task = allTasks.find(t => t.id === taskId);
                    
                    if (isCompleted && task && task.subtasks && task.subtasks.length > 0 && !task.subtasks.every(s => s.completed)) {
                         // Subtask check logic
                        const alertDiv = taskItem.querySelector('.subtask-alert');
                        if (alertDiv) {
                            alertDiv.textContent = "Please complete all subtasks first!";
                            alertDiv.classList.remove('hidden');
                            taskItem.classList.add('flash-error');
                            setTimeout(() => {
                                alertDiv.classList.add('hidden');
                                taskItem.classList.remove('flash-error');
                            }, 3000);
                        }
                        e.target.checked = false; 
                        return; 
                    }
                    
                    const updatePayload = { id: taskId, completed: isCompleted, completedAt: isCompleted ? new Date().toISOString() : null }; 
                    if (isCompleted) { handleRecurringTask(allTasks.find(t => t.id === taskId)); } 
                    crudOperation('update', updatePayload);
                });

                // Details Button (for List View and Calendar Modal)
                taskItem.querySelector('.details-btn')?.addEventListener('click', e => {
                    const details = taskItem.querySelector('.details-container'); 
                    const icon = e.target.closest('.details-btn').querySelector('svg'); 
                    details.classList.toggle('open'); 
                    icon.classList.toggle('rotate-180');
                });
                
                // Edit/Delete buttons (only visible in List View)
                taskItem.querySelector('.edit-btn')?.addEventListener('click', () => showEditModal(taskId));
                taskItem.querySelector('.delete-btn')?.addEventListener('click', () => showConfirmModal(taskId));
                taskItem.querySelector('.show-history-btn')?.addEventListener('click', () => showHistoryModal(taskId));

                // Subtask/Notes logic (only present in List View's full details-container)
                const notesTextarea = taskItem.querySelector('.task-notes-textarea');
                if (notesTextarea) {
                    notesTextarea.addEventListener('input', e => autoResizeTextarea(e.target));
                    notesTextarea.addEventListener('blur', e => {
                        crudOperation('update', { id: taskId, notes: e.target.value });
                    }, true);
                }
                
                // Add subtask logic
                taskItem.querySelector('.add-subtask-btn')?.addEventListener('click', e => {
                    const textInput = taskItem.querySelector('.new-subtask-input');
                    const dateInput = taskItem.querySelector('.new-subtask-date-input');
                    if (textInput.value.trim()) {
                        handleAddSubtask(taskId, textInput.value.trim(), dateInput.value);
                        textInput.value = '';
                        dateInput.value = '';
                    }
                });
            });
        }
        
        async function handleAddTask(e) { 
            e.preventDefault(); 
            const { addTaskText, addTaskDate, addTaskTime, addTaskCategory, addTaskPriority, addTaskTags, addTaskRepeat } = DOMElements; 
            if (addTaskText.value.trim() && addTaskDate.value) { 
                await crudOperation('add', { 
                    text: addTaskText.value.trim(), 
                    date: addTaskDate.value, 
                    time: addTaskTime.value, 
                    category: addTaskCategory.value, 
                    priority: addTaskPriority.value, 
                    completed: false, 
                    notes: '', 
                    subtasks: [], 
                    completedAt: null, 
                    tags: parseTags(addTaskTags.value), 
                    recurrence: addTaskRepeat.value, 
                    history: [] // Initialize history on creation
                }); 
                hideAddTaskModal(); 
            } 
        }

        async function handleEditTask(e) { 
            e.preventDefault(); 
            const task = allTasks.find(t => t.id === taskToEditId); 
            if (task) { 
                // Prepare new task data, crudOperation will handle history logging inside
                const newTaskData = { 
                    ...task, 
                    id: taskToEditId, 
                    text: DOMElements.editTaskText.value, 
                    date: DOMElements.editTaskDate.value, 
                    time: DOMElements.editTaskTime.value, 
                    category: DOMElements.editTaskCategory.value, 
                    priority: DOMElements.editTaskPriority.value, 
                    notes: DOMElements.editTaskNotes.value, 
                    tags: parseTags(DOMElements.editTaskTags.value) 
                };

                await crudOperation('update', newTaskData); 
                hideEditModal(); 
            } 
        }

        function showEditModal(taskId) { 
            taskToEditId = taskId; 
            const task = allTasks.find(t => t.id === taskId); 
            if(task) { 
                DOMElements.editTaskText.value = task.text; 
                DOMElements.editTaskDate.value = task.date; 
                DOMElements.editTaskTime.value = task.time || ''; 
                DOMElements.editTaskCategory.value = task.category; 
                DOMElements.editTaskPriority.value = task.priority || 'medium'; 
                DOMElements.editTaskNotes.value = task.notes || ''; 
                DOMElements.editTaskTags.value = task.tags ? task.tags.join(', ') : ''; 
                
                // Conditional display of history button
                if (task.history && task.history.length > 0) {
                    DOMElements.editHistoryBtn.classList.remove('hidden');
                } else {
                    DOMElements.editHistoryBtn.classList.add('hidden');
                }
                
                DOMElements.editModal.classList.remove('hidden'); 
            } 
        }
        
        function handleCategoryFilter(e) { const target = e.target.closest('.category-filter-btn'); if (target) { currentCategoryFilter = target.dataset.category; styleCategoryFilters(); refreshDynamicContent(); } }
        function handleAddSubtask(taskId, text, dueDate) { const task = allTasks.find(t => t.id === taskId); if (task) { const newSubtask = { text, completed: false, completedAt: null, dueDate: dueDate || null }; crudOperation('update', { id: taskId, subtasks: [...(task.subtasks || []), newSubtask] }); } }
        function handleSubtaskToggle(taskId, subtaskIndex) { const task = allTasks.find(t => t.id === taskId); if (task) { const newSubtasks = task.subtasks.map((sub, i) => { if (i === subtaskIndex) { const isCompleted = !sub.completed; return { ...sub, completed: isCompleted, completedAt: isCompleted ? new Date().toISOString() : null }; } return sub; }); crudOperation('update', { id: taskId, subtasks: newSubtasks }); } }
        function handleEditSubtask(button) { const { taskId, subtaskIndex } = button.dataset; const task = allTasks.find(t => t.id === taskId); if (!task) return; const oldText = task.subtasks[subtaskIndex].text; const newText = prompt("Edit subtask:", oldText); if (newText && newText.trim() !== oldText) { const newSubtasks = task.subtasks.map((sub, i) => i == subtaskIndex ? { ...sub, text: newText.trim() } : sub); crudOperation('update', { id: taskId, subtasks: newSubtasks }); } }
        function handleDeleteSubtask(button) { if (!confirm('Are you sure you want to delete this subtask?')) return; const { taskId, subtaskIndex } = button.dataset; const task = allTasks.find(t => t.id === taskId); if (task) { const newSubtasks = task.subtasks.filter((_, i) => i != subtaskIndex); crudOperation('update', { id: taskId, subtasks: newSubtasks }); } }
        
        function calculateAndShowStats() { const completedTasks = allTasks.filter(t => t.completed); const totalCompleted = completedTasks.length; const totalTasks = allTasks.length; DOMElements.statsTotalCompleted.textContent = totalCompleted; DOMElements.statsCompletionRate.textContent = totalTasks > 0 ? `${Math.round((totalCompleted / totalTasks) * 100)}%` : 'N/A'; const now = new Date(); const startOfWeek = new Date(now.setDate(now.getDate() - now.getDay())); startOfWeek.setHours(0,0,0,0); const completedThisWeek = completedTasks.filter(t => new Date(t.completedAt) >= startOfWeek).length; DOMElements.statsCompletedThisWeek.textContent = completedThisWeek; const completedByCategory = completedTasks.reduce((acc, task) => { const category = task.category || 'Other'; if (!acc[category]) acc[category] = []; acc[category].push(task); return acc; }, {}); DOMElements.statsCategoryBreakdown.innerHTML = Object.keys(completedByCategory).length > 0 ? Object.entries(completedByCategory).map(([cat, tasks]) => `<div class="mt-2"><h5 class="font-semibold text-sm ${categoryMap[cat].color.split(' ')[1]} flex items-center gap-2">${categoryMap[cat].icon} ${cat}</h5><ul class="space-y-1">${tasks.map(t => { const dueTime = t.time ? ` at ${formatTo12Hour(t.time)}` : ''; const completedTime = t.completedAt ? ` at ${formatTo12Hour(new Date(t.completedAt).toTimeString())}` : ''; return `<li class="ml-4 text-sm text-gray-600"><span class="font-medium">${t.text}</span><div class="text-xs text-gray-400 pl-2">Due: ${new Date(t.date + 'T00:00:00').toLocaleDateString('en-US',{month:'short',day:'numeric'})}${dueTime} | Completed: ${new Date(t.completedAt).toLocaleDateString('en-US',{month:'short',day:'numeric'})}${completedTime}</div></li>`}).join('')}</ul></div>`).join('') : `<p class="text-gray-500 text-sm">No completed tasks yet.</p>`; DOMElements.statsModal.classList.remove('hidden'); }
        
        function handleLogout() { 
            DOMElements.logoutPanel.classList.add('hidden'); // Hide the confirmation panel
            showIndicator("Logging out...", "info", true); 
            signOut(auth).then(() => { 
                hideIndicator(); 
                DOMElements.loginForm.reset(); 
                showFlashMessage(DOMElements.logoutSuccessMessage, "Successfully logged out!"); 
            }); 
        }

        function handleTodayClick() { renderTodayPanel(); DOMElements.todayPanel.classList.toggle('hidden'); }
        function renderTodayPanel() { const today = new Date(); const todayStr = today.toISOString().split('T')[0]; today.setDate(today.getDate() + 1); const tomorrowStr = today.toISOString().split('T')[0]; const overdueTasks = allTasks.filter(t => !t.completed && t.date < todayStr); const todayTasks = allTasks.filter(t => !t.completed && t.date === todayStr); const tomorrowTasks = allTasks.filter(t => !t.completed && t.date === tomorrowStr); const renderSection = (title, tasks, color) => { if(tasks.length === 0) return `<div class="p-3"><h4 class="font-semibold text-sm ${color} mb-2">${title}</h4><p class="text-xs text-gray-400">Nothing to show.</p></div>`; return `<div class="p-3"><h4 class="font-semibold text-sm ${color} mb-2">${title}</h4><ul class="space-y-2">${tasks.map(t => `<li class="text-xs text-gray-700 flex items-center gap-2"><span class="priority-flag ${priorityFlagMap[t.priority]}"></span>${t.text}</li>`).join('')}</ul></div>`; }; DOMElements.todayPanelContent.innerHTML = renderSection('Overdue', overdueTasks, 'text-red-500') + renderSection('Today', todayTasks, 'text-indigo-500') + renderSection('Tomorrow', tomorrowTasks, 'text-gray-500'); }
        function handleDeleteAllData() { const confirmation = prompt('This will permanently delete ALL of your tasks. This action cannot be undone. To confirm, please type "DELETE" below:'); if (confirmation === "DELETE") { showIndicator("Deleting all data...", "warning"); const deletePromises = allTasks.map(task => deleteDoc(doc(db, "users", auth.currentUser.uid, "tasks", task.id))); Promise.all(deletePromises) .then(() => { showIndicator("All tasks deleted", "success"); }) .catch(error => { console.error("Error deleting all tasks:", error); hideIndicator(); alert("An error occurred while deleting your data."); }); } else { alert("Deletion cancelled. Your data is safe."); } }
        function showCalendarTooltip(e) { const dayEl = e.target.closest('.calendar-day:not(.other-month)'); if (!dayEl) return; const day = parseInt(dayEl.dataset.day, 10); const tasks = getFilteredTasks().filter(t => new Date(t.date + 'T00:00:00').getDate() === day && new Date(t.date + 'T00:00:00').getMonth() === currentCalendarDate.getMonth()); if (tasks.length === 0) return; const tooltip = DOMElements.calendarTooltip; tooltip.innerHTML = `<ul class="space-y-1">${tasks.map(t => `<li class="flex items-center gap-2 text-xs"><div class="w-2 h-2 rounded-full ${categoryDotMap[t.category]} flex-shrink-0"></div><span class="${t.completed ? 'completed task-title-completed' : ''}">${t.text}</span></li>`).join('')}</ul>`; const rect = dayEl.getBoundingClientRect(); tooltip.style.left = `${rect.left}px`; tooltip.style.top = `${rect.bottom + 5}px`; tooltip.classList.remove('hidden'); setTimeout(() => tooltip.style.opacity = 1, 10); }
        function hideCalendarTooltip() { const tooltip = DOMElements.calendarTooltip; tooltip.style.opacity = 0; setTimeout(() => tooltip.classList.add('hidden'), 200); }
        function getRelativeDateGroup(dateString) { const today = new Date(); today.setHours(0,0,0,0); const tomorrow = new Date(today); tomorrow.setDate(tomorrow.getDate() + 1); const taskDate = new Date(dateString + 'T00:00:00'); if (taskDate < today) return "Overdue"; if (taskDate.getTime() === today.getTime()) return "Today"; if (taskDate.getTime() === tomorrow.getTime()) return "Tomorrow"; return taskDate.toLocaleDateString('en-US', { weekday: 'long', month: 'short', day: 'numeric'}); }
        function switchView(view) { 
            currentView = view; 
            const views = { list: DOMElements.listViewContainer, board: DOMElements.kanbanViewContainer, calendar: DOMElements.calendarViewContainer }; 
            const buttons = { list: DOMElements.viewListBtn, board: DOMElements.viewBoardBtn, calendar: DOMElements.viewCalendarBtn }; 
            for (const v in views) { 
                views[v].classList.toggle('hidden', v !== view); 
                buttons[v].classList.toggle('bg-indigo-600', v === view); 
                buttons[v].classList.toggle('text-white', v === view); 
            } 
            DOMElements.categoryFilters.classList.toggle('hidden', view === 'calendar'); 
            DOMElements.searchInput.placeholder = view === 'board' ? 'Filter board...' : 'Search tasks or #tags...'; 
            
            // Hide the details modal when switching out of calendar view
            DOMElements.calendarDayModal.classList.add('hidden');
            
            refreshDynamicContent(); 
        }
        function showAddTaskModal() { DOMElements.addTaskForm.reset(); setSmartDefaults(); DOMElements.addTaskModal.classList.remove('hidden'); DOMElements.addTaskText.focus(); }
        function hideAddTaskModal() { DOMElements.addTaskModal.classList.add('hidden'); }
        function hideEditModal() { DOMElements.editModal.classList.add('hidden'); taskToEditId = null; }
        function showConfirmModal(taskId) { taskToDeleteId = taskId; DOMElements.confirmModal.classList.remove('hidden'); }
        function hideConfirmModal() { taskToDeleteId = null; DOMElements.confirmModal.classList.add('hidden'); }
        function getGreeting() { const h = new Date().getHours(); if (h < 12) return 'Good morning'; if (h < 18) return 'Good afternoon'; return 'Good evening'; }
        function getInitials(email) { if (!email) return '?'; const p = email.split('@')[0].split(/[._-]/); return p.length > 1 ? (p[0][0] + p[1][0]) : email[0]; }
        function formatTo12Hour(timeString) { if (!timeString) return ''; const date = new Date(); const [hour, minute] = timeString.split(':'); date.setHours(hour, minute); return date.toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit', hour12: true }); }
        
        // IMPORTANT: Updated updateDynamicElements to correctly apply pulsing effects.
        function updateDynamicElements() { 
            const now = new Date();
            
            // 1. Update urgency for List View tasks and Calendar Modal tasks
            document.querySelectorAll('.task-item:not(.completed), .calendar-task-item:not(.completed)').forEach(el => { 
                const isoDate = el.dataset.datetime; 
                if (isoDate) { 
                    const taskDateTime = new Date(isoDate); 
                    const diffHours = (taskDateTime.getTime() - now.getTime()) / 3600000; 
                    el.classList.remove('pulse-overdue', 'pulse-due-soon'); 
                    if (diffHours < 0) { 
                        el.classList.add('pulse-overdue'); 
                    } else if (diffHours <= 1 && diffHours >= 0) {
                        el.classList.add('pulse-due-soon'); 
                    } 
                } 
            }); 
            
            // 2. Update urgency for Calendar Day Boxes
            document.querySelectorAll('.calendar-day:not(.other-month)').forEach(dayEl => {
                const dateString = dayEl.dataset.date;
                if (!dateString) return;

                const tasksOnDay = allTasks.filter(t => t.date === dateString && !t.completed);
                
                // Remove previous pulse classes to prevent stacking animation
                dayEl.classList.remove('pulse-overdue', 'pulse-due-soon'); 

                const overdueTask = tasksOnDay.some(t => new Date(`${t.date}T${t.time || '23:59:59'}`) < now);
                const dueSoonTask = tasksOnDay.some(t => {
                    const taskDateTime = new Date(`${t.date}T${t.time || '23:59:59'}`);
                    const diffHours = (taskDateTime.getTime() - now.getTime()) / 3600000;
                    return diffHours > 0 && diffHours <= 1;
                });
                
                if (overdueTask) {
                    dayEl.classList.add('pulse-overdue');
                } else if (dueSoonTask) {
                    dayEl.classList.add('pulse-due-soon');
                }
            });

            // 3. Update relative times everywhere
            document.querySelectorAll('.relative-time').forEach(el => { 
                const isoDate = el.dataset.datetime; 
                if (isoDate) { 
                    el.textContent = formatTimeDifference(isoDate); 
                    const diffHours = (new Date(isoDate).getTime() - new Date().getTime()) / 3600000; 
                    el.classList.remove('text-red-600', 'text-yellow-600'); 
                    if (diffHours < 0) { 
                        el.classList.add('text-red-600'); 
                    } else if (diffHours <= 1) { 
                        el.classList.add('text-yellow-600'); 
                    } 
                } 
            }); 
        }

        function formatTimeDifference(isoDate) { const rtf = new Intl.RelativeTimeFormat('en', { numeric: 'auto' }); const diffSeconds = (new Date(isoDate).getTime() - Date.now()) / 1000; const diffDays = diffSeconds / 86400; if (Math.abs(diffDays) >= 1) return rtf.format(Math.round(diffDays), 'day'); const diffHours = diffSeconds / 3600; if (Math.abs(diffHours) >= 1) return rtf.format(Math.round(diffHours), 'hour'); const diffMinutes = diffSeconds / 60; if (Math.abs(diffMinutes) >= 1) return rtf.format(Math.round(diffMinutes), 'minute'); return 'just now'; }
        function showFlashMessage(element, message) { element.textContent = message; element.classList.remove('hidden'); setTimeout(() => element.classList.add('hidden'), 3000); }
        function clearUI() { DOMElements.taskListContainer.innerHTML = ''; if (DOMElements.dashboardPendingBreakdown) DOMElements.dashboardPendingBreakdown.innerHTML = ''; if(DOMElements.dashboardNextTask) DOMElements.dashboardNextTask.innerHTML = `<p class="text-gray-400 text-center">No upcoming tasks!</p>`; if(DOMElements.dashboardOverdue) DOMElements.dashboardOverdue.innerHTML = `<p class="text-gray-400 text-center">No overdue tasks!</p>`; if(DOMElements.dashboardOverdueCount) DOMElements.dashboardOverdueCount.textContent = '0'; }
        function setSmartDefaults() { DOMElements.addTaskDate.value = new Date().toISOString().split('T')[0]; const now = new Date(); DOMElements.addTaskTime.value = `${String((now.getHours() + 1) % 24).padStart(2, '0')}:00`; }
        
        function showIndicator(message, status = "info", showSpinner = false, duration = 500) { 
            DOMElements.indicatorText.textContent = message; 
            DOMElements.appIndicator.className = `fixed bottom-5 left-1/2 -translate-x-1/2 text-white text-sm py-2 px-4 rounded-full shadow-lg flex items-center gap-2 z-50 ${status}`;
            
            const spinner = document.getElementById('indicator-spinner');
            if (spinner) {
                spinner.classList.toggle('hidden', !showSpinner);
            }
            
            DOMElements.appIndicator.classList.remove('hidden'); 

            if (!showSpinner) {
                setTimeout(hideIndicator, duration);
            }
        }
        
        function hideIndicator() { DOMElements.appIndicator.classList.add('hidden'); }
        
        function styleCategoryFilters() { document.querySelectorAll('.category-filter-btn').forEach(btn => { const category = btn.dataset.category; if (category !== 'All' && !btn.querySelector('svg')) { btn.innerHTML = `${categoryMap[category].icon} ${category}`; } btn.className = 'category-filter-btn text-sm font-medium px-3 py-1 rounded-full flex items-center gap-2'; if (category === currentCategoryFilter) { if (category === 'All') btn.classList.add('bg-indigo-600', 'text-white'); else btn.className += ` ${categoryActiveMap[category]}`; } else { if (category === 'All') btn.classList.add('bg-gray-200', 'text-gray-800'); else btn.className += ` ${categoryMap[category].color}`; } }); }
        function parseTags(tagsString) { return tagsString.split(',').map(tag => tag.trim()).filter(tag => tag.length > 0 && tag.startsWith('#')).map(tag => tag.toLowerCase()); }
        function handleRecurringTask(task) { if(!task.recurrence || task.recurrence === 'none') return; const nextDueDate = new Date(task.date + 'T00:00:00'); if(task.recurrence === 'daily') nextDueDate.setDate(nextDueDate.getDate() + 1); else if (task.recurrence === 'weekly') nextDueDate.setDate(nextDueDate.getDate() + 7); else if (task.recurrence === 'monthly') nextDueDate.setMonth(nextDueDate.getMonth() + 1); const { id, completed, completedAt, ...new_task_data } = task; const newTask = { ...new_task_data, date: nextDueDate.toISOString().split('T')[0], completed: false, completedAt: null, }; crudOperation('add', newTask); crudOperation('update', { id: task.id, recurrence: 'none' }); }
        function autoResizeTextarea(textarea) { textarea.style.height = 'auto'; textarea.style.height = textarea.scrollHeight + 'px'; }
        
        DOMElements.loginForm.addEventListener('submit', async (e) => { 
            e.preventDefault(); 
            DOMElements.authError.textContent = ''; 
            showIndicator("Signing in...", "info", true); 
            const email = e.target.elements['login-email'].value; 
            const password = e.target.elements['login-password'].value; 
            try { 
                await signInWithEmailAndPassword(auth, email, password); 
                sessionStorage.setItem('loginJustOccurred', 'true'); 
                hideIndicator();
            } catch (error) { 
                DOMElements.authError.textContent = 'Incorrect email or password.'; 
                hideIndicator(); 
            } 
        });
        
        setupEventListeners();
        styleCategoryFilters();
    </script>
</body>
</html>
