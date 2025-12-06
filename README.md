<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Terminal – Universal Telematics API on Google Cloud</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <meta name="description" content="Terminal is a universal API for commercial trucking telematics, built on Google Cloud. Insurance, financial services and fleet platforms use Terminal to access GPS, safety and vehicle data via a single, secure API." />

  <!-- Google Font -->
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet" />

  <style>
    :root {
      --bg: #050814;
      --bg-elevated: rgba(10, 14, 30, 0.92);
      --accent: #4fd1ff;
      --accent-soft: rgba(79, 209, 255, 0.18);
      --accent-strong: #00b3ff;
      --text-main: #f5f7ff;
      --text-muted: #9aa3c0;
      --border-subtle: rgba(255, 255, 255, 0.06);
      --google-blue: #4285f4;
      --google-green: #34a853;
      --google-yellow: #fbbc05;
      --google-red: #ea4335;
      --shadow-soft: 0 18px 45px rgba(0, 0, 0, 0.65);
      --radius-lg: 18px;
      --radius-pill: 999px;
      --transition-fast: 0.18s ease-out;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      font-family: "Inter", system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: radial-gradient(circle at top left, #111827 0, #020413 45%, #000 100%);
      color: var(--text-main);
      min-height: 100vh;
      overflow-x: hidden;
      position: relative;
    }

    /* Dynamic canvas background */
    #bg-canvas {
      position: fixed;
      inset: 0;
      z-index: -1;
    }

    /* Frosted glass overlay */
    .bg-gradient-overlay {
      position: fixed;
      inset: -20%;
      background:
        radial-gradient(circle at 10% 0%, rgba(79, 209, 255, 0.08) 0, transparent 55%),
        radial-gradient(circle at 100% 20%, rgba(129, 140, 248, 0.08) 0, transparent 55%),
        radial-gradient(circle at 0% 100%, rgba(16, 185, 129, 0.06) 0, transparent 55%);
      pointer-events: none;
      z-index: -1;
      mix-blend-mode: screen;
    }

    header {
      position: sticky;
      top: 0;
      z-index: 30;
      backdrop-filter: blur(22px);
      background: linear-gradient(to bottom, rgba(2, 6, 23, 0.95), rgba(2, 6, 23, 0.4));
      border-bottom: 1px solid var(--border-subtle);
    }

    .nav {
      max-width: 1120px;
      margin: 0 auto;
      padding: 14px 20px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 18px;
    }

    .logo {
      display: flex;
      align-items: center;
      gap: 10px;
      font-weight: 600;
      font-size: 1.1rem;
      letter-spacing: 0.04em;
    }

    .logo-mark {
      width: 32px;
      height: 32px;
      border-radius: 12px;
      background: radial-gradient(circle at 30% 0%, #4fd1ff 0, #22c55e 40%, #111827 80%);
      display: grid;
      place-items: center;
      box-shadow: 0 0 32px rgba(79, 209, 255, 0.7);
    }

    .logo-mark span {
      font-size: 0.8rem;
      font-weight: 800;
      color: #0b1020;
    }

    .nav-pill {
      display: flex;
      align-items: center;
      gap: 12px;
      padding: 6px 14px;
      border-radius: var(--radius-pill);
      background: rgba(15, 23, 42, 0.95);
      border: 1px solid rgba(148, 163, 184, 0.3);
      font-size: 0.75rem;
      color: var(--text-muted);
    }

    .nav-pill-dot {
      width: 8px;
      height: 8px;
      border-radius: 999px;
      background: #22c55e;
      box-shadow: 0 0 10px rgba(34, 197, 94, 0.9);
    }

    .nav-actions {
      display: flex;
      align-items: center;
      gap: 10px;
      font-size: 0.8rem;
    }

    .nav-link {
      color: var(--text-muted);
      text-decoration: none;
      padding: 4px 8px;
      border-radius: 999px;
      transition: color var(--transition-fast), background var(--transition-fast);
    }

    .nav-link:hover {
      color: var(--text-main);
      background: rgba(148, 163, 184, 0.16);
    }

    .nav-cta {
      padding: 7px 14px;
      border-radius: 999px;
      border: none;
      background: linear-gradient(135deg, var(--accent-strong), #16a34a);
      color: #020617;
      font-weight: 600;
      cursor: pointer;
      box-shadow: 0 10px 35px rgba(0, 0, 0, 0.6);
      text-decoration: none;
      transition: transform var(--transition-fast), box-shadow var(--transition-fast), filter var(--transition-fast);
    }

    .nav-cta:hover {
      transform: translateY(-1px);
      filter: brightness(1.03);
      box-shadow: 0 16px 40px rgba(15, 23, 42, 0.9);
    }

    main {
      max-width: 1120px;
      margin: 0 auto;
      padding: 42px 20px 60px;
    }

    .hero {
      display: grid;
      grid-template-columns: minmax(0, 1.5fr) minmax(0, 1.1fr);
      gap: 40px;
      align-items: center;
      margin-bottom: 54px;
    }

    @media (max-width: 880px) {
      .hero {
        grid-template-columns: minmax(0, 1fr);
      }
      header {
        position: static;
      }
    }

    .chip-row {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-bottom: 18px;
    }

    .chip {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      padding: 5px 11px;
      border-radius: var(--radius-pill);
      background: rgba(15, 23, 42, 0.9);
      border: 1px solid rgba(51, 65, 85, 0.9);
      font-size: 0.73rem;
      color: var(--text-muted);
    }

    .chip-label {
      padding: 2px 8px;
      border-radius: 999px;
      font-size: 0.64rem;
      text-transform: uppercase;
      letter-spacing: 0.12em;
      background: rgba(79, 209, 255, 0.1);
      color: var(--accent);
    }

    .hero-title {
      font-size: clamp(2.35rem, 3vw, 2.9rem);
      line-height: 1.12;
      margin-bottom: 16px;
      letter-spacing: 0.01em;
    }

    .hero-title span {
      background: linear-gradient(120deg, #4fd1ff, #a855f7, #22c55e);
      -webkit-background-clip: text;
      background-clip: text;
      color: transparent;
    }

    .hero-subtitle {
      color: var(--text-muted);
      font-size: 0.98rem;
      line-height: 1.7;
      margin-bottom: 20px;
      max-width: 520px;
    }

    .hero-actions {
      display: flex;
      flex-wrap: wrap;
      gap: 12px;
      margin-bottom: 22px;
    }

    .btn-primary,
    .btn-secondary {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      padding: 10px 18px;
      border-radius: 999px;
      font-size: 0.9rem;
      border: 1px solid transparent;
      cursor: pointer;
      text-decoration: none;
      transition: background var(--transition-fast), transform var(--transition-fast), box-shadow var(--transition-fast), border-color var(--transition-fast), color var(--transition-fast);
      white-space: nowrap;
    }

    .btn-primary {
      background: linear-gradient(135deg, var(--accent-strong), #16a34a);
      color: #020617;
      font-weight: 600;
      box-shadow: 0 16px 40px rgba(15, 23, 42, 0.95);
    }

    .btn-primary:hover {
      transform: translateY(-1px);
      box-shadow: 0 20px 55px rgba(15, 23, 42, 0.98);
      filter: brightness(1.03);
    }

    .btn-secondary {
      background: rgba(15, 23, 42, 0.9);
      border-color: rgba(148, 163, 184, 0.4);
      color: var(--text-muted);
    }

    .btn-secondary:hover {
      background: rgba(30, 64, 175, 0.6);
      color: var(--text-main);
      border-color: rgba(129, 140, 248, 0.8);
    }

    .hero-footnote {
      font-size: 0.8rem;
      color: var(--text-muted);
    }

    .hero-footnote strong {
      color: var(--accent);
      font-weight: 500;
    }

    .hero-card {
      background: radial-gradient(circle at top, rgba(15, 23, 42, 0.9), rgba(15, 23, 42, 0.98));
      border-radius: var(--radius-lg);
      border: 1px solid rgba(148, 163, 184, 0.35);
      box-shadow: var(--shadow-soft);
      padding: 20px 18px 16px;
      position: relative;
      overflow: hidden;
    }

    .hero-card::before {
      content: "";
      position: absolute;
      inset: -40%;
      background:
        radial-gradient(circle at 0 0, rgba(79, 209, 255, 0.28) 0, transparent 60%),
        radial-gradient(circle at 100% 100%, rgba(37, 99, 235, 0.3) 0, transparent 60%);
      opacity: 0.7;
      pointer-events: none;
      mix-blend-mode: screen;
    }

    .hero-card-inner {
      position: relative;
      z-index: 1;
    }

    .badge-row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 10px;
      margin-bottom: 14px;
      font-size: 0.78rem;
      color: var(--text-muted);
    }

    .gcp-badge {
      display: inline-flex;
      align-items: center;
      gap: 6px;
      padding: 5px 10px;
      border-radius: 999px;
      background: rgba(15, 23, 42, 0.9);
      border: 1px solid rgba(148, 163, 184, 0.45);
    }

    .gcp-dots {
      display: inline-flex;
      gap: 3px;
    }

    .gcp-dot {
      width: 8px;
      height: 8px;
      border-radius: 999px;
    }

    .gcp-dot.blue { background: var(--google-blue); }
    .gcp-dot.red { background: var(--google-red); }
    .gcp-dot.yellow { background: var(--google-yellow); }
    .gcp-dot.green { background: var(--google-green); }

    .stat-grid {
      display: grid;
      grid-template-columns: repeat(3, minmax(0, 1fr));
      gap: 14px;
      margin-bottom: 20px;
    }

    .stat {
      background: rgba(15, 23, 42, 0.9);
      border-radius: 14px;
      padding: 10px 11px;
      border: 1px solid rgba(51, 65, 85, 0.7);
      font-size: 0.78rem;
    }

    .stat-label {
      color: var(--text-muted);
      margin-bottom: 4px;
    }

    .stat-value {
      font-size: 0.9rem;
      font-weight: 600;
    }

    .pill-row {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      margin-bottom: 10px;
    }

    .pill {
      font-size: 0.72rem;
      padding: 5px 10px;
      border-radius: 999px;
      background: rgba(15, 23, 42, 0.9);
      border: 1px solid rgba(79, 209, 255, 0.26);
      color: var(--text-muted);
      display: inline-flex;
      align-items: center;
      gap: 6px;
    }

    .pill-dot {
      width: 6px;
      height: 6px;
      border-radius: 999px;
      background: var(--accent);
      box-shadow: 0 0 10px rgba(79, 209, 255, 0.9);
    }

    .card-caption {
      font-size: 0.78rem;
      color: var(--text-muted);
      margin-top: 4px;
    }

    section {
      margin-bottom: 46px;
    }

    .section-label {
      font-size: 0.78rem;
      letter-spacing: 0.16em;
      text-transform: uppercase;
      color: var(--text-muted);
      margin-bottom: 8px;
    }

    .section-title {
      font-size: 1.2rem;
      margin-bottom: 16px;
    }

    .two-col {
      display: grid;
      grid-template-columns: minmax(0, 1.1fr) minmax(0, 1.1fr);
      gap: 26px;
    }

    @media (max-width: 880px) {
      .two-col {
        grid-template-columns: minmax(0, 1fr);
      }
    }

    .card {
      background: var(--bg-elevated);
      border-radius: var(--radius-lg);
      border: 1px solid var(--border-subtle);
      padding: 18px 18px 16px;
      box-shadow: 0 18px 40px rgba(15, 23, 42, 0.7);
    }

    .card h3 {
      font-size: 1rem;
      margin-bottom: 8px;
    }

    .card p {
      font-size: 0.9rem;
      color: var(--text-muted);
      line-height: 1.7;
    }

    ul.feature-list {
      list-style: none;
      margin-top: 10px;
      display: grid;
      gap: 8px;
      font-size: 0.9rem;
      color: var(--text-muted);
    }

    ul.feature-list li {
      display: flex;
      gap: 8px;
      align-items: flex-start;
    }

    .feature-bullet {
      width: 7px;
      height: 7px;
      margin-top: 6px;
      border-radius: 999px;
      background: var(--accent);
      box-shadow: 0 0 10px rgba(79, 209, 255, 0.8);
      flex-shrink: 0;
    }

    .tag-row {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      margin-top: 10px;
    }

    .tag {
      font-size: 0.72rem;
      padding: 4px 9px;
      border-radius: 999px;
      border: 1px solid rgba(148, 163, 184, 0.45);
      color: var(--text-muted);
      background: rgba(15, 23, 42, 0.9);
    }

    .code-block {
      font-family: "JetBrains Mono", "SF Mono", Menlo, Monaco, Consolas, monospace;
      font-size: 0.8rem;
      background: #020617;
      color: #e5e7eb;
      padding: 10px 12px;
      border-radius: 10px;
      border: 1px solid rgba(30, 64, 175, 0.9);
      overflow-x: auto;
      margin-top: 10px;
    }

    .footnote {
      font-size: 0.82rem;
      color: var(--text-muted);
      margin-top: 6px;
    }

    .contact {
      display: flex;
      flex-wrap: wrap;
      gap: 18px;
      align-items: center;
      justify-content: space-between;
      font-size: 0.88rem;
    }

    .contact a {
      color: var(--accent);
      text-decoration: none;
    }

    .contact a:hover {
      text-decoration: underline;
    }

    footer {
      max-width: 1120px;
      margin: 0 auto;
      padding: 0 20px 32px;
      font-size: 0.78rem;
      color: var(--text-muted);
      display: flex;
      justify-content: space-between;
      gap: 12px;
      flex-wrap: wrap;
    }
  </style>
</head>
<body>
<canvas id="bg-canvas"></canvas>
<div class="bg-gradient-overlay"></div>

<header>
  <nav class="nav">
    <div class="logo">
      <div class="logo-mark"><span>T</span></div>
      <div>Terminal</div>
    </div>
    <div class="nav-pill">
      <div class="nav-pill-dot"></div>
      Live unified telematics API · Canada
    </div>
    <div class="nav-actions">
      <a href="#solution" class="nav-link">Solution</a>
      <a href="#architecture" class="nav-link">Architecture</a>
      <a href="#contact" class="nav-link">Contact</a>
      <a href="mailto:connor@withterminal.info" class="nav-cta">Book a demo</a>
    </div>
  </nav>
</header>

<main>
  <!-- HERO -->
  <section class="hero" id="solution">
    <div>
      <div class="chip-row">
        <div class="chip">
          <span class="chip-label">Universal API</span>
          Commercial trucking telematics
        </div>
        <div class="chip">
          <span class="chip-label">Built on</span>
          Google Cloud
        </div>
      </div>

      <h1 class="hero-title">
        A single <span>Universal API</span> for commercial trucking telematics.
      </h1>

      <p class="hero-subtitle">
        Terminal unifies GPS, safety events, dash-cam video and vehicle diagnostics from fragmented
        telematics providers into one secure, normalized API. Insurance, financial services and fleet
        software teams plug into Terminal once and ship products faster.
      </p>

      <div class="hero-actions">
        <a href="mailto:connor@withterminal.info?subject=Terminal%20Demo%20Request" class="btn-primary">
          Request a demo
        </a>
        <a href="#architecture" class="btn-secondary">
          View Google Cloud architecture
        </a>
      </div>

      <p class="hero-footnote">
        Trusted by teams building <strong>usage-based insurance</strong>, <strong>fleet safety automation</strong> and
        <strong>embedded financial services</strong> for trucking.
      </p>
    </div>

    <!-- RIGHT HERO CARD -->
    <aside class="hero-card" aria-label="Google Cloud deployment snapshot">
      <div class="hero-card-inner">
        <div class="badge-row">
          <div class="gcp-badge">
            <span class="gcp-dots">
              <span class="gcp-dot blue"></span>
              <span class="gcp-dot red"></span>
              <span class="gcp-dot yellow"></span>
              <span class="gcp-dot green"></span>
            </span>
            Runs on Google Cloud
          </div>
          <span>Region: North America</span>
        </div>

        <div class="stat-grid">
          <div class="stat">
            <div class="stat-label">Daily events</div>
            <div class="stat-value">+10M</div>
          </div>
          <div class="stat">
            <div class="stat-label">Latency</div>
            <div class="stat-value">&lt; 1s</div>
          </div>
          <div class="stat">
            <div class="stat-label">Vendors unified</div>
            <div class="stat-value">10+ APIs</div>
          </div>
        </div>

        <div class="pill-row">
          <div class="pill">
            <span class="pill-dot"></span>
            Cloud Run microservices
          </div>
          <div class="pill">
            <span class="pill-dot"></span>
            Pub/Sub event pipeline
          </div>
          <div class="pill">
            <span class="pill-dot"></span>
            BigQuery analytics lake
          </div>
          <div class="pill">
            <span class="pill-dot"></span>
            Cloud Storage media vault
          </div>
        </div>

        <p class="card-caption">
          Terminal continuously normalizes vendor-specific payloads into a stable schema optimized for
          underwriting, risk scoring and operational visibility.
        </p>
      </div>
    </aside>
  </section>

  <!-- HOW IT WORKS + GCP INTEGRATION -->
  <section>
    <div class="section-label">How Terminal works</div>
    <h2 class="section-title">Plaid-style connectivity for trucking data, powered by Google Cloud.</h2>

    <div class="two-col">
      <div class="card">
        <h3>Unified telematics layer</h3>
        <p>
          Terminal connects to multiple telematics providers, ELDs and dash-cam systems, then exposes a
          single, well-documented API. Clients use one integration to retrieve GPS traces, speeding and
          harsh-braking events, engine fault codes, fuel usage and dash-cam media instead of dealing with
          dozens of proprietary endpoints.
        </p>

        <ul class="feature-list">
          <li>
            <span class="feature-bullet"></span>
            <span>Normalized data model for trips, vehicles, drivers and events.</span>
          </li>
          <li>
            <span class="feature-bullet"></span>
            <span>Consistent webhooks for real-time risk and operations workflows.</span>
          </li>
          <li>
            <span class="feature-bullet"></span>
            <span>Fine-grained permissions aligned with customer consent and policy terms.</span>
          </li>
        </ul>
      </div>

      <div class="card" id="architecture">
        <h3>Google Cloud architecture</h3>
        <p>
          Terminal is built natively on Google Cloud. Cloud Run hosts stateless API services; Pub/Sub
          streams high-volume telematics events; Cloud Functions handle vendor webhooks and data
          transformation; BigQuery powers analytical queries and modeling; Cloud Storage retains media
          from dash-cams; Cloud Logging, Cloud Monitoring and IAM enforce observability and security.
        </p>

        <ul class="feature-list">
          <li>
            <span class="feature-bullet"></span>
            <span>Encryption in transit (TLS) and at rest with Google-managed keys.</span>
          </li>
          <li>
            <span class="feature-bullet"></span>
            <span>Private connectivity via VPC and service perimeter options for regulated clients.</span>
          </li>
          <li>
            <span class="feature-bullet"></span>
            <span>Scales automatically with traffic from fleets of any size.</span>
          </li>
        </ul>

        <div class="tag-row">
          <span class="tag">Cloud Run</span>
          <span class="tag">Cloud Pub/Sub</span>
          <span class="tag">Cloud Functions</span>
          <span class="tag">BigQuery</span>
          <span class="tag">Cloud Storage</span>
          <span class="tag">Cloud Logging &amp; Monitoring</span>
        </div>
      </div>
    </div>
  </section>

  <!-- USE CASES & API -->
  <section>
    <div class="two-col">
      <div class="card">
        <div class="section-label">Use cases</div>
        <h3>Built for insurance, financial services and fleet platforms.</h3>
        <ul class="feature-list">
          <li>
            <span class="feature-bullet"></span>
            <span><strong>Usage-based insurance:</strong> ingest driving behavior and loss signals to power
              dynamic pricing, underwriting and claims triage.</span>
          </li>
          <li>
            <span class="feature-bullet"></span>
            <span><strong>Fleet safety analytics:</strong> combine harsh events, video and coaching outcomes to
              reduce incident frequency and severity.</span>
          </li>
          <li>
            <span class="feature-bullet"></span>
            <span><strong>Embedded finance:</strong> leverage live utilization and maintenance data to
              underwrite working-capital products for carriers.</span>
          </li>
        </ul>
      </div>

      <div class="card">
        <div class="section-label">Developer experience</div>
        <h3>Simple REST API, webhooks and clear documentation.</h3>
        <p>
          Terminal exposes REST endpoints and webhook subscriptions that follow familiar financial-API
          patterns. Teams integrate once, select vendors and fleets, then immediately start streaming data
          into their own Google Cloud or on-prem environments.
        </p>

        <div class="code-block">
          POST /v1/vehicles/{id}/trips<br />
          Authorization: Bearer &lt;api_key&gt;<br />
          <br />
          Response 200 OK<br />
          {<br />
          &nbsp;&nbsp;"vehicle_id": "trk_123",<br />
          &nbsp;&nbsp;"start_time": "2025-06-15T08:01:00Z",<br />
          &nbsp;&nbsp;"end_time": "2025-06-15T09:12:30Z",<br />
          &nbsp;&nbsp;"distance_km": 86.4,<br />
          &nbsp;&nbsp;"events": [ "speeding", "harsh_brake" ]<br />
          }
        </div>

        <p class="footnote">
          Terminal can deliver data directly into customer-owned Google Cloud projects via Pub/Sub topics or
          secure APIs, supporting joint architectures with insurers and fintechs.
        </p>
      </div>
    </div>
  </section>

  <!-- CONTACT -->
  <section id="contact">
    <div class="section-label">Contact</div>
    <div class="card">
      <div class="contact">
        <div>
          <h3>Work with Terminal</h3>
          <p>
            Terminal is headquartered in Canada and works with customers across North America.
            To discuss a proof-of-concept or Google Cloud deployment, contact us directly.
          </p>
          <p style="margin-top: 8px;">
            Email:
            <a href="mailto:connor@withterminal.info">connor@withterminal.info</a><br />
            Website:
            <a href="https://withterminal.info/" target="_blank" rel="noopener">withterminal.info</a>
          </p>
        </div>
        <div>
          <a href="mailto:connor@withterminal.info?subject=Terminal%20x%20Google%20Cloud" class="btn-primary">
            Talk to us about Google Cloud
          </a>
        </div>
      </div>
    </div>
  </section>
</main>

<footer>
  <span>© <span id="year"></span> Terminal. All rights reserved.</span>
  <span>Universal API for commercial trucking telematics, built on Google Cloud.</span>
</footer>

<script>
  // Dynamic background: simple moving node network
  (function () {
    const canvas = document.getElementById("bg-canvas");
    const ctx = canvas.getContext("2d");
    let width, height, particles;

    function resize() {
      width = canvas.width = window.innerWidth;
      height = canvas.height = window.innerHeight;
      initParticles();
    }

    function initParticles() {
      const count = Math.floor((width * height) / 25000); // density
      particles = [];
      for (let i = 0; i < count; i++) {
        particles.push({
          x: Math.random() * width,
          y: Math.random() * height,
          vx: (Math.random() - 0.5) * 0.6,
          vy: (Math.random() - 0.5) * 0.6
        });
      }
    }

    function step() {
      ctx.clearRect(0, 0, width, height);

      // Draw particles
      ctx.fillStyle = "rgba(148, 163, 184, 0.55)";
      particles.forEach(p => {
        p.x += p.vx;
        p.y += p.vy;

        if (p.x < 0 || p.x > width) p.vx *= -1;
        if (p.y < 0 || p.y > height) p.vy *= -1;

        ctx.beginPath();
        ctx.arc(p.x, p.y, 1.4, 0, Math.PI * 2);
        ctx.fill();
      });

      // Draw connections
      for (let i = 0; i < particles.length; i++) {
        for (let j = i + 1; j < particles.length; j++) {
          const p1 = particles[i];
          const p2 = particles[j];
          const dx = p1.x - p2.x;
          const dy = p1.y - p2.y;
          const dist = Math.sqrt(dx * dx + dy * dy);
          if (dist < 120) {
            const alpha = 1 - dist / 120;
            ctx.strokeStyle = "rgba(79, 209, 255," + (0.18 * alpha) + ")";
            ctx.lineWidth = 0.7;
            ctx.beginPath();
            ctx.moveTo(p1.x, p1.y);
            ctx.lineTo(p2.x, p2.y);
            ctx.stroke();
          }
        }
      }

      requestAnimationFrame(step);
    }

    window.addEventListener("resize", resize);
    resize();
    step();
  })();

  // Footer year
  document.getElementById("year").textContent = new Date().getFullYear();
</script>
</body>
</html>
