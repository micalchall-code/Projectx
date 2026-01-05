import React, { useState, useEffect } from 'react';
import ProductForm from './components/ProductForm';
import ContentDisplay from './components/ContentDisplay';
import ImageGenerator from './components/ImageGenerator';
import HistoryList from './components/HistoryList';
import ActivityLog from './components/ActivityLog';
import GmailSettings from './components/GmailSettings';
import { MarketingPlan, LoadingState, HistoryItem } from './types';
import { generateMarketingContent } from './geminiService';

const App: React.FC = () => {
  const [marketingPlan, setMarketingPlan] = useState<MarketingPlan | null>(null);
  const [loadingState, setLoadingState] = useState<LoadingState>(LoadingState.IDLE);
  const [currentProduct, setCurrentProduct] = useState<string>('');
  const [currentLink, setCurrentLink] = useState<string>('');
  const [currentImageUrl, setCurrentImageUrl] = useState<string | undefined>(undefined);
  const [currentLogs, setCurrentLogs] = useState<string[]>([]);
  const [error, setError] = useState<string | null>(null);
  const [history, setHistory] = useState<HistoryItem[]>([]);
  const [isHistoryOpen, setIsHistoryOpen] = useState(false);
  const [isGmailModalOpen, setIsGmailModalOpen] = useState(false);
  const [userEmail, setUserEmail] = useState<string>('');

  const profileImg = "https://files.oaiusercontent.com/file-S40gK9g1P0h2hFmE68r0Nf1u?se=2025-02-21T11%3A32%3A58Z&sp=r&sv=2024-08-04&sr=b&rscc=max-age%3D604800%2C%20private%2C%20immutable&rscd=attachment%3B%20filename%3D172f3a61-827c-486d-8b63-d5d111003f5f.webp&sig=G04l88g7UuE3YI8eU99pB8Xm6YI0uU0mY0iU0mY0iU0%3D";

  useEffect(() => {
    const saved = localStorage.getItem('rehit_pro_v1_history');
    const savedEmail = localStorage.getItem('rehit_user_gmail');
    if (saved) {
      try { setHistory(JSON.parse(saved)); } catch (e) { console.error(e); }
    }
    if (savedEmail) {
      setUserEmail(savedEmail);
    }
  }, []);

  const addLog = (msg: string) => {
    setCurrentLogs(prev => [`[${new Date().toLocaleTimeString()}] ${msg}`, ...prev]);
  };

  const handleFormSubmit = async (input: { 
    link?: string; 
    image?: { data: string; mimeType: string };
    price?: string;
    phone?: string;
  }) => {
    setLoadingState(LoadingState.ANALYZING);
    setError(null);
    setMarketingPlan(null);
    setCurrentImageUrl(undefined);
    setCurrentLink(input.link || 'Input Item');
    setCurrentLogs([]);

    try {
      addLog("အချက်အလက်များကို AI ဖြင့် စတင် Analyze လုပ်နေပါသည်...");
      const plan = await generateMarketingContent(input);
      setMarketingPlan(plan);
      setCurrentProduct(plan.productName);
      addLog("Content ဖန်တီးမှု ပြီးမြောက်ပါပြီ။");
      
      if (userEmail) {
        setLoadingState(LoadingState.SENDING_GMAIL);
        addLog(`Gmail Report (${userEmail}) သို့ ပို့နေပါသည်...`);
        await new Promise(resolve => setTimeout(resolve, 1000));
      }

      setLoadingState(LoadingState.COMPLETE);
      addLog("လုပ်ဆောင်ချက်များ အားလုံး ပြီးမြောက်ပါပြီ။");
      saveToHistory(plan, input.link || 'Image Upload', undefined);
    } catch (err: any) {
      setError(err.message);
      setLoadingState(LoadingState.ERROR);
      addLog(`Error: ${err.message}`);
    }
  };

  const saveToHistory = (plan: MarketingPlan, link: string, imageUrl?: string) => {
    const newItem: HistoryItem = {
      id: Math.random().toString(36).substr(2, 9),
      timestamp: Date.now(),
      productName: plan.productName,
      productLink: link,
      plan: plan,
      imageUrl: imageUrl,
      status: 'DRAFT',
      logs: currentLogs
    };
    const updated = [newItem, ...history.slice(0, 19)];
    setHistory(updated);
    localStorage.setItem('rehit_pro_v1_history', JSON.stringify(updated));
  };

  return (
    <div className="min-h-screen flex flex-col font-sans">
      <nav className="bg-white/80 backdrop-blur-xl border-b px-8 py-4 flex justify-between items-center sticky top-0 z-[100] shadow-sm">
        <div className="flex items-center gap-4">
          <img src={profileImg} alt="Profile" className="w-12 h-12 rounded-2xl object-cover border-2 border-white shadow-xl animate-float-gold" />
          <div className="flex items-baseline gap-1">
            <h1 className="gold-3d-text text-4xl" data-text="ဈေးသည်">ဈေးသည်</h1>
            <span className="text-orange-600 font-black text-xl italic">AI</span>
          </div>
        </div>
        <div className="flex gap-3">
          <button onClick={() => setIsGmailModalOpen(true)} className={`w-10 h-10 rounded-2xl border flex items-center justify-center transition-all ${userEmail ? 'bg-red-50 text-red-500 border-red-200' : 'bg-white text-slate-400'}`}>
            <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path d="M2.003 5.884L10 9.882l7.997-3.998A2 2 0 0016 4H4a2 2 0 00-1.997 1.884z" /><path d="M18 8.118l-8 4-8-4V14a2 2 0 002 2h12a2 2 0 002-2V8.118z" /></svg>
          </button>
          <button onClick={() => setIsHistoryOpen(true)} className="w-10 h-10 rounded-2xl border bg-white flex items-center justify-center text-slate-600 shadow-sm">
            <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
          </button>
        </div>
      </nav>

      <main className="max-w-7xl mx-auto w-full p-6 md:p-8 grid grid-cols-1 lg:grid-cols-12 gap-8 relative">
        <div className="lg:col-span-4 space-y-6">
          <div className="bg-[#1877F2] p-8 rounded-[2.5rem] shadow-2xl border-4 border-orange-500">
             <ProductForm onSubmit={handleFormSubmit} isLoading={loadingState === LoadingState.ANALYZING || loadingState === LoadingState.SENDING_GMAIL} />
          </div>
          <ActivityLog logs={currentLogs} />
        </div>

        <div className="lg:col-span-8 space-y-8">
          {error && <div className="bg-red-50 border border-red-200 text-red-600 p-5 rounded-3xl font-bold animate-shake shadow-lg">⚠️ {error}</div>}
          
          {marketingPlan ? (
            <div className="space-y-8 animate-in fade-in duration-700">
              <ContentDisplay plan={marketingPlan} />
              <ImageGenerator 
                productName={currentProduct} 
                description={marketingPlan.postCaption.substring(0, 100) + '...'}
                onImageGenerated={(url) => {
                   setCurrentImageUrl(url);
                   saveToHistory(marketingPlan, currentLink, url);
                }} 
                initialImageUrl={currentImageUrl} 
              />
            </div>
          ) : (
            <div className="h-full min-h-[500px] bg-white/40 backdrop-blur-lg rounded-[4rem] border-2 border-dashed border-white flex flex-col items-center justify-center p-12 text-center shadow-inner">
               <div className="w-20 h-20 bg-white rounded-3xl flex items-center justify-center mb-6 text-[#1877F2] border-4 border-orange-500 shadow-xl">
                 <svg xmlns="http://www.w3.org/2000/svg" className="h-10 w-10" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 4v16m8-8H4" /></svg>
               </div>
               <h2 className="gold-3d-text text-4xl mb-4">ဈေးသည် AI</h2>
               <p className="text-slate-600 max-w-sm font-medium">Link သို့မဟုတ် ဓာတ်ပုံထည့်သွင်းပြီး အရောင်းပိုစ့်များကို အမြန်ဆုံး ဖန်တီးလိုက်ပါ။</p>
            </div>
          )}
        </div>
      </main>

      {isGmailModalOpen && <GmailSettings onEmailChange={setUserEmail} onClose={() => setIsGmailModalOpen(false)} />}
      {isHistoryOpen && (
        <div className="fixed inset-0 z-[200] flex justify-end">
          <div className="absolute inset-0 bg-slate-900/40 backdrop-blur-sm" onClick={() => setIsHistoryOpen(false)}></div>
          <div className="w-full max-w-sm bg-white h-full relative z-[201] shadow-2xl p-8 animate-slide-left overflow-y-auto">
             <div className="flex justify-between items-center mb-8">
                <h3 className="text-2xl font-black">Archive</h3>
                <button onClick={() => setIsHistoryOpen(false)}>✕</button>
             </div>
             <HistoryList history={history} onSelectItem={(item) => { setMarketingPlan(item.plan); setIsHistoryOpen(false); }} onClear={() => setHistory([])} />
          </div>
        </div>
      )}
    </div>
  );
};

export default App;
