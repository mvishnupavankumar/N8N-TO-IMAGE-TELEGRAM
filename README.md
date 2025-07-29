# N8N-TO-IMAGE-TELEGRAM
GITHUB WRITER
Custom Gem
📂 Suggested Repo Name:
telegram-ai-video-storyteller

📘 README.md Content:
📸 AI Video Storyteller to Telegram using n8n
📖 Description
💬 For beginners: This n8n workflow lets you send a file (like a text document or JSON) to a Telegram bot. Then, it uses AI to turn that text into a professional video script, breaks it down into individual scenes with image and motion prompts, generates images for each scene, downloads them, and finally sends each scene's image along with its voiceover text and motion description to a specified Telegram chat. It's like having a miniature video production studio in your chat!

💻 For tech users: This workflow is triggered by a Telegram message containing a file. It fetches the file, extracts the JSON content, and then uses a series of HTTP Request nodes to interact with the Google Gemini API for script summarization and scene breakdown. It also leverages the Together AI API for image generation based on generated prompts. Code nodes are heavily used for data manipulation, cleaning, and structuring the content for the AI and for parsing the AI's output into distinct scenes. Set nodes are used to prepare and extract data at various stages. Finally, Telegram nodes send the generated images and associated text back to a designated chat ID. Wait nodes are incorporated to ensure proper spacing between sending multiple images.

⚙️ How It Works (step-by-step with emojis)
🔹 1. Telegram Trigger 🔔
Simple: This is where the magic starts! It waits for you to send a message with a file attached to your Telegram bot.

Tech: Uses the Telegram Trigger node, configured to listen for message updates, enabling the workflow to start when a file is sent.

🔹 2. Get a file 📥
Simple: Once you send a file to the bot, this step grabs that file so the workflow can use its content.

Tech: Uses the Telegram node with the file resource and fileId expression ={{ $json.message.document.file_id }} to download the file sent via the trigger.

🔹 3. EXTRACT JSON 📄
Simple: This step opens the file you sent and reads its contents, assuming it's a special type of text that the workflow understands.

Tech: A Code node processes the binary data of the uploaded file, converting it from base64 to a buffer and then parsing it as a JSON object.

🔹 4. MAKE PREETY WAY ✍️
Simple: This part prepares the extracted text to be sent to an AI for understanding.

Tech: A Code node constructs a prompt for the Google Gemini API, embedding the extracted JSON content from the previous step.

🔹 5. SUMMARIZE IT 💡
Simple: It sends the prepared text to an AI (like a smart assistant) to get a summary or initial understanding of the content.

Tech: An HTTP Request node sends a POST request to the Google Generative Language API (gemini-2.0-flash:generateContent endpoint) with the JSON body constructed in the "MAKE PREETY WAY" node.

🔹 6. EXTRACT DATA 📊
Simple: Takes the AI's response and pulls out the main message or script from it.

Tech: A Set node extracts the text content from the AI's response ($json.body.candidates[0].content.parts[0].text) and assigns it to a new field named MESSAGE.

🔹 7. CLEANS IT UP 🧹
Simple: Removes any messy bits like emojis or special symbols from the AI's response to make it neat.

Tech: A Code node cleans the MESSAGE text by removing emojis, specific characters (arrows, checkmarks), markdown formatting (bold, headings), collapsing multiple newlines, and trimming whitespace.

🔹 8. MAKE PREETY ✨
Simple: Further cleans and formats the text, removing extra spaces and some symbols to get it ready for the next AI step.

Tech: Another Code node performs additional cleaning, removing slashes, replacing newlines with spaces, filtering out non-alphanumeric characters (except . ,), collapsing multiple spaces, and trimming the final string.

🔹 9. SUMMARIZE IT1 📝
Simple: Sends the cleaned text to another AI, this time to turn it into a 60-second voiceover script for a video.

Tech: An HTTP Request node calls the Google Generative Language API again, sending a detailed prompt for generating a voiceover script based on the cleanedText from the previous step.

🔹 10. SENDING SCRIPT 📨
Simple: Saves the freshly generated voiceover script into a temporary spot for later use.

Tech: A Set node assigns the AI-generated script ($json.body.candidates[0].content.parts[0].text) to a field called SCRIPT.

🔹 11. REMOVE GLITCH ✂️
Simple: Does one last cleanup on the script, making sure there are no odd characters that might cause problems.

Tech: A Code node cleans the SCRIPT text, removing slashes, newlines, special characters (except word characters, space, comma, period), and collapses multiple spaces.

🔹 12. SUMMARIZE IT2 🎬
Simple: Sends the polished script to an AI to break it down into different video scenes, complete with descriptions for images and motion.

Tech: An HTTP Request node sends a request to the Google Generative Language API to break the script into cinematic scenes, including detailed prompts for image and motion generation.

🔹 13. SENDING SCRIPT1 📤
Simple: Stores the detailed scene breakdown received from the AI.

Tech: A Set node captures the AI's scene-by-scene output ($json.body.candidates[0].content.parts[0].text) and stores it under the SCRIPT field.

🔹 14. CLEAN SCRIPT 🖌️
Simple: This step carefully separates each scene from the AI's output, making them individual pieces.

Tech: A Code node uses regular expressions to parse the SCRIPT into individual scene objects, extracting details like sceneTitle, dialogueSummary, avatarDialog, hdImagePrompt, and motionFramePrompt.

🔹 15. NAME SCENES 🏷️
Simple: Organizes the scene information, keeping only the important details needed to create the images and voiceovers.

Tech: A Code node iterates through the parsed scenes and removes the dialogueSummary field, preparing the data for the next steps.

🔹 16. SEPERATE SCENES 📦
Simple: Divides all the scenes into separate paths, so each scene can be handled individually.

Tech: This Code node takes all the processed scenes and outputs them as separate items, each containing sceneNumber, sceneMessage, hdImagePrompt, and motionFramePrompt. This allows subsequent nodes to process each scene independently.

🔹 17-22. SCENE 1, SCENE 2, SCENE 3, SCENE 4, SCENE 5, SCENE 6 🎭
Simple: Each of these steps specifically picks out one scene at a time from the separated scenes. For example, "SCENE 1" focuses only on the first scene's details.

Tech: These Code nodes filter the incoming items to select a specific scene number (e.g., scene.sceneNumber === 1 for "SCENE 1"), ensuring that subsequent processing targets individual scenes.

🔹 23-28. Wait, Wait2, Wait3, Wait4, Wait5 ⏳
Simple: These are like little pauses in the workflow, ensuring that we don't send too many requests too fast to other services.

Tech: These Wait nodes pause the workflow for a specified amount of time (e.g., 75 seconds), which can be crucial for API rate limits or to ensure previous operations are complete.

🔹 29-34. EXTRACT DATA, EXTRACT DATA1, EXTRACT DATA2, EXTRACT DATA3, EXTRACT DATA4, EXTRACT DATA5 📝
Simple: For each scene, these steps grab the specific image prompt, motion prompt, and voiceover text.

Tech: These Set nodes extract the hdImagePrompt (assigned to promtp), motionFramePrompt (assigned to MOTION), and sceneMessage (assigned to VOICE) for each respective scene.

🔹 35-40. IMAGE URL GENERATION, IMAGE URL GENERATION1, ..., IMAGE URL GENERATION5 🖼️
Simple: This is where the magic happens! For each scene, it sends the image description to an AI that creates a unique image based on that description.

Tech: These HTTP Request nodes send POST requests to the Together AI image generation API (https://api.together.xyz/v1/images/generations). They use the promtp (image prompt) generated earlier to create a black-forest-labs/FLUX.1-schnell model image with specific dimensions (432x768).

🔹 41-46. IMAGE DOWNLOAD, IMAGE DOWNLOAD1, ..., IMAGE DOWNLOAD5 💾
Simple: Once the AI creates an image, this step downloads that image so it can be sent to Telegram.

Tech: These HTTP Request nodes perform a GET request to the URL of the generated image (obtained from {{ $json.body.data[0].url }}) and download the image file. The response format is set to file.

🔹 47-52. Send a photo message, Send a photo message1, ..., Send a photo message5 📲
Simple: Finally, for each scene, this sends the created image along with its voiceover text and motion instructions to your Telegram chat.

Tech: These Telegram nodes use the sendPhoto operation to send the downloaded binary image data. The chatId is configured, and a caption is dynamically generated to include the scene number, motion prompt, and voiceover text for context.

🔁 Replace the following before running:
🔐 YOUR_GOOGLE_API_KEY
Replace this with your personal Google API Key inside the SUMMARIZE IT, SUMMARIZE IT1, and SUMMARIZE IT2 HTTP Request nodes. This key is used for accessing the Gemini AI model.

🔐 YOUR_TOGETHER_API_KEY
Replace this with your personal Together AI API Key inside all IMAGE URL GENERATION HTTP Request nodes. This key is used to generate images.

💬 YOUR_TELEGRAM_CHAT_ID
Insert your personal Telegram chat ID in all Send a photo message Telegram nodes. This is where the images and captions will be sent.

🤖 REPLACE_ME
Configure your Telegram API credentials in n8n under the Credentials section for all Telegram Trigger and Telegram nodes. This typically involves setting up a Telegram Bot token.

🧪 Testing Instructions
✅ Open n8n at http://localhost:5678 (or your n8n instance URL).
🔧 Click “Import Workflow” and paste the provided JSON.
💬 In Telegram, send a message with a JSON file (containing the content you want to process) to your configured bot.
🧪 Monitor the workflow execution in n8n. You can click “Execute Node” on the "Telegram Trigger" node to manually simulate a trigger for testing without sending a file every time.
🪵 Check the execution log (bottom panel in n8n) to troubleshoot any issues and verify data flowing between nodes.
👀 Observe your Telegram chat for the incoming images and their associated captions.

💡 Tips for Beginners
💡 Use the “Credentials” tab in n8n to set up your Telegram API and Together AI API keys securely.
💡 After executing each node (especially the Code and HTTP Request nodes), check the “Output” tab to see what data is being passed along. This helps in understanding the flow and debugging.
💡 If you're not getting images or messages in Telegram, double-check your Telegram Bot Token and Chat ID for correctness.
💡 If AI requests fail, ensure your Google API Key and Together AI API Key are valid and have the necessary permissions.
💡 The "Wait" nodes are important! They prevent overwhelming the image generation API. You might need to adjust the amount of time they wait based on the API's rate limits and your specific needs.

