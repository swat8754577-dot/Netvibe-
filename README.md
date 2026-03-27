<!DOCTYPE html>
<html lang="ur">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Urdu/English AI Robot</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        body {
            background: radial-gradient(circle, #1a202c 0%, #000000 100%);
            color: white;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            overflow: hidden;
            height: 100vh;
        }

        /* Robot Animation */
        .robot-container {
            perspective: 1000px;
            height: 300px;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .robot {
            width: 150px;
            height: 150px;
            background: linear-gradient(145deg, #2d3748, #4a5568);
            border-radius: 50%;
            position: relative;
            box-shadow: 0 0 50px rgba(66, 153, 225, 0.5);
            animation: float 3s ease-in-out infinite;
        }

        .eye {
            width: 30px;
            height: 30px;
            background: #63b3ed;
            border-radius: 50%;
            position: absolute;
            top: 40px;
            box-shadow: 0 0 20px #63b3ed;
            animation: blink 4s infinite;
        }

        .eye.left { left: 35px; }
        .eye.right { right: 35px; }

        .mouth {
            width: 60px;
            height: 10px;
            background: #63b3ed;
            position: absolute;
            bottom: 40px;
            left: 45px;
            border-radius: 5px;
            transition: height 0.1s;
        }

        .talking .mouth {
            animation: talk 0.2s infinite alternate;
        }

        @keyframes float {
            0%, 100% { transform: translateY(0) rotateX(10deg); }
            50% { transform: translateY(-20px) rotateX(-10deg); }
        }

        @keyframes blink {
            0%, 90%, 100% { transform: scaleY(1); }
            95% { transform: scaleY(0.1); }
        }

        @keyframes talk {
            from { height: 5px; }
            to { height: 25px; }
        }

        .chat-box {
            background: rgba(45, 55, 72, 0.8);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        .typing-indicator {
            display: none;
            font-style: italic;
            color: #a0aec0;
        }
    </style>
</head>
<body class="flex flex-col items-center p-4">

    <!-- Robot Visual -->
    <div class="robot-container mt-10" id="robotViz">
        <div class="robot">
            <div class="eye left"></div>
            <div class="eye right"></div>
            <div class="mouth" id="mouth"></div>
        </div>
    </div>

    <div class="text-center mb-6">
        <h1 class="text-2xl font-bold text-blue-400">FUTURE AI ROBOT</h1>
        <p class="text-gray-400">Urdu aur English dono samajhta hoon</p>
    </div>

    <!-- Chat Interface -->
    <div class="chat-box w-full max-w-md flex flex-col h-[400px] overflow-hidden shadow-2xl">
        <div id="chatWindow" class="flex-1 overflow-y-auto p-4 space-y-4 text-sm">
            <div class="bg-blue-600 self-start p-3 rounded-tr-xl rounded-br-xl rounded-bl-xl max-w-[80%]">
                Assalam-o-Alaikum! Main aapka AI dost hoon. Aap mujhse Urdu ya English mein baat kar sakte hain.
            </div>
        </div>

        <div id="typing" class="px-4 py-1 typing-indicator text-xs">Robot soch raha hai...</div>

        <div class="p-4 bg-gray-800 flex gap-2">
            <input type="text" id="userInput" placeholder="Yahan likhein..." class="flex-1 bg-gray-700 border-none rounded-full px-4 py-2 focus:outline-none text-white">
            <button id="sendBtn" class="bg-blue-500 hover:bg-blue-600 p-3 rounded-full transition">
                <i class="fas fa-paper-plane"></i>
            </button>
        </div>
    </div>

    <script>
        const apiKey = ""; // Runtime provides key
        const chatWindow = document.getElementById('chatWindow');
        const userInput = document.getElementById('userInput');
        const sendBtn = document.getElementById('sendBtn');
        const mouth = document.getElementById('mouth');
        const robotViz = document.getElementById('robotViz');
        const typingIndicator = document.getElementById('typing');

        const systemPrompt = "You are a friendly human-like robot. Your name is Future Robot. You can speak Urdu and English. Respond in a natural, helpful, and friendly way. If the user speaks Urdu, reply in Urdu (Roman or Urdu script). Keep responses concise.";

        // Function to speak (Man Voice)
        async function speak(text) {
            try {
                robotViz.classList.add('talking');
                
                const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${apiKey}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: text }] }],
                        generationConfig: {
                            responseModalities: ["AUDIO"],
                            speechConfig: {
                                voiceConfig: {
                                    prebuiltVoiceConfig: {
                                        voiceName: "Enceladus" // Masculine/Deep voice
                                    }
                                }
                            }
                        }
                    })
                });

                const data = await response.json();
                const audioData = data.candidates[0].content.parts[0].inlineData.data;
                
                // Convert PCM to WAV and play
                const audioBlob = b64toBlob(audioData, 'audio/wav');
                const audioUrl = URL.createObjectURL(audioBlob);
                const audio = new Audio(audioUrl);
                
                audio.onended = () => {
                    robotViz.classList.remove('talking');
                };
                
                audio.play();
            } catch (error) {
                console.error("Speech Error:", error);
                robotViz.classList.remove('talking');
            }
        }

        // Helper to convert base64 to blob
        function b64toBlob(b64Data, contentType) {
            const byteCharacters = atob(b64Data);
            const byteArrays = [];
            for (let offset = 0; offset < byteCharacters.length; offset += 512) {
                const slice = byteCharacters.slice(offset, offset + 512);
                const byteNumbers = new Array(slice.length);
                for (let i = 0; i < slice.length; i++) {
                    byteNumbers[i] = slice.charCodeAt(i);
                }
                const byteArray = new Uint8Array(byteNumbers);
                byteArrays.push(byteArray);
            }
            return new Blob(byteArrays, { type: contentType });
        }

        async function getAIResponse(prompt) {
            typingIndicator.style.display = 'block';
            let retries = 5;
            let delay = 1000;

            for (let i = 0; i < retries; i++) {
                try {
                    const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({
                            contents: [{ parts: [{ text: prompt }] }],
                            systemInstruction: { parts: [{ text: systemPrompt }] }
                        })
                    });
                    const data = await response.json();
                    typingIndicator.style.display = 'none';
                    return data.candidates[0].content.parts[0].text;
                } catch (e) {
                    await new Promise(res => setTimeout(res, delay));
                    delay *= 2;
                }
            }
            typingIndicator.style.display = 'none';
            return "Maafi chahta hoon, network ka masla hai.";
        }

        function appendMessage(text, isUser) {
            const msgDiv = document.createElement('div');
            msgDiv.className = isUser ? 
                "bg-gray-700 self-end p-3 rounded-tl-xl rounded-bl-xl rounded-br-xl max-w-[80%] ml-auto" : 
                "bg-blue-600 self-start p-3 rounded-tr-xl rounded-br-xl rounded-bl-xl max-w-[80%]";
            msgDiv.innerText = text;
            chatWindow.appendChild(msgDiv);
            chatWindow.scrollTop = chatWindow.scrollHeight;
        }

        async function handleAction() {
            const text = userInput.value.trim();
            if (!text) return;

            appendMessage(text, true);
            userInput.value = '';

            const aiText = await getAIResponse(text);
            appendMessage(aiText, false);
            speak(aiText);
        }

        sendBtn.addEventListener('click', handleAction);
        userInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') handleAction();
        });

    </script>
</body>
</html>


