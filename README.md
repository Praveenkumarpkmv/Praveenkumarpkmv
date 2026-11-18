import re, os, math, urllib.request
OUT = "assets"
os.makedirs(OUT, exist_ok=True)

BG0, BG1, BONE, MUTE = "#05070D", "#0B1120", "#EDE6DC", "#7C8597"
COPPER, GOLD, EMBER = "#FF8A4C", "#FFD2A1", "#FF6B2C"
SANS = "'Helvetica Neue',Helvetica,Arial,'Segoe UI',sans-serif"
MONO = "ui-monospace,SFMono-Regular,Menlo,Consolas,'Liberation Mono',monospace"

CSS = f".m{{font-family:{MONO}}}.s{{font-family:{SANS}}}"

def svg(W, H, title, body, defs="", focus=0.7):
    return f'''<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 {W} {H}" width="{W}" height="{H}" role="img" aria-label="{title}">
<defs><style>{CSS}</style>
<clipPath id="clip"><rect width="{W}" height="{H}" rx="18"/></clipPath>
<linearGradient id="bgg" x1="0" y1="0" x2="0" y2="1"><stop offset="0" stop-color="{BG0}"/><stop offset="1" stop-color="{BG1}"/></linearGradient>
<pattern id="grid" width="60" height="60" patternUnits="userSpaceOnUse"><path d="M60 0H0V60" fill="none" stroke="#fff" stroke-opacity=".045"/></pattern>
<radialGradient id="fg" cx="{focus}" cy=".5" r=".75"><stop offset="0" stop-color="#fff"/><stop offset="1" stop-color="#000"/></radialGradient>
<mask id="fm"><rect width="{W}" height="{H}" fill="url(#fg)"/></mask>
<filter id="b6" x="-50%" y="-50%" width="200%" height="200%"><feGaussianBlur stdDeviation="6"/></filter>
<filter id="b2" x="-50%" y="-50%" width="200%" height="200%"><feGaussianBlur stdDeviation="2"/></filter>
{defs}</defs>
<g clip-path="url(#clip)"><rect width="{W}" height="{H}" fill="url(#bgg)"/><rect width="{W}" height="{H}" fill="url(#grid)" mask="url(#fm)"/>
{body}</g>
<rect x=".5" y=".5" width="{W-1}" height="{H-1}" rx="17.5" fill="none" stroke="#fff" stroke-opacity=".08"/>
</svg>'''

def orbit_node(cx, cy, rx, ry, rot, dur, begin, r=4):
    p = f"M{cx-rx},{cy} A{rx},{ry} 0 1 1 {cx+rx},{cy} A{rx},{ry} 0 1 1 {cx-rx},{cy}"
    return (f'<g transform="rotate({rot} {cx} {cy})"><ellipse cx="{cx}" cy="{cy}" rx="{rx}" ry="{ry}" fill="none" stroke="{COPPER}" stroke-opacity=".38"/>'
            f'<g><circle r="{r+4}" fill="{COPPER}" opacity=".35" filter="url(#b2)"/><circle r="{r}" fill="{GOLD}"/>'
            f'<animateMotion dur="{dur}s" begin="{begin}s" repeatCount="indefinite" path="{p}"/></g></g>')

def eclipse(cx, cy, r=104, orbit=True):
    s = f'''<circle cx="{cx}" cy="{cy}" r="{r*2.6}" fill="url(#glow)"/>
<circle cx="{cx}" cy="{cy}" r="{r*1.44}" fill="url(#corona)"><animate attributeName="opacity" values=".8;1;.8" dur="6s" repeatCount="indefinite"/></circle>
<circle cx="{cx}" cy="{cy}" r="{r}" fill="url(#disc)" stroke="url(#rim)" stroke-width="2"/>'''
    a0, a1 = math.radians(200), math.radians(292)
    x0, y0 = cx + r*math.cos(a0), cy + r*math.sin(a0)
    x1, y1 = cx + r*math.cos(a1), cy + r*math.sin(a1)
    arc = f"M{x0:.1f},{y0:.1f} A{r},{r} 0 0 1 {x1:.1f},{y1:.1f}"
    s += f'<path d="{arc}" fill="none" stroke="{COPPER}" stroke-width="7" opacity=".7" filter="url(#b6)"/><path d="{arc}" fill="none" stroke="{GOLD}" stroke-width="3" stroke-linecap="round"/>'
    if orbit:
        s += (f'<circle cx="{cx}" cy="{cy}" r="{r*1.23:.0f}" fill="none" stroke="{GOLD}" stroke-opacity=".45" stroke-dasharray="1 6">'
              f'<animateTransform attributeName="transform" type="rotate" from="0 {cx} {cy}" to="360 {cx} {cy}" dur="120s" repeatCount="indefinite"/></circle>'
              f'<circle cx="{cx}" cy="{cy}" r="{r*1.75:.0f}" fill="none" stroke="#fff" stroke-opacity=".14" stroke-dasharray="2 12">'
              f'<animateTransform attributeName="transform" type="rotate" from="360 {cx} {cy}" to="0 {cx} {cy}" dur="180s" repeatCount="indefinite"/></circle>')
        s += orbit_node(cx, cy, r*2.07, r*.63, -18, 16, 0) + orbit_node(cx, cy, r*2.07, r*.63, -18, 16, -8, 3)
    return s

def eclipse_defs(cx, cy, r=104):
    return f'''<radialGradient id="glow" gradientUnits="objectBoundingBox"><stop offset="0" stop-color="{COPPER}" stop-opacity=".30"/><stop offset=".5" stop-color="{COPPER}" stop-opacity=".07"/><stop offset="1" stop-color="{COPPER}" stop-opacity="0"/></radialGradient>
<radialGradient id="corona"><stop offset=".66" stop-color="{COPPER}" stop-opacity="0"/><stop offset=".70" stop-color="#FFB27A" stop-opacity=".9"/><stop offset=".80" stop-color="{EMBER}" stop-opacity=".32"/><stop offset="1" stop-color="{EMBER}" stop-opacity="0"/></radialGradient>
<radialGradient id="disc" cx=".35" cy=".3" r=".9"><stop offset="0" stop-color="#141C30"/><stop offset="1" stop-color="#03050A"/></radialGradient>
<linearGradient id="rim" x1="0" y1="0" x2="1" y2="1"><stop offset="0" stop-color="{GOLD}"/><stop offset=".5" stop-color="{COPPER}" stop-opacity=".2"/><stop offset="1" stop-color="{EMBER}" stop-opacity=".7"/></linearGradient>'''

# ---------- custom monoline lettering (no fonts needed) ----------
L = {
 "P": "M0 72V0H28A18 18 0 0 1 28 36H0",
 "R": "M0 72V0H28A18 18 0 0 1 28 36H0M24 36L48 72",
 "A": "M0 72L25 0L50 72M8.3 48H41.7",
 "V": "M0 0L25 72L50 0",
 "E": "M50 0H0V72H50M0 36H40",
 "N": "M0 72V0L50 72V0",
 "K": "M0 0V72M50 0L0 40M16 27.5L50 72",
 "U": "M0 0V50A25 22 0 0 0 50 50V0",
 "M": "M0 72V0L25 44L50 0V72",
}
def wordmark(word, x, y, s):
    g = f'<g transform="translate({x},{y}) scale({s})" fill="none" stroke="url(#ng)" stroke-width="5.5" stroke-linejoin="miter">'
    for i, ch in enumerate(word):
        g += f'<path transform="translate({i*72},0)" d="{L[ch]}"/>'
    w = (len(word)-1)*72 + 50
    return g + f'</g><rect x="{x+w*s+16:.0f}" y="{y+72*s-9:.0f}" width="9" height="9" fill="{COPPER}"/>'

def hero():
    W, H, cx, cy = 1200, 480, 965, 236
    d = eclipse_defs(cx, cy) + f'<linearGradient id="ng" x1="0" y1="0" x2="0" y2="1"><stop offset="0" stop-color="{BONE}"/><stop offset="1" stop-color="#F4D9BC"/></linearGradient>'
    b = f'<line x1="540" y1="{cy}" x2="1176" y2="{cy}" stroke="#fff" stroke-opacity=".07"/><line x1="{cx}" y1="24" x2="{cx}" y2="456" stroke="#fff" stroke-opacity=".07"/>'
    b += eclipse(cx, cy)
    for x, y, dx, dy in [(28, 28, 1, 1), (1172, 28, -1, 1), (28, 452, 1, -1), (1172, 452, -1, -1)]:
        b += f'<path d="M{x} {y+dy*18}V{y}H{x+dx*18}" fill="none" stroke="#fff" stroke-opacity=".28"/>'
    b += f'<rect x="72" y="134" width="8" height="8" fill="{COPPER}"/><text x="94" y="142" class="m" font-size="13" letter-spacing="5" fill="{MUTE}">DIGITAL IDENTITY</text>'
    b += wordmark("PRAVEENKUMAR", 72, 184, .8)
    b += f'<text x="72" y="302" class="s" font-size="22" letter-spacing="8" fill="{BONE}">CREATIVE DEVELOPER</text>'
    b += f'<text x="72" y="338" class="s" font-size="15" letter-spacing="6" fill="{GOLD}" fill-opacity=".85">WEB  ·  CLOUD  ·  NETWORKS</text>'
    b += f'<line x1="72" y1="392" x2="560" y2="392" stroke="#fff" stroke-opacity=".12"/><line x1="72" y1="392" x2="124" y2="392" stroke="{COPPER}" stroke-width="2"/>'
    b += f'<circle cx="76" cy="428" r="4" fill="{COPPER}"/><circle cx="76" cy="428" r="4" fill="none" stroke="{COPPER}"><animate attributeName="r" values="4;12" dur="2.4s" repeatCount="indefinite"/><animate attributeName="opacity" values=".8;0" dur="2.4s" repeatCount="indefinite"/></circle>'
    b += f'<text x="96" y="433" class="m" font-size="13" letter-spacing="4" fill="{MUTE}">CURRENTLY BUILDING</text><text x="560" y="433" class="m" font-size="13" letter-spacing="4" fill="{MUTE}" text-anchor="end">INDIA</text>'
    return svg(W, H, "Praveenkumar — Creative Developer: Web, Cloud, Networks", b, d)

def header(n, title, tag):
    b = f'<text x="40" y="58" class="m" font-size="22" fill="{COPPER}">{n}</text><text x="96" y="60" class="s" font-size="40" letter-spacing="10" fill="{BONE}" font-weight="300">{title}</text>'
    b += f'<text x="1160" y="56" class="m" font-size="13" letter-spacing="5" fill="{MUTE}" text-anchor="end">{tag}</text>'
    b += f'<line x1="40" y1="80" x2="1160" y2="80" stroke="#fff" stroke-opacity=".12"/><line x1="40" y1="80" x2="136" y2="80" stroke="{COPPER}" stroke-width="2"/>'
    return svg(1200, 100, f"{n} {title}", b, focus=0.1)

# ---------- ecosystem ----------
SI = "https://raw.githubusercontent.com/simple-icons/simple-icons/16.38.0/icons/{}.svg"
def icon(slug):
    try:
        t = urllib.request.urlopen(SI.format(slug), timeout=20).read().decode()
        return re.search(r' d="([^"]+)"', t).group(1)
    except Exception:
        return None

def tile(x, y, label, slug=None, mono=None):
    s = f'<rect x="{x}" y="{y}" width="72" height="72" rx="16" fill="#fff" fill-opacity=".04" stroke="#fff" stroke-opacity=".11"/>'
    p = icon(slug) if slug else None
    if p:
        s += f'<path transform="translate({x+20},{y+20}) scale(1.333)" d="{p}" fill="{BONE}"/>'
    elif mono == "aws":
        s += f'<text x="{x+36}" y="{y+40}" class="s" font-size="25" font-weight="700" fill="{BONE}" text-anchor="middle">aws</text><path d="M{x+20} {y+50}Q{x+36} {y+60} {x+52} {y+50}" fill="none" stroke="{COPPER}" stroke-width="3" stroke-linecap="round"/>'
    else:
        s += f'<text x="{x+36}" y="{y+45}" class="s" font-size="25" font-weight="700" fill="{BONE}" text-anchor="middle">{"VS" if mono == "vs" else label[:2]}</text>'
    return s + f'<text x="{x+36}" y="{y+94}" class="m" font-size="12" fill="{MUTE}" text-anchor="middle">{label}</text>'

def ecosystem():
    rows = [("FRONTEND", [("HTML", "html5"), ("CSS", "css"), ("JavaScript", "javascript"), ("React", "react"), ("Tailwind", "tailwindcss")]),
            ("CLOUD & SYSTEMS", [("AWS", "aws"), ("Linux", "linux"), ("Bash", "gnubash")]),
            ("WORKFLOW", [("Git", "git"), ("GitHub", "github"), ("VS Code", "vs"), ("npm", "npm")])]
    b = ""
    for i, (name, items) in enumerate(rows):
        y = 28 + i*130
        b += f'<line x1="40" y1="{y}" x2="1160" y2="{y}" stroke="#fff" stroke-opacity=".1"/><text x="40" y="{y+34}" class="m" font-size="14" fill="{COPPER}">0{i+1}</text>'
        b += f'<text x="40" y="{y+68}" class="s" font-size="26" letter-spacing="5" fill="{BONE}" font-weight="300">{name.replace("&", "&amp;")}</text>'
        for j, (lab, key) in enumerate(items):
            b += tile(420 + j*92, y + 18, lab, slug=None if key in ("aws", "vs") else key, mono=key if key in ("aws", "vs") else None)
    return svg(1200, 432, "Technology ecosystem: HTML, CSS, JavaScript, React, Tailwind, AWS, Linux, Bash, Git, GitHub, VS Code, npm", b, focus=0.9)

# ---------- project plates ----------
def art_sushi(cx, cy):
    return f'''<circle cx="{cx}" cy="{cy}" r="148" fill="none" stroke="#FF8B7A" stroke-opacity=".35" stroke-dasharray="2 10"><animateTransform attributeName="transform" type="rotate" from="0 {cx} {cy}" to="360 {cx} {cy}" dur="90s" repeatCount="indefinite"/></circle>
<circle cx="{cx}" cy="{cy}" r="112" fill="#0B1511" stroke="#2F5442" stroke-width="2"/>
<circle cx="{cx}" cy="{cy}" r="92" fill="#F1EBDF"/><circle cx="{cx}" cy="{cy}" r="82" fill="none" stroke="#D8D0C0" stroke-dasharray="3 7"/>
<circle cx="{cx}" cy="{cy}" r="46" fill="url(#salmon)"/>
<path d="M{cx-34} {cy-8}Q{cx} {cy-26} {cx+34} {cy-6}M{cx-38} {cy+10}Q{cx} {cy-6} {cx+38} {cy+12}M{cx-30} {cy+28}Q{cx} {cy+14} {cx+30} {cy+30}" fill="none" stroke="#FFD9CF" stroke-opacity=".6" stroke-width="2"/>
<circle cx="{cx+16}" cy="{cy-12}" r="11" fill="#A8C98F"/>'''

def art_yogurt(cx, cy):
    tri = f"M{cx-64},{cy+6}L{cx+64},{cy+6}L{cx},{cy+140}Z"
    hatch = "".join(f'<line x1="{cx-80+i*22}" y1="{cy}" x2="{cx-20+i*22}" y2="{cy+150}" stroke="{COPPER}" stroke-opacity=".35"/><line x1="{cx+80-i*22}" y1="{cy}" x2="{cx+20-i*22}" y2="{cy+150}" stroke="{COPPER}" stroke-opacity=".35"/>' for i in range(8))
    sw = ""
    for (rx, ry, dy) in [(76, 27, -2), (60, 25, -36), (42, 22, -68)]:
        sw += f'<ellipse cx="{cx}" cy="{cy+dy}" rx="{rx}" ry="{ry}" fill="url(#mint)" stroke="#fff" stroke-opacity=".4"/><path d="M{cx-rx*.6:.0f} {cy+dy-ry*.2:.0f}Q{cx} {cy+dy-ry*.9:.0f} {cx+rx*.6:.0f} {cy+dy-ry*.2:.0f}" fill="none" stroke="#fff" stroke-opacity=".55" stroke-width="2"/>'
    return f'''<circle cx="{cx}" cy="{cy}" r="148" fill="none" stroke="#9EE6C8" stroke-opacity=".3" stroke-dasharray="2 10"><animateTransform attributeName="transform" type="rotate" from="360 {cx} {cy}" to="0 {cx} {cy}" dur="90s" repeatCount="indefinite"/></circle>
<clipPath id="cone"><path d="{tri}"/></clipPath><path d="{tri}" fill="#1A140F" stroke="{GOLD}" stroke-opacity=".6"/><g clip-path="url(#cone)">{hatch}</g>{sw}
<ellipse cx="{cx}" cy="{cy-94}" rx="16" ry="14" fill="url(#mint)"/><circle cx="{cx+4}" cy="{cy-116}" r="9" fill="#FF8FA3"/>'''

def art_borewell(cx, cy):
    x0, x1 = cx-150, cx+150
    bands = [(24, 96, "#3B2D1F"), (96, 160, "#2B251F"), (160, 222, "#1F2528"), (222, 296, "#0F2C35")]
    s = f'<clipPath id="strata"><rect x="{x0}" y="24" width="300" height="272" rx="14"/></clipPath><g clip-path="url(#strata)">'
    for a, b_, c in bands:
        s += f'<rect x="{x0}" y="{a}" width="300" height="{b_-a}" fill="{c}"/><line x1="{x0}" y1="{a}" x2="{x1}" y2="{a}" stroke="#fff" stroke-opacity=".09"/>'
    s += f'<ellipse cx="{cx}" cy="262" rx="104" ry="14" fill="#52D1C0" fill-opacity=".22"/>'
    s += f'<rect x="{cx-8}" y="24" width="16" height="214" fill="#04060A" stroke="#52D1C0" stroke-opacity=".6"/>'
    s += "".join(f'<line x1="{cx+14}" y1="{y}" x2="{cx + (26 if i % 2 == 0 else 20)}" y2="{y}" stroke="{MUTE}"/>' for i, y in enumerate(range(48, 238, 24)))
    s += f'<circle cx="{cx}" cy="262" r="6" fill="none" stroke="#7BE3D4"><animate attributeName="r" values="6;56" dur="3s" repeatCount="indefinite"/><animate attributeName="opacity" values=".9;0" dur="3s" repeatCount="indefinite"/></circle>'
    s += f'<path d="M0,-12C8,0 10,6 0,12C-10,6 -8,0 0,-12Z" fill="#7BE3D4"><animateTransform attributeName="transform" type="translate" values="{cx},70;{cx},250" dur="3s" repeatCount="indefinite"/><animate attributeName="opacity" values="0;1;1;0" dur="3s" repeatCount="indefinite"/></path></g>'
    return s

def plate(num, tag, l1, l2, art, defs, accent, flip, title):
    W, H = 1200, 320
    ax, tx, anc = (300, 1128, "end") if flip else (900, 72, "start")
    b = f'<circle cx="{ax}" cy="160" r="230" fill="{accent}" fill-opacity=".10" filter="url(#b6)"/>' + art(ax, 160)
    b += f'<text x="{tx}" y="78" class="m" font-size="14" letter-spacing="5" fill="{accent}" text-anchor="{anc}">{num} / {tag}</text>'
    b += f'<text x="{tx-2 if not flip else tx}" y="156" class="s" font-size="64" letter-spacing="8" fill="{BONE}" font-weight="300" text-anchor="{anc}">{l1}</text>'
    b += f'<text x="{tx-2 if not flip else tx}" y="226" class="s" font-size="64" letter-spacing="8" fill="{BONE}" font-weight="300" text-anchor="{anc}">{l2}</text>'
    x2 = tx + (-120 if flip else 120)
    b += f'<line x1="{tx}" y1="262" x2="{x2}" y2="262" stroke="{accent}" stroke-width="2"/><text x="{tx + (-136 if flip else 136)}" y="267" class="m" font-size="13" letter-spacing="4" fill="{MUTE}" text-anchor="{anc}">VIEW REPOSITORY</text>'
    return svg(W, H, title, b, defs, focus=0.25 if flip else 0.75)

def footer():
    cx, cy = 600, 400
    d = eclipse_defs(cx, cy, 170)
    b = eclipse(cx, cy, 170, orbit=False)
    b += f'<line x1="540" y1="70" x2="660" y2="70" stroke="{COPPER}" stroke-width="2"/><text x="600" y="112" class="m" font-size="16" letter-spacing="10" fill="{BONE}" fill-opacity=".75" text-anchor="middle">PRAVEENKUMARPKMV</text>'
    return svg(1200, 280, "Praveenkumar — end of profile", b, d, focus=0.5)

def write(name, s):
    open(f"{OUT}/{name}", "w", encoding="utf-8").write(s)

write("hero.svg", hero())
for n, t, tag in [("01", "ABOUT", "EDITORIAL"), ("02", "ECOSYSTEM", "TOOLS OF THE TRADE"), ("03", "SELECTED WORK", "PROJECTS"), ("04", "ANALYTICS", "ACTIVITY"), ("05", "CONNECT", "SAY HELLO")]:
    write("section-" + {"SELECTED WORK":"work"}.get(t, t.lower()) + ".svg", header(n, t, tag))
write("ecosystem.svg", ecosystem())
write("project-sushi.svg", plate("01", "CULINARY", "SUSHI", "STUDIO", art_sushi,
      '<radialGradient id="salmon" cx=".4" cy=".35"><stop offset="0" stop-color="#FFB5A3"/><stop offset="1" stop-color="#FF6B57"/></radialGradient>', "#FF8B7A", False, "Sushi Studio"))
write("project-yogurt.svg", plate("02", "STOREFRONT", "FROZEN", "YOGURT", art_yogurt,
      '<linearGradient id="mint" x1="0" y1="0" x2="0" y2="1"><stop offset="0" stop-color="#F7F1E8"/><stop offset="1" stop-color="#9EE6C8"/></linearGradient>', "#9EE6C8", True, "Frozen Yogurt"))
write("project-borewell.svg", plate("03", "WATER", "BOREWELL", "SOLUTIONS", art_borewell, "", "#52D1C0", False, "Borewell Solutions"))
write("footer.svg", footer())
print("Wrote:", ", ".join(sorted(os.listdir(OUT))))
