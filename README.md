<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nexa AI - Offline Assistant</title>
    <link href="https://fonts.googleapis.com/css2?family=Google+Sans:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        /* CSS variables for theme */
        :root {
            --bg: #ffffff;
            --text: #1f1f1f;
            --user-bg: #e6f4ea; /* Light green for user messages (Gemini-like) */
            --bot-bg: #f0f4f9; /* Light blue-gray for bot messages (Gemini-like) */
            --border-color: #dadce0; /* Softer border color */
            --input-bg: #ffffff;
            --button-bg: #1a73e8; /* Google blue */
            --button-hover: #0d47a1;
            --secondary-text: #5f6368; /* For less prominent text */
            --header-color: #4285f4; /* Google blue for main heading */
            --welcome-bg: #e8f0fe; /* Light blue for welcome card */
            --welcome-border: #d2e3fc;
        }

        body.dark-mode {
            --bg: #202124;
            --text: #e8eaed;
            --user-bg: #343a40; /* Darker gray for user messages */
            --bot-bg: #2a2c2e; /* Darker background for bot messages */
            --border-color: #5f6368;
            --input-bg: #303134;
            --button-bg: #8ab4f8; /* Lighter blue for dark mode button */
            --button-hover: #669df6;
            --secondary-text: #bdc1c6;
            --header-color: #8ab4f8;
            --welcome-bg: #2c2e30;
            --welcome-border: #4d5156;
        }

        body {
            margin: 0;
            font-family: 'Google Sans', Arial, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            transition: background-color 0.3s ease, color 0.3s ease;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
            box-sizing: border-box; /* Include padding in element's total width and height */
        }

        .container {
            max-width: 800px;
            width: 100%; /* Changed to 100% and then max-width for better responsiveness */
            height: 90vh; /* Make chat container take more vertical space */
            display: flex;
            flex-direction: column;
            border-radius: 24px; /* More rounded, modern look */
            background-color: var(--bg);
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1); /* Stronger shadow */
            overflow: hidden; /* Hide overflow from rounded corners */
        }

        h1 {
            text-align: center;
            padding: 20px 0;
            margin: 0;
            font-size: 2.2em;
            color: var(--header-color);
            border-bottom: 1px solid var(--border-color);
            background-color: var(--bg);
            z-index: 10; /* Ensure it stays on top */
        }

        h6 {
            text-align: center;
            margin-top: -15px; /* Pull it closer to the h1 */
            margin-bottom: 20px;
            color: var(--secondary-text);
            font-size: 0.9em;
        }

        #chat-window {
            flex-grow: 1; /* Allows chat window to take available vertical space */
            overflow-y: auto;
            padding: 15px 20px;
            border-bottom: 1px solid var(--border-color); /* Separator from input */
            display: flex;
            flex-direction: column;
            gap: 10px; /* Space between messages */
            scroll-behavior: smooth; /* Smooth scrolling for new messages */
        }

        .bot-message-container {
            display: flex;
            align-items: flex-start;
            animation: fadeIn 0.3s ease-in;
        }

        .bot-avatar {
            width: 32px; /* Slightly smaller avatar */
            height: 32px;
            border-radius: 50%;
            margin-right: 12px;
            flex-shrink: 0;
            background-color: #e0e0e0;
            object-fit: cover;
        }

        .message {
            padding: 12px 18px; /* More padding */
            border-radius: 20px; /* Very rounded bubbles */
            max-width: 80%; /* Allow more width */
            word-wrap: break-word;
            font-size: 1.0em; /* Slightly larger font */
            line-height: 1.5;
            box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05); /* Subtle shadow for bubbles */
        }

        .user-message {
            background: var(--user-bg);
            color: var(--text); /* User messages also use text color, but on a light green bg */
            margin-left: auto;
            align-self: flex-end;
        }

        .bot-message {
            background: var(--bot-bg);
            color: var(--text);
            align-self: flex-start;
        }

        #input-container {
            display: flex;
            gap: 12px;
            padding: 20px;
            background-color: var(--bg);
            border-top: 1px solid var(--border-color); /* Separator */
        }

        #user-input {
            flex: 1;
            padding: 14px 20px; /* Increased padding */
            border-radius: 24px; /* Pill-shaped input */
            border: 1px solid var(--border-color);
            background: var(--input-bg);
            color: var(--text);
            font-size: 1em;
            transition: border-color 0.3s ease, box-shadow 0.3s ease;
            outline: none;
        }

        #user-input:focus {
            border-color: var(--button-bg);
            box-shadow: 0 0 0 3px rgba(26, 115, 232, 0.2);
        }

        button {
            padding: 12px 28px;
            border: none;
            background: var(--button-bg);
            color: white;
            border-radius: 24px; /* Pill-shaped button */
            cursor: pointer;
            font-size: 1em;
            font-weight: 500;
            transition: background 0.3s ease, box-shadow 0.3s ease;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }

        button:hover {
            background: var(--button-hover);
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        }

        .fixed-action-btn {
            position: fixed;
            padding: 10px 18px;
            background: var(--button-bg);
            color: white;
            border-radius: 25px;
            cursor: pointer;
            font-size: 0.9em;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
            transition: background 0.3s ease, box-shadow 0.3s ease;
            z-index: 1000; /* Ensure they are on top */
        }

        .fixed-action-btn:hover {
            background: var(--button-hover);
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
        }

        /* Positions for action buttons */
        .settings-icon {
            top: 20px;
            right: 20px;
            font-size: 1.8em; /* Make the gear icon larger */
            padding: 8px 14px; /* Adjust padding for icon button */
            border-radius: 50%; /* Make it circular */
            width: 25px; /* Fixed width for circle */
            height: 25px; /* Fixed height for circle */
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .new-chat-btn {
            top: 20px;
            right: 100px; /* Positioned left of settings icon */
        }
        .chat-history-btn {
            top: 20px;
            right: 200px; /* Positioned left of new chat */
        }

        /* Settings Menu Styles */
        #settingsMenu {
            display: none; /* Hidden by default */
            position: fixed;
            top: 70px; /* Below settings icon */
            right: 20px;
            background-color: var(--bg);
            border: 1px solid var(--border-color);
            border-radius: 10px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15); /* Stronger shadow */
            padding: 15px;
            flex-direction: column;
            gap: 10px;
            z-index: 1999;
            opacity: 0;
            transform: translateY(-10px);
            transition: opacity 0.3s ease, transform 0.3s ease;
        }

        #settingsMenu.settings-menu-open {
            display: flex;
            opacity: 1;
            transform: translateY(0);
        }

        #settingsMenu button {
            width: 100%; /* Make buttons full width in the menu */
            padding: 10px 15px;
            justify-content: flex-start; /* Align text to left */
            font-size: 0.95em;
            border-radius: 8px; /* Slightly less rounded */
            background: var(--bot-bg); /* Use bot-bg for settings buttons */
            color: var(--text);
            box-shadow: none; /* No shadow inside menu */
        }
        #settingsMenu button:hover {
            background: var(--border-color); /* Hover effect */
            box-shadow: none;
        }


        /* Welcome Screen */
        .welcome-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: var(--bg);
            display: flex;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            z-index: 2000;
            transition: opacity 0.3s ease;
        }

        .welcome-card {
            background-color: var(--welcome-bg);
            border: 1px solid var(--welcome-border);
            padding: 40px;
            border-radius: 16px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
            text-align: center;
            max-width: 400px;
            width: 90%;
        }

        .welcome-card h2 {
            color: var(--header-color);
            margin-bottom: 25px;
            font-size: 1.8em;
        }

        .welcome-card input[type="text"],
        .welcome-card input[type="number"] {
            width: calc(100% - 24px); /* Account for padding */
            padding: 12px;
            margin-bottom: 15px;
            border: 1px solid var(--border-color);
            border-radius: 8px;
            font-size: 1em;
            background-color: var(--input-bg);
            color: var(--text);
        }

        .welcome-card button {
            width: 100%;
            padding: 12px;
            font-size: 1.1em;
        }

        /* Stylish Welcome Message in Chat */
        .stylish-welcome {
            font-size: 2em; /* Much larger */
            font-weight: 700;
            color: var(--header-color);
            text-align: center;
            margin: 20px auto; /* Center it */
            padding: 15px;
            background: var(--welcome-bg);
            border-radius: 15px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
            max-width: 70%;
            animation: slideInUp 0.5s ease-out;
        }

        @keyframes slideInUp {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        /* Chat History Sidebar */
        .chat-history-sidebar {
            position: fixed;
            top: 0;
            right: -300px; /* Hidden by default */
            width: 300px;
            height: 100%;
            background-color: var(--bg);
            border-left: 1px solid var(--border-color);
            box-shadow: -4px 0 10px rgba(0, 0, 0, 0.1);
            transition: right 0.3s ease-in-out;
            z-index: 1500;
            display: flex;
            flex-direction: column;
        }

        .chat-history-sidebar.open {
            right: 0;
        }

        .history-header {
            padding: 15px;
            font-size: 1.2em;
            font-weight: bold;
            color: var(--header-color);
            border-bottom: 1px solid var(--border-color);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .history-close-btn {
            background: none;
            border: none;
            font-size: 1.5em;
            cursor: pointer;
            color: var(--secondary-text);
        }

        .history-list {
            flex-grow: 1;
            overflow-y: auto;
            padding: 15px;
        }

        .history-item {
            padding: 10px;
            margin-bottom: 8px;
            background-color: var(--bot-bg);
            border-radius: 8px;
            cursor: pointer;
            transition: background-color 0.2s ease;
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-size: 0.9em;
        }

        .history-item:hover {
            background-color: var(--border-color);
        }
        
        .history-item .delete-history-btn {
            background: none;
            border: none;
            color: #ff4d4d;
            cursor: pointer;
            font-size: 1.1em;
            margin-left: 10px;
            transition: color 0.2s ease;
        }
        .history-item .delete-history-btn:hover {
            color: #cc0000;
        }

        /* Animation for typing dots */
        .typing-dots .dot {
            opacity: 0;
            animation: fadeInOut 1.5s infinite;
        }

        .typing-dots .dot:nth-child(1) {
            animation-delay: 0s;
        }
        .typing-dots .dot:nth-child(2) {
            animation-delay: 0.2s;
        }
        .typing-dots .dot:nth-child(3) {
            animation-delay: 0.4s;
        }

        @keyframes fadeInOut {
            0% { opacity: 0; }
            50% { opacity: 1; }
            100% { opacity: 0; }
        }

        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(10px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        /* Responsive adjustments */
        @media (max-width: 768px) {
            .container {
                width: 95%;
                height: 95vh; /* Take more height on smaller screens */
                border-radius: 16px;
            }
            h1 {
                font-size: 1.8em;
                padding: 15px 0;
            }
            h6 {
                font-size: 0.8em;
            }
            #chat-window {
                padding: 10px 15px;
            }
            .message {
                font-size: 0.95em;
                padding: 10px 15px;
                max-width: 90%;
            }
            #input-container {
                padding: 15px;
                flex-direction: column;
                gap: 8px;
            }
            #user-input, button {
                width: 100%; /* Full width */
                padding: 12px;
                font-size: 0.95em;
            }
            button {
                padding: 12px 20px; /* Adjust button padding */
            }
            .fixed-action-btn {
                padding: 8px 14px;
                font-size: 0.8em;
            }
            .settings-icon {
                top: 10px;
                right: 10px;
                width: 20px;
                height: 20px;
                font-size: 1.5em;
                padding: 5px 8px;
            }
            .new-chat-btn {
                top: 10px;
                right: 60px;
            }
            .chat-history-btn {
                top: 10px;
                right: 140px;
            }
            #settingsMenu {
                top: 55px; /* Adjust for smaller screens */
                right: 10px;
            }
            .welcome-card {
                padding: 30px;
            }
            .welcome-card h2 {
                font-size: 1.5em;
            }
            .stylish-welcome {
                font-size: 1.5em;
                max-width: 90%;
            }
            .chat-history-sidebar {
                width: 250px;
            }
        }
    </style>
</head>
<body>
    <div class="fixed-action-btn settings-icon" onclick="toggleSettingsMenu()">⚙️</div>
    <div class="fixed-action-btn new-chat-btn" onclick="newChat()">➕ New Chat</div>
    <div class="fixed-action-btn chat-history-btn" onclick="toggleHistorySidebar()">📚 Chat History</div>

    <div id="settingsMenu">
        <button onclick="toggleTheme()">🌙 <span id="darkModeToggleText">Toggle Dark Mode</span></button>
        <button onclick="toggleLanguage()">🇬🇧/🇵🇰 <span id="languageToggleText">Language</span></button>
    </div>

    <div class="welcome-screen" id="welcomeScreen">
        <div class="welcome-card">
            <h2 id="welcomeScreenTitle">Welcome to Nexa AI!</h2>
            <input type="text" id="welcomeNameInput" placeholder="Enter your Name" required>
            <input type="number" id="welcomeAgeInput" placeholder="Enter your Age" required>
            <button id="startChatBtn" onclick="startChat()">Start Chat</button>
        </div>
    </div>

    <div class="container" id="mainChatContainer" style="display: none;">
        <h1 id="mainChatH1">💬 Chat with Nexa AI</h1><h6 id="mainChatH6">+ support of Google</h6>
        <div id="chat-window"></div>
        <div id="input-container">
            <input type="text" id="user-input" placeholder="Type your message...">
            <button id="sendMessageBtn" onclick="sendMessage()">Send</button>
        </div>
    </div>

    <div class="chat-history-sidebar" id="chatHistorySidebar">
        <div class="history-header">
            <span id="chatHistoryHeader">Chat History</span>
            <button class="history-close-btn" onclick="toggleHistorySidebar()">✖</button>
        </div>
        <div class="history-list" id="historyList">
            </div>
    </div>

    <script>
        // Comprehensive knowledge base (as provided by you)
        const qaData = [
            // Famous Personalities
            { keywords: ["who is elon musk", "elon musk kon hai", "elon musk"], answer: "Elon Musk is the CEO of Tesla, SpaceX, and xAI. He’s a billionaire innovator working on electric cars, space travel, and AI to accelerate human progress." },
            { keywords: ["who is bill gates", "bill gates kon hai", "bill gates"], answer: "Bill Gates is the co-founder of Microsoft Corporation and a philanthropist. He’s known for creating Windows and leading the Bill & Melinda Gates Foundation." },
            { keywords: ["who is steve jobs", "steve jobs kon hai", "steve jobs"], answer: "Steve Jobs was the co-founder of Apple Inc., known for revolutionizing tech with the iPhone, iPad, and Mac. He passed away in 2011." },
            { keywords: ["who is mark zuckerberg", "mark zuckerberg kon hai", "mark zuckerberg"], answer: "Mark Zuckerberg is the founder of Facebook (now Meta), a social media platform that connects billions of people worldwide." },
            { keywords: ["who is abdul sattar edhi", "abdul sattar edhi kon hai", "abdul sattar edhi"], answer: "Abdul Sattar Edhi was a Pakistani philanthropist who founded the Edhi Foundation, providing free healthcare and emergency services. He’s a national hero." },
            { keywords: ["who is malala yousafzai", "malala kon hai", "malala yousafzai"], answer: "Malala Yousafzai is a Pakistani activist for girls’ education and a Nobel Peace Prize winner. She survived a Taliban attack and advocates globally." },
            { keywords: ["who is imran khan", "imran khan kon hai", "imran khan"], answer: "Imran Khan is a former Pakistani cricketer and politician, who served as Prime Minister of Pakistan from 2018 to 2022. He’s known for leading Pakistan to the 1992 Cricket World Cup victory and founding the PTI party." },
            { keywords: ["who is albert einstein", "albert einstein kon hai", "albert einstein", "einstein"], answer: "Albert Einstein was a German-born theoretical physicist who developed the theory of relativity, one of the two pillars of modern physics. His work is also known for its influence on the philosophy of science. He is best known for his mass–energy equivalence formula E = mc2." },
            { keywords: ["who is shah rukh khan", "shah rukh khan kon hai", "shah rukh khan"], answer: "Shah Rukh Khan is an Indian actor, known as the 'King of Bollywood,' famous for starring in films like Dilwale Dulhania Le Jayenge and Chennai Express." },
            { keywords: ["who is allama iqbal", "allama iqbal kon hai", "allama iqbal"], answer: "Allama Iqbal was a poet, philosopher, and politician, known as the spiritual founder of Pakistan for his poetry inspiring the Pakistan Movement." },
            // FOUNDER FIXES: More specific keywords for founders
            { keywords: ["who is quaid-e-azam", "quaid-e-azam kon hai", "muhammad ali jinnah", "who is muhammad ali jinnah", "founder of pakistan", "pakistan ka bani kon hai", "pakistan ka founder"], answer: "Quaid-e-Azam Muhammad Ali Jinnah was the founder of Pakistan, leading the All-India Muslim League to establish Pakistan in 1947 as its first Governor-General." },
            { keywords: ["what the name of the founder of india", "india ka founder kon hai", "mahatma gandhi", "gandhi", "india ka bani kon hai"], answer: "Mahatma Gandhi is widely considered the Father of the Nation in India, known for leading India to independence through nonviolent civil disobedience." },
            // Computer-Related Topics
            { keywords: ["full form of pc","what is the full form of pc","pc ki full form kya hai"], answer:"The full form of PC is (Personal Computer)."},
            { keywords: ["what is computer", "computer kya hai"], answer: "A computer is an electronic device that processes data using instructions (software) to perform tasks like calculations, Browse, or gaming." },
            { keywords: ["what is programming", "programming kya hai"], answer: "Programming is writing instructions (code) for computers to perform tasks, like creating apps or websites, using languages like Python or JavaScript." },
            { keywords: ["what is python", "python kya hai"], answer: "Python is a versatile, beginner-friendly programming language used for web development, AI, data analysis, and automation." },
            { keywords: ["what is html", "html kya hai"], answer: "HTML (HyperText Markup Language) is the standard language for creating web pages, defining their structure like headings, paragraphs, and links." },
            { keywords: ["what is css", "css kya hai"], answer: "CSS (Cascading Style Sheets) is used for styling web pages, controlling layout, colors, fonts, and overall appearance." },
            { keywords: ["what is javascript", "javascript kya hai"], answer: "JavaScript is a programming language used to make web pages interactive, like adding animations or handling user inputs." },
            { keywords: ["what is cpu", "cpu kya hai"], answer: "The CPU (Central Processing Unit) is the 'brain' of a computer, executing instructions from programs by performing calculations and operations." },
            { keywords: ["what is ram", "ram kya hai"], answer: "RAM (Random Access Memory) is a computer’s short-term memory, storing data temporarily for quick access while running programs." },
            { keywords: ["what is blockchain", "blockchain kya hai"], answer: "Blockchain is a decentralized digital ledger that records transactions across many computers, used in cryptocurrencies like Bitcoin for secure, transparent data." },
            { keywords: ["what is cloud computing", "cloud computing kya hai"], answer: "Cloud computing is delivering computing services like storage, processing, or software over the internet, using platforms like AWS or Google Cloud." },
            { keywords: ["what is database", "database kya hai"], answer: "A database is an organized collection of data, typically stored and accessed electronically from a computer system, like MySQL or MongoDB." },
            { keywords: ["what is operating system", "os kya hai", "operating system"], answer: "An Operating System (OS) is software that manages computer hardware and software resources, providing common services for computer programs. Examples include Windows, macOS, and Linux." },
            { keywords: ["what is network", "network kya hai"], answer: "A computer network is a group of interconnected computers and devices that can share resources and data, like the internet or a local area network (LAN)." },
            // YouTube and Social Media
            { keywords: ["what is youtube","whats youtube","youtube kya hai","bhai what is youtube","bhaiya youtube kya hai"],answer: "YouTube is a popular online video-sharing platform where users can upload, view, and share videos. It's owned by Google."},
            { keywords: ["how to create yt channel","how to create youtube channel"],answer: "To create a YouTube channel, sign in to YouTube with your Google account. Go to your channel list, click 'Create a new channel' or use a brand account, and then name your channel and customize it."},
            { keywords: ["how to go viral on youtube", "youtube par viral kaise ho"], answer: "To go viral on YouTube, create engaging content on trends (like ASMR, challenges, or current events), use catchy thumbnails, optimize titles with keywords, promote on social media, and post consistently. For local trends, ask about Pakistan/India!" },
            { keywords: ["how to upload videos on youtube", "upload video on youtube"], answer: "Open YouTube, click the 'Create' (camera) icon, select 'Upload Video,' choose your video file, add a title, description, tags, set visibility, and publish. Verify your account for longer videos and custom thumbnails." },
            { keywords: ["trends of pakistan and india", "trend of pk and ind"], answer: "Current popular online trends in Pakistan and India often involve short-form video content on platforms like TikTok and Instagram Reels. Look for trending audio, dance challenges, comedic skits, and cultural content. News and political events also drive trends." },
            // General Knowledge
            { keywords: ["what is gravity", "gravity kya hai"], answer: "Gravity is the force that pulls objects toward each other, like Earth keeping us grounded. It’s why things fall and planets orbit. It's a fundamental force of nature."},
            { keywords: ["capital of india", "india ki rajdhani"], answer: "The capital of India is New Delhi." },
            { keywords: ["capital of pakistan", "pakistan ki rajdhani"], answer: "The capital of Pakistan is Islamabad." },
            { keywords: ["what is ai", "ai kya hai", "artificial intelligence"], answer: "AI (Artificial Intelligence) is when computers mimic human thinking, like answering questions, recognizing images, or making decisions. I’m an example of AI!" },
            { keywords: ["how to make money online", "online paisa kaise kamaye"], answer: "You can earn online through freelancing (e.g., on platforms like Fiverr or Upwork), YouTube monetization, affiliate marketing, e-commerce (selling products), or selling digital products like eBooks or online courses. Pick a skill and start building a presence." },
            { keywords: ["national animal of india", "india ka rashtriya janwar"], answer: "The national animal of India is the Bengal Tiger." },
            { keywords: ["national animal of pakistan", "pakistan ka rashtriya janwar"], answer: "The national animal of Pakistan is the Markhor." },
            { keywords: ["what is solar system", "solar system kya hai"], answer: "The Solar System consists of the Sun and all the celestial objects—planets, dwarf planets, moons, asteroids, and comets—that orbit it. Earth is one of eight planets in our Solar System." },
            { keywords: ["what is internet", "internet kya hai"], answer: "The internet is a global network of computers that allows users to share information and communicate with each other. It's a vast network of interconnected devices." },
            { keywords: ["what is photosynthesis", "photosynthesis kya hai"], answer: "Photosynthesis is the process by which green plants and some other organisms use sunlight to synthesize foods with the help of chlorophyll. It's how plants convert light energy into chemical energy." },
            { keywords: ["what is democracy", "democracy kya hai"], answer: "Democracy is a system of government where citizens exercise power either directly or by electing representatives from among themselves to form a governing body, such as a parliament. It's government by the people." },
            { keywords: ["what is climate change", "climate change kya hai"], answer: "Climate change refers to long-term shifts in temperatures and weather patterns. These shifts may be natural, but since the 1800s, human activities have been the main driver of climate change, primarily due to the burning of fossil fuels." },
            // Greetings and Miscellaneous
            { keywords: ["hello", "hi", "hey", "salam", "assalam o alaikum"], answer: "Salam, {{name}}! I’m Nexa AI 🤖, your offline guru. Ask about famous personalities, computers, or anything else! Kya baat hai?" },
            { keywords: ["thank", "shukriya", "thanks", "thank you"], answer: "Shukriya, {{name}}! 😊 What’s next?" },
            { keywords: ["how are you", "kese ho", "kya haal hai"], answer: "Mai theek hoon, shukriya! Aur aap kaise hain? How can I help you today?" },
            { keywords: ["what is your name", "aapka naam kya hai"], answer: "My name is Nexa AI, a friendly offline assistant. Nice to meet you!" },
            { keywords: ["who made you", "tumhe kisne banaya"], answer: "I was created by a developer to assist you offline with various questions." },
            { keywords: ["where are you from", "kahan ke ho"], answer: "I exist in the digital realm, so I don't have a physical location! But I'm here to help you wherever you are." },
            { keywords: ["tell me a joke", "chutkula sunao"], answer: "Why don't scientists trust atoms? Because they make up everything!" },
            { keywords: ["goodbye", "bye", "khuda hafiz"], answer: "Khuda Hafiz! I hope to chat with you again soon!" },
            { keywords: ["how to spell", "spell word"], answer: "Please tell me the word you want me to spell." },
            { keywords: ["what is the weather like", "weather"], answer: "I'm an offline assistant and can't provide real-time weather information. Please check a weather app or website!" },
            { keywords: ["current events", "news"], answer: "As an offline AI, I don't have access to the latest news or current events. You can check online news sources for that." }

        ];

        let userName = "";
        let userAge = "";
        let isTyping = false;
        let conversationContext = []; // For current chat session's context
        const today = new Date();
        let currentLanguage = "en"; // 'en' for English, 'ur' for Urdu (Romanized)

        // New state variables for learning mode
        let learningMode = false;
        let questionToLearn = "";
        let originalUserQuery = ""; // To store the original query that triggered learning/Google search

        // Load user-added data from local storage
        let userQaData = JSON.parse(localStorage.getItem('nexaUserQaData')) || [];
        // Combine official qaData with user-added data for lookups
        let combinedQaData = [...qaData, ...userQaData];

        // Chat History Data Structure:
        // { id: 'unique_id', topic: 'Chat Topic (e.g., "AI Basics")', messages: [{text: "...", type: "...", timestamp: ...}], timestamp: 'ISO string' }
        let chatHistory = JSON.parse(localStorage.getItem('nexaChatHistory')) || [];
        let currentChatMessages = []; // Messages for the active chat session

        function getLocalizedText(key, replacements = {}) {
            const texts = {
                'en': {
                    'welcome_title': 'Welcome to Nexa AI!',
                    'enter_name': 'Enter your Name',
                    'enter_age': 'Enter your Age',
                    'start_chat': 'Start Chat',
                    'hello_user': 'Hello, {{name}}!',
                    'initial_greeting': 'Salam, {{name}}! I’m Nexa AI 🤖, your offline guru. Ask about famous personalities, computers, or anything else! How can I help you?',
                    'type_message': 'Type your message...',
                    'send_btn': 'Send',
                    'toggle_dark_mode': 'Toggle Dark Mode', // No emoji here, emoji is in HTML
                    'toggle_language': 'Language', // No emoji here, emoji is in HTML
                    'new_chat': 'New Chat',
                    'chat_history': 'Chat History',
                    'chat_history_header': 'Chat History',
                    'close_btn': '✖',
                    'no_answer_google_prompt': 'I don\'t know the answer to: "{{query}}". Would you like to search Google for it? Type **Yes (Google)/No**.',
                    'opening_google': 'Okay, {{name}}! Opening Google for your question: "{{query}}".',
                    'teach_or_skip_prompt': 'Alright, would you like to teach me the answer? If you know it, type the answer, otherwise type **\'skip\'**.',
                    'answer_learned': 'Thank you! I\'ve learned the answer to "{{question}}". This is saved **locally** for you. Remember, if all Nexa users need this, you can share it with the developer!',
                    'no_problem_teach_later': 'No problem, {{name}}. Let me know when you find out!',
                    'thinking': 'Thinking',
                    'cannot_divide_by_zero': 'Cannot divide by zero!',
                    'what_else_can_i_tell': 'What else can I tell you about this? Please ask a specific question.',
                    'thank_you_response': 'Shukriya, {{name}}! 😊 What’s next?',
                    'how_are_you_response': 'I am fine, thank you! And how are you? How can I help you today?',
                    'my_name_is': 'My name is Nexa AI, a friendly offline assistant. Nice to meet you!',
                    'who_made_me': 'I was created by a developer to assist you offline with various questions.',
                    'where_am_i_from': 'I exist in the digital realm, so I don\'t have a physical location! But I\'m here to help you wherever you are.',
                    'joke': 'Why don\'t scientists trust atoms? Because they make up everything!',
                    'goodbye_response': 'Khuda Hafiz! I hope to chat with you again soon!',
                    'spell_word_prompt': 'Please tell me the word you want me to spell.',
                    'no_weather_info': 'I\'m an offline assistant and can\'t provide real-time weather information. Please check a weather app or website!',
                    'no_current_events': 'As an offline AI, I don\'t have access to the latest news or current events. You can check online news sources for that.',
                    'nice_to_meet_you': 'Nice to meet you, {{name}}! 😎 What\'s your question now?',
                    'more_trends': 'Another trend is creating reaction videos to viral songs or prank challenges. Keep it short and use trending hashtags!',
                    'more_coding': 'You can also learn C++ or Java for advanced programming. Try building a simple game or app to practice.',
                    'more_ai': 'AI powers things like chatbots, image recognition, and even self-driving cars. Want details on any AI topic?',
                    'more_computer': 'Computers also rely on GPUs for graphics and SSDs for fast storage. Want to know about a specific component?',
                    'more_internet_network': 'The internet allows for services like email, social media, and online shopping. Networks can be local (LAN) or global (WAN).',
                    'chat_with_nexa_ai': 'Chat with Nexa AI', // New key for H1
                    'support_of_google': 'support of Google', // New key for H6
                    'please_enter_name_age': 'Please enter both your Name and Age.', // New key for alert

                },
                'ur': { // Romanized Urdu
                    'welcome_title': 'Nexa AI mein Khush Aamdeed!',
                    'enter_name': 'Apna Naam Darj Karein',
                    'enter_age': 'Apni Umar Darj Karein',
                    'start_chat': 'Chat Shuru Karein',
                    'hello_user': 'Assalam-o-Alaikum, {{name}}!',
                    'initial_greeting': 'Salam, {{name}}! Mai Nexa AI 🤖 hoon, aapka offline guru. Mashoor shakhsiyaton, computers, ya kisi aur cheez ke bare mein poochhein! Kaise madad kar sakta hoon?',
                    'type_message': 'Apna paigham likhen...',
                    'send_btn': 'Bhejein',
                    'toggle_dark_mode': 'Dark Mode On/Off',
                    'toggle_language': 'Zabaan Badlein',
                    'new_chat': 'Nayi Chat',
                    'chat_history': 'Chat History',
                    'chat_history_header': 'Chat Ki History',
                    'close_btn': '✖',
                    'no_answer_google_prompt': 'Mujhe is sawal ka jawab nahi pata: "{{query}}". Kya aap iske liye Google search karna chahenge? **Yes (Google)/No** likhen.',
                    'opening_google': 'Thheek hai, {{name}}! Aapke sawal "{{query}}" ke liye Google khol raha hoon.',
                    'teach_or_skip_prompt': 'Achha, toh kya aap mujhe iska jawab sikhana chahenge? Agar aapko pata hai toh jawab likhen, warna **\'skip\'** karein.',
                    'answer_learned': 'Shukriya! Maine "{{question}}" ka jawab seekh liya hai. Ye aapke liye **locally save** ho gaya hai. Yaad rakhiye, agar sabhi Nexa users ko iski zaroorat hai, to aap ise developer se share kar sakte hain!',
                    'no_problem_teach_later': 'Koi baat nahi, {{name}}. Jab aapko pata chale to mujhe zaroor sikhana!',
                    'thinking': 'Soch raha hoon',
                    'cannot_divide_by_zero': 'Zero se taqseem nahi kar sakte!',
                    'what_else_can_i_tell': 'Mai is bare mein aur kya bataun? Aap specific sawal poochein.',
                    'thank_you_response': 'Shukriya, {{name}}! 😊 Kya aage?',
                    'how_are_you_response': 'Mai theek hoon, shukriya! Aur aap kaise hain? How can I help you today?',
                    'my_name_is': 'Mera naam Nexa AI hai, ek dostana offline assistant. Aap se mil kar khushi hui!',
                    'who_made_me': 'Mujhe ek developer ne banaya hai taake main aapki offline madad kar sakoon.',
                    'where_am_i_from': 'Main digital duniya mein rehta hoon, isliye mera koi jism nahi! Lekin main aapki madad ke liye har jagah mojood hoon.',
                    'joke': 'Scientists atoms par yakeen kyun nahi karte? Kyunki woh har cheez banate hain!',
                    'goodbye_response': 'Khuda Hafiz! Umeed hai jald aapse phir baat hogi!',
                    'spell_word_prompt': 'Meherbani kar ke woh lafz bataein jo aap spell karwana chahte hain.',
                    'no_weather_info': 'Mai ek offline assistant hoon aur live mausam ki malumat nahi de sakta. Baraye meherbani mausam ki app ya website dekhen!',
                    'no_current_events': 'Ek offline AI hone ke नाते, mere paas taza tareen khabron ya mojooda waqiyat tak rasai nahi hai. Aap iske liye online news sources check kar sakte hain.',
                    'nice_to_meet_you': 'Aap se mil kar khushi hui, {{name}}! 😎 Ab aapka sawal kya hai?',
                    'more_trends': 'Ek aur trend viral gaano par reaction videos ya prank challenges banana hai. Ise mukhtasar rakhen aur trending hashtags ka istemal karein!',
                    'more_coding': 'Aap advanced programming ke liye C++ ya Java bhi seekh sakte hain. Practice ke liye ek simple game ya app banane ki koshish karein.',
                    'more_ai': 'AI chatbots, tasveer ki pehchan, aur yahan tak ke khud-chalane wali gariyon ko bhi power karta hai. Kya aap kisi AI topic par tafseelat chahte hain?',
                    'more_computer': 'Computers graphics ke liye GPUs aur tez storage ke liye SSDs par bhi munhasir hote hain. Kya aap kisi khaas component ke bare mein janna chahte hain?',
                    'more_internet_network': 'Internet email, social media, aur online shopping jaisi services ki ijazat deta hai. Networks local (LAN) ya global (WAN) ho sakte hain.',
                    'chat_with_nexa_ai': 'Nexa AI ke Saath Chat Karein', // New key for H1
                    'support_of_google': 'Google ke support ke saath', // New key for H6
                    'please_enter_name_age': 'Baraye meherbani apna Naam aur Umar dono darj karein.', // New key for alert
                }
            };
            let text = texts[currentLanguage][key];
            for (const placeholder in replacements) {
                text = text.replace(`{{${placeholder}}}`, replacements[placeholder]);
            }
            return text;
        }

        function addMessage(text, type, id = null) {
            const chatWindow = document.getElementById("chat-window");
            const msgContainer = document.createElement("div");

            if (type === "bot-message") {
                msgContainer.className = "bot-message-container";
                const avatar = document.createElement("img");
                avatar.src = "nexa-avatar.png"; 
                avatar.alt = "Nexa AI";
                avatar.className = "bot-avatar";
                msgContainer.appendChild(avatar);
                const msg = document.createElement("div");
                msg.className = "message bot-message";
                msg.innerHTML = text;
                if (id) msg.id = id;
                msgContainer.appendChild(msg);
                chatWindow.appendChild(msgContainer);
            } else if (type === "stylish-welcome") {
                const msg = document.createElement("div");
                msg.className = "stylish-welcome";
                msg.innerText = text;
                chatWindow.appendChild(msg);
            }
            else {
                const msg = document.createElement("div");
                msg.className = `message ${type}`; 
                msg.innerText = text;
                if (id) msg.id = id;
                chatWindow.appendChild(msg);
            }
            chatWindow.scrollTop = chatWindow.scrollHeight;
            
            // Add message to current chat session history
            currentChatMessages.push({ text: text, type: type, timestamp: new Date().toISOString() });
        }

        function showTypingAnimation() {
            if (isTyping) return;
            const chatWindow = document.getElementById("chat-window");
            const typingContainer = document.createElement("div");
            typingContainer.className = "bot-message-container";
            typingContainer.id = "typing";

            const avatar = document.createElement("img");
            avatar.src = "nexa-avatar.png"; 
            avatar.alt = "Nexa AI";
            avatar.className = "bot-avatar";
            typingContainer.appendChild(avatar);

            const typingDiv = document.createElement("div");
            typingDiv.className = "message bot-message typing-dots";
            typingDiv.innerHTML = `${getLocalizedText('thinking')}<span class='dot'>.</span><span class='dot'>.</span><span class='dot'>.</span>`; 
            typingContainer.appendChild(typingDiv);

            chatWindow.appendChild(typingContainer);
            chatWindow.scrollTop = chatWindow.scrollHeight;
            isTyping = true;
        }

        function removeTypingAnimation() {
            const typingDiv = document.getElementById("typing");
            if (typingDiv) typingDiv.remove();
            isTyping = false;
        }

        function correctSpelling(input) {
            return input
                .toLowerCase()
                .replace(/\b(?:vodeo|vdo)\b/g, "video")
                .replace(/\b(?:uplaod|uplod)\b/g, "upload")
                .replace(/\byt\b/g, "youtube")
                .replace(/\b(?:codding|codingg)\b/g, "coding")
                .replace(/\b(?:artifical|artifcl)\b/g, "artificial")
                .replace(/\b(?:pakisthan|pakstan)\b/g, "pakistan")
                .replace(/\b(?:indai|inda)\b/g, "india")
                .replace(/\b(?:elon|elone)\b/g, "elon musk")
                .replace(/\b(?:bil gates|billgate)\b/g, "bill gates")
                .replace(/\b(?:stev jobs|stevejob)\b/g, "steve jobs")
                .replace(/\b(?:mark zukerberg|mark zuck)\b/g, "mark zuckerberg")
                .replace(/\b(?:einsten|ainstein)\b/g, "einstein")
                .replace(/\b(?:gravty|graavity)\b/g, "gravity")
                .replace(/\b(?:komputer|compter)\b/g, "computer")
                .replace(/\b(?:paisa|peesa)\b/g, "paisa")
                .replace(/\b(?:kaise kamayen|kese kamay)\b/g, "kaise kamaye")
                .replace(/\b(?:os|oprating system)\b/g, "operating system")
                .replace(/\b(?:csss|scss)\b/g, "css")
                .replace(/\b(?:javascrip|js)\b/g, "javascript")
                .replace(/\b(?:fotosyntesis|fotosynthesis)\b/g, "photosynthesis")
                .replace(/\b(?:soler sistem|solar sys)\b/g, "solar system")
                .replace(/\b(?:netwotk|netwrk)\b/g, "network");
        }

        function levenshteinDistance(s1, s2) {
            s1 = s1.toLowerCase();
            s2 = s2.toLowerCase();
            const costs = [];
            for (let i = 0; i <= s1.length; i++) {
                let lastValue = i;
                for (let j = 0; j <= s2.length; j++) {
                    if (i === 0) {
                        costs[j] = j;
                    } else if (j > 0) {
                        let newValue = costs[j - 1];
                        if (s1.charAt(i - 1) !== s2.charAt(j - 1)) {
                            newValue = Math.min(Math.min(newValue, lastValue), costs[j]) + 1;
                        }
                        costs[j - 1] = lastValue;
                        lastValue = newValue;
                    }
                }
                if (i > 0) {
                    costs[s2.length] = lastValue;
                }
            }
            return costs[s2.length];
        }

        function fuzzyMatch(input, keyword) {
            input = input.toLowerCase().trim();
            keyword = keyword.toLowerCase().trim();

            if (input === keyword) return true;

            const keywordRegex = new RegExp(`\\b${keyword}\\b`);
            if (keywordRegex.test(input)) return true;
            
            if (keyword.length > 5 && input.includes(keyword)) return true;

            const inputWords = input.split(/\s+/).filter(word => word.length > 0);
            const keywordWords = keyword.split(/\s+/).filter(word => word.length > 0);

            let matchedWordsScore = 0;
            const totalKeywordWords = keywordWords.length;

            for (const kw of keywordWords) {
                let bestMatchDistance = Infinity;
                for (const iw of inputWords) {
                    const distance = levenshteinDistance(iw, kw);
                    bestMatchDistance = Math.min(bestMatchDistance, distance);
                }
                let tolerance = 0;
                if (kw.length <= 3) tolerance = 0; 
                else if (kw.length <= 5) tolerance = 1;
                else if (kw.length <= 8) tolerance = 2;
                else tolerance = 3;

                if (bestMatchDistance <= tolerance) {
                    matchedWordsScore++;
                }
            }
            if (totalKeywordWords > 0 && (matchedWordsScore / totalKeywordWords) >= 0.7) return true;

            const overallMaxLength = Math.max(input.length, keyword.length);
            let overallTolerance = 0;
            if (overallMaxLength <= 5) overallTolerance = 1;
            else if (overallMaxLength <= 10) overallTolerance = 2; 
            else if (overallMaxLength <= 15) overallTolerance = 3;
            else overallTolerance = 4;

            if (levenshteinDistance(input, keyword) <= overallTolerance) return true;

            return false; 
        }

        function handleMathQuery(input) {
            input = input.replace(/plus/g, "+").replace(/minus/g, "-").replace(/times/g, "*").replace(/divide/g, "/");
            const mathMatch = input.match(/(\d+)\s*([+\-*\/xX])\s*(\d+)/);
            if (mathMatch) {
                const num1 = parseInt(mathMatch[1]);
                let operator = mathMatch[2];
                const num2 = parseInt(mathMatch[3]);

                if (operator === 'x' || operator === 'X') operator = '*';

                switch (operator) {
                    case "+": return `${num1} + ${num2} = ${num1 + num2}`;
                    case "-": return `${num1} - ${num2} = ${num1 - num2}`;
                    case "*": return `${num1} × ${num2} = ${num1 * num2}`;
                    case "/": return num2 !== 0 ? `${num1} ÷ ${num2} = ${num1 / num2}` : getLocalizedText('cannot_divide_by_zero');
                    default: return null;
                }
            }
            return null;
        }

        function handleDateTimeQuery(input) {
            const timeOptions = { hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: true, timeZone: 'Asia/Karachi' };
            const dateOptions = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };

            // We need to use exact keywords for date/time for localization to work well.
            // These would typically be part of qaData, or handled by a more robust NLP if a single function is used.
            // For now, let's assume direct English keywords are primarily used for these checks.
            // If full localization is needed for these, we'd need more extensive entries in qaData.
            if (input.includes("time") || input.includes("samay") || input.includes("waqt")) {
                return `The current time is ${today.toLocaleTimeString('en-US', timeOptions)} (PKT).`;
            }
            if (input.includes("date") || input.includes("tarikh") || input.includes("today")) {
                return `Today is ${today.toLocaleDateString('en-US', dateOptions)}.`;
            }
            if (input.includes("day")) {
                return `Today is ${today.toLocaleDateString('en-US', { weekday: 'long' })}.`;
            }
            return null;
        }

        function getResponse(input) {
            input = correctSpelling(input);
            // Conversation context is now part of currentChatMessages
            
            const nameMatch = input.match(/(my name is|mera naam)\s+([\w\s]+)/i);
            if (nameMatch) {
                const detectedName = nameMatch[2].trim();
                userName = detectedName.charAt(0).toUpperCase() + detectedName.slice(1);
                localStorage.setItem("nexaAIName", userName);
                return getLocalizedText('nice_to_meet_you', { name: userName });
            }

            const mathResponse = handleMathQuery(input);
            if (mathResponse) return mathResponse;

            const dateTimeResponse = handleDateTimeQuery(input);
            if (dateTimeResponse) return dateTimeResponse;

            const cleanedInput = input.replace(/who is |what is |kon hai |kya hai /g, '').trim();

            const sortedQaData = [...combinedQaData].sort((a, b) => {
                const aLongestKeyword = Math.max(...a.keywords.map(k => k.length));
                const bLongestKeyword = Math.max(...b.keywords.map(k => k.length));
                return bLongestKeyword - aLongestKeyword; 
            });


            for (let item of sortedQaData) {
                for (let keyword of item.keywords) {
                    if (fuzzyMatch(input, keyword) || (cleanedInput && fuzzyMatch(cleanedInput, keyword))) {
                        let answer = item.answer;
                        if (userName) answer = answer.replace(/{{name}}/g, userName);
                        return answer;
                    }
                }
            }

            // --- Follow-up handling (only for specific follow-up phrases) ---
            // Get last *bot* message from currentChatMessages to infer context
            const lastBotMessage = currentChatMessages.slice().reverse().find(msg => msg.type === 'bot-message');
            const lastBotText = lastBotMessage ? lastBotMessage.text.toLowerCase() : "";
            
            if (input.includes("more") || input.includes("aur batao") || input.includes("tell me more")) {
                if (lastBotText.includes("trends")) {
                    return getLocalizedText('more_trends');
                }
                if (lastBotText.includes("coding") || lastBotText.includes("programming")) {
                    return getLocalizedText('more_coding');
                }
                if (lastBotText.includes("ai")) {
                    return getLocalizedText('more_ai');
                }
                if (lastBotText.includes("computer")) {
                    return getLocalizedText('more_computer');
                }
                if (lastBotText.includes("internet") || lastBotText.includes("network")) {
                    return getLocalizedText('more_internet_network');
                }
                return getLocalizedText('what_else_can_i_tell');
            }
            
            return "SEARCH_GOOGLE_PLEASE";
        }

        function handleNormalQuery(userText) {
            let botResponse = getResponse(userText);
            const responseLength = botResponse.length;
            const typingDuration = Math.min(2000, Math.max(500, responseLength * 15)); 

            setTimeout(() => {
                removeTypingAnimation();
                if (botResponse === "SEARCH_GOOGLE_PLEASE") {
                    originalUserQuery = userText;
                    addMessage(getLocalizedText('no_answer_google_prompt', { query: userText }), "bot-message");
                    learningMode = true;
                    questionToLearn = "";
                } else {
                    addMessage(botResponse, "bot-message");
                }
            }, typingDuration); 
        }

        function sendMessage() {
            const inputBox = document.getElementById("user-input");
            const userText = inputBox.value.trim();
            if (!userText) return;

            addMessage(userText, "user-message");
            inputBox.value = "";
            showTypingAnimation();

            if (learningMode) {
                const lowerCaseUserText = userText.toLowerCase();

                if (!questionToLearn && (lowerCaseUserText === "yes" || lowerCaseUserText === "google" || lowerCaseUserText === "haan" || lowerCaseUserText === "han")) {
                    removeTypingAnimation(); 
                    addMessage(getLocalizedText('opening_google', { name: userName || 'dost', query: originalUserQuery }), "bot-message");
                    setTimeout(() => { 
                        window.open("https://www.google.com/search?q=" + encodeURIComponent(originalUserQuery), "_blank");
                        learningMode = false;
                        questionToLearn = "";
                        originalUserQuery = "";
                    }, 500); 
                } 
                else if (!questionToLearn && (lowerCaseUserText === "no" || lowerCaseUserText === "nahi")) {
                    questionToLearn = originalUserQuery;
                    removeTypingAnimation();
                    addMessage(getLocalizedText('teach_or_skip_prompt'), "bot-message");
                } 
                else if (questionToLearn) {
                    if (lowerCaseUserText === "skip") {
                        removeTypingAnimation();
                        addMessage(getLocalizedText('no_problem_teach_later', { name: userName || 'dost' }), "bot-message");
                        learningMode = false;
                        questionToLearn = "";
                        originalUserQuery = "";
                    } else {
                        const answerToLearn = userText;
                        userQaData.push({ keywords: [questionToLearn.toLowerCase().trim()], 
                                        answer: answerToLearn });
                        localStorage.setItem('nexaUserQaData', JSON.stringify(userQaData));
                        combinedQaData = [...qaData, ...userQaData];
                        removeTypingAnimation(); 
                        addMessage(getLocalizedText('answer_learned', { question: questionToLearn }), "bot-message");
                        learningMode = false;
                        questionToLearn = "";
                        originalUserQuery = "";
                    }
                } 
                else {
                    learningMode = false;
                    questionToLearn = "";
                    originalUserQuery = "";
                    handleNormalQuery(userText); 
                }
                return;
            }
            handleNormalQuery(userText);
        }

        // --- Initial Setup and Welcome Screen ---
        function startChat() {
            userName = document.getElementById("welcomeNameInput").value.trim();
            userAge = document.getElementById("welcomeAgeInput").value.trim();

            if (!userName || !userAge) {
                alert(getLocalizedText('please_enter_name_age')); // Localized alert message
                return;
            }

            localStorage.setItem("nexaAIName", userName);
            localStorage.setItem("nexaAIAge", userAge);

            // Hide welcome screen with transition
            document.getElementById("welcomeScreen").style.opacity = "0";
            setTimeout(() => {
                document.getElementById("welcomeScreen").style.display = "none";
                // Show main chat container
                document.getElementById("mainChatContainer").style.display = "flex";
                initialGreeting(); // Call initial greeting AFTER the container is visible
            }, 300); // Wait for opacity transition to finish
        }

        function initialGreeting() {
            document.getElementById("chat-window").innerHTML = ""; // Clear existing messages
            addMessage(getLocalizedText('hello_user', { name: userName }), "stylish-welcome");
            setTimeout(() => {
                addMessage(getLocalizedText('initial_greeting', { name: userName }), "bot-message");
            }, 1000);
            currentChatMessages = []; // Start a new current chat session
        }

        // --- Chat History Management ---
        function generateUniqueId() {
            return Date.now().toString(36) + Math.random().toString(36).substring(2);
        }

        function saveCurrentChat() {
            if (currentChatMessages.length === 0) return;

            const firstUserQuery = currentChatMessages.find(msg => msg.type === 'user-message');
            const topic = firstUserQuery ? firstUserQuery.text.substring(0, 30) + '...' : 'Untitled Chat';
            const chatObj = {
                id: generateUniqueId(),
                topic: topic,
                messages: currentChatMessages,
                timestamp: new Date().toISOString()
            };
            chatHistory.unshift(chatObj); // Add to the beginning
            localStorage.setItem('nexaChatHistory', JSON.stringify(chatHistory));
        }

        function newChat() {
            if (currentChatMessages.length > 0) {
                saveCurrentChat(); // Save the current chat before starting a new one
            }
            document.getElementById("chat-window").innerHTML = "";
            currentChatMessages = []; // Reset for new chat
            initialGreeting(); // Greet again for the new chat
        }

        function loadChat(chatId) {
            const chatToLoad = chatHistory.find(chat => chat.id === chatId);
            if (chatToLoad) {
                document.getElementById("chat-window").innerHTML = "";
                chatToLoad.messages.forEach(msg => {
                    const msgDiv = document.createElement("div");
                    // Important: For loaded messages, if they were bot messages, recreate the container with avatar
                    if (msg.type === "bot-message") {
                        const botContainer = document.createElement("div");
                        botContainer.className = "bot-message-container";
                        const avatar = document.createElement("img");
                        avatar.src = "nexa-avatar.png"; 
                        avatar.alt = "Nexa AI";
                        avatar.className = "bot-avatar";
                        botContainer.appendChild(avatar);
                        
                        const botMsg = document.createElement("div");
                        botMsg.className = "message bot-message";
                        botMsg.innerHTML = msg.text; // Use innerHTML for dots etc.
                        botContainer.appendChild(botMsg);
                        document.getElementById("chat-window").appendChild(botContainer);
                    } else if (msg.type === "stylish-welcome") {
                         const stylishMsgDiv = document.createElement("div");
                         stylishMsgDiv.className = "stylish-welcome";
                         stylishMsgDiv.innerText = msg.text;
                         document.getElementById("chat-window").appendChild(stylishMsgDiv);
                    }
                    else {
                        msgDiv.className = `message ${msg.type}`;
                        msgDiv.innerText = msg.text;
                        document.getElementById("chat-window").appendChild(msgDiv);
                    }
                });
                currentChatMessages = [...chatToLoad.messages]; // Set current messages to loaded chat
                document.getElementById("chat-window").scrollTop = document.getElementById("chat-window").scrollHeight;
                toggleHistorySidebar(); // Close sidebar after loading
            }
        }

        function deleteChat(chatId, event) {
            event.stopPropagation(); // Prevent loading the chat when deleting
            chatHistory = chatHistory.filter(chat => chat.id !== chatId);
            localStorage.setItem('nexaChatHistory', JSON.stringify(chatHistory));
            renderHistoryList(); // Re-render the list
        }

        function renderHistoryList() {
            const historyList = document.getElementById("historyList");
            historyList.innerHTML = "";
            if (chatHistory.length === 0) {
                historyList.innerHTML = "<p style='text-align: center; color: var(--secondary-text); margin-top: 20px;'>No chat history yet.</p>";
                return;
            }
            chatHistory.forEach(chat => {
                const item = document.createElement("div");
                item.className = "history-item";
                item.onclick = () => loadChat(chat.id);
                
                const title = document.createElement("span");
                title.innerText = chat.topic;
                item.appendChild(title);

                const deleteBtn = document.createElement("button");
                deleteBtn.className = "delete-history-btn";
                deleteBtn.innerText = "✖";
                deleteBtn.onclick = (e) => deleteChat(chat.id, e);
                item.appendChild(deleteBtn);

                historyList.appendChild(item);
            });
        }

        function toggleHistorySidebar() {
            const sidebar = document.getElementById("chatHistorySidebar");
            sidebar.classList.toggle("open");
            renderHistoryList(); // Refresh history list every time it opens
            // Close settings menu if open
            document.getElementById("settingsMenu").classList.remove("settings-menu-open");
        }

        // --- Theme and Language Toggles ---
        function toggleSettingsMenu() {
            const settingsMenu = document.getElementById("settingsMenu");
            settingsMenu.classList.toggle("settings-menu-open");
            // Close chat history sidebar if open
            document.getElementById("chatHistorySidebar").classList.remove("open");
        }

        function toggleTheme() {
            document.body.classList.toggle("dark-mode");
            localStorage.setItem("theme", document.body.classList.contains("dark-mode") ? "dark" : "light");
            updateUILanguage(); // Update text for theme toggle button
        }

        function toggleLanguage() {
            currentLanguage = currentLanguage === "en" ? "ur" : "en";
            localStorage.setItem("language", currentLanguage);
            updateUILanguage();
            
            // Re-display current chat messages in new language if possible
            const chatWindow = document.getElementById("chat-window");
            const tempMessages = [...currentChatMessages]; // Copy messages
            chatWindow.innerHTML = ""; // Clear display
            currentChatMessages = []; // Reset current messages array to prevent duplicates when re-adding

            tempMessages.forEach(msg => {
                // For 'bot-message' and 'stylish-welcome', try to re-localize if they are known phrases.
                // For user messages and learned answers, display as is.
                let reLocalizedText = msg.text;
                if (msg.type === "bot-message" || msg.type === "stylish-welcome") {
                    // This part needs to be more robust if *all* bot answers from qaData should re-localize.
                    // For now, it handles the fixed greetings and system messages.
                    if (msg.text.includes("Salam,") || msg.text.includes("Hello,")) {
                        reLocalizedText = getLocalizedText('initial_greeting', { name: userName });
                    } else if (msg.text.includes("Nice to meet you,")) {
                        reLocalizedText = getLocalizedText('nice_to_meet_you', { name: userName });
                    } else if (msg.text.includes("I don't know the answer to:")) { // Example for unknown answer prompt
                         reLocalizedText = getLocalizedText('no_answer_google_prompt', { query: originalUserQuery || '' });
                    } else if (msg.text.includes("Okay,") && msg.text.includes("Opening Google")) { // Example for opening Google
                         reLocalizedText = getLocalizedText('opening_google', { name: userName || 'dost', query: originalUserQuery || '' });
                    } else if (msg.text.includes("Alright, would you like to teach me the answer?")) { // Example for teach prompt
                         reLocalizedText = getLocalizedText('teach_or_skip_prompt');
                    } else if (msg.text.includes("Thank you! I've learned the answer to")) { // Example for answer learned
                         reLocalizedText = getLocalizedText('answer_learned', { question: questionToLearn || '' });
                    } else if (msg.text.includes("No problem,")) { // Example for no problem
                         reLocalizedText = getLocalizedText('no_problem_teach_later', { name: userName || 'dost' });
                    } else if (msg.text.includes("Thinking")) { // Typing dots
                         reLocalizedText = getLocalizedText('thinking');
                    }
                    // Add more conditions here for other localized bot responses from `getLocalizedText`
                }
                addMessage(reLocalizedText, msg.type); // Add with potentially new localized text
            });
            chatWindow.scrollTop = chatWindow.scrollHeight; // Scroll to bottom
        }

        function updateUILanguage() {
            document.getElementById('mainChatH1').innerHTML = `💬 ${getLocalizedText('chat_with_nexa_ai')}`;
            document.getElementById('mainChatH6').innerHTML = `+ ${getLocalizedText('support_of_google')}`; 
            document.getElementById('user-input').placeholder = getLocalizedText('type_message');
            document.getElementById('sendMessageBtn').innerText = getLocalizedText('send_btn');
            
            // Update settings menu buttons
            document.getElementById('darkModeToggleText').innerText = getLocalizedText('toggle_dark_mode');
            document.getElementById('languageToggleText').innerText = getLocalizedText('toggle_language');

            // Update other fixed action buttons
            document.querySelector('.new-chat-btn').innerHTML = `➕ ${getLocalizedText('new_chat')}`;
            document.querySelector('.chat-history-btn').innerHTML = `📚 ${getLocalizedText('chat_history')}`;
            
            document.getElementById('chatHistoryHeader').innerText = getLocalizedText('chat_history_header');

            // Update placeholders and titles on welcome screen if visible
            if (document.getElementById("welcomeScreen").style.display !== "none") {
                document.getElementById('welcomeNameInput').placeholder = getLocalizedText('enter_name');
                document.getElementById('welcomeAgeInput').placeholder = getLocalizedText('enter_age');
                document.getElementById('welcomeScreenTitle').innerText = getLocalizedText('welcome_title');
                document.getElementById('startChatBtn').innerText = getLocalizedText('start_chat');
            }
        }


        // --- Initialize ---
        window.onload = () => {
            // Load theme
            if (localStorage.getItem("theme") === "dark") {
                document.body.classList.add("dark-mode");
            }

            // Load language
            if (localStorage.getItem("language")) {
                currentLanguage = localStorage.getItem("language");
            }
            updateUILanguage(); // Apply initial language setting to UI elements

            // Check for existing user info
            userName = localStorage.getItem("nexaAIName");
            userAge = localStorage.getItem("nexaAIAge");

            if (userName && userAge) {
                document.getElementById("welcomeScreen").style.display = "none";
                document.getElementById("mainChatContainer").style.display = "flex";
                initialGreeting();
            } else {
                document.getElementById("welcomeScreen").style.display = "flex";
                document.getElementById("mainChatContainer").style.display = "none";
            }

            document.getElementById("user-input").addEventListener("keypress", e => {
                if (e.key === "Enter") sendMessage();
            });
        };
    </script>
</body>
</html>

