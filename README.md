<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Crypto Sentinel Pro - Dashboard</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.5/babel.min.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    * { box-sizing: border-box; }
    body { margin: 0; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; }
    @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.5; } }
    .animate-pulse { animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
  </style>
</head>
<body>
  <div id="root"></div>
  
  <script type="text/babel">
    const { useState, useEffect, useCallback } = React;

    const CONFIG = {
      REFRESH_RATE: 5000,
      FEAR_GREED_API: 'https://api.alternative.me/fng/',
    };

    // Classifications
    const getFearGreedClassification = (value) => {
      if (value <= 24) return { label: 'Extreme Fear', labelFr: 'PEUR EXTRÊME', color: '#ea3943', bgColor: 'rgba(234, 57, 67, 0.15)', emoji: '😱', description: 'Capitulation - Opportunité contrarian potentielle', action: 'Potentiel ACHAT' };
      if (value <= 44) return { label: 'Fear', labelFr: 'PEUR', color: '#ea8c00', bgColor: 'rgba(234, 140, 0, 0.15)', emoji: '😰', description: 'Sentiment négatif dominant', action: 'Observer' };
      if (value <= 55) return { label: 'Neutral', labelFr: 'NEUTRE', color: '#c9b003', bgColor: 'rgba(201, 176, 3, 0.15)', emoji: '😐', description: 'Marché indécis', action: 'Attendre' };
      if (value <= 74) return { label: 'Greed', labelFr: 'AVIDITÉ', color: '#93d900', bgColor: 'rgba(147, 217, 0, 0.15)', emoji: '😊', description: 'Optimisme croissant - FOMO', action: 'Prudence' };
      return { label: 'Extreme Greed', labelFr: 'AVIDITÉ EXTRÊME', color: '#16c784', bgColor: 'rgba(22, 199, 132, 0.15)', emoji: '🤑', description: 'Euphorie - Correction probable', action: 'Prendre profits' };
    };

    const getOpportunityClassification = (value) => {
      if (value <= 20) return { label: 'ÉVITER', color: '#ea3943', bgColor: 'rgba(234, 57, 67, 0.15)', emoji: '🚫', description: 'Conditions défavorables' };
      if (value <= 40) return { label: 'PRUDENCE', color: '#ea8c00', bgColor: 'rgba(234, 140, 0, 0.15)', emoji: '⚠️', description: 'Attendre meilleurs signaux' };
      if (value <= 60) return { label: 'NEUTRE', color: '#c9b003', bgColor: 'rgba(201, 176, 3, 0.15)', emoji: '😐', description: 'Conditions moyennes' };
      if (value <= 80) return { label: 'FAVORABLE', color: '#93d900', bgColor: 'rgba(147, 217, 0, 0.15)', emoji: '✅', description: 'Bonnes conditions' };
      return { label: 'EXCELLENT', color: '#16c784', bgColor: 'rgba(22, 199, 132, 0.15)', emoji: '🎯', description: 'Conditions optimales' };
    };

    // Data generators
    const generateRealisticFearGreedHistory = () => {
      const data = [];
      const now = Date.now();
      const dayMs = 24 * 60 * 60 * 1000;
      let value = 50;
      for (let i = 730; i >= 0; i--) {
        const timestamp = now - (i * dayMs);
        const trend = Math.sin(i / 60) * 20;
        const noise = (Math.random() - 0.5) * 15;
        value = value * 0.95 + (50 + trend + noise) * 0.05;
        value = Math.max(5, Math.min(95, value));
        data.push({ timestamp: Math.floor(timestamp / 1000), value: Math.round(value), date: new Date(timestamp).toISOString().split('T')[0] });
      }
      data[data.length - 1].value = 26;
      data[data.length - 2].value = 28;
      data[data.length - 3].value = 25;
      data[data.length - 7].value = 32;
      return data;
    };

    const generateBtcPriceHistory = (fearGreedData) => {
      return fearGreedData.map((fg) => {
        const basePrice = 60000 + (fg.value - 50) * 800;
        const noise = (Math.random() - 0.5) * 5000;
        return { timestamp: fg.timestamp, price: Math.max(20000, Math.min(150000, basePrice + noise)) };
      });
    };

    // Fear & Greed Index Component
    const FearGreedIndex = ({ currentValue, historicalData, btcPriceData }) => {
      const [timeRange, setTimeRange] = useState('1y');
      const current = getFearGreedClassification(currentValue);
      
      const yesterday = historicalData[historicalData.length - 2]?.value || 0;
      const lastWeek = historicalData[historicalData.length - 8]?.value || 0;
      const lastMonth = historicalData[historicalData.length - 31]?.value || 0;
      
      const yearData = historicalData.slice(-365);
      const yearlyHigh = Math.max(...yearData.map(d => d.value));
      const yearlyLow = Math.min(...yearData.map(d => d.value));
      
      const getFilteredData = () => {
        switch(timeRange) {
          case '30d': return historicalData.slice(-30);
          case '1y': return historicalData.slice(-365);
          default: return historicalData;
        }
      };
      
      const filteredData = getFilteredData();
      const filteredBtc = btcPriceData.slice(-(timeRange === '30d' ? 30 : timeRange === '1y' ? 365 : btcPriceData.length));
      
      return (
        <div className="bg-[#0f2744] rounded-2xl border border-[#1e3a5f] overflow-hidden">
          <div className="grid grid-cols-1 lg:grid-cols-3">
            <div className="p-6 border-b lg:border-b-0 lg:border-r border-[#1e3a5f]">
              <h3 className="text-lg font-bold text-white mb-1">CMC Crypto Fear and Greed Index</h3>
              <p className="text-xs text-gray-500 mb-4">Source: Alternative.me / CoinMarketCap</p>
              
              <div className="flex justify-center mb-4">
                <svg width="200" height="120" viewBox="0 0 200 120">
                  <defs>
                    <linearGradient id="fgArc" x1="0%" y1="0%" x2="100%" y2="0%">
                      <stop offset="0%" stopColor="#ea3943" />
                      <stop offset="25%" stopColor="#ea8c00" />
                      <stop offset="50%" stopColor="#c9b003" />
                      <stop offset="75%" stopColor="#93d900" />
                      <stop offset="100%" stopColor="#16c784" />
                    </linearGradient>
                  </defs>
                  <path d="M 20 100 A 80 80 0 0 1 180 100" fill="none" stroke="#1e3a5f" strokeWidth="16" strokeLinecap="round" />
                  <path d="M 20 100 A 80 80 0 0 1 180 100" fill="none" stroke="url(#fgArc)" strokeWidth="16" strokeLinecap="round" />
                  <g style={{ transform: `rotate(${-90 + (currentValue / 100) * 180}deg)`, transformOrigin: '100px 100px', transition: 'transform 1s ease-out' }}>
                    <line x1="100" y1="100" x2="100" y2="35" stroke={current.color} strokeWidth="3" strokeLinecap="round" />
                    <circle cx="100" cy="100" r="8" fill={current.color} />
                    <circle cx="100" cy="100" r="4" fill="#0f2744" />
                  </g>
                  <text x="100" y="85" textAnchor="middle" fill={current.color} fontSize="36" fontWeight="bold">{currentValue}</text>
                </svg>
              </div>
              
              <div className="text-center mb-6">
                <div className="flex items-center justify-center gap-2 mb-1">
                  <span className="text-3xl">{current.emoji}</span>
                  <span className="text-xl font-bold" style={{ color: current.color }}>{current.label}</span>
                </div>
                <p className="text-xs text-gray-400 max-w-[200px] mx-auto">{current.description}</p>
              </div>
              
              <div className="space-y-2 mb-4">
                <h4 className="text-sm font-semibold text-gray-300">Historical Values</h4>
                {[{ label: 'Yesterday', value: yesterday }, { label: 'Last Week', value: lastWeek }, { label: 'Last Month', value: lastMonth }].map((item, idx) => {
                  const itemClass = getFearGreedClassification(item.value);
                  return (
                    <div key={idx} className="flex items-center justify-between">
                      <span className="text-sm text-gray-400">{item.label}</span>
                      <span className="px-3 py-1 rounded-full text-xs font-semibold" style={{ backgroundColor: itemClass.bgColor, color: itemClass.color, border: `1px solid ${itemClass.color}30` }}>
                        {itemClass.label} - {item.value}
                      </span>
                    </div>
                  );
                })}
              </div>
              
              <div className="space-y-2 pt-4 border-t border-[#1e3a5f]">
                <h4 className="text-sm font-semibold text-gray-300">Yearly High and Low</h4>
                <div className="flex items-center justify-between">
                  <span className="text-xs text-gray-500">Yearly High</span>
                  <span className="px-3 py-1 rounded-full text-xs font-semibold" style={{ backgroundColor: getFearGreedClassification(yearlyHigh).bgColor, color: getFearGreedClassification(yearlyHigh).color }}>
                    {getFearGreedClassification(yearlyHigh).label} - {yearlyHigh}
                  </span>
                </div>
                <div className="flex items-center justify-between">
                  <span className="text-xs text-gray-500">Yearly Low</span>
                  <span className="px-3 py-1 rounded-full text-xs font-semibold" style={{ backgroundColor: getFearGreedClassification(yearlyLow).bgColor, color: getFearGreedClassification(yearlyLow).color }}>
                    {getFearGreedClassification(yearlyLow).label} - {yearlyLow}
                  </span>
                </div>
              </div>
            </div>
            
            <div className="lg:col-span-2 p-6">
              <div className="flex items-center justify-between mb-4">
                <div>
                  <h3 className="text-lg font-bold text-white">Fear and Greed Index Chart</h3>
                  <div className="flex items-center gap-4 mt-2 text-xs">
                    <div className="flex items-center gap-1"><span className="w-3 h-0.5 bg-orange-500 rounded"></span><span className="text-gray-400">Fear & Greed Index</span></div>
                    <div className="flex items-center gap-1"><span className="w-3 h-0.5 bg-gray-500 rounded"></span><span className="text-gray-400">Bitcoin Price</span></div>
                  </div>
                </div>
                <div className="flex gap-1 bg-[#1e3a5f] rounded-lg p-1">
                  {['30d', '1y', 'All'].map(range => (
                    <button key={range} onClick={() => setTimeRange(range)} className={`px-3 py-1 rounded text-xs font-semibold transition-all ${timeRange === range ? 'bg-blue-600 text-white' : 'text-gray-400 hover:text-white'}`}>
                      {range}
                    </button>
                  ))}
                </div>
              </div>
              
              <div className="relative h-72 bg-[#0a1628] rounded-xl overflow-hidden">
                <div className="absolute inset-0 flex flex-col">
                  <div className="flex-1 bg-[#16c784]/5 border-b border-[#16c784]/20"></div>
                  <div className="flex-1 bg-[#93d900]/5 border-b border-[#93d900]/20"></div>
                  <div className="flex-1 bg-[#c9b003]/5 border-b border-[#c9b003]/20"></div>
                  <div className="flex-1 bg-[#ea8c00]/5 border-b border-[#ea8c00]/20"></div>
                  <div className="flex-1 bg-[#ea3943]/5"></div>
                </div>
                
                <div className="absolute right-2 top-0 bottom-0 flex flex-col justify-between py-2 text-[10px] text-gray-500 z-10">
                  <span>100</span>
                  <span className="text-[#16c784]">Extreme Greed</span>
                  <span className="text-[#93d900]">Greed</span>
                  <span className="text-[#c9b003]">Neutral</span>
                  <span className="text-[#ea8c00]">Fear</span>
                  <span className="text-[#ea3943]">Extreme Fear</span>
                  <span>0</span>
                </div>
                
                <svg className="absolute inset-0 w-full h-full" preserveAspectRatio="none" viewBox="0 0 100 100">
                  {[20, 40, 55, 74].map(y => (<line key={y} x1="0" y1={100-y} x2="95" y2={100-y} stroke="rgba(255,255,255,0.05)" strokeWidth="0.2" />))}
                  {filteredData.map((point, idx) => {
                    if (idx === 0) return null;
                    const prev = filteredData[idx - 1];
                    const x1 = ((idx - 1) / (filteredData.length - 1)) * 92;
                    const x2 = (idx / (filteredData.length - 1)) * 92;
                    const y1 = 100 - prev.value;
                    const y2 = 100 - point.value;
                    const avgVal = (point.value + prev.value) / 2;
                    const segColor = getFearGreedClassification(avgVal).color;
                    return <line key={idx} x1={x1} y1={y1} x2={x2} y2={y2} stroke={segColor} strokeWidth="0.4" />;
                  })}
                  {filteredBtc.length > 0 && (
                    <path d={filteredBtc.map((point, idx) => {
                      const x = (idx / (filteredBtc.length - 1)) * 92;
                      const y = 100 - ((point.price - 20000) / (150000 - 20000)) * 100;
                      return `${idx === 0 ? 'M' : 'L'} ${x},${Math.max(0, Math.min(100, y))}`;
                    }).join(' ')} fill="none" stroke="rgba(156, 163, 175, 0.4)" strokeWidth="0.3" />
                  )}
                </svg>
                
                <div className="absolute right-16 px-2 py-1 rounded text-xs font-bold z-20" style={{ top: `${100 - currentValue}%`, backgroundColor: current.color, color: '#000', transform: 'translateY(-50%)' }}>
                  {currentValue}
                </div>
              </div>
              
              <div className="flex justify-between mt-2 text-[10px] text-gray-500 pr-16">
                {timeRange === '30d' && <><span>30 days ago</span><span>20d</span><span>10d</span><span>Today</span></>}
                {timeRange === '1y' && <><span>Feb 2025</span><span>May 2025</span><span>Aug 2025</span><span>Nov 2025</span><span>Feb 2026</span></>}
                {timeRange === 'All' && <><span>Feb 2024</span><span>Aug 2024</span><span>Feb 2025</span><span>Aug 2025</span><span>Feb 2026</span></>}
              </div>
              
              <div className="mt-4 p-3 bg-[#1e3a5f] rounded-lg">
                <p className="text-xs text-gray-400">
                  <strong className="text-gray-300">Comment utiliser:</strong> Un score bas (Fear) indique souvent une opportunité d'achat contrarian. 
                  Un score élevé (Greed) suggère de prendre des profits. "Be fearful when others are greedy, and greedy when others are fearful." - Warren Buffett
                </p>
              </div>
            </div>
          </div>
        </div>
      );
    };

    // Opportunity Index Component
    const OpportunityIndex = ({ score, previousScore, indicators, showDetails, setShowDetails }) => {
      const [animatedScore, setAnimatedScore] = useState(0);
      
      useEffect(() => {
        const timer = setTimeout(() => setAnimatedScore(score), 100);
        return () => clearTimeout(timer);
      }, [score]);
      
      const current = getOpportunityClassification(animatedScore);
      const change = score - previousScore;
      
      return (
        <div className="bg-[#0f2744] rounded-2xl border border-[#1e3a5f] p-6">
          <div className="flex items-center justify-between mb-4">
            <div>
              <div className="flex items-center gap-2">
                <span className="text-2xl">🎯</span>
                <h2 className="text-xl font-bold text-white">Indice d'Opportunité</h2>
              </div>
              <p className="text-xs text-gray-500 mt-1">Basé sur 5 facteurs fondamentaux</p>
            </div>
            <button onClick={() => setShowDetails(!showDetails)} className="px-4 py-2 bg-blue-600/20 border border-blue-600/50 rounded-xl text-blue-400 text-sm font-semibold hover:bg-blue-600/30 transition-colors">
              {showDetails ? 'MASQUER' : 'DÉTAILS'} 📊
            </button>
          </div>
          
          <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
            <div className="flex flex-col items-center">
              <svg width="280" height="160" viewBox="0 0 280 160">
                <defs>
                  <linearGradient id="oppGradient" x1="0%" y1="0%" x2="100%" y2="0%">
                    <stop offset="0%" stopColor="#ea3943" />
                    <stop offset="25%" stopColor="#ea8c00" />
                    <stop offset="50%" stopColor="#c9b003" />
                    <stop offset="75%" stopColor="#93d900" />
                    <stop offset="100%" stopColor="#16c784" />
                  </linearGradient>
                </defs>
                <path d="M 25 140 A 115 115 0 0 1 255 140" fill="none" stroke="#1e3a5f" strokeWidth="18" strokeLinecap="round" />
                <path d="M 25 140 A 115 115 0 0 1 255 140" fill="none" stroke="url(#oppGradient)" strokeWidth="18" strokeLinecap="round" />
                <g style={{ transform: `rotate(${-90 + (previousScore / 100) * 180}deg)`, transformOrigin: '140px 140px' }}>
                  <line x1="140" y1="140" x2="140" y2="40" stroke="rgba(255,255,255,0.3)" strokeWidth="2" strokeDasharray="4 4" />
                </g>
                <g style={{ transform: `rotate(${-90 + (animatedScore / 100) * 180}deg)`, transformOrigin: '140px 140px', transition: 'transform 1.5s ease-out' }}>
                  <polygon points="140,35 135,140 145,140" fill={current.color} />
                  <circle cx="140" cy="140" r="10" fill={current.color} />
                  <circle cx="140" cy="140" r="5" fill="#0f2744" />
                </g>
                <text x="140" y="120" textAnchor="middle" fill={current.color} fontSize="42" fontWeight="bold">{Math.round(animatedScore)}</text>
                <text x="20" y="155" fill="#ea3943" fontSize="10" fontWeight="bold">0</text>
                <text x="250" y="155" fill="#16c784" fontSize="10" fontWeight="bold">100</text>
              </svg>
              
              <div className="text-center mt-2">
                <div className="flex items-center justify-center gap-2">
                  <span className="text-3xl">{current.emoji}</span>
                  <span className="text-2xl font-bold" style={{ color: current.color }}>{current.label}</span>
                </div>
                <p className="text-sm text-gray-400 mt-1">{current.description}</p>
                <div className="flex items-center justify-center gap-4 mt-3">
                  <div className="px-3 py-1 bg-[#1e3a5f] rounded-lg">
                    <div className="text-[10px] text-gray-500">vs 30j</div>
                    <div className={`text-lg font-bold ${change >= 0 ? 'text-emerald-400' : 'text-red-400'}`}>{change >= 0 ? '↑' : '↓'} {Math.abs(change)}</div>
                  </div>
                  <div className="px-3 py-1 bg-[#1e3a5f] rounded-lg">
                    <div className="text-[10px] text-gray-500">Passé</div>
                    <div className="text-lg font-bold text-gray-300">{previousScore}</div>
                  </div>
                </div>
              </div>
            </div>
            
            <div>
              <h3 className="text-sm font-semibold text-gray-300 mb-3">Décomposition du Score</h3>
              <div className="space-y-3">
                {indicators.map((ind, idx) => {
                  const isPositive = ind.current > ind.past;
                  return (
                    <div key={idx} className="bg-[#1e3a5f] rounded-lg p-3">
                      <div className="flex items-center justify-between mb-2">
                        <span className="text-sm text-gray-300">{ind.name}</span>
                        <div className="flex items-center gap-2">
                          <span className="text-xs text-gray-500">{ind.weight}%</span>
                          <span className={`text-sm font-bold ${isPositive ? 'text-emerald-400' : 'text-red-400'}`}>{ind.current}</span>
                        </div>
                      </div>
                      <div className="relative h-2 bg-[#0f2744] rounded-full overflow-hidden">
                        <div className="absolute h-full w-0.5 bg-white/30 z-10" style={{ left: `${ind.past}%` }} />
                        <div className="absolute h-full rounded-full transition-all duration-1000" style={{ width: `${ind.current}%`, backgroundColor: isPositive ? '#22c55e' : '#ef4444' }} />
                      </div>
                      <div className="flex justify-between mt-1 text-[10px]">
                        <span className="text-gray-600">Passé: {ind.past}</span>
                        <span className={isPositive ? 'text-emerald-400' : 'text-red-400'}>{isPositive ? '+' : ''}{ind.current - ind.past}</span>
                      </div>
                    </div>
                  );
                })}
              </div>
            </div>
          </div>
          
          {showDetails && (
            <div className="mt-6 pt-6 border-t border-[#1e3a5f]">
              <h3 className="text-lg font-bold text-white mb-4">📚 Méthodologie de Calcul</h3>
              <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                {[
                  { name: 'USDT Flow', weight: '20%', color: '#3b82f6', items: ['Ratio < 0.90 → 80-100', 'Ratio 0.95-1.05 → 45-65', 'Ratio > 1.15 → 0-25'] },
                  { name: 'Market Cap', weight: '25%', color: '#22c55e', items: ['> 95% ATH → 85-100', '60-80% ATH → 45-65', '< 40% ATH → 0-25'] },
                  { name: 'Institutionnel', weight: '25%', color: '#eab308', items: ['MSTR avg: $42,500', 'ETF avg: $58,200', 'BTC > $90K → 85-100'] },
                  { name: 'Gold/BTC', weight: '15%', color: '#f97316', items: ['Ratio ↓ > 15% → 80-100', 'Ratio ± 5% → 40-60', 'Ratio ↑ > 15% → 0-20'] },
                  { name: 'Social', weight: '15%', color: '#a855f7', items: ['⚠️ CONTRARIAN!', '> 80% positif → TOP?', '< 35% positif → BOTTOM?'] },
                ].map((factor, idx) => (
                  <div key={idx} className="bg-[#1e3a5f] rounded-xl p-4 border-l-4" style={{ borderColor: factor.color }}>
                    <div className="flex items-center justify-between mb-2">
                      <span className="font-semibold text-white">{factor.name}</span>
                      <span className="text-xs px-2 py-0.5 rounded" style={{ backgroundColor: `${factor.color}20`, color: factor.color }}>{factor.weight}</span>
                    </div>
                    <ul className="text-xs text-gray-400 space-y-1">
                      {factor.items.map((item, i) => <li key={i}>{item}</li>)}
                    </ul>
                  </div>
                ))}
              </div>
            </div>
          )}
        </div>
      );
    };

    // Crypto Card Component
    const CryptoCard = ({ crypto, rank }) => {
      const [expanded, setExpanded] = useState(false);
      
      const getSentimentStyle = (val) => {
        if (val <= 30) return { color: '#ea3943', label: 'BEARISH', bg: 'rgba(234, 57, 67, 0.1)' };
        if (val <= 50) return { color: '#ea8c00', label: 'NEUTRE', bg: 'rgba(234, 140, 0, 0.1)' };
        if (val <= 70) return { color: '#93d900', label: 'BULLISH', bg: 'rgba(147, 217, 0, 0.1)' };
        return { color: '#16c784', label: 'V.BULLISH', bg: 'rgba(22, 199, 132, 0.1)' };
      };
      
      const style = getSentimentStyle(crypto.sentiment);
      const sentimentChange = crypto.sentiment - crypto.sentimentPast;
      
      return (
        <div className="bg-[#0f2744] rounded-xl p-4 border border-[#1e3a5f] hover:border-[#2a4a6f] transition-all cursor-pointer" onClick={() => setExpanded(!expanded)}>
          <div className="flex items-center justify-between mb-3">
            <div className="flex items-center gap-2">
              <span className="text-xs text-gray-600 w-6">#{rank}</span>
              <span className="text-lg">{crypto.icon}</span>
              <div>
                <div className="font-semibold text-white text-sm">{crypto.symbol}</div>
                <div className="text-[10px] text-gray-500">{crypto.name}</div>
              </div>
            </div>
            <div className="text-right">
              <div className="text-white text-sm font-mono">${crypto.price < 1 ? crypto.price.toFixed(4) : crypto.price.toLocaleString()}</div>
              <div className={`text-xs font-semibold ${crypto.change24h >= 0 ? 'text-emerald-400' : 'text-red-400'}`}>
                {crypto.change24h >= 0 ? '+' : ''}{crypto.change24h.toFixed(2)}%
              </div>
            </div>
          </div>
          
          <div className="mb-2">
            <div className="flex items-center justify-between text-xs mb-1">
              <span className="text-gray-500">Sentiment</span>
              <span className="font-semibold" style={{ color: style.color }}>{style.label} ({crypto.sentiment}/100)</span>
            </div>
            <div className="relative h-2 bg-[#1e3a5f] rounded-full overflow-hidden">
              <div className="absolute h-full w-0.5 bg-white/30" style={{ left: `${crypto.sentimentPast}%` }} />
              <div className="absolute h-full rounded-full" style={{ width: `${crypto.sentiment}%`, backgroundColor: style.color }} />
            </div>
            <div className="flex justify-between text-[10px] mt-1">
              <span className="text-gray-600">Passé: {crypto.sentimentPast}/100</span>
              <span className={sentimentChange >= 0 ? 'text-emerald-400' : 'text-red-400'}>{sentimentChange >= 0 ? '↑' : '↓'} {Math.abs(sentimentChange)}</span>
            </div>
          </div>
          
          <div className="flex items-center justify-between text-xs px-2 py-1.5 bg-[#1e3a5f] rounded-lg">
            <span className="text-gray-500">Exchange Flow</span>
            <span className={`font-semibold ${crypto.exchangeFlow <= 0 ? 'text-emerald-400' : 'text-red-400'}`}>
              {crypto.exchangeFlow <= 0 ? '↓ Accum.' : '↑ Distrib.'} ({crypto.exchangeFlow}%)
            </span>
          </div>
          
          {expanded && (
            <div className="mt-3 pt-3 border-t border-[#1e3a5f] space-y-2">
              <div className="grid grid-cols-4 gap-2 text-center">
                {[{ label: '24H', value: crypto.change24h }, { label: '7J', value: crypto.change7d }, { label: '30J', value: crypto.change30d }, { label: 'vs BTC', value: crypto.vsBtc }].map((item, i) => (
                  <div key={i} className="bg-[#1e3a5f] rounded p-2">
                    <div className="text-[10px] text-gray-500">{item.label}</div>
                    <div className={`text-xs font-semibold ${item.value >= 0 ? 'text-emerald-400' : 'text-red-400'}`}>{item.value >= 0 ? '+' : ''}{item.value.toFixed(1)}%</div>
                  </div>
                ))}
              </div>
              <div className="bg-[#1e3a5f] rounded-lg p-2">
                <div className="text-[10px] text-gray-500 mb-1">Historique Sentiment</div>
                <div className="flex items-end gap-0.5 h-8">
                  {crypto.sentimentHistory.map((val, i) => (
                    <div key={i} className="flex-1 rounded-t" style={{ height: `${val}%`, backgroundColor: getSentimentStyle(val).color }} />
                  ))}
                </div>
              </div>
            </div>
          )}
        </div>
      );
    };

    // Header Component
    const Header = ({ isLive, lastUpdate, stats }) => (
      <header className="flex flex-col lg:flex-row items-start lg:items-center justify-between mb-6 gap-4">
        <div>
          <h1 className="text-2xl font-bold text-white">CRYPTO SENTINEL PRO</h1>
          <div className="flex items-center gap-3 mt-1">
            <span className={`flex items-center gap-2 px-3 py-1 rounded-full text-xs font-semibold ${isLive ? 'bg-emerald-500/20 text-emerald-400' : 'bg-red-500/20 text-red-400'}`}>
              <span className={`w-2 h-2 rounded-full ${isLive ? 'bg-emerald-400 animate-pulse' : 'bg-red-400'}`} />
              {isLive ? 'LIVE' : 'OFFLINE'}
            </span>
            <span className="text-sm text-gray-500">{lastUpdate}</span>
          </div>
        </div>
        <div className="flex gap-1 bg-[#1e3a5f] rounded-xl p-1">
          {[{ label: 'BULL', value: stats.bullish, color: '#22c55e' }, { label: 'NEUT', value: stats.neutral, color: '#eab308' }, { label: 'BEAR', value: stats.bearish, color: '#ef4444' }].map((stat, i) => (
            <div key={i} className="px-4 py-2 rounded-lg text-center" style={{ backgroundColor: `${stat.color}15` }}>
              <div className="text-[10px] font-semibold" style={{ color: stat.color }}>{stat.label}</div>
              <div className="text-xl font-bold text-white">{stat.value}</div>
            </div>
          ))}
        </div>
      </header>
    );

    // Filters Component
    const Filters = ({ filter, setFilter, search, setSearch, sort, setSort }) => (
      <div className="flex flex-col lg:flex-row items-stretch lg:items-center justify-between gap-4 mb-4 p-4 bg-[#0f2744] rounded-xl border border-[#1e3a5f]">
        <div className="relative w-full lg:w-64">
          <span className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-500">🔍</span>
          <input type="text" placeholder="Rechercher..." value={search} onChange={(e) => setSearch(e.target.value)}
            className="w-full bg-[#1e3a5f] border border-[#2a4a6f] rounded-lg pl-10 pr-4 py-2 text-white text-sm placeholder-gray-500 focus:outline-none focus:border-blue-500" />
        </div>
        <div className="flex flex-wrap gap-2">
          {['ALL', 'BULLISH', 'NEUTRAL', 'BEARISH'].map(f => (
            <button key={f} onClick={() => setFilter(f)} className={`px-4 py-2 rounded-lg text-xs font-semibold transition-all ${filter === f ? 'bg-blue-600 text-white' : 'bg-[#1e3a5f] text-gray-400 hover:text-white'}`}>
              {f === 'ALL' ? 'TOUS' : f}
            </button>
          ))}
          <select value={sort} onChange={(e) => setSort(e.target.value)} className="px-3 py-2 bg-[#1e3a5f] border border-[#2a4a6f] rounded-lg text-white text-xs focus:outline-none">
            <option value="rank">Rang</option>
            <option value="sentiment">Sentiment ↓</option>
            <option value="change24h">24h ↓</option>
          </select>
        </div>
      </div>
    );

    // Top 50 Data
    const generateTop50 = () => [
      { id: 1, symbol: 'BTC', name: 'Bitcoin', icon: '₿', price: 97845, change24h: 2.34, change7d: 5.67, change30d: 12.45, sentiment: 72, sentimentPast: 65, sentimentHistory: [58, 62, 65, 68, 72], exchangeFlow: -1.2, vsBtc: 0 },
      { id: 2, symbol: 'ETH', name: 'Ethereum', icon: 'Ξ', price: 3456, change24h: 1.87, change7d: 4.23, change30d: 8.91, sentiment: 68, sentimentPast: 62, sentimentHistory: [55, 58, 60, 64, 68], exchangeFlow: -0.8, vsBtc: -2.3 },
      { id: 3, symbol: 'BNB', name: 'BNB', icon: '◇', price: 612, change24h: -0.54, change7d: 2.11, change30d: 5.34, sentiment: 55, sentimentPast: 58, sentimentHistory: [60, 58, 56, 55, 55], exchangeFlow: 0.5, vsBtc: -4.2 },
      { id: 4, symbol: 'SOL', name: 'Solana', icon: '◎', price: 198, change24h: 4.21, change7d: 8.45, change30d: 22.3, sentiment: 81, sentimentPast: 72, sentimentHistory: [65, 70, 74, 78, 81], exchangeFlow: -2.1, vsBtc: 8.5 },
      { id: 5, symbol: 'XRP', name: 'XRP', icon: '✕', price: 2.34, change24h: -1.23, change7d: 3.45, change30d: 15.67, sentiment: 48, sentimentPast: 52, sentimentHistory: [55, 53, 51, 50, 48], exchangeFlow: 1.8, vsBtc: -6.1 },
      { id: 6, symbol: 'ADA', name: 'Cardano', icon: '₳', price: 0.89, change24h: 0.67, change7d: 2.34, change30d: 6.78, sentiment: 52, sentimentPast: 48, sentimentHistory: [45, 46, 48, 50, 52], exchangeFlow: 0.3, vsBtc: 1.2 },
      { id: 7, symbol: 'DOGE', name: 'Dogecoin', icon: 'Ð', price: 0.32, change24h: 3.45, change7d: 12.34, change30d: 25.67, sentiment: 65, sentimentPast: 58, sentimentHistory: [50, 54, 58, 62, 65], exchangeFlow: -0.5, vsBtc: 5.8 },
      { id: 8, symbol: 'AVAX', name: 'Avalanche', icon: '▲', price: 38.5, change24h: 2.12, change7d: 5.67, change30d: 11.23, sentiment: 61, sentimentPast: 55, sentimentHistory: [48, 51, 54, 58, 61], exchangeFlow: -0.9, vsBtc: 3.2 },
      { id: 9, symbol: 'DOT', name: 'Polkadot', icon: '●', price: 7.23, change24h: -0.89, change7d: 1.23, change30d: 3.45, sentiment: 45, sentimentPast: 50, sentimentHistory: [54, 52, 50, 48, 45], exchangeFlow: 0.7, vsBtc: -5.3 },
      { id: 10, symbol: 'LINK', name: 'Chainlink', icon: '⬡', price: 18.9, change24h: 1.56, change7d: 4.56, change30d: 9.87, sentiment: 67, sentimentPast: 60, sentimentHistory: [52, 55, 58, 63, 67], exchangeFlow: -1.1, vsBtc: 4.1 },
      { id: 11, symbol: 'MATIC', name: 'Polygon', icon: '⬢', price: 0.54, change24h: -2.34, change7d: -1.23, change30d: -5.67, sentiment: 38, sentimentPast: 52, sentimentHistory: [58, 52, 46, 42, 38], exchangeFlow: 1.5, vsBtc: -12.4 },
      { id: 12, symbol: 'SHIB', name: 'Shiba Inu', icon: '🐕', price: 0.000024, change24h: 5.67, change7d: 15.34, change30d: 28.9, sentiment: 62, sentimentPast: 50, sentimentHistory: [42, 48, 52, 58, 62], exchangeFlow: -0.4, vsBtc: 9.2 },
      { id: 13, symbol: 'UNI', name: 'Uniswap', icon: '🦄', price: 12.4, change24h: 0.98, change7d: 3.21, change30d: 7.65, sentiment: 54, sentimentPast: 51, sentimentHistory: [48, 49, 50, 52, 54], exchangeFlow: 0.1, vsBtc: 0.5 },
      { id: 14, symbol: 'ATOM', name: 'Cosmos', icon: '⚛', price: 9.87, change24h: -1.45, change7d: 0.89, change30d: 2.34, sentiment: 46, sentimentPast: 50, sentimentHistory: [52, 50, 49, 48, 46], exchangeFlow: 0.6, vsBtc: -3.8 },
      { id: 15, symbol: 'LTC', name: 'Litecoin', icon: 'Ł', price: 98.5, change24h: 0.23, change7d: 2.45, change30d: 5.67, sentiment: 51, sentimentPast: 49, sentimentHistory: [47, 48, 48, 50, 51], exchangeFlow: 0.2, vsBtc: 0.8 },
      { id: 16, symbol: 'NEAR', name: 'NEAR', icon: 'Ⓝ', price: 5.67, change24h: 3.21, change7d: 7.89, change30d: 14.56, sentiment: 66, sentimentPast: 56, sentimentHistory: [48, 52, 56, 62, 66], exchangeFlow: -0.9, vsBtc: 6.3 },
      { id: 17, symbol: 'APT', name: 'Aptos', icon: '◈', price: 9.12, change24h: -0.67, change7d: 1.89, change30d: 4.56, sentiment: 44, sentimentPast: 50, sentimentHistory: [54, 52, 50, 47, 44], exchangeFlow: 0.8, vsBtc: -4.1 },
      { id: 18, symbol: 'ARB', name: 'Arbitrum', icon: '🔷', price: 1.23, change24h: 2.89, change7d: 6.78, change30d: 12.34, sentiment: 62, sentimentPast: 54, sentimentHistory: [46, 50, 54, 58, 62], exchangeFlow: -0.5, vsBtc: 4.5 },
      { id: 19, symbol: 'OP', name: 'Optimism', icon: '🔴', price: 2.45, change24h: 1.78, change7d: 5.43, change30d: 10.21, sentiment: 58, sentimentPast: 53, sentimentHistory: [48, 50, 52, 55, 58], exchangeFlow: -0.3, vsBtc: 2.8 },
      { id: 20, symbol: 'INJ', name: 'Injective', icon: '💉', price: 24.3, change24h: 4.56, change7d: 9.87, change30d: 18.76, sentiment: 74, sentimentPast: 64, sentimentHistory: [56, 60, 64, 70, 74], exchangeFlow: -1.6, vsBtc: 7.8 },
    ].concat(Array.from({ length: 30 }, (_, i) => ({
      id: 21 + i,
      symbol: ['FIL', 'IMX', 'RENDER', 'SUI', 'TIA', 'STX', 'SEI', 'FET', 'PEPE', 'WIF', 'AAVE', 'GRT', 'MKR', 'XLM', 'ALGO', 'HBAR', 'VET', 'ICP', 'RUNE', 'SAND', 'MANA', 'AXS', 'FLOW', 'THETA', 'EOS', 'XTZ', 'EGLD', 'GALA', 'ENJ', 'CRV'][i],
      name: ['Filecoin', 'Immutable', 'Render', 'Sui', 'Celestia', 'Stacks', 'Sei', 'Fetch.ai', 'Pepe', 'dogwifhat', 'Aave', 'The Graph', 'Maker', 'Stellar', 'Algorand', 'Hedera', 'VeChain', 'ICP', 'THORChain', 'Sandbox', 'Decentraland', 'Axie', 'Flow', 'Theta', 'EOS', 'Tezos', 'MultiversX', 'Gala', 'Enjin', 'Curve'][i],
      icon: ['📁', '🎮', '🎨', '💧', '🌌', '📚', '🌊', '🤖', '🐸', '🎩', '👻', '📊', '🏛️', '✨', '🔺', 'ℏ', '✓', '∞', '⚡', '🏖️', '🌐', '🎮', '🌊', 'θ', '◉', 'ꜩ', '⬡', '🎪', '🎯', '📈'][i],
      price: [5.89, 2.15, 8.76, 3.45, 12.3, 2.34, 0.56, 2.12, 0.000012, 2.87, 267, 0.23, 1890, 0.12, 0.21, 0.089, 0.034, 12.4, 5.67, 0.45, 0.52, 7.89, 0.78, 1.89, 0.82, 1.12, 42.3, 0.045, 0.34, 0.89][i],
      change24h: [-1.23, 2.34, 3.45, 5.67, 2.89, 1.23, 3.21, 4.32, 6.78, 5.43, 1.89, 2.34, 0.98, 1.45, 0.89, 2.12, 1.67, -0.56, 2.89, 1.23, 0.78, 1.56, 2.34, 1.12, 0.67, 0.89, 1.78, 3.45, 2.12, 1.89][i],
      change7d: [2.34, 5.67, 8.9, 12.34, 7.65, 4.56, 8.76, 11.23, 18.9, 14.32, 5.43, 6.78, 3.21, 4.32, 2.67, 5.89, 4.56, 1.89, 7.65, 3.45, 2.34, 4.32, 5.67, 3.45, 2.12, 2.67, 4.89, 8.9, 5.43, 4.56][i],
      change30d: [5.67, 11.23, 16.78, 24.56, 15.43, 9.87, 17.65, 23.45, 35.67, 28.76, 12.34, 13.45, 7.65, 9.87, 5.43, 11.23, 8.9, 4.32, 14.32, 7.89, 5.67, 9.12, 11.23, 7.89, 4.56, 5.89, 10.23, 17.65, 11.23, 9.12][i],
      sentiment: [44, 58, 70, 76, 68, 56, 64, 72, 65, 70, 66, 58, 52, 50, 46, 56, 54, 42, 63, 48, 46, 52, 57, 50, 42, 46, 55, 61, 54, 56][i],
      sentimentPast: [48, 52, 62, 66, 60, 52, 56, 64, 54, 60, 60, 52, 50, 48, 48, 50, 50, 46, 56, 46, 44, 50, 52, 48, 44, 44, 50, 54, 50, 52][i],
      sentimentHistory: [[50, 48, 46, 45, 44], [50, 52, 54, 56, 58], [58, 62, 65, 68, 70], [62, 66, 70, 74, 76], [56, 60, 62, 66, 68], [50, 52, 53, 55, 56], [52, 56, 58, 62, 64], [60, 64, 66, 70, 72], [48, 54, 58, 62, 65], [56, 60, 64, 68, 70], [56, 58, 60, 64, 66], [50, 52, 54, 56, 58], [50, 50, 51, 51, 52], [47, 48, 48, 50, 50], [50, 48, 47, 46, 46], [48, 50, 52, 54, 56], [48, 50, 51, 53, 54], [48, 46, 44, 43, 42], [52, 55, 58, 60, 63], [44, 45, 46, 47, 48], [43, 44, 45, 45, 46], [48, 49, 50, 51, 52], [50, 52, 54, 55, 57], [47, 48, 49, 49, 50], [46, 44, 43, 43, 42], [44, 44, 45, 45, 46], [48, 50, 52, 54, 55], [50, 54, 56, 58, 61], [48, 50, 51, 53, 54], [50, 52, 53, 55, 56]][i],
      exchangeFlow: [0.9, -0.7, -1.3, -1.9, -1.0, -0.4, -0.8, -1.5, -0.6, -1.2, -0.9, -0.5, 0.2, 0.3, 0.6, -0.4, -0.3, 1.0, -0.8, 0.4, 0.5, 0.2, -0.5, 0.3, 0.7, 0.4, -0.3, -0.7, -0.4, -0.5][i],
      vsBtc: [-3.2, 3.5, 6.2, 10.2, 5.8, 2.1, 5.4, 8.9, 12.3, 11.5, 4.2, 3.1, 0.8, 1.2, -2.3, 3.4, 1.8, -4.5, 5.1, 0.6, 0.3, 1.5, 2.8, 0.9, -3.1, 0.2, 2.4, 6.8, 2.2, 2.6][i]
    })));

    // Main App
    function App() {
      const [isLive, setIsLive] = useState(true);
      const [lastUpdate, setLastUpdate] = useState(new Date().toLocaleTimeString());
      const [cryptos, setCryptos] = useState(generateTop50());
      const [filter, setFilter] = useState('ALL');
      const [search, setSearch] = useState('');
      const [sort, setSort] = useState('rank');
      const [showOpportunityDetails, setShowOpportunityDetails] = useState(false);
      
      const [fearGreedHistory] = useState(generateRealisticFearGreedHistory());
      const [btcPriceHistory] = useState(generateBtcPriceHistory(fearGreedHistory));
      const fearGreedValue = fearGreedHistory[fearGreedHistory.length - 1]?.value || 26;
      
      const opportunityScore = 68;
      const previousOpportunityScore = 62;
      const opportunityIndicators = [
        { name: 'USDT Flow', current: 72, past: 65, weight: 20 },
        { name: 'Market Cap', current: 78, past: 70, weight: 25 },
        { name: 'Institutionnel', current: 82, past: 75, weight: 25 },
        { name: 'Gold/BTC Ratio', current: 58, past: 52, weight: 15 },
        { name: 'Sentiment Social', current: 48, past: 55, weight: 15 },
      ];
      
      const stats = {
        bullish: cryptos.filter(c => c.sentiment > 60).length,
        neutral: cryptos.filter(c => c.sentiment >= 40 && c.sentiment <= 60).length,
        bearish: cryptos.filter(c => c.sentiment < 40).length,
      };
      
      useEffect(() => {
        const interval = setInterval(() => {
          setLastUpdate(new Date().toLocaleTimeString());
          setCryptos(prev => prev.map(c => ({
            ...c,
            price: c.price * (1 + (Math.random() - 0.5) * 0.002),
            change24h: c.change24h + (Math.random() - 0.5) * 0.05,
            sentiment: Math.round(Math.max(0, Math.min(100, c.sentiment + (Math.random() - 0.5) * 0.3))),
          })));
        }, CONFIG.REFRESH_RATE);
        return () => clearInterval(interval);
      }, []);
      
      const filtered = cryptos
        .filter(c => {
          if (search && !c.symbol.toLowerCase().includes(search.toLowerCase()) && !c.name.toLowerCase().includes(search.toLowerCase())) return false;
          if (filter === 'BULLISH') return c.sentiment > 60;
          if (filter === 'NEUTRAL') return c.sentiment >= 40 && c.sentiment <= 60;
          if (filter === 'BEARISH') return c.sentiment < 40;
          return true;
        })
        .sort((a, b) => {
          if (sort === 'sentiment') return b.sentiment - a.sentiment;
          if (sort === 'change24h') return b.change24h - a.change24h;
          return a.id - b.id;
        });
      
      return (
        <div className="min-h-screen bg-[#0a1628] text-white p-4 md:p-6">
          <div className="max-w-7xl mx-auto">
            <Header isLive={isLive} lastUpdate={lastUpdate} stats={stats} />
            <div className="mb-6"><FearGreedIndex currentValue={fearGreedValue} historicalData={fearGreedHistory} btcPriceData={btcPriceHistory} /></div>
            <div className="mb-6"><OpportunityIndex score={opportunityScore} previousScore={previousOpportunityScore} indicators={opportunityIndicators} showDetails={showOpportunityDetails} setShowDetails={setShowOpportunityDetails} /></div>
            <Filters filter={filter} setFilter={setFilter} search={search} setSearch={setSearch} sort={sort} setSort={setSort} />
            <div className="flex justify-between mb-4 text-xs text-gray-500">
              <span><strong className="text-white">{filtered.length}</strong> cryptos sur 50</span>
              <span>💡 Cliquez sur une carte pour l'historique</span>
            </div>
            <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 gap-4">
              {filtered.map(c => <CryptoCard key={c.id} crypto={c} rank={c.id} />)}
            </div>
            <footer className="mt-8 pt-6 border-t border-[#1e3a5f] text-center text-xs text-gray-500">
              <p>CRYPTO SENTINEL PRO • Fear & Greed via Alternative.me • Refresh: {CONFIG.REFRESH_RATE/1000}s</p>
              <p className="mt-1">API: https://api.alternative.me/fng/?limit=365</p>
            </footer>
          </div>
        </div>
      );
    }

    ReactDOM.render(<App />, document.getElementById('root'));
  </script>
</body>
</html>
