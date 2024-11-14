Table of Contents
Project Setup
Creating the Backend with Node.js and Express
Setting Up the Frontend Structure
Implementing the Main Screen Features
Implementing the Template Page
Integrating SMS Functionality
Implementing PWA Features
Testing and Deployment
Extensibility Considerations
Conclusion
1. Project Setup
1.1. Prerequisites
Ensure you have the following installed on your development machine:

Node.js (v12 or higher)
npm (comes with Node.js)
A modern web browser (e.g., Chrome, Firefox)
1.2. Create the Project Directory
Create a new directory for your project and navigate into it:

bash
Copy code
mkdir gsm-relay-controller
cd gsm-relay-controller
1.3. Initialize the Node.js Project
Initialize a new Node.js project with default settings:

bash
Copy code
npm init -y
This command creates a package.json file with default settings.

1.4. Install Required Dependencies
Since we're using Express for the backend, install it:

bash
Copy code
npm install express
For serving the PWA files, we don't need additional dependencies at this point.

2. Creating the Backend with Node.js and Express
2.1. Create the Server File
Create a file named server.js at the root of your project directory:

bash
Copy code
touch server.js
2.2. Set Up the Basic Express Server
Open server.js in your code editor and add the following code:

javascript
Copy code
const express = require('express');
const path = require('path');
const app = express();

// Serve static files from the 'public' directory
app.use(express.static(path.join(__dirname, 'public')));

// Start the server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
This code sets up a basic Express server that serves static files from the public directory.

2.3. Create the Public Directory
Create a public directory where we'll place our frontend files:

bash
Copy code
mkdir public
3. Setting Up the Frontend Structure
3.1. Create HTML Files
In the public directory, create the following files:

index.html (Main Screen)
templates.html (Template Page)
bash
Copy code
cd public
touch index.html templates.html
3.2. Create CSS and JavaScript Files
Create a css directory and a js directory, along with their respective files:

bash
Copy code
mkdir css js
touch css/styles.css js/main.js js/templates.js
4. Implementing the Main Screen Features
4.1. Building the Main Screen Layout
Open index.html and set up the basic HTML structure:

html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>GSM Relay Controller</title>
  <link rel="stylesheet" href="css/styles.css">
</head>
<body>
  <h1>GSM Relay Controller</h1>
  <div class="container">
    <input type="tel" id="phoneNumber" placeholder="Enter GSM Module Phone Number">
    <div class="buttons">
      <button id="turnOn">Turn ON</button>
      <button id="turnOff">Turn OFF</button>
      <button id="setLatchTime">Set Latch Time</button>
    </div>
    <button id="editTemplates">Edit Command Templates</button>
  </div>
  <script src="js/main.js"></script>
</body>
</html>
4.2. Styling the Main Screen
Open css/styles.css and add basic styling:

css
Copy code
body {
  font-family: Arial, sans-serif;
  text-align: center;
}

.container {
  margin-top: 50px;
}

#phoneNumber {
  width: 80%;
  padding: 10px;
  font-size: 1em;
}

.buttons {
  margin: 20px 0;
}

.buttons button {
  width: 30%;
  padding: 15px;
  font-size: 1em;
  margin: 5px;
}

#editTemplates {
  padding: 10px 20px;
  font-size: 1em;
}
4.3. Implementing the Main Screen Functionality
Open js/main.js and add event listeners for the buttons:

javascript
Copy code
document.addEventListener('DOMContentLoaded', () => {
  const phoneNumberInput = document.getElementById('phoneNumber');
  const turnOnButton = document.getElementById('turnOn');
  const turnOffButton = document.getElementById('turnOff');
  const setLatchTimeButton = document.getElementById('setLatchTime');
  const editTemplatesButton = document.getElementById('editTemplates');

  // Load saved phone number from localStorage
  phoneNumberInput.value = localStorage.getItem('gsmPhoneNumber') || '';

  // Save phone number when changed
  phoneNumberInput.addEventListener('input', () => {
    localStorage.setItem('gsmPhoneNumber', phoneNumberInput.value);
  });

  // Event listeners for action buttons
  turnOnButton.addEventListener('click', () => {
    sendSMS('turnOnCommand');
  });

  turnOffButton.addEventListener('click', () => {
    sendSMS('turnOffCommand');
  });

  setLatchTimeButton.addEventListener('click', () => {
    sendSMS('setLatchTimeCommand');
  });

  // Navigate to the Template Page
  editTemplatesButton.addEventListener('click', () => {
    window.location.href = 'templates.html';
  });

  // Function to send SMS using sms: URL scheme
  function sendSMS(commandKey) {
    const phoneNumber = phoneNumberInput.value.trim();
    if (!phoneNumber) {
      alert('Please enter the GSM module phone number.');
      return;
    }

    const commandTemplate = localStorage.getItem(commandKey) || getDefaultCommand(commandKey);
    const smsUrl = `sms:${phoneNumber}?body=${encodeURIComponent(commandTemplate)}`;
    window.location.href = smsUrl;
  }

  // Default command templates
  function getDefaultCommand(commandKey) {
    const defaultCommands = {
      turnOnCommand: '1234GON##',
      turnOffCommand: '1234GOFF##',
      setLatchTimeCommand: '1234GOT030#'
    };
    return defaultCommands[commandKey];
  }
});
Explanation:
Loading and Saving Phone Number: We use localStorage to save and load the phone number.
Event Listeners: We attach click events to the buttons to trigger the sendSMS function.
Sending SMS: The sendSMS function constructs an SMS URL using the sms: scheme and redirects the browser to it, which opens the native SMS app.
Command Templates: We fetch command templates from localStorage or use default ones.
5. Implementing the Template Page
5.1. Building the Template Page Layout
Open templates.html and set up the structure:

html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Edit Command Templates</title>
  <link rel="stylesheet" href="css/styles.css">
</head>
<body>
  <h1>Edit Command Templates</h1>
  <div class="container">
    <label for="turnOnCommand">Turn ON Command:</label><br>
    <input type="text" id="turnOnCommand" value=""><br><br>

    <label for="turnOffCommand">Turn OFF Command:</label><br>
    <input type="text" id="turnOffCommand" value=""><br><br>

    <label for="setLatchTimeCommand">Set Latch Time Command:</label><br>
    <input type="text" id="setLatchTimeCommand" value=""><br><br>

    <button id="saveTemplates">Save Templates</button>
    <button id="backButton">Back to Main Screen</button>
  </div>
  <script src="js/templates.js"></script>
</body>
</html>
5.2. Implementing the Template Page Functionality
Open js/templates.js and add the following code:

javascript
Copy code
document.addEventListener('DOMContentLoaded', () => {
  const turnOnInput = document.getElementById('turnOnCommand');
  const turnOffInput = document.getElementById('turnOffCommand');
  const setLatchTimeInput = document.getElementById('setLatchTimeCommand');
  const saveButton = document.getElementById('saveTemplates');
  const backButton = document.getElementById('backButton');

  // Load saved templates or use default
  turnOnInput.value = localStorage.getItem('turnOnCommand') || '1234GON##';
  turnOffInput.value = localStorage.getItem('turnOffCommand') || '1234GOFF##';
  setLatchTimeInput.value = localStorage.getItem('setLatchTimeCommand') || '1234GOT030#';

  // Save templates to localStorage
  saveButton.addEventListener('click', () => {
    localStorage.setItem('turnOnCommand', turnOnInput.value.trim());
    localStorage.setItem('turnOffCommand', turnOffInput.value.trim());
    localStorage.setItem('setLatchTimeCommand', setLatchTimeInput.value.trim());
    alert('Templates saved successfully!');
  });

  // Navigate back to the main screen
  backButton.addEventListener('click', () => {
    window.location.href = 'index.html';
  });
});
Explanation:
Loading and Saving Templates: We use localStorage to persist the command templates.
Navigation: A back button allows the user to return to the main screen.
6. Integrating SMS Functionality
6.1. Using the sms: URL Scheme
We already implemented the SMS functionality in main.js by constructing a URL like sms:phoneNumber?body=message.

javascript
Copy code
const smsUrl = `sms:${phoneNumber}?body=${encodeURIComponent(commandTemplate)}`;
window.location.href = smsUrl;
This approach uses the device's native SMS app to send messages.

6.2. Handling Different Platforms
Be aware that the sms: URL scheme behaves differently across devices and browsers. On most mobile devices, it should work as expected. However, during testing on desktops or unsupported browsers, this functionality may not work.

7. Implementing PWA Features
7.1. Adding a Manifest File
Create a manifest.json file in the public directory:

json
Copy code
{
  "name": "GSM Relay Controller",
  "short_name": "GSM Controller",
  "start_url": "./index.html",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#ffffff",
  "icons": [
    {
      "src": "icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
7.2. Linking the Manifest in HTML
In both index.html and templates.html, add the following line in the <head> section:

html
Copy code
<link rel="manifest" href="manifest.json">
7.3. Creating Icons
Create an icons directory inside public and place your app icons there (icon-192.png and icon-512.png).

7.4. Adding a Service Worker
7.4.1. Create the Service Worker File
Create sw.js in the public directory:

javascript
Copy code
const CACHE_NAME = 'gsm-relay-controller-cache-v1';
const urlsToCache = [
  '/',
  '/index.html',
  '/templates.html',
  '/css/styles.css',
  '/js/main.js',
  '/js/templates.js',
  '/manifest.json',
  '/icons/icon-192.png',
  '/icons/icon-512.png'
];

self.addEventListener('install', event => {
  // Perform install steps
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => {
        console.log('Opened cache');
        return cache.addAll(urlsToCache);
      })
  );
});

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        // Cache hit - return response
        if (response) {
          return response;
        }
        return fetch(event.request);
      }
    )
  );
});
7.4.2. Register the Service Worker
In both main.js and templates.js, add the following code to register the service worker:

javascript
Copy code
// At the end of the DOMContentLoaded event
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('sw.js')
    .then(registration => {
      console.log('Service Worker registered with scope:', registration.scope);
    }, err => {
      console.log('Service Worker registration failed:', err);
    });
}
7.5. Testing PWA Functionality
To test PWA features like offline support and add-to-home-screen prompts, you'll need to serve the app over HTTPS or use localhost.

8. Testing and Deployment
8.1. Running the App Locally
Start the Express server:

bash
Copy code
node server.js
Visit http://localhost:3000 in your browser or mobile device (ensure they're on the same network).

8.2. Testing on a Mobile Device
For full functionality, especially the SMS sending feature, test the app on a mobile device.

Connect your mobile device to the same network as your development machine.
Find your computer's local IP address (e.g., 192.168.1.10).
In your mobile browser, navigate to http://192.168.1.10:3000.
8.3. Deployment Considerations
For deployment, consider hosting your app on a platform that supports Node.js, such as Heroku, Vercel, or a VPS.

Ensure your hosting solution provides HTTPS support to leverage all PWA features.
Remember to update the start_url in manifest.json if necessary.
9. Extensibility Considerations
9.1. Adding More Action Buttons
To add more action buttons:

Frontend:
Add new buttons in index.html.
Assign unique IDs to the buttons.
JavaScript:
In main.js, add event listeners for the new buttons.
Define new command keys and default templates.
Template Page:
Update templates.html and templates.js to include inputs for the new commands.
9.2. Dynamic Button Generation
For greater flexibility, consider dynamically generating buttons based on a configuration stored in localStorage.

Store Button Configurations:
Store an array of button objects with properties like label, commandKey, and defaultTemplate.
Render Buttons Dynamically:
In main.js, loop through the array to create buttons.
Edit Templates Dynamically:
Adjust templates.html and templates.js to handle a dynamic list of commands.
10. Conclusion
You've now built a Progressive Web App that allows you to control your APC Connect 4V GSM relay module via SMS commands. The app is designed to be simple, responsive, and user-friendly, with the flexibility to customize command templates and add new actions in the future.

Key Features Recap:
Main Screen:
Three large action buttons: Turn ON, Turn OFF, Set Latch Time.
Input field for the GSM module's phone number.
Button to navigate to the Template Page.
Template Page:
Edit and save SMS command templates using localStorage.
SMS Integration:
Uses the device's native SMS app via the sms: URL scheme.
PWA Implementation:
Manifest file for add-to-home-screen functionality.
Service worker for offline support.
Backend:
Node.js and Express server to serve the PWA files.
Next Steps:
Testing:
Thoroughly test the app on various devices and browsers to ensure compatibility.
Enhancements:
Implement error handling and validation where necessary.
Consider adding features like command history or feedback from the GSM module.
Deployment:
Deploy the app to a live server with HTTPS support for full PWA functionality.
