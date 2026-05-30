Inter id:CITS2678
import { useState, useRef, useCallback } from "react";

// ── Styles ─────────────────────────────────────────────────────────────────────
const CSS = `
@import url('https://fonts.googleapis.com/css2?family=Rajdhani:wght@400;500;600;700&family=Share+Tech+Mono&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg: #020b14;
  --panel: #041220;
  --panel2: #071c2e;
  --border: rgba(0,210,255,0.12);
  --border2: rgba(0,210,255,0.25);
  --cyan: #00d2ff;
  --cyan-dim: rgba(0,210,255,0.5);
  --green: #00ff88;
  --red: #ff3b5c;
  --amber: #ffb800;
  --text: #cde8f5;
  --muted: rgba(205,232,245,0.4);
  --mono: 'Share Tech Mono', monospace;
  --head: 'Rajdhani', sans-serif;
}

body { background: var(--bg); color: var(--text); font-family: var(--head); }

/* HUD scanline */
.scanline {
  position: fixed; inset: 0; pointer-events: none; z-index: 999;
  background: repeating-linear-gradient(
    to bottom,
    transparent 0px,
    transparent 3px,
    rgba(0,0,0,0.04) 3px,
    rgba(0,0,0,0.04) 4px
  );
}

/* Grid overlay */
.hud-grid {
  position: fixed; inset: 0; pointer-events: none; z-index: 0;
  background-image:
    linear-gradient(var(--border) 1px, transparent 1px),
    linear-gradient(90deg, var(--border) 1px, transparent 1px);
  background-size: 64px 64px;
  mask-image: radial-gradient(ellipse 80% 80% at 50% 50%, black, transparent);
}

.app { min-height: 100vh; display: flex; flex-direction: column; position: relative; z-index: 1; }

/* ── Header ── */
.header {
  border-bottom: 1px solid var(--border2);
  padding: 16px 32px;
  display: flex; align-items: center; justify-content: space-between;
  background: rgba(4,18,32,0.9);
  backdrop-filter: blur(12px);
  position: sticky; top: 0; z-index: 10;
}
.header-left { display: flex; align-items: center; gap: 16px; }
.logo-box {
  width: 40px; height: 40px;
  border: 1.5px solid var(--cyan);
  display: flex; align-items: center; justify-content: center;
  position: relative;
  clip-path: polygon(8px 0%, 100% 0%, 100% calc(100% - 8px), calc(100% - 8px) 100%, 0% 100%, 0% 8px);
}
.logo-box::before {
  content: ''; position: absolute; inset: 3px;
  border: 1px solid rgba(0,210,255,0.3);
  clip-path: polygon(6px 0%, 100% 0%, 100% calc(100% - 6px), calc(100% - 6px) 100%, 0% 100%, 0% 6px);
}
.logo-icon { font-size: 1.2rem; position: relative; z-index: 1; }

.header-title { font-size: 1.3rem; font-weight: 700; letter-spacing: 0.15em; text-transform: uppercase; }
.header-sub { font-family: var(--mono); font-size: 0.62rem; color: var(--cyan-dim); letter-spacing: 0.1em; margin-top: 2px; }

.status-row { display: flex; align-items: center; gap: 20px; }
.status-pill {
  display: flex; align-items: center; gap: 7px;
  font-family: var(--mono); font-size: 0.65rem; letter-spacing: 0.1em; text-transform: uppercase;
  padding: 5px 12px; border: 1px solid var(--border2);
  clip-path: polygon(6px 0%, 100% 0%, 100% calc(100% - 6px), calc(100% - 6px) 100%, 0% 100%, 0% 6px);
}
.status-dot { width: 6px; height: 6px; border-radius: 50%; animation: pulse 2s infinite; }
.status-dot.online { background: var(--green); box-shadow: 0 0 6px var(--green); }
@keyframes pulse { 0%,100%{opacity:1;transform:scale(1)} 50%{opacity:0.5;transform:scale(0.8)} }

/* ── Main layout ── */
.main { flex: 1; display: grid; grid-template-columns: 1fr 380px; gap: 0; min-height: calc(100vh - 73px); }
@media (max-width: 860px) { .main { grid-template-columns: 1fr; } }

/* ── Left: upload + preview ── */
.left-panel { border-right: 1px solid var(--border); padding: 28px; display: flex; flex-direction: column; gap: 24px; }

.panel-header {
  display: flex; align-items: center; gap: 10px;
  font-size: 0.7rem; font-family: var(--mono); color: var(--cyan-dim);
  letter-spacing: 0.12em; text-transform: uppercase; margin-bottom: 4px;
}
.corner-bracket {
  width: 14px; height: 14px; border-top: 1.5px solid var(--cyan); border-left: 1.5px solid var(--cyan);
}
.corner-bracket.br { border-top: none; border-left: none; border-bottom: 1.5px solid var(--cyan); border-right: 1.5px solid var(--cyan); }

/* Drop zone */
.drop-zone {
  border: 1.5px dashed var(--border2); border-radius: 2px;
  min-height: 240px; display: flex; flex-direction: column;
  align-items: center; justify-content: center; gap: 12px;
  cursor: pointer; transition: all 0.2s; position: relative; overflow: hidden;
  background: rgba(0,210,255,0.02);
}
.drop-zone:hover, .drop-zone.drag { border-color: var(--cyan); background: rgba(0,210,255,0.05); }
.drop-zone-icon { font-size: 2.8rem; filter: drop-shadow(0 0 12px var(--cyan-dim)); }
.drop-zone-title { font-size: 1rem; font-weight: 600; letter-spacing: 0.1em; }
.drop-zone-sub { font-family: var(--mono); font-size: 0.68rem; color: var(--muted); }
.drop-zone input { position: absolute; inset: 0; opacity: 0; cursor: pointer; }

/* Image preview */
.preview-wrap { position: relative; border: 1px solid var(--border2); border-radius: 2px; overflow: hidden; background: #000; }
.preview-img { width: 100%; max-height: 420px; object-fit: contain; display: block; }

/* Result overlay boxes drawn on image */
.overlay-canvas { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; }

/* Corner decorations on preview */
.preview-corner {
  position: absolute; width: 20px; height: 20px;
  border-color: var(--cyan); border-style: solid; opacity: 0.6;
}
.preview-corner.tl { top: 8px; left: 8px; border-width: 2px 0 0 2px; }
.preview-corner.tr { top: 8px; right: 8px; border-width: 2px 2px 0 0; }
.preview-corner.bl { bottom: 8px; left: 8px; border-width: 0 0 2px 2px; }
.preview-corner.br { bottom: 8px; right: 8px; border-width: 0 2px 2px 0; }

.analyze-btn {
  width: 100%; padding: 14px;
  background: linear-gradient(135deg, rgba(0,210,255,0.15), rgba(0,210,255,0.05));
  border: 1.5px solid var(--cyan);
  color: var(--cyan); font-family: var(--head); font-size: 0.88rem; font-weight: 700;
  letter-spacing: 0.2em; text-transform: uppercase; cursor: pointer;
  clip-path: polygon(10px 0%, 100% 0%, 100% calc(100% - 10px), calc(100% - 10px) 100%, 0% 100%, 0% 10px);
  transition: all 0.2s; position: relative; overflow: hidden;
}
.analyze-btn::before {
  content: ''; position: absolute; inset: 0;
  background: linear-gradient(135deg, transparent 40%, rgba(0,210,255,0.08));
}
.analyze-btn:hover:not(:disabled) { background: linear-gradient(135deg, rgba(0,210,255,0.25), rgba(0,210,255,0.1)); box-shadow: 0 0 20px rgba(0,210,255,0.2); }
.analyze-btn:disabled { opacity: 0.35; cursor: not-allowed; }
.analyze-btn.loading { animation: borderFlicker 0.8s linear infinite; }
@keyframes borderFlicker { 0%,100%{opacity:1} 50%{opacity:0.6} }

/* ── Right panel ── */
.right-panel { display: flex; flex-direction: column; background: var(--panel); }

.right-section { border-bottom: 1px solid var(--border); padding: 22px 24px; }

/* Stats row */
.stats-row { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 1px; background: var(--border); }
.stat-cell { background: var(--panel); padding: 16px 14px; text-align: center; }
.stat-val { font-size: 1.8rem; font-weight: 700; line-height: 1; margin-bottom: 4px; }
.stat-val.green { color: var(--green); text-shadow: 0 0 12px rgba(0,255,136,0.4); }
.stat-val.red { color: var(--red); text-shadow: 0 0 12px rgba(255,59,92,0.4); }
.stat-val.cyan { color: var(--cyan); text-shadow: 0 0 12px rgba(0,210,255,0.4); }
.stat-label { font-family: var(--mono); font-size: 0.6rem; color: var(--muted); letter-spacing: 0.1em; text-transform: uppercase; }

/* Detection list */
.detection-list { display: flex; flex-direction: column; gap: 10px; }

.detection-card {
  border: 1px solid var(--border); padding: 14px;
  clip-path: polygon(8px 0%, 100% 0%, 100% calc(100% - 8px), calc(100% - 8px) 100%, 0% 100%, 0% 8px);
  animation: slideIn 0.3s ease forwards; opacity: 0;
  background: rgba(0,0,0,0.3);
}
@keyframes slideIn { from{opacity:0;transform:translateX(12px)} to{opacity:1;transform:translateX(0)} }

.detection-card.masked { border-color: rgba(0,255,136,0.3); background: rgba(0,255,136,0.04); }
.detection-card.unmasked { border-color: rgba(255,59,92,0.3); background: rgba(255,59,92,0.04); }
.detection-card.partial { border-color: rgba(255,184,0,0.3); background: rgba(255,184,0,0.04); }

.det-top { display: flex; align-items: center; justify-content: space-between; margin-bottom: 8px; }
.det-id { font-family: var(--mono); font-size: 0.65rem; color: var(--muted); }
.det-status {
  font-family: var(--mono); font-size: 0.65rem; font-weight: 600; letter-spacing: 0.1em; text-transform: uppercase;
  padding: 2px 8px;
}
.det-status.masked { color: var(--green); border: 1px solid rgba(0,255,136,0.4); }
.det-status.unmasked { color: var(--red); border: 1px solid rgba(255,59,92,0.4); }
.det-status.partial { color: var(--amber); border: 1px solid rgba(255,184,0,0.4); }

.det-conf-row { display: flex; align-items: center; gap: 8px; }
.det-conf-bar { flex: 1; height: 3px; background: var(--border); border-radius: 1px; }
.det-conf-fill { height: 100%; border-radius: 1px; transition: width 0.6s ease; }
.det-conf-fill.masked { background: var(--green); }
.det-conf-fill.unmasked { background: var(--red); }
.det-conf-fill.partial { background: var(--amber); }
.det-conf-pct { font-family: var(--mono); font-size: 0.65rem; color: var(--muted); min-width: 32px; text-align: right; }

.det-desc { font-family: var(--mono); font-size: 0.68rem; color: var(--muted); line-height: 1.6; margin-top: 8px; }

/* Empty state */
.empty-state { padding: 40px 24px; text-align: center; }
.empty-icon { font-size: 2.5rem; margin-bottom: 12px; opacity: 0.3; }
.empty-text { font-family: var(--mono); font-size: 0.72rem; color: var(--muted); line-height: 1.7; }

/* Loading animation */
.loading-state { padding: 32px 24px; display: flex; flex-direction: column; align-items: center; gap: 16px; }
.radar {
  width: 80px; height: 80px; border-radius: 50%;
  border: 1.5px solid var(--cyan-dim); position: relative;
}
.radar::before {
  content: ''; position: absolute; inset: 0; border-radius: 50%;
  background: conic-gradient(from 0deg, transparent 70%, rgba(0,210,255,0.4));
  animation: radarSpin 1.5s linear infinite;
}
.radar::after {
  content: ''; position: absolute; inset: 6px; border-radius: 50%;
  border: 1px solid rgba(0,210,255,0.2);
}
@keyframes radarSpin { to { transform: rotate(360deg); } }
.loading-text { font-family: var(--mono); font-size: 0.72rem; color: var(--cyan-dim); letter-spacing: 0.1em; animation: textBlink 1s ease-in-out infinite; }
@keyframes textBlink { 0%,100%{opacity:1} 50%{opacity:0.4} }

/* Algorithm section */
.algo-cards { display: flex; flex-direction: column; gap: 8px; }
.algo-card { background: var(--panel2); border: 1px solid var(--border); padding: 12px 14px; }
.algo-title { font-size: 0.78rem; font-weight: 600; letter-spacing: 0.08em; margin-bottom: 4px; }
.algo-desc { font-family: var(--mono); font-size: 0.64rem; color: var(--muted); line-height: 1.6; }

/* Bottom info strip */
.info-strip {
  margin-top: auto; padding: 16px 24px;
  border-top: 1px solid var(--border);
  font-family: var(--mono); font-size: 0.62rem; color: var(--muted);
  display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 8px;
}
`;

// ── Detection result card ──────────────────────────────────────────────────────
function DetCard({ det, idx }) {
  const cls = det.status.toLowerCase().replace(" ", "");
  const statusKey = det.status === "MASKED" ? "masked" : det.status === "UNMASKED" ? "unmasked" : "partial";
  return (
    <div className={`detection-card ${statusKey}`} style={{ animationDelay: `${idx * 0.1}s` }}>
      <div className="det-top">
        <span className="det-id">FACE_{String(idx + 1).padStart(2, "0")}</span>
        <span className={`det-status ${statusKey}`}>{det.status}</span>
      </div>
      <div className="det-conf-row">
        <div className="det-conf-bar">
          <div className={`det-conf-fill ${statusKey}`} style={{ width: `${det.confidence}%` }} />
        </div>
        <span className="det-conf-pct">{det.confidence}%</span>
      </div>
      {det.notes && <div className="det-desc">{det.notes}</div>}
    </div>
  );
}

// ── Main Component ─────────────────────────────────────────────────────────────
export default function App() {
  const [imageFile, setImageFile] = useState(null);
  const [imageUrl, setImageUrl] = useState(null);
  const [imageBase64, setImageBase64] = useState(null);
  const [dragging, setDragging] = useState(false);
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState(null);
  const [error, setError] = useState("");
  const fileRef = useRef();

  const handleFile = (file) => {
    if (!file || !file.type.startsWith("image/")) return;
    setImageFile(file);
    setResults(null);
    setError("");
    const url = URL.createObjectURL(file);
    setImageUrl(url);
    const reader = new FileReader();
    reader.onload = (e) => {
      const b64 = e.target.result.split(",")[1];
      setImageBase64(b64);
    };
    reader.readAsDataURL(file);
  };

  const onDrop = (e) => {
    e.preventDefault(); setDragging(false);
    handleFile(e.dataTransfer.files[0]);
  };

  const analyze = async () => {
    if (!imageBase64) return;
    setLoading(true);
    setResults(null);
    setError("");

    const prompt = `You are an expert AI computer vision system specialized in face mask detection.

Analyze this image carefully and detect ALL faces present. For each face found:
1. Determine mask status: MASKED (fully wearing mask), UNMASKED (no mask), or PARTIAL (mask worn incorrectly, below nose, etc.)
2. Estimate your confidence percentage (0-100)
3. Provide brief notes about what you observed

Respond ONLY with a valid JSON object in this exact format (no markdown, no backticks):
{
  "total_faces": <number>,
  "masked_count": <number>,
  "unmasked_count": <number>,
  "partial_count": <number>,
  "scene_description": "<brief 1-sentence description of the image>",
  "detections": [
    {
      "status": "MASKED" | "UNMASKED" | "PARTIAL",
      "confidence": <0-100>,
      "notes": "<brief observation about this face, position, mask type if any>"
    }
  ],
  "compliance_rate": <percentage 0-100>,
  "overall_assessment": "<1-2 sentence summary of mask compliance in this image>"
}

If no faces are detected, return total_faces: 0 with empty detections array.`;

    try {
      const response = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          messages: [{
            role: "user",
            content: [
              {
                type: "image",
                source: {
                  type: "base64",
                  media_type: imageFile.type || "image/jpeg",
                  data: imageBase64,
                },
              },
              { type: "text", text: prompt },
            ],
          }],
        }),
      });

      const data = await response.json();
      if (data.error) throw new Error(data.error.message);

      const raw = data.content?.map(b => b.text || "").join("") || "";
      const clean = raw.replace(/```json|```/g, "").trim();
      const parsed = JSON.parse(clean);
      setResults(parsed);
    } catch (e) {
      setError("Analysis failed: " + (e.message || "Unknown error. Please try again."));
    }
    setLoading(false);
  };

  const complianceColor = results
    ? results.compliance_rate >= 80 ? "green"
      : results.compliance_rate >= 50 ? "amber" : "red"
    : "cyan";

  return (
    <>
      <style>{CSS}</style>
      <div className="scanline" />
      <div className="hud-grid" />
      <div className="app">

        {/* Header */}
        <header className="header">
          <div className="header-left">
            <div className="logo-box"><span className="logo-icon">😷</span></div>
            <div>
              <div className="header-title">MaskNet AI</div>
              <div className="header-sub">FACE MASK DETECTION SYSTEM · v3.0</div>
            </div>
          </div>
          <div className="status-row">
            <div className="status-pill">
              <div className="status-dot online" />
              Vision Model Online
            </div>
          </div>
        </header>

        <div className="main">

          {/* ── Left panel ── */}
          <div className="left-panel">
            <div>
              <div className="panel-header">
                <div className="corner-bracket" />
                INPUT · IMAGE UPLOAD
              </div>
            </div>

            {!imageUrl ? (
              <div
                className={`drop-zone ${dragging ? "drag" : ""}`}
                onDragOver={e => { e.preventDefault(); setDragging(true); }}
                onDragLeave={() => setDragging(false)}
                onDrop={onDrop}
                onClick={() => fileRef.current.click()}
              >
                <input ref={fileRef} type="file" accept="image/*" onChange={e => handleFile(e.target.files[0])} />
                <div className="drop-zone-icon">📷</div>
                <div className="drop-zone-title">Upload Image</div>
                <div className="drop-zone-sub">Drag & drop or click · JPG, PNG, WEBP</div>
                <div className="drop-zone-sub" style={{ fontSize: "0.6rem", marginTop: 4 }}>
                  Upload photos of people to detect mask compliance
                </div>
              </div>
            ) : (
              <div className="preview-wrap">
                <img src={imageUrl} alt="Upload" className="preview-img" />
                <div className="preview-corner tl" />
                <div className="preview-corner tr" />
                <div className="preview-corner bl" />
                <div className="preview-corner br" />
                {results && (
                  <svg className="overlay-canvas" viewBox="0 0 100 100" preserveAspectRatio="none">
                    {/* Visual scan line animation over image */}
                    <line x1="0" y1="0" x2="100" y2="0" stroke="rgba(0,210,255,0.4)" strokeWidth="0.3">
                      <animate attributeName="y1" values="0;100;0" dur="3s" repeatCount="indefinite" />
                      <animate attributeName="y2" values="0;100;0" dur="3s" repeatCount="indefinite" />
                    </line>
                  </svg>
                )}
              </div>
            )}

            {imageUrl && (
              <button
                className={`analyze-btn ${loading ? "loading" : ""}`}
                onClick={analyze}
                disabled={loading || !imageBase64}
              >
                {loading ? "⟳  SCANNING..." : "▶  ANALYZE MASK COMPLIANCE"}
              </button>
            )}

            {imageUrl && (
              <button
                style={{
                  background: "transparent", border: "1px solid var(--border)",
                  color: "var(--muted)", padding: "10px", cursor: "pointer",
                  fontFamily: "var(--mono)", fontSize: "0.7rem", letterSpacing: "0.1em",
                  textTransform: "uppercase", width: "100%"
                }}
                onClick={() => { setImageUrl(null); setImageBase64(null); setImageFile(null); setResults(null); setError(""); }}
              >
                ✕ Clear Image
              </button>
            )}

            {error && (
              <div style={{ fontFamily: "var(--mono)", fontSize: "0.72rem", color: "var(--red)", padding: "12px", border: "1px solid rgba(255,59,92,0.3)", background: "rgba(255,59,92,0.05)" }}>
                ⚠ {error}
              </div>
            )}

            {/* Overall assessment */}
            {results?.overall_assessment && (
              <div style={{ background: "var(--panel)
