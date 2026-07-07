# Prompt de diseño — identidad visual de Lumea

Prompt listo para pegar en Claude (claude.ai, Claude Code) o en cualquier
herramienta de diseño con IA, para refinar la dirección visual elegida («Sol»,
decisión D-10 del SDD) sin cambiar su carácter.

---

```
You are the design lead for "Lumea", a free web/PWA app (Spanish, Uruguay)
where independent household workers — cleaners, babysitters, car detailers —
and the homeowners who hire them track worked hours and expenses, approve
them, and close a monthly payment report. Users are non-technical and use
the app mostly on their phones. The brand promise is trust through clarity:
every number in the app can be explained (who logged it, who approved it,
at what rate).

BRAND
- Name: Lumea (invented, evokes "lumen"/light — clarity in the accounts).
- Chosen direction to refine, NOT replace: "Sol" — warm paper background,
  golden-amber accent (currently #C77E14 on #FBF7F0 paper, dark ink #2A241C),
  a small sun as the logo mark, friendly rounded humanist typography,
  lowercase wordmark "lumea".
- Personality: warm, honest, simple, optimistic. Never corporate, never
  childish, never fintech-cold.

YOUR TASK
Produce 3 refined variations of this same "Sol" direction (do not propose
new directions in other hues):
1. A refined color system: paper/surface tones, ink tones, the golden
   accent, and soft fills — as design tokens with hex values, for BOTH
   light and dark themes (dark must be designed, not inverted).
2. Logo exploration: 4-6 sun mark concepts (geometric rays, half sun
   rising over a horizon/roofline, sun through a window, sun + clock
   hybrid, abstract "a" with sun counter). Simple enough to work at
   24px and as an app icon. Show wordmark lockups.
3. Typography: pair a characterful display face and a highly readable
   body face (free / open-source), with a type scale for mobile.
4. Apply it: mock up these 3 screens on a phone frame, all copy in
   Spanish (Rioplatense "vos" forms): (a) home with per-client month
   summary, (b) time-entry form with live amount calculation
   ("4 horas × $ 290 = $ 1.160"), (c) monthly report with a stacked
   bar chart of hours per day, one color per client.

HARD CONSTRAINTS
- Status colors are reserved and must stay distinct from the brand
  accent: approved = green, pending = neutral gray chip, rejected = red.
- Chart client colors: one fixed color per client, must pass contrast
  (>= 3:1 vs surface) and color-vision-deficiency separation on both
  light and dark surfaces; never reuse status colors for clients.
- Currency format: "$ 1.160" (UYU, dot as thousands separator);
  dates DD/MM; all UI text in Spanish.
- Big touch targets, one primary action per screen, readable by users
  who do not use office software.
- Free-to-use fonts only; no stock photography; flat/vector style.

Deliver each variation with: token table (name, hex light, hex dark),
logo sheet, and the 3 phone screens. Explain in one short paragraph what
changed vs. the base and why it strengthens "trust through clarity".
```

---

**Cómo usarlo**: pegalo tal cual. Si la herramienta acepta imágenes, adjuntá
capturas del prototipo actual (`docs/maquetas/lumea-prototipo.html`, sistema
«Sol») para que la variación parta de lo existente. Cuando elijas una
variación, traé los tokens (tabla de colores) de vuelta a este repo y se
aplican al prototipo y al SDD.
