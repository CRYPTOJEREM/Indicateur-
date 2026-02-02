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
    @keyframes pulse { 0%,100%{opacity:1}50%{opacity:.5} }
    .animate-pulse { animation: pulse 2s cubic-bezier(0.4,0,0.6,1) infinite }
  </style>
</head>

<body>
  <div id="root"></div>

{% raw %}
<script type="text/babel">

const { useState, useEffect } = React;

/* ================= CONFIG ================= */
const CONFIG = {
  REFRESH_RATE: 5000,
};

/* ================= HELPERS ================= */
const getFearGreedClassification = (value) => {
  if (value <= 24) return { label:'Extreme Fear', color:'#ea3943', emoji:'😱' };
  if (value <= 44) return { label:'Fear', color:'#ea8c00', emoji:'😰' };
  if (value <= 55) return { label:'Neutral', color:'#c9b003', emoji:'😐' };
  if (value <= 74) return { label:'Greed', color:'#93d900', emoji:'😊' };
  return { label:'Extreme Greed', color:'#16c784', emoji:'🤑' };
};

/* ================= DATA ================= */
const generateFearGreedHistory = () => {
  const data = [];
  let v = 50;
  for (let i = 365; i >= 0; i--) {
    v += (Math.random() - 0.5) * 6;
    v = Math.max(5, Math.min(95, v));
    data.push({ value: Math.round(v) });
  }
  data[data.length - 1].value = 26;
  return data;
};

/* ================= COMPONENTS ================= */

const FearGreedIndex = ({ value }) => {
  const c = getFearGreedClassification(value);
  const rotation = -90 + (value / 100) * 180;

  return (
    <div className="bg-[#0f2744] rounded-2xl p-6 border border-[#1e3a5f]">
      <h2 className="text-lg font-bold text-white mb-3">Fear & Greed Index</h2>

      <svg width="220" height="130" viewBox="0 0 220 130">
        <path d="M20 110 A90 90 0 0 1 200 110"
              fill="none" stroke="#1e3a5f" strokeWidth="16" />
        <g style={{
          transform:`rotate(${rotation}deg)`,
          transformOrigin:'110px 110px',
          transition:'transform 1s ease-out'
        }}>
          <line x1="110" y1="110" x2="110" y2="35"
                stroke={c.color} strokeWidth="4" />
        </g>
        <circle cx="110" cy="110" r="6" fill={c.color} />
        <text x="110" y="95" textAnchor="middle"
              fill={c.color} fontSize="36" fontWeight="bold">
          {value}
        </text>
      </svg>

      <div className="text-center mt-2">
        <span className="text-3xl">{c.emoji}</span>
        <div className="font-bold" style={{color:c.color}}>
          {c.label}
        </div>
      </div>
    </div>
  );
};

/* ================= APP ================= */

function App() {
  const [history] = useState(generateFearGreedHistory());
  const current = history[history.length - 1].value;

  return (
    <div className="min-h-screen bg-[#0a1628] p-6 text-white">
      <div className="max-w-xl mx-auto">
        <FearGreedIndex value={current} />
      </div>
    </div>
  );
}

ReactDOM.render(<App />, document.getElementById('root'));

</script>
{% endraw %}

</body>
</html>
