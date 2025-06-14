<script lang="ts">
    import { onMount, afterUpdate } from 'svelte';
    import { fade, fly } from 'svelte/transition';
    import { PUBLIC_BACKEND_URL } from '$env/static/public';
    import SvelteMarkdown from 'svelte-markdown';

    let messages: { text: string; isUser: boolean }[] = [];
    let inputMessage = '';
    let chatContainer: HTMLElement;

    async function sendMessage() {
        if (!inputMessage.trim()) return;

        // Add user message
        messages = [...messages, { text: inputMessage, isUser: true }];
        const userMessage = inputMessage;
        inputMessage = '';

        try {
            // Send message to bot and get response
            const response = await fetch(PUBLIC_BACKEND_URL + '/mortgage/invoke', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ 
                        input: {
                            query: userMessage 
                        }
                    })
            });
            const data = await response.json();

            // Add bot response
            messages = [...messages, { text: data.output.response, isUser: false }];
        } catch (error) {
            console.error('Error fetching bot response:', error);
            messages = [...messages, { text: 'Sorry, I encountered an error.', isUser: false }];
        }
    }

    afterUpdate(() => {
        if (chatContainer) {
            chatContainer.scrollTop = chatContainer.scrollHeight;
        }
    });

    $: if (chatContainer) chatContainer.scrollTop = chatContainer.scrollHeight;
</script>

<main>
    <h1>Chat with Bot</h1>
    <div class="chat-container" bind:this={chatContainer}>
        {#each messages as message, i (i)}
            <div 
                class={`message ${message.isUser ? 'user-message' : 'bot-message'}`}
                transition:fly="{{ y: 20, duration: 300 }}"
            >
                <div class="message-content">
                    <SvelteMarkdown source={message.text} />
                </div>
            </div>
        {/each}
    </div>
    <form on:submit|preventDefault={sendMessage}>
        <input type="text" bind:value={inputMessage} placeholder="Type your message...">
        <button type="submit">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
                <path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/>
            </svg>
        </button>
    </form>
</main>

<style>
    :root {
        --primary-color: #4a90e2;
        --background-color: #f5f7fa;
        --user-message-color: #e3effd;
        --bot-message-color: #ffffff;
        --text-color: #333333;
    }

    main {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
        font-family: 'Arial', sans-serif;
        background-color: var(--background-color);
        border-radius: 10px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    }

    h1 {
        text-align: center;
        color: var(--primary-color);
    }

    .chat-container {
        height: 500px;
        overflow-y: auto;
        padding: 20px;
        margin-bottom: 20px;
        background-color: #ffffff;
        border-radius: 10px;
        box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    .message {
        margin-bottom: 15px;
        display: flex;
    }

    .message-content {
        max-width: 80%;
        padding: 10px 15px;
        border-radius: 18px;
        box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
        line-height: 1.4;
    }

    .user-message {
        justify-content: flex-end;
    }

    .user-message .message-content {
        background-color: var(--user-message-color);
        color: var(--text-color);
        border-bottom-right-radius: 4px;
    }

    .bot-message .message-content {
        background-color: var(--bot-message-color);
        color: var(--text-color);
        border-bottom-left-radius: 4px;
    }

    form {
        display: flex;
        background-color: #ffffff;
        border-radius: 25px;
        padding: 5px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    input {
        flex-grow: 1;
        border: none;
        padding: 10px 15px;
        font-size: 16px;
        border-radius: 20px;
        outline: none;
    }

    button {
        background-color: var(--primary-color);
        color: white;
        border: none;
        border-radius: 50%;
        width: 40px;
        height: 40px;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        transition: background-color 0.3s ease;
    }

    button:hover {
        background-color: #3a7bc8;
    }

    /* Custom scrollbar styles */
    .chat-container::-webkit-scrollbar {
        width: 8px;
    }

    .chat-container::-webkit-scrollbar-track {
        background: #f1f1f1;
    }

    .chat-container::-webkit-scrollbar-thumb {
        background: #888;
        border-radius: 4px;
    }

    .chat-container::-webkit-scrollbar-thumb:hover {
        background: #555;
    }
</style>
