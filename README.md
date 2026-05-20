import React, { useState, useRef } from 'react';
import { MapPin, X, Upload, Edit3, Eye, Plus, Trash2, Search, Image as ImageIcon } from 'lucide-react';

export default function App() {
  // --- STATE ---
  const [mapImage, setMapImage] = useState('Kaart Arnhem vroegah.jpg'); 
  const [mode, setMode] = useState('edit'); 
  const [activePopup, setActivePopup] = useState(null); 
  const [draggingPin, setDraggingPin] = useState(null);

  // Standaard pinnen (nu met een 'image' veld)
  const [pins, setPins] = useState([
    {
      id: 1,
      x: 50,
      y: 50,
      title: "De Plaats Delict: Helstraat",
      description: "Hier in de modder werd het breekbare, luxe parfumflesje gevonden. Hoe komt een object van dergelijke waarde in deze straat terecht?",
      color: "text-red-600",
      image: null
    },
    {
      id: 2,
      x: 30,
      y: 40,
      title: "De Latijnse School",
      description: "Vlakbij lag de Latijnse School. Was het een welgestelde docent die het verloor? Of een rebelse leerling met een geheim die het stiekem had meegenomen?",
      color: "text-blue-700",
      image: null
    },
    {
      id: 3,
      x: 70,
      y: 60,
      title: "De Koopmansfamilie",
      description: "In de archieven vond ik deze familie die in de straat woonde. Zij hadden de middelen voor zo'n flesje. Was het van de vrouw des huizes die haastig langs liep?",
      color: "text-amber-700",
      image: null
    },
    {
      id: 4,
      x: 80,
      y: 20,
      title: "Onderzoek bij Rozet",
      description: "Een parfumflesje was een luxueus artikel. Dit onderzoek naar de herkomst leidde uiteindelijk via een stamboom-puzzel tot een publiekslezing bij Rozet.",
      color: "text-slate-800",
      image: null
    }
  ]);

  const mapContainerRef = useRef(null);

  // --- FUNCTIES ---

  const handleMainMapUpload = (e) => {
    const file = e.target.files[0];
    if (file) {
      const imageUrl = URL.createObjectURL(file);
      setMapImage(imageUrl);
    }
  };

  // Nieuwe functie: Upload een afbeelding voor een specifieke pin
  const handlePinImageUpload = (id, e) => {
    const file = e.target.files[0];
    if (file) {
      const imageUrl = URL.createObjectURL(file);
      setPins(pins.map(pin => pin.id === id ? { ...pin, image: imageUrl } : pin));
    }
  };

  const removePinImage = (id) => {
    setPins(pins.map(pin => pin.id === id ? { ...pin, image: null } : pin));
  };

  const handlePointerDown = (id, e) => {
    if (mode !== 'edit') {
      setActivePopup(pins.find(p => p.id === id));
      return;
    }
    e.preventDefault();
    setDraggingPin(id);
  };

  const handlePointerMove = (e) => {
    if (mode === 'edit' && draggingPin !== null && mapContainerRef.current) {
      const rect = mapContainerRef.current.getBoundingClientRect();
      let newX = ((e.clientX - rect.left) / rect.width) * 100;
      let newY = ((e.clientY - rect.top) / rect.height) * 100;
      newX = Math.max(0, Math.min(100, newX));
      newY = Math.max(0, Math.min(100, newY));
      setPins(pins.map(pin => pin.id === draggingPin ? { ...pin, x: newX, y: newY } : pin));
    }
  };

  const handlePointerUp = () => {
    setDraggingPin(null);
  };

  const addPin = () => {
    const newPin = {
      id: Date.now(),
      x: 50,
      y: 50,
      title: "Nieuw Onderdeel",
      description: "Pas deze tekst aan...",
      color: "text-slate-800",
      image: null
    };
    setPins([...pins, newPin]);
  };

  const deletePin = (id) => {
    setPins(pins.filter(pin => pin.id !== id));
  };

  const updatePinInfo = (id, field, value) => {
    setPins(pins.map(pin => pin.id === id ? { ...pin, [field]: value } : pin));
  };

  // --- RENDER ---

  return (
    <div 
      className="min-h-screen bg-[#f4f1ea] text-slate-800 font-sans flex flex-col"
      onPointerMove={handlePointerMove}
      onPointerUp={handlePointerUp}
      onPointerLeave={handlePointerUp}
    >
      {/* HEADER */}
      <header className="bg-[#2c3e50] text-[#f4f1ea] p-4 shadow-md flex justify-between items-center z-10 relative">
        <div className="flex items-center gap-3">
          <Search className="w-6 h-6 text-amber-400" />
          <h1 className="text-xl md:text-2xl font-serif font-bold tracking-wide">
            Het Parfumflesje van de Helstraat
          </h1>
        </div>
        
        {/* Toggle tussen Bewerk en Presentatie modus */}
        <button 
          onClick={() => {
            setMode(mode === 'edit' ? 'presentation' : 'edit');
            setActivePopup(null);
          }}
          className={`flex items-center gap-2 px-4 py-2 rounded-md font-semibold transition-colors ${
            mode === 'edit' 
              ? 'bg-amber-500 hover:bg-amber-600 text-slate-900' 
              : 'bg-slate-600 hover:bg-slate-500 text-white'
          }`}
        >
          {mode === 'edit' ? <Eye className="w-4 h-4" /> : <Edit3 className="w-4 h-4" />}
          {mode === 'edit' ? 'Start Presentatie' : 'Bewerk Modus'}
        </button>
      </header>

      {/* MAIN CONTENT */}
      <main className="flex-grow flex flex-col md:flex-row overflow-hidden relative cursor-default">
        
        {/* KAART GEDEELTE */}
        <div className="flex-grow relative bg-[#e3dfd5] flex items-center justify-center p-4 overflow-hidden shadow-inner">
          
          {!mapImage ? (
            <div className="text-center bg-white p-8 rounded-lg shadow-lg max-w-md border-2 border-dashed border-slate-400">
              <MapPin className="w-16 h-16 mx-auto text-slate-400 mb-4" />
              <h2 className="text-xl font-bold mb-2">Upload je Historische Kaart</h2>
              <label className="cursor-pointer bg-[#2c3e50] text-white px-6 py-3 rounded-md inline-flex items-center gap-2">
                <Upload className="w-5 h-5" />
                Selecteer Hoofdkaart
                <input type="file" accept="image/*" onChange={handleMainMapUpload} className="hidden" />
              </label>
            </div>
          ) : (
            <div 
              ref={mapContainerRef}
              className="relative shadow-2xl rounded-sm border-8 border-white inline-block max-w-full max-h-full"
              style={{ touchAction: 'none' }} 
            >
              <img 
                src={mapImage} 
                alt="Historische Kaart Arnhem" 
                className="max-w-full max-h-[85vh] object-contain select-none"
                draggable="false"
                onError={() => { if (mapImage === 'Kaart Arnhem vroegah.jpg') setMapImage(null); }}
              />

              {/* Render de Pinnen */}
              {pins.map((pin) => (
                <div
                  key={pin.id}
                  onPointerDown={(e) => handlePointerDown(pin.id, e)}
                  className="absolute transform -translate-x-1/2 -translate-y-full cursor-pointer group"
                  style={{ left: `${pin.x}%`, top: `${pin.y}%`, zIndex: draggingPin === pin.id ? 50 : 10 }}
                >
                  <MapPin 
                    className={`w-10 h-10 ${pin.color} drop-shadow-md transition-transform ${mode === 'edit' ? 'hover:scale-110' : 'hover:-translate-y-1'}`} 
                    fill={mode === 'edit' ? 'white' : '#fdfbf7'}
                  />
                  {mode === 'presentation' && (
                    <div className="opacity-0 group-hover:opacity-100 absolute bottom-full left-1/2 transform -translate-x-1/2 mb-2 bg-slate-900 text-white text-xs px-3 py-1.5 rounded whitespace-nowrap transition-opacity pointer-events-none shadow-lg">
                      {pin.title}
                    </div>
                  )}
                </div>
              ))}
            </div>
          )}
        </div>

        {/* ZIJBALK: BEWERK MODUS */}
        {mode === 'edit' && mapImage && (
          <div className="w-full md:w-80 bg-white border-l border-slate-200 p-4 overflow-y-auto flex-shrink-0 shadow-[-4px_0_15px_rgba(0,0,0,0.05)] z-20">
            <h2 className="text-lg font-bold mb-4 flex items-center gap-2 border-b pb-2">
              <Edit3 className="w-5 h-5" /> Instellingen
            </h2>
            
            <div className="mb-6">
              <label className="cursor-pointer bg-slate-200 hover:bg-slate-300 text-slate-700 w-full py-2 rounded text-sm font-medium transition-colors flex justify-center items-center gap-2">
                <Upload className="w-4 h-4" /> Andere Kaart Inladen
                <input type="file" accept="image/*" onChange={handleMainMapUpload} className="hidden" />
              </label>
            </div>

            <div className="space-y-6">
              {pins.map((pin, index) => (
                <div key={pin.id} className="bg-slate-50 p-3 rounded border border-slate-200 relative">
                  <div className="flex justify-between items-center mb-2">
                    <span className={`font-bold text-sm ${pin.color}`}>Pin {index + 1}</span>
                    <button onClick={() => deletePin(pin.id)} className="text-red-400 hover:text-red-600" title="Verwijder onderdeel">
                      <Trash2 className="w-4 h-4" />
                    </button>
                  </div>
                  
                  <input
                    type="text"
                    value={pin.title}
                    onChange={(e) => updatePinInfo(pin.id, 'title', e.target.value)}
                    className="w-full mb-2 p-1 text-sm border border-slate-300 rounded font-semibold"
                    placeholder="Titel..."
                  />
                  <textarea
                    value={pin.description}
                    onChange={(e) => updatePinInfo(pin.id, 'description', e.target.value)}
                    className="w-full mb-2 p-1 text-sm border border-slate-300 rounded h-20 resize-none"
                    placeholder="Beschrijving..."
                  />

                  {/* FOTO UPLOAD VOOR PIN */}
                  <div className="mb-3 border-t border-slate-200 pt-2">
                    {pin.image ? (
                      <div className="relative">
                        <img src={pin.image} alt="Preview" className="w-full h-24 object-cover rounded border border-slate-300" />
                        <button 
                          onClick={() => removePinImage(pin.id)}
                          className="absolute top-1 right-1 bg-red-500 text-white rounded-full p-1 hover:bg-red-600"
                          title="Verwijder foto"
                        >
                          <X className="w-3 h-3" />
                        </button>
                      </div>
                    ) : (
                      <label className="cursor-pointer flex items-center justify-center gap-2 w-full p-2 border-2 border-dashed border-slate-300 rounded text-xs text-slate-500 hover:bg-slate-100 transition-colors">
                        <ImageIcon className="w-4 h-4" /> Foto toevoegen
                        <input 
                          type="file" 
                          accept="image/*" 
                          onChange={(e) => handlePinImageUpload(pin.id, e)} 
                          className="hidden" 
                        />
                      </label>
                    )}
                  </div>

                  <div className="mt-2 flex gap-2">
                    {['text-red-600', 'text-blue-700', 'text-amber-700', 'text-purple-700', 'text-green-700', 'text-slate-800'].map(color => (
                      <button
                        key={color}
                        onClick={() => updatePinInfo(pin.id, 'color', color)}
                        className={`w-5 h-5 rounded-full bg-current ${color} ${pin.color === color ? 'ring-2 ring-offset-1 ring-slate-400' : 'opacity-50 hover:opacity-100'}`}
                      />
                    ))}
                  </div>
                </div>
              ))}
            </div>

            <button 
              onClick={addPin}
              className="mt-6 w-full flex items-center justify-center gap-2 bg-slate-200 hover:bg-slate-300 text-slate-800 py-2 rounded font-medium transition-colors"
            >
              <Plus className="w-4 h-4" /> Extra Pin Toevoegen
            </button>
          </div>
        )}
      </main>

      {/* POP-UP / MODAL (PRESENTATIE MODUS) */}
      {mode === 'presentation' && activePopup && (
        <div className="absolute inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm">
          <div className="bg-[#fdfbf7] w-full max-w-2xl rounded-md shadow-2xl overflow-hidden animate-in fade-in zoom-in duration-200 relative border border-[#e2d5b8]">
            
            <div className="absolute top-0 left-0 w-full h-2 bg-[#2c3e50]"></div>

            <div className="p-8 relative">
              <button 
                onClick={() => setActivePopup(null)}
                className="absolute top-4 right-4 text-slate-400 hover:text-slate-800 transition-colors bg-white rounded-full p-1 shadow-sm"
              >
                <X className="w-6 h-6" />
              </button>

              <div className="flex items-center gap-3 mb-6">
                <MapPin className={`w-8 h-8 ${activePopup.color}`} fill="currentColor" fillOpacity="0.2"/>
                <h3 className="text-3xl font-serif font-bold text-slate-900 border-b-2 border-slate-200 pb-2 flex-grow">
                  {activePopup.title}
                </h3>
              </div>
              
              <div className="flex flex-col gap-6">
                {/* ALS ER EEN FOTO IS GEÜPLOAD, TOON DEZE HIER */}
                {activePopup.image && (
                  <div className="w-full bg-white p-2 border border-slate-200 rounded shadow-sm">
                    <img 
                      src={activePopup.image} 
                      alt={activePopup.title} 
                      className="w-full max-h-64 object-contain rounded"
                    />
                  </div>
                )}

                <div className="prose prose-slate max-w-none">
                  <p className="text-lg leading-relaxed text-slate-700 font-medium whitespace-pre-wrap">
                    {activePopup.description}
                  </p>
                </div>
              </div>

              <div className="mt-8 pt-4 border-t border-slate-200 flex justify-between items-center text-xs text-slate-400 font-mono uppercase tracking-wider">
                <span>Dossier #1750-1825</span>
                <span>Archief Arnhem</span>
              </div>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
