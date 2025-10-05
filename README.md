{\rtf1\ansi\ansicpg1252\cocoartf2865
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0

\f0\fs24 \cf0 <!DOCTYPE html>\
<html lang="en">\
<head>\
    <meta charset="UTF-8">\
    <meta name="viewport" content="width=device-width, initial-scale=1.0">\
    <title>Interactive ECE Degree Plan Explorer</title>\
    <!-- Chosen Palette: Warm Neutrals (Slate, Stone, with UT Orange/Blue accents) -->\
    <!-- Application Structure Plan: A dashboard-style SPA with a primary filter control for years (1-5). The main view displays course cards in a semester grid, alongside a dynamic donut chart visualizing the course-type distribution. This structure was chosen to allow users to easily switch between a high-level overview (all years) and a detailed annual view, making the 5-year academic journey digestible and explorable rather than a static list. -->\
    <!-- Visualization & Content Choices: Source: Course data from CSV. Goal: Visualize the academic focus per year. Viz: A dynamic Chart.js donut chart shows the breakdown of course types (Math, Physics, ECE, Core, etc.). Interaction: The chart updates on a year filter click, providing immediate visual feedback on the changing curriculum focus (e.g., more core in early years, more specialization later). Justification: This visual summary is more intuitive than reading a long table and highlights the curriculum's progression. All course info is presented in HTML cards for clarity. NO SVG/Mermaid used. -->\
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->\
    <script src="https://cdn.tailwindcss.com"></script>\
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>\
    <style>\
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');\
        body \{\
            font-family: 'Inter', sans-serif;\
            background-color: #f8fafc;\
        \}\
        .chart-container \{\
            position: relative;\
            height: 400px;\
            width: 100%;\
            max-width: 400px;\
            margin: auto;\
        \}\
         .active-filter \{\
            background-color: #bf5700 !important;\
            color: white !important;\
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);\
        \}\
        .calendar-grid \{\
            display: grid;\
            grid-template-columns: 50px repeat(5, 1fr);\
            grid-template-rows: 30px repeat(10, 1fr);\
            gap: 2px;\
        \}\
        .time-label \{\
            font-size: 10px;\
            text-align: right;\
            padding-right: 5px;\
            color: #64748b;\
        \}\
        .day-header \{\
            font-size: 12px;\
            font-weight: 600;\
            text-align: center;\
            color: #475569;\
        \}\
        .class-block \{\
            border-radius: 4px;\
            padding: 4px 6px;\
            font-size: 11px;\
            overflow: hidden;\
            color: white;\
            line-height: 1.2;\
        \}\
    </style>\
</head>\
<body class="text-slate-800">\
\
    <div class="max-w-7xl mx-auto p-4 sm:p-6 lg:p-8">\
        <header class="text-center mb-10">\
            <h1 class="text-4xl font-bold text-slate-900">B.S. + M.S. ECE Degree Plan</h1>\
            <p class="text-lg text-slate-600 mt-2">Interactive Explorer: TAMS to UT Austin</p>\
            <p class="text-md text-slate-500 mt-1">Specialization: Fields, Waves, & Electromagnetic Systems</p>\
        </header>\
\
        <main>\
            <div id="controls" class="flex flex-wrap justify-center gap-2 mb-12">\
                <button data-year="all" class="filter-btn active-filter font-semibold py-2 px-5 rounded-full bg-white text-slate-700 shadow-sm hover:bg-slate-100 transition-all">All Years</button>\
                <button data-year="1" class="filter-btn font-semibold py-2 px-5 rounded-full bg-white text-slate-700 shadow-sm hover:bg-slate-100 transition-all">Year 1</button>\
                <button data-year="2" class="filter-btn font-semibold py-2 px-5 rounded-full bg-white text-slate-700 shadow-sm hover:bg-slate-100 transition-all">Year 2</button>\
                <button data-year="3" class="filter-btn font-semibold py-2 px-5 rounded-full bg-white text-slate-700 shadow-sm hover:bg-slate-100 transition-all">Year 3</button>\
                <button data-year="4" class="filter-btn font-semibold py-2 px-5 rounded-full bg-white text-slate-700 shadow-sm hover:bg-slate-100 transition-all">Year 4</button>\
                <button data-year="5" class="filter-btn font-semibold py-2 px-5 rounded-full bg-white text-slate-700 shadow-sm hover:bg-slate-100 transition-all">Year 5</button>\
            </div>\
\
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">\
                <div id="plan-display" class="lg:col-span-2 space-y-8">\
                    <!-- Dynamic Content Here -->\
                </div>\
                \
                <aside class="lg:col-span-1 lg:sticky top-8 self-start">\
                     <div class="bg-white p-6 rounded-2xl shadow-lg">\
                         <h3 id="chart-title" class="text-xl font-bold text-center mb-4 text-slate-900">Course Breakdown: All Years</h3>\
                         <div class="chart-container">\
                             <canvas id="course-chart"></canvas>\
                         </div>\
                         <p class="text-xs text-slate-500 mt-4 text-center">This chart visualizes the distribution of course types for the selected academic year(s). Notice the shift from foundational math and physics to specialized ECE courses as you progress.</p>\
                     </div>\
                </aside>\
            </div>\
            \
            <div id="weekly-schedules" class="mt-16">\
                 <h2 class="text-3xl font-bold text-center text-slate-900 mb-10">Sample Weekly Schedules (UT Austin)</h2>\
                 <div class="grid grid-cols-1 md:grid-cols-2 gap-8">\
                    <!-- Weekly Schedules Rendered Here -->\
                 </div>\
            </div>\
\
        </main>\
    </div>\
\
    <script>\
        const courseData = [\
            \{ year: 1, semester: "Fall", institution: "TAMS/UNT", number: "MATH 1710", title: "Calculus I", notes: "UT Equivalent: M 408C" \},\
            \{ year: 1, semester: "Fall", institution: "TAMS/UNT", number: "PHYS 1710 & 1730", title: "Mechanics + (Lab)", notes: "UT Equivalent: PHY 303K & 105M" \},\
            \{ year: 1, semester: "Fall", institution: "TAMS/UNT", number: "ENGL 1315", title: "College Writing I", notes: "Fulfills part of Core Curriculum" \},\
            \{ year: 1, semester: "Fall", institution: "TAMS/UNT", number: "EENG 1910", title: "Engineering Foundations I", notes: "Covers intro ECE concepts" \},\
            \{ year: 1, semester: "Fall", institution: "TAMS/UNT", number: "HIST 2610", title: "US History to 1865", notes: "Fulfills part of Core Curriculum" \},\
            \{ year: 1, semester: "Spring", institution: "TAMS/UNT", number: "MATH 1720", title: "Calculus II", notes: "UT Equivalent: M 408D" \},\
            \{ year: 1, semester: "Spring", institution: "TAMS/UNT", number: "MATH 2700", title: "Linear Algebra", notes: "UT Equivalent: M 340L" \},\
            \{ year: 1, semester: "Spring", institution: "TAMS/UNT", number: "PHYS 2220 & 2240", title: "Electricity & Magnetism (+Lab)", notes: "UT Equivalent: PHY 303L & 105N" \},\
            \{ year: 1, semester: "Spring", institution: "TAMS/UNT", number: "ENGL 1325", title: "College Writing II", notes: "Fulfills part of Core (RHE 306)" \},\
            \{ year: 1, semester: "Spring", institution: "TAMS/UNT", number: "EENG 2610", title: "Circuit Analysis", notes: "UT Equivalent: ECE 411" \},\
            \{ year: 2, semester: "Fall", institution: "TAMS/UNT", number: "MATH 2730", title: "Multivariable Calculus", notes: "Advanced Math Credit" \},\
            \{ year: 2, semester: "Fall", institution: "TAMS/UNT", number: "CSCE 1030", title: "Computer Science I", notes: "UT Equivalent: ECE 406" \},\
            \{ year: 2, semester: "Fall", institution: "TAMS/UNT", number: "HIST 2620", title: "US History Since 1865", notes: "Fulfills Core Curriculum" \},\
            \{ year: 2, semester: "Fall", institution: "TAMS/UNT", number: "PSCI 2305", title: "US Political Behavior & Policy", notes: "UT Equivalent: GOV 310L" \},\
            \{ year: 2, semester: "Fall", institution: "TAMS/UNT", number: "EENG 3510", title: "Electronics I", notes: "UT Equivalent: ECE 339" \},\
            \{ year: 2, semester: "Spring", institution: "TAMS/UNT", number: "MATH 3410", title: "Differential Equations", notes: "UT Equivalent: M 427J" \},\
            \{ year: 2, semester: "Spring", institution: "TAMS/UNT", number: "PSCI 2306", title: "US & Texas Constitutions", notes: "UT Equivalent: GOV 312L/P" \},\
            \{ year: 2, semester: "Spring", institution: "TAMS/UNT", number: "EENG 2710", title: "Digital Logic Design", notes: "Foundational ECE Course" \},\
            \{ year: 2, semester: "Spring", institution: "TAMS/UNT", number: "-", title: "TAMS Elective", notes: "" \},\
            \{ year: 3, semester: "Fall", institution: "UT Austin", number: "ECE 333T", title: "Engineering Communications", notes: "Core Requirement" \},\
            \{ year: 3, semester: "Fall", institution: "UT Austin", number: "ECE 351K", title: "Probability/Random Processes", notes: "Upper-Division Requirement" \},\
            \{ year: 3, semester: "Fall", institution: "UT Austin", number: "M 427L", title: "Advanced Calculus for Applications II", notes: "Required Math for Specialization" \},\
            \{ year: 3, semester: "Fall", institution: "UT Austin", number: "ECE 325", title: "Electromagnetic Engineering", notes: "Core Specialization Course" \},\
            \{ year: 3, semester: "Fall", institution: "UT Austin", number: "ECE 370C", title: "Physical and Applied Optics", notes: "Fulfills Advanced Tech Elective" \},\
            \{ year: 3, semester: "Spring", institution: "UT Austin", number: "DES 308", title: "Design in Daily Life", notes: "Core: Visual & Performing Arts" \},\
            \{ year: 3, semester: "Spring", institution: "UT Austin", number: "ECO 301", title: "Introduction to Economics", notes: "Core: Social & Behavioral Science" \},\
            \{ year: 3, semester: "Spring", institution: "UT Austin", number: "ECE 363M", title: "Engineering Computation", notes: "Specialization Course (Theory focus)" \},\
            \{ year: 3, semester: "Spring", institution: "UT Austin", number: "ECE 438", title: "Analog Electronics", notes: "Fulfills Lab Requirement" \},\
            \{ year: 3, semester: "Spring", institution: "UT Austin", number: "-", title: "Introduction to Astrophysics", notes: "Astrophysics/Quantum Elective" \},\
            \{ year: 4, semester: "Fall", institution: "UT Austin", number: "ECE 364D", title: "Intro to Engineering Design", notes: "Prepares for Senior Design" \},\
            \{ year: 4, semester: "Fall", institution: "UT Austin", number: "-", title: "General Relativity", notes: "Astrophysics/Quantum Elective" \},\
            \{ year: 4, semester: "Fall", institution: "UT Austin", number: "-", title: "Computational Electromagnetics", notes: "Technical Elective 1" \},\
            \{ year: 4, semester: "Fall", institution: "UT Austin", number: "-", title: "Fiber Optics", notes: "Technical Elective 2" \},\
            \{ year: 4, semester: "Fall", institution: "UT Austin", number: "-", title: "Quantum Information & Computation", notes: "Additional Advanced Elective" \},\
            \{ year: 4, semester: "Spring", institution: "UT Austin", number: "ECE 464K/H", title: "Senior Design Project", notes: "Capstone Course" \},\
            \{ year: 4, semester: "Spring", institution: "UT Austin", number: "-", title: "Quantum Electronics", notes: "Technical Elective 3" \},\
            \{ year: 4, semester: "Spring", institution: "UT Austin", number: "-", title: "Quantum Mechanics", notes: "Fulfills Adv. Math/Science Elective" \},\
            \{ year: 4, semester: "Spring", institution: "UT Austin", number: "-", title: "Partial Differential Equations", notes: "Fulfills remaining degree hours" \},\
            \{ year: 5, semester: "Fall", institution: "UT Austin", number: "-", title: "Advanced Electromagnetic Theory I", notes: "Core Graduate Course" \},\
            \{ year: 5, semester: "Fall", institution: "UT Austin", number: "-", title: "Advanced Solid-State Devices", notes: "Graduate Elective" \},\
            \{ year: 5, semester: "Fall", institution: "UT Austin", number: "-", title: "Laser Systems", notes: "Graduate Elective" \},\
            \{ year: 5, semester: "Fall", institution: "UT Austin", number: "-", title: "Advanced Fiber & Optical Comm.", notes: "Graduate Elective" \},\
            \{ year: 5, semester: "Fall", institution: "UT Austin", number: "ECE 398M", title: "Master's Report / Thesis I", notes: "" \},\
            \{ year: 5, semester: "Spring", institution: "UT Austin", number: "-", title: "Advanced Electromagnetic Theory II", notes: "Core Graduate Course" \},\
            \{ year: 5, semester: "Spring", institution: "UT Austin", number: "-", title: "Statistical Optics", notes: "Graduate Elective" \},\
            \{ year: 5, semester: "Spring", institution: "UT Austin", number: "-", title: "Plasma Processing", notes: "Graduate Elective" \},\
            \{ year: 5, semester: "Spring", institution: "UT Austin", number: "-", title: "Computational Physics/Math", notes: "Graduate Elective" \},\
            \{ year: 5, semester: "Spring", institution: "UT Austin", number: "ECE 398M", title: "Master's Report / Thesis II", notes: "" \},\
        ];\
        \
        const weeklyScheduleData = [\
            \{ name: "Year 3 - Fall Semester", classes: [\
                \{ title: "Adv. Calculus", days: "MWF", start: 9, end: 10, cat: "Math" \},\
                \{ title: "EM Engineering", days: "MWF", start: 11, end: 12, cat: "Specialization" \},\
                \{ title: "Probability", days: "TTh", start: 11, end: 12.5, cat: "Specialization" \},\
                \{ title: "Optics", days: "TTh", start: 14, end: 15.5, cat: "Specialization" \},\
                \{ title: "Eng. Comms", days: "W", start: 15, end: 17, cat: "Core Curriculum" \},\
            ]\},\
            \{ name: "Year 3 - Spring Semester", classes: [\
                \{ title: "Eng. Computation", days: "MWF", start: 10, end: 11, cat: "Specialization" \},\
                \{ title: "Astrophysics", days: "MWF", start: 13, end: 14, cat: "Physics" \},\
                \{ title: "Analog Elec. (Lab)", days: "M", start: 14, end: 17, cat: "Specialization" \},\
                \{ title: "Design in Daily Life", days: "TTh", start: 9.5, end: 11, cat: "Core Curriculum" \},\
                \{ title: "Economics", days: "TTh", start: 14, end: 15.5, cat: "Core Curriculum" \},\
            ]\},\
            \{ name: "Year 4 - Fall Semester", classes: [\
                \{ title: "General Relativity", days: "MWF", start: 10, end: 11, cat: "Physics" \},\
                \{ title: "Quantum Info", days: "MWF", start: 14, end: 15, cat: "Specialization" \},\
                \{ title: "Comp. EM", days: "TTh", start: 9.5, end: 11, cat: "Specialization" \},\
                \{ title: "Fiber Optics", days: "TTh", start: 12.5, end: 14, cat: "Specialization" \},\
                \{ title: "Eng. Design", days: "TTh", start: 15.5, end: 17, cat: "Specialization" \},\
            ]\},\
            \{ name: "Year 4 - Spring Semester", classes: [\
                \{ title: "Quantum Mechanics", days: "MWF", start: 11, end: 12, cat: "Physics" \},\
                \{ title: "PDEs", days: "TTh", start: 11, end: 12.5, cat: "Math" \},\
                \{ title: "Quantum Electronics", days: "TTh", start: 14, end: 15.5, cat: "Specialization" \},\
                \{ title: "Senior Design", days: "F", start: 13, end: 16, cat: "Specialization" \},\
            ]\},\
            \{ name: "Year 5 - Fall Semester (Grad)", classes: [\
                \{ title: "Adv. EM Theory I", days: "MW", start: 9, end: 10.5, cat: "Graduate" \},\
                \{ title: "Adv. Solid-State", days: "MW", start: 13, end: 14.5, cat: "Graduate" \},\
                \{ title: "Laser Systems", days: "TTh", start: 10, end: 11.5, cat: "Graduate" \},\
                \{ title: "Adv. Fiber Optics", days: "TTh", start: 14, end: 15.5, cat: "Graduate" \},\
                \{ title: "Thesis/Report", days: "F", start: 10, end: 12, cat: "Graduate" \},\
            ]\},\
            \{ name: "Year 5 - Spring Semester (Grad)", classes: [\
                \{ title: "Adv. EM Theory II", days: "MW", start: 9, end: 10.5, cat: "Graduate" \},\
                \{ title: "Statistical Optics", days: "MW", start: 13, end: 14.5, cat: "Graduate" \},\
                \{ title: "Plasma Processing", days: "TTh", start: 10, end: 11.5, cat: "Graduate" \},\
                \{ title: "Comp. Physics", days: "TTh", start: 14, end: 15.5, cat: "Graduate" \},\
                \{ title: "Thesis/Report", days: "F", start: 10, end: 12, cat: "Graduate" \},\
            ]\},\
        ];\
\
        let myChart;\
        const planDisplay = document.getElementById('plan-display');\
        const chartTitle = document.getElementById('chart-title');\
        const filterButtons = document.querySelectorAll('.filter-btn');\
        const weeklySchedulesContainer = document.querySelector('#weekly-schedules .grid');\
\
        const categoryColors = \{\
            'Math': '#0d9488', \
            'Physics': '#2563eb', \
            'Foundational ECE': '#9333ea', \
            'Specialization': '#c026d3',\
            'Core Curriculum': '#db2777',\
            'Elective': '#e11d48',\
            'Graduate': '#f97316'\
        \};\
\
        const getCourseCategory = (course) => \{\
            const title = course.title.toLowerCase();\
            const num = course.number;\
            \
            if (course.year === 5) return 'Graduate';\
            if (num.startsWith('MATH') || num.startsWith('M ')) return 'Math';\
            if (num.startsWith('PHYS') || title.includes('physics') || title.includes('astrophysics') || title.includes('relativity')) return 'Physics';\
            if (num.startsWith('EENG') || ['Electronic Devices'].includes(course.title)) return 'Foundational ECE';\
            if (num.startsWith('ECE') || ['Computational Electromagnetics', 'Fiber Optics', 'Quantum Electronics', 'Quantum Information & Computation'].includes(course.title)) return 'Specialization';\
            if (['College Writing', 'US History', 'US Political', 'US & Texas', 'Design in Daily Life', 'Introduction to Economics'].some(t => title.includes(t.toLowerCase()))) return 'Core Curriculum';\
            if (num.startsWith('CSCE')) return 'Foundational ECE';\
            return 'Elective';\
        \};\
\
        const renderPlan = (filteredData) => \{\
            planDisplay.innerHTML = '';\
            const years = [...new Set(filteredData.map(c => c.year))].sort((a,b) => a - b);\
            years.forEach(year => \{\
                const yearData = filteredData.filter(c => c.year === year);\
                const yearDiv = document.createElement('div');\
                yearDiv.innerHTML = `<div class="sticky top-0 z-10 bg-slate-100 py-3 px-4 rounded-t-xl"><h2 class="text-2xl font-bold text-slate-800">Year $\{year\} <span class="text-base font-medium text-slate-500">$\{yearData[0].year > 2 ? (year > 4 ? 'UT Austin (M.S.)' : 'UT Austin (B.S.)') : 'TAMS / UNT'\}</span></h2></div>`;\
                const semestersDiv = document.createElement('div');\
                semestersDiv.className = 'grid grid-cols-1 md:grid-cols-2 gap-4 bg-white p-4 rounded-b-xl shadow-lg';\
                const fallCourses = yearData.filter(c => c.semester === 'Fall');\
                const springCourses = yearData.filter(c => c.semester === 'Spring');\
                semestersDiv.innerHTML = `<div><h3 class="font-semibold text-lg mb-3 pb-2 border-b border-slate-200">Fall Semester</h3><div class="space-y-3">$\{fallCourses.map(course => createCourseCard(course)).join('')\}</div></div><div><h3 class="font-semibold text-lg mb-3 pb-2 border-b border-slate-200">Spring Semester</h3><div class="space-y-3">$\{springCourses.map(course => createCourseCard(course)).join('')\}</div></div>`;\
                yearDiv.appendChild(semestersDiv);\
                planDisplay.appendChild(yearDiv);\
            \});\
        \};\
\
        const createCourseCard = (course) => \{\
            const category = getCourseCategory(course);\
            const color = categoryColors[category] || '#64748b';\
            return `<div class="border-l-4 p-3 bg-slate-50 rounded-lg" style="border-color: $\{color\};"><p class="font-semibold text-slate-900">$\{course.title\}</p><p class="text-sm text-slate-500">$\{course.number\}</p><p class="text-xs text-slate-400 mt-1">$\{course.notes\}</p></div>`;\
        \};\
\
        const updateChart = (filteredData) => \{\
            const categories = filteredData.map(getCourseCategory);\
            const counts = categories.reduce((acc, category) => \{ acc[category] = (acc[category] || 0) + 1; return acc; \}, \{\});\
            const labels = Object.keys(counts);\
            const data = Object.values(counts);\
            const backgroundColors = labels.map(label => categoryColors[label]);\
            if (myChart) \{ myChart.destroy(); \}\
            const ctx = document.getElementById('course-chart').getContext('2d');\
            myChart = new Chart(ctx, \{\
                type: 'doughnut',\
                data: \{ labels: labels, datasets: [\{ label: 'Course Count', data: data, backgroundColor: backgroundColors, borderColor: '#ffffff', borderWidth: 2, hoverOffset: 4 \}] \},\
                options: \{ responsive: true, maintainAspectRatio: false, plugins: \{ legend: \{ position: 'bottom', labels: \{ padding: 15, font: \{ family: "'Inter', sans-serif" \} \} \}, tooltip: \{ callbacks: \{ label: function(context) \{ let label = context.label || ''; if (label) \{ label += ': '; \} if (context.parsed !== null) \{ label += context.parsed; \} return label; \} \} \} \} \}\
            \});\
        \};\
        \
        const renderSchedules = () => \{\
            weeklySchedulesContainer.innerHTML = '';\
            weeklyScheduleData.forEach(semester => \{\
                const semesterWrapper = document.createElement('div');\
                semesterWrapper.className = 'bg-white p-4 rounded-xl shadow-lg';\
\
                let calendarHTML = `<h3 class="text-lg font-bold text-center mb-4">$\{semester.name\}</h3><div class="calendar-grid bg-slate-100 p-2 rounded-lg">`;\
                calendarHTML += `<div></div><div class="day-header">Mon</div><div class="day-header">Tue</div><div class="day-header">Wed</div><div class="day-header">Thu</div><div class="day-header">Fri</div>`;\
\
                for (let i = 8; i <= 17; i++) \{\
                    calendarHTML += `<div class="time-label">$\{i\}:00</div>` + '<div class="bg-white"></div>'.repeat(5);\
                \}\
\
                semester.classes.forEach(c => \{\
                    const days = c.days.split('');\
                    const startRow = (c.start - 8) + 2;\
                    const duration = c.end - c.start;\
                    const dayMap = \{ 'M': 2, 'T': 3, 'W': 4, 'Th': 5, 'F': 6 \};\
                    if(c.days === "TTh") \{\
                        dayMap['T'] = 3;\
                        dayMap['Th'] = 5;\
                    \}\
                    \
                    const dayIndices = c.days.match(/T(?!h)/g) ? [dayMap['T']] : [];\
                    if (c.days.includes('M')) dayIndices.push(dayMap['M']);\
                    if (c.days.includes('W')) dayIndices.push(dayMap['W']);\
                    if (c.days.includes('F')) dayIndices.push(dayMap['F']);\
                    if (c.days.includes('Th')) dayIndices.push(dayMap['Th']);\
                     if (c.days.match(/T(?!h)/)) dayIndices.push(dayMap['T']);\
\
\
                    [...new Set(c.days.replace('Th', 'H').split(''))].forEach(dayChar => \{\
                        const dayStr = dayChar === 'H' ? 'Th' : dayChar;\
                        const dayCol = \{M:2, T:3, W:4, H:5, F:6\}[dayChar];\
                        \
                        calendarHTML += `<div class="class-block" style="background-color: $\{categoryColors[c.cat]\}; grid-column: $\{dayCol\}; grid-row: $\{startRow\} / span $\{Math.ceil(duration * 2) / 2\};">$\{c.title\}</div>`;\
                    \});\
                \});\
                \
                calendarHTML += '</div>';\
                semesterWrapper.innerHTML = calendarHTML;\
                weeklySchedulesContainer.appendChild(semesterWrapper);\
            \});\
        \};\
        \
        const handleFilterClick = (e) => \{\
            const year = e.target.dataset.year;\
            filterButtons.forEach(btn => btn.classList.remove('active-filter'));\
            e.target.classList.add('active-filter');\
            const filteredData = (year === 'all') ? courseData : courseData.filter(c => c.year == year);\
            renderPlan(filteredData);\
            updateChart(filteredData);\
            chartTitle.textContent = (year === 'all') ? 'Course Breakdown: All Years' : `Course Breakdown: Year $\{year\}`;\
        \};\
\
        filterButtons.forEach(button => \{\
            button.addEventListener('click', handleFilterClick);\
        \});\
\
        // Initial render\
        renderPlan(courseData);\
        updateChart(courseData);\
        renderSchedules();\
    </script>\
</body>\
</html>\
\
}
