import React, { createContext, useContext, useState, useEffect } from 'react';

// Create a context for audio functionality
const AudioContext = createContext(null);

export function AudioProvider({ children }) {
  const [isMuted, setIsMuted] = useState(false);
  const [volume, setVolume] = useState(0.8);
  const [audioEnabled, setAudioEnabled] = useState(false);
  const [audioQueue, setAudioQueue] = useState([]);
  const [isPlaying, setIsPlaying] = useState(false);
  
  // Initialize speech synthesis
  useEffect(() => {
    // Check if browser supports speech synthesis
    if ('speechSynthesis' in window) {
      setAudioEnabled(true);
    } else {
      console.warn('Browser tidak mendukung sintesis suara');
    }
  }, []);

  // Toggle mute/unmute
  const toggleMute = () => {
    setIsMuted(!isMuted);
  };
  
  // Adjust volume
  const adjustVolume = (newVolume) => {
    setVolume(Math.max(0, Math.min(1, newVolume)));
  };
  
  // Play instruction audio
  const speak = (text) => {
    if (!audioEnabled || isMuted) return;
    
    // Add to queue
    setAudioQueue(prev => [...prev, text]);
  };
  
  // Process audio queue
  useEffect(() => {
    if (audioQueue.length > 0 && !isPlaying && !isMuted) {
      const textToSpeak = audioQueue[0];
      
      if ('speechSynthesis' in window) {
        setIsPlaying(true);
        
        const utterance = new SpeechSynthesisUtterance(textToSpeak);
        utterance.lang = 'id-ID'; // Indonesian language
        utterance.volume = volume;
        utterance.rate = 1.0;
        utterance.pitch = 1.1;
        
        // Available voices
        let voices = window.speechSynthesis.getVoices();
        
        // Try to find an Indonesian voice
        const indonesianVoice = voices.find(voice => 
          voice.lang === 'id-ID' || voice.lang.startsWith('id')
        );
        
        if (indonesianVoice) {
          utterance.voice = indonesianVoice;
        }
        
        // When speech ends, remove from queue and set isPlaying to false
        utterance.onend = () => {
          setAudioQueue(prev => prev.slice(1));
          setIsPlaying(false);
        };
        
        // Speak the text
        window.speechSynthesis.speak(utterance);
      }
    }
  }, [audioQueue, isPlaying, isMuted, volume]);
  
  // Stop all speech
  const stopSpeech = () => {
    if ('speechSynthesis' in window) {
      window.speechSynthesis.cancel();
      setAudioQueue([]);
      setIsPlaying(false);
    }
  };
  
  // Speak cooking instruction
  const speakInstruction = (instruction) => {
    speak(instruction);
  };
  
  // Speak recipe completed message
  const speakRecipeCompleted = (recipeName) => {
    speak(`Selamat! Anda telah berhasil membuat ${recipeName}. Hidangan lezat!`);
  };
  
  // Speak recipe failed message
  const speakRecipeFailed = () => {
    speak(`Ups! Bahan yang digunakan tidak tepat. Silakan coba lagi.`);
  };
  
  // Speak game start message
  const speakGameStart = (difficulty) => {
    let difficultyText = '';
    switch(difficulty) {
      case 'easy': difficultyText = 'mudah'; break;
      case 'hard': difficultyText = 'sulit'; break;
      default: difficultyText = 'sedang';
    }
    
    speak(`Selamat datang di Masak Yuk! Anda bermain dalam mode ${difficultyText}. Mari mulai memasak!`);
  };
  
  return (
    <AudioContext.Provider value={{
      isMuted,
      volume,
      toggleMute,
      adjustVolume,
      audioEnabled,
      speakInstruction,
      speakRecipeCompleted,
      speakRecipeFailed,
      speakGameStart,
      stopSpeech
    }}>
      {children}
    </AudioContext.Provider>
  );
}

// Custom hook to use audio context
export function useAudio() {
  const context = useContext(AudioContext);
  if (!context) {
    throw new Error('useAudio must be used within an AudioProvider');
  }
  return context;
}
