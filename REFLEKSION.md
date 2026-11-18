# Refleksion – Figma til kode

**Gruppemedlemmer:** Mikkel Ekhart Storm & Denzel Daniel Orbigo Padilla

## Eksempel 1: Popover API og CSS-nesting

### Hvor og hvorfor?

Vi bruger Popover API i `src/components/Header.astro` til login-boksen. Det smarte er, at vi slipper for JavaScript til at åbne og lukke vinduet, fordi browseren selv lukker den, når man klikker ved siden af eller trykker ESC. Vi brugte samtidig CSS-nesting for at holde stylingen samlet inde i komponenten.

### Relevant kode

```astro
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
```

```css
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

- **Vi testede:** Vi testede popoveren i Chrome på bærbar og i DevTools mobilview.
- **Vi observerede:** Al styling forsvandt helt på siden, fordi vi havde nestet `&::backdrop` inde i `#login-popover`, hvilket fik browseren til at smide hele stilarket ud.
- **Vi ændrede eller mangler:** Vi rykkede `#login-popover::backdrop` ud på sit eget rod-niveau i CSS'en, så stylingen slog igennem igen.

## Eksempel 2: Skriv navnet på det valgte benspænd her

### Hvor og hvorfor?

Hvor i løsningen bruger du teknikken, og hvilket konkret problem løser den? Henvis gerne til en fil, fx `src/components/MinKomponent.astro`.

### Relevant kode

Indsæt en kort kodeblok fra din løsning her. Vælg fx `astro`, `css` eller `js`.

```astro
// Din kode her
```

### Afprøvning og ændringer

- **Vi testede:** Beskriv situationen, fx en smal skærm, lang tekst eller tastaturbetjening.
- **Vi observerede:** Hvad skete der helt konkret under testen?
- **Vi ændrede eller mangler:** Hvad rettede du, eller hvad mangler stadig at blive fixet?

## Eksempel 3: CSS Subgrid i layout

### Hvor og hvorfor?

Vi bruger det i `src/layouts/Layout.astro` til sidens overordnede layout-grid. Det gør, at alle vores sektioner arver de samme kolonner, så overskrifter og tekst automatisk flugter ned igennem siden uden ekstra wrappers.

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

### Afprøvning og ændringer

- **Vi testede:** Vi testede sidens bredde ved at trække i browseren på computeren.
- **Vi observerede:** Teksten blev mast alt for langt ind mod midten, fordi vi både havde subgrid i layoutet og `max-width: 1200px` inde i sektionerne.
- **Vi ændrede eller mangler:** Vi slettede de lokale bredder og ekstra divs i sektionerne, så det er subgriddet alene, der styrer bredden.

## Fallback og robusthed

- **Fallback/progressive enhancement:** Vi bruger `backdrop-filter: blur(6px)` på popoverens baggrund. I browsere der ikke kan vise blur, vises den mørke farve (`rgba(0,0,0,0.45)`) bare i stedet. Vi har testet i Chrome og Firefox.
- **Defensive CSS:** Vi har sat `white-space: nowrap` på login-knappen, så teksten ikke deler sig over to linjer på små skærme, og `width: min(340px, 90vw)` på popoveren, så den ikke laver vandret scroll på mobiler.
- **Global CSS og komponent-CSS:** Vi har design-tokens og makrolayout liggende i `tokens.css` og `Layout.astro`. Al styling af selve komponenterne ligger lokalt inde i de enkelte `.astro`-filer, så de ikke forstyrrer hinanden.

## Brug af AI

- **Hvad brugte I den til?**
  Vi brugte Ai som en makker, vi kunne spørge til råds, når vi stødte på fejl, og til at tjekke op på regler i Popover API og subgrid f.eks.

- **Hvad ændrede eller fravalgte I i svaret?**
  Vi sagde nej til flere af de hurtige forslag:
  - Da AI foreslog bare at skjule links på mobil med `display: none`, afviste vi det, fordi folk på mobilen jo stadig skal kunne finde rundt.
  - Den gav os et tip om, hvorfor vores header crashede, men vi rettede og rykkede selv CSS'en ud af nestingen bagefter.

- **Hvad lærte I, og hvordan kontrollerede I løsningen?**
  Vi fandt ud af, at `::backdrop` ikke altid må nestes i Astro, og vi lærte, hvordan popover-attributter virker direkte i browseren uden JS. Vi tjekkede hele tiden koden selv ved at åbne Chrome DevTools, teste forskellige skærmstørrelser og klikke rundt for at sikre, at det virkede i praksis. Samt en masse om hvordan nesting fungere, og skal sættes op korrekt. 
