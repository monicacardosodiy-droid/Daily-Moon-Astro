<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sky Right Now</title>

<!-- Astronomy Engine -->
<script src="https://cdn.jsdelivr.net/npm/astronomy-engine/astronomy.browser.min.js"></script>

<style>

    :root {
        --bg:        #f4f0e6;
        --card-bg:   #f4f0e6;
        --ink:       #1c1a17;
        --ink-soft:  #4a463f;
        --grey:      #a39c8e;
        --grey-line: #ddd6c6;
        --accent:    #c1703f;
        --box-line:  #cfc7b3;
    }

    * {
        box-sizing: border-box;
    }

    body {
        margin: 0;
        padding: 40px 20px;
        background: var(--bg);
        color: var(--ink);
        font-family: "Iowan Old Style", "Palatino Linotype", Georgia, "Times New Roman", serif;
    }

    .sky-widget {
        max-width: 480px;
        margin: auto;
    }

    /* =====================================================
       HEADER
       ===================================================== */

    .title {
        font-size: 32px;
        font-weight: 700;
        letter-spacing: -0.01em;
        margin: 0 0 12px;
        color: var(--ink);
    }

    .meta {
        font-family: "SFMono-Regular", "IBM Plex Mono", Menlo, Consolas, monospace;
        font-size: 11px;
        letter-spacing: 0.14em;
        text-transform: uppercase;
        color: var(--grey);
        margin-bottom: 28px;
        line-height: 1.7;
    }

    /* =====================================================
       PLANET LIST
       ===================================================== */

    .planet-list {
        width: 100%;
        border-top: 1px solid var(--grey-line);
    }

    .planet {
        display: grid;
        grid-template-columns: 44px 1fr auto;
        align-items: center;
        column-gap: 14px;
        padding: 15px 0;
        border-bottom: 1px solid var(--grey-line);
    }

    .planet-symbol {
        width: 32px;
        height: 32px;
        border: 1px solid var(--box-line);
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 15px;
        color: var(--ink-soft);
        font-family: Georgia, serif;
        background: rgba(255,255,255,0.35);
    }

    .planet-name {
        font-size: 17px;
        color: var(--ink);
        font-weight: 400;
    }

    .planet-right {
        display: flex;
        align-items: baseline;
        justify-content: flex-end;
        gap: 10px;
        white-space: nowrap;
    }

    .sign {
        font-size: 16px;
        font-style: italic;
        color: var(--ink-soft);
    }

    .degree {
        font-size: 14px;
        color: var(--grey);
        min-width: 58px;
        text-align: right;
        font-variant-numeric: tabular-nums;
    }

    .retrograde {
        font-size: 10px;
        font-weight: 700;
        color: var(--accent);
        border: 1px solid var(--accent);
        padding: 1px 4px;
        letter-spacing: 0.02em;
    }

    /* =====================================================
       MOBILE
       ===================================================== */

    @media (max-width: 420px) {
        body {
            padding: 26px 16px;
        }
        .title {
            font-size: 27px;
        }
        .planet-symbol {
            width: 28px;
            height: 28px;
            font-size: 13px;
        }
        .planet-name {
            font-size: 15px;
        }
        .sign {
            font-size: 14px;
        }
        .degree {
            font-size: 12px;
            min-width: 48px;
        }
    }

</style>
</head>


<body>

<div class="sky-widget">

    <h1 class="title">Sky right now</h1>

    <div id="meta" class="meta">Loading…</div>

    <div id="planet-list" class="planet-list">
        <!-- JavaScript inserts planet rows here -->
    </div>

</div>


<script>

    /* =====================================================
       ZODIAC SIGNS
       ===================================================== */

    const zodiac = [
        { name: "Aries" }, { name: "Taurus" }, { name: "Gemini" },
        { name: "Cancer" }, { name: "Leo" }, { name: "Virgo" },
        { name: "Libra" }, { name: "Scorpio" }, { name: "Sagittarius" },
        { name: "Capricorn" }, { name: "Aquarius" }, { name: "Pisces" }
    ];

    /* =====================================================
       PLANETS  (glyphs rendered as text inside boxes)
       ===================================================== */

    const planets = [
        { name: "Sun",     body: Astronomy.Body.Sun,     symbol: "☉" },
        { name: "Moon",    body: Astronomy.Body.Moon,    symbol: "☾" },
        { name: "Mercury", body: Astronomy.Body.Mercury, symbol: "☿" },
        { name: "Venus",   body: Astronomy.Body.Venus,   symbol: "♀" },
        { name: "Mars",    body: Astronomy.Body.Mars,    symbol: "♂" },
        { name: "Jupiter", body: Astronomy.Body.Jupiter, symbol: "♃" },
        { name: "Saturn",  body: Astronomy.Body.Saturn,  symbol: "♄" },
        { name: "Uranus",  body: Astronomy.Body.Uranus,  symbol: "♅" },
        { name: "Neptune", body: Astronomy.Body.Neptune, symbol: "♆" },
        { name: "Pluto",   body: Astronomy.Body.Pluto,   symbol: "♇" }
    ];

    /* =====================================================
       ZODIAC POSITION
       ===================================================== */

    function getZodiacPosition(body, date) {

        const vector = Astronomy.GeoVector(body, date, true);
        const ecliptic = Astronomy.Ecliptic(vector);

        let degrees = ecliptic.elon;

        const signIndex = Math.floor(degrees / 30);
        const degreeInSign = degrees - (signIndex * 30);
        const wholeDegree = Math.floor(degreeInSign);
        const minutes = Math.floor((degreeInSign - wholeDegree) * 60);

        return {
            sign: zodiac[signIndex],
            degree: wholeDegree,
            minutes: minutes
        };
    }

    /* =====================================================
       RETROGRADE DETECTION
       ===================================================== */

    function isRetrograde(body, date) {

        const before = new Date(date.getTime() - (60 * 60 * 1000));
        const after  = new Date(date.getTime() + (60 * 60 * 1000));

        const beforeVector = Astronomy.GeoVector(body, before, true);
        const afterVector  = Astronomy.GeoVector(body, after, true);

        const beforeEcl = Astronomy.Ecliptic(beforeVector);
        const afterEcl  = Astronomy.Ecliptic(afterVector);

        let movement = afterEcl.elon - beforeEcl.elon;

        if (movement > 180) movement -= 360;
        if (movement < -180) movement += 360;

        return movement < 0;
    }

    /* =====================================================
       META LINE (date · timezone · UTC offset)
       ===================================================== */

    function updateMeta() {

        const now = new Date();

        const tz = Intl.DateTimeFormat().resolvedOptions().timeZone;

        const dateText = new Intl.DateTimeFormat("en-US", {
            weekday: "long",
            month: "long",
            day: "numeric",
            timeZone: tz
        }).format(now);

        // Compute UTC offset for the local timezone, formatted like "UTC-4"
        const offsetMinutes = -now.getTimezoneOffset();
        const offsetHours = offsetMinutes / 60;
        const sign = offsetHours >= 0 ? "+" : "-";
        const offsetLabel = "UTC" + sign + Math.abs(offsetHours);

        document.getElementById("meta").innerHTML =
            dateText.toUpperCase() + " &middot; " + tz.toUpperCase() + " &middot; " + offsetLabel;
    }

    /* =====================================================
       ROW BUILDER
       ===================================================== */

    function createPlanetRow(planet, date) {

        const position = getZodiacPosition(planet.body, date);
        const retro = isRetrograde(planet.body, date);

        const row = document.createElement("div");
        row.className = "planet";

        row.innerHTML = `
            <div class="planet-symbol">${planet.symbol}</div>
            <div class="planet-name">${planet.name}</div>
            <div class="planet-right">
                <span class="sign">${position.sign.name}</span>
                ${retro ? '<span class="retrograde">R</span>' : ''}
                <span class="degree">${position.degree}&deg;${String(position.minutes).padStart(2, "0")}&prime;</span>
            </div>
        `;

        return row;
    }

    /* =====================================================
       BUILD WIDGET
       ===================================================== */

    function updateSky() {

        const now = new Date();

        updateMeta();

        const container = document.getElementById("planet-list");
        container.innerHTML = "";

        planets.forEach(planet => {
            container.appendChild(createPlanetRow(planet, now));
        });
    }

    updateSky();
    setInterval(updateSky, 60000);

</script>

</body>
</html>
