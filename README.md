# goal-traker
Goal Tracker Bot
Features of this Chatbot:
Voice Input → User can speak
import { useState, useEffect, useRef } from "react";

Voice Output → Bot speaks back
const tips = [
  { text: "50-30-20 Rule for Smart Saving", img: "https://i.imgur.com/3XJmYfZ.png" },
  { text: "Track every expense to avoid overspending", img: "https://i.imgur.com/7VBBsNk.png" },
];

Text Chat → Manual typing option
export default function SmartBudgetChatbot() {
  const [messages, setMessages] = useState([
    { type: "bot", text: "Hi! I'm your Smart Budget Goal Tracker 💰" },
  ]);
  const [input, setInput] = useState("");
  const [goals, setGoals] = useState(
    JSON.parse(localStorage.getItem("goals")) || []
  );

Images → Tips with visuals
  const recognitionRef = useRef(null);

Goal Setting & Tracking
  useEffect(() => {
    localStorage.setItem("goals", JSON.stringify(goals));
  }, [goals]);

  useEffect(() => {
    if ("webkitSpeechRecognition" in window) {
      const SpeechRecognition = window.webkitSpeechRecognition;
      const recognition = new SpeechRecognition();
      recognition.lang = "en-US";
      recognition.interimResults = false;
      recognition.maxAlternatives = 1;

      recognition.onresult = (event) => {
        const voiceInput = event.results[0][0].transcript;
        setInput(voiceInput);
        handleSend(voiceInput);
      };

      recognitionRef.current = recognition;
    }
  }, []);

  const speak = (text) => {
    const synth = window.speechSynthesis;
    const utterance = new SpeechSynthesisUtterance(text);
    synth.speak(utterance);
  };

  const startListening = () => {
    if (recognitionRef.current) {
      recognitionRef.current.start();
    }
  };

  const handleSend = (customInput) => {
    const userInput = customInput || input;
    if (!userInput.trim()) return;

    setMessages([...messages, { type: "user", text: userInput }]);

    let botResponse = "Type 'Set Goal', 'View Goals', or 'Tip'";

    if (userInput.toLowerCase().includes("set goal")) {
      botResponse = "Enter goal name & target amount (eg: Phone 30000)";
    } else if (userInput.match(/^[a-zA-Z ]+\s\d+$/)) {
      const [name, amount] = userInput.split(" ");
      const newGoal = { name, target: parseInt(amount), saved: 0 };
      setGoals([...goals, newGoal]);
      botResponse = `Goal '${name}' added! 🎯`;
    } else if (userInput.toLowerCase().includes("view goals")) {
      if (goals.length === 0) {
        botResponse = "No goals yet!";
      } else {
        goals.forEach((g) => {
          setMessages((prev) => [...prev, { type: "bot", text: `${g.name}: ₹${g.saved}/₹${g.target}` }]);
          speak(`${g.name} saved ₹${g.saved} of ₹${g.target}`);
        });
        botResponse = "Here are your goals:";
      }
    } else if (userInput.toLowerCase().includes("tip")) {
      const tip = tips[Math.floor(Math.random() * tips.length)];
      setMessages((prev) => [...prev, { type: "bot", text: tip.text, img: tip.img }]);
      speak(tip.text);
      setInput("");
      return;
    }

    setMessages((prev) => [...prev, { type: "bot", text: botResponse }]);
    speak(botResponse);

    setInput("");
  };

  return (
    <div className="max-w-md mx-auto mt-10 p-4 border rounded-2xl shadow-xl">
      <h1 className="text-xl font-bold mb-4 text-center">Smart Budget Goal Tracker 🤖</h1>

      <div className="h-80 overflow-y-auto mb-4 space-y-2">
        {messages.map((m, i) => (
          <div key={i} className={`p-2 rounded-xl ${m.type === "bot" ? "bg-gray-200" : "bg-green-200 text-right"}`}>
            {m.text}
            {m.img && <img src={m.img} alt="tip" className="mt-2 w-32 mx-auto" />}
          </div>
        ))}
      </div>

      <div className="flex gap-2">
        <input
          className="border rounded-xl px-4 py-2 w-full"
          placeholder="Type here or use voice..."
          value={input}
          onChange={(e) => setInput(e.target.value)}
        />
        <button onClick={() => handleSend()} className="bg-blue-500 text-white px-4 py-2 rounded-xl">
          Send
        </button>
        <button onClick={startListening} className="bg-green-500 text-white px-4 py-2 rounded-xl">
          🎤
        </button>
      </div>
    </div>
  );
}
