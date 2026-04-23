<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Doraemon Vocab Game</title>
    
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- React & ReactDOM -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
    
    <!-- Babel for translating JSX in the browser -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef, useMemo } = React;

        // --- VOCABULARY DATA ---
        const PLACES = [
          { word: "park", clue: "You want to play outside and see trees (樹木). Where are you going?", emoji: "🌳", img: "https://images.unsplash.com/photo-1604328698692-f76ea9498e76?w=600&q=80" },
          { word: "library", clue: "You want to read and borrow books (書). Where are you going?", emoji: "📚", img: "https://images.unsplash.com/photo-1521587760476-6c12a4b040da?w=600&q=80" },
          { word: "hospital", clue: "You are sick and need to see a doctor (醫生). Where are you going?", emoji: "🏥", img: "https://images.unsplash.com/photo-1586773860418-d37222d8fce3?w=600&q=80" },
          { word: "supermarket", clue: "You want to buy food (食物) and groceries. Where are you going?", emoji: "🛒", img: "https://images.unsplash.com/photo-1578916171728-46686eac8d58?w=600&q=80" },
          { word: "bookstore", clue: "You want to buy (買) new books. Where are you going?", emoji: "📖", img: "https://images.unsplash.com/photo-1507842217343-583bb7270b66?w=600&q=80" },
          { word: "tea shop", clue: "You want to drink tea (茶) or boba. Where are you going?", emoji: "🧋", img: "https://images.unsplash.com/photo-1558160074-4d7d8bdf4256?w=600&q=80" },
          { word: "post office", clue: "You want to send a letter (信) or package. Where are you going?", emoji: "✉️", img: "https://images.unsplash.com/photo-1578909196400-59f8f8156a05?w=600&q=80" },
          { word: "zoo", clue: "You want to see wild animals (動物). Where are you going?", emoji: "🦁", img: "https://images.unsplash.com/photo-1534567153574-2b12153a87f0?w=600&q=80" }
        ];

        const COLORS = ['bg-red-500', 'bg-blue-500', 'bg-yellow-400', 'bg-green-500'];
        const BORDER_COLORS = ['border-red-500', 'border-blue-500', 'border-yellow-400', 'border-green-500'];
        const TEXT_COLORS = ['text-red-500', 'text-blue-500', 'text-yellow-500', 'text-green-500'];

        // --- AUDIO SYNTHESIS ENGINE ---
        class GameAudio {
          constructor() {
            try {
              this.ctx = new (window.AudioContext || window.webkitAudioContext)();
            } catch(e) {
              console.warn("Audio Context blocked or not supported by browser.");
            }
            this.bgmOsc = null;
            this.bgmGain = null;
            this.isPlaying = false;
          }
          playTone(freq, type, duration, vol = 0.1) {
            if (!this.ctx) return;
            try {
              if (this.ctx.state === 'suspended') this.ctx.resume();
              const osc = this.ctx.createOscillator();
              const gain = this.ctx.createGain();
              osc.type = type;
              osc.frequency.setValueAtTime(freq, this.ctx.currentTime);
              gain.gain.setValueAtTime(vol, this.ctx.currentTime);
              gain.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + duration);
              osc.connect(gain);
              gain.connect(this.ctx.destination);
              osc.start();
              osc.stop(this.ctx.currentTime + duration);
            } catch(e) {}
          }
          playCorrect() {
            this.playTone(440, 'sine', 0.1, 0.2); 
            setTimeout(() => this.playTone(554.37, 'sine', 0.1, 0.2), 100); 
            setTimeout(() => this.playTone(659.25, 'sine', 0.3, 0.2), 200); 
          }
          playWrong() {
            this.playTone(150, 'sawtooth', 0.3, 0.3);
            setTimeout(() => this.playTone(100, 'sawtooth', 0.4, 0.3), 150);
          }
          playWhistle() {
            this.playTone(1200, 'sine', 0.2, 0.2);
            setTimeout(() => this.playTone(1400, 'sine', 0.4, 0.2), 100);
          }
          playDunk() {
            this.playTone(600, 'sine', 0.1, 0.3); 
            setTimeout(() => this.playTone(900, 'sine', 0.1, 0.3), 150);
            setTimeout(() => this.playTone(1200, 'sine', 0.3, 0.3), 300);
          }
        }

        // --- DORAYAKI SHOWER COMPONENT ---
        const DorayakiShower = ({ imageSrc }) => {
          const drops = useMemo(() => Array.from({ length: 50 }).map((_, i) => ({
            id: i,
            left: `${Math.random() * 100}%`,
            animDuration: `${1.5 + Math.random() * 2}s`, 
            animDelay: `${Math.random() * 0.5}s`, 
            scale: 0.4 + Math.random() * 0.8, 
          })), []);

          return (
            <div className="absolute inset-0 z-[100] pointer-events-none overflow-hidden">
              {drops.map(d => (
                <div
                  key={d.id}
                  className="absolute dorayaki-drop"
                  style={{
                    left: d.left,
                    top: '-150px',
                    animationDuration: d.animDuration,
                    animationDelay: d.animDelay,
                  }}
                >
                  <div style={{ transform: `scale(${d.scale})` }}>
                    <img 
                      src={imageSrc} 
                      alt="Dorayaki" 
                      className="w-24 h-24 object-cover rounded-full drop-shadow-2xl bg-transparent"
                      onError={(e) => {
                        e.target.onerror = null; 
                        e.target.src = 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><text y=".9em" font-size="90">🥞</text></svg>';
                      }}
                    />
                  </div>
                </div>
              ))}
            </div>
          );
        };

        // --- IMAGE COMPONENT ---
        const CharacterImage = ({ character }) => {
          return (
            <img 
              src={character.img} 
              alt={character.name} 
              className="w-full h-full object-cover rounded-full border-4 border-blue-500 shadow-[0_0_15px_rgba(59,130,246,0.8)] bg-white pointer-events-none"
              onError={(e) => {
                e.target.onerror = null; 
                e.target.src = `https://placehold.co/200x200/000000/ffffff?text=${character.name}`;
              }}
            />
          );
        };

        function App() {
          const [gameState, setGameState] = useState('start'); 
          const [teams, setTeams] = useState([
            { id: 1, name: 'Doraemon', img: 'https://placehold.co/200x200/3B82F6/ffffff?text=Doraemon', score: 0, locked: false },
            { id: 2, name: 'Nobita', img: 'https://placehold.co/200x200/FBBF24/ffffff?text=Nobita', score: 0, locked: false },
            { id: 3, name: 'Shizuka', img: 'https://placehold.co/200x200/EC4899/ffffff?text=Shizuka', score: 0, locked: false },
            { id: 4, name: 'Gian', img: 'https://placehold.co/200x200/F97316/ffffff?text=Gian', score: 0, locked: false },
            { id: 5, name: 'Suneo', img: 'https://placehold.co/200x200/10B981/ffffff?text=Suneo', score: 0, locked: false }
          ]);
          const [options, setOptions] = useState([]);
          const [currentQuestion, setCurrentQuestion] = useState(null);
          const [roundOver, setRoundOver] = useState(false);
          const [message, setMessage] = useState('');
          const [dunkTarget, setDunkTarget] = useState(null); 
          const [showShower, setShowShower] = useState(false); 

          const [showerImg, setShowerImg] = useState("DORAYAKIDORAEMON.png");

          const [timeLeft, setTimeLeft] = useState(60);
          const [prepTime, setPrepTime] = useState(0);

          const audioRef = useRef(null);
          const questionQueue = useRef([]); 

          useEffect(() => {
            let timer;
            if (gameState === 'playing') {
              timer = setInterval(() => {
                setTimeLeft(prev => {
                  if (prev <= 1) {
                    setGameState('gameOver');
                    clearInterval(timer);
                    return 0;
                  }
                  return prev - 1;
                });
              }, 1000);
            }
            return () => clearInterval(timer);
          }, [gameState]);

          useEffect(() => {
            let timer;
            if (gameState === 'playing' && prepTime > 0) {
              timer = setInterval(() => {
                setPrepTime(prev => prev - 1);
              }, 1000);
            }
            return () => clearInterval(timer);
          }, [gameState, prepTime]);

          const handleFileUpload = (teamId, e) => {
            const file = e.target.files[0];
            if (file) {
              const url = URL.createObjectURL(file);
              setTeams(prev => prev.map(t => t.id === teamId ? { ...t, img: url } : t));
            }
          };

          const handleShowerUpload = (e) => {
            const file = e.target.files[0];
            if (file) {
              const url = URL.createObjectURL(file);
              setShowerImg(url);
            }
          };

          const handleNameChange = (teamId, newName) => {
            setTeams(prev => prev.map(t => t.id === teamId ? { ...t, name: newName } : t));
          };

          const startGame = (e) => {
            if (e) e.preventDefault(); 
            
            try {
              if (!audioRef.current) audioRef.current = new GameAudio();
              audioRef.current.playWhistle();
            } catch(err) {}
            
            questionQueue.current = [...PLACES].sort(() => 0.5 - Math.random());
            
            setTimeLeft(60); 
            setGameState('playing');
            generateRound(true);
          };

          const generateRound = (isStart = false) => {
            if (questionQueue.current.length === 0) {
              questionQueue.current = [...PLACES].sort(() => 0.5 - Math.random());
            }

            const answer = questionQueue.current.pop();
            
            const wrongOptions = PLACES.filter(p => p.word !== answer.word)
                                       .sort(() => 0.5 - Math.random())
                                       .slice(0, 3);
                                       
            let selected = [answer, ...wrongOptions].sort(() => 0.5 - Math.random());
            
            const randomOrder = [0, 1, 2, 3].sort(() => Math.random() - 0.5);

            let roundOptions = selected.map((place, index) => ({
              ...place,
              colorClass: COLORS[index],
              borderColorClass: BORDER_COLORS[index],
              textColorClass: TEXT_COLORS[index],
              isCorrect: place.word === answer.word,
              occupant: null,
              originalIndex: index,
              displayOrder: randomOrder[index] 
            }));

            setOptions(roundOptions);
            setCurrentQuestion(answer);
            setRoundOver(false);
            setMessage('');
            setDunkTarget(null);
            setShowShower(false); 
            setPrepTime(1); 
            
            setTeams(prev => prev.map(t => ({ 
              ...t, 
              locked: false,
              ...(isStart ? { score: 0 } : {}) 
            })));
          };

          const handleAnswer = (teamId, optionIndex) => {
            if (roundOver || prepTime > 0) return;
            
            const teamIndex = teams.findIndex(t => t.id === teamId);
            if (teams[teamIndex].locked) return;

            const selectedOption = options[optionIndex];
            const newTeams = [...teams];

            if (selectedOption.isCorrect) {
              setRoundOver(true);
              setDunkTarget(optionIndex); 
              setShowShower(true); 
              if (audioRef.current) audioRef.current.playDunk();

              setTimeout(() => {
                if (audioRef.current) audioRef.current.playCorrect();
                newTeams[teamIndex].score += 2;
                setTeams(newTeams);
                
                const character = newTeams[teamIndex];
                setMessage(`DORAYAKI GET! +2 TO ${character.name.toUpperCase()}!`);
                
                const newOptions = [...options];
                newOptions[optionIndex].occupant = character;
                setOptions(newOptions);

                setTimeout(() => {
                  setGameState(prev => { if (prev === 'playing') generateRound(false); return prev; });
                }, 3500);
              }, 1000);

            } else {
              if (audioRef.current) audioRef.current.playWrong();
              newTeams[teamIndex].score -= 1;
              newTeams[teamIndex].locked = true;
              setTeams(newTeams);
              
              if (newTeams.every(t => t.locked)) {
                setRoundOver(true);
                setMessage("OH NO! EVERYONE MISSED!");
                setTimeout(() => {
                   setGameState(prev => { if (prev === 'playing') generateRound(false); return prev; });
                }, 3000);
              }
            }
          };

          const renderCardBlock = (blockId) => {
            const renderedOptions = [...options].sort((a, b) => a.displayOrder - b.displayOrder);
            return (
              <div className="flex gap-4 md:gap-6 pr-4 md:pr-6 h-full py-2 flex-shrink-0 items-center justify-center">
                {renderedOptions.map((opt) => {
                  const i = opt.originalIndex;
                  return (
                    <div 
                      key={`${blockId}-${i}`} 
                      className={`w-[45vw] sm:w-[35vw] md:w-[23vw] lg:w-[280px] h-full max-h-[320px] rounded-2xl border-4 ${opt.borderColorClass} flex flex-col shadow-[0_0_20px_rgba(0,0,0,0.4)] relative overflow-hidden transition-all duration-500 transform ${roundOver && opt.isCorrect ? 'scale-105 z-20 shadow-[0_0_30px_rgba(250,204,21,0.6)]' : ''} flex-shrink-0`}
                      style={{
                        backgroundImage: `linear-gradient(to bottom, rgba(0,0,0,0.4), rgba(0,0,0,0.9)), url(${opt.img})`,
                        backgroundSize: 'cover', backgroundPosition: 'center'
                      }}
                    >
                      {dunkTarget === i && (
                        <div className="absolute inset-0 z-40 flex flex-col items-center justify-start pt-4 bg-black bg-opacity-40">
                          <img 
                            src={showerImg} 
                            alt="Dorayaki" 
                            className="anim-dunk w-24 h-24 object-cover rounded-full absolute z-50 drop-shadow-2xl bg-transparent" 
                            onError={(e) => {
                              e.target.onerror = null; 
                              e.target.src = 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><text y=".9em" font-size="90">🥞</text></svg>';
                            }}
                          />
                          <div className="absolute top-[40%] flex flex-col items-center z-40 anim-net">
                             <div className="w-28 h-2 bg-white border-2 border-black rounded-full z-10"></div>
                             <div className="w-28 h-16 bg-white border-x-2 border-b-2 border-black rounded-b-[40px] shadow-[inset_0_5px_10px_rgba(0,0,0,0.2)] -mt-1"></div>
                          </div>
                        </div>
                      )}

                      {opt.occupant && dunkTarget !== i && (
                        <div className="absolute inset-0 z-30 flex items-center justify-center bg-black bg-opacity-60 backdrop-blur-sm animate-[bounce_0.5s_ease-in-out]">
                          <div className="w-32 h-32 md:w-40 md:h-40 anim-float">
                            <CharacterImage character={opt.occupant} />
                          </div>
                        </div>
                      )}
                      
                      <div className="flex-1 flex items-center justify-center opacity-95">
                        <span className="text-7xl md:text-8xl drop-shadow-[0_5px_5px_rgba(0,0,0,1)]">{opt.emoji}</span>
                      </div>
                      <div className={`h-16 ${opt.colorClass} flex items-center justify-center bg-opacity-90 backdrop-blur-md border-t-2 border-white/20`}>
                        <span className="text-2xl md:text-3xl font-black lowercase tracking-widest text-white drop-shadow-md" style={{ fontFamily: 'Impact' }}>
                          {opt.word}
                        </span>
                      </div>

                      {roundOver && !opt.isCorrect && !opt.occupant && (
                         <div className="absolute inset-0 bg-red-900 bg-opacity-70 flex items-center justify-center z-20 backdrop-blur-sm">
                           <span className="text-red-500 text-9xl font-black opacity-90 drop-shadow-[0_0_20px_rgba(0,0,0,1)]">X</span>
                         </div>
                      )}
                    </div>
                  );
                })}
              </div>
            );
          };

          return (
            <React.Fragment>
              <style>{`
                @keyframes floatChar {
                  0%, 100% { transform: translateY(0px) scale(1); }
                  50% { transform: translateY(-8px) scale(1.02); }
                }
                .anim-float { animation: floatChar 3s ease-in-out infinite; }
                
                @keyframes dropDorayaki {
                  0% { transform: translateY(-400px) rotate(0deg) scale(1.5); }
                  50% { transform: translateY(0px) rotate(180deg) scale(1.2); }
                  80% { transform: translateY(100px) rotate(360deg) scale(1); opacity: 1; }
                  100% { transform: translateY(200px) rotate(450deg) scale(0.8); opacity: 0; }
                }
                .anim-dunk { animation: dropDorayaki 1s ease-in forwards; }
                
                @keyframes openPocket {
                  0%, 100% { transform: scaleY(1); }
                  50% { transform: scaleY(1.3) translateY(5px); }
                  70% { transform: scaleY(1.1) translateY(2px); }
                }
                .anim-net {
                  animation: openPocket 0.5s ease-in-out 0.4s forwards;
                  transform-origin: top;
                }

                @keyframes dorayakiRain {
                  0% { transform: translateY(0) rotate(0deg); }
                  100% { transform: translateY(120vh) rotate(720deg); }
                }
                .dorayaki-drop {
                  animation-name: dorayakiRain;
                  animation-timing-function: linear;
                  animation-fill-mode: forwards;
                }

                @keyframes marquee {
                  0% { transform: translateX(0); }
                  100% { transform: translateX(-50%); }
                }
                .anim-marquee { 
                  animation: marquee 15s linear infinite; 
                }
              `}</style>

              <div className="min-h-screen flex flex-col font-sans select-none overflow-hidden relative bg-cover bg-center"
                   style={{ backgroundImage: 'url(https://images.unsplash.com/photo-1536244636800-a3f74db0f3cf?q=80&w=2000&auto=format&fit=crop)' }}>
                <div className="absolute inset-0 bg-blue-900 bg-opacity-40 pointer-events-none z-0"></div>

                {/* --- START / TITLE SCREEN --- */}
                {gameState === 'start' && (
                  <div className="absolute inset-0 z-50 flex items-center justify-center p-8 bg-blue-900 bg-opacity-80 backdrop-blur-sm">
                    <div className="text-center">
                      <h1 className="text-7xl md:text-9xl font-black italic uppercase text-transparent bg-clip-text bg-gradient-to-b from-white via-blue-300 to-blue-600 mb-8 drop-shadow-[0_0_20px_rgba(59,130,246,0.8)]" style={{ fontFamily: 'Impact, sans-serif' }}>
                        DORAEMON VOCAB
                      </h1>
                      <button 
                        onClick={() => setGameState('select')}
                        className="bg-blue-500 text-white text-5xl font-black py-6 px-20 rounded-full hover:bg-blue-400 hover:scale-110 transition-all shadow-[0_0_30px_rgba(59,130,246,1)] uppercase border-4 border-white"
                      >
                        Enter Gadget Pocket
                      </button>
                    </div>
                  </div>
                )}

                {/* --- CHARACTER SELECTION SCREEN --- */}
                {gameState === 'select' && (
                  <div className="absolute inset-0 z-50 flex flex-col bg-blue-950 bg-opacity-95 text-white overflow-hidden">
                    
                    <div className="flex-shrink-0 p-4 text-center shadow-lg bg-blue-900 bg-opacity-50 z-10 flex flex-col items-center">
                       <h2 className="text-3xl md:text-5xl font-black italic text-yellow-400 uppercase mb-4" style={{ fontFamily: 'Impact' }}>
                         Gadget Pocket: Prepare Your Teams
                       </h2>
                       
                       <label className="cursor-pointer bg-amber-600 hover:bg-amber-500 text-white px-4 py-2 rounded-full text-sm md:text-lg font-bold border-2 border-white shadow transition-colors flex items-center gap-3">
                          <span>📷 Upload Custom Dorayaki for Shower</span>
                          <img 
                            src={showerImg} 
                            className="w-8 h-8 object-cover rounded-full bg-white border border-gray-300" 
                            onError={(e) => { e.target.src = 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><text y=".9em" font-size="90">🥞</text></svg>' }}
                            alt="Preview"
                          />
                          <input type="file" accept="image/*" className="hidden" onChange={handleShowerUpload} />
                       </label>
                    </div>

                    <div className="flex-1 overflow-y-auto p-2 md:p-4 flex items-start justify-center pb-32 mt-4">
                      <div className="flex flex-row flex-wrap md:flex-nowrap justify-center gap-2 md:gap-4 w-full max-w-[100%] px-2">
                        {teams.map((team, idx) => (
                          <div key={team.id} className="bg-blue-800 border-4 border-blue-400 rounded-2xl p-2 md:p-4 flex flex-col items-center flex-1 min-w-[140px] max-w-[240px] shadow-[0_0_20px_rgba(59,130,246,0.5)]">
                            <h3 className="text-sm md:text-xl font-bold mb-2 text-blue-200">Team {idx + 1}</h3>
                            <div className="w-16 h-16 md:w-28 md:h-28 anim-float mb-4 relative">
                              <CharacterImage character={team} />
                            </div>
                            
                            <label className="mb-4 cursor-pointer bg-blue-500 hover:bg-blue-400 text-white px-2 md:px-4 py-2 rounded-full text-xs md:text-sm font-bold border-2 border-white shadow transition-colors w-full text-center whitespace-nowrap overflow-hidden text-ellipsis">
                              📷 Upload Player
                              <input type="file" accept="image/*" className="hidden" onChange={(e) => handleFileUpload(team.id, e)} />
                            </label>

                            <input 
                              type="text" 
                              value={team.name} 
                              onChange={(e) => handleNameChange(team.id, e.target.value)}
                              className="w-full text-center text-sm md:text-xl font-black text-blue-900 rounded p-1 md:p-2 border-2 border-blue-300 focus:outline-none focus:border-yellow-400"
                              placeholder="Team Name"
                            />
                          </div>
                        ))}
                      </div>
                    </div>
                    <div className="flex-shrink-0 p-4 bg-blue-900 border-t-4 border-blue-400 flex justify-center items-center shadow-[0_-10px_20px_rgba(0,0,0,0.5)] z-20">
                      <button 
                        onClick={startGame} 
                        className="bg-yellow-400 text-blue-900 text-3xl md:text-5xl font-black py-4 px-20 rounded-full hover:bg-yellow-300 hover:scale-105 transition-all shadow-[0_0_30px_rgba(250,204,21,0.8)] uppercase border-4 border-white"
                      >
                        Let's Go!
                      </button>
                    </div>
                  </div>
                )}

                {/* --- MAIN GAME UI --- */}
                {gameState === 'playing' && (
                  <React.Fragment>
                    {/* DORAYAKI SHOWER OVERLAY */}
                    {showShower && <DorayakiShower imageSrc={showerImg} />}

                    {/* GLOBAL TIMER UI */}
                    <div className="absolute top-4 right-4 md:top-6 md:right-8 z-[70] bg-white border-4 border-blue-500 rounded-full w-16 h-16 md:w-24 md:h-24 flex items-center justify-center shadow-[0_0_20px_rgba(59,130,246,0.6)]">
                      <span className={`text-3xl md:text-5xl font-black ${timeLeft <= 10 ? 'text-red-600 animate-pulse' : 'text-blue-600'}`} style={{ fontFamily: 'Impact' }}>
                        {timeLeft}
                      </span>
                    </div>

                    {/* GET READY PREP OVERLAY */}
                    {prepTime > 0 && (
                      <div className="absolute inset-0 z-[60] flex flex-col items-center justify-center bg-blue-900 bg-opacity-70 backdrop-blur-md">
                         <h2 className="text-6xl md:text-9xl font-black italic text-yellow-400 uppercase animate-bounce drop-shadow-[0_0_20px_rgba(250,204,21,0.8)]" style={{ fontFamily: 'Impact' }}>
                            GET READY!
                         </h2>
                      </div>
                    )}

                    {/* TOP JUMBOTRON / QUESTION */}
                    <div className="h-1/4 flex flex-col items-center justify-center p-4 relative z-10 mt-4">
                      <div className="bg-blue-900 bg-opacity-90 border-4 border-blue-400 rounded-3xl shadow-[0_0_30px_rgba(59,130,246,0.8)] w-full max-w-5xl flex flex-col items-center justify-center p-6 relative overflow-hidden">
                        <div className="absolute top-0 left-0 w-full h-2 bg-gradient-to-r from-blue-500 via-white to-blue-500 opacity-80"></div>
                        {message ? (
                          <h2 className="text-5xl md:text-7xl font-black italic text-yellow-400 uppercase animate-pulse drop-shadow-[0_0_15px_rgba(250,204,21,0.8)] text-center" style={{ fontFamily: 'Impact' }}>
                            {message}
                          </h2>
                        ) : (
                          <div className="text-center w-full px-16">
                            <div className="text-blue-300 font-bold text-xl md:text-2xl tracking-widest uppercase mb-3 animate-pulse">Find this Location</div>
                            <h2 className="text-3xl md:text-5xl lg:text-6xl font-extrabold text-white drop-shadow-lg leading-tight">"{currentQuestion?.clue}"</h2>
                          </div>
                        )}
                      </div>
                    </div>

                    {/* MIDDLE: PLACE CARDS (INFINITE SCROLLING MARQUEE) */}
                    <div className="h-2/4 w-full overflow-hidden relative z-10 mx-auto bg-black bg-opacity-20 shadow-[inset_0_0_20px_rgba(0,0,0,0.5)]">
                       <div className="flex h-full w-max anim-marquee" style={{ animationPlayState: (roundOver || prepTime > 0) ? 'paused' : 'running' }}>
                         {renderCardBlock('block1')}
                         {renderCardBlock('block2')}
                       </div>
                    </div>

                    {/* BOTTOM: TEAM CONTROL DESKS */}
                    <div className="h-[28%] md:h-1/4 w-full bg-blue-950 bg-opacity-95 border-t-4 border-blue-500 flex justify-between items-stretch p-1 md:p-3 shadow-[0_-10px_30px_rgba(59,130,246,0.4)] z-10 backdrop-blur-md overflow-hidden">
                      {teams.map(team => {
                        return (
                          <div key={team.id} className={`flex-1 min-w-0 mx-0.5 md:mx-1 rounded-xl border-2 ${team.locked ? 'border-red-900 bg-blue-950 opacity-60' : 'border-blue-400 bg-blue-800'} flex flex-col items-center p-1 md:p-2 relative transition-all`}>
                            
                            {team.locked && (
                              <div className="absolute inset-0 bg-black bg-opacity-80 z-40 rounded-xl flex items-center justify-center backdrop-blur-sm">
                                <span className="text-red-500 font-black text-lg md:text-4xl transform -rotate-12 border-2 md:border-4 border-red-500 px-1 md:px-3 py-1 rounded bg-black tracking-widest uppercase shadow-[0_0_15px_rgba(239,68,68,1)]">Miss</span>
                              </div>
                            )}

                            <div className="flex w-full items-center justify-between px-1 mb-1 md:mb-2 overflow-hidden">
                              <div className="flex items-center gap-1 md:gap-2 overflow-hidden">
                                <div className="w-8 h-8 md:w-16 md:h-16 anim-float flex-shrink-0">
                                   <CharacterImage character={team} />
                                </div>
                                <div className="flex flex-col hidden md:flex overflow-hidden">
                                  <span className="text-white font-black text-xs md:text-lg uppercase tracking-wider leading-tight truncate">{team.name}</span>
                                </div>
                              </div>
                              <div className="bg-blue-950 px-2 py-1 md:px-4 md:py-2 rounded border-2 border-yellow-400 text-yellow-400 font-mono font-bold text-lg md:text-4xl shadow-[inset_0_0_10px_rgba(250,204,21,0.3)] flex-shrink-0 ml-1">
                                {team.score}
                              </div>
                            </div>

                            <div className="grid grid-cols-4 gap-1 md:gap-2 w-full flex-grow mt-1 px-1">
                              {COLORS.map((color, index) => (
                                <button
                                  key={index}
                                  onClick={() => handleAnswer(team.id, index)}
                                  disabled={team.locked || roundOver || prepTime > 0}
                                  className={`rounded-full ${color} w-full h-full aspect-square border-b-2 md:border-b-4 border-black border-opacity-60 shadow-[0_5px_10px_rgba(0,0,0,0.5)] active:scale-90 active:border-b-0 hover:brightness-125 hover:shadow-[0_0_15px_rgba(255,255,255,0.4)] flex items-center justify-center transition-all ${(roundOver || prepTime > 0) ? 'opacity-50' : 'opacity-100'}`}
                                  style={{ backgroundImage: 'radial-gradient(circle at 30% 30%, rgba(255,255,255,0.4) 5%, transparent 50%)' }}
                                >
                                  <svg viewBox="0 0 100 100" className="w-full h-full opacity-40 p-1 pointer-events-none">
                                    <circle cx="50" cy="50" r="45" fill="none" stroke="white" strokeWidth="6" />
                                    <circle cx="50" cy="50" r="20" fill="none" stroke="white" strokeWidth="4" />
                                  </svg>
                                </button>
                              ))}
                            </div>
                          </div>
                        )
                      })}
                    </div>
                  </React.Fragment>
                )}

                {/* --- GAME OVER / LEADERBOARD SCREEN --- */}
                {gameState === 'gameOver' && (
                  <div className="absolute inset-0 z-[100] flex flex-col items-center justify-center p-4 bg-blue-950 bg-opacity-95 text-white overflow-y-auto">
                    <h1 className="text-6xl md:text-8xl font-black italic text-yellow-400 uppercase mb-8 drop-shadow-[0_0_20px_rgba(250,204,21,0.8)]" style={{ fontFamily: 'Impact' }}>
                       TIME'S UP!
                    </h1>
                    
                    <div className="flex flex-wrap justify-center items-end gap-4 md:gap-6 w-full max-w-6xl mb-12">
                       {[...teams].sort((a, b) => b.score - a.score).map((team, idx) => (
                          <div key={team.id} className={`bg-blue-800 border-4 ${idx === 0 ? 'border-yellow-400 scale-110 shadow-[0_0_30px_rgba(250,204,21,0.8)] z-10' : 'border-blue-400 shadow-lg'} rounded-2xl p-4 flex flex-col items-center min-w-[140px] max-w-[200px] transition-all`}>
                             <div className={`font-black mb-2 ${idx === 0 ? 'text-4xl text-yellow-400' : 'text-2xl text-white'}`}>
                                {idx === 0 ? '🏆 1ST' : `#${idx + 1}`}
                             </div>
                             <div className="w-20 h-20 md:w-28 md:h-28 relative mb-3">
                                <CharacterImage character={team} />
                             </div>
                             <div className="text-lg md:text-xl font-bold truncate w-full text-center text-blue-200">{team.name}</div>
                             <div className="text-4xl md:text-5xl font-black text-white mt-1">{team.score}</div>
                          </div>
                       ))}
                    </div>
                    
                    <button 
                       onClick={() => setGameState('select')} 
                       className="bg-yellow-400 text-blue-900 text-3xl md:text-5xl font-black py-4 px-16 rounded-full hover:bg-yellow-300 hover:scale-105 transition-all shadow-[0_0_30px_rgba(250,204,21,0.8)] uppercase border-4 border-white"
                    >
                      Play Again
                    </button>
                  </div>
                )}

              </div>
            </React.Fragment>
          );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
