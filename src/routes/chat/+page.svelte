<script>
	import bot from '$lib/images/bot.png';
	import user from '$lib/images/user.png';
  
	/**
	 * @type {HTMLDivElement}
	 */
	let chatContainer;
	let loadInterval;
  
	function loader(element) {
	  element.textContent = '';
  
	  loadInterval = setInterval(() => {
		element.textContent += '.';
  
		if (element.textContent === '....') {
		  element.textContent = '';
		}
	  }, 300)
	}
  
	function typeText(element, text) {
	  let index = 0;
  
	  let typeInterval = setInterval(() => {
		if (index < text.length) {
		  element.innerHTML += text.charAt(index);
		  index++;
		} else {
		  clearInterval(typeInterval);
		}
	  }, 40)
	}
  
	function generateUniqueId() {
	  const timestamp = Date.now();
	  const randomNumber = Math.random();
	  const hexadecimalString = randomNumber.toString(16);
  
	  return `id-${timestamp}-${hexadecimalString}`;
	}
  
	function chatStripe(isAi, value, uniqueId) {
	  return (
		`
		<div class="wrapper ${isAi && 'ai'}">
		  <div class="chat">
			<div class="profile">
			  <img 
				src="${isAi ? bot : user}"
				alt="${isAi ? 'bot' : 'user'}"
			  />
			</div>
			<div class="message" id="${uniqueId}">
			  ${value}
			</div>
		  </div>
		</div>
	  `
	  )
	}
  
	async function handleSubmit(event) {
	  event.preventDefault();
  
	  const data = new FormData(event.target);
  
	  // user's chat
	  chatContainer.innerHTML += chatStripe(false, data.get('prompt'));
	  event.target.reset();
  
	  // bot's chat
	  const uniqueId = generateUniqueId();
	  chatContainer.innerHTML += chatStripe(true, " ", uniqueId);
  
	  chatContainer.scrollTop = chatContainer.scrollHeight;
  
	  const messageDiv = document.getElementById(uniqueId);
  
	  loader(messageDiv);
  
	  // Fetch data from server using OpenAI api
	  const response = await fetch(import.meta.env.VITE_API_URL, {
		method: 'POST',
		headers: {
		  'Content-Type': 'application/json'
		},
		body: JSON.stringify({
		  prompt: data.get('prompt')
		})
	  });
  
	  clearInterval(loadInterval);
	  messageDiv.innerHTML = '';
  
	  if (response.ok) {
		const responseData = await response.json();
		const parsedData = responseData.bot.trim();
  
		typeText(messageDiv, parsedData);
	  } else {
		const error = await response.text();
		messageDiv.innerHTML = "Something went wrong :(";
  
		alert(error);
	  }
	}
  </script>
  
  <div id="app">
	<div bind:this={chatContainer} id="chat_container"></div>
  
	<form id="form" on:submit={handleSubmit}>
	  <textarea name="prompt" rows="1" cols="1" placeholder="Ask Anything..."></textarea>
	  <button type="submit"><img src="$lib/images/send.png" alt="Submit"/></button>
	</form>
  </div>
  

<style>
	@import url('https://fonts.googleapis.com/css2?family=Alegreya+Sans:wght@100;300;400;500;700;800;900&display=swap');

	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
		font-family: 'Alegreya Sans', sans-serif;
	}

	#app {
		flex: auto;
		width: 100%;
		height: 100%;
		background: #343541;

		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: space-between;
	}

	#chat_container {
		flex: 1;
		width: 100%;
		height: 100%;
		overflow-y: scroll;

		display: flex;
		flex-direction: column;
		gap: 10px;

		-ms-overflow-style: none;
		scrollbar-width: none;

		padding-bottom: 20px;
		scroll-behavior: smooth;
	}

	/* hides scrollbar */
	#chat_container::-webkit-scrollbar {
		display: none;
	}

	.wrapper {
		width: 100%;
		padding: 15px;
	}

	.ai {
		background: #40414f;
	}

	.chat {
		width: 100%;
		max-width: 1280px;
		margin: 0 auto;

		display: flex;
		flex-direction: row;
		align-items: flex-start;
		gap: 10px;
	}

	.profile {
		width: 36px;
		height: 36px;
		border-radius: 0px;

		/* background: #5436DA; */

		display: flex;
		justify-content: center;
		align-items: center;
	}

	/* .ai .profile {
  background: #10a37f;
}
 */
	.profile img {
		width: 100%;
		height: 100%;
		object-fit: contain;
	}

	.message {
		flex: 1;

		color: #dcdcdc;
		font-size: 20px;

		max-width: 100%;
		overflow-x: scroll;

		/*
   * white space refers to any spaces, tabs, or newline characters that are used to format the CSS code
   * specifies how white space within an element should be handled. It is similar to the "pre" value, which tells the browser to treat all white space as significant and to preserve it exactly as it appears in the source code.
   * The pre-wrap value allows the browser to wrap long lines of text onto multiple lines if necessary.
   * The default value for the white-space property in CSS is "normal". This tells the browser to collapse multiple white space characters into a single space, and to wrap text onto multiple lines as needed to fit within its container.
  */
		white-space: pre-wrap;

		-ms-overflow-style: none;
		scrollbar-width: none;
	}

	/* hides scrollbar */
	.message::-webkit-scrollbar {
		display: none;
	}

	#form {
		width: 100%;
		max-width: 1280px;
		margin: 0 auto;
		padding: 10px;
		background: #40414f;

		display: flex;
		flex-direction: row;
		gap: 10px;
	}

	button {
		outline: 0;
		border: 0;
		cursor: pointer;
		background: transparent;
	}
</style>
