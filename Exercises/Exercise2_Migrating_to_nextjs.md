# Route Map & Component Tree

```
🟦 Server Component
🟨 Client Component

app/
├── layout.tsx 🟦                       # App-weites Layout mit Navbar, Footer – Server Component
├── page.tsx 🟦                         # Startseite (PLZ-Eingabe, Editionen-Auswahl) – Server Component
├── about/
│   └── page.tsx 🟦                     # Über uns – Server Component
├── profile/
│   ├── layout.tsx 🟦                   # Layout für geschützte Bereiche – Server Component
│   ├── (addresses)/
│   │   └── page.tsx 🟨                 # Nutzerprofil – Client Component (interaktiv, z. B. Adressbearbeitung)
│   ├── (abos)/
│   │   └── page.tsx 🟨                 # Übersicht Abo-Daten – Client Component (Formulare, Abo kündigen)
│   └── (password)/
│       └── page.tsx 🟨                 # Passwort ändern – Client Component (Formulare, Validierung)
├── login/
│   └── page.tsx 🟨                     # Login-Seite – Client Component (Interaktion, Validierung)
├── register/
│   └── page.tsx 🟨                     # Registrierung – Client Component (Interaktion, Validierung)
├── checkout/
│   ├── layout.tsx 🟦                   # Optionales Layout für Checkout-Bereich – Server Component
│   ├── page.tsx 🟨                     # Checkout-Seite – Client Component (Form, Zwischenspeicherung)
│   └── summary/
│       └── page.tsx 🟦                 # Bestellübersicht – Server Component
├── order/
│   ├── success/
│   │   └── page.tsx 🟦                 # Erfolgsseite – Server Component
│   └── summary/
│       └── page.tsx 🟦                 # Dynamische Bestellansicht – Server Component
├── products/
│   └── [product]/
│       ├── page.tsx 🟦                 # Produktdetailseite – Server Component (ggf. Client-Komponenten)
│       └── (modals)/
│           ├── ProductConfigModal.tsx 🟨  # Produktkonfiguration – Client Component (Modal, interaktiv)
│           └── AddressModal.tsx 🟨        # Adresseingabe – Client Component (Modal, Formular)
├── (modals)/
│   ├── login-modal.client.tsx 🟨       # Login-Modale – Client Component
│   └── register-modal.client.tsx 🟨    # Registrierungsmodale – Client Component
components/
├── Navbar.client.tsx 🟦
├── ThemeSwitch.client.tsx 🟦
├── FloatingAboButton.client.tsx 🟦
├── BackButton.tsx 🟦
├── footer.tsx 🟦
├── icons.tsx 🟦
├── primitives.ts
├── Stepper/
│   └── StepIndicator.client.tsx 🟦
store/
├── store.ts 
├── hook.ts 
lib/
├── api/
│   ├── api.ts 
│   ├── _Database.ts 
│   ├── Api.d.ts
│   └── Readme.txt
types/
├── types.ts
├── index.ts
styles/
├── globals.css

```

# Data-Fetching & Rendering Strategy

### Startseite
Rendering: SSG oder PPR

Begründung: Auswahl der Editionen + PLZ-Eingabe ist öffentlich, seltene Änderungen. Kann bei Build statisch generiert oder dynamisch hydratet werden, z. B. Editions-Daten.

### /about – Über uns
Rendering: SSG

Begründung: Reiner statischer Inhalt ohne Interaktivität oder benutzerbezogene Daten. Perfekt für statisches Rendering.

### /profile (Layout)
Rendering: SSR (Server Side Rendering)

Begründung: Layout erfordert Authentifizierung und ggf. Rollenprüfung. Muss auf Basis von Cookies/Session Tokens gerendert werden. 

### /profile/addresses – Nutzerprofil
Rendering: CSR (Client-Side Rendering)

Begründung: Starke Interaktivität (Adressverwaltung, Formulare), benötigt Nutzerkontext, keine SEO-Relevanz.

### /profile/abos – Abo-Verwaltung
Rendering: CSR (Client-Side Rendering)

Begründung: Interaktive Elemente (Kündigung, Anzeigen von Abo-Daten)
### /profile/password – Passwort ändern
Rendering: CSR

Begründung: Sensible, ausschließlich interaktive Operation, keine Daten zum Vorladen nötig, kein SEO-Bedarf.

### /login + /register
Rendering: CSR

Begründung: Formulare mit Validierung, keine externen Daten nötig, vollständig interaktiv

### /checkout – Bezahlseite
Rendering: CSR

Begründung: Eingabemaske, Zwischenspeichern von Formulardaten, Validierung – alles clientseitig, kein SEO relevant.

### /checkout/summary – Bestell-Zusammenfassung
Rendering: SSR

Begründung: Dynamischer Inhalt aus Datenbank, aber nach Abschluss relativ statisch

### /order/success
Rendering: SSR

Begründung: Einmaliger Seitenaufruf nach Bestellung, mit ggf. dynamischen Parametern (z. B. Order ID, Server-Validierung).

### /order/summary
Rendering: SSR

Begründung: Sensible, auf Nutzerkontext beruhende Bestelldaten. Kein statisches oder ISR-Rendering geeignet.

### /products/[product] – Produktdetailseite
Rendering: ISR mit Revalidate (z. B. alle 60 Sekunden)

Begründung: Produktinformationen ändern sich selten, aber regelmäßig. Schneller Seitenaufbau bei gleichzeitiger Aktualität.

### Modals wie ProductConfigModal.tsx, AddressModal.tsx
Rendering: CSR

Begründung: Interaktive Modale mit Auswahl- und Formlogik, ausschließlich clientseitig.

# SEO Metadata Plan

Die zentrale Einstiegsseite (/) enthält ein Formular zur PLZ-Eingabe.
Die verfügbaren Zeitungseditionen sind abhängig von der eingegebenen PLZ.
Daraus ergibt sich eine Herausforderung: Inhalte sind personalisiert, aber SEO-relevante Inhalte sollten möglichst statisch und indexierbar sein.

Daher: 
- Die primäre Einstiegsseite wird als statische Server Component gerendert. Enthält ein generisches Formular zur Auswahl der PLZ.

Metadaten werden per `export const metadata` statisch definiert. Die Description der statischen Metadaten soll dennoch möglichst den Scope der Seite vermitteln, sodass potentielle User den Konfigurator finden, wenn sie über Suchmaschinen nach lokalen Zeitungen suchen.

```
export const metadata = {
  title: "Zeitungsabos für Stuttgart und Umgebung",
  description: "Finden Sie verfügbare Zeitungsabonnements in Ihrer Region – einfach PLZ eingeben. Entecken Sie unsere Lokal-, Regional- und Sportausgaben für den Großraum Stuttgart!",
  openGraph: {
    title: "Zeitungsabos nach PLZ",
    description: "Individuelle Abos für Ihre Region entdecken.",
    url: "https://example.com/",
    type: "website",
  },
  twitter: {
    card: "summary_large_image",
    title: "Zeitungsabos für Stuttgart und Umgebung",
    description: "PLZ eingeben und regionale Zeitungen entdecken.",
  },
}
```

Die Produktseiten (/products/[product]) dagegen enthalten die wesentlichen Informationen und Metadaten zum jeweiligen Produkt selbst und sind daher hochrelevant für SEO.
Werden per SSG vorgerendert und sind unabhängig von der PLZ.
Für diesen Use-Case ist die Verwendung von generateMetadata({ params }) zur dynamischen Metadatengenerierung basierend auf dem Produkt-Slug vorgesehen.

Die Datei `app/robots.ts` steuert das Crawling der Startseite und Produktseiten. Profilseiten, sowie Produktübersichts- und Bestellübersichtsseiten sollen nicht indexiert werden.

`app/sitemap.xml` listet die relevantesten statischen Seiten wie `/`, `/about`, und `/products/*`.

In app/layout.tsx definierte export const metadata liefert globale Defaults.

Seiten wie `/products/[product]/page.tsx` nutzen `generateMetadata` zur produktspezifischen Überschreibung.
Modals und rein clientseitige Seiten erhalten keine Metadaten.

# Performance Considerations
### Route-Segment Code Splitting
Next.js lädt nur den für ein bestimmtes Route-Segment notwendigen JavaScript-Code - es werden also nur benötigte Unterseiten und deren Layouts initial in den Browser geladen. Dadurch verbessert sich die initiale Ladezeit. Beim Auswählen der PLZ und der Zeitungsauswahl wird beim Kunden also nicht der Code für die Profil-Seite und die Order-Summary/-Success- oder Checkout-Page geladen. Über Prefetching kann konkret gesteuert werden, welche Seiten/Routen bereits im Voraus nachgeladen werden, indem next/link verwendet wird. Beispiel: Auf der Startseite wird der Link zu einer Produktdetailseite vorab geladen, noch bevor der Nutzer klickt. Außerdem kann man durch gemeinsame Layouts für (z.B. app/layout.tsx) vom Layout-Caching profitieren, da dieses nur einmal initial geladen wird.

### Optimizations to hit Core-Web-Vitals
- Image Optimization: Verwendung von `next/image`. Angabe von festen Breiten und Höhen, um Cumulative Layout Shifts zu vermeiden.
- Verwendung von `next/fonts`, um das Laden/Preloading von Schriftarten zu optimieren. Dadurch läuft man keine Gefahr, dass die Schriftarten zuletzt geladen werden und sich die Seite nach dem Laden nochmal dahingehend verändert (LCP).
- Verwendung von Fallback-Layouts (z.B. für die Produkt-Detail-Seite) (Interaction to Next Paint)
- Verwendung von Suspense zum dynamischen Nachladen von Inhalten (z.B. Konfigurationsoptionen im ProductConfigModal (Interaction to Next Paint))

# Optional Bonus
Ein mögliches optionales Feature zur Verbesserung der Nutzererfahrung und Reichweite wäre die Implementierung eines dynamischen Social-Preview-Bildes. Beim Teilen der Startseite in sozialen Netzwerken wie Facebook oder Twitter könnten personalisierte Vorschau-Bilder angezeigt werden, die automatisch auf Basis der vom Nutzer eingegebenen Postleitzahl generiert werden. Dies ließe sich mit ImageResponse von Next.js umsetzen. Das generierte Bild könnte beispielsweise den regionalen Zeitungsnamen, ein Symbolbild der aktuellen Ausgabe sowie den gewählten PLZ-Bereich beinhalten. Diese dynamischen Vorschauen würden in der generateMetadata()-Funktion eingebunden und durch einen Route Handler (z. B. /api/og) erzeugt. Dies erhöht die Relevanz beim Teilen und steigert potenziell die Klickrate – insbesondere bei lokal ausgerichteten Kampagnen.

Ein weiteres sinnvolles Feature wäre der gezielte Einsatz einer Server Action im Checkout-Prozess. Sobald der Nutzer die Bestelldaten eingibt, könnte eine Server Action aufgerufen werden, die die Validierung, Speicherung der Bestellung sowie eventuelle Zahlungsabwicklung übernimmt. Nach erfolgreicher Verarbeitung würde der Nutzer direkt auf die Seite /order/success weitergeleitet. Die Vorteile dieser Herangehensweise liegen in der stärkeren Absicherung sensibler Daten, da sie nicht unnötig im Client verarbeitet werden müssen. Zudem vereinfacht sie die Architektur, da auf einen klassischen API-Endpunkt verzichtet werden kann. Durch die serverseitige Weiterleitung lassen sich unnötige Roundtrips vermeiden, was die User Experience verbessert. Die Server Action wäre typischerweise direkt in der page.tsx-Datei des checkout/-Segments implementiert und würde in einer Client Component verwendet werden.