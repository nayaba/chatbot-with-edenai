# AI Chat Bot using React and Bootstrap

## **Step 1: Set Up a New Vite + React Project**
1. **Create a new Vite project**:
   ```sh
   npm create vite@latest chatbot --template react
   ```

2. **Navigate into the project folder**:
   ```sh
   cd chatbot
   ```

3. **Install dependencies**:
   ```sh
   npm install bootstrap
   ```

4. **Start the development server**:
   ```sh
   npm run dev
   ```

---

## **Step 2: Set Up Bootstrap**
We want minimal styling, but let’s **import Bootstrap** so we can use basic layouts.

Open `src/main.jsx` and add:

```jsx
import 'bootstrap/dist/css/bootstrap.min.css';
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

---

## **Step 3: Get API Key from EdenAI**
1. Sign up at [EdenAI](https://www.edenai.co/)
2. Go to **API Keys** and generate a key.
3. **Create a `.env` file** in your project root and add:
   ```
   VITE_EDENAI_API_KEY=your_api_key_here
   ```
4. **Restart your Vite server** so the `.env` file is loaded.

---

## **Step 4: Create the Chatbot Component**
Now, we’ll build our chatbot **step by step** rather than just pasting code.

### **1. Create a new component file**
Inside `src/components/`, create a new file:  
`Chatbot.jsx`

### **2. Set up initial state**
We need:
- `messages` to store chat history.
- `formData` to track the input field.

```jsx
import React, { useState } from 'react';

const Chatbot = () => {
  const [messages, setMessages] = useState([]);
  const [formData, setFormData] = useState({ text: '' });

  const handleChange = (event) => {
    setFormData({ ...formData, [event.target.name]: event.target.value });
  };

  return (
    <div className="container mt-3">
      <h2>Chatbot</h2>
      <div style={{ height: '300px', overflowY: 'auto', border: '1px solid #ccc', padding: '10px' }}>
        {messages.map((msg, index) => (
          <p key={index}><strong>{msg.sender}:</strong> {msg.text}</p>
        ))}
      </div>
      <form>
        <input 
          type="text" 
          name="text"
          value={formData.text} 
          onChange={handleChange} 
          className="form-control"
          placeholder="Type a message..."
        />
      </form>
    </div>
  );
};

export default Chatbot;
```

At this point, we can **see the chat messages and input field** but don’t have functionality to send messages.

---

## **Step 5: Add Form Handling & API Call**
### **1. Add `handleSubmit`**
We want to send a request to EdenAI when the user submits the form.

```jsx
const handleSubmit = async (event) => {
  event.preventDefault();  // Prevents page reload

  if (!formData.text.trim()) return;  // Ignore empty messages

  const userMessage = { sender: 'You', text: formData.text };
  setMessages(prev => [...prev, userMessage]);

  const response = await fetch('https://api.edenai.run/v2/text/chat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${import.meta.env.VITE_EDENAI_API_KEY}`
    },
    body: JSON.stringify({
      providers: ['openai'],
      text: formData.text,
      chatbot_global_action: '',
      previous_history: messages.map(msg => ({ role: msg.sender, message: msg.text })),
      temperature: 0.5,
    })
  });

  const data = await response.json();
  const botMessage = { sender: 'Bot', text: data.openai.generated_text };

  setMessages(prev => [...prev, botMessage]);
  setFormData({ text: '' }); // Clear input field
};
```

### **2. Connect the Form to `handleSubmit`**
Modify the form to include a submit button and `onSubmit` event.

```jsx
<form onSubmit={handleSubmit} className="mt-3">
  <input 
    type="text" 
    name="text"
    value={formData.text} 
    onChange={handleChange} 
    className="form-control"
    placeholder="Type a message..."
  />
  <button type="submit" className="btn btn-primary mt-2">Send</button>
</form>
```

---

## **Step 6: Integrate the Chatbot in App**
Open `src/App.jsx` and replace the contents with:

```jsx
import React from 'react';
import Chatbot from './components/Chatbot';

const App = () => {
  return (
    <div className="container mt-5">
      <Chatbot />
    </div>
  );
};

export default App;
```

---

## **Step 7: Run the Chatbot**
1. Start the app:
   ```sh
   npm run dev
   ```
2. Open **http://localhost:5173/** and test your chatbot.

