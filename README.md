<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Simple Chatroom</title>
    <style>
        /* General Body Styling */
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #f0f2f5; /* Light grey background */
            color: #333;
        }

        /* Chat Container */
        #chat-container {
            background-color: #ffffff;
            border-radius: 10px;
            box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1); /* Soft shadow */
            width: 90%;
            max-width: 600px; /* Max width for larger screens */
            display: flex;
            flex-direction: column;
            overflow: hidden; /* Ensures rounded corners are visible */
            height: 80vh; /* Takes 80% of viewport height */
        }

        /* Chat Header */
        #chat-header {
            background-color: #007bff; /* Blue header */
            color: white;
            padding: 15px 20px;
            font-size: 1.3em;
            font-weight: bold;
            text-align: center;
            border-top-left-radius: 10px;
            border-top-right-radius: 10px;
        }

        /* Messages Display Area */
        #messages {
            flex-grow: 1; /* Takes up available space */
            padding: 15px;
            overflow-y: auto; /* Scroll if content overflows */
            background-color: #e9ebee; /* Very light grey background for messages */
            border-bottom: 1px solid #ddd;
            display: flex;
            flex-direction: column; /* Stacks messages vertically */
        }

        /* Individual Message Row */
        .message-row {
            display: flex;
            margin-bottom: 10px;
        }

        /* Message Bubble Styling */
        .message-bubble {
            padding: 8px 12px;
            border-radius: 18px; /* Rounded corners for bubbles */
            max-width: 75%; /* Messages don't span full width */
            word-wrap: break-word; /* Prevents long words from breaking layout */
            line-height: 1.4;
            display: inline-block; /* Helps with content wrapping */
        }

        /* Styling for other users' messages */
        .message-other .message-bubble {
            background-color: #f0f0f0; /* Light grey for others */
            margin-right: auto; /* Aligns to left */
            border-bottom-left-radius: 2px; /* Slightly less rounded at bottom-left */
            text-align: left;
        }

        /* Styling for current user's messages */
        .message-you .message-bubble {
            background-color: #007bff; /* Blue for self */
            color: white;
            margin-left: auto; /* Aligns to right */
            border-bottom-right-radius: 2px; /* Slightly less rounded at bottom-right */
            text-align: right;
        }
        .message-you {
            justify-content: flex-end; /* Aligns sender's message container to the right */
        }

        /* Username Styling */
        .message-other .username, .message-you .username {
            font-size: 0.8em;
            font-weight: bold;
            margin-bottom: 3px;
            display: block; /* Ensures username is on its own line */
        }
        .message-you .username { color: rgba(255, 255, 255, 0.7); } /* Lighter username for self */
        .message-other .username { color: #555; } /* Darker username for others */

        /* System Messages (e.g., connect/disconnect) */
        .system-message {
            text-align: center;
            font-style: italic;
            color: #777;
            margin-bottom: 10px;
            font-size: 0.9em;
        }

        /* Input Area */
        #input-area {
            display: flex;
            padding: 15px;
            border-top: 1px solid #eee;
            background-color: #fff;
            align-items: center; /* Aligns items vertically in the middle */
        }

        /* Username Input Field */
        #username-input {
            width: 100px; /* Fixed width for username */
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 20px;
            font-size: 1em;
            margin-right: 10px;
            outline: none; /* Removes outline on focus */
            transition: border-color 0.3s; /* Smooth transition for border color */
        }

        /* Message Input Field */
        #message-input {
            flex-grow: 1; /* Takes up remaining space */
            padding: 12px 15px;
            border: 1px solid #ccc;
            border-radius: 20px;
            font-size: 1em;
            margin-right: 10px;
            outline: none;
            transition: border-color 0.3s;
        }

        /* Focus State for Inputs */
        #message-input:focus, #username-input:focus {
            border-color: #007bff; /* Blue border on focus */
        }

        /* Send Button */
        #send-button {
            padding: 10px 20px;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 20px;
            cursor: pointer;
            font-size: 1em;
            transition: background-color 0.3s;
        }
        #send-button:hover {
            background-color: #0056b3; /* Darker blue on hover */
        }

        /* Responsive Adjustments for smaller screens */
        @media (max-width: 768px) {
            #chat-container {
                width: 100%;
                height: 100vh; /* Full screen on mobile */
                border-radius: 0; /* No rounded corners on full screen */
            }
            body {
                margin: 0;
            }
            #input-area {
                flex-wrap: wrap; /* Allows input fields to wrap to new line */
            }
            #username-input, #message-input {
                margin-bottom: 10px; /* Space between wrapped items */
                width: 100%; /* Full width when wrapped */
            }
            #send-button {
                width: 100%; /* Full width when wrapped */
            }
        }
    </style>
</head>
<body>
    <div id="chat-container">
        <div id="chat-header">My Simple Chatroom</div>
        <div id="messages"></div>
        <div id="input-area">
            <input type="text" id="username-input" placeholder="आपका नाम" value="Anonymous">
            <input type="text" id="message-input" placeholder="अपना संदेश टाइप करें..." autofocus>
            <button id="send-button">भेजें</button>
        </div>
    </div>

    <script>
        const chatMessages = document.getElementById('messages');
        const messageInput = document.getElementById('message-input');
        const usernameInput = document.getElementById('username-input');
        const sendButton = document.getElementById('send-button');

        // Connect to the WebSocket server
        // IMPORTANT:
        // If you are accessing this from a DIFFERENT device (like your phone) on the SAME local network,
        // you MUST replace 'localhost' with your computer's LOCAL IP ADDRESS
        // (e.g., 'ws://192.168.1.5:8765').
        // If you want it live on the internet, you'd need a public IP/domain and hosting.
        const socket = new WebSocket('ws://localhost:8765');

        function appendMessage(type, username, message) {
            const messageRow = document.createElement('div');
            messageRow.className = `message-row ${type === 'you' ? 'message-you' : 'message-other'}`;
            
            const messageBubble = document.createElement('div');
            messageBubble.className = 'message-bubble';

            const usernameSpan = document.createElement('span');
            usernameSpan.className = 'username';
            usernameSpan.textContent = username;

            const messageText = document.createElement('span');
            messageText.textContent = message;

            messageBubble.appendChild(usernameSpan);
            messageBubble.appendChild(messageText);
            messageRow.appendChild(messageBubble);

            chatMessages.appendChild(messageRow);
            chatMessages.scrollTop = chatMessages.scrollHeight; // Auto-scroll to bottom
        }

        function appendSystemMessage(message) {
            const systemDiv = document.createElement('div');
            systemDiv.className = 'system-message';
            systemDiv.innerHTML = `<em>${message}</em>`; // Italic text for system messages
            chatMessages.appendChild(systemDiv);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }

        // WebSocket Event Handlers
        socket.onopen = function(event) {
            console.log('Connected to WebSocket server');
            appendSystemMessage('चैट से कनेक्ट हो गए!');
        };

        socket.onmessage = function(event) {
            const data = JSON.parse(event.data);
            if (data.type === 'chat_message') {
                appendMessage('other', data.username, data.message);
            } else if (data.type === 'your_message') {
                appendMessage('you', data.username, data.message);
            }
        };

        socket.onclose = function(event) {
            console.log('Disconnected from WebSocket server', event.reason);
            appendSystemMessage('चैट से डिस्कनेक्ट हो गए.');
        };

        socket.onerror = function(error) {
            console.error('WebSocket error:', error);
            appendSystemMessage('चैट से कनेक्ट होने में त्रुटि.');
        };

        // Function to send messages
        function sendMessage() {
            const message = messageInput.value.trim();
            const username = usernameInput.value.trim() || 'Anonymous'; // Default to Anonymous if no name is entered

            if (message !== '') {
                const chatData = {
                    username: username,
                    message: message
                };
                socket.send(JSON.stringify(chatData)); // Send message as JSON string
                messageInput.value = ''; // Clear input field after sending
            }
        }

        // Event Listeners for sending messages
        sendButton.onclick = sendMessage; // Send on button click
        messageInput.addEventListener('keypress', function(e) {
            if (e.key === 'Enter') {
                sendMessage(); // Send on Enter key press
            }
        });
    </script>
</body>
</html>
