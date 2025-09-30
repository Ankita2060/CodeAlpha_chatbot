# CodeAlpha_chatbot

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Professional FAQ Chatbot</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Smooth animations */
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .message-enter {
            animation: fadeIn 0.3s ease-out;
        }
        
        .typing-indicator span {
            display: inline-block;
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background-color: #4b5563;
            animation: bounce 1.4s infinite;
        }
        
        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-3px); }
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen flex items-center justify-center p-4">
    <!-- Chatbot Container -->
    <div class="w-full max-w-md bg-white rounded-xl shadow-xl overflow-hidden flex flex-col h-[80vh]">
        <!-- Header -->
        <div class="bg-blue-600 text-white p-4 flex items-center">
            <div class="w-10 h-10 rounded-full bg-blue-500 flex items-center justify-center text-xl mr-3">🤖</div>
            <div>
                <h2 class="font-semibold">Professional FAQ Assistant</h2>
                <p class="text-xs opacity-80">Online and ready to help</p>
            </div>
        </div>
        
        <!-- Messages Area -->
        <div id="chat-messages" class="flex-1 p-4 overflow-y-auto space-y-3">
            <!-- Initial greeting -->
            <div class="flex items-start">
                <div class="bg-blue-100 rounded-xl p-3 max-w-[80%]">
                    <p>Hello! I'm your professional FAQ assistant. Please ask me any questions you have about our services or products.</p>
                </div>
            </div>
        </div>
        
        <!-- Input Area -->
        <div class="border-t p-4">
            <!-- Suggested questions -->
            <div id="suggested-questions" class="flex flex-wrap gap-2 mb-3">
                <button class="suggested-question bg-gray-100 hover:bg-gray-200 text-gray-800 text-sm px-3 py-1 rounded-full transition">
                    What is your pricing?
                </button>
                <button class="suggested-question bg-gray-100 hover:bg-gray-200 text-gray-800 text-sm px-3 py-1 rounded-full transition">
                    How do I contact support?
                </button>
                <button class="suggested-question bg-gray-100 hover:bg-gray-200 text-gray-800 text-sm px-3 py-1 rounded-full transition">
                    Do you offer refunds?
                </button>
            </div>
            
            <!-- Input box -->
            <div class="flex">
                <input 
                    id="user-input" 
                    type="text" 
                    placeholder="Type your question here..." 
                    class="flex-1 border border-gray-300 rounded-l-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                >
                <button 
                    id="send-btn" 
                    class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-r-lg transition"
                >
                    Send
                </button>
            </div>
        </div>
    </div>

    <script>
        // Enhanced FAQ database
        const faqs = [
            {
                question: "What is your pricing?",
                answer: "We offer several pricing tiers to suit different needs:<br><br>" +
                        "• <strong>Basic</strong>: $9.99/month (essential features)<br>" +
                        "• <strong>Pro</strong>: $29.99/month (advanced features + analytics)<br>" +
                        "• <strong>Enterprise</strong>: Custom pricing (dedicated support + API access)",
                related: ["Do you offer discounts?", "Is there a free trial?"]
            },
            {
                question: "How do I contact support?",
                answer: "You can reach our support team through multiple channels:<br><br>" +
                        "<strong>Email</strong>: support@yourcompany.com<br>" +
                        "<strong>Phone</strong>: +1 (800) 123-4567 (8AM-8PM EST)<br>" +
                        "<strong>Live Chat</strong>: Available in your account dashboard",
                related: ["What are your support hours?", "Do you offer 24/7 support?"]
            },
            {
                question: "Do you offer refunds?",
                answer: "Yes, we have a <strong>30-day money-back guarantee</strong> for all annual plans. " +
                        "Monthly plans can be canceled anytime with no long-term commitment. " +
                        "To request a refund, please contact our billing department.",
                related: ["What's your cancellation policy?", "How long do refunds take?"]
            },
            {
                question: "Is there a free trial?",
                answer: "We offer a <strong>14-day free trial</strong> with full access to all Pro features. " +
                        "No credit card is required to start your trial. You can upgrade at any time during or after your trial period.",
                related: ["What features are included?", "What happens after the trial ends?"]
            },
            {
                question: "What payment methods do you accept?",
                answer: "We accept all major credit cards (Visa, Mastercard, American Express), PayPal, " +
                        "and bank transfers for enterprise customers. All payments are securely processed through PCI-compliant systems.",
                related: ["Is my payment information secure?", "Do you store credit card details?"]
            }
        ];

        // Chat functionality
        document.addEventListener('DOMContentLoaded', () => {
            const chatMessages = document.getElementById('chat-messages');
            const userInput = document.getElementById('user-input');
            const sendBtn = document.getElementById('send-btn');
            const suggestedQuestions = document.querySelectorAll('.suggested-question');
            
            // Add message to chat
            function addMessage(text, sender = 'bot', isHTML = false) {
                const messageDiv = document.createElement('div');
                messageDiv.className = `flex items-start ${sender === 'user' ? 'justify-end' : ''}`;
                
                const bubbleDiv = document.createElement('div');
                bubbleDiv.className = `rounded-xl p-3 max-w-[80%] message-enter ${
                    sender === 'user' ? 'bg-blue-600 text-white' : 'bg-blue-100'
                }`;
                
                if (isHTML) {
                    bubbleDiv.innerHTML = text;
                } else {
                    bubbleDiv.textContent = text;
                }
                
                // Add timestamp
                const timestamp = document.createElement('div');
                timestamp.className = 'text-xs opacity-60 mt-1';
                timestamp.textContent = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
                
                messageDiv.appendChild(bubbleDiv);
                bubbleDiv.appendChild(timestamp);
                chatMessages.appendChild(messageDiv);
                
                // Scroll to bottom
                chatMessages.scrollTop = chatMessages.scrollHeight;
            }
            
            // Show typing indicator
            function showTypingIndicator() {
                const typingDiv = document.createElement('div');
                typingDiv.className = 'flex items-start';
                typingDiv.id = 'typing-indicator';
                
                const bubbleDiv = document.createElement('div');
                bubbleDiv.className = 'bg-blue-100 rounded-xl p-3 max-w-[80%] flex space-x-1';
                bubbleDiv.innerHTML = '<span></span><span></span><span></span>';
                
                typingDiv.appendChild(bubbleDiv);
                chatMessages.appendChild(typingDiv);
                chatMessages.scrollTop = chatMessages.scrollHeight;
            }
            
            // Hide typing indicator
            function hideTypingIndicator() {
                const typingDiv = document.getElementById('typing-indicator');
                if (typingDiv) typingDiv.remove();
            }
            
            // Find best matching FAQ
            function findBestMatch(question) {
                const normalizedQuestion = question.toLowerCase().replace(/[^\w\s]/g, '');
                let bestMatch = null;
                let highestScore = 0;
                
                for (const faq of faqs) {
                    const normalizedFAQ = faq.question.toLowerCase().replace(/[^\w\s]/g, '');
                    const words = new Set([...normalizedQuestion.split(' '), ...normalizedFAQ.split(' ')]);
                    
                    // Simple similarity score
                    let score = 0;
                    for (const word of normalizedQuestion.split(' ')) {
                        if (normalizedFAQ.includes(word)) {
                            score += word.length; // Longer matches get more weight
                        }
                    }
                    
                    score = score / normalizedFAQ.length; // Normalize by length
                    
                    if (score > highestScore) {
                        highestScore = score;
                        bestMatch = faq;
                    }
                }
                
                // Only return matches with significant similarity
                return highestScore > 0.5 ? bestMatch : null;
            }
            
            // Process user question
            function processUserQuestion(question) {
                showTypingIndicator();
                
                setTimeout(() => {
                    hideTypingIndicator();
                    
                    const bestMatch = findBestMatch(question);
                    
                    if (bestMatch) {
                        let response = bestMatch.answer;
                        
                        // Add related questions if available
                        if (bestMatch.related && bestMatch.related.length > 0) {
                            response += '<br><br><div class="text-sm text-blue-600">Related questions:<div class="flex flex-wrap gap-1 mt-1">';
                            bestMatch.related.forEach(q => {
                                response += `<span class="inline-block bg-blue-50 px-2 py-1 rounded-md">${q}</span>`;
                            });
                            response += '</div></div>';
                        }
                        
                        addMessage(response, 'bot', true);
                    } else {
                        addMessage("I couldn't find an exact answer to your question. Please try rephrasing or ask about a different topic.", 'bot');
                    }
                }, 1000 + Math.random() * 2000); // Simulate processing time
            }
            
            // Event listeners
            sendBtn.addEventListener('click', () => {
                const question = userInput.value.trim();
                if (question) {
                    addMessage(question, 'user');
                    userInput.value = '';
                    processUserQuestion(question);
                }
            });
            
            userInput.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') {
                    const question = userInput.value.trim();
                    if (question) {
                        addMessage(question, 'user');
                        userInput.value = '';
                        processUserQuestion(question);
                    }
                }
            });
            
            // Suggested questions
            suggestedQuestions.forEach(button => {
                button.addEventListener('click', () => {
                    const question = button.textContent.trim();
                    addMessage(question, 'user');
                    processUserQuestion(question);
                });
            });
        });
    </script>
</body>
</html>
