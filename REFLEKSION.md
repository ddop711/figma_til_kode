Skriv jeres fælles refleksion direkte i denne fil. Erstat hjælpeteksterne med jeres egne erfaringer, og slet Markdown-guiden og demoen inden aflevering. Skriv kort og konkret, og brug eksempler fra jeres egen kode.

Åbn forhåndsvisningen i VS Code med **Cmd + Shift + V** (Mac) eller **Ctrl + Shift + V** (Windows). Så ser I, hvordan Markdown bliver vist. På GitHub vises formateringen automatisk, når I åbner filen.

### Mini-guide til Markdown

- `# Titel` er dokumentets hovedoverskrift. Brug kun én.
- `## Afsnit` og `### Underafsnit` giver overskrifter i flere niveauer.
- `**vigtig tekst**` bliver til **vigtig tekst**.
- En bindestreg efterfulgt af et mellemrum laver en punktopstilling som denne.
- Skriv kode inde i en sætning mellem enkelte backticks, fx `getTeamMembers()`.
- Links skrives sådan: `[Astros dokumentation](https://docs.astro.build/)`.
- Lav et nyt afsnit med en tom linje. Brug også en tom linje før og efter lister og kodeblokke.

En kodeblok starter og slutter med tre backticks. Skriv sproget efter de første, fx `js`, `css`, `html` eller `astro`. Se et eksempel i filens kildekode nedenfor.

### Kort demo – sådan kan tekst, kode og link kombineres

> Dette er et opdigtet eksempel på formen, ikke en færdig refleksion eller et ekstra krav.

Vi flyttede datahentningen til en fælles funktion, så endpointet kun skal vedligeholdes ét sted.

```js
export function getServices() {
  return apiFetch("https://ftk-api.pages.dev/services");
}
```

I komponenten kalder vi `getServices()`. Vi kontrollerede, at de samme servicetitler blev vist før og efter ændringen. Næste skridt er at undersøge, hvad der sker, hvis API'et returnerer en fejl.

Reference: [Datahentning i Astro](https://docs.astro.build/en/guides/data-fetching/).

---

# Refleksion – Figma til kode

_Gruppemedlemmer:_ Mikkel Ekhart Storm & Denzel Daniel Orbigo Padilla

## Eksempel 1: Popover API og CSS-nesting

### Hvor og hvorfor?

Vi bruger Popover API i src/components/Header.astro til login-boksen. Det smarte er, at vi slipper for JavaScript til at åbne og lukke vinduet, fordi browseren selv lukker den, når man klikker ved siden af eller trykker ESC. Vi brugte samtidig CSS-nesting for at holde stylingen samlet inde i komponenten.

### Relevant kode

```
<!-- src/components/Header.astro -->
<button class="login-btn" popovertarget="login-popover" type="button">
  Login
</button>

<div id="login-popover" popover>
  <div class="popover-content">
    <h3>Sign In</h3>
    <form class="login-form">...</form>
  </div>
</div>

#login-popover {
width: min(340px, 90vw);
background-color: var(--color-ui-primary);

.popover-content {
display: flex;
flex-direction: column;
gap: var(--space-md);
}
}
```

### Afprøvning og ændringer

- _Vi testede:_ Vi testede popoveren i Chrome på bærbar og i DevTools mobilview.
- _Vi observerede:_ Al styling forsvandt helt på siden, fordi vi havde nestet &::backdrop inde i #login-popover, hvilket fik browseren til at smide hele stilarket ud.
- _Vi ændrede eller mangler:_ Vi rykkede #login-popover::backdrop ud på sit eget rod-niveau i CSS'en, så stylingen slog igennem igen.

## Eksempel 2: Fælles komponent

### Hvor og hvorfor?

På team-siden og about-siden skulle vi bruge en fælles card komponent, men have forskelligt layout og design. Det smarte ved at kunne gøre det, er at de deler samme html-indhold og struktur, og egentlig bare rette css'en til på deres egne sider.

### Relevant kode

```astro
<section class="team-preview">
  <div class="team-preview-grid">
    <div class="preview-intro">
      <span class="section-label">OUR TEAM</span>

      <h2>We champion<br />the bold</h2>

      <p>
        Our success stems from the synergy between dedicated teams and valued
        clients. We prioritize a company culture that fosters exceptional
        productivity.
      </p>

      <a href="/team" class="button">Meet the Team</a>
    </div>

    <div class="preview-members">
      {sarah && <TeamCard employee={sarah} />}
      {john && <TeamCard employee={john} />}
    </div>
  </div>
</section>
```

### Afprøvning og ændringer

- _Vi testede:_ Vi prøvede først at gøre så vores komponent havde al html og general css, og så give de 2 sider forskellig specifik css, men det blev ved med at drille, så vi endte bare med at beslutte os for bare at bruge det præcis samme card på begge sider.
- _Vi observerede:_ Vi prøvede også give komponenten all css til team-siden og prøve override det på about-siden, men det virkede heller ikke.
- _Vi ændrede eller mangler:_ Vi endte med at bruge det samme card på begge sider, fordi det var den løsning, der fungerede bedst for os.

## Eksempel 3: CSS Subgrid i layout

### Hvor og hvorfor?

Vi bruger det i src/layouts/Layout.astro til sidens overordnede layout-grid. Det gør, at alle vores sektioner arver de samme kolonner, så overskrifter og tekst automatisk flugter ned igennem siden uden ekstra wrappers.

### Relevant kode

```
/_ src/layouts/Layout.astro _/
.page-grid {
display: grid;
grid-template-columns:
[full-start] minmax(var(--space-md), 1fr)
[content-start] min(100% - (var(--space-md) \* 2), 1200px) [content-end]
minmax(var(--space-md), 1fr) [full-end];
}

.page-grid > :global(section) {
grid-column: full;
display: grid;
grid-template-columns: subgrid;
}

.page-grid > :global(section > \*) {
grid-column: content;
}
```

### Afprøvning og ændringer

- _Vi testede:_ Vi testede sidens bredde ved at trække i browseren på computeren.
- _Vi observerede:_ Teksten blev mast alt for langt ind mod midten, fordi vi både havde subgrid i layoutet og max-width: 1200px inde i sektionerne.
- _Vi ændrede eller mangler:_ Vi slettede de lokale bredder og ekstra divs i sektionerne, så det er subgriddet alene, der styrer bredden.

## Fallback og robusthed

- _Fallback/progressive enhancement:_ Vi bruger backdrop-filter: blur(6px) på popoverens baggrund. I browsere der ikke kan vise blur, vises den mørke farve (rgba(0,0,0,0.45)) bare i stedet. Vi har testet i Chrome og Firefox.
- _Defensive CSS:_ Vi har sat white-space: nowrap på login-knappen, så teksten ikke deler sig over to linjer på små skærme, og width: min(340px, 90vw) på popoveren, så den ikke laver vandret scroll på mobiler.
- _Global CSS og komponent-CSS:_ Vi har design-tokens og makrolayout liggende i tokens.css og Layout.astro. Al styling af selve komponenterne ligger lokalt inde i de enkelte `.astro`-filer, så de ikke forstyrrer hinanden.

## Brug af AI

- _Hvad brugte I den til?_
  Vi brugte Ai som en makker, vi kunne spørge til råds, når vi stødte på fejl, og til at tjekke op på regler i Popover API og subgrid f.eks.

- _Hvad ændrede eller fravalgte I i svaret?_
  Vi sagde nej til flere af de hurtige forslag:
  - Da AI foreslog bare at skjule links på mobil med display: none, afviste vi det, fordi folk på mobilen jo stadig skal kunne finde rundt.
  - Den gav os et tip om, hvorfor vores header crashede, men vi rettede og rykkede selv CSS'en ud af nestingen bagefter.

- _Hvad lærte I, og hvordan kontrollerede I løsningen?_
  Vi fandt ud af, at ::backdrop ikke altid må nestes i Astro, og vi lærte, hvordan popover-attributter virker direkte i browseren uden JS. Vi tjekkede hele tiden koden selv ved at åbne Chrome DevTools, teste forskellige skærmstørrelser og klikke rundt for at sikre, at det virkede i praksis. Samt en masse om hvordan nesting fungere, og skal sættes op korrekt.

```

```
