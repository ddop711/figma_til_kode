# Refleksion – Figma til kode

**Gruppemedlemmer:** Mikkel Ekhart Storm & Denzel Daniel Orbigo Padilla

## Eksempel 1: Popover API, CSS Anchor Positioning og CSS-nesting

### Hvor og hvorfor?

Vi bruger HTML Popover API sammen med CSS Anchor Positioning i `src/components/Header.astro` til login-boksen. Med Popover API slipper vi for JavaScript til at åbne og lukke vinduet, fordi browseren selv håndterer top-layer, ESC-tast og klik udenfor. Med Anchor Positioning binder vi login-panelet direkte fast under login-knappen uden faste skærmkoordinater, og CSS-nesting holder hele komponentens styling samlet.

### Relevant kode

```astro
<!-- src/components/Header.astro -->
<div class="login-trigger">
  <Button color="var(--color-surface-primary)" popovertarget="login-popover" text="Login" textColor="var(--color-neutrals-900)"/>
</div>

<div id="login-popover" popover>
  <div class="popover-content">
    <h3>Sign In</h3>
    <p>Enter your credentials.</p>
    <form class="login-form" onsubmit="event.preventDefault()">...</form>
  </div>
</div>
```

```css
/* src/components/Header.astro */
.login-trigger {
  white-space: nowrap;
  anchor-name: --login-anchor;
}

#login-popover {
  width: min(340px, 90vw);
  position: fixed;
  position-anchor: --login-anchor;
  top: anchor(bottom);
  right: anchor(right);
  margin-top: 0.5rem;

  .popover-content {
    display: flex;
    flex-direction: column;
    gap: var(--space-md, 1rem);
  }
}

#login-popover::backdrop {
  background-color: rgba(0, 0, 0, 0.45);
  backdrop-filter: blur(6px);
}
```

### Afprøvning og ændringer

- **Vi testede:** Vi testede popoveren og placeringen af login-boksen i Chrome på bærbar og i DevTools mobilview.
- **Vi observerede:** Tidligere forsvandt al styling på headeren, fordi `&::backdrop` var nestet inde i `#login-popover`, hvilket fik CSS-parseren til at afvise reglen. Derudover fulgte en fast `top`/`right`-placering ikke knappen på forskellige skærmbredder.
- **Vi ændrede eller mangler:** Vi rykkede `#login-popover::backdrop` ud på rod-niveau i CSS'en, tilføjede `white-space: nowrap` på `.login-trigger` og koblede boksen til knappen med `anchor-name` og `position-anchor`.

## Eksempel 2: Genbrugelig komponent med `variant`-prop (`CoreValues.astro`) og animation med `interpolate-size` (`Faq.astro`)

### Hvor og hvorfor?

På forsiden og about-siden (`src/pages/about.astro`) bruger vi den samme `CoreValues.astro`-komponent, men med en mørk variant på forsiden og en lys variant på `/about`. Ved at styre temaet med en `variant`-prop slipper vi for at duplikere HTML og grid-opsætning. I `src/components/Faq.astro` bruger vi desuden `interpolate-size: allow-keywords` og `::details-content` til at animere `<details>`-elementerne blødt mellem `0` og `auto` uden JavaScript.

### Relevant kode

```astro
<!-- src/components/CoreValues.astro -->
---
const { variant = "dark" } = Astro.props;
---

<section class={`core-values ${variant === "light" ? "light" : ""}`}>
  <!-- Indhold -->
</section>
```

```css
/* src/components/Faq.astro */
:root {
  interpolate-size: allow-keywords;
}

.faq-list details::details-content {
  block-size: 0;
  overflow: hidden;
  transition:
    block-size 0.3s ease,
    content-visibility 0.3s allow-discrete;
}

.faq-list details[open]::details-content {
  block-size: auto;
}
```

### Afprøvning og ændringer

- **Vi testede:** Vi testede tema-skiftet på `CoreValues.astro` på tværs af forsiden og `/about` samt åbning/lukning af spørgsmål i `Faq.astro`.
- **Vi observerede:** Først havde vi kun én mørk udgave af `CoreValues.astro`, og `<details>` i FAQ'en åbnede uden overgang, fordi CSS normalt ikke kan animere til `height: auto` / `block-size: auto`.
- **Vi ændrede eller mangler:** Vi tilføjede `variant`-proppen og `.light`-modifieren i `CoreValues.astro` og indførte `interpolate-size: allow-keywords` samt `::details-content` i `Faq.astro`.

## Eksempel 3: CSS Subgrid i layout og Size Container Query i `TeamCard.astro`

### Hvor og hvorfor?

Vi bruger CSS Subgrid i `src/layouts/Layout.astro` til sidens overordnede `.page-grid`, så alle direkte sektioner under `<main>` arver de samme `full`- og `content`-kolonner. Derudover bruger vi en size container query i `src/components/TeamCard.astro`, så medarbejderkortene skifter fra et lodret kort til et horisontalt grid-layout, når kortet får tildelt mindst `440px` i bredden (f.eks. i 2-kolonne visningen på `/about` vs. 3-kolonne visningen på `/team`).

### Relevant kode

```css
/* src/layouts/Layout.astro */
.page-grid {
  display: grid;
  grid-template-columns:
    [full-start] minmax(var(--space-md), 1fr)
    [content-start] min(100% - (var(--space-md) * 2), 1200px) [content-end]
    minmax(var(--space-md), 1fr) [full-end];
}

.page-grid > :global(section) {
  grid-column: full;
  display: grid;
  grid-template-columns: subgrid;
}

.page-grid > :global(section > *) {
  grid-column: content;
}
```

```css
/* src/components/TeamCard.astro */
.team-card {
  container-type: inline-size;
  container-name: team-card;
}

@container team-card (min-width: 440px) {
  .team-card-inner {
    display: grid;
    grid-template-columns: 180px 1fr;
    grid-template-rows: 1fr auto;
    column-gap: var(--space-lg);
    align-items: center;
  }
}
```

### Afprøvning og ændringer

- **Vi testede:** Vi testede sidens bredde ved at trække i browservinduet samt kortenes layout på `/team` og `/about`.
- **Vi observerede:** Flere sektioner (som `Hero.astro`, `Contact.astro`, `Experiences.astro` og `WhatToExpect.astro`) havde stadig lokale `max-width: 1200px`-begrænsninger, hvilket gav dobbelt indrykning oven i subgriddet. Derudover forsøgte vores første container query at ændre `padding` og `gap` direkte på container-elementet, hvilket ikke virker i CSS.
- **Vi ændrede eller mangler:** Vi slettede de lokale `max-width: 1200px`-regler i sektionerne, så subgriddet i `Layout.astro` styrer bredden alene, og vi flyttede `@container`-reglerne ned på det indre `.team-card-inner`-element i `TeamCard.astro`.

## Fallback og robusthed

- **Fallback/progressive enhancement:**
  - **Scroll-animationer:** Vi har testet siden i både Chrome (v130) og Firefox (v132). Scroll-animationerne på sektionerne og procentringene virker i Chrome (fra v115+), mens Firefox og Safari (v18) bare viser alting med det samme uden bevægelse, så man ikke går glip af noget indhold.
  - **Login-boks og sløret baggrund:** At login-boksen hænger fast lige under login-knappen virker i nyere Chrome og Edge (v125+). I Firefox (v132) åbner boksen i stedet bare midt på skærmen, og hvis en browser ikke kan sløre baggrunden, bliver den bare mørk i stedet, så man stadig nemt kan læse og bruge den.
  - **FAQ og team-kort:** Den glidende åbning i FAQ'en virker i nyere Chrome (v129+), mens Firefox (v132) og Safari (v18) bare åbner svarene med det samme uden animation. Team-kortene tilpasser sig pladsen i alle nyere browsere (Chrome v105+, Firefox v110+, Safari v16+), og i helt gamle browsere vises kortene bare i deres almindelige opstilling oven på hinanden.
- **Defensive CSS:** Vi har sørget for, at logoet og login-knappen ikke deler sig over to linjer på små skærme (`white-space: nowrap`), og at login-boksen automatisk tilpasser sin bredde på mobilen (`width: min(340px, 90vw)`), så siden ikke får vandret scroll.
- **Global CSS og komponent-CSS:** Farver, skrifttyper og det overordnede layout ligger samlet i `src/styles/main.css` og `src/layouts/Layout.astro`. Resten af CSS'en ligger inde i hver enkelt komponentfil, så vi ikke kommer til at ødelægge stylingen andre steder på siden.

## Brug af AI

- **Hvad brugte I den til?**
  Vi brugte AI som en sparringspartner, vi kunne spørge til råds, når vi sad fast i fejl, og til at tjekke op på regler i Popover API, CSS Anchor Positioning, container queries, `interpolate-size` og subgrid. Vi brugte den for eksempel, da hele vores header mistede sin styling, og da vores container query ikke slog igennem på selve container-elementet. Og til at hjælpe med at sætte vores refleksion op flot med markdown.

- **Hvad ændrede eller fravalgte I i svaret?**
  Vi sagde nej til flere af de hurtige forslag:
  - Da den foreslog bare at skjule links på mobil med `display: none`, droppede vi det, fordi man jo stadig skal kunne finde rundt på mobilen.
  - Den gav os et tip om, hvorfor vores header crashede, men vi rykkede selv `::backdrop` ud af nestingen bagefter og tilpassede koden til vores eget `<Button>`-komponent.

- **Hvad lærte I, og hvordan kontrollerede I løsningen?**
  Vi fandt ud af, at `::backdrop` ikke må nestes i Astro, at en `@container`-regel kun kan style børneelementer inde i containeren og ikke containeren selv, samt hvordan `@property`, `conic-gradient` og `offset-path` kan bruges til at bygge animerede doughnut-diagrammer i ren CSS. Vi tjekkede hele tiden koden selv ved at åbne Chrome DevTools og Firefox, teste forskellige skærmstørrelser og klikke rundt for at være sikre på, at det virkede i praksis.
