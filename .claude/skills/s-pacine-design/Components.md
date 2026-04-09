# Pa' Cine -- Reusable Components

Copy and adapt these components when building new pages. They reflect the exact patterns used in the production app.

---

## Header

Dark navbar with the Pa' Cine brand logo (three red stripes + text) and a hamburger menu.

```tsx
"use client";

import { useState } from "react";
import Link from "next/link";
import { Menu, Film, Building2 } from "lucide-react";

export function Header() {
  const [menuOpen, setMenuOpen] = useState(false);

  return (
    <header className="bg-zinc-900">
      <div className="max-w-7xl mx-auto px-4 h-12 flex items-center justify-between">
        {/* Brand */}
        <Link href="/" className="flex items-center gap-2 hover:opacity-80 transition-opacity">
          <div className="flex items-center gap-0.5">
            <div className="w-2 h-8 bg-red-500 rounded-sm" />
            <div className="w-2 h-8 bg-red-500 rounded-sm" />
            <div className="w-2 h-8 bg-red-500 rounded-sm" />
          </div>
          <div>
            <div className="text-xl font-black tracking-tight leading-none text-white">
              PA&apos;<span className="text-red-500">CINE</span>
            </div>
            <div className="text-[10px] text-white tracking-wider uppercase">
              El cine en Colombia
            </div>
          </div>
        </Link>

        {/* Menu button */}
        <button
          type="button"
          aria-label="Abrir menú"
          onClick={() => setMenuOpen(!menuOpen)}
          className="p-2 text-white hover:text-zinc-300 transition-colors"
        >
          <Menu className="h-6 w-6" />
        </button>
      </div>

      {/* Mobile nav drawer (simplified -- production uses shadcn Sheet) */}
      {menuOpen && (
        <nav className="bg-zinc-900 border-t border-zinc-700 px-4 py-2 flex flex-col gap-1">
          <Link href="/" onClick={() => setMenuOpen(false)}
            className="flex items-center gap-3 px-3 py-2.5 rounded-lg text-white hover:bg-zinc-800 transition-colors">
            <Film className="h-5 w-5" /> Cartelera
          </Link>
          <Link href="/all_cines" onClick={() => setMenuOpen(false)}
            className="flex items-center gap-3 px-3 py-2.5 rounded-lg text-white hover:bg-zinc-800 transition-colors">
            <Building2 className="h-5 w-5" /> Todos los cines
          </Link>
        </nav>
      )}
    </header>
  );
}
```

Key patterns:
- `bg-zinc-900` background
- Height: `h-12`
- Brand logo: 3x `w-2 h-8 bg-red-500 rounded-sm` stripes
- Brand text: `text-xl font-black tracking-tight` with red accent on "CINE"
- Subtitle: `text-[10px] tracking-wider uppercase`
- Nav links: `px-3 py-2.5 rounded-lg hover:bg-zinc-800 transition-colors`

---

## Footer

Dark footer with Colombian flag heart, copyright, and legal links.

```tsx
import Link from "next/link";

export function Footer() {
  return (
    <footer className="bg-zinc-900 text-zinc-400 mt-auto">
      <div className="max-w-7xl mx-auto px-4 py-6">
        <div className="flex flex-col sm:flex-row items-center justify-between gap-4 text-sm">
          <p>
            Hecho con{" "}
            <svg viewBox="0 0 64 64" width="1.95em" height="1.95em"
              style={{ verticalAlign: "-0.35em", display: "inline" }}>
              <defs>
                <clipPath id="heartClip">
                  <path d="M32 56s-18-11.8-25.2-21.4C1.6 27.6 4 18.5 11.2 14.8c6.2-3.2 13.3-1.1 17 4.2
                    3.7-5.3 10.8-7.4 17-4.2C60 18.5 62.4 27.6 57.2 34.6 50 44.2 32 56 32 56z" />
                </clipPath>
              </defs>
              <g clipPath="url(#heartClip)">
                <rect x="0" y="0" width="64" height="26" fill="#FCD116" />
                <rect x="0" y="26" width="64" height="16" fill="#003893" />
                <rect x="0" y="42" width="64" height="22" fill="#CE1126" />
              </g>
              <path d="M32 56s-18-11.8-25.2-21.4C1.6 27.6 4 18.5 11.2 14.8c6.2-3.2 13.3-1.1 17 4.2
                3.7-5.3 10.8-7.4 17-4.2C60 18.5 62.4 27.6 57.2 34.6 50 44.2 32 56 32 56z"
                fill="none" stroke="currentColor" strokeWidth="2" opacity="0.25" />
            </svg>
            {" "}en Colombia
          </p>
          <p>&copy; {new Date().getFullYear()} Pa&apos; Cine</p>
          <nav className="flex gap-6">
            <Link href="/privacidad" className="hover:text-white transition-colors">
              Privacidad
            </Link>
            <Link href="/terminos" className="hover:text-white transition-colors">
              Términos
            </Link>
          </nav>
        </div>
      </div>
    </footer>
  );
}
```

Key patterns:
- `bg-zinc-900 text-zinc-400 mt-auto` (sticks to bottom with flex layout)
- Inner: `max-w-7xl mx-auto px-4 py-6`
- Links: `hover:text-white transition-colors`
- Colombian flag SVG heart with clip-path

---

## MovieCard

Poster card with image, title, and hover effects.

```tsx
import Image from "next/image";
import Link from "next/link";
import { useState } from "react";

interface MovieCardProps {
  movie: { slug: string; poster_url: string; title_es: string };
  singleLineTitle?: boolean;
}

export function MovieCard({ movie, singleLineTitle = false }: MovieCardProps) {
  const [isImageLoaded, setIsImageLoaded] = useState(false);
  const [hasImageError, setHasImageError] = useState(false);

  return (
    <div className="card-base overflow-hidden hover:shadow-md transition-shadow">
      <Link href={`/movies/${movie.slug}`}>
        <div className="aspect-[2/3] relative overflow-hidden bg-zinc-200">
          {movie.poster_url && !hasImageError ? (
            <>
              {!isImageLoaded && <div className="absolute inset-0 bg-zinc-200" />}
              <Image
                src={movie.poster_url}
                alt={movie.title_es}
                fill
                sizes="(max-width: 768px) 33vw, 16vw"
                onLoad={() => setIsImageLoaded(true)}
                onError={() => setHasImageError(true)}
                className={`object-cover hover:scale-105 transition-transform duration-300 ${
                  isImageLoaded ? "opacity-100" : "opacity-0"
                }`}
              />
            </>
          ) : (
            <div className="w-full h-full flex items-center justify-center text-zinc-400 text-xs sm:text-base">
              Sin imagen
            </div>
          )}
        </div>
        <div className="px-2 py-1 sm:px-4 sm:py-2">
          <h3
            title={movie.title_es}
            className={`font-semibold text-xs sm:text-base lg:text-lg text-zinc-900 mb-0.5 sm:mb-1 hover:text-red-500 transition-colors ${
              singleLineTitle ? "truncate whitespace-nowrap" : "line-clamp-2"
            }`}
          >
            {movie.title_es}
          </h3>
        </div>
      </Link>
    </div>
  );
}
```

Key patterns:
- `.card-base` for consistent card styling
- `aspect-[2/3]` for movie poster ratio
- Image loading state with opacity transition
- `hover:scale-105 transition-transform duration-300` on image
- `hover:text-red-500` on title
- Responsive text sizes: `text-xs sm:text-base lg:text-lg`

---

## Tabs

Active/inactive tab buttons used on the home page.

```tsx
<div role="tablist" className="flex items-center gap-4 pb-2">
  <button
    type="button"
    role="tab"
    aria-selected={activeTab === "cartelera"}
    onClick={() => setActiveTab("cartelera")}
    className={`pb-1 text-2xl font-black transition-colors border-b-[3px] ${
      activeTab === "cartelera"
        ? "text-zinc-900 border-red-500"
        : "text-zinc-300 border-transparent hover:text-zinc-500"
    }`}
  >
    Cartelera
  </button>
  <button
    type="button"
    role="tab"
    aria-selected={activeTab === "cines"}
    onClick={() => setActiveTab("cines")}
    className={`pb-1 text-2xl font-black transition-colors border-b-[3px] ${
      activeTab === "cines"
        ? "text-zinc-900 border-red-500"
        : "text-zinc-300 border-transparent hover:text-zinc-500"
    }`}
  >
    Cines
  </button>
</div>
```

Key patterns:
- `text-2xl font-black` for tab labels
- Active: `text-zinc-900 border-red-500 border-b-[3px]`
- Inactive: `text-zinc-300 border-transparent hover:text-zinc-500`

---

## Loading Spinner

Centered spinner used during data fetching.

```tsx
<div className="flex flex-col items-center justify-center py-20">
  <div className="h-10 w-10 animate-spin rounded-full border-4 border-zinc-300 border-t-red-500" />
  <p className="mt-4 text-sm text-zinc-500">Cargando...</p>
</div>
```

---

## Empty State

Centered message when no data is available.

```tsx
<div className="text-center py-16">
  <p className="text-zinc-500 text-lg">
    No hay resultados disponibles.
  </p>
</div>
```

---

## Page Shell

Standard page wrapper with Header, main content, and Footer.

```tsx
<div className="min-h-screen bg-zinc-100 flex flex-col">
  <Header />
  <main className="max-w-7xl mx-auto px-4 py-2 flex-1 w-full">
    {/* Page content */}
  </main>
  <Footer />
</div>
```

Key patterns:
- `min-h-screen flex flex-col` for full-height layout
- `flex-1` on main pushes footer to bottom
- `max-w-7xl mx-auto px-4` for content width
- `bg-zinc-100` page background
