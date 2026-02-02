<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <title>Fear & Greed Indicator</title>

  <!-- React -->
  <script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>

  <!-- Babel (pour JSX dans GitHub Pages) -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <style>
    body {
      font-family: Arial, sans-serif;
      background: #0f172a;
      color: white;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
    }

    .card {
      background: #020617;
      padding: 24px;
      border-radius: 16px;
      box-shadow: 0 20px 40px rgba(0,0,0,.4);
      text-align: center;
      width: 320px;
    }

    h1 {
      font-size: 20px;
      margin-bottom: 12px;
    }
  </style>
</head>

<body>
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect } = React;

    function FearGreedIndex() {
      const [currentValue, setCurrentValue] = useState(50);

      useEffect(() => {
        const interval = setInterval(() => {
          setCurrentValue(Math.floor(Math.random() * 100));
        }, 3000);
        return () => clearInterval(interval);
      }, []);

      const zones = [
        { label: "Fear", color: "#ef4444" },
        { label: "Neutral", color: "#facc15" },
        { label: "Greed", color: "#22c55e" }
      ];

      const current =
        currentValue < 33 ? zones[0] :
        currentValue < 66 ? zones[1] :
        zones[2];

      const rotation = -90 + (currentValue / 100) * 180;

      return (
        <div className="card">
          <h1>Fear & Greed Index</h1>

          <svg width="200" height="120" viewBox="0 0 200 120">
            <path d="M20 100 A80 80 0 0 1 180 100" fill="none" stroke="#334155" strokeWidth="12" />

            {/* Aiguille */}
            <g
              style={{
                transform: `rotate(${rotation}deg)`,
                transformOrigin: "100px 100px",
                transition: "transform 0.8s ease-out"
              }}
            >
              <line
                x1="100"
                y1="100"
                x2="100"
                y2="30"
                stroke={current.color}
                strokeWidth="4"
                strokeLinecap="round"
              />
            </g>

            <circle cx="100" cy="100" r="5" fill="white" />
          </svg>

          <p style={{ marginTop: "12px" }}>
            <strong>{currentValue}</strong> — {current.label}
          </p>
        </div>
      );
    }

    ReactDOM.createRoot(document.getElementById("root")).render(
      <FearGreedIndex />
    );
  </script>
</body>
</html>
